# Removing unnecessary Linux kernels

## Ubuntu
Ubuntuで不要Linuxカーネルを削除する方法

### 現行のLinuxカーネルを確認する
```linenums="0"
uname -a
```

### インストール済みカーネル確認
```linenums="0"
dpkg --get-selections | grep linux-
```

### 削除テスト
例:
```linenums="0"
sudo apt-get --dry-run remove --purge linux-{image,headers,modules}-<バージョン>
```
--dry-run オプションをつけて実行する
```linenums="0"
sudo apt-get --dry-run autoremove --purge linux-{image,headers,modules}-5.15.0-{56,58}
```

### 削除本番

例:
```linenums="0"
sudo apt-get remove --purge linux-{image,headers,modules}-<バージョン>
```
--dry-run オプションを外して実行する
```linenums="0"
sudo apt-get autoremove --purge linux-{image,headers,modules}-5.15.0-{56,58}
```

### 確認
```linenums="0"
dpkg --get-selections | grep linux-
```

### パッケージをクリーンアップ
```
sudo apt-get autoremove
sudo apt-get clean
```

### GRUBのアップデート
GRUBブートローダーを更新して、削除したカーネルがブートメニューに表示されないようにする
```linenums="0"
sudo update-grub
```

## CentOS / Rocky Linux / Alma Linux
RedHat系Linuxで不要Linuxカーネルを削除する方法

### 現行のLinuxカーネルを確認する
```linenums="0"
uname -a
```

### インストール済みカーネルの確認
```linenums="0"
dnf list --installed | grep kernel
```

### 削除(最新のみ残す)
```linenums="0"
sudo dnf remove --oldinstallonly
```

### 削除(最新2世代を残す)
```linenums="0"
sudo dnf remove --oldinstallonly --setopt installonly_limit=2 kernel
```
