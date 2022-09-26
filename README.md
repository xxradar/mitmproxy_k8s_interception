# mitmproxy_k8s_interception

### Installing via docker
```
docker pull mitmproxy/mitmproxy
docker run --rm -it -v ~/.mitmproxy:/home/mitmproxy/.mitmproxy -p 8080:8080 mitmproxy/mitmproxy
```

### Verifying if everything is wworking fine
```
http_proxy=http://localhost:8080/ curl http://www.radarhack.com/
https_proxy=http://localhost:8080/  curl -kv https://www.radarhack.com/
```
Note: Check the certificate returned by mitmproxy. 
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
