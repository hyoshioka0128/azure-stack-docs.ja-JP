---
title: Azure Stack HCI のホスト ネットワーク要件
description: Azure Stack HCI のホスト ネットワーク要件について
author: v-dasis
ms.topic: how-to
ms.date: 11/25/2020
ms.author: v-dasis
ms.reviewer: JasonGerend
ms.openlocfilehash: 3e31852e554c85ffab18aacaa336a007a97874f2
ms.sourcegitcommit: 9b0e1264ef006d2009bb549f21010c672c49b9de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98255301"
---
# <a name="host-network-requirements-for-azure-stack-hci"></a>Azure Stack HCI のホスト ネットワーク要件

> 適用対象: Azure Stack HCI バージョン 20H2

このトピックでは、Azure Stack HCI のホスト ネットワークに関する考慮事項と要件について説明します。 データ センターのアーキテクチャおよびサーバー間の物理接続の詳細については、「[物理ネットワークの要件](physical-network-requirements.md)」を参照してください。

## <a name="network-traffic-types"></a>ネットワーク トラフィックの種類

Azure Stack HCI のネットワーク トラフィックは、その使用目的によって分類することができます。

- **コンピューティング トラフィック** - 仮想マシン (VM) が送信元または送信先となるトラフィック
- **ストレージ トラフィック** - サーバー メッセージ ブロック (SMB) を使用した記憶域スペース ダイレクト (S2D) のトラフィック
- **管理トラフィック** - Active Directory、リモート デスクトップ、Windows Admin Center、Windows PowerShell などの、クラスター管理を行う管理者にとって重要なトラフィック。

## <a name="selecting-a-network-adapter"></a>ネットワーク アダプターの選択

Azure Stack HCI の場合、Standard または Premium の追加資格基準 (AQ) のある、Windows Server の Software Defined Data Center (SDDC) 認定を取得しているネットワーク アダプターを選択する必要があります。 これらのアダプターでは、最も高度なプラットフォーム機能をサポートしており、ハードウェア パートナーによるテストが最も多く実施されています。 通常、このレベルの精査が行われることにより、ハードウェアとドライバーに関連する品質問題を減らすことができます。

Standard または Premium の AQ 付きのアダプターを特定するには、アダプターの [Windows Server Catalog](https://www.windowsservercatalog.com/) と該当するオペレーティング システムのバージョンを確認します。 Premium AQ の表記の例を次に示します。

:::image type="content" source="media/plan-networking/windows-certified.png" alt-text="Windows 認定" lightbox="media/plan-networking/windows-certified.png":::

## <a name="overview-of-key-network-adapter-capabilities"></a>主なネットワーク アダプター機能の概要

Azure Stack HCI によって利用されるネットワーク アダプターの重要な機能には、次のものがあります。

- 動的仮想マシン マルチキュー (動的 VMMQ または d.VMMQ)
- リモート ダイレクト メモリ アクセス (RDMA)
- ゲスト RDMA
- スイッチが埋め込まれたチーミング (設定)

### <a name="dynamic-vmmq"></a>動的 VMMQ

Premium AQ を取得しているすべてのネットワーク アダプターでは、動的 VMMQ をサポートしています。 動的 VMMQ では、スイッチ埋め込みチーミングを使用する必要があります。

**適用可能なトラフィックの種類**: コンピューティング

**必要な認定**:Premium

動的 VMMQ は、従来の仮想マシン キュー (VMQ)、仮想 Receive Side Scaling (vRSS)、VMMQ に基づいて構築されたインテリジェントな受信側テクノロジであり、次の 3 つの主要な機能が強化されています。

- CPU コアを使用してホスト効率を最適化する
- CPU コアへのネットワーク トラフィック処理を自動チューニングすることで、VM で期待されるスループットを満たして維持できるようにする
- 予想されるトラフィック量を “バースト” ワークロードで受信できるようにする

動的 VMMQ の詳細については、ブログ記事「[統合アクセラレータ](https://techcommunity.microsoft.com/t5/networking-blog/synthetic-accelerations-in-a-nutshell-windows-server-2019/ba-p/653976)」を参照してください。

### <a name="rdma"></a>RDMA

RDMA はネットワーク アダプターに対するネットワーク スタック オフロードであり、SMB ストレージ トラフィックはオペレーティング システムを経由せずに処理されます。

RDMA は、ホスト CPU リソースの使用を最小限に抑えながら、高スループットで待機時間の短いネットワークを実現します。 これらのホスト CPU リソースを使用して、追加の VM またはコンテナーを実行できます。

**適用可能なトラフィックの種類**: ホスト ストレージ

**必要な認定**:Standard

Standard または Premium の AQ を取得しているすべてのアダプターは、RDMA (リモート ダイレクト メモリ アクセス) をサポートしています。 RDMA は Azure Stack HCI のストレージ ワークロード用に推奨されるデプロイの選択肢であり、必要に応じて、VM のストレージ ワークロード (SMB を使用) に対して有効にすることができます。 後述の「**ゲスト RDMA**」セクションを参照してください。

Azure Stack HCI では、iWARP または RoCE プロトコル実装のいずれかを使用して RDMA をサポートしています。

> [!IMPORTANT]
> RDMA アダプターは、同じ RDMA プロトコル (iWARP または RoCE) を実装する他の RDMA アダプターでのみ機能します。

ベンダーのすべてのネットワーク アダプターが RDMA をサポートしているわけではありません。 次の表に、Premium 認定 RDMA アダプターを提供するベンダー (アルファベット順) を示します。 ただし、この一覧に含まれていないハードウェア ベンダーも RDMA をサポートしています。 RDMA のサポートを確認するには、「[Windows Server Catalog](https://www.windowsservercatalog.com/)」を参照してください。

> [!NOTE]
> InfiniBand (IB) は、Azure Stack HCI. ではサポートされていません。

|NIC ベンダー|iWARP|RoCE|
|----|----|----|
|Broadcom|いいえ|はい|
|Chelsio|はい|いいえ|
|Intel|Yes|はい (一部のモデル)|
|Marvell (Qlogic/Cavium)|はい|Yes|
|Nvidia (Mellanox)|いいえ|はい|

> [!NOTE]
> ベンダーのすべてのアダプターが RDMA をサポートしているわけではありません。 特定のネットワーク カード ベンダーに RDMA のサポートを確認してください。

RDMA のデプロイの詳細情報については、[SDN GitHub リポジトリ](https://github.com/Microsoft/SDN/blob/master/Diagnostics/S2D%20WS2016_ConvergedNIC_Configuration.docx)から Word 文書をダウンロードしてください。

#### <a name="internet-wide-area-rdma-protocol-iwarp"></a>インターネット ワイド エリア RDMA プロトコル (iWARP)

iWARP では伝送制御プロトコル (TCP) を使用します。また、必要に応じて、データ センター ブリッジング (DCB) の優先順位ベースのフロー制御 (PFC) と拡張伝送サービス (ETS) を使用して拡張することもできます。

次の場合は、iWARP を使用することをお勧めします。

- ネットワークの経験がほとんどまたはまったくない、またはネットワーク スイッチの管理に不安がある場合
- ToR スイッチを制御しない場合
- デプロイ後にソリューションを管理しない場合
- 既に iWARP を使用したデプロイが存在する場合
- どのオプションを選択したらよいかわからない場合

#### <a name="rdma-over-converged-ethernet-roce"></a>RDMA over Converged Ethernet (RoCE)

RoCE は、ユーザー データグラム プロトコル (UDP) を使用しており、信頼性を確保するためにデータ センター ブリッジングの PFC と ETS を必要とします。

次の場合は、RoCE を使用することをお勧めします。

- データ センターに RoCE を使用したデプロイが既に存在する場合
- 物理ネットワークの要件を熟知している場合

### <a name="guest-rdma"></a>ゲスト RDMA

ゲスト RDMA を使用すると、ホスト上で RDMA を使用する場合と同じ利点を VM の SMB ワークロードで得ることができます。

**適用可能なトラフィックの種類**: コンピューティング

**必要な認定**:Premium

 ゲスト RDMA を使用する主な利点は次のとおりです。

- ネットワーク トラフィック処理のための NIC への CPU オフロード
- 非常に短い待機時間
- 高スループット

ゲスト RDMA のデプロイ方法などの詳細情報については、[SDN GitHub リポジトリ](https://github.com/Microsoft/SDN/blob/master/Diagnostics/S2D%20WS2016_ConvergedNIC_Configuration.docx)から Word 文書をダウンロードしてください。

### <a name="switch-embedded-teaming-set"></a>スイッチが埋め込まれたチーミング (設定)

スイッチ埋め込みチーミング (SET) は、Windows Server 2016 以降の Windows Server オペレーティング システムに含まれているソフトウェア ベースのチーミング テクノロジです。 SET は、使用されるネットワーク アダプターの種類に依存しません。

**適用可能なトラフィックの種類**: コンピューティング、ストレージ、管理

**必要な認定:** なし (OS に組み込み済み)

SET は、Azure Stack HCI でサポートされている唯一のチーミング テクノロジです。 負荷分散フェールオーバー (LBFO) は、Windows Server で一般的に使用される別のチーミング テクノロジですが、Azure Stack HCI ではサポートされていません。 Azure Stack HCI の LBFO の詳細情報については、ブログ記事「[Azure Stack HCI でのチーミング](https://techcommunity.microsoft.com/t5/networking-blog/teaming-in-azure-stack-hci/ba-p/1070642)」を参照してください。 SET は、コンピューティング、ストレージ、管理の各トラフィックのいずれでも効果的に機能します。

SET は、次のことを可能にする唯一のチーミング テクノロジであるため、Azure Stack HCI にとって重要です。

- RDMA アダプターのチーミング (必要な場合)
- ゲスト RDMA
- 動的 VMMQ
- その他の主な Azure Stack HCI 機能 (「[Azure Stack HCI でのチーミング](https://techcommunity.microsoft.com/t5/networking-blog/teaming-in-azure-stack-hci/ba-p/1070642)」を参照)

SET は、品質やパフォーマンスの向上など、LBFO を超える追加機能を提供します。 これを実現するためには、SET では対称 (同一) アダプターを使用する必要があります。非対称アダプターのチーミングはサポートされていません。 対称ネットワーク アダプターとは、次のものが同じであるアダプターです。

- メーカー (ベンダー)
- モデル (バージョン)
- 速度 (スループット)
- configuration

アダプターが対称であるかどうかを識別する最も簡単な方法は、速度が同じで、かつインターフェイスの説明が一致するかどうかを確認することです。 相違が許容されるのは、説明に記載されている数字のみです。 [`Get-NetAdapterAdvancedProperty`](/powershell/module/netadapter/get-netadapteradvancedproperty) コマンドレットを使用して、報告された構成に必ず同じプロパティ値が表示されるようにします。

次の表で、数字 (#) のみが異なっているインターフェイスの説明の例を確認してください。

|名前|インターフェイスの説明|リンク速度|
|----|----|----|
|NIC1|ネットワーク アダプター #1|25 Gbps|
|NIC2|ネットワーク アダプター #2|25 Gbps|
|NIC3|ネットワーク アダプター #3|25 Gbps|
|NIC4|ネットワーク アダプター #4|25 Gbps|

### <a name="rdma-traffic-considerations"></a>RDMA トラフィックに関する考慮事項

データ センター ブリッジング (DCB) を実装する場合は、ネットワーク スイッチを含むすべてのネットワーク ポート全体で、PFC と ETS の構成が正しく実装されていることを確認する必要があります。 DCB は、RoCE の場合は必須であり、iWARP の場合は省略可能です。

RDMA のデプロイ方法の詳細情報については、[SDN GitHub リポジトリ](https://github.com/Microsoft/SDN/blob/master/Diagnostics/S2D%20WS2016_ConvergedNIC_Configuration.docx)から Word 文書をダウンロードしてください。

RoCE ベースの Azure Stack HCI 実装では、ファブリックとすべてのホストで、既定のトラフィック クラスを含む 3 つの PFC トラフィック クラスを構成する必要があります。

#### <a name="cluster-traffic-class"></a>クラスター トラフィック クラス

このトラフィック クラスは、クラスターのハートビート用に十分な帯域幅があることを保証します。

- 必須: はい
- PFC 対応:いいえ
- 推奨されるトラフィックの優先順位:優先順位 7
- 推奨される帯域幅予約:
    - 10 GbE 以下の RDMA ネットワーク = 2%
    - 25 GbE 以上の RDMA ネットワーク = 1%

#### <a name="rdma-traffic-class"></a>RDMA トラフィック クラス

このトラフィック クラスは、SMB ダイレクトを使用して無損失 RDA 通信用に十分な帯域幅があることを保証します。

- 必須: はい
- PFC 対応:はい
- 推奨されるトラフィックの優先順位:優先順位 3 または 4
- 推奨される帯域幅予約:50%

#### <a name="default-traffic-class"></a>既定のトラフィック クラス

このトラフィック クラスには、VM トラフィックや管理トラフィックなど、クラスターまたは RDMA のトラフィック クラスで定義されていない他のすべてのトラフィックが記載されています。

- 必須: 既定 (ホストでの構成は不要)
- フロー制御 (PFC) 対応:いいえ
- 推奨されるトラフィック クラス:既定 (優先順位 0)
- 推奨される帯域幅予約:既定 (ホスト構成は不要)

## <a name="storage-traffic-models"></a>ストレージ トラフィック モデル

SMB には、SMB マルチチャネルを含む Azure Stack HCI 用のストレージ プロトコルとして、多くの利点があります。 SMB マルチチャネルはこのトピックの範囲外ですが、SMB マルチチャネルが使用できるあらゆるリンクでトラフィックが多重化されていることを理解することが重要です。

> [!NOTE]
>複数のサブネットと VLAN を使用して、Azure Stack HCI でストレージ トラフィックを分離することをお勧めします。

次の 4 つのノードを持つクラスターの例を考えてみましょう。 各サーバーには 2 つのストレージ ポート (左側と右側) があります。 各アダプターは同じサブネットと VLAN 上にあるため、SMB マルチチャネルでは、利用可能なすべてのリンクに接続が分散されます。 そのため、最初のサーバー (192.168.1.1) の左側のポートは、2 番目のサーバー (192.168.1.2) の左側のポートに接続します。 最初のサーバー (192.168.1.12) の右側のポートは、2 番目のサーバーの右側のポートに接続します。 3 番目と 4 番目のサーバーにも同様の接続が確立されます。

ただし、これにより、不要な接続が作成され、Top-of-Rack (ToR) スイッチに接続するインターリンク (マルチシャーシ リンク アグリゲーション グループまたは MC-LAG) で輻輳が発生します (X 印の部分)。 次の図を参照してください。

:::image type="content" source="media/plan-networking/four-node-cluster-1.png" alt-text="4 つのノードを持つクラスターの同じサブネット" lightbox="media/plan-networking/four-node-cluster-1.png":::

推奨されるアプローチは、アダプターのセットごとに個別のサブネットと VLAN を使用する方法です。 次の図では、右側のポートでサブネット 192.168.2.x /24 と VLAN2 が使用されています。 これにより、左側のポートのトラフィックは TOR1 に保持され、右側のポートのトラフィックは TOR2 に保持されます。 次の図を参照してください。

:::image type="content" source="media/plan-networking/four-node-cluster-2.png" alt-text="4 つのノードを持つクラスターの異なるサブネット" lightbox="media/plan-networking/four-node-cluster-2.png":::

## <a name="traffic-bandwidth-allocation"></a>トラフィックの帯域幅の割り当て

以下の表は、Azure Stack HCI での一般的なアダプター速度を使用した、さまざまな種類のトラフィックの帯域幅割り当ての例を示しています。 これは集中型ソリューションの例であり、すべてのトラフィックの種類 (コンピューティング、ストレージ、管理) が同じ物理アダプター上で実行され、SET を使用してチーミングされていることに注意してください。

このユース ケースでは、最も多くの制約が課されるため、適切なベースラインが示されています。 ただし、これは、アダプターの数と速度の順列を考慮して、サポート要件ではなく、1 つの例として考える必要があります。

この例では、次のことを前提としています。

- チームごとに 2 つのアダプターがある
- 記憶域バス レイヤー (SBL)、クラスター共有ボリューム (CSV)、Hyper-V (ライブ マイグレーション) トラフィックについて

    - 同じ物理アダプターを使用する
    - SMB を使用する
- SMB に、データ センター ブリッジングを使用して 50% の帯域幅が割り当てられている
    - SBL/CSV は最も優先順位の高いトラフィックであり、SMB 帯域幅の予約の 70% を受け取り、かつ
    - ライブ マイグレーション (LM) は `Set-SMBBandwidthLimit` コマンドレットを使用して制限され、残りの帯域幅の 29% を受け取る
        - ライブ マイグレーションで使用可能な帯域幅が 5 Gbps 以上で、ネットワーク アダプターに対応している場合は、RDMA を使用する。 これを行うには、次のコマンドレットを使用する。

            ```Powershell
            Set-VMHost VirtualMachineMigrationPerformanceOption SMB
            ```

        - ライブ マイグレーションで使用可能な帯域幅が 5 Gbps 未満の場合は、圧縮を使用してブラックアウト時間を短縮する。 これを行うには、次のコマンドレットを使用する。

            ```Powershell
            Set-VMHost -VirtualMachineMigrationPerformanceOption Compression
            ```

 - ライブ マイグレーションで RDMA を使用している場合は、SMB 帯域幅の制限を使用して、ライブ マイグレーション トラフィックが RDMA トラフィック クラスに割り当てられた帯域幅全体を消費できないようにします。 このコマンドレットは 1 秒あたりのバイト数 (Bps) で入力されますが、ネットワーク アダプターは 1 秒あたりのビット数 (bps) で表示されるので注意してください。 次のコマンドレットを使用して、たとえば帯域幅の制限を 6 Gbps に設定します。

    ```Powershell
    Set-SMBBandwidthLimit -Category LiveMigration -BytesPerSecond 750MB
    ```

    > [!NOTE]
    >この例の 750 MBps は 6 Gbps に相当します

以下に、帯域幅の割り当て表の例を示します。

|NIC 速度|チーム化された帯域幅|SMB 帯域幅予約**|SBL/CSV %|SBL/CSV 帯域幅|ライブ マイグレーション %|最大ライブ マイグレーション帯域幅|ハートビート %|ハートビート帯域幅|
|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|
|10 Gbps|20 Gbps|10 Gbps|70%|7 Gbps|*|*|2%|200 Mbps|
|25 Gbps|50 Gbps|25 Gbps|70%|17.5 Gbps|29%|7.25 Gbps|1%|250 Mbps|
|40 Gbps|80 Gbps|40 Gbps|70%|28 Gbps|29%|11.6 Gbps|1%|400 Mbps|
|50 Gbps|100 Gbps|50 Gbps|70%|35 Gbps|29%|14.5 Gbps|1%|500 Mbps|
|100 Gbps|200 Gbps|100 Gbps|70%|70 Gbps|29%|29 Gbps|1%|1 Gbps|
|200 Gbps|400 Gbps|200 Gbps|70%|140 Gbps|29%|58 Gbps|1%|2 Gbps|

*- ライブ マイグレーション トラフィックに対する帯域幅の割り当てが 5 Gbps 未満の場合は、RDMA ではなく圧縮を使用する必要があります

**- 50% は、この例における帯域幅予約の例です

## <a name="stretched-cluster-considerations"></a>ストレッチ クラスターに関する考慮事項

ストレッチ クラスターを使用すると、複数のデータ センターにまたがるディザスター リカバリーを実現できます。 最も単純な形式では、Azure Stack HCI のストレッチ クラスター ネットワークは次のようになります。

:::image type="content" source="media/plan-networking/stretched-cluster.png" alt-text="ストレッチ クラスター" lightbox="media/plan-networking/stretched-cluster.png":::

ストレッチ クラスターには、次の要件と特性があります。

- RDMA は単一サイトに限定されており、異なるサイトまたはサブネット間ではサポートされていません。
- 同じサイト内のサーバーは、同じラックおよびレイヤー 2 の境界内に存在する必要があります。
- サイト間の通信はレイヤー 3 の境界を越えます。ストレッチ済みのレイヤー 2 のトポロジはサポートされていません。

- サイトでストレージ アダプターに RDMA を使用している場合、これらのアダプターはサイト間をルーティングできない個別のサブネットと VLAN 上に存在する必要があります。 これにより、記憶域レプリカはサイト間で RDMA を使用できなくなります。
- サイト間の通信に使用されるアダプターについて

    - 物理または仮想 (ホスト vNIC) にすることができます。 仮想の場合は、物理 NIC ごとに、独自のサブネットと VLAN で 1 つの vNIC をプロビジョニングする必要があります。
    - サイト間でルーティングできる独自のサブネットと VLAN 上に配置する必要があります。
    - RDMA は、`Disable-NetAdapterRDMA` コマンドレットを使用して無効にする必要があります。 `Set-SRNetworkConstraint` コマンドレットを使用して、記憶域レプリカに特定のインターフェイスを使用するように明示的に要求することをお勧めします。
    - 記憶域レプリカの追加要件を満たしている必要があります。
-   別のサイトへのフェールオーバーが発生した場合は、他のサイトでワークロードを実行するのに十分な帯域幅が利用可能であることを確認する必要があります。 安全なオプションは、使用可能な容量の 50% でサイトをプロビジョニングすることです。 フェールオーバー中のパフォーマンス低下を許容できる場合、これは難しい要件ではありません。

## <a name="next-steps"></a>次の手順

- ネットワーク スイッチと物理ネットワークの要件について学習する。 「[物理ネットワークの要件](physical-network-requirements.md)」を参照してください。
- フェールオーバー クラスタリングの基礎を復習する。 [フェールオーバー クラスタリング ネットワークの基礎](https://techcommunity.microsoft.com/t5/failover-clustering/failover-clustering-networking-basics-and-fundamentals/ba-p/1706005?s=09)に関するページを参照
- SET の使用を復習する。 「[リモート ダイレクト メモリ アクセス (RDMA) とスイッチ埋め込みチーミング (SET)](/windows-server/virtualization/hyper-v-virtual-switch/rdma-and-switch-embedded-teaming)」を参照
- 展開については、[Windows Admin Center を使用したクラスターの作成](../deploy/create-cluster.md)に関する記事を参照
- 展開については、[Windows PowerShell を使用したクラスターの作成](../deploy/create-cluster-powershell.md)に関する記事を参照