###step 1: deplooy flaskapp

```bash

# deploy flaskapp
[root@localhost flaskapp]# ls
flaskapp.istio.yaml
[root@localhost flaskapp]# pwd
/root/istio-for-beginner/code/flaskapp
[root@localhost flaskapp]# istioctl kube-inject -f flaskapp.istio.yaml | kubectl apply -f -
service/flaskapp created
deployment.extensions/flaskapp-v1 created
deployment.extensions/flaskapp-v2 created
[root@localhost flaskapp]# kubectl get po -w
NAME                           READY   STATUS    RESTARTS   AGE
flaskapp-v1-545bb6954d-c6xpt   2/2     Running   0          12s
flaskapp-v2-57666cf44c-vv7zc   2/2     Running   0          12s

```

###step 2: deploy client sleep.yaml

```bash
[root@localhost code]# cd sleep/
[root@localhost sleep]# ls
sleep.istio.yaml  sleep.yaml
[root@localhost sleep]# istioctl kube-inject -f sleep.yaml | kubectl apply -f -
service/sleep created
deployment.extensions/sleep created
[root@localhost sleep]# kubectl get po -w
NAME                           READY   STATUS    RESTARTS   AGE
flaskapp-v1-545bb6954d-c6xpt   2/2     Running   0          7m36s
flaskapp-v2-57666cf44c-vv7zc   2/2     Running   0          7m36s
sleep-5d984b9969-8wr7t         2/2     Running   0          20s

```


### Step 3: verify the service

```bash

[root@localhost sleep]# kubectl exec -it sleep-5d984b9969-8wr7t -c sleep bash
bash-4.4# for i in `seq 10`; do http --body http://flaskapp/env/version; done
v1

v2

v1

v2

v2

v1

v1

v2

v2

v1

bash-4.4#

```

### Step 4: DesinationRule & defaultRouting

```bash
[root@localhost chapter4]# ls
flaskapp-default-vs-v2.yaml  flaskapp-destinationrule.yaml

[root@localhost chapter4]# cat flaskapp-destinationrule.yaml 
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: flaskapp
spec:
  host: flaskapp
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2

[root@localhost chapter4]# kubectl apply -f flaskapp-destinationrule.yaml 
destinationrule.networking.istio.io/flaskapp created

[root@localhost chapter4]# cat flaskapp-default-vs-v2.yaml 
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: flaskapp-default-v2
spec:
  hosts:
    - flaskapp
  http:
    - route:
        - destination:
            host: flaskapp
            subset: v2

#apply the virtualservice to the cluster.
[root@localhost chapter4]# kubectl apply -f flaskapp-default-vs-v2.yaml 
virtualservice.networking.istio.io/flaskapp-default-v2 created

# use the shell client, check the routing rule.

kubectl exec -it sleep-5d984b9969-8wr7t -c sleep bash

[root@localhost chapter4]# kubectl exec -it sleep-5d984b9969-8wr7t -c sleep bash
bash-4.4# for i in `seq 10`; do http --body http://flaskapp/env/version; done
v2

v2

v2

v2

v2

v2

v2

v2

v2

v2

bash-4.4# v1
```