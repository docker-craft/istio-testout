### listChecker adapter

```bash
[root@tpl-centos chapter8]# cat chaos.listchecker.yaml
apiVersion: config.istio.io/v1alpha2
kind: listchecker
metadata:
  name: chaos
spec:
  overrides: ["v1", "v3"]
  blacklist: true

[root@tpl-centos chapter8]# cat version.listentry.yaml
apiVersion: config.istio.io/v1alpha2
kind: listentry
metadata:
  name: version
spec:
  value: source.labels["version"]

[root@tpl-centos chapter8]# cat listentry.rule.yaml
apiVersion: config.istio.io/v1alpha2
kind: rule
metadata:
  name: checkversion
spec:
  match: destination.labels["app"] == "httpbin"
  actions:
    - handler: chaos.listchecker
      instances:
        - version.listentry

```

### test out

```bash

[root@tpl-centos chapter8]# kubectl apply -f listentry.rule.yaml  -n httpbin                             rule.config.istio.io/checkversion created
[root@tpl-centos chapter8]# kubectl apply -f version.listentry.yaml  -n httpbin
listentry.config.istio.io/version created
[root@tpl-centos chapter8]# kubectl apply -f chaos.listchecker.yaml -n httpbin
listchecker.config.istio.io/chaos created
[root@tpl-centos chapter8]# kubectl -n httpbin exec -it sleep-v2-f6d86c68d-xhxw6 -c sleep bash
bash-4.4# http http://httpbin:8000/ip
HTTP/1.1 200 OK
access-control-allow-credentials: true
access-control-allow-origin: *
content-length: 28
content-type: application/json
date: Wed, 24 Apr 2019 05:52:58 GMT
server: envoy
x-envoy-upstream-service-time: 11

{
    "origin": "127.0.0.1"
}

[root@tpl-centos chapter8]# kubectl -n httpbin exec -it sleep-v1-5466775485-c266h -c sleep bash
bash-4.4# http http://httpbin:8000/ip
HTTP/1.1 403 Forbidden
content-length: 61
content-type: text/plain
date: Wed, 24 Apr 2019 05:53:23 GMT
server: envoy
x-envoy-upstream-service-time: 3

PERMISSION_DENIED:chaos.listchecker.httpbin:v1 is blacklisted
```