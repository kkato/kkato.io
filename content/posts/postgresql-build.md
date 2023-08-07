---
title: "PostgreSQLをソースコードからビルドする"
date: 2023-06-13T19:32:35+09:00
draft: false
tags: ["postgresql"]
---

今回はPostgreSQLをソースからビルドする方法をご紹介します。

予め必要なライブラリをインストールします。

PostgreSQLのビルドに必要なライブラリ:
- GNU make
- Cコンパイラ
- GNU Readlineライブラリ
- zlib
- Flex
- Bison

TAPテスト(PostgreSQLのクライアントツールなどを対象とする追加テスト)に必要なPerlモジュール:
- IPC::Run
- Test::Simple
- Time::HiRes
- Test::Harness

### RHELの場合

PostgreSQLのビルドに必要なライブラリをインストールします。
```
sudo dnf install make gcc readline readline-devel zlib-devel bison flex
```

TAPテストに必要なライブラリをインストールします。
```sh
sudo dnf install perl-CPAN
sudo cpan -i IPC::Run Test::Simple Time::HiRes Test::Harness
```

ドキュメントを生成するためのライブラリは[こちら](https://www.postgresql.jp/document/15/html/docguide-toolsets.html)です。

### Ubuntuの場合
PostgreSQLのビルドに必要なライブラリをインストールします。
```sh
sudo apt install make gcc libreadline-dev zlib1g-dev bison flex
```

TAPテストに必要なライブラリをインストールします。
```sh
sudo apt install libpc-run-perl libtest-simple-perl libtime-hires-perl libtest-harness-perl
```

## ビルド

PostgreSQLのgitリポジトリをクローンします。
```sh
git clone git://git.postgresql.org/git/postgresql.git
cd postgresql
```

PostgreSQLをコンパイルします。
configureのオプションについては[こちら](https://www.postgresql.jp/document/15/html/install-procedure.html#CONFIGURE-OPTIONS)を参考にしてください。
```sh
./configure --enable-debug --enable-cassert --enable-tap-tests --prefix=$HOME/postgres/pgsql-master CFLAGS=-O0
make -j 4
make install
```


PostgreSQLのリグレッションテストを実行します。
```sh
make -j 4 check-world
```

PostgreSQLを起動します。
```sh
bin/initdb -D data --locale=C --encoding=UTF8
```