---
title: "NUC上にk8sクラスタを構築する"
date: 2024-06-05T21:20:47+09:00
draft: false
tags: ["kubernetes"]
---

しばらく放置していたNUC上にk8sをインストールして、おうちクラスタを運用していこうと思います。
今回はkubeadmを使ってk8sをインストールしようと思います。

![](/images/nuc/nuc.jpg)

## 前提

- ベアメタル(Intel NUC11PAHi5)上に構築
- Control Planex1台とWorkerx3台の4台構成
- OSはRocky Linux9.3
- ルーター側の設定で固定IPを割り当て
- 各ノードのスペックは以下

| CPU   | メモリ | ストレージ |
|-------|--------|------------|
| 4コア | 16GB   | 500GB      |

## kubeadmのインストール

以下の手順を参考にします。（「kubeadm, kubelet, kubectl」のインストールの記述が古かったので、英語版を参照しました。）
- [Installing kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

### 始める前に

swapを無効化します。

```console
sudo swapoff -a
```

### ポートの開放

Control Planeノードのポートを開放します。

```console
sudo firewall-cmd --add-port=6443/tcp --permanent
sudo firewall-cmd --add-port=2379-2380/tcp --permanent
sudo firewall-cmd --add-port=10250/tcp --permanent
sudo firewall-cmd --add-port=10257/tcp --permanent
sudo firewall-cmd --add-port=10259/tcp --permanent
```

ポートが開放されたことを確認します。

```console
sudo firewall-cmd --reload
sudo firewall-cmd --list-ports
```

続いて各Workerノードのポートを開放します。

```coonsole
sudo firewall-cmd --add-port=30000-32767/tcp --permanent
sudo firewall-cmd --add-port=10250/tcp --permanent
```

ポートが開放されたことを確認します。

```console
sudo firewall-cmd --reload
sudo firewall-cmd --list-ports
```
### コンテナランタイムのインストール

まずはカーネルモジュールを起動時に自動でロードするに設定します。
```console
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

以下のコマンドを実行して`br_netfilter`と`overlay`モジュールが読み込まれていることを確認します。

```console
lsmod | grep br_netfilter
lsmod | grep overlay
```

続いて、ネットワーク周りのカーネルパラメータを設定します。

```console
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

以下のコマンドを実行して、`net.bridge.bridge-nf-call-iptables`、`net.bridge.bridge-nf-call-ip6tables`、`net.ipv4.ip_forward`カーネルパラメーターが1に設定されていることを確認します。

```console
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

contaierdをインストールします。

```console
sudo yum config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y containerd.io
sudo sh -c "containerd config default > /etc/containerd/config.toml"
```

その後、以下の設定を「false」から「true」に変更します。

```console
SystemdCgroup = true
```

containerdを有効化します。

```console
systemctl enable --now containerd.service
```

### kubeadm, kubelet, kubectlのインストール

kubeadm, kubelet, kubectlをインストールします。

```console
sudo sh -c "cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF"

# SELinuxをpermissiveモードに設定する(効果的に無効化する)
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

sudo systemctl enable --now kubelet
```

## kubernetesクラスタの作成
以下の手順を参考にします。
- [kubeadmを使用したクラスターの作成](https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

### Control Planeノードのデプロイ

`kubeadm init`コマンドを使ってControl Planeノードをデプロイします。

```console
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

### Workerノードのデプロイ

`kubeadm join`コマンドを使って、Workerノードをデプロイします。

```
sudo kubeadm join 192.168.10.121:6443 --token bccqut.hxu0wkyo2y04i88c \
	--discovery-token-ca-cert-hash sha256:b8db95c90e2d485ac499efb266aef624b464770cc89ff74b8a041fc0a2bab0b9
```

### kubectlのインストール

先にkubectlをインストールします。
[macOS上でのkubectlのインストールおよびセットアップ](https://kubernetes.io/ja/docs/tasks/tools/install-kubectl-macos/)

```console
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | shasum -a 256 --check
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
sudo chown root: /usr/local/bin/kubectl
```

以下を~/.zshrcに記述することで、kubectlのエイリアスが作成され、タブ補完が効くようになります。

```console
alias k=kubectl
autoload -Uz compinit && compinit
source <(kubectl completion zsh)
```

### kubeconfigの設定

kubectlを使ってk8sクラスタ操作できるようにするために、以下を実行します。

```console
mkdir -p $HOME/.kube
scp nuc01:/etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

以下が表示されればOKです。

```console
$ % kubectl get pods
No resources found in default namespace.
```

### アドオンのインストール

CNIプラグインであるFlannelをインストールします。

```console
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

### まとめ

NUCを使ってk8sクラスタを組んでみました。Rocky Linuxへのcontainerdのインストールやkubernetes package repositoryの変更などいくつかはまりましたが、なんとかk8sクラスタを構築することができました。運用がんばるぞ！
