---
layout: post
title:  "kinect2をelectron上で動かすまでの記録"
date:   2016-03-03 12:00:00
categories: document
---

## はじめに

やりたいことは、 **kinect2** を **electron** 上で動かすこと。

kinect2 を node.js からアクセスするための npmモジュール は、いろいろ検索した結果、

[wouterverweirder/kinect2: Nodejs library to access the kinect 2 data from the official MS SDK](https://github.com/wouterverweirder/kinect2)

こちらを使わせてもらうことにしました。

これ以外は、最終コミットが1年以上前だったりしたので。

あとは、 electron の example がかなりあったので、試しやすかったためです。

## kinect2 の SDK を入れる

[Download Kinect for Windows SDK 2.0 from Official Microsoft Download Center](https://www.microsoft.com/en-us/download/details.aspx?id=44561)

## node-gyp のために、 python and Visual Studio を入れる

[Windowsでnode-gypのビルドを通す - なにか作る](http://create-something.hatenadiary.jp/entry/2014/07/13/021655)

[Windowsでnode-gypのビルドを通す - その2 - なにか作る](http://create-something.hatenadiary.jp/entry/2014/07/21/185404)

こちらの記事と同じような現象になったので、 **node-gyp** のために、 **python** を入れました。

バージョン : 2.7 64bit版

次は **Visual Studio** です。

ぼくはあんまり何も考えずには、 **Visual Studio Community 2015** を入れたんですが、以下のエラーが出て動きませんでした。

[無料開発ツール - Visual Studio Community 2015](https://www.visualstudio.com/products/visual-studio-community-vs)

    MSB8003: Could not find WindowsSDKDir variable from the registry.  TargetFrameworkVersion or PlatformToolset may be set to an invalid version number.
    
    TRACKER : error TRK0005

なので、 **2013** のほうを 2015 が入ったままでインストールしました。
  
[Download Microsoft Visual Studio Express 2013 for Windows Desktop Update 4 from Official Microsoft Download Center](https://www.microsoft.com/ja-jp/download/details.aspx?id=44914)

    TRACKER : error TRK0005:

これで、1つめのエラーは出なくなったのですが、もうひとつ残っています。

[node.js - node gyp error TRACKER : error TRK0005: Failed to locate: "CL.exe". The system cannot find the file specified - Stack Overflow](http://stackoverflow.com/questions/33183161/node-gyp-error-tracker-error-trk0005-failed-to-locate-cl-exe-the-system-c)

こちらを参考に、以下のように **npm config** を設定し、エラーが解消されました。

つまり、どちらのバージョンを見に行くのか、設定してあげる必要があるようです。

2015 で動かなかったので、以下のように 2013 を指定します。

Windows の環境設定に設定してもダイジョウブのようです。

    $ npm config set msvs_version 2013 --global

or

    GYP_MSVS_VERSION=2013
  
確認したい場合は、

    $ npm config list

## kinect2 の npmモジュールを npm i する

    $ npm i kinect2 --save

## npm i は終わったが、 electron を起動すると console にエラーが出る

    Uncaught Error: A dynamic link library (DLL) initialization routine failed.
  
これは、以下の URL のように、ネイティブコードは **electron-rebuild** をしてあげないとダメのようです。

[electron/using-native-node-modules.md at master · atom/electron](https://github.com/atom/electron/blob/master/docs/tutorial/using-native-node-modules.md)

今回の kinect2 npmモジュールは、この処理を簡単にできるようにスクリプトを用意してくれています。

    $ node tools\electronbuild.js --target=0.30.2 --arch=x64

target は使っている electron のバージョン、 arch に環境を入れます。

上記のように、 electronbuild.js を使ったり、
  
    $ npm install --save-dev electron-rebuild
    
    # Every time you run "npm install", run this
    ./node_modules/.bin/electron-rebuild
    
    # On Windows if you have trouble, try:
    .\node_modules\.bin\electron-rebuild.cmd

このように、自分で electron-rebuild を呼び出してみると、 dynamic link に関するエラーは出なくなるのですが、代わりに以下のエラーが出ました。

    Uncaught Error: %1 is not a valid Win32 application

[Uncaught Error: %1 is not a valid Win32 application. · Issue #5 · Level/electron-demo](https://github.com/Level/electron-demo/issues/5)

どうも **x64** だと、ダメなようで、**ia32** に変えると console のエラーは消え、 kinect2 はちゃんと動作しているようです。

    $ node tools\electronbuild.js --target=0.30.2 --arch=ia32
    
ためしに、 OS を調べるコードを書いてみると、

```javascript
var os = require('os');
console.log(os.platform(), os.arch());
```

結果は、 **win32 ia32** でした。

うーむ、コントロールパネルで調べると 64ビット なんですが、なぜか os の判定は ia32 、、、

どこかのビルド時に 32ビット になっているんですかねー

ここは引き続き検証ゾーン！

## 今回のサンプルリポジトリ

[electron-various-tutorials/electron-kinect2 at master · hisasann/electron-various-tutorials](https://github.com/hisasann/electron-various-tutorials/tree/master/electron-kinect2)

## 関連リンク

[Electronを使った環境でnode-ffiが動かない(19029)｜teratail](https://teratail.com/questions/19029)

[node.js - node gyp error TRACKER : error TRK0005: Failed to locate: "CL.exe". The system cannot find the file specified - Stack Overflow](http://stackoverflow.com/questions/33183161/node-gyp-error-tracker-error-trk0005-failed-to-locate-cl-exe-the-system-c)

🍜
