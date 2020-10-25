---
title: 特権エンドポイント (PEP) を使用して診断ログを収集する
description: 管理者ポータルまたは PowerShell スクリプトを使用して Azure Stack Hub 内で診断ログをオンデマンドで収集する方法について説明します。
author: justinha
ms.custom: conteperfq4
ms.topic: article
ms.date: 09/02/2020
ms.author: justinha
ms.reviewer: shisab
ms.lastreviewed: 09/02/2020
ms.openlocfilehash: 95e12f2f90cd7e33fb3a2cc3a5f016f35ac0e54f
ms.sourcegitcommit: a1e2003fb9c6dacdc76f97614ff5a26a5b197b49
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/01/2020
ms.locfileid: "91623219"
---
# <a name="send-azure-stack-hub-diagnostic-logs-by-using-the-privileged-endpoint-pep"></a>特権エンドポイント (PEP) を使用して Azure Stack Hub 診断ログを送信する

<!--how do you look up the PEP IP address. You look up the azurestackstampinfo.json--->

統合システムで Get-AzureStackLog を実行するには、特権エンドポイント (PEP) へのアクセス権が必要です。 PEP を使用してログを収集するのに実行できるスクリプト例を次に示します。 実行中のログ収集をキャンセルして新しいログ収集を開始する場合は、新しいログ収集を開始するまで 5 分待ち、`Remove-PSSession -Session $session` を入力してください。


```powershell
$ipAddress = "<IP ADDRESS OF THE PEP VM>" # You can also use the machine name instead of IP here.

$password = ConvertTo-SecureString "<CLOUD ADMIN PASSWORD>" -AsPlainText -Force
$cred = New-Object -TypeName System.Management.Automation.PSCredential ("<DOMAIN NAME>\CloudAdmin", $password)

$shareCred = Get-Credential

$session = New-PSSession -ComputerName $ipAddress -ConfigurationName PrivilegedEndpoint -Credential $cred

$fromDate = (Get-Date).AddHours(-8)
$toDate = (Get-Date).AddHours(-2) # Provide the time that includes the period for your issue

Invoke-Command -Session $session { Get-AzureStackLog -OutputSharePath "<EXTERNAL SHARE ADDRESS>" -OutputShareCredential $using:shareCred -FilterByRole Storage -FromDate $using:fromDate -ToDate $using:toDate}

if ($session) {
    Remove-PSSession -Session $session
}
```

### <a name="examples"></a>例

* すべてのロールのすべてのログを収集します。

  ```powershell
  Get-AzureStackLog -OutputSharePath "<path>" -OutputShareCredential $cred
  ```

* VirtualMachines ロールと BareMetal ロールからログを収集します。

  ```powershell
  Get-AzureStackLog -OutputSharePath "<path>" -OutputShareCredential $cred -FilterByRole VirtualMachines,BareMetal
  ```

* ログ ファイルに対する過去 8 時間以内の日付範囲のフィルター処理で、VirtualMachines ロールと BareMetal ロールからログを収集します。

  ```powershell
  Get-AzureStackLog -OutputSharePath "<path>" -OutputShareCredential $cred -FilterByRole VirtualMachines,BareMetal -FromDate (Get-Date).AddHours(-8)
  ```

* ログ ファイルに対する過去 8 時間から 2 時間以内の日付範囲のフィルター処理で、VirtualMachines ロールと BareMetal ロールからログを収集します。

  ```powershell
  Get-AzureStackLog -OutputSharePath "<path>" -OutputShareCredential $cred -FilterByRole VirtualMachines,BareMetal -FromDate (Get-Date).AddHours(-8) -ToDate (Get-Date).AddHours(-2)
  ```

* Azure Stack 上で自己管理の Kubernetes クラスター (AKS エンジン) を実行しているテナントのデプロイから、ログを収集します。 Kubernetes ログは、コレクションの時間範囲を適用できる形式でテナント ストレージのアカウントに格納する必要があります。 

  ```powershell
  Get-AzureStackLog -OutputPath <Path> -InputSasUri "<Blob Service Sas URI>" -FromDate "<Beginning of the time range>" -ToDate "<End of the time range>"
  ```

  次に例を示します。

  ```powershell
  Get-AzureStackLog -OutputPath C:\KubernetesLogs -InputSasUri "https://<storageAccountName>.blob.core.windows.net/<ContainerName><SAS token>" -FromDate (Get-Date).AddHours(-8) -ToDate (Get-Date).AddHours(-2) 
  ```

* 付加価値 RP のログを収集します。 一般的な構文は次のとおりです。
 
  ```powershell
  Get-AzureStackLog -FilterByResourceProvider <<value-add RP name>>
  ```
 
  IoT Hub のログを収集するには: 

  ```powershell
  Get-AzureStackLog -FilterByResourceProvider iothubServiceHealth
  ```
 
  Event Hubs のログを収集するには:

  ```powershell
  Get-AzureStackLog -FilterByResourceProvider eventhub
  ```
 
  Azure Stack Edge のログを収集するには:

  ```powershell
  Get-AzureStackLog -FilterByResourceProvide databoxedge
  ```

* ログを収集して、指定された Azure Storage BLOB コンテナーに保存します。 この操作の一般的な構文は次のとおりです。

  ```powershell
  Get-AzureStackLog -OutputSasUri "<Blob service SAS Uri>"
  ```

  次に例を示します。

  ```powershell
  Get-AzureStackLog -OutputSasUri "https://<storageAccountName>.blob.core.windows.net/<ContainerName><SAS token>"
  ```

  > [!NOTE]
  > この手順は、ログをアップロードする場合に役立ちます。 アクセス可能な SMB 共有やインターネット アクセスがない場合でも、Azure Stack Hub で BLOB ストレージ アカウントを作成してログを転送し、クライアントを使用してそれらのログを取得することができます。  

  ストレージ アカウントの SAS トークンを生成するには、次のアクセス許可が必要です。

  * Blob Storage サービスへのアクセス。
  * コンテナー リソースの種類へのアクセス。

  `-OutputSasUri` パラメーターに使用する SAS URI の値を生成するには、次の手順に従います。

  1. [こちらの記事](/azure/storage/common/storage-quickstart-create-account)の手順に従って、ストレージ アカウントを作成します。
  2. Azure Storage Explorer のインスタンスを開きます。
  3. 手順 1 で作成したストレージ アカウントに接続します。
  4. **[Storage Service]\(Storage サービス\)** の **[BLOB コンテナー]** に移動します。
  5. **[Create a new container]\(新しいコンテナーの作成\)** を選択します。
  6. 新しいコンテナーを右クリックして、 **[Get Shared Access Signature]\(Shared Access Signature の取得\)** を選択します。
  7. お客様の要件に合わせて、有効な **[開始時間]** と **[終了時間]** を選択します。
  8. 必要なアクセス許可については、 **[読み取り]** 、 **[書き込み]** 、 **[一覧]** を選択します。
  9. **作成** を選択します。
  10. Shared Access Signature が表示されます。 URL の部分をコピーし、`-OutputSasUri` パラメーターに入力します。

### <a name="parameter-considerations"></a>パラメーターに関する考慮事項

* **OutputSharePath** パラメーターと **OutputShareCredential** パラメーターは、ユーザー指定の場所にログを格納するために使用されます。
* **FromDate** パラメーターと **ToDate** パラメーターを使用して、特定の期間のログを収集できます。 これらのパラメーターが指定されない場合、既定では過去 4 時間のログが収集されます。
* コンピューター名でログをフィルター処理するには、**FilterByNode** パラメーターを使用します。 次に例を示します。

    ```powershell
    Get-AzureStackLog -OutputSharePath "<path>" -OutputShareCredential $cred -FilterByNode azs-xrp01
    ```

* 種類でログをフィルター処理するには、**FilterByLogType** パラメーターを使用します。 File、Share、または WindowsEvent を選択してフィルター処理できます。 次に例を示します。

    ```powershell
    Get-AzureStackLog -OutputSharePath "<path>" -OutputShareCredential $cred -FilterByLogType File
    ```

* **TimeOutInMinutes** パラメーターを使用して、ログ収集のタイムアウトを設定できます。 既定では 150 (2.5 時間) に設定されています。
* ダンプ ファイルのログ収集は既定で無効になっています。 これを有効にするには、**IncludeDumpFile** スイッチ パラメーターを使用します。
* 現時点では **FilterByRole** パラメーターを使用して、次のロールごとにログ収集をフィルター処理できます。

:::row:::
   :::column span="":::

      ACS

      ACSBlob

      ACSDownloadService

      ACSFabric

      ACSFabric

      ACSMetrics

      ACSMigrationService

      ACSMonitoringService

      ACSSettingsService

      ACSTableMaster

      ACSFabric

      ACSWac

      ADFS

      ApplicationController

      ASAppGateway

      AzureBridge

      AzureMonitor

      BareMetal

      BRP

      CA

      CacheService

      Compute

      CPI

      CRP

      DeploymentMachine

      DiskRP

      Domain

   :::column-end:::
   :::column span="":::

      ECE

      EventAdminRP

      EventRP

      ExternalDNS

      FabricRing

      FabricRingServices

      FirstTierAggregationService

      FRP

      Gateway

      HealthMonitoring

      HintingServiceV2

      HRP

      IBC

      InfraServiceController

      KeyVaultAdminResourceProvider

      KeyVaultControlPlane

      KeyVaultDataPlane

      KeyVaultInternalControlPlane

      KeyVaultInternalDataPlane

      KeyVaultNamingService

      MDM

      MetricsAdminRP

      MetricsRP

      MetricsServer

      MetricsStoreService

      MonAdminRP

      MonRP

   :::column-end:::
   :::column span="":::

      NC

      NonPrivilegedAppGateway

      NRP

      OboService

      OEM

      OnboardRP

      PXE

      QueryServiceCoordinator

      QueryServiceWorker

      SeedRing

      SeedRingServices

      SLB

      SQL

      SRP

      ストレージ

      StorageController

      URP

      SupportBridgeController

      SupportRing

      SupportRingServices

      SupportBridgeRP

      UsageBridge

      VirtualMachines

      WAS

      WASPUBLIC
   
   :::column-end:::
:::row-end:::

### <a name="additional-considerations-on-diagnostic-logs"></a>診断ログに関するその他の考慮事項

* このコマンドは、どのロールに基づいてログが収集されるかによって、実行にいくらかの時間がかかります。 また、関連する要素には、ログ収集に指定した期間と、Azure Stack Hub 環境のノード数が含まれます。
* ログの収集と同時に、コマンドで指定した **OutputSharePath** パラメーターに作成された新しいフォルダーを確認します。
* ロールごとに、個別の zip ファイル内にログがあります。 収集されたログのサイズによっては、1 つのロールのログが複数の zip ファイルに分割されることがあります。 そのようなロールのすべてのログ ファイルを単一のフォルダーに解凍したい場合は、一括で解凍できるツールを使用します。 そのロールのすべての圧縮済みファイルを選択し、 **[extract here]\(ここに展開\)** を選択します。 そのロールのすべてのログ ファイルが 1 つのマージされたフォルダーに解凍されます。
* また、**Get-AzureStackLog_Output.log** と呼ばれるファイルが、圧縮済みログ ファイルを含むフォルダーに作成されます。 このファイルはコマンド出力のログで、ログ収集中の問題のトラブルシューティングに使用できます。 ログ収集の実行後に予期したログ ファイルがない場合を除き、無視してかまわない `PS>TerminatingError` エントリがログ ファイルに含まれることがあります。
* 特定のエラーを調べるには、複数のコンポーネントのログが必要な場合があります。

  * インフラストラクチャのすべての VM のシステム ログとイベント ログは、**VirtualMachines** ロールで収集されます。
  * すべてのホストのシステム ログとイベント ログは、**BareMetal** ロールで収集されます。
  * フェールオーバー クラスターおよび Hyper-V のイベント ログは、**Storage** ロールで収集されます。
  * ACS ログは、**Storage** ロールと **ACS** ロールで収集されます。

> [!NOTE]
> 記憶域スペースを効率的に使用し、ログで占領されないようにすることは極めて重要であるため、収集されるログにはサイズと有効期間の制限が強制されます。 ただし、問題を診断する場合に、こうした制限のためにもう存在していないログが必要となることがあります。 このため、外部の記憶域スペース (Azure のストレージ アカウント、オンプレミスの追加の記憶装置など) に 8 から 12 時間ごとにログをオフロードし、要件に応じて 1 から 3 か月間そこに保存しておくことを**強くお勧めします**。 また、この記憶域の場所が暗号化されるようにする必要もあります。

### <a name="invoke-azurestackondemandlog"></a>Invoke-AzureStackOnDemandLog

**Invoke-AzureStackOnDemandLog** コマンドレットを使用して、特定のロールのオンデマンド ログを生成できます (このセクションの末尾にある一覧を参照)。 このコマンドレットで生成されるログは、既定では、**Get-AzureStackLog** コマンドレットの実行時に受け取るログ バンドルには表示されません。 また、これらのログは、Microsoft サポート チームから要求された場合にのみ収集することをお勧めします。

現時点では `-FilterByRole` パラメーターを使用して、次のロールごとにログ収集をフィルター処理できます。

* OEM
* NC
* SLB
* Gateway

#### <a name="example-of-collecting-on-demand-diagnostic-logs"></a>オンデマンドの診断ログの収集例

```powershell
$ipAddress = "<IP ADDRESS OF THE PEP VM>" # You can also use the machine name instead of IP here.

$password = ConvertTo-SecureString "<CLOUD ADMIN PASSWORD>" -AsPlainText -Force
$cred = New-Object -TypeName System.Management.Automation.PSCredential ("<DOMAIN NAME>\CloudAdmin", $password)

$shareCred = Get-Credential

$session = New-PSSession -ComputerName $ipAddress -ConfigurationName PrivilegedEndpoint -Credential $cred

$fromDate = (Get-Date).AddHours(-8)
$toDate = (Get-Date).AddHours(-2) # Provide the time that includes the period for your issue

Invoke-Command -Session $session {
   Invoke-AzureStackOnDemandLog -Generate -FilterByRole "<on-demand role name>" # Provide the supported on-demand role name e.g. OEM, NC, SLB, Gateway
   Get-AzureStackLog -OutputSharePath "<external share address>" -OutputShareCredential $using:shareCred -FilterByRole Storage -FromDate $using:fromDate -ToDate $using:toDate
}

if ($session) {
   Remove-PSSession -Session $session
}
```

### <a name="how-diagnostic-log-collection-using-the-pep-works"></a>PEP を使用した診断ログ収集のしくみ

Azure Stack Hub の診断ツールにより、ログ収集を簡単かつ効率的にすることができます。 次の図は、診断ツールの動作を示しています。

![Azure Stack Hub の診断ツールのワークフロー図](media/azure-stack-diagnostics/get-azslogs.png)


#### <a name="trace-collector"></a>トレース コレクター

トレース コレクターは既定で有効になり、Azure Stack Hub コンポーネント サービスからすべての Windows イベント トレーシング (ETW) ログを収集するためにバック グラウンドで継続的に実行されます。 ETW ログは、共通のローカル共有に 5 日間を上限として格納されます。 この制限に達した場合、新しいファイルが作成されると最も古いファイルは削除されます。 各ファイルで許容される既定の最大サイズは 200 MB です。 サイズ チェックは 2 分間隔で行われ、現在のファイルが 200 MB 以上であれば保存され、新しいファイルが生成されます。 また、イベント セッションごとに生成されるファイルの合計サイズには、8 GB の制限が設定されています。

#### <a name="get-azurestacklog"></a>Get-AzureStackLog

PowerShell コマンドレット Get-AzureStackLog を使用して、Azure Stack Hub 環境のすべてのコンポーネントからログを収集できます。 これらは、ユーザーの定義した場所に zip ファイルで保存されます。 Azure Stack Hub テクニカル サポート チームが問題のトラブルシューティングを行うためにログを必要とする場合、Get-AzureStackLog を実行するようにお願いすることがあります。

> [!CAUTION]
> これらのログ ファイルには個人を特定できる情報 (PII) が含まれている可能性があります。 ログ ファイルを公開する前に、この点を考慮してください。

以下に、収集されるログの種類をいくつか例示します。

* **Azure Stack Hub のデプロイ ログ**
* **Windows イベント ログ**
* **Panther ログ**
* **クラスター ログ**
* **Storage 診断ログ**
* **ETW ログ**

これらのファイルは、トレース コレクターによって収集され、共有に保存されます。 必要に応じて、Get-AzureStackLog を使用して、それらを収集できます。

