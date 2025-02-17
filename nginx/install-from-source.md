

### Install package require:  
```console
yum install perl gcc patch -y

wget https://github.com/PCRE2Project/pcre2/releases/download/pcre2-10.45/pcre2-10.45.tar.gz
tar -zxf pcre2-10.45.tar.gz


wget https://zlib.net/fossils/zlib-1.3.1.tar.gz
tar -zxf zlib-1.3.1.tar.gz

wget http://www.openssl.org/source/openssl-1.1.1w.tar.gz
tar -zxf openssl-1.1.1w.tar.gz
#or https://github.com/openssl/openssl/releases/download/OpenSSL_1_1_1w/openssl-1.1.1w.tar.gz


wget https://nginx.org/download/nginx-1.27.4.tar.gz
tar -zxf nginx-1.27.4.tar.gz
```

```console
cd nginx-1.27.4
./configure --prefix=/etc/nginx --with-pcre=../pcre2-10.45 --with-zlib=../zlib-1.3.1 --with-http_ssl_module --with-stream --with-openssl=../openssl-1.1.1w  --with-http_realip_module --with-http_stub_status_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module

make
make install
```


Create systemd file
```console
cat << EOF > /etc/systemd/system/nginx.service
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/etc/nginx/logs/nginx.pid
ExecStartPre=/etc/nginx/sbin/nginx -t
ExecStart=/etc/nginx/sbin/nginx
ExecReload=/etc/nginx/sbin/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```


Start and enable NGINX service:
```console
systemctl start nginx.service
systemctl enable nginx.service
```

