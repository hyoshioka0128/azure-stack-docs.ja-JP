---
title: PowerShell で Azure Stack のバックアップを有効にする - MDC
description: PowerShell で Infrastructure Backup サービスを有効にし、障害が発生した場合に Azure Stack を復元できるようにする方法について学習します。 Modular Data Center (MDC) の場合。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: tzl
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/27/2020
ms.author: sethm
ms.reviewer: hectorl
ms.lastreviewed: 10/27/2020
ms.openlocfilehash: f3d28fa9f5610f91ab8b35ad017c836745f5117d
ms.sourcegitcommit: d719f148005e904fa426a001a687e80730c91fda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/06/2021
ms.locfileid: "97910876"
---
# <a name="configure-backup-for-azure-stack-with-powershell---modular-data-center-mdc"></a>PowerShell を使用して Azure Stack のバックアップを構成する - Modular Data Center (MDC)

*適用対象:Modular Data Center、Azure Stack Hub ラグド*

インフラストラクチャのバックアップを外部の保存場所にエクスポートするように、PowerShell を使用して Infrastructure Backup サービスを構成できます。 インフラストラクチャのバックアップは、サービスの低下を修正するためにサポートが使用できます。

## <a name="prepare-powershell-environment"></a>PowerShell 環境を準備する

PowerShell 環境の構成方法については、「[PowerShell for Azure Stack をインストールする](../../operator/powershell-install-az-module.md)」をご覧ください。 Azure Stack にサインインする場合は、「[オペレーター環境の構成と Azure Stack へのサインイン](../../operator/azure-stack-powershell-configure-admin.md)」を参照してください。

## <a name="provide-the-backup-share-credentials-and-encryption-key-to-enable-backup"></a>バックアップを有効にするためのバックアップ共有、認証情報、暗号化キーを提供する

同じ PowerShell セッションで、環境変数を追加して次の PowerShell スクリプトを編集します。 更新されたスクリプトを実行して、インフラストラクチャ バックアップ サービスにバックアップ共有、資格情報、および暗号化キーを提供します。

|変数  |説明  |
|---------|---------|
|$username     | ファイルを読み書きするための十分なアクセス権がある共有ドライブの場所のドメインとユーザー名を使用して **ユーザー名** を入力します。 たとえば、「Contoso\\backupshareuser」とします。        |
|$password     | ユーザーの **パスワード** を入力します。        |
|$path     | **バックアップ ストレージの場所** のパスを入力します。 別のデバイスでホストされるファイル共有へのパスの場合、汎用名前付け規則 (UNC) の文字列を使用する必要があります。 UNC 文字列は、共有ファイルやデバイスといった、リソースの場所を指定します。 バックアップ データを確実に利用できるようにするためには、保存デバイスは別の場所に配置する必要があります。        |
|$backupfrequencyinhours     | 頻度 (時間単位) はバックアップの作成頻度を決定します。 既定値は 12 です。 Scheduler では、最小値 4 から最大値 12 までをサポートします。        |
|$backupretentionperiodindays     | 保有期間 (日単位) は、バックアップが外部の保存場所に保持される日数を決定します。 既定値は 7 です。 Scheduler では、最小値 2 から最大値 14 までをサポートします。 保有期間より前のバックアップは、外部の保存場所から自動的に削除されます。        |
|$encryptioncertpath     | デプロイ時に提供された証明書。 外部の保存場所を構成する際に、新たに指定する必要はありません。 暗号化証明書パスは、.CER ファイルのファイル パスと、データ暗号化に使用される公開キーを指定します。        |
|$isbackupschedulerenabled     | 自動バックアップを有効または無効にします。 既定では、共有と資格情報を指定した後、自動バックアップが有効になります。        |

## <a name="configure-backup"></a>バックアップの構成

```powershell
# Example username
$username = "domain\backupadmin"

# Example share path
$sharepath = "\\serverIP\AzSBackupStore\contoso.com\seattle"

$password = Read-Host -Prompt ("Password for: " + $username) -AsSecureString

# Set the backup settings with the name, password, share, and CER certificate file.
Set-AzsBackupConfiguration -BackupShare path -Username $username -Password $password
```

## <a name="confirm-backup-settings"></a>バックアップ設定を確認する

同じ PowerShell セッションで、次のコマンドを実行します。

```powershell
Get-AzsBackupConfiguration | Select-Object -Property Path, UserName
```

結果は次の例が示す出力のようになります。

```shell
Path : \\serverIP\AzsBackupStore\contoso.com\seattle
UserName : domain\backupadmin
```

## <a name="update-backup-settings"></a>バックアップ設定の更新

同じ PowerShell セッションで、バックアップの保有期間と頻度の既定値を更新できます。

```powershell
# Set the backup frequency and retention period values.
$frequencyInHours = 10
$retentionPeriodInDays = 5

Set-AzsBackupConfiguration -BackupFrequencyInHours $backupfrequencyInHours -BackupRetentionPeriodInDays $backupretentionPeriodInDays

Get-AzsBackupConfiguration | Select-Object -Property Path, UserName, AvailableCapacity, BackupFrequencyInHours, BackupRetentionPeriodInDays
```

## <a name="update-encryption-certificate"></a>暗号化証明書を更新する

同じ PowerShell セッションで、バックアップ データの暗号化に使用する暗号化証明書を更新することができます。 新しい暗号化証明書が使用されるのは、新しいバックアップのみです。 既存のすべてのバックアップは、前の証明書で暗号化されたまま残ります。 古い証明書で暗号化された以前のバックアップを確実に消去するため、以前の証明書のコピーは、少なくとも 1 か月は保管しておくようにしてください。

```powershell
#Set the new encryption certificate by providing local path to CER file.
Set-AzsBackupConfiguration -EncryptionCertPath "c:\temp\cert.cer"
```

## <a name="enable-or-disable-automatic-backups"></a>自動バックアップの有効化または無効化

同じ PowerShell セッションで、自動バックアップを有効または無効にすることができます。 既定では、自動バックアップが有効になります。

```powershell
#Disable automatic backups.
Set-AzsBackupConfiguration - Isbackupschedulerenabled $false

#Enable automatic backups.
Set-AzsBackupConfiguration - Isbackupschedulerenabled $true
```

## <a name="next-steps"></a>次のステップ

バックアップが実行されたことを確認する方法については、[管理ポータルでのバックアップ完了の確認](../../operator/azure-stack-backup-back-up-azure-stack.md)に関するページを参照してください。
