---
layout: post
title:  "サーバーサイドnode.jsでCanvasを使い色情報を抽出する"
date:   2018-10-23 12:00:00
category: node.js
tags:
    - Canvas
---

## やりたいこと

サーバーサイド Canvas を使用して、画像から色情報を抽出したい！


## node-canvas をインストールします

**node.js** で canvas を使うとなるとおそらく以下のモジュールがメジャーなんじゃないかなーと思います。

また、実際にプロダクトで使用していますが、とくに font や日本語の問題など特に発生していないので、まったく問題ないかと思います。

[Automattic/node-canvas: Node canvas is a Cairo backed Canvas implementation for NodeJS.](https://github.com/Automattic/node-canvas)

### Windows

ぼくは普段 Windows を使っているので、 **Chocolatey** 経由でインストールしました。

Mac と比べると少し面倒な手順でした。

[Installation: Windows · Automattic/node-canvas Wiki](https://github.com/Automattic/node-canvas/wiki/Installation%3A-Windows)

### Ubuntu and other Debian based systems

docker イメージの `FROM node:8` で使う場合は、 **Debian** になるので、以下を見ながらインストールしました。

[Installation: Ubuntu and other Debian based systems · Automattic/node-canvas Wiki](https://github.com/Automattic/node-canvas/wiki/Installation:-Ubuntu-and-other-Debian-based-systems)

### サンプルコード

**node-canvas** を使ったサンプルコードは以下に置いておきました。

<script src="https://gist.github.com/hisasann/a36714168e60a6528d3bbf2133d19ad9.js"></script>

[hisasann/node-generate-image: generate image with node-canvas in node.js](https://github.com/hisasann/node-generate-image)


## 画像から色情報を抽出します

### color-thief モジュールを使います

**Dominant Color** や **Palette** 情報を取得できるライブラリ、 [Color Thief](https://lokeshdhakar.com/projects/color-thief/) が楽でよかったです。

[ドミナントカラー（dominant color）：色彩検定ガイド～色彩コーディネーターの資格～](http://www.color-sp.com/2006/02/dominant_color.html)

ただし、そのままではブラウザ（DOM）に依存したコードがあり、サーバーサイドで使うとエラーになってしまうので、サーバーサイドで使えるものを選ぶ必要があります。

[lokesh/color-thief: Grabs the dominant color or a representative color palette from an image. Uses javascript and canvas.](https://github.com/lokesh/color-thief/)

[doda/node-color-thief: Grabs the dominant color or a representative color palette from an image. Uses javascript and canvas.](https://github.com/doda/node-color-thief)

などあったのでですが、うまく動かず、最終的に、

[jeanmatthieud/color-thief-jimp: Grabs the dominant color or a representative color palette from an image with node. Uses javascript and jimp.](https://github.com/jeanmatthieud/color-thief-jimp)

こちらを使用しました。

なので、 **dependencies** 的には、

```json
  "dependencies": {
    "canvas": "next",
    "color-thief-jimp": "^2.0.2",
    "jimp": "^0.2.28",
  },
 ```

こんな感じになりました！

### 使い方

すごく簡単に **RGB** を抽出できました！

<script src="https://gist.github.com/hisasann/863f871c2e1a2fdf30eff098459e4e15.js"></script>


## RGB を HSV 色空間に変換して、 Hue 値を取得します

**HSV** とは、色を **色相(Hue)** **彩度(Saturation)** **明度(Value・Brightness)** の3要素で表現する方式です。

[RGBとHSV・HSBの相互変換ツールと変換計算式 - PEKO STEP](https://www.peko-step.com/tool/hsvrgb.html)

[HSV色空間とは : PEKO STEP](https://www.peko-step.com/html/hsv.html)

Hue の値（0°～360°の範囲の値）を見ると、色相上どのあたりの色なのか把握することができます。

なので、割と色認識するときに使い勝手がよかったりします。

RGB to HSV をしてくれるコードは以下のを使わせていただきました。

[JavaScriptでRGBをHSVに変換する方法](https://lab.syncer.jp/Web/JavaScript/Snippet/66/)

<script src="https://gist.github.com/hisasann/449353260e26399ce83c4fc6813ae316.js"></script>


## あとがき

なんとなくサーバーサイドで Canvas を使うのは、敷居が高いのかなーと思っていたのですが、すでに多くの OSS があるおかげで比較的に楽に実装可能でした。

ではでは！


## 読んだ記事

[Node.jsでCanvas(ImageData)を使った簡単な画像処理 - Qiita](https://qiita.com/redshoga/items/d5afef65081b7fdf60cc)

[Node.jsで任意のサイズの画像を生成するスクリプトを書いた - 30歳からのプログラミング](https://numb86-tech.hatenablog.com/entry/2017/10/15/112736)

[numb86/generate-image-file: You can generate image file of any size and byte.](https://github.com/numb86/generate-image-file)

[Bluemix + Node-Canvas で日本語を表示する : まだプログラマーですが何か？](http://dotnsf.blog.jp/archives/1066073209.html)
