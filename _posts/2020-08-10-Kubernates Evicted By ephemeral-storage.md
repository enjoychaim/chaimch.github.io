---
title: Kubernates Evicted By ephemeral-storage
tags: kubernates evicted
key: "kubernates:evicted:2020-08-10"
---

## 报错如下

```
  Type     Reason     Age                From                                     Message
  ----     ------     ----               ----                                     -------
  Warning  Unhealthy  52m                kubelet, ip-172-30-200-182.ec2.internal  Readiness probe failed: Get http://172.30.206.82:10902/-/ready: dial tcp 172.30.206.82:10902: connect: connection refused
  Normal   Pulled     52m (x2 over 22h)  kubelet, ip-172-30-200-182.ec2.internal  Container image "quay.io/thanos/thanos:v0.14.0" already present on machine
  Normal   Created    52m (x2 over 22h)  kubelet, ip-172-30-200-182.ec2.internal  Created container thanos
  Normal   Started    52m (x2 over 22h)  kubelet, ip-172-30-200-182.ec2.internal  Started container thanos
  Warning  Evicted    28m                kubelet, ip-172-30-200-182.ec2.internal  The node was low on resource: ephemeral-storage. Container prometheus was using 20Ki, which exceeds its request of 0. Container thanos was using 16Ki, which exceeds its request of 0.
  Normal   Killing    28m                kubelet, ip-172-30-200-182.ec2.internal  Stopping container prometheus
  Normal   Killing    28m                kubelet, ip-172-30-200-182.ec2.internal  Stopping container thanos
```



## 解决办法

```
        resources:
          limits:
            cpu: "1500m"
            memory: "14.5Gi"
            ephemeral-storage: "10Gi"
          requests:
            cpu: "1500m"
            memory: "14.5Gi"
            ephemeral-storage: "10Gi"
```

