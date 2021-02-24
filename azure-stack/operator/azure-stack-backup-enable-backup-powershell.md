---
title: PowerShell で Azure Stack Hub のバックアップを有効にする
description: PowerShell で Infrastructure Backup サービスを有効にし、障害が発生した場合に Azure Stack Hub を復元できるようにする方法について学習します。
author: PatAltimore
ms.topic: article
ms.date: 04/25/2019
ms.author: patricka
ms.reviewer: hectorl
ms.lastreviewed: 03/14/2019
ms.openlocfilehash: 0e1bdbf55556d8168e517a5d9cfb30d6747c02f3
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97871754"
---
# <a name="enable-backup-for-azure-stack-hub-with-powershell"></a>PowerShell で Azure Stack Hub のバックアップを有効にする

Windows PowerShell で Infrastructure Backup サービスを有効にして、次のものを定期的にバックアップします。
 - 内部 ID サービスとルート証明書
 - ユーザー プラン、オファー、サブスクリプション
 - コンピューティング、ストレージ、ネットワーク ユーザー クォータ
 - ユーザー Key Vault シークレット
 - ユーザー RBAC ロールとポリシー
 - ユーザー ストレージ アカウント

PowerShell コマンドレットにアクセスして、オペレーター管理エンドポイント経由でバックアップを有効にし、バックアップを開始し、バックアップ情報を取得できます。

## <a name="prepare-powershell-environment"></a>PowerShell 環境を準備する

PowerShell 環境の構成方法については、[PowerShell for Azure Stack Hub のインストール](powershell-install-az-module.md)に関するページをご覧ください。 Azure Stack Hub にサインインする場合は、[オペレーター環境の構成と Azure Stack Hub へのサインイン](azure-stack-powershell-configure-admin.md)に関するページを参照してください。

## <a name="provide-the-backup-share-credentials-and-encryption-key-to-enable-backup"></a>バックアップを有効にするためのバックアップ共有、認証情報、暗号化キーを提供する

同じ PowerShell セッションで、環境変数を追加して次の PowerShell スクリプトを編集します。 更新されたスクリプトを実行して、インフラストラクチャ バックアップ サービスにバックアップ共有、資格情報、および暗号化キーを提供します。

| 変数        | 説明   |
|---              |---                                        |
| `$username`       | ファイルを読み書きするための十分なアクセス権がある共有ドライブの場所のドメインとユーザー名を使用して **ユーザー名** を入力します。 たとえば、「 `Contoso\backupshareuser` 」のように入力します。 |
| `$password`       | ユーザーの **パスワード** を入力します。 |
| `$sharepath`      | **バックアップ ストレージの場所** のパスを入力します。 別のデバイスでホストされるファイル共有へのパスの場合、汎用名前付け規則 (UNC) の文字列を使用する必要があります。 UNC 文字列は、共有ファイルやデバイスといった、リソースの場所を指定します。 バックアップ データを確実に利用できるようにするためには、保存デバイスは別の場所に配置する必要があります。 |
| `$frequencyInHours` | 頻度 (時間単位) はバックアップの作成頻度を決定します。 既定値は 12 です。 Scheduler では、最大値 12 から最小値 4 までをサポートします。|
| `$retentionPeriodInDays` | 保有期間 (日単位) は、バックアップが外部の保存場所に保持される日数を決定します。 既定値は 7 です。 Scheduler では、最大値 14 から最小値 2 までをサポートします。 保有期間より前のバックアップは、外部の保存場所から自動的に削除されます。|
| `$encryptioncertpath` | 1901 以降に適用されます。 パラメーターは、Azure Stack Hub Module バージョン 1.7 以降で利用できます。 暗号化証明書パスは、.CER ファイルのファイル パスと、データ暗号化に使用される公開キーを指定します。 |
| `$encryptionkey` | ビルド 1811 以前に適用されます。 パラメーターは、Azure Stack Hub Module バージョン 1.6 以前で利用できます。 暗号化キーはデータの暗号化に使用されます。 [New-AzsEncryptionKeyBase64](/powershell/module/azs.backup.admin/new-azsencryptionkeybase64) コマンドレットを使用して、新しいキーを生成します。 |
|     |     |

### <a name="enable-backup-on-1901-and-later-using-certificate"></a>証明書を使用して 1901 以降に対してバックアップを有効にする
```powershell
    # Example username:
    $username = "domain\backupadmin"
 
    # Example share path:
    $sharepath = "\\serverIP\AzSBackupStore\contoso.com\seattle"

    $password = Read-Host -Prompt ("Password for: " + $username) -AsSecureString

    # Create a self-signed certificate using New-SelfSignedCertificate, export the public key portion and save it locally.

    $cert = New-SelfSignedCertificate `
        -DnsName "www.contoso.com" `
        -CertStoreLocation "cert:\LocalMachine\My" 

    New-Item -Path "C:\" -Name "Certs" -ItemType "Directory" 

    #make sure to export the PFX format of the certificate with the public and private keys and then delete the certificate from the local certificate store of the machine where you created the certificate
    
    Export-Certificate `
        -Cert $cert `
        -FilePath c:\certs\AzSIBCCert.cer 

    # Set the backup settings with the name, password, share, and CER certificate file.
    Set-AzsBackupConfiguration -BackupShare $sharepath -Username $username -Password $password -EncryptionCertPath "c:\temp\cert.cer"
```
### <a name="enable-backup-on-1811-or-earlier-using-certificate"></a>証明書を使用して 1811 以前に対してバックアップを有効にする
```powershell
    # Example username:
    $username = "domain\backupadmin"
 
    # Example share path:
    $sharepath = "\\serverIP\AzSBackupStore\contoso.com\seattle"

    $password = Read-Host -Prompt ("Password for: " + $username) -AsSecureString

    # Create a self-signed certificate using New-SelfSignedCertificate, export the public key portion and save it locally.

    $key = New-AzsEncryptionKeyBase64
    $Securekey = ConvertTo-SecureString -String ($key) -AsPlainText -Force

    # Set the backup settings with the name, password, share, and CER certificate file.
    Set-AzsBackupConfiguration -BackupShare $sharepath -Username $username -Password $password -EncryptionKey $Securekey
```

   
##  <a name="confirm-backup-settings"></a>バックアップ設定を確認する

同じ PowerShell セッションで、次のコマンドを実行します。

   ```powershell
    Get-AzsBackupConfiguration | Select-Object -Property Path, UserName
   ```

結果は次の例が示す出力のようになります。

   ```powershell
    Path                        : \\serverIP\AzsBackupStore\contoso.com\seattle
    UserName                    : domain\backupadmin
   ```

## <a name="update-backup-settings"></a>バックアップ設定の更新
同じ PowerShell セッションで、バックアップの保有期間と頻度の既定値を更新できます。 

   ```powershell
    #Set the backup frequency and retention period values.
    $frequencyInHours = 10
    $retentionPeriodInDays = 5

    Set-AzsBackupConfiguration -BackupFrequencyInHours $frequencyInHours -BackupRetentionPeriodInDays $retentionPeriodInDays

    Get-AzsBackupConfiguration | Select-Object -Property Path, UserName, AvailableCapacity, BackupFrequencyInHours, BackupRetentionPeriodInDays
   ```

結果は次の例が示す出力のようになります。

   ```powershell
    Path                        : \\serverIP\AzsBackupStore\contoso.com\seattle
    UserName                    : domain\backupadmin
    AvailableCapacity           : 60 GB
    BackupFrequencyInHours      : 10
    BackupRetentionPeriodInDays : 5
   ```

### <a name="azure-stack-hub-powershell"></a>Azure Stack Hub PowerShell 
インフラストラクチャ バックアップを構成するための PowerShell コマンドレットは Set-AzsBackupConfiguration です。 以前のリリースでは、コマンドレットは Set-AzsBackupShare でした。 このコマンドレットは証明書の提供を要求します。 暗号化キーを使用してインフラストラクチャ バックアップが構成されている場合、暗号化キーの更新またはプロパティの表示は行えません。 Admin PowerShell のバージョン 1.6 を使用する必要があります。

1901 に更新する前にインフラストラクチャ バックアップが構成された場合、管理者の PowerShell のバージョン 1.6 を使用して暗号化キーを設定および表示できます。 バージョン 1.6 では、暗号化キーから証明書ファイルへの更新ができなくなります。
モジュールの正しいバージョンのインストールについて詳しくは、[Azure Stack Hub PowerShell のインストール](powershell-install-az-module.md)に関するページを参照してください。


## <a name="next-steps"></a>次のステップ

[Azure Stack Hub のバックアップ](azure-stack-backup-back-up-azure-stack.md)方法に関するページでバックアップの実行方法を学びます。

「[Confirm backup completed in administration portal](azure-stack-backup-back-up-azure-stack.md)」(管理ポータルでのバックアップ完了の確認) でバックアップが実行されたことを確認する方法について学びます。
