# Nginx 事件驱动模块连接处理

## 概述

由于 Nginx 工作在 master-worker 多进程模式，若所有 worker 进程在同一时间监听同一个端口，当该端口有新的连接事件出现时，每个worker 进程都会调用函数ngx\_event\_accept 试图与新的连接建立通信，即所有worker 进程都会被唤醒，这就是所谓的“惊群”问题，这样会导致系统性能下降。幸好在Nginx 采用了ngx\_accept\_mutex 同步锁机制，即只有获得该锁的worker 进程才能去处理新的连接事件，也就在同一时间只能有一个worker 进程监听某个端口。虽然这样做解决了“惊群”问题，但是随之会出现另一个问题，若每次出现的新连接事件都被同一个worker 进程获得锁的权利并处理该连接事件，这样会导致进程之间不均衡的状态，即在所有worker 进程中，某些进程处理的连接事件数量很庞大，而某些进程基本上不用处理连接事件，一直处于空闲状态。因此，这样会导致worker 进程之间的负载不均衡，会影响Nginx 的整体性能。为了解决负载失衡的问题，Nginx 在已经实现同步锁的基础上定义了负载阈值ngx\_accept\_disabled，当某个worker 进程的负载阈值大于 0 时，表示该进程处于负载超重的状态，则Nginx 会控制该进程，使其没机会试图与新的连接事件进行通信，这样就会为其他没有负载超重的进程创造了处理新连接事件的机会，以此达到进程间的负载均衡。

## 连接事件处理

新连接事件由函数 ngx\_event\_accept 处理。

```text

void
ngx_event_accept(ngx_event_t *ev)
{
    socklen_t          socklen;
    ngx_err_t          err;
    ngx_log_t         *log;
    ngx_uint_t         level;
    ngx_socket_t       s;
    ngx_event_t       *rev, *wev;
    ngx_listening_t   *ls;
    ngx_connection_t  *c, *lc;
    ngx_event_conf_t  *ecf;
    u_char             sa[NGX_SOCKADDRLEN];
#if (NGX_HAVE_ACCEPT4)
    static ngx_uint_t  use_accept4 = 1;
#endif

    if (ev->timedout) {
        if (ngx_enable_accept_events((ngx_cycle_t *) ngx_cycle) != NGX_OK) {
            return;
        }

        ev->timedout = 0;
    }

    
    ecf = ngx_event_get_conf(ngx_cycle->conf_ctx, ngx_event_core_module);

    if (ngx_event_flags & NGX_USE_RTSIG_EVENT) {
        ev->available = 1;

    } else if (!(ngx_event_flags & NGX_USE_KQUEUE_EVENT)) {
        ev->available = ecf->multi_accept;
    }

    lc = ev->data;
    ls = lc->listening;
    ev->ready = 0;

    ngx_log_debug2(NGX_LOG_DEBUG_EVENT, ev->log, 0,
                   "accept on %V, ready: %d", &ls->addr_text, ev->available);

    do {
        socklen = NGX_SOCKADDRLEN;

        
#if (NGX_HAVE_ACCEPT4)
        if (use_accept4) {
            s = accept4(lc->fd, (struct sockaddr *) sa, &socklen,
                        SOCK_NONBLOCK);
        } else {
            s = accept(lc->fd, (struct sockaddr *) sa, &socklen);
        }
#else
        s = accept(lc->fd, (struct sockaddr *) sa, &socklen);
#endif

        
        if (s == (ngx_socket_t) -1) {
            err = ngx_socket_errno;

            if (err == NGX_EAGAIN) {
                ngx_log_debug0(NGX_LOG_DEBUG_EVENT, ev->log, err,
                               "accept() not ready");
                return;
            }

            level = NGX_LOG_ALERT;

            if (err == NGX_ECONNABORTED) {
                level = NGX_LOG_ERR;

            } else if (err == NGX_EMFILE || err == NGX_ENFILE) {
                level = NGX_LOG_CRIT;
            }

#if (NGX_HAVE_ACCEPT4)
            ngx_log_error(level, ev->log, err,
                          use_accept4 ? "accept4() failed" : "accept() failed");

            if (use_accept4 && err == NGX_ENOSYS) {
                use_accept4 = 0;
                ngx_inherited_nonblocking = 0;
                continue;
            }
#else
            ngx_log_error(level, ev->log, err, "accept() failed");
#endif

            if (err == NGX_ECONNABORTED) {
                if (ngx_event_flags & NGX_USE_KQUEUE_EVENT) {
                    ev->available--;
                }

                if (ev->available) {
                    continue;
                }
            }

            if (err == NGX_EMFILE || err == NGX_ENFILE) {
                if (ngx_disable_accept_events((ngx_cycle_t *) ngx_cycle)
                    != NGX_OK)
                {
                    return;
                }

                if (ngx_use_accept_mutex) {
                    if (ngx_accept_mutex_held) {
                        ngx_shmtx_unlock(&ngx_accept_mutex);
                        ngx_accept_mutex_held = 0;
                    }

                    ngx_accept_disabled = 1;

                } else {
                    ngx_add_timer(ev, ecf->accept_mutex_delay);
                }
            }

            return;
        }

#if (NGX_STAT_STUB)
        (void) ngx_atomic_fetch_add(ngx_stat_accepted, 1);
#endif

        
        ngx_accept_disabled = ngx_cycle->connection_n / 8
                              - ngx_cycle->free_connection_n;

        
        c = ngx_get_connection(s, ev->log);

        if (c == NULL) {
            if (ngx_close_socket(s) == -1) {
                ngx_log_error(NGX_LOG_ALERT, ev->log, ngx_socket_errno,
                              ngx_close_socket_n " failed");
            }

            return;
        }

#if (NGX_STAT_STUB)
        (void) ngx_atomic_fetch_add(ngx_stat_active, 1);
#endif

        
        c->pool = ngx_create_pool(ls->pool_size, ev->log);
        if (c->pool == NULL) {
            ngx_close_accepted_connection(c);
            return;
        }

        c->sockaddr = ngx_palloc(c->pool, socklen);
        if (c->sockaddr == NULL) {
            ngx_close_accepted_connection(c);
            return;
        }

        ngx_memcpy(c->sockaddr, sa, socklen);

        log = ngx_palloc(c->pool, sizeof(ngx_log_t));
        if (log == NULL) {
            ngx_close_accepted_connection(c);
            return;
        }

        

        
        if (ngx_inherited_nonblocking) {
            if (ngx_event_flags & NGX_USE_AIO_EVENT) {
                if (ngx_blocking(s) == -1) {
                    ngx_log_error(NGX_LOG_ALERT, ev->log, ngx_socket_errno,
                                  ngx_blocking_n " failed");
                    ngx_close_accepted_connection(c);
                    return;
                }
            }

        } else {
            
            if (!(ngx_event_flags & (NGX_USE_AIO_EVENT|NGX_USE_RTSIG_EVENT))) {
                if (ngx_nonblocking(s) == -1) {
                    ngx_log_error(NGX_LOG_ALERT, ev->log, ngx_socket_errno,
                                  ngx_nonblocking_n " failed");
                    ngx_close_accepted_connection(c);
                    return;
                }
            }
        }

        *log = ls->log;

        
        c->recv = ngx_recv;
        c->send = ngx_send;
        c->recv_chain = ngx_recv_chain;
        c->send_chain = ngx_send_chain;

        c->log = log;
        c->pool->log = log;

        c->socklen = socklen;
        c->listening = ls;
        c->local_sockaddr = ls->sockaddr;
        c->local_socklen = ls->socklen;

        c->unexpected_eof = 1;

#if (NGX_HAVE_UNIX_DOMAIN)
        if (c->sockaddr->sa_family == AF_UNIX) {
            c->tcp_nopush = NGX_TCP_NOPUSH_DISABLED;
            c->tcp_nodelay = NGX_TCP_NODELAY_DISABLED;
#if (NGX_SOLARIS)
            
            c->sendfile = 0;
#endif
        }
#endif

        
        rev = c->read;
        wev = c->write;

        
        wev->ready = 1;

        if (ngx_event_flags & (NGX_USE_AIO_EVENT|NGX_USE_RTSIG_EVENT)) {
            
            rev->ready = 1;
        }

        if (ev->deferred_accept) {
            rev->ready = 1;
#if (NGX_HAVE_KQUEUE)
            rev->available = 1;
#endif
        }

        rev->log = log;
        wev->log = log;

        

        c->number = ngx_atomic_fetch_add(ngx_connection_counter, 1);

#if (NGX_STAT_STUB)
        (void) ngx_atomic_fetch_add(ngx_stat_handled, 1);
#endif

#if (NGX_THREADS)
        rev->lock = &c->lock;
        wev->lock = &c->lock;
        rev->own_lock = &c->lock;
        wev->own_lock = &c->lock;
#endif

        if (ls->addr_ntop) {
            c->addr_text.data = ngx_pnalloc(c->pool, ls->addr_text_max_len);
            if (c->addr_text.data == NULL) {
                ngx_close_accepted_connection(c);
                return;
            }

            c->addr_text.len = ngx_sock_ntop(c->sockaddr, c->socklen,
                                             c->addr_text.data,
                                             ls->addr_text_max_len, 0);
            if (c->addr_text.len == 0) {
                ngx_close_accepted_connection(c);
                return;
            }
        }

#if (NGX_DEBUG)
        {

        struct sockaddr_in   *sin;
        ngx_cidr_t           *cidr;
        ngx_uint_t            i;
#if (NGX_HAVE_INET6)
        struct sockaddr_in6  *sin6;
        ngx_uint_t            n;
#endif

        cidr = ecf->debug_connection.elts;
        for (i = 0; i < ecf->debug_connection.nelts; i++) {
            if (cidr[i].family != (ngx_uint_t) c->sockaddr->sa_family) {
                goto next;
            }

            switch (cidr[i].family) {

#if (NGX_HAVE_INET6)
            case AF_INET6:
                sin6 = (struct sockaddr_in6 *) c->sockaddr;
                for (n = 0; n < 16; n++) {
                    if ((sin6->sin6_addr.s6_addr[n]
                        & cidr[i].u.in6.mask.s6_addr[n])
                        != cidr[i].u.in6.addr.s6_addr[n])
                    {
                        goto next;
                    }
                }
                break;
#endif

#if (NGX_HAVE_UNIX_DOMAIN)
            case AF_UNIX:
                break;
#endif

            default: 
                sin = (struct sockaddr_in *) c->sockaddr;
                if ((sin->sin_addr.s_addr & cidr[i].u.in.mask)
                    != cidr[i].u.in.addr)
                {
                    goto next;
                }
                break;
            }

            log->log_level = NGX_LOG_DEBUG_CONNECTION|NGX_LOG_DEBUG_ALL;
            break;

        next:
            continue;
        }

        }
#endif

        ngx_log_debug3(NGX_LOG_DEBUG_EVENT, log, 0,
                       "*%uA accept: %V fd:%d", c->number, &c->addr_text, s);

        
        if (ngx_add_conn && (ngx_event_flags & NGX_USE_EPOLL_EVENT) == 0) {
            if (ngx_add_conn(c) == NGX_ERROR) {
                ngx_close_accepted_connection(c);
                return;
            }
        }

        log->data = NULL;
        log->handler = NULL;

        
        ls->handler(c);

        
        if (ngx_event_flags & NGX_USE_KQUEUE_EVENT) {
            ev->available--;
        }

    } while (ev->available);
}


static ngx_int_t
ngx_enable_accept_events(ngx_cycle_t *cycle)
{
    ngx_uint_t         i;
    ngx_listening_t   *ls;
    ngx_connection_t  *c;

    
    ls = cycle->listening.elts;
    
    for (i = 0; i < cycle->listening.nelts; i++) {

        
        c = ls[i].connection;

        
        if (c->read->active) {
            
            continue;
        }

        
        if (ngx_event_flags & NGX_USE_RTSIG_EVENT) {

            if (ngx_add_conn(c) == NGX_ERROR) {
                return NGX_ERROR;
            }

        } else {
            
            if (ngx_add_event(c->read, NGX_READ_EVENT, 0) == NGX_ERROR) {
                return NGX_ERROR;
            }
        }
    }

    return NGX_OK;
}


static ngx_int_t
ngx_disable_accept_events(ngx_cycle_t *cycle)
{
    ngx_uint_t         i;
    ngx_listening_t   *ls;
    ngx_connection_t  *c;

    
    ls = cycle->listening.elts;
    for (i = 0; i < cycle->listening.nelts; i++) {

        
        c = ls[i].connection;

        if (!c->read->active) {
            continue;
        }

        
        if (ngx_event_flags & NGX_USE_RTSIG_EVENT) {
            if (ngx_del_conn(c, NGX_DISABLE_EVENT) == NGX_ERROR) {
                return NGX_ERROR;
            }

        } else {
            
            if (ngx_del_event(c->read, NGX_READ_EVENT, NGX_DISABLE_EVENT)
                == NGX_ERROR)
            {
                return NGX_ERROR;
            }
        }
    }

    return NGX_OK;
}

```

当出现新连接事件时，只有获得同步锁的进程才可以处理该连接事件，避免了“惊群”问题，进程试图处理新连接事件由函数 ngx\_trylock\_accept\_mutex 实现。

```text

ngx_int_t
ngx_trylock_accept_mutex(ngx_cycle_t *cycle)
{
    
    if (ngx_shmtx_trylock(&ngx_accept_mutex)) {

        ngx_log_debug0(NGX_LOG_DEBUG_EVENT, cycle->log, 0,
                       "accept mutex locked");

        
        if (ngx_accept_mutex_held
            && ngx_accept_events == 0
            && !(ngx_event_flags & NGX_USE_RTSIG_EVENT))
        {
            return NGX_OK;
        }

        
        if (ngx_enable_accept_events(cycle) == NGX_ERROR) {
            
            ngx_shmtx_unlock(&ngx_accept_mutex);
            return NGX_ERROR;
        }

        
        ngx_accept_events = 0;
        ngx_accept_mutex_held = 1;

        return NGX_OK;
    }

    ngx_log_debug1(NGX_LOG_DEBUG_EVENT, cycle->log, 0,
                   "accept mutex lock failed: %ui", ngx_accept_mutex_held);

    
    if (ngx_accept_mutex_held) {
        
        if (ngx_disable_accept_events(cycle) == NGX_ERROR) {
            return NGX_ERROR;
        }

        ngx_accept_mutex_held = 0;
    }

    return NGX_OK;
}

```

Nginx 通过负载阈值 ngx\_accept\_disabled 控制进程是否处理新连接事件，避免进程间负载均衡问题。

```text
if(ngx_accept_disabled > 0){
   ngx_accept_disabled --;
}else{
  if(ngx_trylock_accept_mutex(cycle) == NGX_ERROR){
        return;
   }
...
}

```

参考资料：

《深入理解Nginx》

《深入剖析Nginx》

《[Nginx源码分析-事件循环](http://www.alidata.org/archives/1267)》

《关[于ngx\_trylock\_accept\_mutex的一些解释](http://blog.csdn.net/brainkick/article/details/9081017)》

