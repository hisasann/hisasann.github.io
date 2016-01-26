---
layout: post
title:  "electronのクリック座標をキャリブレーションしてUnity世界で何かする"
date:   2016-01-26 15:46:16
categories: node.js es6 react electron unity osc
---

<p align="center">
  <img src="https://i.gyazo.com/464071701f13a25177d51659049cc08a.gif">
</p>

## はじめに

やりたいことは、 **electron** と **Unity** を **OSC** で通信をして、さらに electron 側の座標をキャリブレーション（変換）して Unity 世界での同じ座標にオブジェクトを出してみる。

↑の gif では、 Unity の手前にまったく同じサイズの透明な electron をレイヤーとして置いています。

つまり、クリックは electron がフックし、その座標をもとに Unity で Cube を出しています。

下準備としては以下のエントリーをご覧ください。

[electronとUnityをOSCで繋いでみた - DJレモンサワーのレモン日記](http://hisasann.github.io/2016/01/27/connect-the-electron-and-unity-in-osc/)

## キャリブレーション

今回はとくにセンサーとかではなく、単純に electron の px 座標を Unity のワールド座標に変換するという感じです。

electron 上で 500px ☓ 500px、Unity 上でも同様に 500px ☓ 500px の状態を用意します。

この場合、electron 側つまり web 側は **左上** が  **(0, 0)** で、 Unity では **(-5, 5)** になります。

**右下** は web側は **(500, 500)** 、 Unity 側は、 **(5, -5)** とまったく値が違います。

この値を合わせる作業が **キャリブレーション** です。

合わせるのは、簡単で、クリックされた座標を縦横の px 値で割り算し、 0 〜 1の値にすればよいのです。

その値を、 Unity 側で、さらに変換してあげます。

## electron

<script src="https://gist.github.com/hisasann/255fcea7c9271d8d62c0.js"></script>

の

```javascript
msg.addArgument('f', event.pageX / 500);
msg.addArgument('f', event.pageY / 500);
```

ここが、まずは web 側の変換作業です。


## Unity

これを今回のパターンに使えるように、少しいじります。

<script src="https://gist.github.com/hisasann/9a1cb441b306cf84a0da.js"></script>

少し長くなってしまいましたが、以下の箇所がポイントです。

```java
Vector3 p = mainCamera.ViewportToWorldPoint (new Vector3 (x, y, 1));

Debug.Log (p.x + " : " + p.y);

GameObject go;
for (int i = 0; i < makeCount; i++) {
  go = this.dropObject;
  // y 座標は unity の世界では逆なので、反転させる
  MakeTouchObject (go, new Vector3 (p.x, (p.y * -1), p.z), 1);
}
```

**mainCamera.ViewportToWorldPoint** で electron 側からもらった x と y の値を入れてワールド座標に変換しています。

これで、ほぼ寸分狂わずに、electron の座標を Unity のワールド座標に復元しています。

一箇所気をつけるのが、 y 座標は、そのままだと逆転してしまっているので、 **-1 を掛けています**。
 
[Unity - Scripting API:](http://docs.unity3d.com/ScriptReference/Camera.ViewportToWorldPoint.html)

## 雑感

これの何がうれしいのかということですが、たとえば、 Unity 側ではグラフィックを最大限に活かしたものを作り、そして、その情報を DOM 側に表示したり、またその逆などいろいろできるようになります。

Unity のいいところと、 electron のいいところを使うという、全然違うプラットフォームを合わせるとまた面白いものができそうですね。

エンジニアの分業もできそうです！

## 参考リンク

[「天才けんけんぱ」:Unityとセンサーの連携（2/4） - 1ビットの孤独](http://koki0702.hatenablog.com/entry/unity_article_02)

[[Unity] [OSC] UnityでOSCを使う（UnityOSC） - The jonki](http://www.jonki.net/entry/20130813/1376398204)

[[Unity] [Arduino] [OSC] UnityとAruduino間でOSCでハスハスする - The jonki](http://www.jonki.net/entry/20130814/1376470996)

( ・∀・)ｲｲ!!

