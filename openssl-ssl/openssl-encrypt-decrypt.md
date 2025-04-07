
### File
```console
openssl enc -aes-256-cbc -salt -in file.txt -out file.txt.enc
openssl enc -aes-256-cbc -d -in file.txt.enc -out file_decrypted.txt
```

### Pipeline
```console
echo "Hello, world!" | openssl enc -aes-256-cbc -a -salt -pass pass:123456
echo "Kết_quả_base64" | openssl enc -aes-256-cbc -a -d -pass pass:123456

```