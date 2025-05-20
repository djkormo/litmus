https://stackoverflow.com/questions/67231714/how-to-add-trusted-root-ca-to-docker-alpine

Note: When you're using update-ca-certificates, you need to place your cert file into /usr/local/share/ca-certificates/ first. Otherwise it will be removed from /etc/ssl/certs/ca-certificates.crt the first time you run update-ca-certificates.




```console
make docker-build docker-tag-version docker-publish-version
```
