---
title: "kubesprayでk8sクラスタを構築する"
date: 2023-04-10T18:05:06+09:00
draft: false
categories: "tech"
tags: ["kubernetes"]
---

## はじめに

k8s構築ツールはいろいろありますが、公式ドキュメントでは以下が紹介されています。

- [学習環境](https://kubernetes.io/ja/docs/setup/learning-environment/)
    - Minikube
    - Kind
- [プロダクション環境](https://kubernetes.io/ja/docs/setup/production-environment/)
    - kubeadm
    - kops
    - kubespray

※各ツールの違いについては[こちら](https://github.com/kubernetes-sigs/kubespray/blob/master/docs/comparisons.md)

今回はその中でも[kubespray](https://github.com/kubernetes-sigs/kubespray)というツールを使ってk8sクラスタを構築してみようと思います。kubeadmは1台ずつインストールを実施するのに対し、kubesprayはAnsibleを使い、各ノードで一斉にインストールを実施します。なので、kubesprayは台数が多いときに便利です。

ちなみに、kubesprayは内部でkubeadmを使っているので、kubespray = kubeadm + Ansibleという感じです。また、kubesprayを使うと、コンテナランタイム(デフォルトはcontainerd)やPod間通信のネットワークプラグイン(デフォルトはcalico)などが自動でインストールされるので、非常に便利です。

## 構築

前提: 
- ベアメタル(Intel NUC11PAHi5)上に構築
- Control Planex1台とWorkerx3台の4台構成
- OSはRocky Linux9.1
- ルーター側の設定で固定IPを割り当て
- 各ノードのスペックは以下

| CPU   | メモリ | ストレージ |
|-------|--------|------------|
| 4コア | 16GB   | 500GB      |

### ssh公開認証の設定
手元のPCからパスワードなしでssh接続できるように公開鍵認証の設定をします。
ssh-keygenのパスワードには空文字を指定します。
```
kkato@bastion:~$ ssh-keygen
kkato@bastion:~$ scp ~/.ssh/id_rsa.pub 192.168.10.xxx:~/
```

公開鍵の情報を各ノードのauthorized_keysに追記します。
```
kkato@nuc01:~$ mkdir ~/.ssh
kkato@nuc01:~$ chmod 700 /.ssh
kkato@nuc01:~$ cat id_rsa.pub >> ~/.ssh/authorized_keys
kkato@nuc01:~$ chmod 600 ~/.ssh/authorized_keys
```

### /etc/hostsの編集
手元のPCから各ノードへホスト名で接続できるように、/etc/hostsを編集します。
```
kkato@bastion:~$ cat /etc/hosts
---
192.168.10.121 nuc01
192.168.10.122 nuc02
192.168.10.123 nuc03
192.168.10.124 nuc04
```

### ユーザーの設定
各ノードのユーザーがパスワードなしでsudo実行できるよう設定します。
```
kkato@nuc01:~$ sudo visudo
---
kkato   NOPASSWD:ALL
```

### Firewallの無効化
各ノードのFirewallを無効化します。
```
kkato@nuc01:~$ sudo systemctl stop firewalld
kkato@nuc01:~$ sudo systemctl disable firewalld
kkato@nuc01:~$ sudo systemctl status firewalld
```

### kubesprayのダウンロード
kubesprayのgitリポジトリをクローンし、最新バージョンのブランにに移動します。
```
kkato@bastion:~$ git clone https://github.com/kubernetes-sigs/kubespray.git
kkato@bastion:~$ cd kubespray
kkato@bastion:~/kubespray$ git branch -a
kkato@bastion:~/kubespray$ git checkout remotes/origin/release-2.21
```

### 必要なパッケージのインストール
必要なパッケージをインストールします。
```
kkato@bastion:~/kubespray$ sudo pip3 install -r requirements.txt
```

### インベントリファイルの編集
Ansibleのインベントリファイルの雛形を作成します。
```
kkato@bastion:~/kubespray$ cp -rfp inventory/sample inventory/mycluster
kkato@bastion:~/kubespray$ declare -a IPS=(192.168.10.xxx 192.168.10.xxx 192.168.10.xxx 192.168.10.xxx)
kkato@bastion:~/kubespray$ CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```

作成されたインベントリファイルを編集します。
```
kkato@bastion:~/kubespray$ cat inventory/mycluster/hosts.yml
---
all:                                                                            
  hosts:                                                                        
    nuc01:                                                             
      ansible_host: 192.168.10.121                                              
      ip: 192.168.10.121                                                        
      access_ip: 192.168.10.121                                                 
    nuc02:                                                                   
      ansible_host: 192.168.10.122                                              
      ip: 192.168.10.122                                                        
      access_ip: 192.168.10.122                                                 
    nuc03:                                                                   
      ansible_host: 192.168.10.123                                              
      ip: 192.168.10.123                                                        
      access_ip: 192.168.10.123                                                 
    nuc04:                                                                   
      ansible_host: 192.168.10.124                                              
      ip: 192.168.10.124                                                        
      access_ip: 192.168.10.124                                                 
  children:                                                                     
    kube_control_plane:                                                         
      hosts:                                                                    
        nuc01:                                                         
    kube_node:                                                                  
      hosts:                                                                    
        nuc02:                                                               
        nuc03:                                                               
        nuc04:                                                               
    etcd:                                                                       
      hosts:                                                                    
        nuc01:                                                         
    k8s_cluster:                                                                
      children:                                                                 
        kube_control_plane:                                                     
        kube_node:                                                              
    calico_rr:                                                                  
      hosts: {}  
```

`inventory/mycluster/group_vars/all/all.yml`や`inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml`のパラメータを確認・変更します。
以下では、kubectlを手元のPCにインストールする、`~/.kube/config`の設定ファイルを作成するように設定しています。
```
kkato@bastion:~/kubespray$ diff -r inventory/sample/group_vars/k8s_cluster/k8s-cluster.yml inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml
256c256
< # kubeconfig_localhost: false
---
> kubeconfig_localhost: true
260c260
< # kubectl_localhost: false
---
> kubectl_localhost: true
```


### kubespray実行
kubesprayを実行し、`failed=0`になっていることを確認します。
```
kkato@bastion:~/kubespray$ ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml
---
PLAY RECAP ************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
nuc01                      : ok=727  changed=67   unreachable=0    failed=0    skipped=1236 rescued=0    ignored=7   
nuc02                      : ok=506  changed=34   unreachable=0    failed=0    skipped=755  rescued=0    ignored=1   
nuc03                      : ok=506  changed=34   unreachable=0    failed=0    skipped=754  rescued=0    ignored=1   
nuc04                      : ok=506  changed=34   unreachable=0    failed=0    skipped=754  rescued=0    ignored=1
```

### kubectlの設定
kubectlがインスールされていることを確認します。そして、kubeconfigを`~/.kube/config`に配置します。
```
kkato@bastion:~/kubespray$ ls /usr/local/bin/ | grep kubectl
kubectl
kkato@bastion:~/kubespray$ cp -ip inventory/mycluster/artifacts/admin.conf ~/.kube/config
```

kubectlでタブ補完がされるように設定します。また、毎回kubectlと打つのはめんどくさいので、kというエイリアスを設定します。
```
kkato@bastion:~$ sudo apt install bash-completion
kkato@bastion:~$ cho 'source <(kubectl completion bash)' >> ~/.bashrc
kkato@bastion:~$ kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl
kkato@bastion:~$ echo 'alias k=kubectl' >> ~/.bashrc
kkato@bastion:~$ echo 'complete -F __start_kubectl k' >> ~/.bashrc
```

### 動作確認
kubectlコマンドを実行すると、先程設定したノードが認識されていることがわかります。
```
kkato@bastion:~$ k get nodes 
NAME    STATUS   ROLES           AGE     VERSION
nuc01   Ready    control-plane   5m23s   v1.25.6
nuc02   Ready    <none>          4m8s    v1.25.6
nuc03   Ready    <none>          4m20s   v1.25.6
nuc04   Ready    <none>          4m7s    v1.25.6

```

## おわりに
kubesprayを使ってk8sクラスタを構築しました。想像以上に簡単にk8sクラスタを構築できるので、ぜひ一度試してみてください。

# 参考
- [GitHub - kubernetes-sigs/kubespray: Deploy a Production Ready Kubernetes Cluster](https://github.com/kubernetes-sigs/kubespray)
- [kubesprayを使ったKubernetesのインストール](https://kubernetes.io/ja/docs/setup/production-environment/tools/kubespray/)