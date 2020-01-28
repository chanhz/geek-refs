## 介绍
对象(Object)描述了 K8s 集群中的资源，这些对象可以在 yaml 文件中作为一种 API 类型来配置。

 
## 分类
资源对象可以分为以下几类：
* 资源对象<br>
  Pod、ReplicaSet、ReplicationController、Deployment、StatefulSet、DaemonSet、Job、CronJob、HorizontalPodAutoscaling、Node、Namespace、Service、Ingress、Label、CustomResourceDefinition
* 存储对象<br>
  Volume、PersistentVolume、Secret、ConfigMap
* 策略对象<br>
  SecurityContext、ResourceQuota、LimitRange
* 身份对象<br>
  ServiceAccount、Role、ClusterRole

## 理解这些对象

在 Kubernetes 系统中， 对象是持久化的条目，这些条目表示整个集群的状态。特别地，它们描述了如下（配置）信息：
 - 什么容器应用在运行以及在哪个 Node 上
 - 可以被应用使用的资源
 - 关于应用表现的策略，比如重启策略、升级策略和容器策略

Kubernetes 对象的创建、修改或删除，需要使用 [Kubernetes API](https://git.k8s.io/community/contributors/devel/api-conventions.md)。调用 API 有两种方式
1. 使用 `kubectl` 命令行接口命令行调用
2. 使用编程语言客户端库进行调用，如 [golang-client](https://github.com/kubernetes/client-go)、[Python-client](https://github.com/kubernetes-incubator/client-python)


## 对象的期望状态和实际状态
每个 Kubernetes 对象包含两个嵌套的对象字段：
- **spec**:描述对象的期望状态，不能为空
- **status**:描述对象的实际状态，由 Kubernetes 维护
它们负责管理对象的配置。

*eg:*
Kubernetes Deployment 对象能够表示运行在集群中的应用。当创建 Deployment 时，可能需要设置 Deployment 的 spec，指定该应用运行 3 个副本。Kubernetes 会读取 Deployment spec, 启动我们所期望的 3 个实例，并执行某些操作，更新状态与 spec 相匹配。若实例启动失败，Kubernetes 系统通过修正来响应 spec 和状态之间的不一致 —— 这种情况，启动一个新的实例来替换。


