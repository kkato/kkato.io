---
title: "NodePortを試してみる"
date: 2024-06-08T12:10:01+09:00
draft: false
tags: ["kubernetes"]
---

最近おうちk8sクラスタを作成したので、NodePortを使ってNginx Podにアクセスしてみようと思います。

まずはNginxのPodとServiceを作成します。
`nginx.yaml`というファイルに以下の内容を記述します。
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      nodePort: 30080
      targetPort: 80
```

上記のマニフェストをapplyします。

```console
k apply -f nginx.yaml
```

PodとServiceが問題なく作成されたら、k8sクラスタのWorkerノードのプライベートアドレスに上記で設定したNodePortの番号を指定してアクセスします。
（ノードのプライベートアドレスは適宜置き換えてください。）

```console
http://192.168.10.122:30080
```

問題なく画面が表示されました &#x1f389;

![](/images/nodeport/nginx.png)

## まとめ
簡単な動作確認も含めて、NodePortを試してみました。
あまり難しいことはしていませんが、ちゃんと画面が表示されるとちょっと嬉しいですね。
