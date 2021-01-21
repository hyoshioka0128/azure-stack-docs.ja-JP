---
title: Azure Stack HCI の物理ネットワーク要件
description: Azure Stack HCI の物理ネットワーク要件について
author: v-dasis
ms.topic: how-to
ms.date: 11/30/2020
ms.author: v-dasis
ms.reviewer: JasonGerend
ms.openlocfilehash: e26299a53583fa7d4e4b4b61fc46be7d2faa9eba
ms.sourcegitcommit: 9b0e1264ef006d2009bb549f21010c672c49b9de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98255012"
---
# <a name="physical-network-requirements-for-azure-stack-hci"></a>Azure Stack HCI の物理ネットワーク要件

> 適用対象: Azure Stack HCI バージョン 20H2

このトピックでは、Azure Stack HCI の物理 (ファブリック) ネットワークに関する考慮事項と要件、特にネットワーク スイッチについて説明します。

> [!NOTE]
> Azure Stack HCI バージョンの要件は、将来変更される可能性があります。

## <a name="network-switches-for-azure-stack-hci"></a>Azure Stack HCI のネットワーク スイッチ

Microsoft では、以下の「**ネットワーク スイッチの要件**」セクションで特定された標準とプロトコルに対して Azure Stack HCI をテストします。 Microsoft ではネットワーク スイッチを認定していませんが、ベンダーと協力して、Azure Stack HCI 要件をサポートするデバイスを特定します。

これらの要件は、「[Windows ハードウェア互換性プログラムの仕様とポリシー](/windows-hardware/design/compatibility/whcp-specifications-policies)」でも公開されています。  **[仕様とポリシー、バージョン 1809 をダウンロードする]** を選択し、ZIP ファイルを開き、**WHCP-Components-Peripherals-Specification-1809.pdf** を開いて、「**Device.Network.Switch.SDDC**」セクションを参照します。

> [!IMPORTANT]
> ここに記載されていないテクノロジやプロトコルを使用するその他のネットワーク スイッチも動作する可能性がありますが、Microsoft は Azure Stack HCI での動作を保証できません。また、発生する問題のトラブルシューティングをサポートできないおそれがあります。

ネットワーク スイッチを購入する際には、スイッチのベンダーに問い合わせて、デバイスがすべての Azure Stack HCI 要件を満たしていることを確認してください。 以下のベンダー (アルファベット順) は、自社のスイッチが Azure Stack HCI の要件に対応していることを確認しています。

|ベンダー|10 GbE|25 GbE|100 GbE|
|-----|-----|-----|-----|
|Dell|[S41xx シリーズ](https://www.dell.com/learn/us/en/45/shared-content~data-sheets~en/documents~dell-emc-networking-s4100-series-spec-sheet.pdf)|[S52xx シリーズ](https://www.delltechnologies.com/resources/en-us/asset/data-sheets/products/networking/dell_emc_networking-s5200_on_spec_sheet.pdf)|[S52xx シリーズ](https://www.delltechnologies.com/resources/en-us/asset/data-sheets/products/networking/dell_emc_networking-s5200_on_spec_sheet.pdf)|
|Lenovo|[G8272](https://lenovopress.com/tips1267-lenovo-rackswitch-g8272)、[NE1032](https://lenovopress.com/lp0605-thinksystem-ne1032-rackswitch)|[NE2572](https://lenovopress.com/lp0608-lenovo-thinksystem-ne2572-rackswitch) |[NE10032](https://lenovopress.com/lp0609-lenovo-thinksystem-ne10032-rackswitch) |

> [!IMPORTANT]
> Microsoft では、ネットワーク スイッチのベンダーからの変更通知を受けて、この一覧を更新しています。

お使いのスイッチが含まれていない場合は、スイッチのベンダーに問い合わせて、スイッチのモデルとスイッチのオペレーティング システムのバージョンが、次のセクションの要件に対応していることを確認してください。

## <a name="network-switch-requirements"></a>ネットワーク スイッチの要件

このセクションでは、すべての Azure Stack HCI のデプロイで使用されるネットワーク スイッチに必須の業界標準を列挙して説明します。 これらの標準は、Azure Stack HCI クラスターのデプロイにおいて、ノード間に信頼性の高い通信を確保するのに役立ちます。

> [!NOTE]
> コンピューティング、ストレージ、管理の各トラフィックに使用されるネットワーク アダプターには、イーサネットが必要です。 詳細については、「[ホスト ネットワークの要件](host-network-requirements.md)」を参照してください。

必須の IEEE 標準と仕様は次のとおりです。

### <a name="standard-ieee-8021q"></a>標準: IEEE 802.1Q

イーサネット スイッチは、VLAN を定義する IEEE 802.1 Q 仕様に準拠している必要があります。 VLAN は Azure Stack HCI のいくつかの側面に必要であり、すべてのシナリオで必要です。

### <a name="standard-ieee-8021qbb"></a>標準: IEEE 802.1Qbb

イーサネット スイッチは、プライオリティ フロー制御 (PFC) を定義する IEEE 802.1Qbb 仕様に準拠している必要があります。 PFC は、データ センター ブリッジング (DCB) を使用する場合に必要です。 DCB は RoCE と iWARP RDMA の両方のシナリオで使用できるため、802.1 Qbb はすべてのシナリオで必要です。 スイッチ機能またはポート速度をダウングレードせずに、少なくとも 3 つのサービスのクラス (CoS) の優先順位が必要です。 これらのトラフィック クラスのうち少なくとも 1 つは、無損失通信を提供する必要があります。

### <a name="standard-ieee-8021qaz"></a>標準: IEEE 802.1Qaz

イーサネット スイッチは、拡張送信選択 (ETS) を定義する IEEE 802.1Qaz 仕様に準拠している必要があります。 ETS は、DCB を使用する場合に必要です。 DCB は RoCE と iWARP RDMA の両方のシナリオで使用できるため、802.1Qaz はすべてのシナリオで必要です。 スイッチ機能またはポート速度をダウングレードせずに、少なくとも 3 つの CoS の優先順位が必要です。

> [!NOTE]
> ハイパーコンバージド インフラストラクチャは、同じラック内での East-West レイヤー 2 通信への依存度が高いため、ETS が必要となります。 Microsoft では、DSCP (Differentiated Services Code Point) を使用した Azure Stack HCI のテストを行っていません。 

### <a name="standard-ieee-8021ab"></a>標準: IEEE 802.1AB

イーサネット スイッチは、Link Layer Discovery Protocol (LLDP) を定義する IEEE 802.1AB 仕様に準拠している必要があります。 LLDP は Azure Stack HCI に必要であり、物理ネットワーク構成のトラブルシューティングにも使用できます。

LLDP の Type-Length-Values (TLV) の構成は、動的に有効にする必要があります。 スイッチには、特定の TLV を有効化する以外に、追加の構成は必要ありません。 たとえば、802.1 サブタイプ 3 を有効にすると、スイッチ ポートで使用可能なすべての VLAN が自動的にアドバタイズされます。

### <a name="custom-tlv-requirements"></a>カスタム TLV の要件

LLDP を使用すると、組織は独自のカスタム TLV を定義してエンコードすることができます。 これらは、組織固有の TLV と呼ばれます。 すべての組織固有の TLV は、127 という LLDP TLV Type 値で開始されます。 次の表に、どのような組織固有のカスタム TLV (TLV Type 127) サブタイプが必要であるかを示します。

|必須のバージョン|Organization|TLV サブタイプ|
|-----|-----|-----|-----|
|20H2 以降|IEEE 802.1|VLAN 名 (サブタイプ = 3)|
|20H2 以降|IEEE 802.3|最大フレーム サイズ (サブタイプ = 4)|

## <a name="network-traffic-and-architecture"></a>ネットワーク トラフィックとアーキテクチャ

このセクションは、主にネットワーク管理者を対象としています。

Azure Stack HCI は、2 層 (Spine-Leaf) や 3 層 (Core-Aggregation-Access) など、さまざまなデータ センター アーキテクチャで機能することができます。 このセクションでは、Azure Stack HCI などのハイパーコンバージド インフラストラクチャのワークロードで一般的に使用される Spine-Leaf トポロジの概念を詳しく説明します。

## <a name="network-models"></a>ネットワーク モデル

ネットワーク トラフィックは、その方向によって分類することができます。 従来の記憶域ネットワーク (SAN) 環境は、トラフィックの向きは主に North-South であり、トラフィックは、レイヤー 3 (IP) 境界を越えて、コンピューティング層からストレージ層に流れます。 ハイパーコンバージド インフラストラクチャは、East-West への依存がより高く、トラフィックの大部分がレイヤー 2 (VLAN) 境界内にとどまります。

> [!IMPORTANT]
>サイト内のすべてのクラスター ノードを物理的に同じラック内に配置し、同じ Top-of-Rack (ToR) スイッチに接続することを強くお勧めします。

### <a name="north-south-traffic-for-azure-stack-hci"></a>Azure Stack HCI の North-South トラフィック

North-South トラフィックには次のような特徴があります。

- トラフィックは ToR スイッチからスパインに流れるか、スパインから ToR スイッチに流れる
- トラフィックは物理ラックから出ていくか、レイヤー 3 境界 (IP) を越える
- 管理 (PowerShell、Windows Admin Center)、コンピューティング (VM)、サイト間のストレッチ クラスターの各トラフィックを含む
- 物理ネットワークへの接続にイーサネット スイッチを使用する

### <a name="east-west-traffic-for-azure-stack-hci"></a>Azure Stack HCI の East-West トラフィック

East-West トラフィックには次のような特徴があります。

- トラフィックは ToR スイッチとレイヤー 2 境界 (VLAN) 内にとどまる
- 同じクラスター内と (ストレッチ クラスターを使用している場合は) 同じサイト内のノード間で、ストレージ トラフィックまたはライブ マイグレーション トラフィックを含む
- 次の 2 つのセクションで説明するように、イーサネット スイッチ (スイッチ) または直接 (スイッチレス) 接続を使用可能

## <a name="using-switches"></a>スイッチの使用

North-South トラフィックでは、スイッチを使用する必要があります。 Azure Stack HCI に必要なプロトコルをサポートするイーサネット スイッチを使用することに加え、最も重要なのはネットワーク ファブリックの適切なサイズ設定です。

イーサネット スイッチがサポートできる「非ブロッキング」ファブリック帯域幅を理解し、ネットワークのオーバーサブスクリプションを最小化 (または可能であれば排除) することが不可欠です。

一般的な輻輳ポイントとオーバーサブスクリプション (パス冗長性に使用される[マルチシャーシ リンク集計グループ](https://en.wikipedia.org/wiki/Multi-chassis_link_aggregation_group)など) は、サブネットと VLAN を適切に使用することで排除できます。 また、「[ホスト ネットワークの要件](host-network-requirements.md)」も参照してください。

ネットワーク ベンダーまたはネットワーク サポート チームと協力して、実行しようとするワークロードに合わせてネットワーク スイッチのサイズが適切に設定されていることを確認してください。

## <a name="using-switchless"></a>スイッチレスの使用

Azure Stack HCI では、クラスター内の各ノードがクラスター内のすべてのノードに冗長接続している限り、すべてのクラスター サイズの East-West トラフィックのスイッチレス (直接) 接続をサポートしています。 これは "フルメッシュ" 接続と呼ばれます。

> [!NOTE]
>ネットワーク アダプターの必要数に合わせてクラスターが 3 つのノードより大きくなるほど、スイッチレス デプロイの利点は減少していきます。

### <a name="advantages-of-switchless-connections"></a>スイッチレス接続のメリット

- East-West トラフィックでは、スイッチを購入する必要はありません。 North-South トラフィックではスイッチが 1 つ必要です。 これにより、資本支出 (CAPEX) コストが削減できますが、クラスター内のノード数によって異なります。
- スイッチがないため、ホストの構成を行うだけで済み、必要な構成手順の潜在的な数を減らすことができます。 このメリットは、クラスターのサイズが大きくなるにつれて減少します。

### <a name="disadvantages-of-switchless-connections"></a>スイッチレス接続のデメリット

- クラスター内のノードの数が増加していくと、ネットワーク アダプターのコストがネットワーク スイッチを使用するコストを超過する可能性があります。
- IP とサブネットのアドレス指定スキームには、より綿密な計画が必要です。
- 提供されるのはローカル ストレージ アクセスのみです。 VM トラフィック、管理トラフィック、および North-South アクセスを必要とするその他のトラフィックでは、これらのアダプターを使用できません。
- 一般に、3 つのノード クラスターを大きく超えて拡張することはできません。

## <a name="next-steps"></a>次の手順

- ネットワーク アダプターとホストの要件について学習する。 「[ホスト ネットワークの要件](host-network-requirements.md)」を参照してください。
- フェールオーバー クラスタリングの基礎を復習する。 [フェールオーバー クラスタリング ネットワークの基礎](https://techcommunity.microsoft.com/t5/failover-clustering/.failover-clustering-networking-basics-and-fundamentals/ba-p/1706005?s=09)に関するページを参照してください。
- SET の使用を復習する。 「[リモート ダイレクト メモリ アクセス (RDMA) とスイッチ埋め込みチーミング (SET)](/windows-server/virtualization/hyper-v-virtual-switch/rdma-and-switch-embedded-teaming)」を参照してください。
- デプロイについては、[Windows Admin Center を使用したクラスターの作成](../deploy/create-cluster.md)に関する記事を参照してください。
- デプロイについては、[Windows PowerShell を使用したクラスターの作成](../deploy/create-cluster-powershell.md)に関する記事を参照してください。