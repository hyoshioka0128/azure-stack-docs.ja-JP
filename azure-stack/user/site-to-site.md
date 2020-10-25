---
title: Azure Stack Hub でのサイト間 VPN 接続のトラブルシューティング
description: オンプレミス ネットワークと Azure Stack Hub 仮想ネットワークの間のサイト間 VPN 接続を構成した後に実行できるトラブルシューティング手順。
author: sethmanheim
ms.author: sethm
ms.date: 10/01/2020
ms.topic: article
ms.reviewer: sranthar
ms.lastreviewed: 05/12/2020
ms.openlocfilehash: 7d4fc17759365434f6179229674b0bb21137f58b
ms.sourcegitcommit: a1e2003fb9c6dacdc76f97614ff5a26a5b197b49
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/01/2020
ms.locfileid: "91623338"
---
# <a name="troubleshoot-site-to-site-vpn-connections"></a>サイト間 VPN 接続のトラブルシューティング

この記事では、オンプレミスのネットワークと Azure Stack Hub 仮想ネットワークの間にサイト間 (S2S) VPN 接続を構成した後、接続が突然動作を停止して再接続できなくなったときのトラブルシューティング手順について説明します。

この記事で Azure Stack Hub の問題に対処できない場合、[Azure Stack Hub MSDN フォーラム](https://social.msdn.microsoft.com/Forums/azure/home?forum=azurestack)を参照してください。

Azure サポート要求を送信することもできます。 [Azure Stack Hub サポート](../operator/azure-stack-manage-basics.md#where-to-get-support)を参照してください。

> [!NOTE]
> 2 つの Azure Stack Hub デプロイ間に作成できるのは、1 つのサイト間 VPN 接続だけです。 これは、同じ IP アドレスに対して許容される VPN 接続が 1 つだけであるというプラットフォームの制限によるものです。 Azure Stack Hub では、Azure Stack Hub システム内のすべての VPN Gateway に対して単一のパブリック IP を使用するマルチテナント ゲートウェイが活用されているため、2 つの Azure Stack Hub システム間に存在できる VPN 接続は 1 つだけです。 この制限は、単一の IP アドレスを使用する VPN ゲートウェイに複数のサイト間 VPN 接続を接続する場合にも当てはまります。 Azure Stack Hub では、同じ IP アドレスを使用して複数のローカル ネットワーク ゲートウェイ リソースを作成することはできません。

## <a name="initial-troubleshooting-steps"></a>初回のトラブルシューティングの手順

[1910 ビルドより](../user/azure-stack-vpn-gateway-settings.md#ike-phase-1-main-mode-parameters)、IPsec/IKEV2 の Azure Stack Hub の既定のパラメーターが変更されました。ビルド バージョンの詳細については、Azure Stack Hub オペレーターにお問い合わせください。

> [!IMPORTANT]
> S2S トンネルを使用する場合、パケットは追加のヘッダーでさらにカプセル化されます。 このカプセル化により、パケットの全体的なサイズが増加します。 これらのシナリオでは、TCP **MSS** を **1350** で固定する必要があります。 お使いの VPN デバイスで MSS クランプがサポートされていない場合、代わりにトンネル インターフェイスの MTU を **1400** バイトに設定できます。 詳細については、[仮想ネットワークの TCPIP パフォーマンス チューニング](/azure/virtual-network/virtual-network-tcpip-performance-tuning)に関する記事を参照してください。

- VPN 構成がルートベース (IKEv2) であることを確認します。 Azure Stack Hub では、ポリシーベース (IKEv1) 構成はサポートされていません。

- 使っている [VPN デバイスとオペレーティング システム バージョンが検証済み](/azure/vpn-gateway/vpn-gateway-about-vpn-devices#devicetable)であるかどうかを確認します。 検証済みの VPN デバイスではない場合、互換性の問題があるかどうかをデバイスの製造元に問い合わせてください。

- Azure Stack Hub 仮想ネットワークとオンプレミス ネットワークの間に重複する IP 範囲がないことを確認します。 これにより、接続の問題が発生する可能性があります。 

- VPN ピア IP を確認する

  - Azure Stack Hub の **[ローカル ネットワーク ゲートウェイ]** オブジェクトの IP 定義が、オンプレミス デバイスの IP と一致している必要があります。

  - オンプレミス デバイスに設定されている Azure Stack Hub ゲートウェイの IP 定義が、Azure Stack Hub ゲートウェイの IP と一致している必要があります。

## <a name="status-not-connected---intermittent-disconnects"></a>状態 "未接続" - 断続的な切断

- オンプレミスの VPN デバイスと AzSH 仮想ネットワーク VPN の共有キーを比較し、両者が一致することを確認します。 AzSH VPN 接続の共有キーを確認するには、次のいずれかの方法を使います。

  - **Azure Stack Hub テナント ポータル**:作成した VPN ゲートウェイ サイト間接続に移動します。 **[設定]** セクションで **[共有キー]** を選択します。

      :::image type="content" source="media/site-to-site/vpn-connection.png" alt-text="VPN 接続":::

  - **Azure PowerShell**:次の PowerShell コマンドを使用します。

      ```powershell
      Get-AzureRMVirtualNetworkGatewayConnectionSharedKey -Name <Connection name> -ResourceGroupName <Resource group>
      ```

## <a name="status-connected--traffic-not-flowing"></a>状態 "接続済み" - トラフィック フローなし

- ゲートウェイ サブネットのユーザー定義ルート (UDR) とネットワーク セキュリティ グループ (NSG) があれば削除し、その結果をテストします。 問題が解決した場合は、適用されている UDR または NSG の設定を確認します。

   ゲートウェイ サブネット上のユーザー定義ルートによって、制限されるトラフィックと許可されるトラフィックが生じることがあります。 その場合、一部のトラフィックについてだけ VPN 接続が不安定で、他のトラフィックについては接続状態が良好であるように見えます。

- オンプレミス VPN デバイスの外部インターフェイスのアドレスを確認します。 

  - VPN デバイスのインターネット接続 IP アドレスが Azure Stack Hub の **[ローカル ネットワーク]** の定義に含まれていると、散発的に接続が切れることがあります。

  - デバイスの外部インターフェイスは、インターネットに直接接続されている必要があります。 インターネットとデバイスとの間にネットワーク アドレス変換またはファイアウォールを配置することはできません。

  - 仮想 IP が割り当てられるようにファイアウォール クラスタリングを構成するには、クラスターを分割し、ゲートウェイから対話できるパブリック インターフェイスに直接 VPN アプライアンスを公開する必要があります。

- サブネットが厳密に一致することを確認します。

  - Azure Stack Hub 仮想ネットワークとオンプレミスの定義で、仮想ネットワーク アドレス空間が完全に一致することを確認します。

  - **ローカル ネットワーク ゲートウェイ**とオンプレミス ネットワークのオンプレミス定義との間でサブネットが完全に一致することを確認します。

## <a name="create-a-support-ticket"></a>サポート チケットの作成

上記の手順で問題が解決しない場合、[サポート チケット](../operator/azure-stack-manage-basics.md#where-to-get-support)を作成し、[オンデマンド ログ収集ツール](../operator/azure-stack-diagnostic-log-collection-overview.md)を使用してログを提供してください。
