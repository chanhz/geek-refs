## 介绍
Kubernetes 中内建了很多 controller（控制器），这些相当于一个状态机，用来控制Pod的具体状态和行为。
这是一个控制器列表：
- Deployment
- StatefulSet
- DaemonSet
- ReplicationController 和 ReplicaSet
- Job
- CronJob
- Horizontal Pod Autoscaling
- 准入控制器


## DaemonSet
容器化守护进程。
DaemonSet 的主要作用，是让你在 Kubernetes 集群里，运行一个 Daemon Pod。 所以，这个 Pod 有如下三个特征：
- 这个 Pod 运行在 Kubernetes 集群里的每一个节点（Node）上；
- 每个节点上只有一个这样的 Pod 实例；当有新的节点加入 Kubernetes 集群后，该 Pod 会自动地在新节点上被创建出来；
- 而当旧节点被删除后，它上面的 Pod 也相应地会被回收掉。


这个机制听起来很简单，但 Daemon Pod 的意义确实是非常重要的。我随便给你列举几个例子：

- 各种网络插件的 Agent 组件，都必须运行在每一个节点上，用来处理这个节点上的容器网络；
- 各种存储插件的 Agent 组件，也必须运行在每一个节点上，用来在这个节点上挂载远程存储目录，操作容器的 Volume 目录；
- 各种监控组件和日志组件，也必须运行在每一个节点上，负责这个节点上的监控信息和日志搜集。


DaemonSet 是一个非常简单的控制器。在它的控制循环中，只需要遍历所有节点，然后根据节点上是否有被管理 Pod 的情况，来决定是否要创建或者删除一个 Pod。在创建每个 Pod 的时候，DaemonSet 会自动给这个 Pod 加上一个 nodeAffinity，从而保证这个 Pod 只会在指定节点上启动。同时，它还会自动给这个 Pod 加上一个 Toleration，从而忽略节点的 unschedulable“污点”

相比于 Deployment，DaemonSet 只管理 Pod 对象，然后通过 nodeAffinity 和 Toleration 这两个调度器的小功能，保证了每个节点上有且只有一个 Pod .


## Job 与 CronJob -- 离线业务

在 Borg 项目中，Google 对作业进行了分类处理，提出了 LRS（Long Running Service）和 Batch Jobs 两种作业形态，对它们进行“分别管理”和“混合调度”。

跟其他控制器不同的是，Job 对象并不要求你定义一个 spec.selector 来描述要控制哪些 Pod。
