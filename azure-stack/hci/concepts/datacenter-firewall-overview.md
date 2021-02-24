---
title: Azure Stack HCI におけるデータセンターのファイアウォールの概要
description: このトピックでは、Azure Stack HCI におけるデータセンターのファイアウォールについて説明します。
author: khdownie
ms.author: v-kedow
ms.topic: overview
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 11/17/2020
ms.openlocfilehash: 34efe18aee4a62481d81a5e9a810a0a71d97b063
ms.sourcegitcommit: 40d3f3f0ac088d1590d1fb64ca05ac1dabf4e00c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/18/2020
ms.locfileid: "94874959"
---
# <a name="what-is-datacenter-firewall"></a>データセンターのファイアウォールとは

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

データセンターのファイアウォールは、ネットワーク層における、5 タプル (プロトコル、発信元と送信先のポート番号、送信元と送信先の IP アドレス) から成る、ステートフルなマルチテナント ソフトウェア定義ネットワーク (SDN) ファイアウォールです。 データセンターのファイアウォールでは、仮想ネットワークと従来の VLAN ネットワークを含むネットワーク層全体で、東西および北南のトラフィック フローを保護します。

## <a name="how-datacenter-firewall-works"></a>データセンターのファイアウォールのしくみ

データセンターのファイアウォールを有効にして構成するには、サブネットまたはネットワーク インターフェイスに適用されるアクセス制御リスト (ACL) を作成します。 ファイアウォール ポリシーは、各テナント仮想マシン (VM) の vSwitch ポートで適用されます。 ポリシーはテナント ポータルを通じてプッシュされ、[ネットワーク コントローラー](network-controller-overview.md)によってすべての該当するホストに配布されます。

テナント管理者は、ファイアウォール ポリシーをインストールして構成することにより、インターネットおよびイントラネット ネットワークから送信される不要なトラフィックから自分のネットワークを保護できます。

:::image type="content" source="media/datacenter-firewall/multitenant-firewall-overview.png" alt-text="ネットワーク スタックにおけるデータセンターのファイアウォール" border="false":::

サービス プロバイダー管理者またはテナント管理者は、ネットワーク コントローラーおよび Northbound API を使用してデータセンターのファイアウォールのポリシーを管理できます。 Windows Admin Center を使用して、データセンターのファイアウォールのポリシーを構成および管理することもできます。

## <a name="advantages-for-cloud-service-providers"></a>クラウド サービス プロバイダーにとっての利点

CSP にとって、データセンターのファイアウォールには次のような利点があります。

- 拡張性、管理可能性、診断可能性が高いソフトウェアベースのファイアウォール ソリューションをテナントに提供できます

- テナントのファイアウォールのポリシーに違反せずに、テナントの VM を別のコンピューティング ホストに自由に移動できます

    - vSwitch ポートのホスト エージェント ファイアウォールとしてデプロイできます

    - ポリシーは、テナント VM の vSwitch のホスト エージェント ファイアウォールに割り当てられます

    - ファイアウォール規則は、VM を実行している実際のホストとは無関係に、各 vSwitch ポートで構成されます

- テナントのゲスト オペレーティング システムとは無関係に、テナント VM が保護されます

## <a name="advantages-for-tenants"></a>テナントにとっての利点

テナントにとって、データセンターのファイアウォールには次のような利点があります。

- ファイアウォール規則を定義することによって、インターネットに接続されたワークロードとネットワーク上の内部ワークロードを保護できます

- ファイアウォール規則を定義することによって、同じレイヤー 2 (L2) サブネット上の VM 間および異なる L2 サブネット上の VM 間のトラフィックを保護できます

- ファイアウォール規則を定義することによって、テナントのオンプレミス ネットワークと、サービス プロバイダーの仮想ネットワーク間のネットワーク トラフィックを保護および分離できます

- 従来の VLAN ネットワークとオーバーレイベースの仮想ネットワークにファイアウォール ポリシーを適用できます

## <a name="next-steps"></a>次のステップ

関連情報については、以下もご覧ください。

- [Azure Stack HCI 内でソフトウェア定義ネットワークにデータセンターのファイアウォールを使用する](../manage/use-datacenter-firewall.md)
- [Azure Stack HCI における SDN](software-defined-networking.md)