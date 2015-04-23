---
layout: post
title:  "Macの起動時にChromeをキオスク(kiosk)モードで開く方法"
date:   2015-04-23 15:45:16
categories: mac kiosk chrome
---
こんな感じのシェルを *.sh* ファイルを作っておいて、

```shell-session
open "/Applications/Google Chrome.app" --args --kiosk --disable-background-mode "http://hisasann.com/" --user-data-dir=/Users/[UserName]/Library/Application\ Support/Google/Chrome/kioskmode
```

Macのログイン時の起動項目に追加するだけ。

これで、サイネージなどに起動した瞬間にキオスクモードのChromeが立ち上がります。

細かい起動オブションは以下を参考にするとよいです。

[起動オプション - Google Chrome まとめWiki](http://chrome.half-moon.org/43.html)
