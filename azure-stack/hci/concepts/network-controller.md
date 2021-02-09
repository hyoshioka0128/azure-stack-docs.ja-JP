---
title: ネットワーク コントローラーのデプロイを計画する
description: このトピックでは、一連の仮想マシン (VM) 上での、Windows Admin Center を使用したネットワーク コントローラーのデプロイを計画する方法について説明します。
author: AnirbanPaul
ms.author: anpaul
ms.topic: conceptual
ms.date: 02/02/2021
ms.openlocfilehash: bfea9216cefdc64c7749f8b49d5ecc3a422e5130
ms.sourcegitcommit: 0e58c5cefaa81541d9280c0e8a87034989358647
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99510790"
---
# <a name="plan-to-deploy-network-controller"></a>ネットワーク コントローラーのデプロイを計画する

>適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019、Windows Server 2016

Windows Admin Center を使用したネットワーク コントローラーのデプロイを計画するには、Azure Stack HCI または Windows Server オペレーティング システムを実行する一連の仮想マシン (VM) が必要です。 ネットワーク コントローラーは、可用性が高く、スケーラブルなサーバーの役割を担います。ネットワークで高可用性を実現するために、少なくとも 3 つの VM を必要とします。

   >[!NOTE]
   > ネットワーク コントローラーは、専用の VM にデプロイすることをお勧めします。

## <a name="network-controller-requirements"></a>ネットワーク コントローラーの要件
ネットワーク コントローラーをデプロイするには、次のものが必要です。
- ネットワーク コントローラー VM を作成するための Azure Stack HCI オペレーティング システム用の仮想ハード ディスク (VHD)。
- ネットワーク コントローラー VM をドメインに参加させるためのドメイン名と資格情報。
- Windows Admin Center のクラスター作成ウィザードを使用して構成する、1 つ以上の仮想スイッチ。
- このセクションのトポロジ オプションのいずれかに一致する物理ネットワーク構成。

    Windows Admin Center により、Hyper-V ホスト内に構成が作成されます。 ただし、管理ネットワークは、次の 3 つのオプションのいずれかに従ってホストの物理アダプターに接続する必要があります。

    **オプション 1**:管理ネットワークがワークロード ネットワークから物理的に分離されています。 このオプションを選択すると、コンピューティングとストレージの両方に 1 つの仮想スイッチが使用されます。

    :::image type="content" source="./media/network-controller/topology-option-1.png" alt-text="ネットワーク コントローラーの物理ネットワークを作成するためのオプション 1。" lightbox="./media/network-controller/topology-option-1.png":::

    **オプション 2**:管理ネットワークがワークロード ネットワークから物理的に分離されています。 このオプションを選択すると、コンピューティング専用に 1 つの仮想スイッチが使用されます。

    :::image type="content" source="./media/network-controller/topology-option-2.png" alt-text="ネットワーク コントローラーの物理ネットワークを作成するためのオプション 2。" lightbox="./media/network-controller/topology-option-2.png":::

    **オプション 3**:管理ネットワークがワークロード ネットワークから物理的に分離されています。 このオプションを選択すると、2 つの仮想スイッチ (1 つはコンピューティング用で、もう 1 つはストレージ用) が使用されます。

    :::image type="content" source="./media/network-controller/topology-option-3.png" alt-text="ネットワーク コントローラーの物理ネットワークを作成するためのオプション 3。" lightbox="./media/network-controller/topology-option-3.png":::

- 管理物理アダプターをチーム化して、同じ管理スイッチを使用することもできます。 この場合でも、このセクションのオプションのいずれかを使用することをお勧めします。
- ネットワーク コントローラーが Windows Admin Center および Hyper-V ホストと通信するために使用する管理ネットワーク情報。
- ネットワーク コントローラー VM の DHCP ベースまたは静的ネットワーク ベースのアドレス指定。
- 管理クライアントがネットワーク コントローラーと通信するために使用する、ネットワーク コントローラーの Representational State Transfer (REST) の完全修飾ドメイン名 (FQDN)。

   >[!NOTE]
   > 現在、Windows Admin Center によって、REST クライアントとの通信、またはネットワーク コントローラー VM 間の通信に対して、ネットワーク コントローラー認証はサポートされていません。 PowerShell を使用してデプロイおよび管理する場合は、Kerberos ベースの認証を使用できます。

## <a name="configuration-requirements"></a>構成要件
ネットワーク コントローラーのクラスター ノードは、同じサブネットまたは異なるサブネットにデプロイできます。 ネットワーク コントローラーのクラスター ノードを別のサブネットにデプロイする場合は、デプロイ プロセス中にネットワーク コントローラーの REST DNS 名を指定する必要があります。

詳細については、「[ネットワーク コントローラーの動的 DNS 登録を構成する](/windows-server/networking/sdn/plan/installation-and-preparation-requirements-for-deploying-network-controller#step-3-configure-dynamic-dns-registration-for-network-controller)」を参照してください。

## <a name="next-steps"></a>次のステップ
これで、VM にネットワーク コントローラーをデプロイする準備ができました。

## <a name="see-also"></a>関連項目
- [Azure Stack HCI クラスターを作成する](../deploy/create-cluster.md)
- [SDN Express を使用して SDN インフラストラクチャをデプロイする](../manage/sdn-express.md)
- [ネットワーク コント ローラーの概要](network-controller-overview.md)
- [ネットワーク コントローラーの高可用性](/windows-server/networking/sdn/technologies/network-controller/network-controller-high-availability)