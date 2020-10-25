---
title: Azure Stack HCI をデプロイする前に
description: Azure Stack HCI のデプロイを準備する方法。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 10/14/2020
ms.openlocfilehash: 4ff495aba1f46824a6ab47c95601687402d24edb
ms.sourcegitcommit: 8122672409954815e472a5b251bb7319fab8f951
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/14/2020
ms.locfileid: "92060108"
---
# <a name="before-you-deploy-azure-stack-hci"></a>Azure Stack HCI をデプロイする前に

> 適用対象:Azure Stack HCI バージョン 20H2

この攻略ガイドでは、以下の方法について説明します。

- お使いのハードウェアが標準 (単一サイト) またはストレッチ (2 サイト) の Azure Stack HCI クラスターの基本要件を満たしているかどうかを判断する
- サポートされている最大ハードウェア仕様を超えていないことを確認する
- デプロイが成功するために必要な情報を収集する
- 管理用 PC またはサーバーに Windows Admin Center をインストールする

Azure Stack HCI での Azure Kubernetes Service の要件については、[Azure Stack HCI での AKS の要件](../../aks-hci/overview.md#what-you-need-to-get-started)に関するページを参照してください。

## <a name="determine-hardware-requirements"></a>ハードウェア要件を判断する

Microsoft のパートナーから Azure Stack HCI ハードウェア/ソフトウェアの検証済みソリューションを購入することをお勧めします。 これらのソリューションは、互換性と信頼性を確保するために、Microsoft の参照アーキテクチャに照らして設計、組み立て、検証されているため、迅速に稼働させることができます。 お使いのシステム、コンポーネント、デバイス、ドライバーが、Windows Server カタログで認定されている Windows Server 2019 であることを確認します。 検証済みソリューションについては、[Azure Stack HCI ソリューション](https://azure.microsoft.com/overview/azure-stack/hci)の Web サイトを参照してください。

### <a name="server-requirements"></a>サーバーの要件

- 標準の Azure Stack HCI クラスターでは、最低 2 台、最大 16 台のサーバーが必要です。ただし、クラスター セットを使用して複数のクラスターを結合し、数百台のノードからなる HCI プラットフォームを作成することができます。
- ストレッチ クラスターでは、2 つの別個のサイトにサーバーをデプロイする必要があります。 これらのサイトは、異なる国、異なる都市、異なる階、または異なる部屋に配置できます。 ストレッチ クラスターでは、最低 4 台 (サイトあたり 2 台)、最大 16台 (サイトあたり 8 台) のサーバーが必要になります。
- サーバーはすべて、同じ製造元とモデルのものを使用することをお勧めします。また、64 ビット Intel Nehalem グレード、AMD EPYC グレード、または、第 2 レベルのアドレス変換 (SLAT) を備えたそれ以降の互換プロセッサを使用することが推奨されます。 Intel Optane DC 永続メモリをサポートするには、第 2 世代の Intel Xeon スケーラブル プロセッサが必要です。 プロセッサは 1.4 GHz 以上で、x64 命令セットと互換性がある必要があります。
- サーバーのオペレーティング システム、VM、およびその他のアプリやワークロードに対応するために、サーバーにはノードごとに少なくとも 32 GB の RAM が搭載されていることを確認してください。 さらに、記憶域スペース ダイレクトのメタデータ用に各サーバーで 1 テラバイト (TB) のキャッシュ ドライブ容量ごとに 4 GB の RAM を確保します。
- BIOS または UEFI で以下の仮想化サポートが有効になっていることを確認してください。
    - ハードウェア補助による仮想化。 これは、仮想化オプションを含むプロセッサ、具体的には Intel Virtualization Technology (Intel VT) または AMD Virtualization (AMD-V) テクノロジーを使用したプロセッサで利用できます。
    - ハードウェアによるデータ実行防止 (DEP) が使用可能で、有効になっている必要があります。 Intel システムの場合、これは XD ビット (execute disable bit) です。 AMD システムの場合、これは NX ビット (no execute bit) です。
- Windows Server でサポートされている任意のブート デバイス ([SATADOM](https://cloudblogs.microsoft.com/windowsserver/2017/08/30/announcing-support-for-satadom-boot-drives-in-windows-server-2016/) を含む) を使用できます。 RAID 1 ミラーは**不要**ですが、ブートでサポートされています。 200 GB の最小サイズが推奨されます。
- Hyper-V の機能固有のその他の要件については、「[Windows Server 上の Hyper-v のシステム要件](/windows-server/virtualization/hyper-v/system-requirements-for-hyper-v-on-windows)」を参照してください。

### <a name="networking-requirements"></a>ネットワーク要件

Azure Stack HCI クラスターでは、各サーバー ノード間に信頼性が高く、高帯域幅で低待機時間のネットワーク接続が必要になります。 以下の項目について確認します。

- クラスター管理専用に少なくとも 1 つのネットワーク アダプターが使用可能であることを確認します。
- 使用するすべての VLAN 上のトラフィックを許可するようにネットワーク内の物理スイッチが構成されていることを確認します。

サーバー ノード間で行われる通信には、次のような複数の種類があります。

- クラスター通信 (ノードの結合、クラスターの更新、レジストリの更新)
- クラスター ハートビート
- クラスター共有ボリューム (CSV) のリダイレクト トラフィック
- 仮想マシンのライブ マイグレーション トラフィック

記憶域スペース ダイレクトでは、次のような追加のネットワーク トラフィックを考慮する必要があります。

- 記憶域バス レイヤー (SBL) – ノード間で分散されるエクステントまたはデータ
- 正常性 – オブジェクト (ノード、ドライブ、ネットワーク カード、クラスター サービス) の監視と管理

ストレッチ クラスターの場合は、さらにサイト間の記憶域レプリカのトラフィックも発生します。 記憶域バス レイヤー (SBL) とクラスター共有ボリューム (CSV) のトラフィックは、サイト間ではなく、各サイト内のサーバー ノード間でのみ発生します。

ホスト ネットワークの計画に関する考慮事項と要件については、「[Azure Stack HCI のホスト ネットワークを計画する](../concepts/plan-host-networking.md)」を参照してください。

### <a name="software-defined-networking-requirements"></a>ソフトウェア定義ネットワークの要件

Windows Admin Center を使用して Azure Stack HCI クラスターを作成するときには、ネットワーク コントローラーをデプロイしてソフトウェア定義ネットワーク (SDN) を有効にするオプションが用意されています。 Azure Stack HCI で SDN を使用する場合:

- ホスト サーバーに、ネットワーク コントローラー VM を作成するための空き領域が少なくとも 50 ～ 100 GB あることを確認してください。

- ネットワーク コントローラー VM を作成するには、Azure Stack HCI オペレーティング システムの仮想ハード ディスク (VHD) を、クラスター内の最初のノードにコピーする必要があります。 VHD は、[Sysprep](/windows-hardware/manufacture/desktop/sysprep-process-overview) を使用するか、[Convert-WindowsImage](https://gallery.technet.microsoft.com/scriptcenter/Convert-WindowsImageps1-0fe23a8f) スクリプトを実行し、.iso ファイルを VHD に変換することで準備できます。

Azure Stack HCI で SDN を使用するための準備の詳細については、「[ソフトウェア定義ネットワーク インフラストラクチャを計画する](../concepts/plan-software-defined-networking-infrastructure.md)」と「[ネットワーク コントローラーのデプロイを計画する](../concepts/network-controller.md)」を参照してください。

### <a name="domain-requirements"></a>ドメインの要件

Azure Stack HCI にはドメイン機能レベルの特別な要件はありません。お使いのドメイン コントローラー用に現在サポートされているオペレーティング システムのバージョンのみ確認が必要になります。 一般的なベスト プラクティスとして、Active Directory のごみ箱機能を有効にすることをお勧めします (まだ行っていない場合)。

### <a name="storage-requirements"></a>ストレージの要件

- Azure Stack HCI は直接接続された SATA、SAS、NVMe、または永続メモリのドライブに対応しています。これらは、それぞれ 1 台のサーバーのみに物理的に接続されます。
- 同じクラスター内のすべてのサーバーで、ドライブの種類が同じであり、それぞれの種類のドライブ数も同じである必要があります。 また、ドライブは同じサイズおよびモデルにすることをお勧めします (必須ではありません)。 ドライブは、サーバーの内部、または 1 台のサーバーにのみ接続されている外部エンクロージャに配置できます。 詳細については、[ドライブの対称性に関する考慮事項](../concepts/drive-symmetry-considerations.md)に関するページを参照してください。
- クラスター内の各サーバーには、ログ用の専用ボリュームが必要になり、ログ ストレージの速度は少なくともデータ ストレージと同じでなければなりません。 ストレッチ クラスターには少なくとも 2 つのボリュームが必要です。1 つはレプリケートされたデータ用で、もう 1 つはログ データ用です。
- スロットのマッピングと識別のために SCSI エンクロージャ ービス (SES) が必要になります。 各外部エンクロージャは、一意識別子 (一意の ID) を提示する必要があります。 **サポート対象外:** RAID コントローラー カードまたは SAN (ファイバー チャネル、iSCSI、FCoE) 記憶域、複数のサーバーに接続されている共有 SAS エンクロージャ、または複数のパスからドライブにアクセスできる任意の形式のマルチパス IO (MPIO)。 ホスト バス アダプター (HBA) カードは、単純なパススルー モードを実装する必要があります。
- 詳細については、[ドライブの選択](../concepts/choose-drives.md)に関するトピック、または「[記憶域スペース ダイレクトのハードウェア要件](/windows-server/storage/storage-spaces/storage-spaces-direct-hardware-requirements)」を参照してください。
- ボリュームと回復性に使用できるオプションについては、「[記憶域スペース ダイレクトのボリュームの計画](/windows-server/storage/storage-spaces/plan-volumes#choosing-the-resiliency-type)」のトピックを参照してください。
- 仮想マシンおよび仮想化された記憶域もデプロイする場合は、「[ゲスト仮想マシン クラスターでの記憶域スペース ダイレクトの使用](/windows-server/storage/storage-spaces/storage-spaces-direct-in-vm)」を参照してください。

## <a name="review-maximum-supported-hardware-specifications"></a>サポートされているハードウェア最大仕様を確認する

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

## <a name="gather-information"></a>情報を収集する

デプロイを準備するために、環境に関する以下の詳細情報を収集してください。

- **サーバー名:** コンピューター、ファイル、パス、およびその他のリソースに関する組織の名前付けポリシーについて理解を深めてください。 複数のサーバーをプロビジョニングする場合、それぞれに一意の名前を付ける必要があります。
- **ドメイン名:** ドメインの名前付けとドメイン参加に関する組織のポリシーについて理解を深めてください。 サーバーをドメインに参加させ、ドメイン名を指定する必要があります。
- **静的 IP アドレス:** Azure Stack HCI では、記憶域およびワークロード (VM) トラフィックに静的 IP アドレスを使用する必要があります。DHCP を使用した動的 IP アドレスの割り当てはこの高速ネットワークではサポートされていません。 管理ネットワーク アダプターで DHCP を使用できます。ただし、チームで 2 つ使用している場合は DHCP を使用できず、静的 IP を使用する必要があります。 クラスター内の各サーバーに使用する IP アドレスについては、ネットワーク管理者にお問い合わせください。
- **RDMA ネットワーク:** RDMA プロトコルには、iWarp と RoCE の2種類があります。 ネットワーク アダプターでどちらの種類が使用されているか確認し、RoCE の場合は、そのバージョン (v1 または v2) にも注意してください。 RoCE では、Top-of-Rack スイッチのモデルも確認してください。
- **VLAN ID:** サーバーのネットワーク アダプターに使用する VLAN ID をメモしておきます (存在する場合)。 これは、ネットワーク管理者から入手できるはずです。
- **サイト名:** ストレッチ クラスターの場合、ディザスター リカバリーに 2 つのサイトが使用されます。 サイトは、Active Directory Domain Services を使用して設定できるほか、クラスターの作成ウィザードで自動的に設定することもできます。 サイトの設定については、ドメイン管理者に問い合わせてください。 また、詳細については、「[Active Directory Domain Services の概要](/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)」を参照してください。

## <a name="install-windows-admin-center"></a>Windows Admin Center をインストールする

Windows Admin Center は、ローカルにデプロイされるブラウザーベースのアプリであり、Azure Stack HCI の管理に使用します。 [Windows Admin Center をインストールする](/windows-server/manage/windows-admin-center/deploy/install)方法として最も簡単なのはローカル管理 PC にインストールすること (デスクトップ モード) ですが、サーバーにインストールすること (サービス モード) もできます。

Windows Admin Center をサーバーにインストールする場合、CredSSP が必要なタスク (クラスター作成や更新プログラムのインストールなど) では、Windows Admin Center サーバー上のゲートウェイ管理者グループのメンバーであるアカウントを使用する必要があります。 詳細については、「[ユーザー アクセス制御とアクセス許可を構成する](/windows-server/manage/windows-admin-center/configure/user-access-control#gateway-access-role-definitions)」の最初の 2 つのセクションを参照してください。

## <a name="next-steps"></a>次のステップ

次の記事に進んで、Azure Stack HCI オペレーティング システムのデプロイ方法を学習してください。
> [!div class="nextstepaction"]
> [Azure Stack HCI オペレーティング システムのデプロイ](operating-system.md)
