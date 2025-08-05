# Identify programs that expose files

## Overview
Linuxのlsofコマンドはファイルを開いているプログラムを特定するためのコマンドです。

### ポートを利用しているプロセスの確認
``` bash
lsof -I
```
ポートを開いている全てのプロセスが表示されます。

ポート番号で絞る場合には以下のように実行します。
```bash
lsof -I:ポート番号
```
ex lsof -I:80

### プロセスが掴んでいるファイルをサイズの小さい順でソートする
プロセスが掴んでいるファイルが増大してディスク容量を圧迫する場合があります。調査に有用なコマンドライン。  
ex Rsyslogを再起動せずに、書き込んでいるファイルを削除してしまったら、存在しないファイルに書き込み続けてディスク使用量が増大し続けるなど。

``` bash
lsof \
| grep REG \
| grep -v "stat: No such file or directory" \
| grep -v DEL \
| awk '{if ($NF=="(deleted)") {x=3;y=1} else {x=2;y=0}; {print $(NF-x) "  " $(NF-y) } }'  \
| sort -n -u  \
| numfmt  --field=1 --to=iec
```
