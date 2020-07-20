# Nginx 基本数据结构

## 概述

在学习 Nginx 之前首先了解其基本的数据结构是非常重要的，这是入门必须了解的一个步骤。本节只是简单介绍了 Nginx 对基本数据的一种封装，包括 基本整型数据类型、字符串数据类型、缓冲区类型以及 chain 数据类型。

## 基本数据类型

### 整型数据

```text





typedef intptr_t        ngx_int_t;
typedef uintptr_t       ngx_uint_t;
typedef intptr_t        ngx_flag_t;



#if __WORDSIZE == 64
# ifndef __intptr_t_defined
typedef long int		intptr_t;
#  define __intptr_t_defined
# endif
typedef unsigned long int	uintptr_t;
#else
# ifndef __intptr_t_defined
typedef int			intptr_t;
#  define __intptr_t_defined
# endif
typedef unsigned int		uintptr_t;



```

### 字符串类型

```text




typedef struct {
    size_t      len;    
    u_char     *data;   
} ngx_str_t;

typedef struct {
    ngx_str_t   key;
    ngx_str_t   value;
} ngx_keyval_t;

typedef struct {
    unsigned    len:28;

    unsigned    valid:1;
    unsigned    no_cacheable:1;
    unsigned    not_found:1;
    unsigned    escape:1;

    u_char     *data;
} ngx_variable_value_t;


#define ngx_string(str) {sizeof(str)-1, (u_char *) str}
#define ngx_null_string {0, NULL}



#define ngx_str_set(str, text)
    (str)->len = sizeof(text)-1; (str)->data = (u_char *)text

#define ngx_str_null(str)   (str)->len = 0; (str)->data = NULL



ngx_str_t str1 = ngx_string("hello nginx");
ngx_str_t str2 = ngx_null_string;


ngx_str_t str1, str2;
str1 = ngx_string("hello nginx");   
str2 = ngx_null_string;             


ngx_str_t str1, str2;
ngx_str_set(&str1, "hello nginx");
ngx_str_null(&str2);


```

### 内存池类型

内存池类型即是 ngx\_pool\_t ，有关内存池的讲解可参考前文《[Nginx 内存池管理](http://blog.csdn.net/chenhanzhun/article/details/42365605)》

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

### 缓冲区数据类型

缓冲区 ngx\_buf\_t 的定义如下：

```text

typedef void *            ngx_buf_tag_t;

typedef struct ngx_buf_s  ngx_buf_t;

struct ngx_buf_s {
    u_char          *pos;   
    u_char          *last;  
    
    off_t            file_pos;
    off_t            file_last;

    
    u_char          *start;         
    u_char          *end;           
    ngx_buf_tag_t    tag;
    ngx_file_t      *file;          
    
    ngx_buf_t       *shadow;

    
    
    unsigned         temporary:1;

    
    
    unsigned         memory:1;

    
    
    unsigned         mmap:1;

    
    unsigned         recycled:1;
    unsigned         in_file:1; 
    unsigned         flush:1;   
    unsigned         sync:1;    
    unsigned         last_buf:1;
    unsigned         last_in_chain:1;

    unsigned         last_shadow:1;
    unsigned         temp_file:1;

     int   num;
};

```

### chain 数据类型

ngx\_chain\_t 数据类型是与缓冲区类型 ngx\_buf\_t 相关的链表结构，定义如下：

```text
struct ngx_chain_s {
    ngx_buf_t    *buf;  
    ngx_chain_t  *next; 
};
typedef struct {

```

链表图如下：  


参考资料：

《深入理解 Nginx 》

《[Nginx 从入门到精通](http://tengine.taobao.org/book/chapter_02.html#id3)》

《[Nginx 代码研究](https://code.google.com/p/nginxsrp/wiki/NginxCodeReview#ngx%E7%9A%84%E5%86%85%E5%AD%98%E6%B1%A0)》

