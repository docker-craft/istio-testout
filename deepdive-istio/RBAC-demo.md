### RBAC demo

```bash
#启用 RBAC

[root@tpl-centos chapter9]# cat rbac.yaml
apiVersion: "rbac.istio.io/v1alpha1"
kind: RbacConfig
metadata:
  name: default
spec:
  mode: "ON_WITH_INCLUSION"
  inclusion:
    namespaces: ["default"]

---
apiVersion: authentication.istio.io/v1alpha1
kind: "MeshPolicy"
metadata:
  name: "default"
spec:
  peers:
  - mtls: {}

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sleep

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sleep-v2



[root@tpl-centos chapter9]# cat servicerole.yaml
apiVersion: "rbac.istio.io/v1alpha1"
kind: ServiceRole
metadata:
  name: service-viewer
spec:
  rules:
  - services:
    - "*"
    methods:
    - "GET"

[root@tpl-centos chapter9]# cat servicerolebinding.yaml
apiVersion: "rbac.istio.io/v1alpha1"
kind: ServiceRoleBinding
metadata:
  name: bind-service-viewer
spec:
  subjects:
    - properties:
        source.namespace: "default"
  roleRef:
    kind: ServiceRole
    name: "service-viewer"

[root@tpl-centos chapter9]# cat servicerole-owner.yaml
apiVersion: "rbac.istio.io/v1alpha1"
kind: ServiceRole
metadata:
  name: service-owner
spec:
  rules:
    - services: ["*"]
      methods: ["GET", "POST"]

[root@tpl-centos chapter9]# cat servicerolebinding-owner.yaml
apiVersion: "rbac.istio.io/v1alpha1"
kind: ServiceRoleBinding
metadata:
  name: bind-service-owner
spec:
  subjects:
    - user: "cluster.local/ns/default/sa/sleep"
  roleRef:
    kind: ServiceRole
    name: "service-owner"
[root@tpl-centos chapter9]#


[root@tpl-centos chapter9]# kubectl apply -f <(istioctl kube-inject -f sleep.istio.yaml)
service/sleep configured
deployment.extensions/sleep-v1 created
deployment.extensions/sleep-v2 created
[root@tpl-centos chapter9]# kubectl apply -f <(istioctl kube-inject -f httpbin.yaml)
service/httpbin created
deployment.extensions/httpbin created
[root@tpl-centos chapter9]# kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
flaskapp-v1-8f7fc8c9f-wt2lj    2/2     Running   4          6d12h
flaskapp-v2-7df7896bc6-9tqls   2/2     Running   4          6d12h
httpbin-74547d7498-svr2w       2/2     Running   0          61s
sleep-576577f454-nhsnd         2/2     Running   4          6d12h

[root@tpl-centos chapter9]# kubectl apply -f servicerole.yaml
servicerole.rbac.istio.io/service-viewer created
[root@tpl-centos chapter9]# kubectl apply -f servicerolebinding.yaml
servicerolebinding.rbac.istio.io/bind-service-viewer created

```