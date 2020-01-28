### Docker 和传统虚拟化方式的不同之处
 
传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；而容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。

### Docker 的优势：
更高效的利用系统资源
更快速的启动时间
一致的运行环境
持续交付和部署
更轻松的迁移
更轻松的维护和扩展

对比传统虚拟机总结



# 底层实现

Docker 底层的核心技术包括 Linux 上的命名空间（Namespaces）、控制组（Control groups）、Union 文件系统（Union file systems）和容器格式（Container format）。
在操作系统中，内核、文件系统、网络、PID、UID、IPC、内存、硬盘、CPU ，所有的资源都是应用进程直接共享的。 要想实现虚拟化，除了要实现对内存、CPU、网络IO、硬盘IO、存储空间等的限制外，还要实现文件系统、网络、PID、UID、IPC等等的相互隔离。 前者相对容易实现一些，后者则需要宿主机系统的深入支持。


随着 Linux 系统对于命名空间功能的完善实现，程序员已经可以实现上面的所有需求，让某些进程在彼此隔离的命名空间中运行。大家虽然都共用一个内核和某些运行时环境（例如一些系统命令和系统库），但是彼此却看不到，都以为系统中只有自己的存在。这种机制就是容器（Container），
**利用命名空间来做权限的隔离控制，利用 cgroups 来做资源分配。**

## 隔离原理和机制



## 命名空间

命名空间是 Linux 内核一个强大的特性。每个容器都有自己单独的命名空间，运行在其中的应用都像是在独立的操作系统中运行一样。命名空间保证了容器之间彼此互不影响。

- PID
- net
- ipc
- mnt
- uts
- user


### cgroups / namespace



### rootfs

Monut Namespace 修改的，是容器进程对文件系统“挂载点”的认知。可以理解为改进 chroot，真正发生隔离是在进行 mount 之后，实现文件系统的隔离。
Mount Namespace 正是基于对 chroot 的不断改良才被发明出来的，它也是 Linux 操作系统里的第一个 Namespace。

需要明确的是，rootfs 只是一个操作系统所包含的文件、配置和目录，并不包括操作系统内核。在 Linux 操作系统中，这两部分是分开存放的，操作系统只有在开机启动时才会加载指定版本的内核镜像。所以说，rootfs 只包括了操作系统的“躯壳”，并没有包括操作系统的“灵魂”。

由于 rootfs 里打包的不只是应用，而是整个操作系统的文件和目录，也就意味着，应用以及它运行所需要的所有依赖，都被封装在了一起。
对一个应用来说，操作系统本身才是它运行所需要的最完整的“依赖库”。
这种深入到操作系统级别的运行环境一致性，打通了应用在本地开发和远端执行环境之间难以逾越的鸿沟。

Docker 在镜像的设计中，引入了层（layer）的概念。也就是说，用户制作镜像的每一步操作，都会生成一个层，也就是一个增量 rootfs。
来实现对基础的增量修改。

### 联合文件系统（Union File System）
最主要的功能是将多个不同位置的目录联合挂载（union mount）到同一个目录下。
我们可以使用 `docker image inspect image-name:version-no.` 来查看Docker镜像的层

一般 docker 镜像的挂载在是 `/var/lib/docker/aufs/mnt/` 目录下。 该目录下，不出意外，每一个子目录都是一个完整的文件系统。

如何查看各个层的信息？
首先，通过查看 AuFS 的挂载信息，我们可以找到这个目录对应的 AuFS 的内部 ID（也叫：si）：
```bash
$ cat /proc/mounts| grep aufs
none /var/lib/docker/aufs/mnt/6e3be5d2ecccae7cc0fc... aufs rw,relatime,si=972c6d361e6b32ba,dio,dirperm1 0 0
```
然后使用这个 ID，你就可以在 /sys/fs/aufs 下查看被联合挂载在一起的各个层的信息：
```bash
$ cat /sys/fs/aufs/si_972c6d361e6b32ba/br[0-9]*
# 读写层，增量增删改，对应一个 commit
/var/lib/docker/aufs/diff/6e3be5d2ecccae7cc...=rw
# init 层，只读。Docker 项目单独生成的一个内部层，专门用来存放 /etc/hosts、/etc/resolv.conf 等信息
/var/lib/docker/aufs/diff/6e3be5d2ecccae7cc...-init=ro+wh
# 只读层， 每一层都以增量方式分别包含 ubuntu 系统的一部分
/var/lib/docker/aufs/diff/32e8e20064858c0f2...=ro+wh
/var/lib/docker/aufs/diff/2b8858809bce62e62...=ro+wh
/var/lib/docker/aufs/diff/20707dce8efc0d267...=ro+wh
/var/lib/docker/aufs/diff/72b0744e06247c7d0...=ro+wh
/var/lib/docker/aufs/diff/a524a729adadedb90...=ro+wh
```
从这些信息里，我们可以看到，镜像的层都放置在 /var/lib/docker/aufs/diff 目录下，然后被联合挂载在 /var/lib/docker/aufs/mnt 里面。

最终，这 7 个层都被联合挂载到 /var/lib/docker/aufs/mnt 目录下，表现为一个完整的 Ubuntu 操作系统供容器使用。

总结：
通过结合使用 Mount Namespace 和 rootfs，容器就能够为进程构建出一个完善的文件系统隔离环境。当然，这个功能的实现还必须感谢 chroot 和 pivot_root 这两个系统调用切换进程根目录的能力。

既然容器的 rootfs（比如，Ubuntu 镜像），是以只读方式挂载的，那么又如何在容器里修改 Ubuntu 镜像的内容呢？（提示：Copy-on-Write）

1. 上面的读写层通常也称为容器层，下面的只读层称为镜像层，所有的增删查改操作都只会作用在容器层，相同的文件上层会覆盖掉下层。知道这一点，就不难理解镜像文件的修改，比如修改一个文件的时候，首先会从上到下查找有没有这个文件，找到，就复制到容器层中，修改，修改的结果就会作用到下层的文件，这种方式也被称为copy-on-write。


除了 AuFS，你知道 Docker 项目还支持哪些 UnionFS 实现吗？你能说出不同宿主机环境下推荐使用哪种实现吗？

2. 查了一下，包括但不限于以下这几种：aufs, device mapper, btrfs, overlayfs, vfs, zfs。aufs是ubuntu 常用的，device mapper 是 centos，btrfs 是 SUSE，overlayfs ubuntu 和 centos 都会使用，现在最新的 docker 版本中默认两个系统都是使用的 overlayfs，vfs 和 zfs 常用在 solaris 系统。