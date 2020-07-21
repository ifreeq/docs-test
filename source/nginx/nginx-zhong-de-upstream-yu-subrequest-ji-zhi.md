# Nginx 中的 upstream 与 subrequest 机制

## 概述

Nginx 提供了两种全异步方式与第三方服务进行通信：**upstream**和**subrequest**。upstream 在与第三方服务器交互时（包括建立TCP 连接、发送请求、接收响应、关闭TCP 连接），不会阻塞Nginx 进程处理其他请求。subrequest 只是分解复杂请求的一种设计模式，它可以把原始请求分解为多个子请求，使得诸多请求协同完成一个用户请求，并且每个请求只关注一个功能。subrequest 访问第三方服务最终也是基于upstream 实现的。

upstream 被定义为访问上游服务器，它把Nginx 定义为反代理服务器，首要功能是透传，其次才是以TCP 获取第三方服务器的内容。Nginx 的HTTP 反向代理模块是基于 upstream 方式实现的。subrequest 是子请求，也就是说subrequest 将会为用户创建子请求，即将一个复杂的请求分解为多个子请求，每个子请求负责一种功能项，而最初的原始请求负责构成并发送响应给用户。当subrequest 访问第三服务时，首先派生出子请求访问上游服务器，父请求在完全取得上游服务器的响应后再决定如何处理来自客户端的请求。

因此，若希望把是第三方服务的内容原封不动地返回给用户时，则使用 upstream 方式。若访问第三方服务是为了获取某些信息，再根据这些信息来构造响应并发给用户，则应使用 subrequest 方式。

## upstream 使用方式

upstream 模块不产生自己的内容，而是通过请求后端服务器得到内容。Nginx 内部封装了请求并取得响应内容的整个过程，所以upstream 模块只需要开发若干回调函数，完成构造请求和解析响应等具体的工作。

## ngx\_http\_request\_t 结构体

首先了解 upstream 是如何嵌入到一个请求中，这里必须从请求结构体 ngx\_http\_request\_t  入手，在该结构体中具有一个ngx\_http\_upstream\_t  结构体类型的成员upstream。请求结构体 ngx\_http\_request\_t 定义在文件 [src/http/ngx\_http\_request.h](http://lxr.nginx.org/source/src/http/ngx_http_request.h) 中如下:

```text
struct ngx_http_request_s {
    uint32_t                          signature;         

    
    ngx_connection_t                 *connection;

    
    void                            **ctx;
    void                            **main_conf;
    void                            **srv_conf;
    void                            **loc_conf;

    
    ngx_http_event_handler_pt         read_event_handler;
    ngx_http_event_handler_pt         write_event_handler;

#if (NGX_HTTP_CACHE)
    ngx_http_cache_t                 *cache;
#endif

    
    ngx_http_upstream_t              *upstream;
    ngx_array_t                      *upstream_states;
                                         

    
    ngx_pool_t                       *pool;
    
    ngx_buf_t                        *header_in;

    
    ngx_http_headers_in_t             headers_in;
    
    ngx_http_headers_out_t            headers_out;

    
    ngx_http_request_body_t          *request_body;

    
    time_t                            lingering_time;
    
    time_t                            start_sec;
    
    ngx_msec_t                        start_msec;

    
    ngx_uint_t                        method;       
    ngx_uint_t                        http_version; 

    ngx_str_t                         request_line; 
    ngx_str_t                         uri;          
    ngx_str_t                         args;         
    ngx_str_t                         exten;        
    ngx_str_t                         unparsed_uri; 

    ngx_str_t                         method_name;  
    ngx_str_t                         http_protocol;

    
    ngx_chain_t                      *out;
    
    ngx_http_request_t               *main;
    
    ngx_http_request_t               *parent;
    
    ngx_http_postponed_request_t     *postponed;
    ngx_http_post_subrequest_t       *post_subrequest;
    
    ngx_http_posted_request_t        *posted_requests;

    
    ngx_int_t                         phase_handler;
    
    ngx_http_handler_pt               content_handler;
    
    ngx_uint_t                        access_code;

    ngx_http_variable_value_t        *variables;

#if (NGX_PCRE)
    ngx_uint_t                        ncaptures;
    int                              *captures;
    u_char                           *captures_data;
#endif

    
    size_t                            limit_rate;
    size_t                            limit_rate_after;

    
    
    size_t                            header_size;

    
    off_t                             request_length;

    
    ngx_uint_t                        err_status;

    
    ngx_http_connection_t            *http_connection;
#if (NGX_HTTP_SPDY)
    ngx_http_spdy_stream_t           *spdy_stream;
#endif

    
    ngx_http_log_handler_pt           log_handler;

    
    ngx_http_cleanup_t               *cleanup;

    
    
    unsigned                          subrequests:8;
    
    unsigned                          count:8;
    
    unsigned                          blocked:8;

    
    unsigned                          aio:1;

    unsigned                          http_state:4;

    
    unsigned                          complex_uri:1;

    
    unsigned                          quoted_uri:1;

    
    unsigned                          plus_in_uri:1;

    
    unsigned                          space_in_uri:1;

    unsigned                          invalid_header:1;

    unsigned                          add_uri_to_alias:1;
    unsigned                          valid_location:1;
    unsigned                          valid_unparsed_uri:1;
    
    unsigned                          uri_changed:1;
    
    unsigned                          uri_changes:4;

    unsigned                          request_body_in_single_buf:1;
    unsigned                          request_body_in_file_only:1;
    unsigned                          request_body_in_persistent_file:1;
    unsigned                          request_body_in_clean_file:1;
    unsigned                          request_body_file_group_access:1;
    unsigned                          request_body_file_log_level:3;

    
    unsigned                          subrequest_in_memory:1;
    unsigned                          waited:1;

#if (NGX_HTTP_CACHE)
    unsigned                          cached:1;
#endif

#if (NGX_HTTP_GZIP)
    unsigned                          gzip_tested:1;
    unsigned                          gzip_ok:1;
    unsigned                          gzip_vary:1;
#endif

    unsigned                          proxy:1;
    unsigned                          bypass_cache:1;
    unsigned                          no_cache:1;

    
    unsigned                          limit_conn_set:1;
    unsigned                          limit_req_set:1;

#if 0
    unsigned                          cacheable:1;
#endif

    unsigned                          pipeline:1;
    unsigned                          chunked:1;
    unsigned                          header_only:1;
    
    unsigned                          keepalive:1;
    
    unsigned                          lingering_close:1;
    
    unsigned                          discard_body:1;
    
    unsigned                          internal:1;
    unsigned                          error_page:1;
    unsigned                          ignore_content_encoding:1;
    unsigned                          filter_finalize:1;
    unsigned                          post_action:1;
    unsigned                          request_complete:1;
    unsigned                          request_output:1;
    
    unsigned                          header_sent:1;
    unsigned                          expect_tested:1;
    unsigned                          root_tested:1;
    unsigned                          done:1;
    unsigned                          logged:1;

    
    unsigned                          buffered:4;

    unsigned                          main_filter_need_in_memory:1;
    unsigned                          filter_need_in_memory:1;
    unsigned                          filter_need_temporary:1;
    unsigned                          allow_ranges:1;
    unsigned                          single_range:1;

#if (NGX_STAT_STUB)
    unsigned                          stat_reading:1;
    unsigned                          stat_writing:1;
#endif

    

    
    ngx_uint_t                        state;

    ngx_uint_t                        header_hash;
    ngx_uint_t                        lowcase_index;
    u_char                            lowcase_header[NGX_HTTP_LC_HEADER_LEN];

    u_char                           *header_name_start;
    u_char                           *header_name_end;
    u_char                           *header_start;
    u_char                           *header_end;

    

    u_char                           *uri_start;
    u_char                           *uri_end;
    u_char                           *uri_ext;
    u_char                           *args_start;
    u_char                           *request_start;
    u_char                           *request_end;
    u_char                           *method_end;
    u_char                           *schema_start;
    u_char                           *schema_end;
    u_char                           *host_start;
    u_char                           *host_end;
    u_char                           *port_start;
    u_char                           *port_end;

    unsigned                          http_minor:16;
    unsigned                          http_major:16;
};

```

若没有实现 upstream 机制，则请求结构体 ngx\_http\_request\_t 中的upstream成员设置为NULL，否则必须设置该成员。首先看下 HTTP 模块启动upstream 机制的过程：

1. 调用函数 ngx\_http\_upstream\_create 为请求创建upstream；
2. 设置上游服务器的地址；可通过配置文件 nginx.conf 配置好上游服务器地址；也可以通过ngx\_http\_request\_t 中的成员resolved 设置上游服务器地址；
3. 设置 upstream 的回调方法；
4. 调用函数 ngx\_http\_upstream\_init 启动upstream；

upstream 启动过程如下图所示：

## ngx\_http\_upstream\_t 结构体

upstream 结构体是 ngx\_http\_upstream\_t，该结构体只在 upstream 模块内部使用，其定义在文件：[src/http/ngx\_http\_upstream.h](http://lxr.nginx.org/source/src/http/ngx_http_upstream.h)

```text

struct ngx_http_upstream_s {
    
    ngx_http_upstream_handler_pt     read_event_handler;
    
    ngx_http_upstream_handler_pt     write_event_handler;

    
    ngx_peer_connection_t            peer;

    
    ngx_event_pipe_t                *pipe;

    
    ngx_chain_t                     *request_bufs;

    
    ngx_output_chain_ctx_t           output;
    ngx_chain_writer_ctx_t           writer;

    
    ngx_http_upstream_conf_t        *conf;

    
    ngx_http_upstream_headers_in_t   headers_in;

    
    ngx_http_upstream_resolved_t    *resolved;

    
    ngx_buf_t                        from_client;

    
    ngx_buf_t                        buffer;
    off_t                            length;

    
    ngx_chain_t                     *out_bufs;
    
    ngx_chain_t                     *busy_bufs;
    
    ngx_chain_t                     *free_bufs;

    
    ngx_int_t                      (*input_filter_init)(void *data);
    
    ngx_int_t                      (*input_filter)(void *data, ssize_t bytes);
    
    void                            *input_filter_ctx;

#if (NGX_HTTP_CACHE)
    ngx_int_t                      (*create_key)(ngx_http_request_t *r);
#endif
    
    ngx_int_t                      (*create_request)(ngx_http_request_t *r);
    
    ngx_int_t                      (*reinit_request)(ngx_http_request_t *r);
    
    ngx_int_t                      (*process_header)(ngx_http_request_t *r);
    
    void                           (*abort_request)(ngx_http_request_t *r);
    
    void                           (*finalize_request)(ngx_http_request_t *r,
                                         ngx_int_t rc);
    
    ngx_int_t                      (*rewrite_redirect)(ngx_http_request_t *r,
                                         ngx_table_elt_t *h, size_t prefix);
    ngx_int_t                      (*rewrite_cookie)(ngx_http_request_t *r,
                                         ngx_table_elt_t *h);

    ngx_msec_t                       timeout;

    
    ngx_http_upstream_state_t       *state;

    ngx_str_t                        method;
    
    ngx_str_t                        schema;
    ngx_str_t                        uri;

    
    ngx_http_cleanup_pt             *cleanup;

    

    
    unsigned                         store:1;
    
    unsigned                         cacheable:1;
    unsigned                         accel:1;
    
    unsigned                         ssl:1;
#if (NGX_HTTP_CACHE)
    unsigned                         cache_status:3;
#endif

    
    unsigned                         buffering:1;
    
    unsigned                         keepalive:1;
    unsigned                         upgrade:1;

    
    unsigned                         request_sent:1;
    
    unsigned                         header_sent:1;
};

```

下面看下 upstream 处理上游响应包体的三种方式：

1. 当请求结构体 ngx\_http\_request\_t 中的成员subrequest\_in\_memory 标志位为 1 时，upstream 不转发响应包体到下游，并由HTTP 模块实现的 input\_filter\(\) 方法处理包体；
2. 当请求结构体 ngx\_http\_request\_t 中的成员subrequest\_in\_memory 标志位为 0 时，且ngx\_http\_upstream\_conf\_t 配置结构体中的成员buffering 标志位为 1 时，upstream 将开启更多的内存和磁盘文件用于缓存上游的响应包体（此时，上游网速更快），并转发响应包体；
3. 当请求结构体 ngx\_http\_request\_t 中的成员subrequest\_in\_memory 标志位为 0 时，且ngx\_http\_upstream\_conf\_t 配置结构体中的成员buffering 标志位为 0 时，upstream 将使用固定大小的缓冲区来转发响应包体；

## ngx\_http\_upstream\_conf\_t 结构体

在结构体 ngx\_http\_upstream\_t 的成员conf 中，conf 是一个结构体ngx\_http\_upstream\_conf\_t 变量，该变量设置了upstream 的限制性参数。ngx\_http\_upstream\_conf\_t 结构体定义如下：[src/http/ngx\_http\_upstream.h](http://lxr.nginx.org/source/src/http/ngx_http_upstream.h)

```text

typedef struct {
    
    ngx_http_upstream_srv_conf_t    *upstream;

    
    ngx_msec_t                       connect_timeout;
    
    ngx_msec_t                       send_timeout;
    
    ngx_msec_t                       read_timeout;
    ngx_msec_t                       timeout;

    
    size_t                           send_lowat;
    
    size_t                           buffer_size;

    size_t                           busy_buffers_size;
    
    size_t                           max_temp_file_size;
    
    size_t                           temp_file_write_size;

    size_t                           busy_buffers_size_conf;
    size_t                           max_temp_file_size_conf;
    size_t                           temp_file_write_size_conf;

    
    ngx_bufs_t                       bufs;

    
    ngx_uint_t                       ignore_headers;
    
    ngx_uint_t                       next_upstream;
    
    ngx_uint_t                       store_access;
    
    ngx_flag_t                       buffering;
    ngx_flag_t                       pass_request_headers;
    ngx_flag_t                       pass_request_body;

    
    ngx_flag_t                       ignore_client_abort;
    ngx_flag_t                       intercept_errors;
    
    ngx_flag_t                       cyclic_temp_file;

    
    ngx_path_t                      *temp_path;

    
    ngx_hash_t                       hide_headers_hash;
    
    ngx_array_t                     *hide_headers;
    
    ngx_array_t                     *pass_headers;

    
    ngx_http_upstream_local_t       *local;

#if (NGX_HTTP_CACHE)
    ngx_shm_zone_t                  *cache;

    ngx_uint_t                       cache_min_uses;
    ngx_uint_t                       cache_use_stale;
    ngx_uint_t                       cache_methods;

    ngx_flag_t                       cache_lock;
    ngx_msec_t                       cache_lock_timeout;

    ngx_flag_t                       cache_revalidate;

    ngx_array_t                     *cache_valid;
    ngx_array_t                     *cache_bypass;
    ngx_array_t                     *no_cache;
#endif

    
    ngx_array_t                     *store_lengths;
    ngx_array_t                     *store_values;

    
    signed                           store:2;
    
    unsigned                         intercept_404:1;
    
    unsigned                         change_buffering:1;

#if (NGX_HTTP_SSL)
    ngx_ssl_t                       *ssl;
    ngx_flag_t                       ssl_session_reuse;
#endif

    
    ngx_str_t                        module;
} ngx_http_upstream_conf_t;

```

在 HTTP 反向代理模块在配置文件 nginx.conf 提供的配置项大都是用来设置结构体 ngx\_http\_upstream\_conf\_t 的成员。3 个超时时间成员是必须要设置的，因为他们默认是 0，即若不设置这 3 个成员，则无法与上游服务器建立TCP 连接。每一个请求都有独立的ngx\_http\_upstream\_conf\_t 结构体，因此，每个请求都可以拥有不同的网络超时时间等配置。

例如，将 nginx.conf 文件中的 upstream\_conn\_timeout 配置项解析到 ngx\_http\_hello\_conf\_t 结构体中的成员upstream.conn\_timeout 中。可定义如下的连接超时时间，并把ngx\_http\_hello\_conf\_t 配置项的 upstream 成员赋给 ngx\_http\_upstream\_t 中的conf 即可；

```text
typedef struct  
{  
        ... 
        ngx_http_upstream_conf_t    upstream;
}ngx_http_hello_conf_t;  
  
  
static ngx_command_t ngx_http_hello_commands[] = {  
   {  
                ngx_string("upstream_conn_timeout"),  
                NGX_HTTP_LOC_CONF|NGX_CONF_TAKE1,  
                ngx_conf_set_msec_slot,  
                NGX_HTTP_LOC_CONF_OFFSET,  
                offsetof(ngx_http_hello_conf_t, upstream.conn_timeout),  
                NULL },  
  
  
        ngx_null_command  
};  

  
static ngx_int_t  
ngx_http_hello_handler(ngx_http_request_t *r)  
{  
     ...
     ngx_http_hello_conf_t *mycf = (ngx_http_hello_conf_t *)ngx_http_get_module_loc_conf(r,ngx_http_hello_module);
     r->upstream->conf = &mycf->upstream;
     ...
}

```

## 设置第三方服务器地址

在 ngx\_http\_upstream\_t 结构体中的resolved 成员可直接设置上游服务器的地址，也可以由nginx.conf 文件中配置upstream 模块，并指定上游服务器的地址。resolved 类型定义如下：

```text
typedef struct {
    
    ngx_str_t                        host;
    
    in_port_t                        port;
    ngx_uint_t                       no_port; 

    
    ngx_uint_t                       naddrs;
    
    ngx_addr_t                      *addrs;

    
    struct sockaddr                 *sockaddr;
    
    socklen_t                        socklen;

    ngx_resolver_ctx_t              *ctx;
} ngx_http_upstream_resolved_t;

```

## 设置回调方法

在结构体 ngx\_http\_upstream\_t 中定义了 8 个回调方法：

```text
    
    ngx_int_t                      (*input_filter_init)(void *data);
    
    ngx_int_t                      (*input_filter)(void *data, ssize_t bytes);
    
    void                            *input_filter_ctx;

    
    ngx_int_t                      (*create_request)(ngx_http_request_t *r);
    
    ngx_int_t                      (*reinit_request)(ngx_http_request_t *r);
    
    ngx_int_t                      (*process_header)(ngx_http_request_t *r);
    
    void                           (*abort_request)(ngx_http_request_t *r);
    
    void                           (*finalize_request)(ngx_http_request_t *r,
                                         ngx_int_t rc);
    
    ngx_int_t                      (*rewrite_redirect)(ngx_http_request_t *r,
                                         ngx_table_elt_t *h, size_t prefix);

```

在这些回调方法中，其中有 3 个非常重要，在模块中是必须要实现的，这 3 个回调函数为：

```text

    ngx_int_t                      (*create_request)(ngx_http_request_t *r);

    ngx_int_t                      (*process_header)(ngx_http_request_t *r);

    void                           (*finalize_request)(ngx_http_request_t *r,
                                         ngx_int_t rc);

```

create\_request 在初始化 upstream 时被调用，生成发送到后端服务器的请求缓冲（缓冲链）。reinit\_request 在某台后端服务器出错的情况，Nginx 会尝试连接到另一台后端服务器。Nginx 选定新的服务器以后，会先调用此函数，以重新初始化upstream 模块的工作状态，然后再次进行 upstream 连接。process\_header 是用于解析上游服务器返回的基于TCP 的响应头部。finalize\_request 在正常完成与后端服务器的请求后 或 失败 导致销毁请求时，该方法被调用。input\_filter\_init 和input\_filter 都用于处理上游的响应包体，因为在处理包体前HTTP 模块可能需要做一些初始化工作。初始化工作由input\_filter\_init 完成，实际处理包体由 input\_filter 方法完成。

## 启动 upstream 机制

调用 ngx\_http\_upstream\_init 方法便可启动upstream 机制，此时，必须通过返回NGX\_DONE 通知HTTP 框架暂停执行请求的下一个阶段，并且需要执行r-&gt;main-&gt;count++ 告知HTTP 框架将当前请求的引用计数增加 1，即告知ngx\_http\_hello\_handler 方法暂时不要销毁请求，因为HTTP 框架只有在引用计数为 0 时才真正销毁请求。例如：

```text
static ngx_int_t ngx_http_hello_handler(ngx_http_request_t *r)
{
    ...
    r->main->count++;
    ngx_http_upstream_init(r);
    return NGX_DONE;
}

```

## subrequest 使用方式

subrequest 只是分解复杂请求的一种设计模式，它可以把原始请求分解为多个子请求，使得诸多请求协同完成一个用户请求，并且每个请求只关注一个功能。首先，若不是完全将上游服务器的响应包体转发到下游客户端，基本都会使用subrequest 创建子请求，并由子请求使用upstream 机制访问上游服务器，然后由父请求根据上游响应重新构造返回给下游客户端的响应。

subrequest 的使用步骤如下：

1. 在 nginx.conf 配置文件中配置好子请求的处理方式；
2. 启动 subrequest 子请求；
3. 实现子请求执行结束时的回调函数；
4. 实现父请求被激活时的回调函数；

## 配置子请求的处理方式

子请求并不是由 HTTP 框架解析所接收到客户端网络包而得到的，而是由父请求派生的。它的配置和普通请求的配置相同，都是在nginx.conf 文件中配置相应的处理模块。例如：可以在配置文件nginx.conf 中配置以下的子请求访问 [https://github.com](https://github.com/)

```text
location /subrq { 
  	    rewrite ^/subrq(.*)$ $1 break;
  	    proxy_pass https://github.com;
	}

```

## 启动 subrequest 子请求

subrequest 是在父请求的基础上派生的子请求，subrequest 返回的内容会被附加到父请求上面，他的实现方法是调用ngx\_http\_subrequest 函数，该函数定义在文件：[src/http/ngx\_http\_core\_module.h](http://lxr.nginx.org/source/src/http/ngx_http_core_module.h)

```text
ngx_int_t ngx_http_subrequest(ngx_http_request_t *r,
     ngx_str_t *uri, ngx_str_t *args, ngx_http_request_t **psr,
     ngx_http_post_subrequest_t *ps, ngx_uint_t flags);

```

该函数的参数如下：引用自文件《[Emiller's Advanced Topics In Nginx Module Development](http://www.evanmiller.org/nginx-modules-guide-advanced.html#subrequests)》

* \*r is the original request（当前的请求，即父请求）；
* _uri and_argsrefer to the sub-request（\*uri 是子请求的URI，\*args是子请求URI 的参数）；
* \*\*psr is a reference to a NULL pointer that will point to the new \(sub-\)request structure（\*\*psr 是指向返回子请求，相当于值-结果传递，作为参数传递进去是指向 NULL 指针，输出结果是指向新创建的子请求）；
* \*ps is a callback for when the subrequest is finished. （\*ps 是指出子请求结束时必须回调的处理方法）；
* flags can be a bitwise-OR'ed combination of:
* NGX\_HTTP\_ZERO\_IN\_URI: the URI contains a character with ASCII code 0 \(also known as '\0'\), or contains "%00"
* NGX\_HTTP\_SUBREQUEST\_IN\_MEMORY: store the result of the subrequest in a contiguous chunk of memory \(usually not necessary\) （将子请求的subrequest\_in\_memory 标志位为 1，表示发起的子请求，访问的网络资源返回的响应将全部在内存中处理）；
* NGX\_HTTP\_SUBREQUEST\_WAITED: store the result of the subrequest in a contiguous chunk of memory \(usually not necessary\) （将子请求的waited 标志位为 1，表示子请求完成后会设置自身的r-&gt;done 标志位，可以通过判断该标志位得知子请求是否完成）；

该函数 ngx\_http\_subrequest 的返回值如下：

* NGX\_OK:the subrequest finished without touching the network（成功建立子请求）；
* NGX\_DONE:the client reset the network connection（客户端重置网络连接）；
* NGX\_ERROR:there was a server error of some sort（建立子请求失败）；
* NGX\_AGAIN:the subrequest requires network activity（子请求需要激活网络）；

该子请求返回的结果附加在你期望的位置。若要修改子请求的结果，可以使用 another filter（或同一个）。并告知该 filter 对父请求或子请求进行操作：具体实例可参照模块["addition" module](http://lxr.nginx.org/source/src/http/modules/ngx_http_addition_filter_module.c)

```text
if (r == r->main) { 
    
} else {
    
}

```

以下是子请求函数 ngx\_http\_subrequest 的源码剖析，其源码定义在文件：[src/http/ngx\_http\_core\_module.c](http://lxr.nginx.org/source/src/http/ngx_http_core_module.c)

```text

ngx_int_t
ngx_http_subrequest(ngx_http_request_t *r,
    ngx_str_t *uri, ngx_str_t *args, ngx_http_request_t **psr,
    ngx_http_post_subrequest_t *ps, ngx_uint_t flags)
{
    ngx_time_t                    *tp;
    ngx_connection_t              *c;
    ngx_http_request_t            *sr;
    ngx_http_core_srv_conf_t      *cscf;
    ngx_http_postponed_request_t  *pr, *p;

    
    r->main->subrequests--;

    
    if (r->main->subrequests == 0) {
        ngx_log_error(NGX_LOG_ERR, r->connection->log, 0,
                      "subrequests cycle while processing \"%V\"", uri);
        r->main->subrequests = 1;
        return NGX_ERROR;
    }

    
    sr = ngx_pcalloc(r->pool, sizeof(ngx_http_request_t));
    if (sr == NULL) {
        return NGX_ERROR;
    }

    
    sr->signature = NGX_HTTP_MODULE;

    
    c = r->connection;
    sr->connection = c;

    
    sr->ctx = ngx_pcalloc(r->pool, sizeof(void *) * ngx_http_max_module);
    if (sr->ctx == NULL) {
        return NGX_ERROR;
    }

    
    if (ngx_list_init(&sr->headers_out.headers, r->pool, 20,
                      sizeof(ngx_table_elt_t))
        != NGX_OK)
    {
        return NGX_ERROR;
    }

    
    cscf = ngx_http_get_module_srv_conf(r, ngx_http_core_module);
    sr->main_conf = cscf->ctx->main_conf;
    sr->srv_conf = cscf->ctx->srv_conf;
    sr->loc_conf = cscf->ctx->loc_conf;

    
    sr->pool = r->pool;

    
    sr->headers_in = r->headers_in;

    ngx_http_clear_content_length(sr);
    ngx_http_clear_accept_ranges(sr);
    ngx_http_clear_last_modified(sr);

    
    sr->request_body = r->request_body;

#if (NGX_HTTP_SPDY)
    sr->spdy_stream = r->spdy_stream;
#endif

    
    sr->method = NGX_HTTP_GET;
    
    sr->http_version = r->http_version;

    
    sr->request_line = r->request_line;
    
    sr->uri = *uri;

    
    if (args) {
        sr->args = *args;
    }

    ngx_log_debug2(NGX_LOG_DEBUG_HTTP, c->log, 0,
                   "http subrequest \"%V?%V\"", uri, &sr->args);

    
    sr->subrequest_in_memory = (flags & NGX_HTTP_SUBREQUEST_IN_MEMORY) != 0;
    sr->waited = (flags & NGX_HTTP_SUBREQUEST_WAITED) != 0;

    sr->unparsed_uri = r->unparsed_uri;
    sr->method_name = ngx_http_core_get_method;
    sr->http_protocol = r->http_protocol;

    ngx_http_set_exten(sr);

    
    sr->main = r->main;
    sr->parent = r;
    sr->post_subrequest = ps;
    
    sr->read_event_handler = ngx_http_request_empty_handler;
    sr->write_event_handler = ngx_http_handler;

    
    if (c->data == r && r->postponed == NULL) {
        c->data = sr;
    }

    
    sr->variables = r->variables;

    
    sr->log_handler = r->log_handler;

    pr = ngx_palloc(r->pool, sizeof(ngx_http_postponed_request_t));
    if (pr == NULL) {
        return NGX_ERROR;
    }

    pr->request = sr;
    pr->out = NULL;
    pr->next = NULL;

    
    if (r->postponed) {
        for (p = r->postponed; p->next; p = p->next) {  }
        p->next = pr;

    } else {
        r->postponed = pr;
    }

    
    sr->internal = 1;

    
    sr->discard_body = r->discard_body;
    sr->expect_tested = 1;
    sr->main_filter_need_in_memory = r->main_filter_need_in_memory;

    sr->uri_changes = NGX_HTTP_MAX_URI_CHANGES + 1;

    tp = ngx_timeofday();
    sr->start_sec = tp->sec;
    sr->start_msec = tp->msec;

    
    r->main->count++;

    *psr = sr;

    
    return ngx_http_post_request(sr, NULL);
}

ngx_int_t
ngx_http_post_request(ngx_http_request_t *r, ngx_http_posted_request_t *pr)
{
    ngx_http_posted_request_t  **p;

    if (pr == NULL) {
        pr = ngx_palloc(r->pool, sizeof(ngx_http_posted_request_t));
        if (pr == NULL) {
            return NGX_ERROR;
        }
    }

    pr->request = r;
    pr->next = NULL;

    for (p = &r->main->posted_requests; *p; p = &(*p)->next) {  }

    *p = pr;

    return NGX_OK;
}

```

## 子请求结束时的回调函数

在子请求结束时（正常或异常结束）Nginx 会调用ngx\_http\_post\_subrequest\_pt 回调处理方法。下面是回调方法的定义：

```text
typedef struct {
    ngx_http_post_subrequest_pt       handler;
    void                             *data;
} ngx_http_post_subrequest_t;

typedef ngx_int_t (*ngx_http_post_subrequest_pt)(ngx_http_request_t *r,
    void *data, ngx_int_t rc);

```

在结构体 ngx\_http\_post\_subrequest\_t 中，生成该结构体的变量时，可把用户的任意数据赋给指针data ，ngx\_http\_post\_subrequest\_pt 回调方法的参数data 就是用户把数据赋给结构体 ngx\_http\_post\_subrequest\_t 中的成员指针data 所指的数据。ngx\_http\_post\_subrequest\_pt 回调方法中的参数rc 是子请求结束时的状态，它的取值由函数ngx\_http\_finalize\_request 销毁请求时传递给参数rc。 函数ngx\_http\_finalize\_request 的部分源码，具体可查阅文件：[src/http/ngx\_http\_request.c](http://lxr.nginx.org/source/src/http/ngx_http_request.c)

```text
void
ngx_http_finalize_request(ngx_http_request_t *r, ngx_int_t rc) 
{
  ...
    
    if (r != r->main && r->post_subrequest) {
        rc = r->post_subrequest->handler(r, r->post_subrequest->data, rc);
    }

  ...
    
    
    if (r != r->main) {  
        
        if (r->buffered || r->postponed) {
            
            if (ngx_http_set_write_handler(r) != NGX_OK) {
                ngx_http_terminate_request(r, 0);
            }

            return;
        }
        ...
        pr = r->parent;
        

        
        if (r == c->data) { 

            r->main->count--;

            if (!r->logged) {

                clcf = ngx_http_get_module_loc_conf(r, ngx_http_core_module);

                if (clcf->log_subrequest) {
                    ngx_http_log_request(r);
                }

                r->logged = 1;

            } else {
                ngx_log_error(NGX_LOG_ALERT, c->log, 0,
                              "subrequest: \"%V?%V\" logged again",
                              &r->uri, &r->args);
            }

            r->done = 1;
            
            if (pr->postponed && pr->postponed->request == r) {
                pr->postponed = pr->postponed->next;
            }
            
            c->data = pr;   

        } else {
            
            r->write_event_handler = ngx_http_request_finalizer;

            if (r->waited) {
                r->done = 1;
            }
        }
        
        if (ngx_http_post_request(pr, NULL) != NGX_OK) {
            r->main->count++;
            ngx_http_terminate_request(r, 0);
            return;
        }

        return;
    }
    
    if (r->buffered || c->buffered || r->postponed || r->blocked) {

        if (ngx_http_set_write_handler(r) != NGX_OK) {
            ngx_http_terminate_request(r, 0);
        }

        return;
    }

 ...
} 

```

## 父请求被激活后的回调方法

父请求被激活后的回调方法由指针 ngx\_http\_event\_pt 实现。该方法负责把响应包发送给用户。如下所示：

```text
typedef void(*ngx_http_event_handler_pt)(ngx_http_request_t *r);

struct ngx_http_request_s{
      ...
      ngx_http_event_handler_pt      write_event_handler;
      ...
};

```

一个请求中，只能调用一次 subrequest，即不能一次创建多个子请求，但是可以在新创建的子请求中再创建新的子请求。

参考资料：

《深入理解Nginx 》

《[Emiller's Advanced Topics In Nginx Module Development](http://www.evanmiller.org/nginx-modules-guide-advanced.html#subrequests-single)》

《[nginx subrequest的实现解析](http://blog.csdn.net/fengmo_q/article/details/6685840)》

《[ngx\_http\_request\_t结构体](http://blog.csdn.net/xiajun07061225/article/details/9189505)》

