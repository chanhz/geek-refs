# Volumn

## PV、PVC 和 Storage Class
PVC 描述的，是 Pod 想要使用的持久化存储的属性，比如存储的大小、读写权限等。PV 描述的，则是一个具体的 Volume 的属性，比如 Volume 的类型、挂载目录、远程存储服务器地址等。而 StorageClass 的作用，则是充当 PV 的模板。并且，只有同属于一个 StorageClass 的 PV 和 PVC，才可以绑定在一起。

### Persistent Volume（PV）和 Persistent Volume Claim（PVC）
**PV** 描述的是持久化存储数据卷。例如，运维人员定义一个 PV:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 10.244.1.4
    path: "/"
```

**PVC** 描述的是 Pod 所希望使用的持久化存储的属性，例如，开发人员声明一个 1 GiB 的 PVC:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: manual
  resources:
    requests:
      storage: 1Gi

```

PVC 必须先和 PV 进行绑定，才能被容器使用起来。绑定时需要检查两个条件:
- 第一个条件，是 PV 和 PVC 的 spec 字段。比如，PV 的存储（storage）大小，就必须满足 PVC 的要求。
- 第二个条件，是 PV 和 PVC 的 storageClassName 字段必须一样。

绑定之后，Pod 就可以使用这个 PVC 了：
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    role: web-frontend
spec:
  containers:
  - name: web
    image: nginx
    ports:
      - name: web
        containerPort: 80
    volumeMounts:
        - name: nfs
          mountPath: "/usr/share/nginx/html"
  volumes:
  - name: nfs
    persistentVolumeClaim:
      claimName: nfs
```

可能存在的问题：
1. Pod 启动时，如找不到合适的 PV 和 与其定义的 PVC 绑定，启动就会报错失败。
2. 该机制由 K8S 中的  PersistentVolumeController 持久化存储控制器来控制。

PersistentVolumeController 会不断地查看当前每一个 PVC，是不是已经处于 Bound（已绑定）状态。如果不是，那它就会遍历所有的、可用的 PV，并尝试将其与这个“单身”的 PVC 进行绑定。这样，Kubernetes 就可以保证用户提交的每一个 PVC，只要有合适的 PV 出现，它就能够很快进入绑定状态，从而结束“单身”之旅。而所谓将一个 PV 与 PVC 进行“绑定”，其实就是将这个 PV 对象的名字，填在了 PVC 对象的 spec.volumeName 字段上。所以，接下来 Kubernetes 只要获取到这个 PVC 对象，就一定能够找到它所绑定的 PV。


Q：**PV 对象，是如何变成容器里的一个持久化存储的呢**？
所谓容器的 Volume，其实就是将一个宿主机上的目录，跟一个容器里的目录绑定挂载在了一起。
而所谓的“持久化 Volume”，指的就是这个宿主机上的目录，具备“持久性”。**即：这个目录里面的内容，既不会因为容器的删除而被清理掉，也不会跟当前的宿主机绑定。这样，当容器被重启或者在其他节点上重建出来之后，它仍然能够通过挂载这个 Volume，访问到这些内容。**

大多数情况下，持久化 Volume 的实现，往往依赖于一个远程存储服务，比如：远程文件存储（比如，NFS、GlusterFS）、远程块存储（比如，公有云提供的远程磁盘）等等。而 Kubernetes 需要做的工作，就是使用这些存储服务，来为容器准备一个持久化的宿主机目录，以供将来进行绑定挂载时使用。而所谓“持久化”，指的是容器在这个目录里写入的文件，都会保存在远程存储中，从而使得这个目录具备了“持久性”。这个准备“持久化”宿主机目录的过程，可以形象地称为“两阶段处理”。

例子：
当一个 Pod 调度到一个节点上之后，kubelet 就要负责为这个 Pod 创建它的 Volumn。默认情况下，kubelet 为 Volumn 创建的目录是宿主机上的一个路径：
```bash
/var/lib/kubelet/pods/<Pod的ID>/volumes/kubernetes.io~<Volume类型>/<Volume名字>
```
如果 Volumn 类型是块存储，比如 Google Cloud 的 Persistent Disk， 那么 kubelet 就需要先调用 Goolge Cloud 的 API，将它所提供的 Persistent Disk 挂载到 Pod 所在的宿主机上。
**为虚拟机挂载远程磁盘的操作，对应的正是“两阶段处理”的第一阶段**。在 Kubernetes 中，我们把这个阶段称为 **Attach**。

Attach 阶段完成后，为了能够使用这个远程磁盘，kubelet 还要进行第二个操作，即：格式化这个磁盘设备，然后**将它挂载到宿主机指定的挂载点**上。不难理解，这个挂载点，正是我在前面反复提到的 Volume 的宿主机目录。

**将磁盘设备格式化并挂载到 Volume 宿主机目录的操作，对应的正是“两阶段处理”的第二个阶段**，我们一般称为：**Mount**。

如果 Volume 类型是**远程文件存储**（比如 NFS），kubelet 的处理过程就会更简单一些。因为在这种情况下，kubelet 可以跳过“第一阶段”（Attach）的操作，这是因为一般来说，远程文件存储并没有一个“存储设备”需要挂载在宿主机上。


这一步，kubelet 需要作为 client，将远端 NFS 服务器的目录（比如：“/”目录），挂载到 Volume 的宿主机目录上，即相当于执行如下所示的命令：
```bash
$ mount -t nfs <NFS服务器地址>:/ /var/lib/kubelet/pods/<Pod的ID>/volumes/kubernetes.io~<Volume类型>/<Volume名字>
```

- Attach，Kubernetes 提供的可用参数是 nodeName，即宿主机的名字;
- Mount，Kubernetes 提供的可用参数是 dir，即 Volume 的宿主机目录。

经过了“两阶段处理”，我们就得到了一个“持久化”的 Volume 宿主机目录。所以，接下来，**kubelet 只要把这个 Volume 目录通过 CRI 里的 Mounts 参数，传递给 Docker，然后就可以为 Pod 里的容器挂载这个“持久化”的 Volume 了。**

其实，这一步相当于执行了如下所示的命令：
```bash
$ docker run -v /var/lib/kubelet/pods/<Pod的ID>/volumes/kubernetes.io~<Volume类型>/<Volume名字>:/<容器内的目标目录> 我的镜像
```

前面人工管理 PV 的方式就叫作 Static Provisioning。

**Dynamic Provisioning**  机制工作的核心，在于一个名叫 **StorageClass** 的 API 对象。其作用是创建 PV 的模板。
Storage Class 会定义：
- PV 的属性。比如，存储类型、Volume 的大小等等。
- 创建这种 PV 需要用到的存储插件。比如，Ceph 等等

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: block-service
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```


