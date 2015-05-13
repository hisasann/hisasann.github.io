---
layout: post
title:  "Herokuのdockerツールを使ってみた"
date:   2015-05-12 15:45:16
categories: mac docker heroku express node.js
---

[Herokuの'docker:release'の動き |
SOTA](http://deeeet.com/writing/2015/05/07/heroku-docker/) こちらの記事を読ん
で、dockerをあまり触ったことがなかったので、遊んでみた。

入門としてはこちらの記事がすごく参考になりました。

[How to Use Docker on OS X: The Missing Guide Viget](http://viget.com/extend/how-to-use-docker-on-os-x-the-missing-guide)

ちなみに、Herokuにデプロイしたのはシンプルな `Expressアプリケーション` になりま
す。

Express-generatorからサクッと作っただけのデフォルト状態です。

[Express application generator](http://expressjs.com/starter/generator.html)

## VirtualBoxをインストールする

以下リンクからインストール。

[Oracle VM VirtualBox](https://www.virtualbox.org/)

## Homebrewからdocker boot2dockerをインストールする

```bash
$ brew install docker boot2docker
```

[Mac OS X - Docker Documentation](https://docs.docker.com/installation/mac/)

Yosemiteにしてからはじめてbrewを触ったので、エラーが出てしまった。

こちらのリンクをもとに解決しました。

[homebrewユーザーがYosemiteにアプグレしたらbrewコマンドが使えなくなった問題解決法 - 現代版徒然草 (生まれてきたら負け)](http://rrt.hateblo.jp/entry/2014/10/19/031312)

```bash
$ boot2docker init
Wrote Dockerfile (node)
$ boot2docker up
Waiting for VM and Docker daemon to start...
..........................ooooooooooooooo
Started.
Writing /Users/hisamatsu/.boot2docker/certs/boot2docker-vm/ca.pem
Writing /Users/hisamatsu/.boot2docker/certs/boot2docker-vm/cert.pem
Writing /Users/hisamatsu/.boot2docker/certs/boot2docker-vm/key.pem

To connect the Docker client to the Docker daemon, please set:
    export DOCKER_TLS_VERIFY=1
    export DOCKER_HOST=tcp://192.168.59.104:2376
    export DOCKER_CERT_PATH=/Users/hisamatsu/.boot2docker/certs/boot2docker-vm
```

以下のエラーが出たので、

```bash
$ docker info
FATA[0000] Get http:///var/run/docker.sock/v1.18/info: dial unix /var/run/docker.sock: no such file or directory. Are you trying to connect to a TLS-enabled daemon without TLS?
```

一度deleteしてからやりなおした。

```bash
$ boot2docker delete
$ boot2docker init
$ boot2docker up
$ $(boot2docker shellinit)
```

[Upgraded to 1.3.0 now I can't run docker commands from my mac · Issue #576 · boot2docker/boot2docker](https://github.com/boot2docker/boot2docker/issues/576)

## 環境変数の設定

ここのipアドレスを間違えると、以降のコマンドは失敗する。

```bash
export DOCKER_HOST=tcp://192.168.59.104:2376
```

## コンテナにnginxをインストールする

```bash
$ docker run -d -P --name web nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from nginx
3cb35ae859e7: Pull complete
41b730702607: Pull complete
97d05af69c46: Pull complete
55516e2f2530: Pull complete
7ed37354d38d: Pull complete
e7e840eed70b: Pull complete
0b5e8be9b692: Pull complete
439e7909f795: Pull complete
ee8776c93fde: Pull complete
50c46b6286b9: Pull complete
e59ba510498b: Pull complete
42a3cf88f3f0: Already exists
nginx:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
Digest: sha256:a0270163dd1bbb456d77996eb77114b0abd6f82208bd3a30527a8bf523879b8b
Status: Downloaded newer image for nginx:latest
a7925aab5a9ea9ef4224109e5ebcfc454d5fb147b07f10525c715a92a1c1952b
```

## curlしてみる

```bash
$ curl localhost:32769
```

これはうまくいかない。
docker側のipにする必要があるので、ipアドレスを調べる。

```bash
$ boot2docker ip
192.168.59.104
```

毎回ipアドレスを調べるのもアレなので、以下のようにする。

```bash
$ curl $(boot2docker ip):32769
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## herokuコマンドからdockerコンテナを起動する

```bash
$ heroku docker:start
/Users/hisamatsu/.heroku/client/lib/heroku/jsplugin.rb:81: warning: Insecure world writable dir /Users/hisamatsu/_/dev in PATH, mode 040777
building image...
Sending build context to Docker daemon 12.26 MB
Sending build context to Docker daemon
Step 0 : FROM heroku/cedar:14
Pulling repository heroku/cedar
66766a8c5395: Download complete
c40496787f7c: Download complete
f90b0e99841d: Download complete
ac0bf528748f: Download complete
Status: Downloaded newer image for heroku/cedar:14
 ---> 66766a8c5395
Step 1 : RUN useradd -d /app -m app
 ---> Running in df07d94f6428
 ---> bcfd4b40cda0
Removing intermediate container df07d94f6428
Step 2 : USER app
 ---> Running in 96eef9995430
 ---> 1b954e74b9f6
Removing intermediate container 96eef9995430
Step 3 : WORKDIR /app
 ---> Running in da1750b8adec
 ---> 0c04cdec71a9
Removing intermediate container da1750b8adec
Step 4 : ENV HOME /app
 ---> Running in 8ca8e25cfddd
 ---> 9202bd354a05
Removing intermediate container 8ca8e25cfddd
Step 5 : ENV NODE_ENGINE 0.10.38
 ---> Running in defe10b39bd0
 ---> 92134117e6b7
Removing intermediate container defe10b39bd0
Step 6 : ENV PORT 3000
 ---> Running in da19165cf927
 ---> 28010e3971b6
Removing intermediate container da19165cf927
Step 7 : RUN mkdir -p /app/heroku/node
 ---> Running in 90b7a934e7a8
 ---> efbf45fc1176
Removing intermediate container 90b7a934e7a8
Step 8 : RUN mkdir -p /app/src
 ---> Running in 55ef082814cd
 ---> a3aed414541a
Removing intermediate container 55ef082814cd
Step 9 : RUN curl -s https://s3pository.heroku.com/node/v$NODE_ENGINE/node-v$NODE_ENGINE-linux-x64.tar.gz | tar --strip-components=1 -xz -C /app/heroku/node
 ---> Running in 2bf41b93109b
 ---> babb63e214ee
Removing intermediate container 2bf41b93109b
Step 10 : ENV PATH /app/heroku/node/bin:$PATH
 ---> Running in ab80fc9cbda2
 ---> 68220ae24bfb
Removing intermediate container ab80fc9cbda2
Step 11 : RUN mkdir -p /app/.profile.d
 ---> Running in 8496923f1c9e
 ---> 5277bd507981
Removing intermediate container 8496923f1c9e
Step 12 : RUN echo "export PATH=\"/app/heroku/node/bin:/app/bin:/app/src/node_modules/.bin:\$PATH\"" > /app/.profile.d/nodejs.sh
 ---> Running in ae628a61e86a
 ---> a74042c6c973
Removing intermediate container ae628a61e86a
Step 13 : RUN echo "cd /app/src" >> /app/.profile.d/nodejs.sh
 ---> Running in 282b8630118e
 ---> 9a9e8228d30a
Removing intermediate container 282b8630118e
Step 14 : WORKDIR /app/src
 ---> Running in 462c542f3e5a
 ---> dcd626df8494
Removing intermediate container 462c542f3e5a
Step 15 : EXPOSE 3000
 ---> Running in 7225219561a4
 ---> df6f6381165c
Removing intermediate container 7225219561a4
Step 16 : ONBUILD copy . /app/src
 ---> Running in 675c8cc664e5
 ---> a70f472e3134
Removing intermediate container 675c8cc664e5
Step 17 : ONBUILD run npm install
 ---> Running in c7a8fc3550fb
 ---> 3234f1460538
Removing intermediate container c7a8fc3550fb
Successfully built 3234f1460538
building image...
Sending build context to Docker daemon 12.26 MB
Sending build context to Docker daemon
Step 0 : FROM heroku-docker-7f288643a4fbb7b0921d46a204ff9dca
# Executing 2 build triggers
Trigger 0, COPY . /app/src
Step 0 : COPY . /app/src
Trigger 1, RUN npm install
Step 0 : RUN npm install
 ---> Running in 0e6797c4cc22
 ---> 2a0a9ba141b5
Removing intermediate container 4cc0de4ea194
Removing intermediate container 0e6797c4cc22
Successfully built 2a0a9ba141b5

starting container...
web process will be available at http://192.168.59.104:3000/

> heroku-docker-sample@0.0.0 start /app/src
> node ./bin/www
```

ブラウザから `http://192.168.59.104:3000/` で見れるようになった！

## herokuにリリースする

```bash
$ heroku docker:release
```

openでブラウザ起動！

```bash
$ heroku open
```

## 参考リンク

[はじめてのDocker on Mac OS X ｜ Developers.IO](http://dev.classmethod.jp/tool/docker/getting-started-docker-on-osx/)
[Heroku on Docker | Century Link Labs](http://www.centurylinklabs.com/heroku-on-docker/)
[【個人メモ】buildingを使って気軽にDocker containerをつくろう - Qiita](http://qiita.com/futoase/items/21167e9d064b0e336e8f)
