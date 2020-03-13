## 介绍
Pod 扮演的是传统部署环境里“虚拟机”的角色。使得用户从传统环境（虚拟机环境）向 Kubernetes（容器环境）的迁移，更加平滑。

对象(Object)描述了 K8s 集群中的资源，这些对象可以在 yaml 文件中作为一种 API 类型来配置。

 
## Pod属性
凡是调度、网络、存储，以及安全相关的属性，基本上是 Pod 级别的。
这些属性的共同特征是，它们描述的是“机器”这个整体，而不是里面运行的“程序”。比如，配置这个“机器”的网卡（即：Pod 的网络定义），配置这个“机器”的磁盘（即：Pod 的存储定义），配置这个“机器”的防火墙（即：Pod 的安全定义）。更不用说，这台“机器”运行在哪个服务器之上（即：Pod 的调度）。
- NodeSelector
- NodeName
- HostAliases
- hostNetwork: true 
- hostIPC: true
- hostPID: true
- shareProcessNamespace: true
- containers
Image（镜像）、Command（启动命令）、workingDir（容器的工作目录）、Ports（容器要开发的端口），以及 volumeMounts（容器要挂载的 Volume）都是构成 Kubernetes 项目中 Container 的主要字段 。

特别字段：
ImagePullPolicy
Lifecycle：
- postStart：在容器启动后，立刻执行一个指定的操作。需要明确的是，postStart 定义的操作，虽然是在 Docker 容器 ENTRYPOINT 执行之后，但它并不严格保证顺序。也就是说，在 postStart 启动时，ENTRYPOINT 有可能还没有结束。
- preStop：发生的时机，是容器被杀死之前（比如，收到了 SIGKILL 信号）。而需要明确的是，preStop 操作的执行，是同步的。所以，它会阻塞当前的容器杀死流程，直到这个 Hook 定义操作完成之后，才允许容器被杀死，这跟 postStart 不一样。


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

# Pod 进阶

## Projected Volume 投射数据卷

Kubernetes 支持的 Projected Volume 一共有四种：
- Secret：加密的数据存放在 etcd 中，通过在 Pod 挂载 Volumn 的方式访问。
- ConfigMap： 
- Downward API
- ServiceAccountToken

### Secret
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-projected-volume 
spec:
  containers:
  - name: test-secret-volume
    image: busybox
    args:
    - sleep
    - "86400"
    volumeMounts:
    - name: mysql-cred
      mountPath: "/projected-volume"  # focus on this
      readOnly: true
  volumes:
  - name: mysql-cred
    projected:
      sources:
      - secret:
          name: user
      - secret:
          name: pass
```

这样通过挂载方式进入到容器里的 Secret，一旦其对应的 Etcd 里的数据被更新，这些 Volume 里的文件内容，同样也会被更新。其实，这是 kubelet 组件在定时维护这些 Volume。需要注意的是，这个更新可能会有一定的延时。所以在编写应用程序时，在发起数据库连接的代码处写好重试和超时的逻辑，绝对是个好习惯。


### ConfigMap
ConfigMap 保存的是不需要加密的、应用所需的配置信息。而 ConfigMap 的用法几乎与 Secret 完全相同：你可以使用 kubectl create configmap 从文件或者目录创建 ConfigMap，也可以直接编写 ConfigMap 对象的 YAML 文件。


### Downward API
它的作用是：让 Pod 里的容器能够直接获取到这个 Pod API 对象本身的信息。
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-downwardapi-volume
  labels:
    zone: us-est-coast
    cluster: test-cluster1
    rack: rack-22
spec:
  containers:
    - name: client-container
      image: k8s.gcr.io/busybox
      command: ["sh", "-c"]
      args:
      - while true; do
          if [[ -e /etc/podinfo/labels ]]; then
            echo -en '\n\n'; cat /etc/podinfo/labels; fi;
          sleep 5;
        done;
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
          readOnly: false
  volumes:
    - name: podinfo
      projected:
        sources:
        - downwardAPI:
            items:
              - path: "labels"
                fieldRef:
                  fieldPath: metadata.labels
```
Downward API 能够获取到的信息,一定是 Pod 里的容器进程启动之前就能够确定下来的信息。而如果你想要获取 Pod 容器运行后才会出现的信息，比如，容器进程的 PID，那就肯定不能使用 Downward API 了，而应该考虑在 Pod 里定义一个 sidecar 容器。

支持字段
1. 使用fieldRef可以声明使用:
- spec.nodeName - 宿主机名字
- status.hostIP - 宿主机IP
- metadata.name - Pod的名字
- metadata.namespace - Pod的Namespace
- status.podIP - Pod的IP
- spec.serviceAccountName - Pod的Service Account的名字
- metadata.uid - Pod的UID
- metadata.labels['<KEY>'] - 指定<KEY>的Label值
- metadata.annotations['<KEY>'] - 指定<KEY>的Annotation值
- metadata.labels - Pod的所有Label
- metadata.annotations - Pod的所有Annotation

2. 使用resourceFieldRef可以声明使用:
- 容器的CPU limit
- 容器的CPU request
- 容器的memory limit
- 容器的memory request


### Service Account



## 健康检查和恢复

