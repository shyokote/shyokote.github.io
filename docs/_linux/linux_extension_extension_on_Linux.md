# LVM extension extension on Linux

## 目的
/ の容量を50Gから+50Gして合計100Gにする

## VM内でディスクを認識

### 既存の構成を確認する

```bash
lsblk
```
出力例
```bash
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0  50G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   48G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   48G  0 lvm  /
sr0                        11:0    1    4M  0 rom
sr1                        11:1    1 1024M  0 rom
```
sdaの容量を増加させることがわかる。

### Diskサイズの変更
仮想マシンのDiskを増加変更する。50G追加

### 構成確認
50G増えているか確認をする
```bash
lsblk
```
出力例
```bash
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0  100G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   48G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   48G  0 lvm  /
sr0                        11:0    1    4M  0 rom
sr1                        11:1    1 1024M  0 rom
```
sdaの欄のSIZEが100Gになっていることを確認


## パーディションを拡張
### パーティションの確認
```bash
sudo fdisk -l /dev/sda
```
出力例
```bash
GPT PMBR size mismatch (104857599 != 209715199) will be corrected by write.
The backup GPT table is not on the end of the device.
Disk /dev/sda: 100 GiB, 107374182400 bytes, 209715200 sectors
Disk model: QEMU HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: F549E4D1-A1CB-49B0-8905-724AFB465E24

Device       Start       End   Sectors Size Type
/dev/sda1     2048      4095      2048   1M BIOS boot
/dev/sda2     4096   4198399   4194304   2G Linux filesystem
/dev/sda3  4198400 104855551 100657152  48G Linux filesystem
```

### fdiskでパーティションを拡張
```bash
sudo fdisk /dev/sda
```

途中出力される以下のメッセージは注意です！
!!! danger
    Created a new partition 3 of type 'Linux filesystem' and of size 98 GiB.
    Partition #3 contains a LVM2_member signature.
    Do you want to remove the signature? [Y]es/[N]o:

    このメッセージは

    ・LVM2_member signature = そのパーティションが LVM の一部として使われている印（メタデータ）。

    ・既存の /dev/sda3 を拡張しようとしているので、既存のLVMシグネチャを消してはいけません。

    したがって、ここは N (No) を選んでください。


出力例
```bash
Welcome to fdisk (util-linux 2.39.3).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

GPT PMBR size mismatch (104857599 != 209715199) will be corrected by write.
The backup GPT table is not on the end of the device. This problem will be corrected by write.
This disk is currently in use - repartitioning is probably a bad idea.
It's recommended to umount all file systems, and swapoff all swap
partitions on this disk.


Command (m for help): p

Disk /dev/sda: 100 GiB, 107374182400 bytes, 209715200 sectors
Disk model: QEMU HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: F549E4D1-A1CB-49B0-8905-724AFB465E24

Device       Start       End   Sectors Size Type
/dev/sda1     2048      4095      2048   1M BIOS boot
/dev/sda2     4096   4198399   4194304   2G Linux filesystem
/dev/sda3  4198400 104855551 100657152  48G Linux filesystem

Command (m for help): d
Partition number (1-3, default 3): 3

Partition 3 has been deleted.

Command (m for help): p
Disk /dev/sda: 100 GiB, 107374182400 bytes, 209715200 sectors
Disk model: QEMU HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: F549E4D1-A1CB-49B0-8905-724AFB465E24

Device     Start     End Sectors Size Type
/dev/sda1   2048    4095    2048   1M BIOS boot
/dev/sda2   4096 4198399 4194304   2G Linux filesystem

Command (m for help): n
Partition number (3-128, default 3): 3
First sector (4198400-209715166, default 4198400):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4198400-209715166, default 209713151):

Created a new partition 3 of type 'Linux filesystem' and of size 98 GiB.
Partition #3 contains a LVM2_member signature.

Do you want to remove the signature? [Y]es/[N]o: N

Command (m for help): p

Disk /dev/sda: 100 GiB, 107374182400 bytes, 209715200 sectors
Disk model: QEMU HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: F549E4D1-A1CB-49B0-8905-724AFB465E24

Device       Start       End   Sectors Size Type
/dev/sda1     2048      4095      2048   1M BIOS boot
/dev/sda2     4096   4198399   4194304   2G Linux filesystem
/dev/sda3  4198400 209713151 205514752  98G Linux filesystem

Command (m for help): w
The partition table has been altered.
Syncing disks.
```
### 作成した新しいパーティションをカーネルに通知
```bash
sudo partprobe
```

## LVMの物理ボリュームを拡張

### 新しいパーティションを LVM の物理ボリュームとして認識させる
```bash
sudo pvresize /dev/sda3
```

## LVMの論理ボリュームを拡張

### ボリュームグループ名を確認する
```bash
sudo vgdisplay
```
出力例
```bash
  --- Volume group ---
  VG Name               ubuntu-vg
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <48.00 GiB
  PE Size               4.00 MiB
  Total PE              12287
  Alloc PE / Size       12287 / <48.00 GiB
  Free  PE / Size       0 / 0
  VG UUID               B874I4-bIDN-Koeo-NFmh-qu6l-qWob-ALGkV7
```

### 論理ボリューム名を確認する
```bash
sudo lvdisplay
```
出力例
```bash
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/ubuntu-lv
  LV Name                ubuntu-lv
  VG Name                ubuntu-vg
  LV UUID                F6QWub-Jw1x-GevC-r8h7-u9GX-fSUA-eksfod
  LV Write Access        read/write
  LV Creation host, time ubuntu-server, 2025-01-11 18:42:34 +0900
  LV Status              available
  # open                 1
  LV Size                <48.00 GiB
  Current LE             12287
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:0
```

### 論理ボリュームを必要な分だけ拡張する
ここの例では追加分を全て適用する
```bash
sudo lvextend -l +100%FREE /dev/mapper/<volume-group-name>-<logical-volume-name>
```
/dev/mapper/<volume-group-name>-<logical-volume-name>の部分は、
vgdisplayコマンドとlvdisplayコマンドで確認できます。

!!! tip
    &lt;volume-group-name&gt;-&lt;logical-volume-name&gt;"の部分をvgdisplayコマンドとlvdisplayコマンドで確認すると
    ubuntu-vg-ubuntu-lvなるはずですがr、lsblkやdfの結果だとubuntu--vg-ubuntu--lvとなっています。
    デバイスマッパー的には、 - は「VGとLVを区切るための記号」なので、デバイスマッパーが展開した形だと
    ハイフンが増えます。これはエスケープの仕様です。ubuntu-vg-ubuntu-lvをコマンドに渡すとエラーになります。
    そのため、コマンドで使用する際には、以下のどちらかで指定してください。

    ・ハイフンを一個足してパスではないと明示的にする →  /dev/mapper/ubuntu--vg-ubuntu--lv

    ・パスの-を/に置き換えて渡す → /dev/ubuntu-vg/ubuntu-lv


例
```bash
% sudo  lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
  Size of logical volume ubuntu-vg/ubuntu-lv changed from <48.00 GiB (12287 extents) to <98.00 GiB (25087 extents).
  Logical volume ubuntu-vg/ubuntu-lv successfully resized.
root@pxc02:~# resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
resize2fs 1.47.0 (5-Feb-2023)
Filesystem at /dev/mapper/ubuntu--vg-ubuntu--lv is mounted on /; on-line resizing required
old_desc_blocks = 6, new_desc_blocks = 13
The filesystem on /dev/mapper/ubuntu--vg-ubuntu--lv is now 25689088 (4k) blocks long.
```

## ファイルシステムを拡張

### Ext4の場合
```bash
sudo resize2fs /dev/mapper/<volume-group-name>-<logical-volume-name>
```
例
```bash
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```

### XFSの場合
```bash
sudo xfs_growfs /mount/point
```
/mount/pointの部分は、df -Th か lsblk コマンドで確認してください。

例
```bash
sudo xfs_growfs /
```

## ディスク容量の確認
```bash
df -Th
```
例
```bash
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              794M  1.1M  793M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   97G  5.2G   87G   6% /
tmpfs                              3.9G     0  3.9G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
tmpfs                              3.9G     0  3.9G   0% /run/qemu
/dev/sda2                          2.0G   95M  1.7G   6% /boot
tmpfs                              794M   16K  794M   1% /run/user/1001
```
/ が100Gに拡張されています。

