# 什么是 Kubean?

KuBean 是基于 Kubespray 技术实现的一个自动安装及管理 Kubernetes 集群的管理组件。KuBean 可用于在混合云、公有云、私有云、裸金属环境中部署及管理集群。

管理集群 ，主要用于子集群的全生命周期管理（包括子集群的创建，更新，升级等)。管理集群本身为 Kubernetes 集群，并同时对工作集群的生命周期进行管理，如何创建 管理集群 可参考 [创建管理集群](03userguide/clusters/createDKG.md)。

工作集群主要用于承载业务应用容器，工作集群可以基于管理集群进行 创建及管理，管理集群可以基于不同的基础设施（公有云，私有云，裸金属）进行创建，如何创建工作集群 可参考 [创建工作集群](03userguide/clusters/createDKE.md)。



#### 集群全生命周期管理

- **集群的一键式创建**

  - 集群的快速部创建，快速部署企业级的 Kubernetes 集群，快速搭建企业级容器云台，适配物理机和虚拟机底层环境。
  - 创建过程中轻松配置网络、存储、日志等参数，轻松配置生产就绪的 Kubernetes 集群。

  **集群的生命周期管理**

  - 支持集群的一键式证书升级，一键 Kubernetes 版本版本升级，以及系统组件的统一升级及管理。

  **集群的高可用性**

  - 部署的集群支持 控制节点高可用模式，保障业务系统在主机故障下业务不受影响，提高生产环境的稳定性，降低业务中断风

#### 完善的插件机制

- 丰富完善的 Addon 机制，完善容器集群能力，创建集群时可根据不同场景选择性部署需要的插件，目前支持的插件如下：

  - 网络相关组件： Kubeproxy、CoreDNS、Ingress Nginx、MetalLB 、CNI (Calico，Cillium，Multus，Spiderpool，Spiderflat，Macvlan ，Submariner等 )，

  - 存储相关组件 ：RBD Provisioner、Local Path Provisioner、CephFS Provisioner、RBD Provisioner，

  - 其他组件：Metrics Server、Cert Manager

    
