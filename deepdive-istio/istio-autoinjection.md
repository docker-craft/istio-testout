### istio-autoinject=enabled test

```bash

[root@tpl-centos sleep]# ls
sleep.injected.yaml  sleep.istio.yaml  sleep.yaml
[root@tpl-centos sleep]# kubectl create ns auto
namespace/auto created
[root@tpl-centos sleep]# kubectl label namespaces auto istio-injection=enabled
namespace/auto labeled
[root@tpl-centos sleep]# kubectl create ns manual
namespace/manual created
[root@tpl-centos sleep]# kubectl apply -f sleep.yaml -n auto
service/sleep created
deployment.extensions/sleep created
[root@tpl-centos sleep]# kubectl apply -f sleep.yaml -n manual
service/sleep created
deployment.extensions/sleep created
[root@tpl-centos sleep]# kubectl get pods -n auto
NAME                     READY   STATUS    RESTARTS   AGE
sleep-5466775485-9mkv8   2/2     Running   0          54s
[root@tpl-centos sleep]# kubectl get pods -n manual
NAME                     READY   STATUS    RESTARTS   AGE
sleep-5466775485-pb8mp   1/1     Running   0          46s

```