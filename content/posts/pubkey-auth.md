---
title: "Linuxで公開鍵認証をする"
date: 2023-06-13T19:32:35+09:00
draft: true
tags: ["linux"]
---

## 接続元での設定
```
ssh-keygen
scp ~/id_rsa.pub username@server:~/
```

## 接続先の設定
```
mkdir .ssh
chmod 700 .ssh
cat id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 authorized_keys
 ```

```
sudo vi /etc/ssh/sshd_config
---
PubkeyAuthentication yes
PasswordAuthentication no
```

```
systemctl restart sshd
```