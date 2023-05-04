---
title: "Raspberry Piでk8sクラスタを構築する"
date: 2023-05-01T10:07:31+09:00
draft: false
tags: ["kubernetes", "raspi"]
---

## はじめに

最近ラズパイを手に入れたので、kubeadmを使ってk8sクラスタを組んでみたいと思います。

![](/images/raspi/raspi_cluster.jpg)

準備したものは以下のとおりです。
| アイテム | 個数 |
|---|---|
| [Raspberry Pi 4 Model B / 4GB](https://www.switch-science.com/products/5680)  | 4 |
| [Raspberry Pi PoE+ HAT](https://www.switch-science.com/products/7172?_pos=1&_sid=e013ece27&_ss=r) | 4 |
| [ケース](https://www.amazon.co.jp/dp/B07TJ15YL1/ref=twister_B07TJZG2F2?_encoding=UTF8&th=1)  | 1 |
| [microSD 64GB](https://www.amazon.co.jp/%E3%83%90%E3%83%83%E3%83%95%E3%82%A1%E3%83%AD%E3%83%BC-Nintendo-%E3%83%89%E3%83%A9%E3%82%A4%E3%83%96%E3%83%AC%E3%82%B3%E3%83%BC%E3%83%80%E3%83%BC-%E3%83%87%E3%83%BC%E3%82%BF%E5%BE%A9%E6%97%A7%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9%E5%AF%BE%E5%BF%9C-RMSD-128U11HA/dp/B0937BHBQC/ref=sr_1_2?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&crid=ZWQLD0ODN6Q1&keywords=SD%E3%82%AB%E3%83%BC%E3%83%89%2Bbuffalo&qid=1680933109&s=electronics&sprefix=sd%E3%82%AB%E3%83%BC%E3%83%89%2Bbuffalo%2Celectronics%2C208&sr=1-2&th=1) | 4 |
| [スイッチングハブ](https://www.amazon.co.jp/%E3%80%90Amazon-co-jp%E9%99%90%E5%AE%9A%E3%80%91TP-Link-%E3%82%B9%E3%82%A4%E3%83%83%E3%83%81%E3%83%B3%E3%82%B0%E3%83%8F%E3%83%96-1000Mbps-%E3%83%A9%E3%82%A4%E3%83%95%E3%82%BF%E3%82%A4%E3%83%A0%E4%BF%9D%E8%A8%BC-TL-SG105V4-0/dp/B00A128S24/ref=sr_1_6?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&crid=1BI99DUS9HW2V&keywords=TP-link+%E3%82%B9%E3%82%A4%E3%83%83%E3%83%81&qid=1680933201&sprefix=tp-link+%E3%82%B9%E3%82%A4%E3%83%83%E3%83%81%2Caps%2C210&sr=8-6) | 1 |
| [LANケーブル 0.15m](https://www.amazon.co.jp/%E3%80%902009%E5%B9%B4%E3%83%A2%E3%83%87%E3%83%AB%E3%80%91ELECOM-LAN%E3%82%B1%E3%83%BC%E3%83%96%E3%83%AB-%E3%80%90PlayStation-LD-GPA-BU1/dp/B07KSFTVJ7/ref=sr_1_8?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&crid=1V64PASGC7ZS3&keywords=LAN%E3%82%B1%E3%83%BC%E3%83%96%E3%83%AB&qid=1680933313&sprefix=lan%E3%82%B1%E3%83%BC%E3%83%96%E3%83%AB%2Caps%2C230&sr=8-8&th=1) | 4 |
| [LANケーブル 1m](https://www.amazon.co.jp/%E3%80%902009%E5%B9%B4%E3%83%A2%E3%83%87%E3%83%AB%E3%80%91ELECOM-LAN%E3%82%B1%E3%83%BC%E3%83%96%E3%83%AB-%E3%80%90PlayStation-LD-GPA-BU1/dp/B076J9RMBY/ref=sr_1_8?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&crid=1V64PASGC7ZS3&keywords=LAN%E3%82%B1%E3%83%BC%E3%83%96%E3%83%AB&qid=1680933313&sprefix=lan%E3%82%B1%E3%83%BC%E3%83%96%E3%83%AB%2Caps%2C230&sr=8-8&th=1) | 1 |
| [SDカードリーダー](https://www.amazon.co.jp/%E3%83%90%E3%83%83%E3%83%95%E3%82%A1%E3%83%AD%E3%83%BC-%E3%83%9D%E3%83%BC%E3%82%BF%E3%83%96%E3%83%AB%E3%82%AB%E3%83%BC%E3%83%89%E3%83%AA%E3%83%BC%E3%83%80%E3%83%BC%E3%80%90-microSDXC-microSDHC-BSCR125U3CBK/dp/B0BCDMPLX5/ref=sr_1_5?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&crid=20QJ9IRPRRFYM&keywords=buffalo+sd%E3%82%AB%E3%83%BC%E3%83%89%E3%83%AA%E3%83%BC%E3%83%80%E3%83%BC&qid=1680933443&sprefix=buffalo+sd%E3%82%AB%E3%83%BC%E3%83%89%E3%83%AA%E3%83%BC%E3%83%80%E3%83%BC%2Caps%2C216&sr=8-5) | 1 |
| [HDMI変換アダプター](https://www.amazon.co.jp/%E3%82%A8%E3%83%AC%E3%82%B3%E3%83%A0-%E5%A4%89%E6%8F%9B%E3%82%A2%E3%83%80%E3%83%97%E3%82%BF-%E3%83%A1%E3%82%B9-HDMI-Micro-AD-HDAD3BK/dp/B01017VGR2?th=1)| 1 |

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
```
$ sudo spy update
$ sudo apt upgrade -y
```

新しくユーザを作成し、ユーザにsudo権限を付与します。sudoをパスワードなしでできるように追加の設定もします。
```
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
```
$ arp -an
? (192.168.10.111) at dc:a6:32:70:52:2a [ether] on wlp0s20f3
? (192.168.10.113) at e4:5f:01:e2:56:aa [ether] on wlp0s20f3
? (192.168.10.114) at e4:5f:01:e2:57:88 [ether] on wlp0s20f3
? (192.168.10.112) at e4:5f:01:e2:57:98 [ether] on wlp0s20f3
? (192.168.10.1) at 80:22:a7:26:71:5c [ether] on wlp0s20f3
```

## kubeadmのインストール

### ポートの開放
kubernetesのコンポーネントが互いに通信するために、[これらのポート](https://kubernetes.io/ja/docs/reference/networking/ports-and-protocols/)を開く必要があります。

ufwコマンドを使って、コントロールプレーンノードのポートを開放します。
```
$ sudo ufw enable
$ sudo ufw allow 22/tcp
$ sudo ufw allow 6443/tcp
$ sudo ufw allow 2379:2380/tcp
$ sudo ufw allow 10250/tcp
$ sudo ufw allow 10259/tcp
$ sudo ufw allow 10257/tcp
```
ポートが開放されたことを確認します。
```
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

続いて各ワーカーノードのポートを開放します。
```
$ sudo ufw enable
$ sudo ufw allow 22/tcp
$ sudo ufw allow 10250/tcp
$ sudo ufw allow 30000:32767/tcp
```
ポートが開放されたことを確認します。
```
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

2023年5月現在では、containeredやCRI-Oなどがコンテナランタイムとしてサポートされています。
> Kubernetes supports container runtimes such as containerd, CRI-O, and any other implementation of the Kubernetes CRI (Container Runtime Interface).

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

```
sudo modprobe overlay
sudo modprobe br_netfilter
```

```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

```
sudo sysctl --system
```

```
sudo apt install containerd -y
```

### kubeadm, kubelet, kubectlのインストール

```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

```
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

```
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## kubernetesクラスタの作成

### Control Planeノードのデプロイ

```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
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

```
sudo kubeadm join 192.168.10.111:6443 --token s0px1g.7s2e6kwrj5qaiysr \
	--discovery-token-ca-cert-hash sha256:bbcfefdab5e92525d070ff0f7a8de077d72bad39f897193a288486f76462424d
```


### Workerノードのデプロイ



## おわりに

## 参考
- [kubeadmのインストール](https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
- [コンテナランタイム](https://kubernetes.io/ja/docs/setup/production-environment/container-runtimes/)
- [kubeadmを使用したクラスターの作成](https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)