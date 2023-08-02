---
title: "EC2 User"
date: 2023-08-02T13:11:46+09:00
draft: true
---


```
sudo su -
useradd -m ユーザ名
usermod -aG wheel ユーザ名
passwd ユーザ名
```

```
mkdir /home/ユーザ名/.ssh
cp /home/ec2-user/.ssh/authrized_keys /home/ユーザ名/.ssh
chown -R ユーザ名:ユーザ名 /home/ユーザ名/.ssh
chmod 700 /home/ユーザ名/.ssh
chmod 600 /home/ユーザ名/.ssh/authorized_keys
```

```
sudo userdel -r ec2-user
```