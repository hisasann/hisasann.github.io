---
layout: post
title:  "electronとUnityをOSCで繋いでみた"
date:   2016-01-26 15:45:16
categories: node.js es6 react electron unity osc
---

<p align="center">
  <img src="https://i.gyazo.com/a59b33cd06e6af7fd0e97d16e394c423.gif">
</p>

## はじめに

やりたいことは、 **electron** と **Unity** を **OSC** で通信をしてみる。

## electron

まずは、 node.js で使える OSCモジュールを探しました。

わりと、この手のモジュールを書いてる人が結構いて、どれを使えばいいのか迷いましたが、そういえば、トモダチが書いていたのを思い出し

[hrfm/node-oscsocket - JavaScript](https://github.com/hrfm/node-oscsocket)

こちらを使わせてもらうことにしました。

使い方はとても簡単で直感的です。

### sender（OSCを送信する側）

```javascript
const sock = new osc.OSCSocket();

let msg = new osc.OSCMessage();
msg.address = '/osc/from/electron';
msg.addArgument('i', 100);
msg.addArgument('s', 'String value.');
sock.send(msg, 6666, '127.0.0.1');
```

### listener（OSCを受信する側）

```javascript
let socket = new osc.OSCSocket();
socket.bind({
  port: 5555,
  address: '127.0.0.1'
});
socket.on('/osc/from/unity', (message) => {
  console.log('/osc/from/unity');
  console.log(message);

  this.setState({
    message: JSON.stringify(message)
  });
});
```

### webpack 経由だと node.js のコアモジュールにアクセスできない問題

以下のリンクにあるように、webpack などの browserify 経由で bundle するとコアモジュールにアクセスできないので、

index.html に直接 require でグローバル変数に入れています。

[Resolving in node_modules · Issue #9 · webpack/coffee-loader](https://github.com/webpack/coffee-loader/issues/9)

[Cannot resolve module 'fs' · Issue #8 · webpack/jade-loader](https://github.com/webpack/jade-loader/issues/8)

[Webpack issue with node libraries · Issue #29 · kriasoft/react-starter-kit](https://github.com/kriasoft/react-starter-kit/issues/29)

```javascript
<script>
  window.osc = require('oscsocket');
</script>
```
 
### サンプルリポジトリ

[hisasann/electron-osc: OSC communication sample on the electron.](https://github.com/hisasann/electron-osc)

## Unity

Unity 側での OSC 通信をサポートするライブラリは以下を使いました。

ググると、わりとこちらを使っている人が多いので、 最終コミットはかなり前ですが、採用！

[jorgegarcia/UnityOSC: Open Sound Control (OSC) C# classes interface for the Unity3d game engine](https://github.com/jorgegarcia/UnityOSC)

src 以下を Unity のプロジェクトに import or drag and drop して、セットアップします。

OSCController.cs ファイルを作成し、適当な GameObject にアタッチします。

こちらが、送信、受信のサンプルです。

<script src="https://gist.github.com/hisasann/1a071ef57edc32a2cc04.js"></script>

↑の gif ではキューブを適当な数表示しているので、その出し方とかは、リポジトリの参照をお願いします。

[unity-osc/OSCController.cs at master · hisasann/unity-osc](https://github.com/hisasann/unity-osc/blob/master/Assets/OSCController.cs)

UnityOSC/src の中にある OSCHandler.cs の一部を書き換えます。

CreateServer の第一引数は electron 側の文字列と合わせる必要があります。

```java
public void Init()
{
      //Initialize OSC clients (transmitters)
      //Example:		
  CreateClient("UnityOSC", IPAddress.Parse("127.0.0.1"), 5555);

      //Initialize OSC servers (listeners)
      //Example:

  CreateServer("/osc/from/electron", 6666);
}
```

[hisasann/unity-osc - C#](https://github.com/hisasann/unity-osc)

## 雑感

実際に http で開かれるサイトで OSC 通信をするには、さらに WebSocket などで補強しないといけないですが、 electron は file:// で開かれるので、

node.js のコアモジュールに直接アクセスできるので便利ですね。（ちょっと慎重にしないといけないですが）

今回は、 electron と Unity を OSC で繋ぐところまでやりましたが、次回は、 electron でクリックした座標を Unity 側でキャリブレーションしてクリックされた座標に何か出してみたいと思います。

## 参考リンク

[「天才けんけんぱ」:Unityとセンサーの連携（2/4） - 1ビットの孤独](http://koki0702.hatenablog.com/entry/unity_article_02)

( ・∀・)ｲｲ!!

