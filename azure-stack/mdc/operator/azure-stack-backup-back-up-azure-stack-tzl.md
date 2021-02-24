---
title: Azure Stack のバックアップ - MDC | Microsoft Docs
description: Modular Data Center (MDC) の Azure Stack でオンデマンド バックアップを行う方法について学習します。
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
ms.date: 10/26/2020
ms.author: sethm
ms.reviewer: hectorl
ms.lastreviewed: 10/26/2020
ms.openlocfilehash: f64b656b6a0a0d5310b6d6e2fbb8ff26d5206ad1
ms.sourcegitcommit: d719f148005e904fa426a001a687e80730c91fda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/06/2021
ms.locfileid: "97910893"
---
# <a name="back-up-azure-stack---modular-data-center-mdc"></a>Azure Stack のバックアップ - Modular Data Center (MDC)

*適用対象:Modular Data Center、Azure Stack Hub ラグド*

この記事では、Azure Stack のオンデマンド バックアップを行う方法について説明します。

## <a name="start-backup"></a>バックアップの開始

バックアップは自動的にスケジュールされます。 オンデマンド バックアップは、バックアップを手動で行うようトリガーするアラートを受け取った場合にのみ開始する必要があります。 オンデマンド バックアップを開始するには、 **[今すぐバックアップ]** を選択します。 オンデマンド バックアップでは、次回のスケジュールされたバックアップの時間は変更されません。 タスクの完了後、 **[要点] ブレード** で設定をご確認いただけます。

![バックアップの設定](media/azure-stack-backup-back-up-azure-stack-tzl/on-demand-backup.png)

## <a name="start-azure-stack-backup"></a>Azure Stack のバックアップを開始する

また、Azure Stack 管理コンピューターで PowerShell コマンドレット **Start-AzsBackup** を実行することもできます。

### <a name="start-a-new-backup-without-job-progress-tracking"></a>ジョブの進行状況を追跡せずに新しいバックアップを開始する

**Start-AzSBackup** を使用して、ジョブの進行状況を追跡せずに新しいバックアップを迅速に開始します。

```powershell
Start-AzsBackup -Force
```

## <a name="start-azure-stack-backup-with-job-progress-tracking"></a>ジョブの進行状況を追跡して Azure Stack のバックアップを開始する

**Start-AzSBackup** を使用して、 **-AsJob** パラメーターを指定して新しいバックアップを開始し、バックアップ ジョブの進捗状況を追跡する変数として保存します。

> [!NOTE]
> バックアップ ジョブは、ジョブが完了する約 10 から 15 分前に、正常に完了したことがポータルに表示されます。
>
> 実際の状態は、次のコードを使用するとより適切に監視できます。

最初に 1 ミリ秒の遅延を入れたのは、コードがジョブを正しく登録するには早すぎるためです。 コードにより、**PSBeginTime** なしが報告され、次にジョブの **State** なしが報告されます。

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
    $Pattern = '^P?T?((?<Years>\d+)Y)?((?<Mohs>\d+)M)?((?<Weeks>\d+)W)?((?<Days>\d+)D)?(T((?<Hours>\d+)H)?((?<Minutes>\d+)M)?((?<Seconds>\d*(\.)?\d*)S)?)$'
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

```shell
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

Azure Stack 管理者ポータルを使用して、以下の手順でバックアップが正常に完了したことを確認します。

1. [Azure Stack 管理者ポータル](../../operator/azure-stack-manage-portals.md)を開きます。

2. **[すべてのサービス]** を選択し、 **[管理]** カテゴリで **[インフラストラクチャ バックアップ]** を選択します。 **[Infrastructure backup]\(インフラストラクチャ バックアップ\)** ブレードで **[構成]** を選択します。

3. **[利用可能なバックアップ]** リストから、バックアップの **[名前]** と **[完了日]** を見つけます。

4. **[状態]** が **[成功]** であることを確認します。

## <a name="next-steps"></a>次のステップ

[インフラストラクチャのバックアップのベスト プラクティス](azure-stack-backup-best-practices-tzl.md)について確認します。
