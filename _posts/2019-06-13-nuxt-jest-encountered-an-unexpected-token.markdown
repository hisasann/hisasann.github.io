---
layout: post
title:  "Nuxt.jsで「Jest encountered an unexpected token」が出たときの対処法"
date:   2019-06-13 12:00:00
category: Nuxt.js
tags:
    - Vue.js
    - Nuxt.js
    - Webpack
    - vue-loader
---

## まえがき

以下のリポジトリで `TypeScript + Nuxt.js + Jest` な構成で boilerplate を作っているんですが、

[hisasann/typescript-nuxtjs-boilerplate: Nuxt.js with TypeScript and Run with docker and docker-compose](https://github.com/hisasann/typescript-nuxtjs-boilerplate)

**.vue** ファイルを Jest でテストすると以下のようなエラーが出てしまいました。

```bash
Jest encountered an unexpected token

    This usually means that you are trying to import a file which Jest cannot parse, e.g. it's not plain JavaScript.

    By default, if Jest sees a Babel config, it will use that to transform your files, ignoring "node_modules".

    Here's what you can do:
     • To have some of your "node_modules" files transformed, you can specify a custom "transformIgnorePatterns" in your config.
     • If you need a custom transformation specify a "transform" option in your config.
     • If you simply want to mock your non-JS modules (e.g. binary assets) you can stub them out with the "moduleNameMapper" config option.

    You'll find more details and examples of these config options in the docs:
    https://jestjs.io/docs/en/configuration.html

    Details:

    /Users/hisamatsu/_/js-code/typescript-nuxtjs-boilerplate/node_modules/@babel/runtime/helpers/esm/typeof.js:3
    export default function _typeof(obj) {
    ^^^^^^

    SyntaxError: Unexpected token export

       6 | import { Component, Prop, Vue } from 'nuxt-property-decorator'
       7 |
    >  8 | @Component
         |                               ^
       9 | export default class Logo extends Vue {
      10 |   @Prop({ default: 'Hello world !' })
      11 |   msg: string

      at ScriptTransformer._transformAndBuildScript (node_modules/@jest/transform/build/ScriptTransformer.js:471:17)
      at ScriptTransformer.transform (node_modules/@jest/transform/build/ScriptTransformer.js:513:25)
      at src/components/Logo.vue:8:39
      at Object.<anonymous> (src/components/Logo.vue:84:3)

Test Suites: 1 failed, 1 total
Tests:       0 total
Snapshots:   0 total
Time:        3.108s
Ran all test suites.
error Command failed with exit code 1.
```

この手のエラーは結構書かれてはいるんですが、どうしても解決できず、さらに深淵へ。

[Nuxt+Vuex, Jest tested powered by TypeScript – Al-un – Medium](https://medium.com/@Al_un/nuxt-vuex-jest-tested-powered-by-typescript-70441600ef39)

[Default unit tests are failing · Issue #1879 · vuejs/vue-cli](https://github.com/vuejs/vue-cli/issues/1879)

[How to solve Jest error with Vue – Samuel Pouyt – Medium](https://medium.com/@samuelpouyt/how-to-solve-vue-error-with-jest-e8e9ff7d0cec)

[vue + jest + typescript: Unexpected token import · Issue #134 · vuejs/vue-jest](https://github.com/vuejs/vue-jest/issues/134)


## 結果的に

**.babelrc** に記載してしまっていた、 "@nuxt/babel-preset-app" が原因でした。

[nuxt.js/packages/babel-preset-app at dev · nuxt/nuxt.js](https://github.com/nuxt/nuxt.js/tree/dev/packages/babel-preset-app)

これは、 @babel/preset-env のラッパーのようなのですが、

@babel/plugin-syntax-dynamic-import, @babel/plugin-proposal-decorators, @babel/plugin-proposal-class-properties, @babel/plugin-transform-runtime

このあたりのポリーフィルを含んでいるようです。

確かに↑のエラーを見ると、ポリーフィル的な箇所でコケているので、う〜ん、という感じです。

```json
{
  "env": {
    "test": {
      "presets": [
        [
          "@babel/preset-env",
          {
            "targets": {
              "node": "current"
            }
          }
        ],
        "@nuxt/babel-preset-app"
      ]
    }
  }
}
```


## あとがき

なので、 **.babelrc** を以下のようにすれば問題なく **.vue** ファイルも Jest でテストすることができました。

```json
{
  "env": {
    "test": {
      "presets": [
        [
          "@babel/preset-env",
          {
            "targets": {
              "node": "current"
            }
          }
        ]
      ]
    }
  }
}
```

ちなみに、これを発見したのは、新たに Nuxt プロジェクトを作ってファイルを見比べていってようやく発見できました。

なかなか、こうゆう設定まわりはハマりますねー

ではでは！
