# Nginx 定时器事件

## 概述

在 Nginx 中定时器事件的实现与内核无关。在事件模块中，当等待的事件不能在指定的时间内到达，则会触发Nginx 的超时机制，超时机制会对发生超时的事件进行管理，并对这些超时事件作出处理。对于定时事件的管理包括两方面：定时事件对象的组织形式 和 定时事件对象的超时检测。

## 定时事件的组织

Nginx 的定时器由红黑树实现的。在保存事件的结构体ngx\_event\_t 中有三个关于时间管理的成员，如下所示：

```text
struct ngx_event_s{
    ...
      
    unsigned         timedout:1;  
      
    unsigned         timer_set:1;  
      
    ngx_rbtree_node_t   timer; 
    ...
};

```

Nginx 设置两个关于定时器的全局变量。在文件[src/event/ngx\_event\_timer.c](http://lxr.nginx.org/source/src/event/ngx_event_timer.c)中定义：

```text

ngx_thread_volatile ngx_rbtree_t  ngx_event_timer_rbtree;

static ngx_rbtree_node_t          ngx_event_timer_sentinel;

```

这棵红黑树的每一个节点代表一个事件 ngx\_event\_t 结构体中的成员timer，ngx\_rbtree\_node\_t 节点代表事件的超时时间，以这个超时时间的大小组成的红黑树ngx\_event\_timer\_rbtree，则该红黑树中最左边的节点代表最可能超时的事件。

定时器事件初始化实际上调用红黑树的初始化，其在文件 [src/event/ngx\_event\_timer.c](http://lxr.nginx.org/source/src/event/ngx_event_timer.c)中定义：

```text

ngx_int_t
ngx_event_timer_init(ngx_log_t *log)
{
    
    ngx_rbtree_init(&ngx_event_timer_rbtree, &ngx_event_timer_sentinel,
                    ngx_rbtree_insert_timer_value);

    
#if (NGX_THREADS)

    if (ngx_event_timer_mutex) {
        ngx_event_timer_mutex->log = log;
        return NGX_OK;
    }

    ngx_event_timer_mutex = ngx_mutex_init(log, 0);
    if (ngx_event_timer_mutex == NULL) {
        return NGX_ERROR;
    }

#endif

    return NGX_OK;
}

```

## 定时事件的超时检测

当需要对某个事件进行超时检测时，只需要将该事件添加到定时器红黑树中即可，由函数 ngx\_event\_add\_timer，将一个事件从定时器红黑树中删除由函数 ngx\_event\_del\_timer 实现。以下的函数都在文件 [src/event/ngx\_event\_timer.h](http://lxr.nginx.org/source/src/event/ngx_event_timer.h)中定义：

```text

static ngx_inline void
ngx_event_del_timer(ngx_event_t *ev)
{
    ngx_log_debug2(NGX_LOG_DEBUG_EVENT, ev->log, 0,
                   "event timer del: %d: %M",
                    ngx_event_ident(ev->data), ev->timer.key);

    ngx_mutex_lock(ngx_event_timer_mutex);

    
    ngx_rbtree_delete(&ngx_event_timer_rbtree, &ev->timer);

    ngx_mutex_unlock(ngx_event_timer_mutex);

#if (NGX_DEBUG)
    ev->timer.left = NULL;
    ev->timer.right = NULL;
    ev->timer.parent = NULL;
#endif

    
    ev->timer_set = 0;
}


static ngx_inline void
ngx_event_add_timer(ngx_event_t *ev, ngx_msec_t timer)
{
    ngx_msec_t      key;
    ngx_msec_int_t  diff;

    
    key = ngx_current_msec + timer;

    
    if (ev->timer_set) {

        

        diff = (ngx_msec_int_t) (key - ev->timer.key);

        if (ngx_abs(diff) < NGX_TIMER_LAZY_DELAY) {
            ngx_log_debug3(NGX_LOG_DEBUG_EVENT, ev->log, 0,
                           "event timer: %d, old: %M, new: %M",
                            ngx_event_ident(ev->data), ev->timer.key, key);
            return;
        }

        ngx_del_timer(ev);
    }

    ev->timer.key = key;

    ngx_log_debug3(NGX_LOG_DEBUG_EVENT, ev->log, 0,
                   "event timer add: %d: %M:%M",
                    ngx_event_ident(ev->data), timer, ev->timer.key);

    ngx_mutex_lock(ngx_event_timer_mutex);

    
    ngx_rbtree_insert(&ngx_event_timer_rbtree, &ev->timer);

    ngx_mutex_unlock(ngx_event_timer_mutex);

    
    ev->timer_set = 1;
}

```

判断一个函数是否超时由函数 ngx\_event\_find\_timer 实现，检查定时器所有事件由函数ngx\_event\_expire\_timer 实现。以下的函数都在文件[src/event/ngx\_event\_timer.c](http://lxr.nginx.org/source/src/event/ngx_event_timer.c)中定义：

```text

ngx_msec_t
ngx_event_find_timer(void)
{
    ngx_msec_int_t      timer;
    ngx_rbtree_node_t  *node, *root, *sentinel;

    
    if (ngx_event_timer_rbtree.root == &ngx_event_timer_sentinel) {
        return NGX_TIMER_INFINITE;
    }

    ngx_mutex_lock(ngx_event_timer_mutex);

    root = ngx_event_timer_rbtree.root;
    sentinel = ngx_event_timer_rbtree.sentinel;

    
    node = ngx_rbtree_min(root, sentinel);

    ngx_mutex_unlock(ngx_event_timer_mutex);

    
    timer = (ngx_msec_int_t) (node->key - ngx_current_msec);

    
    return (ngx_msec_t) (timer > 0 ? timer : 0);
}


void
ngx_event_expire_timers(void)
{
    ngx_event_t        *ev;
    ngx_rbtree_node_t  *node, *root, *sentinel;

    sentinel = ngx_event_timer_rbtree.sentinel;

    
    for ( ;; ) {

        ngx_mutex_lock(ngx_event_timer_mutex);

        root = ngx_event_timer_rbtree.root;

        
        if (root == sentinel) {
            return;
        }

        
        node = ngx_rbtree_min(root, sentinel);

        

        
        if ((ngx_msec_int_t) (node->key - ngx_current_msec) <= 0) {
            
            ev = (ngx_event_t *) ((char *) node - offsetof(ngx_event_t, timer));

            
#if (NGX_THREADS)

            if (ngx_threaded && ngx_trylock(ev->lock) == 0) {

                

                ngx_log_debug1(NGX_LOG_DEBUG_EVENT, ev->log, 0,
                               "event %p is busy in expire timers", ev);
                break;
            }
#endif

            ngx_log_debug2(NGX_LOG_DEBUG_EVENT, ev->log, 0,
                           "event timer del: %d: %M",
                           ngx_event_ident(ev->data), ev->timer.key);

            
            ngx_rbtree_delete(&ngx_event_timer_rbtree, &ev->timer);

            ngx_mutex_unlock(ngx_event_timer_mutex);

#if (NGX_DEBUG)
            ev->timer.left = NULL;
            ev->timer.right = NULL;
            ev->timer.parent = NULL;
#endif

            
            ev->timer_set = 0;

            
#if (NGX_THREADS)
            if (ngx_threaded) {
                ev->posted_timedout = 1;

                ngx_post_event(ev, &ngx_posted_events);

                ngx_unlock(ev->lock);

                continue;
            }
#endif

            
            ev->timedout = 1;

            
            ev->handler(ev);

            continue;
        }

        break;
    }

    ngx_mutex_unlock(ngx_event_timer_mutex);
}

```

参考资料

《深入剖析Nginx》

《深入理解Nginx》

