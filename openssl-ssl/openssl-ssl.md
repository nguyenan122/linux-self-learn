### OpenSSL
```console
Step 1: Create PrivateKey: 
# openssl genrsa       -out ca.key 2048
# openssl genrsa -des3 -out ca.key 2048  (create key with pass, can increase change 2048 -> 4096)
 
Step 2: Create CSR 
 [  openssl req -new -key ca.key -out ca.csr  (SHA1) 
    openssl req -new -sha256 -key ca.key -out ca.csr (SHA256)   ] ---------------- check csr information :    openssl req -in ca.csr  -noout -text

Country Name (2 letter code) [GB]:XX
State or Province Name (full name) [Berkshire]:XXX
Locality Name (eg, city) [Newbury]:XXX
Organization Name (eg, company) [My Company Ltd]:XXX
Organizational Unit Name (eg, section) []:IT Department
Common Name (eg, your name or your server's hostname) []: helloworld.com
Email Address []:info@xxxxxxxxxxx.com

Step 3: Create Self CRT
openssl x509 -req -days 3650 -in ca.csr -signkey ca.key -out ca.crt   

#check detail certificate file:
openssl x509 -in ca.crt -text -noout
```