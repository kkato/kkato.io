---
title: "Helm Chartのvaluesを確認する方法"
date: 2023-08-07T21:32:46+09:00
draft: false
---

Helm Chartのvaluesを確認する方法をよく忘れてしまうので、備忘録としてメモを残しておきます。

## valuesを確認するまでの流れ

まずはchart repositoriesを追加します。
```sh
helm repo add [NAME] [URL]
```

次に追加されたリポジトリを確認します。
```sh
helm search repo [KEYWORD]
```

そして、valuesをyamlファイルとして書き出します。
```sh
helm show values [CHART] --version [VERSION] > values.yaml
```

## 参考
- https://helm.sh/docs/helm/helm/