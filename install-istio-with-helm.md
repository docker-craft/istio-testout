```bash
$ kubectl apply -f install/kubernetes/helm/helm-service-account.yaml

$ helm init --service-account tiller

$ helm install install/kubernetes/helm/istio-init --name istio-init --namespace istio-system

$ kubectl get crds | grep 'istio.io\|certmanager.k8s.io' | wc -l
53

$ helm install install/kubernetes/helm/istio --name istio --namespace istio-system

kubectl get svc -n istio-system
kubectl get pods -n istio-system

$ helm template install/kubernetes/helm/istio --name istio --namespace istio-system | kubectl delete -f -
$ kubectl delete namespace istio-system

$ kubectl delete -f install/kubernetes/helm/istio-init/files
```


●