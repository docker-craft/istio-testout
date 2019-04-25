### use denier to block http request

```bash

[root@tpl-centos chapter8]# cat denier.yaml
apiVersion: "config.istio.io/v1alpha2"
kind: denier
metadata:
  name: code-7
spec:
  status:
    code: 7
    message: Not allowed

[root@tpl-centos chapter8]# cat checknothing.yaml
apiVersion: "config.istio.io/v1alpha2"
kind: checknothing
metadata:
  name: place-holder
spec:
[root@tpl-centos chapter8]# cat denier.rule.yaml
apiVersion: "config.istio.io/v1alpha2"
kind: rule
metadata:
  name: deny-sleep-v1-to-httpbin
spec:
  match: destination.labels["app"] == "httpbin" && source.labels["app"]=="sleep" && source.labels["version"] == "v1"
  actions:
    - handler: code-7.denier
      instances: [place-holder.checknothing]

[root@tpl-centos chapter8]# kubectl apply -f denier.yaml -n httpbin
denier.config.istio.io/code-7 created
[root@tpl-centos chapter8]# ls
chaos.listchecker.2.yaml  debug.yaml        fluentd              mem-quota    version.listentry.yaml
chaos.listchecker.yaml    denier.rule.yaml  listentry.rule.yaml  prom
checknothing.yaml         denier.yaml       log                  redis-quota
[root@tpl-centos chapter8]# kubectl apply -f checknothing.yaml -n httpbin
checknothing.config.istio.io/place-holder created
[root@tpl-centos chapter8]# kubeclt apply -f denier.rule.yaml -n httpbin
-bash: kubeclt: command not found
[root@tpl-centos chapter8]# kubectl apply -f denier.rule.yaml -n httpbin
rule.config.istio.io/deny-sleep-v1-to-httpbin created
[root@tpl-centos chapter8]# kubectl get pods -n httpbin
NAME                       READY   STATUS    RESTARTS   AGE
httpbin-54f5bb4957-876xw   2/2     Running   0          15m
sleep-5466775485-qng8k     2/2     Running   0          13m
[root@tpl-centos chapter8]# kubectl exec -it sleep-5466775485-qng8k -n httpbin -c sleep bash
bash-4.4# http http://httbin:8000/ip

http: error: ConnectionError: HTTPConnectionPool(host='httbin', port=8000): Max retries exceeded with url: /ip (Caused by NewConnectionError('<urllib3.connection.HTTPConnection object at 0x7f8f116c7780>: Failed to establish a new connection: [Errno -2] Name does not resolve',)) while doing GET request to URL: http://httbin:8000/ip
bash-4.4# exit
exit
command terminated with exit code 1
[root@tpl-centos chapter8]# ls
chaos.listchecker.2.yaml  debug.yaml        fluentd              mem-quota    version.listentry.yaml
chaos.listchecker.yaml    denier.rule.yaml  listentry.rule.yaml  prom
checknothing.yaml         denier.yaml       log                  redis-quota

[root@tpl-centos chapter8]# kubectl delete -f denier.rule.yaml -n httpbin
rule.config.istio.io "deny-sleep-v1-to-httpbin" deleted
[root@tpl-centos chapter8]# kubectl exec -it sleep-5466775485-qng8k -n httpbin -c sleep bash
bash-4.4# http http://httpbin:8000/ip
HTTP/1.1 200 OK
access-control-allow-credentials: true
access-control-allow-origin: *
content-length: 28
content-type: application/json
date: Wed, 24 Apr 2019 05:30:48 GMT
server: envoy
x-envoy-upstream-service-time: 7

{
    "origin": "127.0.0.1"
}

```