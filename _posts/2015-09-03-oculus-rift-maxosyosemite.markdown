---
layout: post
title:  "MacOSYosemiteでOculus Riftを動かしてみた（2015/09/03時点）"
date:   2015-09-03 15:45:16
category: Oculus
tags:
    - Oculus
---

**Oculus** を使って、遊んでみようと思ったのですが、もうすでに公式サイトではMacOS用の **runtime** と **sdk** が配布されなくなっていたので、

とりあえず、まずはMacOS YosemiteでOculusを試せる環境を作るまでをメモメモ。

[OpenNI SDK (open-source)-Download | OpenNI](http://openni.ru/openni-sdk/index.html) からOpenNI2をDLしてきてセットアップしてみたが、結果的にうまくいかなかった。

とりあえずやったことを書いておく。

## OculusがMacOSを対応しなくなった？

もともとは以下のURLに、MacOS用のリンクがあったがもうすでになくなっている。

[Developer Center — Downloads | Oculus](https://developer.oculus.com/downloads/)

これはおそらく以下のページに書いてあるとおり、今後OculusがWindowsメインになっていくようなので、その前兆とでも言うことだろう。

[No Oculus Rift for the Mac, but your Mac couldn’t handle it anyway | Macworld](http://www.macworld.com/article/2922722/no-oculus-rift-for-the-mac-but-your-mac-couldnt-handle-it-anyway.html)

なので、本来ならWindowsPCで試すのがいいのだが、MacOSで試せる環境を作ってみたかったので、少し古いバージョンのRuntimeとSDKを拾ってきてみました。

## Rumtimeをインストールする

以下からダウンロードする。

[Developers — Build The Future | Oculus](https://developer.oculus.com/downloads/pc/0.5.0.1-beta/Oculus_Runtime_for_OS_X/)

インストーラーからインストールする。

## SDKをインストールする

以下からダウンロードする。

[ovr_sdk_macos_0.5.0.1.tar.gz](http://static.oculus.com/sdk-downloads/ovr_sdk_macos_0.5.0.1.tar.gz)

適当なディレクトリに配置するだけ。

## ファームウェアのアップデート

ぼくは結果的にアップデートが必要なかったんですが、一応やりかたをメモしておきます。

```
Applications > Oculus > Tools > RiftConfigUtil.app
```

インストールしたディレクトリから、このアプリを起動します。

メニューから、

```
Tools > Advanced > Firmware Update
```

を選んで、

```
Applications > Oculus > Tools > Frimware > DK2 > DK2Firmware_2_12.ovrf
```

を選択します。

## OculusをMacに繋げて、接続確認する

```
Applications > Oculus > Tools > RiftConfigUtil.app
```

を起動します。

ぼくの環境では、いくら抜き差ししても認識しなかったんですが、Macを再起動したら認識しました！

接続がうまくいくと、アプリにOculusが表示され、Oculus自体のLEDが光ります。

## Oculusのディスプレイの設定をする

[Oculus Rift DK2届いた！macbook proで動かしてみた！ - ちょっと未来](http://takahi5.hatenablog.com/entry/2014/08/20/004818)

こちらの記事にあるように、

    なぜかDK2の画面が縦向きだったので90度回転して合わせました。
    画面はミラーリングにしておくのが、操作とかやりやすそうです。

をしたら、いい感じになりました。

## Oculusのデモを起動してみる

SDKに同封されている **OculusWorldDemo.app** を起動します。

ディスプレイの設定がいい感じになっていると、画面いっぱいにアプリが表示されると思います。

こでOculusを装着すれば、YES！

<p align="center">
  <img src="/public/images/blog/IMG_9847.JPG">
</p>

( ・∀・)ｲｲ!!
