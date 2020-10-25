---
title: サーバーを Azure Virtual Network に接続する - Azure ネットワーク アダプター
description: この記事では、Azure ネットワーク アダプターを使用してサーバーを Azure Virtual Network に接続するための要件と手順について説明します。
ms.topic: article
author: thomasmaurer
ms.author: thmaure
ms.date: 07/14/2020
ms.openlocfilehash: 523ee87a5673a79dbb17c18251fc8837b9d2f7e4
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90573651"
---
# <a name="use-azure-network-adapter-to-connect-a-server-to-an-azure-virtual-network"></a>Azure ネットワーク アダプターを使用してサーバーを Azure Virtual Network に接続する

>適用対象:Windows Server 2019、Windows Server 2016、Windows Server 2012 R2

オンプレミスおよびマルチクラウド環境で実行されている多数のワークロードには、Microsoft Azure で実行されている仮想マシン (VM) への接続が必要です。 サーバーを Azure Virtual Network に接続するには、サイト間 VPN、Azure Express Route、ポイント対サイト VPN など、いくつかのオプションがあります。

Windows Admin Center と Azure ネットワーク アダプターにより、[ポイント対サイト VPN](/azure/vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal) 接続を使用して、サーバーを仮想ネットワークに接続するワンクリック エクスペリエンスが提供されます。 このプロセスにより、仮想ネットワーク ゲートウェイとオンプレミス VPN クライアントの構成が自動化されます。

## <a name="when-to-use-azure-network-adapter"></a>Azure ネットワーク アダプターを使用する場合
Azure ネットワーク アダプターのポイント対サイト VPN 接続は、ブランチ オフィスやストアなど、リモートの場所から仮想ネットワークに接続する場合に便利です。 仮想ネットワークに接続するために数台のサーバーのみが必要な場合は、サイト間 VPN の代わりに Azure ネットワーク アダプターを使用することもできます。 Azure ネットワーク アダプターの接続には、VPN デバイスと公開 IP アドレスは必要ありません。

### <a name="requirements"></a>必要条件
Azure ネットワーク アダプターを使用して仮想ネットワークに接続するには、次のものが必要です。
- 少なくとも 1 つのアクティブなサブスクリプションを持つ Azure アカウント。
- 既存の仮想ネットワーク。
- Azure 仮想ネットワークに接続するターゲット サーバーへのインターネット アクセス。
- Windows Admin Center から Azure への接続。
  詳細については、[Azure 統合の構成](/windows-server/manage/windows-admin-center/azure/azure-integration)に関する記事をご覧ください。
- Windows Admin Center の最新バージョン。
  詳細については、[Windows Admin Center](https://www.microsoft.com/windows-server/windows-admin-center) に関するページをご覧ください。

> [!NOTE]
> Windows Admin Center は、Azure に接続するサーバーにインストールする必要はありません。 ただし、単一サーバーのシナリオで行うことはできます。

## <a name="add-an-azure-network-adapter-to-a-server"></a>Azure ネットワーク アダプターをサーバーに追加する
Azure ネットワーク アダプターを構成するには、Windows Admin Center でそれのネットワーク拡張機能にアクセスします。

Windows Admin Center で、次のようにします。
1. Azure ネットワーク アダプターに追加する VM がホストされているサーバーに移動します。
1. **[ツール]** の下で、 **[ネットワーク]** を選択します。
1. **[Add Azure Network Adapter]\(Azure ネットワーク アダプターの追加\)** を選択します。
1. **[Add Azure Network Adapter]\(Azure ネットワーク アダプターの追加\)** ペインで、次の必須情報を入力し、 **[作成]** を選択します。
    - **サブスクリプション**
    - **場所**
    - **Virtual Network**
    - **ゲートウェイ サブネット** (存在しない場合)
    - **ゲートウェイ SKU** (存在しない場合)
    - **クライアント アドレス空間**

        クライアント アドレス プールとは、指定するプライベート IP アドレスの範囲です。 ポイント対サイト VPN 経由で接続するクライアントは、この範囲内の IP アドレスを動的に受け取ります。 接続元であるオンプレミスの場所、または接続先とする仮想ネットワークと重複しないプライベート IP アドレス範囲を使用します。 プライベート ネットワーク用に指定されている範囲 (10.x.x.x、192.168.x.x、または 172.16.0.0 から 172.31.255.255) の IP アドレスを使用することをお勧めします。

    - **認証証明書**

        証明書は、ポイント対サイト VPN 接続を介して仮想ネットワークに接続するクライアントを認証するために、Azure によって使用されます。 ルート証明書の公開キー情報を Azure にアップロードします。 ルート証明書は、仮想ネットワークへのポイント対サイト接続のために、Azure によって "信頼済み" と見なされます。 クライアント証明書が、信頼済みのルート証明書から生成され、クライアント サーバーにインストールされている必要があります。 クライアント証明書は、クライアントで仮想ネットワークへの接続を開始するときに、そのクライアントを認証するために使用されます。
    
        詳細については、「[ネイティブ Azure 証明書認証を使用した VNet へのポイント対サイト VPN 接続の構成:Azure portal](/azure/vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal)」の「認証の種類の構成」セクションを参照してください。

    :::image type="content" source="media/azure-network-adapter/add-azure-network-adapter.png" alt-text="Windows Admin Center の [Add Azure Network Adapter]\(Azure ネットワーク アダプターの追加\) ペイン。":::

> [!NOTE]
> 仮想ネットワーク内で実行される VPN Gateway や Application Gateway などのネットワーク アプライアンスには、追加コストがかかります。 詳細については、[Virtual Network の価格](https://azure.microsoft.com/pricing/details/virtual-network/)に関するページをご覧ください。

既存の Azure Virtual Network ゲートウェイが存在しない場合は、Windows Admin Center によって作成されます。 この設定プロセスには最大 25 分かかることがあります。 Azure ネットワーク アダプターを作成した後は、サーバーから直接、仮想ネットワーク内の VM へのアクセスを開始できます。

接続が不要になった場合は、 **[ネットワーク]** の下で、切断する Azure ネットワーク アダプターを選択し、上部のメニューから **[切断]** を選択します。次に、 **[Disconnect VPN Confirmation]\(VPN 切断の確認\)** ポップアップ ウィンドウで、 **[はい]** を選択します。

## <a name="next-steps"></a>次のステップ
Azure Virtual Network の詳細については、次の項目も参照してください。

- [Azure 仮想ネットワークについてよく寄せられる質問 (FAQ)](/azure/virtual-network/virtual-networks-faq)
