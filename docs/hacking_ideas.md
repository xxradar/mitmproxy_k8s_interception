## Updating all pods in namespace 
Note: You cannot update a pod, but you can update a deploypent, rc, ...<br>
https://www.mankier.com/1/kubectl-set-env

```
kubectl set env deploy -n hacking --all "https_proxy=http://mitmproxy-svc.mitmproxy:8080/"
```
```
kubectl set env deploy -n hacking --all "http_proxy=http://mitmproxy-svc.mitmproxy:8080/" "https_proxy=http://mitmproxy-svc.mitmproxy:8080/"
```
Note: pods will be restarted at this point, but will have the env variables. <br>


## Sometimes you miss the binaries (and you can't reboot the pod :))
```
POD=radarhack-deployment-6899c8bc7c-h2lx4
kubectl cp mitmproxy-ca.pem $POD:/tmp/mitmproxy-ca.crt
kubectl exec $POD -- mkdir /usr/local/share/ca-certificates/
kubectl exec $POD -- cp /tmp/mitmproxy-ca.crt /usr/local/share/ca-certificates/
```
## When the vars are already set 
##      .... you can temporaly disable (it's only in the context of the shell)
```
kubectl exec  $POD -- bash -c "unset http_proxy; unset https_proxy; apt-get update; apt-get install -y ca-certificates curl "
```
```
kubectl exec $POD -- sh -c "/usr/sbin/update-ca-certificates --fresh"
```
```
kubectl exec $POD -- sh -c "curl https://www.radarhack.com"
```
