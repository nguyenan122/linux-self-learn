

### 1. Reverse proxy dynamic
```console
server {
    listen       8888;
    location / {
        #proxy_pass http://$http_host$uri$is_args$args;    #same with http://$host$request_uri
        resolver 8.8.8.8; #must setting option for resolve domain
        proxy_pass $scheme://$host$request_uri;
    }
}

server {
    listen       8889 ssl;
    ssl_certificate      /etc/nginx/conf/cert/ca.crt;
    ssl_certificate_key  /etc/nginx/conf/cert/ca.key;
    location / {
        resolver 8.8.8.8; #must setting option for resolve domain
        proxy_pass $scheme://$host$request_uri;
    }
}

```
How to test it?
```console
curl -vk -H 'Host: dantri.com.vn' http://localhost:8888
<html>
<center><h1>301 Moved Permanently</h1></center>
</html>
```
```console
curl -vk -H 'Host: dantri.com.vn' https://localhost:8889
 > return web content
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
    proxy_connect_allow            443; #or all
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
How to test it?
```console

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
How to test it?
```console

```