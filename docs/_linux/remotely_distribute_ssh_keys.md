# SSH authentication key public key remote distribution

## 目的
リモートサーバーの~/.ssh/authorized_keysに追記する

## 前提
- パスワード認証でSSH接続が可能なこと
- ~/.ssh/authorized_keysがパーミッション644で作成されていること

## ssh-copy-id の利用
```linenums="0"
ssh-copy-id ${USER}@${target_host}
```
-iオプションで任意の公開鍵を指定することもできる
```linenums="0"
ssh-copy-id -i ${identity_file} ${USER}@${target_host}
```

!!! tip
	引数で公開鍵を指定しない場合には、.ssh/id_rsa.pubとなる (デフォルト) 


SSHのログインをパスワード認証から公開鍵認証に変更する
