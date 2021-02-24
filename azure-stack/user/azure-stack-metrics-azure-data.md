---
title: Azure Stack Hub 上の Azure Monitor を使用する
description: Azure Stack Hub 上の Azure Monitor の使用方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 2/1/2021
ms.author: mabrigg
ms.lastreviewed: 12/01/2019
ms.openlocfilehash: 9ba37820acd43e9a6ea50716bfa5d901dd02a4a9
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99247270"
---
# <a name="use-azure-monitor-on-azure-stack-hub"></a>Azure Stack Hub 上の Azure Monitor を使用する

この記事では、Azure Stack Hub 上の Azure Monitor サービスの概要を説明します。 Azure Monitor の動作について説明し、Azure Stack Hub での Azure Monitor の使用方法に関するその他の情報を紹介します。

Azure Monitor の概要については、[Azure Stack Hub 上の Azure Monitor の概要](/azure/monitoring-and-diagnostics/monitoring-get-started)に関するグローバル Azure の記事を参照してください。

![Azure Stack Hub の Monitor ブレード](./media/azure-stack-metrics-azure-data/azs-monitor.png)

Azure Monitor は、Azure リソースを監視するための 1 つのソースを提供するプラットフォーム サービスです。 Azure Monitor では、Azure のリソースのメトリックとログを視覚化、クエリ、ルーティング、アーカイブし、そのメトリックとログに対してその他のアクションを実行できます。 このデータは、Azure Stack Hub 管理者ポータル、Monitor PowerShell コマンドレット、クロス プラットフォーム CLI、または Azure Monitor REST API を使用して操作できます。 Azure Stack Hub でサポートされている特定の接続については、「[Azure Stack Hub の監視データの使用](azure-stack-metrics-monitor.md)」を参照してください。

> [!NOTE]
> Azure Stack Development Kit では、メトリックと診断ログを使用できません。

## <a name="prerequisites-for-azure-monitor-on-azure-stack-hub"></a>Azure Stack Hub 上の Azure Monitor の前提条件

サブスクリプションのオファー リソース プロバイダー設定で **Microsoft.insights** リソース プロバイダーを登録します。 サブスクリプションに関連付けられているオファーでリソース プロバイダーが使用可能であることを確認できます。

1. Azure Stack Hub ユーザー ポータルを開きます。
2. **[サブスクリプション]** を選択します。
3. 登録するサブスクリプションを選択します。
4. **[設定]** で、 **[リソース プロバイダー]** を選択します。 
5. 一覧内で **Microsoft.Insights** を見つけ、状態が **[登録済み]** になっていることを確認します。

## <a name="overview-of-azure-monitor-on-azure-stack-hub"></a>Azure Stack Hub 上の Azure Monitor の概要

Azure 上の Azure Monitor と同様、Azure Stack Hub 上の Azure Monitor では、ほとんどのサービスに対して標準となるインフラストラクチャのメトリックおよびログを提供します。

## <a name="azure-monitor-sources-compute-subset"></a>Azure Monitor のソース: コンピューティングのサブセット

![Azure Stack Hub 上の Azure Monitor のソース: コンピューティングのサブセット](media//azure-stack-metrics-azure-data/azs-monitor-computersubset.svg)

Azure Stack Hub 上の **Microsoft.Compute** リソース プロバイダーには、以下が含まれます。
 - 仮想マシン 
 - 仮想マシン スケール セット

### <a name="application---diagnostics-logs-app-logs-and-metrics"></a>アプリケーション - 診断ログ、アプリ ログ、およびメトリック

アプリは、**Microsoft.Compute** リソース プロバイダーで実行されている VM の OS で実行できます。 これらのアプリと VM からは、それぞれに固有のログとメトリックのセットが出力されます。 Azure Monitor は、Azure 診断拡張機能 (Windows または Linux) を利用して、アプリ レベルのメトリックとログの大半を収集します。

メジャーの種類は次のとおりです。
 - パフォーマンス カウンター
 - アプリ ログ
 - Windows イベント ログ
 - .NET イベント ソース
 - IIS ログ
 - マニフェスト ベースの ETW
 - クラッシュ ダンプ
 - カスタム エラー ログ

> [!NOTE]  
> Azure Stack Hub 上の Linux 診断拡張機能はサポートされていません。

### <a name="host-and-guest-vm-metrics"></a>ホスト VM とゲスト VM のメトリック

上記のコンピューティング リソースには、専用のホスト VM とゲスト OS が存在します。 ホスト VM およびゲスト OS は、Hyper-V ハイパーバイザーのルート VM およびゲスト VM に相当します。 ホスト VM とゲスト OS の両方のメトリックを収集できます。 また、ゲスト OS の診断ログを収集することもできます。 Azure Stack Hub 上のホスト VM とゲスト VM に関して収集可能なメトリックの一覧については、[Azure Stack Hub 上の Azure Monitor でサポートされるメトリック](azure-stack-metrics-supported.md)に関するページを参照してください。 

### <a name="activity-log"></a>アクティビティ ログ

計算リソースの情報については、Azure Stack Hub インフラストラクチャで確認できるようなアクティビティ ログを検索できます。 このログには、リソースが作成された時点や破棄された時点などの情報が記載されています。 Azure Stack Hub のアクティビティ ログには、Azure との整合性があります。 詳細については、[Azure のアクティビティ ログの概要](/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)に関するページを参照してください。 


## <a name="azure-monitor-sources-everything-else"></a>Azure Monitor のソース: 上記以外のすべて

![Azure Stack Hub 上の Azure Monitor のソース: 上記以外のすべて](media//azure-stack-metrics-azure-data/azs-monitor-othersubset.svg)

### <a name="resources---metrics-and-diagnostics-logs"></a>リソース - メトリックと診断ログ

収集可能なメトリックと診断ログは、リソースの種類によって異なります。 Azure Stack Hub 上の各リソースの収集可能なメトリックの一覧については、サポートされるメトリックに関する記事を参照してください。 詳細については、[Azure Stack Hub 上の Azure Monitor でサポートされるメトリック](azure-stack-metrics-supported.md)に関するページを参照してください。

### <a name="activity-log"></a>アクティビティ ログ

アクティビティ ログは、コンピューティング リソースの場合と同じです。 

### <a name="uses-for-monitoring-data"></a>監視データの用途

**保存とアーカイブ**  

一部の監視データは既に保存されており、Azure Monitor で一定期間使用できます。 
 - メトリックは 90 日間保存されます。 
 - アクティビティ ログのエントリは 90 日間保存されます。 
 - 診断ログは保存されません。
 - より長期間保存するには、データをストレージ アカウントにアーカイブしてください。

**クエリ**  

Azure Monitor REST API、クロスプラットフォーム コマンドライン インターフェイス (CLI) コマンド、PowerShell コマンドレット、または .NET SDK を使えば、システム ストレージまたは Azure ストレージ内のデータにアクセスできます。 

**視覚化**

グラフィックスやグラフで監視データを視覚化すると、データ自体を見るよりも格段に早く傾向を把握できます。 

視覚化の方法には、次のようなものがあります。
 - Azure Stack Hub ユーザーおよび管理者ポータルを使用する。
 - データを Microsoft Power BI にルーティングする。
 - ライブ ストリーミングを使ってサード パーティ製の視覚化ツールにデータをルーティングするか、ツールに Azure Storage のアーカイブからデータを読み込ませる。

## <a name="methods-of-accessing-azure-monitor-on-azure-stack-hub"></a>Azure Stack Hub 上の Azure Monitor にアクセスする方法

次のいずれかの方法を使用して、データの追跡、ルーティング、および取得を行うことができます。 ただし、アクションやデータの種類によっては利用できない方法もあります。 

 - [Azure Stack Hub ユーザー ポータル ](azure-stack-use-portal.md)
 - [PowerShell](/azure/monitoring-and-diagnostics/insights-powershell-samples)
 - [クロスプラットフォーム コマンド ライン インターフェイス (CLI)](/azure/monitoring-and-diagnostics/insights-cli-samples)
 - [REST API](/rest/api/monitor)
 - [.NET SDK](https://www.nuget.org/packages/Microsoft.Azure.Management.Monitor)

> [!Important]  
> VM のパフォーマンス グラフを表示しているときに **リソースが見つからない** というエラーが発生した場合は、VM に関連付けられているサブスクリプションに対して、Microsoft.insights が登録されていることを確認してください。

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub の監視データの使用](azure-stack-metrics-monitor.md)に関するページを参照し、Azure Stack Hub での監視データの使用の詳細を確認してください。
