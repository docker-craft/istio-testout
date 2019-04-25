### Enable mTLS

```bash

kubectl create ns mesh
kubectl create ns plain

kubectl label ns mesh istio-injection=enabled

kubectl apply -f httpbin.yaml -f sleep.istio.yaml -n mesh

kubectl apply -f httpbin.yaml -f sleep.istio.yaml -n plain

export PLAIN_SLEEP=$(kubectl get pod -n plain -l app=sleep,verison=v1 -o jsonpath={.items..metadata.name})

[root@tpl-centos chapter9]# kubectl apply -f sleep.yaml -f httpbin.yaml -n plain
service/sleep created
deployment.extensions/sleep created
[root@tpl-centos chapter9]# kubectl apply -f sleep.yaml -f httpbin.yaml -n mesh
service/sleep created
deployment.extensions/sleep created

[root@tpl-centos chapter9]# kubectl get pods -n plain
NAME                       READY   STATUS    RESTARTS   AGE
httpbin-54f5bb4957-mlkjz   1/1     Running   0          10m
sleep-5466775485-fvtjk     1/1     Running   0          27s
[root@tpl-centos chapter9]# kubectl get pods -n mesh
NAME                       READY   STATUS    RESTARTS   AGE
httpbin-54f5bb4957-jwrtw   2/2     Running   0          11m
sleep-5466775485-94ncm     2/2     Running   0          24s



[root@tpl-centos chapter9]# kubectl exec -it -n mesh sleep-5466775485-94ncm -c sleep bash
bash-4.4# http http://httpbin.plain:8000/get
HTTP/1.1 200 OK
access-control-allow-credentials: true
access-control-allow-origin: *
content-length: 852
content-type: application/json
date: Wed, 24 Apr 2019 15:25:40 GMT
server: envoy
x-envoy-upstream-service-time: 27

{
    "args": {},
    "headers": {
        "Accept": "*/*",
        "Accept-Encoding": "gzip, deflate",
        "Content-Length": "0",
        "Host": "httpbin.plain:8000",
        "User-Agent": "HTTPie/0.9.9",
        "X-B3-Sampled": "1",
        "X-B3-Spanid": "76a3820047d7cdd4",
        "X-B3-Traceid": "f498feceaf0b86ac76a3820047d7cdd4",
        "X-Envoy-Decorator-Operation": "httpbin.plain.svc.cluster.local:8000/*",
        "X-Istio-Attributes": "CjgKCnNvdXJjZS51aWQSKhIoa3ViZXJuZXRlczovL3NsZWVwLTU0NjY3NzU0ODUtOTRuY20ubWVzaAo9ChhkZXN0aW5hdGlvbi5zZXJ2aWNlLmhvc3QSIRIfaHR0cGJpbi5wbGFpbi5zdmMuY2x1c3Rlci5sb2NhbAo7ChdkZXN0aW5hdGlvbi5zZXJ2aWNlLnVpZBIgEh5pc3RpbzovL3BsYWluL3NlcnZpY2VzL2h0dHBiaW4KJQoYZGVzdGluYXRpb24uc2VydmljZS5uYW1lEgkSB2h0dHBiaW4KKAodZGVzdGluYXRpb24uc2VydmljZS5uYW1lc3BhY2USBxIFcGxhaW4="
    },
    "origin": "172.20.0.76",
    "url": "http://httpbin.plain:8000/get"
}

bash-4.4# exit
exit
[root@tpl-centos chapter9]# kubectl exec -it -n plain sleep-5466775485-fvtjk -c sleep bash
bash-4.4# http http://httpbin.mesh:8000/get
HTTP/1.1 200 OK
access-control-allow-credentials: true
access-control-allow-origin: *
content-length: 387
content-type: application/json
date: Wed, 24 Apr 2019 15:26:50 GMT
server: istio-envoy
x-envoy-decorator-operation: httpbin.mesh.svc.cluster.local:8000/*
x-envoy-upstream-service-time: 13

{
    "args": {},
    "headers": {
        "Accept": "*/*",
        "Accept-Encoding": "gzip, deflate",
        "Content-Length": "0",
        "Host": "httpbin.mesh:8000",
        "User-Agent": "HTTPie/0.9.9",
        "X-B3-Sampled": "1",
        "X-B3-Spanid": "7d15e117ca982fc5",
        "X-B3-Traceid": "c28a0dcc307a6fbe7d15e117ca982fc5"
    },
    "origin": "127.0.0.1",
    "url": "http://httpbin.mesh:8000/get"
}


#apply meshpolicy.yaml

[root@tpl-centos chapter9]# cat meshpolicy.yaml
apiVersion: "authentication.istio.io/v1alpha1"
kind: "MeshPolicy"
metadata:
  name: "default"
spec:
  peers:
    - mtls: {}

[root@tpl-centos chapter9]# kubectl apply -f meshpolicy.yaml
meshpolicy.authentication.istio.io/default configured


[root@tpl-centos chapter9]# kubectl exec -it -n plain sleep-5466775485-fvtjk -c sleep bash
bash-4.4# http http://httpbin.mesh:8000/get

http: error: ConnectionError: ('Connection aborted.', ConnectionResetError(104, 'Connection reset by peer')) while doing GET request to URL: http://httpbin.mesh:8000/get
bash-4.4# http http://httpbin.mesh:8000/get

http: error: ConnectionError: ('Connection aborted.', ConnectionResetError(104, 'Connection reset by peer')) while doing GET request to URL: http://httpbin.mesh:8000/get
bash-4.4# exit
exit
command terminated with exit code 1
[root@tpl-centos chapter9]# kubectl exec -it -n mesh sleep-5466775485-94ncm -c sleep bash bash-4.4# http http://httpbin.plain:8000/get
HTTP/1.1 200 OK
access-control-allow-credentials: true
access-control-allow-origin: *
content-length: 852
content-type: application/json
date: Wed, 24 Apr 2019 15:32:28 GMT
server: envoy
x-envoy-upstream-service-time: 4

{
    "args": {},
    "headers": {
        "Accept": "*/*",
        "Accept-Encoding": "gzip, deflate",
        "Content-Length": "0",
        "Host": "httpbin.plain:8000",
        "User-Agent": "HTTPie/0.9.9",
        "X-B3-Sampled": "1",
        "X-B3-Spanid": "2728dcd05dd28045",
        "X-B3-Traceid": "f219486b82c140892728dcd05dd28045",
        "X-Envoy-Decorator-Operation": "httpbin.plain.svc.cluster.local:8000/*",
        "X-Istio-Attributes": "CiUKGGRlc3RpbmF0aW9uLnNlcnZpY2UubmFtZRIJEgdodHRwYmluCigKHWRlc3RpbmF0aW9uLnNlcnZpY2UubmFtZXNwYWNlEgcSBXBsYWluCjgKCnNvdXJjZS51aWQSKhIoa3ViZXJuZXRlczovL3NsZWVwLTU0NjY3NzU0ODUtOTRuY20ubWVzaAo9ChhkZXN0aW5hdGlvbi5zZXJ2aWNlLmhvc3QSIRIfaHR0cGJpbi5wbGFpbi5zdmMuY2x1c3Rlci5sb2NhbAo7ChdkZXN0aW5hdGlvbi5zZXJ2aWNlLnVpZBIgEh5pc3RpbzovL3BsYWluL3NlcnZpY2VzL2h0dHBiaW4="
    },
    "origin": "172.20.0.76",
    "url": "http://httpbin.plain:8000/get"
}

bash-4.4# http http://httpbin.mesh:8000/get
HTTP/1.1 503 Service Unavailable
content-length: 57
content-type: text/plain
date: Wed, 24 Apr 2019 15:35:13 GMT
server: envoy

upstream connect error or disconnect/reset before headers

[root@tpl-centos chapter9]# cat meshpolicy.2.yaml
apiVersion: "authentication.istio.io/v1alpha1"
kind: "MeshPolicy"
metadata:
  name: "default"
spec:
  peers:
  - mtls:
     mode: PERMISSIVE
[root@tpl-centos chapter9]# kubectl apply -f meshpolicy.2.yaml
meshpolicy.authentication.istio.io/default configured

bash-4.4# http http://httpbin.mesh:8000/get
HTTP/1.1 200 OK
access-control-allow-credentials: true
access-control-allow-origin: *
content-length: 433
content-type: application/json
date: Wed, 24 Apr 2019 15:36:23 GMT
server: envoy
x-envoy-upstream-service-time: 10

{
    "args": {},
    "headers": {
        "Accept": "*/*",
        "Accept-Encoding": "gzip, deflate",
        "Content-Length": "0",
        "Host": "httpbin.mesh:8000",
        "User-Agent": "HTTPie/0.9.9",
        "X-B3-Parentspanid": "4a98b29d82a2d36c",
        "X-B3-Sampled": "1",
        "X-B3-Spanid": "df1d2af16b7746af",
        "X-B3-Traceid": "81675bd68b8457154a98b29d82a2d36c"
    },
    "origin": "127.0.0.1",
    "url": "http://httpbin.mesh:8000/get"
}


```