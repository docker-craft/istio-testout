```bash

kubectl edit svc kiali -n istio-system

type : change ClusterIP --> NodePode


[root@localhost templates]# kubectl get svc -n istio-system 
NAME                     TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                                                                                      AGE
grafana                  NodePort       10.68.125.37    <none>        3000:30210/TCP                                                                                                                               43m
istio-citadel            ClusterIP      10.68.103.57    <none>        8060/TCP,15014/TCP                                                                                                                           43m
istio-galley             ClusterIP      10.68.68.10     <none>        443/TCP,15014/TCP,9901/TCP                                                                                                                   43m
istio-ingressgateway     LoadBalancer   10.68.243.83    <pending>     15020:29102/TCP,80:31380/TCP,443:31390/TCP,31400:31400/TCP,15029:28091/TCP,15030:23401/TCP,15031:20301/TCP,15032:32491/TCP,15443:21398/TCP   43m
istio-pilot              ClusterIP      10.68.182.74    <none>        15010/TCP,15011/TCP,8080/TCP,15014/TCP                                                                                                       43m
istio-policy             ClusterIP      10.68.44.216    <none>        9091/TCP,15004/TCP,15014/TCP                                                                                                                 43m
istio-sidecar-injector   ClusterIP      10.68.13.195    <none>        443/TCP                                                                                                                                      43m
istio-telemetry          ClusterIP      10.68.218.176   <none>        9091/TCP,15004/TCP,15014/TCP,42422/TCP                                                                                                       43m
jaeger-agent             ClusterIP      None            <none>        5775/UDP,6831/UDP,6832/UDP                                                                                                                   43m
jaeger-collector         ClusterIP      10.68.225.76    <none>        14267/TCP,14268/TCP                                                                                                                          43m
jaeger-query             NodePort       10.68.181.79    <none>        16686:34340/TCP                                                                                                                              43m
kiali                    NodePort       10.68.115.39    <none>        20001:23417/TCP                                                                                                                              43m
prometheus               NodePort       10.68.111.233   <none>        9090:20588/TCP                                                                                                                               43m
servicegraph             ClusterIP      10.68.15.112    <none>        8088/TCP                                                                                                                                     43m
tracing                  ClusterIP      10.68.72.243    <none>        80/TCP                                                                                                                                       43m
zipkin                   ClusterIP      10.68.50.72     <none>        9411/TCP                                                                                                                                     43m


#kiali console address:

http://xxx:port/kiali/console



[root@localhost templates]# kubectl create secret generic kiali -n istio-system --from-literal "username=admin" --from-literal "passphrase=admin"


[root@localhost templates]# kubectl delete pods kiali-5df77dc9b6-tjt65 -n istio-system


```