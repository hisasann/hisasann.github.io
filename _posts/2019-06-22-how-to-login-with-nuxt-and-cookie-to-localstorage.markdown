---
layout: post
title:  "Nuxt.jsでログインをどうやるか、そしてCookieからlocalStorageへ"
date:   2019-06-22 12:00:00
category: Nuxt.js
tags:
    - Vue.js
    - Nuxt.js
    - login
---

## 🦑 まえがき

Nuxt.js でいわゆる `basic なログインの仕組み` （ユーザーID・パスワードを入れる系）を作ろうとすると、なかなか大変で、これは Cookie が SSR 時と CSR 時に API サーバーまで勝手に送信するしないの話などあり、結構気にする箇所は多くなります。

`Cookie を使ったパターン` では、 Cookie はあくまでも Nuxt.js 側のみで使用し、 BFF との通信は、リクエストヘッダーにログイントークンをのせて送信しています。

また、ログイン処理後はレスポンスヘッダーからログイントークンをもらい、それを Cookie に保存します。

それについての考察は以下にまとめております。

[Nuxt.jsを使ったログイン周りの仕組みについて · hisasann/typescript-nuxtjs-boilerplate Wiki](https://github.com/hisasann/typescript-nuxtjs-boilerplate/wiki/Nuxt.js%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%9F%E3%83%AD%E3%82%B0%E3%82%A4%E3%83%B3%E5%91%A8%E3%82%8A%E3%81%AE%E4%BB%95%E7%B5%84%E3%81%BF%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6)

また、 OAuth 版ではありますが、こちらのリポジトリには大変助けられ、 Nuxt.js で何をどこにどうやって保持するかなどは以下のリポジトリを参考にしております。

[nuxt/example-auth0: A simple example that shows how to use Nuxt.js with Auth0.](https://github.com/nuxt/example-auth0)

今回、ログイン処理で `Cookie 版` と `localStorage 版` の二種類を考察するに至ったのは、

[ITP2.1対策　Safari 12.1 でCookieの有効期限を8日以上に延長する方法 - Qiita](https://qiita.com/yossaito/items/6ffb1b8bb3f9d91107b8)

こちらの内容がもととなっています。

つまり、 Safari では JavaScript からセットされた Cookie は7日間以上保持しないので、8日以上維持したい場合には考慮が必要となります。

また、さらに ITP2.2 では1日しか保持しないなど、極端になる可能性があるので、 Nuxt.js などの SPA 系がかなりキツくなってきました。


## 🧘🏻‍♂️ ログイン処理 - Cookie 版

Cookie 版はすでに以下のリポジトリで使っているので、もしよろしければこちらをご覧ください。

一応、ログイン処理に関わるところを抜粋しておきます。

[typescript-nuxtjs-boilerplate/src at master · hisasann/typescript-nuxtjs-boilerplate](https://github.com/hisasann/typescript-nuxtjs-boilerplate/tree/master/src)

### src/plugins/libraries/axios.ts

`axios` でリクエストする直前、レスポンスを受け取る直前にフックして、 header からログイントークンを取得し、それを Cookie に Save します。

```typescript
export default ({ $axios, app, req, error }): void => {
  /**
   * $axios.onRequest
   */
  $axios.onRequest(
    (config: AxiosRequestConfig): void => {
      const token = getTokenFromCookie(req)

      // トークンがあればログイン済みなのでリクエストヘッダで送信する
      if (token) {
        config.headers[app.$C.ACCESS_TOKEN_NAME] = token
      }
    }
  )

  /**
   * $axios.onResponse
   */
  $axios.onResponse(
    (response): void => {
      const token = response.headers[app.$C.ACCESS_TOKEN_NAME]

      // CSR のときだけトークンをセットする、 SSR のときは nuxtClientInit でセットしている
      if (process.client) {
        if (token) {
          setToken(token)
        } else {
          unsetToken()
        }
      }
    }
  )
}
```

### src/store/index.ts

[Vuex ストア - Nuxt.js](https://ja.nuxtjs.org/guide/vuex-store/#nuxtserverinit-%E3%82%A2%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3)

    nuxtServerInit というアクションがストア内に定義されているときは、Nuxt.js はそれをコンテキストとともに呼び出します（ただしサーバーサイドに限ります）。サーバーサイドからクライアントサイドに直接渡したいデータがあるときに便利です。

`nuxtServerInit` と `nuxtClientInit` の合わせ技は本当に便利で、 nuxtServerInit でセットした Vuex の Store に nuxtClientInit でアクセスすることができます。

ちょっと気持ちが悪いですが、 SSR 時の JavaScript がフロント側に降りてきたあとに含まれているイメージです。

これにより、 Cookie を SSR 側で判断してログインチェックが可能になります。

```typescript
export const actions = {
  /**
   * サーバー初期化時の処理
   */
  async nuxtServerInit(
    { dispatch, commit, state }: ActionContext<any, any>,
    { req, res, error }
  ): Promise<void> {
    await console.log('nuxtServerInit')
    commit('setIsServerInitCalled')

    // ログインチェック
    await dispatch('auth/loginCheck', {} as ILoginCheckPayload)
  },
  /**
   * クライアント初期化時の処理
   */
  nuxtClientInit({ commit, getters }: ActionContext<any, any>, { app }): void {
    console.log('nuxtClientInit')
    commit('setIsClientInitCalled')

    // nuxtServerInit でログインチェックした state を元に token を cookie にセットし直す
    const token = getters['auth/getToken']
    const loggedIn = getters['auth/isAuthenticated']
    console.log('token', getters['auth/getToken'], 'loggedIn:', loggedIn)
    if (token && loggedIn) {
      setToken(token)
    } else {
      unsetToken()
    }
  }
}
```

### src/middleware/authenticated.ts

あとは、 `middleware` でログイン必須用のを作っておいて、コンポーネント側に当てるだけです。

```typescript
import { ILoginCheckPayload } from '@/interfaces/api/User/ILoginCheck'

export default async function({ store, redirect }): Promise<void> {
  console.log('authenticated')

  await store.dispatch('auth/loginCheck', {} as ILoginCheckPayload)

  if (!store.getters['auth/isAuthenticated']) {
    await redirect('/example/auth/sign-in')
  }
}
```

コンポーネント側のコードです。

```typescript
import { Component, Vue } from 'nuxt-property-decorator'

@Component({
  middleware: 'authenticated'
})
```

これが Cookie を使用したログイン処理のパターンになります。


## ログイン処理 - localStorage 版

さてここからが問題です。

Cookie を使ったパターンもなかなかでしたが、こっちに比べればまだまだ簡単でした。

結果だけ先に書いてしまうのですが、 localStorage 版の厄介なところは  **SSR 時にログインしているかの確認** ができないことです。

つまり、リロードした場合や CSR した場合は、あくまでもフロント側でしかログインチェックができません。

ですので、

* リロードしたとき、フロント側でログインチェック
* CSR したとき、必要な画面であればログインチェック

をフロント側のどこでやるかを探すというのがここでのポイントになります。

まずは、割り込めそうなところがどこかをあたっていきます。

### middleware

middleware は `nuxt.config` の router に書くと、 SSR と CSR で毎回呼ばれるようになります。

```typescript
const config: NuxtConfiguration = {
  router: {
    // リロードのタイミングでは SSR 側で実行される
    // ルーティングの度に CSR 側で実行される
    // ログインの必要のない画面でも middleware が実行されるので注意が必要
    middleware: ['check-auth']
  }
}
```

さらに .vue で指定すると、こちらも SSR と CSR で呼ばれます。

```typescript
@Component({
  middleware: 'authenticated'
})
```

これだとリロードしたときに CSR では呼ばれないので、ログインチェックは厳しそうです。

### plugin

[プラグイン - Nuxt.js](https://ja.nuxtjs.org/guide/plugins/)

プラグインはフロント側だけで動くように `mode` を持っていて、これはすごく便利なんですが、ただ実行する場合や `inject` して使うのが主流で自動的にこの内部でログイン処理をするのに向いているのかの判断がつきませんでした。

```typescript
const config: NuxtConfiguration = {
  plugins: [
    { src: '@/plugins/auth/auth-plugin.client.ts', mode: 'client' }
  ]
}
```

ただ、以下のように async/await で待たせると、 page のライフサイクルイベントも待たせてくれるので、何かチェックするのには向いているのかもしれません。

```typescript
import { sleep } from '@/utilities/'

export default async ({
  $axios,
  app,
  route,
  router,
  store,
  req,
  error
}): Promise<void> => {
  console.log('auth-plugin')
  await sleep(5000)
  console.log('auth-plugin2')
}
```

### nuxtClientInit

nuxtClientInit はたとえ async/await を使ったとしても、 `画面のレンダリングを阻害しません` 。

逆に言うと、 mode: 'client' の plugin のように待ってはくれないので、非同期にページをロードしたあとに一度だけ実行されます。

その後の CSR 時には動きません。

これは、もともとやりたかったことの一つ、

* リロードしたとき、フロント側でログインチェック

を満たしてくれそうです。

```typescript
export const actions = {
  /**
   * クライアント初期化時の処理
   */
  // @ts-ignore
  async nuxtClientInit(
    { dispatch, commit, getters }: ActionContext<any, any>,
    { app }
  ): Promise<void> {
    console.log('nuxtClientInit')
    commit('setIsClientInitCalled')

    // ログインチェック
    await dispatch('auth/loginCheck', {
      data: {} as ILoginCheckPayload
    })
  }
}
```

### page.vue

nuxtClientInit で一つは解決しましたが、以下がまだです。

* CSR したとき、必要な画面であればログインチェック

以下の結果ですが、決して良いとは言えないかもですが、落ち着いたのが各画面でログインチェックが必要であれば各画面でやるです。

ここのリダイレクトを plugin 化するのも良いかもしれません。

こうすると、 middleware はログインチェックで使えなくなりますが、必要なところで必要な分できるので、シンプルになりました。

```typescript
  public async created() {
    // ログインチェック
    // dispatch した return 値を受け取っているところがミソ
    const data = await this.$store.dispatch('auth/loginCheck', {
      data: {} as ILoginCheckPayload
    })

    if (!data.loginFlg) {
      const { fullPath } = this.$route
      // リダイレクト
      this.$router.replace(this.$C.PAGE.LOGIN + '?from=' + fullPath)
    }
  }
```

はじめは、以下のように Vuex を @Watch してみたんですが、これもややこしく、やめてしまいました。

```typescript
import { Component, Watch, Vue } from 'nuxt-property-decorator'

export default class Page extends Vue {
  /** computed */
  public get isAuthenticated(): INotice[] {
    return this.$store.getters['auth/isAuthenticated']
  }

  /**
   * ログインチェック
   * { immediate: true } でメソッドを一度即座に実行している
   * これは、CSR で遷移してきた場合は this.$store.getters['auth/isAuthenticated'] の変化がないため Watch が fire されないため
   */
  @Watch('isAuthenticated', { immediate: true })
  onIsAuthenticated(val: string, oldVal: string) {
    console.log('onIsAuthenticated', val, oldVal)
  }
}
```


## 🍜 あとがき

localStorage 版のダメなところは、ログイン必須画面において、画面がチラッと見えてしまうことです。

このあたりは、値が入っていないマイページみたいなのが一瞬見えてからログイン画面に行くので、問題はなさそうですが気持ちが悪い感じは残ってしまいました。

また、 JavaScript から Cookie をセットしてるのがダメなら API を一本用意して Set-Cookie だけさせるのはどうかという案もありますが、この場合そのリンクを踏むと誰でも同じ人としてログインしてしまう問題があるので、こちらも難しいところです。

あとは、ログイン処理だけど、何かしらサーバーサイドの画面でやる、とかですかね。

もう少し策を練ってみますが、難しいですね〜

ではでは！
