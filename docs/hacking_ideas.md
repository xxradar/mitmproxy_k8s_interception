## Updating all pods in namespace 
Note: You cannot update a pod, but you can update a deploypent, rc, ...<br>
https://www.mankier.com/1/kubectl-set-env

```
kubectl set env deploy -n hacking --all "https_proxy=http://mitmproxy-svc.mitmproxy:8080/"
```
```
kubectl set env deploy -n hacking --all "http_proxy=http://mitmproxy-svc.mitmproxy:8080/" "https_proxy=http://mitmproxy-svc.mitmproxy:8080/"
```



## Sonetimes you miss the binaries


```
kubectl cp mitmproxy-ca.pem   radarhack-deployment-6899c8bc7c-h2lx4:/tmp/mitmproxy-ca.crt
kubectl exec radarhack-deployment-6899c8bc7c-h2lx4 -- mkdir /usr/local/share/ca-certificates/
kubectl exec radarhack-deployment-6899c8bc7c-h2lx4 -- cp /tmp/mitmproxy-ca.crt /usr/local/share/ca-certificates/
kubectl exec radarhack-deployment-6899c8bc7c-h2lx4 -- sh -c "/usr/sbin/update-ca-certificates --fresh"
```
## When the vars are already set .... you can temporaly disable (it's only in the context of the shell
```
kubectl exec  radarhack-deployment-6899c8bc7c-h2lx4 -- bash -c "unset http_proxy; unset https_proxy; apt-get update; apt-get install -y ca-certificates curl "
```


```
kubectl exec radarhack-deployment-6899c8bc7c-h2lx4 -- sh -c "/usr/sbin/update-ca-certificates --fresh"
```

```
kubectl exec radarhack-deployment-6899c8bc7c-h2lx4 -- sh -c "curl https://www.radarhack.com"
```
