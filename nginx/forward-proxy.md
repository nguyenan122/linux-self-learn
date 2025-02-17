

### 1. Forward-Proxy in HTTP
```console
server {
    listen       8888;

    location / {
        resolver 8.8.8.8; # may or may not be necessary.
        #proxy_pass http://$http_host$uri$is_args$args;
        proxy_pass http://$http_host$request_uri;
    }
}
```


### 2. Forward-Proxy in HTTPS 
Install ngx_http_proxy_connect_module   
https://github.com/chobits/ngx_http_proxy_connect_module

```console
server {
    listen                         3128;

    # dns resolver used by forward proxying
    resolver                       8.8.8.8;

    # forward proxy for CONNECT requests
    proxy_connect;
    proxy_connect_allow            443;
    proxy_connect_connect_timeout  10s;
    proxy_connect_data_timeout     10s;
    proxy_connect_read_timeout     10s;
    proxy_connect_send_timeout     10s;    

    # defined by yourself for non-CONNECT requests
    location / {
        proxy_pass http://$host;
        proxy_set_header Host $host;
    }
}
```


### 3. Forward-Proxy by Steam-Module
https://www.alibabacloud.com/blog/how-to-use-nginx-as-an-https-forward-proxy-server_595799

```console
stream {
    resolver 8.8.8.8;
    server {
        listen 443;
        ssl_preread on;
        proxy_connect_timeout 5s;
        proxy_pass $ssl_preread_server_name:$server_port;
    }
}
```