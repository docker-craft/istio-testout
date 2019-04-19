## Istio installation guide

```bash

#install
curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.1.3 sh -

cd istio-1.1.3

for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl apply -f $i; done

kubectl apply -f install/kubernetes/istio-demo.yaml

#Verifying the installation
kubectl get svc -n istio-system

kubectl get pods -n istio-system

#uninstall
kubectl delete -f install/kubernetes/istio-demo.yaml

for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl delete -f $i; done

```