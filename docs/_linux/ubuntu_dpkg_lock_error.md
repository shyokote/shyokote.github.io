# How to fix Ubuntu dpkg lock error

## 事象
apt upgrade を実施して「Waiting for cache lock」エラーが発生してアップデートできない

## エラー内容
```bash
Waiting for cache lock: Could not get lock /var/lib/dpkg/lock-frontend. It is held by process 3822
Waiting for cache lock: Could not get lock /var/lib/dpkg/lock-frontend. It is held by process 3822
Waiting for cache lock: Could not get lock /var/lib/dpkg/lock-frontend. It is held by process 3822
Waiting for cache lock: Could not get lock /var/lib/dpkg/lock-frontend. It is held by process 3822
Waiting for cache lock: Could not get lock /var/lib/dpkg/lock-frontend. It is held by process 3822
Waiting for cache lock: Could not get lock /var/lib/dpkg/lock-frontend. It is held by process 3822
```
## ロックファイルの削除
```bash
sudo rm -rf /var/lib/dpkg/lock /var/lib/dpkg/lock-frontend
```
念の為、キャッシュのクリアもしておく
```bash
sudo apt clean 
```

!!! tip
    今回はロックファイルを削除しただけだったが、色々なケースがあるようなのでその都度エラーメッセージを読んでの対応が必要