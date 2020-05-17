---
layout: post
title:  "Elastic Beanstalkでnginxの設定を更新する方法🥒"
date:   2020-02-28 01:00:00
category: ElasticBeanstalk
tags:
    - ElasticBeanstalk
    - AWS
    - node.js
---

## 🦑 まえがき

**Elastic Beanstalk** はまったく触ったことがなかったのですが、サクッと EC2 に Web アプリケーションがデプロイできるようなので、勉強がてら触ってみました！

今回の検証の内容は以下のリポジトリに push 済みです。

[hisasann/elastic-beanstalk-nginx-proxy-conf](https://github.com/hisasann/elastic-beanstalk-nginx-proxy-conf)

## eb（Elastic Beanstalk）の環境を作っていく

**eb** には

1. AWS マネジメントコンソールからポチポチ作っていきそこでサンプルアプリケーションもデプロイする
1. eb cli で作っていき、サンプルアプリケーションをデプロイする

の2種類があるようです。

（他にも aws cdk 経由とかありそうですが、一旦ベーシックにはこの2つのパターンがあります）

今回は画面からポチポチではなく、  **eb cli 経由** で作っていきます。

## eb cli をインストールする

以下を参照ください。

[macOS で EB CLI をインストールします。 - AWS Elastic Beanstalk](https://docs.aws.amazon.com/ja_jp/elasticbeanstalk/latest/dg/eb-cli3-install-osx.html)

## eb アプリケーションを作成する

こちらのリンクを参考に作りました。

[Express アプリケーションを Elastic Beanstalk にデプロイする - AWS Elastic Beanstalk](https://docs.aws.amazon.com/ja_jp/elasticbeanstalk/latest/dg/create_deploy_nodejs_express.html)

なるべくシンプルな構成にしたかったので、 Express のジェネレート機能は使っていません。

```bash
$ mkdir eb-nginx-proxy-conf
$ cd eb-nginx-proxy-conf
```

```bash
$ git init
```

```bash
$ eb init --platform node.js --region ap-northeast-1
```

このコマンドは、.elasticbeanstalk という名前のフォルダに、アプリケーションの環境作成用の設定ファイルを作成し、現在のフォルダに基づいた名前で Elastic Beanstalk アプリケーションを作成します。

```bash
$ eb create --sample node-express-env
```

サンプルアプリケーションの環境をまず作ります。

```bash
$ eb open
```
    
ブラウザが起動して、デフォルトのサンプルアプリケーションが開かれると思います。

## eb の nginx の設定ファイルを見に行ってみる

eb の先には **EC2** インスタンスが立っていますが、そのインスタンスを覗きに行くにはどうにかして `ssh` 接続する必要があります。

### AWS Systems Manager のセッションマネージャーを使ってみる

[AWS Systems Manager（運用時の洞察を改善し、実行）AWS](https://aws.amazon.com/jp/systems-manager/)

**AWS Systems Manager** が提供している **セッションマネージャー** を使うとローカルから ssh するのではなく、 AWS マネジメントコンソールから ssh することが可能になります。

### EC2 にセッションマネージャーからアクセスできる IAM ロールをアタッチする

セッションマネージャを用いて EC2 にアクセスするには、そのサーバ上で SSM エージェントが動作している必要があります。

ただ、今の eb から作られる EC2 インスタンスには **はじめから SSM エージェントが入っている** ようなので、インストールは不要でした。

必要なのは、セッションマネージャーから EC2 にアクセスできるようにするための IAM ロールをアタッチします。

EC2 インスタンスのページから IAM ロール（aws-elasticbeanstalk-ec2-role）をクリックし、インラインポリシーの追加から以下の JSON をコピペします。

### policy-for-ec2-server-ssm という名前で作りました

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ssm:DescribeAssociation",
                "ssmmessages:*",
                "ssm:UpdateInstanceInformation",
                "ec2messages:*"
            ],
            "Resource": "*"
        }
    ]
}
```

[チームにAWS セッションマネージャを導入した話 - Qiita](https://qiita.com/paulxll/items/ff547bb08a854e41a104)

あとは、セッションマネージャーの画面からセッションを開始するだけで、ブラウザ上で EC2 に ssh することができました！

## nginx の設定がある場所

以下の場所にあります。

```bash
$ cd /etc/nginx/conf.d
```

ここで、一つ違いがあったのですが、画面ポチポチから eb アプリケーションを作る場合は、 `00_elastic_beanstalk_proxy.conf` というファイルがなかったのですが、

eb コマンド経由だと `00_elastic_beanstalk_proxy.conf` がありました。

このあたりは作り方によっては、環境に違いが出てくるもんなんだろうなーという印象です。

    00_elastic_beanstalk_proxy.conf
    virtual.conf
    
 がありました。
 
 画面ポチポチの場合は、
 
    00_application.conf
    
 というファイルがあり、中身は、

```
location / {
    proxy_pass          http://127.0.0.1:8080;
    proxy_http_version  1.1;

    proxy_set_header    Connection          $connection_upgrade;
    proxy_set_header    Upgrade             $http_upgrade;
    proxy_set_header    Host                $host;
    proxy_set_header    X-Real-IP           $remote_addr;
    proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
}
```

こんな感じでした。

## 自前の Express アプリケーションをデプロイしてみる

```bash
$ yarn init -y
$ yarn add express
```

app.js

```javascript
const express = require('express')
const app = express()
const port = process.env.PORT || 3000

console.log(`port is ${port}`)

app.get('/', (req, res) => res.send('Hello World!!'))

app.listen(port, () => console.log(`Example app listening at http://localhost:${port}`))
```

[elastic-beanstalk-nginx-proxy-conf/app.js at master · hisasann/elastic-beanstalk-nginx-proxy-conf](https://github.com/hisasann/elastic-beanstalk-nginx-proxy-conf/blob/master/app.js)

これだけです。

`process.env.PORT` ですが、この場合のポート番号は、以下のようになるそうです。

    デフォルトの nginx 設定では、nodejs にある 127.0.0.1:8081 という名前のアップストリームサーバーにトラフィックを転送します。

[プロキシサーバーを設定する - AWS Elastic Beanstalk](https://docs.aws.amazon.com/ja_jp/elasticbeanstalk/latest/dg/nodejs-platform-proxy.html)

このポート番号は設定で変更することができるようですが、うまくできなかったので今後の課題です。

## 自前 Express アプリケーションをデプロイする

### .ebextensions/nodecommand.config

```
option_settings:
  aws:elasticbeanstalk:container:nodejs:
    NodeCommand: "npm start"
```

[elastic-beanstalk-nginx-proxy-conf/nodecommand.config at master · hisasann/elastic-beanstalk-nginx-proxy-conf](https://github.com/hisasann/elastic-beanstalk-nginx-proxy-conf/blob/master/.ebextensions/nodecommand.config)

### package.json

```json
"scripts": {
    "start": "node app.js"
}
```

### eb コマンドでデプロイする

```bash
$ git add .
$ git commit -m "First express app"
$ eb deploy
```

しばらくかかるのと、 AWS マネジメントコンソール の eb のページもぐるぐるし始めます。

終わったら、ページを開きましょう。

```bash
eb open
```

`Hello World!!` と表示されていれば成功です。

`var/app/current` アプリケーションが配置されているディレクトリはこちらになります。

## nginx の設定を変更してみる

ここまでは下準備です！（長い！）

[Elastic Beanstalk（Nodeプラットフォーム）でhttpアクセスをhttpsアクセスにリダイレクトする - Qiita](https://qiita.com/shoma2da/items/be8afbf4db1c6e284b1e)

[プロキシサーバーを設定する - AWS Elastic Beanstalk](https://docs.aws.amazon.com/ja_jp/elasticbeanstalk/latest/dg/nodejs-platform-proxy.html)

最終的なファイル構成は以下のようになります。

これは上記の記事を読んでそのまま使っております。

    .ebextensions/
    ├── nginx
    │   └── conf.d
    │       └── 00_elastic_beanstalk_proxy.conf
    ├── nodecommand.config
    └── proxy.config

`00_elastic_beanstalk_proxy.conf` がデフォルトのファイルで、それを `proxy.config` で上書きます。

なので、 `proxy.config` を作る場合は、まずは `00_elastic_beanstalk_proxy.conf` の内容を丸っとコピーしてそこから微調整していきます。

実際の内容は、リポジトリを参照ください。

[elastic-beanstalk-nginx-proxy-conf/00_elastic_beanstalk_proxy.conf at master · hisasann/elastic-beanstalk-nginx-proxy-conf](https://github.com/hisasann/elastic-beanstalk-nginx-proxy-conf/blob/master/.ebextensions/nginx/conf.d/00_elastic_beanstalk_proxy.conf)

[elastic-beanstalk-nginx-proxy-conf/proxy.config at master · hisasann/elastic-beanstalk-nginx-proxy-conf](https://github.com/hisasann/elastic-beanstalk-nginx-proxy-conf/blob/master/.ebextensions/proxy.config)

そして、デプロイです。

```bash
$ eb deploy
$ eb open
```

## nginx のコマンドたち（備忘録）

```bash
$ sudo nginx -s reload
$ sudo nginx -s stop
$ sudo nginx
```

## 番外編ハマりポイント

eb deploy で git commit に **絵文字** があると以下のエラーが出ました。

```
ERROR: InvalidParameterValueError - Invalid unicode xml character in CreateApplicationVersionMessage.Description at index 0
```

これは以下のようなことです。（1時間ほどハマりました！）
    
```
🍦 chore: add package.json
```

## 🍜 あとがき

インフラまわりは、ハマると時間がどんどん溶けていきますが、楽しいですね〜

さて、 Elastic Beanstalk は EC2 を使うので、お金が掛かります。

アプリケーションをターミネートしますかね。

```bash
$ eb terminate
```

ではでは。
