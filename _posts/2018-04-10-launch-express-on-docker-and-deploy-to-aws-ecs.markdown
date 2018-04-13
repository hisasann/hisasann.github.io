---
layout: post
title:  "AWS ECSにdockerを使ってexpressをデプロイしてみた"
date:   2018-04-10 12:00:00
category: docker
tags:
    - docker
    - AWS
    - ECS
    - node.js
    - express
    - Windows
---

## やりたいこと

**docker** をほとんど触ったことがなかったので、軽く hello world くらいためしつつ、 **AWS ECS** が前から気になっていたので、それを使ってみたときのメモです。

やった内容は、

docker コンテナ上で **express** を可動させて、 http でアクセスできるようにし、それを AWS ECS にデプロイしてみました。

ローカル環境は Windows10 です。


## docker の基本操作

基本操作は以下の記事を参考に何度も読ませていただきました。

[WindowsでDocker Hubからイメージをダウンロードしたりアップロードしたりしてみる - Qiita](https://qiita.com/fkooo/items/8a29e308f9eb9ce7648e)

[Windowsで構成情報をDockerfileに定義してイメージを作成してみる - Qiita](https://qiita.com/fkooo/items/53b2ea865e8c2c7fec27)

[Docker for Windowsでイメージからコンテナを生成／操作してみる - Qiita](https://qiita.com/fkooo/items/ad7d023b59df71cc9a60)

**docker hub** にイメージを push することになるので、予めアカウントを作っておきます。

ぼくのアカウントはこちら。

[hisasann - Docker Hub](https://hub.docker.com/u/hisasann/)


## すごくシンプルな Dockerfile を作ってみる

```bash
$ touch Dockerfile
```

```dockerfile
# ベースとなるイメージ
FROM centos

# イメージ作成時に実行されるコマンド
RUN echo Hello

# コンテナ内で実行されるコマンド
CMD ["echo", "World"]
```

**CMD** は Dockerfile 内では **一回しか使えない** ので、ここでコンテナ内で実行したいコマンドを書くようなイメージです。

**RUN** は複数回呼ぶことができます。

```bash
$ docker build -t sample .

Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM centos
 ---> 2d194b392dd1
Step 2/3 : RUN echo Hello
 ---> Running in ca587e1a0363
Hello
Removing intermediate container ca587e1a0363
 ---> 14627a9bb2c9
Step 3/3 : CMD ["echo", "World"]
 ---> Running in e7cb10a3bc65
Removing intermediate container e7cb10a3bc65
 ---> 7135dc3ea843
Successfully built 7135dc3ea843
Successfully tagged sample:latest
SECURITY WARNING: You are building a Docker image from Windows against a non-Windows Docker host. All files and directories added to build context will have '-rwxr-xr-x' permissions. It is recommended to double check and reset permissions for sensitive files and directories.
```

イメージ作成時 Hello を表示され、イメージ作成後の history で World が echo されています。

```bash
$ docker history sample

IIMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
7135dc3ea843        16 minutes ago      /bin/sh -c #(nop)  CMD ["echo" "World"]         0BMAGE               CREATED             CREATED BY                                      SIZE
```

イメージが作成されたか見てみましょう。

```bash
$ docker images

REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
sample                     latest              7135dc3ea843        13 minutes ago      195MB
```

試しに run してみましょう。


```bash
$ docker run sample
World
```


## express が入った Dockerfile を作る

以下のコードを参考にさせていただきました。

[enokd/docker-node-hello: Node.js Hello World app running on CentOS using docker](https://github.com/enokd/docker-node-hello)

### express のサンプルコードを用意する

package.json

```json
{
  "name": "docker-express",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.16.3"
  }
}
```

index.js

```js
var express = require('express');

// Constants
var DEFAULT_PORT = 4567;
var PORT = process.env.PORT || DEFAULT_PORT;

// App
var app = express();
app.get('/', function (req, res) {
  res.send('Hello World\n');
});

app.listen(PORT)
console.log('Running on http://localhost:' + PORT);
```

### Dockerfile を作成する

```dockerfile
FROM centos

# Enable Extra Packages for Enterprise Linux (EPEL) for CentOS
RUN yum install -y epel-release
# Install Node.js and npm
RUN yum install -y nodejs npm

# Install app dependencies
COPY package.json /src/package.json
RUN cd /src; npm install --production

# Bundle app source
COPY . /src

EXPOSE  4567
CMD ["node", "/src/index.js"]
```

**EXPOSE** はコンテナの公開するポート番号を指定する場合に使います。

ローカルでも動くか一応確認します。

```bash
$ npm i
$ node index.js
```

Powershell で、

```bash
Invoke-RestMethod -Uri "http://localhost:4567/" -Method GET
```

をします。

### build をしてローカルから docker コンテナ内の express にアクセスします

```bash
$ docker build -t <自分のdocker hubのユーザ名>/docker-sample .
```

```bash
$ docker images
```

でイメージを確認しておきましょう。

```bash
$ docker run -d -p 4567:4567 hisasann/docker-express
```

docker コンテナ内の 4567 ポートで express が起動しますので、ローカルの 4567 ポートからポートフォワーディングします。

```bash
$ docker ps

CONTAINER ID        IMAGE                         COMMAND                CREATED             STATUS              PORTS                    NAMES
33f53d8b9333        hisasann/docker-express:0.1   "node /src/index.js"   10 seconds ago      Up 9 seconds        0.0.0.0:4567->4567/tcp   tender_poitras
```

```bash
$ docker logs container-id
Running on http://localhost:4567
```

ブラウザからアクセスするとアクセスできました。

ちなみにコンテナを停止するには、

```bash
$ docker stop container-id
```

をします。

### docker コンテナのシェルにアクセスする

```bash
$ docker run -it --rm hisasann/docker-express bash
```

-it コンテナ起動時に起動時にシェルでログインする

--rm コンテナから抜けるタイミングでコンテナを破棄する

[DockerImageをDockerHubに登録の仕方 - Qiita](https://qiita.com/kon_yu/items/7c40f4dfbd1cce006ce7)

参考記事

[Node.js ウェブ・アプリの Docker 化 — Docker-docs-ja 1.11.0 ドキュメント](http://docs.docker.jp/engine/examples/nodejs_web_app.html)

[Docker で Node.js(Express)アプリケーションを動かす - Qiita](https://qiita.com/hkusu/items/8c1546d593d283c4e97e)


## docker hub に push してみる

ログインします。

```bash
$ docker login
```

docker hub に push する用の tag を切ったイメージを作ってみましょう。

元となるイメージから tag: 0.1 を作ります。

```bash
$ docker tag hisasann/docker-express hisasann/docker-express:0.1
```

```bash
$ docker push hisasann/docker-express:0.1
```

これで docker hub のダッシュボードに表示されるようになります。

表示されるまで、少し時間が掛かります。

[Docker Hub](https://hub.docker.com/)


## AWS ECS に デプロイしてみる

AWS ECS に関しては、用語がなかなかややこしいので、以下のリンクを参照ください。

[Amazon ECS(EC2 Container Service)についての簡単なメモ - Qiita](https://qiita.com/mokemokechicken/items/d45144dcd1979c10e336)

[ECS運用のノウハウ - Qiita](https://qiita.com/naomichi-y/items/d933867127f27524686a)

[AWS ECSでDockerコンテナ管理入門（基本的な使い方、Blue/Green Deployment、AutoScalingなどいろいろ試してみた） - Qiita](https://qiita.com/uzresk/items/6acc90e80b0a79b961ce)

だいたいのイメージだと以下の感じかと思います。

1. クラスタ

    EC2インスタンス

1. タスク定義

    クラスタ上で動かすコンテナの情報

    ここで docker hub のリポジトリを登録したりする

1. サービスの作成

    クラスタとタスクを結びつける

### クラスターの作成

クラスターから **クラスター作成** ボタンをクリックする。

**EC2 Linux + ネットワーキング** が選択された状態で次へ。

クラスター名: hisasann-docker-express

EC2 インスタンスタイプ: t2.micro

EC2 で使う **キーペア** を選択

**VPC** を選ぶ

**セキュリティグループ** を選ぶ

**サブネット** を選ぶ

作成をクリックする。

少し時間が掛かるが、これで EC2 インスタンスも作られる。

### タスクの作成

タスク定義から **新しいタスク定義の作成** をクリックする。

タスク定義名: docker-express-task

ネットワークモード: bridge

**コンテナの追加** をクリックする。

コンテナ名: docker-express-container

イメージ: hisasann/docker-express:0.1

ハード制限: 128

ポートマッピング: 4567 4567

ログ設定:
    awslogs

    awslogs-group: HisasannDockerExpressLog

    awslogs-region: ap-northeast-1

    awslogs-stream-prefix: express

追加をクリックする。

作成をクリックする。

### サービスの作成

クラスターからサービスタブで作成をクリックする。

サービス名: docker-express-service

タスクの数: 1（起動するコンテナの数）

次のステップ

ELB タイプ: なし

次のステップ

Service Auto Scaling: しない

次のステップ

サービスの作成をクリックする。


## デプロイされた express にアクセスしてみる

クラスターから **ECS インスタンス** を開きます。

ここに **EC2 インスタンス** 名が書いてあるので、 EC2 の画面から該当のインスタンスを探します。

そこから **IPv4 パブリック IP** でポートに **4567** をつけてブラウザからアクセスします。

「Hello World」 が表示されれば OK です！

CloudWatch のほうを見に行くと、

```
08:05:18 Running on http://localhost:4567
```

と出ています。


## github と docker hub を連携してみる

github: [hisasann/docker-express](https://github.com/hisasann/docker-express)

docker hub の Create から **Create Automated Build** を選択します。

ここで github とリンクをしていない場合は、リンクしておきましょう。

その後、該当のリポジトリを選びます。

すでに docker-express というリポジトリは docker hub 上にあるので、別名で作ってみます。

**hisasann/docker-express-github**

[hisasann/docker-express-github - Docker Hub](https://hub.docker.com/r/hisasann/docker-express-github/)

リポジトリの **Build Settings** で Tag Name を入れて **Trigger** をクリックします。


## 新しいコンテナをデプロイしてみる

新しいコンテナのデプロイには、タスク定義から新しいリビジョンを作成し、サービスから更新作業を行います。

タスク定義からデプロイしたいタスクをチェックし、 **新しいリビジョンを作成** をクリックします。

コンテナのところですでにあるコンテナのリンクをクリックし、コンテナの編集に入ります。

イメージ: hisasann/docker-express-github:0.1

更新をクリックする。

作成をクリックする。

これでタスク定義に **docker-express-task:2** が追加されました。

docker-express-task:2 にチェックを入れて、アクションから **サービスの更新** を選びます。

クラスターのサービスを選び、デプロイタブを見ると、 PRIMARY が実行中: 0 で ACTIVE の実行中が 1 になっています

今回はインスタンスが一つなので、ちょっとシンプルになってしまいましたが、以下のリンクにかかれているようにサービスの更新から自動的にデプロイされる仕組みがあるそうです。

[AWS ECSでDockerコンテナ管理入門（基本的な使い方、Blue/Green Deployment、AutoScalingなどいろいろ試してみた） - Qiita](https://qiita.com/uzresk/items/6acc90e80b0a79b961ce)

この記事ではクラスターからすでに動いている docker-express-task:1 を停止して、 docker-express-task:2 をタスクの実行から実行してみました。

これで、

1. docker-express-task:1 のときは docker hub に push したイメージを使う

1. docker-express-task:2 のときは github と docker hub を連携してイメージの作成は docker hub 側に任せる

ということができました。

なお、本当に新しいイメージがデプロイされたかを確認するために **Hello World → Hello World 2** と表示する内容を変えて試しました。


## あとがき

**immutable infrastructure** や **code as infrastructure** など、新しくデプロイするときは以前のコンテナは破棄してあたらしく作り直すというのは、ひじょうに安心で、ローカルでのテストでも非常に強力です。

EC2 を直接建てて、管理となるとやっぱりたいへんで、とくに再起動したら必要なデーモンが動いてなかったりと設定漏れが発生しやすいですね。

まだ、こっち界隈には全然足を伸ばせていなかったので、今後少しずつ試していこうかと思います。

ではでは！

## 参考記事

[AWS ECSでDockerコンテナ管理入門（基本的な使い方、Blue/Green Deployment、AutoScalingなどいろいろ試してみた） - Qiita](https://qiita.com/uzresk/items/6acc90e80b0a79b961ce)

[DockerImageをDockerHubに登録の仕方 - Qiita](https://qiita.com/kon_yu/items/7c40f4dfbd1cce006ce7)

[Node.js ウェブ・アプリの Docker 化 — Docker-docs-ja 1.11.0 ドキュメント](http://docs.docker.jp/engine/examples/nodejs_web_app.html)
