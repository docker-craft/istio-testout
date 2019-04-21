https://stackoverflow.com/questions/44110876/kubernetes-service-external-ip-pending

```bash

[root@localhost ~]# kubectl edit svc istio-ingressgateway -n istio-system

add the following:

spec:
  type: LoadBalancer
  externalIPs:
  - 192.168.56.11
  - 10.0.2.11

[root@localhost ~]# kubectl get svc istio-ingressgateway -n istio-system
NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP               PORT(S)                                                                                                                                      AGE
istio-ingressgateway   LoadBalancer   10.68.243.83   10.0.2.11,192.168.56.11   15020:29102/TCP,80:31380/TCP,443:31390/TCP,31400:31400/TCP,15029:28091/TCP,15030:23401/TCP,15031:20301/TCP,15032:32491/TCP,15443:21398/TCP   3h51m


curl http://10.0.2.11/hello

curl http://192.168.56.11/hello


```