# Create a Bootable USB from an ISO image using Mac's standard functions

## 準備
USBメモリを接続してターミナルから操作をする。

## 方法

### USBデバイスの確認
USBメモリを接続する。
USBメモリのデバイスを確認する。

```linenums="0"
diskutil list
```
以下の出力例では /dev/disk8 がUSBメモリ
```
/dev/disk8 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *31.0 GB    disk8
   1:               Windows_NTFS TOSHIBA32G              31.0 GB    disk8s1
```

### USBメモリの初期化
対象ディスクをMS-DOS形式で初期化する。

```linenums="0"
diskutil eraseDisk MS-DOS UNTITLED /dev/disk8
```
UNTITLEDになっているものはラベルなので自由に。

### ISOイメージをディスクに書き込む
USBディスクに書き込むため、一旦アンマウント

```linenums="0"
diskutil unmountDisk /dev/disk8
```

ISOイメージをディスクに書き込む

```linenums="0"
sudo dd if=./xxxxxx.iso of=/dev/rdisk8 bs=16m
```

#### ofオプションについて

- disk … 通常のランダムアクセス
- rdisk … シーケンシャル(順次)アクセス

#### bsオプションについて

- 転送バイト数を指定しますが、デフォルトは512バイト。

### USBの取り出し

```linenums="0"
diskutil eject /dev/disk8
```

