---
title: "Raspberry Pi上にk8sクラスタを構築する"
date: 2023-04-23T16:13:42+09:00
draft: false
tags: ["kubernetes", "raspberry pi"]
---

最近ラズパイを手に入れたので、kubeadmを使ってk8sクラスタを組んでみたいと思います。
Control Planeノードx1 Workerノードx3の構成です。

![](/images/raspi/raspi_cluster.jpg)

## 準備

準備したものは以下のとおりです。
| アイテム | 個数 |
|---|---|
| [Raspberry Pi 4 Model B / 4GB](https://www.switch-science.com/products/5680)  | 4 |
| [Raspberry Pi PoE+ HAT](https://www.switch-science.com/products/7172?_pos=1&_sid=e013ece27&_ss=r) | 4 |
| [ケース](https://amzn.asia/d/8DQ2Vjn)  | 1 |
| [microSD 64GB](https://amzn.asia/d/1a7lXpn) | 4 |
| [スイッチングハブ](https://amzn.asia/d/6lrvPog) | 1 |
| [LANケーブル 0.15m](https://amzn.asia/d/cSs0irO) | 4 |
| [LANケーブル 1m](https://amzn.asia/d/0IejUmF) | 1 |
| [SDカードリーダー](https://amzn.asia/d/4oTN3E9) | 1 |
| [HDMI変換アダプター](https://amzn.asia/d/a8TZ1OH)| 1 |

PoE(Power over Ethernet)+ HATを使うと、LANケーブルから電源供給できるのでとても便利です。今回はPoE+ HATを使っているので、スイッチングハブもPoE対応のものを購入しています。ラズパイのOSをSDカードにインストールする必要があるので、SDカードリーダーも購入しました。あとは、ディスプレイと繋ぐときにmicro HDMIに変換するためのアダプタも購入しました。

## OSの設定

### OSのインストール
手元のPCはUbuntu 22.04 LTSなので、以下のコマンドでRaspberry Pi Imagerをインストールします。
```
$ sudo apt install rpi-imager
```

そして、microSDカード4枚全てにUbuntu Server 22.10 (64-bit)を焼きます。
```
$ rpi-imager
```
![](/images/raspi/raspi_imager.png)

microSDカードを差し込み、ディスプレイ(micro HDMI)とキーボード(USB)を接続し、OSの初期設定を行います。初期ユーザー名とパスワードはubuntuです。パッケージを最新にしておきます。
```sh
$ sudo spy update
$ sudo apt upgrade -y
```

新しくユーザを作成し、ユーザにsudo権限を付与します。sudoをパスワードなしでできるように追加の設定もします。
```sh
$ sudo adduser kkato

$ sudo usermod -aG sudo kkato

$ sudo visudo
---
kkato ALL=NOPASSWD: ALL
```

### 固定IPの設定
OS側で固定IPを設定する方法もありますが、今回はルーター側で設定してみたいと思います。
まずは以下のコマンドを使って、MACアドレスを確認します。(以下だと、eth0の`dc:a6:32:70:52:2a`です。)
```
$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether dc:a6:32:70:52:2a brd ff:ff:ff:ff:ff:ff
3: wlan0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether dc:a6:32:70:52:2b brd ff:ff:ff:ff:ff:ff
```

Aterm WG1200HS4というルーターを使っており、[http://aterm.me/](http://aterm.me/)から固定IPを設定できます。ここで先ほど確認したMACアドレスと、固定したいIPアドレスを指定します。その後、ルーターを再起動し、設定が反映されているか確認します。
![](/images/raspi/router.png)

手元のPCからきちんと設定されているか確認します。(111~114がラズパイです。)
```sh
$ arp -an
? (192.168.10.111) at dc:a6:32:70:52:2a [ether] on wlp0s20f3
? (192.168.10.113) at e4:5f:01:e2:56:aa [ether] on wlp0s20f3
? (192.168.10.114) at e4:5f:01:e2:57:88 [ether] on wlp0s20f3
? (192.168.10.112) at e4:5f:01:e2:57:98 [ether] on wlp0s20f3
? (192.168.10.1) at 80:22:a7:26:71:5c [ether] on wlp0s20f3
```

## kubeadmのインストール
以下の手順を参考にします。
- [kubeadmを使用したクラスターの作成](https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
- [コンテナランタイム](https://kubernetes.io/ja/docs/setup/production-environment/container-runtimes/)
- [kubeadmのインストール](https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

### ポートの開放
kubernetesのコンポーネントが互いに通信するために、[これらのポート](https://kubernetes.io/ja/docs/reference/networking/ports-and-protocols/)を開く必要があります。

ufwコマンドを使って、Control Planeノードのポートを開放します。
```sh
# Control Planeノードで実行
$ sudo ufw enable
$ sudo ufw allow 22/tcp
$ sudo ufw allow 6443/tcp
$ sudo ufw allow 2379:2380/tcp
$ sudo ufw allow 10250/tcp
$ sudo ufw allow 10259/tcp
$ sudo ufw allow 10257/tcp
```
ポートが開放されたことを確認します。
```sh
$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere  
6443/tcp                   ALLOW       Anywhere                  
2379:2380/tcp              ALLOW       Anywhere                  
10250/tcp                  ALLOW       Anywhere                  
10259/tcp                  ALLOW       Anywhere                  
10257/tcp                  ALLOW       Anywhere                                  
```

続いて各Workerノードのポートを開放します。
```sh
# 各Workerノードで実行
$ sudo ufw enable
$ sudo ufw allow 22/tcp
$ sudo ufw allow 10250/tcp
$ sudo ufw allow 30000:32767/tcp
```
ポートが開放されたことを確認します。
```sh
# 各Workerノードで実行
$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
10250/tcp                  ALLOW       Anywhere                  
30000:32767/tcp            ALLOW       Anywhere                  
```

### コンテナランタイムのインストール
コンテナランタイムとはkubernetesノード上のコンテナとコンテナイメージを管理するためのソフトウェアです。

2023年5月現在では、containeredやCRI-Oなどがコンテナランタイムとしてサポートされています。今回はcontainerdをインストールしようと思います。
> Kubernetes supports container runtimes such as containerd, CRI-O, and any other implementation of the Kubernetes CRI (Container Runtime Interface).

まずはカーネルモジュールを起動時に自動でロードするに設定します。`overlay`はコンテナに必要で、`br_netfilter`はPod間通信のために必要です。
```sh
# 各ノードで実行
$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

続いて、ネットワーク周りのカーネルパラメータを設定します。以下を設定することにより、iptablesを使用してブリッジのトラフィック制御が可能になります。
```sh
# 各ノードで実行
$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

$ sudo sysctl --system
```

最後にcontainerdをインストールします。
```sh
# 各ノードで実行
$ sudo apt install containerd -y
```

### kubeadm, kubelet, kubectlのインストール

aptのパッケージ一覧を更新し、Kubernetesのaptリポジトリを利用するのに必要なパッケージをインストールします。
```sh
# 各ノードで実行
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

Google Cloudの公開鍵をダウンロードします。
```sh
# 各ノードで実行
$ sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

Kubernetesのaptリポジトリを追加します。
```sh
# 各ノードで実行
$ echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

aptのパッケージ一覧を更新し、kubelet、kubeadm、kubectlをインストールします。そしてバージョンを固定します。
```sh
# 各ノードで実行
$ sudo apt-get update
$ sudo apt-get install -y kubelet kubeadm kubectl
$ sudo apt-mark hold kubelet kubeadm kubectl
```

## kubernetesクラスタの作成
以下の手順を参考にします。
- [kubeadmを使用したクラスターの作成](https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
- [kubectlのインストールおよびセットアップ](https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/)


### Control Planeノードのデプロイ
`kubeadm init`コマンドを使ってControl Planeノードをデプロイします。

```sh
# Control Planeノードで実行
$ sudo kubeadm init
---
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:
```

### アドオンのインストール

CNIプラグインであるFlannelをインストールします。\
https://github.com/flannel-io/flannel#deploying-flannel-manually
```sh
$ kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

### Workerノードのデプロイ
`kubeadm join`コマンドを使って、Workerノードをデプロイします。

```sh
# 各Workerノードで実行
$ sudo kubeadm join 10.168.10.111:6443 --token s0px1g.7s2e6kwrj5qaiysr \
	  --discovery-token-ca-cert-hash sha256:bbcfefdab5e92525d070ff0f7a8de077d72bad39f897193a288486f76462424d
```

### kubectlのインストール

kubectlのバイナリをダウンロードします。
```sh
# 手元のPCで実行
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

バイナリを実行可能にします。
```sh
# 手元のPCで実行
$ chmod +x ./kubectl
```

kubectlをPATHの中に移動します。
```sh
# 手元のPCで実行
$ sudo mv ./kubectl /usr/local/bin/kubectl
```

kubectlのタブ補完を設定します。
```sh
# 手元のPCで実行
$ sudo dnf install bash-completion
$ echo 'source <(kubectl completion bash)' >>~/.bashrc
$ kubectl completion bash >/etc/bash_completion.d/kubectl
$ echo 'alias k=kubectl' >>~/.bashrc
$ echo 'complete -F __start_kubectl k' >>~/.bashrc
```

### kubeconfigの設定

kubeadm initコマンド実行後に表示された説明に沿って、kubeconfigを設定します。
```sh
# 手元のPCで実行
$ mkdir -p $HOME/.kube
$ scp raspi01:/etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 接続確認
k8sクラスタに接続できることを確認します。
```sh
# 手元のPCで実行
$ k get pods
No resources found in default namespace.
```

## まとめ
ラズパイを使ってk8sクラスタを組んでみました。ずっとお家k8sクラスタを構築してみたいと思っていたので、やっと実現できてよかったです。k8sの構築方法を一通り体験したので、これでk8sを完全理解したと言えますね。