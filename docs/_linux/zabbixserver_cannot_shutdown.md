# Zabbix server cannot be stopped or shut down

## 概要
Ubuntuで構築されたZabbixサーバーで
「job zabbix-server.service/stop runnning」や 「A stop job is running for Zabbix Server」と表示されたまま
OSのシャットダウンができない状態になる

## ユニット単位のタイムアウト設定を編集
今回はZabbixなのでzabbix-serverのファイルを探す

ユニット毎の設定ファイルは、man 5 systemd.unit を確認すればわかる。
```bash
   System Unit Search Path
       /etc/systemd/system.control/*
       /run/systemd/system.control/*
       /run/systemd/transient/*
       /run/systemd/generator.early/*
       /etc/systemd/system/*
       /etc/systemd/system.attached/*
       /run/systemd/system/*
       /run/systemd/system.attached/*
       /run/systemd/generator/*
       ...
       /usr/lib/systemd/system/*
       /run/systemd/generator.late/*

   User Unit Search Path
       ~/.config/systemd/user.control/*
       $XDG_RUNTIME_DIR/systemd/user.control/*
       $XDG_RUNTIME_DIR/systemd/transient/*
       $XDG_RUNTIME_DIR/systemd/generator.early/*
       ~/.config/systemd/user/*
       $XDG_CONFIG_DIRS/systemd/user/*
       /etc/systemd/user/*
       $XDG_RUNTIME_DIR/systemd/user/*
       /run/systemd/user/*
       $XDG_RUNTIME_DIR/systemd/generator/*
       $XDG_DATA_HOME/systemd/user/*
       $XDG_DATA_DIRS/systemd/user/*
       ...
       /usr/lib/systemd/user/*
       $XDG_RUNTIME_DIR/systemd/generator.late/*
```

```bash
for i in /etc/systemd/system.control /run/systemd/system.control /run/systemd/transient /run/systemd/generator.early /etc/systemd/system /etc/systemd/system.attached /run/systemd/system /run/systemd/system.attached /run/systemd/generator /usr/lib/systemd/system /run/systemd/generator.late; do  [ -f $i/*zabbix-server* ] && ls -l $i/*zabbix-server* ; done
```

実行結果から
```bash
-rw-r--r-- 1 root root 555 Sep 30 08:48 /usr/lib/systemd/system/zabbix-server.service
```
/usr/lib/systemd/system/zabbix-server.service であることが確認できる


ファイルの中身を確認すると
```bash
[Unit]
Description=Zabbix Server
After=syslog.target
After=network.target
After=mysql.service
After=mysqld.service
After=mariadb.service

[Service]
Environment="CONFFILE=/etc/zabbix/zabbix_server.conf"
EnvironmentFile=-/etc/default/zabbix-server
Type=forking
Restart=on-failure
PIDFile=/run/zabbix/zabbix_server.pid
KillMode=control-group
ExecStart=/usr/sbin/zabbix_server -c $CONFFILE
ExecStop=/bin/sh -c '[ -n "$1" ] && kill -s TERM "$1"' -- "$MAINPID"
RestartSec=10s
TimeoutSec=infinity
LimitNOFILE=65536:1048576

[Install]
WantedBy=multi-user.target
```
TimeoutSec=infinity の部分です。デフォルトで制限なしになっているので、
この行をコメントアウトします。
```bash
#TimeoutSec=infinity
```

## 設定の反映
```bash
sudo systemctl daemon-reload
```

!!! Tip
    zabbix-serverがアップデートされると設定が元に戻ってしまうので再度設定する必要があります
