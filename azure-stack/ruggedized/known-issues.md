---
title: Azure Stack Hub ラグドの Azure Stack Hub の既知の問題
description: Azure Stack Hub ラグドの Azure Stack Hub の既知の問題
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/08/2020
ms.author: sethm
ms.reviewer: sranthar
ms.lastreviewed: 12/08/2020
ms.openlocfilehash: 4b75024a1ff747d39f4f2194d0d86c3882475dee
ms.sourcegitcommit: 50b362d531c2d35a3a935811fee71252971bd5d8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/09/2020
ms.locfileid: "96941312"
---
# <a name="known-issues-in-azure-stack-hub-ruggedized"></a>Azure Stack Hub ラグドの既知の問題 

この記事では、Azure Stack Hub ラグドにおける既知の問題の一覧を示します。 新しい問題が特定されると、この一覧は更新されます。

## <a name="update"></a>更新

Azure Stack Hub の更新に関する既知の問題については、[Azure Stack Hub の更新プログラムに関するトラブルシューティング](../operator/azure-stack-troubleshooting.md#troubleshoot-azure-stack-hub-updates)に関する記事を参照してください。

### <a name="update-failed-to-install-package-microsoftazurestackcomputeinstaller-to-ca-vm"></a>更新による CA VM へのパッケージ Microsoft.AzureStack.Compute.Installer のインストールに失敗しました

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:更新中、CA VM にコピーする必要がある新しいコンテンツで、プロセスがロックを取得します。 更新が失敗すると、ロックは解除されます。
- 修復: 更新プログラムを再開します。
- 発生頻度: 珍しい

## <a name="portal"></a>ポータル

### <a name="administrative-subscriptions"></a>管理サブスクリプション

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: バージョン 1804 で導入された 2 つの管理サブスクリプションは使用しないでください。 このサブスクリプションの種類は **Metering** サブスクリプションと **Consumption** サブスクリプションです。
- 修復: これら 2 つのサブスクリプション上でリソースが実行されている場合は、ユーザー サブスクリプションで再作成してください。
- 発生頻度: 共通

## <a name="networking"></a>ネットワーク

### <a name="network-security-groups"></a>ネットワーク セキュリティ グループ

#### <a name="denyalloutbound-rule-cannot-be-created"></a>DenyAllOutbound ルールを作成できません

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: VM の作成中は、インターネットに対する明示的な **DenyAllOutbound** 規則を NSG に作成することはできません。VM のデプロイを完了するために必要な通信が妨げられるためです。 また、VM をデプロイするために必要な次の 2 つの重要な IP も拒否されます:DHCP IP:169.254.169.254 および DNS IP:168.63.129.16

- 修復: VM の作成中はインターネットのアウトバウンド トラフィックを許可しておき、VM の作成後に、必要なトラフィックをブロックするよう NSG を変更してください。
- 発生頻度: 共通

### <a name="virtual-network-gateway"></a>Virtual Network ゲートウェイ

#### <a name="documentation"></a>ドキュメント

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 仮想ネットワーク ゲートウェイの概要ページにあるドキュメントのリンクは、Azure Stack Hub ではなく Azure 固有のドキュメントにリンクされています。 Azure Stack Hub のドキュメントについては、次のリンクを使用してください。

  - [ゲートウェイ SKU](../user/azure-stack-vpn-gateway-about-vpn-gateways.md#gateway-skus)
  - [高可用性接続](../user/azure-stack-vpn-gateway-about-vpn-gateways.md#gateway-availability)
  - [Azure Stack Hub での BGP の構成](../user/azure-stack-vpn-gateway-settings.md#gateway-requirements)
  - [ExpressRoute 回線](../operator/azure-stack-connect-expressroute.md)
  - [カスタムの IPsec/IKE ポリシーの指定](../user/azure-stack-vpn-gateway-settings.md#ipsecike-parameters)

### <a name="load-balancer"></a>Load Balancer

#### <a name="load-balancer-directing-traffic-to-one-backend-vm-in-specific-scenarios"></a>Load Balancer が特定のシナリオでトラフィックを 1 つのバックエンド VM に送信する

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。 
- 原因: ロード バランサーで **セッション アフィニティ** を有効にすると、2 タプル ハッシュで、VM に割り当てられたプライベート IP ではなく、PA IP (物理アドレス IP) が使用されます。 ロード バランサーに送信されるトラフィックが VPN 経由で到着する、またはすべてのクライアント VM (ソース IP) が同じノード上に存在し、セッション アフィニティが有効になっているシナリオでは、すべてのトラフィックが 1 つのバックエンド VM に送信されます。
- 発生頻度: 共通
