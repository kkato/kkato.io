---
title: "Virt-Managerを使用してKVMで仮想マシンを作成する"
date: 2023-06-13T19:29:18+09:00
draft: true
tags: ["linux"]
---

## CUI(virt-install)から作成
以下のコマンドで仮想マシンを作成する。
```
$ virt-install \
--name vm01 \
--os-type linux \
--os-variant rhel8 \
--arch x86_64 \
--vcpus 2 \
--memory 2048 \
--network default \
--disk /var/lib/libvirt/images/pgrex01.img,size=20 \
--location /usr/local/src/rhel-8.4-x86_64-dvd.iso \
--graphics vnc,listen=0.0.0.0 \
--extra-args="console=tty0 console=ttyS0,115200n8r"
```

```
$ virsh start vm01
$ virsh shutdown vm01
```


## GUI(virt-manager)から作成
```
$ vi /etc/default/grub
$ grub2-mkconfig -o /boot/grub2/grub.cfg
```