

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
yum install perl gcc patch wget -y

wget https://github.com/PCRE2Project/pcre2/releases/download/pcre2-10.45/pcre2-10.45.tar.gz
tar -zxf pcre2-10.45.tar.gz


wget https://zlib.net/fossils/zlib-1.3.1.tar.gz
tar -zxf zlib-1.3.1.tar.gz

wget http://www.openssl.org/source/openssl-1.1.1w.tar.gz
tar -zxf openssl-1.1.1w.tar.gz
#or https://github.com/openssl/openssl/releases/download/OpenSSL_1_1_1w/openssl-1.1.1w.tar.gz


wget https://nginx.org/download/nginx-1.27.4.tar.gz
tar -zxf nginx-1.27.4.tar.gz

#Other module if NEED
wget https://github.com/chobits/ngx_http_proxy_connect_module/archive/refs/tags/v0.0.7.tar.gz
tar -xvzf v0.0.7.tar.gz
```
Compile Nginx with ngx_http_proxy_connect_module-0.0.7
```console
cd nginx-1.27.4
patch -p1 < ../ngx_http_proxy_connect_module-0.0.7/patch/proxy_connect_rewrite_102101.patch
./configure --prefix=/etc/nginx --with-pcre=../pcre2-10.45 --with-zlib=../zlib-1.3.1 --with-http_ssl_module --with-stream --add-dynamic-module=../ngx_http_proxy_connect_module-0.0.7 --with-openssl=../openssl-1.1.1w  --with-http_realip_module --with-http_stub_status_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module

make
make install
```
Setting proxy_connect
```console
worker_processes  auto;
load_module /etc/nginx/modules/ngx_http_proxy_connect_module.so;
.....
server {
    listen                         3128;

    # dns resolver used by forward proxying
    resolver                       8.8.8.8;

    # forward proxy for CONNECT requests
    proxy_connect;
    proxy_connect_allow            443 8080;
    proxy_connect_connect_timeout  10s;
    proxy_connect_data_timeout     10s;

    # defined by yourself for non-CONNECT requests
    # Example: reverse proxy for non-CONNECT requests
    location / {
        proxy_pass http://$host;
        proxy_set_header Host $host;
    }
}
```
How to test it?
```console
curl https://dantri.com.vn/ -v -x 127.0.0.1:3128
```

### 3. Forward-Proxy as stream tcp
https://www.alibabacloud.com/blog/how-to-use-nginx-as-an-https-forward-proxy-server_595799

```console
stream {
    resolver 8.8.8.8;
    server {
        listen 3129;
        ssl_preread on;
        proxy_connect_timeout 5s;
        proxy_pass $ssl_preread_server_name:$server_port;
    }
}
```
How to test it?
```console
curl -v --resolve dantri.com.vn:3129:127.0.0.1 https://dantri.com.vn
```