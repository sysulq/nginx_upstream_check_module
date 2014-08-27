Introduction
===========================

This module can be used in pure nginx-1.4.7 to check upstream servers, with several patches applied.
This module's main purpose is to add proactive health check for the upstream servers.
The core source file comes from Tengine, which is developed and maintained by Alibaba.

For more details, just check [this website](http://tengine.taobao.org/document/http_upstream_check.html).

Synopsis
===========================
```
http {
    upstream cluster1 {
        # simple round-robin
        server 192.168.0.1:80;
        server 192.168.0.2:80;

        check interval=3000 rise=2 fall=5 timeout=1000 type=http;
        check_http_send "HEAD / HTTP/1.0\r\n\r\n";
        check_http_expect_alive http_2xx http_3xx;
    }

    upstream cluster2 {
        # simple round-robin
        server 192.168.0.3:80;
        server 192.168.0.4:80;

        check interval=3000 rise=2 fall=5 timeout=1000 type=http;
        check_keepalive_requests 100;
        check_http_send "HEAD / HTTP/1.1\r\nConnection: keep-alive\r\n\r\n";
        check_http_expect_alive http_2xx http_3xx;
    }

    server {
        listen 80;

        location /1 {
            proxy_pass http://cluster1;
        }

        location /2 {
            proxy_pass http://cluster2;
        }

        location /status {
            check_status;

            access_log   off;
            allow SOME.IP.ADD.RESS;
            deny all;
        }
    }
}
```

Installation
===========================

You can patch this patch if you only use the original upstream load balance algrithm.

```
patch -p1 < /path/to/nginx-1.4.7.patch
```

If you need nginx_upstream_fair and nginx_upstream_hash, just apply these two patches.

```
patch -p1 < /path/to/nginx_upstream_fair.patch
patch -p1 < /path/to/nginx_upstream_hash.patch
```

After all patched have been applied, just try to config and make.
```
./configure --add-module=/path/to/nginx_upstream_check_module
            --add-module=/path/to/nginx-upstream-fair
            --add-module=/path/to/nginx_upstream_hash
            
make
make install
```
