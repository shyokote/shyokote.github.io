# Building a git environment

## 1. SSHキー生成
```linenums="0"
ssh-keygen -t ed25519 -C ""
```

## 2. githubにキーを登録
SSHキー生成で作成されたパブリックキーをコピーする
```linenums="0"
cat ~/.ssh/id_ed25519.pub
```

- githubのページにアクセスして、右上のユーザーアイコンクリック > settings > SSH and GPG keys に移動してSSH keysのNew SSH keyをクリックして登録
    - Titleは自分のわかりやすい名前を入力、Keyの部分にコピーしたパブリックキーを貼り付け

## 3. github初期設定
```linenums="0"
# 設定確認(初期は何も出力されない)
git config --list

# ユーザー名設定 (githubのアカウント名を指定する)
git config --global user.name "USERNAME"

# 設定確認
git config --list
# user.name=USERNAME

# configファイルがなければ作成
touch ~/.ssh/config
```
configファイルに以下の内容を記載．IdentityFileは作成したファイルパスを記載．
```title="~/.ssh/config" linenums="0"
Host github.com
  User git
  HostName github.com
  IdentityFile ~/.ssh/id_ed25519   # <-- SSH秘密鍵を指定
  IdentitiesOnly yes
  AddKeysToAgent yes
  UseKeychain yes     # <---Macの場合のみ。Linuxなどだとエラーになるのでこの項目は削除
  ServerAliveInterval 600
  TCPKeepAlive yes
  IPQoS lowdelay throughput
  GSSAPIAuthentication no
```

## 4. gitの拡張機能を入れる
```linenums="0"
# 入力補完
wget https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash -O ~/.git-completion.bash
chmod a+x ~/.git-completion.bash
echo "source ~/.git-completion.bash" >> ~/.bashrc

# 拡張情報表示
wget https://raw.githubusercontent.com/git/git/master/contrib/completion/git-prompt.sh -O ~/.git-prompt.sh
chmod a+x ~/.git-prompt.sh
echo "source ~/.git-prompt.sh" >> ~/.bashrc
```
### 4.1. ~/.bashrcに設定を追加する
```title="~/.bashrc" linenums="0"
# プロンプトに各種情報を表示 1 or 空欄
GIT_PS1_SHOWDIRTYSTATE=1 # addされてないときに*、commitされてないときに+を表示
GIT_PS1_SHOWUPSTREAM=1 # 現在のブランチがupstreamより進んでいたら>、遅れていたら<、遅れてて変更ありなら<>が表示
GIT_PS1_SHOWUNTRACKEDFILES= # addされてない新規ファイルがあったら%を表示
GIT_PS1_SHOWSTASHSTATE=1 # stashがあったら$を表示

#プロンプト設定 (Ubuntuの場合。お好みでカスタマイズ)
if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[00;32m\]\u@\h\[\033[00m\]:\[\033[00;34m\]\w\[\033[00m\]\[\033[00;31m\]$(__git_ps1)\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w$(__git_ps1 " (%s)")\$ '
fi
unset color_prompt force_color_prompt
```

### 4.2. .bashrcを再読み込みして設定を反映
```linenums="0"
source ~/.bashrc
```

## 5. GitでSSHのパスフレーズ入力を省略する
### 5.1. SSHエージェントの設定
```linenums="0"
echo 'eval "$(ssh-agent -s)"' >> ~/.bashrc
source ~/.bashrc
```
### 5.2. ssh-add でSSH秘密鍵をssh-agentに登録する
```linenums="0"
ssh-add /Users/[user_name]/.ssh/id_ed25519

Enter passphrase for /Users/[user_name]/.ssh/id_ed25519: <- SSHのパスフレーズを入力
Identity added: /Users/[user_name]/.ssh/id_ed25519 (<作成時に入力したコメント>)
```
### 5.3. 登録確認
```linenums="0"
ssh-add -l
```

## 6. ssh-addで登録したSSH秘密鍵の永続化

- Ubuntuで手順5を実施してもログインし直すと登録したSSH秘密鍵が消える問題がある(Ubuntu以外のディストリビューションでも同じかもしれないが未確認)
- keychainを利用して永続化する

### 6-1. keychainのインストール
```linenums="0"
sudo apt install keychain
```

### 6.2 ~/.bashrcに設定を追加する

- SSH公開鍵と秘密鍵がないと登録できないので.ssh以下に公開鍵と秘密鍵を用意しておく

```linenums="0"
echo "/usr/bin/keychain $HOME/.ssh/id_rsa" >> ~/.bashrc
echo "source $HOME/.keychain/`hostname`-sh" >> ~/.bashrc
```
この設定をする場合は手順5で設定した eval "$(ssh-agent -s) の設定は削除して問題ない。

### .bashrcを再読み込みして設定を反映
```linenums="0"
source ~/.bashrc
```
