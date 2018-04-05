---
layout: post
title:  "Oculus RiftとWebブラウザで体感するWebVRの世界（環境設定編）"
date:   2015-09-04 15:45:16
category: Oculus
tags:
    - Oculus
    - WebGL
    - WebVR
    - three.js
    - Mac
---

**Oculus** と **WebGL** を連携して遊べることは知っていたのですが、実際やるとなかなかハマりどころもあるので、

まずは動く環境を作るまでをメモメモ。

## vr.js

[benvanik/vr.js](https://github.com/benvanik/vr.js)

検索してみると、このライブラリが多数ヒットしたので、試してみましたが、

NPAPIプラグインがどうもPluginを認識してくれませんでした。

[表示しようとしているプラグインベースのコンテンツが Chrome で動作しない - Chrome ヘルプ](https://support.google.com/chrome/answer/6213033?hl=ja)

    NPAPI プラグインが動作しなくなった理由

    以前は多くのプラグインが NPAPI という古いシステムで開発されていましたが、
    現在 NPAPI プラグインを使用するサイトは少なくなりました。NPAPI プラグインは、ウェブサイトにセキュリティ リスクをもたらすことがよくありました。

    より安全に、高速に、安定して Chrome でのブラウジングができるように、Google では 2015 年 9 月 1 日をもって NPAPI プラグインのサポートを終了しました。

なるほどー

ちょうど試す3日前にサポートが終了してた！

## いろんなライブラリがある、そして動く環境を作る

[three.js - WebでOculus Riftを使用するためのライブラリやツール - Qiita](http://qiita.com/gtk2k/items/5e26336d0b267822d2c4)

ここを見ていると、いろいろな方法でWebVRを実現しているが、なかなかたいへんだなーという印象。

そこで、いろいろググってみると、 **WebVR** という仕組みがあるらしく。

ためしにChromeでthree.jsの [three.js/vr_cubes.html at master · mrdoob/three.js](https://github.com/mrdoob/three.js/blob/master/examples/vr_cubes.html) を開いてみたが、

    Your browser is not VR Ready
    
と出てしまった。

**Canary** でも同様。

そこで、 [three.jsとOculusRiftを使ってのVR化 | KnockKnock](http://www.knockknock.jp/archives/590) こちらに書かれているとおり、[WebVR Builds](https://drive.google.com/folderview?id=0BzudLt22BqGRbW9WTHMtOWMzNjQ#list) からダウンロードして使ってみました。

ただし、そのままでは動作しなかったので、

    chrome://flags/#enable-webvr

ここで、WebVRを有効にする必要があります。

Canaryにも実装されていないのをみると、この仕様はまだまだこれからっといった感じなんでしょうかね。

## three.jsのvr_cubes.htmlを動かしてみる

Chromiumを使って動きそうな環境が整ったので、さっそくthree.jsのexampleを実行してみましたが、画面にCubeがでなくなってしまいました。

ただ、Oculusを動かすと画面も動くので、トラッキングはできているんだろうと思いますが、Cubeが壊れている。

consoleを見てみると、以下のエラーが出てました。

    Uncaught (in promise) TypeError: vrHMD.getEyeTranslation is not a function
        at gotVRDevices (http://localhost:63342/oculus-webgl/javascripts/lib/three.js/examples/js/effects/VREffect.js:50:38)
    296VREffect.js:245 Uncaught TypeError: Cannot read property 'upDegrees' of undefined
    2060VREffect.js:245 Uncaught TypeError: Cannot read property 'upDegrees' of undefined

今回の目玉なjsの **VREffect.js** でエラーが出てる！

## webvr-boilerplateを試す

比較的最終commitが新しいこちらのboilerplateを発見したので、試す。

[borismus/webvr-boilerplate](https://github.com/borismus/webvr-boilerplate)

<p align="center">
  <img src="/public/images/blog/2015-09-04 16.18.52.png">
</p>

結果、↑の画像のように、期待している画面が出つつ、Oculusのトラッキングも出来ていました。（エラーも出てない）

( ・∀・)ｲｲ!!

