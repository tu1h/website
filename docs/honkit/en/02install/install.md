# Offline Install

Installing Kubean with offline resources can be divided into four steps. Before start the installation, you need to prepare the following resources/tools:

**Prerequisites**

- [`MinIO`](https://docs.min.io/docs/minio-quickstart-guide.html) to provide storage service
- [`Docker`](https://hub.docker.com/_/registry)/[`Harbor`](https://goharbor.io/docs/2.0.0/install-config/) registry to provide registry service
- [`skopeo`](https://github.com/containers/skopeo/blob/main/install.md) to import images
- [`minio client`](https://docs.min.io/docs/minio-client-quickstart-guide.html) to import binary files

## Step 1: Download the package

You can download Kubean offline resources at [Github Releases](https://github.com/kubean-io/kubean/releases).

``` bash
├── files.list                                  # list of files
├── files-${tag}.tar.gz                         # compressed file package, incl. import scripts
├── images.list                                 # list of images
├── images-${tag}.tar.gz                        # compressed images, incl. import scripts
└── os-pkgs-${linux_distribution}-${tag}.tar.gz # packages of relevant systems, incl. import scripts
```

## Step 2: Import offline resources into relevant services

### Import binaries into MinIO

1. Unzip `files-${tag}.tar.gz`
   
   ``` bash
   files/
   ├── import_files.sh       # script for importing files to minio
   └── offline-files.tar.gz  # compressed binaries

2. Import binaries into MinIO with this command:

   ``` bash
   $ MINIO_USER=${username} MINIO_PASS=${password} ./import_files.sh ${minio_address}
    ```

   >`minio_address` is the address of `minio API Server`; the port usually is 9000. For example:`http://1.2.3.4:9000`.

### Import images into registry

1. Unzip `images-${tag}.tar.gz`
   
   ``` bash
   images/
   ├── import_images.sh       # script for importing images into registry
   └── offline-images.tar.gz  # compressed images
   ```

2. Import the images into docker/harbor registry with this command:
   
   ``` bash
   # 1. password-free mode
   $ DEST_TLS_VERIFY=false ./import_images.sh ${registry_address}

   # 2. password-required mode
   $ DEST_USER=${username} DEST_PASS=${password} ./import_images.sh ${registry_address}
   ```

- If `DEST_TLS_VERIFY=false`, use not-secure HTTP mode to upload images
- If the registry requires a user name and the password, you need to set `DEST_USER` and `DEST_PASS`
- `registry_address` is the address of the registry, e.g., `1.2.3.4:5000`
  
### Import OS packages into MinIO

> Currently only OS Packages of Centos are available.

1. Unzip `os-pkgs-${linux_distribution}-${tag}.tar.gz`

   ``` bash
   os-pkgs
   ├── import_ospkgs.sh              # script for importing os packages to MinIO
   ├── os-pkgs-v0.1.1-amd64.tar.gz   # os packages of amd64
   ├── os-pkgs-v0.1.1-arm64.tar.gz   # os packages of arm64
   └── os-pkgs-v0.1.1.sha256sum.txt  # sha256sum of os packages
   ```

2. Import os packages to MinIO with this command:

   ``` bash
   $ MINIO_USER=${username} MINIO_PASS=${password} ./import_ospkgs.sh ${MinIO_address} os-pkgs-${tag}-${arch}.tar.gz
   ```

## Step 3: Build offline source and config files

You need to build the ISO image source and extras software source.

### Build ISO image source

OS Packages are used mainly for providing installation dependencies for docker-ce. In real offline deployment, you may also need other packages. Therefore, you need to build a local ISO image source.

> You need to download in advance the ISO image designed for your host. Currently only the ISO image source of Centos is available.

After the ISO image is downloaded, you need to mount the image and create repo config files. Use  `artifacts/gen_repo_conf.sh` script to run the following commands:

``` bash
# basic syntax
$ ./gen_repo_conf.sh --iso-mode ${linux_distribution} ${iso_image_file}

# run the script to create the ISO image source
$ ./gen_repo_conf.sh --iso-mode centos CentOS-7-x86_64-Everything-2207-02.iso
# check ISO image mount
$ df -h | grep mnt
/dev/loop0               9.6G  9.6G     0 100% /mnt/centos-iso
# check config of the ISO image source
$ cat /etc/yum.repos.d/Kubean-ISO.repo
[kubean-iso]
name=Kubean ISO Repo
baseurl=file:///mnt/centos-iso
enabled=1
gpgcheck=0
sslverify=0
```
  
If you want to use an online source:

1. Import the ISO image source to `minio server` with this script: `artifacts/import_iso.sh` and the following commands:

    ```bash
    MINIO_USER=${username} MINIO_PASS=${password} ./import_iso.sh ${minio_address} Centos-XXXX.ISO
    ```

2. Create `/etc/yum.repos.d/centos-iso-online.repo` file in the host:

    ```bash
    [kubean-iso-online]
    name=Kubean ISO Repo Online
    # change `${minio_address}` to the address of minio API Server
    baseurl=${minio_address}/kubean/centos-iso/$releasever/os/$basearch
    enabled=1
    gpgcheck=0
    sslverify=0
    ```

### Build extra software source

> Currently only the Centos distribution is available.

When installing a Kubernetes cluster, you also need to install some extra software, such as `container-selinux`. These software are provided in the os packages. After the packages are imported into MinIO, you need to create config files for the extra repo on each node.

To do this, use `artifacts/gen_repo_conf.sh` script to run the following commands:

``` bash
$ ./gen_repo_conf.sh --url-mode ${linux_distribution} ${repo_base_url}

# run the script to create URL source config file
# you need to change `${minio_address}` to the actual address of `minio API Server`
$ ./gen_repo_conf.sh --url-mode centos ${minio_address}/centos/\$releasever/os/\$basearch
# check URL source config file
$ cat /etc/yum.repos.d/Kubean-URL.repo
[kubean-extra]
name=Kubean Extra Repo
# If `repo_base_url` contains `$`, you need to escape it as `\$`
baseurl=http://10.20.30.40:9000/centos/$releasever/os/$basearch
enabled=1
gpgcheck=0
sslverify=0
```

### Create source config file with KubeanClusterOps and playbook

>Currently you can only add Centos yum repo

Creating the source involves all nodes in the cluster, making it troublesome to use scripts. A more convenient solution is provided with playbook:

``` yaml
apiVersion: kubeanclusterops.kubean.io/v1alpha1
kind: KuBeanClusterOps
metadata:
  name: cluster-ops-01
spec:
  kuBeanCluster: sample
  image: ghcr.io/kubean-io/kubean/spray-job:latest
  backoffLimit: 0
  actionType: playbook
  action: cluster.yml
  preHook:
    - actionType: playbook
      action: ping.yml
    - actionType: playbook
      action: enable-repo.yml  # before deploying the cluster, run the palybook of the enable-repo to assign url source config for each node
      extraArgs: |
        -e "{yum_repo_url_list: ['http://10.20.30.40:9000/centos/\$releasever/os/\$basearch']}"
    - actionType: playbook
      action: disable-firewalld.yml
  postHook:
    - actionType: playbook
      action: cluster-info.yml
```

## Step 4: Config before cluster deployment

As for offline configuration, refer to [`kubespray's config file`](https://github.com/kubernetes-sigs/kubespray): `kubespray/inventory/sample/group_vars/all/offline.yml`.

In real configuration, you need to adjust the following file according to the real scenarios, and replace `{{ registry_address }}` 和 `{{ minio_address }}` in particular. Then, you should add the new configuration into `artifacts/offlineDemo/vars-conf-cm.yml`and change the cluster node IP and user name & password in `artifacts/offlineDemo/hosts-conf-cm.yml`. Finally, launch `kubeanClusterOps` to install a Kubernetes cluster with the command `kubectl apply -f artifacts/offlineDemo`.

``` yaml
---
## global offline config
### config address of the private registry
registry_host: "{{ registry_address }}"

### config address of the binaries
files_repo: "{{ minio_address }}"

### if use CentOS / RedHat / AlmaLinux / Fedora, you need to config the address of yum source file
yum_repo: "{{ minio_address }}"

### if use Debian, config:
debian_repo: "{{ minio_address }}"

### if use Ubuntu, config:
ubuntu_repo: "{{ minio_address }}"

### if containerd uses not-secure HTTP mode that requires no authentication, config:
containerd_insecure_registries:
  "{{ registry_address }}": "http://{{ registry_address }}"

### if docker uses not-secure HTTP mode that requires no authentication, config:
docker_insecure_registries:
  - {{ registry_address }}

## Kubernetes components
kubeadm_download_url: "{{ files_repo }}/storage.googleapis.com/kubernetes-release/release/{{ kubeadm_version }}/bin/linux/{{ image_arch }}/kubeadm"
kubectl_download_url: "{{ files_repo }}/storage.googleapis.com/kubernetes-release/release/{{ kube_version }}/bin/linux/{{ image_arch }}/kubectl"
kubelet_download_url: "{{ files_repo }}/storage.googleapis.com/kubernetes-release/release/{{ kube_version }}/bin/linux/{{ image_arch }}/kubelet"

## CNI Plugins
cni_download_url: "{{ files_repo }}/github.com/containernetworking/plugins/releases/download/{{ cni_version }}/cni-plugins-linux-{{ image_arch }}-{{ cni_version }}.tgz"

## cri-tools
crictl_download_url: "{{ files_repo }}/github.com/kubernetes-sigs/cri-tools/releases/download/{{ crictl_version }}/crictl-{{ crictl_version }}-{{ ansible_system | lower }}-{{ image_arch }}.tar.gz"

## [Optional] etcd: only if you **DON'T** use etcd_deployment=host
etcd_download_url: "{{ files_repo }}/github.com/etcd-io/etcd/releases/download/{{ etcd_version }}/etcd-{{ etcd_version }}-linux-{{ image_arch }}.tar.gz"

# [Optional] Calico: If using Calico network plugin
calicoctl_download_url: "{{ files_repo }}/github.com/projectcalico/calico/releases/download/{{ calico_ctl_version }}/calicoctl-linux-{{ image_arch }}"
calicoctl_alternate_download_url: "{{ files_repo }}/github.com/projectcalico/calicoctl/releases/download/{{ calico_ctl_version }}/calicoctl-linux-{{ image_arch }}"
# [Optional] Calico with kdd: If using Calico network plugin with kdd datastore
calico_crds_download_url: "{{ files_repo }}/github.com/projectcalico/calico/archive/{{ calico_version }}.tar.gz"

# [Optional] Flannel: If using Falnnel network plugin
flannel_cni_download_url: "{{ files_repo }}/kubernetes/flannel/{{ flannel_cni_version }}/flannel-{{ image_arch }}"

# [Optional] helm: only if you set helm_enabled: true
helm_download_url: "{{ files_repo }}/get.helm.sh/helm-{{ helm_version }}-linux-{{ image_arch }}.tar.gz"

# [Optional] crun: only if you set crun_enabled: true
crun_download_url: "{{ files_repo }}/github.com/containers/crun/releases/download/{{ crun_version }}/crun-{{ crun_version }}-linux-{{ image_arch }}"

# [Optional] kata: only if you set kata_containers_enabled: true
kata_containers_download_url: "{{ files_repo }}/github.com/kata-containers/kata-containers/releases/download/{{ kata_containers_version }}/kata-static-{{ kata_containers_version }}-{{ ansible_architecture }}.tar.xz"

# [Optional] cri-dockerd: only if you set container_manager: docker
cri_dockerd_download_url: "{{ files_repo }}/github.com/Mirantis/cri-dockerd/releases/download/v{{ cri_dockerd_version }}/cri-dockerd-{{ cri_dockerd_version }}.{{ image_arch }}.tgz"

# [Optional] runc,containerd: only if you set container_runtime: containerd
runc_download_url: "{{ files_repo }}/github.com/opencontainers/runc/releases/download/{{ runc_version }}/runc.{{ image_arch }}"
containerd_download_url: "{{ files_repo }}/github.com/containerd/containerd/releases/download/v{{ containerd_version }}/containerd-{{ containerd_version }}-linux-{{ image_arch }}.tar.gz"
nerdctl_download_url: "{{ files_repo }}/github.com/containerd/nerdctl/releases/download/v{{ nerdctl_version }}/nerdctl-{{ nerdctl_version }}-{{ ansible_system | lower }}-{{ image_arch }}.tar.gz"

```
