# WSUS再インストール（SEPM共存環境）
## 1. 概要
本手順書は、Windows Server 2016上で稼働する Symantec Endpoint Protection Manager (SEPM) と共存している WSUS (Windows Server Update Services) を、SEPM環境に影響を与えずに再インストール（クリーンインストール）するための手順である。

対象環境
OS: Windows Server 2016 Standard
共存アプリケーション: Symantec Endpoint Protection Manager (SEPM)
作業目的: WSUSの不具合解消のための役割削除および再構築

!!! danger

    重要注意事項（作業前に必ず読むこと）

    * IIS (Web Server) の役割は絶対に削除しないこと
    * SEPMはIIS、またはIISとポートを共有するコンポーネントに依存しています。誤ってIIS全体を削除すると、SEPM管理コンソールおよびクライアント通信が停止します。
    * WID データベースファイルの削除

    役割の削除だけではデータベースファイル (SUSDB.mdf) が残留します。これを手動で削除しない限り、再インストールしても過去の不具合（DB破損など）を引き継ぐ可能性があります。

## 2. 事前準備・確認フェーズ
### 2.1. システムバックアップ
- フルバックアップの取得
- 仮想マシンの場合はスナップショットを取得する。
- 物理サーバーの場合はシステムバックアップを取得する。
- 目的: IIS構成破損時の切り戻し用

### 2.2. 現状構成の記録
- WSUS設定の記録 (スクリーンショット等)
    - 「製品とクラス」の選択状況
    - 「更新ファイルと言語」の設定
    - 「同期スケジュール」
    - 「アップストリームサーバー」の設定

- IISバインディングの確認
    - IISマネージャーを開き、既存のサイト構成を確認する。
    - Default Web Site および Symantec Endpoint Protection Manager のポート番号を控える。
    - 目的: 再構築後のポート競合回避

## 3. アンインストール作業フェーズ
### 3.1. WSUSの役割削除
1. サーバーマネージャーを起動し、[管理] > [役割と機能の削除] をクリック。
1. 対象サーバーを選択。
    -  サーバーの役割 の選択画面にて：
    -  Windows Server Update Services のチェックを 外す。
    -  Web Server (IIS) のチェックは 入れたままにする（変更しない）。
1. ウィザードを進め、[削除] を実行する。
1. 削除完了後、OSを再起動 する。

### 3.2. 残存データのクリーンアップ
再起動後、以下の手順で不要ファイルを削除する。

1. IISサイトの残存確認
    - IISマネージャーを開く。
    - WSUS Administration サイトが残っている場合は、右クリックして [削除] する。
    - 他のサイト（SEPM関連）は削除しないこと。
1. WID データベースの削除
    - PowerShell（管理者）または「サービス」画面から、WIDサービスを停止する。
```title="PowerShell" linenums="0"
Stop-Service -Name "WID"
```
※サービス名が見つからない場合は Windows Internal Database を探す。
- エクスプローラーで以下のパスを開く。 C:\Windows\WID\Data
- 以下の2ファイルを削除（または _old 等へリネーム退避）する。
    - SUSDB.mdf
    - SUSDB_log.ldf
- WIDサービスを開始する。
```title="PowerShell" linenums="0"
Start-Service -Name "WID"
```
1. コンテンツフォルダの処置
- 既存の保存先（例: C:\WSUS）を C:\WSUS_OLD などにリネームするか、削除する。
- 新たに空のフォルダ（例: C:\WSUS）を作成しておく。

### 3.3. 中間動作確認
SEPMの動作確認
- SEPM管理コンソールへログインできることを確認する。
- 既存のIISサイトが停止していないか確認する。

## 4. 再インストール作業フェーズ
### 4.1. WSUSの役割追加
1. サーバーマネージャー > [管理] > [役割と機能の追加]。
1. サーバーの役割 にて Windows Server Update Services にチェックを入れる。
1. 機能 選択画面はデフォルトのまま次へ。
1. WSUS - 役割サービス の選択画面にて以下にチェックが入っていることを確認。
    - [x] WID Connectivity
    - [x] WSUS Services
1. コンテンツの場所 の選択画面にて、作成したパス（例: C:\WSUS）を指定。
1. [インストール] を実行する。
1. インストール完了後、必要に応じてOS再起動を行う（保留中の再起動がある場合）。

### 4.2. 展開後構成 (Post-Install)
役割追加だけではIISへの登録やDB作成が行われていないため、以下を実施する。

1. 構成タスクの実行
- 管理者権限でPowerShellを開き、以下のコマンドを実行する。
```title="PowerShell" linenums="0"
& 'C:\Program Files\Update Services\Tools\wsusutil.exe' postinstall CONTENT_DIR=C:\WSUS
```
※ C:\WSUS は実際のパスに置き換える。
- Postinstall completed successfully と表示されることを確認する。

## 5. 初期設定・動作確認フェーズ
### 5.1. IIS構成の確認
1. IISマネージャーを開く。
1. WSUS Administration サイトが作成されていることを確認。
1. 画面右側の「バインディング」を確認。
    - ポート 8530 (http)
    - ポート 8531 (https)
    - 上記が設定されており、SEPMの使用ポートと競合していないことを確認する。

### 5.2. WSUS初期設定
1. スタートメニューから「Windows Server Update Services」を開く。
1. 「WSUS構成ウィザード」が開始されるので、記録しておいた設定値に従って進める。
    - アップストリームサーバーの指定
    - プロキシ設定
    - 接続開始（最初の同期）
    - 言語、製品、クラスの選択
    - 同期スケジュール

### 5.3. 最終動作確認
- 同期テスト: 手動同期を実行し、エラーなく完了するか。
- クライアント接続: クライアントPCからWindows Updateを実行し、WSUSと通信できるか。
- SEPM確認: 全作業終了後、再度SEPMコンソールへのログインおよびクライアント通信に異常がないか確認する。


#### トラブルシューティング

- WIDデータが削除できない場合:
    - プロセスが掴んでいる可能性があります。サーバーを再起動した後、すぐにファイル削除を試みてください。
- Post-Installが失敗する場合:
    - IISにて WSUS Administration サイトが中途半端に残っている可能性があります。IISマネージャーからサイトを削除してから再実行してください。
    - ログファイルを確認してください: %TEMP%\Wsustup.log
