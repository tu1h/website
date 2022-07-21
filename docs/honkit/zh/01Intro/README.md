# 什么是 Kubean

KuBean 是一款自动安装生产环境就绪的 Kubernetes 集群的工具。KuBean 可用于在混合云、公有云、私有云、裸金属环境中部署管理集群，同时自身也可作为管理集群创建工作集群，并对工作集群的生命周期进行管理。

## 集群全生命周期管理

- 创建集群
KuBean 支持通过 Web 界面快速部署企业级的 Kubernetes 集群，快速搭建企业级容器云台，适配物理机和虚拟机底层环境。创建过程中可以轻松配置网络、存储、日志等参数，轻松配置生产就绪的 Kubernetes 集群。

- 升级集群
KuBean 支持一键升级 Kubernetes 版本，统一管理系统组件升级。

- 集群的高可用
KuBean 内置集群容灾和备份能力，保障业务系统在主机故障、机房中断、自然灾害等情况下可恢复，提高生产环境的稳定性，降低业务中断风险。

## 丰富完善的插件机制和完善容器管理平台能力

KuBean 支持多种插件，完善的插件机制造就了 KuBean 强大的容器管理平台能力。目前支持的插件类型分为：

- Core Addon
核心插件默认会跟随集群的创建而部署，主要的核心插件有： Kubeproxy、CoreDNS、CNI (Calico，Cillium，Multus，Spiderpool，Spiderflat，Macvlan 等 )。您可以根据不同场景需求选择合适的网络插件。

- Core Additional Addon
主要包括Local Path Provisioner，Hwameistor，MetaLB，Metrics Server，Ngnix Ingress 等。这类插件在创建集群时可以作为可选项通过 Helm Install 方式进行安装。

- Additional Addon
主要包括 Cert Manager，Contour，Submariner， Nvidia GPU Device Plugin 等。这类插件需要通过 Helm Install 方式进行安装。
