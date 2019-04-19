##Istio installation guide

###step 1 --> install k8s

after installation, you can do varifcation and see:

```bash
kubectl version
kubectl get componentstatus # 可以看到scheduler/controller-manager/etcd等组件 Healthy
kubectl cluster-info # 可以看到kubernetes master(apiserver)组件 running
kubectl get node # 可以看到单 node Ready状态
kubectl get pod --all-namespaces # 可以查看所有集群pod状态，默认已安装网络插件、coredns、metrics-server等
kubectl get svc --all-namespaces # 可以查看所有集群服务状态
```

```bash
[root@localhost ~]# kubectl version
Client Version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.5", GitCommit:"2166946f41b36dea2c4626f90a77706f426cdea2", GitTreeState:"clean", BuildDate:"2019-03-25T15:26:52Z", GoVersion:"go1.11.5", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.5", GitCommit:"2166946f41b36dea2c4626f90a77706f426cdea2", GitTreeState:"clean", BuildDate:"2019-03-25T15:19:22Z", GoVersion:"go1.11.5", Compiler:"gc", Platform:"linux/amd64"}

[root@localhost ~]# kubectl get componentstatus
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok                  
controller-manager   Healthy   ok                  
etcd-0               Healthy   {"health":"true"}


[root@localhost ~]# kubectl cluster-info
Kubernetes master is running at https://192.168.56.12:6443
CoreDNS is running at https://192.168.56.12:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://192.168.56.12:6443/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

[root@localhost ~]# kubectl get node
NAME            STATUS   ROLES    AGE   VERSION
192.168.56.12   Ready    master   38m   v1.13.5

[root@localhost ~]# kubectl get pods
No resources found.

[root@localhost ~]# kubectl get pod --all-namespaces
NAMESPACE     NAME                                    READY   STATUS             RESTARTS   AGE
kube-system   coredns-7c5785cbcc-fw7l7                0/1     CrashLoopBackOff   11         37m
kube-system   coredns-7c5785cbcc-ghnhs                0/1     CrashLoopBackOff   11         37m
kube-system   heapster-5b9b6b6597-kbzk8               1/1     Running            0          37m
kube-system   kube-flannel-ds-amd64-87fs9             1/1     Running            0          38m
kube-system   kubernetes-dashboard-76479d66bb-v4z5s   0/1     CrashLoopBackOff   12         37m
kube-system   metrics-server-79558444c6-2n4w7         0/1     CrashLoopBackOff   12         37m

[root@localhost ~]# kubectl get svc --all-namespaces
NAMESPACE     NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes             ClusterIP   10.68.0.1       <none>        443/TCP                  38m
kube-system   heapster               ClusterIP   10.68.83.108    <none>        80/TCP                   37m
kube-system   kube-dns               ClusterIP   10.68.0.2       <none>        53/UDP,53/TCP,9153/TCP   38m
kube-system   kubernetes-dashboard   NodePort    10.68.89.197    <none>        443:35835/TCP            37m
kube-system   metrics-server         ClusterIP   10.68.156.125   <none>        443/TCP                  37m


```


```bash
[root@localhost ~]# docker images
REPOSITORY                                          TAG                 IMAGE ID            CREATED             SIZE
coredns/coredns                                     1.4.0               a9e015907f63        6 weeks ago         42.6MB
jmgao1983/flannel                                   v0.11.0-amd64       ff281650a721        2 months ago        52.6MB
dustise/flaskapp                                    latest              2f3e1e2fc0cc        4 months ago        196MB
mirrorgooglecontainers/kubernetes-dashboard-amd64   v1.10.1             f9aed6605b81        4 months ago        122MB
coredns/coredns                                     1.2.6               f59dcacceff4        5 months ago        40MB
mirrorgooglecontainers/metrics-server-amd64         v0.3.1              61a0c90da56e        7 months ago        40.8MB
mirrorgooglecontainers/heapster-amd64               v1.5.4              72d68eecf40c        8 months ago        75.3MB
mirrorgooglecontainers/pause-amd64                  3.1                 da86e6ba6ca1        16 months ago       742kB

```
## step 2 --> install istio


download istio from https://github.com/istio/istio/releases/tag/1.1.2

```bash
tar xzvf istio-1.1.2-linux.tar.gz
cd istio-1.1.2
[root@localhost istio-1.1.2]# ls
bin  install  istio.VERSION  LICENSE  README.md  samples  tools
[root@localhost istio-1.1.2]# which kubectl
/opt/kube/bin/kubectl
[root@localhost istio-1.1.2]# cd bin/
[root@localhost bin]# ls
istioctl
[root@localhost bin]# cp istioctl /opt/kube/bin/
[root@localhost bin]# istioctl version
version.BuildInfo{Version:"1.1.2", GitRevision:"2b1331886076df103179e3da5dc9077fed59c989", User:"root", Host:"35adf5bb-5570-11e9-b00d-0a580a2c0205", GolangVersion:"go1.10.4", DockerHub:"docker.io/istio", BuildStatus:"Clean", GitTag:"1.1.1"}
```

## Step 3 --> configuration

reference : https://istio.io/docs/setup/kubernetes/install/kubernetes/

```bash
[root@localhost ~]# scp chenm@192.168.56.1:/home/chenm/tools/monitoring-tools.tar .
Password:
monitoring-tools.tar                                                                        100%  687MB 120.3MB/s   00:05    
[root@localhost ~]# scp chenm@192.168.56.1:/home/chenm/tools/istio.tar .
Password:
istio.tar                                                                                   100% 1492MB 110.4MB/s   00:13    
[root@localhost ~]# docker load < istio.tar
b0dad1646e69: Loading layer [==================================================>]    258kB/258kB
38ed19b27fe1: Loading layer [==================================================>]  2.048kB/2.048kB
772f7d8f3bab: Loading layer [==================================================>]  57.44MB/57.44MB
58f80fb5fa76: Loading layer [==================================================>]  6.155MB/6.155MB
Loaded image: istio/citadel:1.1.2
191da41fae95: Loading layer [==================================================>]  52.33MB/52.33MB
58f80fb5fa76: Loading layer [==================================================>]  6.155MB/6.155MB
Loaded image: istio/sidecar_injector:1.1.2
ad495fa535c7: Loading layer [==================================================>]  79.85MB/79.85MB
58f80fb5fa76: Loading layer [==================================================>]  6.155MB/6.155MB
Loaded image: istio/mixer:1.1.2
aa54c2bc1229: Loading layer [==================================================>]  121.6MB/121.6MB
7dd604ffa87f: Loading layer [==================================================>]  15.87kB/15.87kB
2f0d1e8214b2: Loading layer [==================================================>]  11.78kB/11.78kB
297fd071ca2f: Loading layer [==================================================>]  3.072kB/3.072kB
0003b9d50594: Loading layer [==================================================>]  142.2MB/142.2MB
25a6ed6649fb: Loading layer [==================================================>]   55.4MB/55.4MB
efd94f7b955c: Loading layer [==================================================>]   55.4MB/55.4MB
58f80fb5fa76: Loading layer [==================================================>]  6.155MB/6.155MB
Loaded image: istio/kubectl:1.1.2
def70ac2c3e4: Loading layer [==================================================>]  78.97MB/78.97MB
58f80fb5fa76: Loading layer [==================================================>]  6.155MB/6.155MB
Loaded image: istio/galley:1.1.2
0314be9edf00: Loading layer [==================================================>]   1.36MB/1.36MB
a22ec1315073: Loading layer [==================================================>]  2.644MB/2.644MB
99765b91cbfd: Loading layer [==================================================>]  71.17MB/71.17MB
c03996727d71: Loading layer [==================================================>]     45MB/45MB
f085c16f89c3: Loading layer [==================================================>]  3.584kB/3.584kB
a3c5e59b406a: Loading layer [==================================================>]   12.8kB/12.8kB
ffc240f60e27: Loading layer [==================================================>]  27.65kB/27.65kB
d5b500279dd1: Loading layer [==================================================>]  3.072kB/3.072kB
e1e38dc3bc80: Loading layer [==================================================>]   5.12kB/5.12kB
Loaded image: prom/prometheus:v2.3.1
8823818c4748: Loading layer [==================================================>]    119MB/119MB
4a7a5ec0f29e: Loading layer [==================================================>]  15.87kB/15.87kB
87a2d0000622: Loading layer [==================================================>]  14.85kB/14.85kB
07663827a77f: Loading layer [==================================================>]  5.632kB/5.632kB
d7232280c8c4: Loading layer [==================================================>]  3.072kB/3.072kB
f042240fe1f6: Loading layer [==================================================>]  137.2MB/137.2MB
ca118579e51b: Loading layer [==================================================>]  24.92MB/24.92MB
633423ad9761: Loading layer [==================================================>]  38.71MB/38.71MB
9f5f41fbc96c: Loading layer [==================================================>]  15.36kB/15.36kB
4aff1756d8f5: Loading layer [==================================================>]  11.78kB/11.78kB
297f9e4727cf: Loading layer [==================================================>]  10.75kB/10.75kB
d3fd1679cf75: Loading layer [==================================================>]  23.04kB/23.04kB
26a00971e464: Loading layer [==================================================>]  11.78kB/11.78kB
a89b2a6bdafe: Loading layer [==================================================>]  4.096kB/4.096kB
ed275cf76620: Loading layer [==================================================>]  5.632kB/5.632kB
f1005343cfad: Loading layer [==================================================>]    258kB/258kB
fd71237f7616: Loading layer [==================================================>]  63.63MB/63.63MB
cae5fbcc8bec: Loading layer [==================================================>]  476.7kB/476.7kB
Loaded image: istio/proxyv2:1.1.2
a1791b71c0e6: Loading layer [==================================================>]  29.28MB/29.28MB
c7acd669fe1e: Loading layer [==================================================>]  23.04kB/23.04kB
58f80fb5fa76: Loading layer [==================================================>]  6.155MB/6.155MB
Loaded image: istio/proxy_init:1.1.2
0afafb5c0ff3: Loading layer [==================================================>]  67.87MB/67.87MB
a9ceabdaa229: Loading layer [==================================================>]  217.6kB/217.6kB
58f80fb5fa76: Loading layer [==================================================>]  6.155MB/6.155MB
Loaded image: istio/pilot:1.1.2
bcc97fbfc9e1: Loading layer [==================================================>]  208.2MB/208.2MB
c8c1a0db7b79: Loading layer [==================================================>]  2.048kB/2.048kB
c0348fea4ccf: Loading layer [==================================================>]  40.35MB/40.35MB
e8442c528257: Loading layer [==================================================>]  31.87MB/31.87MB
0823d0f24f24: Loading layer [==================================================>]  31.87MB/31.87MB
Loaded image: kiali/kiali:v0.14
[root@localhost ~]# docker load < monitoring-tools.tar
ef68f6734aa4: Loading layer [==================================================>]  58.44MB/58.44MB
8ffe95a4cd37: Loading layer [==================================================>]   2.56kB/2.56kB
0922e45a8a40: Loading layer [==================================================>]  30.49MB/30.49MB
99f926b425b7: Loading layer [==================================================>]  159.8MB/159.8MB
f5c397d3d761: Loading layer [==================================================>]    192kB/192kB
759d23924c4b: Loading layer [==================================================>]   5.12kB/5.12kB
Loaded image: grafana/grafana:5.4.0
Loaded image: prom/prometheus:v2.3.1
19f595203217: Loading layer [==================================================>]  37.33MB/37.33MB
00749d336c60: Loading layer [==================================================>]  3.072kB/3.072kB
Loaded image: jaegertracing/all-in-one:1.9
Loaded image: kiali/kiali:v0.14
0b97b1c81a32: Loading layer [==================================================>]  1.416MB/1.416MB
Loaded image: busybox:1.30.1
[root@localhost ~]# docker images
REPOSITORY                                          TAG                 IMAGE ID            CREATED             SIZE
istio/proxyv2                                       1.1.2               c7fb421f087e        2 weeks ago         378MB
busybox                                             1.30.1              af2f74c517aa        2 weeks ago         1.2MB
istio/sidecar_injector                              1.1.2               f78c16bc1d82        2 weeks ago         58.5MB
istio/proxy_init                                    1.1.2               95e1b811b656        2 weeks ago         152MB
istio/pilot                                         1.1.2               eda955df2131        2 weeks ago         332MB
istio/mixer                                         1.1.2               c7a6b574db2f        2 weeks ago         86.2MB
istio/kubectl                                       1.1.2               1a63aa2792f9        2 weeks ago         374MB
istio/galley                                        1.1.2               b1ef250d17a1        2 weeks ago         342MB
istio/citadel                                       1.1.2               69ca23784a8d        2 weeks ago         63.8MB
coredns/coredns                                     1.4.0               a9e015907f63        6 weeks ago         42.6MB
kiali/kiali                                         v0.14               e65674279630        2 months ago        304MB
jmgao1983/flannel                                   v0.11.0-amd64       ff281650a721        2 months ago        52.6MB
jaegertracing/all-in-one                            1.9                 dbcbb85b2777        2 months ago        37.3MB
dustise/flaskapp                                    latest              2f3e1e2fc0cc        4 months ago        196MB
mirrorgooglecontainers/kubernetes-dashboard-amd64   v1.10.1             f9aed6605b81        4 months ago        122MB
grafana/grafana                                     5.4.0               591703edc9b7        4 months ago        243MB
coredns/coredns                                     1.2.6               f59dcacceff4        5 months ago        40MB
dustise/sleep                                       latest              b3b59106864d        5 months ago        117MB
mirrorgooglecontainers/metrics-server-amd64         v0.3.1              61a0c90da56e        7 months ago        40.8MB
mirrorgooglecontainers/heapster-amd64               v1.5.4              72d68eecf40c        8 months ago        75.3MB
prom/prometheus                                     v2.3.1              b82ef1f3aa07        10 months ago       119MB
mirrorgooglecontainers/pause-amd64                  3.1                 da86e6ba6ca1        16 months ago       742kB

```

installation

```bash
[root@localhost ~]# cd istio-1.1.2/
[root@localhost istio-1.1.2]# ls
bin  install  istio.VERSION  LICENSE  README.md  samples  tools
[root@localhost istio-1.1.2]# for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl apply -f $i; done
customresourcedefinition.apiextensions.k8s.io/virtualservices.networking.istio.io created
customresourcedefinition.apiextensions.k8s.io/destinationrules.networking.istio.io created
customresourcedefinition.apiextensions.k8s.io/serviceentries.networking.istio.io created
customresourcedefinition.apiextensions.k8s.io/gateways.networking.istio.io created
customresourcedefinition.apiextensions.k8s.io/envoyfilters.networking.istio.io created
customresourcedefinition.apiextensions.k8s.io/clusterrbacconfigs.rbac.istio.io created
customresourcedefinition.apiextensions.k8s.io/policies.authentication.istio.io created
customresourcedefinition.apiextensions.k8s.io/meshpolicies.authentication.istio.io created
customresourcedefinition.apiextensions.k8s.io/httpapispecbindings.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/httpapispecs.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/quotaspecbindings.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/quotaspecs.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/rules.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/attributemanifests.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/bypasses.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/circonuses.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/deniers.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/fluentds.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/kubernetesenvs.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/listcheckers.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/memquotas.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/noops.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/opas.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/prometheuses.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/rbacs.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/redisquotas.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/signalfxs.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/solarwindses.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/stackdrivers.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/statsds.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/stdios.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/apikeys.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/authorizations.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/checknothings.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/kuberneteses.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/listentries.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/logentries.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/edges.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/metrics.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/quotas.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/reportnothings.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/tracespans.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/rbacconfigs.rbac.istio.io created
customresourcedefinition.apiextensions.k8s.io/serviceroles.rbac.istio.io created
customresourcedefinition.apiextensions.k8s.io/servicerolebindings.rbac.istio.io created
customresourcedefinition.apiextensions.k8s.io/adapters.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/instances.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/templates.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/handlers.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/cloudwatches.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/dogstatsds.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/sidecars.networking.istio.io created
customresourcedefinition.apiextensions.k8s.io/zipkins.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/clusterissuers.certmanager.k8s.io created
customresourcedefinition.apiextensions.k8s.io/issuers.certmanager.k8s.io created
customresourcedefinition.apiextensions.k8s.io/certificates.certmanager.k8s.io created
customresourcedefinition.apiextensions.k8s.io/orders.certmanager.k8s.io created
customresourcedefinition.apiextensions.k8s.io/challenges.certmanager.k8s.io created
[root@localhost istio-1.1.2]# kubectl apply -f install/kubernetes/istio-demo.yaml
namespace/istio-system created
customresourcedefinition.apiextensions.k8s.io/virtualservices.networking.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/destinationrules.networking.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/serviceentries.networking.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/gateways.networking.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/envoyfilters.networking.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/clusterrbacconfigs.rbac.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/policies.authentication.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/meshpolicies.authentication.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/httpapispecbindings.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/httpapispecs.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/quotaspecbindings.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/quotaspecs.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/rules.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/attributemanifests.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/bypasses.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/circonuses.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/deniers.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/fluentds.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/kubernetesenvs.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/listcheckers.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/memquotas.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/noops.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/opas.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/prometheuses.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/rbacs.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/redisquotas.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/signalfxs.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/solarwindses.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/stackdrivers.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/statsds.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/stdios.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/apikeys.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/authorizations.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/checknothings.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/kuberneteses.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/listentries.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/logentries.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/edges.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/metrics.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/quotas.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/reportnothings.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/tracespans.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/rbacconfigs.rbac.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/serviceroles.rbac.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/servicerolebindings.rbac.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/adapters.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/instances.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/templates.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/handlers.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/cloudwatches.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/dogstatsds.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/sidecars.networking.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/zipkins.config.istio.io unchanged
customresourcedefinition.apiextensions.k8s.io/clusterissuers.certmanager.k8s.io unchanged
customresourcedefinition.apiextensions.k8s.io/issuers.certmanager.k8s.io unchanged
customresourcedefinition.apiextensions.k8s.io/orders.certmanager.k8s.io unchanged
customresourcedefinition.apiextensions.k8s.io/challenges.certmanager.k8s.io unchanged
secret/kiali created
configmap/istio-galley-configuration created
configmap/istio-grafana-custom-resources created
configmap/istio-grafana-configuration-dashboards-galley-dashboard created
configmap/istio-grafana-configuration-dashboards-istio-mesh-dashboard created
configmap/istio-grafana-configuration-dashboards-istio-performance-dashboard created
configmap/istio-grafana-configuration-dashboards-istio-service-dashboard created
configmap/istio-grafana-configuration-dashboards-istio-workload-dashboard created
configmap/istio-grafana-configuration-dashboards-mixer-dashboard created
configmap/istio-grafana-configuration-dashboards-pilot-dashboard created
configmap/istio-grafana created
configmap/kiali created
configmap/prometheus created
configmap/istio-security-custom-resources created
configmap/istio created
configmap/istio-sidecar-injector created
serviceaccount/istio-galley-service-account created
serviceaccount/istio-egressgateway-service-account created
serviceaccount/istio-ingressgateway-service-account created
serviceaccount/istio-grafana-post-install-account created
clusterrole.rbac.authorization.k8s.io/istio-grafana-post-install-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-grafana-post-install-role-binding-istio-system created
job.batch/istio-grafana-post-install-1.1.2 created
serviceaccount/kiali-service-account created
serviceaccount/istio-mixer-service-account created
serviceaccount/istio-pilot-service-account created
serviceaccount/prometheus created
serviceaccount/istio-cleanup-secrets-service-account created
clusterrole.rbac.authorization.k8s.io/istio-cleanup-secrets-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-cleanup-secrets-istio-system created
job.batch/istio-cleanup-secrets-1.1.2 created
serviceaccount/istio-security-post-install-account created
clusterrole.rbac.authorization.k8s.io/istio-security-post-install-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-security-post-install-role-binding-istio-system created
job.batch/istio-security-post-install-1.1.2 created
serviceaccount/istio-citadel-service-account created
serviceaccount/istio-sidecar-injector-service-account created
serviceaccount/istio-multi created
clusterrole.rbac.authorization.k8s.io/istio-galley-istio-system created
clusterrole.rbac.authorization.k8s.io/istio-egressgateway-istio-system created
clusterrole.rbac.authorization.k8s.io/istio-ingressgateway-istio-system created
clusterrole.rbac.authorization.k8s.io/kiali created
clusterrole.rbac.authorization.k8s.io/istio-mixer-istio-system created
clusterrole.rbac.authorization.k8s.io/istio-pilot-istio-system created
clusterrole.rbac.authorization.k8s.io/prometheus-istio-system created
clusterrole.rbac.authorization.k8s.io/istio-citadel-istio-system created
clusterrole.rbac.authorization.k8s.io/istio-sidecar-injector-istio-system created
clusterrole.rbac.authorization.k8s.io/istio-reader created
clusterrolebinding.rbac.authorization.k8s.io/istio-galley-admin-role-binding-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-egressgateway-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-ingressgateway-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-kiali-admin-role-binding-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-mixer-admin-role-binding-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-pilot-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-citadel-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-sidecar-injector-admin-role-binding-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-multi created
role.rbac.authorization.k8s.io/istio-ingressgateway-sds created
rolebinding.rbac.authorization.k8s.io/istio-ingressgateway-sds created
service/istio-galley created
service/istio-egressgateway created
service/istio-ingressgateway created
service/grafana created
service/kiali created
service/istio-policy created
service/istio-telemetry created
service/istio-pilot created
service/prometheus created
service/istio-citadel created
service/istio-sidecar-injector created
deployment.extensions/istio-galley created
deployment.extensions/istio-egressgateway created
deployment.extensions/istio-ingressgateway created
deployment.extensions/grafana created
deployment.extensions/kiali created
deployment.extensions/istio-policy created
deployment.extensions/istio-telemetry created
deployment.extensions/istio-pilot created
deployment.extensions/prometheus created
deployment.extensions/istio-citadel created
deployment.extensions/istio-sidecar-injector created
deployment.extensions/istio-tracing created
horizontalpodautoscaler.autoscaling/istio-egressgateway created
horizontalpodautoscaler.autoscaling/istio-ingressgateway created
horizontalpodautoscaler.autoscaling/istio-policy created
horizontalpodautoscaler.autoscaling/istio-telemetry created
horizontalpodautoscaler.autoscaling/istio-pilot created
service/jaeger-query created
service/jaeger-collector created
service/jaeger-agent created
service/zipkin created
service/tracing created
mutatingwebhookconfiguration.admissionregistration.k8s.io/istio-sidecar-injector created
poddisruptionbudget.policy/istio-galley created
poddisruptionbudget.policy/istio-egressgateway created
poddisruptionbudget.policy/istio-ingressgateway created
poddisruptionbudget.policy/istio-policy created
poddisruptionbudget.policy/istio-telemetry created
poddisruptionbudget.policy/istio-pilot created
attributemanifest.config.istio.io/istioproxy created
attributemanifest.config.istio.io/kubernetes created
handler.config.istio.io/stdio created
logentry.config.istio.io/accesslog created
logentry.config.istio.io/tcpaccesslog created
rule.config.istio.io/stdio created
rule.config.istio.io/stdiotcp created
metric.config.istio.io/requestcount created
metric.config.istio.io/requestduration created
metric.config.istio.io/requestsize created
metric.config.istio.io/responsesize created
metric.config.istio.io/tcpbytesent created
metric.config.istio.io/tcpbytereceived created
metric.config.istio.io/tcpconnectionsopened created
metric.config.istio.io/tcpconnectionsclosed created
handler.config.istio.io/prometheus created
rule.config.istio.io/promhttp created
rule.config.istio.io/promtcp created
rule.config.istio.io/promtcpconnectionopen created
rule.config.istio.io/promtcpconnectionclosed created
handler.config.istio.io/kubernetesenv created
rule.config.istio.io/kubeattrgenrulerule created
rule.config.istio.io/tcpkubeattrgenrulerule created
kubernetes.config.istio.io/attributes created
destinationrule.networking.istio.io/istio-policy created
destinationrule.networking.istio.io/istio-telemetry created
[root@localhost istio-1.1.2]# kubectl get svc -n istio-system
NAME                     TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                                                                                      AGE
grafana                  ClusterIP      10.68.105.114   <none>        3000/TCP                                                                                                                                     13s
istio-citadel            ClusterIP      10.68.95.125    <none>        8060/TCP,15014/TCP                                                                                                                           13s
istio-egressgateway      ClusterIP      10.68.170.67    <none>        80/TCP,443/TCP,15443/TCP                                                                                                                     13s
istio-galley             ClusterIP      10.68.114.31    <none>        443/TCP,15014/TCP,9901/TCP                                                                                                                   13s
istio-ingressgateway     LoadBalancer   10.68.225.137   <pending>     80:31380/TCP,443:31390/TCP,31400:31400/TCP,15029:29327/TCP,15030:30704/TCP,15031:28700/TCP,15032:32411/TCP,15443:35261/TCP,15020:32727/TCP   13s
istio-pilot              ClusterIP      10.68.48.117    <none>        15010/TCP,15011/TCP,8080/TCP,15014/TCP                                                                                                       13s
istio-policy             ClusterIP      10.68.143.95    <none>        9091/TCP,15004/TCP,15014/TCP                                                                                                                 13s
istio-sidecar-injector   ClusterIP      10.68.68.119    <none>        443/TCP                                                                                                                                      13s
istio-telemetry          ClusterIP      10.68.55.248    <none>        9091/TCP,15004/TCP,15014/TCP,42422/TCP                                                                                                       13s
jaeger-agent             ClusterIP      None            <none>        5775/UDP,6831/UDP,6832/UDP                                                                                                                   13s
jaeger-collector         ClusterIP      10.68.170.122   <none>        14267/TCP,14268/TCP                                                                                                                          13s
jaeger-query             ClusterIP      10.68.194.230   <none>        16686/TCP                                                                                                                                    13s
kiali                    ClusterIP      10.68.246.142   <none>        20001/TCP                                                                                                                                    13s
prometheus               ClusterIP      10.68.141.165   <none>        9090/TCP                                                                                                                                     13s
tracing                  ClusterIP      10.68.70.170    <none>        80/TCP                                                                                                                                       13s
zipkin                   ClusterIP      10.68.2.129     <none>        9411/TCP                                                                                                                                     13s
[root@localhost istio-1.1.2]# kubectl get pods -n istio-system
NAME                                      READY   STATUS              RESTARTS   AGE
grafana-57586c685b-7dbp2                  0/1     ContainerCreating   0          28s
istio-citadel-7579f8fbb9-hm8dr            0/1     Pending             0          28s
istio-cleanup-secrets-1.1.2-n845w         0/1     ContainerCreating   0          29s
istio-egressgateway-568dcfdd-j78hq        0/1     ContainerCreating   0          28s
istio-galley-79d4c5d9f7-j6nll             0/1     ContainerCreating   0          28s
istio-grafana-post-install-1.1.2-4vcth    0/1     ContainerCreating   0          29s
istio-ingressgateway-5fbcf4488f-4px8l     0/1     ContainerCreating   0          28s
istio-pilot-bd75b9c7d-vqq7g               0/2     ContainerCreating   0          28s
istio-policy-655f7f85b6-btxzd             0/2     ContainerCreating   0          28s
istio-security-post-install-1.1.2-wbvkh   0/1     Error               1          29s
istio-sidecar-injector-57f445c786-jbfr5   0/1     Pending             0          28s
istio-telemetry-7ff4c757b4-h7dn5          0/2     ContainerCreating   0          28s
istio-tracing-656f9fc99c-g9h27            0/1     Running             0          27s
kiali-69d6978b45-j9pm5                    0/1     ContainerCreating   0          28s
prometheus-66c9f5694-s88gm                0/1     Pending             0          28s

```

```bash
for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl apply -f $i; done

kubectl apply -f install/kubernetes/istio-demo.yaml

or

kubectl apply -f install/kubernetes/istio-demo-auth.yaml


```

varify

```bash
$ kubectl get svc -n istio-system
NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                                                                                                                   AGE
grafana                  ClusterIP      172.21.211.123   <none>          3000/TCP                                                                                                                  2m
istio-citadel            ClusterIP      172.21.177.222   <none>          8060/TCP,9093/TCP                                                                                                         2m
istio-egressgateway      ClusterIP      172.21.113.24    <none>          80/TCP,443/TCP                                                                                                            2m
istio-galley             ClusterIP      172.21.132.247   <none>          443/TCP,9093/TCP                                                                                                          2m
istio-ingressgateway     LoadBalancer   172.21.144.254   52.116.22.242   80:31380/TCP,443:31390/TCP,31400:31400/TCP,15011:32081/TCP,8060:31695/TCP,853:31235/TCP,15030:32717/TCP,15031:32054/TCP   2m
istio-pilot              ClusterIP      172.21.105.205   <none>          15010/TCP,15011/TCP,8080/TCP,9093/TCP                                                                                     2m
istio-policy             ClusterIP      172.21.14.236    <none>          9091/TCP,15004/TCP,9093/TCP                                                                                               2m
istio-sidecar-injector   ClusterIP      172.21.155.47    <none>          443/TCP                                                                                                                   2m
istio-telemetry          ClusterIP      172.21.196.79    <none>          9091/TCP,15004/TCP,9093/TCP,42422/TCP                                                                                     2m
jaeger-agent             ClusterIP      None             <none>          5775/UDP,6831/UDP,6832/UDP                                                                                                2m
jaeger-collector         ClusterIP      172.21.135.51    <none>          14267/TCP,14268/TCP                                                                                                       2m
jaeger-query             ClusterIP      172.21.26.187    <none>          16686/TCP                                                                                                                 2m
kiali                    ClusterIP      172.21.155.201   <none>          20001/TCP                                                                                                                 2m
prometheus               ClusterIP      172.21.63.159    <none>          9090/TCP                                                                                                                  2m
tracing                  ClusterIP      172.21.2.245     <none>          80/TCP                                                                                                                    2m
zipkin                   ClusterIP      172.21.182.245   <none>          9411/TCP  

$ kubectl get pods -n istio-system
NAME                                                           READY   STATUS      RESTARTS   AGE
grafana-f8467cc6-rbjlg                                         1/1     Running     0          1m
istio-citadel-78df5b548f-g5cpw                                 1/1     Running     0          1m
istio-cleanup-secrets-release-1.1-20190308-09-16-8s2mp         0/1     Completed   0          2m
istio-egressgateway-78569df5c4-zwtb5                           1/1     Running     0          1m
istio-galley-74d5f764fc-q7nrk                                  1/1     Running     0          1m
istio-grafana-post-install-release-1.1-20190308-09-16-2p7m5    0/1     Completed   0          2m
istio-ingressgateway-7ddcfd665c-dmtqz                          1/1     Running     0          1m
istio-pilot-f479bbf5c-qwr28                                    2/2     Running     0          1m
istio-policy-6fccc5c868-xhblv                                  2/2     Running     2          1m
istio-security-post-install-release-1.1-20190308-09-16-bmfs4   0/1     Completed   0          2m
istio-sidecar-injector-78499d85b8-x44m6                        1/1     Running     0          1m
istio-telemetry-78b96c6cb6-ldm9q                               2/2     Running     2          1m
istio-tracing-69b5f778b7-s2zvw                                 1/1     Running     0          1m
kiali-99f7467dc-6rvwp                                          1/1     Running     0          1m
prometheus-67cdb66cbb-9w2hm                                    1/1     Running     0          1m

```

###Deploy your application
```bash
#When you deploy your application using kubectl apply, the Istio sidecar injector will automatically inject Envoy containers into your application pods if they are started in namespaces labeled with istio-injection=enabled:
$ kubectl label namespace <namespace> istio-injection=enabled
$ kubectl create -n <namespace> -f <your-app-spec>.yaml


#In namespaces without the istio-injection label, you can use istioctl kube-inject to manually inject Envoy containers in your application pods before deploying them
$ istioctl kube-inject -f <your-app-spec>.yaml | kubectl apply -f -

```

###Uninstall

```bash
kubectl delete -f install/kubernetes/istio-demo-auth.yaml

or

kubectl delete -f install/kubernetes/istio-demo.yaml

 for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl delete -f $i; done

```
