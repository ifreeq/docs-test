# Nginx 队列双向链表结构 ngx\_queue\_t

## 队列链表结构

队列双向循环链表实现文件：文件：src/core/ngx\_queue.h/.c。在 Nginx 的队列实现中，实质就是具有头节点的双向循环链表，这里的双向链表中的节点是没有数据区的，只有两个指向节点的指针。需注意的是队列链表的内存分配不是直接从内存池分配的，即没有进行内存池管理，而是需要我们自己管理内存，所有我们可以指定它在内存池管理或者直接在堆里面进行管理，最好使用内存池进行管理。节点结构定义如下：

```text

typedef struct ngx_queue_s  ngx_queue_t;


struct ngx_queue_s {
    ngx_queue_t  *prev;
    ngx_queue_t  *next;
};
```

## 队列链表操作

其基本操作如下：

```text

ngx_queue_int(h)


ngx_queue_empty(h)


ngx_queue_insert_head(h, x)


ngx_queue_insert_tail(h, x)


ngx_queue_head(h)


ngx_queue_last(h)


ngx_queue_sentinel(h)


ngx_queue_remove(x)


ngx_queue_split(h, q, n)


ngx_queue_add(h, n)


ngx_queue_middle(h)


ngx_queue_sort(h, cmpfunc)


ngx_queue_next(q)


ngx_queue_prev(q)


ngx_queue_data(q, type, link)


ngx_queue_insert_after(q, x)

```

下面是队列链表操作源码的实现：

### 初始化链表

```text

#define ngx_queue_init(q)                                                     \
    (q)->prev = q;                                                            \
    (q)->next = q


#define ngx_queue_empty(h)                                                    \
    (h == (h)->prev)
```

### 获取指定的队列链表中的节点

```text

#define ngx_queue_head(h)                                                     \
    (h)->next


#define ngx_queue_last(h)                                                     \
    (h)->prev

#define ngx_queue_sentinel(h)                                                 \
    (h)


#define ngx_queue_next(q)                                                     \
    (q)->next


#define ngx_queue_prev(q)                                                     \
    (q)->prev
```

### 插入节点

在头节点之后插入新节点：

```text

#define ngx_queue_insert_head(h, x)                                           \
    (x)->next = (h)->next;                                                    \
    (x)->next->prev = x;                                                      \
    (x)->prev = h;                                                            \
    (h)->next = x

```

插入节点比较简单，只是修改指针的指向即可。下图是插入节点的过程：注意：虚线表示断开连接，实线表示原始连接，破折线表示重新连接，图中的数字与源码步骤相对应。

在尾节点之后插入节点

```text

#define ngx_queue_insert_tail(h, x)                                           \
    (x)->prev = (h)->prev;                                                    \
    (x)->prev->next = x;                                                      \
    (x)->next = h;                                                            \
    (h)->prev = x
```

下图是插入节点的过程：

### 删除节点

删除指定的节点，删除节点只是修改相邻节点指针的指向，并没有实际将该节点的内存释放，内存释放必须由我们进行处理。

```text
#if (NGX_DEBUG)

#define ngx_queue_remove(x)                                                   \
    (x)->next->prev = (x)->prev;                                              \
    (x)->prev->next = (x)->next;                                              \
    (x)->prev = NULL;                                                         \
    (x)->next = NULL

#else

#define ngx_queue_remove(x)                                                   \
    (x)->next->prev = (x)->prev;                                              \
    (x)->prev->next = (x)->next

#endif
```

删除节点过程如下图所示：

### 拆分链表

```text

#define ngx_queue_split(h, q, n)                                              \
    (n)->prev = (h)->prev;                                                    \
    (n)->prev->next = n;                                                      \
    (n)->next = q;                                                            \
    (h)->prev = (q)->prev;                                                    \
    (h)->prev->next = h;                                                      \
    (q)->prev = n;
```

该宏有 3 个参数，h 为队列头\(即链表头指针\)，将该队列从 q 节点将队列\(链表\)拆分为两个队列\(链表\)，q 之后的节点组成的新队列的头节点为 n 。链表拆分过程如下图所示：

### 合并链表

```text

#define ngx_queue_add(h, n)                                                   \
    (h)->prev->next = (n)->next;                                              \
    (n)->next->prev = (h)->prev;                                              \
    (h)->prev = (n)->prev;                                                    \
    (h)->prev->next = h;                                                      \
    (n)->prev = (n)->next = n;

```

其中，h、n分别为两个队列的指针，即头节点指针，该操作将n队列链接在h队列之后。具体操作如下图所示：

### 获取中间节点

```text

ngx_queue_t *
ngx_queue_middle(ngx_queue_t *queue)
{
    ngx_queue_t  *middle, *next;

    
    middle = ngx_queue_head(queue);

    
    if (middle == ngx_queue_last(queue)) {
        return middle;
    }

    
    next = ngx_queue_head(queue);

    for ( ;; ) {
        
        middle = ngx_queue_next(middle);

        next = ngx_queue_next(next);

        
        if (next == ngx_queue_last(queue)) {
            return middle;
        }

        next = ngx_queue_next(next);

        
        if (next == ngx_queue_last(queue)) {
            return middle;
        }
    }
}
```

### 链表排序

队列链表排序采用的是稳定的简单插入排序方法，即从第一个节点开始遍历，依次将当前节点\(q\)插入前面已经排好序的队列\(链表\)中，下面程序中，前面已经排好序的队列的尾节点为prev。操作如下：

```text



void
ngx_queue_sort(ngx_queue_t *queue,
    ngx_int_t (*cmp)(const ngx_queue_t *, const ngx_queue_t *))
{
    ngx_queue_t  *q, *prev, *next;

    q = ngx_queue_head(queue);

    
    if (q == ngx_queue_last(queue)) {
        return;
    }

    
    for (q = ngx_queue_next(q); q != ngx_queue_sentinel(queue); q = next) {

        prev = ngx_queue_prev(q);
        next = ngx_queue_next(q);

        
        ngx_queue_remove(q);

        
        do {
            if (cmp(prev, q) <= 0) {
                break;
            }

            prev = ngx_queue_prev(prev);

        } while (prev != ngx_queue_sentinel(queue));

        
        ngx_queue_insert_after(prev, q);
    }
}
```

### 获取队列中节点数据地址

由队列基本结构和以上操作可知，nginx 的队列操作只对链表指针进行简单的修改指向操作，并不负责节点数据空间的分配。因此，用户在使用nginx队列时，要自己定义数据结构并分配空间，且在其中包含一个 ngx\_queue\_t 的指针或者对象，当需要获取队列节点数据时，使用ngx\_queue\_data宏，其定义如下：

```text

#define ngx_queue_data(q, type, link)                                         \
    (type *) ((u_char *) q - offsetof(type, link))
```

测试程序：

```text
#include <stdio.h>
#include "ngx_queue.h"
#include "ngx_conf_file.h"
#include "ngx_config.h"
#include "ngx_palloc.h"
#include "nginx.h"
#include "ngx_core.h"

#define MAX     10
typedef struct Score
{
    unsigned int score;
    ngx_queue_t Que;
}ngx_queue_score;
volatile ngx_cycle_t  *ngx_cycle;

void ngx_log_error_core(ngx_uint_t level, ngx_log_t *log, ngx_err_t err,  
            const char *fmt, ...)
{
}

ngx_int_t CMP(const ngx_queue_t *x, const ngx_queue_t *y)
{
    ngx_queue_score *xinfo = ngx_queue_data(x, ngx_queue_score, Que);
    ngx_queue_score *yinfo = ngx_queue_data(y, ngx_queue_score, Que);

    return(xinfo->score > yinfo->score);
}

void print_ngx_queue(ngx_queue_t *queue)
{
    ngx_queue_t *q = ngx_queue_head(queue);

    printf("score: ");
    for( ; q != ngx_queue_sentinel(queue); q = ngx_queue_next(q))
    {
        ngx_queue_score *ptr = ngx_queue_data(q, ngx_queue_score, Que);
        if(ptr != NULL)
            printf(" %d\t", ptr->score);
    }
    printf("\n");
}

int main()
{
    ngx_pool_t *pool;
    ngx_queue_t *queue;
    ngx_queue_score *Qscore;

    pool = ngx_create_pool(1024, NULL);

    queue = ngx_palloc(pool, sizeof(ngx_queue_t));
    ngx_queue_init(queue);

    int i;
    for(i = 1; i < MAX; i++)
    {
        Qscore = (ngx_queue_score*)ngx_palloc(pool, sizeof(ngx_queue_score));
        Qscore->score = i;
        ngx_queue_init(&Qscore->Que);

        if(i%2)
        {
            ngx_queue_insert_tail(queue, &Qscore->Que);
        }
        else
        {
            ngx_queue_insert_head(queue, &Qscore->Que);
        }
    }

    printf("Before sort: ");
    print_ngx_queue(queue);

    ngx_queue_sort(queue, CMP);

    printf("After sort: ");
    print_ngx_queue(queue);

    ngx_destroy_pool(pool);
    return 0;

}

```

输出结果：

```text
$./queue_test 
Before sort: score:  8	 6	 4	 2	 1	 3	 5	 7	 9	
After sort: score:  1	 2	 3	 4	 5	 6	 7	 8	 9	

```

## 总结

在 Nginx 的队列链表中，其维护的是指向链表节点的指针，并没有实际的数据区，所有对实际数据的操作需要我们自行操作，队列链表实质是双向循环链表，其操作是双向链表的基本操作。

