# 什么是 Kubean

KuBean 是基于 Kubespray 技术实现的一个自动安装并管理 Kubernetes 集群的管理组件。KuBean 可用于在混合云、公有云、私有云、裸金属环境中部署并管理集群。

Kubean 可以创建两类集群：管理集群和工作集群。

- 管理集群

  主要用于工作集群的全生命周期管理（包括工作集群的创建、更新、升级等)。管理集群本身为 Kubernetes 集群，管理集群可以基于不同的基础设施（公有云，私有云，裸金属）进行创建。有关如何创建管理集群，可参考[创建管理集群](03userguide/clusters/create-control-cluster.md)。

- 工作集群
  
  主要用于承载业务应用容器，工作集群可以基于管理集群进行创建并管理。有关如何创建工作集群，可参考[创建工作集群](03userguide/clusters/create-worker-cluster.md)。

Kubean 的主要功能特性如下：

## 集群全生命周期管理

- 创建集群
  
  支持通过 Web 界面快速部署企业级的 Kubernetes 集群，快速搭建企业级容器云台，适配物理机和虚拟机底层环境。创建过程中可以轻松配置网络、存储、日志等参数，轻松配置生产就绪的 Kubernetes 集群。

- 升级集群
  
  容器集群生命周期管理工具支持一键升级 Kubernetes 版本，统一管理系统组件升级。

- 卸载集群
  
  一键卸载集群，释放资源，

- 集群的高可用
  
  容器集群生命周期管理工具内置集群容灾和备份能力，保障业务系统在主机故障、机房中断、自然灾害等情况下可恢复，提高生产环境的稳定性，降低业务中断风险。

## 完善的插件机制

Kubean 支持多种插件，完善的插件机制造就了容器集群生命周期管理工具强大的容器管理平台能力。目前支持的插件类型分为：

- 网络相关组件：Kubeproxy、CoreDNS、Ingress Nginx、MetalLB 、CNI (Calico，Cillium，Multus，Spiderpool，Spiderflat，Macvlan ，Submariner等 )

- 存储相关组件：RBD Provisioner、Local Path Provisioner、CephFS Provisioner、RBD Provisioner，

- 其他组件：Metrics Server、Cert Manager
