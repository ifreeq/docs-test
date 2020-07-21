# Nginx 配置解析

## 概述

在上一篇文章《 [Nginx 启动初始化过程](http://blog.csdn.net/chenhanzhun/article/details/42611315)》简单介绍了 Nginx 启动的过程，并分析了其启动过程的源码。在启动过程中有一个步骤非常重要，就是调用函数 ngx\_init\_cycle\(\)，该函数的调用为配置解析提供了接口。配置解析接口大概可分为两个阶段：**准备数据阶段**和**配置解析阶段**；

准备数据阶段包括：

* 准备内存；
* 准备错误日志；
* 准备所需数据结构；

配置解析阶段是调用函数：

```text
      
    if (ngx_conf_param(&conf) != NGX_CONF_OK) {  
        environ = senv;  
        ngx_destroy_cycle_pools(&conf);  
        return NULL;  
    }  
  
    if (ngx_conf_parse(&conf, &cycle->conf_file) != NGX_CONF_OK) {  
        environ = senv;  
        ngx_destroy_cycle_pools(&conf);  
        return NULL;  
    }  

```

## 配置解析

## ngx\_conf\_t 结构体

该结构体用于 Nginx 在解析配置文件时描述每个指令的属性，也是Nginx 程序中非常重要的一个数据结构，其定义于文件：[src/core/ngx\_conf\_file.h](http://lxr.nginx.org/source/src/core/ngx_conf_file.h)

```text

struct ngx_conf_s {
    char                 *name;     
    ngx_array_t          *args;     

    ngx_cycle_t          *cycle;    
    ngx_pool_t           *pool;     
    ngx_pool_t           *temp_pool;
    ngx_conf_file_t      *conf_file;
    ngx_log_t            *log;      

    void                 *ctx;      
    ngx_uint_t            module_type;
    ngx_uint_t            cmd_type; 

    ngx_conf_handler_pt   handler;  
    char                 *handler_conf;
};

```

### 配置文件信息 conf\_file

conf\_file 是存放 Nginx 配置文件的相关信息。ngx\_conf\_file\_t 结构体的定义如下：

```text
typedef struct {
    ngx_file_t            file;     
    ngx_buf_t            *buffer;   
    ngx_uint_t            line;     
} ngx_conf_file_t;
```

### 配置上下文 ctx

Nginx 的配置文件是分块配置的，常见的有http 块、server 块、location 块以及upsteam 块和 mail 块等。每一个这样的配置块代表一个作用域。高一级配置块的作用域包含了多个低一级配置块的作用域，也就是有作用域嵌套的现象。这样，配置文件中的许多指令都会同时包含在多个作用域内。比如，http 块中的指令都可能同时处于http 块、server 块和location 块等三层作用域内。

在 Nginx 程序解析配置文件时，每一条指令都应该记录自己所属的作用域范围，而配置文件上下文ctx 变量就是用来存放当前指令所属的作用域的。在Nginx 配置文件的各种配置块中，http 块可以包含子配置块，这在存储结构上比较复杂。

### 指令类型 type

Nginx 程序中的不同的指令类型以宏的形式定义在不同的源码头文件中，指令类型是core 模块类型的定义在文件：[src/core/ngx\_conf\_file.h](http://lxr.nginx.org/source/src/core/ngx_conf_file.h)

```text
#define NGX_DIRECT_CONF            0x00010000  
#define NGX_MAIN_CONF              0x01000000  
#define NGX_ANY_CONF               0x0F000000 
```

这些是 core 类型模块支持的指令类型。其中的 NGX\_DIRECT\_CONF类指令在 Nginx 程序进入配置解析函数之前已经初始化完成，所以在进入配置解析函数之后可以将它们直接解析并存储到实际的数据结构中，从配置文件的结构上来看，它们一般指的就是那些游离于配置块之外、处于配置文件全局块部分的指令。NGX\_MAIN\_CONF 类指令包括event、http、mail、upstream 等可以形成配置块的指令，它们没有自己的初始化函数。Nginx 程序在解析配置文件时如果遇到 NGX\_MAIN\_CONF 类指令，将转入对下一级指令的解析。  
       以下是 event 类型模块支持的指令类型。

```text
#define NGX_EVENT_CONF            0x02000000 
```

以下是 http 类型模块支持的指令类型，其定义在文件：[src/http/ngx\_http\_config.h](http://lxr.nginx.org/source/src/http/ngx_http_config.h)

```text
#define NGX_HTTP_MAIN_CONF          0x02000000  
#define NGX_HTTP_SRV_CONF           0x04000000  
#define NGX_HTTP_LOC_CONF           0x08000000  
#define NGX_HTTP_UPS_CONF           0x10000000  
#define NGX_HTTP_SIF_CONF           0x20000000  
#define NGX_HTTP_LIF_CONF           0x40000000  
#define NGX_HTTP_LMT_CONF           0x80000000  
```

## 通用模块配置解析

配置解析模块在 [src/core/ngx\_conf\_file.c](http://lxr.nginx.org/source/src/core/ngx_conf_file.c) 中实现。模块提供的接口函数主要是ngx\_conf\_parse。另外，模块提供另一个单独的接口ngx\_conf\_param，用来解析命令行传递的配置，这个接口也是对ngx\_conf\_parse 的包装。首先看下配置解析函数 ngx\_conf\_parse，其定义如下：

```text

char *
ngx_conf_parse(ngx_conf_t *cf, ngx_str_t *filename)
{
    char             *rv;
    ngx_fd_t          fd;
    ngx_int_t         rc;
    ngx_buf_t         buf;
    ngx_conf_file_t  *prev, conf_file;
    enum {
        parse_file = 0,
        parse_block,
        parse_param
    } type;

#if (NGX_SUPPRESS_WARN)
    fd = NGX_INVALID_FILE;
    prev = NULL;
#endif

    if (filename) {

        

        
        fd = ngx_open_file(filename->data, NGX_FILE_RDONLY, NGX_FILE_OPEN, 0);
        if (fd == NGX_INVALID_FILE) {
            ngx_conf_log_error(NGX_LOG_EMERG, cf, ngx_errno,
                               ngx_open_file_n " \"%s\" failed",
                               filename->data);
            return NGX_CONF_ERROR;
        }

        prev = cf->conf_file;

        cf->conf_file = &conf_file;

        if (ngx_fd_info(fd, &cf->conf_file->file.info) == NGX_FILE_ERROR) {
            ngx_log_error(NGX_LOG_EMERG, cf->log, ngx_errno,
                          ngx_fd_info_n " \"%s\" failed", filename->data);
        }

        cf->conf_file->buffer = &buf;

        buf.start = ngx_alloc(NGX_CONF_BUFFER, cf->log);
        if (buf.start == NULL) {
            goto failed;
        }

        buf.pos = buf.start;
        buf.last = buf.start;
        buf.end = buf.last + NGX_CONF_BUFFER;
        buf.temporary = 1;

        
        cf->conf_file->file.fd = fd;
        cf->conf_file->file.name.len = filename->len;
        cf->conf_file->file.name.data = filename->data;
        cf->conf_file->file.offset = 0;
        cf->conf_file->file.log = cf->log;
        cf->conf_file->line = 1;

        type = parse_file;  

    } else if (cf->conf_file->file.fd != NGX_INVALID_FILE) {

        type = parse_block; 

    } else {
        type = parse_param; 
    }

    for ( ;; ) {
        
        rc = ngx_conf_read_token(cf);

        

        if (rc == NGX_ERROR) {
            goto done;
        }

        
        if (rc == NGX_CONF_BLOCK_DONE) {

            if (type != parse_block) {
                ngx_conf_log_error(NGX_LOG_EMERG, cf, 0, "unexpected \"}\"");
                goto failed;
            }

            goto done;
        }

        
        if (rc == NGX_CONF_FILE_DONE) {

            if (type == parse_block) {
                ngx_conf_log_error(NGX_LOG_EMERG, cf, 0,
                                   "unexpected end of file, expecting \"}\"");
                goto failed;
            }

            goto done;
        }

        if (rc == NGX_CONF_BLOCK_START) {

            if (type == parse_param) {
                ngx_conf_log_error(NGX_LOG_EMERG, cf, 0,
                                   "block directives are not supported "
                                   "in -g option");
                goto failed;
            }
        }

        

        
        if (cf->handler) {

            

            if (rc == NGX_CONF_BLOCK_START) {
                ngx_conf_log_error(NGX_LOG_EMERG, cf, 0, "unexpected \"{\"");
                goto failed;
            }

            
            rv = (*cf->handler)(cf, NULL, cf->handler_conf);
            if (rv == NGX_CONF_OK) {
                continue;
            }

            if (rv == NGX_CONF_ERROR) {
                goto failed;
            }

            ngx_conf_log_error(NGX_LOG_EMERG, cf, 0, rv);

            goto failed;
        }

        
        rc = ngx_conf_handler(cf, rc);

        if (rc == NGX_ERROR) {
            goto failed;
        }
    }

failed:

    rc = NGX_ERROR;

done:

    if (filename) {
        if (cf->conf_file->buffer->start) {
            ngx_free(cf->conf_file->buffer->start);
        }

        if (ngx_close_file(fd) == NGX_FILE_ERROR) {
            ngx_log_error(NGX_LOG_ALERT, cf->log, ngx_errno,
                          ngx_close_file_n " %s failed",
                          filename->data);
            return NGX_CONF_ERROR;
        }

        cf->conf_file = prev;
    }

    if (rc == NGX_ERROR) {
        return NGX_CONF_ERROR;
    }

    return NGX_CONF_OK;
}

```

从配置解析函数的源码可以看出，该函数分为两个阶段：**语法分析**和 **指令解析**。语法分析由 ngx\_conf\_read\_token\(\)函数完成。指令解析有两种方式：一种是Nginx 内建的指令解析机制；另一种是自定义的指令解析机制。自定义指令解析源码如下所示：

```text
        
        if (cf->handler) {

            

            if (rc == NGX_CONF_BLOCK_START) {
                ngx_conf_log_error(NGX_LOG_EMERG, cf, 0, "unexpected \"{\"");
                goto failed;
            }

            
            rv = (*cf->handler)(cf, NULL, cf->handler_conf);
            if (rv == NGX_CONF_OK) {
                continue;
            }

            if (rv == NGX_CONF_ERROR) {
                goto failed;
            }

            ngx_conf_log_error(NGX_LOG_EMERG, cf, 0, rv);

            goto failed;
        }

```

而Nginx 内置解析机制有函数ngx\_conf\_handler\(\) 实现。其定义如下：

```text

static ngx_int_t
ngx_conf_handler(ngx_conf_t *cf, ngx_int_t last)
{
    char           *rv;
    void           *conf, **confp;
    ngx_uint_t      i, found;
    ngx_str_t      *name;
    ngx_command_t  *cmd;

    name = cf->args->elts;

    found = 0;

    for (i = 0; ngx_modules[i]; i++) {

        cmd = ngx_modules[i]->commands;
        if (cmd == NULL) {
            continue;
        }

        for (  ; cmd->name.len; cmd++) {

            if (name->len != cmd->name.len) {
                continue;
            }

            if (ngx_strcmp(name->data, cmd->name.data) != 0) {
                continue;
            }

            found = 1;

            
            if (ngx_modules[i]->type != NGX_CONF_MODULE
                && ngx_modules[i]->type != cf->module_type)
            {
                continue;
            }

            

            if (!(cmd->type & cf->cmd_type)) {
                continue;
            }

            
            if (!(cmd->type & NGX_CONF_BLOCK) && last != NGX_OK) {
                ngx_conf_log_error(NGX_LOG_EMERG, cf, 0,
                                  "directive \"%s\" is not terminated by \";\"",
                                  name->data);
                return NGX_ERROR;
            }

            
            if ((cmd->type & NGX_CONF_BLOCK) && last != NGX_CONF_BLOCK_START) {
                ngx_conf_log_error(NGX_LOG_EMERG, cf, 0,
                                   "directive \"%s\" has no opening \"{\"",
                                   name->data);
                return NGX_ERROR;
            }

            

            
            if (!(cmd->type & NGX_CONF_ANY)) {

                
                if (cmd->type & NGX_CONF_FLAG) {

                    if (cf->args->nelts != 2) {
                        goto invalid;
                    }

                } else if (cmd->type & NGX_CONF_1MORE) {

                    if (cf->args->nelts < 2) {
                        goto invalid;
                    }

                } else if (cmd->type & NGX_CONF_2MORE) {

                    if (cf->args->nelts < 3) {
                        goto invalid;
                    }

                } else if (cf->args->nelts > NGX_CONF_MAX_ARGS) {

                    goto invalid;

                } else if (!(cmd->type & argument_number[cf->args->nelts - 1]))
                {
                    goto invalid;
                }
            }

            

            conf = NULL;

            if (cmd->type & NGX_DIRECT_CONF) {
                conf = ((void **) cf->ctx)[ngx_modules[i]->index];

            } else if (cmd->type & NGX_MAIN_CONF) {
                conf = &(((void **) cf->ctx)[ngx_modules[i]->index]);

            } else if (cf->ctx) {
                confp = *(void **) ((char *) cf->ctx + cmd->conf);

                if (confp) {
                    conf = confp[ngx_modules[i]->ctx_index];
                }
            }

            
            rv = cmd->set(cf, cmd, conf);

            if (rv == NGX_CONF_OK) {
                return NGX_OK;
            }

            if (rv == NGX_CONF_ERROR) {
                return NGX_ERROR;
            }

            ngx_conf_log_error(NGX_LOG_EMERG, cf, 0,
                               "\"%s\" directive %s", name->data, rv);

            return NGX_ERROR;
        }
    }

    if (found) {
        ngx_conf_log_error(NGX_LOG_EMERG, cf, 0,
                           "\"%s\" directive is not allowed here", name->data);

        return NGX_ERROR;
    }

    ngx_conf_log_error(NGX_LOG_EMERG, cf, 0,
                       "unknown directive \"%s\"", name->data);

    return NGX_ERROR;

invalid:

    ngx_conf_log_error(NGX_LOG_EMERG, cf, 0,
                       "invalid number of arguments in \"%s\" directive",
                       name->data);

    return NGX_ERROR;
}

```

## HTTP 模块配置解析

这里主要是结构体 _ngx\_command\_t_ ，我们在文章 《[Nginx 模块开发](http://blog.csdn.net/chenhanzhun/article/details/42528951)》 对该结构体作了介绍，其定义如下：

```text
struct ngx_command_s {  
      
    ngx_str_t             name;  
      
    ngx_uint_t            type;  
      
    char               *(*set)(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);  
      
    ngx_uint_t            conf;  
    ngx_uint_t            offset;  
      
    void                 *post;  
}; 

```

若在上面的通用配置解析中，定义了如下的 http 配置项结构，则回调用http 配置项，并对该http 配置项进行解析。此时，解析的是http block 块设置。

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

http 是作为一个 core 模块被 nginx 通用解析过程解析的，其核心就是http{} 块指令回调，它完成了http 解析的整个功能，从初始化到计算配置结果。http{} 块指令的流程是：

* 创建并初始化上下文结构；
* 调用通用模块配置解析流程解析；
* 根据解析结果进行配置项合并处理；

## 创建并初始化上下文结构

当 Nginx 检查到 http{…} 配置项时，HTTP 配置模型就会启动，则会建立一个_ngx\_http\_conf\_ctx\_t_ 结构，该结构定义在文件中：[src/http/ngx\_http\_config.h](http://lxr.nginx.org/source/src/core/ngx_conf_file.h)

```text
typedef struct{
　　
   void **main_conf;
   
   void **srv_conf;
   
   void **loc_conf;
}ngx_http_conf_ctx_t;

```

此时，HTTP 框架为所有 HTTP 模块建立 3 个数组，分别存放所有 HTTP 模块的_create\_main\_conf_、_create\_srv\_conf_ 、_create\_loc\_conf_ 方法返回的地址指针。_ngx\_http\_conf\_ctx\_t_ 结构的三个成员分别指向这 3 个数组。例如下面的例子是设置 _create\_main\_conf_、_create\_srv\_conf_ 、_create\_loc\_conf_  返回的地址。

```text
ngx_http_conf_ctx *ctx;

ctx = ngx_pcalloc(cf->pool, sizeof(ngx_http_conf_ctx_t));

*(ngx_http_conf_ctx_t **) conf = ctx;

...

ctx->main_conf = ngx_pcalloc(cf->pool,
                             sizeof(void *) * ngx_http_max_module);

ctx->srv_conf = ngx_pcalloc(cf->pool, sizeof(void *) * ngx_http_max_module);

ctx->loc_conf = ngx_pcalloc(cf->pool, sizeof(void *) * ngx_http_max_module);


for (m = 0; ngx_modules[m]; m++) {
    if (ngx_modules[m]->type != NGX_HTTP_MODULE) {
        continue;
    }

    module = ngx_modules[m]->ctx;
    mi = ngx_modules[m]->ctx_index;

    
    if (module->create_main_conf) {
        ctx->main_conf[mi] = module->create_main_conf(cf);
    }
    
    if (module->create_srv_conf) {
        ctx->srv_conf[mi] = module->create_srv_conf(cf);
    }
    
    if (module->create_loc_conf) {
        ctx->loc_conf[mi] = module->create_loc_conf(cf);
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

```

## 调用通用模块配置解析流程解析

从源码 [src/http/ngx\_http.c](http://lxr.nginx.org/source/src/http/ngx_http.c) 中可以看到，http 块的配置解析是调用通用模块的配置解析函数，其实现如下：

```text
    
    

    cf->module_type = NGX_HTTP_MODULE;
    cf->cmd_type = NGX_HTTP_MAIN_CONF;
    rv = ngx_conf_parse(cf, NULL);

    if (rv != NGX_CONF_OK) {
        goto failed;
    }

```

## 根据解析结果进行配置项合并处理

```text
    
    

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

```

## HTTP 配置解析流程

从上面的分析中可以总结出 HTTP 配置解析的流程如下：

* Nginx 进程进入主循环，在主循环中调用配置解析器解析配置文件_nginx.conf_;
* 在配置文件中遇到 http{} 块配置，则 HTTP 框架开始初始化并启动，其由函数 ngx\_http\_block\(\) 实现；
* HTTP 框架初始化所有 HTTP 模块的序列号，并创建 3 个类型为 _ngx\_http\_conf\_ctx\_t 结构的数组用于存储所有HTTP 模块的_create\_main\_conf、_create\_srv\_conf_、_create\_loc\_conf_方法返回的指针地址；
* 调用每个 HTTP 模块的 preconfiguration 方法；
* HTTP 框架调用函数 ngx\_conf\_parse\(\) 开始循环解析配置文件 \*nginx.conf \*中的http{}块里面的所有配置项；
* HTTP 框架处理完毕 http{} 配置项，根据解析配置项的结果，必要时进行配置项合并处理；
* 继续处理其他 http{} 块之外的配置项，直到配置文件解析器处理完所有配置项后通知Nginx 主循环配置项解析完毕。此时，Nginx 才会启动Web 服务器；

## 合并配置项

HTTP 框架解析完毕 http{} 块配置项时，会根据解析的结果进行合并配置项操作，即合并 http{}、server{}、location{} 不同块下各HTTP 模块生成的存放配置项的结构体。其合并过程如下所示：

* 若 HTTP 模块实现了 _merge\_srv\_conf_ 方法，则将 http{} 块下_create\_srv\_conf_ 生成的结构体与遍历每一个 server{}配置块下的结构体进行_merge\_srv\_conf_ 操作；
* 若 HTTP 模块实现了 _merge\_loc\_conf_ 方法，则将 http{} 块下_create\_loc\_conf_ 生成的结构体与嵌套每一个server{} 配置块下的结构体进行_merge\_loc\_conf_ 操作；
* 若 HTTP 模块实现了 _merge\_loc\_conf_ 方法，则将server{} 块下_create\_loc\_conf_ 生成的结构体与嵌套每一个location{}配置块下的结构体进行_merge\_loc\_conf_ 操作；
* 若 HTTP 模块实现了 _merge\_loc\_conf_ 方法，则将location{} 块下_create\_loc\_conf_ 生成的结构体与嵌套每一个location{}配置块下的结构体进行_merge\_loc\_conf_ 操作；

以下是合并配置项操作的源码实现：

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

## 处理自定义的配置

在文章中 《[Nginx 模块开发](http://blog.csdn.net/chenhanzhun/article/details/42528951)》，我们给出了“Hello World” 的开发例子，在这个开发例子中，我们定义了自己的配置项，配置项名称的结构体定义如下：

```text
typedef struct  
{  
        ngx_str_t hello_string;  
        ngx_int_t hello_counter;  
}ngx_http_hello_loc_conf_t;  

```

为了处理我们定义的配置项结构，因此，我们把 _ngx\_command\_t_ 结构体定义如下：

```text
static ngx_command_t ngx_http_hello_commands[] = {  
   {  
                ngx_string("hello_string"),  
                NGX_HTTP_LOC_CONF|NGX_CONF_NOARGS|NGX_CONF_TAKE1,  
                ngx_http_hello_string,  
                NGX_HTTP_LOC_CONF_OFFSET,  
                offsetof(ngx_http_hello_loc_conf_t, hello_string),  
                NULL },  
  
        {  
                ngx_string("hello_counter"),  
                NGX_HTTP_LOC_CONF|NGX_CONF_FLAG,  
                ngx_http_hello_counter,  
                NGX_HTTP_LOC_CONF_OFFSET,  
                offsetof(ngx_http_hello_loc_conf_t, hello_counter),  
                NULL },  
  
        ngx_null_command  
};  

```

处理方法 _ngx\_http\_hello\_string_ 和_ngx\_http\_hello\_counter_ 定义如下：

```text
static char *  
ngx_http_hello_string(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)  
{  
  
        ngx_http_hello_loc_conf_t* local_conf;  
  
  
        local_conf = conf;  
        char* rv = ngx_conf_set_str_slot(cf, cmd, conf);  
  
        ngx_conf_log_error(NGX_LOG_EMERG, cf, 0, "hello_string:%s", local_conf->hello_string.data);  
  
        return rv;  
}  
  
  
static char *ngx_http_hello_counter(ngx_conf_t *cf, ngx_command_t *cmd,  
        void *conf)  
{  
        ngx_http_hello_loc_conf_t* local_conf;  
  
        local_conf = conf;  
  
        char* rv = NULL;  
  
        rv = ngx_conf_set_flag_slot(cf, cmd, conf);  
  
  
        ngx_conf_log_error(NGX_LOG_EMERG, cf, 0, "hello_counter:%d", local_conf->hello_counter);  
        return rv;  
}  

```

## error 日志

Nginx 日志模块为其他模块提供了基本的日志记录功能，日志模块定义如下：[src/​core/​ngx\_log.c](http://lxr.nginx.org/source/src/core/ngx_log.c)

```text
static ngx_command_t  ngx_errlog_commands[] = {

    {ngx_string("error_log"),
     NGX_MAIN_CONF|NGX_CONF_1MORE,
     ngx_error_log,
     0,
     0,
     NULL},

    ngx_null_command
};

static ngx_core_module_t  ngx_errlog_module_ctx = {
    ngx_string("errlog"),
    NULL,
    NULL
};

ngx_module_t  ngx_errlog_module = {
    NGX_MODULE_V1,
    &ngx_errlog_module_ctx,                
    ngx_errlog_commands,                   
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

Nginx 日志模块对于支持可变参数提供了三个接口，这三个接口定义在文件：[src/​core/​ngx\_log.h](http://lxr.nginx.org/source/src/core/ngx_log.h)

```text
#define ngx_log_error(level, log, ...)                                        \
    if ((log)->log_level >= level) ngx_log_error_core(level, log, __VA_ARGS__)

void ngx_log_error_core(ngx_uint_t level, ngx_log_t *log, ngx_err_t err,
    const char *fmt, ...);

#define ngx_log_debug(level, log, ...)                                        \
    if ((log)->log_level & level)                                             \
        ngx_log_error_core(NGX_LOG_DEBUG, log, __VA_ARGS__)

```

Nginx 日志模块记录日志的核心功能是由ngx\_log\_error\_core 方法实现，ngx\_log\_error 和ngx\_log\_debug 宏定义只是对其进行简单的封装，一般情况下日志调用只需要这两个宏定义。  
       ngx\_log\_error 和 ngx\_log\_debug 宏定义都包括参数 level、log、err、fmt，下面分别对这些参数进行简单的介绍：

**level 参数**：对于 ngx\_log\_error 宏来说，level 表示当前日志的级别，其取值如下所示：

```text

#define NGX_LOG_STDERR            0     
#define NGX_LOG_EMERG             1
#define NGX_LOG_ALERT             2
#define NGX_LOG_CRIT              3
#define NGX_LOG_ERR               4
#define NGX_LOG_WARN              5
#define NGX_LOG_NOTICE            6
#define NGX_LOG_INFO              7
#define NGX_LOG_DEBUG             8     

```

使用 ngx\_log\_error 宏记录日志时，若传入的level 级别小于或等于log 参数中的日志级别，就会输出日志内容，否则忽略该日志。  
       在使用 ngx\_log\_debug 宏时，参数level 不同于ngx\_log\_error 宏的level 参数，它表达的不是日志级别，而是日志类型。ngx\_log\_debug 宏记录日志时必须是NGX\_LOG\_DEBUG 调试级别，这里的level 取值如下：

```text

#define NGX_LOG_DEBUG_CORE        0x010 
#define NGX_LOG_DEBUG_ALLOC       0x020 
#define NGX_LOG_DEBUG_MUTEX       0x040 
#define NGX_LOG_DEBUG_EVENT       0x080 
#define NGX_LOG_DEBUG_HTTP        0x100 
#define NGX_LOG_DEBUG_MAIL        0x200 
#define NGX_LOG_DEBUG_MYSQL       0x400 

```

当 HTTP 模块调用ngx\_log\_debug 宏记录日志时，传入的level 参数是NGX\_LOG\_DEBUG\_HTTP，此时，若log 参数不属于HTTP 模块，若使用event 事件模块的log，则不会输出任何日志。

**log 参数**：log 参数的结构定义如下：[src/core/ngx\_log.h](http://lxr.nginx.org/source/src/core/ngx_log.h)；从其结构中可以知道，若只想把相应的信息记录到日志文件中，则不需要关系参数 log 的构造。

```text

struct ngx_log_s {
    
    ngx_uint_t           log_level;
    
    ngx_open_file_t     *file;

    
    ngx_atomic_uint_t    connection;

    
    ngx_log_handler_pt   handler;
    
    void                *data;

    

    char                *action;

    
    ngx_log_t           *next;
};

```

**err 参数**：err 参数是错误编码，一般是执行系统调用失败后取得的errno 参数。当err 不为 0 时，Nginx 日志模块将会在正常日志输出这个错误编码以及其对应的字符串形成的错误信息。

**fmt 参数**：fmt 参数类似于C 语言中的printf 函数的输出格式。

参考资料：

《深入理解 Nginx 》

《[nginx 启动阶段](http://tengine.taobao.org/book/chapter_11.html)》

《Nginx高性能Web服务器详解》

