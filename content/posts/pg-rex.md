---
title: "PG-REX構築メモ"
date: 2023-06-13T19:33:49+09:00
draft: true
tags: ["postgresql"]
---

## 仮想マシン作成



## NIC追加

| s-lan          | d-lan          | ic-lan         | stonith-lan      |
| -------------- | -------------- | -------------- | ---------------- |
| 192.168.0.1/24 | 192.168.1.1/24 | 192.168.2.1/24 | 172.20.144.41/24 |

既に存在する仮想ネットワーク(default)のxmlを参考に、新しい仮想ネットワークを定義するためのxmlを作成する。
```
virsh net-dumpxml default > s-lan.xml
```

![](https://hackmd.io/_uploads/SkB4PunV3.png)


xmlを編集し終わったら、以下のコマンドで仮想ネットワークを作成する。
```
virsh net-create s-lan.xml
```

ちゃんと仮想ネットワークが作成されているか確認する。
```
virsh net-list
```
![](https://hackmd.io/_uploads/SkvcPuhE2.png)


仮想マシンにNICを追加する。
```
virsh attach-interface --type network --source s-lan --model virtio pgrex01 --config
```

ちゃんと仮想マシンにNICが追加されたか確認する。
```
virsh domiflist pgrex01
```
![](https://hackmd.io/_uploads/Sk4jvd2V3.png)


上で確認したNICのMACアドレスを`<host mac=''`に指定する。

そうすることで、IPを固定することができる。
```
virsh net-edit s-lan
```
![](https://hackmd.io/_uploads/HJd6wu3Nn.png)
chmod 600 authorized_keys


仮想ネットワークを起動する。
```
virsh net-start s-lan
virsh net-autostart s-lan
```

その他役立ったコマンド集: 
```
virsh net-destroy s-lan #仮想ネットワークの停止
virsh net-undefine s-lan #仮想ネットワークの削除
```


## ネットワーク設定

仮想マシンのコンソールに入る。
```
virsh console pgrex01
```

仮想マシンに入ると新しく追加したNICが反映されている。
```
nmcli
```
![](https://hackmd.io/_uploads/HyeY_u2Eh.png)


deviceは既に作成されているので、connectionを作成する。
```
nmcli con add type ethernet con-name enp7s0 ifname enp7s0
```

connectionが正常に起動しているか確認する。
```
nmcli
```
![](https://hackmd.io/_uploads/ryxZwOOh4h.png)


## Pacemakerインストール

Pacemakerをインストールするためには、RHELのインストールDVD、RHEL HA Add-onのISOイメージ、pm_extra_toolsを準備する必要がある。
> [RHELのインストールDVDとRHEL HA Add-onのISOイメージはどちらもrhel-8.5-x86_64-dvd.isoという名前なので注意！]


RHELのインストールDVDを仮想マシンに追加する。設定を反映させるためには仮想マシンを再起動する必要がある。
```
virsh attach-disk pgrex01 /usr/local/src/rhel-8.5-x86_64-dvd.iso vdb --config
```

RHEL HA Add-onのISOイメージとpm_extra_toolsを仮想マシンに転送する。
```
scp rhel-8.4-x86_64-dvd.iso root@192.168.0.2:~
```

利用マニュアルにしたがってPacemakerをインストールする。
<details><summary>利用マニュアル</summary>

mountコマンドでRHELのインストールDVDを/mediaにマウントする。
```
mount -o ro /dev/sdb /media
```

/mnt/HighAvailabilityを作成し、mountコマンドでRHEL HA Add-OnのISOイメージをマウントする。
```
mkdir /mnt/HighAvailability
mount -o ro /var/tmp/rhel-8.4-x86_64-dvd.iso /mnt/HighAvailability
```
media配下、及び/mnt/HighAvailability配下をyumコマンドで参照されるリポジトリに追加する 設定をする。　
/etc/yum.repos.d配下にrheldvd.repoを新規に作成し、以下の内容を記述します。
```
[BaseOS]
name=Red Hat Enterprise Linux $releasever - $basearch - BaseOS
baseurl=file:///media/BaseOS
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release

[AppStream]
name=Red Hat Enterprise Linux $releasever - $basearch - AppStream
baseurl=file:///media/AppStream
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```

/etc/yum.repos.d配下にrhel-ha.repoを新規に作成し、以下の内容を記述する。
```
[HighAvailability]
name=Red Hat Enterprise Linux $releasever - $basearch - HighAvailability
baseurl=file:///mnt/HighAvailability
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```
    
yumのキャッシュをクリアする。
```
yum clean all
```
    
Pacemakerをインストールする。
```
yum install pcs pacemaker fence-agents-all -y
```
    
pm_extra_toolsをインストールする。
```
yum install pm_extra_tools-1.3-1.el8.noarch.rpm -y
```
    
追加したyumコマンドの/media配下、及び/mnt/HighAvailability配下への参照を無効化する。

/etc/yum.repos.d/rheldvd.repoを編集する。
```
[BaseOS]
name=Red Hat Enterprise Linux $releasever - $basearch - BaseOS
baseurl=file:///media/BaseOS
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release

[AppStream]
name=Red Hat Enterprise Linux $releasever - $basearch - AppStream
baseurl=file:///media/AppStream
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```
    
/etc/yum.repos.d/rhel-ha.repoを編集する。
```
[HighAvailability]
name=Red Hat Enterprise Linux $releasever - $basearch - HighAvailability
baseurl=file:///mnt/HighAvailability
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```
    
umountコマンドでRHELのインストールDVD、RHEL HA Add-OnのISOイメージをアンマウン トする。
```
umount /media
umount /mnt/HighAvailability
```
</details>

<br />

仮想ディスクを仮想マシンから取り外す。
```
virsh detach-disk pgrex01 vdb
```


## HAクラスタ構築

利用マニュアルに従い構築する。

<details><summary>利用マニュアル</summary>

pcsdサービスを起動し、自動起動を有効にする。(2つのノードで実行)
```
systemctl start pcsd.service
systemctl enable pcsd.service
systemctl is-enabled pcsd.service
```
    
haclusterユーザのパスワードを設定する。(2つのノードで実行)
```
passwd hacluster
```
    
各ノードのホスト名、管理用LANのIPアドレスを指定し、ノードの認証を行う。(いずれか１つのノードで実行)

認証の際はユーザ名にhacluster、パスワードに前項で設定したhaclusterのパスワードを入力する。
```
pcs host pgrex01 addr=172.20.144.42 pgrex02 addr=172.20.144.43
```
    
Pacemaker設定ファイルを変更する。(2つのノードで実行)
/etc/sysconfig/pacemakerのPCMK_fail_fastのコメントアウトを外し、以下の通り編集する。
```
PCMK_fail_fast=yes
```
    
ファイヤーウォールがHAクラスタ構築の障壁になる場合がある。
```
systemctl stop firewalld
systemctl disable firewalld
```
</details>

## PostgreSQLインストール

PostgreSQLのインストールに必要なRPMは以下の通り:

- postgresql14-libs-14.0-1PGDG.rhel8.x86_64.rpm
- postgresql14-14.0-1PGDG.rhel8.x86_64.rpm
- postgresql14-server-14.0-1PGDG.rhel8.x86_64.rpm
- postgresql14-contrib-14.0-1PGDG.rhel8.x86_64.rpm
- postgresql14-docs-14.0-1PGDG.rhel8.x86_64.rpm

リンク
https://www.postgresql.org/download/linux/redhat/
 
```
# Install the repository RPM:
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# Disable the built-in PostgreSQL module:
sudo dnf -qy module disable postgresql

# Install PostgreSQL:
sudo dnf install -y postgresql14-libs \
postgresql14 \
postgresql14-server \
postgresql14-contrib \
postgresql14-docs
```


利用マニュアルに従い設定をする。

<details><summary>利用マニュアル</summary>
    
::: warning
本作業はpostgresユーザで実施する。
:::
    
PG-REXではpostgresユーザのuidを26、postgresグループのgidを26であることを前提とする。以下のコマンドを実行し、postgresユーザが規定のuid、gidで作成されていることを確認する。
```
id postgres
```
    
RPMのインストールにより新規でユーザが作成された場合は、パスワードを設定する。
```
passwd postgres
```
    
環境変数の設定
```
export PATH=/usr/pgsql-14/bin:$PATH
export PGDATA=/dbfp/pgdata/data
```
    
DBクラスタ用ディレクトリの作成

::: warning
本作業はrootユーザで実施する。
:::

| ディレクトリ     | パス               | 所有者   | グループ | 権限 |
| ---------------- | ------------------ | -------- | -------- | ---- |
| DBクラスタ用     | /dbfp/pgdata/data  | postgres | postgres | 700  |
| WAL用            | /dbfp/pgwal/pg_wal | postgres | postgres | 700  |
| アーカイブログ用 | /dbfp/pgarch/arc1  | postgres | postgres | 700  |
</details>

::: warning
本作業はpgrex01のみで実施する。
:::

<details><summary>利用マニュアル</summary>
    
::: warning
本作業はpostgresユーザで実施する。
:::

DBクラスタの初期化
```
initdb -D /dbfp/pgdata/data -X /dbfp/pgwal/pg_wal --encoding=UTF-8 --no-locale --data-checksums
```
    
postgresql.confの編集
```
listen_addresses = '*'
password_encryption = scram-sha-256
wal_level = replica
synchronous_commit = on
archive_mode = always
archive_command = '/bin/cp %p /dbfp/pgarch/arc1/%f'
max_wal_senders = 10
wal_keep_size = 512MB
wal_sender_timeout = 20s
max_replication_slots = 10
hot_standby = on
max_standby_archive_delay = -1
max_standby_streaming_delay = -1
hot_standby_feedback = on
wal_receiver_timeout = 20s
restart_after_crash = off
```
    
レプリケーションユーザの作成

PostgreSQLを一度起動する。
```
pg_ctl start
```
    
CREATE ROLEコマンドでレプリケーションユーザを作成する。
```
psql -c "CREATE ROLE repuser REPLICATION LOGIN PASSWORD 'reppasswd'"
```
    
PostgreSQLを停止する。
```
pg_ctl stop
```
    
pg_hba.confの編集
```
 TYPE DATABASE    USER    ADDRESS        METHOD
host   replication repuser 192.168.2.1/32 scram-sha-256
host   replication repuser 192.168.2.2/32 scram-sha-256
```
    
::: warning
本作業はpostgresユーザで実施する。
:::
    
パスワードファイルの作成

postgresユーザのホームディレクトリに、600の権限でパスワードファイル.pgpssを作成する。

レプリケーション受付用の仮想IPアドレス、および相手のノードのD-LANのIPアドレスについて記述する。
```
# hostname:port:database:username:password
192.168.2.4:5432:replication:repuser:reppasswd
192.168.2.3:5432:replication:repuser:reppasswd
```
</details>


## PG-REX運用補助ツールインストール



## トラブルシューティング

/usr/local/bin/pg-rex_primary_start

/usr/local/share/perl5/PGRex/common.pm

