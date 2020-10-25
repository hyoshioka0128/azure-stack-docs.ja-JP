---
title: Azure Stack Hub に高可用性ネットワーク仮想アプライアンスをデプロイする
description: Azure Stack Hub に高可用性ネットワーク仮想アプライアンスをデプロイする方法について説明します。
author: mattbriggs
ms.topic: how-to
ms.date: 08/24/2020
ms.author: mabrigg
ms.reviewer: kivenkat
ms.lastreviewed: 11/01/2019
ms.openlocfilehash: 31635f6ecfefc0c513fddec5ec00da1006b44f44
ms.sourcegitcommit: 53b0dde60a6435936a5e0cb9e931245f262d637a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/23/2020
ms.locfileid: "91107007"
---
# <a name="deploy-highly-available-network-virtual-appliances-on-azure-stack-hub"></a>Azure Stack Hub に高可用性ネットワーク仮想アプライアンスをデプロイする

この記事では、高可用性のネットワーク仮想アプライアンス (NVA) セットを Azure Stack Hub にデプロイする方法を示します。 NVA は、通常は、境界ネットワーク (DMZ とも呼ばれます) から他のネットワークまたはサブネットへのネットワーク トラフィックのフローを制御するために使用されます。 この記事には、イングレスのみ、エグレスのみ、イングレスとエグレスの両方を行うアーキテクチャの例が含まれています。

[Azure Stack Hub Marketplace](../operator/azure-stack-marketplace-azure-items.md) では、さまざまなベンダーからの NVA を利用することができ、そのうちの 1 つを使用することで最適なパフォーマンスが得られます。

このアーキテクチャには次のコンポーネントがあります。

## <a name="networking-and-load-balancing"></a>ネットワークと負荷分散

-   **仮想ネットワークとサブネット**。 すべての Azure VM が、サブネットにセグメント化できる仮想ネットワーク内にデプロイされます。 階層ごとに個別のサブネットを作成します。

-   **第 7 層のロード バランサー。** Azure Stack Hub では Application Gateway がまだ利用できないため、[Azure Stack Hub Marketplace](../operator/azure-stack-marketplace-azure-items.md) には代替手段が用意されています ([KEMP LoadMaster Load Balancer ADC Content Switch](https://azuremarketplace.microsoft.com/marketplace/apps/kemptech.vlm-azure)/ [f5 Big-IP Virtual Edition](https://azuremarketplace.microsoft.com/marketplace/apps/f5-networks.f5-big-ip-best) または [A10 vThunder ADC](https://azuremarketplace.microsoft.com/marketplace/apps/a10networks.vthunder-414-gr1) など)。

-   **ロード バランサー**。 [Azure Load Balancer](/azure/load-balancer/load-balancer-overview) は、Web 層からビジネス層へ、ビジネス層から SQL Server へとネットワーク トラフィックを分散するために使用します。

-   **ネットワーク セキュリティ グループ** (NSG)。 NSG を使用して、仮想ネットワーク内のネットワーク トラフィックを制限します。 たとえば、ここに示されている 3 層アーキテクチャでは、データベース層は、Web フロントエンドからではなく、管理サブネットからのトラフィックのみを受信します。

-   **UDR。** ["*ユーザー定義ルート*"](/azure/virtual-network/virtual-networks-udr-overview/) (UDR) を使用して、トラフィックを特定のロード バランサーにルーティングします。

この記事では、Azure Stack Hub ネットワークを基本的に理解していることを前提としています。

## <a name="architecture-diagrams"></a>アーキテクチャ図

NVA は、さまざまなアーキテクチャの境界ネットワークにデプロイできます。 たとえば、次の図では、イングレス用の単一の NVA の使用が示されています。

![イングレス用の単一の NVA の使用を示すスクリーンショット。](./media/iaas-architecture-nva-architecture/iaas-architecture-nva-architecture-image1.svg)

このアーキテクチャでは、NVA は、すべての着信ネットワーク トラフィックと発信ネットワーク トラフィックをチェックし、ネットワーク セキュリティ ルールを満たしているトラフィックのみを渡すことによって、セキュリティで保護されたネットワーク境界を提供します。 すべてのネットワーク トラフィックが NVA を通過する必要があるという事実は、NVA がネットワークの単一障害点であることを意味します。 この NVA で障害が発生した場合、ネットワーク トラフィック用の他のパスが存在しないため、すべてのバックエンド サブネットが使用不能になります。

NVA に高可用性を持たせるには、複数の NVA を可用性セットにデプロイします。

以下のアーキテクチャは、高可用性の NVA で必要なリソースと構成について説明しています。

| 解決策 | メリット | 考慮事項 |
| --- | --- | --- |
| 第 7 層で NVA を使用するイングレス | すべての NVA ノードがアクティブ。 | 接続を終了し SNAT を使用できる NVA が必要。<br>エンタープライズ ネットワークまたはインターネットからのトラフィックと Azure Stack Hub からのトラフィックに対して別個の NVA セットが必要。<br>Azure Stack Hub の外部から発信されるトラフィックに対してのみ使用可能。  |
| 第 7 層で NVA を使用するエグレス | すべての NVA ノードがアクティブ。 | 接続を終了し、ソース ネットワーク アドレス変換 (SNAT) を実装できる NVA が必要。 |
| 第 7 層で NVA を使用するイングレスとエグレス | すべてのノードがアクティブ。<br>Azure Stack Hub で発信されたトラフィックを処理可能。 | 接続を終了し SNAT を使用できる NVA が必要。<br>エンタープライズ ネットワークまたはインターネットからのトラフィックと Azure Stack Hub からのトラフィックに対して別個の NVA セットが必要。 |

## <a name="ingress-with-layer-7-nvas"></a>第 7 層で NVA を使用するイングレス

次の図は、インターネットに接続するロード バランサーの背後でイングレス境界ネットワークを実装する高可用性アーキテクチャを示しています。 このアーキテクチャは、HTTP や HTTPS などの第 7 層のトラフィックで、Azure Stack Hub ワークロードへの接続を提供するように設計されています。

![自動生成されたマップ説明のスクリーンショット](./media/iaas-architecture-nva-architecture/iaas-architecture-nva-architecture-image2.svg)

このアーキテクチャの利点は、すべての NVA がアクティブであることであり、片方の NVA で障害が発生すると、ロード バランサーが他方の NVA にネットワーク トラフィックを送信します。 両方の NVA がトラフィックを内部ロード バランサーにルーティングするため、1 つの NVA がアクティブである限り、トラフィックが停止することはありません。 Web 層の VM を対象とする SSL トラフィックを終了するには、NVA が必要です。 エンタープライズ ネットワークのトラフィックには独自のネットワーク ルートを持つ NVA の別の専用セットが必要であるため、エンタープライズ ネットワークのトラフィックを処理することを目的としてこれらの NVA を拡張することはできません。

## <a name="egress-with-layer-7-nvas"></a>第 7 層で NVA を使用するエグレス

第 7 層で NVA を使用するイングレスを拡張して、Azure Stack Hub ワークロードから発信される要求に対して、エグレス境界ネットワークを提供できます。 次のアーキテクチャは、HTTP や HTTPS などの第 7 層のトラフィックで、境界ネットワーク内の高可用性を持つ NVA を提供するように設計されています。

![自動的に生成された携帯電話の説明のスクリーンショット](./media/iaas-architecture-nva-architecture/iaas-architecture-nva-architecture-image4.svg)

このアーキテクチャでは、Azure Stack Hub から発信されるすべてのトラフィックが内部ロード バランサーにルーティングされます。 ロード バランサーは、発信要求を NVA セットに分散します。 これらの NVA は、それぞれのパブリック IP アドレスを使用して、インターネットにトラフィックを送信します。

## <a name="ingress-egress-with-layer-7--nvas"></a>第 7 層で NVA を使用するイングレスとエグレス

イングレスとエグレスという 2 つのアーキテクチャでは、イングレス用とエグレス用に別個の境界ネットワークがありました。 次のアーキテクチャは、HTTP や HTTPS などの第 7 層のトラフィックのイングレスとエグレスの両方で使用できる境界ネットワークの作成方法を示しています。

![自動的に生成されたソーシャル メディアの投稿についての説明のスクリーンショット](./media/iaas-architecture-nva-architecture/iaas-architecture-nva-architecture-image4.svg)

第 7 層で NVA を使用するイングレスとエグレスのアーキテクチャでは、その NVA によって第 7 層のロード バランサーからの受信要求が処理されます。 NVA は、ロード バランサーのバックエンド プール内のワークロード VM から発信された要求も処理します。 受信トラフィックは第 7 層のロード バランサーによってルーティングされ、発信トラフィックは SLB (Azure Stack Hub Basic Load Balancer) によってルーティングされるため、NVA はセッション アフィニティも担当します。 つまり、着信要求と発信要求のマッピングは第 7 層のロード バランサーによって管理されるため、最初の要求元に適切な応答を転送できます。 ただし、内部ロード バランサーは第 7 層のロード バランサーのマッピングにアクセスできないため、独自のロジックを使用して NVA への応答を送信します。 ロード バランサーは、第 7 層のロード バランサーから要求を受信していない NVA に応答を送信する場合があります。 この場合、適切な NVA が第 7 層のロード バランサーに応答を転送できるように、NVA 間で通信を行って応答を転送する必要があります。

> [!NOTE]  
> NVA がインバウンドのソース ネットワーク アドレス変換 (SNAT) を確実に実行することで、非対称ルーティングの問題を解決することもできます。 これにより、要求元のソース IP が、インバウンド フローで使用される NVA のいずれかの IP アドレスに置き換えられます。 これで、ルートの対称性を維持しながら、一度に複数の NVA を使用できます。

## <a name="next-steps"></a>次のステップ

- Azure Stack Hub VM の詳細については、「[Azure Stack Hub VM の機能](azure-stack-vm-considerations.md)」を参照してください。  
- Azure のクラウド パターンの詳細については、「[Cloud Design Pattern (クラウド設計パターン)](/azure/architecture/patterns)」を参照してください。
