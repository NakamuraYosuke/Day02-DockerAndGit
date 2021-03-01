# Day02-DockerとGitを触ってみよう
## Docker編
### Dockerによるコンテナイメージの取得
アプリケーションは、独⽴し制御された実⾏環境をコンテナーに』供する⼿段として、コンテナー内で実⾏できます。

コンテナー化されたアプリケーションを実⾏する、つまりコンテナー内でアプリケーションを実⾏するには、コンテナーイメージ、すべてのアプリケーションファイルを』供するファイルシステムバンドル、ライブラリ、およびアプリケーションの実⾏に必要な依存関係が必要となります。

コンテナーイメージはイメージレジストリーにあります。これによりサービスでは、ユーザーがコンテナーイメージを検索して取得できます。

Dockerユーザーは search サブコマンドを使⽤して、リモートまたはローカルのレジストリーから使⽤可能なイメージを検索できます。

下記の例では、rhel7.6のコンテナーイメージを検索します。
```
$ docker search registry.access.redhat.com/rhel7.6
```
下記のような結果が表示されるはずです。
```
NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
rhel7.6   This platform image provides a minimal runti…   0                    
```

検索したイメージを取得する場合には、pullサブコマンドを使用してローカルのレジストリに保存します。
```
$ docker pull registry.access.redhat.com/rhel7.6
```

下記のような出力がされれば、自身のローカルレジストリにコンテナーイメージを格納されました。
```
sing default tag: latest
latest: Pulling from rhel7.6
c9281c141a1b: Pull complete 
31114e120ca0: Pull complete 
Digest: sha256:6189d115a2898c6c3a6d5f72db760f5062e571e12aabbedd81067facc1152377
Status: Downloaded newer image for registry.access.redhat.com/rhel7.6:latest
registry.access.redhat.com/rhel7.6:latest
```

自身のローカルレジストリにあるイメージの一覧を確認するには、imagesサブコマンドを使用します。
```
$ docker images
REPOSITORY                           TAG       IMAGE ID       CREATED         SIZE
registry.access.redhat.com/rhel7.6   latest    31cd91012c57   19 months ago   203MB
```

### コンテナの実行
先ほど取得したrhel7.6のコンテナイメージをローカルで実行します。

コンテナーイメージは、エントリーポイントとして知られるコンテナー内で開始されるプロセスを指定します。

docker run コマンドは、コンテナーのエントリーポイントコマンドとして、イメージ名後のすべてのパラメーターを使⽤します。

次の例は、Red Hat Enterprise Linux イメージからコンテナーを起動しています。このコンテナーのエントリーポイントは echo "Helloworld" コマンドに設定されています。

```
$ docker run registry.access.redhat.com/rhel7.6 echo "Hello"
```

コンテナの起動状態を確認するには、psサブコマンドを使用します。
```
$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

何も出力されないはずです。これはコンテナの実行をフォアグラウンドで実行したためです。
バックグラウンドで実行するには、`-d`オプションを使用します。


次に、コンテナにログインしてみましょう。
`-it`オプションを付与します。（--interactive、--ttyを使用可能にします）
```
$ docker run -it -d registry.access.redhat.com/rhel7.6
5694e1e05e2e74446d0be0d3fd6d8c51f47845d7e6674e90afac512df5a7c096
```

次に起動中のDockerプロセスを確認します。
```
$ docker ps
CONTAINER ID   IMAGE                                COMMAND       CREATED          STATUS          PORTS     NAMES
5694e1e05e2e   registry.access.redhat.com/rhel7.6   "/bin/bash"   52 seconds ago   Up 51 seconds             stoic_hawking
```

`CONTAINER ID`列に表示されたID（例では5694e1e05e2e）を利用してログインします。

```
docker exec -it 5694e1e05e2e /bin/bash
[root@5694e1e05e2e /]#
```

これでrhel7.6のコンテナ内に入ることが出来ました。

中は通常のRedHat Enterprise Linuxです。
```
# cat /etc/redhat-release 
Red Hat Enterprise Linux Server release 7.6 (Maipo)
```

コンテナから抜ける際は、`exit`と入力するか`control + d`で可能です。

コンテナを停止・削除します。
```
$ docker rm -f 5694e1e05e2e
```

自身のローカルレジストリからrhel7.6イメージを削除する場合には、`rmi`サブコマンドを使用します。
```
$ docker rmi registry.access.redhat.com/rhel7.6
Untagged: registry.access.redhat.com/rhel7.6:latest
Untagged: registry.access.redhat.com/rhel7.6@sha256:6189d115a2898c6c3a6d5f72db760f5062e571e12aabbedd81067facc1152377
Deleted: sha256:31cd91012c575f2b5463140597bf8d988520d5c7a5e7548e40211977e9aa0afe
Deleted: sha256:99f2b64bdf6ecf3142361d1e011d351ef5c176b3103960df7d2e62f69e030100
Deleted: sha256:e10ae1eb4f450b747303a918e08c2fc37ad9965f5f14b364fff6b94b0c187374
```

### コンテナの実行（応用）
コンテナを実行する際、環境変数を渡してあげることが出来ます。
MySQLのコンテナを取得し、ユーザ/パスワード/データベース名/ルートパスワードを渡して起動します。

MySQLのコンテナイメージを取得します。
```
$ docker pull registry.access.redhat.com/rhscl/mysql-57-rhel7
Using default tag: latest
latest: Pulling from rhscl/mysql-57-rhel7
1c9f515fc6ab: Pull complete 
1d2c4ce43b78: Pull complete 
f1e961fe4c51: Pull complete 
9f1840c3b3bd: Pull complete 
Digest: sha256:88d5bc2fbdf703c0b0e072751af2cd54fb527649433f38feb359489b252ec905
Status: Downloaded newer image for registry.access.redhat.com/rhscl/mysql-57-rhel7:latest
registry.access.redhat.com/rhscl/mysql-57-rhel7:latest
```

MySQLコンテナをmysql-basicというコンテナ名で起動します。
下記の環境変数を利用します。
```
MYSQL_USER=user1
MYSQL_PASSWORD=mypa55
MYSQL_DATABASE=items
MYSQL_ROOT_PASSWORD=r00tpa55
```

```
$ docker run --name mysql-basic -e MYSQL_USER=user1 -e MYSQL_PASSWORD=mypa55 -e MYSQL_DATABASE=items -e MYSQL_ROOT_PASSWORD=r00tpa55 -d registry.access.redhat.com/rhscl/mysql-57-rhel7:latest
```

エラーなしで起動したことを確認します。
```
$ docker ps  --format "{{.ID}} {{.Image}} {{.Names}}"
00189317502b registry.access.redhat.com/rhscl/mysql-57-rhel7:latest mysql-basic
```

次に、起動したMySQLコンテナにログインします。
```
$ docker exec -it mysql-basic /bin/bash
bash-4.2$
```

MySQLデータベースにログインします。
```
bash-4.2$ mysql -u user1 -p mypa55 -D items
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.24 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.
・・・
```
テーブルを作成し、１行追加してみます。
```
mysql> CREATE TABLE Projects (id int(11) NOT NULL,
    -> name varchar(255) DEFAULT NULL,
    -> code varchar(255) DEFAULT NULL,
    -> PRIMARY KEY (id));
Query OK, 0 rows affected (0.01 sec)

mysql> show tables;
+-----------------+
| Tables_in_items |
+-----------------+
| Projects        |
+-----------------+
1 row in set (0.00 sec)

mysql> insert into Projects (id, name, code) values (1,'openshift','CRC');
Query OK, 1 row affected (0.00 sec)
```

MySQLのプロンプトを閉じ、コンテナを終了し削除します。
```
mysql> exit
Bye
bash-4.2$ exit
exit

$ docker ps
CONTAINER ID   IMAGE                                                    COMMAND                  CREATED          STATUS          PORTS      NAMES
00189317502b   registry.access.redhat.com/rhscl/mysql-57-rhel7:latest   "container-entrypoin…"   17 minutes ago   Up 17 minutes   3306/tcp   mysql-basic

$ docker rm -f 00189317502b
```

## Dockerによるコンテナーライフサイクル管理
Dockerには、コンテナーを作成および管理するためのサブコマンドセットがあります。
開発者はこれらのサブコマンドを使⽤して、コンテナーおよびコンテナーイメージのライフサイクルを管理します。
コンテナーとイメージの状態を変更するサブコマンドで最も使⽤頻度の⾼いものについて、次の図に概要を⽰します。

