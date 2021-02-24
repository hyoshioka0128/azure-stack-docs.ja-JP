---
title: SQL Server を使用した Azure Stack Hub の Windows N 層アプリケーション
description: SQL Server を使用して Azure Stack Hub で Windows N 層アプリケーションを実行する方法について説明します。
author: mattbriggs
ms.topic: how-to
ms.date: 2/1/2021
ms.author: mabrigg
ms.reviewer: kivenkat
ms.lastreviewed: 11/01/2019
ms.openlocfilehash: 5af32be701617f7ff357d4776557b2edbbdee729
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99247491"
---
# <a name="windows-n-tier-application-on-azure-stack-hub-with-sql-server"></a>SQL Server を使用した Azure Stack Hub の Windows N 層アプリケーション

この参照アーキテクチャでは、Windows 上の SQL Server をデータ層に使用して、[N 層](/azure/architecture/guide/architecture-styles/n-tier)アプリケーション用に構成された仮想マシン (VM) と仮想ネットワークをデプロイする方法を示します。 

## <a name="architecture"></a>Architecture

このアーキテクチャには次のコンポーネントがあります。

![この図は、次の 6 つのサブネット (Application Gateway、管理、Web 層、ビジネス層、データ層、Active Directory ) から成る仮想ネットワークを示しています。 データ層サブネットはクラウド監視を使用します。 3 つのロードバランサーがあります。](./media/iaas-architecture-windows-sql-n-tier/image1.png)

## <a name="general"></a>全般

-   **リソース グループ**。 [リソース グループ](/azure/azure-resource-manager/resource-group-overview)は、Azure リソースをグループ化して、有効期間、所有者、またはその他の条件別に管理できるようにするために使用されます。

-   **可用性セット。** 可用性セットは、VM の冗長性と可用性を提供するデータセンター構成です。 Azure Stack Hub 内のこのような構成により、計画的または計画外メンテナンス イベント中に、少なくとも 1 個の仮想マシンが確実に利用可能になります。 VM は、複数の障害ドメイン (Azure Stack Hub ホスト) に VM が分散される可用性セットに配置されます。

## <a name="networking-and-load-balancing"></a>ネットワークと負荷分散

-   **仮想ネットワークとサブネット**。 すべての Azure VM が、サブネットにセグメント化できる仮想ネットワーク内にデプロイされます。 階層ごとに個別のサブネットを作成します。

-   **第 7 層のロード バランサー。** Azure Stack Hub では Application Gateway がまだ利用できないため、[Azure Stack Hub Marketplace](../operator/azure-stack-marketplace-azure-items.md) には代替手段が用意されています ([KEMP LoadMaster Load Balancer ADC Content Switch](https://azuremarketplace.microsoft.com/marketplace/apps/kemptech.vlm-azure)/ [f5 Big-IP Virtual Edition](https://azuremarketplace.microsoft.com/marketplace/apps/f5-networks.f5-big-ip-best) または [A10 vThunder ADC](https://azuremarketplace.microsoft.com/marketplace/apps/a10networks.vthunder-414-gr1) など)。

-   **ロード バランサー**。 [Azure Load Balancer](/azure/load-balancer/load-balancer-overview) は、Web 層からビジネス層へ、ビジネス層から SQL Server へとネットワーク トラフィックを分散するために使用します。

-   **ネットワーク セキュリティ グループ** (NSG)。 NSG を使用して、仮想ネットワーク内のネットワーク トラフィックを制限します。 たとえば、ここに示されている 3 層アーキテクチャでは、データベース層は Web フロントエンドからのトラフィックを受信せず、ビジネス層と管理サブネットからのトラフィックのみ受信します。

-   **DNS**。 Azure Stack Hub では独自の DNS ホスティング サービスが提供されていないため、AD DS で DNS サーバーを使用してください。

**仮想マシン**

-   **SQL Server Always On 可用性グループ**。 レプリケーションとフェールオーバーを有効にすることで、データ層で高い可用性を提供します。 これは、Windows Server フェールオーバー クラスター (WSFC) テクノロジを使用してフェールオーバーを行います。

-   **Active Directory Domain Services (AD DS) サーバー。** フェールオーバー クラスターと、これに関連するクラスター化されたロールを表すコンピューター オブジェクトは、Active Directory Domain Services (AD DS) に作成されます。 他の VM を AD DS に参加させるには、同じ仮想ネットワーク内にある VM 内の AD DS サーバーをセットアップすることをお勧めします。 また、VPN 接続を使用して仮想ネットワークをエンタープライズ ネットワークに接続することで、VM を既存のエンタープライズ AD DS に参加させることもできます。 どちらの方法でも、仮想ネットワーク DNS を AD DS DNS サーバー (仮想ネットワークまたは既存のエンタープライズ ネットワーク内) に変更して、AD DS ドメインの FQDN を解決する必要があります。

-   **クラウド監視**。 フェールオーバー クラスターでは、そのノードの半数以上が実行されている必要があり、"クォーラムに達している" と呼びます。 クラスターにノードが 2 つしかない場合は、ネットワークのパーティションにより、各ノードは自身がマスター ノードであると認識する可能性があります。 この場合、"*監視*" によって優先順位を決定し、クォーラムを確立する必要があります。 監視は、クォーラムを確立する際の優先順位決定者として動作可能な、共有ディスクなどのリソースです。 クラウド監視は、Azure Blob Storage を使用する一種の監視です。 クォーラムの概念の詳細については、「[Understanding cluster and pool quorum (クラスターとプール クォーラムについて)](/windows-server/storage/storage-spaces/understand-quorum)」を参照してください。 クラウド監視の詳細については、「[Deploy a cloud witness for a Failover Cluster (フェールオーバー クラスター用にクラウド監視をデプロイする)](/windows-server/failover-clustering/deploy-cloud-witness)」を参照してください。 Azure Stack Hub では、クラウド監視エンドポイントはグローバル Azure とは異なります。 

次のようになります。

- グローバル Azure の場合:  
  `https://mywitness.blob.core.windows.net/`

- Azure Stack Hub の場合:  
  `https://mywitness.blob.<region>.<FQDN>`

-   **Jumpbox**。 [要塞ホスト](https://en.wikipedia.org/wiki/Bastion_host)とも呼ばれます。 管理者が他の VM に接続するために使用するネットワーク上のセキュアな VM です。 jumpbox の NSG は、セーフ リストにあるパブリック IP アドレスからのリモート トラフィックのみを許可します。 NSG は、リモート デスクトップ (RDP) トラフィックを許可する必要があります。

## <a name="recommendations"></a>推奨事項

実際の要件は、ここで説明するアーキテクチャとは異なる場合があります。 これらの推奨事項を開始点として使用してください。

### <a name="virtual-machines"></a>仮想マシン

VM の構成に関する推奨事項については、「[Azure Stack Hub で Windows 仮想マシンを実行する](iaas-architecture-vm-windows.md)」を参照してください。

### <a name="virtual-network"></a>仮想ネットワーク

仮想ネットワークを作成するときに、各サブネット内のリソースが要求する IP アドレスの数を決定します。 [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) 表記を使用して、必要な IP アドレスにとって十分な規模のサブネット マスクとネットワーク アドレス範囲を指定します。 標準的な[プライベート IP アドレス ブロック](https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces) (10.0.0.0/8、172.16.0.0/12、192.168.0.0/16) の範囲内にあるアドレス空間を使用します。

後で仮想ネットワークとオンプレミスのネットワークとの間にゲートウェイを設定する必要がある場合は、オンプレミスのネットワークと重複しないアドレス範囲を選択します。 仮想ネットワークを作成した後に、アドレス範囲を変更することはできません。

機能とセキュリティの要件を念頭に置いてサブネットを設計します。 同じ層または同じロール内のすべての VM は、同じサブネットに入れる必要があります。これがセキュリティ境界になります。 仮想ネットワークとサブネットの設計の詳細については、[Azure Virtual Network の計画と設計](/azure/virtual-network/virtual-network-vnet-plan-design-arm)に関するページを参照してください。

### <a name="load-balancers"></a>ロード バランサー

VM は直接インターネットに公開せず、代わりに各 VM にプライベート IP アドレスを付与します。 クライアントでは、レイヤー 7 ロード バランサーに関連付けられているパブリック IP アドレスを使用して接続が行われます。

ロード バランサー規則を定義して、ネットワーク トラフィックを VM に転送します。 たとえば、HTTP トラフィックを有効にするには、フロントエンド構成からのポート 80 をバックエンド アドレス プールのポート 80 にマップします。 クライアントがポート 80 に HTTP 要求を送信するときに、ロード バランサーは、発信元 IP アドレスを含む[ハッシュ アルゴリズム](/azure/load-balancer/concepts#limitations)を使用して、バックエンド IP アドレスを選択します。 クライアント要求が、バックエンド アドレス プールのすべての VM に分散されます。

### <a name="network-security-groups"></a>ネットワーク セキュリティ グループ

NSG ルールを使用して階層間のトラフィックを制限します。 上記の 3 層アーキテクチャでは、Web 層はデータベース層と直接通信しません。 この規則を適用するには、データベース層が Web 層のサブネットからの着信トラフィックをブロックする必要があります。

1.  仮想ネットワークからのすべての受信トラフィックを拒否します。 (ルールで VIRTUAL_NETWORK タグを使用します。)

2.  ビジネス層のサブネットからの受信トラフィックを許可します。

3.  データベース層のサブネットからの受信トラフィックを許可します。 このルールにより、データベースのレプリケーションやフェールオーバーに必要な、データベース VM 間の通信が可能になります。

4.  ジャンプボックスのサブネットからの RDP トラフィック (ポート 3389) を許可します。 このルールによって、管理者がジャンプボックスからデータベース層に接続できるようにします。

最初のルールよりも高い優先順位を設定してルール 2 から 4 を作成し、最初のルールをオーバーライドします。

## <a name="sql-server-always-on-availability-groups"></a>SQL Server Always On 可用性グループ

SQL Server の高可用性のために [Always On 可用性グループ](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-ver15&preserve-view=true)の使用をお勧めします。 Windows Server 2016 に先立って、Always On 可用性グループはドメイン コントローラーを必要とし、可用性グループ内のすべてのノードが同じ AD ドメイン内にある必要があります。

VM レイヤーの高可用性を実現するには、すべての SQL VM が可用性セットに含まれている必要があります。

他の層は[可用性グループ リスナー](/sql/database-engine/availability-groups/windows/listeners-client-connectivity-application-failover?view=sql-server-ver15&preserve-view=true)を使用してデータベースに接続します。 リスナーを使用することで、SQL クライアントは SQL Server の物理インスタンスの名前を知らなくても接続できます。 データベースにアクセスする VM はドメインに参加している必要があります。 クライアント (ここでは、別の層) は、DNS を使用してリスナーの仮想ネットワーク名を IP アドレスに解決します。

SQL Server Always On 可用性グループを構成する手順は、次のとおりです。

1.  Windows Server フェールオーバー クラスタリング (WSFC) クラスター、SQL Server Always On 可用性グループ、プライマリ レプリカを作成します。 詳細については、「[AlwaysOn 可用性グループの概要](/sql/database-engine/availability-groups/windows/getting-started-with-always-on-availability-groups-sql-server?view=sql-server-ver15&preserve-view=true)」を参照してください。

2.  静的プライベート IP アドレスを持つ内部ロード バランサーを作成します。

3.  可用性グループ リスナーを作成し、リスナーの DNS 名を内部ロード バランサーの IP アドレスにマッピングします。

4.  SQL Server リスニング ポート (既定ではTCP ポート 1433) に対するロード バランサーのルールを作成します。 ロード バランサーのルールでは *Floating IP* (Direct Server Return とも呼ばれます) を有効にする必要があります。 これにより VM が直接クライアントに応答でき、プライマリ レプリカへの直接接続が可能になります。

> [!NOTE]
> Floating IP が有効になっている場合は、フロントエンド ポート番号を、ロード バランサーのルール内のバックエンド ポート番号と同じにする必要があります。

SQL クライアントが接続を試みると、ロード バランサーがプライマリ レプリカに接続要求をルーティングします。 別のレプリカへのフェールオーバーが発生した場合は、ロード バランサーは新しい要求を自動的に新しいプライマリ レプリカにルーティングします。 詳細については、[SQL Server Always On 可用性グループの ILB リスナーの構成](/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-alwayson-int-listener)に関するページを参照してください。

フェールオーバー中は、既存のクライアント接続は閉じられます。 フェールオーバーが完了すると、新しい接続は新しいプライマリ レプリカにルーティングされます。

アプリケーションで書き込みよりも多くの読み取りが行われる場合は、読み取り専用クエリの一部をセカンダリ レプリカにオフロードできます。 「[リスナーを使用した読み取り専用セカンダリ レプリカへの接続 (読み取り専用ルーティング)](/sql/database-engine/availability-groups/windows/listeners-client-connectivity-application-failover?view=sql-server-ver15&preserve-view=true#ConnectToSecondary)」を参照してください。

可用性グループの[手動フェールオーバーの強制](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-ver15&preserve-view=true)によってデプロイをテストします。

SQL パフォーマンスの最適化については、記事「[Azure Stack Hub におけるパフォーマンスを最適化するための SQL サーバーのベスト プラクティス](./azure-stack-sql-server-vm-considerations.md)」も参照できます。

**Jumpbox**

アプリケーション ワークロードを実行する VM へのパブリック インターネットからの RDP アクセスを許可しないでください。 代わりに、これらの VM へのすべての RDP アクセスは、ジャンプ ボックスを経由するようにします。 管理者はジャンプボックスにログインし、次にジャンプボックスから他の VM にログインします。 ジャンプボックスは、既知の安全な IP アドレスからのみ、インターネットからの RDP トラフィックを許可します。

ジャンプボックスのパフォーマンス要件は最小限に抑えられているので、小さな VM サイズを選択します。 ジャンプボックス用に[パブリック IP アドレス](/azure/virtual-network/virtual-network-ip-addresses-overview-arm)を作成します。 ジャンプ ボックスを、他の VM と同じ仮想ネットワーク内の、個別の管理サブネット内に配置します。

ジャンプボックスをセキュリティで保護するには、安全な一連のパブリック IP アドレスからのみ RDP 接続を許可する NSG ルールを追加します。 他のサブネットに対しても NSG を構成して、管理サブネットからの RDP トラフィックを許可します。

## <a name="scalability-considerations"></a>スケーラビリティに関する考慮事項

### <a name="scale-sets"></a>スケール セット

Web 層とビジネス層については、個別の VM をデプロイするのではなく、[仮想マシン スケール セット](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview)を使用することを検討してください。 スケール セットを使用すると、同一の VM のセットを簡単にデプロイして管理できます。 VM をすばやくスケール アウトする必要がある場合は、スケール セットを検討してください。

スケール セットにデプロイされる VM を構成するには、2 つの基本的な方法があります。

-   デプロイ後に、拡張機能を使用して VM を構成します。 この方法では、新しい VM インスタンスは、拡張機能なしの VM よりも起動に時間がかかる場合があります。

-   カスタム ディスク イメージと共に[マネージド ディスク](./azure-stack-managed-disk-considerations.md)をデプロイします。 このオプションの方が早くデプロイできる場合があります。 ただし、イメージを最新の状態に保つ必要があります。

詳細については、「[スケール セットの設計上の考慮事項](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview)」を参照してください。 この設計上の考慮事項はほぼ Azure Stack Hub に当てはまりますが、次の点に注意する必要があります。

-   Azure Stack Hub の仮想マシン スケール セットでは、オーバープロビジョニングまたはローリング アップグレードはサポートされていません。

-   Azure Stack Hub の仮想マシン スケール セットを自動スケールすることはできません。

-   仮想マシン スケール セットのアンマネージド ディスクではなく、Azure Stack Hub のマネージド ディスクを使用することを強くお勧めします。

-   現在、Azure Stack Hub では VM の数に 700 個という制限があり、これにはすべての Azure Stack Hub インフラストラクチャの VM、個々の VM、スケール セット インスタンスが含まれます。

## <a name="subscription-limits"></a>サブスクリプションの制限

各 Azure Stack Hub テナント サブスクリプションには、Azure Stack Hub オペレーターによって構成されたリージョンごとの VM の最大数など、既定の制限があります。 詳細については、「[Azure Stack Hub サービス、プラン、オファー、サブスクリプションの概要](../operator/service-plan-offer-subscription-overview.md)」を参照してください。 「[Azure Stack Hub のクォータの種類](../operator/azure-stack-quota-types.md)」も参照してください。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

仮想ネットワークは、Azure のトラフィックの分離境界です。 既定では、ある仮想ネットワーク内の VM が、別の仮想ネットワーク内の VM と直接通信することはできません。

**NSG**。 [ネットワーク セキュリティ グループ](/azure/virtual-network/virtual-networks-nsg) (NSG) を使用して、インターネットとの間で送受信されるトラフィックを制限します。 詳細については、「[Microsoft クラウド サービスとネットワーク セキュリティ](/azure/best-practices-network-security)」をご覧ください。

**DMZ**。 ネットワーク仮想アプライアンス (NVA) を追加してインターネットと Azure Virtual Network の間の DMZ を作成することを検討してください。 NVA とは、ネットワーク関連のタスク (ファイアウォール、パケット インスペクション、監査、カスタム ルーティングなど) を実行できる仮想アプライアンスの総称です。

**暗号化**。 機密の保存データを暗号化し、[Azure Stack Hub の Key Vault](./azure-stack-key-vault-manage-portal.md) を使用してデータベース暗号化キーを管理します。 詳細については、 [Azure VM 上の SQL Server に関する Azure Key Vault 統合の構成](/azure/azure-sql/virtual-machines/windows/azure-key-vault-integration-configure)に関するページを参照してください。 データベース接続文字列などのアプリケーション シークレットも Key Vault に格納することをお勧めします。

## <a name="next-steps"></a>次のステップ

- Azure のクラウド パターンの詳細については、「[Cloud Design Pattern (クラウド設計パターン)](/azure/architecture/patterns)」を参照してください。
