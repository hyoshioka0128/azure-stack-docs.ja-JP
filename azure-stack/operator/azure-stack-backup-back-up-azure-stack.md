---
title: Azure Stack Hub のバックアップ
description: Azure Stack Hub でオンデマンド バックアップを行う方法について学習します。
author: PatAltimore
ms.topic: article
ms.date: 02/12/2019
ms.author: patricka
ms.reviewer: hectorl
ms.lastreviewed: 09/05/2019
ms.openlocfilehash: 5df894d32d335b488ad51b09bb38c3011f754e15
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97871839"
---
# <a name="back-up-azure-stack-hub"></a>Azure Stack Hub のバックアップ

この記事では、Azure Stack Hub でオンデマンド バックアップを行う方法について説明します。 PowerShell 環境の構成方法については、[PowerShell for Azure Stack Hub のインストール](powershell-install-az-module.md)に関するページをご覧ください。 Azure Stack Hub にサインインするには、「[Azure Stack Hub で管理者ポータルを使用する](azure-stack-manage-portals.md)」をご覧ください。

## <a name="start-azure-stack-hub-backup"></a>Azure Stack Hub のバックアップを開始する

### <a name="start-a-new-backup-without-job-progress-tracking"></a>ジョブの進行状況を追跡せずに新しいバックアップを開始する
Start-AzSBackup を使用して、ジョブの進行状況を追跡せずに新しいバックアップを迅速に開始します。

```powershell
   Start-AzsBackup -Force
```

### <a name="start-azure-stack-hub-backup-with-job-progress-tracking"></a>ジョブの進行状況を追跡して Azure Stack Hub のバックアップを開始する
Start-AzSBackup を使用して、 **-AsJob** パラメーターを指定して新しいバックアップを開始し、バックアップ ジョブの進捗状況を追跡する変数として保存します。

> [!NOTE]
> バックアップ ジョブは、ジョブが完了する約 10 から 15 分前に、正常に完了したことがポータルに表示されます。
>
> 実際の状態は、次のコードを使用するとより適切に監視できます。

> [!IMPORTANT]
> 最初に 1 ミリ秒の遅延を入れたのは、コードがジョブを正しく登録するには早すぎて、**PSBeginTime** なしで戻り、次にジョブの **State** なしで戻ることへの対処としてです。

```powershell
    $BackupJob = Start-AzsBackup -Force -AsJob
    While (!$BackupJob.PSBeginTime) {
        Start-Sleep -Milliseconds 1
    }
    Write-Host "Start time: $($BackupJob.PSBeginTime)"
    While ($BackupJob.State -eq "Running") {
        Write-Host "Job is currently: $($BackupJob.State) - Duration: $((New-TimeSpan -Start ($BackupJob.PSBeginTime) -End (Get-Date)).ToString().Split(".")[0])"
        Start-Sleep -Seconds 30
    }

    If ($BackupJob.State -eq "Completed") {
        Get-AzsBackup | Where-Object {$_.BackupId -eq $BackupJob.Output.BackupId}
        $Duration = $BackupJob.Output.TimeTakenToCreate
        $Pattern = '^P?T?((?<Years>\d+)Y)?((?<Months>\d+)M)?((?<Weeks>\d+)W)?((?<Days>\d+)D)?(T((?<Hours>\d+)H)?((?<Minutes>\d+)M)?((?<Seconds>\d*(\.)?\d*)S)?)$'
        If ($Duration -match $Pattern) {
            If (!$Matches.ContainsKey("Hours")) {
                $Hours = ""
            } 
            Else {
                $Hours = ($Matches.Hours).ToString + 'h '
            }
            $Minutes = ($Matches.Minutes)
            $Seconds = [math]::round(($Matches.Seconds))
            $Runtime = '{0}{1:00}m {2:00}s' -f $Hours, $Minutes, $Seconds
        }
        Write-Host "BackupJob: $($BackupJob.Output.BackupId) - Completed with Status: $($BackupJob.Output.Status) - It took: $($Runtime) to run" -ForegroundColor Green
    }
    ElseIf ($BackupJob.State -ne "Completed") {
        $BackupJob
        $BackupJob.Output
    }
```

## <a name="confirm-backup-has-completed"></a>バックアップの完了を確認する

### <a name="confirm-backup-has-completed-using-powershell"></a>PowerShell を使用してバックアップが完了したことを確認する
次の PowerShell コマンドを使用して、バックアップが正常に完了したことを確実にします。

```powershell
   Get-AzsBackup
```

結果は次の出力のようになります。

```powershell
    BackupDataVersion : 1.0.1
    BackupId          : <backup ID>
    RoleStatus        : {NRP, SRP, CRP, KeyVaultInternalControlPlane...}
    Status            : Succeeded
    CreatedDateTime   : 7/6/2018 6:46:24 AM
    TimeTakenToCreate : PT20M32.364138S
    DeploymentID      : <deployment ID>
    StampVersion      : 1.1807.0.41
    OemVersion        : 
    Id                : /subscriptions/<subscription ID>/resourceGroups/System.local/providers/Microsoft.Backup.Admin/backupLocations/local/backups/<backup ID>
    Name              : local/<local name>
    Type              : Microsoft.Backup.Admin/backupLocations/backups
    Location          : local
    Tags              : {}
```

### <a name="confirm-backup-has-completed-in-the-administrator-portal"></a>管理者ポータルでバックアップの完了を確認する
Azure Stack Hub 管理者ポータルを使用すると、バックアップが正常に完了したことを以下の手順で確認できます。

1. [Azure Stack Hub 管理者ポータル](azure-stack-manage-portals.md)を開きます。
2. **[すべてのサービス]** を選択し、 **[管理]** カテゴリで **[Infrastructure backup]** を選択します。 **[Infrastructure backup]\(インフラストラクチャ バックアップ\)** ブレードで **[構成]** を選択します。
3. **[利用可能なバックアップ]** リストから、バックアップの **[名前]** と **[完了日]** を見つけます。
4. **[状態]** が **[成功]** であることを確認します。

## <a name="next-steps"></a>次のステップ

[データ損失イベントから復旧する](azure-stack-backup-recover-data.md)ためのワークフローについて学びます。
