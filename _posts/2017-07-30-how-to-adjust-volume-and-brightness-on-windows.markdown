---
layout: post
title:  "C#からWindows上のボリュームと輝度を変更する方法"
date:   2017-07-30 12:00:00
categories: C# windows visualstudio
---

意外とまとまった情報がなかったのでここにメモしておきます。

VisualStudio でコマンドラインツールとしてプロジェクトを作成し、その後プロジェクトのプロパティで「出力の種類」を **Windows アプリケーション** にすると完全に UI レスで exe を実行できました。

試した環境は Windows 10 です。

cmd.exe から引数を指定して実行すれば、ボリュームと輝度をプログラムから変更可能です。


## Windows 上でボリュームを変更するコード

0 ～ 100 の値を渡します。

<script src="https://gist.github.com/hisasann/afd734e688893df345e05185864b5345.js"></script>

呼び出すコード

```c
VolumeController vc = new VolumeController();
vc.SetVolume(30);
```


## Windows 上で輝度を変更するコード

0 ～ 100 の値を渡します。

<script src="https://gist.github.com/hisasann/5c861191c70ab8b9cd43757de5f709e5.js"></script>

呼び出すコード

```c
BrightnessController bc = new BrightnessController();
bc.SetBrightness(50);
```
