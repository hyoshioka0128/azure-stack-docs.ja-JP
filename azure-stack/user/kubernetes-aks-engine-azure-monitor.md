---
title: Azure Stack Hub で Azure Monitor for containers を使用する
description: Azure Stack Hub で Azure Monitor for containers を使用する方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 9/2/2020
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 9/2/2020
ms.openlocfilehash: 3535a60a7689250e88b9839a071265d8f28eb92a
ms.sourcegitcommit: b80d529ff47b15b8b612d8a787340c7b0f68165b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2020
ms.locfileid: "89472785"
---
# <a name="use-azure-monitor-for-containers-on-azure-stack-hub"></a>Azure Stack Hub で Azure Monitor for containers を使用する

[Azure Monitor](/azure/azure-monitor/) for containers を使用して、AKS エンジンによって Azure Stack Hub にデプロイされた Kubernetes クラスター内のコンテナーを監視できます。 

> [!IMPORTANT]
> Azure Stack Hub 上のコンテナー用 Azure Monitor は、現在パブリック プレビュー段階です。
> このプレビュー バージョンはサービス レベル アグリーメントなしで提供されています。運用環境のワークロードに使用することはお勧めできません。 特定の機能はサポート対象ではなく、機能が制限されることがあります。 詳しくは、[Microsoft Azure プレビューの追加使用条件](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)に関するページをご覧ください。

Azure Monitor では、Kubernetes で使用可能なコントローラー、ノード、コンテナーから Metrics API 経由でメモリやプロセッサのメトリックを収集することにより、コンテナーのパフォーマンスを確認できます。 また、サービスではコンテナーのログが収集されます。 これらのログを使用して、Azure からオンプレミス クラスターでの問題を診断できます。 Kubernetes クラスターからの監視を設定すると、これらのメトリックとログが自動的に収集されます。 Linux 用の Azure Monitor Log Analytics エージェントのコンテナー化されたバージョンにより、ログが収集されます。 Azure Monitor を使うと、Azure サブスクリプションでアクセスできる Log Analytics ワークスペースに、メトリックとログが格納されます。

クラスターで Azure Monitor を有効にするには、2 つの方法があります。 どちらの方法でも、Azure で Azure Monitor Log Analytics ワークスペースを設定する必要があります。

## <a name="prerequisites"></a>前提条件

どちらの方法でも、「[Azure Monitor – コンテナー](https://github.com/Helm/charts/tree/master/incubator/azuremonitor-containers)」に記載されている[前提条件](https://github.com/Helm/charts/tree/master/incubator/azuremonitor-containers#pre-requisites)が必要です。

## <a name="method-one"></a>方法 1

[Helm](https://helm.sh/) チャートを使用して、クラスターに監視エージェントをインストールすることもできます。 記事「[Azure Monitor – コンテナー](https://github.com/Helm/charts/tree/master/incubator/azuremonitor-containers)」の手順に従ってください。

## <a name="method-two"></a>方法 2

AKS エンジン クラスター仕様の JSON ファイルで、**アドオン**を指定できます。 このファイルは API モデルとも呼ばれます。 このアドオンで、監視情報が格納される Azure Log Analytics ワークスペースの **WorkspaceGUID** と **WorkspaceKey** の base64 エンコード バージョンを提供します。

Azure Stack Hub クラスターに対してサポートされる API 定義については、[kubernetes-container-monitoring_existing_workspace_id_and_key.json](https://github.com/Azure/aks-engine/blob/master/examples/addons/container-monitoring/kubernetes-container-monitoring_existing_workspace_id_and_key.json) の例をご覧ください。 具体的には、**kubernetesConfig** の **addons** プロパティを探します。

```JSON  
 "orchestratorType": "Kubernetes",
       "kubernetesConfig": {
         "addons": [
           {
             "name": "container-monitoring",
             "enabled": true,
             "config": {
               "workspaceGuid": "<Azure Log Analytics Workspace Guid in Base-64 encoded>",
               "workspaceKey": "<Azure Log Analytics Workspace Key in Base-64 encoded>"
             }
           }
         ]
       }
```

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub 上の AKS エンジン](azure-stack-kubernetes-aks-engine-overview.md)を確認してください  
- 「[コンテナーに対する Azure Monitor の概要](/azure/azure-monitor/insights/container-insights-overview)」を読む
