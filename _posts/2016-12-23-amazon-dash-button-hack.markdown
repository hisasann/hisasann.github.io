---
layout: post
title:  "Amazon Dash ButtonとUnityをOSCで繋いでみた"
date:   2016-12-22 12:00:00
categories: node.js unity osc iot
---

<p align="center">
  <img src="/public/images/blog/IMG_1967.JPG">
</p>

## はじめに

安価な IoT コントローラとして使えそうだったので、 **Amazon Dash Button** をハックして遊んでみました。

[hisasann/amazon-dash-button-hack: amazon dash button hack](https://github.com/hisasann/amazon-dash-button-hack)

こちらの記事を参考にさせていただきました。

[Amazon Dash Buttonを(正しくない方向で)使ってみた - Qiita](http://qiita.com/takustaqu/items/8539b33780c9675c8657)

使った npm module は [ide/dash-button: A small Node.js server that reacts to Amazon Dash buttons on your WiFi network](https://github.com/ide/dash-button) です。


## Amazon Dash Button をセットアップする

同封されている手順書通り、 iPhone の Amazon アプリから設定をしました。

ここで注意は、 **商品の選択** 画面でその先に進まないことです。

先に進んでセットアップしてしまうと、 **本当に商品が届いてしまう** ので、ここで止めておきます。

この状態で次は npm から scan するプログラムを実行しましょう。


## dash-button を npm i する

```bash
npm init

npm install --save dash-button
```


## package.json に npm script を追加する


```json
{
  "scripts": {
    "scan": "dash-button scan"
  }
}
```


```bash
$ sudo npm run scan
```

sudo を付けないと、パーミッションのエラーが出てしまうので付けます。

すると、以下のように出力されるので、 from の後 Macアドレスをコピーしておきましょう。

    Scanning for DHCP requests and ARP probes on en0...
    Detected a DHCP request or ARP probe from xx:xx:xx:xx:xx:xx


## サンプルコード

```javascript
import DashButton from 'dash-button';

import DASH_BUTTON_MAC_ADDRESS from './dash-button-mac-address';

let button = new DashButton(DASH_BUTTON_MAC_ADDRESS);

console.log('listening');

button.addListener(() => {
  console.log('It works.');

  // something codes
});
```

ぼくは npm script に上記コードを実行するのを追加して

```bash
$ sudo npm run main
```

として実行しました。

node.js のコードを実行した状態で、 Amazon Dash Button を押してみましょう。

It works. と出力されたら出来上がりです！

今回は、前に書いた Unity と Electron を OSC で繋いだサンプルを使いました。

<iframe width="500" height="281" src="https://www.youtube.com/embed/F2Cb_Pvdlxc" frameborder="0" allowfullscreen></iframe>

[hisasann/unity-osc: OSC communication sample on the unity.](https://github.com/hisasann/unity-osc)

``Amazon Dash Button → node.js → OSC → Unity``

こんな感じですね。

node.js でフックできるので、もうこれはなんでもできますね！

[hisasann/amazon-dash-button-hack: amazon dash button hack](https://github.com/hisasann/amazon-dash-button-hack)


## 参考リンク

[Amazon Dash Buttonをハックしてワンプッシュで宅配ピザを注文したりコーヒーを淹れたりタクシーを呼んだりする改造例まとめ - GIGAZINE](http://gigazine.net/news/20161209-amazon-dash-button-hack/)
