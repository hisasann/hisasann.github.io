---
layout: post
title:  "AWS ECS をプロダクトで使う場合の流れメモ"
date:   2018-10-9 12:00:00
category: docker
tags:
    - docker
    - AWS
    - ECR
    - ECS
    - EC2
    - AutoScaling
---

## やりたいこと

以下の過去記事でも取り上げたんですが、 AWS ECS をここ最近プロダクトで使い初めてすごく便利で勉強になったのでそれらをメモしておきます。

[AWS ECSにdockerを使ってexpressをデプロイしてみた - DJレモンサワーのレモン日記](https://hisasann.github.io/2018/04/10/launch-express-on-docker-and-deploy-to-aws-ecs/)

また、まだこうかなー？ああかなー？と模索しているところも多々あるので、今後 AWS ECS について調べたことはこの記事に加筆していく予定です。


## AWS ECR を使うための事前準備

configure で今回試す用の **profile** を作成します。

```bash
aws --profile aws-ecs-profile configure
```

**アクセスキー** と **シークレットアクセスキー** は事前に管理者にもらっておきます。

```
aws_access_key_id: もらったのを入力
aws_secret_access_key: もらったのを入力
region: ap-northeast-1
output format: json
```

疎通確認で s3 の list を取りにいってみます。

```bash
aws s3 ls --profile=aws-ecs-profile
```

S3 のリストが取れたら OK です。

以後、ログインする場合は、 profile に `aws-ecs-profile` を指定します。

```bash
aws --profile aws-ecs-profile ecr get-login --no-include-email --region ap-northeast-1
```


## AWS ECR リポジトリを作成

[Amazon ECR](https://ap-northeast-1.console.aws.amazon.com/ecs/home?region=ap-northeast-1#/repositories)

`リポジトリの作成` をクリックします。

リポジトリ名を入力します。

`完了` をクリックします。

AWS にログインしたり細かい docker コマンドの操作は、 `プッシュコマンドの表示` をクリックすると出てきます。

`Windows システムの場合は、AWS Tools for PowerShell を使用します:` を書かれているが、 PowerShell コマンドを覚えるのも大変なので、そのまま macOS や Linux と同等のコマンドを使用します。

（特に問題は発生しなかったので Power Shell は使わなくてもダイジョウブかも）

いずれの手順で、

`リポジトリの URI`: ここには数字.dkr.ecr.ap-northeast-1.amazonaws.com/aws-ecs-sample-app

のような **URI** が必要になります。


## ECS のクラスターを作成します

[Amazon ECS](https://ap-northeast-1.console.aws.amazon.com/ecs/home?region=ap-northeast-1#/clusters)

### 1.クラスターテンプレートの選択

`ネットワーキングのみ` を選択します。

EC2 自体はのちほど AutoScaling で設定するので、ここでは `空っぽのクラスター` を作成します。

（ちなみに、お試しで作る場合は `EC2 Linux + ネットワーキング` が楽だと思いますが、今回は既存の VPC に EC2 を含めたかったので、`空っぽのクラスター` を選んでいます）

次のステップへ

### 2: クラスターの設定

`クラスター名`: aws-ecs-sample-app

作成をクリックします。


## CloudWatch でロググループを作成します

[CloudWatch Management Console](https://ap-northeast-1.console.aws.amazon.com/cloudwatch/home?region=ap-northeast-1#logs:)

AWS ECS のタスクの作成で、 CloudWatch が必要になるので、このタイミングで作っておきます。

`ロググループ名`: aws-ecs-sample-app


## AWS ECS のタスクを作成します

[Amazon ECS](https://ap-northeast-1.console.aws.amazon.com/ecs/home?region=ap-northeast-1#/taskDefinitions)

`新しいタスク定義の作成` をクリックします。

### 1: 起動タイプの互換性の選択

以下の設定は各自使いたい値を入力ください。

`EC2` を選択します。

次のステップへ

`タスク定義名`: aws-ecs-sample-app

`タスクロール`: aws-ecs-sample-app-task

`ネットワークモード`: bridge （**awsvpc** を使うとデプロイ毎にポート番号がインクリメントされるのではなく IP アドレスが変化する）

`タスク実行ロール`: なし

`タスクメモリ (MiB)`: --

`タスク CPU (単位)`: --

`コンテナの追加` をクリックします

`コンテナ名`: aws-ecs-sample-app

`イメージ`: ここには数字.dkr.ecr.ap-northeast-1.amazonaws.com/aws-ecs-sample-app:0.0.1

`メモリ制限 ハード制限`: 500

`ポートマッピング ホストポート`: 0

`コンテナポート`: 3000

    ひとつの EC2 でコンテナをデプロイする場合、一瞬でも 2 つのコンテナが同居するタイミングが存在します。

    そのときに、ホストポートを 3000 で固定すると、次に上がってくるコンテナも 3000 で上げようとしてバッティングしてしまう。

    ホスト側を 0 にことで、 ECS 側で自動的にポート割当を行ってもらえるので、バッティングが発生しません。

`ログ設定`: awslog

`awslogs-group`: aws-ecs-sample-app

`awslogs-region`: ap-northeast-1

`awslogs-stream-prefix`: aws-ecs-sample-app

追加をクリックします。

作成をクリックします。


## EC2 の起動設定を作成します

[EC2 Management Console](https://ap-northeast-1.console.aws.amazon.com/ec2/autoscaling/home?region=ap-northeast-1#LaunchConfigurations:)

`起動設定の作成` をクリックします。

左の `コミュニティ AMI` をクリックします。

検索ボックスで、 `amzn-ami-2018.03.b-amazon-ecs-optimized - ami-2b4d26c6` を入力しエンター。

出てきた AMI を選択します。

`t2.medium` を選択します。（ここはプロダクトによってタイプを選択します）

    ひとつ注意点としては、 ECS タスクのほうでメモリを 500MB と指定しているので、デプロイした際に二つのタスクが一瞬でも同居することを考えておく必要があります。

    つまり、 500MB + 500MB （実際にはもう少し大きくメモリが確保される）ので、 2GB 以上のメモリが搭載された EC2 AMI を選ぶ必要があります。

    例えば、 500MB な 1 つのタスクで新たにデプロイしようとした場合、もう 500MB 確保されますので、一瞬でも 1GB を超える状況になります。

    このときに t2.micro だとデプロイしようとしてるタスクのメモリが確保できず無限ループに陥る可能性があります。

次へ

`名前`: aws-ecs-sample-app

`IAM ロール`: ecs-task-execution

高度な詳細を開きます。

`ユーザーデータ`:

```
#cloud-config
repo_update: false

write_files:
- path: /etc/ecs/ecs.config
  owner: root:root
  permissions: '0644'
  content: |
    ECS_CLUSTER=aws-ecs-sample-app
```

このユーザーデータにより、 EC2 と ECS が紐付かれます。

`IP アドレスタイプ`: パブリック IP アドレスを各インスタンスに割り当てます。 にチェックを入れます。

（これを忘れると GIP が振られないので、 ssh でログインなどできなくなります）

次へ

ストレージのところは触らなくてよい

次へ

起動設定の作成

`既存のセキュリティグループを選択する` を選択します。

`sg-xxxxxxxx` を選択します。

確認へ

`起動設定の作成` をクリックします。

既存のキーペアを選択します。

`aws-ecs-key` を選択します。

`選択したプライベートキーファイル（aws-ecs-key.pem）へのアクセス権があり、このファイルなしではインスタンスにログインできないことに同意します。` をチェックします。

`起動設定の作成` をクリックします。


## EC2 の AutoScaling グループを作成する

[EC2 Management Console](https://ap-northeast-1.console.aws.amazon.com/ec2/autoscaling/home?region=ap-northeast-1#AutoScalingGroups:id=aws-ecs-sample-app;view=details)

`AutoScaling グループの作成` をクリックします。

`起動設定` にチェックを入れます。

`既存の起動設定を使用する` にチェックを入れます。

`aws-ecs-sample-app` にチェックを入れます。

次のステップへ

### 1. Auto Scaling グループの詳細設定

`グループ名`: aws-ecs-sample-app

`ネットワーク`: vpc-xxxxxxxx

`サブネット`: subnet-xxxxxxxx subnet-xxxxxxxx

次へ

### 2. スケーリングポリシーの設定

`このグループを初期のサイズに維持する` をチェックします。

次へ

### 3. 通知の設定

いったんなし（後々追加が可能）

### 4. タグを設定

`キー`: Name

`値`: aws-ecs-sample-app

`新しいインスタンスにタグ付けする` をチェックします。

    Name キーをつけておくと、インスタンス作成されたあとに自動的に EC2 の名前になってくれるので、あとあと一覧から探しやすい

確認へ

作成

AutoScaling を作成したら、 EC2 の画面で新しく作られるインスタンスを見ていよう。

`初期化しています` が終わるまで見ていよう。

また、 Name タグのところに自動的に名前が入るはず。

ECS のクラスター一覧で `登録済みインスタンス` のところが 1 と表示されているはずです。

#### 豆知識:

あとから、起動設定を変更した場合、 AutoScaling の編集から起動設定をつけ直します。

この場合、インスタンスはそのままなので、一度 **デタッチ** してから

AutoScaling の min-size を一度 0 にしないとデタッチはできない。

デタッチ後、 min-size と希望を 1 に戻します。


## ALB を作成します

[EC2 Management Console](https://ap-northeast-1.console.aws.amazon.com/ec2/v2/home?region=ap-northeast-1#LoadBalancers:sort=loadBalancerName)

ALB はサービスを作成するときに必要なので、ここで事前に作っておきます。

ロードバランサーの作成をクリックします。

ロードバランサーの種類の選択

`Application Load Balancer` を作成します。

### 1: ロードバランサーの設定

`名前`: aws-ecs-sample-app

`スキーム`: 内部（internal）

（今回は internal を選びました）

リスナー

`ロードバランサーのプロトコル`: http

`ロードバランサーのポート`: 80

アベイラビリティーゾーン

`vpc`: dev

リストに出てくる2つにチェックを入れます。

次へ

### 2. セキュリティ設定の構成

次へ

### 3: セキュリティグループの設定

sg-xxxxxxxx sg-xxxxxxxx

次へ

### 4: ルーティングの設定

ターゲットグループ

`名前`: aws-ecs-sample-app-def

`プロトコル`: http

`ポート`: 80

`ターゲットの種類`: instance

ヘルスチェック

`プロトコル`: http

パス: /health-check

（ここは各自 http でアクセスできる health check の URL を書いてください）

次へ

### 5: ターゲットの登録

AutoScaling で作成された EC2 インスタンス（aws-ecs-sample-app）し、 80 番ポートで `登録済みに追加` をクリックします。

### 6. 確認

作成


## AWS ECS のサービスを作成します

[Amazon ECS](https://ap-northeast-1.console.aws.amazon.com/ecs/home?region=ap-northeast-1#/clusters/aws-ecs-sample-app/services)

作成をクリックします。

### 1: サービスの設定

`起動タイプ`: EC2

`タスク定義`: aws-ecs-sample-app

`Revision`: latest

`サービス名`: aws-ecs-sample-app

`サービスタイプ`: REPLICA

`タスクの数`: 1（ここを 1 以上にしないとコンテナが起動されない）

次へ

### 2: ネットワーク構成

Elastic Load Balancing（オプション）

`Application Load Balancer` にチェックを入れます。

`サービス用の IAM ロールの選択`: AWSServiceRoleForECS

`ELB 名`: aws-ecs-sample-app

`ELBへの追加` をクリックします。

`リスナーポート`: 80:HTTP

`リスナープロトコル`: HTTP

`ターゲットグループ名`: aws-ecs-sample-app-default

それ以外はそのままで次へ

### 3: Auto Scaling (オプション)

`サービスの必要数を直接調整しない` にチェックを入れます。

次へ

作成

サービスのデプロイタブの `実行数` のところを定期的に更新して見ていると、 **1** となりデプロイが完了します。

続いて、タスクタブでタスク名のところをクリックし、さらに名前のところを開くと実際の EC2 の外部リンクが表示されます。

ここで、不要な ALB のターゲットグループから 80 番ポートを削除します。

（もう少し良い手順があるかもしれません）


## 手動デプロイしてみます

実際には [継続的デリバリー](https://aws.amazon.com/jp/devops/continuous-delivery/) を意識すると **CodePipeline** を使うのですが、いったんここでは手動でデプロイする方法を書いていきます。

のちのちの記事で CodePipeline 版も掲載します。

### ECR にログインします

[Amazon ECS](https://ap-northeast-1.console.aws.amazon.com/ecs/home?region=ap-northeast-1#/repositories)

↑のほうで作った profile を指定してログインします。

```bash
aws --profile aws-ecs-profile ecr get-login --no-include-email --region ap-northeast-1
```

ビルドして docker イメージを作成しつつ、ここではサンプルとして 3000 番ポートを使ったイメージを実行しています。

```bash
docker build -t aws-ecs-sample-app .
docker run -d -p 3000:3000 aws-ecs-sample-app
```

### ECR に push します

```bash
docker tag aws-ecs-sample-app:latest ここには数字.dkr.ecr.ap-northeast-1.amazonaws.com/aws-ecs-sample-app:latest
docker push ここには数字.dkr.ecr.ap-northeast-1.amazonaws.com/aws-ecs-sample-app:latest
```

### タスク定義から新しいタスクを作成します

[Amazon ECS](https://ap-northeast-1.console.aws.amazon.com/ecs/home?region=ap-northeast-1#/taskDefinitions)

タスク定義をクリックしていき、 **新しいリビジョンの作成** をクリックします。

`タスク定義の新しいリビジョンの作成` の `コンテナ名` のリンクをクリックします。

`イメージ` のところで指定されているイメージ URL の **タグ** の部分を先程 push したものに変更します。

今回は `ここには数字.dkr.ecr.ap-northeast-1.amazonaws.com/aws-ecs-sample-app:latest` なので、 **latest** になります。

更新をクリックします。

### サービスを更新します

[Amazon ECS](https://ap-northeast-1.console.aws.amazon.com/ecs/home?region=ap-northeast-1#/clusters)

ターゲットとなるクラスターをクリックし、サービスタブが表示された状態にします。

サービスにチェックを入れ、 `更新` をクリックします。

`サービスの設定` でタスク定義の `Revision` を `latest` にします。

次へのステップにいき、更新を実行します。

### デプロイされているのを確認します

ターゲットとなるサービスの `デプロイ` タブを見ていると、 2 つのタスクが実行されていき、入れ替わっていくのがわかると思います。

今回は bridge を使っているので、ホストポートはデプロイするたびにインクリメントされていきます。

32791 が 32792 になったりしていると思います。


## あとがき

ふだんフロントエンドを中心としてプログラムを書いているんですが、ここ最近 **docker** や **Kubernetes** など、とても興味深く面白い技術が流行ってきてるので、サーバーサイドも久しぶりにやっております。

とくに node.js でサーバーを書いているんですが、フロントエンドもバックエンドも JavaScript なので、頭のスイッチをそこまで切り替えなくてもよいのが楽ですね。

ではでは！
