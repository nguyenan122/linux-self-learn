# mTLS

Mutual TLS (mTLS) ensures that both the client and server authenticate each other using certificates, significantly enhancing the security of communications.

### 1.1 Generate the CA Certificate and Key
```console
openssl genpkey -algorithm RSA -out /etc/nginx/cert/ca.key
openssl req -days 3650 -new -x509 -key /etc/nginx/cert/ca.key -out /etc/nginx/cert/ca.crt -subj "/OU=CA-OrgUnit/CN=CA-MASTER"
openssl x509 -text -noout -in /etc/nginx/cert/ca.crt
```


### 1.2 Generate the Server Certificate and Key
```console
openssl genpkey -algorithm RSA -out /etc/nginx/cert/server.key
openssl req -new -key /etc/nginx/cert/server.key -out /etc/nginx/cert/server.csr -subj "/OU=MyOrgUnit/CN=localhost"
openssl x509 -req -days 3650 -in /etc/nginx/cert/server.csr -CA /etc/nginx/cert/ca.crt -CAkey /etc/nginx/cert/ca.key -CAcreateserial -out /etc/nginx/cert/server.crt
openssl x509 -text -noout -in /etc/nginx/cert/server.crt
```

### 1.3 Generate the Client Certificate and Key
```console
openssl genpkey -algorithm RSA -out /etc/nginx/cert/client.key
openssl req -new -key /etc/nginx/cert/client.key -out /etc/nginx/cert/client.csr -subj "/OU=MyOrgUnit/CN=localhost"
openssl x509 -req -days 3650 -in /etc/nginx/cert/client.csr -CA /etc/nginx/cert/ca.crt -CAkey /etc/nginx/cert/ca.key -CAcreateserial -out /etc/nginx/cert/client.crt
openssl x509 -text -noout -in /etc/nginx/cert/client.crt
```

### 2 Configure Nginx for SSL/TLS and mTLS
```console
 server {
 listen 443 ssl;
    # Server certificate and key
     ssl_certificate /etc/nginx/cert/server.crt;
     ssl_certificate_key /etc/nginx/cert/server.key;

    # CA certificate for client verification
     ssl_client_certificate /etc/nginx/cert/ca.crt;
     ssl_verify_client optional;

          location / {
                  default_type text/plain;
                  if ($ssl_client_verify != SUCCESS) {
                      return 403 'blocked access to mTLS-protected resource';
                  }
                  return 200 'access to mTLS-protected resource';
          }

}

```
Reload nginx apply new config  
`nginx -s reload`


### 3: Testing mTLS with cURL
```console
curl -vk https://localhost
> blocked access to mTLS-protected resource
```


```console
curl --cert /etc/nginx/cert/client.crt --key /etc/nginx/cert/client.key --cacert /etc/nginx/cert/ca.crt https://localhost
> access to mTLS-protected resource
```
```console
openssl s_client -connect localhost:443 -cert /etc/nginx/cert/client.crt -key /etc/nginx/cert/client.key -CAfile /etc/nginx/cert/ca.crt
```





---
Refer: [https:/medium.com/@mahernaija/new-2025-how-to-configure-mutual-tls-mtls-for-secure-nginx-206f983ba571](https:/medium.com/@mahernaija/new-2025-how-to-configure-mutual-tls-mtls-for-secure-nginx-206f983ba571)
