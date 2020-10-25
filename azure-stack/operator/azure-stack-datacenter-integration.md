---
title: Azure Stack Hub 統合システムのデータセンター統合を計画する
description: Azure Stack Hub 統合システムとのデータセンター統合を計画し、準備する方法について説明します。
author: IngridAtMicrosoft
ms.topic: conceptual
ms.date: 04/02/2020
ms.author: inhenkel
ms.reviewer: wfayed
ms.lastreviewed: 09/12/2019
ms.openlocfilehash: 825db573a614d1a1dd9b54cd87d8894c981fce4b
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90573035"
---
# <a name="datacenter-integration-planning-considerations-for-azure-stack-hub-integrated-systems"></a>Azure Stack Hub 統合システムのデータセンター統合計画に関する考慮事項

Azure Stack Hub 統合システムに関心がある場合は、デプロイに関する主な計画の考慮事項と、このシステムがご利用のデータセンターにどのように適合するかについて理解しておく必要があります。 この記事では、Azure Stack Hub 統合システムに関するインフラストラクチャの重要な決定を行うときに役立つこれらの考慮事項の概要について説明します。 これらの考慮事項を理解していると、OEM ハードウェア ベンダーと協力して Azure Stack Hub をデータセンターにデプロイする場合に役立ちます。  

> [!NOTE]  
> Azure Stack Hub 統合システムは、承認されたハードウェア ベンダーからのみ購入できます。

Azure Stack Hub をデプロイするには、デプロイが開始される前に、使用するソリューション プロバイダーに計画の情報を提供して、プロセスが迅速かつ円滑に進むようにする必要があります。 必要な情報はネットワーク、セキュリティ、および ID に関する情報全般にわたり、多くの重要な決定にはさまざまな領域の知識と意思決定者が必要になる可能性があります。 デプロイが開始される前に必要なすべての情報を確実に準備するために、組織内の複数のチームからメンバーを集める必要があります。 ハードウェア ベンダーから有用なアドバイスを得られる可能性があるため、これらの情報を収集する際には、それらのベンダーに相談することをお勧めします。

必要な情報を調査および収集している間に、ネットワーク環境へのデプロイ前の構成変更がいくつか必要になることがあります。 これらの変更には、Azure Stack Hub ソリューションのための IP アドレス領域の予約や、新しい Azure Stack Hub ソリューション スイッチへの接続を準備するためのご利用のルーター、スイッチ、ファイアウォールの構成が含まれる可能性があります。 計画の策定に役立つサブジェクト領域のエキスパートを確保するようにしてください。

## <a name="capacity-planning-considerations"></a>容量計画に関する考慮事項

取得のために Azure Stack Hub ソリューションを評価するとき、その Azure Stack Hub ソリューションの容量全体に直接影響するハードウェア構成の選択を行います。 これらには、CPU、メモリ密度、記憶域構成、全体的なソリューションの規模 (サーバー数など) といった従来の選択肢が含まれます。 従来の仮想化ソリューションとは異なり、これらのコンポーネントを単純に計算して使用可能な容量を決定することはできません。 1 つ目の理由は、Azure Stack Hub は、ソリューション自体の中でインフラストラクチャまたは管理コンポーネントをホストするように設計されているためです。 2 つ目の理由は、テナントのワークロードの中断を最小限に抑えるようにソリューションのソフトウェアを更新することで、回復性をサポートするためにソリューションの容量の一部が確保されることです。

[Azure Stack Hub Capacity Planner スプレッドシート](https://aka.ms/azstackcapacityplanner)を使用すると、2 つの方法で容量計画のための情報に基づく意思決定ができるようになります。 1 つ目は、ハードウェア製品を選択してからリソースの組み合わせを適合させる方法です。 2 つ目は、Azure Stack Hub が実行するワークロードを定義して、それをサポートできる使用可能なハードウェア SKU を確認する方法です。 最後に、このスプレッドシートは、Azure Stack Hub の計画と構成に関連して決定を行う際に役立つガイドとして使用することもできます。

このスプレッドシートは、独自の調査や分析に代わるものとしては利用できません。 Microsoft は、スプレッドシートで提供される情報に対して、明示または黙示を問わず、一切の表明または保証を行わないものとします。

## <a name="management-considerations"></a>管理の考慮事項

Azure Stack Hub は、アクセス許可とネットワークの両方の観点から制限されたインフラストラクチャを持つシールド システムです。 ネットワーク アクセス制御リスト (ACL) を適用することで、承認されていないすべての受信トラフィックと、インフラストラクチャ コンポーネント間のすべての不要な通信をブロックします。 このシステムは、承認されていないユーザーがシステムにアクセスすることを困難にします。

日常の管理と運用に対して、インフラストラクチャへの無制限の管理者アクセスはありません。 Azure Stack Hub オペレーターは、管理者ポータルまたは Azure Resource Manager から (PowerShell または REST API を使用して) システムを管理する必要があります。 Hyper-V マネージャーやフェールオーバー クラスター マネージャーなどの他の管理ツールでシステムにアクセスすることはできません。 システム保護の目的で、サード パーティのソフトウェア (エージェントなど) は Azure Stack Hub インフラストラクチャのコンポーネント内部にインストールできません。 PowerShell または REST API を介して、管理とセキュリティ用の外部ソフトウェアとの相互運用性が提供されます。

アラート修正の手順を使用して解決できない問題のトラブルシューティングを行うために上位レベルのアクセス権が必要な場合は、Microsoft サポートにお問い合わせください。 サポートを通して、より高度な操作のためにシステムに対する一時的な管理者のフル権限を得る方法があります。

## <a name="identity-considerations"></a>ID に関する考慮事項

### <a name="choose-identity-provider"></a>ID プロバイダーの選択

Azure Stack Hub のデプロイに Azure AD と AD FS のどちらの ID プロバイダーを使用するかを検討する必要があります。 デプロイ後に ID プロバイダーを切り替えるには、システム全体を再デプロイする必要があります。 Azure AD アカウントを持っておらず、クラウド ソリューション プロバイダーから提供されたアカウントを使用している場合、プロバイダーを切り替えて別の Azure AD アカウントを使用するには、ソリューション プロバイダーに連絡し、有料でソリューションの再デプロイを依頼する必要があります。

ID プロバイダーの選択は、テナントの仮想マシン (VM)、ID システム、使用するアカウント、Active Directory ドメインに参加できるかどうかなどには関係しません。 これらの事項は別個のものです。

[Azure Stack Hub 統合システムの接続モデルの記事](./azure-stack-connection-models.md)では、ID プロバイダーの選択に関する詳細を学習できます。

### <a name="ad-fs-and-graph-integration"></a>AD FS と Graph の統合

ID プロバイダーとして AD FS を使用して Azure Stack Hub をデプロイする場合は、フェデレーションの信頼を介して Azure Stack Hub 上の AD FS インスタンスと既存の AD FS インスタンスを統合する必要があります。 この統合により、既存の Active Directory フォレスト内の ID を Azure Stack Hub 内のリソースに対して認証できます。

Azure Stack Hub の Graph サービスを既存の Active Directory と統合することもできます。 この統合により、Azure Stack Hub でロールベースのアクセス制御 (RBAC) を管理することができます。 リソースへのアクセスが委任されると、Graph のコンポーネントは LDAP プロトコルを使用して、既存の Active Directory フォレストでユーザー アカウントを検索します。

次の図は、統合された AD FS と Graph のトラフィック フローを示しています。<br/><br/>
![AD FS と Graph のトラフィック フローの図](media/azure-stack-datacenter-integration/ADFSIntegration.svg)

## <a name="licensing-model"></a>ライセンス モデル

使用するライセンス モデルを決める必要があります。 使用可能なオプションは、インターネットに接続された Azure Stack Hub をデプロイするかどうかによって異なります。

- [接続されたデプロイ](azure-stack-connected-deployment.md)の場合、従量制ライセンスか容量ベースのライセンスを選択できます。 従量制では、Azure との通信を通じて課金される使用量を報告するために Azure に接続する必要があります。 
- インターネットに[接続していないデプロイ](azure-stack-disconnected-deployment.md)では、容量ベースのライセンスのみがサポートされます。 

ライセンス モデルの詳細については、「[Microsoft Azure Stack Hub のパッケージと料金](https://azure.microsoft.com/mediahandler/files/resourcefiles/5bc3f30c-cd57-4513-989e-056325eb95e1/Azure-Stack-packaging-and-pricing-datasheet.pdf)」を参照してください。


## <a name="naming-decisions"></a>名前付けの決定事項

Azure Stack Hub の名前空間 (特にリージョン名と外部のドメイン名) を計画する方法について検討する必要があります。 Azure Stack Hub のデプロイで公開されたエンドポイントの外部完全修飾ドメイン名 (FQDN) は、&lt;*region*&gt;.&lt;*fqdn*&gt; という 2 つの名前の組み合わせです。 たとえば、*east.cloud.fabrikam.com* となります。 この例では、Azure Stack Hub ポータルが次の URL で使用可能になります。

- `https://portal.east.cloud.fabrikam.com`
- `https://adminportal.east.cloud.fabrikam.com`

> [!IMPORTANT]
> Azure Stack Hub のデプロイに選択したリージョン名は一意でなければならず、ポータル アドレスに表示されます。 

次の表は、これらのドメイン名前付けの決定事項をまとめたものです。

| 名前 | 説明 |
| -------- | ------------- |
|リージョン名 | 最初の Azure Stack Hub リージョンの名前。 この名前は、Azure Stack Hub によって管理されるパブリック仮想 IP アドレス (VIP) の FQDN の一部として使用されます。 通常、リージョン名は、データセンターの場所などの物理的な場所の識別子になります。<br><br>リージョン名は文字と 0 から 9 までの数字で作成する必要があります。 特殊文字 (`-`、`#` など) は使用できません。| 
| 外部ドメイン名 | 外部接続 VIP を持つエンドポイントのドメイン ネーム システム (DNS) ゾーンの名前。 パブリック VIP の FQDN に使用します。 | 
| プライベート (内部) ドメイン名 | インフラストラクチャ管理のために Azure Stack Hub 上に作成されるドメイン (および内部の DNS ゾーン) の名前。
| | |

## <a name="certificate-requirements"></a>証明書の要件

デプロイに使用するために、公開されたエンドポイントの Secure Sockets Layer (SSL) 証明書を提供する必要があります。 大まかに言うと、証明書には以下の要件があります。

- 1 つのワイルドカード証明書を使用するか、専用証明書のセットを使用して、記憶域や Key Vault などのエンドポイントに対してのみワイルドカードを使用できます。
- 証明書は、信頼された公的証明機関 (CA) または顧客によって管理される CA が発行できます。

Azure Stack Hub をデプロイするために必要な PKI 証明書、およびその取得方法の詳細については、「[Azure Stack Hub 公開キー インフラストラクチャ証明書の要件](azure-stack-pki-certs.md)」を参照してください。  

> [!IMPORTANT]
> 提供されている PKI 証明書の情報は、一般的なガイダンスとして使用してください。 Azure Stack Hub に対する PKI 証明書を取得する前に、ご利用の OEM ハードウェア パートナーに連絡してください。 証明書のガイダンスと要件の詳細が提供されます。

## <a name="time-synchronization"></a>時刻同期

Azure Stack Hub を同期するために使用される特定のタイム サーバーを選択する必要があります。 時刻同期は、Kerberos チケットの生成に使用されるため、Azure Stack Hub とそのインフラストラクチャ ロールにとって重要です。 Kerberos チケットは、内部サービスを互いに認証するために使用されます。

時刻同期サーバーの IP を指定する必要があります。 インフラストラクチャ内のほとんどのコンポーネントは URL を解決できますが、一部は IP アドレスしかサポートしません。 切断されたデプロイのオプションを使用している場合は、Azure Stack Hub のインフラストラクチャ ネットワークから確実に到達できるご利用の企業ネットワーク上のタイム サーバーを指定する必要があります。

## <a name="connect-azure-stack-hub-to-azure"></a>Azure への Azure Stack Hub の接続

ハイブリッド クラウド シナリオでは、Azure Stack Hub を Azure に接続する方法を計画する必要があります。 Azure Stack Hub 内の仮想ネットワークを Azure 内の仮想ネットワークに接続する 2 つの方法がサポートされています。

- **サイト間**:IPsec (IKE v1 および IKE v2) 経由の仮想プライベート ネットワーク (VPN) 接続。 この種の接続には、VPN デバイスまたは RRAS (ルーティングとリモート アクセス サービス) が必要です。 Azure での VPN Gateway の詳細については、「[VPN Gateway について](/azure/vpn-gateway/vpn-gateway-about-vpngateways)」を参照してください。 このトンネル経由の通信は暗号化されており、安全です。 ただし、帯域幅はトンネルの最大スループット (100 - 200 Mbps) によって制限されます。

- **送信 NAT**:既定では、Azure Stack Hub 内のすべての VM が送信 NAT を経由して外部ネットワークに接続できます。 Azure Stack Hub 内に作成される各仮想ネットワークには、パブリック IP アドレスが割り当てられます。 VM にパブリック IP アドレスが直接割り当てられているか、パブリック IP アドレスを持つロード バランサーを介しているかにかかわらず、仮想ネットワークの VIP を使用して送信 NAT 経由の送信アクセスが可能です。 この方法は、VM によって開始され、外部ネットワーク (インターネットまたはイントラネット) を接続先とする通信についてのみ有効です。 外部から VM と通信するために使用することはできません。

### <a name="hybrid-connectivity-options"></a>ハイブリッド接続のオプション

ハイブリッド接続に関して重要なことは、提供するデプロイの種類とデプロイ先の場所です。 ネットワーク トラフィックをテナントごとに分離する必要があるかどうか、イントラネットまたはインターネットをデプロイするかどうかを検討する必要があります。

- **シングル テナントの Azure Stack Hub**: 少なくともネットワークの観点からは 1 つのテナントと見なされる Azure Stack Hub のデプロイです。 多数のテナント サブスクリプションが存在できますが、イントラネット サービスと同様に、すべてのトラフィックが同じネットワークを経由して送信されます。 1 つのサブスクリプションからのネットワーク トラフィックは、別のサブスクリプションと同じネットワーク接続を経由して送信されます。暗号化されたトンネルを使用して分離する必要はありません。

- **マルチテナントの Azure Stack Hub**: テナント サブスクリプションごとに、Azure Stack Hub の外部ネットワークを送信先とするトラフィックを他のテナントのネットワーク トラフィックから分離する必要がある Azure Stack Hub のデプロイです。

- **イントラネット デプロイ**:一般的に 1 つ以上のファイアウォールの内側にあるプライベート IP アドレス空間である企業のイントラネット上に配置される Azure Stack Hub のデプロイです。 パブリック IP アドレスは、パブリック インターネット経由で直接ルーティングできないため、実際にはパブリックではありません。

- **インターネット デプロイ**: パグリック インターネットに接続され、インターネットでルーティング可能なパブリック IP アドレスをパブリック VIP 範囲として使用する Azure Stack Hub のデプロイです。 このデプロイはファイアウォールの内側への配置が可能でありながら、パブリック VIP 範囲はパブリック インターネットおよび Azure から直接到達可能です。

次の表は、ハイブリッド接続のシナリオの長所と短所、ユース ケースをまとめたものです。

| シナリオ | 接続方法 | 長所 | 短所 | 適した用途 |
| -- | -- | --| -- | --|
| シングル テナントの Azure Stack Hub、イントラネット デプロイ | 送信 NAT | 帯域幅向上による転送の高速化。 実装が簡単。ゲートウェイは必要ありません。 | トラフィックは暗号化されません。スタックの外部の分離や暗号化は行われません。 | すべてのテナントが平等に信頼されているエンタープライズ デプロイ。<br><br>Azure への Azure ExpressRoute 回線を所有している企業。 |
| マルチテナントの Azure Stack Hub、イントラネット デプロイ | サイト間 VPN | テナント VNet から送信先へのトラフィックはセキュリティで保護されます。 | 帯域幅がサイト間 VPN トンネルによって制限されます。<br><br>仮想ネットワークにゲートウェイが、また接続先ネットワークに VPN デバイスが必要です。 | 一部のテナント トラフィックを他のテナントから分離して保護する必要があるエンタープライズ デプロイ。 |
| シングル テナントの Azure Stack Hub、インターネット デプロイ | 送信 NAT | 帯域幅向上による転送の高速化。 | トラフィックは暗号化されません。スタックの外部の分離や暗号化は行われません。 | テナント専用に Azure Stack Hub をデプロイし、Azure Stack Hub 環境への専用回線を使用するホスティングのシナリオ。 例として、ExpressRoute とマルチプロトコル ラベル スイッチング (MPLS) があります。
| マルチテナントの Azure Stack Hub、インターネット デプロイ | サイト間 VPN | テナント VNet から送信先へのトラフィックはセキュリティで保護されます。 | 帯域幅がサイト間 VPN トンネルによって制限されます。<br><br>仮想ネットワークにゲートウェイが、また接続先ネットワークに VPN デバイスが必要です。 | プロバイダーがマルチテナント クラウドを提供する必要があるホスティング シナリオ。テナント相互の信頼関係がないため、トラフィックを暗号化する必要があります。
|  |  |  |  |  |

### <a name="using-expressroute"></a>ExpressRoute の使用

シングル テナントのイントラネットとマルチテナントの両方のシナリオで、[ExpressRoute](/azure/expressroute/expressroute-introduction) を経由して Azure Stack Hub を Azure に接続することができます。 [接続プロバイダー](/azure/expressroute/expressroute-locations)経由でプロビジョニングされている ExpressRoute 回線が必要です。

次の図は、シングル テナント シナリオの ExpressRoute ("顧客の接続" が ExpressRoute 回線) を示しています。

![シングル テナントの ExpressRoute シナリオを示した図](media/azure-stack-datacenter-integration/ExpressRouteSingleTenant.svg)

次の図は、マルチテナント シナリオの ExpressRoute を示しています。<br/><br/>

![マルチテナントの ExpressRoute シナリオを示した図](media/azure-stack-datacenter-integration/ExpressRouteMultiTenant.svg)

## <a name="external-monitoring"></a>外部の監視
Azure Stack Hub のデプロイとデバイスからすべてのアラートを 1 つのビューとして取得し、チケット発行用の既存の IT サービス マネジメント ワークフローにアラートを統合するために、[外部のデータセンター監視ソリューションと Azure Stack Hub を統合](azure-stack-integrate-monitor.md)することができます。

Azure Stack Hub ソリューションに含まれているハードウェア ライフサイクル ホストは、Azure Stack Hub の外部にあるコンピューターで、ハードウェアを対象にした OEM ベンダー提供の管理ツールを実行しています。 それらのツールを使用することも、データセンター内の既存の監視ソリューションに直接統合されたソリューションを使用することもできます。

次の表は、現在、使用可能なオプションの一覧です。

| 領域 | 外部の監視ソリューション |
| -- | -- |
| Azure Stack Hub ソフトウェア | [Operations Manager 用 Azure Stack Hub 管理パック](https://azure.microsoft.com/blog/management-pack-for-microsoft-azure-stack-now-available/)<br>[Nagios プラグイン](https://exchange.nagios.org/directory/Plugins/Cloud/Monitoring-AzureStack-Alerts/details)<br>REST ベースの API 呼び出し | 
| 物理サーバー (IPMI 経由 BMC) | OEM ハードウェア - Operations Manager ベンダー管理パック<br>OEM ハードウェア ベンダー提供のソリューション<br>ハードウェア ベンダーの Nagios プラグイン<br>OEM パートナーがサポートしている監視ソリューション (含まれている) | 
| ネットワーク デバイス (SNMP) | Operations Manager ネットワーク デバイスの検出<br>OEM ハードウェア ベンダー提供のソリューション<br>Nagios スイッチ プラグイン |
| テナント サブスクリプションの正常性の監視 | [Windows Azure 用 System Center 管理パック](https://www.microsoft.com/download/details.aspx?id=50013) | 
|  |  |

以下の要件に注意してください。

- 使用するソリューションは、エージェントレスである必要があります。 Azure Stack Hub コンポーネント内にサード パーティ製エージェントをインストールすることはできません。
- System Center Operations Manager を使用する場合は、Operations Manager 2012 R2 または Operations Manager 2016 が必要です。

## <a name="backup-and-disaster-recovery"></a>バックアップと障害復旧

バックアップとディザスター リカバリーの計画では、次の両方を計画に含める必要があります。つまり、基になる Azure Stack Hub インフラストラクチャ (IaaS VM と PaaS サービスをホストする) と、テナント アプリとデータです。 これらは別々に計画してください。

### <a name="protect-infrastructure-components"></a>インフラストラクチャ コンポーネントの保護

指定した SMB 共有に [Azure Stack Hub インフラストラクチャ コンポーネントをバックアップ](azure-stack-backup-back-up-azure-stack.md)できます。

- 既存の Windows ベース ファイル サーバーまたはサード パーティのデバイス上に、外部 SMB ファイル共有が必要です。
- この同じ共有を、ネットワーク スイッチとハードウェア ライフサイクル ホストのバックアップに使用します。 これらのコンポーネントは Azure Stack Hub の外部にあるため、これらのコンポーネントのバックアップと復元については、ご利用の OEM ハードウェア ベンダーに問い合わせてください。 OEM ベンダーの推奨に基づいたバックアップ ワークフローを実行することは、お客様の責任となります。

致命的なデータ消失が発生した場合は、インフラストラクチャのバックアップを使用して、次のようなデプロイ データを再生成できます。 

- デプロイの入力と ID
- サービス アカウント
- CA ルート証明書
- フェデレーション リソース (切断されたデプロイ内)
- プラン、オファー、サブスクリプション、クォータ
- RBAC のポリシーとロールの割り当て
- キー コンテナーのシークレット

### <a name="protect-tenant-apps-on-iaas-vms"></a>IaaS VM 上のテナント アプリの保護

Azure Stack Hub でテナント アプリとデータはバックアップされません。 Azure Stack Hub の外部にあるターゲットに対するバックアップとディザスター リカバリーの保護を計画する必要があります。 テナントの保護は、テナント ドリブン アクティビティです。 IaaS VM に対しては、テナントはゲスト内テクノロジを使用して、ファイル フォルダー、アプリ データ、システム状態を保護できます。 ただし、エンタープライズまたはサービス プロバイダーの場合は、同じデータセンター内または外部のクラウドでバックアップと復元のソリューションを提供することができます。

Linux または Windows の IaaS VM をバックアップするには、ゲスト オペレーティング システムへのアクセス権を持つバックアップ製品を使用して、ファイル、フォルダー、オペレーティング システムの状態、アプリ データを保護する必要があります。 Azure Backup、System Center Datacenter Protection Manager、またはサポートされているサード パーティ製品を使用できます。

セカンダリ ロケーションにデータをレプリケートし、障害発生時のアプリケーションのフェールオーバーを調整するには、Azure Site Recovery またはサポートされているサード パーティの製品を使用することができます。 また、ネイティブ レプリケーション (Microsoft SQL Server など) をサポートするアプリは、アプリが実行されている別の場所にデータをレプリケートできます。

## <a name="learn-more"></a>詳細情報

- ユース ケース、購入、パートナー、OEM ハードウェア ベンダーの詳細については、[Azure Stack Hub](https://azure.microsoft.com/overview/azure-stack/) の製品ページを参照してください。
- Azure Stack Hub 統合システムのロードマップと地理的な可用性については、ホワイト ペーパー「[Azure Stack Hub: An extension of Azure](https://azure.microsoft.com/resources/azure-stack-an-extension-of-azure/)」 (Azure Stack: Azure の拡張機能) を参照してください。 

## <a name="next-steps"></a>次のステップ
[Azure Stack Hub デプロイの接続モデル](azure-stack-connection-models.md)
