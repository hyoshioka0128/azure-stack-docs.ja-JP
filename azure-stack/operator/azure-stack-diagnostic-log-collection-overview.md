---
title: Azure Stack Hub での診断ログの収集
description: Azure Stack Hub の [ヘルプとサポート] における診断ログ収集について説明します。
author: myoungerman
ms.topic: article
ms.date: 08/24/2020
ms.author: v-myoung
ms.reviewer: shisab
ms.lastreviewed: 08/24/2020
ms.openlocfilehash: 88f2b267f493c05fdcd8a5419718d08d5b6efe37
ms.sourcegitcommit: 868887e4b13b1572f15004a9db2c334e60d8add2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/06/2020
ms.locfileid: "91778247"
---
# <a name="diagnostic-log-collection-in-azure-stack-hub"></a>Azure Stack Hub での診断ログの収集

Azure Stack Hub は、相互にやり取りする Windows コンポーネントとオンプレミスの Azure サービスからなるコレクションです。 これらのすべてのコンポーネントとサービスは、独自のログ セットを生成します。 Microsoft サポートではこれらのログを使用して問題が効率的に特定され修正されるため、Microsoft では診断ログの収集機能を提供しています。 診断ログ収集によって、お客様は診断ログを迅速に収集し、Microsoft サポートと共有できます。これは PowerShell を必要としない簡単なユーザー インターフェイスです。 他のインフラストラクチャ サービスが停止している場合でも、ログは収集されます。  

このログ収集方法を使用し、管理者ポータルまたは [ヘルプとサポート] ブレードが使用できない場合のみ[特権エンドポイント (PEP) を使用](azure-stack-get-azurestacklog.md)することをお勧めします。 

>[!NOTE]
>診断ログ収集を使用するには、Azure Stack Hub が登録済みである必要があります。 Azure Stack Hub が登録済みでない場合は、[特権エンドポイント (PEP)](azure-stack-get-azurestacklog.md) を使用してログを共有します。 

## <a name="collection-options-and-data-handling"></a>収集オプションとデータの処理

::: moniker range=">= azs-2005"

Azure Stack Hub には、Azure との接続に応じて、診断ログを収集、保存したり、CSS に送信したりするための適切な方法が用意されています。 Azure Stack Hub が Azure に接続できる場合、**事前ログ収集**を有効にする方法が推奨されます。事前ログ収集では、重要なアラートが発生したときに、Microsoft が管理する Azure のストレージ BLOB に診断ログが自動的にアップロードされます。 または、 **[Send logs now]\(今すぐログを送信する\)** を使用してオンデマンドでログを収集することもできます。Azure Stack Hub が Azure から切断されている場合には、ログをローカルに保存することも可能です。 

次のセクションでは、各オプションと、各ケースでのデータの処理方法について説明します。 

::: moniker-end

診断ログ収集機能には、ログを送信するための 2 つの方法が用意されています。
* ログを事前に送信する
* 今すぐログを送信する

また、ログをローカルに保存することもできます。

これは、それぞれの場合に診断ログを送信するために使用するオプションを示しています。 

![Microsoft にログを今すぐ送信する方法を示すフローチャート](media/azure-stack-help-and-support/send-logs-now-flowchart.png)

次のセクションでは、各オプションと、各ケースでのデータの処理方法について説明します。 

### <a name="send-logs-proactively"></a>ログを事前に送信する

事前ログ収集では、ユーザーがサポート ケースを開く前に、診断ログが自動的に収集され、Azure Stack Hub から Microsoft に送信されます。 これらのログは、[システム正常性アラート](#proactive-diagnostic-log-collection-alerts)が発生した場合にのみ収集され、サポート ケースのコンテキストで Microsoft サポートによってのみアクセスされます。

事前ログ収集は、いつでも無効にして再度有効にすることができます。 事前ログ収集をセットアップするには、次の手順を実行します。

1. Azure Stack Hub 管理者ポータルにサインインします。
1. **[Help and support Overview]\(ヘルプとサポートの概要\)** を開きます。
1. バナーが表示されたら、 **[Enable proactive log collection]\(事前ログ収集を有効にする\)** を選択します。 または、 **[設定]** を選択して **[Proactive log collection]\(事前ログ収集\)** を **[有効]** にし、 **[保存]** を選択することもできます。

>[!NOTE]
>ログの場所の設定がローカル ファイル共有に対して構成されている場合、共有ストレージがそのサイズ クォータに達するのを、ライフサイクル管理ポリシーが未然に防ぎます。 Azure Stack Hub は、ローカル ファイル共有を監視せず、アイテム保持ポリシーを強制することもしません。   

### <a name="send-logs-now"></a>今すぐログを送信する

> [!TIP]
> 今すぐログを送信するのではなく、[事前ログ収集](#send-logs-proactively)を使用して時間を節約できます。

"今すぐログを送信" オプションでは、通常、ユーザーがサポート ケースを開く前に、Azure Stack Hub から手動で診断ログを収集してアップロードします。

お客様は、管理者ポータルまたは PowerShell を使用して、Microsoft サポートに診断ログを手動で送信できます。 Azure Stack Hub が Azure に接続されている場合は、管理者ポータルの使用をお勧めします。これは、この方法がログを Microsoft に直接送信する最も簡単な方法であるためです。 ポータルが使用できない場合、ユーザーは代わりに PowerShell を使用してログを送信する必要があります。

今すぐログを送信するには:

1. **[ヘルプとサポート] > [ログの収集] > [Send logs now]\(今すぐログを送信する\)** を開きます。 
1. ログ収集の開始時刻と終了時刻を指定します。 
1. ローカル タイム ゾーンを選択します。
1. **[Collect and Upload]\(収集してアップロード\)** を選択します。

インターネットに接続していない場合、またはローカルでのみログを保存する場合は、[Get-AzureStackLog](azure-stack-get-azurestacklog.md) メソッドを使用してログを送信します。 

::: moniker range=">= azs-2005"

### <a name="save-logs-locally"></a>ログをローカルに保存する

Azure Stack Hub が Azure から切断されている場合、ログをローカルのサーバー メッセージ ブロック (SMB) 共有に保存できます。 **[設定]** ブレードでパスを入力し、共有への書き込みアクセス許可があるユーザー名とパスワードを入力します。 サポート ケース時に、それらのローカル ログを転送する方法についての詳細な手順が Microsoft サポートから提供されます。 管理者ポータルが使用できない場合は [Get-AzureStackLog](azure-stack-get-azurestacklog.md) を使用してローカルにログを保存できます。

![診断ログ収集オプションのスクリーンショット](media/azure-stack-help-and-support/save-logs-locally.png)

::: moniker-end

## <a name="bandwidth-considerations"></a>帯域幅に関する考慮事項

診断ログの収集の平均サイズは、事前実行するか手動で実行するかによって異なります。 **事前ログ収集** の平均サイズは約 2 GB です。 **[Send logs now]\(今すぐログを送信する\)** の収集サイズは、収集される時間の長さによって異なります。

次の表に、Azure への制限付きまたは従量制課金接続の環境の場合の考慮事項を示します。

| ネットワーク接続 | 影響 |
|----|---|
| 低帯域幅/待機時間が長い接続 | ログのアップロードは、完了までに長時間かかります。 |
| 共有接続 | アップロードは、ネットワーク接続を共有する他のアプリやユーザーに影響を与える場合があります。 |
| 従量制課金接続 | 追加のネットワーク使用に対して、ISP からの追加料金が発生する場合があります。 |

## <a name="parameter-considerations"></a>パラメーターに関する考慮事項 

* **FromDate** パラメーターと **ToDate** パラメーターを使用して、特定の期間のログを収集できます。 これらのパラメーターが指定されない場合、既定では過去 4 時間のログが収集されます。

* コンピューター名でログをフィルター処理するには、**FilterByNode** パラメーターを使用します。 次に例を示します。

  ```powershell
  Send-AzureStackDiagnosticLog -FilterByNode azs-xrp01
  ```

* 種類でログをフィルター処理するには、**FilterByLogType** パラメーターを使用します。 File、Share、または WindowsEvent を選択してフィルター処理できます。 次に例を示します。

  ```powershell
  Send-AzureStackDiagnosticLog -FilterByLogType File
  ```

* **FilterByResourceProvider** パラメーターを使用して、付加価値リソース プロバイダー (RP) の診断ログを送信します。 一般的な構文は次のとおりです。
 
  ```powershell
  Send-AzureStackDiagnosticLog -FilterByResourceProvider <<value-add RP name>>
  ```
 
  IoT Hub の診断ログを送信するには: 

  ```powershell
  Send-AzureStackDiagnosticLog -FilterByResourceProvider IotHub
  ```
 
  Event Hub の診断ログを送信するには:

  ```powershell
  Send-AzureStackDiagnosticLog -FilterByResourceProvider eventhub
  ```
 
  Azure Stack Edge の診断ログを送信するには:

  ```powershell
  Send-AzureStackDiagnosticLog -FilterByResourceProvide databoxedge
  ```

* **FilterByRole** パラメーターを使用して、VirtualMachines ロールと BareMetal ロールから診断ログを送信します。

  ```powershell
  Send-AzureStackDiagnosticLog -FilterByRole VirtualMachines,BareMetal
  ```

* ログ ファイルに対する過去 8 時間以内の日付範囲のフィルター処理で、VirtualMachines ロールと BareMetal ロールから診断ログを送信するには:

  ```powershell
  Send-AzureStackDiagnosticLog -FilterByRole VirtualMachines,BareMetal -FromDate (Get-Date).AddHours(-8)
  ```

* ログ ファイルに対する過去 8 時間から 2 時間以内の日付範囲のフィルター処理で、VirtualMachines ロールと BareMetal ロールから診断ログを送信するには:

  ```powershell
  Send-AzureStackDiagnosticLog -FilterByRole VirtualMachines,BareMetal -FromDate (Get-Date).AddHours(-8) -ToDate (Get-Date).AddHours(-2)
  ```

Azure Stack Hub でアラートが発生したときに診断ログを事前に収集することで、カスタマー サポートに要する時間を節約します。

システム正常性状態を調査する必要がある場合、サポート ケースを開く前に Microsoft サポートにより分析のためにログを自動的にアップロードできます。

>[!NOTE]
>インターネットに接続していない場合、またはローカルでのみログを保存する場合は、[Get-AzureStackLog](azure-stack-get-azurestacklog.md) メソッドを使用してログを送信します。 

## <a name="view-log-collection"></a>ログ収集の表示

Azure Stack Hub から収集されたログの履歴は、 **[ヘルプとサポート]** の **[Log collection]\(ログ収集\)** ページに次の日付と時刻と共に表示されます。

- **収集された時刻**:ログ収集操作が開始されたとき。
- **状態**: 進行中または完了。
- **ログの開始**:収集する期間の開始。
- **ログの終了**:期間の終了。
- **[種類]** :手動ログ収集または事前ログ収集のいずれか。

![ヘルプとサポートのログ収集](media/azure-stack-help-and-support/azure-stack-log-collection.png)

## <a name="proactive-diagnostic-log-collection-alerts"></a>事前診断ログ収集アラート

有効にすると、次のいずれかのイベントが発生した場合にのみ、事前ログ収集によってログがアップロードされます。

たとえば、**更新失敗**は、事前診断ログ収集をトリガーするアラートです。 これが有効になっていると、Microsoft サポートによる問題のトラブルシューティングに役立つように、更新エラーの間に前もって診断ログがキャプチャされます。 診断ログは、**更新失敗**のアラートが発生した場合にだけ収集されます。

| アラートのタイトル | FaultIdType |
|---|---|
|リモート サービスに接続できない | UsageBridge.NetworkError|
|更新失敗 | Urp.UpdateFailure |
|ストレージ リソース プロバイダーのインフラストラクチャまたは依存関係を利用できない |    StorageResourceProviderDependencyUnavailable |
|ノードがコントローラーに接続されていない| ServerHostNotConnectedToController |  
|ルート パブリケーションの失敗 | SlbMuxRoutePublicationFailure |
|ストレージ リソースプロバイダーの内部データ ストアを利用できない |    StorageResourceProvider. DataStoreConnectionFail |
|ストレージ デバイスの障害 | Microsoft.Health.FaultType.VirtualDisks.Detached |
|正常性コントローラーが外部ストレージ アカウントにアクセスできない | Microsoft.Health.FaultType.StorageError |
|物理ディスクへの接続が失われた | Microsoft.Health.FaultType.PhysicalDisk.LostCommunication |
|ノードで Blob service が実行されていない | StorageService.The.blob.service.is.not.running.on.a.node-Critical |
|インフラストラクチャ ロールの異常 | Microsoft.Health.FaultType.GenericExceptionFault |
|Table service のエラー | StorageService.Table.service.errors-Critical |
|ファイル共有の使用率が 80% を超えている | Microsoft.Health.FaultType.FileShare.Capacity.Warning.Infra |
|Scale unit node is offline\(スケール ユニットがオフラインです\) | FRP.Heartbeat.PhysicalNode |
|インフラストラクチャ ロール インスタンスを利用できない | FRP.Heartbeat.InfraVM |
|インフラストラクチャ ロール インスタンスを利用できない  | FRP.Heartbeat.NonHaVm |
|インフラストラクチャ ロールのディレクトリ管理で時刻同期のエラーが報告された | DirectoryServiceTimeSynchronizationError |
|保留中の外部証明書の有効期限 | CertificateExpiration.ExternalCert.Warning |
|保留中の外部証明書の有効期限 | CertificateExpiration.ExternalCert.Critical |
|メモリ容量不足のため、特定のクラスとサイズの仮想マシンをプロビジョニングできない | AzureStack.ComputeController.VmCreationFailure.LowMemory |
|Node inaccessible for virtual machine placement\(仮想マシンを配置するためのノードにアクセスできません\) | AzureStack.ComputeController.HostUnresponsive |
|バックアップ失敗  | AzureStack.BackupController.BackupFailedGeneralFault |
|失敗した操作と競合するため、スケジュールされたバックアップがスキップされた    | AzureStack.BackupController.BackupSkippedWithFailedOperationFault |

## <a name="how-the-data-is-handled"></a>データの処理方法

**[Send logs proactively]\(ログを事前に送信する\)** を有効にする場合は、お客様は、Azure Stack Hub のシステム正常性アラートのみに基づいて、Microsoft による定期的な自動ログ収集に同意します。 また、Microsoft によって管理および制御されている Azure ストレージ アカウントに、これらのログをアップロードして保持することを承認し、同意します。

このデータはシステム正常性アラートをトラブルシューティングするためにのみ使用され、お客様の同意なしに、マーケティング、広告、その他の商業目的で使用されることはありません。 このデータは最大 90 日間保持でき、Microsoft が収集したすべてのデータは、[標準的なプライバシーに関する声明](https://privacy.microsoft.com/)に従って処理されます。

お客様の同意を得て以前に収集したすべてのデータは、お客様の許可の取り消しによる影響を受けません。

## <a name="see-also"></a>関連項目

[Azure Stack Hub でのログおよび顧客データの処理](./azure-stack-data-collection.md)
