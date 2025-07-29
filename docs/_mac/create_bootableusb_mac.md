# Create a Bootable USB from an ISO image using Mac's standard functions

## 概要
USBメモリを接続してターミナルから操作をする。

## 方法

### USBデバイスの確認
USBメモリを接続する。
USBメモリのデバイスを確認する。

[コマンド]
``` bash
diskutil list
```
以下の出力例では /dev/disk8 がUSBメモリ
``` bash
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER 0:      GUID_partition_scheme                        *500.3 GB   disk0 1:             Apple_APFS_ISC Container disk1         524.3 MB   disk0s1
   2:                 Apple_APFS Container disk3         494.4 GB   disk0s2
   3:        Apple_APFS_Recovery Container disk2         5.4 GB     disk0s3

/dev/disk3 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +494.4 GB   disk3
                                 Physical Store disk0s2 1:                APFS Volume Macintosh HD            10.7 GB    disk3s1 2:              APFS Snapshot com.apple.os.update-... 10.7 GB    disk3s1s1 3:                APFS Volume Preboot                 6.6 GB     disk3s2
   4:                APFS Volume Recovery                976.5 MB   disk3s3
   5:                APFS Volume Data                    148.4 GB   disk3s5
   6:                APFS Volume VM                      1.1 GB     disk3s6

/dev/disk4 (disk image):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        +8.6 GB     disk4
   1:                 Apple_APFS Container disk5         8.6 GB     disk4s1

/dev/disk5 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +8.6 GB     disk5
                                 Physical Store disk4s1
   1:                APFS Volume iOS 18.0 Simulator B... 8.4 GB     disk5s1

/dev/disk6 (disk image):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        +19.1 GB    disk6
   1:                 Apple_APFS Container disk7         19.1 GB    disk6s1

/dev/disk7 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +19.1 GB    disk7
                                 Physical Store disk6s1
   1:                APFS Volume iOS 18.0 Simulator      18.6 GB    disk7s1

/dev/disk8 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *31.0 GB    disk8
   1:               Windows_NTFS TOSHIBA32G              31.0 GB    disk8s1
```

### USBメモリの初期化
対象ディスクをMS-DOS形式で初期化する。

[コマンド]
``` bash
diskutil eraseDisk MS-DOS UNTITLED /dev/disk8
```
UNTITLEDになっているものはラベルなので自由に。

### ISOイメージをディスクに書き込む
USBディスクに書き込むため、一旦アンマウント

[コマンド]
``` bash
diskutil unmountDisk /dev/disk8
```

ISOイメージをディスクに書き込む
[コマンド]
``` bash
sudo dd if=./xxxxxx.iso of=/dev/rdisk8 bs=16m
```

#### ofオプションについて

- disk … 通常のランダムアクセス
- rdisk … シーケンシャル(順次)アクセス

#### bsオプションについて

- 転送バイト数を指定しますが、デフォルトは512バイト。

### USBの取り出し

[コマンド]
``` bash
diskutil eject /dev/disk8
```

