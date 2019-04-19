```bash

[root@localhost helm]# helm search redis
NAME                            	CHART VERSION	APP VERSION	DESCRIPTION                                                 
stable/prometheus-redis-exporter	1.0.2        	0.28.0     	Prometheus exporter for Redis metrics                       
stable/redis                    	6.4.5        	4.0.14     	Open source, advanced key-value store. It is often referr...
stable/redis-ha                 	3.4.0        	5.0.3      	Highly available Kubernetes implementation of Redis         
stable/sensu                    	0.2.3        	0.28       	Sensu monitoring framework backed by the Redis transport    
[root@localhost helm]# helm install stable/redis
NAME:   ignoble-liger
LAST DEPLOYED: Fri Apr 19 19:39:43 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                        DATA  AGE
ignoble-liger-redis         3     0s
ignoble-liger-redis-health  3     0s

==> v1/Pod(related)
NAME                                        READY  STATUS             RESTARTS  AGE
ignoble-liger-redis-master-0                0/1    Pending            0         0s
ignoble-liger-redis-slave-855c59fccc-s6bpn  0/1    ContainerCreating  0         0s

==> v1/Secret
NAME                 TYPE    DATA  AGE
ignoble-liger-redis  Opaque  1     0s

==> v1/Service
NAME                        TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)   AGE
ignoble-liger-redis-master  ClusterIP  10.68.15.15    <none>       6379/TCP  0s
ignoble-liger-redis-slave   ClusterIP  10.68.154.215  <none>       6379/TCP  0s

==> v1beta1/Deployment
NAME                       READY  UP-TO-DATE  AVAILABLE  AGE
ignoble-liger-redis-slave  0/1    1           0          0s

==> v1beta2/StatefulSet
NAME                        READY  AGE
ignoble-liger-redis-master  0/1    0s


NOTES:
** Please be patient while the chart is being deployed **
Redis can be accessed via port 6379 on the following DNS names from within your cluster:

ignoble-liger-redis-master.default.svc.cluster.local for read/write operations
ignoble-liger-redis-slave.default.svc.cluster.local for read-only operations


To get your password run:

    export REDIS_PASSWORD=$(kubectl get secret --namespace default ignoble-liger-redis -o jsonpath="{.data.redis-password}" | base64 --decode)

To connect to your Redis server:

1. Run a Redis pod that you can use as a client:

   kubectl run --namespace default ignoble-liger-redis-client --rm --tty -i --restart='Never' \
    --env REDIS_PASSWORD=$REDIS_PASSWORD \
   --image docker.io/bitnami/redis:4.0.14 -- bash

2. Connect using the Redis CLI:
   redis-cli -h ignoble-liger-redis-master -a $REDIS_PASSWORD
   redis-cli -h ignoble-liger-redis-slave -a $REDIS_PASSWORD

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/ignoble-liger-redis 6379:6379 &
    redis-cli -h 127.0.0.1 -p 6379 -a $REDIS_PASSWORD

```