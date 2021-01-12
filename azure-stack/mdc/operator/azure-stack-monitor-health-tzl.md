---
title: Azure Stack での正常性およびアラートの監視 - MDC | Microsoft Docs
description: Modular Data Center (MDC) の Azure Stack で正常性およびアラートを監視する方法について説明します。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/26/2020
ms.author: mabrigg
ms.lastreviewed: 10/26/2020
ms.openlocfilehash: bdc027e8c61053fddddd5ad365eff8069acb840d
ms.sourcegitcommit: d719f148005e904fa426a001a687e80730c91fda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/06/2021
ms.locfileid: "97910824"
---
# <a name="monitor-health-and-alerts-in-azure-stack---modular-data-center-mdc"></a>Azure Stack での正常性およびアラートの監視 - Modular Data Center (MDC)

Azure Stack には、ユーザーが Azure Stack リージョンの正常性とアラートを表示できるようにするインフラストラクチャ監視機能が含まれています。 **[リージョン管理]** タイルには、Azure Stack のすべてのデプロイ済みリージョンが一覧表示されます。 それは、既定では、既定のプロバイダー サブスクリプション用の管理者ポータルに固定されています。 タイルには、アクティブな重大アラートおよび警告アラートの数がリージョンごとに表示されます。 このタイルは、Azure Stack の正常性とアラートの機能へのエントリ ポイントです。

![Azure Stack 管理者ポータルの [リージョン管理] タイル](media/azure-stack-monitor-health/region-management.png)

## <a name="understand-health-in-azure-stack"></a>Azure Stack の正常性について

正常性リソース プロバイダーでは、正常性とアラートが管理されます。 Azure Stack のインフラストラクチャ コンポーネントは、Azure Stack のデプロイおよび構成時に正常性リソース プロバイダーに登録されます。 この登録により、各コンポーネントの正常性とアラートの表示が有効になります。 Azure Stack の正常性の概念は単純です。 コンポーネントの登録済みインスタンスのアラートが存在する場合、そのコンポーネントの正常性の状態は、アクティブなアラートの最大の重大度、つまり警告または重大を反映します。

## <a name="alert-severity-definition"></a>アラートの重大度の定義

Azure Stack では、アラートは、2 つの重大度(**警告** または **重大**) でのみ生成されます。

- **警告**  
  オペレーターは、予定された方法で警告アラートに対処できます。 このアラートは、通常、ユーザーのワークロードに影響を及ぼしません。

- **[重大]**  
  オペレーターは、重大アラートには緊急で対応する必要があります。 これらのアラートは、Azure Stack ユーザーに現在影響を及ぼしているか、まもなく影響を及ぼす問題があることを示しています。


## <a name="view-and-manage-component-health-state"></a>コンポーネントの正常性状態の表示および管理

コンポーネントの正常性の状態は、管理者ポータルのほか、Rest API および PowerShell で確認できます。

ポータルで正常性の状態を表示するには、**[Region management]** \(リージョン管理) タイルで表示するリージョンをクリックします。 インフラストラクチャ ロールとリソース プロバイダーの正常性の状態を表示できます。

![インフラストラクチャ ロールのリスト](media/azure-stack-monitor-health/roles.png)

リソース プロバイダーまたはインフラストラクチャ ロールをクリックすると、詳細情報を表示できます。

> [!WARNING]  
> インフラストラクチャ ロールとロール インスタンスを順にクリックすると、**[Start]\(起動\)**、**[Restart]\(再起動\)**、または **[Shutdown]\(シャットダウン\)** のオプションが表示されます。 統合システムに更新プログラムを適用する場合は、これらのアクションを使用しないでください。 <!-- TZLASDKFIXAlso, do **not** use these options in an Azure Stack Development Kit (ASDK) environment. These options are only designed for an integrated systems environment, where there's more than one role instance per infrastructure role. Restarting a role instance (especially AzS-Xrp01) in the ASDK causes system instability.--> トラブルシューティングの支援のため、問題を [Azure Stack フォーラム](https://aka.ms/azurestackforum)に投稿してください。
>

## <a name="view-alerts"></a>アラートを表示する

各 Azure Stack リージョンのアクティブなアラートのリストは、**[Region management]** \(リージョン管理) ブレードから直接使用できます。 既定の構成の最初のタイルは **[Alerts]** \(アラート) タイルで、リージョンの重大アラートと警告アラートの概要が表示されます。 アラート タイルは、このブレードの他のタイルと同様、ダッシュボードに固定してすばやくアクセスできます。

![Azure Stack 管理者ポータルの警告を表示している [アラート] タイル](media/azure-stack-monitor-health/alerts.png)

 リージョンのすべてのアクティブなアラートの一覧を表示するには、**[アラート]** タイルの上部を選択します。 フィルター処理されたアラートの一覧 ([重大] または [警告]) を表示するには、タイル内の行項目 (**[重大]** または **[警告]**) のいずれかを選択します。

**[Alerts]\(アラート\)** ブレードは、状態 ([Active]\(アクティブ\) または [Closed]\(終了\)) と重大度 ([Critical]\(重大\) または [Warning]\(警告\)) の両方でフィルター処理する機能をサポートしています。 既定のビューには、すべてのアクティブなアラートが表示されます。 終了したアラートはすべて、7 日後にシステムから削除されます。

>[!Note]
>アラートはアクティブなままだが、1日の間に更新されていない場合は、問題が報告されていなければ、[Test-AzureStack](../../operator/azure-stack-diagnostic-test.md)を実行してアラートを閉じることができます。

![Azure Stack 管理者ポータルで [重大] または [警告] 状態でフィルター処理するためのフィルター ウィンドウ](media/azure-stack-monitor-health/alert-view.png)

また、**[View API]\(APIの表示\)** アクションには、リスト ビューの生成に使用された REST API が表示されます。 このアクションにより、アラートの照会に使用できる REST API 構文をすばやく理解できます。 この API は、自動化、または既存のデータ センターの監視、レポート、およびチケット発行ソリューションとの統合に使用できます。

特定のアラートをクリックすると、アラートの詳細を表示できます。 アラートの詳細には、アラートに関連付けられているすべてのフィールドが表示され、影響を受けているコンポーネントとアラートのソースにすばやく移動できます。 たとえば、インフラストラクチャ ロール インスタンスの 1 つがオフラインになるか、アクセスできない場合、次のアラートが発生します。 アラートの詳細の **[Remediation]\(修復\)** セクションにある [相互参照リンク](aks-refer-table-tzl.md)を参照して、この記事を確認できます。

![Azure Stack 管理者ポータルの [アラートの詳細] ブレード](media/azure-stack-monitor-health/alert-detail.png)

## <a name="repair-alerts"></a>アラートの修復

一部のアラートでは、**[修復]** を選択できます。

選択すると、**[修復]** アクションによって、そのアラートに固有の手順が実行されて問題の解決が試みられます。 選択すると、 **[修復]** アクションの状態がポータルの通知として表示されます。

![アラート修復アクションが進行中](media/azure-stack-monitor-health/repair-in-progress.png)

同じポータル通知ブレードで、**[修復]** アクションの正常完了またはアクションの失敗がレポートされます。  アラートの修復アクションが失敗した場合は、アラートの詳細から **[修復]** アクションを再実行することができます。 **[修復]** アクションが正常に完了した場合は、再実行 **しないでください**。

![修復アクションが正常に完了](media/azure-stack-monitor-health/repair-completed.png)

インフラストラクチャ ロール インスタンスがオンラインに戻ると、このアラートは自動的に閉じます。 根本的な問題が解決されると、多くのアラートは自動的に閉じますが、すべてのアラートがそうなるわけではありません。 [修復] アクションのボタンが用意されているアラートは、Azure Stack によって問題が解決されると自動的に閉じます。 その他すべてのアラートについては、修復手順を実行した後、**[アラートを閉じる]** を選択してください。 問題が解決しなかった場合は、Azure Stack で新しいアラートが生成されます。 問題が解決した場合は、アラートが閉じたままとなり、それ以上の手順は不要となります。

## <a name="next-steps"></a>次のステップ

[Azure Stack での更新の管理](../../operator/azure-stack-updates.md)

[Azure Stack でのリージョン管理](../../operator/azure-stack-region-management.md)
