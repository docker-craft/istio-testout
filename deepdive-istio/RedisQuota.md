###

```bash

[root@tpl-centos redis-quota]# cat redis.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis
  labels:
   name: redis
spec:
  replicas: 1
  selector:
   name: redis
  template:
   metadata:
     labels:
       name: redis
   spec:
     containers:
     - name: redis
       image: redis
       ports:
       - containerPort: 6379
---
apiVersion: v1
kind: Service
metadata:
  name: redis
  labels:
   name: redis
spec:
  ports:
  - port: 6379
    targetPort: 6379
  selector:
    name: redis

[root@tpl-centos redis-quota]# cat quota-instance.yaml
apiVersion: "config.istio.io/v1alpha2"
kind: quota
metadata:
  name: dest-quota
spec:
  dimensions:
    destination: destination.labels["app"] | destination.service | "unknown"

[root@tpl-centos redis-quota]# cat redisquota.yaml
apiVersion: "config.istio.io/v1alpha2"
kind: redisquota
metadata:
  name: handler
spec:
  redisServerUrl: "redis.default:6379"
  quotas:
    - name: dest-quota.quota.default
      maxAmount: 20
      bucketDuration: 1s
      validDuration: 10s
      rateLimitAlgorithm: ROLLING_WINDOW
      overrides:
        - dimensions:
            destination: httpbin
          maxAmount: 1
[root@tpl-centos redis-quota]# cat quotaspec.yaml
apiVersion: config.istio.io/v1alpha2
kind: QuotaSpec
metadata:
  name: request-count
spec:
  rules:
    - quotas:
        - charge: "5"
          quota: dest-quota
[root@tpl-centos redis-quota]# cat spec-binding.yaml
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


```