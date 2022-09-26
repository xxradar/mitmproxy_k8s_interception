
docker run --rm -it -v ~/.mitmproxy:/home/mitmproxy/.mitmproxy -p 8080:8080 -p 127.0.0.1:8081:8081 mitmproxy/mitmproxy mitmweb --web-host 0.0.0.0

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
    env:
    - name: DEMO_GREETING
      value: "Hello from the environment"
    - name: DEMO_FAREWELL
      value: "Such a sweet sorrow"                    
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
