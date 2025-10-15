# Disable automatic service restart after apt upgrade

## 概要
Ubuntu / Debian ではapt upgrade後に必要に応じてサービスのリスタートがかかります。
リスタートを運用者でコントロールしたい場合には無効化(List表示のみ)にした方が良い場合もあります。

## 方法
needrestart というツールの設定を変更して対処します。

### 設定ファイルの編集
```bash
sudo vi /etc/needrestart/needrestart.conf
```

編集項目

以下の部分に注目してください。
```bash
#$nrconf{restart} = 'i';
```
上記を以下のように変更します。
```bash
$nrconf{restart} = 'l';
```
これでapt upgrade後に自動でサービスをリスタートすることなく、リストの表示に留まってくれます。
