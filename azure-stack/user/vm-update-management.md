---
title: Azure Stack Hub での VM の更新と管理の自動化
description: Azure Automation の Azure Monitor for VMs、Update Management、Change Tracking、Inventory ソリューションを使用して、Azure Stack Hub にデプロイされている Windows および Linux の VM を管理します。
author: mattbriggs
ms.topic: article
ms.date: 11/22/2020
ms.author: mabrigg
ms.reviewer: rtiberiu
ms.lastreviewed: 11/22/2020
ms.openlocfilehash: c30d254e3b9e17fa817d778e8f43b9611cb390cf
ms.sourcegitcommit: 8c745b205ea5a7a82b73b7a9daf1a7880fd1bee9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/24/2020
ms.locfileid: "95517363"
---
# <a name="vm-update-and-management-automation-in-azure-stack-hub"></a>Azure Stack Hub での VM の更新と管理の自動化
以下の Azure Automation ソリューション機能を使用して、Azure Stack Hub を使用してデプロイされている Windows および Linux の仮想マシン (VM) を管理できます。

- **[Update Management](/azure/automation/update-management/overview)** :Update Management ソリューションでは、すべてのエージェント コンピューターで利用可能な更新プログラムの状態をすばやく評価し、Windows および Linux の VM に必要な更新プログラムをインストールするプロセスを管理できます。

- **[Change Tracking](/azure/automation/automation-change-tracking)** :監視対象サーバーにインストールされているソフトウェア、Windows サービス、Windows レジストリとファイル、および Linux デーモンの変更が、クラウドの Azure Monitor サービスに送信され、処理されます。 受信したデータにロジックが適用され、クラウド サービスによってそのデータが記録されます。 [変更の追跡] ダッシュボードの情報を使用して、サーバー インフラストラクチャで行われた変更を簡単に確認できます。

- **[Inventory](/azure/automation/automation-vm-inventory)** . Azure Stack Hub VM のインベントリ追跡では、インベントリ コレクションを設定して構成するためのブラウザー ベースのユーザー インターフェイスが提供されます。

- **[Azure Monitor for VMs](/azure/azure-monitor/insights/vminsights-overview)** :Azure Monitor for VMs では、Azure と Azure Stack Hub の VM および仮想マシン スケール セットの大規模な監視が行われます。 これにより、ご利用の Windows VM および Linux VM のパフォーマンスおよび正常性が分析され、それらのプロセスとその他のリソースおよび外部プロセスとの依存関係も監視されます。

> [!IMPORTANT]
> これらのソリューションは、Azure VM の管理に使用されるものと同じです。 Azure と Azure Stack Hub の VM はどちらも、同じインターフェイスから同じツールを使用して同じ方法で管理されます。 また、Azure Stack で Update Management、Change Tracking、Inventory、Azure Monitor for VMs ソリューションを使用した場合、Azure Stack Hub VM も Azure VM と同じように課金されます。

## <a name="prerequisites"></a>前提条件
これらの機能を使用して Azure Stack Hub の VM を更新および管理するには、事前にいくつかの前提条件を満たす必要があります。 これらには、Azure portal および Azure Stack Hub 管理ポータルで実行する必要のある手順が含まれます。

### <a name="in-the-azure-portal"></a>Azure ポータルで次の操作を行います。
Azure Stack Hub の VM に対して Azure Monitor for VMs、Inventory、Change Tracking、Update Management の Azure Automation 機能を使用するには、まず、Azure でこれらのソリューションを有効にする必要があります。

> [!TIP]
> Azure VM に対してこれらの機能をすでに有効にしてある場合は、Log Analytics ワークスペースの既存の資格情報を使用できます。 使用する Log Analytics のワークスペース ID とプライマリ キーが既にある場合は、[次のセクション](./vm-update-management.md#in-the-azure-stack-hub-administrator-portal)に進んでください。 それ以外の場合は、引き続きこのセクションで新しい Log Analytics ワークスペースと Automation アカウントを作成します。

これらのソリューションを有効にする最初のステップは、Azure サブスクリプションで [Log Analytics ワークスペースを作成する](/azure/log-analytics/log-analytics-quick-create-workspace)ことです。 Log Analytics ワークスペースは、独自のデータ リポジトリ、データ ソース、ソリューションを備えた固有の Azure Monitor ログ環境です。 ワークスペースを作成した後、ワークスペース ID とキーを書き留めておいてください。 この情報は、ワークスペース ブレードに移動し、 **[詳細設定]** をクリックして、 **[ワークスペース ID]** と **[プライマリ キー]** で確認できます。 

次に、[Automation アカウントを作成する](/azure/automation/automation-create-standalone-account)必要があります。 Automation アカウントは、Azure Automation リソースのコンテナーです。 お使いの環境を分離したり、Automation のワークフローとリソースをさらに整理したりする手段を提供します。 Automation アカウントを作成した後は、Inventory、Change Tracking、Update Management の各機能を有効にする必要があります。 各機能を有効にするには、次の手順に従います。

1. Azure portal で、使用する Automation アカウントに移動します。

2. 有効にするソリューションを選択します ( **[インベントリ]** 、 **[変更の追跡]** 、または **[更新の管理]** )。

3. **[ワークスペースの選択]** ドロップダウン リストで、使用する Log Analytics ワークスペースを選択します。

4. 残りのすべての情報が正しいことを確認し、 **[有効化]** をクリックしてソリューションを有効にします。

5. 手順 2 ～ 4 を繰り返して、3 つのソリューションをすべて有効にします。 

   [![[問題の診断と解決] ウィンドウに、2 つの一覧が表示され、3 つのオプションが強調表示されています。[インベントリ] が選択されています。また、[Log Analytics ワークスペース] ドロップダウン リストと [有効] ボタンもあります。](media//vm-update-management/1-sm.PNG "Azure Automation アカウントの機能を有効にする")](media//vm-update-management/1-lg.PNG)

### <a name="enable-azure-monitor-for-vms"></a>Azure Monitor for VMs の有効化

Azure Monitor for VMs では、ご利用の Azure VM および仮想マシン スケール セットの大規模な監視が行われます。 これにより、ご利用の Windows VM および Linux VM のパフォーマンスおよび正常性が分析され、それらのプロセスとその他のリソースおよび外部プロセスとの依存関係も監視されます。

ソリューションとして、Azure Monitor for VMs では、オンプレミスまたは別のクラウド プロバイダーでホストされている VM を対象にしたパフォーマンスおよびアプリの依存関係の監視がサポートされています。 次の 3 つの主要な機能により、詳細な分析情報が提供されます。

1. Windows および Linux が稼働している Azure VM の論理コンポーネントは、事前に構成された正常性基準と照らし合わせて測定され、評価条件が満たされると通知が届きます。

2. ゲスト VM オペレーティング システムのコア パフォーマンス メトリックを表示する、事前に定義された傾向を示すパフォーマンス グラフ。

3. さまざまなリソース グループおよびサブスクリプションからの VM と相互接続されているコンポーネントを表示する依存関係マップ。

Log Analytics ワークスペースが作成されたら、Linux および Windows VM 上のコレクションに対し、ワークスペースのパフォーマンス カウンターを有効にします。 次に、ワークスペースに ServiceMap および InfrastructureInsights ソリューションをインストールして有効にします。 このプロセスについて詳しくは、「[Azure Monitor for VMs をデプロイする](/azure/azure-monitor/insights/vminsights-enable-overview)」をご覧ください。

### <a name="in-the-azure-stack-hub-administrator-portal"></a>Azure Stack Hub 管理者ポータルで
Azure portal で Azure Automation のソリューションを有効にしたら、次に、クラウド管理者として Azure Stack Hub 管理者ポータルにサインインし、Azure Stack Hub Marketplace の **Azure Monitor、更新および構成管理** および **Linux 用 Azure Monitor、更新および構成管理** という拡張機能をダウンロードします。

   ![[ホーム] > [Marketplace Management] > [Azure から追加する] > [Azure Monitor, Update and Configuration Management]\(Azure Monitor、更新および構成管理拡張機能\) ダイアログ ボックスに、拡張機能の説明と [ダウンロード] ボタンが表示されています。](media//vm-update-management/2.PNG) 

Azure Monitor for VMs マップ ソリューションを有効にして、ネットワークの依存関係の分析情報を取得するには、**Azure Monitor Dependency Agent** をダウンロードします。

   ![[ホーム] > [Marketplace Management] > [Azure から追加する] > [Azure Monitor Dependency Agent] ダイアログ ボックスに、拡張機能の説明と [ダウンロード] ボタンが表示されています。](media//vm-update-management/2-dependency.PNG) 

## <a name="enable-update-management-for-azure-stack-hub-vms"></a>Update Management を Azure Stack Hub VM に対して有効にする
以下の手順のようにして、Azure Stack Hub VM の更新管理を有効にします。

1. Azure Stack Hub のユーザー ポータルにサインインします。

2. Azure Stack Hub ユーザー ポータルで、ソリューションを有効にする VM の [拡張機能] ブレードに移動して、 **[+ 追加]** をクリックし、 **[Azure Update and Configuration Management]\(Azure 更新および構成管理\)** 拡張機能を選択して、 **[作成]** をクリックします。

    ![[Azure Update and Configuration Management]\(Azure 更新および構成管理\) ダイアログ ボックスに、説明情報、拡張機能を追加するための [作成] ボタン (強調表示)、詳細情報へのリンクが表示されています。](media//vm-update-management/3-sm.PNG "VM 拡張機能ブレード")

3. 前に作成したワークスペース ID とプライマリ キーを入力して、エージェントと Log Analytics ワークスペースをリンクします。 次に、 **[OK]** をクリックして拡張機能をデプロイします。

   [![[拡張機能のインストール] ダイアログ ボックスに、Azure WorkspaceID と WorkspaceKey 用のテキスト ボックスが表示されています。](media//vm-update-management/4-sm.PNG "ワークスペース ID とキーの指定")](media//vm-update-management/4-lg.PNG) 

4. [Update Management のドキュメント](/azure/automation/update-management/overview)の説明に従って、管理する各 VM に対して Update Management ソリューションを有効にする必要があります。 ワークスペースに報告するすべての VM でソリューションを有効にするには、 **[更新の管理]** を選択し、 **[マシンの管理]** をクリックして、 **[使用可能なマシンと今後のマシンすべてで有効にします]** オプションを選択します。

   [![[マシンの管理 - 更新の管理] ダイアログ ボックスに、[更新の管理] が有効になっていないマシンが表示されています。3 つの有効化オプションが用意されており、[使用可能なマシンと今後のマシンすべてで有効にします] が選択され、強調表示されています。[有効] ボタンがあります。](media//vm-update-management/5-sm.PNG "すべてのマシンで Update Management ソリューションを有効にする")](media//vm-update-management/5-lg.PNG) 

   > [!TIP]
   > この手順を繰り返して、ワークスペースに報告する Azure Stack Hub の VM で各ソリューションを有効にします。 
  
Azure Update and Configuration Management (Azure 更新および構成管理) 拡張機能が有効になると、管理対象の各 VM で 1 日に 2 回スキャンが実行されます。 15 分ごとに API が呼び出され、最後の更新時間を照会することで、状態が変化したかどうかが確認されます。 状態が変更された場合、コンプライアンス スキャンが開始されます。

VM のスキャン後、それらは、Update Management ソリューションの Azure Automation アカウントに表示されます。 

   [![スキャンされたマシンが一覧表示されています。それぞれについて、コンプライアンス状態、プラットフォーム、オペレーティング システム、欠落している重要な更新プログラムの数が表示されています。注意が必要なマシンの合計数、欠落している更新プログラムの合計数などが表示されています。](media//vm-update-management/6-sm.PNG "Update Management の Azure Automation アカウント")](media//vm-update-management/6-lg.PNG) 

> [!IMPORTANT]
> 管理対象のコンピューターの更新されたデータがダッシュボードに表示されるまでに、30 分～ 6 時間かかる場合があります。

Azure Stack Hub の VM は、Azure VM と共に、スケジュールされた更新プログラムのデプロイに含めることができるようになります。

##  <a name="create-an-update-deployment-schedule"></a>更新プログラムのデプロイ スケジュールを作成する

更新プログラムのデプロイ スケジュールを作成するには、PowerShell コマンドレットまたは Azure REST API を使用してマシンを渡す必要があります。 次の PowerShell の例を使用して、マシンをスケジュールしてください。 スケジュールを作成するには、`ForUpdateConfiguration` パラメーターを指定して **[New-AzAutomationSchedule](/powershell/module/az.automation/new-azautomationschedule)** コマンドレットを使用します。 次に、 **[New-AzAutomationSoftwareUpdateConfiguration](/powershell/module/az.automation/new-azautomationsoftwareupdateconfiguration)** コマンドレットを使用して、Azure Stack Hub マシンを `NonAzureComputer` パラメーターに渡します。 このスクリプトを実行するには、グローバル [Azure PowerShell Az モジュール](/powershell/azure/) を使用する必要があります。

以下の例は、その方法を示しています。

### <a name="az-modules"></a>[Az モジュール](#tab/az)

```Powershell  
$nonAzurecomputers = @("server-01", "server-02")

$startTime = ([DateTime]::Now).AddMinutes(10)

$s = New-AzAutomationSchedule -ResourceGroupName mygroup -AutomationAccountName myaccount -Name myupdateconfig -Description test-OneTime -OneTime -StartTime $startTime -ForUpdateConfiguration

New-AzAutomationSoftwareUpdateConfiguration  -ResourceGroupName $rg -AutomationAccountName $aa -Schedule $s -Windows -AzureVMResourceId $azureVMIdsW -NonAzureComputer $nonAzurecomputers -Duration (New-TimeSpan -Hours 2) -IncludedUpdateClassification Security,UpdateRollup -ExcludedKbNumber KB01,KB02 -IncludedKbNumber KB100
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm)

```Powershell  
$nonAzurecomputers = @("server-01", "server-02")

$startTime = ([DateTime]::Now).AddMinutes(10)

$s = New-AzureRMAutomationSchedule -ResourceGroupName mygroup -AutomationAccountName myaccount -Name myupdateconfig -Description test-OneTime -OneTime -StartTime $startTime -ForUpdateConfiguration

New-AzureRMAutomationSoftwareUpdateConfiguration  -ResourceGroupName $rg -AutomationAccountName $aa -Schedule $s -Windows -AzureVMResourceId $azureVMIdsW -NonAzureComputer $nonAzurecomputers -Duration (New-TimeSpan -Hours 2) -IncludedUpdateClassification Security,UpdateRollup -ExcludedKbNumber KB01,KB02 -IncludedKbNumber KB100
```

---



## <a name="enable-azure-monitor-for-vms-running-on-azure-stack-hub"></a>Azure Stack Hub で実行している Azure Monitor for VMs を有効化する
**Azure Monitor、更新、および構成管理** および **Azure Monitor Dependency Agent** の拡張機能が VM にインストールされると、[Azure Monitor for VMs](/azure/azure-monitor/insights/vminsights-overview) ソリューション内のデータのレポートが開始されます。 

> [!TIP]
> **Azure Monitor Dependency Agent** 拡張機能にはパラメーターは必要ありません。 Azure Monitor for VMs マップの Dependency Agent でデータ自体が送信されることはないため、ファイアウォールやポートを変更する必要はありません。 マップ データは、Log Analytics エージェントによって常に Azure Monitor サービスに直接送信されます。または、ご利用の IT セキュリティ ポリシーでネットワーク上のコンピューターがインターネットに接続することが許可されていない場合には、[OMS ゲートウェイ](/azure/azure-monitor/platform/gateway)を経由して送信されます。

Azure Monitor for VMs には、VM がどの程度効果的に実行されているかを判定するのに役立ついくつかの主要業績評価指標 (KPI) を対象とする一連のパフォーマンス グラフが含まれています。 グラフには、一定の期間のリソースの使用が表示されるため、ボトルネックや異常を特定できます。 また、各マシンのパースペクティブな一覧に切り替えて、選択したメトリックに基づいて、リソースの使用を表示することもできます。 パフォーマンスを扱うときに考慮すべき要素は数多くありますが、Azure Monitor for VMs では、プロセッサ、メモリ、ネットワーク アダプター、およびディスクの使用に関連するオペレーティング システムの主要なパフォーマンス指標を監視します。 パフォーマンス グラフは、正常性監視機能を補完し、システム コンポーネントの障害の可能性を示す問題を明らかにするために役立ちます。 また Azure Monitor for VMs は、キャパシティ プランニングとチューニングおよび最適化もサポートし、効率を高めます。

   ![Azure Monitor VM [パフォーマンス] タブ](http:/docs.microsoft.com/azure/azure-monitor/insights/media/vminsights-performance/vminsights-performance-aggview-01.png)

Azure Stack Hub で実行されている Windows および Linux VM で検出されたアプリ コンポーネントの表示は、Azure Monitor for VMs による 2 つの方法で観察できます。 1 つ目は VM から直接、2 つ目は Azure Monitor から VM のグループ全体に対するものです。
「[Azure Monitor for VMs のマップ機能を使用してアプリケーション コンポーネントを把握する](/azure/azure-monitor/insights/vminsights-maps)」の記事は、2 つの観点のエクスペリエンスとマップ機能の使用方法を理解するために役立ちます。

   ![Azure Monitor VMs [マップ] タブ]((http:/docs.microsoft.com/azure/azure-monitor/insights/media/vminsights-maps/map-multivm-azure-monitor-01.png)

[Azure Monitor for VMs](/azure/azure-monitor/insights/vminsights-overview) にパフォーマンス データが表示されない場合は、[LogAnalytics Workspace](/azure/azure-monitor/platform/data-sources-performance-counters) の [詳細設定] で Windows および Linux のパフォーマンス データの収集を有効にする必要があります。

## <a name="enable-update-management-using-a-resource-manager-template"></a>Resource Manager テンプレートを使用して Update Management を有効にする
Azure Stack Hub の VM の数が多い場合は、[この Azure Resource Manager テンプレート](https://aka.ms/aa6zdzy)を使用して、さらに簡単に VM にソリューションをデプロイできます。 このテンプレートでは、Microsoft Monitoring Agent 拡張機能が既存の Azure Stack Hub VM にデプロイされ、その VM が既存の Azure LogAnalytics ワークスペースに追加されます。
 
## <a name="next-steps"></a>次のステップ
[SQL Server VM のパフォーマンスを最適化する](azure-stack-sql-server-vm-considerations.md)
