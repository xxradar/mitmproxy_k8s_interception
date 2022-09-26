## MITMPROXY intercept from docker container
```
export MITMPROXY_IP=192.168.0.131

docker run -it --rm -e "http_proxy=http://$MITMPROXY_IP:8080/" \
                    -e "https_proxy=http://$MITMPROXY_IP:8080/" \
                    xxradar/hackon
```

```
curl -kv http://www.radarhack.com
...
curl -kv https://www.radarhack.com
...
```


## Solving the untrusted certificate manually
```
docker run -it --rm -e "http_proxy=http://$MITMPROXY_IP:8080/" \
                    -e "https_proxy=http://$MITMPROXY_IP:8080/" \
                    -v ~/.mitmproxy/mitmproxy-ca-cert.pem:/usr/local/share/ca-certificates/mitmproxy-ca-cert.crt \
                    xxradar/hackon
```
```
update-ca-certificates
```
```
curl https://www.radarhack.com
```
## Solving the untrusted certificate gracefully
Create this Dockerfile
```
FROM xxradar/hackon
ADD mitmproxy-ca-cert.pem  /usr/local/share/ca-certificates/mitmproxy-ca-cert.crt
RUN update-ca-certificates
CMD bash
```
```
docker build -t demo_container .
```
```
docker run -it --rm -e "http_proxy=http://$MITMPROXY_IP:8080/" \
                    -e "https_proxy=http://$MITMPROXY_IP:8080/" \
                    demo_container
```
```
curl https://www.radarhack.com
...
```
