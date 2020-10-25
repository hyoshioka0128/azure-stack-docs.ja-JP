---
title: Azure Stack HCI オペレーティング システムのデプロイ
description: Azure Stack HCI OS をデプロイした後、Windows Admin Center を使用してサーバーに接続する方法について説明します。 サーバー クラスターを作成する方法、およびサーバーの最新の Windows 更新プログラムとファームウェアを取得する方法について説明します。
author: JohnCobb1
ms.author: v-johcob
ms.topic: tutorial
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 09/24/2020
ms.openlocfilehash: cf34506f5fbeec1c6e0531eda231219ae2949b59
ms.sourcegitcommit: 69cfff119ab425d0fbb71e38d1480d051fc91216
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/30/2020
ms.locfileid: "91572604"
---
# <a name="deploy-the-azure-stack-hci-operating-system"></a>Azure Stack HCI オペレーティング システムのデプロイ

> 適用対象:Azure Stack HCI バージョン 20H2

「[Azure Stack HCI をデプロイする前に](before-you-start.md#install-windows-admin-center)」の手順を完了した後、Azure Stack HCI をデプロイする最初のステップは、[Azure Stack HCI をダウンロード](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)し、クラスター化する各サーバーにオペレーティング システムをインストールすることです。 この記事では、オペレーティング システムを展開し、Windows Admin Center を使用してサーバーに接続するさまざまな方法について説明します。

オペレーティング システムの展開が済むと、サーバー クラスターの作成に関連するガイダンスを使用し、サーバーの最新の Windows 更新プログラムとファームウェアの更新プログラムを取得できるようになります。[Azure Stack HCI クラスターの作成](create-cluster.md)に関する記事を参照してください。

## <a name="prerequisites"></a>前提条件

- クラスター化するサーバーにアクセスできるシステム上にセットアップされている Windows Admin Center。「[Azure Stack HCI をデプロイする前に](before-you-start.md#install-windows-admin-center)」を参照してください。
- 任意のハードウェア ベンダーからの Microsoft 検証済みハードウェア、Azure Stack HCI オペレーティング システム、Azure サービスが提供される Azure Stack HCI ソリューション。[Azure Stack HCI ソリューション](https://azure.microsoft.com/products/azure-stack/hci/)に関する記事を参照してください。

## <a name="deployment-preparation"></a>展開の準備

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

### <a name="server-manufacturer-preinstallation"></a>サーバー製造元のプレインストール

Azure Stack HCI オペレーティング システムのエンタープライズ展開では、好みのハードウェア パートナーの Azure Stack HCI Integrated System ソリューション ハードウェアを使用することをお勧めします。 ソリューション ハードウェアには、オペレーティング システムがプレインストールされています。また、ハードウェア製造元から提供されるドライバーとファームウェアを、Windows Admin Center を使用して展開および更新することがサポートされています。

ソリューション ハードウェアは、ノード数が 2 から 16 個の範囲で、Microsoft とパートナー ベンダーによってテストおよび検証されています。 好みのハードウェア パートナーからの Azure Stack HCI ソリューション ハードウェアを検索するには、[Azure Stack HCI カタログ](https://www.microsoft.com/cloud-platform/azure-stack-hci-catalog)のページを参照してください。

### <a name="headless-deployment"></a>ヘッドレス展開

応答ファイルを使用して、オペレーティング システムのヘッドレス展開を行うことができます。 応答ファイルでは、オペレーティング システムの無人インストールの間の構成設定と値が、XML 形式で定義されています。

この展開オプションでは、Windows システム イメージ マネージャーを使用して unattend.xml 応答ファイルを作成し、サーバーにオペレーティング システムを展開することができます。 Windows システム イメージ マネージャーを使用して無人応答ファイルを作成するときは、グラフィカル ツールのコンポーネント セクションで構成の質問に対する "回答" を定義してから、ファイルの形式と構文が正しいことを確認します。
Windows システム イメージ マネージャー ツールは、Windows アセスメント & デプロイメント キット (Windows ADK) で使用できます。 作業を開始するには:[Windows ADK をダウンロードしてインストールします](/windows-hardware/get-started/adk-install)。

### <a name="system-center-virtual-machine-manager-vmm-deployment"></a>System Center Virtual Machine Manager (VMM) の展開

System Center Virtual Machine Manager (VMM) は System Center スイートの一部です。 VMM を使用すると、ベアメタル ハードウェア上に Azure Stack HCI オペレーティング システムを展開し、サーバーをクラスター化することができます。 VMM については、「[System Center Virtual Machine Manager のシステム要件](/system-center/vmm/system-requirements)」を参照してください。

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

    :::image type="content" source="../media/operating-system/azure-stack-hci-install-which-type.png" alt-text="Azure Stack HCI のインストール ウィザードの言語ページ。":::

1. [Where do you want to install Azure Stack HCI?]\(Azure Stack HCI のインストール場所を選んでください\) ページで、オペレーティング システムをインストールするドライブの場所を確認するか、更新して、 **[次へ]** を選択します。

    :::image type="content" source="../media/operating-system/azure-stack-hci-install-where.png" alt-text="Azure Stack HCI のインストール ウィザードの言語ページ。":::

1. [Installing Azure Stack HCI]\(Azure Stack HCI をインストールしています\) ページが表示され、プロセスの状態が示されます。

    :::image type="content" source="../media/operating-system/azure-stack-hci-installing.png" alt-text="Azure Stack HCI のインストール ウィザードの言語ページ。":::

    > [!NOTE]
    > インストール プロセスにおいては、プロセスを完了するためにオペレーティング システムが 2 回再起動され、管理者コマンド プロンプトが開かれる前に、サービス開始の通知が表示されます。

1. 管理者コマンド プロンプトで、オペレーティング システムにサインインする前にユーザーのパスワードを変更するには、 **[OK]** を選択して、Enter キーを押します。

    :::image type="content" source="../media/operating-system/azure-stack-hci-change-admin-password.png" alt-text="Azure Stack HCI のインストール ウィザードの言語ページ。":::

1. [Enter new credential for Administrator]\(管理者の新しい資格情報を入力してください\) というプロンプトで、新しいパスワードを入力し、確認のためにもう一度入力してから、Enter キーを押します。
1. [Your password has been changed]\(パスワードが変更されました\) という確認メッセージが表示されたら、Enter キーを押します。

    :::image type="content" source="../media/operating-system/azure-stack-hci-admin-password-changed.png" alt-text="Azure Stack HCI のインストール ウィザードの言語ページ。":::

これで、サーバー構成ツール (Sconfig) を使用して重要なタスクを実行する準備ができました。 Sconfig を使用するには、Azure Stack HCI オペレーティング システムが実行されているサーバーにログオンします。 それには、キーボードとモニターを使用してローカルに行うか、リモート管理 (ヘッドレスまたは BMC) コントローラーまたはリモート デスクトップを使用します。 サーバーにログオンすると、Sconfig ツールが自動的に開きます。

:::image type="content" source="../media/operating-system/azure-stack-hci-sconfig-screen.png" alt-text="Azure Stack HCI のインストール ウィザードの言語ページ。" lightbox="../media/operating-system/azure-stack-hci-sconfig-screen.png":::

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
