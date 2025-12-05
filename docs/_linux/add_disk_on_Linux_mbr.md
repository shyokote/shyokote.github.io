# Add disk on Linux (MBR)

## 目的
新規にDiskを追加してマウントまでする
パーティション形式はMBRを利用した場合を説明する

## Disk追加
仮想マシンのDiskを追加しておく
今回追加したDiskは/dev/sdbとして100GB認識されていることを確認

## Diskが認識されていることを確認する
```linenums="0"
sudo fdisk -l
```
出力例
```
Disk /dev/sda: 50 GiB, 53687091200 bytes, 104857600 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xa9cb21b7

Device     Boot   Start       End   Sectors Size Id Type
/dev/sda1  *       2048   2099199   2097152   1G 83 Linux
/dev/sda2       2099200 104857599 102758400  49G 8e Linux LVM

Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/mapper/rl_rocky--linux8-root: 47 GiB, 50461671424 bytes, 98557952 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/mapper/rl_rocky--linux8-swap: 2 GiB, 2147483648 bytes, 4194304 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

## fdiskコマンドを使用し、領域を作成する
```linenums="0"
sudo fdisk /dev/sdb
```
出力例
```
Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x652d9457.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-209715199, default 2048): 2048
Last sector, +sectors or +size{K,M,G,T,P} (2048-209715199, default 209715199): 209715199

Created a new partition 1 of type 'Linux' and of size 100 GiB.

Command (m for help): p
Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x652d9457

Device     Boot Start       End   Sectors  Size Id Type
/dev/sdb1        2048 209715199 209713152  100G 83 Linux

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

## パーティションの更新情報を認識させる
```linenums="0"
sudo partprobe
```

## パーティションをフォーマットする

=== "EXT4の場合"
    ```linenums="0"
    sudo mkfs -t ext4 /dev/sdb1
    ```
    出力例
    ```
    mke2fs 1.45.6 (20-Mar-2020)
    Creating filesystem with 26214144 4k blocks and 6553600 inodes
    Filesystem UUID: 58ed24d3-d503-4d01-b6ca-eb8a4e69177f
    Superblock backups stored on blocks:
	    32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	    4096000, 7962624, 11239424, 20480000, 23887872

    Allocating group tables: done
    Writing inode tables: done
    Creating journal (131072 blocks): done
    Writing superblocks and filesystem accounting information: done
    ```

=== "XFSの場合"
    ```linenums="0"
    sudo mkfs -t xfs /dev/sdb1
    ```

## Diskをマウントする
### マウントするディレクトリを作成する
ここでは/dataとする
```linenums="0"
sudo mkdir /data
```
### 追加したディスクの UUID を確認する
```linenums="0"
sudo blkid
```
出力例
```
/dev/sda1: UUID="7437fbdf-88c9-41a5-972d-2c7bb99fefce" BLOCK_SIZE="512" TYPE="xfs" PARTUUID="a9cb21b7-01"
/dev/sda2: UUID="yljnUK-UATf-KSKJ-coAq-b1DR-v4Xp-TppTkM" TYPE="LVM2_member" PARTUUID="a9cb21b7-02"    ←これ
/dev/sdb1: UUID="58ed24d3-d503-4d01-b6ca-eb8a4e69177f" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="652d9457-01"
/dev/mapper/rl_rocky--linux8-root: UUID="fe6bf2d2-ea61-437d-8e21-216ad20553aa" BLOCK_SIZE="512" TYPE="xfs"
/dev/mapper/rl_rocky--linux8-swap: UUID="8d48d1af-b357-4bc3-9036-8b3681299329" TYPE="swap"
```

### 自動マウント設定を行う。再起動後も自動でマウントされる
```linenums="0"
sudo vi /etc/fstab
```

=== "EXT4の場合"
    ```
    #
    # /etc/fstab
    # Created by anaconda on Thu Apr  6 08:40:19 2023
    #
    # Accessible filesystems, by reference, are maintained under '/dev/disk/'.
    # See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
    #
    # After editing this file, run 'systemctl daemon-reload' to update systemd
    # units generated from this file.
    #
    /dev/mapper/rl_rocky--linux8-root /                       xfs     defaults        0 0
    UUID=7437fbdf-88c9-41a5-972d-2c7bb99fefce /boot                   xfs     defaults        0 0
    /dev/mapper/rl_rocky--linux8-swap none                    swap    defaults        0 0
    UUID=58ed24d3-d503-4d01-b6ca-eb8a4e69177f /data ext4    defaults        1 2        ←これ
    ```

=== "XFSの場合"
    ```
    #
    # /etc/fstab
    # Created by anaconda on Thu Apr  6 08:40:19 2023
    #
    # Accessible filesystems, by reference, are maintained under '/dev/disk/'.
    # See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
    #
    # After editing this file, run 'systemctl daemon-reload' to update systemd
    # units generated from this file.
    #
    /dev/mapper/rl_rocky--linux8-root /                       xfs     defaults        0 0
    UUID=7437fbdf-88c9-41a5-972d-2c7bb99fefce /boot                   xfs     defaults        0 0
    /dev/mapper/rl_rocky--linux8-swap none                    swap    defaults        0 0
    UUID=58ed24d3-d503-4d01-b6ca-eb8a4e69177f /data xfs    defaults        1 2        ←これ
    ```


### 作成したディスクをマウントする
```linenums="0"
sudo mount /dev/sdb1 /data
```

### /dataがマウントされているか確認をする
```
df -h
Filesystem                         Size  Used Avail Use% Mounted on
devtmpfs                           1.9G     0  1.9G   0% /dev
tmpfs                              2.0G     0  2.0G   0% /dev/shm
tmpfs                              2.0G  8.5M  2.0G   1% /run
tmpfs                              2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/mapper/rl_rocky--linux8-root   47G  2.4G   45G   5% /
/dev/sda1                         1014M  207M  808M  21% /boot
tmpfs                              393M     0  393M   0% /run/user/1000
/dev/sdb1                           98G   24K   93G   1% /data    ←これ
```

