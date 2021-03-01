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
![](https://raw.githubusercontent.com/NakamuraYosuke/Day02-DockerAndGit/main/images/dockerlifecycle1.png)

Dockerは、実⾏中および停⽌中のコンテナーに関する情報を⼊⼿するための便利なサブコマンドも提供します。 
これらのサブコマンドを使⽤して、デバッグ、更新、または報告を⾏うためにコンテナーおよびイメージから情報を抽出することができます。
コンテナーとイメージの情報をクエリーするサブコマンドで最も使⽤頻度の⾼いものについて、次の図に概要を⽰します。
![](https://raw.githubusercontent.com/NakamuraYosuke/Day02-DockerAndGit/main/images/dockerlifecycle2.png)

## Dockerネットワーク
![](https://raw.githubusercontent.com/NakamuraYosuke/Day02-DockerAndGit/main/images/dockernetwork.png)
Dockerは同じホスト上にコンテナーを作成すると、各コンテナーに固有の IP アドレスを割り当て、それらすべてを同じソフトウェア定義ネットワークに接続します。
これらのコンテナーは、IP アドレスによって互いに⾃由に通信できます。
異なるホスト上で実⾏されている Podman で作成されたコンテナーは、異なるソフトウェア定義ネットワークに属します。
各 SDN は分離されているため、あるネットワーク内のコンテナーが別のネットワーク内のコンテナーと通信することはできません。ネットワークが分離されているため、ある SDN 内のコンテナーは、異なる SDN 内のコンテナーと同じ IP アドレスを持つことができます。
デフォルトで、すべてのコンテナーネットワークがホストネットワークから隠されていることにも注意してください。
つまり、コンテナーは通常ホストネットワークにアクセスできますが、明⽰的な設定がないと、コンテナーネットワークにアクセスすることはできません。

### ネットワークのポートマッピング
ホストネットワークからコンテナーにアクセスすることが難しい場合があります。 

コンテナーには、利⽤可能なアドレスのプールから IP アドレスが割り当てられています。コンテナーが破棄されると、コンテナーのアドレスは利⽤可能なアドレスのプールに解放されます。別の問題は、コンテナーのソフトウェア定義ネットワークは、コンテナーホストによってのみアクセスできるということです。

これらの問題を解決するために、コンテナーサービスへの外部アクセスを許可するようにポートフォワーディングルールを定義します。-p [<IP address>:][<host port>:]<containerport> オプションを podman run コマンドで使⽤し、外部からアクセス可能なコンテナーを作成します。

![](https://raw.githubusercontent.com/NakamuraYosuke/Day02-DockerAndGit/main/images/portmapping.png)

### 演習：MySQLコンテナのポートをマッピング
（事前作業）mysql-clientをmacにインストールします。
```
$ brew install mysql-client
```
インストール後、環境変数のPATHにmysqlを追加します。
```
$ echo 'export PATH="/usr/local/opt/mysql-client/bin:$PATH"' >> ~/.zshrc
$ source ~/.zshrc
```

次に、MySQLのコンテナを起動します。
標準ポートの`3306`ではなく、`13306`へマッピングしてあげます。

```
$ docker run --name mysql-basic -p 13306:3306 -e MYSQL_USER=user1 -e MYSQL_PASSWORD=mypa55 -e MYSQL_DATABASE=items -e MYSQL_ROOT_PASSWORD=r00tpa55 -d registry.access.redhat.com/rhscl/mysql-57-rhel7:latest
```

起動したMySQLのコンテナへ接続します。
```
$ mysql -uuser1 -h 127.0.0.1 -pmypa55 items
```
下記のようなエラーが出るはずです。
```
mysql: [Warning] Using a password on the command line interface can be insecure.
ERROR 2003 (HY000): Can't connect to MySQL server on '127.0.0.1' (61)
```

ポートを`13306`へマッピングしているため、ポート番号を指定してあげる必要があります。

```
$ mysql -uuser1 -h 127.0.0.1 -pmypa55 -P13306 items
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.24 MySQL Community Server (GPL)
・・・
```

これでマッピングしたポートへアクセス出来ました。

## カスタムイメージの作成
Dockerfileを定義して、自分だけのカスタムイメージを作ります。

Dockerfile は、コンテナーイメージを作成するもう 1 つの⽅法であり、これらの制限に対処します。
Dockerfile は、共有、バージョン管理、再利⽤、拡張が簡単です。

Dockerfile を使⽤すると、親イメージと呼ばれるイメージから⼦イメージと呼ばれる別のイメージを簡単に拡張できます。⼦イメージは親イメージのすべてと、⼦イメージを作成するために⾏われた変更や追加をすべて取り込みます。

以下は、シンプルなApache WebサーバのコンテナをビルドするDockerfileの例になります。
```
# This is a comment line  ①
FROM ubi7/ubi:7.7  ②
LABEL description="This is a custom httpd container image"  ③
MAINTAINER John Doe <jdoe@xyz.com>   ④
RUN yum install -y httpd    ⑤
EXPOSE 80   ⑥
ENV LogLevel "info" ⑦
ADD http://someserver.com/filename.pdf /var/www/html    ⑧
COPY ./src/ /var/www/html/  ⑨
USER apache ➉
ENTRYPOINT ["/usr/sbin/httpd"]  ⑪
CMD ["-D", "FOREGROUND"]    ⑫
```

```
①ハッシュまたはポンド記号 (#) で始まる⾏はコメントです。
②FROM 命令は、新しいコンテナーイメージが ubi7/ubi:7.7 コンテナーベースイメージを拡張することを宣⾔します。Dockerfile は、オペレーティングシステムディストリビューションのイメージだけでなく、他のコンテナーイメージも基本イメージとして使⽤できます。
③LABEL は、イメージに汎⽤メタデータを追加する役割を担います。LABEL はシンプルなキー値ペアです。
④MAINTAINER は、⽣成されたコンテナーイメージのメタデータの Author フィールドを指しています。コマンド docker inspect を使⽤すると、イメージメタデータが表⽰されます。
⑤RUN は、コマンドを現在のイメージの上位の新しいレイヤーで実⾏します。コマンドの実⾏に使⽤されるシェルは /bin/sh です。
⑥EXPOSE は、指定したネットワークポートを、このコンテナーがランタイム時にリッスンすることを⽰します。EXPOSE 命令はメタデータのみを定義し、ホストからポートへのアクセスを可能にすることはできません。podman run コマンドの -p オプションは、ホストからコンテナーポートを公開します。
⑦ENV は、コンテナーで使⽤可能な環境変数を定義する役割を担います。Dockerfile 内にはENV 命令を複数宣⾔できます。env コマンドをコンテナー内で使⽤すると、それぞれの環境変数を表⽰できます。
⑧ADD 命令はローカルまたはリモートソースからファイルまたはフォルダーをコピーし、コンテナーのファイルシステムに追加します。ローカルファイルをコピーするために使⽤する場合は、命令は作業ディレクトリにある必要があります。ADD 命令はローカル .tar ファイルをコピー先のイメージディレクトリに展開します。
⑨COPY は作業ディレクトリからファイルをコピーして、コンテナーのファイルシステムに追加します。この Dockerfile 命令では、URL を使⽤してリモートファイルをコピーすることはできません。
➉USER は、命令 RUN、CMD、ENTRYPOINT でコンテナーイメージを実⾏する際に使⽤するユーザー名または UID を指定します。セキュリティー上の理由から、root とは異なるユーザーを定義することが推奨されます。
⑪ENTRYPOINT は、イメージがコンテナーで実⾏される際に実⾏するデフォルトのコマンドを指定します。省略した場合、デフォルトの ENTRYPOINT は /bin/sh -c になります。
⑫CMD は、ENTRYPOINT 命令のデフォルト引数を』供します。デフォルトの ENTRYPOINT が適⽤される場合 (/bin/sh -c)、CMD はコンテナー起動時に実⾏される実⾏可能なコマンドとパラメーターを形成します。
```
## 演習：Dockerfileによるコンテナ作成
演習用に任意のディレクトリを作成します。
```
$ mkdir -p ~/GitHub/Day02-DockerAndGit/src
```

作成したディレクトリに移動します。
```
$ cd ~/GitHub/Day02-DockerAndGit
```

Dockerfileを作成します。
```
$ vim Dockerfile
```
下記を貼り付けて保存します。
```
FROM registry.access.redhat.com/ubi7/ubi:7.7

MAINTAINER hogehoge

ENV PORT 8080

RUN yum install -y httpd && yum clean all

RUN sed -ri -e "/^Listen 80/c\Listen ${PORT}" /etc/httpd/conf/httpd.conf && \
    chown -R apache:apache /etc/httpd/logs/ && \
    chown -R apache:apache /run/httpd/

USER apache
# Expose the custom port that you provided in the ENV var
EXPOSE ${PORT}
# Copy all files under src/ folder to Apache DocumentRoot (/var/www/html)
COPY ./src/ /var/www/html/
# Start Apache in the foreground
CMD ["httpd", "-D", "FOREGROUND"]
```

次に、srcディレクトリ配下にindex.htmlのサンプルページを配置します。
```
$ vim src/index.html
```
下記を貼り付けて保存します。
```
<html>
 <header><title>Day02-DockerAndGit Hello!</title></header>
 <body>
   Hello World! The dockerfile-review lab works!
 </body>
</html>
```

Dockerfileからカスタムイメージを`custom-apache`という名前でビルドします。

ビルドには、`docker build`コマンドを利用します。

コマンドを実行する際は、Dockerfileと同じ位置から実行してください。

```
$ docker build -t custom-apache .
・・・
Successfully built 7c65a7f35976
Successfully tagged custom-apache:latest
```

`docker images`コマンドを利用して、カスタムイメージが正常にビルドされたことを確認します。
```
$ docker images
REPOSITORY                            TAG       IMAGE ID       CREATED              SIZE
custom-apache                         latest    7c65a7f35976   About a minute ago   236MB
registry.access.redhat.com/ubi7/ubi   7.7       0355cd652bd1   11 months ago        205MB
```

作成したカスタムイメージを起動します。
コンテナ名を`dockerfile`とし、接続ポートを`20080`として起動します。

```
$ docker run -d --name dockerfile -p 20080:8080 custom-apache
```
起動状態を確認します。

```
$ docker ps
CONTAINER ID   IMAGE           COMMAND                 CREATED         STATUS         PORTS                     NAMES
031aa4b503f9   custom-apache   "httpd -D FOREGROUND"   4 seconds ago   Up 4 seconds   0.0.0.0:20080->8080/tcp   dockerfile
```

先ほど作成したindex.htmlが表示されるかを確認します。
```
$ curl 127.0.0.1:20080
<html>
 <header><title>Day02-DockerAndGit Hello!</title></header>
 <body>
   Hello World! The dockerfile-review lab works!
 </body>
</html>
```
このような結果が返って来れば成功です。
