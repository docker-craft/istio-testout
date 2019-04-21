## Hello world istio demo

### 1.istio enabled apps

```bash

git clone https://github.com/docker-craft/kubernetes-course.git

cd kubernetes/istio

cat README.md

kubectl apply -f <(istioctl kube-inject -f helloworld.yaml)
kubectl apply -f helloworld-gw.yaml

kubectl edit pods istio-ingressgateway -n istio-system

curl http://10.68.243.83/hello

curl http://192.168.58.51/hello


```

### 2.traffic routing

```bash

cd kubernetes/istio

kubectl apply -f <(istioctl kube-inject -f helloworld-v2.yaml)

kubectl apply -f helloworld-v2-routing.yaml

[root@tpl-centos istio]# curl http://192.168.58.51/hello
[root@tpl-centos istio]# curl http://192.168.58.51/hello -H "host: hello.example.com"
hello world !!!

[root@tpl-centos istio]# curl http://192.168.58.51/hello -H "host: hello.example.com" -H "end-user: john"
hello, this is v2 !!!


```

### 3.cancary deployment

```bash

[root@tpl-centos istio]# kubectl get virtualservice
NAME                  GATEWAYS               HOSTS                 AGE
flaskapp-default-v2                          [flaskapp]            5d
helloworld            [helloworld-gateway]   [hello.example.com]   5d
[root@tpl-centos istio]# kubectl describe virtualservice helloworld



kubectl apply -f helloworld-v2-canary.yaml

kubectl get virtualservice

[root@tpl-centos istio]# kubectl describe virtualservice helloworld

[root@tpl-centos istio]# kubectl describe virtualservice helloworld
Name:         helloworld
Namespace:    default
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"networking.istio.io/v1alpha3","kind":"VirtualService","metadata":{"annotations":{},"name":"helloworld","namespace":"default..."}
API Version:  networking.istio.io/v1alpha3
Kind:         VirtualService
Metadata:
  Creation Timestamp:  2019-04-16T05:27:07Z
  Generation:          4
  Resource Version:    344123
  Self Link:           /apis/networking.istio.io/v1alpha3/namespaces/default/virtualservices/helloworld
  UID:                 4694b01b-6008-11e9-b412-000c298cf908
Spec:
  Gateways:
    helloworld-gateway
  Hosts:
    hello.example.com
  Http:
    Route:
      Destination:
        Host:  hello.default.svc.cluster.local
        Port:
          Number:  8080
        Subset:    v1
      Weight:      90
      Destination:
        Host:  hello.default.svc.cluster.local
        Port:
          Number:  8080
        Subset:    v2
      Weight:      10
Events:            <none>


for ((i=1;i<=10;i++)); do curl http://192.168.58.51/hello -H "host: hello.example.com"; done

[root@tpl-centos istio]# for ((i=1;i<=10;i++)); do curl http://192.168.58.51/hello -H "host: hello.example.com"; done
hello world !!!

hello world !!!

hello world !!!

hello world !!!

hello, this is v2 !!!
hello world !!!

hello world !!!

hello world !!!

hello world !!!

hello world !!!
```

### 4. retries

```bash

kubectl apply -f helloworld-v3.yaml

[root@tpl-centos istio]# kubectl apply -f helloworld-v3.yaml
deployment.extensions/hello-v3 created
deployment.extensions/hello-v3-latency created
destinationrule.networking.istio.io/hello configured
virtualservice.networking.istio.io/helloworld-v3 created

[root@tpl-centos istio]# kubectl get destinationrule
NAME       HOST                              AGE
flaskapp   flaskapp                          5d
hello      hello.default.svc.cluster.local   29m
[root@tpl-centos istio]# kubectl describe destinationrule hello
Name:         hello
Namespace:    default
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"networking.istio.io/v1alpha3","kind":"DestinationRule","metadata":{"annotations":{},"name":"hello","namespace":"default"},"..."}
API Version:  networking.istio.io/v1alpha3
Kind:         DestinationRule
Metadata:
  Creation Timestamp:  2019-04-21T14:13:30Z
  Generation:          2
  Resource Version:    344873
  Self Link:           /apis/networking.istio.io/v1alpha3/namespaces/default/destinationrules/hello
  UID:                 a4002019-643f-11e9-8e58-000c298cf908
Spec:
  Host:  hello.default.svc.cluster.local
  Subsets:
    Labels:
      Version:  v1
    Name:       v1
    Labels:
      Version:  v2
    Name:       v2
    Labels:
      Version:  v3
    Name:       v3
Events:         <none>

[root@tpl-centos istio]# kubectl get virtualservice
NAME                  GATEWAYS               HOSTS                    AGE
flaskapp-default-v2                          [flaskapp]               5d
helloworld            [helloworld-gateway]   [hello.example.com]      5d
helloworld-v3         [helloworld-gateway]   [hello-v3.example.com]   3m
[root@tpl-centos istio]# kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
details-v1-78c4b45597-vrnh2         2/2     Running   0          5h22m
flaskapp-v1-8f7fc8c9f-wt2lj         2/2     Running   0          3d1h
flaskapp-v2-7df7896bc6-9tqls        2/2     Running   0          3d1h
hello-7979848688-2jd6d              2/2     Running   0          5h20m
hello-7979848688-rqgxt              2/2     Running   0          5h20m
hello-7979848688-z9wnb              2/2     Running   0          5h20m
hello-v2-59fcc67d7-659bw            2/2     Running   0          31m
hello-v2-59fcc67d7-bvhrl            2/2     Running   0          31m
hello-v2-59fcc67d7-kcmhh            2/2     Running   0          31m
hello-v3-fb79d5bbd-bfqg8            1/1     Running   0          3m54s
hello-v3-fb79d5bbd-lc9j9            1/1     Running   0          3m54s
hello-v3-latency-6fb6dd755b-r9tkl   1/1     Running   0          3m54s
helloworld-v1-6bcf84fb77-9gv72      2/2     Running   0          3d1h
helloworld-v2-76567c6c74-k56wz      2/2     Running   0          3d1h
productpage-v1-64647d4c5f-6vfzs     2/2     Running   0          5h21m
ratings-v1-7d85cb4dfc-cdgh8         2/2     Running   0          5h22m
reviews-v1-7cbc499fdb-5cxjd         2/2     Running   0          5h22m
reviews-v2-777846575f-9p8db         2/2     Running   0          5h22m
reviews-v3-6b886bfc44-j9pcn         2/2     Running   0          5h22m
sleep-576577f454-nhsnd              2/2     Running   0          3d1h
world-2-db47d45-5jjxj               2/2     Running   0          5h20m
world-2-db47d45-f4945               2/2     Running   0          5h20m
world-2-db47d45-fq9j2               2/2     Running   0          5h20m
world-7d47b7c77c-lkqlz              2/2     Running   0          5h20m
world-7d47b7c77c-lr72x              2/2     Running   0          5h20m
world-7d47b7c77c-rvlxc              2/2     Running   0          5h20m


[root@tpl-centos istio]# curl http://192.168.58.51/hello -H "Host:hello-v3.example.com"
hello, this is hello-v3-fb79d5bbd-bfqg8


for ((i=1;i<=10;i++)); do time curl http://192.168.58.51/hello -H "host: hello-v3.example.com"; done

[root@tpl-centos istio]# for ((i=1;i<=10;i++)); do time curl http://192.168.58.51/hello -H "host: hello-v3.example.com"; done
hello, this is hello-v3-fb79d5bbd-bfqg8
real    0m0.060s
user    0m0.004s
sys     0m0.007s
upstream request timeout
real    0m2.053s
user    0m0.007s
sys     0m0.020s
upstream request timeout
real    0m2.030s
user    0m0.002s
sys     0m0.008s
hello, this is hello-v3-fb79d5bbd-lc9j9
real    0m0.025s
user    0m0.003s
sys     0m0.007s
upstream request timeout
real    0m2.034s
user    0m0.002s
sys     0m0.013s
hello, this is hello-v3-fb79d5bbd-lc9j9
real    0m0.161s
user    0m0.000s
sys     0m0.011s
hello, this is hello-v3-fb79d5bbd-lc9j9
real    0m0.074s
user    0m0.005s
sys     0m0.007s
hello, this is hello-v3-fb79d5bbd-bfqg8
real    0m0.068s
user    0m0.002s
sys     0m0.010s
upstream request timeout
real    0m2.031s
user    0m0.003s
sys     0m0.010s
hello, this is hello-v3-fb79d5bbd-bfqg8
real    0m0.011s
user    0m0.003s
sys     0m0.007s

```

### 5. mutual TLS

```bash


kubectl apply -f <(istioctl kube-inject -f helloworld-tls.yaml)

kubectl apply -f helloworld-tls-legacy.yaml #no need to inject sidecar




kubectl apply -f <(istioctl kube-inject -f helloworld-tls-enable.yaml)


```

### 6. RBAC

```bash

kubectl create -f helloworld-rbac-enable.yaml

kubectl apply -f <(istioctl kube-inject -f helloworld-rbac.yaml)

kubectl get pods
kubectl get service -n istio-system


curl http://192.168.58.51 -H "Host: hello-rbac.example.com"

kubectl exec -it world-xxx -- sh

/app #  wget hello:8080
/app #  wget world:8080
/app #  wget world-2:8080
/app #  wget localhost:8080
```

### 7.istio authentication with jwt

```bash

kubectl apply -f <(istioctl kube-inject -f helloworld-jwt.yaml)

kubectl get pods


curl http://192.168.58.51/  -H "Host: hello.kubernetes.newtech.academy"


```

### 8.external service