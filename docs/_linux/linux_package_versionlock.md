# Locking a package version

## Ubuntu

###  バージョン固定
``` bash
sudo apt-mark hold パッケージ名
```
例:
``` bash
sudo apt-mark hold nginx
```

### バージョン固定状態確認
``` bash
apt-mark showhold
```

###  バージョン固定解除
``` bash
sudo apt-mark unhold パッケージ名
```
例:
``` bash
sudo apt-mark unhold nginx
```

###  パッケージをどこからインストールしたかの確認
``` bash
apt policy パッケージ名
```

## CentOS / Rocky Linux / Alma Linux

### 一時的な抑止
例: パッケージ名 'httpd' のアップデートを抑止
``` bash
sudo dnf update --exclude=httpd
```

### 永続的な抑止

#### dnfのプラグインをインストール (なければ)
``` bash
sudo dnf install dnf-plugin-versionlock
```

#### バージョン固定
例: パッケージ名 'kernel' のバージョンをロックする
``` bash
sudo dnf versionlock add kernel
```

#### バージョン固定状態確認
``` bash
sudo dnf versionlock list
```

#### バージョン固定解除
例: パッケージ名 'kernel' のロックを解除する
``` bash
sudo dnf versionlock delete kernel
```

### /etc/dnf/dnf.confに記載してバージョンを固定する方法もある

例: httpd で始まるすべてのパッケージと my-packageを固定する
``` bash
[main]
exclude=httpd* my-package*
```
