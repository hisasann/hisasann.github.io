---
layout: post
title:  "Nuxt.jsでdata-src='~assets/lemon-sour.png'をrequire変換する方法"
date:   2019-03-11 12:00:00
category: Nuxt.js
tags:
    - Vue.js
    - Nuxt.js
    - Webpack
    - vue-loader
---

事の発端は、 **Nuxt.js** で **vue-lazyload** を試しに使ってみようと思ったら割とハマったのでメモ。


## vue-lazyload

[hilongjw/vue-lazyload: A Vue.js plugin for lazyload your Image or Component in your application.](https://github.com/hilongjw/vue-lazyload)

また、 [NuxtにLazyLoadを入れてみた - okadak1990’s blog](https://okadak1990.hatenablog.com/entry/2019/01/12/195327) こちらの記事に大変助けられました。


## Nuxt.js のプラグインを作ります

### plugins/vue-lazyload.ts

```javascript
import Vue from 'vue'
import VueLazyload from 'vue-lazyload'

Vue.use(VueLazyload, {
  preLoad: 1.1,
  attempt: 1,
  observer: true,

  observerOptions: {
    rootMargin: '0px',
    threshold: 0.1
  }
})
```


## Component を作ります

### components/LazyImage.vue

```vue
<template lang="pug">
  img(:alt="alt" v-lazy="src")
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'nuxt-property-decorator'

@Component
export default class LazyImage extends Vue {
  @Prop()
  src: string

  @Prop()
  alt: string
}
</script>
```


## 読み込む Page を作ります

### pages/img-lazy-load.vue

```vue
<template lang="pug">
  section
    h1.title
      | assets and static
    div
      h2 assets lazy-image
      <lazy-image src='~assets/images/lemon-sour.medium.jpg' alt='alt'/>
</template>

<script lang="ts">
import { Component, Vue } from 'nuxt-property-decorator'
import LazyImage from '@/components/LazyImage.vue'

@Component({
  components: {
    LazyImage
  }
})
export default class ImgLazyLoad extends Vue {}
</script>
```


## wabpack で vue-loader の設定が必要だった

上記だけだと、 `<lazy-load />` に **src** を渡す際に、 require で Nuxt.js の URL に変換されませんでした。

URL の変換とは、例えば、以下のような感じです。

```html
<img src="~assets/images/lemon-sour.medium.jpg">
```

↓

```html
<img src="require('~assets/images/lemon-sour.medium.jpg')">
```

↓

```html
<img src="/_nuxt/src/assets/images/lemon-sour.medium.jpg">
```

[アセット - Nuxt.js](https://ja.nuxtjs.org/guide/assets/)


## 苦戦してる方はそこそこいるが・・・

### nuxt.confit.js

以下のように書かれている記事が多いが、これは vue-load のバージョンが v14.x 系でしかうごかない。

```javascript
build: {
  extend(config: Configuration, ctx: Context) {
    const vueLoader = config.module!.rules.find((rule) => rule.loader === 'vue-loader')
    vueLoader.options.transformToRequire['lazy-image'] = ['src']
  }
}
```

[vue-loaderでsourceのsrcsetで指定したファイルも自動でrequireする - Qiita](https://qiita.com/ksk1015/items/f5039554b53d0fefc1e1)

[Nuxt.jsで src=&quot;~assets/hoge.jpg&quot; が変換されなくて困った - Qiita](https://qiita.com/xsota/items/a575642f4451e50ab599)

[Nuxt.jsでassetsをCDNから配信する - 豆腐とコンソメ](https://www.tohuandkonsome.site/entry/2018/09/19/215853)

[data-src doesn't load images from assets folder · Issue #2650 · nuxt/nuxt.js · GitHub](https://github.com/nuxt/nuxt.js/issues/2650#issuecomment-360062030)

[import .mp3 file from assets · Issue #3756 · nuxt/nuxt.js · GitHub](https://github.com/nuxt/nuxt.js/issues/3756#issuecomment-415709954)

[Assets path resolution not working for tags different than <img> · Issue #956 · nuxt/nuxt.js](https://github.com/nuxt/nuxt.js/issues/956#issuecomment-311950885)

いまは、 v15.x 系なので、書き方が変わっていた。

### v14 のオプション

**transformToRequire**

[オプションリファレンス · vue-loader](https://vue-loader-v14.vuejs.org/ja/options.html#transformtorequire)

### v15 のオプション

**transformAssetUrls**

[Options Reference Vue Loader](https://vue-loader.vuejs.org/options.html#transformasseturls)

### v15 を考慮した nuxt.config.ts

```javascript
build: {
  extend(config: Configuration, ctx: Context) {
    const vueLoader = config.module!.rules.find((rule) => rule.loader === 'vue-loader')
    vueLoader.options.transformAssetUrls = {
      'lazy-image': 'src'
    }
  }
}
```


## あとがき

Nuxt.js というか webpack というか vue-loader の話でしたが、いやー、これはハマりました。

console.log で options を出力して初めて中身の違和感に気づきました。

今後、この手のものは後方互換がない前提で実装していったほうがよさそうですね。

ではでは。

[hisasann/typescript-nuxtjs-boilerplate: Nuxt.js with typescript and run with docker and docker-compose](https://github.com/hisasann/typescript-nuxtjs-boilerplate)

