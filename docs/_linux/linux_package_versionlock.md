# Locking a package version

## Ubuntu

###  バージョン固定
```linenums="0"
sudo apt-mark hold パッケージ名
```
例:
```linenums="0"
sudo apt-mark hold nginx
```

### バージョン固定状態確認
``` linenums="0"
apt-mark showhold
```

###  バージョン固定解除
``` linenums="0"
sudo apt-mark unhold パッケージ名
```
例:
```linenums="0"
sudo apt-mark unhold nginx
```

###  パッケージをどこからインストールしたかの確認
```linenums="0"
apt policy パッケージ名
```

## CentOS / Rocky Linux / Alma Linux

### 一時的な抑止
例: パッケージ名 'httpd' のアップデートを抑止
```linenums="0"
sudo dnf update --exclude=httpd
```

### 永続的な抑止

#### dnfのプラグインをインストール (なければ)
```linenums="0"
sudo dnf install dnf-plugin-versionlock
```

#### バージョン固定
例: パッケージ名 'kernel' のバージョンをロックする
```linenums="0"
sudo dnf versionlock add kernel
```

#### バージョン固定状態確認
```linenums="0"
sudo dnf versionlock list
```

#### バージョン固定解除
例: パッケージ名 'kernel' のロックを解除する
```linenums="0"
sudo dnf versionlock delete kernel
```

### /etc/dnf/dnf.confに記載してバージョンを固定する方法もある

例: httpd で始まるすべてのパッケージと my-packageを固定する
```linenums="0"
[main]
exclude=httpd* my-package*
```
