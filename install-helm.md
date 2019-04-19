```bash

[root@localhost ~]# git clone https://github.com/docker-craft/kubernetes-course.git

[root@localhost ~]# cd kubernetes-course/helm

[root@localhost helm]# ls
demo-chart      jenkins               README.md
helm-rbac.yaml  put-bucket-policy.sh  setup-s3-helm-repo.sh
[root@localhost helm]# cat README.md 
# Helm

## Install helm
```
wget https://storage.googleapis.com/kubernetes-helm/helm-v2.11.0-linux-amd64.tar.gz
tar -xzvf helm-v2.11.0-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm
```

## Initialize helm

```
kubectl create -f helm-rbac.yaml
helm init --service-account tiller
```

## Setup S3 helm repository
Make sure to set the default region in setup-s3-helm-repo.sh
```
./setup-s3-helm-repo.sh
```

## Package and push demo-chart

```
export AWS_REGION=yourregion # if not set in ~/.aws
helm package demo-chart
helm s3 push ./demo-chart-0.0.1.tgz my-charts
```
[root@localhost helm]# wget https://storage.googleapis.com/kubernetes-helm/helm-v2.13.1-linux-amd64.tar.gz
--2019-04-19 19:27:06--  https://storage.googleapis.com/kubernetes-helm/helm-v2.13.1-linux-amd64.tar.gz
Resolving storage.googleapis.com (storage.googleapis.com)... 172.217.24.48, 2404:6800:4005:803::2010
Connecting to storage.googleapis.com (storage.googleapis.com)|172.217.24.48|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 22949819 (22M) [application/x-tar]
Saving to: ‘helm-v2.13.1-linux-amd64.tar.gz’

100%[======================================>] 22,949,819  12.2MB/s   in 1.8s   

2019-04-19 19:27:08 (12.2 MB/s) - ‘helm-v2.13.1-linux-amd64.tar.gz’ saved [22949819/22949819]

[root@localhost helm]# pwd
/root/kubernetes-course/helm
[root@localhost helm]# ls


[root@localhost helm]# tar -zxvf helm-v2.13.1-linux-amd64.tar.gz
linux-amd64/
linux-amd64/LICENSE
linux-amd64/tiller
linux-amd64/helm
linux-amd64/README.md
[root@localhost helm]# which kubectl
/opt/kube/bin/kubectl

[root@localhost helm]# cp linux-amd64/helm /opt/kube/bin/
[root@localhost helm]# kubectl create -f helm-rbac.yaml
serviceaccount/tiller created
clusterrolebinding.rbac.authorization.k8s.io/tiller created
[root@localhost helm]# helm init --service-account tiller
Creating /root/.helm 
Creating /root/.helm/repository 
Creating /root/.helm/repository/cache 
Creating /root/.helm/repository/local 
Creating /root/.helm/plugins 
Creating /root/.helm/starters 
Creating /root/.helm/cache/archive 
Creating /root/.helm/repository/repositories.yaml 
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com 
Adding local repo with URL: http://127.0.0.1:8879/charts 
$HELM_HOME has been configured at /root/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
Happy Helming!
[root@localhost helm]# kubectl get svc -n kube-system 
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
heapster               ClusterIP   10.68.148.240   <none>        80/TCP                   5h18m
kube-dns               ClusterIP   10.68.0.2       <none>        53/UDP,53/TCP,9153/TCP   5h18m
kubernetes-dashboard   NodePort    10.68.156.114   <none>        443:31586/TCP            5h18m
metrics-server         ClusterIP   10.68.6.66      <none>        443/TCP                  5h18m
tiller-deploy          ClusterIP   10.68.178.120   <none>        44134/TCP                71s


```