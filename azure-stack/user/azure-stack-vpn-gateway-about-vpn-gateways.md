---
title: Azure Stack Hub の VPN ゲートウェイの作成
description: Azure Stack Hub の VPN ゲートウェイを作成して構成します。
author: sethmanheim
ms.topic: conceptual
ms.date: 06/15/2020
ms.author: sethm
ms.lastreviewed: 05/21/2019
ms.openlocfilehash: e21839e5333a03b1a36322f0c632a2b278da9665
ms.sourcegitcommit: 8790b8a4ecf4421409534df5ff510d537cc000da
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/29/2020
ms.locfileid: "97802016"
---
# <a name="create-vpn-gateways-for-azure-stack-hub"></a>Azure Stack Hub の VPN ゲートウェイの作成

Azure 仮想ネットワークとオンプレミスのサイトとの間でネットワーク トラフィックを送信する前に、仮想ネットワーク用の仮想ネットワーク (VPN) ゲートウェイを作成する必要があります。

VPN ゲートウェイは、パブリック接続で暗号化されたトラフィックを送信する仮想ネットワーク ゲートウェイの一種です。 VPN ゲートウェイを使用して、Azure Stack Hub 内の仮想ネットワークと Azure 内の仮想ネットワークとの間で安全にトラフィックを送信できます。 また、仮想ネットワークと、VPN デバイスに接続されている別のネットワーク間で安全にトラフィックを送信することもできます。

仮想ネットワーク ゲートウェイを作成する場合、作成するゲートウェイの種類を指定する必要があります。 Azure Stack Hub では、仮想ネットワーク ゲートウェイの 1 つの種類である **Vpn** がサポートされています。

各仮想ネットワークに配置できる仮想ネットワーク ゲートウェイは 2 つですが、種類はいずれか 1 つのみになります。 選択する設定によっては、1 つの VPN ゲートウェイへの複数の接続を作成できます。 この種類の設定の一例は、マルチサイト接続構成です。

Azure Stack Hub 用の VPN ゲートウェイを作成して構成する前に、[Azure Stack Hub ネットワークに関する考慮事項](azure-stack-network-differences.md)に関する記事を確認し、Azure Stack Hub と Azure での構成方法の違いを理解してください。

> [!NOTE]
> Azure では、選択する VPN ゲートウェイ SKU の帯域幅スループットが、ゲートウェイに接続されるすべての接続に分配される必要があります。 一方、Azure Stack Hub では、VPN ゲートウェイ SKU の帯域幅値が、ゲートウェイに接続される各接続リソースに適用されます。
>
> 次に例を示します。
>
> * Azure では、Basic VPN ゲートウェイ SKU で、約 100 Mbps の総スループットに対応できます。 その VPN ゲートウェイへの接続を 2 つ作成し、1 つの接続で 50 Mbps の帯域幅を使用する場合、もう 1 つの接続では 50 Mbps を使用できます。
> * Azure Stack Hub では、Basic VPN ゲートウェイ SKU への各接続に 100 Mbps のスループットが割り当てられます。

## <a name="configuring-a-vpn-gateway"></a>VPN ゲートウェイの構成

VPN Gateway 接続は、特定の設定で構成された複数のリソースに依存します。 ほとんどのリソースは個別に構成できますが、一定の順序で構成する必要がある場合があります。

### <a name="settings"></a>設定

リソースごとに選択する設定は、適切な接続を作成するうえで非常に重要です。

VPN Gateway の個々のリソースと設定については、[Azure Stack Hub の VPN ゲートウェイの設定](azure-stack-vpn-gateway-settings.md)に関する記事を参照してください。 この記事は、次のことを理解するのに役立ちます。

* ゲートウェイの種類、VPN の種類、接続の種類。
* ゲートウェイ サブネット、ローカル ネットワーク ゲートウェイ、および考慮する場合がある他のリソースの設定。

### <a name="deployment-tools"></a>デプロイ ツール

Azure Portal などの 1 つの構成ツールを使用して、リソースを作成し、構成できます。 後で、追加のリソースを構成したり、該当する場合に既存のリソースを変更したりするために、PowerShell などの別のツールに切り替えることができます。 現時点では、すべてのリソースとリソースの設定を Azure portal で構成することはできません。 各接続トポロジの記事の手順では、特定の構成ツールが必要な場合が指定されています。

## <a name="connection-topology-diagrams"></a>接続トポロジの図

VPN ゲートウェイ接続では、さまざまな構成を利用できます。 どの構成が自分のニーズに最適かを判断します。 以下のセクションでは、次の VPN ゲートウェイ接続に関する情報とトポロジの図を確認できます。

* 利用可能なデプロイメント モデル
* 利用可能な構成ツール
* 記事に直接移動するリンク (利用可能な場合)

以降のセクションの図と説明は、要件を満たす接続トポロジを選択するために役立ちます。 図は主要なベースライン トポロジを示していますが、図をガイドとして使用して、より複雑な構成を構築することもできます。

## <a name="site-to-site-and-multi-site-ipsecike-vpn-tunnel"></a>サイト間とマルチサイト (IPsec/IKE VPN トンネル)

### <a name="site-to-site"></a>サイト間

"*サイト間*" (S2S) VPN ゲートウェイ接続とは、IPsec/IKE (IKEv2) VPN トンネルを介した接続です。 この種類の接続では、オンプレミスに配置され、パブリック IP アドレスが割り当てられている、VPN デバイスが必要です。 このデバイスは NAT の内側に配置することはできません。 S2S 接続は、クロスプレミスおよびハイブリッド構成に使用できます。

![サイト間 VPN 接続の構成の例](media/azure-stack-vpn-gateway-about-vpn-gateways/vpngateway-site-to-site-connection-diagram.png)

### <a name="multi-site"></a>マルチサイト

"*マルチサイト*" 接続は、サイト間接続の一種です。 仮想ネットワーク ゲートウェイから複数の VPN 接続を作成し、通常は複数のオンプレミスのサイトに接続します。 複数の接続を使用する場合は、(クラシック VNet を使用する際に動的ゲートウェイと呼ばれる) ルートベースの VPN の種類を使用する必要があります。 各仮想ネットワークに配置できる VPN ゲートウェイは 1 つのみであるため、ゲートウェイを経由するすべての接続は、使用可能な帯域幅を共有します。

![Azure VPN Gateway マルチサイト接続の例](media/azure-stack-vpn-gateway-about-vpn-gateways/vpngateway-multisite-connection-diagram.png)

## <a name="gateway-skus"></a>ゲートウェイの SKU

Azure Stack Hub の仮想ネットワーク ゲートウェイを作成する場合、使用するゲートウェイ SKU を指定する必要があります。 以下の VPN ゲートウェイ SKU がサポートされています。

* Basic
* Standard
* 高性能

Basic より Standard、Standard または Basic より High Performance というように、より上位のゲートウェイ SKU を選択すると、より多くの CPU とネットワーク帯域幅がゲートウェイに割り当てられます。 その結果、ゲートウェイは、仮想ネットワークに対してより高いネットワーク スループットをサポートできます。

Azure Stack Hub では、Express Route 専用の Ultra Performance ゲートウェイ SKU はサポートされていません。

SKU を選択する場合、次を考慮してください。

* Azure Stack Hub では、ポリシー ベースのゲートウェイはサポートされていません。
* Basic SKU では、ボーダー ゲートウェイ プロトコル (BGP) はサポートされていません。
* ExpressRoute と VPN ゲートウェイが共存する構成は、Azure Stack Hub ではサポートされていません。

## <a name="gateway-availability"></a>ゲートウェイの可用性

高可用性シナリオは、**ハイ パフォーマンス ゲートウェイ** の接続 SKU 上でのみ構成できます。 アクティブ/アクティブ構成とアクティブ/パッシブ構成の両方で可用性を提供する Azure とは異なり、Azure Stack Hub では、アクティブ/パッシブ構成のみがサポートされています。

### <a name="failover"></a>[フェールオーバー]

Azure Stack Hub には 3 つのマルチテナント ゲートウェイ インフラストラクチャ VM があります。 これらの VM のうち 2 つはアクティブ モードで、3 つ目は冗長モードです。 アクティブな VM ではその上に VPN 接続を作成でき、冗長 VM ではフェールオーバーが発生した場合に VPN 接続のみを受け入れます。 アクティブなゲートウェイ VM が使用できなくなった場合、短い時間 (数秒) 接続が失われた後に、VPN 接続が冗長 VM にフェールオーバーします。

## <a name="estimated-aggregate-tunnel-throughput-by-sku"></a>SKU 別の推定合計トンネル スループット

次の表は、ゲートウェイの種類と、ゲートウェイ SKU 別の各トンネル/接続の推定合計スループットを示しています。

|| トンネル スループット (1) | VPN Gateway の IPsec トンネルの最大数 (2) |
|-------|-------|-------|
|**Basic SKU** **(3)** | 100 Mbps | 20 |
|**Standard SKU** | 100 Mbps | 20 |
|**高性能 SKU** | 200 Mbps | 10 |

### <a name="table-notes"></a>表の注記:

**(1)** - トンネル スループットは、インターネット経由でのクロスプレミス接続では保証されたスループットではありません。 この値は、達成可能な最大スループットです。  
**(2)** - トンネルの最大数は、すべてのサブスクリプションの Azure Stack Hub デプロイごとの合計です。  
**(3)** BGP ルーティングは、Basic SKU ではサポートされていません。

> [!NOTE]
> 2 つの Azure Stack Hub デプロイ間に作成できるのは、1 つのサイト間 VPN 接続だけです。 これは、同じ IP アドレスに対して許容される VPN 接続が 1 つだけであるというプラットフォームの制限によるものです。 Azure Stack Hub では、Azure Stack Hub システム内のすべての VPN Gateway に対して単一のパブリック IP を使用するマルチテナント ゲートウェイが活用されているため、2 つの Azure Stack Hub システム間に存在できる VPN 接続は 1 つだけです。 この制限は、単一の IP アドレスを使用する VPN ゲートウェイに複数のサイト間 VPN 接続を接続する場合にも当てはまります。 Azure Stack Hub では、同じ IP アドレスを使用して複数のローカル ネットワーク ゲートウェイ リソースを作成することはできません。

## <a name="next-steps"></a>次のステップ

* [Azure Stack Hub の VPN ゲートウェイ構成設定](azure-stack-vpn-gateway-settings.md)
