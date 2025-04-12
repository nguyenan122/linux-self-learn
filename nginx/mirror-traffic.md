


```console
server {
    listen 80;
    server_name _;

    location / {
      mirror /mirror;
      mirror_request_body on;
      proxy_pass http://192.168.88.100:8080;
    }

    location = /mirror {
      internal;
      proxy_pass http://192.168.88.100:8080$request_uri;
    }
}
```

> internal: chỉ cho phép nội bộ nginx gọi. Không cho người dùng ngoài gọi vào và trả về 404.


-----
Refer: https://dev.to/oivoodoo/traffic-mirroring-by-nginx-mirror-module-2i8j