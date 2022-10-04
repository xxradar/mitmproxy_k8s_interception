## Updating all pods in namespace 
Note: You cannot update a pod, but you can update a deploypent, rc, ...<br>
https://www.mankier.com/1/kubectl-set-env

```
kubectl set env deploy -n hacking --all "https_proxy=http://mitmproxy-svc.mitmproxy:8080/"
```
```
kubectl set env deploy -n hacking --all "http_proxy=http://mitmproxy-svc.mitmproxy:8080/" "https_proxy=http://mitmproxy-svc.mitmproxy:8080/"
```
