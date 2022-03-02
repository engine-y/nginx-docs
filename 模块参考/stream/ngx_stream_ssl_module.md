# ngx_stream_ssl_module

- [示例配置](#example_configuration)
- [指令](#directives)
    - ssl_alpn
      ssl_certificate
      ssl_certificate_key
      ssl_ciphers
      ssl_client_certificate
      ssl_conf_command
      ssl_crl
      ssl_dhparam
      ssl_ecdh_curve
      ssl_handshake_timeout
      ssl_password_file
      ssl_prefer_server_ciphers
      ssl_protocols
      ssl_session_cache
      ssl_session_ticket_key
      ssl_session_tickets
      ssl_session_timeout
      ssl_trusted_certificate
      ssl_verify_client
      ssl_verify_depth

该`ngx_stream_ssl_module`模块 (1.9.0) 为流代理服务器与 `SSL/TLS` 协议一起工作提供了必要的支持。默认情况下不构建此模块，应使用 --with-stream_ssl_module 配置参数启用它。

<a id="example_configuration"></a>

## 示例配置

为减少 CPU 的负载，建议

- 设置 worker 进程数等于 CPU 核心数
- 启用[共享](ngx_stream_ssl_module.md#ssl_session_cache_shared)会话缓存
- 禁用[内置](ngx_stream_ssl_module.md#ssl_session_cache_builtin)会话缓存
- 可增长会话[生命周期](ngx_stream_ssl_module.md#ssl_session_timeout)（默认为 5 分钟）


```nginx
worker_processes auto;

stream {

    ...

    server {
        listen              12345 ssl;

        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers         AES128-SHA:AES256-SHA:RC4-SHA:DES-CBC3-SHA:RC4-MD5;
        ssl_certificate     /usr/local/nginx/conf/cert.pem;
        ssl_certificate_key /usr/local/nginx/conf/cert.key;
        ssl_session_cache   shared:SSL:10m;
        ssl_session_timeout 10m;

        ...
    }
```

按顺序检查规则，直到找到第一个匹配项。在此示例中，仅允许 IPv4 网络 `10.1.1.0/16` 和 `192.168.1.0/24`（不包括地址 `192.168.1.1`）和 IPv6 网络`2001:0db8::/32` 进行访问。

<a id="directives"></a>

## 指令

### ssl_alpn

|\-|说明|
|:------|:------|
|**语法**|**ssl_alpn** `protocol ...`;|
|**默认**|—|
|**上下文**|stream、server|
|**提示**|该指令在 1.21.4 版本中出现|

指定支持的ALPN协议列表，如果客户端使用 [ALPN](https://datatracker.ietf.org/doc/html/rfc7301) ，则必须 [协商其中一种协议](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html#var_ssl_alpn_protocol) ：

```nginx
map $ssl_alpn_protocol $proxy {
    h2                127.0.0.1:8001;
    http/1.1          127.0.0.1:8002;
}

server {
    listen      12346;
    proxy_pass  $proxy;
    ssl_alpn    h2 http/1.1;
}
```

## 原文档
[http://nginx.org/en/docs/stream/ngx_stream_ssl_module.html](http://nginx.org/en/docs/stream/ngx_stream_ssl_module.html)
