# SEPM 14.3 RU10 アップグレード手順書 (WSUS共存環境)

## 1. 概要

本ドキュメントは、Windows Server 2016 環境において、**Symantec Endpoint Protection Manager (SEPM)** をバージョン **14.3 RU3** から **14.3 RU10** へアップグレードする手順を記述します。
対象サーバーには **WSUS (Windows Server Update Services)** が同居しており、ポートおよびリソースの競合を回避しながら作業を行う必要があります。

!!! warning "重要: WSUSとの共存リスクについて"
    本環境では WSUS (IIS) が稼働しています。SEPMとWSUSのポート競合（特に 80/443/8530/8531）によるアップグレード失敗を防ぐため、作業中は **IIS (W3SVC)** および **WSUSサービス** を停止する手順を遵守してください。

### 1.1 環境情報

| 項目 | 詳細 |
| :--- | :--- |
| **OS** | Windows Server 2016 |
| **現行バージョン** | SEPM 14.3 RU3 (ビルド 14.3.5413.3000) |
| **ターゲット** | SEPM 14.3 RU10 (ビルド 14.3.xxxx) |
| **共存アプリ** | WSUS (IIS稼働, ポート8530/8531使用) |
| **作業権限** | ローカル管理者 (Administrator) |

### 1.2 対象サービス一覧

作業対象となるSEPM関連サービスは以下の通りです。すべて稼働中であることを前提とします。

* Symantec Endpoint Protection Manager (`semsrv`)
* Symantec Endpoint Protection Manager APIサービス (`semapisrv`)
* Symantec Endpoint Protection Manager Webサーバー (`semwebsrv`)
* Symantec Endpoint Protection Manager Scan Service
* Symantec Endpoint Protection Manager WSC Service

---

## 2. 事前準備 (Pre-flight Check)

作業開始前に以下の情報を取得し、作業端末のテキストエディタ等に控えてください。

### 2.1 ポート使用状況の記録 (最重要)

アップグレード後の構成ウィザードで設定値を確認するため、現在のポート使用状況を確定させます。

```title="powershell" linenums="0"
# 管理者権限PowerShellで実行
# 現在のListenポートとプロセスIDをファイル出力または画面で確認
Get-NetTCPConnection | Where-Object { $_.State -eq 'Listen' } | Select-Object LocalPort, OwningProcess, @{Name="ProcessName";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}} | Sort-Object LocalPort | Format-Table -AutoSize
```

*確認および記録すべきポート:**
	* SEPMポート:** 8443, 9090, 8014, 8445, 8446 (環境依存のため実機確認必須)
	* WSUSポート:** 8530, 8531 (および 80, 443)

### 2.2 データベース認証情報の確認

* DB管理者パスワード:** 管理サーバー設定ウィザードで入力が必要です。
	* SQL Anywhere (組み込み) の場合はインストール時のパスワード。
	* SQL Server の場合は `sem5` ユーザー等のパスワード。

### 2.3 インストーラーの配置

* `Symantec_Endpoint_Protection_14.3.0_RU10_SEPM_JP.exe` を `C:\Work` 等に配置し、解凍済みであること。

### 2.4 システム要件確認

* Cドライブに **20GB以上** の空き容量があること。

## 3. バックアップ (必須)

!!! danger "バックアップ未実施での作業禁止"
    アップグレード失敗時、またはWSUS環境破損時の切り戻しのため、以下3点のバックアップを必ず実施してください。

### 3.1 災害復旧ファイルの退避

SEPMコンソール接続不能時に復旧するための最重要ファイルです。

* **対象パス:** `C:\Program Files (x86)\Symantec\Symantec Endpoint Protection Manager\Server Private Key Backup\recovery_timestamp.zip`
* **手順:** 上記ファイルをファイルサーバー等の **別筐体** へコピーする。

### 3.2 データベースバックアップ

1. [スタート] > [Symantec Endpoint Protection Manager] > [データベースのバックアップと復元] を起動。
2. [バックアップ] をクリックし、完了まで待機する。

### 3.3 システム完全バックアップ

* **仮想環境:** スナップショットを取得する。
* **物理環境:** イメージバックアップを取得する。

---

## 4. アップグレード実施手順

作業中はSEPMおよびWSUSの通信が停止します。

### 4.1 サービスの停止

ポートロックおよびリソース競合を防ぐため、以下の順序でサービスを停止します。

```title="powershell" linenums="0"
# 管理者PowerShellで実行

# 1. WSUS / IIS の停止 (ポート解放のため最優先)
Stop-Service WsusService
Stop-Service W3SVC

# 2. SEPM 主要サービスの停止
Stop-Service semsrv      # Manager
Stop-Service semapisrv   # API
Stop-Service semwebsrv   # Webserver

# 3. その他のSEPM関連サービス停止（念のため）
Get-Service "Symantec Endpoint Protection Manager*" | Stop-Service -Force
```

### 4.2 インストーラーの実行

1. 解凍フォルダ内の `setup.exe` を右クリックし、**「管理者として実行」** します。
2. **[ようこそ]** 画面: 「次へ」。
3. **[使用許諾契約]** 画面: 「同意します」を選択し「次へ」。
4. **[インストールタイプ]** 画面:
    * **「管理サーバーとコンソールのアップグレード」** が選択されていることを確認し「次へ」。
5. **[警告]** 画面: 内容を確認し「次へ」。(ファイルのコピーが開始されます)
6. **[インストールの完了]** 画面:
    * 「管理サーバー設定ウィザードを起動する」にチェックを入れたまま「完了」をクリック。

### 4.3 管理サーバー設定ウィザード

!!! note "チェックポイント"
    ここでポート設定が初期化されていないか、WSUSのポートになっていないか慎重に確認してください。

1. **[ようこそ]**: 「次へ」。
2. **[データベースサーバーの認証]**:
    * **事前準備2.2 で確認したパスワード** を入力する。
3. **[ポートの構成] (重要)**:
    * 表示されたポート番号が、**事前準備 2.1 で記録した SEPM のポートと一致しているか** 確認する。
    * WSUS のポート (8530/8531) が入力されていないことを確認する。
    * **変更不要であれば、そのまま「次へ」**。
4. **[設定]**:
    * DBスキーマの更新とサービスの再登録が実行されます。15分～数十分程度かかります。
5. 完了画面が表示されたら終了します。

---

## 5. 事後確認

### 5.1 OS再起動

システムの整合性を保つため、サーバーを再起動します。

### 5.2 サービスの起動確認

再起動後、PowerShell で全サービスが `Running` であることを確認します。

```title="powershell" linenums="0"
# SEPM関連サービスの確認
Get-Service | Where-Object { $_.DisplayName -like "Symantec Endpoint Protection Manager*" } | Select-Object Status, DisplayName

# WSUS/IISサービスの確認
Get-Service WsusService, W3SVC
```

### 5.3 ポート共存確認

SEPMとWSUSが正しいポートでリッスンしているか確認します。

```title="powershell" linenums="0"
Get-NetTCPConnection | Where-Object { $_.LocalPort -in 8530, 8443, 9090, 8014, 8446 } | Select-Object LocalPort, OwningProcess, State
```

| ポート | 期待されるプロセス | サービス |
| :--- | :--- | :--- |
| **8530 / 8531** | System / w3wp | WSUS (IIS) |
| **8443 / 9090** | semsrv / httpd | SEPM |
| **8014** | httpd | SEPM Client Comm |
| **8446** | semapisrv | SEPM API |

### 5.4 動作確認

1. SEPMコンソール:** ログインし、[ヘルプ] > [バージョン情報] が **14.3 RU10** であることを確認。
1. WSUSコンソール:** 起動し、接続エラーが出ないことを確認。

## 6. トラブルシューティング

### Webサーバーサービス (semwebsrv) が起動しない場合

IIS (W3SVC) が先に起動し、SEPM 用のポート（特に 443 等）を占有している可能性があります。

1. Apacheエラーログの確認:**
    * パス: `C:\Program Files (x86)\Symantec\Symantec Endpoint Protection Manager\apache\logs\error_xxxx.log`
    * "Address already in use" 等の記述がないか確認。
1. 設定ファイルの確認:**
    * パス: `C:\Program Files (x86)\Symantec\Symantec Endpoint Protection Manager\apache\conf\httpd.conf`
    * エディタ (`vi` 等) で開き、`Listen` ディレクティブに競合ポートが含まれていないか確認する。

### 切り戻し (Rollback)

アップグレードにより致命的な問題が発生した場合は、以下の手順で復旧を行う。

1. SEPM RU10 のアンインストール。
1. SEPM RU3 (旧バージョン) の新規インストール。
1. 構成ウィザードにて「管理サーバーの復旧」を選択し、退避した `recovery_timestamp.zip` を使用して復元する。

## 7. 参考文献

* [Broadcom: Endpoint Protection Manager のアップグレードまたは移行](https://knowledge.broadcom.com/external/article/178523/)
* [Broadcom: Symantec Endpoint Protection 14.x のアップグレードパス](https://knowledge.broadcom.com/external/article/151658/)
