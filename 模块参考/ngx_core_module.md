# 核心功能

- [示例配置](#example)
- [指令](#directives)
    - [accept_mutex](#accept_mutex)
    - [accept_mutex_delay](#accept_mutex_delay)
    - [daemon](#daemon)
    - [debug_connection](#debug_connection)
    - [debug_points](#debug_points)
    - [env](#env)
    - [error_log](#error_log)
    - [events](#events)
    - [include](#include)
    - [load_module](#load_module)
    - [lock_file](#lock_file)
    - [master_process](#master_process)
    - [multi_accept](#multi_accept)
    - [pcre_jit](#pcre_jit)
    - [pid](#pid)
    - [ssl_engine](#ssl_engine)
    - [thread_pool](#thread_pool)
    - [timer_resolution](#timer_resolution)
    - [use](#use)
    - [user](#user)
    - [worker_aio_requests](#worker_aio_requests)
    - [worker_connections](#worker_connections)
    - [worker_cpu_affinity](#worker_cpu_affinity)
    - [worker_priority](#worker_priority)
    - [worker_processes](#worker_processes)
    - [worker_rlimit_core](#worker_rlimit_core)
    - [worker_rlimit_nofile](#worker_rlimit_nofile)
    - [worker_shutdown_timeout](#worker_shutdown_timeout)
    - [working_directory](#working_directory)

<a id="example"></a>

## 示例配置
```nginx
user www www;
worker_processes 2;

error_log /var/log/nginx-error.log info;

events {
    use kqueue;
    worker_connections 2048;
}

...
```

<a id="directives"></a>

## 指令

### accept_mutex

|\-|说明|
|:------|:------|
|**语法**|**accept_mutex** `on` &#124; `off`;|
|**默认**|accept_mutex off;|
|**上下文**|events|

如果启用了 `accept_mutex`，则工作进程将依次接受新连接。否则，将新连接通知给所有工作进程，如果新连接数量较少，某些工作进程可能会浪费系统资源。

> 在支持 [EPOLLEXCLUSIVE](http://nginx.org/en/docs/events.html#epoll) 标志（1.11.3）或使用 [reuseport](#reuseport) 的系统上，不需要启用 `accept_mutex`。

> 在版本 1.11.3 之前，默认值为 `on`。

### accept_mutex_delay

|\-|说明|
|:------|:------|
|**语法**|**accept_mutex_delay** `time`;|
|**默认**|accept_mutex_delay 500ms;|
|**上下文**|events|

如果启用了 [accept_mutex](#accept_mutex)，表示工作进程将依次接受连接。当另一个工作进程正在接受新连接时，当前工作进程重新尝试接受连接的等待时间。

### daemon

|\-|说明|
|:------|:------|
|**语法**|**daemon** `on` &#124; `off`;|
|**默认**|daemon on;|
|**上下文**|main|

是否将 nginx 启动为守护进程模式。主要用于开发，关闭daemon用于接受断点。

### debug_connection

|\-|说明|
|:------|:------|
|**语法**|**debug_connection** `address` &#124; `CIDR` &#124; `unix:`;|
|**默认**|——|
|**上下文**|events|

启用所选客户端连接的调试日志。其他连接将使用由 [error_log](#error_log) 指令设置的日志级别。调试连接由 IPv4 或 IPv6 （1.3.0，1.2.1）地址或网络指定。也可以使用主机名指定连接。对于使用 UNIX 域套接字（1.3.0，1.2.1）的连接，调试日志由 `unix:` 参数启用。

```nginx
events {
    debug_connection 127.0.0.1;
    debug_connection localhost;
    debug_connection 192.0.2.0/24;
    debug_connection ::1;
    debug_connection 2001:0db8::/32;
    debug_connection unix:;
    ...
}
```

> 为了使该指令正常工作，需要使用 `--with-debug` 来构建nginx，请参见[调试日志](../介绍/调试日志.md)。

### debug_points

|\-|说明|
|:------|:------|
|**语法**|**debug_points** `abort` &#124; `stop`;|
|**默认**|——|
|**上下文**|main|

该指令用于调试。

当检测到内部错误时，例如，在重新启动工作流程时，发生套接字泄漏，启用 `debug_points` 会导致核心创建文件（`abort`）或停止进程（`stop`）以使用系统调试器进行进一步分析。


### env

|\-|说明|
|:------|:------|
|**语法**|**env** `variable[=value]`;|
|**默认**|env TZ;|
|**上下文**|main|

默认情况下，nginx 除去 TZ 变量之外的所有继承自父进程的环境变量。该指令允许保留一些继承的变量、更改其值或创建新的环境变量。 这些变量是：

- 在可执行文件[实时升级](../介绍/控制nginx.md#upgrade)期间继承
- 被 [ngx_http_perl_module](http://nginx.org/en/docs/http/ngx_http_perl_module.html) 模块使用
- 被工作进程使用。应该要记住，以这种方式来控制系统库并不总是可行的，因为只有在初始化期间这些库才能检查变量。一种例外情况是可执行文件执行[实时升级](../介绍/控制nginx.md#upgrade)。

除非明确配置，否则 TZ 变量始终会继承并对 [ngx_http_perl_module](http://nginx.org/en/docs/http/ngx_http_perl_module.html) 模块可用。

用法示例：

```nginx
env MALLOC_OPTIONS;
env PERL5LIB=/data/site/modules;
env OPENSSL_ALLOW_PROXY_CERTS=1;
```

> NGINX 环境变量由 nginx 内部使用，不应由用户直接设置。


### error_log

|\-|说明|
|------:|------|
|**语法**|**error_log** `file [level]`;|
|**默认**|error_log logs/error.log error;|
|**上下文**|main、http、mail、stream、server、location|

配置日志。可以在同一级别指定几个日志（1.5.2）。如果在 `main` 配置级别将日志写入一个未明确定义文件，则将使用默认文件。

第一个参数定义了一个将存储日志的 `file`。特殊值 `stderr` 选择标准错误文件。可以通过指定 `syslog:` 前缀来配置记录日志到 [syslog](../介绍/记录日志到syslog.md)。可以通过指定 `memory:` 前缀和缓冲区 `size` 来配置记录日志到[循环内存缓冲区](../介绍/记录日志到syslog.md)，此通常用于调试（1.7.11）。

第二个参数确定日志记录的 `level`，可以是以下之一：`debug`、`info`、`notice`、`warn`、`error`、`crit`、`alert` 或`emerg`。上述日志级别按严重性递增排列。设置某个日志级别将造成所有指定的消息和更严重的日志级别被记录。例如，默认级别 `error` 将导致 `error`、`crit`、`alert`和 `emerg` 消息被记录。如果省略此参数，则使用 `error`。

> 为了使 `debug` 日志记录工作，需要使用 `--with-debug` 来构建nginx，请参见[调试日志](../介绍/调试日志.md)。

> 从 1.7.11 版本开始，该指令可以指定 `stream` 级别，从 1.9.0 版本开始可以指定 `mail` 级别。

日志级别定义：
```
emerg 紧急 - 系统无法使用。 
alert 必须立即采取措施。 
crit 致命情况。 
error 错误情况。 
warn 警告情况。  
notice 一般重要情况。
info 普通信息。 
debug 调试级别信息
```

### events

|\-|说明|
|:------|:------|
|**语法**|**events** `{ ... }`|
|**默认**|——|
|**上下文**|main|

提供配置文件上下文，用于指定影响连接处理的指令。

### include

|\-|说明|
|:------|:------|
|**语法**|**include** `file` &#124; `mask`;|
|**默认**|——|
|**上下文**|any|

包含另一个 `file` 或匹配指定 `mask` 的文件到配置中。包含的文件应该由语法正确的指令和块组成。

使用示例：

```nginx
include mime.types;
include vhosts/*.conf;
```
### load_module

|\-|说明|
|:------|:------|
|**语法**|**load_module** `file`;|
|**默认**|——|
|**上下文**|main|
|**提示**|该指令在 1.9.11 版本中出现|

加载一个动态模块

使用示例：

```nginx
load_module modules/ngx_mail_module.so;
```

|\-|说明|
|:------|:------|
|**语法**|**lock_file** `file`;|
|**默认**|lock_file logs/nginx.lock;|
|**上下文**|main|

nginx 使用锁机制来实现 [accept_mutex](#accept_mutex) 并序列化对共享内存的访问。大多数系统是使用原子操作实现锁，并忽略此指令。在其他系统上是使用“锁文件”机制。此指令用于指定一个锁文件名称的前缀。

### master_process

|\-|说明|
|:------|:------|
|**语法**|**master_process** `on` &#124; `off`;|
|**默认**|master_process on;|
|**上下文**|main|

用于决定是否仅启动工作进程，而不启动主进程。该指令适用于 nginx 开发人员，方便单进程模式下debug程序。

### multi_accept

|\-|说明|
|:------|:------|
|**语法**|**multi_accept** `on` &#124; `off`;|
|**默认**|multi_accept off;|
|**上下文**|events|

如果 `multi_accept` 被禁用，工作进程将一次接受一个新连接。否则，工作进程将一次接受所有新连接。

> 如果使用了 [kqueue](http://nginx.org/en/docs/events.html#kqueue) 连接处理方式，则会忽略该指令，因为它报告了等待接受的新连接的数量。

### pcre_jit

|\-|说明|
|:------|:------|
|**语法**|**pcre_jit** `on` &#124; `off`;|
|**默认**|pcre_jit off;|
|**上下文**|main|
|**提示**|该指令在 1.1.12 版本中出现|

启用或禁用对在配置解析时已知的正则表达式使用“即时编译”（PCRE JIT）。

PCRE JIT 可以明显地加快正则表达式的处理。

> 从 8.20 版本开始，可以使用 `-enable-jit` 配置参数构建 PCRE 库启用 JIT。当使用 nginx（`--with-pcre=`）构建 PCRE 库时，可以通过 `--with-pcre-jit` 配置参数启用 JIT 支持。

### pid

|\-|说明|
|:------|:------|
|**语法**|**pid** `file`;|
|**默认**|——|
|**上下文**|main|

定义一个 `file` 用于存储主进程的进程 ID 。

### ssl_engine

|\-|说明|
|:------|:------|
|**语法**|**ssl_engine** `device`;|
|**默认**|——|
|**上下文**|main|

定义硬件 SSL 加速器的名称。

### thread_pool

|\-|说明|
|:------|:------|
|**语法**|**thread_pool** `name threads=number [max_queue=number]`;|
|**默认**|thread_pool default threads=32 max_queue=65536;|
|**上下文**|main|
|**提示**|该指令在 1.7.11 版本中出现|

定义线程池的名称和参数，用于多线程方式读取和发送文件，而[无需阻塞](#aio)工作进程。

`threads` 参数定义池中的线程数量。

如果池中的所有线程都处于繁忙状态，则新任务将在队列中等待。`max_queue` 参数限制队列中允许等待的任务数。缺省情况下，队列中最多可以等待 65536 个任务。当队列溢出时，任务直接完成并报错。

### timer_resolution

|\-|说明|
|:------|:------|
|**语法**|**timer_resolution** `interval`;|
|**默认**|——|
|**上下文**|main|

减少工作进程中的计时器分辨率（允许一定误差），从而减少 `gettimeofday()` 的系统调用次数。默认情况下，每次接收到内核事件时都会调用 `gettimeofday()`。随着分辨率的降低，`gettimeofday()` 仅在指定的时间间隔内被调用一次。

示例：

```nginx
timer_resolution 100ms;
```

间隔的内部实现取决于使用的方法：

- 如果使用 `kqueue`，就使用 `EVFILT_TIMER` 过滤器
- 如果使用 `eventport`，就使用 `timer_create()`
- 否则使用 `setitimer()`

### use

|\-|说明|
|:------|:------|
|**语法**|**use** `method`;|
|**默认**|——|
|**上下文**|events|

指定要使用的连接处理[方式](../介绍/连接处理方式.md)。通常不需要明确指定它，因为 nginx 将默认使用最有效的方式。

### user

|\-|说明|
|:------|:------|
|**语法**|**user** `user [group]`;|
|**默认**|——|
|**上下文**|events|

定义工作进程使用的 用户（`user`） 和 用户组（`group`） 。如果省略 `group`，则使用其名称与 `user` 相等的组。

### worker_aio_requests

|\-|说明|
|:------|:------|
|**语法**|**worker_aio_requests** `number`;|
|**默认**|worker_aio_requests 32;|
|**上下文**|events|
|**提示**|该指令在 1.1.4 和 1.0.7 版本中出现|

使用 [epoll](#epoll) 连接处理方式的 [aio](#aio) 时，为单个工作进程设置最大未完成异步 I/O 操作`number`（数量）。

### worker_connections

|\-|说明|
|:------|:------|
|**语法**|**worker_connections** `number`;|
|**默认**|worker_connections 512;|
|**上下文**|events|

设置工作进程可以打开的同时连接的最大数量。

应该记住的是，这个数字包括了所有连接（例如与代理服务器的连接等），而不仅仅是客户端的连接。另一个要考虑因素是实际的并发连接数不能超过最大打开文件数的限制，可以通过 [worker_rlimit_nofile](#worker_rlimit_nofile) 来修改。

### worker_cpu_affinity

|\-|说明|
|:------|:------|
|**语法**|**worker_cpu_affinity** `cpumask ...`; <br /> **worker_cpu_affinity** `auto [cpumask]`;|
|**默认**|——|
|**上下文**|main|

CPU亲和性配置。将工作进程绑定到一组 CPU。每个 CPU 组由允许的 CPU 的位掩码表示。应为每个工作进程定义一个单独的组。默认情况下，工作进程不绑定到任何特定的 CPU。

示例：

```nginx
worker_processes    4;
worker_cpu_affinity 0001 0010 0100 1000;
```

将每个工作进程绑定到单独的 CPU

```nginx
worker_processes    2;
worker_cpu_affinity 0101 1010;
```

将第一个工作进程绑定到 CPU0/CPU2，将第二个工作进程绑定到 CPU1/CPU3。第二个例子适用于超线程。

特殊值 `auto`（1.9.10）允许自动绑定工作进程到可用的 CPU：

```nginx
worker_processes auto;
worker_cpu_affinity auto;
```

可选的掩码参数可用于限制自动绑定的可用 CPU：

```nginx
worker_cpu_affinity auto 01010101;
```

> 该指令仅适用于 FreeBSD 和 Linux。

### worker_priority

|\-|说明|
|:------|:------|
|**语法**|**worker_priority** `number`;|
|**默认**|worker_priority 0;|
|**上下文**|main|

定义工作进程的调度优先级，与使用 `nice` 命令完成类似：负数意味着更高的优先级。允许范围通常在 -20 到 20 之间。

示例：

```nginx
worker_priority -10;
```
### worker_processes

|\-|说明|
|:------|:------|
|**语法**|**worker_processes** `number` &#124; `auto`;|
|**默认**|worker_processes 1;|
|**上下文**|main|

定义工作进程的数量。

最优值取决于许多因素，包括（但不限于）CPU 核心数、存储数据的硬盘驱动器数量以及加载模式。当您不确定时，将其设置为可用 CPU 核心的数量是一个不错的做法（值 `auto` 将尝试自动检测）。

> `auto` 参数从 1.3.8 和 1.2.5 版本开始得到支持。

### worker_rlimit_core

|\-|说明|
|:------|:------|
|**语法**|**worker_rlimit_core** `size`;|
|**默认**|——|
|**上下文**|main|

更改工作进程核心文件（`RLIMIT_CORE`）最大大小限制。用于增加限制而无需重新启动主进程。

### worker_rlimit_nofile

|\-|说明|
|:------|:------|
|**语法**|**worker_rlimit_nofile** `number`;|
|**默认**|——|
|**上下文**|main|

更改工作进程最大打开文件数（`RLIMIT_NOFILE`）的限制。用于增加限制而无需重新启动主进程。

### worker_shutdown_timeout

|\-|说明|
|:------|:------|
|**语法**|**worker_shutdown_timeout** `time`;|
|**默认**|——|
|**上下文**|main|
|**提示**|该指令在 1.11.11 版本中出现|

优雅关闭工作进程时的超时等待时间。当等待超时时，nginx 将尝试关闭当前打开的所有连接以便于关闭工作进程。

### working_directory

|\-|说明|
|:------|:------|
|**语法**|**working_directory** `directory`;|
|**默认**|——|
|**上下文**|main|

指定工作进程的工作目录。主要用于写核心文件，工作进程应对指定的目录有写入权限。

## 原文档

[http://nginx.org/en/docs/ngx_core_module.html](http://nginx.org/en/docs/ngx_core_module.html)
