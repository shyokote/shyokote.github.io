# Ansible Vault

## 説明
Ansible Vault は、Ansible で扱う機密情報（パスワードや API キーなど）を暗号化して管理するための機能です。
通常の Playbook では、環境変数や別の管理ツールで機密情報を扱うことが一般的ですが、Ansible Vault を使用することで、Ansible のワークフローの中で安全に機密情報を扱うことができます。

## ファイルの作成
vaultで暗号化したファイルを作成する。例：secrets.yml
```bash
ansible-vault create secrets.yml
```
パスワードの設定を効かれるのでパスワードを設定する
```bash
New Vault password:
```
ファイルが開くのでファイルの内容を記述する

## ファイルの編集
```bash
ansible-vault edit secrets.yml --ask-vault-pass
```

## ファイル内容の確認
```bash
ansible-vault view secrets.yml --ask-vault-pass
```

## 非暗号ファイルを暗号化する方法
```bash
ansible-vault encrypt secrets.yml
```

## group_vars と host_vars
### group_vars
- inventoryに定義されているグループに対する定義
- allディレクトリまたはall.ymlはnsibleが内部で持っている特別なグループ名で、「インベントリに含まれる全てのホスト」を意味します。

#### 構成例例
- 方法1：group_vars直下に設定ファイルを作成する
- 方法2：ディレクトリを作成して管理する

```bash
group_vars/
│   ├── all.yml                  <--- (方法1) 全サーバー用のファイル
│   └── all/                     <--- (方法2) 全サーバー用のディレクトリ
│       ├── main.yml
│       └── vault.yml
```
ディレクトリを作成した方が管理しやすいのでオススメです。

また、inventoryで設定されているグループを指定する場合、(以下webというグループ)の設定も可能です。

```bash
group_vars/
│   ├── web.yml                  <--- (方法1) webサーバーグループ用のファイル
│   └── web/                     <--- (方法2) webサーバーグループ用のディレクトリ
│       ├── main.yml
│       └── vault.yml
```

### host_vars
host_varsも基本的にgroup_varsと同じです。
- inventoryに定義されているホストに対する定義
- ホスト名のディレクトリまたはホスト名.ymlを作成します。

#### 構成例
- 方法1：host_vars直下に設定ファイルを作成する
- 方法2：ディレクトリを作成して管理する

```bash
host_vars/
│   ├── server1.example.com.yml  <--- (方法1) server1用のファイル
│   └── server2.example.com/     <--- (方法2) server2用のディレクトリ
│       ├── main.yml
│       └── vault.yml
```
