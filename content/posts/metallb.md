---
title: "MetalLB　入門"
date: 2023-08-06T18:08:59+09:00
draft: true
tags: ["k8s"]
---

LoadBalancerタイプのServiceは基本的にAWS, Azure, GCPなどのパブリッククラウドのみで使えます。
しかし、MetalLBというOSSを使うとオンプレミスでもLoadBalancerタイプのServiceが使えるようになります。

今回はMetalLBの使い方を簡単にまとめてみました。

## インストール
まずはインストール前の準備として、kube-proxyをIPVSモードにし、strictARPを有効化します。
```sh
$ kubectl edit configmap -n kube-system kube-proxy
mode: "ipvs"
ipvs:
  strictARP: true
```

マニフェストをapplyすることでMetalLBをインストールします。(他にもkustomize, helmなどが使えます。)
```sh
$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml
```

インストールすると、`metallb-system`というnamespace配下に`controller`というdeploymentと`speaker`というデーモンセットが作成されます。(以下は私がインストールした際のもの)
```sh
$ k get all -n metallb-system 
NAME                              READY   STATUS    RESTARTS      AGE
pod/controller-66d6955457-qp884   1/1     Running   1 (80m ago)   81m
pod/speaker-7fxzv                 1/1     Running   0             81m
pod/speaker-cg5kq                 1/1     Running   0             81m
pod/speaker-g5ttq                 1/1     Running   0             81m
pod/speaker-vj7ss                 1/1     Running   0             81m

NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/webhook-service   ClusterIP   10.233.15.209   <none>        443/TCP   81m

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/speaker   4         4         4       4            4           kubernetes.io/os=linux   81m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/controller   1/1     1            1           81m

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/controller-66d6955457   1         1         1       81m

```

## 設定
次は実際にLoadBalancerタイプのServiceを作るための設定をしていきます。
MetalLBには主に2つのモード(L2モードとBGPモード)があり、今回はシンプルなL2モードを使用しようと思います。

ステップは以下のとおりです:
1. [IPAddressPoolカスタムリソースを作成する](#ipaddresspoolカスタムリソースを作成する)
2. [L2Advertisementカスタムリソースを作成する](#l2advertisementカスタムリソースを作成する)
3. [LoadBalancerタイプのServiceを作成する](#loadbalancerタイプのserviceを作成する)

### IPAddressPoolカスタムリソースを作成する
IPAddressPoolでは予めIPアドレスをプールしておき、その中からIPアドレスを払い出します。以下では192.168.0.1~192.168.0.255のIPアドレスをプールしています。
```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: address-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.0.1-192.168.0.255
```

### L2Advertisementカスタムリソースを作成する
L2モードを使うために、L2Advertisementカスタムリソースを作成します。IPAddressPool selectorを使用しない場合は、全てのIPAddressPoolと紐づきます。
```yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb-system
```

### LoadBalancerタイプのServiceを作成する



## 参考
- https://metallb.universe.tf/installation/
- https://metallb.universe.tf/configuration/