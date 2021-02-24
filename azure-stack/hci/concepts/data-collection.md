---
title: Azure Stack HCI のデータ コレクション
description: このトピックでは、Azure Stack HCI によって収集される診断データに関連する設計とポリシーについて説明します。
author: khdownie
ms.author: v-kedow
ms.topic: conceptual
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 12/10/2020
ms.openlocfilehash: 91f578bcb0dc3a9d2debd086e6ed95cb591de90d
ms.sourcegitcommit: afdae61022037b5dba8345cb264049897e0aca8f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/10/2020
ms.locfileid: "97053138"
---
# <a name="azure-stack-hci-data-collection"></a>Azure Stack HCI のデータ コレクション

> 適用対象:Azure Stack HCI バージョン 20H2

このトピックでは、Azure Stack HCI をセキュリティで保護された最新の状態に保ち、2020 年 12 月の一般提供 (GA) で期待どおりに動作するために収集される、必要なデータについて説明します。

以下に説明するデータは、Microsoft から Azure Stack HCI を提供するために必要です。 このデータは 1 日に 1 回収集され、データ コレクション イベントはイベント ログで確認できます。 Azure Stack HCI により、クラスターを最新の状態に保ち、セキュリティで保護し、適切に動作させるために必要な最小限のデータが収集されます。

   > [!IMPORTANT]
   > 次に示す Azure Stack HCI によって収集されるデータは、さまざまなレベルの収集用に構成できる Windows 診断データとは独立しています。 Azure Stack HCI では、Windows 診断データ コレクションの既定の設定は [Security]\(セキュリティ\) (オフ) です。これは、管理者が診断データ設定を変更しない限り、Windows 診断データが送信されないことを意味します。 詳しくは、「[組織内の Windows 診断データの構成](/windows/privacy/configure-windows-diagnostic-data-in-your-organization)」をご覧ください。 Microsoft は、Azure Stack HCI に関連して収集されるあらゆる Windows 診断データの独立したコントローラーです。 Microsoft は、[Microsoft のプライバシーに関する声明](https://privacy.microsoft.com/privacystatement)に従って Windows 診断データを処理します。

## <a name="data-collection-and-residency"></a>データ コレクションと保存場所

この Azure Stack HCI データは: 

- 製品が Azure に登録されるまで、Microsoft に送信されません。 Azure Stack HCI が登録解除されると、このデータ コレクションは停止します。
- Microsoft-AzureStack-HCI/Analytic イベント チャネルに記録されます。 
- JSON 形式であるため、システム管理者は送信されたデータを調べて分析できます。
- 米国内のセキュリティで保護された Microsoft が運営するデータセンターに保存されています。

## <a name="data-retention"></a>データの保持

Azure Stack HCI によってこのデータが収集されると、90 日間保持されます。 集計された匿名化されたデータは、より長く保持される場合があります。

## <a name="what-data-is-collected"></a>どのデータが収集されますか。

Azure Stack HCI によって収集されるもの:

- オペレーティング システムのバージョン、プロセッサ モデル、プロセッサ コアの数、メモリ サイズ、クラスター ID、ハードウェア ID のハッシュなどのサーバーに関する情報
- インストールされている Azure Stack HCI サーバー機能の一覧 (例: BitLocker)
- Azure Stack HCI オペレーティング システムの信頼性を計算するために必要な情報
- 正常性コレクション データの信頼性を計算するために必要な情報
- 更新プログラムのダウンロードに失敗したなど、特定のエラーについてイベント ログから収集された情報
- ストレージの信頼性を計算するための情報
- 物理ディスクの信頼性を計算するための情報
- ボリューム暗号化の信頼性を計算するための情報
- 記憶域スペースの修復の信頼性とパフォーマンスを計算するための情報
- Azure Stack HCI オペレーティング システムのセキュリティを検証するための情報
- Azure Stack HCI オペレーティング システムのウイルス対策またはマルウェア対策の状態の信頼性を計算するための情報
- ネットワーク コンポーネントの信頼性を関連付けるための情報
- ネットワーク パフォーマンスを関連付けるための情報
- 更新プログラムとインストールの信頼性を関連付けるための情報
- Hyper-V の信頼性を測定するための情報
- クラスタリング コンポーネントの信頼性を測定する、または関連付けるための情報
- クラスター対応更新 (CAU) 機能の成功を追跡するための情報
- ディザスター リカバリー機能の信頼性を測定する、または関連付けるための情報
- Azure Stack HCI サーバーに適用される SMB 帯域幅の制限を記述するための情報

## <a name="view-this-data"></a>このデータを表示する

1. 次の PowerShell コマンドを使用して、分析ログを有効にします。

   ```PowerShell
   wevtutil sl Microsoft-AzureStack-HCI/Analytic /e:True
   ```

2. ログを表示して、収集されたデータを確認します。

   ```PowerShell
   Get-WinEvent -LogName Microsoft-AzureStack-HCI/Analytic -Oldest
   ```

3. エクスポートするデータの書式を整えます。

   ```PowerShell
   Get-WinEvent -LogName Microsoft-AzureStack-HCI/Analytic -Oldest `
   | Where-Object Id -eq 802 `
   | ForEach-Object { 
       [pscustomobject] @{
           TimeCreated = $_.TimeCreated 
           EventName=$_.Properties[0].Value 
           Value=$_.Properties[1].Value 
       } 
   }
   ```
 
出力は次のようになります。

```shell
TimeCreated            EventName                                                  Value
-----------            ---------                                                  -----
11/16/2020 10:36:28 AM Microsoft.AzureStack.HCI.Diagnostic.Core                   {"OEMName":"Microsoft Corporation"...
11/16/2020 10:36:28 AM Microsoft.AzureStack.HCI.Diagnostic.ProductFeatures        {"InstalledFeatures":["Server-Core...
11/16/2020 10:36:28 AM Microsoft.AzureStack.HCI.Diagnostic.OSReliability          {"DailyDirtyRestarts":0,"WeeklyDir...
11/16/2020 10:36:28 AM Microsoft.AzureStack.HCI.Diagnostic.DiagnosticHealth       {"DailySuccessfulDiagnosticUploads...
11/16/2020 10:36:28 AM Microsoft.AzureStack.HCI.Diagnostic.ErrorSummary           {"ErrorSummary":[{"EventName":"Win...
11/16/2020 10:36:29 AM Microsoft.AzureStack.HCI.Diagnostic.VolumeSummary          {"VolumeCount":2,"HealthyVolumeCou...
11/16/2020 10:36:29 AM Microsoft.AzureStack.HCI.Diagnostic.DiskSummary            {"DiskCount":33,"Summary":[]}
11/16/2020 10:36:29 AM Microsoft.AzureStack.HCI.Diagnostic.BitlockerVolumeSummary {"BitlockerVolumeCount":0,"Summary...
11/16/2020 10:36:29 AM Microsoft.AzureStack.HCI.Diagnostic.StorageErrors          {"ErrorSummary":[{"EventName":"Sto...
11/16/2020 10:36:29 AM Microsoft.AzureStack.HCI.Diagnostic.StorageRepairSummary   {"DailyRepairStartCount":0,"Weekly...
11/16/2020 10:36:29 AM Microsoft.AzureStack.HCI.Diagnostic.TrustedPlatformModule  {"Manufacturer":"MSFT","Manufactur...
11/16/2020 10:36:29 AM Microsoft.AzureStack.HCI.Diagnostic.MicrosoftDefender      {"AMEngineVersion":"1.1.17600.5","...
11/16/2020 10:36:30 AM Microsoft.AzureStack.HCI.Diagnostic.NetworkInfo            {"NetworkDirect":true,"NetworkDire...
11/16/2020 10:36:30 AM Microsoft.AzureStack.HCI.Diagnostic.NetworkAdapterSummary  {"NetworkAdapterGroup":[{"DriverNa...
11/16/2020 10:36:30 AM Microsoft.AzureStack.HCI.Diagnostic.OSDeploy               {"OSInstallType":0}
11/16/2020 10:36:30 AM Microsoft.AzureStack.HCI.Diagnostic.ClusterProperties      {"Id":"fd2fc061-b924-4d61-a45b-3b3...
11/16/2020 10:36:30 AM Microsoft.AzureStack.HCI.Diagnostic.DisasterRecovery       {"IsDisasterRecoveryEnabled":false...
```
