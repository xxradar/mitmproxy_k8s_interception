## Overwriting the service account token and api CA verification
```
kubectl create ns dev
```
### Create a SA
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: demo-sa
  namespace: dev
EOF
```
### Create a ClusterRole
```
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: my-demo-clusterrole
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["get", "watch", "list"]
EOF
```
### Create clusterrolebindig
```
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: demo-rolebinding-global
subjects:
- kind: ServiceAccount
  name: demo-sa
  namespace: dev
roleRef:
  kind: ClusterRole
  name: my-demo-clusterrole
  apiGroup: rbac.authorization.k8s.io
EOF
```
### Let me test this 
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: dev
spec:
  serviceAccountName: demo-sa
  containers:
  - name: mitm-demo
    image: xxradar/hackon
    command:
    - sleep
    args:
    - 5000s
EOF
```
```
kubectl exec -it -n dev my-pod -- bash
```
```
KUBE_TOKEN=$(</var/run/secrets/kubernetes.io/serviceaccount/token)
KUBE_CA="/var/run/secrets/kubernetes.io/serviceaccount/ca.crt"
echo $KUBE_TOKEN
echo $KUBE_CA

curl -kv   https://kubernetes.default/api/v1/
curl -kv   https://kubernetes.default/version
curl --cacert $KUBE_CA  https://kubernetes.default/api/v1/
curl --cacert $KUBE_CA -H "Authorization: Bearer $KUBE_TOKEN"  https://kubernetes.default/api/v1/
curl --cacert $KUBE_CA -H "Authorization: Bearer $KUBE_TOKEN"  https://kubernetes.default/apis/
curl --cacert $KUBE_CA -H "Authorization: Bearer $KUBE_TOKEN"  https://kubernetes.default/api/v1/namespaces/dev/pods/
```

### Testing token removal (read-only)
```
rm /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
rm: cannot remove '/var/run/secrets/kubernetes.io/serviceaccount/ca.crt': Read-only file system
```

### Installing kubectl
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
```
Seems to work ...
```
./kubectl  get po
```
### Create a kubeconfig file
```
cat >mykubeconfig.tpl <<EOF
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: {{CA_DATA_INSERT}}
    server: https://10.1.2.217:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    token: {{TOKEN_INSERT}}
EOF
```

```
export KUBE_TOKEN=$(</var/run/secrets/kubernetes.io/serviceaccount/token)
export KUBE_CA="/var/run/secrets/kubernetes.io/serviceaccount/ca.crt"
export CA_DATA=$(cat $KUBE_CA | base64 -i -w 0)

echo $KUBE_TOKEN
echo $KUBE_CA
echo $CA_DATA
```
```
cat mykubeconfig.tpl  | sed "s/{{CA_DATA_INSERT}}/$CA_DATA/g" | sed "s/{{TOKEN_INSERT}}/$KUBE_TOKEN/g" >./mykubeconfig
```
### Make the kubeconfig supersede the service account
```
export KUBECONFIG=./mykubeconfig
```
or
```
cp ./mykubeconfig ~/.kube/config
```
### Give it a try
Seems to work ... (change something in the kubeconfig to see if throws an error to verify ;-)
```
./kubectl  get po
```
## Getting mitmproxy in the loop 
### Get the mitmproxy ca certificate
```
cat ca_mitm.crt
```
```
-----BEGIN CERTIFICATE-----
MIIDNTCCAh2gAwIBAgIUVJ2XVAwjjZG/qZ4yORGZqXcvq1swDQYJKoZIhvcNAQEL
BQAwKDESMBAGA1UEAwwJbWl0bXByb3h5MRIwEAYDVQQKDAltaXRtcHJveHkwHhcN
MjIxMDAxMTkzOTIxWhcNMzIwOTMwMTkzOTIxWjAoMRIwEAYDVQQDDAltaXRtcHJv
eHkxEjAQBgNVBAoMCW1pdG1wcm94eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCC
AQoCggEBAML757zwjBwU2g6xvwogaraK3Hz3IjHAgzUCkPse0+ujknJbVWUeGIfR
zF2TSCn+13VX/W2+P3jmU2zzMfshpg2P5kk6ZNXyA0C68SPuB1WqamP/zrzoM7wY
lHukXa6bhbI4ZsYgYKkhc7pgxRkQ4wwJpZ9z1OSQGXNEhh0midoCw4lX6/RLcOaz
iRIZ/9q86Cxsz7f6nQsZIZd0PX1zA6ayiLOFdDcF4OyZ9ZrXDI7akkEEVMP5onmo
t/FTsfAMBPhB/A3ZZge1Y5BhPmm9XgkCAwEAAaNXMFUwDwYDVR0TAQH/BAUwAwEB
/zATBgNVHSUEDDAKBggrBgEFBQcDATAOBgNVHQ8BAf8EBAMCAQYwHQYDVR0OBBYE
FFXF0ebnUA/aJfHOFVwCqEBt8u6oMA0GCSqGSIb3DQEBCwUAA4IBAQAtfZkZggqC
UCK8VgsIz5eUM4A0x2NujzkYQ+IRrKeAHJHlsBzxrdL6yp/0eWYhbM0So/oICaT3
InP3Iz+3Xj9areSc/H6LICbcDoVQNOqebcCnZKn0DBnN4Qts95GGHqfJwedDd6oI
cXjrVnAT/GHlbnriFunitotibs3KmZT2aScY2evnLpELyWnXkNAdeITu16TjzHEc
G1p1srkalfa4IXSh/DGL5CvkDg24ascch0y40Nzmu43h2gcyA4PjCOdJDqE/ZiCs
XililI5Cv+B78y2GSf04QNtcufdI/qzk8KGj16ydtpYYDCSvVfIt4DD3Uo7pJd4X
ULq56XXiNsty
-----END CERTIFICATE-----
```
### Store the certificate in an env variable
```
export CA_DATA=$(cat ca_mitm.crt | base64 -i -w 0)
```
### Update kubeconfig
```
cat mykubeconfig.tpl  | sed "s/{{CA_DATA_INSERT}}/$CA_DATA/g" | sed "s/{{TOKEN_INSERT}}/$KUBE_TOKEN/g" >./mykubeconfig
```
```
cp ./mykubeconfig ~/.kube/config
```
You should now experience a validation error
```
root@my-pod:/# ./kubectl get po
Unable to connect to the server: x509: certificate signed by unknown authority
root@my-pod:/#
```
### Redirect to mitmproxy
```
export https_proxy="http://mitmproxy-svc.mitmproxy:8080/"
export http_proxy="http://mitmproxy-svc.mitmproxy:8080/"
```
```
root@my-pod:/# ./kubectl get po
NAME      READY   STATUS    RESTARTS         AGE
my-pod    1/1     Running   30 (50m ago)     42h
my-pod2   1/1     Running   31 (2m42s ago)   43h
root@my-pod:/#
```
![mitmproxy](./images/mitmproxy2.png)
