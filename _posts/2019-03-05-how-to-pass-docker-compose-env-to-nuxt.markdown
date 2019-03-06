---
layout: post
title:  "docker-composeの環境変数.envをNuxt.jsに渡す方法"
date:   2019-03-05 12:00:00
category: Nuxt.js
tags:
    - docker
    - Nuxt.js
---

ここ最近 **TypeScript** もサポートされたこともあり **Nuxt.js** のアーキテクチャを作っていて楽しんでいるんですが、なかなか四苦八苦しております。

そしてひとつなかなかハマりまして、すごく勉強になったのでその内容をメモメモ。


## やりたいこと

**docker-compose** 内の変数を **Nuxt.js** 側 の **process.env.NODE_ENV** に渡したい。


## なぜやりたいのか

まず先に、 Nuxt.js を使うにあたり、実際にサーバーにデプロイするシーンを考えてみました。

それは、 **docker** コンテナをデプロイするイメージで、たとえば **AWS ECS** です。

過去に、 [AWS ECSをプロダクトで使う場合の流れメモ - DJレモンサワーのレモン日記](https://hisasann.github.io/2018/10/09/flow-of-using-aws-ecs-as-a-product/) こんな記事を書いていますが、デプロイ先は AWS ECS でローカルで開発環境ではなく試しに動かしたりするときは **docker-compose** を使っていました。

docker-compose を使うことで、 docker 内に変数などを使わずに、 **環境変数** を外だしできます。

これは AWS ECS でも同様に環境変数を作ることで、 docker イメージは一つだが、環境変数によって切り替えられるというのが大きな利点です。


## docker-compose の環境変数とは

docker-compose を使う場合の環境変数は **.env** ファイルに記述することで切り替えられます。

ぼくは、このファイルを docker-compose コマンドを実行する前に、環境ごとのファイルから生成するようにしています。

node の scripts で `.env.prod` などを実際の `.env` にファイル名を変更してます。

こんな感じです。

### package.json

```json
{
    "prod:docker": "node preprocessor.js prod && docker-compose up --build"
}
```

### preprocessor.js

```javascript
const fs = require('fs')

fs.copyFile(`.env.${process.argv[2]}`, '.env', err => {
  if (err) {
    console.log(err.stack)
    return
  }

  console.log(`Done. .env.${process.argv[2]} to .env`)
})
```

[typescript-nuxtjs/preprocessor.js at master · hisasann/typescript-nuxtjs](https://github.com/hisasann/typescript-nuxtjs/blob/master/preprocessor.js)


## Nuxt.js の env とは

実は Nuxt.js の env 側がかなり曲者で、 `nuxt build` の段階で、

    console.log（process.env） は {} を出力しますが、 console.log（process.env.you_var） は個別に定義しマップされた値を出力します。
    コードが webpack でコンパイルされると、 process.env.your_var と記述されたすべての箇所が、定義した値に置き換えられます。
    すなわち、 env.test = 'testing123' は、 コード中に process.env.test と記述してある箇所が、testing123 へ置き換えられます。

[API: env プロパティ - Nuxt.js](https://ja.nuxtjs.org/api/configuration-env/)

となるようで、つまりコンパイル時に `process.env.xxx` は書き換えられてしまいます。

### ビフォー

```javascript
if (process.env.test == 'testing123')
```

### アフター

```javascript
if ('testing123' == 'testing123')
```

### Nuxt.js が env を書き換えるタイミング

[docker-compose environment vars · Issue #2851 · nuxt/nuxt.js](https://github.com/nuxt/nuxt.js/issues/2851#issuecomment-414057811)

にもあるとおり、

    Env vars embed at build time, so if you need another env vars you need to rebuild.
    Other option would be to use nuxt env module, which allow to use runtime env cars rather than build time

と、コンパイル時に環境変数は埋め込まれる（この場合は置換ですね）が、ランタイム時は無視されると。


## Nuxt.js におけるコンパイル時とランタイム時とは

`nuxt build` がコンパイル時で、 `nuxt start` がランタイム時ですね。

で Dockerfile で以下のように書くと、 `RUN` のところがコンパイル時で `CMD` のところがランタイム時です。

```dockerfile
RUN npm run build

CMD ["npm", "run", "start"]
```

* RUN：ビルド時にコンテナ内で実行される
* CMD：完成したイメージからコンテナを作成するときに実行される

[DockerのRUNとCMDの違い - Qiita](https://qiita.com/YusukeHigaki/items/044164837daa5e845d50)


## docker-compose の environment はいつ Nuxt.js 側に渡るのか

結局のところ、ここの理由により `nuxt start` 時 .env が渡せないことになります。

↑のほうで書いた script の内容は、

```bash
$ docker-compose up --build
```
ですが、これは Dockerfile をもとに docker イメージを作成しつつ、 docker コンテナを起動するコマンドです。

[Environment variables in Compose Docker Documentation](https://docs.docker.com/compose/environment-variables/)

    Set environment variables in containers
    You can set environment variables in a service’s containers with the ‘environment’ key, just like with
    docker run -e VARIABLE=VALUE ...:

で、 compose のサイトを読んでみると、 compose の environment は **docker run** のときに渡されています。

ここが答えですが、

docker-compose の .env に書いた内容は docker-compose.yml を経由して Dockerfile の CMD 実行時（コンパイル時）に
実行されて欲しかったが、実際には RUN の nuxt start するタイミングで渡されている
そして、 nuxt start 時は Nuxt.js 側は process.env.xxx に値を設定してくれない

という感じです。


## Dockerfile に書いた ENV はどうか

これは、コンパイル時に Nuxt.js に渡されるのでもちろん process.env.xxx に渡される。

ただ、これをやると Dockerfile を環境ごとに作成しないといけなくなるので、一番はじめに書いた docker イメージ一つで環境切り替えすることができなくなる。


## Nuxt.js のランタイム時に環境変数を渡す方法はないのか

[samtgarson/nuxt-env: Inject env vars for your Nuxt app at runtime](https://github.com/samtgarson/nuxt-env)

実はこちらのモジュールが実現してくれている。

ただし、 Vue のライフサイクル上でしか使えない。

```javascript
export default {
  computed: {
    testValue () { return this.$env.TEST_VALUE }
  }
}
```

で、こちらを少し細工して Nuxt の Plugin で実行して、使いやすくした記事が以下です。

[【Nuxt.js】nuxt-envを少し使いやすくする - qrunch.net](https://qrunch.net/@dapontinus/entries/RafMTW3qRVZeBsbw)

```javascript
// plugins/injectEnv.js
export default (ctx) => {
  for (let k in ctx.app.$env) {
    if (!process.env[k]) {
      process.env[k] = ctx.app.$env[k]
    }
  }
}
```

これで、 `process.env.FOO` でアクセスできますが、 Plugin が実行されるタイミングは以下のとおりで、 Vue 内でしかアクセスできません。

    Nuxt.js では JavaScript プラグインを定義することができ、それはルートの Vue.js アプリケーションがインスタンス化される前に実行されます。
    この機能は、自前のライブラリや外部のモジュールを使用する際にとりわけ有用です。

[プラグイン - Nuxt.js](https://ja.nuxtjs.org/guide/plugins/)

ここまでして、ランタイム時に環境変数を渡すのもなかなかだなーと思いつつ、とはいえ Dockerfile を環境ごとに作るのもなーという感じです。

まだ、いろいろ試してみておりますので、また何か発見しましたら追記します！


## 読んだ記事

[NODE_ENV either set to development or production, no way to add staging environment · Issue #2741 · nuxt/nuxt.js](https://github.com/nuxt/nuxt.js/issues/2741)

[Docker-Compose の変数定義について - Qiita](https://qiita.com/kimullaa/items/f556431b8103e686f356)

[Nuxtでcross-envを使い環境ごとに環境変数を分ける - Qiita](https://qiita.com/TakahiRoyte/items/c152ad8baa191ed1f8ae)
