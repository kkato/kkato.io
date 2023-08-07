---
title: "Helm Chartのvaluesを確認する方法"
date: 2023-08-07T21:32:46+09:00
draft: false
---

Helm Chartのvaluesを確認する方法をよく忘れてしまうので、備忘録としてメモを残しておきます。

## valuesを確認するまでの流れ

まずはchart repositoriesを追加します。(今回はbitnami/thanosを例にご紹介します。)
```sh
$ helm repo add bitnami https://charts.bitnami.com/bitnami
```

次に追加されたリポジトリを確認します。
```sh
$ helm search repo bitnami/thanos
NAME          	CHART VERSION	APP VERSION	DESCRIPTION                                       
bitnami/thanos	12.10.1      	0.31.0     	Thanos is a highly available metrics system tha... 
```

そして、valuesをyamlファイルとして書き出します。
```sh
$ helm show values bitnami/thanos --version 12.10.1 > values_thanos-v12.10.1.yaml
```

## 参考
- https://helm.sh/docs/helm/helm/