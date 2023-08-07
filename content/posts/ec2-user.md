---
title: "EC2のデフォルトユーザを削除する"
date: 2023-08-02T13:11:46+09:00
draft: false
tags: ["aws"]
---

AWSでEC2インスタンスを建てると、デフォルトで"ec2-user"という名前のユーザが作成されます。公開鍵認証なので高いセキュリティレベルが担保されており、ユーザ名が知られていても特に問題はないと思います。しかし、万が一を考えてデフォルトのec2-userを削除し、新しいユーザでアクセスできるように設定したいと思います。

## ec2-userを削除するまでの流れ

`useradd`コマンドを使い新しくユーザを追加します。追加したユーザにsudo権限を付与します。また、追加したユーザのパスワードも登録します。
```sh
ssh -i ~/.ssh/秘密鍵の名前 ec2-user@EC2インスタンスのパブリックIP
sudo su -
useradd -m ユーザ名
usermod -aG wheel ユーザ名
passwd ユーザ名
```

追加したユーザのホームディレクトリにauthorized_keys(公開鍵の情報)をコピーし、適切な権限を付与します。
```sh
mkdir /home/ユーザ名/.ssh
cp /home/ec2-user/.ssh/authrized_keys /home/ユーザ名/.ssh
chown -R ユーザ名:ユーザ名 /home/ユーザ名/.ssh
chmod 700 /home/ユーザ名/.ssh
chmod 600 /home/ユーザ名/.ssh/authorized_keys
```

その後新しく追加したユーザで接続し直し、接続できることを確認したら、ec2-userを削除します。
```sh
exit
ssh -i ~/.ssh/秘密鍵の名前 ユーザ名@EC2インスタンスのパブリックIP
sudo userdel -r ec2-user
```