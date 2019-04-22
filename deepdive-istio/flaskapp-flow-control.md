## Istio Http Flow control 

### 1. Default DestinationRule

```bash
[root@localhost chapter7]# kubectl get svc 
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
flaskapp     ClusterIP   10.68.142.101   <none>        80/TCP     19h

[root@localhost chapter7]# cat flaskapp.destinationrule.yaml.yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: flaskapp
spec:
  host: flaskapp.default.svc.cluster.local
  trafficPolicy:
    loadBalancer:
      simple: LEAST_CONN
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2


[root@localhost chapter7]# kubectl apply -f flaskapp.destinationrule.yaml.yaml
destinationrule.networking.istio.io/flaskapp created


cat flaskapp.virtualservice.yaml

[root@localhost chapter7]# cat flaskapp.virtualservice.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: flaskapp
spec:
  hosts:
    - flaskapp.default.svc.cluster.local
  http:
    - route:
        - destination:
            host: flaskapp.default.svc.cluster.local
            subset: v1



[root@localhost chapter7]# kubectl apply -f flaskapp.virtualservice.yaml 
virtualservice.networking.istio.io/flaskapp created

export SOURCE_POD=$(kubectl get pod -l app=sleep,version=v1 -o jsonpath={.items..metadata.name})

[root@localhost chapter7]# export SOURCE_POD=$(kubectl get pod -l app=sleep,version=v1 -o jsonpath={.items..metadata.name})
[root@localhost chapter7]# echo $SOURCE_POD
sleep-6b87888584-xmb59
[root@localhost chapter7]# kubectl exec -it -c sleep $SOURCE_POD bash
bash-4.4# http --body http://flaskapp/env/version
v1

bash-4.4# http --body http://flaskapp.default/env/version
v1


```

### 2. Http request split

```bash

[root@localhost chapter7]# cat flaskapp.virtualservice-split.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: flaskapp
spec:
  hosts:
    - flaskapp.default.svc.cluster.local
  http:
    - route:
        - destination:
            host: flaskapp.default.svc.cluster.local
            subset: v1
          weight: 70
        - destination:
            host: flaskapp.default.svc.cluster.local
            subset: v2
          weight: 30


[root@localhost chapter7]# kubectl exec -it -c sleep $SOURCE_POD bash

bash-4.4# http --body http://flaskapp/env/version
v2

bash-4.4# http --body http://flaskapp/env/version
v1

bash-4.4# http --body http://flaskapp/env/version
v2

bash-4.4# http --body http://flaskapp/env/version
v1



```

### 3. Canary deployment

```bash
[root@localhost chapter7]# cat flaskapp.virtualservice-canary.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: flaskapp
spec:
  hosts:
    - flaskapp.default.svc.cluster.local
  http:
    - match:
        - headers:
            lab:
              exact: canary
      route:
        - destination:
            host: flaskapp.default.svc.cluster.local
            subset: v2
    - route:
        - destination:
            host: flaskapp.default.svc.cluster.local
            subset: v1


[root@localhost chapter7]# kubectl apply -f flaskapp.virtualservice-canary.yaml
virtualservice.networking.istio.io/flaskapp configured
[root@localhost chapter7]# kubectl exec -it -c sleep $SOURCE_POD bash
bash-4.4# http --body http://flaskapp/env/version lab:canary
v2

bash-4.4# http --body http://flaskapp/env/version 
v1

bash-4.4# http --body http://flaskapp/env/version lab:phoenix
v1

```

### 4. sourceLabels dispatch

```bash

[root@localhost chapter7]# cat flaskapp.virtualservice-version.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: flaskapp
spec:
  hosts:
    - flaskapp.default.svc.cluster.local
  http:
    - match:
        - sourceLabels:
            app: sleep
            version: v1
      route:
        - destination:
            host: flaskapp.default.svc.cluster.local
            subset: v1
    - route:
        - destination:
            host: flaskapp.default.svc.cluster.local
            subset: v2


kubectl get pod -l app=sleep,version=v1 -o jsonpath={.items..metadata.name

[root@localhost chapter7]# SOURCE_POD=$(kubectl get pod -l app=sleep,version=v1 -o jsonpath={.items..metadata.name})

kubectl exec -it -c sleep $SOURCE_POD http  http://flaskapp/evn/version

[root@localhost chapter7]# kubectl exec -it -c sleep $SOURCE_POD http  http://flaskapp/env/version
HTTP/1.1 200 OK
content-length: 2
content-type: text/html; charset=utf-8
date: Mon, 22 Apr 2019 03:21:54 GMT
server: envoy
x-envoy-upstream-service-time: 0

v1


SOURCE_POD=$(kubectl get pod -l app=sleep,version=v2 -o jsonpath={.items..metadata.name})

kubectl exec -it -c sleep $SOURCE_POD http  http://flaskapp/env/version


```

### 5. redirect

```bash
[root@localhost chapter7]# cat flaskapp.virtualservice-redirect.yaml 
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: flaskapp
spec:
  hosts:
    - flaskapp.default.svc.cluster.local
  http:
    - match:
        - sourceLabels:
            app: sleep
            version: v1
          uri:
            exact: "/env/HOSTNAME"
      redirect:
        uri: /env/version
    - route:
        - destination:
            host: flaskapp.default.svc.cluster.local
            subset: v2
[root@localhost chapter7]# kubectl apply -f flaskapp.virtualservice-redirect.yaml 
virtualservice.networking.istio.io/flaskapp configured
[root@localhost chapter7]# SOURCE_POD=$(kubectl get pod -l app=sleep,version=v1 -o jsonpath={.items..metadata.name})
[root@localhost chapter7]# kubectl exec -it -c sleep $SOURCE_POD http  http://flaskapp/env/versionHTTP/1.1 200 OK
content-length: 2
content-type: text/html; charset=utf-8
date: Mon, 22 Apr 2019 04:46:46 GMT
server: envoy
x-envoy-upstream-service-time: 0

v2

[root@localhost chapter7]# SOURCE_POD=$(kubectl get pod -l app=sleep,version=v2 -o jsonpath={.items..metadata.name})
[root@localhost chapter7]# kubectl exec -it -c sleep $SOURCE_POD http  http://flaskapp/env/versionHTTP/1.1 200 OK
content-length: 2
content-type: text/html; charset=utf-8
date: Mon, 22 Apr 2019 04:46:57 GMT
server: envoy
x-envoy-upstream-service-time: 0

v2

```

### 6. timeout control

```bash

[root@localhost httpbin]# kubectl apply -f <(istioctl kube-inject -f httpbin.yaml)
service/httpbin created
deployment.extensions/httpbin created

[root@localhost httpbin]# kubectl apply -f httpbin.virtualservice.yaml 
virtualservice.networking.istio.io/httpbin created


[root@localhost chapter7]# cat httpbin.virtualservice.timeout.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin
spec:
  hosts:
    - httpbin.default.svc.cluster.local
  http:
    - timeout: 3s
      route:
        - destination:
            host: httpbin.default.svc.cluster.local


[root@localhost chapter7]# kubectl apply -f httpbin.virtualservice.timeout.yaml

[root@localhost chapter7]# kubectl exec -it sleep-v1-cc749f79c-4fcnp -c sleep bash
bash-4.4# http --body http://httpbin:8000/delay/5
upstream request timeout

bash-4.4# http --body http://httpbin:8000/delay/2
{
    "args": {},
    "data": "",
    "files": {},
    "form": {},
    "headers": {
        "Accept": "*/*",
        "Accept-Encoding": "gzip, deflate",
        "Content-Length": "0",
        "Host": "httpbin:8000",
        "User-Agent": "HTTPie/0.9.9",
        "X-B3-Parentspanid": "891ad5a80465b462",
        "X-B3-Sampled": "0",
        "X-B3-Spanid": "a97f8e8dc208c1aa",
        "X-B3-Traceid": "5e72f00d982ef4c8891ad5a80465b462"
    },
    "origin": "127.0.0.1",
    "url": "http://httpbin:8000/delay/2"
}
```


### 7.timeout retry

```bash
[root@localhost chapter7]# cat httpbin.virtualservice.timeout.retry.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin
spec:
  hosts:
    - httpbin.default.svc.cluster.local
  http:
    - route:
        - destination:
            host: httpbin.default.svc.cluster.local
      retries:
          attempts: 3
          perTryTimeout: 1s
      timeout: 7s
[root@localhost chapter7]# kubectl apply -f httpbin.virtualservice.timeout.retry.yaml
virtualservice.networking.istio.io/httpbin configured


[root@localhost chapter7]# kubectl exec -it sleep-v1-cc749f79c-4fcnp -c sleep bash
bash-4.4# http http://httpbin:8000/delay/10
HTTP/1.1 504 Gateway Timeout
content-length: 24
content-type: text/plain
date: Mon, 22 Apr 2019 05:53:50 GMT
server: envoy

upstream request timeout


```

### 8. expose services with Gateway

```bash

[root@localhost chapter7]# cat flaskapp.virtualservice.gateway.2.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: flaskapp
spec:
  hosts:
    - flaskapp.default.svc.cluster.local
    - flaskapp.microservice.rocks
    - flaskapp.microservice.xyz
  gateways:
    - mesh
    - example-gateway
  http:
    - match:
        - gateways:
            - example-gateway
      route:
        - destination:
            host: flaskapp.default.svc.cluster.local
            subset: v1
    - route:
        - destination:
            host: flaskapp.default.svc.cluster.local
            subset: v2
[root@localhost chapter7]# kubectl apply -f flaskapp.virtualservice.gateway.2.yaml
virtualservice.networking.istio.io/flaskapp configured


[root@localhost chapter7]# cat example.gateway.yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: example-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
    - port:
        number: 80
        name: http
        protocol: HTTP
      hosts:
        - "*.microservice.rocks"
        - "*.microservice.xyz"

[root@localhost chapter7]# kubectl apply -f example.gateway.yaml
gateway.networking.istio.io/example-gateway created

[root@localhost chapter7]# kubectl exec -it sleep-v1-cc749f79c-4fcnp -c sleep bash

bash-4.4# http flaskapp.microservice.rocks/env/version
HTTP/1.1 200 OK
content-length: 2
content-type: text/html; charset=utf-8
date: Mon, 22 Apr 2019 07:15:00 GMT
server: envoy
x-envoy-upstream-service-time: 1

v2

```

### 9. visit outside network

```bash

#solution 1 , values.yarml proxy.includeIPRanges

#solution 2, add annotations to traffic.sidecar.istio.io/includeOutboundIPRanges

[root@localhost sleep]# cat sleep.yaml 
apiVersion: v1
kind: Service
metadata:
  name: sleep
  labels:
    app: sleep
    version: v1
spec:
  selector:
    app: sleep
    version: v1
  ports:
    - name: ssh
      port: 80
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: sleep-v1
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: sleep
        version: v1
      annotations:
        traffic.sidecar.istio.io/includeOutboundIPRanges : 10.245.0.0/16
    spec:
      containers:
        - name: sleep
          image: dustise/sleep
          imagePullPolicy: IfNotPresent
          env:
            - name: version
              value: v1
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: sleep-v2
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: sleep
        version: v2
    spec:
      containers:
        - name: sleep
          image: dustise/sleep
          imagePullPolicy: IfNotPresent
          env:
            - name: version
              value: v2

[root@localhost sleep]# kubectl exec -it sleep-v1-7b875587bf-p2htv -c sleep bash
bash-4.4# http http://api.jd.com
HTTP/1.1 200 OK
Cache-Control: max-age=0
Connection: close
Content-Encoding: gzip
Content-Type: text/html
Date: Mon, 22 Apr 2019 07:40:30 GMT
ETag: W/"131-1553588768000"
Expires: Mon, 22 Apr 2019 07:40:30 GMT
Last-Modified: Tue, 26 Mar 2019 08:26:08 GMT
Server: jfe
Transfer-Encoding: chunked
Vary: Accept-Encoding

<html>
<script type="text/javascript">
    window.location="http://jos.jd.com";
</script>
<body>
<h2></h2>
</body>
</html>

```

### 10. SeraviceEntry

```bash
[root@localhost chapter7]# cat httpbin.entry.yaml 
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: httpbin-ext
spec:
  hosts:
    - httpbin.org
  ports:
    - number: 80
      name: http
      protocol: HTTP
  resolution: DNS
[root@localhost chapter7]# kubectl apply -f httpbin.entry.yaml
serviceentry.networking.istio.io/httpbin-ext created
[root@localhost chapter7]# kubectl exec -it sleep-v1-7b875587bf-p2htv -c sleep bash

bash-4.4# http http://httpbin.org/get
HTTP/1.1 200 OK
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Encoding: gzip
Content-Length: 180
Content-Type: application/json
Date: Mon, 22 Apr 2019 07:50:11 GMT
Referrer-Policy: no-referrer-when-downgrade
Server: nginx
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block

{
    "args": {},
    "headers": {
        "Accept": "*/*",
        "Accept-Encoding": "gzip, deflate",
        "Host": "httpbin.org",
        "User-Agent": "HTTPie/0.9.9"
    },
    "origin": "180.167.120.186, 180.167.120.186",
    "url": "https://httpbin.org/get"
}

#add timeout limit to ServiceEntry

[root@localhost chapter7]# cat serviceentry.virtualservice.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin-service
spec:
  hosts:
    - httpbin.org
  http:
    - timeout: 3s
      route:
        - destination:
            host: httpbin.org
[root@localhost chapter7]# kubectl apply -f serviceentry.virtualservice.yaml
virtualservice.networking.istio.io/httpbin-service created

[root@localhost chapter7]# kubectl exec -it sleep-v1-7b875587bf-p2htv -c sleep bash

bash-4.4# http http://httpbin.org/get
HTTP/1.1 200 OK
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Encoding: gzip
Content-Length: 180
Content-Type: application/json
Date: Mon, 22 Apr 2019 07:50:11 GMT
Referrer-Policy: no-referrer-when-downgrade
Server: nginx
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block

{
    "args": {},
    "headers": {
        "Accept": "*/*",
        "Accept-Encoding": "gzip, deflate",
        "Host": "httpbin.org",
        "User-Agent": "HTTPie/0.9.9"
    },
    "origin": "180.167.120.186, 180.167.120.186",
    "url": "https://httpbin.org/get"
}

```

### 11. Circuit Breaker

```bash
[root@localhost chapter7]# cat httpbin.cb.yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: httpbin
spec:
  host: httpbin
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 1
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
    outlierDetection:
      consecutiveErrors: 1
      interval: 1s
      baseEjectionTime: 3m
      maxEjectionPercent: 100

[root@localhost chapter7]# kubectl apply -f httpbin.cb.yaml
destinationrule.networking.istio.io/httpbin created


bash-4.4# wrk -c 3 -t 3 http://httpbin:8000/ip
Running 10s test @ http://httpbin:8000/ip
  3 threads and 3 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    18.06ms   70.97ms 513.59ms   94.35%
    Req/Sec     0.96k   751.86     3.77k    77.00%
  27485 requests in 10.44s, 8.55MB read
  Non-2xx or 3xx responses: 13305
Requests/sec:   2631.81
Transfer/sec:    838.52KB

[root@localhost chapter7]# kubectl delete -f httpbin.cb.yaml
destinationrule.networking.istio.io "httpbin" deleted

bash-4.4# wrk -c 3 -t 3 http://httpbin:8000/ip
Running 10s test @ http://httpbin:8000/ip
  3 threads and 3 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    30.60ms   93.28ms 519.79ms   91.24%
    Req/Sec   715.51    250.18     1.60k    80.99%
  19007 requests in 10.09s, 6.05MB read
Requests/sec:   1884.58
Transfer/sec:    614.70KB


```

### 12. inject fault response, to simulate fault respose, (used for testing purpose)

```bash

[root@localhost chapter7]# cat httpbin.virtualservice.fault.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin
spec:
  hosts:
  - httpbin.default.svc.cluster.local
  http:
  - match:
    - sourceLabels:
        app: sleep
        version: v1
    route:
      - destination:
          host: httpbin.default.svc.cluster.local
    fault:
      abort:
        httpStatus: 500
        percent: 100
  - route:
    - destination:
        host: httpbin.default.svc.cluster.local


[root@localhost chapter7]# kubectl apply -f httpbin.virtualservice.fault.yaml
virtualservice.networking.istio.io/httpbin created

[root@localhost chapter7]# kubectl exec -it sleep-v1-7b875587bf-p2htv -c sleep bash
bash-4.4# http --body http://httpbin:8000/ip
{
    "origin": "127.0.0.1"
}


```