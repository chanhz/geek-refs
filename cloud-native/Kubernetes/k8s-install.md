# Kubernetes 安装

## 使用Kubeadm 安装

- 在所有节点上安装 Docker 和 kubeadm；
- 部署 Kubernetes Master；
- 部署容器网络插件；
- 部署 Kubernetes Worker；
- 部署 Dashboard 可视化插件；
- 部署容器存储插件。

```
# 创建一个Master节点
$ kubeadm init

# 将一个Node节点加入到当前集群中
$ kubeadm join <Master节点的IP和端口>
```

### 使用 kubeadm 安装详细步骤

#### 第一步：机器上手动安装 kubeadm、kubelet 和 kubectl 这三个二进制文件
```bash
$ apt-get install kubeadm
```

### 第二步：kubeadm init
工作流程：
**1. Preflight Checks**：
- Linux 内核的版本必须是否是 3.10 以上？
- Linux Cgroups 模块是否可用？
- 机器的 hostname 是否标准？
- 在 Kubernetes 项目里，机器的名字以及一切存储在 Etcd 中的 API 对象，都必须使用标准的 DNS 命名（RFC 1123）。
- 用户安装的 kubeadm 和 kubelet 的版本是否匹配？
- 机器上是不是已经安装了 Kubernetes 的二进制文件？
- Kubernetes 的工作端口 10250/10251/10252 端口是不是已经被占用？
- ip、mount 等 Linux 指令是否存在？Docker 是否已经安装？

**2. 生成 Kubernetes 对外提供服务所需的各种证书和对应的目录**：


**3.证书生成后，kubeadm 接下来会为其他组件生成访问 kube-apiserver 所需的配置文件。这些文件的路径是：/etc/kubernetes/xxx.conf：**:
```
ls /etc/kubernetes/
admin.conf  controller-manager.conf  kubelet.conf  scheduler.conf
```

**4. kubeadm 会为 Master 组件生成 Pod 配置文件**
Kubernetes 三个 Master 组件 
- kube-apiserver
- kube-controller-manager
- kube-scheduler
- etcd (非组件)
而它们都会被使用 Pod 的方式部署起来

这三个组件的 pod 是怎么部署起来的呢? 是 docker run 吗？

不是的。

在 K8S 中， 有一种特殊的容器启动方法叫 “Static Pod”。当Kubelet 启动时，它会自动检查某一个目录的 Pod YAML 文件，然后在这台机器上启动它们。

可以看出，kubelet 是核心的完全独立组件，其他 Mater 组件，更像是辅助性的系统容器。

在这一步完成后，kubeadm 还会再生成一个 Etcd 的 Pod YAML 文件，用来通过同样的 Static Pod 的方式启动 Etcd。所以，最后 Master 组件的 Pod YAML 文件如下所示：

```bash
$ ls /etc/kubernetes/manifests/
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
```

Master 容器启动后，kubeadm 会通过检查 localhost:6443/healthz 这个 Master 组件的健康检查 URL，等待 Master 组件完全运行起来。

一旦这些 YAML 文件出现在被 kubelet 监视的 /etc/kubernetes/manifests 目录下，kubelet 就会自动创建这些 YAML 文件中定义的 Pod，即 Master 组件的容器。

**5. kubeadm 就会为集群生成一个 bootstrap token**

只要持有这个 token，任何一个安装了 kubelet 和 kubadm 的节点，都可以通过 kubeadm join 加入到这个集群当中。这个 token 的值和使用方法会，会在 kubeadm init 结束后被打印出来。

**6. 保存证书配置导configMap**

 token 生成之后，kubeadm 会将 ca.crt 等 Master 节点的重要信息，通过 ConfigMap 的方式保存在 Etcd 当中，供后续部署 Node 节点使用。

 **7. 安装默认插件**
 kubeadm init 的最后一步，就是安装默认插件。Kubernetes 默认 **kube-proxy** 和 **DNS** 这两个插件是必须安装的。它们分别用来提供整个集群的**服务发现**和 **DNS** 功能。其实，这两个插件**也只是两个容器镜像**而已，所以 kubeadm 只要用 Kubernetes 客户端创建两个 Pod 就可以了。


 #### kubeadm join 的工作流程
 使用  bootstrap token “不安全”访问 apiserver 从而拿到 cluster-info 的授权信息。

 只要有了 cluster-info 里的 kube-apiserver 的地址、端口、证书，kubelet 就可以以“安全模式”连接到 apiserver 上，这样一个新的节点就部署完成了。

 ### 其他
 **配置 kubeadm 的部署参数**：
 要指定 kube-apiserver 的启动参数，该怎么办？在这里，我强烈推荐你在使用 kubeadm init 部署 Master 节点时，使用下面这条指令：
 ```bash
$ kubeadm init --config kubeadm.yaml
```


kubeadm 能够用于生产环境吗？

到目前为止（2018 年 9 月），这个问题的答案是：不能。

因为 kubeadm 目前最欠缺的是，一键部署一个高可用的 Kubernetes 集群，即：Etcd、Master 组件都应该是多节点集群，而不是现在这样的单点。

这，当然也正是 kubeadm 接下来发展的主要方向。另一方面，Lucas 也正在积极地把 kubeadm phases 开放给用户，即：用户可以更加自由地定制 kubeadm 的每一个部署步骤。这些举措，都可以让这个项目更加完善，我对它的发展走向也充满了信心。

[规模化部署k8s-kops] (https://github.com/kubernetes/kops)
[二进制安装](https://github.com/SongCF/kubesh)