
```
yum -y install epel-release
yum -y install certbot-nginx
apt-get install certbot
certbot certonly --manual --preferred-challenges=dns --email xxxx@gmail.com --server https://acme-v02.api.letsencrypt.org/directory --agree-tos -d "*.test.vn" -d "test.vn"
```