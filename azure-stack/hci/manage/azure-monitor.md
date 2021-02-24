---
title: Azure Monitor を使用して Azure Stack HCI を監視する
description: Windows Admin Center から、Azure Monitor を使用してサーバーを監視し、アラートを構成します。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.date: 07/21/2020
ms.openlocfilehash: f721b16d6742cde5e27fae8b81d8d256c7defa2a
ms.sourcegitcommit: 0e52f460295255b799bac92b40122a22bf994e27
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/21/2020
ms.locfileid: "86866809"
---
# <a name="monitor-azure-stack-hci-with-azure-monitor"></a>Azure Monitor を使用して Azure Stack HCI を監視する

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

[Azure Monitor](/azure/azure-monitor/overview) により、オンプレミスとクラウドの両方の Windows サーバーや仮想マシン (VM) を含むさまざまなリソースから、テレメトリが収集、分析、および処理されます。 Azure Monitor では Azure VM と他の Azure リソースからデータがプルされますが、この記事では、Azure Stack HCI で実行されているオンプレミスのサーバーと VM が Azure Monitor でどのように処理されるかを、具体的に Windows Admin Center を使用する場合について重点的に説明します。

## <a name="how-does-azure-monitor-work"></a>Azure Monitor のしくみ
:::image type="content" source="media/monitor/azure-monitor-diagram.png" alt-text="Azure Monitor のしくみを示す図" border="false":::
オンプレミスの Windows Server から生成されたデータは、Azure Monitor の Log Analytics ワークスペースに収集されます。 ワークスペース内では、さまざまな監視ソリューション (特定のシナリオに関する分析情報を示す一連のロジック) を有効にすることができます。 たとえば、Azure Update Management、Azure Security Center、Azure Monitor for VMs はすべて、ワークスペース内で有効にできる監視ソリューションです。

Log Analytics ワークスペースで監視ソリューションを有効にすると、そのワークスペースに報告を行うすべてのサーバーが、そのソリューションに関連するデータの収集を開始します。これにより、このソリューションはワークスペース内のすべてのサーバーに関する分析情報を生成できます。

オンプレミスのサーバーで診断データを収集して Log Analytics ワークスペースにプッシュするには、Azure Monitor に Microsoft Monitoring Agent (MMA) のインストールが必要です。 特定の監視ソリューションではセカンダリ エージェントも必要です。 たとえば、Azure Monitor for VMs は、このソリューションで提供される追加機能のために ServiceMap エージェントにも依存しています。

Azure Update Management などの一部のソリューションは Azure Automation にも依存しています。これにより、Azure 環境と非 Azure 環境でリソースを一元的に管理できます。 たとえば、Azure Update Management では Azure Automation を使用することで、Azure portal から一元的に、環境内のコンピューター間で更新プログラムのインストールをスケジュール設定し、調整します。

## <a name="what-data-does-azure-monitor-collect"></a>Azure Monitor が収集するデータ

Azure Monitor によって収集されたすべてのデータは、2 つの基本的な種類であるメトリックとログのどちらかに該当します。

1. [メトリック](/azure/azure-monitor/platform/data-platform#metrics)は、特定の時点におけるシステムの何らかの側面を表す数値です。 メトリックは軽量であり、リアルタイムに近いシナリオをサポートできます。 Azure Monitor で収集されたデータは、Azure portal の **[概要]** ページで参照できます。

    :::image type="content" source="media/monitor/metrics.png" alt-text="メトリックス エクスプローラーへのメトリックの取り込みの画像" border="false":::

2. [ログ](/azure/azure-monitor/platform/data-platform#logs)には、種類ごとに異なるプロパティ セットを持つレコードに編成されたさまざまな種類のデータが含まれます。 イベントやトレースなどの利用統計情報は、組み合わせて分析できるように、パフォーマンス データとともにログとして格納されます。 Azure Monitor が収集したログ データは、収集されたデータをすばやく検索、統合、分析する[クエリ](/azure/azure-monitor/log-query/log-query-overview)を使用して分析できます。 Azure portal で [Log Analytics](/azure/azure-monitor/log-query/portals) を使用してクエリを作成およびテストした後、これらのツールを使用してデータを直接分析できるほか、クエリを保存して[視覚化](/azure/azure-monitor/visualizations)または[アラート ルール](/azure/azure-monitor/platform/alerts-overview)に利用することができます。

    :::image type="content" source="media/monitor/logs.png" alt-text="Log Analytics へのログの取り込みの画像" border="false":::

## <a name="how-does-windows-admin-center-enable-you-to-use-azure-monitor"></a>Windows Admin Center で Azure Monitor を使用できるようにする方法

Windows Admin Center 内から、次の 3 つの監視ソリューションを有効にすることができます。

- [クラスター用の Azure Monitor](#onboard-your-cluster-using-windows-admin-center)
- [Azure Update Management](/windows-server/manage/windows-admin-center/azure/azure-update-management) (**更新プログラム** ツール内)
- Azure Monitor for VMs (サーバー設定内)、別名: 仮想マシン分析情報

これらのどのツールからも Azure Monitor の使用を開始できます。 これまで Azure Monitor を使用したことがない場合、Windows Admin Center によって Log Analytics ワークスペース (および必要に応じて Azure Automation アカウント) が自動的にプロビジョニングされ、ターゲット サーバーで MMA がインストールされ、構成されます。 その後に、対応するソリューションがワークスペースにインストールされます。

たとえば、Azure Update Management を設定するために初めて**更新プログラム** ツールにアクセスすると、Windows Admin Center で以下が行われます。

1. コンピューターに MMA がインストールされる。
2. Log Analytics ワークスペースと Azure Automation アカウントが作成される (この場合は Azure Automation アカウントが必要であるため)。
3. 新しく作成されたワークスペースに Update Management ソリューションがインストールされる。

Windows Admin Center 内から同じサーバーに別の監視ソリューションを追加する場合は、Windows Admin Center では、単にそのサーバーが接続されている既存のワークスペースにそのソリューションがインストールされます。 Windows Admin Center では、その他の必要なエージェントも追加でインストールされます。

別のサーバーに接続する場合は、(Windows Admin Center を使用するか、Azure portal で手動により) Log Analytics ワークスペースを既に設定済みであれば、MMA をそのサーバーにインストールして、既存のワークスペースに接続することもできます。 サーバーをワークスペースに接続すると、サーバーは自動的にデータの収集と、そのワークスペースにインストールされているソリューションへの報告を開始します。

## <a name="azure-monitor-for-virtual-machines-virtual-machine-insights"></a>仮想マシン用の Azure Monitor (仮想マシン分析情報)

**[サーバー設定]** で Azure Monitor for VMs を設定すると、Windows Admin Center により、Azure Monitor for VMs ソリューション (仮想マシン分析情報とも呼ばれる) が有効になります。 このソリューションでは、サーバーの正常性とイベントの監視、電子メール アラートの作成、環境全体のサーバー パフォーマンスの統合ビューの取得、および特定のサーバーに接続されているアプリ、システム、およびサービスの視覚化を行うことができます。

> [!NOTE]
> 名前に反して、仮想マシン分析情報は、仮想マシンだけでなく物理サーバーに対しても機能します。

Azure Monitor の顧客ごとの 1 か月あたり 5 GB の無料データを使用すると、課金されることを心配をせずに、1 台か 2 台のサーバーでこれを試すことができます。 引き続き、環境内のサーバー全体のシステム パフォーマンスの統合ビューの取得など、サーバーを Azure Monitor にオンボードすることによるその他のメリットを見ていきます。

## <a name="onboard-your-cluster-using-windows-admin-center"></a>Windows Admin Center を使用してクラスターをオンボードする

クラスターを Azure Monitor にオンボードする最も簡単な方法は、Windows Admin Center の自動化ワークフローを使用して、ヘルス サービスと Log Analytics を構成し、MMA をインストールすることです。

:::image type="content" source="media/monitor/onboarding.gif" alt-text="Azure Monitor にクラスターをオンボードする画像":::

サーバー接続の [概要] ページで、新しいボタン **[アラートの管理]** をクリックするか、 **[サーバー設定] > [Monitoring and alerts]\(監視とアラート\)** へ移動します。 このページで、 **[Set up]\(設定\)** をクリックして設定ウィンドウを完了し、サーバーを Azure Monitor にオンボードします。 Azure Log Analytics ワークスペースのプロビジョニング、必要なエージェントのインストール、および仮想マシン分析情報ソリューションが構成されていることの確認が、Admin Center によって行われます。 完了すると、サーバーから Azure Monitor にパフォーマンス カウンター データが送信され、Azure portal から、このサーバーに基づいて電子メール アラートを表示および作成できるようになります。

## <a name="onboard-your-cluster-manually-using-powershell"></a>PowerShell を使用して手動でクラスターをオンボードする

クラスターを手動でオンボードする場合は、次の手順に従います。

### <a name="configure-health-service"></a>ヘルス サービスの構成

まず、クラスターを構成する必要があります。 ご存じのように、[ヘルス サービス](/windows-server/failover-clustering/health-service-overview)によって、記憶域スペース ダイレクトが実行されているクラスターでの日常的な監視と操作エクスペリエンスが向上します。

上記で説明したように、Azure Monitor では、クラスター内で実行されている各ノードからログが収集されます。 そのため、イベント チャネルに書き込まむようにヘルス サービスを構成する必要があります。イベント チャネルは以下であるとします。

```
Event Channel: Microsoft-Windows-Health/Operational
Event ID: 8465
```

ヘルス サービスを構成するには、以下を実行します。

```PowerShell
get-storagesubsystem clus* | Set-StorageHealthSetting -Name "Platform.ETW.MasTypes" -Value "Microsoft.Health.EntityType.Subsystem,Microsoft.Health.EntityType.Server,Microsoft.Health.EntityType.PhysicalDisk,Microsoft.Health.EntityType.StoragePool,Microsoft.Health.EntityType.Volume,Microsoft.Health.EntityType.Cluster"
```

上記のコマンドレットを実行すると、正常性の設定によって、*Microsoft-Windows-Health/Operational* イベント チャネルへのイベントの書き込みが開始されます。

### <a name="configure-log-analytics"></a>Log Analytics の構成

クラスターで適切なログ記録を設定したので、次の手順では Log Analytics を適切に構成します。

おおまかに説明すると、[Azure Log Analytics](/azure/azure-monitor/platform/agent-windows) では、詳細な分析と相関のために、データセンターやその他のクラウド環境内の物理的または仮想的な Windows コンピューターから直接データを 1 つのリポジトリに収集することができます。

サポートされている構成を確認するには、「[サポートされている Windows オペレーティング システム](/azure/azure-monitor/platform/log-analytics-agent#supported-windows-operating-systems)」と「[ネットワーク ファイアウォールの構成](/azure/azure-monitor/platform/log-analytics-agent#network-firewall-requirements)」をご覧ください。

Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) を作成してください。

#### <a name="log-in-to-azure-portal"></a>Azure Portal へのログイン

Azure Portal ([https://portal.azure.com](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)) にログインします。

#### <a name="create-a-workspace"></a>ワークスペースの作成

以下に示す手順の詳細については、[Azure Monitor のドキュメント](/azure/azure-monitor/learn/quick-collect-windows-computer)を参照してください。

1. Azure Portal で、 **[すべてのサービス]** をクリックします。 リソースの一覧で、「**Log Analytics**」と入力します。 入力を始めると、入力内容に基づいて、一覧がフィルター処理されます。 **[Log Analytics]** を選択します。

    :::image type="content" source="media/monitor/azure-portal-01.png" alt-text="Azure Portal":::

2. **[作成]** をクリックし、次の項目について選択します。

   * 新しい **Log Analytics ワークスペース**の名前 (*DefaultLAWorkspace* など) を指定します。
   * 関連付ける**サブスクリプション**をドロップダウン リストから選択します (既定値が適切でない場合)。
   * **[リソース グループ]** では、1 つ以上の Azure Virtual Machines を含む既存のリソース グループを選択します。

    :::image type="content" source="media/monitor/create-loganalytics-workspace-02.png" alt-text="Log Analytics リソースの作成ブレード":::

3. **[Log Analytics ワークスペース]** ウィンドウで必要な情報を入力したら、 **[OK]** をクリックします。

情報が検証され、ワークスペースが作成されている間、メニューの **[通知]** でその進行状況を追跡することができます。

#### <a name="obtain-workspace-id-and-key"></a>ワークスペース ID とキーを取得する
Windows 用 MMA をインストールする前に、Log Analytics ワークスペースのワークスペース ID とキーが必要です。  この情報は、設定ウィザードがエージェントを適切に構成し、そのエージェントを Log Analytics と正常に通信できるようにするために必要です。

1. Azure Portal の左上隅にある **[すべてのサービス]** をクリックします。 リソースの一覧で、「**Log Analytics**」と入力します。 入力を始めると、入力内容に基づいて、一覧がフィルター処理されます。 **[Log Analytics]** を選択します。
2. Log Analytics ワークスペースの一覧で、前の手順で作成した *DefaultLAWorkspace* を選択します。
3. **[詳細設定]** を選択します。
    :::image type="content" source="media/monitor/log-analytics-advanced-settings-01.png" alt-text="Log Analytics の詳細設定":::
4. **[接続されたソース]** 、 **[Windows サーバー]** の順に選択します。
5. **[ワークスペース ID]** と **[主キー]** の右側に値が表示されます。 一時的に両方を保存しておきます。ここでは、任意のエディターに両方をコピーして貼り付けます。

### <a name="installing-the-mma-on-windows"></a>Windows に MMA をインストールする
次の手順では、Microsoft Monitoring Agent をインストールして構成します。

> [!IMPORTANT]
> 必ず、クラスター内の各サーバーにこのエージェントをインストールし、このエージェントを Windows の起動時に実行するように指定してください。

1. **[Windows サーバー]** ページの **[Windows エージェントのダウンロード]** で、Windows オペレーティング システムのプロセッサ アーキテクチャに応じた適切なバージョンを選択します。
2. セットアップを実行して、コンピューターにエージェントをインストールします。
3. **[ようこそ]** ページで **[次へ]** をクリックします。
4. **[ライセンス条項]** ページの記述内容を確認し、 **[同意する]** をクリックします。
5. **[インストール先フォルダー]** ページで、既定のインストール フォルダーを変更するか、変更せずに **[次へ]** をクリックします。
6. **[エージェントのセットアップ オプション]** ページで、エージェントを接続する Azure Log Analytics をクリックし、 **[次へ]** をクリックします。
7. **[Azure Log Analytics]** ページで、先ほどコピーしておいた**ワークスペース ID** と**ワークスペース キー (主キー)** を貼り付けます。 コンピューターがプロキシ サーバーを介して Log Analytics サービスと通信する必要がある場合は、 **[詳細]** をクリックし、プロキシ サーバーの URL とポート番号を指定します。 プロキシ サーバーで認証が必要な場合には、プロキシ サーバーにアクセスするためのユーザー名とパスワードを入力し、 **[次へ]** をクリックします。
8. 必要な構成設定をしたら、 **[次へ]** をクリックします。
    :::image type="content" source="media/monitor/log-analytics-mma-setup-laworkspace.png" alt-text="ワークスペース ID と主キーの貼り付け":::
9. **[インストールの準備完了]** ページで、設定内容を確認し、 **[インストール]** をクリックします。
10. **[構成は正常に終了しました]** ページで **[完了]** をクリックします。

完了すると、**コントロール パネル**に **Microsoft Monitoring Agent** が表示されます。 構成を検証して、エージェントが Log Analytics に接続されていることを確認できます。 接続時に、 **[Azure ログ分析]** タブで、エージェントに以下を示すメッセージが表示されます。**Microsoft Monitoring Agent は、Microsoft ログ分析サービスに正常に接続されています。**

:::image type="content" source="media/monitor/log-analytics-mma-laworkspace-status.png" alt-text="Log Analytics への MMA 接続の状態":::

サポートされている構成を確認するには、「[サポートされている Windows オペレーティング システム](/azure/azure-monitor/platform/log-analytics-agent#supported-windows-operating-systems)」と「[ネットワーク ファイアウォールの構成](/azure/azure-monitor/platform/log-analytics-agent#network-firewall-requirements)」をご覧ください。

## <a name="setting-up-alerts-using-windows-admin-center"></a>Windows Admin Center を使用してアラートを設定する

サーバーを Azure Monitor にアタッチしたら、 **[設定] > [Monitoring and alerts]\(監視とアラート\)** ページ内のインテリジェント ハイパーリンクを使用して、Azure portal に移動できます。 Windows Admin Center では、Log Analytics ワークスペース内のすべてのサーバーに適用される既定のアラートを簡単に構成できます。 Windows Admin Center では自動的にパフォーマンス カウンターを収集できるため、事前に定義された多くのクエリのいずれかをカスタマイズするか独自のものを作成して、[新しいアラートを作成](/azure/azure-monitor/platform/alerts-log)できます。

:::image type="content" source="media/monitor/setup1.gif" alt-text="アラートの構成のスクリーンショット":::

選択できるアラートとその既定の条件を次に示します。

| アラート名                | 既定の条件                                  |
|---------------------------|----------------------------------------------------|
| CPU 使用率           | 10 分の間 85% を超えている                            |
| ディスク容量使用率 | 10 分の間 85% を超えている                            |
| メモリ使用率        | 使用可能なメモリが 10 分の間 100 MB 未満である   |
| Heartbeat                 | 5 分間のビート数が 2 未満である                   |
| システムの重大なエラー     | クラスター システムのイベント ログ内に重大なアラートがある |
| ヘルス サービスのアラート      | クラスター上にヘルス サービスのエラーがある            |

Windows Admin Center でアラートを構成すると、Azure の Log Analytics ワークスペースにアラートが表示されます。

:::image type="content" source="media/monitor/setup2.gif" alt-text="アラートの表示のスクリーンショット":::

### <a name="collecting-event-and-performance-data"></a>イベントとパフォーマンス データを収集する

Log Analytics は、イベントを Windows イベント ログから収集でき、長期分析およびレポートのために指定されたパフォーマンス カウンターからも収集できます。また、特定の条件が検出された場合はアクションを実行できます。 以下の手順に従って、Windows イベント ログと、手始めとしていくつかの一般的なパフォーマンス カウンターからのイベントの収集を構成します。

1. Azure ポータルで、左下隅にある **[その他のサービス]** をクリックします。 リソースの一覧で、「**Log Analytics**」と入力します。 入力を始めると、入力内容に基づいて、一覧がフィルター処理されます。 **[Log Analytics]** を選択します。
2. **[詳細設定]** を選択します。
    :::image type="content" source="media/monitor/log-analytics-advanced-settings-01.png" alt-text="Log Analytics の詳細設定":::
3. **[データ]** を選択してから、 **[Windows イベント ログ]** を選択します。
4. ここで、次の名前を入力してヘルス サービス イベント チャネルを追加し、プラス記号 **+** をクリックします。
   ```
   Event Channel: Microsoft-Windows-Health/Operational
   ```
5. テーブルで、重大度の **[エラー]** と **[警告]** のチェック ボックスをオンにします。
6. ページの上部にある **[保存]** をクリックして構成を保存します。
7. **[Windows パフォーマンス カウンター]** を選択して、Windows コンピューターでのパフォーマンス カウンターの収集を有効にします。
8. 新しい Log Analytics ワークスペースの Windows パフォーマンス カウンターを初めて構成する場合は、いくつかの一般的なカウンターをすばやく作成するためのオプションが表示されます。 それぞれのオプションの横には、チェック ボックスが表示されます。
    :::image type="content" source="media/monitor/windows-perfcounters-default.png" alt-text="既定の Windows パフォーマンス カウンターが選択されている状態":::
    **[選択したパフォーマンス カウンターを追加する]** をクリックします。  カウンターが追加され、10 秒間の収集サンプル間隔でプリセットされます。
9. ページの上部にある **[保存]** をクリックして構成を保存します。

## <a name="create-queries-and-alerts-based-on-log-data"></a>ログ データに基づいてクエリとアラートを作成する

ここまで進むと、クラスターから Log Analytics にログとパフォーマンス カウンターが送信されるようになります。 次の手順では、ログ検索を一定の間隔で自動的に実行するアラート ルールを作成します。 ログ検索の結果が特定の条件に一致した場合、電子メールまたはテキスト通知を送信するアラートが発生します。 以下で、これについて説明します。

### <a name="create-a-query"></a>クエリを作成する

最初に、**ログ検索**ポータルを開きます。

1. Azure Portal で、 **[すべてのサービス]** をクリックします。 リソースの一覧で「**Monitor**」と入力します。 入力を始めると、入力内容に基づいて、一覧がフィルター処理されます。 **[モニター]** を選択します。
2. **[モニター]** ナビゲーション メニューで **[Log Analytics]** を選択し、ワークスペースを選択します。

テーブルのすべてのレコードを返すシンプルなクエリを使用すると、作業データを最も簡単に取得できます。 検索ボックスに次のクエリを入力し、検索ボタンをクリックします。

```
Event
```

データは既定のリスト ビューに返されます。ここで、返されたレコードの合計数を確認できます。

:::image type="content" source="media/monitor/log-analytics-portal-eventlist-01.png" alt-text="簡単なクエリのスクリーンショット":::

画面の左側はフィルター ウィンドウです。ここでは、クエリを直接変更せずに、フィルターを追加することができます。  そのレコードの種類について、レコードのプロパティがいくつか表示されます。1 つまたは複数のプロパティ値を選択して検索結果を絞り込むことができます。

**[EVENTLEVELNAME]** の下にある **[エラー]** の横のチェックボックスをオンにするか、次のように入力して結果をエラー イベントに限定します。

```
Event | where (EventLevelName == "Error")
```

:::image type="content" source="media/monitor/log-analytics-portal-eventlist-02.png" alt-text="フィルターのスクリーンショット":::

確認するイベントに関する適切なクエリを作成したら、次の手順のために保存します。

### <a name="create-alerts"></a>アラートを作成する
ここで、アラートの作成例を見ていきましょう。

1. Azure Portal で、 **[すべてのサービス]** をクリックします。 リソースの一覧で、「**Log Analytics**」と入力します。 入力を始めると、入力内容に基づいて、一覧がフィルター処理されます。 **[Log Analytics]** を選択します。
2. 左側のウィンドウで、 **[アラート]** を選択し、ページの上部の **[新しいアラート ルール]** をクリックして新しいアラートを作成します。
    :::image type="content" source="media/monitor/alert-rule-02.png" alt-text="新しいアラート ルールの作成のスクリーンショット":::
3. 最初の手順では、 **[アラートの作成]** セクションで、リソースとして Log Analytics ワークスペースを選択します。これがログ ベースのアラート シグナルであるためです。  複数のサブスクリプションがある場合は、ドロップダウン リストから、先ほど作成した Log Analytics ワークスペースが含まれている特定の**サブスクリプション**を選択して結果をフィルターします。  ドロップダウン リストから **[Log Analytics]** を選択して **[リソースの種類]** をフィルターします。  最後に、 **[リソース**  **DefaultLAWorkspace]** を選択し、 **[完了]** をクリックします。
    :::image type="content" source="media/monitor/alert-rule-03.png" alt-text="新しいアラート ルールの作成の手順 1 のスクリーンショット":::
4. **[アラートの条件]** セクションで、 **[条件の追加]** をクリックして保存済みのクエリを選択し、アラート ルールが従うロジックを指定します。
5. 次の情報を指定して、アラートを構成します。a. **[基準]** ドロップダウン リストで **[メトリック測定]** を選択します。  メトリック測定では、クエリの対象となったオブジェクトのうち、値が指定されたしきい値を上回っているオブジェクトについて、それぞれ別個にアラートが生成されます。
   b. **[条件]** では、 **[より大きい]** を選択し、しきい値を指定します。
   c. 次に、アラートをトリガーするタイミングを定義します。 たとえば、 **[連続する違反]** を選択し、ドロップダウン リストから **[より大きい]** を選択し、値「3」を入力します。
   d. [評価基準] セクションで、 **[期間]** の値を **30** 分に変更し、 **[頻度]** を 5 に変更します。 ルールは 5 分ごとに実行され、現在の時刻から直近の 30 分以内に作成されたレコードが返されます。  この期間をより広い時間枠に設定すると、潜在的なデータの待機時間の原因となるため、クエリでは、アラートが決して発生しない検知漏れを回避するために確実にデータが返されるようにします。
6. **[完了]** をクリックして、アラート ルールを完成させます。
    :::image type="content" source="media/monitor/alert-signal-logic-02.png" alt-text="アラート シグナルの構成のスクリーンショット":::
7. 2 番目の手順に進み、 **[アラート ルール名]** フィールドに「**すべてのエラー イベントでのアラート**」のようなアラートの名前を入力します。  アラートの詳細を説明する **[説明]** を指定し、表示されたオプションから **[重大度]** 値として **[重大 (重大度 0)]** を選択します。
8. 作成時にアラート ルールをすぐにアクティブ化するには、 **[ルールの作成時に有効にする]** の既定値を受け入れます。
9. 3 番目および最後の手順では、 **[アクション グループ]** を指定します。これにより、アラートがトリガーされるたびに同じアクションが実行され、定義する各ルールに同じアクションを使用できます。 次の情報を指定して、新しいアクション グループを構成します。a. **[新しいアクション グループ]** を選択すると、 **[アクション グループの追加]** ウィンドウが表示されます。
   b. **[アクション グループ名]** で、「**IT 操作 - 通知**」のような名前と **itops-n** などの **[短い名前]** を指定します。
   c. **[サブスクリプション]** と **[リソース グループ]** の既定値が正しいことを確認します。 正しくない場合は、ドロップダウン リストから正しいものを 1 つ選択します。
   d. [アクション] セクションで、「**電子メールの送信**」などのアクション名を指定し、 **[アクションの種類]** でドロップダウン リストから **[電子メール/SMS/プッシュ/音声]** を選択します。 追加情報を指定するために **[電子メール/SMS/プッシュ/音声]** プロパティ ウィンドウが右側に開きます。
   e. **[電子メール/SMS/プッシュ/音声]** ウィンドウで、任意のものを選択して設定します。 たとえば、 **[電子メール]** を有効にし、メッセージの配信先の有効な電子メール SMTP アドレスを指定します。
   f. **[OK]** をクリックして変更を保存します。<br><br>

    :::image type="content" source="media/monitor/action-group-properties-01.png" alt-text="新しいアクション グループの作成のスクリーンショット":::

10. **[OK]** をクリックしてアクション グループを完成させます。
11. **[アラート ルールの作成]** をクリックしてアラート ルールを完成させます。 すぐに実行が開始されます。
    :::image type="content" source="media/monitor/alert-rule-01.png" alt-text="新しいアラート ルールの作成の完了のスクリーンショット":::

### <a name="example-alert"></a>アラートの例

参考のため、Azure でのアラートの例を次に示します。

:::image type="content" source="media/monitor/alert.gif" alt-text="Azure アラートのスクリーンショット":::

Azure Monitor によって送信される電子メールの例を次に示します。

:::image type="content" source="media/monitor/warning.png" alt-text="アラートの電子メールの例のスクリーンショット":::

## <a name="create-custom-kusto-queries-in-log-analytics"></a>Log Analytics でカスタム Kusto クエリを作成する

Azure Monitor で Kusto クエリ言語を使用して、1 つ以上の仮想マシンからデータを収集するための[カスタム ログ クエリを作成](/azure/azure-monitor/log-query/get-started-queries)することもできます。

## <a name="get-a-consolidated-view-across-multiple-servers"></a>複数のサーバー全体の統合ビューを取得する

Azure Monitor 内で複数のサーバーを 1 つの Log Analytics ワークスペースにオンボードすると、Azure Monitor 内の [仮想マシン分析情報](/azure/azure-monitor/insights/vminsights-overview)ソリューションからこれらすべてのサーバーの統合ビューを取得できます。 (オンプレミス サーバーでは、Azure Monitor の仮想マシン分析情報の [パフォーマンス] タブと [マップ] タブのみが動作することに注意してください。正常性のタブは Azure VM でのみ機能します。)これを Azure portal で表示するには、 **[Azure Monitor] > [仮想マシン]** ([分析情報] の下) に進み、 **[パフォーマンス]** または **[マップ]** タブに移動します。

## <a name="visualize-connected-services"></a>接続済みサービスを視覚化する

Windows Admin Center によってサーバーが Azure Monitor 内の仮想マシン分析情報ソリューションにオンボードされると、[Service Map](/azure/azure-monitor/insights/service-map) と呼ばれる機能も利用できるようになります。 この機能によってアプリケーション コンポーネントが自動的に検出され、サービス間の通信がマップされるため、Azure portal から簡単に、サーバー間の接続を詳細に視覚化できます。 これを見つけるには、Azure portal に移動し、 **[Azure Monitor] > [仮想マシン]** ([分析情報] の下) を選択し、 **[マップ]** タブに移動します。

> [!NOTE]
> Azure Monitor の仮想マシン分析情報の視覚化は、現在 6 つのパブリック リージョンで提供されています。  最新情報については、[Azure Monitor for VMs のドキュメント](/azure/azure-monitor/insights/vminsights-onboard#log-analytics)を参照してください。 上記で説明した仮想マシン分析情報ソリューションで提供される追加のメリットを得るには、サポートされているリージョンのいずれかに Log Analytics ワークスペースをデプロイする必要があります。

## <a name="disabling-monitoring"></a>監視を無効化する

Log Analytics ワークスペースからサーバーを完全に切り離すには、MMA をアンインストールします。 これは、このサーバーがデータをワークスペースに送信しなくなり、そのワークスペースにインストールされたすべてのソリューションが、そのサーバーからのデータの収集と処理を行わなくなることを意味します。 ただし、これはワークスペース自体には影響しません。そのワークスペースに報告を行うすべてのリソースはその動作を続けます。 Windows Admin Center 内で MMA エージェントをアンインストールするには、サーバーに接続してから **[インストール済みアプリ]** に移動し、Microsoft Monitoring Agent を見つけて、 **[削除]** を選択します。

ワークスペース内の特定のソリューションを無効にする場合は、[Azure portal から監視ソリューションを削除する](/azure/azure-monitor/insights/solutions#remove-a-management-solution)必要があります。 監視ソリューションを削除すると、そのソリューションによって作成される分析情報が、そのワークスペースに報告を行う_どのサーバーでも_生成されなくなります。 たとえば、Azure Monitor for VMs ソリューションをアンインストールすると、ワークスペースに接続されているどのマシンからも VM またはサーバーのパフォーマンスに関する分析情報が表示されなくなります。

## <a name="next-steps"></a>次のステップ

関連トピックについては、以下も参照してください。

- [Windows Admin Center との Azure の統合の詳細について確認する](/windows-server/manage/windows-admin-center/azure/)
- [Azure Monitor を使用してヘルス サービスのエラーの電子メールを送信する](/windows-server/storage/storage-spaces/configure-azure-monitor)
