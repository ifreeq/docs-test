# Nginx 内存池管理

## 概述

Nginx 使用内存池对内存进行管理，内存管理的实现类似于前面文章介绍的《[STL源码剖析——空间配置器](http://blog.csdn.net/chenhanzhun/article/details/39153797#t7)》，把内存分配归结为**大内存分配**和**小内存分配**。若申请的内存大小比同页的内存池最大值 max 还大，则是大内存分配，否则为小内存分配。

1. 大块内存的分配请求不会直接在内存池上分配内存来满足请求，而是直接向系统申请一块内存（就像直接使用 malloc 分配内存一样），然后将这块内存挂到内存池头部的 large 字段下。
2. 小块内存分配，则是从已有的内存池数据区中分配出一部分内存。

Nginx 内存管理相关文件：

1. src/os/unix/ngx\_alloc.h/.c

* 内存相关的操作，封装了最基本的内存分配函数。
* 如 free / malloc / memalign / posix\_memalign，分别被封装为 ngx\_free，ngx\_alloc / ngx\_calloc, ngx\_memalign
* ngx\_alloc：封装malloc分配内存
* ngx\_calloc：封装malloc分配内存，并初始化空间内容为0
* ngx\_memalign：返回基于一个指定 alignment 的大小为 size 的内存空间，且其地址为 alignment 的整数倍，alignment 为2的幂。

1. src/core/ngx\_palloc.h/.c

* 封装创建/销毁内存池，从内存池分配空间等函数。

Nginx 内存分配总流图如下：**其中 size 是用户请求分配内存的大小，pool是现有内存池。**

## 内存池基本结构

Nginx 内存池基本结构定义如下：

```text


typedef struct {
    u_char               *last; 
    u_char               *end;  
    ngx_pool_t           *next; 
    ngx_uint_t            failed;
} ngx_pool_data_t;  

struct ngx_pool_s {
    ngx_pool_data_t       d;    
    size_t                max;  
    ngx_pool_t           *current;
    ngx_chain_t          *chain;
    ngx_pool_large_t     *large;
    ngx_pool_cleanup_t   *cleanup;
    ngx_log_t            *log;
};

typedef struct ngx_pool_s   ngx_pool_t;
typedef struct ngx_chain_s  ngx_chain_t;

```

大块内存分配的数据结构如下：

```text
typedef struct ngx_pool_large_s ngx_pool_large_t;  

struct ngx_pool_large_s{  
          ngx_pool_large_t  *next;    
          void    *alloc;             
};  
```

其他数据结构如下：

```text
typedef void (*ngx_pool_cleanup_pt)(void *data);    

typedef struct ngx_pool_cleanup_s ngx_pool_cleanup_t;  
 
struct ngx_pool_cleanup_s{  
    ngx_pool_cleanup_pt handler;  
    void    *data;              
    ngx_pool_cleanup_t *next;   
};  
  
 
typedef struct {  
    ngx_fd_t   fd;  
    u_char    *name;  
    ngx_log_t *log;  
} ngx_pool_cleanup_file_t;  
```

内存池基本机构之间的关系如下图所示：

**ngx\_pool\_t 的逻辑结构**

上面数据结构之间逻辑结构图如下：**该图是采用 UML 画的，第一行黑色粗体表示对应数据结构，第二行是结构内的成员，冒号左边是变量，冒号右边是变量的类型；**

## 内存池的操作

## **创建内存池**

```text

ngx_pool_t *
ngx_create_pool(size_t size, ngx_log_t *log)
{
    ngx_pool_t  *p; 

    
    
    p = ngx_memalign(NGX_POOL_ALIGNMENT, size, log);
    if (p == NULL) {
        return NULL;
    }

    

    p->d.last = (u_char *) p + sizeof(ngx_pool_t);
    p->d.end = (u_char *) p + size;
    p->d.next = NULL;
    p->d.failed = 0;

    size = size - sizeof(ngx_pool_t);   
    
    p->max = (size < NGX_MAX_ALLOC_FROM_POOL) ? size : NGX_MAX_ALLOC_FROM_POOL;

    p->current = p; 
    p->chain = NULL;
    p->large = NULL;
    p->cleanup = NULL;
    p->log = log;

    return p;
}

```

其中内存分配函数 ngx\_memalign 定义如下：

```text
void *  
ngx_memalign(size_t alignment, size_t size, ngx_log_t *log)  
{  
    void  *p;  
    int    err;  
      
    err = posix_memalign(&p, alignment, size);  
    
      
    if (err) {  
        ngx_log_error(NGX_LOG_EMERG, log, err,  
            "posix_memalign(%uz, %uz) failed", alignment, size);  
        p = NULL;  
    }  
      
    ngx_log_debug3(NGX_LOG_DEBUG_ALLOC, log, 0,  
        "posix_memalign: %p:%uz @%uz", p, size, alignment);  
      
    return p;  
}  

#define NGX_POOL_ALIGNMENT       16  
```

## 销毁内存池

销毁内存池由  void ngx\_destroy\_pool\(ngx\_pool\_t \*pool\) 函数完成。该函数将遍历内存池链表，释放所有内存，如果注册了clenup \(也是一个链表结构\)，亦将遍历该 cleanup 链表结构依次调用 clenup 的 handler 清理。同时，还将遍历 large 链表，释放大块内存。

```text


void
ngx_destroy_pool(ngx_pool_t *pool)
{
    ngx_pool_t          *p, *n;
    ngx_pool_large_t    *l;
    ngx_pool_cleanup_t  *c;

    
    for (c = pool->cleanup; c; c = c->next) {
        if (c->handler) {
            ngx_log_debug1(NGX_LOG_DEBUG_ALLOC, pool->log, 0,
                           "run cleanup: %p", c);
            c->handler(c->data);
        }
    }

    
    for (l = pool->large; l; l = l->next) {

        ngx_log_debug1(NGX_LOG_DEBUG_ALLOC, pool->log, 0, "free: %p", l->alloc);

        if (l->alloc) {
            ngx_free(l->alloc); 
        }
    }

    
#if (NGX_DEBUG)

    

    for (p = pool, n = pool->d.next; ; p = n, n = n->d.next) {
        ngx_log_debug2(NGX_LOG_DEBUG_ALLOC, pool->log, 0,
                       "free: %p, unused: %uz", p, p->d.end - p->d.last);

        if (n == NULL) {
            break;
        }
    }

#endif

    
    for (p = pool, n = pool->d.next; ; p = n, n = n->d.next) {
        ngx_free(p);

        if (n == NULL) {
            break;
        }
    }
}

```

## 重置内存池

重置内存池由  void ngx\_reset\_pool\(ngx\_pool\_t \*pool\) 函数完成。该函数将释放所有 large 内存，并且将 d-&gt;last 指针重新指向 ngx\_pool\_t 结构之后数据区的开始位置，使内存池恢复到刚创建时的位置。由于内存池刚被创建初始化时是不包含大块内存的，所以必须释放大块内存。

```text

void
ngx_reset_pool(ngx_pool_t *pool)
{
    ngx_pool_t        *p;
    ngx_pool_large_t  *l;

    
    for (l = pool->large; l; l = l->next) {
        if (l->alloc) {
            ngx_free(l->alloc);
        }
    }

    for (p = pool; p; p = p->d.next) {
        p->d.last = (u_char *) p + sizeof(ngx_pool_t);
        p->d.failed = 0;
    }

    pool->current = pool;
    pool->chain = NULL;
    pool->large = NULL;
}

```

## 内存分配

## 小块内存分配

小块内存分配，即请求分配空间 size 小于内存池最大内存值 max。小内存分配的接口函数如下所示：

```text
void *ngx_palloc(ngx_pool_t *pool, size_t size);
void *ngx_pnalloc(ngx_pool_t *pool, size_t size);
void *ngx_pcalloc(ngx_pool_t *pool, size_t size);
void *ngx_pmemalign(ngx_pool_t *pool, size_t size, size_t alignment);
```

ngx\_palloc 和 ngx\_pnalloc 都是从内存池里分配 size 大小内存。他们的不同之处在于，palloc 取得的内存是对齐的，pnalloc 则不考虑内存对齐问题。ngx\_pcalloc 是直接调用 palloc 分配内存，然后进行一次 0 初始化操作。ngx\_pmemalign 将在分配 size 大小的内存并按 alignment 对齐，然后挂到 large 字段下，当做大块内存处理。

ngx\_palloc的过程一般为，首先判断待分配的内存是否大于 pool-&gt;max，如果大于则使用 ngx\_palloc\_large 在 large 链表里分配一段内存并返回， 如果小于测尝试从链表的 pool-&gt;current 开始遍历链表，尝试找出一个可以分配的内存，当链表里的任何一个节点都无法分配内存的时候，就调用 ngx\_palloc\_block 生成链表里一个新的节点， 并在新的节点里分配内存并返回， 同时， 还会将pool-&gt;current 指针指向新的位置（从链表里面pool-&gt;d.failed小于等于4的节点里找出） 。

```text


void *
ngx_palloc(ngx_pool_t *pool, size_t size)
{
    u_char      *m;
    ngx_pool_t  *p;

    
    if (size <= pool->max) {

        p = pool->current;

        do {
            
            m = ngx_align_ptr(p->d.last, NGX_ALIGNMENT);

            
            if ((size_t) (p->d.end - m) >= size) {
                p->d.last = m + size;   

                return m;
            }

            
            p = p->d.next;

        } while (p);

        
        return ngx_palloc_block(pool, size);
    }

    
    return ngx_palloc_large(pool, size);
}

static void *
ngx_palloc_block(ngx_pool_t *pool, size_t size)
{
    u_char      *m;
    size_t       psize;
    ngx_pool_t  *p, *new, *current;

    
    psize = (size_t) (pool->d.end - (u_char *) pool);

    
    m = ngx_memalign(NGX_POOL_ALIGNMENT, psize, pool->log);
    if (m == NULL) {
        return NULL;
    }
    
    new = (ngx_pool_t *) m;
    new->d.end = m + psize;
    new->d.next = NULL;
    new->d.failed = 0;
    
    
    m += sizeof(ngx_pool_data_t);
    
    m = ngx_align_ptr(m, NGX_ALIGNMENT);
    new->d.last = m + size;

    current = pool->current;
    for (p = current; p->d.next; p = p->d.next) {
        if (p->d.failed++ > 4) {
            
            current = p->d.next;
        }
    }

    
    p->d.next = new;

    
    pool->current = current ? current : new;

    return m;
}

```

```text

void *
ngx_pcalloc(ngx_pool_t *pool, size_t size)
{
    void *p;

    p = ngx_palloc(pool, size);
    if (p) {
        ngx_memzero(p, size);
    }

    return p;
}


void *
ngx_pmemalign(ngx_pool_t *pool, size_t size, size_t alignment)
{
    void              *p;
    ngx_pool_large_t  *large;

    p = ngx_memalign(alignment, size, pool->log);
    if (p == NULL) {
        return NULL;
    }

    large = ngx_palloc(pool, sizeof(ngx_pool_large_t));
    if (large == NULL) {
        ngx_free(p);
        return NULL;
    }

    large->alloc = p;
    large->next = pool->large;
    pool->large = large;

    return p;
}

```

小内存分配之后如下图所示：

上图是由3个小内存池组成的内存池模型，由于第一个内存池上剩余的内存不够分配了，于是就创建了第二个新的内存池，第三个内存池是由于前面两个内存池的剩余部分都不够分配，所以创建了第三个内存池来满足用户的需求。由图可见：所有的小内存池是由一个单向链表维护在一起的。这里还有两个字段需要关注，failed和current字段。failed表示的是当前这个内存池的剩余可用内存不能满足用户分配请求的次数，即是说：一个分配请求到来后，在这个内存池上分配不到想要的内存，那么就failed就会增加1；这个分配请求将会递交给下一个内存池去处理，如果下一个内存池也不能满足，那么它的failed也会加1，然后将请求继续往下传递，直到满足请求为止（如果没有现成的内存池来满足，会再创建一个新的内存池）。current字段会随着failed的增加而发生改变，如果current指向的内存池的failed达到了4的话，current就指向下一个内存池了。

## 大块内存分配

```text

static void *
ngx_palloc_large(ngx_pool_t *pool, size_t size)
{
    void              *p;
    ngx_uint_t         n;
    ngx_pool_large_t  *large;

    
    p = ngx_alloc(size, pool->log);
    if (p == NULL) {
        return NULL;
    }

    n = 0;

    
    for (large = pool->large; large; large = large->next) {
        if (large->alloc == NULL) {
            large->alloc = p;
            return p;
        }

        if (n++ > 3) {
            break;
        }
    }

    
    large = ngx_palloc(pool, sizeof(ngx_pool_large_t));
    if (large == NULL) {
        ngx_free(p);
        return NULL;
    }

    large->alloc = p;
    large->next = pool->large;
    pool->large = large;

    return p;
}

void *
ngx_alloc(size_t size, ngx_log_t *log)
{
    void  *p;

    p = malloc(size);

    if (p == NULL) {
        ngx_log_error(NGX_LOG_EMERG, log, ngx_errno,
            "malloc() %uz bytes failed", size);
    }

    ngx_log_debug2(NGX_LOG_DEBUG_ALLOC, log, 0, "malloc: %p:%uz", p, size);
    return p;
}


ngx_int_t
ngx_pfree(ngx_pool_t *pool, void *p)
{
    ngx_pool_large_t  *l;

    for (l = pool->large; l; l = l->next) {
        if (p == l->alloc) {
            ngx_log_debug1(NGX_LOG_DEBUG_ALLOC, pool->log, 0,
                           "free: %p", l->alloc);
            ngx_free(l->alloc);
            l->alloc = NULL;

            return NGX_OK;
        }
    }

    return NGX_DECLINED;
}

```

大块内存申请之后如下所示：

## cleanup 资源

```text

ngx_pool_cleanup_t * ngx_pool_cleanup_add(ngx_pool_t *p, size_t size);


 void ngx_pool_run_cleanup_file(ngx_pool_t *p, ngx_fd_t fd);


 void ngx_pool_cleanup_file(void *data);


 void ngx_pool_delete_file(void *data);

 
ngx_pool_cleanup_t *
ngx_pool_cleanup_add(ngx_pool_t *p, size_t size)
{
    ngx_pool_cleanup_t  *c;

    c = ngx_palloc(p, sizeof(ngx_pool_cleanup_t));
    if (c == NULL) {
        return NULL;
    }

    if (size) {
        c->data = ngx_palloc(p, size);
        if (c->data == NULL) {
            return NULL;
        }

    } else {
        c->data = NULL;
    }

    c->handler = NULL;
    c->next = p->cleanup;

    p->cleanup = c;

    ngx_log_debug1(NGX_LOG_DEBUG_ALLOC, p->log, 0, "add cleanup: %p", c);

    return c;
}


void
ngx_pool_run_cleanup_file(ngx_pool_t *p, ngx_fd_t fd)
{
    ngx_pool_cleanup_t       *c;
    ngx_pool_cleanup_file_t  *cf;

    
    for (c = p->cleanup; c; c = c->next) {
        if (c->handler == ngx_pool_cleanup_file) {

            cf = c->data;

            if (cf->fd == fd) {
                c->handler(cf);
                c->handler = NULL;
                return;
            }
        }
    }
}


void
ngx_pool_cleanup_file(void *data)
{
    ngx_pool_cleanup_file_t  *c = data; 

    ngx_log_debug1(NGX_LOG_DEBUG_ALLOC, c->log, 0, "file cleanup: fd:%d",
                   c->fd);

    
    if (ngx_close_file(c->fd) == NGX_FILE_ERROR) {
        ngx_log_error(NGX_LOG_ALERT, c->log, ngx_errno,
                      ngx_close_file_n " \"%s\" failed", c->name);
    }
}


void
ngx_pool_delete_file(void *data)
{
    ngx_pool_cleanup_file_t  *c = data;

    ngx_err_t  err;

    ngx_log_debug2(NGX_LOG_DEBUG_ALLOC, c->log, 0, "file cleanup: fd:%d %s",
                   c->fd, c->name);

    
    if (ngx_delete_file(c->name) == NGX_FILE_ERROR) {
        err = ngx_errno;

        if (err != NGX_ENOENT) {
            ngx_log_error(NGX_LOG_CRIT, c->log, err,
                          ngx_delete_file_n " \"%s\" failed", c->name);
        }
    }

    
    if (ngx_close_file(c->fd) == NGX_FILE_ERROR) {
        ngx_log_error(NGX_LOG_ALERT, c->log, ngx_errno,
                      ngx_close_file_n " \"%s\" failed", c->name);
    }
}

```

参考资料：

《 [Nginx源码剖析之内存池，与内存管理](http://blog.csdn.net/v_july_v/article/details/7040425)》

《[nginx源码分析—内存池结构ngx\_pool\_t及内存管理](http://blog.csdn.net/livelylittlefish/article/details/6586946)》

《[Nginx内存池实现源码分析](http://blog.chinaunix.net/uid-24830931-id-3764858.html) 》

《[Nginx源码分析-内存池](http://www.alidata.org/archives/1390)》

《[Ningx代码研究](https://code.google.com/p/nginxsrp/wiki/NginxCodeReview)》

