---
layout: post
title:  "node-gypまたはnanを使ってnative addonを作る方法"
date:   2017-02-26 12:00:00
categories: node.js mac gyp addon nan
---

## はじめに

ここずっと、 electron を触っていて、ちょいちょいこの **node-gyp** に関して関心が出てきたので、これを機会に軽くしらべてみました。

[hisasann/node-native-addon-using-node-gyp: node native addon using node gyp](https://github.com/hisasann/node-native-addon-using-node-gyp)

[hisasann/node-native-addon-using-nan: node native addon using nan](https://github.com/hisasann/node-native-addon-using-nan)


## node-gyp とは？

[nodejs/node-gyp: Node.js native addon build tool](https://github.com/nodejs/node-gyp)

GYP（Generate Your Projects）の略で、Chromium team のビルドシステムで使われていて、ワンソースからマルチプラットフォーム用にビルドすることが可能です。

どうやら昔は **node-waf** というものを使っていたが、 node の v0.8 から node-gyp に置き換わったようです。

ちょっと前に、 [kinect2をelectron上で動かすまでの記録 - DJレモンサワーのレモン日記](http://hisasann.github.io/2016/03/02/kinect2-on-electron/) で、 kinect2 を electron で動かすのに苦労したが、このときも node-gyp が影響していました。


## ビルド例

```
 ~/_/js-dev/node-native-addon ⮀  ⮀  ⮀ npm run rebuild                                                                                               ⮂

> node-native-addon@1.0.0 rebuild /Users/yhisamatsu/_/js-dev/node-native-addon
> node-gyp rebuild

  CXX(target) Release/obj.target/hello/src/hello.o
  SOLINK_MODULE(target) Release/hello.node
 ~/_/js-dev/node-native-addon ⮀  ⮀  ⮀ npm run test                                                                                                  ⮂

> node-native-addon@1.0.0 test /Users/yhisamatsu/_/js-dev/node-native-addon
> node src/hello.js

world
```


## binding.gyp

node-gyp を使うときに **binding.gyp** が必要になってきます。

ここには json のような形式で、どのソースをビルドするか、または include したいディレクトリなどを指定して、実際にビルドされるときの設定を記載します。

json のようなと書いたが、このファイルはコメントも書くことができます。

```json
{
  "targets": [
    {
      "target_name": "hello",
      "sources": [
        "src/hello.cc"
      ]
    }, # ここにカンマを入れても問題なし！
  ]
}
```

実際には条件分や変数なども使えるようですが、今回はシンプルに動作確認していきます。


## Commands

[node-gyp/README.md at master · nodejs/node-gyp](https://github.com/nodejs/node-gyp/blob/master/README.md#commands) から拝借してきましたが、コマンドは以下のとおり。

--------

`node-gyp` responds to the following commands:

| **Command**   | **Description**
|:--------------|:---------------------------------------------------------------
| `help`        | Shows the help dialog
| `build`       | Invokes `make`/`msbuild.exe` and builds the native addon
| `clean`       | Removes the `build` directory if it exists
| `configure`   | Generates project build files for the current platform
| `rebuild`     | Runs `clean`, `configure` and `build` all in a row
| `install`     | Installs node header files for the given version
| `list`        | Lists the currently installed node header versions
| `remove`      | Removes the node header files for the given version

ここで主要なコマンドは、 `clean` `configure` `build` or `rebuild` あたりです。

ちなみに `rebuild` は `clean` `configure` `build` を合わせたものになるます。


## hello-world - node-gyp

では実際に hello-world してみましょう！

今回は2種類の hello-world を用意しました。

一つ目は、 node-gyp だけを使ったパターンです。

[node/test/addons/hello-world at master · nodejs/node](https://github.com/nodejs/node/tree/master/test/addons/hello-world)

node-gyp だけを使ったパターンは、 [node/test/addons/hello-world at master · nodejs/node](https://github.com/nodejs/node/tree/master/test/addons/hello-world) を使っています。

こちらは V8 のクラスをそのまま使っていています。

<script src="https://gist.github.com/hisasann/41211539ddfb7bbcb6928d77f7ed40a8.js"></script>

サンプルリポジトリは以下です。

[hisasann/node-native-addon-using-node-gyp: node native addon using node gyp](https://github.com/hisasann/node-native-addon-using-node-gyp)


## hello-world - using nan

[nodejs/nan: Native Abstractions for Node.js](https://github.com/nodejs/nan)

そして二つ目は、 **nan** と **bindings** を使ったパターンです。

[node.jsのnative addonを作るときはNANを使おう。 - from scratch](http://yosuke-furukawa.hatenablog.com/entry/2014/04/07/205935)

こちらに書かれているとおり、 nativeモジュールを作っていくにあたり、 V8 のバージョンアップで後方互換がないケースをうまいことラップしてくれているモジュールが nan です。

なので、ソースコードも node-gyp だけを使ったパターンとは使うクラスが変わってきます。

nan を使ったパターンは、 [node-addon-examples/1_hello_world/nan at master · nodejs/node-addon-examples](https://github.com/nodejs/node-addon-examples/tree/master/1_hello_world/nan) を使っています。

<script src="https://gist.github.com/hisasann/3c08ad66199eea5694e0b73a8735b6fd.js"></script>

bindings モジュールは、以下のようにパスを見に行くのを bindings 経由で native モジュールを取ってくれるので、楽ちんです。

```javascript
// buindings 経由で取りにいく場合
var hello = require('bindings')('hello');
```

```javascript
// パスで取りに行く場合
var hello = require('../build/Release/hello.node');
```

サンプルリポジトリは以下です。

[hisasann/node-native-addon-using-nan: node native addon using nan](https://github.com/hisasann/node-native-addon-using-nan)


## node-gyp or nan

nodejs 側が nan を押しているので、今後の native モジュールの開発では、 nan を使うのが良いでしょう。

[atom/node-nslog: Access to NSLog from inside node!](https://github.com/atom/node-nslog) ちょいちょいエラーで苦しめられた **NSLog** も nan を使っていました。

node-nslog のリポジトリはシンプルですがとても参考になりました。

ちょっと不思議だったのが、 build を実行しなくても **npm install** したタイミングで .gyp ファイルが存在していると自動的に rebuild してくれるようです。

なんだか TitaniumMobile で iOS Android 用のアドオンを作っていたときの辛さが蘇ってきましたが、 node-gyp を使った native モジュールは実際に Chrome にも採用されているので、いろいろと試していきたいところです。


## 参考リンク

[nodejs/node-addon-examples: Node.js C++ addon examples from http://nodejs.org/docs/latest/api/addons.html](https://github.com/nodejs/node-addon-examples)

[node-gypを使ったネイティブモジュールの作成](http://www.slideshare.net/shigeki_ohtsu/nodegakuen5-ohtsu)


## 番外編 - gonode

[jgranstrom/gonode: gonode introduces a way to combine the asynchronous nature of node with the simplicity of concurrency in Go.](https://github.com/jgranstrom/gonode)

node.js から golang のプログラムを呼び出すことが可能なモジュールです。

もちろん、戻り値なども golang 側から渡せます。

### main.js

```javascript
var Go = require('gonode').Go;
var go = new Go({
  path    : 'gofile.go'
});

go.init(function(err) {
  if (err) throw err;

  // TODO: Add code to execute commands
  var text = 'Hello';
  go.execute({
    text: text
  }, function(result, response) {
    console.log(result, response);
    if(result.ok) { // Check if response is ok
      console.log('Go responded: ' + response.text);
    }
  });

  go.close();
});
```

### gofile.go

```go
package main

import (
  gonode "github.com/jgranstrom/gonodepkg"
  json "github.com/jgranstrom/go-simplejson"
)

func main() {
  gonode.Start(process)
}

func process(cmd *json.Json) (response *json.Json) {
  response, m, _ := json.MakeMap()

  text := cmd.Get("text").MustString()

  if(text == "Hello") {
    m["text"] = "Well hello there!"
  } else {
    m["text"] = "What?"
  }

  return
}
```


