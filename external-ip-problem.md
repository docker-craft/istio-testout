https://stackoverflow.com/questions/44110876/kubernetes-service-external-ip-pending

```bash

[root@localhost ~]# kubectl edit svc istio-ingressgateway -n istio-system

add the following:

spec:
  type: LoadBalancer
  externalIPs:
  - 192.168.56.11
  - 10.0.2.11

```