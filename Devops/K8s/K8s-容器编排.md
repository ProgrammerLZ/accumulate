# 容器编排原理

## Pod的概念

### 为什么需要Pod？

1. 复习：容器的本质是一个进程；容器的镜像可以被认为是一个安装包；K8s可以被认为是一个操作系统。
2. 在一个真正的操作系统当中，进程不是“孤苦伶仃”的，而是以进程组的方式组织在一起的。
3. K8s需要将在真正的操作系统重的进程组的概念映射到容器技术中。
4. Docker Swarm无法提供完备的成组调度的解决方案。工业界以及学术界对承租调度也提出了很多可选的解决方案，但是方案都谈不上完美。
5. K8s通过Pod提供了一种简单的概念来解决上述问题，Pod是K8s里的原子调度单位。



### 关于Pod

* Pod只是一种逻辑概念，并不存在Pod的边界或者隔离环境。
* Pod中的所有容器共享Network Namespace，并且可以声明共享同一个Volume。
* 为了维护一个Pod中的容器的关系是对等的，Pod的实现使用了一个中间容器——Infra容器，它永远是第一个被创建出来的容器。其他容器通过Join Network Namespace的方式与Infra容器关联在一起。

<img src="assets/K8s-%E5%AE%B9%E5%99%A8%E7%BC%96%E6%8E%92/image-20230130105625493.png" alt="image-20230130105625493"/>

* Pod中的容器具有超亲密关系。
* Pod扮演的实际上是传统基础设施里的虚拟机的角色，容器则是这个虚拟机里运行的用户程序。
* 一个容器就是一个进程，这是容器的“天性”，**将一个原本在虚拟机中运行的应用（通常是一个进程组）无缝的迁移到容器当中是跟容器的本质相悖的**。
* Docker Swarm采用**单容器的编**排方式，这种方式在某些情况下超亲密关系的容器无法被正常编排，这也是Docker Swam没有真正使用起来的原因。



## Pod的使用

### Projected Volume（投射数据卷）

存放为容器预先定义好的数据。常用的Projected Volume有4种：

* Secret

  把Pod想要访问的加密数据存放在etcd中，就可以在Pod的容器里通过挂载Volume的方式访问这些Secret中保存的信息。

* ConfigMap

  与Secret类似，区别在于ConfigMap保存的是无需加密的、应用需要的配置信息。

* DownwardAPI

  让Pod里的容器能直接获取这个Pod API对象本身的信息。

* ServiceAccountToken

  * ServiceAccount

    K8s内置的一种服务账户，是K8s进行权限分配的对象。ServiceAccount的授权信息和文件，实际上保存在他所绑定的一个Secret对象里，这是一种特殊的Secret对象。

    ```shell
    ➜   kubectl get serviceaccount -n ms
    NAME             SECRETS   AGE
    default          1         146d
    skywalking-oap   1         30d
    
    ➜   kubectl describe serviceaccount/default -n ms
    Name:                default
    Namespace:           ms
    Labels:              <none>
    Annotations:         <none>
    Image pull secrets:  <none>
    Mountable secrets:   default-token-st6q4
    Tokens:              default-token-st6q4
    Events:              <none>
    
    ➜   kubectl get secret -n ms
    NAME                               TYPE                                  DATA   AGE
    default-token-st6q4                kubernetes.io/service-account-token   3      146d
    
    ➜   kubectl describe secret/default-token-st6q4 -n ms
    Name:         default-token-st6q4
    Namespace:    ms
    Labels:       <none>
    Annotations:  kubernetes.io/service-account.name: default
                  kubernetes.io/service-account.uid: 92346e7c-3b13-4cee-9e3b-0a6bf0788ffb
    
    Type:  kubernetes.io/service-account-token
    
    Data
    ====
    namespace:  2 bytes
    token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IjlCMHFJWk0wT0tZbGxUZE82blZFcENoRDdiZEstaUVpbm5RN0tqZVNEUzgifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJtcyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkZWZhdWx0LXRva2VuLXN0NnE0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImRlZmF1bHQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI5MjM0NmU3Yy0zYjEzLTRjZWUtOWUzYi0wYTZiZjA3ODhmZmIiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6bXM6ZGVmYXVsdCJ9.FDkTKO_p94xbwfjKFR5URIajyeI7F5XDCtAHvHYBveqF_tvwEjW-fa-Sa8LDmLljUVEAX1jhhOwXCLViGk4kaxzJ5K5XbxgnOlF_PA4eCMyq0vslfHe6SUDKO4V1zH_RxGuDO3xJkNN0zpFXEbaV_wHsF0Ng3iVQa3CERCF8xjzJoY9PN3YOEINCO4tkbUiqZOdHNW_PbaP8HoQd3Fa-ujfXTJBh2W5ZEQ1ElXN2czWCFBZkQVTibOLvpDe_YZvCeSFdxfKC1OweJoR5ZzgvZoTRhY6Vt7TQS2eCseSf5esBsHQnYQP89lradNq4SxFGYmZmLKhMVLL3iA8h0PYKnQ
    ca.crt:     1017 bytes
    ```



### 容器健康检查和恢复机制

**健康检查**，K8s中可以为Pod里的容器定义一个健康检查“探针”（Probe），kubelet回根据探针的返回值决定这个容器的状态，而不是直接以容器是否运行作为依据。

**Pod恢复机制**也叫restartPolicy。是Pod的Spec部分的一个标准字段，默认值是`Always`，还有`OnFailure`和`Never`两个选项可供选择。

两个基本的设计原理：

1. 只要Pod的restartPolicy指定的策略允许重启异常的容器，那么这个Pod的就会保持Running状态并重启容器，否则Pod会进入Farled状态
2. 包含多个容器的Pod，只有所有容器都进入异常状态后，Pod才会进入Failed状态。

> 📢 Pod的恢复过程永远发生在当前节点上，而不会跑到别的节点上。想让Pod出现在其他的可用节点上，要使用Deployment这样的“控制器”来管理Pod。



### Pod预设置（PodPreset）

开发人员只需要提交一个基本的Pod Yaml，运维人员可以预先定义好一些信息，，K8s会自动给对应的Pod对象加上其他必要信息。PodPreset的目的是降低开发人员编写Pod Yaml的门槛。

> 如果定义了同事作用于一个Pod对象的多个PodPreset，K8s会合并这两个PodPreset要做的修改；如果有冲突，冲突字段不会被修改。



## 控制器思想

Pod实际上是对容器的进一步抽象和封装，K8s操作Pod的逻辑由**控制器**完成。kube-controller-manager组件就是一系列控制器的组合。

![image-20230131131302707](assets/K8s-%E5%AE%B9%E5%99%A8%E7%BC%96%E6%8E%92/image-20230131131302707.png)

这些控制器每一个控制器都以独有的方式负责某种编排功能。

这些控制器统一采用同一种编排模式——**控制循环**（也叫控制器模式）。

![image-20230131131545884](assets/K8s-%E5%AE%B9%E5%99%A8%E7%BC%96%E6%8E%92/image-20230131131545884.png)

控制循环的结果通常是对被控制对象的写操作。



## Deployment

Deployment实现了K8s中两个非常重要的编排动作：

* 水平扩容/缩容
* 滚动更新

Deployment通过ReplicaSet来实现这两个编排动作，Deployment实际操作ReplicaSet，ReplicaSet操作Pod，他们是一种通过控制器模式进行“层层控制”的关系，如下图：

<img src="assets/K8s-%E5%AE%B9%E5%99%A8%E7%BC%96%E6%8E%92/image-20230131152130696.png" alt="image-20230131152130696" style="zoom:100%;" />

修改Deployment的副本数，会通过控制循环修改ReplicaSet的属性，从而让ReplicaSet达到期望状态；而ReplicaSet的期望状态改变后，又会通过控制器循环将Pod更新至期望状态，这就是一个水平扩容/缩容。

滚动更新会在修改PodTemplate的时候发生，当Deployment的PodTemplate发生改变，Deployment会产生一个新版本的ReplicaSet，新旧版本的ReplicaSet会交替调整Pod的个数以达到滚动更新的目的，如下图：

![image-20230131153023154](assets/K8s-%E5%AE%B9%E5%99%A8%E7%BC%96%E6%8E%92/image-20230131153023154.png)

滚动更新的优点是：在升级失败的情况下，由于新老版本的ReplicaSet的Pod是交替增加和减少的，所以旧的ReplicaSet的Pod的数量不至于缩容为0，这样能够让应用持续提供服务。此外，滚动更新失败的时候可以随时回滚至历史版本。



## StatefulSet

### 拓扑状态

应用的拓扑状态指的是：应用多个Pod是不对等的，他们必须：

1. Pod要按照某种顺序启动。
2. Pod被删除后，再次启动时仍然能够保证顺序。
3. 新创建出来的Pod的网络标识必须和原来完全一样。

下面描述这3点是如何被StatefulSet满足的。

StatefulSet在启动Pod的时候会对他所管理的Pod进行编号，并且启动顺序严格按照编号进行，如下图。

![image-20230131162907150](assets/K8s-%E5%AE%B9%E5%99%A8%E7%BC%96%E6%8E%92/image-20230131162907150.png)

当Pod被删除后，StatefulSet会按照原来的编号顺序，重新创建出两个Pod。

Headless类型的Service，会为他所代理的所有Pod绑定一个稳定的DNS记录，使用这个DNS记录就可以访问到该Pod，该记录的规则如下：

```
<pod-name>.<svc-name>.<namespace>.svc.cluster.local
```

由于StatefulSet给Pod编号的特性保证了<pod-name>不会改变，加上headless类型的service的DNS记录规则，就保证了通过StatefulSet进行编排的每个Pod都具有一个稳定的网络标识。

> 📢 虽然无论如何重启，都能通过一个不变的DNS记录访问到Pod，但是该记录解析的IP地址每次可能都是不一样的。



### 存储状态

应用的存储状态依赖PVC来完成。

1. StatefulSet的yaml文件中会声明volumeClaimTemplate。
2. StatefulSet的控制器在使用控制器模式创建Pod的时候，会为PVC进行编号，所有Pod都会使用编号的PVC。
3. 当删除Pod后，PV以及PVC不会被删除，也就是说已经写进Volume中的数据不会被删除。
4. StatefulSet的控制器循环会重新创建新Pod，并且这个新的Pod会使用与旧Pod相同的PVC。
5. Pod启动后，保存在Volume中的存储状态被正常恢复。



StatefulSet实际上是一种特殊的Deployment，其特殊的地方在于其所有的Pod都被编号了。并且，StatefulSet直接操作Pod，而Deployment直接操作的是ReplicaSet。



## DaemonSet

DaemonSet会在集群中运行Daemon Pod，该Pod有3个特征：

1. Pod在集群中的每个结点上运行。
2. 每个节点上只有一个这样的Pod实例。
3. 有新节点加入后，Pod会自动地在新节点上创建出来；旧结点被删除后，Pod会相应的被回收。

在DeamonSet的控制循环中，只需要遍历所有结点，然后根据节点上是否有被管理的Pod，来决定创建或删除一个Pod。

在控制循环中，创建每个Pod时，DaemonSet会根据当前节点的名称，自动给这个Pod加上一个nodeAffinity（结点亲和性）的字段，从而让这个Pod可以在当前循环到的这个Node上启动。此外，DaemonSet还会给Pod自动加上一个tolerations（容忍）字段，该字段与调度相关，表示Pod能够“容忍”某些节点的污点（Taint），从而保证Pod能够被正常调度。

需要注意，DatemonSet上一般应该加上resources字段，来限制它的CPU和内存使用，防止它占用过多宿主机资源。



### 版本管理

DaemonSet和Deployment一样，也可以进行版本管理，它的版本管理是依靠一个**ControllerRevision**的对象来维护的。一个ControllerRevision对象对应一个版本，ControllerRevision的data字段会完整描述该版本的DaemonSet的完整定义，并且在Annoation字段保存了创建这个对象使用的kubectl命令。



## Job与CronJob



## 附录

### 常用指令

```shell
# 查看API对象的历史
$ kubectl rollout history daemonset fluentd-elasticsearch -n kube-system
# 直接设置API对象的Pod模板镜像
$ kubectl set image ds/fluentd-elasticsearch fluentd-elasticsearch=k8s.gcr.io/fluentd-elasticsearch --record -n kube-system
# 查看正在滚动更新的API对象的状态 
$ kubectl rollout status ds/fluentd-elasticsearch -n kube-system
# 回滚API对象至某个版本
$ kubectl rollout undo daemonset fluentd-elasticsearch --to-revision=1 -n kube-system daemonset.extensions/fluentd-elasticsearch rolled back

```





































