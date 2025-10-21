# SSH settings requiring one-time passwords (TOTP) using Google Authenticator

## Rocky Linux
### 概要
SSHログイン時にGoogle Authenticatorによるワンタイムパスワード（TOTP）を必須として
セキュリティを向上させます。
Linux踏み台などに有効な方法です。

### 方法
!!! tip
	* SELinuxが有効な場合は権限で弾かれます。SELinuxは無効を推奨します。
	* 有効なSSHセッションを複数維持して作業をしましょう。
	* スマートフォンとサーバーの時刻がズレていると認証に失敗するので時刻を正確にしましょう。

#### epelリポジトリの追加
```bash
sudo dnf install epel-release
```

#### 必要パッケージのインストール
Google AuthenticatorのPAMモジュールをインストール
```bash
sudo dnf install google-authenticator
```
ターミナル画面にQRコードを表示するためのライブラリをインストール
```bash
sudo dnf install qrencode
```

#### Google Authenticatorの初期設定
```bash
google-authenticator
```
ワンタイムパスワードを時間ベースにしますか？ → y を選択
```bash
Do you want authentication tokens to be time-based (y/n) y 
```
QRコードやシークレットキー、リカバリコードが表示されます。QRコードはスマートフォンのGoogle Authenticatorアプリで読み取ってください。

設定ファイルをホームディレクトリに保存しますか？ → y を選択
```bash
Do you want me to update your "/home/user/.google_authenticator" file? (y/n) y
```
同じワンタイムパスワードの再利用を禁止しますか？ → y を選択
```bsah
Do you want to disallow multiple uses of the same authentication
token? This restricts you to one login about every 30s, but it increases
your chances to notice or even prevent man-in-the-middle attacks (y/n) y
```

サーバーとクライアントの時刻ズレを考慮して有効期間を延ばしますか？ → 通常は n で問題ありません
```bash
By default, a new token is generated every 30 seconds by the mobile app.
In order to compensate for possible time-skew between the client and the server,
we allow an extra token before and after the current time. This allows for a
time skew of up to 30 seconds between authentication server and client. If you
experience problems with poor time synchronization, you can increase the window
from its default size of 3 permitted codes (one previous code, the current
code, the next code) to 17 permitted codes (the 8 previous codes, the current
code, and the 8 next codes). This will permit for a time skew of up to 4 minutes
between client and server.
Do you want to do so? (y/n) n
```

ブルートフォース攻撃対策としてレートリミットを有効にしますか？ → y
```bash
If the computer that you are logging into isn't hardened against brute-force
login attempts, you can enable rate-limiting for the authentication module.
By default, this limits attackers to no more than 3 login attempts every 30s.
Do you want to enable rate-limiting? (y/n) y
```

#### PAM (Pluggable Authentication Modules) の設定
```bash
sudo vim /etc/pam.d/sshd
```
ファイルの一番上に、以下の1行を追加します。
!!! danger
	nullok オプションは、まだ .google_authenticator ファイルを作成していないユーザーでも、一時的に公開鍵認証のみでログインできるようにするためのものです。全ユーザーの設定が完了したら、セキュリティ向上のためこのオプションは削除することを推奨します。

```bash
auth       required     pam_google_authenticator.so nullok
```
ファイルの以下の1行をコメントアウトする
```bash
#auth       substack     password-auth
```

#### SSHデーモンの設定変更
##### sshd_configの編集
```bash
sudo vim /etc/ssh/sshd_config
```
以下の設定を追加
!!! Note
	この設定で「最初に公開鍵認証を行い、成功したら次に対話形式の認証（今回はGoogle Authenticator）を行う」という認証フローを定義します。

```bash
AuthenticationMethods publickey,keyboard-interactive
```
##### 50-redhat.confの編集
```bash
sudo vim /etc/ssh/sshd_config.d/50-redhat.conf
```
以下の1行を変更します。

[変更前]
```bash
ChallengeResponseAuthentication no
```

[変更後] yesに変更します。
```bash
ChallengeResponseAuthentication yes
```

### 設定の反映
```bash
sudo systemctl restart sshd
```

#### SSHログインの確認
ワンタイムコードでログインができたらPAMの設定を適切に設定します。
```bash
sudo vim /etc/pam.d/sshd
```
[変更前]
```bash
auth       required     pam_google_authenticator.so nullok
```

[変更後] nullokオプションを削除します
```bash
auth       required     pam_google_authenticator.so
```

#### 設定の反映
```bash
sudo systemctl restart sshd
```

## Ubuntu

