---
title: "PostgreSQLをソースコードからビルドする"
date: 2023-06-13T19:17:52+09:00
draft: true
tags: ["postgresql"]
---

## はじめに

今回はPostgreSQLをソースからビルドする方法をご紹介します。 \
参考: https://www.postgresql.jp/document/15/html/install-procedure.html

## 準備

予め必要なライブラリをインストールします。\
RHELとUbuntuに必要なライブラリを紹介します。

### RHELに必要なライブラリ

ソースの入手、コンパイルに必要なライブラリをインストールします。
```
sudo dnf install git gcc make bison flex readline readline-devel zlib-devel
```

TAPテスト(PostgreSQLのクライアントツールなどを対象とする追加テスト)に必要なライブラリをインストールします。
```build-essential libreadline-dev zlib1g-dev
sudo dnf install perl-CPAN
sudo cpan -i IPC::Run Test::Simple Time::HiRes Test::Harness
```

### Ubuntuに必要なライブラリ

```
sudo apt install build-essential libreadline-dev zlib1g-dev libpc-run-perl bison flex libxml2-utils xsltproc docbook-to-man
```

## ビルド手順

PostgreSQLのgitリポジトリをクローンします。
```
git clone git://git.postgresql.org/git/postgresql.git
```

PostgreSQLをコンパイルします。
```
./configure --enable-debug --enable-cassert --enable-tap-tests --prefix=$HOME/postgres/pgsql-master CFLAGS=-O0
make -j 4
make install
```

PostgreSQLのリグレッションテストを実行します。
```
make -j 4 check-world
```

PostgreSQLを起動します。
```
bin/initdb -D data --locale=C --encoding=UTF8
```