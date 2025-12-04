# Apps won't disappear from the Windows Start menu

## Overview
Windowsでアプリをアンインストールしているのにも関わらず、スタートメニューから消えないアプリを消す方法

## Method
Windowsから右クリック、ファイル名を指定して実行から

```linenums="0"
%appdata%\Microsoft\Windows\Start Menu\Programs
```
を入力するとスタートメニューに表示されているアプリやフォルダが見られる。

不要なものをそこから削除する。
