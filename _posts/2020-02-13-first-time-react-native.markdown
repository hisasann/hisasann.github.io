---
layout: post
title:  "初めてのReactNative-ビルド編🍱"
date:   2020-02-13 01:00:00
category: ReactNative
tags:
    - React
    - node.js
    - Android
    - iOS
---

## 🦑 まえがき

[Getting Started · React Native](https://facebook.github.io/react-native/docs/getting-started.html#starting-the-android-virtual-device)

RN なプロジェクトを作って、ビルドを通すまでのメモになります。

Mac をせっかく新しくしたので、いちからやってみました！

## ReactNativeプロジェクトをWebStormから作る

[React Native - 公式ヘルプ WebStorm](https://pleiades.io/help/webstorm/react-native.html)

WebStorm にせっかく RN プロジェクトを作成する機能があるので、使ってみました。

    File -> -> New -> Project -> ReactNative

から作成しました。

しばらく待つとプロジェクトが作成されますので、 iOS と Android でビルドしていきます。

## iOSビルドをする

プロジェクトが作成される段階の、

    ✔ Processing template
    ✔ Installing CocoaPods dependencies (this may take a few minutes)

ここの段階で **CocoaPods** が入っていないエラーが出ました。

```bash
? CocoaPods (https://cocoapods.org/) is not installed. CocoaPods is necessary for the iOS project to run correctly. Do you want to install it? Yes, with Homebrew
```

Homebrew or gem と選択肢が出てきたので、 Homebrew を選んだが、結果的にうまくいかなかったので、 **gem 経由** でインストールしました。

```bash
checking whether the C compiler works... no
xcrun: error: SDK "iphoneos" cannot be located
xcrun: error: SDK "iphoneos" cannot be located
xcrun: error: SDK "iphoneos" cannot be located
xcrun: error: unable to lookup item 'Path' in SDK 'iphoneos'
/Users/hisasann/Library/Caches/CocoaPods/Pods/External/glog/2263bd123499e5b93b5efe24871be317-1f3da/missing: Unknown `--is-lightweight' option
Try `/Users/hisasann/Library/Caches/CocoaPods/Pods/External/glog/2263bd123499e5b93b5efe24871be317-1f3da/missing --help' for more information
configure: WARNING: 'missing' script is too old or missing
configure: error: in `/Users/hisasann/Library/Caches/CocoaPods/Pods/External/glog/2263bd123499e5b93b5efe24871be317-1f3da':
configure: error: C compiler cannot create executables
See `config.log' for more details

(node:17941) UnhandledPromiseRejectionWarning: Error: Failed to install CocoaPods dependencies for iOS project, which is required by this template.
Please try again manually: "cd ./ReactNativeFirstTime/ios && pod install".
CocoaPods documentation: https://cocoapods.org/
    at installPods (/Users/hisasann/_/js-code/ReactNativeFirstTime/node_modules/@react-native-community/cli/build/tools/installPods.js:207:13)
    at processTicksAndRejections (internal/process/task_queues.js:94:5)
(node:17941) UnhandledPromiseRejectionWarning: Unhandled promise rejection. This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). (rejection id: 3)
(node:17941) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
Done
```

また、そもそも、

```bash
xcrun: error: SDK "iphoneos" cannot be located
```

と `XCode` のパスが合っていないようだったので、こちらも、コマンドから修正しました。

```bash
$ sudo xcode-select --switch /Applications/Xcode.app
```

[Mac - XCode - SDK "iphoneos" cannot be located - how to fix](https://www.ryadel.com/en/xcode-sdk-iphoneos-cannot-be-located-mac-osx-error-fix/)

これで CocoaPods がやっとこさ入りました。

```bash
$ sudo gem install cocoapods
```

[CocoaPods.org](https://cocoapods.org/)

あとは、再度 ios ディレクトリで pod install を実行しましたが、

```bash
$ cd ./ReactNativeFirstTime/ios && pod install
```

以下のようなエラーが出ました。

```bash
$ pod install                                                                                                                                                                                            Tue Feb 11 18:00:28 2020
Traceback (most recent call last):
  4: from /usr/local/bin/pod:23:in `<main>'
  3: from /usr/local/bin/pod:23:in `load'
  2: from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.8.4/bin/pod:55:in `<top (required)>'
  1: from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.8.4/lib/cocoapods/command.rb:50:in `run'
/Library/Ruby/Gems/2.6.0/gems/cocoapods-1.8.4/lib/cocoapods/command.rb:169:in `verify_xcode_license_approved!': [!] You have not agreed to the Xcode license, which you must do to use CocoaPods. Agree to the license by running: `xcodebuild -license`. (Pod::Informative)
```

エラーに書いてあるとおり

```bash
$ xcodebuild -license
```

を実行します。

```bash
Agreeing to the Xcode/iOS license requires admin privileges, please run “sudo xcodebuild -license” and then retry this command.
```

と出たので、

```bash
$ sudo xcodebuild -license
```

```bash
You have not agreed to the Xcode license agreements. You must agree to both license agreements below in order to use Xcode.
```

いたちごっこすぎる〜と思ったのですが、そういえば Mac を新しくしてまだ一回も XCode を起動していないことに気が付き、起動しライセンスに承認しました。

そして再び、

```bash
$ pod install
Analyzing dependencies
Downloading dependencies
Generating Pods project
Integrating client project
Pod installation complete! There are 28 dependencies from the Podfile and 26 total pods installed.
```

うまく依存ライブラリたちがインストールされました！

そして、 WebStorm 上で iOS ビルドしたらシミュレーターが起動してアプリもちゃんと起動しました。

## Androidビルドする

次は、 Android ビルドです。

さっそくエラーが出ました。

```bash
info Installing the app...
No Java runtime present, requesting install.

Error: Command failed: ./gradlew app:installDebug -PreactNativeDevServerPort=8081
No Java runtime present, requesting install.
```

そもそもですが、 macOS にはデフォルトで `Java が入らなくなった` のを思い出したので、自前でインストールしました。

```bash
$ java --version                                                                                                                                                                          Tue Feb 11 18:22:02 2020
No Java runtime present, requesting install.
```

`asdf` で各言語のバージョンを管理しているので、 java のプラグインを入れつつ、 jdk をインストールしていきました。

[asdf-vmで各言語のバージョン管理をしてみた🧞‍♀️ - DJ lemon-Sour's diary (prod.hisasann)](https://hisasann.github.io/2020/02/10/asdf-vm/)

[skotchpine/asdf-java: A Java plugin for asdf-vm.](https://github.com/skotchpine/asdf-java)

```bash
$ asdf plugin-add java
$ asdf list all java
```

```bash
$ asdf install java adopt-openjdk-13.0.2+8
Install jq to continue. Aborting.⏎
```

ここで、 `jq` がないとのことなので、

```bash
$ brew install jq
```

をしてから、再度 java のインストールを行いました。

```bash
$ asdf global java adopt-openjdk-13.0.2+8
```

```bash
$ java --version
openjdk 13.0.2 2020-01-14
OpenJDK Runtime Environment AdoptOpenJDK (build 13.0.2+8)
OpenJDK 64-Bit Server VM AdoptOpenJDK (build 13.0.2+8, mixed mode, sharing)
```

[donbeave ( Alexey Zhokhov )](https://www.gitmemory.com/donbeave)

java をインストールできました！

そして Android ビルドを再度行いましたが、 `adb` が入っていないとエラーになりました。

```bash
info Launching emulator...
/bin/sh: adb: command not found
```

[Android Studio のインストール  Android デベロッパー  Android Developers](https://developer.android.com/studio/install?hl=ja)

AndroidStudio を入れると Android 開発に必要なもろもろが自動的にインストールされるので、こちらに頼ります。

そして、各ツールにパスが通るように `config.fish` ファイルを編集します。

```bash
$ vim ~/.config/fish/config.fish
```

```bash
set -U FZF_LEGACY_KEYBINDINGS 0
set -U FZF_REVERSE_ISEARCH_OPTS "--reverse --height=100%"

set ANDROID_HOME $HOME/Library/Android/sdk
set PATH $ANDROID_HOME/emulator $PATH
set PATH $ANDROID_HOME/tools $PATH
set PATH $ANDROID_HOME/tools/bin $PATH
set PATH $ANDROID_HOME/platform-tools $PATH

source ~/.asdf/asdf.fish
source /usr/local/opt/asdf/asdf.fish
source ~/.asdf/asdf.fish
source ~/.asdf/asdf.fish
```

[macos - Fish adb not found - Ask Different](https://apple.stackexchange.com/questions/270408/fish-adb-not-found)

この時点では、まだ WebStorm で Android ビルドしてもエラーが出ます。

```bash
info Launching emulator...
error Failed to launch emulator. Reason: Emulator exited before boot..
warn Please launch an emulator manually or connect a device. Otherwise app may fail to launch.
```

Android シミュレーターの作成ができていないので、

Android Studio で `AVD Manager` からシミュレーターとして起動するデバイスを選択しました。

ここまで準備して、 Android Studio で RN プロジェクトの **android** ディレクトを開き、ビルドしてシミュレーターを起動した状態にします。

すると、 `8081` ポートで listen 状態になるので、 WebStorm で Android ビルドします。

これで無事シミュレーターで RN プロジェクトが起動しました！

## HMRについて

上記のように iOS と Android ビルドした状態で **App.js** を編集すると、即座にシミュレーターに反映されました。

## 🍜 あとがき

Titanium Mobile や Apache Cordova 、 Unity などでも `1ソース2パブリッシュ` をしてきましたが、いまのところ HMR の速さ具合はかなり良いです。

TypeScript 版もこれから試しますが、ひじょうに楽しそうです。

追伸：シミュレーターを起動すると、やっぱり Mac のファンが回っちゃいますね〜
