# Removing unnecessary Linux kernels

## Ubuntu
Ubuntuで不要Linuxカーネルを削除する方法

### 現行のLinuxカーネルを確認する
``` bash
uname -a
```

### インストール済みカーネル確認
``` bash
dpkg --get-selections | grep linux-
```

### 削除テスト
例:
``` bash
sudo apt-get --dry-run remove --purge linux-{image,headers,modules}-<バージョン>
```
--dry-run オプションをつけて実行する
``` bash
sudo apt-get --dry-run autoremove --purge linux-{image,headers,modules}-5.15.0-{56,58}
```

### 削除本番

例:
``` bash
sudo apt-get remove --purge linux-{image,headers,modules}-<バージョン>
```
--dry-run オプションを外して実行する
``` bash
sudo apt-get autoremove --purge linux-{image,headers,modules}-5.15.0-{56,58}
```

### 確認
``` bash
dpkg --get-selections | grep linux-
```

### パッケージをクリーンアップ
``` bash
sudo apt-get autoremove
sudo apt-get clean
```

### GRUBのアップデート
GRUBブートローダーを更新して、削除したカーネルがブートメニューに表示されないようにする
``` bash
sudo update-grub
```


## CentOS / Rocky Linux / Alma Linux
RedHat系Linuxで不要Linuxカーネルを削除する方法

### 現行のLinuxカーネルを確認する
``` bash
uname -a
```

### インストール済みカーネルの確認
``` bash
dnf list --installed | grep kernel
```

### 削除(最新のみ残す)
``` bash
sudo dnf remove --oldinstallonly
```

### 削除(最新2世代を残す)
``` bash
sudo dnf remove --oldinstallonly --setopt installonly_limit=2 kernel
```
