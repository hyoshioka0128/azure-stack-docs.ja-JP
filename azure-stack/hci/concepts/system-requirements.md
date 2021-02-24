---
title: Azure Stack HCI のシステム要件
description: Azure Stack HCI のサーバー、ストレージ、およびネットワーク コンポーネントを選択する方法。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 01/14/2021
ms.openlocfilehash: 3b29dfb35bad91ef02feebc22255b89116da1505
ms.sourcegitcommit: 8526f642ef859b0006c3991d966f93608a87288a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/15/2021
ms.locfileid: "98243461"
---
# <a name="system-requirements-for-azure-stack-hci"></a>Azure Stack HCI のシステム要件

> 適用対象:Azure Stack HCI バージョン 20H2

このトピックを使用して、Azure Stack HCI のサーバー、ストレージ、およびネットワークのシステム要件を評価します。

## <a name="server-requirements"></a>サーバーの要件

標準の Azure Stack HCI クラスターでは、最低 2 台、最大 16 台のサーバーが必要です。ただし、クラスター セットを使用して複数のクラスターを結合し、数百台のノードからなる HCI プラットフォームを作成することができます。

さまざまな種類の Azure Stack HCI デプロイについては、次の点に注意してください。

- ストレッチ クラスターでは、2 つの別個のサイトにサーバーをデプロイする必要があります。 これらのサイトは、異なる国、異なる都市、異なる階、または異なる部屋に配置できます。 ストレッチ クラスターでは、最低 4 台 (サイトあたり 2 台)、最大 16台 (サイトあたり 8 台) のサーバーが必要になります。

- サーバーはすべて、同じ製造元とモデルのものを使用することをお勧めします。また、64 ビット Intel Nehalem グレード、AMD EPYC グレード、または、第 2 レベルのアドレス変換 (SLAT) を備えたそれ以降の互換プロセッサを使用することが推奨されます。 Intel Optane DC 永続メモリをサポートするには、第 2 世代の Intel Xeon スケーラブル プロセッサが必要です。 プロセッサは 1.4 GHz 以上で、x64 命令セットと互換性がある必要があります。

- サーバーのオペレーティング システム、VM、およびその他のアプリやワークロードに対応するために、サーバーにはノードごとに少なくとも 32 GB の RAM が搭載されていることを確認してください。 さらに、記憶域スペース ダイレクトのメタデータ用に各サーバーで 1 テラバイト (TB) のキャッシュ ドライブ容量ごとに 4 GB の RAM を確保します。

- BIOS または UEFI で以下の仮想化サポートが有効になっていることを確認してください。
    - ハードウェア補助による仮想化。 これは、仮想化オプションを含むプロセッサ、具体的には Intel Virtualization Technology (Intel VT) または AMD Virtualization (AMD-V) テクノロジーを使用したプロセッサで利用できます。
    - ハードウェアによるデータ実行防止 (DEP) が使用可能で、有効になっている必要があります。 Intel システムの場合、これは XD ビット (execute disable bit) です。 AMD システムの場合、これは NX ビット (no execute bit) です。

- Windows Server でサポートされている任意のブート デバイス ([SATADOM](https://cloudblogs.microsoft.com/windowsserver/2017/08/30/announcing-support-for-satadom-boot-drives-in-windows-server-2016/) を含む) を使用できます。 RAID 1 ミラーは **不要** ですが、ブートでサポートされています。 200 GB の最小サイズが推奨されます。

- Hyper-V の機能固有のその他の要件については、「[Windows Server 上の Hyper-v のシステム要件](/windows-server/virtualization/hyper-v/system-requirements-for-hyper-v-on-windows)」を参照してください。

## <a name="storage-requirements"></a>ストレージの要件

Azure Stack HCI は直接接続された SATA、SAS、NVMe、または永続メモリのドライブに対応しています。これらは、それぞれ 1 台のサーバーのみに物理的に接続されます。

最適な結果を得るには、以下に従ってください。

- 同じクラスター内のすべてのサーバーで、ドライブの種類が同じであり、それぞれの種類のドライブ数も同じである必要があります。 また、ドライブは同じサイズおよびモデルにすることをお勧めします (必須ではありません)。 ドライブは、サーバーの内部、または 1 台のサーバーにのみ接続されている外部エンクロージャに配置できます。 詳細については、[ドライブの対称性に関する考慮事項](drive-symmetry-considerations.md)に関するページを参照してください。

- クラスター内の各サーバーには、ログ用の専用ボリュームが必要になり、ログ ストレージの速度は少なくともデータ ストレージと同じでなければなりません。 ストレッチ クラスターには少なくとも 2 つのボリュームが必要です。1 つはレプリケートされたデータ用で、もう 1 つはログ データ用です。

- スロットのマッピングと識別のために SCSI エンクロージャ ービス (SES) が必要になります。 各外部エンクロージャは、一意識別子 (一意の ID) を提示する必要があります。 

   > [!IMPORTANT]
   > **サポート対象外:** RAID コントローラー カードまたは SAN (ファイバー チャネル、iSCSI、FCoE) 記憶域、複数のサーバーに接続されている共有 SAS エンクロージャ、または複数のパスからドライブにアクセスできる任意の形式のマルチパス IO (MPIO)。 ホスト バス アダプター (HBA) カードは、単純なパススルー モードを実装する必要があります。

## <a name="networking-requirements"></a>ネットワーク要件

Azure Stack HCI クラスターでは、各サーバー ノード間に信頼性が高く、高帯域幅で低待機時間のネットワーク接続が必要になります。

- クラスター管理専用に少なくとも 1 つのネットワーク アダプターが使用可能であることを確認します。
- 使用するすべての VLAN 上のトラフィックを許可するようにネットワーク内の物理スイッチが構成されていることを確認します。

ホスト ネットワークの考慮事項と要件については、「[ホスト ネットワークの要件](host-network-requirements.md)」を参照してください。

## <a name="software-defined-networking-sdn-requirements"></a>ソフトウェア定義ネットワーク (SDN) の要件

Windows Admin Center を使用して Azure Stack HCI クラスターを作成するときには、ネットワーク コントローラーをデプロイしてソフトウェア定義ネットワーク (SDN) を有効にするオプションが用意されています。 Azure Stack HCI で SDN を使用する場合:

- ホスト サーバーに、ネットワーク コントローラー VM を作成するための空き領域が少なくとも 50 ～ 100 GB あることを確認してください。

- ネットワーク コントローラー VM を作成するには、Azure Stack HCI オペレーティング システムの仮想ハード ディスク (VHD) を、クラスター内の最初のノードにコピーする必要があります。 VHD は、[Sysprep](/windows-hardware/manufacture/desktop/sysprep-process-overview) を使用するか、[Convert-WindowsImage](https://gallery.technet.microsoft.com/scriptcenter/Convert-WindowsImageps1-0fe23a8f) スクリプトを実行し、.iso ファイルを VHD に変換することで準備できます。

Azure Stack HCI で SDN を使用するための準備の詳細については、「[ソフトウェア定義ネットワーク インフラストラクチャを計画する](plan-software-defined-networking-infrastructure.md)」と「[ネットワーク コントローラーのデプロイを計画する](../concepts/network-controller.md)」を参照してください。

   > [!NOTE]
   > SDN は、ストレッチ (マルチサイト) クラスターではサポートされていません。

### <a name="sdn-hardware-requirements"></a>SDN ハードウェア要件

このセクションでは、SDN 環境を計画するときの NIC と物理スイッチのネットワーク ハードウェアの要件について説明します。

#### <a name="network-interface-cards-nics"></a>ネットワーク インターフェイス カード (NIC)

Hyper-V ホストとストレージ ホストで使用する NIC には、最適なパフォーマンスを実現するために特定の機能が必要です。

リモート ダイレクト メモリ アクセス (RDMA) は、ホストの CPU を使用せずに大量のデータを転送できるようにするカーネル バイパス手法です。これにより、CPU が解放され、他の処理を実行できます。 スイッチ埋め込みチーミング (SET) は、Hyper-V と SDN スタックを含む環境で使用できる代替の NIC チーミング ソリューションです。 セットは、HYPER-V 仮想スイッチにいくつかの NIC チーミング機能を統合します。

詳細については、「[リモート ダイレクト メモリ アクセス (RDMA) とスイッチ埋め込みチーミング (SET)](/windows-server/virtualization/hyper-v-virtual-switch/rdma-and-switch-embedded-teaming)」を参照してください。

VXLAN または NVGRE カプセル化ヘッダーによって発生するテナント仮想ネットワーク トラフィックのオーバーヘッドを考慮するために、レイヤー 2 ファブリック ネットワーク (スイッチとホスト) の最大転送単位 (MTU) を 1674 バイト以上 (\(レイヤー 2 イーサネット ヘッダーを含む\)) に設定する必要があります。

新しい *EncapOverhead* 拡張アダプター キーワードをサポートする NIC では、ネットワーク コントローラー ホスト エージェントを介して MTU が自動的に設定されます。 新しい *EncapOverhead* キーワードをサポートしていない NIC では、*JumboPacket* \(または同等の\) キーワードを使用して、各物理ホスト上で MTU サイズを手動で設定する必要があります。

#### <a name="switches-and-routers"></a>スイッチとルーター

ご使用の SDN 環境に合わせて物理スイッチとルーターを選択するときは、次の一連の機能がサポートされていることを確認してください。
- スイッチ ポートの MTU 設定 \(必須\)
- MTU を 1674 バイト以上に設定 \(L2 イーサネット ヘッダーを含む\)
- L3 プロトコル \(必須\)
- 等コスト マルチパス (ECMP) ルーティング
- BGP \(IETF RFC 4271\) ベースの ECMP

実装で、次の IETF 標準の必須ステートメントをサポートする必要があります。
- RFC 2545: [IPv6 ドメイン間ルーティング用の BGP-4 マルチプロトコル拡張機能](https://tools.ietf.org/html/rfc2545)
- RFC 4760: [BGP-4 用のマルチプロトコル拡張機能](https://tools.ietf.org/html/rfc4760)
- RFC 4893: [4 オクテットの AS 番号空間の BGP サポート](https://tools.ietf.org/html/rfc4893)
- RFC 4456: [BGP ルート リフレクション: フル メッシュ内部 BGP (IBGP) の代替手段](https://tools.ietf.org/html/rfc4456)
- RFC 4724: [BGP のグレースフル再起動メカニズム](https://tools.ietf.org/html/rfc4724)

次のタグ付けプロトコルが必要です。
- VLAN - さまざまな種類のトラフィックの分離
- 802.1q トランク

次の項目は、リンク制御を提供します。
- サービスの品質 \(QoS\) \(RoCE を使用する場合に限り RFC が必要\)
- 拡張トラフィックの選択 \(802.1Qaz\)
- 優先順位に基づくフロー制御 (PFC) \(802.1p/Q および 802.1Qbb\)

次の項目は、可用性と冗長性を提供します。
- スイッチの可用性 (必須)
- ゲートウェイ機能を実行するには、高可用性ルーターが必要です。 これは、マルチシャーシのスイッチ/ルーター、または仮想ルーター冗長プロトコル (VRRP) などのテクノロジを使用して提供できます。

### <a name="domain-requirements"></a>ドメインの要件

Azure Stack HCI にはドメイン機能レベルの特別な要件はありません。お使いのドメイン コントローラー用に現在サポートされているオペレーティング システムのバージョンのみ確認が必要になります。 一般的なベスト プラクティスとして、Active Directory のごみ箱機能を有効にすることをお勧めします (まだ行っていない場合)。 詳細については、「[Active Directory Domain Services の概要](/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)」をご覧ください。

## <a name="maximum-supported-hardware-specifications"></a>サポートされているハードウェア最大仕様

次の仕様を超える Azure Stack HCI のデプロイはサポートされていません。

| リソース                     | 最大値 |
| ---------------------------- | --------|
| クラスターあたりの物理サーバー数 | 16      |
| ホストあたりの VM 数                 | 1,024   |
| VM あたりのディスク数 (SCSI)          | 256     |
| クラスターあたりのストレージ          | 4 PB    |
| サーバーあたりのストレージ           | 400 TB  |
| ホストあたりの論理プロセッサ数  | 512     |
| ホストあたりの RAM                 | 24 TB   |
| VM あたりの RAM                   | 12 TB (第 2 世代 VM) または 1 TB (第 1 世代)|
| ホストあたりの仮想プロセッサ数  | 2,048   |
| VM あたりの仮想プロセッサ数    | 240 (第 2 世代 VM) または 64 (第 1 世代)|

## <a name="next-steps"></a>次のステップ

関連情報については、以下もご覧ください。

- [ドライブの選択](choose-drives.md)
- [記憶域スペース ダイレクトのハードウェア要件](/windows-server/storage/storage-spaces/storage-spaces-direct-hardware-requirements)
