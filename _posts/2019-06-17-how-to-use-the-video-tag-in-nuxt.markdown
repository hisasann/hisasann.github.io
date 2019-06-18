---
layout: post
title:  "Nuxt.jsでvideoタグを使うときのfile-loader奮闘記"
date:   2019-06-17 12:00:00
category: Nuxt.js
tags:
    - Vue.js
    - Nuxt.js
    - Webpack
    - file-loader
---

## 🦑 まえがき

かなり basic なタイトルになってしまいましたが、いま現在の Nuxt.js が内包している `file-loader` の場合、 **mp4** を読み込ませると動画がうまく再生されなかったので、それのメモです。


## file-loader で奮闘

[webpack-contrib/file-loader: File Loader](https://github.com/webpack-contrib/file-loader)

まず audio タグですが、 **mp3** を

```html
<audio src="@/assets/water.mp3" controls></audio> 
```

このように読み込ませるには、 nuxt.config に設定の追加が必要です。

これは、 

[Nuxt.jsでdata-src='~assets/lemon-sour.png'をrequire変換する方法 - DJ lemon-Sour's diary (prod.hisasann)](https://hisasann.github.io/2019/03/11/how-to-convert-data-src-to-require-in-nuxt/)

この記事で言及しましたが、なかなか厄介なところです。

それで、以下の設定だとうまく video タグが再生されず、 audio タグは再生されたのですが、ちょっとハマりました。

そして、Nuxt.js のサイトに audio を require させるドキュメントがあったので、これで試してみたんですが、

[assets ディレクトリからオーディオファイルを読み込む - Nuxt.js](https://ja.nuxtjs.org/faq/webpack-audio-files/)

ここでも余計なひと手間を加えてしまい、再生できなかったです。そして最終的にどうなったかは以下をご覧ください。


## 古い nuxt.config.ts の設定

```typescript
build: {
  /**
   * You can extend webpack config here
   */
  extend(
    config: WebpackConfiguration,
    ctx: {
      isDev: boolean
      isClient: boolean
      isServer: boolean
      loaders: any
    }
  ): void {
    const vueLoader: any = config.module!.rules.find(
      (rule): boolean => rule.loader === 'vue-loader'
    )
    // 特殊なアトリビュートでも webpack の require がかまされるようにする
    vueLoader.options.transformAssetUrls = {
      audio: 'src',
      video: ['src', 'poster'],
      source: 'src',
      img: ['src', 'data-src'],
      image: 'xlink:href',
      Test: 'source',
      'lazy-image': 'src'
    }
  }
}
```


### 新しい nuxt.config.ts の設定

```typescript
test: /\.(ogg|mp3|wav|mpe?g)$/i
```

これを、

```typescript
test: /\.(ogg|mp3|mp4|wav|mpe?g)$/i
```

こうすると、 [206 Partial Content - HTTP | MDN](https://developer.mozilla.org/ja/docs/Web/HTTP/Status/206) が返ってきてしまいました。

察するに file-loader が mp4 コンテンツをよしなにしてくれた結果なのかなーと予想しています。


```typescript
build: {
  loaders: {
    // https://ja.nuxtjs.org/faq/webpack-audio-files/
    vue: {
      // 特殊なアトリビュートでも webpack の require がかまされるようにする
      transformAssetUrls: {
        audio: 'src',
        video: ['src', 'poster'],
        source: 'src',
        img: ['src', 'data-src'],
        'lazy-image': 'src'
      }
    }
  },

  /**
   * You can extend webpack config here
   */
  extend(
    config: WebpackConfiguration,
    ctx: {
      isDev: boolean
      isClient: boolean
      isServer: boolean
      loaders: any
    }
  ): void {
    // https://ja.nuxtjs.org/faq/webpack-audio-files/
    config.module!.rules.push({
      test: /\.(ogg|mp3|wav|mpe?g)$/i,
      loader: 'file-loader',
      options: {
        name: '[path][name].[ext]'
      }
    })
  }
}
```


## 参考記事

[Load video from assets folder · Issue #2008 · nuxt/nuxt.js](https://github.com/nuxt/nuxt.js/issues/2008)


## 🍜 あとがき

file-loader が何をしているのか、ちょいと探りにいってきます！

ではでは！
