# mitmproxy_install
### Installing via docker
### CLI
```
docker pull mitmproxy/mitmproxy
docker run --rm -it -v ~/.mitmproxy:/home/mitmproxy/.mitmproxy -p 8080:8080 mitmproxy/mitmproxy
```
### WebUI
```
docker run --rm -it -v ~/.mitmproxy:/home/mitmproxy/.mitmproxy -p 8080:8080 -p 127.0.0.1:8081:8081 mitmproxy/mitmproxy mitmweb --web-host 0.0.0.0
```
### Verifying if everything is working fine
```
http_proxy=http://localhost:8080/ curl http://www.radarhack.com/
https_proxy=http://localhost:8080/  curl -kv https://www.radarhack.com/
```
Note: Check the certificate returned by mitmprox. <br>
`*  issuer: CN=mitmproxy; O=mitmproxy `
```
...
* Server certificate:
*  subject: CN=www.radarhack.com
*  start date: Sep 24 09:34:19 2022 GMT
*  expire date: Sep 26 09:34:19 2023 GMT
*  issuer: CN=mitmproxy; O=mitmproxy
*  SSL certificate verify result: unable to get local issuer certificate (20), continuing anyway.
> GET / HTTP/1.1
> Host: www.radarhack.com
> User-Agent: curl/7.64.1
> Accept: */*
>
< HTTP/1.1 200 OK
...
```
