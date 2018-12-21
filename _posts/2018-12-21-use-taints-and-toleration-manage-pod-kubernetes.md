---
layout: post
title:  "在Kubernetes中使用Taints和Toleration划分专用节点"
date:   2018-12-21 15:53:49
categories: kubernetes Docker
tags: kubernetes
---

#### 背景

在生产环境中，可能会遇到某些节点只为提供给专门的Pod运行。这里可以使用k8s提供的`容忍`和`污点`组合来解决这样的困境。

#### 概念

节点亲和性可以让pod调度到指定的一类节点，污点(taint)则相反,它让节点能排斥一类Pod。在k8s集群里面所有的非系统Pod对于带有污点的节点是不会调度过去的。

结合背景，Pods的服务其实想调度到指定的节点，然后这个节点不能运行其他的Pods。


#### 操作

下面创建一个Busybox服务，我希望Busybox只能调度到一类标记有busybox的节点，并且这类节点不允许调度其他的Pod

1. 首先我们给节点打上标签BusyBox，并且不允许其他节点调度

```bash
kubectl label node node-name  node-role=busybox

#添加
kubectl taint nodes node-name role=busybox:NoSchedule
#移除
kubectl taint nodes node-name role=NoSchedule-

``` 

2. 创建busybox YAML文件

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox
  labels:
    app: busybox
spec:
  replicas: 1 
  selector:
    matchLabels:
      app: busybox
 
  template:
    metadata:
      labels:
        app: busybox
    spec:
      tolerations:
      - key: "role"
        operator: "Equal"
        value: "busybox"
        effect: "NoSchedule"
        tolerationSeconds: 2000
      nodeSelector:
        node-role: busybox     
      containers:
      - image: busybox
        command:
          - sleep
          - "3600"
        imagePullPolicy: IfNotPresent
        name: busybox
        volumeMounts:
        - name: tz-config
          mountPath: /etc/localtime
      restartPolicy: Always
      volumes:
        - name: tz-config
          hostPath:
            path: /usr/share/zoneinfo/Asia/Shanghai  
```

相比较常用的yaml配置增加了下面一段`tolerations`和`nodeSelector`.`nodeSelector`是选择打有`busybox`标记的节点.`tolerations`是允许带有`role=busybox`污点的节点调度上去。`nodeSelector`是选择节点，`tolerations`是让节点排除`role=busybox`条件成立的Pods。

`tolerationSeconds` 是当 pod 需要被驱逐时，可以继续在 node 上运行的时间,单位秒。


```yaml
tolerations:
- key: "role"
  operator: "Equal"
  value: "busybox"
  effect: "NoSchedule"
  tolerationSeconds: 2000
nodeSelector:
  node-role: busybox 
``` 

**关于operator值**

operator的值为Exists，这时无需指定value
operator的值为Equal并且value相等

在官方资料里面，标记节点污点参数（对应`effect`）有三个可选值。

- NoSchedule 新的Pod如果没有查找到带有污点的节点就不调度，在节点已经存在的继续运行

- PreferNoSchedule 如果一个Pod没有声明容忍这个Taint，则系统会尽量避免把这个pod调度到这一节点上去，但不是强制的。节点上已经存在的pod继续运行

- NoExecute 定义pod的驱逐行为，以应对节点故障。NoExecute这个Taint效果对节点上正在运行的pod有以下影响：

没有设置Toleration的Pod会被立刻驱逐

配置了对应Toleration的pod，如果没有为tolerationSeconds赋值，则会一直留在该节点

配置了对应Toleration的pod且指定了tolerationSeconds值，则会在指定时间后驱逐



参考资料:

- [Advanced Scheduling and Taints and Tolerations](https://docs.okd.io/latest/admin_guide/scheduling/taints_tolerations.html)
- [Taints and Tolerations](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)
