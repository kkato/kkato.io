---
title: "Linuxで公開鍵認証をする"
date: 2023-07-12T21:23:52+09:00
draft: false
tags: ["linux"]
---

Linuxで公開鍵認証をするときの設定を簡単にまとめました。主に自分用のメモです。

## 接続元での設定

接続元で鍵を生成し、公開鍵を接続先のサーバーへコピーします。

```
ssh-keygen
scp ~/id_rsa.pub username@server:~/
```

## 接続先の設定

接続先のサーバーで公開鍵の情報をauthorized_keysに登録し、適切な権限を設定します。

```
mkdir .ssh
chmod 700 .ssh
cat id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 authorized_keys
 ```

sshdの設定で、公開鍵認証を有効化し、パスワード認証を無効化します。

```
sudo vi /etc/ssh/sshd_config
---
PubkeyAuthentication yes
PasswordAuthentication no
```

sshdを再起動します。

```
systemctl restart sshd
```