
- K8S 资源和控制器，新特性
最小资源单位：Pod
Pod 特性：
Infro Container(k8s.gcr.io/pause) 是 Pod 中负责 Namespace 管理的容器，Pod 中其他容器通过 Join Network Namespace 的方式与 Infro 关联。

同一个 Pod 里面的所有用户容器来说，它们的进出流量，也可以认为都是通过 Infra 容器完成的。

Init Container, Pod 中最先启动的容器。


- MySQL 实战索引章节笔记

- Docker 底层技术

Linux Namespace 提供基本的隔离环境
Linux Control Group 做资源限制

Docker Engine 其实是对进程资源、命名空间的一种沙箱技术。


Cgroups 技术是用来**制造约束**的主要手段，而 Namespace 技术则是用来修改**进程视图**的主要方法。
Namespace汇总：PID、Mount、UTS、IPC、Network 、 User 这些 Namespace用来对各种不同的进程上下文进行“障眼法”操作。

Docker 默认启用了哪些 Namespace?
ipc pid mnt net uts user cgroup
进程间通信
进程唯一标识
mount 挂在卷
网络隔离
主机名隔离
用户空间

https://www.cnblogs.com/yinzhengjie/p/7771645.html
https://cyc2018.github.io/CS-Notes/#/notes/%E8%AE%A1%E7%AE%97%E6%9C%BA%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%20-%20%E8%BF%9B%E7%A8%8B%E7%AE%A1%E7%90%86
https://cyc2018.github.io/CS-Notes/#/notes/Socket
