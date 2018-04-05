---
layout: post
title:  "AWS-CLIのsync機能でローカルファイルをS3に同期させる"
date:   2017-05-15 12:00:00
category: AWS
tags:
    - AWS
    - S3
    - Sync
    - Log
---

## pip のサイトからインストールスクリプトをダウンロードし実行する

```bash
$ curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"

$ sudo python get-pip.py
```


## pip を使用して AWS CLI をインストールする

```bash
$ sudo pip install awscli
```

すでに入ってる場合は一応アップデートする。

```bash
$ sudo pip install --upgrade awscli
```


## AWS で S3 にアクセスするための IAM を作る

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "s3:ListAllMyBuckets"
            ],
            "Effect": "Allow",
            "Resource": "arn:aws:s3:::*"
        },
        {
            "Sid": "Stmt1492067460000",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::test-s3-sync-bucket",
                "arn:aws:s3:::test-s3-sync-bucket/*"
            ]
        }
    ]
}
```


## AWS で 同期用の S3 バケットを作る

```json
{
    "Id": "Policy1492071213217",
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1492071176541",
            "Action": [
                "s3:PutObject"
            ],
            "Effect": "Allow",
            "Resource": "arn:aws:s3:::test-s3-sync-bucket/*",
            "Principal": "*"
        }
    ]
}
```

ACL のほうは Any authenticated AWS user だけフルアクセスにする。


## 同期元のパソコン内で AWS プロファイルを作成する

AWS コンソールの IAM から、

ユーザー→認証情報→アクセスキーの作成

より、 **Access key ID,Secret access key** を作成しメモしておく。

```bash
$ aws configure --profile test-s3-sync
```

- AWS Access Key ID: メモした AWS Access Key ID

- Secret access key: メモした Secret access key

- Default region name: ap-northeast-1

- Default output format: json


## プロファイルが設定されたか確認する

```bash
$ cat ~/.aws/config

$ cat ~/.aws/credentials
```


## S3 との接続を確認する

```bash
aws s3 ls s3://test-s3-sync-bucket/ --profile=test-s3-sync
```

```bash
aws s3 cp ./logs/log.log s3://test-s3-sync-bucket/ --profile=test-s3-sync
```

```bash
aws s3 sync ./logs s3://test-s3-sync-bucket/ --profile=test-s3-sync
```


## S3 sync の利点

ここ最近 [electron-log-rotate](https://www.npmjs.com/package/electron-log-rotate) の開発をしていて、ログを S3 に上げていつでも見れるようにしようとしているんですが、

ログはどんどん肥大かするのを想定して、ログを削除する機能をつけました。

ただ、そうすると、過去のログが見れなくなってしまうが、 S3 は

    「--delete」オプションを使用すると 同期元にない、同期先のファイルが削除されます。

[S3 sync で s3からファイルを同期させる時の注意点 ｜ Developers.IO](http://dev.classmethod.jp/cloud/aws/s3-sync-exact-timestamps/)

と、--delete オプションを付けないならば、ローカルのファイルを消したとしても S3 にはそれが反映されないので、ログは残っていくという感じです。

## 参考リンク

[S3 sync で s3からファイルを同期させる時の注意点 ｜ Developers.IO](http://dev.classmethod.jp/cloud/aws/s3-sync-exact-timestamps/)

[【AWS】CLIの初期設定について（認証情報とコマンド補完） - TASK NOTES](http://www.task-notes.com/entry/20141026/1414322858)

[S3のアクセスコントロールまとめ - Qiita](http://qiita.com/ryo0301/items/791c0a666feeea0a704c)

[amazon S3 のアクセス制御をIAMポリシーで行うメモ - teketeke_55の日記](http://teketeke55.hatenablog.com/entry/2012/12/11/151743)
