---
title: Azure Stack HCI におけるソフトウェアによるネットワーク制御 (SDN)
description: ソフトウェアによるネットワーク制御 (SDN) は、ネットワークとネットワーク サービス (データ センターにおけるスイッチング、ルーティング、負荷分散など) を一元的に構成、管理する手段となります。
author: khdownie
ms.author: v-kedow
ms.topic: conceptual
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 09/29/2020
ms.openlocfilehash: e14d0475c941f568f08a027de9e71eb25f00a999
ms.sourcegitcommit: 69cfff119ab425d0fbb71e38d1480d051fc91216
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/30/2020
ms.locfileid: "91572655"
---
# <a name="software-defined-networking-sdn-in-azure-stack-hci"></a>Azure Stack HCI におけるソフトウェアによるネットワーク制御 (SDN)

> 適用対象: Azure Stack HCI バージョン 20H2、Windows Server 2019

ソフトウェアによるネットワーク制御 (SDN) は、ネットワークとネットワーク サービス (データ センターにおけるスイッチング、ルーティング、負荷分散など) を一元的に構成、管理する手段となります。 SDN を使用すると、ネットワークの作成、セキュリティ保護、接続を動的に行うことで、絶えず変化するアプリケーションのニーズに対応することができます。 1 日あたり何万件ものネットワーク変更を効率的に実行する、Microsoft Azure のようなサービスのための世界規模のデータセンター ネットワークは、ひとえに SDN の存在によって運用が実現されています。

[Hyper-V 仮想スイッチ](/windows-server/virtualization/hyper-v-virtual-switch/hyper-v-virtual-switch)、[Hyper-V ネットワーク仮想化](/windows-server/networking/sdn/technologies/hyper-v-network-virtualization/hyper-v-network-virtualization)、[ソフトウェア負荷分散](/windows-server/networking/sdn/technologies/network-function-virtualization/software-load-balancing-for-sdn)、[RAS ゲートウェイ](/windows-server/networking/sdn/technologies/network-function-virtualization/ras-gateway-for-sdn)などの仮想ネットワーク要素は、SDN インフラストラクチャの不可欠な要素として設計されています。 また、既存の SDN 適合デバイスを使用して、仮想ネットワークで実行されるワークロードと物理ネットワークで実行されるワークロードの統合を強化することもできます。

Azure Stack HCI には、主要な SDN コンポーネントとしてネットワーク コントローラー、ソフトウェア ロード バランサー、ゲートウェイの 3 つがあり、どれをデプロイするかは自分で選ぶことができます。

## <a name="network-controller"></a>ネットワーク コントローラー

[ネットワーク コントローラー](/windows-server/networking/sdn/technologies/Software-Defined-Networking-Technologies#network-controller)は、データ センターの仮想ネットワーク インフラストラクチャの管理、構成、監視、およびトラブルシューティングの自動化を行う、一元的でプログラミング可能なポイントを提供します。 Service Fabric を使用して高可用性を実現するきわめてスケーラブルなサーバー ロールです。 ネットワーク コントローラーは、専用の VM にデプロイする必要があります。

ネットワーク コントローラーをデプロイすることで、次の機能が実現します。

- 仮想ネットワークとサブネットを作成して管理する。 仮想マシン (VM) を仮想サブネットに接続します。
- 仮想ネットワークまたは従来の VLAN ベースのネットワークに接続された VM 用にマイクロセグメンテーションを構成して管理する。
- 仮想ネットワークに仮想アプライアンスをアタッチする。
- 仮想ネットワークまたは従来の VLAN ベースのネットワークにアタッチされた VM を対象に QoS (サービスの品質) ポリシーを構成する。

[クラスターの作成時にネットワーク コントローラーをデプロイする](../deploy/create-cluster.md#step-5-sdn-optional)ことをお勧めします。 あるいは、Azure Stack HCI クラスターの作成後に、[PowerShell を使用してネットワーク コントローラーをデプロイする](../deploy/network-controller-powershell.md)こともできます。

## <a name="software-load-balancing"></a>ソフトウェア負荷分散

[ソフトウェア負荷分散](/windows-server/networking/sdn/technologies/network-function-virtualization/software-load-balancing-for-sdn) (SLB) を使用すると、カスタマー ネットワーク トラフィックを複数の VM に対して均等に分散することができます。 複数のサーバーで同じワークロードをホストできるようになり、高可用性とスケーラビリティを実現できます。 SLB は、[Border Gateway Protocol](/windows-server/remote/remote-access/bgp/border-gateway-protocol-bgp) を使用して、仮想 IP アドレスを物理ネットワークにアドバタイズします。

## <a name="gateway"></a>Gateway

ゲートウェイは、仮想ネットワークと別のネットワーク (ローカルまたはリモート) の間でネットワーク トラフィックをルーティングする目的で使用されます。 ゲートウェイを使用して、次のことができます。

- SDN 仮想ネットワークと外部のカスタマー ネットワークの間に、インターネットを介した安全なサイト間 IPsec 接続を作成する。
- SDN 仮想ネットワークと外部ネットワークの間で Generic Routing Encapsulation (GRE) 接続を作成する。 サイト間接続と GRE 接続の違いは、後者は暗号化された接続ではないという点です。 GRE 接続シナリオの詳細については、[Windows Server における GRE トンネリング](/windows-server/remote/remote-access/ras-gateway/gre-tunneling-windows-server)に関するページを参照してください。
- SDN 仮想ネットワークと外部ネットワークの間でレイヤー 3 接続を作成する。 この場合、SDN ゲートウェイは単に、仮想ネットワークと外部ネットワークの間のルーターとして機能します。

ゲートウェイでは、[Border Gateway Protocol](/windows-server/remote/remote-access/bgp/border-gateway-protocol-bgp) を使用して GRE エンドポイントをアドバタイズし、ポイントツーポイント接続を確立します。 SDN のデプロイでは、あらゆる種類の接続をサポートするデフォルト ゲートウェイ プールが作成されます。 このプール内には、アクティブなゲートウェイで障害が発生した場合に備え、スタンバイとして予約されるゲートウェイの数を指定できます。

## <a name="next-steps"></a>次のステップ

関連情報については、以下もご覧ください。

- [ソフトウェア定義ネットワーク インフラストラクチャを計画する](plan-software-defined-networking-infrastructure.md)
- [Windows Server の SDN の概要](/windows-server/networking/sdn/software-defined-networking)
- [スクリプトを使用してソフトウェア定義ネットワーク インフラストラクチャを展開する](/windows-server/networking/sdn/deploy/deploy-a-software-defined-network-infrastructure-using-scripts)
