
```
mkdir /Users/xxradar/certs
```
```
kubectl create ns mitmproxy
```
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: mitmproxy
  namespace: mitmproxy
  labels:
    proxy: mitmproxy
spec:
  containers:
  - name: mitmweb
    image: mitmproxy/mitmproxy
    command: ["mitmweb"]
    args: ["--web-host","0.0.0.0"]
    volumeMounts:
    - name: mitmproxymount
      mountPath: /root/.mitmproxy
    env:
    - name: DEMO_GREETING
      value: "Hello from the environment"
    - name: DEMO_FAREWELL
      value: "Such a sweet sorrow"
  volumes:
  - name: mitmproxymount
    hostPath: 
      path: /Users/xxradar/certs
EOF
```
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: mitmproxy-svc
  namespace: mitmproxy
spec:
  selector:
    proxy: mitmproxy
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
      name: mitmproxy
    - protocol: TCP
      port: 8081
      targetPort: 8081
      name: mitmweb
  type: NodePort
EOF
```
Check if the certs are available
```
ls -la /Users/xxradar/certs
```


## Create a secret 
```
kubectl create secret generic mitmproxysecret -n mitmproxy  --from-file=/Users/xxradar/certs/mitmproxy-ca.pem
```

## Mounting the secret in a deployment
```

```
