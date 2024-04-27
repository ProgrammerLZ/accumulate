## 隔离与限制

* Linux的Namespace会提供隔离功能。一个正在运行的Docker容器其实就是一个启用了多个Linux Namespace的应用进程；

* Linux的Cgroups提供资源限制功能。

容器是一个单进程模型，容器的本质就是一个进程。

> **Linux之proc**
>
> /proc目录存储的是记录当前内核运行状态的一系列特殊文件，用户可以通过访问这些文件查看系统以及当前正在运行的进程信息，比如CPU的使用情况、内存占用率等，这些文件也是top指令的数据源。



## 深入理解容器镜像

### Mount Namespace

容器里的应用进程理应“看到”一套完全独立的文件系统，这样他就可以在自己的容器目录下进行操作，而不会受到宿主机和其他容器的影响。

首先，想到的Docker实现这种隔离功能，应该是通过Mount Namespace去做，然而实际情况不不是这样。Mount Namespace只是修改了容器进程对文件系统“挂载点”的认知，也就是说，只有在“挂载”这个操作发生之后，进程的视图才会改变，在此之前**新创建的容器会直接继承宿主机的各个挂载点**。

我们希望，每当创建一个容器时，容器进程”看到“的文件系统就是一个独立的隔离环境，而不是继承自宿主机的文件系统。那么，就可以在*容器进程启动之前*重新挂载它的整个根目录，并且由于Mount Namespace的存在，宿主机完全感知不到这个挂载的存在，只有容器自己能够感知到，因此容器进程可以在里边随便折腾了。



### chroot

那么这种功能是如何实现的呢？在Linux操作系统中，有一个`chroot（change root file system）`命令可以完成这项工作。该命令能够：**改变进程的根目录到指定位置**。



### rootfs

挂载在容器进程根目录上用来为容器进程提供隔离后执行环境的文件系统，就是所谓的“容器镜像”。专业名字：rootfs（根文件系统）。

目前为止，总结下，Docker项目最核心的原理实际上就是为待创建的用户进程：

1. 启用Linux Namespace配置；
2. 设置指定的Cgroups参数；
3. 切换进程的根目录

以上完成后，一个完整的容器就诞生了。

> **Linux之pivot_roct**
>
> 实际上，Docker在切换进程的根目录时，会优先使用pivot_roct系统调用，如果系统不支持才会使用chroot。

rootfs只是一个操作系统所包含的文件、配置和目录，并不包含操作系统内核，同一台机器上的所有容器都共享主机操作系统的内核。

> 在Linux操作系统中，系统内核与文件、配置、目录是分开存放的，在开机启动时，操作系统才会加载指定版本的内核镜像。

rootfs为容器提供了非常重要的一个特性：*一致性*。rootfs里打包的不只是应用，而是整个操作系统的文件和目录，应用以及它运行所需要的所有依赖都被封装到了一起，这样，容器就拥有了一致性：无论在本地、云端还是在任何地方的一台机器上，只要是同一个rootfs，应用运行所需的完整执行环境就能重现。



### rootfs的层

Docker将镜像的层都放置在/var/lib/docker/aufs/diff目录下，然后被**联合挂载**到/var/lib/docker/aufs/mnt中。

> **UnionFS（联合文件系统）**
>
> 它最主要的功能是将不同位置的目录“联合挂载”到同一个目录下。

![image-20230116111959453](assets/Docker%E7%9A%84%E4%B8%BB%E8%A6%81%E5%86%85%E5%AE%B9/image-20230116111959453.png)

1. 只读层

   只读层是这个容器的rootfs最下面的5层，对应的正式ubuntu:latest镜像的5层，挂载方式都是只读（ro+wh，即readonly+whiteout）。

2. 可读写层

   挂载方式为可读写（rw）。在写入文件前，这个目录是空的，一旦进行了写操作，修改产生的内容就会以增量的方式出现在该层当中。如果想删除只读层的文件，可以在可读写层创建一个whiteout文件，把只读层里边的文件“遮挡”起来。

   可读写层专门用来存放修改rootfs后产生的增量，对文件的增删改都发生在这里。`docker commit`和`docker push`可以保存这个修改过的可读写层并push到镜像仓库。

3. Init层

   Init层是Docker项目单独生成的一个内部层，专门用来存放/etc/hosts、/etc/resolv.confg等信息。然而，这些文件本来属于只读的Ubuntu镜像的一部分，但是用户往往需要在**启动容器**的时候写入一些指定的值，所以需要在可读写层修改他们，但是这些修改往往只对当前容器有效，不希望再commit的时候连通这些修改一同提交。

   因此，Docker就搞出来了一个Init层，这些文件在被修改了之后会放在一个单独的层中，docker commit的时候只会提交可读写层，不包含Init层中的内容。



## 重新认识Linux容器

```dockerfile
FROM python:2.7-slim

WORKDIR /app

ADD . /app

RUN pip install --trusted-host pypi.python.org -r requirements.txt

EXPOSE 80

ENV NAME World

# 设置容器的进程为：python app.py，即这个Python应用的启动命令
CMD ["python","app.py"]

```

* CMD指的是指定python app.py为该容器的进程
* ENTRYPOINT和CMD都是Docker容器进程启动所必需的参数，完整的执行格式是：ENTRYPOINT CMD，Docker默认提供一个隐含的ENTRYPOINT，也就是/bin/sh -c，所以在不指定ENTRYPOINT的时候，实际上在容器里运行的完整进程是/bin/sh -c "python app.py"，即CMD的内容就是ENTIRYPOINT的参数。

```shell
# 制作Docker镜像
docker build -t helloworld .

# 给镜像命名
docker tag helloworld 16604113102/helloworld:v1

# push到镜像仓库
docker push 16604113102/hellowold:v1

# 进入镜像启动/bin/sh进程
dcoker exec -it xxxxxxx /bin/sh

# 在rootfs中创建一个新文件
touch test.txt

# 退出容器
exit

# 提交修改为一个新的镜像
docker commit xxxxxxx 16604113102/helloworld:v2
```

**docker exec的原理**

一个进程可以选择加入进程已有的某个Namespace当中，从而“进入”该进程所在容器。

**volumn原理**

在rootfs准备好之后，在执行chroot之前，把Volume指定的宿主机目录挂载到指定的容器目录在宿主机上对应的目录上，这个Volume的挂载工作就完成了。



## 容器相较于虚拟机的缺点

* 与宿主机共享操作系统内核，如果容器内的应用程序需要配置内核参数、加载额外的内核模块，以及跟内核直接交互，对于该系统其上的所有容器来说，可能牵一发而动全身。