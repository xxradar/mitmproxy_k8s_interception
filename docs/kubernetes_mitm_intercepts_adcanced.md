
# WIP
## Deploy mitmproxy in kubernetes
**Need cleanup for the storage of the mitmproxy cert
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

## Copy the `mitmproxy-ca.pem` from the mitmproxy pod
```
kubectl cp mitmproxy/mitmproxy:/root/.mitmproxy/mitmproxy-ca.pem  ./mitmproxy-ca.pem
```

## Create a secret 
```
kubectl create secret generic mitmproxysecret  --from-file=mitmproxy-ca.pem
```

## Mounting the secret in a deployment
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: kubernetes-demo
spec:
  containers:
  - args:
    - 5000s
    command:
    - sleep
    image: xxradar/hackon
    lifecycle:
      postStart:
        exec:
          command:
          - bash
          - -c
          - cp /certs/mitmproxy-ca.pem /usr/local/share/ca-certificates/mitmproxy-ca.crt ; update-ca-certificates --fresh
    name: kubernetes-demo
    env:
    - name: http_proxy
      value: "http://mitmproxy-svc.mitmproxy:8080/"
    - name: https_proxy
      value: "http://mitmproxy-svc.mitmproxy:8080/"
    volumeMounts:
    - mountPath: /certs
      name: mitmproxysecret
      readOnly: true
  volumes:
  - name: mitmproxysecret
    secret:
      secretName: mitmproxysecret
EOF
```