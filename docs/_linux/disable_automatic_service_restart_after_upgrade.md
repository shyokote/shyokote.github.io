# Disable automatic service restart after apt upgrade

## 概要
Ubuntu / Debian ではapt upgrade後に必要に応じてサービスのリスタートがかかります。
リスタートを運用者でコントロールしたい場合には無効化(List表示のみ)にした方が良い場合もあります。

## 方法
needrestart というツールの設定を変更して対処します。

### 設定ファイルの編集
```linenums="0"
sudo vi /etc/needrestart/needrestart.conf
```

編集項目

以下の部分に注目してください。
```title="/etc/needrestart/needrestart.conf" linenums="0"
#$nrconf{restart} = 'i';
```
上記を以下のように変更します。
```title="/etc/needrestart/needrestart.conf" linenums="0"
$nrconf{restart} = 'l';
```
これでapt upgrade後に自動でサービスをリスタートすることなく、リストの表示に留まってくれます。
