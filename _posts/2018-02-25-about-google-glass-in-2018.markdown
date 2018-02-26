---
layout: post
title:  "GoogleGlassを今更ながら検証してみました in 2018"
date:   2018-02-25 12:00:00
categories: googleglass android ble webrtc webcam
---

## やりたいこと

2017 年の 8 月ごろに Glass Enterprise Edition の販売が始まったので、そもそも GoogleGlass 何ができるのか、また調べたいことが多々あったので、検証したときのメモになります。

[新型 Google Glass 発売、約20万円で法人向けソリューションつき - Engadget 日本版](http://japanese.engadget.com/2017/08/07/google-glass-20/)

1. WebRTC で他のデバイスとビデオチャットの確立
1. Webcam として動作するのかどうか
1. 専用アプリの開発はどうやるのか

ちなみにですが、 GoogleGlass は日本では販売しておらず、基本的には海外の企業向けに販売されているようで、手に入れるのはかなり困難な状態です。

今回は特殊なルートで入手したのですが、そのルートもかなり待たされ、根本的ですが、 GoogleGlass を日本で大量に仕入れて何かするのはすごくハードルが高いと思われます。

なので、あくまでもどうゆうものなのか、と↑に書いたようなことができるのかどうかだけを検証してみました。

そして、 **GoogleGlass のファームのアップデートがうまくいかず、結果的にたいした検証にはなっておりません。**

うまくいくようになりましたら、また追記します。


## 使う GoogleGlass

唯一手に入ったのが **Google Glass XE-C** なので、この Explorer Edition で検証していきます。

[Google Glass XE-C - What's Inside - Electronic Products](https://www.electronicproducts.com/Google_Glass_XE-C-whatsinside-166.aspx)

[グーグルグラス、アップデートXE12をリリース。瞬きでスナップショット、iOS対応など - PRONEWS](https://www.pronews.jp/news/1312241105.html)


## スマホデバイスとのペアリングについて

GoogleGlass は Android or iPhone とペアリングするか、または PC で Wifi の設定をして上げるなどで初めてインターネットに出れます。

基本的には BLE で GoogleGlass を見つけて、 GoogleGlass でもタップし、スマホ側もタップすると確立されます。

### Android

**Android** は BLE で繋げられます。

Android は本当にかんたんでした。何もハマるとこがなかったです。

また、 Android 版の MyGlass は GoogleGlass に写っている映像を Screencast できるので、他の人にいまこうゆう画面が出てるよと伝えられる。

GoogleGlass に対しての Wifi の設定もこのアプリからできます。

なので、 Android を介したインターネット接続と、 GoogleGlass に設定した Wifi の両方で接続ができるのだろうと思います。

### iPhone

**iPhone** は BLE と Personal Hotspot でテザリングとして機能するようです。

つまり Wifi は使えず 4G 回線になってしまう。

しかし、手元の iPhone 6S で散々試してみたが、 iPhone と GoogleGlass との接続までは出来たが

必要なアプリの **MyGlass** において Google アカウントへのサインインができませんでした。

いろいろなアカウントで試してみましたが、 Connecting のところでずっとぐるぐるしっぱなしでした。

以下の記事で、アプリを一度削除するとよいと書かれていたので試してみたがダメでした。

[GoogleGlassとiPhoneのペアリングが完了しない場合の対処方法 - Qiita](https://qiita.com/gm_kou/items/8ee025d0754d879f5f2f)


## PC とのペアリングについて

以下の URL から設定を行う。

[Glass](https://glass.google.com/u/0/myglass)

PC からは **Google のアカウントが必要** で、さらに Wifi 設定をした QR コードを GoogleGlass に読み込ますことで完了します。

参考リンク

[Setting up Wi-Fi - Google Glass Help](https://support.google.com/glass/answer/2725950?hl=en)


## GoogleGlass のファームのアップデートについて

実はここがまだうまくいっていなく、どんハマりしていて、これ以降の内容はファームのバージョンアップがされれば気にしなくてもよいものがある可能性があります。

アップデート処理は以下のようにしておけば自動的にされるようです。

ただし、いまだ **アップデート処理は走っていないです** 。

1. Power Glass on.
1. Ensure Glass is connected to Wifi.
1. Plug Glass in to its charger.
1. Wait until Glass is charged to 50%.
1. On-Head Detection をオフにする

[Updating Glass software - Google Glass Help](https://support.google.com/glass/answer/3226482?hl=en)

もしかしたら IP 制限のようなことがされているのかもしれませんね。


## PC への USB 接続について

GoogleGlass は中身が Android なので、

```bash
$ adb devices
```

で、一覧に出てくるまでをここでは行っています。

なので AndroidStudio のインストールは事前に行っています。

GoogleGlass 側のデバッグを有効にします。

**Settings → Device info → Turn on debug**

### GoogleGlass が adb devices に出てこない場合

そしてこれはぼくのケースだったのですが、

```
List of devices attached
```

とだけでて一覧に出てきませんでした。

そこで、以下の記事にあるように、

[usb - Google Glass is not listed as Android Device by ADB - Stack Overflow](https://stackoverflow.com/questions/20435778/google-glass-is-not-listed-as-android-device-by-adb)

```
C:\Users\hoge\AppData\Local\Android\Sdk\extras\google\usb_driver\android_winusb.inf
```

ファイルの以下のポイントに2行ずつ追加しました。

この場合の hardware id は、デバイスマネージャーの **Android ADB Interface** から引っ張ってきてます。

[Google.NTx86]

```
%SingleAdbInterface%        = USB_Install, USB\VID_18D1&PID_9001
%CompositeAdbInterface%     = USB_Install, USB\VID_18D1&PID_9001&MI_01
```

の下に、

```
%SingleAdbInterface%        = USB_Install, USB\VID_18D1&PID_9001&REV_0216
%CompositeAdbInterface%     = USB_Install, USB\VID_18D1&PID_9001&MI_01
```

[Google.NTamd64]

```
%SingleAdbInterface%        = USB_Install, USB\VID_18D1&PID_9001
%CompositeAdbInterface%     = USB_Install, USB\VID_18D1&PID_9001&MI_01
```

の下に、

```
%SingleAdbInterface%        = USB_Install, USB\VID_18D1&PID_9001&REV_0216
%CompositeAdbInterface%     = USB_Install, USB\VID_18D1&PID_9001&MI_01
```

そして、デバイスマネージャーから一旦削除し、 USB を繋ぎ直したら、おそらくこの inf ファイルを見に行ってくれたのか **Glass1** と外付けデバイスとして認識されました。

この状態で、以下を実行すると、

```bash
$ adb devices
```

このように出てきました。

```
List of devices attached
0160551E16015014        device
```

この後何度か抜き差ししても認識しているので、おそらくこれで大丈夫だと思います。

参考リンク

[Google glassのアプリ開発記録　その１　開発環境作り - ( ﾟÅﾟ)誰んー？乳酸菌魂でおっぱいがいっぱい覚書的なメモ置き場かもしれない](http://nnn.hatenablog.com/entry/2014/05/24/204053)

[Google Glassの簡単なアプリを作る Lab](http://blog.techfirm.co.jp/2014/05/29/google-glass%E3%81%AE%E7%B0%A1%E5%8D%98%E3%81%AA%E3%82%A2%E3%83%97%E3%83%AA%E3%82%92%E4%BD%9C%E3%82%8B/)

[Google Glass: Goole Glass Developers Jump Start](http://glassdev.blogspot.jp/2013/05/goole-glass-developers-jump-start.html)

[Google Glass not showing in Android ADB utility ? hedgehogjim](https://jwhh.com/2014/03/17/google-glass-not-showing-in-android-adb-utility/)


## Webcam として機能するのかどうか

### PC の場合

まずは USB で PC に接続して試してみましたが、ダメでした。

以下の記事を読むと、

**XE19.1 - July 15, 2014** で対応すると書かれていますが、今回使っている GoogleGlass が **XE12** でさらにアップデートできていないので、動かなかったです。

```
Added support for USB webcam On-The-Go (OTG) on Glass. Known issues include:
```

[Google Glass Gains Support For USB Webcams, Because Two Head-Mounted Cameras Are Better Than One](https://www.androidpolice.com/2014/07/24/google-glass-gains-support-for-usb-webcams-because-two-head-mounted-cameras-are-better-than-one/)

Chrome と何かしらの方法で Google Glass を繋いでいる模様（動画）

見た感じ USB と PC を接続しているように見えますが、同じように試しても Chrome 上のカメラを選ぶ行為のところに出てきませんでした。

[Google Glass streaming view with WebRTC on Vimeo](https://vimeo.com/156397171)


### Android の場合

Chrome 上で WebRTC で接続して試してみたのですが、音声だけは GoogleGlass から出ました。

これも XE12 のままなのが影響している可能性があります。


## GoogleGlass アプリを作る方法

以下を見ながらローカルに開発環境をセットアップしていきます。

[GDK Quick Start   Glass Explorer Edition   Google Developers](https://developers.google.com/glass/develop/gdk/quick-start#setting_up_the_development_environment)

専用アプリを作るには GDK という **Glass Development Kit Preview(GDK)** が必要になります。

また、これは AndroidStudio 上だと、 **Android 4.4.2 (API 19)** ここに存在します。

Android Studio で GDK の install できないと思いましたら、Show Package Details で開かないとダメでした（当分触ってなく UI 変わっていたため）

[google gdk - GDK for Andriod Studio 3.0.1? - Stack Overflow](https://stackoverflow.com/questions/47835866/gdk-for-andriod-studio-3-0-1)

で、なんとなく Hello World を作ろうと以下の記事を拝見しました。

[Google Glassアプリケーション開発の基礎知識 - Build Insider](https://www.buildinsider.net/mobile/googleglass/01)

記事も古いので、いろいろと読み替えてやってみましたがかなり大変でした。

また、 [Google Glass](https://github.com/googleglass) で公開されているサンプルも4年前のもので、 AndroidStudio でビルドは通るのですが、その後の Run がうまく動かずいったん諦めました。

そして、いよいよ Hello World を動かそうと作ったプロジェクトを Run してみました。

**app/build.gradle** は以下の感じです。

```
defaultConfig {
    applicationId "com.example.hoge.google_glass_sample"
    minSdkVersion 19
    targetSdkVersion 19
    versionCode 1
    versionName "1.0"
}
```

すると、 GoogleGlass の API は 15 で入れようとしてるのが 19 だから入らないよとエラーがでました。

このタイミングで、 GoogleGlass のファームが古いのに気が付きます。

で、 minSdkVersion を 15 にしてみて再度 Run してみましたが、一応 apk は転送されているっぽいですが、もちろんエラーで落ちてしまいました。

内容は、 API 19 向けに書いたコードは API 15 には新しすぎてクラスが存在してなかったのが原因です。

ここでいったん Hello World は諦めました。


## 強制的にファームのバージョンを変える方法

まだ試していないのですが、 [Disable Google Glass Auto Upgrade - Stack Overflow](https://stackoverflow.com/questions/24902410/disable-google-glass-auto-upgrade) を読むとそういったことができるようです。

以下から、 boot.img を落としてきて、 GoogleGlass のカスタム領域を書き換えて Reset をすることで対象のバージョンになるようです。

[System and Kernel Downloads   Glass Explorer Edition   Google Developers](https://developers.google.com/glass/tools-downloads/system)

参考リンク

[Google Glass XE16 Update Factory Image and Rooted Boot Image Now Available](https://www.xda-developers.com/google-glass-xe16-update-factory-image-and-rooted-bootloader-now-available/)


## 出荷時に戻す方法

1. 15秒間、電源ボタンを押す
1. その後、一度ボタンを離してから、電源ボタンを数秒押す

[NextSphere BLOG: Google Glass が起動しない場合 (再起動／ハードリセット)](http://nextsphere-blog.blogspot.jp/2013/12/google-glass_18.html)

これか、

GoogleGlass 上で **Settings → Device Info → Factory Reset** でも行けました。


## そもそも WebRTC ができるのかどうか

<p align="center">
  <img src="https://www.ericsson.com/research-blog/wp-content/uploads/2014/06/Patrik2.png" />
</p>

via [Field Service Support with Google Glass and WebRTC  Ericsson Research Blog](https://www.ericsson.com/research-blog/field-service-support-google-glass-webrtc/)

一応だが、理論的にできそうではありました。

アーキテクチャとしては、

ブラウザ と WebRTC シグナリングサーバーを繋ぎ、 GoogleGlass （専用アプリ）と WebRTC シグナリングサーバーを繋ぎ、

その後は、ブラウザと GoogleGlass（専用アプリ） が直接接続する、というのが理論的にできる可能性があるアーキテクチャになります。

つまり専用のアプリを作らないことには始まらないようです。

余談ですが、 GoogleGlass で映像を数十分でも撮るとものすごい発熱になるらしいです。

参考リンク

WebRTC on Google Glass 、サービスとしてやっているところはあるみたいです。今もやっているかは不明です。

[BISTRI, Live Communications PaaS for developers ? Bistri for Android works on Google Glass !!! The...](http://blog.bistri.com/post/54056426253/bistri-for-android-works-on-google-glass-the)

nodejs と WebRTC と Google Glass でやってる記事

[Field Service Support with Google Glass and WebRTC  Ericsson Research Blog](https://www.ericsson.com/research-blog/field-service-support-google-glass-webrtc/)

Bistri という PaaS を使えとコメントあり

[android - Google Glass and Web RTC - approaches - Stack Overflow](https://stackoverflow.com/questions/29422976/google-glass-and-web-rtc-approaches)

github にやってる人見つけた、ソースはかなり古い

[nanodust/glassrtc: RTC on Google Glass](https://github.com/nanodust/glassrtc)


## あとがき

試しにどんな感じなのかの検証だったのですが、やはりファームのアップデートが走らないことがすべての作業をうまくいかせてくれませんでした。

とはいえ、 GoogleGlass に触れたのは良い体験でした。

引き続きいろいろ触っていこうとは思っています。

それと、検証している故に長時間 GoogleGlass をつけていたので、片方の目がしょばしょばしました。

ではでは！
