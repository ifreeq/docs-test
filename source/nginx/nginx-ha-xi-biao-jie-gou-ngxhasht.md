# Nginx 哈希表结构 ngx\_hash\_t

## 概述

关于哈希表的基本知识在前面的文章《[数据结构-哈希表](http://blog.csdn.net/chenhanzhun/article/details/38091431)》已作介绍。哈希表结合了数组和链表的特点，使其寻址、插入以及删除操作更加方便。哈希表的过程是将关键字通过某种哈希函数映射到相应的哈希表位置，即对应的哈希值所在哈希表的位置。但是会出现多个关键字映射相同位置的情况导致冲突问题，为了解决这种情况，哈希表使用两个可选择的方法：**拉链法**和 **开放寻址法**。

Nginx 的哈希表中使用开放寻址来解决冲突问题，为了处理字符串，Nginx 还实现了支持通配符操作的相关函数，下面对 Nginx 中哈希表的源码进行分析。源码文件：src/core/ngx\_hash.h/.c。

## 哈希表结构

\*\*ngx\_hash\_elt\_t 结构 \*\*

哈希表中关键字元素的结构 ngx\_hash\_elt\_t，哈希表元素结构采用 键-值 形式，即&lt;key，value&gt; 。其定义如下：

```text

typedef struct {
    void             *value;    
    u_short           len;      
    u_char            name[1];  
} ngx_hash_elt_t;
```

**ngx\_hash\_t 结构**

哈希表基本结构 ngx\_hash\_t，其结构定义如下：

```text

typedef struct {
    ngx_hash_elt_t  **buckets;  
    ngx_uint_t        size;     
} ngx_hash_t;
```

元素结构图以及基本哈希结构图如下所示：

\*\*ngx\_hash\_init\_t 初始化结构 \*\*

哈希初始化结构 ngx\_hash\_init\_t，Nginx 的 hash 初始化结构是 ngx\_hash\_init\_t，用来将其相关数据封装起来作为参数传递给ngx\_hash\_init\(\)，其定义如下：

```text
typedef ngx_uint_t (*ngx_hash_key_pt) (u_char *data, size_t len);


typedef struct {
    ngx_hash_t       *hash;         
    ngx_hash_key_pt   key;          

    ngx_uint_t        max_size;     
    ngx_uint_t        bucket_size;  

    char             *name;         
    ngx_pool_t       *pool;         
    
    ngx_pool_t       *temp_pool;
} ngx_hash_init_t;
```

哈希元素数据 ngx\_hash\_key\_t，该结构也主要用来保存要 hash 的数据，即键-值对&lt;key,value&gt;，在实际使用中，一般将多个键-值对保存在 ngx\_hash\_key\_t 结构的数组中，作为参数传给ngx\_hash\_init\(\)。其定义如下：

```text

typedef struct {
    ngx_str_t         key;      
    ngx_uint_t        key_hash; 
    void             *value;    
} ngx_hash_key_t;
```

## 哈希操作

哈希操作包括初始化函数、查找函数；其中初始化函数是 Nginx 中哈希表比较重要的函数，由于 Nginx 的 hash 表是静态只读的，即不能在运行时动态添加新元素的，一切的结构和数据都在配置初始化的时候就已经规划完毕。

### 哈希函数

哈希表中使用哈希函数把用户数据映射到哈希表对应的位置中，下面是 Nginx 哈希函数的定义：

```text

#define ngx_hash(key, c)   ((ngx_uint_t) key * 31 + c)
ngx_uint_t ngx_hash_key(u_char *data, size_t len);
ngx_uint_t ngx_hash_key_lc(u_char *data, size_t len);
ngx_uint_t ngx_hash_strlow(u_char *dst, u_char *src, size_t n);
#define ngx_hash(key, c)   ((ngx_uint_t) key * 31 + c)

ngx_uint_t
ngx_hash_key(u_char *data, size_t len)
{
    ngx_uint_t  i, key;

    key = 0;

    for (i = 0; i < len; i++) {
        
        key = ngx_hash(key, data[i]);
    }

    return key;
}


ngx_uint_t
ngx_hash_key_lc(u_char *data, size_t len)
{
    ngx_uint_t  i, key;

    key = 0;

    for (i = 0; i < len; i++) {
        
        key = ngx_hash(key, ngx_tolower(data[i]));
    }

    return key;
}


ngx_uint_t
ngx_hash_strlow(u_char *dst, u_char *src, size_t n)
{
    ngx_uint_t  key;

    key = 0;

    while (n--) {
        *dst = ngx_tolower(*src);
        key = ngx_hash(key, *dst);
        dst++;
        src++;
    }

    return key;
}
```

### 哈希初始化函数

hash 初始化由 ngx\_hash\_init\(\) 函数完成，其 names 参数是 ngx\_hash\_key\_t 结构的数组，即键-值对 &lt;key,value&gt; 数组，nelts 表示该数组元素的个数。该函数初始化的结果就是将 names 数组保存的键-值对&lt;key,value&gt;，通过 hash 的方式将其存入相应的一个或多个 hash 桶\(即代码中的 buckets \)中。hash 桶里面存放的是 ngx\_hash\_elt\_t 结构的指针\(hash元素指针\)，该指针指向一个基本连续的数据区。该数据区中存放的是经 hash 之后的键-值对&lt;key',value'&gt;，即 ngx\_hash\_elt\_t 结构中的字段 &lt;name,value&gt;。每一个这样的数据区存放的键-值对&lt;key',value'&gt;可以是一个或多个。其定义如下：

```text
#define NGX_HASH_ELT_SIZE(name)                                               \
    (sizeof(void *) + ngx_align((name)->key.len + 2, sizeof(void *)))



ngx_int_t
ngx_hash_init(ngx_hash_init_t *hinit, ngx_hash_key_t *names, ngx_uint_t nelts)
{
    u_char          *elts;
    size_t           len;
    u_short         *test;
    ngx_uint_t       i, n, key, size, start, bucket_size;
    ngx_hash_elt_t  *elt, **buckets;

    for (n = 0; n < nelts; n++) {
        
        if (hinit->bucket_size < NGX_HASH_ELT_SIZE(&names[n]) + sizeof(void *))
        {
            ngx_log_error(NGX_LOG_EMERG, hinit->pool->log, 0,
                          "could not build the %s, you should "
                          "increase %s_bucket_size: %i",
                          hinit->name, hinit->name, hinit->bucket_size);
            return NGX_ERROR;
        }
    }

    
    test = ngx_alloc(hinit->max_size * sizeof(u_short), hinit->pool->log);
    if (test == NULL) {
        return NGX_ERROR;
    }

    
    bucket_size = hinit->bucket_size - sizeof(void *);

    
    start = nelts / (bucket_size / (2 * sizeof(void *)));
    start = start ? start : 1;

    
    if (hinit->max_size > 10000 && nelts && hinit->max_size / nelts < 100) {
        start = hinit->max_size - 1000;
    }

    
    for (size = start; size <= hinit->max_size; size++) {

        ngx_memzero(test, size * sizeof(u_short));

        for (n = 0; n < nelts; n++) {
            if (names[n].key.data == NULL) {
                continue;
            }

            
            key = names[n].key_hash % size;
            test[key] = (u_short) (test[key] + NGX_HASH_ELT_SIZE(&names[n]));

#if 0
            ngx_log_error(NGX_LOG_ALERT, hinit->pool->log, 0,
                          "%ui: %ui %ui \"%V\"",
                          size, key, test[key], &names[n].key);
#endif

            
            if (test[key] > (u_short) bucket_size) {
                goto next;
            }
        }

        
        goto found;

    next:

        continue;
    }

    ngx_log_error(NGX_LOG_WARN, hinit->pool->log, 0,
                  "could not build optimal %s, you should increase "
                  "either %s_max_size: %i or %s_bucket_size: %i; "
                  "ignoring %s_bucket_size",
                  hinit->name, hinit->name, hinit->max_size,
                  hinit->name, hinit->bucket_size, hinit->name);

found:

    
    for (i = 0; i < size; i++) {
        test[i] = sizeof(void *);
    }

    
    for (n = 0; n < nelts; n++) {
        if (names[n].key.data == NULL) {
            continue;
        }

        
        key = names[n].key_hash % size;
        test[key] = (u_short) (test[key] + NGX_HASH_ELT_SIZE(&names[n]));
    }

    len = 0;

    
    for (i = 0; i < size; i++) {
        if (test[i] == sizeof(void *)) {
            continue;
        }

        test[i] = (u_short) (ngx_align(test[i], ngx_cacheline_size));

        len += test[i];
    }

    
    if (hinit->hash == NULL) {
        hinit->hash = ngx_pcalloc(hinit->pool, sizeof(ngx_hash_wildcard_t)
                                             + size * sizeof(ngx_hash_elt_t *));
        if (hinit->hash == NULL) {
            ngx_free(test);
            return NGX_ERROR;
        }

        
        buckets = (ngx_hash_elt_t **)
                      ((u_char *) hinit->hash + sizeof(ngx_hash_wildcard_t));

    } else {
        buckets = ngx_pcalloc(hinit->pool, size * sizeof(ngx_hash_elt_t *));
        if (buckets == NULL) {
            ngx_free(test);
            return NGX_ERROR;
        }
    }

    
    elts = ngx_palloc(hinit->pool, len + ngx_cacheline_size);
    if (elts == NULL) {
        ngx_free(test);
        return NGX_ERROR;
    }

    elts = ngx_align_ptr(elts, ngx_cacheline_size);

    
    for (i = 0; i < size; i++) {
        if (test[i] == sizeof(void *)) {
            continue;
        }

        buckets[i] = (ngx_hash_elt_t *) elts;
        elts += test[i];

    }

    
    for (i = 0; i < size; i++) {
        test[i] = 0;
    }

    
    for (n = 0; n < nelts; n++) {
        if (names[n].key.data == NULL) {
            continue;
        }

        key = names[n].key_hash % size;
        elt = (ngx_hash_elt_t *) ((u_char *) buckets[key] + test[key]);

        elt->value = names[n].value;
        elt->len = (u_short) names[n].key.len;

        ngx_strlow(elt->name, names[n].key.data, names[n].key.len);

        
        test[key] = (u_short) (test[key] + NGX_HASH_ELT_SIZE(&names[n]));
    }

    
    for (i = 0; i < size; i++) {
        if (buckets[i] == NULL) {
            continue;
        }

        elt = (ngx_hash_elt_t *) ((u_char *) buckets[i] + test[i]);

        elt->value = NULL;
    }

    ngx_free(test);

    hinit->hash->buckets = buckets;
    hinit->hash->size = size;

#if 0

    for (i = 0; i < size; i++) {
        ngx_str_t   val;
        ngx_uint_t  key;

        elt = buckets[i];

        if (elt == NULL) {
            ngx_log_error(NGX_LOG_ALERT, hinit->pool->log, 0,
                          "%ui: NULL", i);
            continue;
        }

        while (elt->value) {
            val.len = elt->len;
            val.data = &elt->name[0];

            key = hinit->key(val.data, val.len);

            ngx_log_error(NGX_LOG_ALERT, hinit->pool->log, 0,
                          "%ui: %p \"%V\" %ui", i, elt, &val, key);

            elt = (ngx_hash_elt_t *) ngx_align_ptr(&elt->name[0] + elt->len,
                                                   sizeof(void *));
        }
    }

#endif

    return NGX_OK;
}
```

### 哈希查找函数

hash 查找操作由 ngx\_hash\_find\(\) 函数完成，查找时由 key 直接计算所在的 bucket，该 bucket 中保存其所在 ngx\_hash\_elt\_t 数据区的起始地址；然后根据长度判断并用 name 内容匹配，匹配成功，其 ngx\_hash\_elt\_t 结构的 value 字段即是所求。其定义如下：

```text

void *
ngx_hash_find(ngx_hash_t *hash, ngx_uint_t key, u_char *name, size_t len)
{
    ngx_uint_t       i;
    ngx_hash_elt_t  *elt;

#if 0
    ngx_log_error(NGX_LOG_ALERT, ngx_cycle->log, 0, "hf:\"%*s\"", len, name);
#endif

    
    elt = hash->buckets[key % hash->size];

    if (elt == NULL) {
        return NULL;
    }

    while (elt->value) {
        if (len != (size_t) elt->len) {
            goto next;
        }

        for (i = 0; i < len; i++) {
            if (name[i] != elt->name[i]) {
                goto next;
            }
        }

        
        return elt->value;

    next:

        elt = (ngx_hash_elt_t *) ngx_align_ptr(&elt->name[0] + elt->len,
                                               sizeof(void *));
        continue;
    }

    return NULL;
}

```

测试程序：

```text
#include <stdio.h>
#include "ngx_config.h"
#include "ngx_conf_file.h"
#include "nginx.h"
#include "ngx_core.h"
#include "ngx_string.h"
#include "ngx_palloc.h"
#include "ngx_array.h"
#include "ngx_hash.h"
volatile ngx_cycle_t  *ngx_cycle;
void ngx_log_error_core(ngx_uint_t level, ngx_log_t *log, ngx_err_t err, const char *fmt, ...) { }

static ngx_str_t names[] = {ngx_string("www.baidu.com"),
                            ngx_string("www.google.com.hk"),
                            ngx_string("www.github.com")};
static char* descs[] = {"baidu: 1","google: 2", "github: 3"};


int main()
{
    ngx_uint_t          k;
    ngx_pool_t*         pool;
    ngx_hash_init_t     hash_init;
    ngx_hash_t*         hash;
    ngx_array_t*        elements;
    ngx_hash_key_t*     arr_node;
    char*               find;
    int                 i;
    ngx_cacheline_size = 32;

    pool = ngx_create_pool(1024*10, NULL);

    
    hash = (ngx_hash_t*) ngx_pcalloc(pool, sizeof(hash));
    
    hash_init.hash      = hash;                      
    hash_init.key       = &ngx_hash_key_lc;          
    hash_init.max_size  = 1024*10;                   
    hash_init.bucket_size = 64;
    hash_init.name      = "test_hash_error";
    hash_init.pool           = pool;
    hash_init.temp_pool      = NULL;

    

    elements = ngx_array_create(pool, 32, sizeof(ngx_hash_key_t));
    for(i = 0; i < 3; i++) {
        arr_node            = (ngx_hash_key_t*) ngx_array_push(elements);
        arr_node->key       = (names[i]);
        arr_node->key_hash  = ngx_hash_key_lc(arr_node->key.data, arr_node->key.len);
        arr_node->value     = (void*) descs[i];

        printf("key: %s , key_hash: %u\n", arr_node->key.data, arr_node->key_hash);
    }

    
    if (ngx_hash_init(&hash_init, (ngx_hash_key_t*) elements->elts, elements->nelts) != NGX_OK){
        return 1;
    }

    
    k    = ngx_hash_key_lc(names[0].data, names[0].len);
    printf("%s key is %d\n", names[0].data, k);
    find = (char*)
        ngx_hash_find(hash, k, (u_char*) names[0].data, names[0].len);

    if (find) {
        printf("get desc: %s\n",(char*) find);
    }

    ngx_array_destroy(elements);
    ngx_destroy_pool(pool);

    return 0;
}

```

输出结果：

```text
 ./hash_test 
key: www.baidu.com , key_hash: 270263191
key: www.google.com.hk , key_hash: 2472785358
key: www.github.com , key_hash: 2818415021
www.baidu.com key is 270263191
get desc: baidu: 1

```

参考资料：

《深入理解 Nginx 》

《 [nginx中hash表的设计与实现](http://blog.csdn.net/brainkick/article/details/7816420)》

《[Nginx 代码研究](http://code.google.com/p/nginxsrp/wiki/NginxCodeReview#ngx_hash)》

