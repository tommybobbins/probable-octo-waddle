# probable-octo-waddle
###  Install kind 
https://kind.sigs.k8s.io/
```
$ sudo kind create cluster --image=kindest/node:v1.28.0@sha256:b7a4cad12c197af3ba43202d3efe03246b3f0793f162afb40a33c923952d5b31
enabling experimental podman provider
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.28.0) 🖼 
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Thanks for using kind! 😊

$ sudo cat /root/.kube/config  >> ~/.kube/config 

```
### Installing nginx gateway fabric
```
$ kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v0.8.1/standard-install.yaml
$ kubectl apply -f https://github.com/nginxinc/nginx-gateway-fabric/releases/download/v1.0.0/crds.yaml
$ kubectl apply -f https://github.com/nginxinc/nginx-gateway-fabric/releases/download/v1.0.0/nginx-gateway.yaml
$ kubectl get pods -n nginx-gateway
$ kubectl apply -f cafe-example/cafe.yaml
$ kubectl apply -f cafe-example/gateway.yaml
$ kubectl -n nginx-gateway port-forward $(kubectl get pods -n nginx-gateway -o name | cut -f2 -d '/') 8080:80 8443:443
```

### Test
```
$ curl 127.0.0.1:8080/tea -H "Host: cafe.example.com" 
Server address: 10.244.0.14:8080
Server name: tea-9d8868bb4-vbn6q
Date: 06/Nov/2023:14:02:34 +0000
URI: /tea
Request ID: 50fc369f79d96bd3cb9c66ea8abcbca3
$ curl 127.0.0.1:8080/coffee -H "Host: cafe.example.com" 
Server address: 10.244.0.13:8080
Server name: coffee-6b8b6d6486-svtj6
Date: 06/Nov/2023:14:02:43 +0000
URI: /coffee
Request ID: 470e58dd9f651ebd33a568aa218a2608
```
### Logs
```
$ kubectl logs $(kubectl get pods -n nginx-gateway -o name | cut -f2 -d '/') -n nginx-gateway -c nginx
127.0.0.1 - - [06/Nov/2023:14:02:34 +0000] "GET /tea HTTP/1.1" 200 154 "-" "curl/7.88.1"
2023/11/06 14:02:35 [info] 137#137: *25 client 127.0.0.1 closed keepalive connection
127.0.0.1 - - [06/Nov/2023:14:02:43 +0000] "GET /coffee HTTP/1.1" 200 161 "-" "curl/7.88.1"
2023/11/06 14:02:44 [info] 138#138: *27 client 127.0.0.1 closed keepalive connection
```

#### Deploying Gateway API  HTTPRoute for blue/green traffic split
Taken from https://gateway-api.sigs.k8s.io/guides/traffic-splitting/

Create the deployments and services:
```
$ kubectl create -f cafe-examples/mocha.yaml
```

Create the backend routes with a 90/10 traffic split:
```
$ kubectl create -f cafe-examples/mocha-routes.yaml
```
Test:
It can be seen that 10% of the requests to mocha are sent to the mochawokachocachino pod.
```
$ while true; do curl -s 127.0.0.1:8080/mocha -H "Host: cafe.example.com" |egrep "Server name"; sleep 1; done
Server name: mocha-54dd499f6d-w742f
Server name: mocha-54dd499f6d-w742f
Server name: mocha-54dd499f6d-w742f
.....
Server name: mochawokachocachino-798cf8d448-fj4pj
Server name: mocha-54dd499f6d-w742f
Server name: mocha-54dd499f6d-w742f
Server name: mocha-54dd499f6d-w742f
Server name: mocha-54dd499f6d-w742f
```
