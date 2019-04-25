### MemQuata

```bash

[root@tpl-centos mem-quota]# cat memquota-handler.yaml
apiVersion: "config.istio.io/v1alpha2"
kind: memquota
metadata:
  name: handler
spec:
  quotas:
    - name: dest-quota.quota.default
      maxAmount: 20
      validDuration: 10s
      overrides:
        - dimensions:
            destination: httpbin
          maxAmount: 1
          validDuration: 5s
[root@tpl-centos mem-quota]# cat quota-instance.yaml
apiVersion: "config.istio.io/v1alpha2"
kind: quota
metadata:
  name: dest-quota
spec:
  dimensions:
    destination: destination.labels["app"] | destination.service | "unknown"

[root@tpl-centos mem-quota]# cat rule.yaml
apiVersion: config.istio.io/v1alpha2
kind: rule
metadata:
  name: quota
spec:
  actions:
    - handler: handler.memquota
      instances:
        - dest-quota.quota
        
[root@tpl-centos mem-quota]# cat quotaspec.yaml
apiVersion: config.istio.io/v1alpha2
kind: QuotaSpec
metadata:
  name: request-count
spec:
  rules:
    - quotas:
        - charge: "5"
          quota: dest-quota
[root@tpl-centos mem-quota]# cat spec-binding.yaml
apiVersion: config.istio.io/v1alpha2
kind: QuotaSpecBinding
metadata:
  name: spec-sleep
spec:
  quotaSpecs:
  - name: request-count
    namespace: default
  services:
    - name: httpbin
      namespace: default

[root@tpl-centos mem-quota]#

```