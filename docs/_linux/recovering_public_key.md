# Recovering a public key from an SSH private key

## 目的
SSHの鍵ペアのうち公開鍵を紛失してしまった場合は以下のコマンドで秘密鍵から復元できる。

## SSH公開鍵の復元
```bash
 ssh-keygen -y -f ~/.ssh/id_rsa > ~/.ssh/id_rsa.pub
```
上記はRSA鍵の場合であるが、DSA鍵も同様の手順で復元できる。
```bash
ssh-keygen -y -t dsa -f ~/.ssh/id_dsa > ~/.ssh/id_dsa.pub
```
なお、秘密鍵を紛失してしまった場合は復元する術がないので再度鍵ペアを作成する必要がある。#
