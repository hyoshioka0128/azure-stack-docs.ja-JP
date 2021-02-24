---
title: Azure Stack Hub で Event Hubs を管理する方法
description: Azure Stack Hub で Event Hubs リソース プロバイダーを管理する方法について説明します。
author: BryanLa
ms.author: bryanla
ms.service: azure-stack
ms.topic: how-to
ms.date: 12/09/2019
ms.reviewer: jfggdl
ms.lastreviewed: 12/09/2019
ms.openlocfilehash: 9cf3339398516688865f7aab3a696562f4974699
ms.sourcegitcommit: 0983c1f90734b7ea5e23ae614eeaed38f9cb3c9a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/19/2021
ms.locfileid: "98571333"
---
# <a name="how-to-manage-event-hubs-on-azure-stack-hub"></a>Azure Stack Hub で Event Hubs を管理する方法

Event Hubs の管理エクスペリエンスを使用すると、サービスを制御し、その状態とアラートを視覚化することができます。 

## <a name="overview"></a>概要

Event Hubs の管理ページにアクセスするには、次の手順に従います。

1. Azure Stack Hub 管理者ポータルにサインインします。
2. 左側のペインから **[すべてのサービス]** を選択します。
3. 「Event Hubs」と検索し、このサービスを選択します。 Event Hubs サービスが見つからない場合は、まず、リソース プロバイダーを[インストールする](event-hubs-rp-install.md)必要があります。
4. Event Hubs の管理の概要ページが表示されます。 左側のペインには、4 つのセクションがあります。
   - **概要**: 全般ビューと、特定の管理領域へのアクセスを提供します。
   - **アラート**: Event Hubs の重要なアラートおよび警告アラートをすべて表示します。 詳細については、[「アラート」セクション](#alerts)を参照してください。
   - **クォータ**: クォータの作成、更新、削除を行うことができます。 詳細については、[「クォータ」セクション](#quotas)を参照してください。
   - **Event Hubs クラスター**: 構成されているすべてのクラスターの一覧を表示します。 詳細については、[「Event Hubs クラスター」セクション](#event-hubs-clusters)を参照してください。

   [![Event Hubs を管理する](media/event-hubs-rp-manage/1-manage-event-hubs.png)](media/event-hubs-rp-manage/1-manage-event-hubs.png#lightbox)

## <a name="quotas"></a>Quotas (クォータ)

メイン ページで **[クォータ]** を選択すると、クォータを指定する関連プランを含め、使用中のクォータの一覧が表示されます。 
 
[![Event Hubs を管理する - クォータ](media/event-hubs-rp-manage/3-quotas.png)](media/event-hubs-rp-manage/3-quotas.png#lightbox)

Event Hubs で定義されているクォータの種類の詳細については、[クォータの種類](azure-stack-quota-types.md#event-hubs-quota-types)に関する記事を参照してください

## <a name="alerts"></a>警告

Event Hubs リソース プロバイダーでは、次のアラートがサポートされています。
   
| カテゴリ | アラート: | 種類 | 条件 |
|----------|-------|------|-----------|
| パフォーマンス | | | |
| | EventHub-CpuUsage | 警告 | 過去 6 時間の Event Hubs クラスターの CPU 使用率の平均が、50% を超えている場合。 |
| | EventHub-MemoryUsage | 警告 | 過去 6 時間の Event Hubs クラスターの空きメモリ容量の平均が、50% を下回っている場合。 |
| | EventHub-DiskUsage | 警告 | 過去 6 時間の Event Hubs クラスターのデータ ディスク (E:) 使用率の平均が、50% を超えている場合。 |
| 使用量およびクォータ | | | |
| | EventHub-QuotaExceeded | 警告 | 過去 6 時間以内にクォータ超過エラーが発生した場合。 |
| | EventHub-NamespaceCreditUsage | 警告 | 過去 6 時間の名前空間クレジットの使用量の合計が、10000.0 を超えている場合。 |
| サービスの低下 | | | |
| | EventHub-InternalServerError | 警告 | 過去 6 時間以内に内部サーバー エラーが発生した場合。 |
| | EventHub-ServerBusy | 警告 | 過去 6 時間にサーバー ビジー エラーが発生した場合。 |
| Client | | | |
| | EventHub-ClientError | 警告 | 過去 6 時間にクライアント エラーが発生した場合。 |
| リソース | | | |
| | EventHub-PendingDeletingResources | 警告 | 過去 6 時間に保留中のリソース削除の合計が 100 を超えている場合。 |
| | EventHub-ProvisioningQueueLength | 警告 | 過去 6 時間のプロビジョニング キューの長さの平均が 30 を超えている場合。 |

メイン ページで **[アラート]** を選択すると、発行されたアラートの一覧が表示されます。

[![Event Hubs を管理する - アラートの概要](media/event-hubs-rp-manage/2-alerts-summary.png)](media/event-hubs-rp-manage/2-alerts-summary.png#lightbox)

一覧からアラートを選択すると、右側のパネルに **アラートの詳細** が表示されます。

[![Event Hubs を管理する - アラートの詳細](media/event-hubs-rp-manage/2-alerts-detail.png)](media/event-hubs-rp-manage/2-alerts-detail.png#lightbox)

アラートなど、Azure Stack Hub の監視機能の詳細については、[正常性とアラートの監視](azure-stack-monitor-health.md)に関するページを参照してください。 ログ収集の詳細については、[Azure Stack 診断ログの収集の概要](./diagnostic-log-collection.md)に関するページを参照してください。

## <a name="event-hubs-clusters"></a>Event Hubs クラスター

メイン ページで **[Event Hubs クラスター]** を選択すると、使用可能なユーザー クラスターの一覧が表示されます。 この一覧には、各クラスターに関する次の情報が含まれています。

- 大まかな構成情報。
- サービスの正常性。
- バックアップの状態。

[![Event Hubs を管理する - ユーザー リソース](media/event-hubs-rp-manage/4-user-resources.png)](media/event-hubs-rp-manage/4-user-resources.png#lightbox)

**[正常性]** または **[バックアップ]** の下にあるリンクを選択すると、それぞれ Event Hubs の正常性とバックアップの状態に関する詳細情報が表示されます。 **[名前]** の下のリンクには、次のようなクラスターの詳細情報が表示されます。
- 状態と構成情報。
- クラスターのサービス制限の一覧。

[![Event Hubs を管理する - ユーザー クラスター](media/event-hubs-rp-manage/4-user-clusters.png)](media/event-hubs-rp-manage/4-user-clusters.png#lightbox)

サービスの制限は、Event Hubs の運用境界を定義する構成パラメーターです。 使用可能なサービス制限は、Azure Event Hubs Dedicated に用意されているものと似ています。 **[Config value]\(構成値\)** の下にあるリンクを選択すると、割り当てられた値を変更できます。

> [!IMPORTANT]
> サービスの制限を変更する前に、時間をかけてすべての影響を分析する必要があります。 サービスの制限を変更すると、イベントを使用および生成するソリューションの動作が影響を受ける可能性があります。 また、変更すると、Azure Stack の容量からのリソース消費にも影響が及ぶ可能性があります。

## <a name="next-steps"></a>次のステップ

詳細情報:

- Event Hubs に対して定義されているクォータの種類については、[クォータの種類](azure-stack-quota-types.md#event-hubs-quota-types)に関するページを参照してください。
- アラートなど、Azure Stack Hub の監視機能については、[正常性とアラートの監視](azure-stack-monitor-health.md)に関するページを参照してください。 
- Azure Stack Hub のログの収集については、[Azure Stack 診断ログの収集の概要](./diagnostic-log-collection.md)に関するページを参照してください。
