---
title: Azure Stack HCI ソリューションの概要
description: Azure Stack HCI は、オンプレミスのハイブリッド環境で仮想化された Windows および Linux のワークロードとそのストレージをホストするハイパーコンバージド インフラストラクチャ (HCI) クラスター ソリューションです。 Azure のハイブリッド サービスでは、クラウドベースの監視、Site Recovery、VM バックアップなどの機能、および Azure portal 内のすべての Azure Stack HCI デプロイの一元的なビューにより、クラスターが強化されます。
ms.topic: overview
author: khdownie
ms.author: v-kedow
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 09/24/2020
ms.openlocfilehash: 135cf9147b69410b4276860dcde2b167e771a695
ms.sourcegitcommit: 69cfff119ab425d0fbb71e38d1480d051fc91216
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/30/2020
ms.locfileid: "91572536"
---
# <a name="azure-stack-hci-solution-overview"></a>Azure Stack HCI ソリューションの概要

Azure Stack HCI は、オンプレミスのハイブリッド環境で仮想化された Windows および Linux のワークロードとそのストレージをホストするハイパーコンバージド インフラストラクチャ (HCI) クラスター ソリューションです。 Azure のハイブリッド サービスでは、クラウドベースの監視、Site Recovery、VM バックアップなどの機能、および Azure portal 内のすべての Azure Stack HCI デプロイの一元的なビューにより、クラスターが強化されます。 Windows Admin Center、System Center、PowerShell などの既存のツールを使用して、クラスターを管理できます。

Azure Stack HCI バージョン 20H2 は、現在パブリック プレビュー中の新しいオペレーティング システムであり、[ダウンロード可能](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)です。 その対象は、ハイブリッド クラウド接続が組み込まれ、仮想化されたワークロードが実行されている、オンプレミスのクラスターです。 そのため、Azure Stack HCI は Azure サービスとして提供され、Azure サブスクリプションで課金されます。 Azure Stack HCI には、Azure Kubernetes サービスをホストする機能も含まれるようになりました。詳細については、「[Azure Stack HCI の Azure Kubernetes Service](../aks-hci/overview.md)」を参照してください。

Azure Stack HCI の高レベルの機能については、数分時間を取ってビデオをご覧ください。

> [!VIDEO https://www.youtube.com/embed/fw8RVqo9dcs]

その中核である Azure Stack HCI は、次のものが組み合わされたソリューションです。

- OEM パートナーからの検証済みハードウェア
- Azure Stack HCI オペレーティング システム
- Windows Admin Center
- Azure サービス

:::image type="content" source="media/overview/azure-stack-hci-solution.png" alt-text="Azure Stack HCI OS は、検証済みハードウェア上で実行され、Windows Admin Center によって管理され、Azure に接続される" border="false":::

Azure Stack HCI バージョン 20H2 では、ストレージ コスト パフォーマンスが優れた記憶域スペース ダイレクトを使用するハイパーコンバージド クラスターを Windows Admin Center で作成する機能など、Windows Server には存在しない新機能が提供されています。 これには、サイト間の自動フェールオーバーのために、サイトをまたいでクラスターをストレッチするオプションが含まれます。 詳細については、「[Azure Stack HCI の新機能](#whats-new-in-azure-stack-hci)」を参照してください。

## <a name="use-cases-for-azure-stack-hci"></a>Azure Stack HCI のユース ケース

Azure Stack HCI には多くのユース ケースがありますが、仮想化されていないワークロードのためのものではありません。 多くの場合、お客様は次のようなシナリオで Azure Stack HCI を選択します。

### <a name="data-center-consolidation-and-modernization"></a>データ センターの統合と近代化

Azure Stack HCI を使用して老朽化した仮想化ホストを更新して統合すると、スケーラビリティが向上し、環境の管理とセキュリティ保護が容易になります。 また、レガシ SAN ストレージを退役させて、占有領域と総保有コストを削減するのにもよい機会です。 運用とシステムの管理は、統合されたツールとインターフェイス、および単一のサポートにより簡素化されます。

### <a name="remote-and-branch-offices"></a>リモート オフィスとブランチ オフィス

Azure Stack HCI では、1 箇所あたり 2 万ドル未満で 2 サーバーのクラスター ソリューションを始めることができ、リモート オフィスとブランチ オフィス、小売店、現場サイトの最新化を低コストで行うのに最適です。 革新的な入れ子になった回復性を使用すると、複数のハードウェアで同時に障害が発生した場合でも、ボリュームをオンラインでアクセス可能な状態に維持できます。 クラウド監視テクノロジを使用すると、クラスター クォーラムに対する軽量の決定機構として Azure を使用でき、3 番目のホストにコストをかけることなくスプリット ブレイン状況を防止できます。 また、お客様は、リモート環境の Azure Stack HCI のデプロイを Azure portal で一元的に見ることもできます。

### <a name="virtual-desktops"></a>仮想デスクトップ

多くの組織は、低待機時間とデータ主権のため、仮想デスクトップをオンプレミスでホストすることを望んでいます。 Azure Stack HCI では、ローカル環境と同様のパフォーマンスを提供できます。

### <a name="high-performance-virtualized-workloads"></a>高パフォーマンスの仮想化ワークロード

Azure Stack HCI では、百万単位のストレージ IOPS や 1 秒あたりデータベース トランザクション数を必要とする SQL Server データベースや他のパフォーマンス重視の仮想化ワークロードに対して、業界最高レベルのパフォーマンスを提供することができ、NVMe SSD、堅牢な RDMA スタック、および永続メモリにより一貫した低待機時間が提供されます。

## <a name="azure-integration-benefits"></a>Azure 統合の利点

ハイブリッド インフラストラクチャに関して Azure Stack HCI と同じような製品は他にはなく、クラウドとオンプレミスのリソースを連携させて利用し、監視、セキュリティ保護、クラウドに対するバックアップをネイティブに行うことができます。

Azure Stack HCI クラスターを Azure に登録した後は、最初に Azure portal を使用して以下のことができます。

- **監視:** すべての Azure Stack HCI クラスターを 1 つのグローバルなビューで表示し、リソース グループ別にグループ化したり、タグを付けたりします。
- **課金**: Azure サブスクリプションを通じて Azure Stack HCI の支払いを行います (パブリック プレビュー期間中は料金は発生しません)。

追加機能の作成に取り組んでいますので、注目していてください。

また、追加の Azure ハイブリッド サービスをサブスクライブすることもできます。

- **Azure Site Recovery**: 高可用性およびサービスとしてのディザスター リカバリー (DRaaS)。
- **Azure Monitor**: AI による高度な分析を使用して、お客様のアプリ、ネットワーク、インフラストラクチャ全体で起こっていることを追跡するための集中ハブ。
- **クラウド監視**: Azure をクラスター クォーラム用の軽量な決定機構として使用。
- **Azure Backup**: オフサイト データ保護およびランサムウェアに対する保護を提供。
- **Azure Update Management**: Azure およびオンプレミスで実行されている Windows VM に対する更新プログラムの評価と更新プログラムのデプロイ。
- **Azure Network Adapter**: ポイント対サイト VPN 経由で、オンプレミスのリソースを Azure 内のお客様の VM に接続。
- **Azure File Sync**: ファイル サーバーとクラウドを同期。

詳細については、「[Azure ハイブリッド サービスへの Windows Server の接続](/windows-server/manage/windows-admin-center/azure/index)」を参照してください。

## <a name="why-azure-stack-hci"></a>Azure Stack HCI にする理由

Azure Stack HCI は、既に大規模にデプロイされている実証済みのテクノロジ (Hyper-V、記憶域スペース ダイレクト、Azure によって触発されたソフトウェアによるネットワーク (SDN) など) に基づいて構築された、世界クラスの統合された仮想化スタックです。 お客様は以下のようなさまざまな理由で Azure Stack HCI を選択します。

- Hyper-V やサーバーの管理者にとっては使い慣れたものであり、既存の仮想化およびストレージの概念とスキルを活用できます。
- Microsoft System Center、Active Directory、グループ ポリシー、PowerShell スクリプトなど、データ センターの既存のプロセスとツールで動作します。
- バックアップ、セキュリティ、監視に関する一般的なサードパーティ製ツールで動作します。
- ハードウェアを柔軟に選択できるので、お客様は、最寄りの地域でサービスとサポートが最も優れたベンダーを選択できます。
- Microsoft とハードウェア ベンダーの共同サポートにより、カスタマー エクスペリエンスが向上します。
- シームレスで完全なスタックの更新により、最新の状態を簡単に維持できます。
- 柔軟で広範なエコシステムにより、IT プロフェッショナルはニーズに最適なソリューションを柔軟に構築できます。

## <a name="what-you-need-for-azure-stack-hci"></a>Azure Stack HCI に必要なもの

使い始めるには、次のものが必要です。

- 好みの Microsoft ハードウェア パートナーから購入した、[Azure Stack HCI カタログ](https://azure.microsoft.com/products/azure-stack/hci/catalog/)に掲載されている 2 台以上のサーバーのクラスター
- [Azure サブスクリプション](https://azure.microsoft.com/)
- クラスター内の各サーバーのインターネット接続。少なくとも 30 日ごとに HTTPS 送信トラフィック経由で次のエンドポイントに接続できること: *-azurestackhci-usage.azurewebsites.net
- 複数のサイトに拡張されるクラスターの場合は、サイト間に 1 Gb 以上の接続が必要であり (25 Gb の RDMA 接続を推奨)、両方のサイトで書き込みが同時に発生する同期レプリケーションを行う場合は、平均待機時間が 5 ミリ秒のラウンドトリップが必要です

詳細については、「[開始する前に](deploy/before-you-start.md)」を参照してください。 Azure Stack HCI での Azure Kubernetes Service の要件については、[Azure Stack HCI での AKS の要件](../aks-hci/overview.md#what-you-need-to-get-started)に関するページを参照してください。

## <a name="hardware-partners"></a>ハードウェア パートナー

好みの Microsoft パートナーから検証済みの Azure Stack HCI ソリューションを購入し、時間のかかる設計とビルド時間なしで稼働させることができます。 また、Microsoft パートナーは、単一窓口で実装およびサポート サービスの問い合わせに対応します。 検証済みのノードまたは統合システムを購入できます。これには、プレインストールされている Azure Stack HCI オペレーティング システムと共に、ドライバーとファームウェアの更新プログラムに対するパートナー拡張機能も含まれます。

[Azure Stack HCI ソリューション](https://azure.microsoft.com/overview/azure-stack/hci)のページにアクセスするか、[Azure Stack HCI カタログ](https://azure.microsoft.com/products/azure-stack/hci/catalog/)を参照してください。現在、ASUS、Axellio、Blue Chip、DataON、Dell EMC、富士通、HPE、日立、Huawei、Lenovo、NEC、primeLine Solutions、QCT、SecureGUARD、Supermicro などの Microsoft パートナーの、70 以上の Azure Stack HCI ソリューションを利用できます。

## <a name="software-partners"></a>ソフトウェア パートナー

さまざまな Microsoft パートナーが、IT 管理者が使い慣れたツールを使用できるようにと、Azure Stack HCI の機能を拡張するソフトウェアの作業を行っています。 たとえば、バックアップ ソリューションの世界規模のプロバイダーであり Microsoft ゴールド パートナーの Altaro は、その Altaro VM Backup ソリューションで Azure Stack HCI をサポートすることを約束しています。 これにより、お客様や管理サービス プロバイダーは、2021 年 6 月末まで、Azure Stack HCI で実行されている仮想マシンを無料でバックアップできます。 [この発表の詳細についてはこちらをご覧ください](http://www.altaro.com/news/single/News-Altaro-applies-its-expertise-in-Hyper-V-backup-to-support-Microsoft.php)。

## <a name="licensing-billing-and-pricing"></a>ライセンス、請求、価格

Azure Stack HCI の請求は、永続的ライセンスではなく、物理プロセッサ コアあたりの月額サブスクリプション料金に基づきます。 お客様が Azure に接続すると、使用されているコアの数が自動的にアップロードされ、請求のために評価されます。 コストは、物理プロセッサ コア以外の消費量では変わりません。つまり、VM の数が増えてもコストが高くなることはなく、より高密度の仮想環境を実行できるお客様ほど得をします。

お客様は、Azure Stack HCI オペレーティング システムがプレインストールされている検証済みサーバーをハードウェア パートナーから購入できます。または、OEM から検証済みベア メタル サーバーを購入した後、Azure Stack HCI サービスをサブスクライブし、[Azure portal](https://azure.microsoft.com/products/azure-stack/hci/) から Azure Stack HCI オペレーティング システムをダウンロードすることができます。

## <a name="management-tools"></a>管理ツール

Azure Stack HCI では、お客様がクラスターに対する完全な管理者権限を持ち、そのテクノロジをすべて直接管理できます。

- [Hyper-V](/windows-server/virtualization/hyper-v/hyper-v-on-windows-server)
- [記憶域スペース ダイレクト](/windows-server/storage/storage-spaces/storage-spaces-direct-overview)
- [ソフトウェアによるネットワーク](/windows-server/networking/sdn/)
- [フェールオーバー クラスタリング](/windows-server/failover-clustering/failover-clustering-overview)

これらのテクノロジの管理には、次の管理ツールを使用できます。

- [Windows Admin Center](/windows-server/manage/windows-admin-center/overview)
- [System Center](https://www.microsoft.com/cloud-platform/system-center)
- [PowerShell](/powershell/)
- [サーバー マネージャー](/windows-server/administration/server-manager/server-manager)や MMC スナップインなどの他の管理ツール
- 5Nine Manager などの Microsoft 以外のツール

## <a name="faq"></a>よく寄せられる質問

### <a name="how-does-azure-stack-hci-relate-to-windows-server"></a>Azure Stack HCI は Windows Server とどのような関係になっていますか。

Windows Server はほぼすべての Azure 製品の基盤であり、お持ちのすべての機能は引き続き出荷され、Windows Server でサポートされます。 Azure Stack HCI の初期オファリングは、Windows Server 2019 が基になっており、従来の Windows Server ライセンス モデルを使用していました。 現在の Azure Stack HCI には、独自のオペレーティング システムとサブスクリプション ベースのライセンス モデルがあります。 Azure Stack HCI は、弊社のパートナーから提供され、Microsoft による検証を受けたハードウェアを使用して HCI をオンプレミスでデプロイするためのお勧めの方法です。

### <a name="does-azure-stack-hci-need-to-connect-to-azure"></a>Azure Stack HCI は Azure に接続する必要がありますか。

はい。クラスターは、課金のためにコア数を評価するため、少なくとも 30 日ごとに 1 回、Azure に接続する必要があります。 また、オフサイトのバックアップやディザスター リカバリー、クラウドベースの監視と更新管理などのハイブリッド シナリオのために Azure との統合を利用できますが、それは省略可能です。 長期間インターネットから切断されている状態で実行しても問題ありません。

### <a name="can-i-upgrade-from-windows-server-2019-to-azure-stack-hci"></a>Windows Server 2019 から Azure Stack HCI にアップグレードできますか。

現時点では、Windows Server から Azure Stack HCI へのインプレース アップグレードはありません。 Windows Server 2019 および 2016 に基づくハイパーコンバージド クラスターを実行しているお客様向けの具体的な移行ガイダンスについては、しばらくお待ちください。

### <a name="what-do-azure-stack-hub-and-azure-stack-hci-solutions-have-in-common"></a>Azure Stack Hub と Azure Stack HCI ソリューションはどのような点が共通していますか。

Azure Stack HCI には、Azure Stack Hub と同じ、Hyper-V ベースのソフトウェアによるコンピューティング、ストレージ、ネットワーク テクノロジが搭載されています。 どちらのオファリングも、厳しいテストおよび検証基準を満たしており、信頼性および基盤となるハードウェア プラットフォームとの互換性が確保されています。

### <a name="how-are-they-different"></a>これらはどのように違うのでしょうか。

Azure Stack Hub では、オンプレミスでクラウド サービスを運用します。 Azure IaaS および PaaS サービスをオンプレミスで実行して、一貫性を保ちながら任意の場所でクラウド アプリを構築および実行し、オンプレミスで Azure portal を使用して管理できます。

Azure Stack HCI では、仮想化されたワークロードをオンプレミスで実行し、Windows Admin Center と使い慣れた Windows Server ツールを使用して管理します。 また、クラウドベースのサイトの回復、監視などのハイブリッド シナリオのために Azure に接続できます。

### <a name="why-is-microsoft-bringing-its-hci-offering-to-the-azure-stack-family"></a>Microsoft が HCI オファリングを Azure Stack ファミリに追加するのはなぜですか。

Microsoft のハイパーコンバージド テクノロジは既に Azure Stack Hub の基盤となっています。

Microsoft のお客様の多くは複雑な IT 環境を抱えています。Microsoft のゴールは、そうした環境に対応するソリューションを、ビジネス ニーズに合った適切なテクノロジと共に提供することです。 Azure Stack HCI は、以前は弊社のハードウェア パートナーから提供されていた Windows Server のソフトウェアによる (WSSD) ソリューションを進化させたものです。 これを Azure Stack ファミリに追加したのは、インフラストラクチャ管理サービスのために Azure とシームレスに接続する新しいオプションの提供を開始したからです。

### <a name="can-i-upgrade-from-azure-stack-hci-to-azure-stack-hub"></a>Azure Stack HCI から Azure Stack Hub にアップグレードできますか。

いいえ。ただし、お客様は Azure Stack HCI から Azure Stack Hub または Azure にワークロードを移行できます。

### <a name="what-azure-services-can-i-connect-to-azure-stack-hci"></a>どの Azure サービスを Azure Stack HCI に接続できますか。

Azure Stack HCI を接続できる Azure サービスの最新一覧については、「[Connecting Windows Server to Azure hybrid services (Windows Server と Azure ハイブリッド サービスの接続)](/windows-server/manage/windows-admin-center/azure/index)」をご覧ください。

### <a name="does-azure-stack-hci-collect-any-data-from-my-system"></a>Azure Stack HCI ではユーザーのシステムからデータが収集されますか。

はい、非常に限られたデータのセットが収集されます。 このデータは、適切に動作するように HCI を最新の状態に保ち、情報を Azure portal に提供し、請求のためにクラスター内のプロセッサ コアの数を評価するために使用されます。

### <a name="to-which-endpoints-is-the-data-transmitted"></a>データはどのエンドポイントに送信されますか。  

Azure Stack HCI では、請求データの送信に次のエンドポイントが使用されます: *-azurestackhci-usage.azurewebsites.net

### <a name="how-do-i-identify-an-azure-stack-hci-server"></a>Azure Stack HCI サーバーを識別するにはどうすればよいですか。

Windows Admin Center では、[すべての接続] の一覧や他のさまざまな場所に、オペレーティング システムの一覧が表示されます。または、次の PowerShell コマンドを使用して、オペレーティング システムの名前とバージョンを照会することもできます。

```PowerShell
Get-ComputerInfo -Property 'osName', 'osDisplayVersion'
```

出力例を次に示します。

```
OsName                    OSDisplayVersion
------                    ----------------
Microsoft Azure Stack HCI 20H2
```

## <a name="the-azure-stack-family"></a>Azure Stack ファミリ

Azure Stack HCI は、Azure と Azure Stack ファミリの一員であり、Azure Stack Hub と同じ、ソフトウェアによるコンピューティング、ストレージ、およびネットワーク ソフトウェアを利用します。 さまざまなソリューションの簡単な概要を次に示します。 詳細については、[Azure Stack エコシステムの比較](../operator/compare-azure-azure-stack.md)に関するページを参照してください。

- [Azure](https://azure.microsoft.com) - オンデマンドのセルフサービス コンピューティング リソースのためのパブリック クラウド サービスを利用して、既存のアプリを移行して最新化し、新しいクラウドネイティブ アプリを構築します。
- [Azure Stack Edge](/azure/databox-online/data-box-edge-overview) - 機械学習ワークロードを加速し、オンプレミスのコンテナー化されたアプリや仮想化されたワークロードをクラウド管理アプライアンスで実行します。
- [Azure Stack HCI](https://azure.microsoft.com/overview/azure-stack/hci) - 仮想化されたアプリをオンプレミスで実行し、古くなったサーバー インフラストラクチャを交換および整理し、クラウド サービスを利用するために Azure に接続します。
- [Azure Stack Hub](../operator/azure-stack-overview.md) - 接続していない場合や、規制要件を満たすために、一貫性のある Azure サービスを使用し、クラウド アプリをオンプレミスで実行します。

:::image type="content" source="media/overview/azure-family-updated.png" alt-text="Azure Stack HCI OS は、検証済みハードウェア上で実行され、Windows Admin Center によって管理され、Azure に接続される" border="false":::

## <a name="compare-windows-server-and-azure-stack-hci"></a>Windows Server と Azure Stack HCI を比較する

多くのお客様は、Windows Server または Azure Stack HCI が自分のニーズに合っているかどうか迷います。 次の表は、組織に適したものを判断するのに役立ちます。 Windows Server と Azure Stack HCI のどちらでも、新しいリリースのロード マップに従って、同じ高品質のユーザー エクスペリエンスが提供されます。

| Windows Server | Azure Stack HCI |
| --------------- | --------------- |
| ゲストおよび従来のサーバーとして最適です | 記憶域スペース ダイレクトなど、ソフトウェアによるデータ センター用の仮想化ホストに最適です |
| 従来のソフトウェア ライセンス モデルを使用して、どこでも実行できます | 好みのベンダーのハードウェアで実行できますが、Azure サービスとして提供され、Azure アカウントに課金されます |
| 2 つのインストール オプション:デスクトップ エクスペリエンスを備えたサーバーまたはサーバー コア | わずかにカスタマイズされたサーバー コアに基づきます |

### <a name="when-to-use-windows-server"></a>Windows Server を使用する場合

| Windows Server | Azure Stack HCI |
| --------------- | --------------- |
| Windows Server は、非常に汎用性のある多目的なオペレーティング システムであり、ゲスト権限など、数十の役割と数百の機能を備えています。 | Azure Stack HCI にはゲスト権限は含まれておらず、最新のハイパーコンバージド アーキテクチャに使用することが意図されています。 |
| VM を実行する場合、または Active Directory、ファイル サービス、DNS、DHCP、インターネット インフォメーション サービス (IIS)、コンテナー ホストおよびゲスト、SQL Server、Exchange Server、ホスト ガーディアン サービス (HGS) など、従来のすべてのサーバーの役割を含むベア メタル インストール用には、Windows Server を使用します。 | Hyper-V 仮想化ホストとして意図されている Azure Stack HCI は、少数のサーバーの役割を直接実行するためだけにライセンスを付与されます。他のロールはすべて、VM 内で実行する必要があります。 |

### <a name="when-to-use-azure-stack-hci"></a>Azure Stack HCI を使用する局面

| Windows Server | Azure Stack HCI |
| --------------- | --------------- |
| Windows Server はオンプレミスまたはクラウドで実行できますが、それ自体は完全なハイパーコンバージド オファリングではありません。| オンプレミスで VM を実行し、必要に応じて 2 つのサイトに拡張して Azure ハイブリッド サービスへの接続を使用するには、Azure Stack HCI を使用します。 データ センターやブランチ オフィスを最新化してセキュリティで保護し、SQL Server データベースで業界最高のパフォーマンスを実現し、低待機時間とデータ主権のためにオンプレミスで仮想デスクトップを実行するための、簡単な方法です|
| Windows Server は、仮想化されているかどうかに関係なく、Windows Server のすべての役割を対象とする優れた多目的の "万能製品" です。 | Exchange、SharePoint、SQL Server などの従来のエンタープライズ アプリを仮想化する場合、またはファイル サーバー、DNS、DHCP、IIS、AD などの Windows Server の役割を仮想化する場合は、Azure Stack HCI を使用します。 シールドされた VM など、すべての Hyper-V 機能への無制限のアクセスが含まれます。|
| Windows Server の多くの展開は、古いハードウェアで実行されています。 | 大規模な再設計なしで、古いストレージ アレイやネットワーク アプライアンスの代わりにソフトウェアによるインフラストラクチャを使用する場合に、Azure Stack HCI を使用します。 組み込みの Hyper-V、記憶域スペース ダイレクト、およびソフトウェアによるネットワーク (SDN) は直接アクセスして管理できます。 Windows VM または Linux VM 内でアプリを実行します。|

## <a name="compare-azure-stack-hub-and-azure-stack-hci"></a>Azure Stack Hub と Azure Stack HCI の比較

お客様の組織がデジタル変革を進める中で、パブリック クラウド サービスを使用して、最新のアーキテクチャ上に構築し、従来のアプリを更新することで、より迅速に前進できることに気付くことがあります。 ただし、技術上の障害や規制の問題などの理由により、多くのワークロードをオンプレミスに残す必要があります。 次の表を使用すると、必要なところに必要なものを提供し、ワークロードの場所に関係なくそのクラウド イノベーションを実現する Microsoft ハイブリッド クラウド戦略を判断するのに役立ちます。

| Azure Stack Hub | Azure Stack HCI |
| --------------- | --------------- |
| 新しいスキル、革新的なプロセス | 同じスキル、使い慣れたプロセス |
| お客様のデータセンター内での Azure サービス | お客様のデータセンターを Azure サービスに接続する |

### <a name="when-to-use-azure-stack-hub"></a>Azure Stack Hub を使用する局面

| Azure Stack Hub | Azure Stack HCI |
| --------------- | --------------- |
| 併置された複数のテナントの強力な分離および正確な使用状況の追跡とチャージバックを実現するセルフサービスのインフラストラクチャとしてのサービス (IaaS) に Azure Stack Hub を使用します。 サービス プロバイダーやエンタープライズ プライベート クラウドに最適です。 テンプレートは Azure Marketplace にあります。 | Azure Stack HCI では、マルチテナントをネイティブに適用したり、提供したりすることはありません。 |
| オンプレミスで Web Apps、Functions、Event Hubs などのプラットフォームとしてのサービス (PaaS) サービスを利用するアプリの開発と実行に Azure Stack Hub を使用します。 これらのサービスは、Azure 内とまったく同じように Azure Stack Hub 上で実行され、一貫性のあるハイブリッドの開発およびランタイム環境を提供します。 | Azure Stack HCI では、PaaS サービスをオンプレミスで実行しません。 |
| DevOps プラクティス (コードとしてのインフラストラクチャ、継続的インテグレーションと継続的デプロイ (CI/CD) など) と便利な機能 (Azure と整合性のある VM 拡張機能など) を利用したアプリのデプロイと操作の現代化に Azure Stack Hub を使用します。 開発チームや DevOps チームに最適です。 | Azure Stack HCI には、DevOps ツールがネイティブに含まれていません。 |

### <a name="when-to-use-azure-stack-hci"></a>Azure Stack HCI を使用する局面

| Azure Stack Hub | Azure Stack HCI |
| --------------- | --------------- |
| Azure Stack Hub には、最小 4 つのノードとそれ専用のネットワーク スイッチが必要です。 | リモート オフィスやブランチの場合に占有領域を最小限に抑えるには Azure Stack HCI を使用します。 たった 2 個のサーバー ノードとスイッチレスのバックツーバック ネットワークから開始でき、ピークがわかりやすく、価格が手頃です。 ハードウェアのオファーは、ノードあたり 4 台のドライブ、64 GB のメモリ、10,000 ドル未満からです。 |
| Azure Stack Hub では、Azure との整合性を確保するために、Hyper-V の構成機能と機能セットが制限されています。 | Exchange、SharePoint、SQL Server などの従来のエンタープライズ アプリ向けの必要最低限の Hyper-V 仮想化や、ファイル サーバー、DNS、DHCP、IIS、AD などの Windows Server ロールの仮想化に、Azure Stack HCI を使用します。 シールドされた VM など、すべての Hyper-V 機能に無制限にアクセスできます。|
| Azure Stack Hub では、これらのインフラストラクチャ テクノロジは公開されません。 | 大規模な再設計なしで、古いストレージ アレイやネットワーク アプライアンスの代わりにソフトウェアによるインフラストラクチャを使用する場合に、Azure Stack HCI を使用します。 組み込みの Hyper-V、記憶域スペース ダイレクト、およびソフトウェアによるネットワーク (SDN) は直接アクセスして管理できます。 |

## <a name="whats-new-in-azure-stack-hci"></a>Azure Stack HCI の新機能

Windows Admin Center バージョン 2009 では、次のような多数の機能が Azure Stack HCI に追加されています。

- **Azure Kubernetes Service のホスティング機能**:[Azure Stack HCI 上の Azure Kubernetes Service](https://azure.microsoft.com/products/azure-stack/hci/hci-download/) のプレビュー バージョンをインストールできるようになりました。
- **クラスター作成ウィザードでのソフトウェア定義ネットワークの追加**:クラスター作成ウィザードで、[クラスターの作成](deploy/create-cluster.md#step-5-sdn-optional)中に、[ソフトウェア定義ネットワーク (SDN)](concepts/software-defined-networking.md) ネットワーク コントローラーの機能をデプロイするオプションが追加されました。

Windows Admin Center の新機能の詳細については、[Windows Admin Center のブログ](https://techcommunity.microsoft.com/t5/windows-admin-center-blog/bg-p/Windows-Admin-Center-Blog)を参照してください。

Windows Server 2019 ベースのソリューションと比較して、Azure Stack HCI バージョン 20H2 を実行するクラスターには、次の新機能があります。

- **Windows Admin Center の新機能**: 直感的な UI を使用してハイパーコンバージド クラスターを作成および更新する機能により、Azure Stack HCI はこれまで以上に簡単に使用できるようになりました。
- **自動フェールオーバーのためのストレッチ クラスター**: 記憶域レプリカ レプリケーションと自動 VM フェールオーバーを使用するマルチサイト クラスタリングでは、記憶域スペース ダイレクトを使用するクラスターに対して、ネイティブのディザスター リカバリーとビジネス継続性が提供されます。
- **アフィニティと非アフィニティのルール**: これらは、ストレッチ クラスターなどの複数の障害ドメインを持つクラスターで、VM とストレージをまとめて、または分離して維持するために Azure で Availability Zones が使用される方法と同様に使用できます。
- **Azure portal の統合**: Azure Stack HCI に対する Azure portal のエクスペリエンスは、開発中の新機能を使用して、世界中のすべての Azure Stack HCI クラスターを表示するように設計されています。
- **高パフォーマンス ワークロード用の GPU アクセラレーション**: AI/ML アプリケーションでは、GPU を使用したパフォーマンスの向上によってメリットがあります。
- **BitLocker 暗号化**: BitLocker を使用して Azure Stack HCI 上のデータ ボリュームの内容を暗号化できるようになりました。これにより、政府や他のお客様は FIPS 140-2 や HIPAA などの標準に準拠した状態を維持できます。
- **記憶域スペース ダイレクト ボリュームの修復速度の向上**: すばやくシームレスにボリュームを修復します。

Windows Admin Center バージョン 20H2 では、Windows Server ベースのクラスターに対する新しいクラスター更新 UI も提供されています (元の Azure Stack HCI ソリューションを含む)。 また、Windows Server では新しいクラスター作成ウィザードを使用できますが、記憶域スペース ダイレクトを使用して Windows Server クラスターを作成することはできません。そのためには、Azure Stack HCI オペレーティング システムが必要です。

## <a name="roles-you-can-run-without-virtualizing"></a>仮想化なしで実行できる役割

Azure Stack HCI は、仮想マシンですべてのワークロードを実行する仮想化ホストとして意図されているため、Azure Stack HCI の条件では、仮想マシンをホストするために必要なものしか実行できません。

これは、次のサーバーの役割を実行できることを意味します。

- Hyper-V
- ソフトウェアによるネットワーク (SDN) に必要なネットワーク コントローラーおよびその他のコンポーネント

一方、その他の役割とアプリは VM 内で実行する必要があります。 ホストされている VM の管理と正常性に必要なユーティリティ、アプリ、サービスを実行できることに注意してください。

## <a name="video-based-learning"></a>ビデオでの学習

Azure 拡張ネットワーク デオは次のとおりです。

- [Windows Server とハイブリッド ットワークを使用した Azure へのシームレスな接続](https://www.youtube.com/watch?v=do2_4Y2p9dk)

Azure Stack HCI の元の Windows Server 2019 ベースのバージョンに関するビデオ:

- [Azure Stack および Azure Stack HCI の概要ビュー](https://aka.ms/AzureStackOverviewVideo)
- [Microsoft Ignite Live 2019 - Azure Stack HCI の概要](https://www.youtube.com/watch?v=vueHIBqNIEU)
- [Azure Stack HCI の探索](https://www.youtube.com/watch?v=4aGZK0Ndmh8&list=PLQXpv_NQsPICdXZoH-EzlIFa4P6VS5m11&index=13&t=0s)
- [Azure Stack HCI による小売店やブランチ オフィスの最新化](https://www.youtube.com/watch?v=-JzLhjfkhmM&list=PLQXpv_NQsPICdXZoH-EzlIFa4P6VS5m11&index=9&t=0s)
- [Azure Stack HCI の新機能: 45 分で 45 の機能をご紹介](https://www.youtube.com/watch?v=C5J4IEnlS_E&list=PLQXpv_NQsPICdXZoH-EzlIFa4P6VS5m11&index=12&t=0s)
- [Azure Stack HCI のデプロイをすぐに開始する](https://www.youtube.com/watch?v=gxaPJLrWy5w&list=PLQXpv_NQsPICdXZoH-EzlIFa4P6VS5m11&index=11&t=0s)
- [データ圧縮のケース: Azure Stack HCI でのデータ重複除去](https://www.youtube.com/watch?v=fmm4iDbDiY4&list=PLQXpv_NQsPICdXZoH-EzlIFa4P6VS5m11&index=23&t=0s)
- [Azure Stack HCI のフィールドからの Dave Kawula のメモ](https://www.youtube.com/watch?v=OXv7fLlz0ew&list=PLQXpv_NQsPICdXZoH-EzlIFa4P6VS5m11&index=2&t=0s)

ハイブリッド クラウド仮想イベントのビデオをご紹介します。

- [Azure Stack HCI | ハイブリッド クラウド仮想イベント](https://www.youtube.com/watch?v=nxpoEva-R2Y)

## <a name="next-steps"></a>次のステップ

- [Azure Stack HCI をダウンロードする](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)
- [Windows Admin Center で Azure Stack HCI を使用する](get-started.md)
