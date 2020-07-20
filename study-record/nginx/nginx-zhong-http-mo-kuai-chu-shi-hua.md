# Nginx 中 HTTP 模块初始化

## 概述

在前面的文章《 [Nginx 配置解析](http://blog.csdn.net/chenhanzhun/article/details/42615433)》简单讲解了通用模块的配置项解析，并且大概讲解了HTTP 模块的配置项解析过程，本文更具体的分析HTTP 模块的初始化过程。HTTP 模块初始化过程主要有：上下文结构初始化、配置项解析、配置项合并、server 相关端口设置。

## HTTP 模块接口

## ngx\_http\_module\_t 结构体

在 Nginx 中，结构体 ngx\_module\_t 是 Nginx 模块最基本的接口。对于每一种不同类型的模块，都有一个具体的结构体来描述这一类模块的通用接口。在Nginx 中定义了HTTP 模块的通用接口ngx\_http\_module\_t 结构体，该结构体定义在文件[src/http/ngx\_http\_config.h](http://lxr.nginx.org/source/src/http/ngx_http_config.h)：我们把直属于http{}、server{}、location{} 块的配置项分别称为main、srv、loc 级别配置项。

```text

typedef struct {
    
    ngx_int_t   (*preconfiguration)(ngx_conf_t *cf);
    
    ngx_int_t   (*postconfiguration)(ngx_conf_t *cf);

    
    void       *(*create_main_conf)(ngx_conf_t *cf);
    
    char       *(*init_main_conf)(ngx_conf_t *cf, void *conf);

    
    void       *(*create_srv_conf)(ngx_conf_t *cf);
    
    char       *(*merge_srv_conf)(ngx_conf_t *cf, void *prev, void *conf);

    
    void       *(*create_loc_conf)(ngx_conf_t *cf);
    
    char       *(*merge_loc_conf)(ngx_conf_t *cf, void *prev, void *conf);
} ngx_http_module_t;

```

## ngx\_http\_conf\_ctx\_t 结构体

在 HTTP 模块中，管理 HTTP 模块配置项的结构由 ngx\_http\_conf\_ctx\_t 实现，该结构有三个成员，分别指向三个指针数组，指针数组是由相应地 HTTP 模块create\_main\_conf、create\_srv\_conf、create\_loc\_conf 方法创建的结构体指针组成的数组。ngx\_http\_conf\_ctx\_t 结构体定义在文件[src/http/ngx\_http\_config.h](http://lxr.nginx.org/source/src/http/ngx_http_config.h) 中：

```text

typedef struct {
    
    void        **main_conf;
    
    void        **srv_conf;
    
    void        **loc_conf;
} ngx_http_conf_ctx_t;

```

## ngx\_http\_module 核心模块

## ngx\_http\_module 核心模块定义

ngx\_http\_module 是 HTTP 模块的核心模块，该模块的功能是：定义新的 HTTP 模块类型，并为每个HTTP 模块定义通用接口ngx\_http\_module\_t 结构体，管理HTTP 模块生成的配置项结构体，并解析HTTP 类配置项。该模块定义在文件[src/http/ngx\_http.c](http://lxr.nginx.org/source/src/http/ngx_http.c) 中：

```text

ngx_module_t  ngx_http_module = {
    NGX_MODULE_V1,
    &ngx_http_module_ctx,                  
    ngx_http_commands,                     
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

ngx\_http\_module 作为核心模块，必须定义核心模块的通用接口上下文结构，其通用接口上下文结构定义在文件[src/http/ngx\_http.c](http://lxr.nginx.org/source/src/http/ngx_http.c) 中：在 ngx\_http\_module 核心模块中只定义了 http 模块的名称。

```text

static ngx_core_module_t  ngx_http_module_ctx = {
    ngx_string("http"),
    NULL,
    NULL
};

```

在 ngx\_http\_module 模块中定义了http{} 块感兴趣的配置项数组，配置项数组定义在文件[src/http/ngx\_http.c](http://lxr.nginx.org/source/src/http/ngx_http.c) 中：

```text

static ngx_command_t  ngx_http_commands[] = {

    { ngx_string("http"),
      NGX_MAIN_CONF|NGX_CONF_BLOCK|NGX_CONF_NOARGS,
      ngx_http_block,
      0,
      0,
      NULL },

      ngx_null_command
};

```

从 ngx\_http\_module 模块的定义中可以知道，该模块只有一个初始化处理方法ngx\_http\_block，该处理方法是HTTP 模块的初始化作用。

## ngx\_http\_module 核心模块初始化

在上面 ngx\_http\_module 模块的定义中已经知道，HTTP 模块的初始化过程由函数ngx\_http\_block 实现，首先先给出该函数的整体分析，接着对该函数进行具体的分析。该函数定义在文件[src/http/ngx\_http.c](http://lxr.nginx.org/source/src/http/ngx_http.c) 中：

```text

static char *
ngx_http_block(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)
{
    char                        *rv;
    ngx_uint_t                   mi, m, s;
    ngx_conf_t                   pcf;
    ngx_http_module_t           *module;
    ngx_http_conf_ctx_t         *ctx;
    ngx_http_core_loc_conf_t    *clcf;
    ngx_http_core_srv_conf_t   **cscfp;
    ngx_http_core_main_conf_t   *cmcf;

    

    
    ctx = ngx_pcalloc(cf->pool, sizeof(ngx_http_conf_ctx_t));
    if (ctx == NULL) {
        return NGX_CONF_ERROR;
    }

    
    *(ngx_http_conf_ctx_t **) conf = ctx;

    

    
    ngx_http_max_module = 0;
    for (m = 0; ngx_modules[m]; m++) {
        if (ngx_modules[m]->type != NGX_HTTP_MODULE) {
            continue;
        }

        ngx_modules[m]->ctx_index = ngx_http_max_module++;
    }

    
    

    ctx->main_conf = ngx_pcalloc(cf->pool,
                                 sizeof(void *) * ngx_http_max_module);
    if (ctx->main_conf == NULL) {
        return NGX_CONF_ERROR;
    }

    
    

    ctx->srv_conf = ngx_pcalloc(cf->pool, sizeof(void *) * ngx_http_max_module);
    if (ctx->srv_conf == NULL) {
        return NGX_CONF_ERROR;
    }

    
    

    ctx->loc_conf = ngx_pcalloc(cf->pool, sizeof(void *) * ngx_http_max_module);
    if (ctx->loc_conf == NULL) {
        return NGX_CONF_ERROR;
    }

    

    
    for (m = 0; ngx_modules[m]; m++) {
        if (ngx_modules[m]->type != NGX_HTTP_MODULE) {
            continue;
        }

        module = ngx_modules[m]->ctx;
        mi = ngx_modules[m]->ctx_index;

        
        if (module->create_main_conf) {
            ctx->main_conf[mi] = module->create_main_conf(cf);
            if (ctx->main_conf[mi] == NULL) {
                return NGX_CONF_ERROR;
            }
        }

        
        if (module->create_srv_conf) {
            ctx->srv_conf[mi] = module->create_srv_conf(cf);
            if (ctx->srv_conf[mi] == NULL) {
                return NGX_CONF_ERROR;
            }
        }

        
        if (module->create_loc_conf) {
            ctx->loc_conf[mi] = module->create_loc_conf(cf);
            if (ctx->loc_conf[mi] == NULL) {
                return NGX_CONF_ERROR;
            }
        }
    }

    
    pcf = *cf;
    
    cf->ctx = ctx;

    
    for (m = 0; ngx_modules[m]; m++) {
        if (ngx_modules[m]->type != NGX_HTTP_MODULE) {
            continue;
        }

        module = ngx_modules[m]->ctx;

        if (module->preconfiguration) {
            if (module->preconfiguration(cf) != NGX_OK) {
                return NGX_CONF_ERROR;
            }
        }
    }

    
    

    cf->module_type = NGX_HTTP_MODULE;
    cf->cmd_type = NGX_HTTP_MAIN_CONF;
    
    rv = ngx_conf_parse(cf, NULL);

    if (rv != NGX_CONF_OK) {
        goto failed;
    }

    
    

    
    cmcf = ctx->main_conf[ngx_http_core_module.ctx_index];
    
    cscfp = cmcf->servers.elts;

    
    for (m = 0; ngx_modules[m]; m++) {
        if (ngx_modules[m]->type != NGX_HTTP_MODULE) {
            continue;
        }

        
       module = ngx_modules[m]->ctx;
       
       mi = ngx_modules[m]->ctx_index;

        
        

        if (module->init_main_conf) {
            rv = module->init_main_conf(cf, ctx->main_conf[mi]);
            if (rv != NGX_CONF_OK) {
                goto failed;
            }
        }

        
        rv = ngx_http_merge_servers(cf, cmcf, module, mi);
        if (rv != NGX_CONF_OK) {
            goto failed;
        }
    }

    

    
    

    
    for (s = 0; s < cmcf->servers.nelts; s++) {

        
        clcf = cscfp[s]->ctx->loc_conf[ngx_http_core_module.ctx_index];

        
        if (ngx_http_init_locations(cf, cscfp[s], clcf) != NGX_OK) {
            return NGX_CONF_ERROR;
        }

        
        if (ngx_http_init_static_location_trees(cf, clcf) != NGX_OK) {
            return NGX_CONF_ERROR;
        }
    }

    
    if (ngx_http_init_phases(cf, cmcf) != NGX_OK) {
        return NGX_CONF_ERROR;
    }

    
    if (ngx_http_init_headers_in_hash(cf, cmcf) != NGX_OK) {
        return NGX_CONF_ERROR;
    }

    
    for (m = 0; ngx_modules[m]; m++) {
        if (ngx_modules[m]->type != NGX_HTTP_MODULE) {
            continue;
        }

        module = ngx_modules[m]->ctx;

        if (module->postconfiguration) {
            if (module->postconfiguration(cf) != NGX_OK) {
                return NGX_CONF_ERROR;
            }
        }
    }

    if (ngx_http_variables_init_vars(cf) != NGX_OK) {
        return NGX_CONF_ERROR;
    }

    

    *cf = pcf;

    
    if (ngx_http_init_phase_handlers(cf, cmcf) != NGX_OK) {
        return NGX_CONF_ERROR;
    }

    

    
    if (ngx_http_optimize_servers(cf, cmcf, cmcf->ports) != NGX_OK) {
        return NGX_CONF_ERROR;
    }

    return NGX_CONF_OK;

failed:

    *cf = pcf;

    return rv;
}

```

从上面的分析中可以总结出 HTTP 模块初始化的流程如下：

* Nginx 进程进入主循环，在主循环中调用配置解析器解析配置文件_nginx.conf_;
* 在配置文件中遇到 http{} 块配置，则 HTTP 框架开始初始化并启动，其由函数 ngx\_http\_block\(\) 实现；
* HTTP 框架初始化所有 HTTP 模块的序列号，并创建 3 个类型为 _ngx\_http\_conf\_ctx\_t 结构的数组用于存储所有HTTP 模块的_create\_main\_conf、_create\_srv\_conf_、_create\_loc\_conf_方法返回的指针地址；
* 调用每个 HTTP 模块的 preconfiguration 方法；
* HTTP 框架调用函数 ngx\_conf\_parse\(\) 开始循环解析配置文件 \*nginx.conf \*中的http{}块里面的所有配置项，http{} 块内可嵌套多个server{} 块，而 server{} 块可嵌套多个 location{}，location{} 依旧可以嵌套location{}，因此配置项解析函数是递归调用；
* HTTP 框架处理完毕 http{} 配置项，根据解析配置项的结果，必要时调用ngx\_http\_merge\_servers 方法进行配置项合并处理，即合并main、srv、loc 级别下server、location 相关的配置项；
* 初始化可添加处理方法的 HTTP 阶段的动态数组；
* 调用所有 HTTP 模块的 postconfiguration 方法使 HTTP 模块可以处理HTTP 阶段，即将HTTP 模块的ngx\_http\_handler\_pt 处理方法添加到HTTP 阶段中；
* 根据 HTTP 模块处理 HTTP 阶段的方法构造 phase\_engine\_handlers 数组；
* 构造 server 相关的监听端口，并设置新连接事件的回调方法为ngx\_http\_init\_connection ；
* 继续处理其他 http{} 块之外的配置项，直到配置文件解析器处理完所有配置项后通知Nginx 主循环配置项解析完毕。此时，Nginx 才会启动Web 服务器；

## ngx\_http\_core\_module 模块

## ngx\_http\_core\_main\_conf\_t 结构体

在初始化 HTTP 模块的过程中，会调用配置项解析函数ngx\_conf\_parse 解析http{} 块内的配置项，当遇到server{} 块、location{} 块配置项时，此时会调用配置项解析函数解析server{} 和location{} 块，在解析这两个配置块的过程中依旧会创建属于该块的配置项结构srv\_conf、loc\_conf，因此就会导致与http{} 块所创建的配置项结构体重复，这时候就需要对这些配置项进行管理与合并。首先先看下结构体ngx\_http\_core\_main\_conf\_t，ngx\_http\_core\_main\_conf\_t 是ngx\_http\_core\_module 的 main\_conf，存储了 http{} 层的配置参数。该结构体定义在文件[src/http/ngx\_http\_core\_module.h](http://lxr.nginx.org/source/src/http/ngx_http_core_module.h)中：

```text

typedef struct {
    
    ngx_array_t                servers;         

    
    ngx_http_phase_engine_t    phase_engine;

    ngx_hash_t                 headers_in_hash;

    ngx_hash_t                 variables_hash;

    ngx_array_t                variables;       
    ngx_uint_t                 ncaptures;

    
    ngx_uint_t                 server_names_hash_max_size;
    
    ngx_uint_t                 server_names_hash_bucket_size;

    ngx_uint_t                 variables_hash_max_size;
    ngx_uint_t                 variables_hash_bucket_size;

    ngx_hash_keys_arrays_t    *variables_keys;

    
    ngx_array_t               *ports;

    ngx_uint_t                 try_files;       

    
    ngx_http_phase_t           phases[NGX_HTTP_LOG_PHASE + 1];
} ngx_http_core_main_conf_t;

```

## ngx\_http\_core\_srv\_conf\_t 结构体

结构体 ngx\_http\_core\_srv\_conf\_t，ngx\_http\_core\_srv\_conf\_t 是ngx\_http\_core\_module 的srv\_conf，存储了server{} 层的配置参数。该结构体定义在文件[src/http/ngx\_http\_core\_module.h](http://lxr.nginx.org/source/src/http/ngx_http_core_module.h)中：

```text

typedef struct {
    
    ngx_array_t                 server_names;

    
    
    ngx_http_conf_ctx_t        *ctx;

    
    ngx_str_t                   server_name;

    size_t                      connection_pool_size;
    size_t                      request_pool_size;
    size_t                      client_header_buffer_size;

    ngx_bufs_t                  large_client_header_buffers;

    ngx_msec_t                  client_header_timeout;

    ngx_flag_t                  ignore_invalid_headers;
    ngx_flag_t                  merge_slashes;
    ngx_flag_t                  underscores_in_headers;

    unsigned                    listen:1;
#if (NGX_PCRE)
    unsigned                    captures:1;
#endif

    ngx_http_core_loc_conf_t  **named_locations;
} ngx_http_core_srv_conf_t;

```

## ngx\_http\_core\_loc\_conf\_t 结构体

结构体 ngx\_http\_core\_loc\_conf\_t，ngx\_http\_core\_loc\_conf\_t 是ngx\_http\_core\_module 的loc\_conf，存储了location{} 层的配置参数。该结构体定义在文件[src/http/ngx\_http\_core\_module.h](http://lxr.nginx.org/source/src/http/ngx_http_core_module.h)中：

```text
struct ngx_http_core_loc_conf_s {
    
    ngx_str_t     name;          

#if (NGX_PCRE)
    ngx_http_regex_t  *regex;
#endif

    unsigned      noname:1;   
    unsigned      lmt_excpt:1;
    unsigned      named:1;

    unsigned      exact_match:1;
    unsigned      noregex:1;

    unsigned      auto_redirect:1;
#if (NGX_HTTP_GZIP)
    unsigned      gzip_disable_msie6:2;
#if (NGX_HTTP_DEGRADATION)
    unsigned      gzip_disable_degradation:2;
#endif
#endif

    ngx_http_location_tree_node_t   *static_locations;
#if (NGX_PCRE)
    ngx_http_core_loc_conf_t       **regex_locations;
#endif

    
    
    void        **loc_conf;

    uint32_t      limit_except;
    void        **limit_except_loc_conf;

    ngx_http_handler_pt  handler;

    
    size_t        alias;
    ngx_str_t     root;                    
    ngx_str_t     post_action;

    ngx_array_t  *root_lengths;
    ngx_array_t  *root_values;

    ngx_array_t  *types;
    ngx_hash_t    types_hash;
    ngx_str_t     default_type;

    off_t         client_max_body_size;    
    off_t         directio;                
    off_t         directio_alignment;      

    size_t        client_body_buffer_size; 
    size_t        send_lowat;              
    size_t        postpone_output;         
    size_t        limit_rate;              
    size_t        limit_rate_after;        
    size_t        sendfile_max_chunk;      
    size_t        read_ahead;              

    ngx_msec_t    client_body_timeout;     
    ngx_msec_t    send_timeout;            
    ngx_msec_t    keepalive_timeout;       
    ngx_msec_t    lingering_time;          
    ngx_msec_t    lingering_timeout;       
    ngx_msec_t    resolver_timeout;        

    ngx_resolver_t  *resolver;             

    time_t        keepalive_header;        

    ngx_uint_t    keepalive_requests;      
    ngx_uint_t    keepalive_disable;       
    ngx_uint_t    satisfy;                 
    ngx_uint_t    lingering_close;         
    ngx_uint_t    if_modified_since;       
    ngx_uint_t    max_ranges;              
    ngx_uint_t    client_body_in_file_only; 

    ngx_flag_t    client_body_in_single_buffer;
                                           
    ngx_flag_t    internal;                
    ngx_flag_t    sendfile;                
#if (NGX_HAVE_FILE_AIO)
    ngx_flag_t    aio;                     
#endif
    ngx_flag_t    tcp_nopush;              
    ngx_flag_t    tcp_nodelay;             
    ngx_flag_t    reset_timedout_connection; 
    ngx_flag_t    server_name_in_redirect; 
    ngx_flag_t    port_in_redirect;        
    ngx_flag_t    msie_padding;            
    ngx_flag_t    msie_refresh;            
    ngx_flag_t    log_not_found;           
    ngx_flag_t    log_subrequest;          
    ngx_flag_t    recursive_error_pages;   
    ngx_flag_t    server_tokens;           
    ngx_flag_t    chunked_transfer_encoding; 
    ngx_flag_t    etag;                    

#if (NGX_HTTP_GZIP)
    ngx_flag_t    gzip_vary;               

    ngx_uint_t    gzip_http_version;       
    ngx_uint_t    gzip_proxied;            

#if (NGX_PCRE)
    ngx_array_t  *gzip_disable;            
#endif
#endif

#if (NGX_HAVE_OPENAT)
    ngx_uint_t    disable_symlinks;        
    ngx_http_complex_value_t  *disable_symlinks_from;
#endif

    ngx_array_t  *error_pages;             
    ngx_http_try_file_t    *try_files;     

    ngx_path_t   *client_body_temp_path;   

    ngx_open_file_cache_t  *open_file_cache;
    time_t        open_file_cache_valid;
    ngx_uint_t    open_file_cache_min_uses;
    ngx_flag_t    open_file_cache_errors;
    ngx_flag_t    open_file_cache_events;

    ngx_log_t    *error_log;

    ngx_uint_t    types_hash_max_size;
    ngx_uint_t    types_hash_bucket_size;

    
    ngx_queue_t  *locations;

#if 0
    ngx_http_core_loc_conf_t  *prev_location;
#endif
};

typedef struct{

      
      ngx_queue_t queue;

      
       ngx_http_core_loc_conf_t *exact;

       
       ngx_http_core_loc_conf_t *inclusive;

       
       ngx_str_t *name;
       ...

}ngx_http_location_queue_t; 

```

## ngx\_http\_core\_module 模块定义

ngx\_http\_core\_module 模块是HTTP 模块中的第一个模块，该模块管理srv、loc 级别的配置项结构。该模块在文件[src/http/ngx\_http\_core\_module.c](http://lxr.nginx.org/source/src/http/ngx_http_core_module.c)中定义：

```text
ngx_module_t  ngx_http_core_module = {
    NGX_MODULE_V1,
    &ngx_http_core_module_ctx,             
    ngx_http_core_commands,                
    NGX_HTTP_MODULE,                       
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

在模块的定义中，其中定义了 HTTP 模块的上下文结构ngx\_http\_core\_module\_ctx，该上下文结构体定义如下：

```text
static ngx_http_module_t  ngx_http_core_module_ctx = {
    ngx_http_core_preconfiguration,        
    NULL,                                  

    ngx_http_core_create_main_conf,        
    ngx_http_core_init_main_conf,          

    ngx_http_core_create_srv_conf,         
    ngx_http_core_merge_srv_conf,          

    ngx_http_core_create_loc_conf,         
    ngx_http_core_merge_loc_conf           
};

```

由于该模块中感兴趣的配置项太多，这里只列出 server、location 配置项。定义如下：

```text
    ...

    { ngx_string("server"),
      NGX_HTTP_MAIN_CONF|NGX_CONF_BLOCK|NGX_CONF_NOARGS,
      ngx_http_core_server,
      0,
      0,
      NULL },

    ...

    { ngx_string("location"),
      NGX_HTTP_SRV_CONF|NGX_HTTP_LOC_CONF|NGX_CONF_BLOCK|NGX_CONF_TAKE12,
      ngx_http_core_location,
      NGX_HTTP_SRV_CONF_OFFSET,
      0,
      NULL },

      ...
      ngx_null_command
};

```

## 管理 HTTP 模块不同级别的配置项结构体

在 HTTP 模块的 http{} 配置项解析过程中，可能遇到多个嵌套 server{} 块以及location{}，不同块之间的解析都会创建相应的结构体保存配置项参数，但是由于属于嵌套关系，所有必须管理好不同块之间的配置项结构体，方便解析完毕后合并相应的配置项，以下针对不同级别的配置项结构体进行分析。

## 获取不同级别配置项结构

根据不同结构体变量的参数获取不同级别的配置项结构体由宏定义实现，在文件 [src/http/ngx\_http\_config.h](http://lxr.nginx.org/source/src/http/ngx_http_config.h)中定义如下：

```text

#define ngx_http_get_module_main_conf(r, module)                             \
    (r)->main_conf[module.ctx_index]
#define ngx_http_get_module_srv_conf(r, module)  (r)->srv_conf[module.ctx_index]
#define ngx_http_get_module_loc_conf(r, module)  (r)->loc_conf[module.ctx_index]


#define ngx_http_conf_get_module_main_conf(cf, module)                        \
    ((ngx_http_conf_ctx_t *) cf->ctx)->main_conf[module.ctx_index]
#define ngx_http_conf_get_module_srv_conf(cf, module)                         \
    ((ngx_http_conf_ctx_t *) cf->ctx)->srv_conf[module.ctx_index]
#define ngx_http_conf_get_module_loc_conf(cf, module)                         \
    ((ngx_http_conf_ctx_t *) cf->ctx)->loc_conf[module.ctx_index]


#define ngx_http_cycle_get_module_main_conf(cycle, module)                    \
    (cycle->conf_ctx[ngx_http_module.index] ?                                 \
        ((ngx_http_conf_ctx_t *) cycle->conf_ctx[ngx_http_module.index])      \
            ->main_conf[module.ctx_index]:                                    \
        NULL)

```

## main 级别的配置项结构体

在 ngx\_http\_module 模块 http{} 块配置项解析的初始化过程中由函数 ngx\_http\_block 实现，在实现过程中创建并初始化了HTTP 模块main 级别的配置项main\_conf、srv\_conf、loc\_conf 结构体。main 级别的配置项结构体之间的关系如下图所示：

## server 级别的配置项结构体

在 ngx\_http\_module 模块在调用函数ngx\_conf\_parse 解析 http{} 块main 级别配置项时，若遇到server{} 块配置项，则会递归调用函数ngx\_conf\_parse 解析ngx\_http\_core\_module 模块中 server{} 块配置项，并调用方法ngx\_http\_core\_server 初始化server{} 块 ，该方法创建并初始化了HTTP 模块srv 级别的配置项srv\_conf、loc\_conf 结构体。server{} 块配置项的初始化函数创建配置项结构体的源码如下所示：

```text
static char *
ngx_http_core_server(ngx_conf_t *cf, ngx_command_t *cmd, void *dummy)
{
    char                        *rv;
    void                        *mconf;
    ngx_uint_t                   i;
    ngx_conf_t                   pcf;
    ngx_http_module_t           *module;
    struct sockaddr_in          *sin;
    ngx_http_conf_ctx_t         *ctx, *http_ctx;
    ngx_http_listen_opt_t        lsopt;
    ngx_http_core_srv_conf_t    *cscf, **cscfp;
    ngx_http_core_main_conf_t   *cmcf;

    
    ctx = ngx_pcalloc(cf->pool, sizeof(ngx_http_conf_ctx_t));
    if (ctx == NULL) {
        return NGX_CONF_ERROR;
    }

    
    http_ctx = cf->ctx;
    ctx->main_conf = http_ctx->main_conf;

    

    
    ctx->srv_conf = ngx_pcalloc(cf->pool, sizeof(void *) * ngx_http_max_module);
    if (ctx->srv_conf == NULL) {
        return NGX_CONF_ERROR;
    }

    

    
    ctx->loc_conf = ngx_pcalloc(cf->pool, sizeof(void *) * ngx_http_max_module);
    if (ctx->loc_conf == NULL) {
        return NGX_CONF_ERROR;
    }

    
    for (i = 0; ngx_modules[i]; i++) {
        if (ngx_modules[i]->type != NGX_HTTP_MODULE) {
            continue;
        }

        module = ngx_modules[i]->ctx;

        
        if (module->create_srv_conf) {
            mconf = module->create_srv_conf(cf);
            if (mconf == NULL) {
                return NGX_CONF_ERROR;
            }

            ctx->srv_conf[ngx_modules[i]->ctx_index] = mconf;
        }

        
        if (module->create_loc_conf) {
            mconf = module->create_loc_conf(cf);
            if (mconf == NULL) {
                return NGX_CONF_ERROR;
            }

            ctx->loc_conf[ngx_modules[i]->ctx_index] = mconf;
        }
    }

    
    

    cscf = ctx->srv_conf[ngx_http_core_module.ctx_index];
    cscf->ctx = ctx;

    cmcf = ctx->main_conf[ngx_http_core_module.ctx_index];

    cscfp = ngx_array_push(&cmcf->servers);
    if (cscfp == NULL) {
        return NGX_CONF_ERROR;
    }

    *cscfp = cscf;

    
    

    pcf = *cf;
    cf->ctx = ctx;
    cf->cmd_type = NGX_HTTP_SRV_CONF;

    rv = ngx_conf_parse(cf, NULL);

    
    *cf = pcf;

    if (rv == NGX_CONF_OK && !cscf->listen) {
        ngx_memzero(&lsopt, sizeof(ngx_http_listen_opt_t));

        sin = &lsopt.u.sockaddr_in;

        sin->sin_family = AF_INET;
#if (NGX_WIN32)
        sin->sin_port = htons(80);
#else
        sin->sin_port = htons((getuid() == 0) ? 80 : 8000);
#endif
        sin->sin_addr.s_addr = INADDR_ANY;

        lsopt.socklen = sizeof(struct sockaddr_in);

        lsopt.backlog = NGX_LISTEN_BACKLOG;
        lsopt.rcvbuf = -1;
        lsopt.sndbuf = -1;
#if (NGX_HAVE_SETFIB)
        lsopt.setfib = -1;
#endif
#if (NGX_HAVE_TCP_FASTOPEN)
        lsopt.fastopen = -1;
#endif
        lsopt.wildcard = 1;

        (void) ngx_sock_ntop(&lsopt.u.sockaddr, lsopt.socklen, lsopt.addr,
                             NGX_SOCKADDR_STRLEN, 1);

        if (ngx_http_add_listen(cf, cscf, &lsopt) != NGX_OK) {
            return NGX_CONF_ERROR;
        }
    }

    return rv;
}

```

srv 级别的配置项结构体之间的关系如下图所示：

## location 级别的配置项结构体

在 ngx\_http\_core\_module 模块在调用函数ngx\_conf\_parse 解析 server{} 块srv 级别配置项时，若遇到 location{} 块配置项，则会递归调用函数ngx\_conf\_parse 解析ngx\_http\_core\_module 模块中 location{} 块配置项，并调用方法ngx\_http\_core\_location 初始化location{} 块 ，该方法创建并初始化了HTTP 模块loc 级别的配置项loc\_conf 结构体。location{} 块配置项的初始化函数创建配置项结构体的源码如下所示：

```text
static char *
ngx_http_core_location(ngx_conf_t *cf, ngx_command_t *cmd, void *dummy)
{
    char                      *rv;
    u_char                    *mod;
    size_t                     len;
    ngx_str_t                 *value, *name;
    ngx_uint_t                 i;
    ngx_conf_t                 save;
    ngx_http_module_t         *module;
    ngx_http_conf_ctx_t       *ctx, *pctx;
    ngx_http_core_loc_conf_t  *clcf, *pclcf;

    
    ctx = ngx_pcalloc(cf->pool, sizeof(ngx_http_conf_ctx_t));
    if (ctx == NULL) {
        return NGX_CONF_ERROR;
    }

    
    pctx = cf->ctx;
    ctx->main_conf = pctx->main_conf;
    ctx->srv_conf = pctx->srv_conf;

    
    ctx->loc_conf = ngx_pcalloc(cf->pool, sizeof(void *) * ngx_http_max_module);
    if (ctx->loc_conf == NULL) {
        return NGX_CONF_ERROR;
    }

    
    for (i = 0; ngx_modules[i]; i++) {
        if (ngx_modules[i]->type != NGX_HTTP_MODULE) {
            continue;
        }

        module = ngx_modules[i]->ctx;

        
        if (module->create_loc_conf) {
            ctx->loc_conf[ngx_modules[i]->ctx_index] =
                                                   module->create_loc_conf(cf);
            
            if (ctx->loc_conf[ngx_modules[i]->ctx_index] == NULL) {
                 return NGX_CONF_ERROR;
            }
        }
    }

    clcf = ctx->loc_conf[ngx_http_core_module.ctx_index];
    clcf->loc_conf = ctx->loc_conf;

    value = cf->args->elts;

    
    if (cf->args->nelts == 3) {

        len = value[1].len;
        mod = value[1].data;
        name = &value[2];

        if (len == 1 && mod[0] == '=') {

            clcf->name = *name;
            clcf->exact_match = 1;

        } else if (len == 2 && mod[0] == '^' && mod[1] == '~') {

            clcf->name = *name;
            clcf->noregex = 1;

        } else if (len == 1 && mod[0] == '~') {

            if (ngx_http_core_regex_location(cf, clcf, name, 0) != NGX_OK) {
                return NGX_CONF_ERROR;
            }

        } else if (len == 2 && mod[0] == '~' && mod[1] == '*') {

            if (ngx_http_core_regex_location(cf, clcf, name, 1) != NGX_OK) {
                return NGX_CONF_ERROR;
            }

        } else {
            ngx_conf_log_error(NGX_LOG_EMERG, cf, 0,
                               "invalid location modifier \"%V\"", &value[1]);
            return NGX_CONF_ERROR;
        }

    } else {

        name = &value[1];

        if (name->data[0] == '=') {

            clcf->name.len = name->len - 1;
            clcf->name.data = name->data + 1;
            clcf->exact_match = 1;

        } else if (name->data[0] == '^' && name->data[1] == '~') {

            clcf->name.len = name->len - 2;
            clcf->name.data = name->data + 2;
            clcf->noregex = 1;

        } else if (name->data[0] == '~') {

            name->len--;
            name->data++;

            if (name->data[0] == '*') {

                name->len--;
                name->data++;

                if (ngx_http_core_regex_location(cf, clcf, name, 1) != NGX_OK) {
                    return NGX_CONF_ERROR;
                }

            } else {
                if (ngx_http_core_regex_location(cf, clcf, name, 0) != NGX_OK) {
                    return NGX_CONF_ERROR;
                }
            }

        } else {

            clcf->name = *name;

            if (name->data[0] == '@') {
                clcf->named = 1;
            }
        }
    }

    pclcf = pctx->loc_conf[ngx_http_core_module.ctx_index];

    if (pclcf->name.len) {

        

#if 0
        clcf->prev_location = pclcf;
#endif

        if (pclcf->exact_match) {
            ngx_conf_log_error(NGX_LOG_EMERG, cf, 0,
                               "location \"%V\" cannot be inside "
                               "the exact location \"%V\"",
                               &clcf->name, &pclcf->name);
            return NGX_CONF_ERROR;
        }

        if (pclcf->named) {
            ngx_conf_log_error(NGX_LOG_EMERG, cf, 0,
                               "location \"%V\" cannot be inside "
                               "the named location \"%V\"",
                               &clcf->name, &pclcf->name);
            return NGX_CONF_ERROR;
        }

        if (clcf->named) {
            ngx_conf_log_error(NGX_LOG_EMERG, cf, 0,
                               "named location \"%V\" can be "
                               "on the server level only",
                               &clcf->name);
            return NGX_CONF_ERROR;
        }

        len = pclcf->name.len;

#if (NGX_PCRE)
        if (clcf->regex == NULL
            && ngx_filename_cmp(clcf->name.data, pclcf->name.data, len) != 0)
#else
        if (ngx_filename_cmp(clcf->name.data, pclcf->name.data, len) != 0)
#endif
        {
            ngx_conf_log_error(NGX_LOG_EMERG, cf, 0,
                               "location \"%V\" is outside location \"%V\"",
                               &clcf->name, &pclcf->name);
            return NGX_CONF_ERROR;
        }
    }

    
    if (ngx_http_add_location(cf, &pclcf->locations, clcf) != NGX_OK) {
        return NGX_CONF_ERROR;
    }

    save = *cf;
    cf->ctx = ctx;
    cf->cmd_type = NGX_HTTP_LOC_CONF;

    
    rv = ngx_conf_parse(cf, NULL);

    *cf = save;

    return rv;
}

```

loc 级别的配置项结构体之间的关系如下图所示：若 location 是精确匹配、正则表达式、@命名则exact 字段有效，否则就是 inclusive 字段有效，画图过程中只画出exact 字段有效。

## 合并配置项

HTTP 框架解析完毕 http{} 块配置项时，会根据解析的结果进行合并配置项操作，即合并 http{}、server{}、location{} 不同级别下各 HTTP 模块生成的存放配置项的结构体。其合并过程在文件[src/http/ngx\_http.c](http://lxr.nginx.org/source/src/http/ngx_http.c)中定义，如下所示：

* 若 HTTP 模块实现了 _merge\_srv\_conf_ 方法，则将 http{} 块下由 _create\_srv\_conf_ 生成的 _main_ 级别结构体与遍历每一个 server{}块下由 _create\_srv\_conf_生成的_srv_ 级别的配置项结构体进行 _merge\_srv\_conf_ 操作；
* 若 HTTP 模块实现了 _merge\_loc\_conf_ 方法，则将 http{} 块下由 _create\_loc\_conf_ 生成的 _main_ 级别的配置项结构体与嵌套在每一个server{} 块下由 _create\_loc\_conf 生成的_srv级别的配置项结构体进行_merge\_loc\_conf_ 操作；
* 若 HTTP 模块实现了 _merge\_loc\_conf_ 方法，由于在上一步骤已经将main、srv级别由_create\_loc\_conf_ 生成的结构体进行合并，只要把上一步骤合并的结果在 server{} 块下与嵌套每一个location{}块下由_create\_loc\_conf_ 生成的配置项结构体再次进行_merge\_loc\_conf_ 操作；
* 若 HTTP 模块实现了 _merge\_loc\_conf_ 方法，则将上一步骤的合并结果与与嵌套每一个location{}块下由 _create\_loc\_conf_ 生成的的配置项结构体再次进行_merge\_loc\_conf_ 操作；

具体合并过程如下图所示：

```text

static char *
ngx_http_merge_servers(ngx_conf_t *cf, ngx_http_core_main_conf_t *cmcf,
    ngx_http_module_t *module, ngx_uint_t ctx_index)
{
    char                        *rv;
    ngx_uint_t                   s;
    ngx_http_conf_ctx_t         *ctx, saved;
    ngx_http_core_loc_conf_t    *clcf;
    ngx_http_core_srv_conf_t   **cscfp;

    
    cscfp = cmcf->servers.elts;
    ctx = (ngx_http_conf_ctx_t *) cf->ctx;
    saved = *ctx;
    rv = NGX_CONF_OK;

    
    for (s = 0; s < cmcf->servers.nelts; s++) {

        

        
        ctx->srv_conf = cscfp[s]->ctx->srv_conf;

        
        if (module->merge_srv_conf) {
            rv = module->merge_srv_conf(cf, saved.srv_conf[ctx_index],
                                        cscfp[s]->ctx->srv_conf[ctx_index]);
            if (rv != NGX_CONF_OK) {
                goto failed;
            }
        }

        
        if (module->merge_loc_conf) {

            

            ctx->loc_conf = cscfp[s]->ctx->loc_conf;

            rv = module->merge_loc_conf(cf, saved.loc_conf[ctx_index],
                                        cscfp[s]->ctx->loc_conf[ctx_index]);
            if (rv != NGX_CONF_OK) {
                goto failed;
            }

            

            

            
            clcf = cscfp[s]->ctx->loc_conf[ngx_http_core_module.ctx_index];

            rv = ngx_http_merge_locations(cf, clcf->locations,
                                          cscfp[s]->ctx->loc_conf,
                                          module, ctx_index);
            if (rv != NGX_CONF_OK) {
                goto failed;
            }
        }
    }

failed:

    *ctx = saved;

    return rv;
}

static char *
ngx_http_merge_locations(ngx_conf_t *cf, ngx_queue_t *locations,
    void **loc_conf, ngx_http_module_t *module, ngx_uint_t ctx_index)
{
    char                       *rv;
    ngx_queue_t                *q;
    ngx_http_conf_ctx_t        *ctx, saved;
    ngx_http_core_loc_conf_t   *clcf;
    ngx_http_location_queue_t  *lq;

    
    if (locations == NULL) {
        return NGX_CONF_OK;
    }

    ctx = (ngx_http_conf_ctx_t *) cf->ctx;
    saved = *ctx;

    

    
    for (q = ngx_queue_head(locations);
         q != ngx_queue_sentinel(locations);
         q = ngx_queue_next(q))
    {
        lq = (ngx_http_location_queue_t *) q;

        
        clcf = lq->exact ? lq->exact : lq->inclusive;
        
        ctx->loc_conf = clcf->loc_conf;

        
        rv = module->merge_loc_conf(cf, loc_conf[ctx_index],
                                    clcf->loc_conf[ctx_index]);
        if (rv != NGX_CONF_OK) {
            return rv;
        }

        
        rv = ngx_http_merge_locations(cf, clcf->locations, clcf->loc_conf,
                                      module, ctx_index);
        if (rv != NGX_CONF_OK) {
            return rv;
        }
    }

    *ctx = saved;

    return NGX_CONF_OK;
}

```

## HTTP 请求处理阶段

按照下列顺序将各个模块设置的phase handler依次加入cmcf-&gt;phase\_engine.handlers列表，各个phase的phase handler的checker不同。checker主要用于限定某个phase的框架逻辑，包括处理返回值。 在Nginx 定义了 11 个处理阶段，有一部分是不能添加 phase handler 方法的。在文件[src/http/ngx\_http\_core\_module.h](http://lxr.nginx.org/source/src/http/ngx_http_core_module.h)中定义，如下所示：

```text

typedef enum {
    
    NGX_HTTP_POST_READ_PHASE = 0,

    
    NGX_HTTP_SERVER_REWRITE_PHASE,

    
    NGX_HTTP_FIND_CONFIG_PHASE,
    
    NGX_HTTP_REWRITE_PHASE,
    
    NGX_HTTP_POST_REWRITE_PHASE,

    
    NGX_HTTP_PREACCESS_PHASE,

    
    NGX_HTTP_ACCESS_PHASE,
    
    NGX_HTTP_POST_ACCESS_PHASE,

    
    NGX_HTTP_TRY_FILES_PHASE,
    
    NGX_HTTP_CONTENT_PHASE,

    
    NGX_HTTP_LOG_PHASE
} ngx_http_phases;

```

每个HTTP的checker方法与handler处理如下所示：

```text
typedef struct ngx_http_phase_handler_s  ngx_http_phase_handler_t;

typedef ngx_int_t (*ngx_http_phase_handler_pt)(ngx_http_request_t *r,
    ngx_http_phase_handler_t *ph);

struct ngx_http_phase_handler_s {
    
    ngx_http_phase_handler_pt  checker;
    
    ngx_http_handler_pt        handler;
    
    ngx_uint_t                 next;
};

```

完成 http{} 块的解析后，根据 \*nginx.conf \*文件中配置产生由 ngx\_http\_phase\_handler\_t  组成的数组，在处理 HTTP 请求时，一般情况下按照阶段的方向顺序 phase handler 加入到回调表中。ngx\_http\_phase\_engine\_t 结构体由所有ngx\_http\_phase\_handler\_t 组成的数组，如下所示：

```text
typedef struct {
    
    ngx_http_phase_handler_t  *handlers;
    
    ngx_uint_t                 server_rewrite_index;
    
    ngx_uint_t                 location_rewrite_index;
} ngx_http_phase_engine_t;

```

ngx\_http\_phase\_engine\_t 中保存在当前_nginx.conf_ 配置下，一个用户请求可能经历的所有 ngx\_http\_handler\_pt 处理方法。

```text
typedef struct {
    
    ngx_array_t                handlers;
} ngx_http_phase_t;

```

在 HTTP模块初始化过程中，HTTP模块通过postconfiguration方法将自定义的方法添加到handler数组中，即该方法会被添加到phase\_engine数组中。下面以NGX\_HTTP\_POST\_READ\_PHASE阶段为例，讲解了该阶段的checker方法的实现：

```text
ngx_int_t
ngx_http_core_generic_phase(ngx_http_request_t *r, ngx_http_phase_handler_t *ph)
{
    ngx_int_t  rc;

    

    ngx_log_debug1(NGX_LOG_DEBUG_HTTP, r->connection->log, 0,
                   "generic phase: %ui", r->phase_handler);

    
    rc = ph->handler(r);

    
    if (rc == NGX_OK) {
        r->phase_handler = ph->next;
        return NGX_AGAIN;
    }

    
    if (rc == NGX_DECLINED) {
        r->phase_handler++;
        return NGX_AGAIN;
    }

    
    if (rc == NGX_AGAIN || rc == NGX_DONE) {
        return NGX_OK;
    }

    

    
    ngx_http_finalize_request(r, rc);

    return NGX_OK;
}

```

