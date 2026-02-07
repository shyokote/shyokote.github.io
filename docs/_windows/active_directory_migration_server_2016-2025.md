# Active Directory 移行手順書（Windows Server 2016 to 2025）
## 要件
1. ADが利用できない時間は許容しない
1. Microsoftの推奨手順を踏襲する
1. 新サーバーは旧サーバーのホスト名とIPアドレスを引き継ぐ

## 環境
- ADDS01 (Windows Server 2016)
- ADDS02 (Windows Server 2016)
- 新ADDS01 (Windows Server 2025)
- 新ADDS02 (Windows Server 2025)

## 1. 移行前：前提条件・健全性確認フェーズ
移行作業を開始する前に、既存ドメインが「新しいDCを受け入れられる状態か」を以下の手順で確認します。

### ① ドメイン・フォレスト機能レベルの確認
Windows Server 2025を導入するには、機能レベルが Windows Server 2008 以上である必要があります。

確認コマンド:
```title="PowerShell" linenums="0"
# ドメイン機能レベルの確認
Get-ADDomain | fl Name, DomainMode
# フォレスト機能レベルの確認
Get-ADForest | fl Name, ForestMode
判断基準: 結果が Windows2008Domain（またはForest）以上であればOKです。もし Windows2003 等の場合は、事前にレベルアップ操作が必要です。
```
### ② SYSVOL 複製方式の確認 (FRS or DFSR)
Windows Server 2019以降、古い複製方式であるFRSはサポートされていません。

確認コマンド
```title="DOS" linenums="0"
dfsrmig /getglobalstate
判断基準: 出力が 「状態 '削除済み' (Eliminated)」 であれば移行済み。それ以外の場合は、まずDFSRへの移行作業（dfsrmig プロセス）を完遂させる必要があります。
```
### ③ 現行サーバーの健全性診断
確認コマンド:
```title="DOS" linenums="0"
dcdiag /v /c /d /e /s:ADDC01 > dcdiag_check.log
dcdiag_check.log  をテキストエディタなどで確認し、"failed" を検索
```
## 2. 第1段階：ADDC02 のリプレース手順
### ① 旧 ADDC02 の降格と撤去
FSMOの所在確認: netdom query fsmo で、全ての役割が ADDC01 にあることを確認。

降格実行: ADDC02にて「役割と機能の削除」からAD DSを削除（降格）し、ワークグループに参加させシャットダウン。

メタデータ・クリーンアップ確認: ADDC01側の「ADユーザーとコンピューター」および「DNSマネージャー」から、ADDC02の古いレコード（特に _msdcs 配下）が消えていることを目視確認。残っている場合は手動削除。

### ② 新 ADDC02 (2025) の構築と昇格
OS設定: 旧ADDC02と同一のホスト名・固定IPを付与。

優先DNS設定: 自身のIPではなく、稼働中の ADDC01のIP を優先DNSに指定。

AD昇格: 「既存のドメインにドメインコントローラーを追加する」オプションで昇格。

同期確認:
```title="DOS" linenums="0"
repadmin /showrepl /v
repadmin /replsummary
```
## 3. 第2段階：FSMO移行と ADDC01 のリプレース手順
### ① FSMO役割の転送
新ADDC02(2025)へ全ての役割を移します。

実行コマンド:
```title="PowerShell" linenums="0"
Move-ADDirectoryServerOperationMasterRole -Identity "ADDC02" -OperationMasterRole SchemaMaster, DomainNamingMaster, PDCEmulator, RIDMaster, InfrastructureMaster
```
### ② 旧 ADDC01 の降格と撤去
ADDC02の時と同様に降格・シャットダウン・メタデータ確認を実施。

### ③ 新 ADDC01 (2025) の構築と昇格
旧ADDC01と同一のホスト名・固定IPでセットアップ。

優先DNSを ADDC02のIP に向けて昇格実行。

## 4. 第3段階：FSMOの再転送
### ① FFSMOの再転送（ADDC02 → ADDC01）
```title="PowerShell" linenums="0"
Move-ADDirectoryServerOperationMasterRole -Identity "ADDC01" -OperationMasterRole SchemaMaster, DomainNamingMaster, PDCEmulator, RIDMaster, InfrastructureMaster
```
### ② 転送後の最終確認
正しくADDC01が5つの役割を保持したか確認します。
```title="DOSl" linenums="0"
netdom query fsmo
```
### ③ DNS参照の最適化
FSMOをADDC01に戻した後は、各DCの「自分自身を指すDNS設定」もベストプラクティスに合わせて最終調整してください。

- ADDC01のDNS設定: 優先 ADDC02のIP / 代替 127.0.0.1 (自分)
- ADDC02のDNS設定: 優先 ADDC01のIP / 代替 127.0.0.1 (自分)

※DC自身が起動する際、相手方のDNSが生きていることで、自身のADサービスがスムーズに開始されるための推奨設定です。

## 5. 最終確認・事後作業
### ① 最終健全性チェック
全DCで dcdiag を実行し、エラーがないことを確認。クライアント端末から新規ログオン、グループポリシーの適用（gpupdate /force）が成功することを確認。

### ② 機能レベルの引き上げ（任意）
全てのDCがWindows Server 2025に置き換わった後、2025の新機能（ADデータベースの32kページサイズ化等）を利用したい場合は、機能レベルを Windows Server 2025 へ引き上げます。

※一度上げると戻せないため慎重に

## 参考文献
- [Microsoft: AD DS の機能レベルについて](https://learn.microsoft.com/ja-jp/windows-server/identity/ad-ds/active-directory-functional-levels)
- [Windows Server 2025 AD DS 新機能の要件](https://learn.microsoft.com/ja-jp/windows-server/get-started/whats-new-windows-server-2025)

