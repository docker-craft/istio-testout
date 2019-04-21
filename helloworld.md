```bash

git clone https://github.com/docker-craft/kubernetes-course.git

cd kubernetes/istio

cat README.md

kubectl apply -f <(istioctl kube-inject -f helloworld.yaml)
kubectl apply -f helloworld-gw.yaml

kubectl edit pods istio-ingressgateway -n istio-system

curl http://10.68.243.83/hello

```