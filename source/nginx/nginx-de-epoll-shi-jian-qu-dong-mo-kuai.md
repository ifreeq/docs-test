# Nginx 的 epoll 事件驱动模块

## 概述

在前面的文章中《[Nginx 事件模块](http://blog.csdn.net/chenhanzhun/article/details/42805757)》介绍了Nginx 的事件驱动框架以及不同类型事件驱动模块的管理。本节基于前面的知识，简单介绍下在Linux 系统下的 epoll 事件驱动模块。关于 epoll 的使用与原理可以参照文章 《[epoll 解析](http://blog.csdn.net/chenhanzhun/article/details/42747127)》。在这里直接介绍Nginx 服务器基于事件驱动框架实现的epoll 事件驱动模块。

## ngx\_epoll\_module 事件驱动模块

## ngx\_epoll\_conf\_t 结构体

ngx\_epoll\_conf\_t 结构体是保存ngx\_epoll\_module 事件驱动模块的配置项结构。该结构体在文件[src/event/modules/ngx\_epoll\_module.c](http://lxr.nginx.org/source/src/event/modules/ngx_epoll_module.c) 中定义：

```text

typedef struct {
    ngx_uint_t  events;         
    ngx_uint_t  aio_requests;   
} ngx_epoll_conf_t;

```

## ngx\_epoll\_module 事件驱动模块的定义

所有模块的定义都是基于模块通用接口 ngx\_module\_t 结构，ngx\_epoll\_module 模块在文件[src/event/modules/ngx\_epoll\_module.c](http://lxr.nginx.org/source/src/event/modules/ngx_epoll_module.c) 中定义如下：

```text

ngx_module_t  ngx_epoll_module = {
    NGX_MODULE_V1,
    &ngx_epoll_module_ctx,               
    ngx_epoll_commands,                  
    NGX_EVENT_MODULE,                    
    NULL,                                
    NULL,                                
    NULL,                                
    NULL,                                
    NULL,                                
    NULL,                                
    NULL,                                
    NGX_MODULE_V1_PADDING
};

```

在 ngx\_epoll\_module 模块的定义中，其中定义了该模块感兴趣的配置项ngx\_epoll\_commands 数组，该配置项数组在文件[src/event/modules/ngx\_epoll\_module.c](http://lxr.nginx.org/source/src/event/modules/ngx_epoll_module.c) 中定义：

```text

static ngx_command_t  ngx_epoll_commands[] = {

    
    { ngx_string("epoll_events"),
      NGX_EVENT_CONF|NGX_CONF_TAKE1,
      ngx_conf_set_num_slot,
      0,
      offsetof(ngx_epoll_conf_t, events),
      NULL },

    
    { ngx_string("worker_aio_requests"),
      NGX_EVENT_CONF|NGX_CONF_TAKE1,
      ngx_conf_set_num_slot,
      0,
      offsetof(ngx_epoll_conf_t, aio_requests),
      NULL },

      ngx_null_command
};

```

在 ngx\_epoll\_module 模块的定义中，定义了该模块的上下文结构ngx\_epoll\_module\_ctx，该上下文结构是基于事件模块的通用接口ngx\_event\_module\_t 结构来定义的。该上下文结构在文件[src/event/modules/ngx\_epoll\_module.c](http://lxr.nginx.org/source/src/event/modules/ngx_epoll_module.c) 中定义：

```text

ngx_event_module_t  ngx_epoll_module_ctx = {
    &epoll_name,
    ngx_epoll_create_conf,               
    ngx_epoll_init_conf,                 

    {
        ngx_epoll_add_event,             
        ngx_epoll_del_event,             
        ngx_epoll_add_event,             
        ngx_epoll_del_event,             
        ngx_epoll_add_connection,        
        ngx_epoll_del_connection,        
        NULL,                            
        ngx_epoll_process_events,        
        ngx_epoll_init,                  
        ngx_epoll_done,                  
    }
};

```

在 ngx\_epoll\_module 模块的上下文事件接口结构中，重点定义了ngx\_event\_actions\_t 结构中的接口回调方法。

## ngx\_epoll\_module 事件驱动模块的操作

ngx\_epoll\_module 模块的操作由ngx\_epoll\_module 模块的上下文事件接口结构中成员actions 实现。该成员实现的方法如下所示：

```text
        ngx_epoll_add_event,             
        ngx_epoll_del_event,             
        ngx_epoll_add_event,             
        ngx_epoll_del_event,             
        ngx_epoll_add_connection,        
        ngx_epoll_del_connection,        
        NULL,                            
        ngx_epoll_process_events,        
        ngx_epoll_init,                  
        ngx_epoll_done,                  

```

### ngx\_epoll\_module 模块的初始化

ngx\_epoll\_module 模块的初始化由函数ngx\_epoll\_init 实现。该函数主要做了两件事：创建epoll 对象 和 创建 event\_list 数组（调用epoll\_wait 函数时用于存储从内核复制的已就绪的事件）；该函数在文件[src/event/modules/ngx\_epoll\_module.c](http://lxr.nginx.org/source/src/event/modules/ngx_epoll_module.c) 中定义：

```text
static int                  ep = -1;    
static struct epoll_event  *event_list; 
static ngx_uint_t           nevents;    


static ngx_int_t
ngx_epoll_init(ngx_cycle_t *cycle, ngx_msec_t timer)
{
    ngx_epoll_conf_t  *epcf;

    
    epcf = ngx_event_get_conf(cycle->conf_ctx, ngx_epoll_module);

    if (ep == -1) {
        
        ep = epoll_create(cycle->connection_n / 2);

        
        if (ep == -1) {
            ngx_log_error(NGX_LOG_EMERG, cycle->log, ngx_errno,
                          "epoll_create() failed");
            return NGX_ERROR;
        }

#if (NGX_HAVE_FILE_AIO)

        
        ngx_epoll_aio_init(cycle, epcf);

#endif
    }

    
    if (nevents < epcf->events) {
        
        if (event_list) {
            ngx_free(event_list);
        }

        
        event_list = ngx_alloc(sizeof(struct epoll_event) * epcf->events,
                               cycle->log);
        if (event_list == NULL) {
            return NGX_ERROR;
        }
    }

    
    nevents = epcf->events;

    
    
    ngx_io = ngx_os_io;

    
    ngx_event_actions = ngx_epoll_module_ctx.actions;

#if (NGX_HAVE_CLEAR_EVENT)
    
    ngx_event_flags = NGX_USE_CLEAR_EVENT
#else
    
    ngx_event_flags = NGX_USE_LEVEL_EVENT
#endif
                      |NGX_USE_GREEDY_EVENT
                      |NGX_USE_EPOLL_EVENT;

    return NGX_OK;
}

```

### ngx\_epoll\_module 模块的事件处理

ngx\_epoll\_module 模块的事件处理由函数ngx\_epoll\_process\_events 实现。ngx\_epoll\_process\_events 函数是实现事件收集、事件发送的接口。该函数在文件[src/event/modules/ngx\_epoll\_module.c](http://lxr.nginx.org/source/src/event/modules/ngx_epoll_module.c) 中定义：

```text

static ngx_int_t
ngx_epoll_process_events(ngx_cycle_t *cycle, ngx_msec_t timer, ngx_uint_t flags)
{
    int                events;
    uint32_t           revents;
    ngx_int_t          instance, i;
    ngx_uint_t         level;
    ngx_err_t          err;
    ngx_event_t       *rev, *wev, **queue;
    ngx_connection_t  *c;

    

    ngx_log_debug1(NGX_LOG_DEBUG_EVENT, cycle->log, 0,
                   "epoll timer: %M", timer);

    
    events = epoll_wait(ep, event_list, (int) nevents, timer);

    
    err = (events == -1) ? ngx_errno : 0;

    
    if (flags & NGX_UPDATE_TIME || ngx_event_timer_alarm) {
        
        ngx_time_update();
    }

    
    if (err) {
        if (err == NGX_EINTR) {

            if (ngx_event_timer_alarm) {
                ngx_event_timer_alarm = 0;
                return NGX_OK;
            }

            level = NGX_LOG_INFO;

        } else {
            level = NGX_LOG_ALERT;
        }

        ngx_log_error(level, cycle->log, err, "epoll_wait() failed");
        return NGX_ERROR;
    }

    
    if (events == 0) {
        if (timer != NGX_TIMER_INFINITE) {
            return NGX_OK;
        }

        ngx_log_error(NGX_LOG_ALERT, cycle->log, 0,
                      "epoll_wait() returned no events without timeout");
        return NGX_ERROR;
    }

    
    ngx_mutex_lock(ngx_posted_events_mutex);

    
    for (i = 0; i < events; i++) {
        
        c = event_list[i].data.ptr;

        
        instance = (uintptr_t) c & 1;
        
        c = (ngx_connection_t *) ((uintptr_t) c & (uintptr_t) ~1);

        
        rev = c->read;

        
        if (c->fd == -1 || rev->instance != instance) {

            

            ngx_log_debug1(NGX_LOG_DEBUG_EVENT, cycle->log, 0,
                           "epoll: stale event %p", c);
            continue;
        }

        
        revents = event_list[i].events;

        ngx_log_debug3(NGX_LOG_DEBUG_EVENT, cycle->log, 0,
                       "epoll: fd:%d ev:%04XD d:%p",
                       c->fd, revents, event_list[i].data.ptr);
        
        
        if (revents & (EPOLLERR|EPOLLHUP)) {
            ngx_log_debug2(NGX_LOG_DEBUG_EVENT, cycle->log, 0,
                           "epoll_wait() error on fd:%d ev:%04XD",
                           c->fd, revents);
        }

#if 0
        if (revents & ~(EPOLLIN|EPOLLOUT|EPOLLERR|EPOLLHUP)) {
            ngx_log_error(NGX_LOG_ALERT, cycle->log, 0,
                          "strange epoll_wait() events fd:%d ev:%04XD",
                          c->fd, revents);
        }
#endif

        
        if ((revents & (EPOLLERR|EPOLLHUP))
             && (revents & (EPOLLIN|EPOLLOUT)) == 0)
        {
            

            revents |= EPOLLIN|EPOLLOUT;
        }

        
        if ((revents & EPOLLIN) && rev->active) {

#if (NGX_HAVE_EPOLLRDHUP)
            
            if (revents & EPOLLRDHUP) {
                rev->pending_eof = 1;
            }
#endif

            if ((flags & NGX_POST_THREAD_EVENTS) && !rev->accept) {
                rev->posted_ready = 1;

            } else {
                
                
                rev->ready = 1;
            }

            
            if (flags & NGX_POST_EVENTS) {
                queue = (ngx_event_t **) (rev->accept ?
                               &ngx_posted_accept_events : &ngx_posted_events);

                ngx_locked_post_event(rev, queue);

            } else {
                
                rev->handler(rev);
            }
        }

        
        wev = c->write;

        
        if ((revents & EPOLLOUT) && wev->active) {

            
            if (c->fd == -1 || wev->instance != instance) {

                

                ngx_log_debug1(NGX_LOG_DEBUG_EVENT, cycle->log, 0,
                               "epoll: stale event %p", c);
                continue;
            }

            if (flags & NGX_POST_THREAD_EVENTS) {
                wev->posted_ready = 1;

            } else {

                
                wev->ready = 1;
            }

            
            if (flags & NGX_POST_EVENTS) {
                ngx_locked_post_event(wev, &ngx_posted_events);

            } else {
                
                wev->handler(wev);
            }
        }
    }

    ngx_mutex_unlock(ngx_posted_events_mutex);

    return NGX_OK;
}

```

### ngx\_epoll\_module 模块的事件添加与删除

ngx\_epoll\_module 模块的事件添加与删除分别由函数ngx\_epoll\_add\_event 与ngx\_epoll\_del\_event 实现。这两个函数的实现都是通过调用epoll\_ctl 函数。具体实现在文件 [src/event/modules/ngx\_epoll\_module.c](http://lxr.nginx.org/source/src/event/modules/ngx_epoll_module.c) 中定义：

```text

static ngx_int_t
ngx_epoll_add_event(ngx_event_t *ev, ngx_int_t event, ngx_uint_t flags)
{
    int                  op;
    uint32_t             events, prev;
    ngx_event_t         *e;
    ngx_connection_t    *c;
    struct epoll_event   ee;

    
    
    c = ev->data;

    
    events = (uint32_t) event;

    

    if (event == NGX_READ_EVENT) {
        
        e = c->write;
        prev = EPOLLOUT;
#if (NGX_READ_EVENT != EPOLLIN|EPOLLRDHUP)
        events = EPOLLIN|EPOLLRDHUP;
#endif

    } else {
        e = c->read;
        prev = EPOLLIN|EPOLLRDHUP;
#if (NGX_WRITE_EVENT != EPOLLOUT)
        events = EPOLLOUT;
#endif
    }

    
    if (e->active) {
        
        op = EPOLL_CTL_MOD;
        events |= prev;

    } else {
        
        op = EPOLL_CTL_ADD;
    }

    
    ee.events = events | (uint32_t) flags;
    
    ee.data.ptr = (void *) ((uintptr_t) c | ev->instance);

    ngx_log_debug3(NGX_LOG_DEBUG_EVENT, ev->log, 0,
                   "epoll add event: fd:%d op:%d ev:%08XD",
                   c->fd, op, ee.events);

    
    if (epoll_ctl(ep, op, c->fd, &ee) == -1) {
        ngx_log_error(NGX_LOG_ALERT, ev->log, ngx_errno,
                      "epoll_ctl(%d, %d) failed", op, c->fd);
        return NGX_ERROR;
    }

    
    ev->active = 1;
#if 0
    ev->oneshot = (flags & NGX_ONESHOT_EVENT) ? 1 : 0;
#endif

    return NGX_OK;
}


static ngx_int_t
ngx_epoll_del_event(ngx_event_t *ev, ngx_int_t event, ngx_uint_t flags)
{
    int                  op;
    uint32_t             prev;
    ngx_event_t         *e;
    ngx_connection_t    *c;
    struct epoll_event   ee;

    

    
    if (flags & NGX_CLOSE_EVENT) {
        ev->active = 0;
        return NGX_OK;
    }

    
    c = ev->data;

    
    if (event == NGX_READ_EVENT) {
        
        e = c->write;
        prev = EPOLLOUT;

    } else {
        
        e = c->read;
        prev = EPOLLIN|EPOLLRDHUP;
    }

    
    if (e->active) {
        op = EPOLL_CTL_MOD;
        ee.events = prev | (uint32_t) flags;
        ee.data.ptr = (void *) ((uintptr_t) c | ev->instance);

    } else {
        
        op = EPOLL_CTL_DEL;
        ee.events = 0;
        ee.data.ptr = NULL;
    }

    ngx_log_debug3(NGX_LOG_DEBUG_EVENT, ev->log, 0,
                   "epoll del event: fd:%d op:%d ev:%08XD",
                   c->fd, op, ee.events);

    
    if (epoll_ctl(ep, op, c->fd, &ee) == -1) {
        ngx_log_error(NGX_LOG_ALERT, ev->log, ngx_errno,
                      "epoll_ctl(%d, %d) failed", op, c->fd);
        return NGX_ERROR;
    }

    
    ev->active = 0;

    return NGX_OK;
}

```

### ngx\_epoll\_module 模块的连接添加与删除

ngx\_epoll\_module 模块的连接添加与删除分别由函数ngx\_epoll\_add\_connection 与ngx\_epoll\_del\_connection 实现。这两个函数的实现都是通过调用epoll\_ctl 函数。具体实现在文件 [src/event/modules/ngx\_epoll\_module.c](http://lxr.nginx.org/source/src/event/modules/ngx_epoll_module.c) 中定义：

```text

static ngx_int_t
ngx_epoll_add_connection(ngx_connection_t *c)
{
    struct epoll_event  ee;

    
    ee.events = EPOLLIN|EPOLLOUT|EPOLLET|EPOLLRDHUP;
    ee.data.ptr = (void *) ((uintptr_t) c | c->read->instance);

    ngx_log_debug2(NGX_LOG_DEBUG_EVENT, c->log, 0,
                   "epoll add connection: fd:%d ev:%08XD", c->fd, ee.events);

    
    if (epoll_ctl(ep, EPOLL_CTL_ADD, c->fd, &ee) == -1) {
        ngx_log_error(NGX_LOG_ALERT, c->log, ngx_errno,
                      "epoll_ctl(EPOLL_CTL_ADD, %d) failed", c->fd);
        return NGX_ERROR;
    }

    
    c->read->active = 1;
    c->write->active = 1;

    return NGX_OK;
}
Nginx


static ngx_int_t
ngx_epoll_del_connection(ngx_connection_t *c, ngx_uint_t flags)
{
    int                 op;
    struct epoll_event  ee;

    

    if (flags & NGX_CLOSE_EVENT) {
        c->read->active = 0;
        c->write->active = 0;
        return NGX_OK;
    }

    ngx_log_debug1(NGX_LOG_DEBUG_EVENTNginx, c->log, 0,
                   "epoll del connection: fd:%d", c->fd);

    op = EPOLL_CTL_DEL;
    ee.events = 0;
    ee.data.ptr = NULL;

    
    if (epoll_ctl(ep, op, c->fd, &ee) == -1) {
        ngx_log_error(NGX_LOG_ALERT, c->log, ngx_errno,
                      "epoll_ctl(%d, %d) failed", op, c->fd);
        return NGX_ERROR;
    }

    
    c->read->active = 0;
    c->write->active = 0;

    return NGX_OK;
}

```

## ngx\_epoll\_module 模块的异步 I/O

在 Nginx 中，文件异步 I/O 事件完成后的通知是集成到 epoll 对象中。该模块的文件异步I/O 实现如下：

```text
#if (NGX_HAVE_FILE_AIO)

int                         ngx_eventfd = -1;   
aio_context_t               ngx_aio_ctx = 0;    

static ngx_event_t          ngx_eventfd_event;  
static ngx_connection_t     ngx_eventfd_conn;   

#endif

#if (NGX_HAVE_FILE_AIO)




static int
io_setup(u_int nr_reqs, aio_context_t *ctx)
{
    return syscall(SYS_io_setup, nr_reqs, ctx);
}


static int
io_destroy(aio_context_t ctx)
{
    return syscall(SYS_io_destroy, ctx);
}


static int
io_getevents(aio_context_t ctx, long min_nr, long nr, struct io_event *events,
    struct timespec *tmo)
{
    return syscall(SYS_io_getevents, ctx, min_nr, nr, events, tmo);
}


static void
ngx_epoll_aio_init(ngx_cycle_t *cycle, ngx_epoll_conf_t *epcf)
{
    int                 n;
    struct epoll_event  ee;

    
    ngx_eventfd = syscall(SYS_eventfd, 0);

    if (ngx_eventfd == -1) {
        ngx_log_error(NGX_LOG_EMERG, cycle->log, ngx_errno,
                      "eventfd() failed");
        ngx_file_aio = 0;
        return;
    }

    ngx_log_debug1(NGX_LOG_DEBUG_EVENT, cycle->log, 0,
                   "eventfd: %d", ngx_eventfd);

    n = 1;

    
    if (ioctl(ngx_eventfd, FIONBIO, &n) == -1) {
        ngx_log_error(NGX_LOG_EMERG, cycle->log, ngx_errno,
                      "ioctl(eventfd, FIONBIO) failed");
        goto failed;
    }

    
    if (io_setup(epcf->aio_requests, &ngx_aio_ctx) == -1) {
        ngx_log_error(NGX_LOG_EMERG, cycle->log, ngx_errno,
                      "io_setup() failed");
        goto failed;
    }

    

    
    ngx_eventfd_event.data = &ngx_eventfd_conn;
    
    ngx_eventfd_event.handler = ngx_epoll_eventfd_handler;
    
    ngx_eventfd_event.log = cycle->log;
    
    ngx_eventfd_event.active = 1;
    
    ngx_eventfd_conn.fd = ngx_eventfd;
    
    ngx_eventfd_conn.read = &ngx_eventfd_event;
    
    ngx_eventfd_conn.log = cycle->log;

    ee.events = EPOLLIN|EPOLLET;
    ee.data.ptr = &ngx_eventfd_conn;

    
    if (epoll_ctl(ep, EPOLL_CTL_ADD, ngx_eventfd, &ee) != -1) {
        return;
    }

    
    ngx_log_error(NGX_LOG_EMERG, cycle->log, ngx_errno,
                  "epoll_ctl(EPOLL_CTL_ADD, eventfd) failed");

    if (io_destroy(ngx_aio_ctx) == -1) {
        ngx_log_error(NGX_LOG_ALERT, cycle->log, ngx_errno,
                      "io_destroy() failed");
    }

failed:

    if (close(ngx_eventfd) == -1) {
        ngx_log_error(NGX_LOG_ALERT, cycle->log, ngx_errno,
                      "eventfd close() failed");
    }

    ngx_eventfd = -1;
    ngx_aio_ctx = 0;
    ngx_file_aio = 0;
}

#endif

#if (NGX_HAVE_FILE_AIO)


static void
ngx_epoll_eventfd_handler(ngx_event_t *ev)
{
    int               n, events;
    long              i;
    uint64_t          ready;
    ngx_err_t         err;
    ngx_event_t      *e;
    ngx_event_aio_t  *aio;
    struct io_event   event[64];    
    struct timespec   ts;

    ngx_log_debug0(NGX_LOG_DEBUG_EVENT, ev->log, 0, "eventfd handler");

    
    n = read(ngx_eventfd, &ready, 8);

    err = ngx_errno;

    ngx_log_debug1(NGX_LOG_DEBUG_EVENT, ev->log, 0, "eventfd: %d", n);

    if (n != 8) {
        if (n == -1) {
            if (err == NGX_EAGAIN) {
                return;
            }

            ngx_log_error(NGX_LOG_ALERT, ev->log, err, "read(eventfd) failed");
            return;
        }

        ngx_log_error(NGX_LOG_ALERT, ev->log, 0,
                      "read(eventfd) returned only %d bytes", n);
        return;
    }

    ts.tv_sec = 0;
    ts.tv_nsec = 0;

    
    while (ready) {

        
        events = io_getevents(ngx_aio_ctx, 1, 64, event, &ts);

        ngx_log_debug1(NGX_LOG_DEBUG_EVENT, ev->log, 0,
                       "io_getevents: %l", events);

        if (events > 0) {
            ready -= events;

            
            for (i = 0; i < events; i++) {

                ngx_log_debug4(NGX_LOG_DEBUG_EVENT, ev->log, 0,
                               "io_event: %uXL %uXL %L %L",
                                event[i].data, event[i].obj,
                                event[i].res, event[i].res2);

                
                e = (ngx_event_t *) (uintptr_t) event[i].data;

                e->complete = 1;
                e->active = 0;
                e->ready = 1;

                aio = e->data;
                aio->res = event[i].res;

                
                ngx_post_event(e, &ngx_posted_events);
            }

            continue;
        }

        if (events == 0) {
            return;
        }

        
        ngx_log_error(NGX_LOG_ALERT, ev->log, ngx_errno,
                      "io_getevents() failed");
        return;
    }
}

#endif

```

参考资料：

《深入理解Nginx》

《[模块ngx\_epoll\_module详解](http://blog.csdn.net/xiajun07061225/article/details/9250341)》

《 [nginx epoll详解](http://blog.csdn.net/freeinfor/article/details/17008131)》

《[Nginx源码分析-Epoll模块](http://www.alidata.org/archives/1296)》

《[关于ngx\_epoll\_add\_event的一些解释](http://blog.csdn.net/brainkick/article/details/9080789)》

