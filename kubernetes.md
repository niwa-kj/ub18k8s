kubernetes

# 作成途中です。失敗も含みます。

ubuntu18.04 server
## ネットワーク設定、固定IP（マスタ、ノード）
このファイルを編集してはいけないらしい。
```
root@master01:~# cat /etc/netplan/50-cloud-init.yaml
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        enp0s3:
            dhcp4: true
        enp0s8:
            dhcp4: true
    version: 2
root@master01:~#
```

```
niwa@master01:~$ cat /etc/netplan/99-persistetip.yaml
network:
    version: 2
    ethernets:
        enp0s3:
            dhcp4: true
        enp0s8:
            dhcp4: no
            addresses: [192.168.56.103/24]
niwa@master01:~$
```

## SELinuxの設定確認（マスタ、ノード）
```
root@master01:~# apt install selinux-utils

root@master01:~# getenforce
Disabled
root@master01:~#
```
## Firewallの停止（マスタ、ノード）
```
root@master01:~# ufw status
Status: inactive

もし、activeなら停止する。
root@master01:~# ufw disable
Firewall stopped and disabled on system startup
root@master01:~# ufw status
Status: inactive
root@master01:~#
```

## IPv6無効化（マスタ、ノード）
下記を追加する
```
/etc/sysctl.d/99-sysctl.conf
 79 net.ipv6.conf.all.disable_ipv6 = 1
 80 net.ipv6.conf.default.disable_ipv6 = 1
 81 net.ipv6.conf.lo.disable_ipv6  =  1
```
IPv6の確認
```
 root@master01:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:e9:da:1f brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 83309sec preferred_lft 83309sec
    inet6 fe80::a00:27ff:fee9:da1f/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:b1:5d:47 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.103/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feb1:5d47/64 scope link
       valid_lft forever preferred_lft forever
root@master01:~#
```
設定の有効化
```
root@master01:~# sysctl -p
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```
設定後のIPv6の確認
```
root@master01:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:e9:da:1f brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 83301sec preferred_lft 83301sec
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:b1:5d:47 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.103/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
root@master01:~#
```
## dockerのインストール（マスタ、ノード）
https://insilico-notebook.com/ubuntu-18-04-dockerce-install/
リポジトリの更新とツールのインストール
```
root@master01:~# apt-get update
root@master01:~# apt-get install \
apt-transport-https \
ca-certificates \
curl \
gnupg-agent \
software-properties-common
```
apt-key(PGP鍵）の取得
```
root@master01:~# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
OK
root@master01:~# apt-key fingerprint 0EBFCD88
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]

root@master01:~#
```
リポジトリの追加（アーキテクチャとリリースを調べる）
```
root@master01:~# arch
x86_64
root@master01:~# lsb_release -cs
bionic
root@master01:~# add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
 stable"
```
Dcokerインストールと起動確認
```
root@master01:~# apt-get install docker-ce docker-ce-cli containerd.io
...
Docker起動
root@master01:~# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:f9dfddf63636d84ef479d645ab5885156ae030f611a56f3a7ac7f2fdd86d7e4e
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

root@master01:~#
```

```
oot@master01:~# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: e
   Active: active (running) since Mon 2020-03-23 06:12:30 UTC; 9min ago
     Docs: https://docs.docker.com
```
## kubeadmをインストール（マスタ、ノード）
```
PGPの鍵を取得
root@master01:~# curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

k8sのリポジトリを追加する。
root@master01:~# apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"

リポジトリ更新
root@master01:~# apt update

インストール
root@master01:~# apt install kubeadm
```
## swapをオフにする。（マスタ、ノード）
```
root@node01:~# swapon -s
Filename                                Type            Size    Used    Priority
/swap.img                               file            2097148 0       -2
root@node01:~# swapoff -a
root@node01:~# swapon -s
root@node01:~#
```
# 課題：恒久的にswapoffする方法

## ipv4のフォワード設定確認 （マスタ、ノード）
```
root@master01:~# cat /proc/sys/net/ipv4/ip_forward
1
root@master01:~#
```
## kubernetesインストール （マスタ）⇐ノードのjoinで失敗する。 --apiserver-advertise-address=を付けないと最初に見つけたNICが採用されるのか？やり直し参照⇓
```
root@master01:~# kubeadm init --pod-network-cidr=10.100.0.0/16
W0323 06:51:23.059972    8764 validation.go:28] Cannot validate ＝kubelet config - no validator is available
W0323 06:51:23.060237    8764 validation.go:28] Cannot validate kube-proxy config - no validator is available
[init] Using Kubernetes version: v1.17.4
[preflight] Running pre-flight checks
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'

[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [master01 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.0.2.15]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [master01 localhost] and IPs [10.0.2.15 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [master01 localhost] and IPs [10.0.2.15 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Wriｓting "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
W0323 06:53:54.056882    8764 manifests.go:214] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
W0323 06:53:54.081067    8764 manifests.go:214] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[kubelet-check] Initial timeout of 40s passed.
[apiclient] All control plane components are healthy after 79.513867 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.17" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node master01 as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node master01 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: acafxc.k9zudmji9125tfkr
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.0.2.15:6443 --token acafxc.k9zudmji9125tfkr \
    --discovery-token-ca-cert-hash sha256:c1116917c4283d1136cf9b4729593d0c8b050ba6a6a486483de64cd3c34d6352
root@master01:~#
```

```
root@master01:~# mkdir -p $HOME/.kube
root@master01:~# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
root@master01:~# chown $(id -u):$(id -g) $HOME/.kube/config
root@master01:~#
```
## コンテナネットワーキング（CNI）としてFlannelを使用する（その他Calicoを使うか迷うところ）
https://qiita.com/ozota/items/3b6e4374688cc989ca12

## コンテナ間通信設定(Calico)（マスタ）
```
root@master01:~# kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
root@master01:~#
root@master01:~# kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
configmap/calico-config created
service/calico-typha created
poddisruptionbudget.policy/calico-typha created
serviceaccount/calico-node created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
unable to recognize "https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml": no matches for kind "Deployment" in version "apps/v1beta1"
unable to recognize "https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml": no matches for kind "DaemonSet" in version "extensions/v1beta1"
root@master01:~#
```
## ノードの設定⇒失敗している
```
root@node01:~# kubeadm join 193.168.56.103:6443 --v=5 --token acafxc.k9zudmji912
5tfkr --discovery-token-ca-cert-hash sha256:c1116917c4283d1136cf9b4729593d0c8b05
0ba6a6a486483de64cd3c34d6352
W0323 13:26:14.614112   25987 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
I0323 13:26:14.615481   25987 join.go:371] [preflight] found NodeName empty; using OS hostname as NodeName
I0323 13:26:14.617583   25987 initconfiguration.go:103] detected and using CRI socket: /var/run/dockershim.sock
[preflight] Running pre-flight checks
I0323 13:26:14.629261   25987 preflight.go:90] [preflight] Running general checks
I0323 13:26:14.630850   25987 checks.go:249] validating the existence and emptiness of directory /etc/kubernetes/manifests
I0323 13:26:14.632228   25987 checks.go:286] validating the existence of file /etc/kubernetes/kubelet.conf
I0323 13:26:14.633044   25987 checks.go:286] validating the existence of file /etc/kubernetes/bootstrap-kubelet.conf
I0323 13:26:14.634206   25987 checks.go:102] validating the container runtime
I0323 13:26:15.321860   25987 checks.go:128] validating if the service is enabled and active
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
I0323 13:26:16.240976   25987 checks.go:335] validating the contents of file /proc/sys/net/bridge/bridge-nf-call-iptables
I0323 13:26:16.242790   25987 checks.go:335] validating the contents of file /proc/sys/net/ipv4/ip_forward
I0323 13:26:16.244559   25987 checks.go:649] validating whether swap is enabled or not
I0323 13:26:16.244870   25987 checks.go:376] validating the presence of executable ip
I0323 13:26:16.247498   25987 checks.go:376] validating the presence of executable iptables
I0323 13:26:16.249674   25987 checks.go:376] validating the presence of executable mount
I0323 13:26:16.256156   25987 checks.go:376] validating the presence of executable nsenter
I0323 13:26:16.256432   25987 checks.go:376] validating the presence of executable ebtables
I0323 13:26:16.256688   25987 checks.go:376] validating the presence of executable ethtool
I0323 13:26:16.257216   25987 checks.go:376] validating the presence of executable socat
I0323 13:26:16.257428   25987 checks.go:376] validating the presence of executable tc
I0323 13:26:16.257643   25987 checks.go:376] validating the presence of executable touch
I0323 13:26:16.258112   25987 checks.go:520] running all checks
I0323 13:26:17.186953   25987 checks.go:406] checking whether the given node name is reachable using net.LookupHost
I0323 13:26:17.191974   25987 checks.go:618] validating kubelet version
I0323 13:26:18.076158   25987 checks.go:128] validating if the service is enabled and active
I0323 13:26:18.180790   25987 checks.go:201] validating availability of port 10250
I0323 13:26:18.184731   25987 checks.go:286] validating the existence of file /etc/kubernetes/pki/ca.crt
I0323 13:26:18.185356   25987 checks.go:432] validating if the connectivity type is via proxy or direct
I0323 13:26:18.187441   25987 join.go:441] [preflight] Discovering cluster-info
I0323 13:26:18.189213   25987 token.go:188] [discovery] Trying to connect to API Server "193.168.56.103:6443"
I0323 13:26:18.198042   25987 token.go:73] [discovery] Created cluster-info discovery client, requesting info from "https://193.168.56.103:6443"
I0323 13:26:28.201838   25987 token.go:78] [discovery] Failed to request cluster info: [Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)]
I0323 13:26:28.201987   25987 token.go:191] [discovery] Failed to connect to API Server "193.168.56.103:6443": Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
I0323 13:26:33.202925   25987 token.go:188] [discovery] Trying to connect to API Server "193.168.56.103:6443"
I0323 13:26:33.208842   25987 token.go:73] [discovery] Created cluster-info discovery client, requesting info from "https://193.168.56.103:6443"
I0323 13:26:43.212637   25987 token.go:78] [discovery] Failed to request cluster info: [Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)]
I0323 13:26:43.212890   25987 token.go:191] [discovery] Failed to connect to API Server "193.168.56.103:6443": Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
I0323 13:26:48.214446   25987 token.go:188] [discovery] Trying to connect to API Server "193.168.56.103:6443"
I0323 13:26:48.224597   25987 token.go:73] [discovery] Created cluster-info discovery client, requesting info from "https://193.168.56.103:6443"
I0323 13:26:58.228800   25987 token.go:78] [discovery] Failed to request cluster info: [Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)]
I0323 13:26:58.228938   25987 token.go:191] [discovery] Failed to connect to API Server "193.168.56.103:6443": Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
I0323 13:27:03.230190   25987 token.go:188] [discovery] Trying to connect to API Server "193.168.56.103:6443"
I0323 13:27:03.236238   25987 token.go:73] [discovery] Created cluster-info discovery client, requesting info from "https://193.168.56.103:6443"
I0323 13:27:13.239066   25987 token.go:78] [discovery] Failed to request cluster info: [Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)]
I0323 13:27:13.239178   25987 token.go:191] [discovery] Failed to connect to API Server "193.168.56.103:6443": Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
I0323 13:27:18.240997   25987 token.go:188] [discovery] Trying to connect to API Server "193.168.56.103:6443"
I0323 13:27:18.249327   25987 token.go:73] [discovery] Created cluster-info discovery client, requesting info from "https://193.168.56.103:6443"
I0323 13:27:28.254468   25987 token.go:78] [discovery] Failed to request cluster info: [Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)]
I0323 13:27:28.254601   25987 token.go:191] [discovery] Failed to connect to API Server "193.168.56.103:6443": Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
I0323 13:27:33.255489   25987 token.go:188] [discovery] Trying to connect to API Server "193.168.56.103:6443"
I0323 13:27:33.269379   25987 token.go:73] [discovery] Created cluster-info discovery client, requesting info from "https://193.168.56.103:6443"
I0323 13:27:43.272349   25987 token.go:78] [discovery] Failed to request cluster info: [Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)]
I0323 13:27:43.272492   25987 token.go:191] [discovery] Failed to connect to API Server "193.168.56.103:6443": Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
I0323 13:27:48.274228   25987 token.go:188] [discovery] Trying to connect to API Server "193.168.56.103:6443"
I0323 13:27:48.281335   25987 token.go:73] [discovery] Created cluster-info discovery client, requesting info from "https://193.168.56.103:6443"
I0323 13:27:58.284829   25987 token.go:78] [discovery] Failed to request cluster info: [Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)]
I0323 13:27:58.284941   25987 token.go:191] [discovery] Failed to connect to API Server "193.168.56.103:6443": Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
I0323 13:28:03.285741   25987 token.go:188] [discovery] Trying to connect to API Server "193.168.56.103:6443"
I0323 13:28:03.295497   25987 token.go:73] [discovery] Created cluster-info discovery client, requesting info from "https://193.168.56.103:6443"
I0323 13:28:13.298011   25987 token.go:78] [discovery] Failed to request cluster info: [Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)]
I0323 13:28:13.298119   25987 token.go:191] [discovery] Failed to connect to API Server "193.168.56.103:6443": Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
I0323 13:28:18.299916   25987 token.go:188] [discovery] Trying to connect to API Server "193.168.56.103:6443"
I0323 13:28:18.310484   25987 token.go:73] [discovery] Created cluster-info discovery client, requesting info from "https://193.168.56.103:6443"
I0323 13:28:28.320365   25987 token.go:78] [discovery] Failed to request cluster info: [Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)]
I0323 13:28:28.320650   25987 token.go:191] [discovery] Failed to connect to API Server "193.168.56.103:6443": Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
I0323 13:28:33.322150   25987 token.go:188] [discovery] Trying to connect to API Server "193.168.56.103:6443"
I0323 13:28:33.329022   25987 token.go:73] [discovery] Created cluster-info discovery client, requesting info from "https://193.168.56.103:6443"
I0323 13:28:43.331890   25987 token.go:78] [discovery] Failed to request cluster info: [Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)]
I0323 13:28:43.332023   25987 token.go:191] [discovery] Failed to connect to API Server "193.168.56.103:6443": Get https://193.168.56.103:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
I0323 13:28:48.332503   25987 token.go:188] [discovery] Trying to connect to API Server "193.168.56.103:6443"
I0323 13:28:48.338167   25987 token.go:73] [discovery] Created cluster-info discovery client, requesting info from "https://193.168.56.103:6443"
^C
root@node01:~#```
```
## kubernetesインストール （マスタ）やり直し（swapoff、2vCPU必要）
```
kubeadm init --apiserver-advertise-address=192.168.56.103 --pod-network-cidr=10.100.0.0/16
root@master01:~# kubeadm init --apiserver-advertise-address=192.168.56.103 --pod
-network-cidr=10.100.0.0/16
W0324 04:58:37.713749    1717 validation.go:28] Cannot validate kubelet config - no validator is available
W0324 04:58:37.714005    1717 validation.go:28] Cannot validate kube-proxy config - no validator is available
[init] Using Kubernetes version: v1.17.4
[preflight] Running pre-flight checks
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [master01 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.56.103]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [master01 localhost] and IPs [192.168.56.103 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [master01 localhost] and IPs [192.168.56.103 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
W0324 05:01:23.311897    1717 manifests.go:214] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
W0324 05:01:23.381860    1717 manifests.go:214] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[kubelet-check] Initial timeout of 40s passed.
[apiclient] All control plane components are healthy after 83.515876 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.17" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node master01 as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node master01 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: dh26vb.zihdd4be2jsq1lmu
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.56.103:6443 --token dh26vb.zihdd4be2jsq1lmu \
    --discovery-token-ca-cert-hash sha256:257b52014a29d60a2edaaf076a9d157f6949009fabe5cc1677eb9f1edc8be54e
root@master01:~#
```
## kubectlユーザ用設定（マスタ）やり直し
```
root@master01:~# mkdir -p $HOME/.kube
root@master01:~# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
root@master01:~# chown $(id -u):$(id -g) $HOME/.kube/config
root@master01:~#
```
## コンテナ間通信設定(Calico)（マスタ）やり直し（★がエラーになる）
```
root@master01:~# kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
root@master01:~#
root@master01:~# kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
configmap/calico-config created
service/calico-typha created
poddisruptionbudget.policy/calico-typha created
serviceaccount/calico-node created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
unable to recognize "https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml": no matches for kind "Deployment" in version "apps/v1beta1"★
unable to recognize "https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml": no matches for kind "DaemonSet" in version "extensions/v1beta1"★
root@master01:~#
```
## ノード設定（ノード01、ノード02）
```
root@node01:~# kubeadm join 192.168.56.103:6443 --token dh26vb.zihdd4be2jsq1lmu \
>     --discovery-token-ca-cert-hash sha256:257b52014a29d60a2edaaf076a9d157f6949009fabe5cc1677eb9f1edc8be54e
W0324 05:06:19.753514   11799 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.17" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

root@node01:~#
```
## ノードの確認
```
root@master01:~# kubectl get nodes
NAME       STATUS     ROLES    AGE     VERSION
master01   NotReady   master   5m31s   v1.17.4
node01     NotReady   <none>   99s     v1.17.4
node02     NotReady   <none>   13s     v1.17.4
root@master01:~#

「コンテナ間通信設定(Calico)（マスタ）やり直し」で失敗しているので⇓こちらを参考にやり直し。
https://docs.projectcalico.org/getting-started/kubernetes/quickstart

root@master01:~# kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
configmap/calico-config configured
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org unchanged
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org unchanged
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org unchanged
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org unchanged
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org unchanged
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org unchanged
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org unchanged
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org unchanged
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org unchanged
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node configured
clusterrolebinding.rbac.authorization.k8s.io/calico-node configured
daemonset.apps/calico-node created
serviceaccount/calico-node unchanged
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
root@master01:~#
```
kube-systemのPodが全部Runningになっていることを確認する。ホストマシンがショボいのでしばらく時間がかかる。
```
oot@master01:~# kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-788d6b9876-6xn2d   1/1     Running   0          4m15s
kube-system   calico-node-j96ss                          1/1     Running   0          4m17s
kube-system   calico-node-nv7zh                          1/1     Running   0          4m17s
kube-system   calico-node-t45zg                          1/1     Running   0          4m17s
kube-system   coredns-6955765f44-pj86m                   1/1     Running   0          24m
kube-system   coredns-6955765f44-qds9b                   1/1     Running   0          24m
kube-system   etcd-master01                              1/1     Running   0          25m
kube-system   kube-apiserver-master01                    1/1     Running   0          25m
kube-system   kube-controller-manager-master01           1/1     Running   0          25m
kube-system   kube-proxy-d8b7k                           1/1     Running   0          21m
kube-system   kube-proxy-gq2mg                           1/1     Running   0          19m
kube-system   kube-proxy-hx8vw                           1/1     Running   0          24m
kube-system   kube-scheduler-master01                    1/1     Running   0          25m
root@master01:~#
```
ノード確認全部Readyになっている。（master01だけINTERNAL-IPが違う）
```
root@master01:~# kubectl get nodes -o wide
NAME       STATUS   ROLES    AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
master01   Ready    master   29m   v1.17.4   10.0.2.15        <none>        Ubuntu 18.04.4 LTS   4.15.0-91-generic   docker://19.3.8
node01     Ready    <none>   25m   v1.17.4   192.168.56.104   <none>        Ubuntu 18.04.4 LTS   4.15.0-91-generic   docker://19.3.8
node02     Ready    <none>   23m   v1.17.4   192.168.56.105   <none>        Ubuntu 18.04.4 LTS   4.15.0-91-generic   docker://19.3.8
root@master01:~#
```
## Calicoのクライアントをインストールする
```
root@master01:~/calico# wget https://github.com/projectcalico/calicoctl/releases/download/v3.9.2/calicoctl
root@master01:~/calico# ll
total 36232
drwxr-xr-x 2 root root     4096 Mar 24 05:46 ./
drwx------ 5 root root     4096 Mar 24 05:46 ../
-rw-r--r-- 1 root root 37090848 Oct 14 22:56 calicoctl
root@master01:~/calico# chmod +x calicoctl
root@master01:~/calico# ./calicoctl
Usage:
  calicoctl [options] <command> [<args>...]
Invalid option: 'calicoctl '. Use flag '--help' to read about a specific subcommand.
root@master01:~/calico#
```
## とりあえずPod作成する
```
root@master01:~/manifest# cat -n pod-nginx.yaml
     1  apiVersion: v1
     2  kind: Pod
     3  metadata:
     4    name: nginx
     5  spec:
     6    containers:
     7    - name: nginx
     8      image: nginx
root@master01:~/manifest#

root@master01:~/manifest# kubectl create -f pod-nginx.yaml

root@master01:~/manifest# kubectl get pod -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          70s   10.100.196.129   node01   <none>           <none>
root@master01:~/manifest#
```

## Podをもう少し詳しく
https://cstoku.dev/posts/2018/k8sdojo-04/
```
root@master01:~/repos/ub18k8s/manifest# cat -n pod-nginx2.yaml
     1  apiVersion: v1
     2  kind: Pod
     3  metadata:
     4    name: nginx2
     5  spec:
     6    containers:
     7    - name: nginx
     8      image: nginx:alpine        # BusyBoxをベースとしたLinux
     9      imagePullPolicy: Always    # 常にコンテナイメージをPullする
    10      command: []                # コンテナで起動させたいプロセス
    11      args: ["nginx", "-g", "daemon off;"] # コンテナ引数。commandとの違いは何か？
    12      env:
    13      - name: MY_NAME            # 環境変数名
    14        value: niwa-kj           # 環境変数の値
    15      ports:
    16      - containerPort: 80        # 晒すポート番号
    17        protocol: TCP            #
    18      workingDir: /tmp           #
    19
root@master01:~/repos/ub18k8s/manifest#
```
Podを起動
```
root@master01:~/repos/ub18k8s/manifest# kubectl create -f pod-nginx2.yaml
pod/nginx2 created
root@master01:~/repos/ub18k8s/manifest# kubectl get pod -o wide
NAME     READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
nginx2   1/1     Running   0          17s   10.100.196.131   node01   <none>           <none>
root@master01:~/repos/ub18k8s/manifest#
```
コンテナ引数が反映されているところ★
```
root@master01:~/repos/ub18k8s/manifest# kubectl exec -it nginx2 -- ps -a
PID   USER     TIME  COMMAND
    1 root      0:00 nginx: master process nginx -g daemon off;★
    6 nginx     0:00 nginx: worker process
   67 root      0:00 ps -a
root@master01:~/repos/ub18k8s/manifest#
```
環境変数との値が反映されている★
```
root@master01:~/repos/ub18k8s/manifest# kubectl exec -it nginx2 -- env | grep MY
MY_NAME=niwa-kj★
root@master01:~/repos/ub18k8s/manifest#
root@master01:~/repos/ub18k8s/manifest# kubectl exec -it nginx2 -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=nginx2
TERM=xterm
MY_NAME=niwa-kj★
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
NGINX_VERSION=1.17.9
NJS_VERSION=0.3.9
PKG_RELEASE=1
HOME=/root
root@master01:~/repos/ub18k8s/manifest#
```
workingDirが反映されていることろ
```
root@master01:~/repos/ub18k8s/manifest# kubectl exec -it nginx2 -- ls -l /proc/1/cwd
lrwxrwxrwx    1 root     root             0 Mar 27 04:42 /proc/1/cwd -> /tmp
root@master01:~/repos/ub18k8s/manifest#
```
## volumeについて(emptyDir)

Volumeを作成してPodから書き込む
```
root@master01:~/repos/ub18k8s/manifest# cat -n pod-vol.yaml
     1  apiVersion: v1
     2  kind: Pod
     3  metadata:
     4    name: pod-cenos-vol
     5  spec:
     6    containers:
     7    - name: date-centos
     8      image: alpine
     9      command: ["sh", "-c"]
    10      args:
    11      - |
    12        exec >> /var/log/date-tail/output.log
    13        echo -n 'Started at: '
    14        while true; do date; sleep 3; done
    15      volumeMounts:                   # Containerのvolume mount
    16      - name: logdate-vol             # mountするVolume名
    17        mountPath: /var/log/date-tail # Container内のmount場所
    18    volumes:
    19    - name: logdate-vol               # mountするVolume名
    20      emptyDir:                       # Podが生きている間は保持される
    21    terminationGracePeriodSeconds: 0  # PodにSIGKILLが送信されるまでの猶予
    22
root@master01:~/repos/ub18k8s/manifest#
```
Pod作成
```
root@master01:~/repos/ub18k8s/manifest# kubectl apply -f pod-vol.yaml
root@master01:~/repos/ub18k8s/manifest# kubectl get pod -o wide
NAME            READY   STATUS    RESTARTS   AGE    IP              NODE     NOMINATED NODE   READINESS GATES
pod-cenos-vol   1/1     Running   0          3m8s   10.100.140.66   node02   <none>           <none>
root@master01:~/repos/ub18k8s/manifest#
```
Pod内での出力を確認する。
```
root@master01:~/repos/ub18k8s/manifest# kubectl exec -it pod-cenos-vol -- tail -f /var/log/date-tail/output.log
Fri Mar 27 05:37:46 UTC 2020
Fri Mar 27 05:37:49 UTC 2020
Fri Mar 27 05:37:52 UTC 2020
Fri Mar 27 05:37:55 UTC 2020
Fri Mar 27 05:37:58 UTC 2020
Fri Mar 27 05:38:01 UTC 2020
Fri Mar 27 05:38:04 UTC 2020
Fri Mar 27 05:38:07 UTC 2020
Fri Mar 27 05:38:10 UTC 2020
Fri Mar 27 05:38:13 UTC 2020
Fri Mar 27 05:38:17 UTC 2020
^Ccommand terminated with exit code 130
root@master01:~/repos/ub18k8s/manifest#
```

## volumeについて(hostPath)
volumeを作成してPodから書き込むところは同じ
```
root@master01:~/repos/ub18k8s/manifest# cat -n pod-pvc.yaml
     1  apiVersion: v1
     2  kind: Pod
     3  metadata:
     4    name: pod-cenos-pvc
     5  spec:
     6    containers:
     7    - name: date-centos-pvc
     8      image: alpine
     9      command: ["sh", "-c"]
    10      args:
    11      - |
    12        exec >> /var/log/date-tail/output.log
    13        echo -n 'Started at: '
    14        while true; do date; sleep 3; done
    15      volumeMounts:                   # Containerのvolume mount
    16      - name: logdate-vol             # mountするVolume名
    17        mountPath: /var/log/date-tail # Container内のmount場所
    18    volumes:
    19    - name: logdate-vol               # mountするVolume名
    20      hostPath:                       #
    21        path: /var/lib/docker/output  # ホストの保存場所
    22    terminationGracePeriodSeconds: 0  # PodにSIGKILLが送信されるまでの猶予
    23
root@master01:~/repos/ub18k8s/manifest#
```
Podを作成してデプロイ先ノードを確認する。
```
root@master01:~/repos/ub18k8s/manifest# kubectl apply -f pod-pvc.yaml
pod/pod-cenos-pvc created
root@master01:~/repos/ub18k8s/manifest# kubectl get pod -o wide
NAME            READY   STATUS    RESTARTS   AGE   IP              NODE     NOMINATED NODE   READINESS GATES
pod-cenos-pvc   1/1     Running   0          19s   10.100.140.67   node02   <none>           <none>
root@master01:~/repos/ub18k8s/manifest#
```
ノード０２に作成されている。node上でtailしてみる。
```
root@node02:~# ll /var/lib/docker/output/
total 12
drwxr-xr-x  2 root root 4096 Mar 27 05:58 ./
drwx--x--x 15 root root 4096 Mar 27 05:58 ../
-rw-r--r--  1 root root 1172 Mar 27 06:00 output.log
root@node02:~# tail -f  /var/lib/docker/output/output.log
Fri Mar 27 06:00:23 UTC 2020
Fri Mar 27 06:00:26 UTC 2020
Fri Mar 27 06:00:29 UTC 2020
Fri Mar 27 06:00:32 UTC 2020
Fri Mar 27 06:00:35 UTC 2020
Fri Mar 27 06:00:38 UTC 2020
Fri Mar 27 06:00:41 UTC 2020
Fri Mar 27 06:00:44 UTC 2020
Fri Mar 27 06:00:47 UTC 2020
Fri Mar 27 06:00:50 UTC 2020
Fri Mar 27 06:00:53 UTC 2020
Fri Mar 27 06:00:56 UTC 2020
^C
```
Podを一度削除して再作成してみる。コンテナ上でtailしてみると保存されたファイルの続き★から書き込んでいる。
```
root@master01:~/repos/ub18k8s/manifest# kubectl delete pod pod-cenos-pvc
pod "pod-cenos-pvc" deleted
root@master01:~/repos/ub18k8s/manifest#
root@master01:~/repos/ub18k8s/manifest# kubectl apply -f pod-pvc.yaml
pod/pod-cenos-pvc created
root@master01:~/repos/ub18k8s/manifest# kubectl exec -it pod-cenos-pvc -- tail -f /var/log/date-tail/output.log
Fri Mar 27 06:01:08 UTC 2020
Fri Mar 27 06:01:11 UTC 2020
Fri Mar 27 06:01:14 UTC 2020
Fri Mar 27 06:01:17 UTC 2020
Started at: Fri Mar 27 06:01:53 UTC 2020★
Fri Mar 27 06:01:56 UTC 2020
Fri Mar 27 06:01:59 UTC 2020
Fri Mar 27 06:02:02 UTC 2020
Fri Mar 27 06:02:05 UTC 2020
Fri Mar 27 06:02:08 UTC 2020
^Ccommand terminated with exit code 130
root@master01:~/repos/ub18k8s/manifest#
```
