---
title: Azure Stack Hub でのリージョンの管理
titleSuffix: Azure Stack Hub
description: Azure Stack Hub でのリージョンの管理の概要です。
author: sethmanheim
ms.topic: article
ms.date: 08/27/2020
ms.author: sethm
ms.reviewer: efemmano
ms.lastreviewed: 11/27/2019
ms.openlocfilehash: e40d79dcac58035172d4e292461fd0950ee77979
ms.sourcegitcommit: 03aad17afe8519536066c735c59ad1bdfe8de083
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/27/2020
ms.locfileid: "89041640"
---
# <a name="region-management-in-azure-stack-hub"></a>Azure Stack Hub でのリージョンの管理

Azure Stack Hub では、Azure Stack Hub インフラストラクチャを構成するハードウェア リソースから成る論理エンティティである "*リージョン*" という概念が使用されています。 リージョン管理では、Azure Stack Hub インフラストラクチャを正常に運用するために必要なすべてのリソースを見つけることができます。

1 つの統合システム デプロイ ("*Azure Stack Hub クラウド*" と呼ばれます) が 1 つのリージョンを構成します。 各 Azure Stack Development Kit (ASDK) には、**local** という名前の 1 つのリージョンがあります。 2 番目の Azure Stack Hub 統合システムをデプロイするか、別のハードウェア上に ASDK の別のインスタンスを設定すると、この Azure Stack Hub クラウドは別のリージョンになります。

## <a name="information-available-through-the-region-management-tile"></a>[リージョンの管理] タイルから使用可能な情報

Azure Stack Hub には、 **[リージョンの管理]** タイルで使用できる一連のリージョン管理機能があります。 このタイルは、管理者ポータルの既定のダッシュボードで Azure Stack Hub オペレーターが使用できます。 この画面では、Azure Stack Hub リージョンと、リージョンに固有なそのコンポーネントを監視および更新できます。

![Azure Stack Hub 管理者ポータルの [リージョンの管理] タイル](media/azure-stack-region-management/image1.png)

**[Region management]\(リージョン管理\)** タイル内のリージョンを選択すると、次の情報にアクセスできます。

[![Azure Stack Hub 管理者ポータルの [リージョンの管理] ブレードにあるペインの説明](media/azure-stack-region-management/regionssm.png "Azure Stack Hub 管理者ポータルの [リージョンの管理] ブレード")](media/azure-stack-region-management/regions.png#lightbox)

1. **リソース メニュー**:さまざまなインフラストラクチャ管理領域にアクセスして、ストレージ アカウントや仮想ネットワークなどのユーザー リソースを表示および管理します。

2. **アラート**:システム全体のアラートの一覧と、各アラートの詳細が表示されます。

3. **更新プログラム**:Azure Stack Hub インフラストラクチャの現在のバージョン、利用可能な更新プログラム、および更新履歴が表示されます。 統合システムを更新することもできます。

4. **リソース プロバイダー**:Azure Stack Hub の実行に必要なコンポーネントによって提供されるユーザー機能を管理します。 リソース プロバイダーごとに管理エクスペリエンスが異なります。 このエクスペリエンスには、プロバイダー固有のアラート、メトリック、およびリソース プロバイダー固有のその他の管理機能が含まれます。

5. **インフラストラクチャ ロール**:Azure Stack Hub の実行に必要なコンポーネントです。 アラートをレポートするインフラストラクチャ ロールのみが一覧表示されます。 ロールを選択すると、ロールおよびそのロールが実行されているロール インスタンスに関連付けられているアラートを表示できます。

6. **[プロパティ]** :リージョン管理ブレードへの環境の登録状態と詳細。 状態には、 **[登録済み]** 、 **[未登録]** または **[有効期限切れ]** があります。 登録済みである場合は、Azure Stack Hub の登録に使用した Azure サブスクリプション ID が、登録のリソース グループおよび名前と共に表示されます。

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub での正常性およびアラートの監視](azure-stack-monitor-health.md)
- [Azure Stack Hub での更新プログラム管理](azure-stack-updates.md)
