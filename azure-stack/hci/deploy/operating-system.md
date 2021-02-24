---
title: Azure Stack HCI オペレーティング システムのデプロイ
description: Azure Stack HCI OS をデプロイした後、Windows Admin Center を使用してサーバーに接続する方法について説明します。 サーバー クラスターを作成する方法、およびサーバーの最新の Windows 更新プログラムとファームウェアを取得する方法について説明します。
author: JohnCobb1
ms.author: v-johcob
ms.topic: tutorial
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 12/7/2020
ms.openlocfilehash: 51a4411e95207d2f7b544fdf507fe8bd8fc98f2e
ms.sourcegitcommit: 61556b7b6e029e3a26a4b7ef97f0b13fbe7cd5a5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/07/2020
ms.locfileid: "96761713"
---
# <a name="deploy-the-azure-stack-hci-operating-system"></a>Azure Stack HCI オペレーティング システムのデプロイ

> 適用対象:Azure Stack HCI バージョン 20H2

Azure Stack HCI をデプロイする最初のステップは、[Azure Stack HCI をダウンロード](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)し、クラスター化する各サーバーにオペレーティング システムをインストールすることです。 この記事では、オペレーティング システムを展開し、Windows Admin Center を使用してサーバーに接続するさまざまな方法について説明します。

> [!NOTE]
> お好みの Microsoft ハードウェア パートナーから [Azure Stack HCI カタログ](https://aka.ms/azurestackhcicatalog) に載せられている Azure Stack HCI 統合システム ソリューション ハードウェアを購入した場合、Azure Stack HCI オペレーティング システムはプレインストールされているはずです。 その場合は、このステップを省略して[Azure Stack HCI クラスターの作成](create-cluster.md)に進むことができます。

## <a name="prerequisites"></a>前提条件

Azure Stack HCI オペレーティング システムをデプロイする前に、次のことを行う必要があります。

- ハードウェアが Azure Stack HCI クラスターの要件を満たしていることを確認する
- デプロイが成功するために必要な情報を収集する
- 管理用 PC またはサーバーに Windows Admin Center をインストールする

Azure Stack HCI での Azure Kubernetes Service の要件については、[Azure Stack HCI での AKS の要件](../../aks-hci/overview.md#what-you-need-to-get-started)に関するページを参照してください。

### <a name="determine-hardware-requirements"></a>ハードウェア要件を判断する

Microsoft のパートナーから Azure Stack HCI ハードウェア/ソフトウェアの検証済みソリューションを購入することをお勧めします。 これらのソリューションは、互換性と信頼性を確保するために、Microsoft の参照アーキテクチャに照らして設計、組み立て、検証されているため、迅速に稼働させることができます。 お使いのシステム、コンポーネント、デバイス、ドライバーが、Windows Server カタログで認定されている Windows Server 2019 であることを確認します。 検証済みソリューションについては、[Azure Stack HCI ソリューション](https://azure.microsoft.com/overview/azure-stack/hci)の Web サイトを参照してください。

少なくとも、2 台のサーバー、サーバー間を結ぶ高信頼、高帯域幅、低遅延のネットワーク接続、それぞれ 1 台のサーバーにのみ物理的に接続されている SATA、SAS、NVMe、または永続メモリ ドライブが必要です。

ただし、ハードウェアの要件はデプロイするクラスターのサイズと構成によって異なることがあります。 デプロイを正常に行うために、Azure Stack HCI の[システム要件](../concepts/system-requirements.md)を確認してください。

### <a name="gather-information"></a>情報を収集する

デプロイを準備するために、環境に関する以下の詳細情報を収集してください。

- **サーバー名:** コンピューター、ファイル、パス、およびその他のリソースに関する組織の名前付けポリシーについて理解を深めてください。 複数のサーバーをプロビジョニングする場合、それぞれに一意の名前を付ける必要があります。
- **ドメイン名:** ドメインの名前付けとドメイン参加に関する組織のポリシーについて理解を深めてください。 サーバーをドメインに参加させ、ドメイン名を指定する必要があります。
- **静的 IP アドレス:** Azure Stack HCI では、記憶域およびワークロード (VM) トラフィックに静的 IP アドレスを使用する必要があります。DHCP を使用した動的 IP アドレスの割り当てはこの高速ネットワークではサポートされていません。 管理ネットワーク アダプターで DHCP を使用できます。ただし、チームで 2 つ使用している場合は DHCP を使用できず、静的 IP を使用する必要があります。 クラスター内の各サーバーに使用する IP アドレスについては、ネットワーク管理者にお問い合わせください。
- **RDMA ネットワーク:** RDMA プロトコルには、iWarp と RoCE の2種類があります。 ネットワーク アダプターでどちらの種類が使用されているか確認し、RoCE の場合は、そのバージョン (v1 または v2) にも注意してください。 RoCE では、Top-of-Rack スイッチのモデルも確認してください。
- **VLAN ID:** サーバーのネットワーク アダプターに使用する VLAN ID をメモしておきます (存在する場合)。 これは、ネットワーク管理者から入手できるはずです。
- **サイト名:** ストレッチ クラスターの場合、ディザスター リカバリーに 2 つのサイトが使用されます。 サイトは、[Active Directory Domain Services](/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview) を使用して設定できるほか、クラスターの作成ウィザードで自動的に設定することもできます。 サイトの設定については、ドメイン管理者に問い合わせてください。

### <a name="install-windows-admin-center"></a>Windows Admin Center をインストールする

Windows Admin Center は、ローカルにデプロイされるブラウザーベースのアプリであり、Azure Stack HCI の管理に使用します。 [Windows Admin Center をインストールする](/windows-server/manage/windows-admin-center/deploy/install)方法として最も簡単なのはローカル管理 PC にインストールすること (デスクトップ モード) ですが、サーバーにインストールすること (サービス モード) もできます。

Windows Admin Center をサーバーにインストールする場合、CredSSP が必要なタスク (クラスター作成や更新プログラムのインストールなど) では、Windows Admin Center サーバー上のゲートウェイ管理者グループのメンバーであるアカウントを使用する必要があります。 詳細については、「[ユーザー アクセス制御とアクセス許可を構成する](/windows-server/manage/windows-admin-center/configure/user-access-control#gateway-access-role-definitions)」の最初の 2 つのセクションを参照してください。

## <a name="prepare-hardware-for-deployment"></a>展開のためのハードウェアの準備

Azure Stack HCI ソリューション用のサーバー ハードウェアを入手したら、それをラックに取り付けてケーブルを接続します。 オペレーティング システムの展開用にサーバー ハードウェアを準備するには、次の手順のようにします。

1. サーバー クラスターで使用するすべてのサーバー ノードをラックに設置します。
1. サーバー ノードをネットワーク スイッチに接続します。
1. パフォーマンスと信頼性が最大になるように、Azure Stack HCI ハードウェア ベンダーからの推奨に従って、サーバーの BIOS または Unified Extensible Firmware Interface (UEFI) を構成します。

## <a name="operating-system-deployment-options"></a>オペレーティング システムの展開オプション

Azure Stack HCI オペレーティング システムは、他の Microsoft オペレーティング システムの展開に使用するのと同じ方法で展開できます。

- サーバー製造元のプレインストール。
- 応答ファイルを使用するヘッドレス展開。
- System Center Virtual Machine Manager (VMM)。
- ネットワーク展開。
- キーボードとモニターをデータセンター内のサーバー ハードウェアに直接接続するか、KVM ハードウェア デバイスをサーバー ハードウェアに接続することによる、手動での展開。

### <a name="server-manufacturer-pre-installation"></a>サーバー製造元のプレインストール

Azure Stack HCI オペレーティング システムのエンタープライズ展開では、好みのハードウェア パートナーの Azure Stack HCI Integrated System ソリューション ハードウェアを使用することをお勧めします。 ソリューション ハードウェアには、オペレーティング システムがプレインストールされています。また、ハードウェア製造元から提供されるドライバーとファームウェアを、Windows Admin Center を使用して展開および更新することがサポートされています。

ソリューション ハードウェアは、ノード数が 2 から 16 個の範囲で、Microsoft とパートナー ベンダーによってテストおよび検証されています。 好みのハードウェア パートナーからの Azure Stack HCI ソリューション ハードウェアを検索するには、[Azure Stack HCI カタログ](https://www.microsoft.com/cloud-platform/azure-stack-hci-catalog)のページを参照してください。

### <a name="headless-deployment"></a>ヘッドレス展開

応答ファイルを使用して、オペレーティング システムのヘッドレス展開を行うことができます。 応答ファイルでは、オペレーティング システムの無人インストールの間の構成設定と値が、XML 形式で定義されています。

この展開オプションでは、Windows システム イメージ マネージャーを使用して unattend.xml 応答ファイルを作成し、サーバーにオペレーティング システムを展開することができます。 Windows システム イメージ マネージャーを使用して無人応答ファイルを作成するときは、グラフィカル ツールのコンポーネント セクションで構成の質問に対する "回答" を定義してから、ファイルの形式と構文が正しいことを確認します。
Windows システム イメージ マネージャー ツールは、Windows アセスメント & デプロイメント キット (Windows ADK) で使用できます。 作業を開始するには:[Windows ADK をダウンロードしてインストールします](/windows-hardware/get-started/adk-install)。

### <a name="system-center-virtual-machine-manager-vmm-deployment"></a>System Center Virtual Machine Manager (VMM) の展開

System Center Virtual Machine Manager を使用すると、ベアメタル ハードウェア上に Azure Stack HCI オペレーティング システムをデプロイし、サーバーをクラスター化することができます。 VMM については、「[System Center Virtual Machine Manager のシステム要件](/system-center/vmm/system-requirements)」を参照してください。

VMM を使用してオペレーティング システムのベアメタル展開を実行する方法の詳細については、「[ベアメタル コンピューターからの Hyper-V ホストまたはクラスターのプロビジョニング](/system-center/vmm/hyper-v-bare-metal)」を参照してください。

### <a name="network-deployment"></a>ネットワーク展開

別の方法として、[Windows 展開サービス](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh831764(v=ws.11))を使用してネットワーク経由で Azure Stack HCI オペレーティング システムをインストールすることもできます。

### <a name="manual-deployment"></a>手動デプロイ

クラスター化する各サーバーのシステム ドライブに Azure Stack HCI オペレーティング システムを手動で展開するには、DVD または USB ドライブからの起動など、任意の方法を使用してオペレーティング システムをインストールします。 サーバー構成ツール (Sconfig) を使用してインストール プロセスを完了し、サーバーをクラスター化用に準備します。 ツールの詳細については、[Sconfig を使用した Server Core インストールの構成](/windows-server/get-started/sconfig-on-ws2016)に関する記事を参照してください。

Azure Stack HCI オペレーティング システムを手動でインストールするには:
1. オペレーティング システムをインストールするサーバーのシステム ドライブで、Azure Stack HCI のインストール ウィザードを開始します。
1. インストールする言語を選択するか、既定の言語設定をそのまま使用し、 **[次へ]** を選択して、ウィザードの次のページで **[今すぐインストール]** を選択します。

    :::image type="content" source="../media/operating-system/azure-stack-hci-install-language.png" alt-text="Azure Stack HCI のインストール ウィザードの言語ページ。":::

1. [適用される通知とライセンス条項] ページで、使用許諾契約書を確認し、 **[ライセンス条項に同意します]** チェック ボックスをオンにして、 **[次へ]** を選択します。
1. [インストールの種類を選んでください] ページで、 **[Custom: Install the newer version of Azure Stack HCI only (advanced)]\(カスタム: 新しいバージョンの Azure Stack HCI のインストールのみを行う (詳細)\)** を選択します。

    > [!NOTE]
    > このリリースのオペレーティング システムでは、アップグレード インストールはサポートされていません。

    :::image type="content" source="../media/operating-system/azure-stack-hci-install-which-type.png" alt-text="Azure Stack HCI のインストール ウィザードのインストールの種類オプション ページ。":::

1. [Where do you want to install Azure Stack HCI?]\(Azure Stack HCI のインストール場所を選んでください\) ページで、オペレーティング システムをインストールするドライブの場所を確認するか、更新して、 **[次へ]** を選択します。

    :::image type="content" source="../media/operating-system/azure-stack-hci-install-where.png" alt-text="Azure Stack HCI のインストール ウィザードのドライブの場所ページ。":::

1. [Installing Azure Stack HCI]\(Azure Stack HCI をインストールしています\) ページが表示され、プロセスの状態が示されます。

    :::image type="content" source="../media/operating-system/azure-stack-hci-installing.png" alt-text="Azure Stack HCI のインストール ウィザードの状態ページ。":::

    > [!NOTE]
    > インストール プロセスにおいては、プロセスを完了するためにオペレーティング システムが 2 回再起動され、管理者コマンド プロンプトが開かれる前に、サービス開始の通知が表示されます。

1. 管理者コマンド プロンプトで、オペレーティング システムにサインインする前にユーザーのパスワードを変更するには、 **[OK]** を選択して、Enter キーを押します。

    :::image type="content" source="../media/operating-system/azure-stack-hci-change-admin-password.png" alt-text="パスワード変更のプロンプト。":::

1. [Enter new credential for Administrator]\(管理者の新しい資格情報を入力してください\) というプロンプトで、新しいパスワードを入力し、確認のためにもう一度入力してから、Enter キーを押します。
1. [Your password has been changed]\(パスワードが変更されました\) という確認メッセージが表示されたら、Enter キーを押します。

    :::image type="content" source="../media/operating-system/azure-stack-hci-admin-password-changed.png" alt-text="パスワード変更確認プロンプト":::

これで、サーバー構成ツール (Sconfig) を使用して重要なタスクを実行する準備ができました。 Sconfig を使用するには、Azure Stack HCI オペレーティング システムが実行されているサーバーにログオンします。 それには、キーボードとモニターを使用してローカルに行うか、リモート管理 (ヘッドレスまたは BMC) コントローラーまたはリモート デスクトップを使用します。 サーバーにログオンすると、Sconfig ツールが自動的に開きます。

:::image type="content" source="../media/operating-system/azure-stack-hci-sconfig-screen.png" alt-text="サーバー構成ツールのインターフェイス。" lightbox="../media/operating-system/azure-stack-hci-sconfig-screen.png":::

Sconfig ツールのメイン ページからは、次の初期構成タスクを実行できます。
- ネットワークを構成するか、動的ホスト構成プロトコル (DHCP) を使用してネットワークが自動的に構成されたことを確認します。
- 既定で自動生成されたサーバー名が適切でない場合、サーバーの名前を変更します。
- サーバーを Active Directory ドメインに参加させます。
- ドメイン ユーザー アカウントまたは指定されたドメイン グループを、ローカル管理者に追加します。
- ローカル サブネットの外部からサーバーを管理する予定で、ドメインに参加しないことを決定した場合は、Windows リモート管理 (WinRM) へのアクセスを有効にします。 (既定のファイアウォール規則では、ローカル サブネットと Active Directory ドメイン サービス内の任意のサブネットの両方からの管理が許可されます)。

必要に応じて各サーバーで Sconfig を使用してオペレーティング システムを構成した後は、Windows Admin Center のクラスター作成ウィザードを使用してサーバーをクラスター化できます。

## <a name="next-steps"></a>次のステップ

この記事に関連する次の管理タスクを実行するには、以下を参照してください。
> [!div class="nextstepaction"]
> [Azure Stack HCI クラスターを作成する](../deploy/create-cluster.md)
