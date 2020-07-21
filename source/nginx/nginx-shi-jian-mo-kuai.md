# Nginx 事件模块

## 概述

Nginx 是以事件的触发来驱动的，事件驱动模型主要包括事件收集、事件发送、事件处理（即事件管理）三部分。在Nginx 的工作进程中主要关注的事件是IO 网络事件 和 定时器事件。在生成的 objs 目录文件中，其中ngx\_modules.c 文件的内容是Nginx 各种模块的执行顺序，我们可以从该文件的内容中看到事件模块的执行顺序为以下所示：注意：由于是在Linux 系统下，所以支持具体的 epoll 事件模块，接下来的文章结构按照以下顺序来写。

```text
extern ngx_module_t  ngx_events_module;
extern ngx_module_t  ngx_event_core_module;
extern ngx_module_t  ngx_epoll_module;

```

## 事件模块接口

## ngx\_event\_module\_t 结构体

在 Nginx 中，结构体 ngx\_module\_t 是 Nginx 模块最基本的接口。对于每一种不同类型的模块，都有一个具体的结构体来描述这一类模块的通用接口，该接口由ngx\_module\_t 中的成员ctx 管理。在 Nginx 中定义了事件模块的通用接口ngx\_event\_module\_t 结构体，该结构体定义在文件[src/event/ngx\_event.h](http://lxr.nginx.org/source/src/event/ngx_event.h) 中：

```text

typedef struct {
    
    ngx_str_t              *name;

    
    void                 *(*create_conf)(ngx_cycle_t *cycle);
    
    char                 *(*init_conf)(ngx_cycle_t *cycle, void *conf);

    
    ngx_event_actions_t     actions;
} ngx_event_module_t;

```

在 ngx\_event\_module\_t 结构体中actions 的类型是ngx\_event\_actions\_t 结构体，该成员结构实现了事件驱动模块的具体方法。该结构体定义在文件[src/event/ngx\_event.h](http://lxr.nginx.org/source/src/event/ngx_event.h) 中：

```text

typedef struct {
    
    ngx_int_t  (*add)(ngx_event_t *ev, ngx_int_t event, ngx_uint_t flags);
    
    ngx_int_t  (*del)(ngx_event_t *ev, ngx_int_t event, ngx_uint_t flags);

    
    ngx_int_t  (*enable)(ngx_event_t *ev, ngx_int_t event, ngx_uint_t flags);
    
    ngx_int_t  (*disable)(ngx_event_t *ev, ngx_int_t event, ngx_uint_t flags);

    
    ngx_int_t  (*add_conn)(ngx_connection_t *c);
    
    ngx_int_t  (*del_conn)(ngx_connection_t *c, ngx_uint_t flags);

    
    ngx_int_t  (*process_changes)(ngx_cycle_t *cycle, ngx_uint_t nowait);
    
    ngx_int_t  (*process_events)(ngx_cycle_t *cycle, ngx_msec_t timer,
                   ngx_uint_t flags);

    
    ngx_int_t  (*init)(ngx_cycle_t *cycle, ngx_msec_t timer);
    
    void       (*done)(ngx_cycle_t *cycle);
} ngx_event_actions_t;

```

## ngx\_event\_t 结构体

在 Nginx 中，每一个具体事件的定义由结构体ngx\_event\_t 来表示，该结构体ngx\_event\_t 用来保存具体事件。该结构体定义在文件 [src/event/ngx\_event.h](http://lxr.nginx.org/source/src/event/ngx_event.h) 中：

```text

struct ngx_event_s {
    
    void            *data;

    
    unsigned         write:1;

    
    unsigned         accept:1;

    
    unsigned         instance:1;

    
    
    unsigned         active:1;

    
    unsigned         disabled:1;

    
    
    unsigned         ready:1;

    
    unsigned         oneshot:1;

    
    
    unsigned         complete:1;

    
    unsigned         eof:1;
    
    unsigned         error:1;

    
    unsigned         timedout:1;
    
    unsigned         timer_set:1;

    
    unsigned         delayed:1;

    
    unsigned         deferred_accept:1;

    
    
    unsigned         pending_eof:1;

    
    unsigned         posted:1;

#if (NGX_WIN32)
    
    unsigned         accept_context_updated:1;
#endif

#if (NGX_HAVE_KQUEUE)
    unsigned         kq_vnode:1;

    
    int              kq_errno;
#endif

    

#if (NGX_HAVE_KQUEUE) || (NGX_HAVE_IOCP)
    int              available;
#else
    
    unsigned         available:1;
#endif

    
    ngx_event_handler_pt  handler;

#if (NGX_HAVE_AIO)

#if (NGX_HAVE_IOCP)
    ngx_event_ovlp_t ovlp;
#else
    
    struct aiocb     aiocb;
#endif

#endif

    
    ngx_uint_t       index;

    
    ngx_log_t       *log;

    
    ngx_rbtree_node_t   timer;

    
    ngx_queue_t      queue;

    
    unsigned         closed:1;

    
    unsigned         channel:1;
    unsigned         resolver:1;

    unsigned         cancelable:1;

#if 0

    

    

    void            *thr_ctx;

#if (NGX_EVENT_T_PADDING)

    

    uint32_t         padding[NGX_EVENT_T_PADDING];
#endif
#endif
};

```

在每个事件结构体 ngx\_event\_t 最重要的成员是handler 回调函数，该回调函数定义了当事件发生时的处理方法。该回调方法原型在文件[src/core/ngx\_core.h](http://lxr.nginx.org/source/src/core/ngx_core.h) 中：

```text
typedef void (*ngx_event_handler_pt)(ngx_event_t *ev);

```

## ngx\_connection\_t 结构体

当客户端向 Nginx 服务器发起连接请求时，此时若Nginx 服务器被动接收该连接，则相对Nginx 服务器来说称为被动连接，被动连接的表示由基本数据结构体ngx\_connection\_t 完成。该结构体定义在文件 [src/core/ngx\_connection.h](http://lxr.nginx.org/source/src/core/ngx_connection.h) 中：

```text

struct ngx_connection_s {
    

    
    void               *data;
    
    ngx_event_t        *read;
    
    ngx_event_t        *write;

    
    ngx_socket_t        fd;

    
    ngx_recv_pt         recv;
    
    ngx_send_pt         send;
    
    ngx_recv_chain_pt   recv_chain;
    
    ngx_send_chain_pt   send_chain;

    
    ngx_listening_t    *listening;

    
    off_t               sent;

    
    ngx_log_t          *log;

    
    ngx_pool_t         *pool;

    
    struct sockaddr    *sockaddr;
    socklen_t           socklen;
    
    ngx_str_t           addr_text;

    ngx_str_t           proxy_protocol_addr;

#if (NGX_SSL)
    ngx_ssl_connection_t  *ssl;
#endif

    
    struct sockaddr    *local_sockaddr;
    socklen_t           local_socklen;

    
    ngx_buf_t          *buffer;

    
    ngx_queue_t         queue;

    
    ngx_atomic_uint_t   number;

    
    ngx_uint_t          requests;

    unsigned            buffered:8;

    unsigned            log_error:3;     

    
    unsigned            unexpected_eof:1;
    
    unsigned            timedout:1;
    
    unsigned            error:1;
    
    unsigned            destroyed:1;

    
    unsigned            idle:1;
    
    unsigned            reusable:1;
    
    unsigned            close:1;

    
    unsigned            sendfile:1;
    
    unsigned            sndlowat:1;
    unsigned            tcp_nodelay:2;   
    unsigned            tcp_nopush:2;    

    unsigned            need_last_buf:1;

#if (NGX_HAVE_IOCP)
    unsigned            accept_context_updated:1;
#endif

#if (NGX_HAVE_AIO_SENDFILE)
    
    unsigned            aio_sendfile:1;
    unsigned            busy_count:2;
    
    ngx_buf_t          *busy_sendfile;
#endif

#if (NGX_THREADS)
    ngx_atomic_t        lock;
#endif
};

```

在处理请求的过程中，若 Nginx 服务器主动向上游服务器建立连接，完成连接建立并与之进行通信，这种相对Nginx 服务器来说是一种主动连接，主动连接由结构体ngx\_peer\_connection\_t 表示，但是该结构体 ngx\_peer\_connection\_t 也是 ngx\_connection\_t 结构体的封装。该结构体定义在文件[src/event/ngx\_event\_connect.h](http://lxr.nginx.org/source/src/event/ngx_event_connect.h) 中：

```text

struct ngx_peer_connection_s {
    
    ngx_connection_t                *connection;

    
    struct sockaddr                 *sockaddr;
    socklen_t                        socklen;
    
    ngx_str_t                       *name;

    
    ngx_uint_t                       tries;

    
    ngx_event_get_peer_pt            get;
    
    ngx_event_free_peer_pt           free;
    
    void                            *data;

#if (NGX_SSL)
    ngx_event_set_peer_session_pt    set_session;
    ngx_event_save_peer_session_pt   save_session;
#endif

#if (NGX_THREADS)
    ngx_atomic_t                    *lock;
#endif

    
    ngx_addr_t                      *local;

    
    int                              rcvbuf;

    
    ngx_log_t                       *log;

    
    unsigned                         cached:1;

                                     
    unsigned                         log_error:2;
};

```

## ngx\_events\_module 核心模块

## ngx\_events\_module 核心模块的定义

ngx\_events\_module 模块是事件的核心模块，该模块的功能是：定义新的事件类型，并为每个事件模块定义通用接口ngx\_event\_module\_t 结构体，管理事件模块生成的配置项结构体，并解析事件类配置项。首先，看下该模块在文件[src/event/ngx\_event.c](http://lxr.nginx.org/source/src/event/ngx_event.c) 中的定义：

```text

ngx_module_t  ngx_events_module = {
    NGX_MODULE_V1,
    &ngx_events_module_ctx,                
    ngx_events_commands,                   
    NGX_CORE_MODULE,                       
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

其中，模块的配置项指令结构 ngx\_events\_commands 决定了该模块的功能。配置项指令结构ngx\_events\_commands 在文件[src/event/ngx\_event.c](http://lxr.nginx.org/source/src/event/ngx_event.c) 中定义如下：

```text

static ngx_command_t  ngx_events_commands[] = {

    { ngx_string("events"),
      NGX_MAIN_CONF|NGX_CONF_BLOCK|NGX_CONF_NOARGS,
      ngx_events_block,
      0,
      0,
      NULL },

      ngx_null_command
};

```

从配置项结构体中可以知道，该模块只对 events{...} 配置块感兴趣，并定义了管理事件模块的方法ngx\_events\_block；ngx\_events\_block 方法在文件[src/event/ngx\_event.c](http://lxr.nginx.org/source/src/event/ngx_event.c) 中定义：

```text

static char *
ngx_events_block(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)
{
    char                 *rv;
    void               ***ctx;
    ngx_uint_t            i;
    ngx_conf_t            pcf;
    ngx_event_module_t   *m;

    if (*(void **) conf) {
        return "is duplicate";
    }

    

    
    ngx_event_max_module = 0;
    for (i = 0; ngx_modules[i]; i++) {
        if (ngx_modules[i]->type != NGX_EVENT_MODULE) {
            continue;
        }

        ngx_modules[i]->ctx_index = ngx_event_max_module++;
    }

    ctx = ngx_pcalloc(cf->pool, sizeof(void *));
    if (ctx == NULL) {
        return NGX_CONF_ERROR;
    }

    
    *ctx = ngx_pcalloc(cf->pool, ngx_event_max_module * sizeof(void *));
    if (*ctx == NULL) {
        return NGX_CONF_ERROR;
    }

    *(void **) conf = ctx;

    
    for (i = 0; ngx_modules[i]; i++) {
        if (ngx_modules[i]->type != NGX_EVENT_MODULE) {
            continue;
        }

        m = ngx_modules[i]->ctx;

        if (m->create_conf) {
            (*ctx)[ngx_modules[i]->ctx_index] = m->create_conf(cf->cycle);
            if ((*ctx)[ngx_modules[i]->ctx_index] == NULL) {
                return NGX_CONF_ERROR;
            }
        }
    }

    
    pcf = *cf;
    cf->ctx = ctx;
    cf->module_type = NGX_EVENT_MODULE;
    cf->cmd_type = NGX_EVENT_CONF;

    
    rv = ngx_conf_parse(cf, NULL);

    *cf = pcf;

    if (rv != NGX_CONF_OK)
        return rv;

    
    for (i = 0; ngx_modules[i]; i++) {
        if (ngx_modules[i]->type != NGX_EVENT_MODULE) {
            continue;
        }

        m = ngx_modules[i]->ctx;

        if (m->init_conf) {
            rv = m->init_conf(cf->cycle, (*ctx)[ngx_modules[i]->ctx_index]);
            if (rv != NGX_CONF_OK) {
                return rv;
            }
        }
    }

    return NGX_CONF_OK;
}

```

另外，在 ngx\_events\_module 模块的定义中有一个成员ctx 指向了核心模块的通用接口结构。核心模块的通用接口结构体定义在文件[src/core/ngx\_conf\_file.h](http://lxr.nginx.org/source/src/core/ngx_conf_file.h) 中：

```text

typedef struct {
    
    ngx_str_t             name;
    
    void               *(*create_conf)(ngx_cycle_t *cycle);
    
    char               *(*init_conf)(ngx_cycle_t *cycle, void *conf);
} ngx_core_module_t;

```

因此，ngx\_events\_module 作为核心模块，必须定义核心模块的通用接口结构。ngx\_events\_module 模块的核心模块通用接口在文件[src/event/ngx\_event.c](http://lxr.nginx.org/source/src/event/ngx_event.c) 中定义：

```text

static ngx_core_module_t  ngx_events_module_ctx = {
    ngx_string("events"),
    NULL,
    
    ngx_event_init_conf
};

```

## 所有事件模块的配置项管理

Nginx 服务器在结构体 ngx\_cycle\_t 中定义了一个四级指针成员 conf\_ctx，整个Nginx 模块都是使用该四级指针成员管理模块的配置项结构，以下events 模块为例对该四级指针成员进行简单的分析，如下图所示：

每个事件模块可以通过宏定义 ngx\_event\_get\_conf 获取它在create\_conf 中分配的结构体的指针；该宏中定义如下：

```text
#define ngx_event_get_conf(conf_ctx, module) \
(*(ngx_get_conf(conf_ctx, ngx_events_module))) [module.ctx_index];


#define ngx_get_conf(conf_ctx, module) conf_ctx[module.index]

```

从上面的宏定义可以知道，每个事件模块获取自己在 create\_conf 中分配的结构体的指针，只需在ngx\_event\_get\_conf 传入参数ngx\_cycle\_t 中的 conf\_ctx 成员，并且传入自己模块的名称即可获取自己分配的结构体指针。

## ngx\_event\_core\_module 事件模块

ngx\_event\_core\_module 模块是一个事件类型的模块，它在所有事件模块中的顺序是第一，是其它事件类模块的基础。它主要完成以下任务：

1. 创建连接池；
2. 决定使用哪些事件驱动机制；
3. 初始化将要使用的事件模块；

## ngx\_event\_conf\_t 结构体

ngx\_event\_conf\_t 结构体是用来保存ngx\_event\_core\_module 事件模块配置项参数的。该结构体在文件[src/event/ngx\_event.h](http://lxr.nginx.org/source/src/event/ngx_event.h) 中定义：

```text

typedef struct {
    
    ngx_uint_t    connections;
    
    ngx_uint_t    use;

    
    ngx_flag_t    multi_accept;
    
    ngx_flag_t    accept_mutex;

    
    ngx_msec_t    accept_mutex_delay;

    
    u_char       *name;

#if (NGX_DEBUG)
    
    ngx_array_t   debug_connection;
#endif
} ngx_event_conf_t;

```

## ngx\_event\_core\_module 事件模块的定义

该模块在文件 [src/event/ngx\_event.c](http://lxr.nginx.org/source/src/event/ngx_event.c) 中定义：

```text

ngx_module_t  ngx_event_core_module = {
    NGX_MODULE_V1,
    &ngx_event_core_module_ctx,            
    ngx_event_core_commands,               
    NGX_EVENT_MODULE,                      
    NULL,                                  
    ngx_event_module_init,                 
    ngx_event_process_init,                
    NULL,                                  
    NULL,                                  
    NULL,                                  
    NULL,                                  
    NGX_MODULE_V1_PADDING
};

```

其中，模块的配置项指令结构 ngx\_event\_core\_commands 决定了该模块的功能。配置项指令结构 ngx\_event\_core\_commands 在文件 [src/event/ngx\_event.c](http://lxr.nginx.org/source/src/event/ngx_event.c) 中定义如下：

```text
static ngx_str_t  event_core_name = ngx_string("event_core");


static ngx_command_t  ngx_event_core_commands[] = {

    
    { ngx_string("worker_connections"),
      NGX_EVENT_CONF|NGX_CONF_TAKE1,
      ngx_event_connections,
      0,
      0,
      NULL },

    
    { ngx_string("connections"),
      NGX_EVENT_CONF|NGX_CONF_TAKE1,
      ngx_event_connections,
      0,
      0,
      NULL },

    
    { ngx_string("use"),
      NGX_EVENT_CONF|NGX_CONF_TAKE1,
      ngx_event_use,
      0,
      0,
      NULL },

    
    { ngx_string("multi_accept"),
      NGX_EVENT_CONF|NGX_CONF_FLAG,
      ngx_conf_set_flag_slot,
      0,
      offsetof(ngx_event_conf_t, multi_accept),
      NULL },

    
    { ngx_string("accept_mutex"),
      NGX_EVENT_CONF|NGX_CONF_FLAG,
      ngx_conf_set_flag_slot,
      0,
      offsetof(ngx_event_conf_t, accept_mutex),
      NULL },

    
    { ngx_string("accept_mutex_delay"),
      NGX_EVENT_CONF|NGX_CONF_TAKE1,
      ngx_conf_set_msec_slot,
      0,
      offsetof(ngx_event_conf_t, accept_mutex_delay),
      NULL },

    
    { ngx_string("debug_connection"),
      NGX_EVENT_CONF|NGX_CONF_TAKE1,
      ngx_event_debug_connection,
      0,
      0,
      NULL },

      ngx_null_command
};

```

其中，每个事件模块都需要实现事件模块的通用接口结构 ngx\_event\_module\_t，ngx\_event\_core\_module 模块的上下文结构 ngx\_event\_core\_module\_ctx 并不真正的负责网络事件的驱动，所有不会实现ngx\_event\_module\_t 结构体中的成员 actions 中的方法。上下文结构 ngx\_event\_core\_module\_ctx 在文件 [src/event/ngx\_event.c](http://lxr.nginx.org/source/src/event/ngx_event.c) 中定义如下：

```text

ngx_event_module_t  ngx_event_core_module_ctx = {
    &event_core_name,
    ngx_event_core_create_conf,            
    ngx_event_core_init_conf,              
    { NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL }
};

```

在模块定义中，实现了两种方法分别为 ngx\_event\_module\_init 和ngx\_event\_process\_init 方法。在Nginx 启动过程中没有使用 fork 出 worker 子进程之前，先调用 ngx\_event\_core\_module 模块中的 ngx\_event\_module\_init 方法，当fork 出 worker 子进程后，每一个 worker 子进程则会调用 ngx\_event\_process\_init 方法。

ngx\_event\_module\_init 方法在文件[src/event/ngx\_event.c](http://lxr.nginx.org/source/src/event/ngx_event.c) 中定义：

```text

static ngx_int_t
ngx_event_module_init(ngx_cycle_t *cycle)
{
    void              ***cf;
    u_char              *shared;
    size_t               size, cl;
    ngx_shm_t            shm;
    ngx_time_t          *tp;
    ngx_core_conf_t     *ccf;
    ngx_event_conf_t    *ecf;

    
    cf = ngx_get_conf(cycle->conf_ctx, ngx_events_module);
    
    ecf = (*cf)[ngx_event_core_module.ctx_index];

    
    if (!ngx_test_config && ngx_process <= NGX_PROCESS_MASTER) {
        ngx_log_error(NGX_LOG_NOTICE, cycle->log, 0,
                      "using the \"%s\" event method", ecf->name);
    }

    
    ccf = (ngx_core_conf_t *) ngx_get_conf(cycle->conf_ctx, ngx_core_module);

    ngx_timer_resolution = ccf->timer_resolution;

#if !(NGX_WIN32)
    {
    ngx_int_t      limit;
    struct rlimit  rlmt;

    
    if (getrlimit(RLIMIT_NOFILE, &rlmt) == -1) {
        ngx_log_error(NGX_LOG_ALERT, cycle->log, ngx_errno,
                      "getrlimit(RLIMIT_NOFILE) failed, ignored");

    } else {
        
        if (ecf->connections > (ngx_uint_t) rlmt.rlim_cur
            && (ccf->rlimit_nofile == NGX_CONF_UNSET
                || ecf->connections > (ngx_uint_t) ccf->rlimit_nofile))
        {
            limit = (ccf->rlimit_nofile == NGX_CONF_UNSET) ?
                         (ngx_int_t) rlmt.rlim_cur : ccf->rlimit_nofile;

            ngx_log_error(NGX_LOG_WARN, cycle->log, 0,
                          "%ui worker_connections exceed "
                          "open file resource limit: %i",
                          ecf->connections, limit);
        }
    }
    }
#endif 

    
    if (ccf->master == 0) {
        return NGX_OK;
    }

    
    if (ngx_accept_mutex_ptr) {
        return NGX_OK;
    }

    
    

    
    cl = 128;

    
    size = cl            
           + cl          
           + cl;         

#if (NGX_STAT_STUB)

    
    size += cl           
           + cl          
           + cl          
           + cl          
           + cl          
           + cl          
           + cl;         

#endif

    
    shm.size = size;
    shm.name.len = sizeof("nginx_shared_zone");
    shm.name.data = (u_char *) "nginx_shared_zone";
    shm.log = cycle->log;

    
    if (ngx_shm_alloc(&shm) != NGX_OK) {
        return NGX_ERROR;
    }

    
    shared = shm.addr;

    ngx_accept_mutex_ptr = (ngx_atomic_t *) shared;
    
    ngx_accept_mutex.spin = (ngx_uint_t) -1;

    if (ngx_shmtx_create(&ngx_accept_mutex, (ngx_shmtx_sh_t *) shared,
                         cycle->lock_file.data)
        != NGX_OK)
    {
        return NGX_ERROR;
    }

    
    ngx_connection_counter = (ngx_atomic_t *) (shared + 1 * cl);

    (void) ngx_atomic_cmp_set(ngx_connection_counter, 0, 1);

    ngx_log_debug2(NGX_LOG_DEBUG_EVENT, cycle->log, 0,
                   "counter: %p, %d",
                   ngx_connection_counter, *ngx_connection_counter);

    ngx_temp_number = (ngx_atomic_t *) (shared + 2 * cl);

    tp = ngx_timeofday();

    ngx_random_number = (tp->msec << 16) + ngx_pid;

#if (NGX_STAT_STUB)

    ngx_stat_accepted = (ngx_atomic_t *) (shared + 3 * cl);
    ngx_stat_handled = (ngx_atomic_t *) (shared + 4 * cl);
    ngx_stat_requests = (ngx_atomic_t *) (shared + 5 * cl);
    ngx_stat_active = (ngx_atomic_t *) (shared + 6 * cl);
    ngx_stat_reading = (ngx_atomic_t *) (shared + 7 * cl);
    ngx_stat_writing = (ngx_atomic_t *) (shared + 8 * cl);
    ngx_stat_waiting = (ngx_atomic_t *) (shared + 9 * cl);

#endif

    return NGX_OK;
}

```

ngx\_event\_process\_init 方法在文件[src/event/ngx\_event.c](http://lxr.nginx.org/source/src/event/ngx_event.c) 中定义：

```text
static ngx_int_t
ngx_event_process_init(ngx_cycle_t *cycle)
{
    ngx_uint_t           m, i;
    ngx_event_t         *rev, *wev;
    ngx_listening_t     *ls;
    ngx_connection_t    *c, *next, *old;
    ngx_core_conf_t     *ccf;
    ngx_event_conf_t    *ecf;
    ngx_event_module_t  *module;

    
    ccf = (ngx_core_conf_t *) ngx_get_conf(cycle->conf_ctx, ngx_core_module);
    
    ecf = ngx_event_get_conf(cycle->conf_ctx, ngx_event_core_module);

    
    if (ccf->master && ccf->worker_processes > 1 && ecf->accept_mutex) {
        ngx_use_accept_mutex = 1;
        ngx_accept_mutex_held = 0;
        ngx_accept_mutex_delay = ecf->accept_mutex_delay;

    } else {
        ngx_use_accept_mutex = 0;
    }

#if (NGX_WIN32)

    

    ngx_use_accept_mutex = 0;

#endif

    ngx_queue_init(&ngx_posted_accept_events);
    ngx_queue_init(&ngx_posted_events);

    
    if (ngx_event_timer_init(cycle->log) == NGX_ERROR) {
        return NGX_ERROR;
    }

    
    for (m = 0; ngx_modules[m]; m++) {
        if (ngx_modules[m]->type != NGX_EVENT_MODULE) {
            continue;
        }

        if (ngx_modules[m]->ctx_index != ecf->use) {
            continue;
        }

        module = ngx_modules[m]->ctx;

        if (module->actions.init(cycle, ngx_timer_resolution) != NGX_OK) {
            
            exit(2);
        }

        break;
    }

#if !(NGX_WIN32)

    
    if (ngx_timer_resolution && !(ngx_event_flags & NGX_USE_TIMER_EVENT)) {
        struct sigaction  sa;
        struct itimerval  itv;

        ngx_memzero(&sa, sizeof(struct sigaction));
        
        
        sa.sa_handler = ngx_timer_signal_handler;
        
        sigemptyset(&sa.sa_mask);

        
        if (sigaction(SIGALRM, &sa, NULL) == -1) {
            ngx_log_error(NGX_LOG_ALERT, cycle->log, ngx_errno,
                          "sigaction(SIGALRM) failed");
            return NGX_ERROR;
        }

        
        itv.it_interval.tv_sec = ngx_timer_resolution / 1000;
        itv.it_interval.tv_usec = (ngx_timer_resolution % 1000) * 1000;
        itv.it_value.tv_sec = ngx_timer_resolution / 1000;
        itv.it_value.tv_usec = (ngx_timer_resolution % 1000 ) * 1000;

        
        if (setitimer(ITIMER_REAL, &itv, NULL) == -1) {
            ngx_log_error(NGX_LOG_ALERT, cycle->log, ngx_errno,
                          "setitimer() failed");
        }
    }

    
    if (ngx_event_flags & NGX_USE_FD_EVENT) {
        struct rlimit  rlmt;

        if (getrlimit(RLIMIT_NOFILE, &rlmt) == -1) {
            ngx_log_error(NGX_LOG_ALERT, cycle->log, ngx_errno,
                          "getrlimit(RLIMIT_NOFILE) failed");
            return NGX_ERROR;
        }

        cycle->files_n = (ngx_uint_t) rlmt.rlim_cur;

        cycle->files = ngx_calloc(sizeof(ngx_connection_t *) * cycle->files_n,
                                  cycle->log);
        if (cycle->files == NULL) {
            return NGX_ERROR;
        }
    }

#endif

    
    cycle->connections =
        ngx_alloc(sizeof(ngx_connection_t) * cycle->connection_n, cycle->log);
    if (cycle->connections == NULL) {
        return NGX_ERROR;
    }

    c = cycle->connections;

    
    cycle->read_events = ngx_alloc(sizeof(ngx_event_t) * cycle->connection_n,
                                   cycle->log);
    if (cycle->read_events == NULL) {
        return NGX_ERROR;
    }

    rev = cycle->read_events;
    for (i = 0; i < cycle->connection_n; i++) {
        rev[i].closed = 1;
        rev[i].instance = 1;
    }

    
    cycle->write_events = ngx_alloc(sizeof(ngx_event_t) * cycle->connection_n,
                                    cycle->log);
    if (cycle->write_events == NULL) {
        return NGX_ERROR;
    }

    wev = cycle->write_events;
    for (i = 0; i < cycle->connection_n; i++) {
        wev[i].closed = 1;
    }

    i = cycle->connection_n;
    next = NULL;

    
    do {
        i--;

        c[i].data = next;
        c[i].read = &cycle->read_events[i];
        c[i].write = &cycle->write_events[i];
        c[i].fd = (ngx_socket_t) -1;

        next = &c[i];

#if (NGX_THREADS)
        c[i].lock = 0;
#endif
    } while (i);

    
    cycle->free_connections = next;
    cycle->free_connection_n = cycle->connection_n;

    

    
    ls = cycle->listening.elts;
    for (i = 0; i < cycle->listening.nelts; i++) {

        
        c = ngx_get_connection(ls[i].fd, cycle->log);

        if (c == NULL) {
            return NGX_ERROR;
        }

        c->log = &ls[i].log;

        c->listening = &ls[i];
        ls[i].connection = c;

        rev = c->read;

        rev->log = c->log;
        rev->accept = 1;

#if (NGX_HAVE_DEFERRED_ACCEPT)
        rev->deferred_accept = ls[i].deferred_accept;
#endif

        if (!(ngx_event_flags & NGX_USE_IOCP_EVENT)) {
            if (ls[i].previous) {

                

                old = ls[i].previous->connection;

                if (ngx_del_event(old->read, NGX_READ_EVENT, NGX_CLOSE_EVENT)
                    == NGX_ERROR)
                {
                    return NGX_ERROR;
                }

                old->fd = (ngx_socket_t) -1;
            }
        }

#if (NGX_WIN32)

        if (ngx_event_flags & NGX_USE_IOCP_EVENT) {
            ngx_iocp_conf_t  *iocpcf;

            rev->handler = ngx_event_acceptex;

            if (ngx_use_accept_mutex) {
                continue;
            }

            if (ngx_add_event(rev, 0, NGX_IOCP_ACCEPT) == NGX_ERROR) {
                return NGX_ERROR;
            }

            ls[i].log.handler = ngx_acceptex_log_error;

            iocpcf = ngx_event_get_conf(cycle->conf_ctx, ngx_iocp_module);
            if (ngx_event_post_acceptex(&ls[i], iocpcf->post_acceptex)
                == NGX_ERROR)
            {
                return NGX_ERROR;
            }

        } else {
            rev->handler = ngx_event_accept;

            if (ngx_use_accept_mutex) {
                continue;
            }

            if (ngx_add_event(rev, NGX_READ_EVENT, 0) == NGX_ERROR) {
                return NGX_ERROR;
            }
        }

#else

        
        rev->handler = ngx_event_accept;

        if (ngx_use_accept_mutex) {
            continue;
        }

        if (ngx_event_flags & NGX_USE_RTSIG_EVENT) {
            if (ngx_add_conn(c) == NGX_ERROR) {
                return NGX_ERROR;
            }

        } else {
        
            if (ngx_add_event(rev, NGX_READ_EVENT, 0) == NGX_ERROR) {
                return NGX_ERROR;
            }
        }

#endif

    }

    return NGX_OK;
}

```

参考资料：

《深入理解 Nginx 》

《[nginx事件模块分析\(二\)](http://blog.csdn.net/freeinfor/article/details/16343223)》

