# QoS(Quality of Service)服务质量
Kubernetes 的资源管理与调度，从资源模型说起。

Pod 是最小的原子调度单位，所有跟调度和资源管理相关的属性，都是 Pod 对象属性的字段。其中最重要的是 Pod 和 CPU 配置。其中，CPU 属于可压缩资源，内存属于不可压缩资源。当可压缩资源不足时，Pod 会饥饿；当不可压缩资源不足时，Pod 就会因为 OOM 被内核杀掉。

Pod ，即容器组，由多个容器组成，其 CPU 和内存配额在 Container 上定义，其 Container 配置的累加值即为 Pod 的配额。

## limits 和 requests

- requests：kube-scheduler 只会按照 requests 的值进行计算。
- limits：kubelet 则会按照 limits 的值来进行设置 Cgroups 限制.


## QoS 模型
- Guaranteed： 同时设置 requests 和 limits，并且 requests 和 limit 值相等。优势一是在资源不足 Eviction 发生时，最后被删除；并且删除的是 Pod 资源用量超过 limits 时才会被删除；优势二是该模型与 docker cpuset 的方式绑定 CPU 核，避免频繁的上下午文切换，性能会得到大幅提升。
- Burstable：不满足 Guaranteed 条件， 但至少有一个 Container 设置了 requests
- BestEffort：没有设置 requests 和 limits。

## Eviction 

```bash
kubelet --eviction-hard=imagefs.available<10%,memory.available<500Mi,nodefs.available<5%,nodefs.inodesFree<5% --eviction-soft=imagefs.available<30%,nodefs.available<10% \
--eviction-soft-grace-period=imagefs.available=2m,nodefs.available=2m \
--eviction-max-pod-grace-period=600
```

两种模式：
- soft: 如 `eviction-soft-grace-period=imagefs.available=2m`  eviction 会在阈值达到 2 分钟后才会开始
- hard：evivtion 会立即开始。

**eviction 计算原理**： 将 Cgroups （limits属性）设置的值和 cAdvisor 监控的数据相比较。


最佳实践：DaemonSet 的 Pod 都设置为 Guaranteed， 避免进入“回收->创建->回收->创建”的“死循环”。