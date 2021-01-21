---
title: Azure Stack Hub の管理の基本 - MDC
titleSuffix: Azure Stack Hub
description: Azure Stack Hub の管理の基本について学習します。 追加サポート情報を参照し、次の手順をご覧ください。
author: PatAltimore
ms.topic: article
ms.date: 10/26/2020
ms.author: patricka
ms.reviewer: thoroet
ms.lastreviewed: 10/26/2020
ms.openlocfilehash: b5221162dd4a75fd9ffbd5fb382ab05560d622f6
ms.sourcegitcommit: 9b0e1264ef006d2009bb549f21010c672c49b9de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98255318"
---
# <a name="azure-stack-hub-administration-basics---modular-data-center-mdc"></a>Azure Stack Hub の管理の基本 - Modular Data Center (MDC)

Azure Stack Hub の管理が初めての場合は、知っておく必要があることがいくつかあります。 この記事では、Azure Stack Hub オペレーターとしての役割の概要と、ユーザーにその生産性を高めるのに役立つように伝える必要がある内容について説明します。

## <a name="understand-the-builds"></a>ビルドを理解する

Azure Stack Hub 統合システムを使用している場合、Azure Stack Hub の更新バージョンは更新プログラム パッケージによって配布されます。 これらのパッケージをインポートし、管理者ポータルの **[更新]** タイルを使用して、これらを適用できます。

## <a name="learn-about-available-services"></a>利用できるサービスの詳細

ユーザーにどのようなサービスを提供できるかを認識しておく必要があります。 Azure Stack Hub では Azure サービスのサブセットがサポートされます。 サポートされるサービスは引き続き増加してゆきます。

### <a name="foundational-services"></a>基礎となるサービス

Azure Stack Hub をデプロイすると、Azure Stack Hub に既定で次の基礎となるサービスが含まれます。

- Compute
- ストレージ
- ネットワーク
- Key Vault

これらの基礎となるサービスによって、最小限の構成でユーザーにサービスとしてのインフラストラクチャ (IaaS) を提供できます。

### <a name="additional-services"></a>その他のサービス

その他に以下の、サービスとしてのプラットフォーム (PaaS) サービスをサポートしています。

- App Service
- Azure Functions
- SQL および MySQL データベース
- Kubernetes
- IoT Hub
- イベント ハブ

これらのサービスをユーザーに提供する前には、追加の構成が必要です。 詳細については、「[Azure Stack Hub のオペレーター ドキュメント](../../operator/index.yml)」の「**チュートリアル**」と、「**操作方法ガイド**」 > 「**サービスの提供**」を参照してください。

### <a name="service-roadmap"></a>サービスのロードマップ

Azure Stack Hub には、引き続き Azure サービスのサポートが追加される予定です。 予想されるロードマップについては、「[Azure Stack Hub: Azure の拡張機能](https://go.microsoft.com/fwlink/?LinkId=842846&clcid=0x409)」ホワイトペーパーを参照してください。 新しい発表については、[Azure Stack Hub のブログ投稿](https://azure.microsoft.com/blog/tag/azure-stack-technical-preview)を確認することもできます。

## <a name="what-account-should-i-use"></a>使用するアカウントについて

Azure Stack Hub を管理する場合に注意するいくつかのアカウントに関する考慮事項があります。 これは特に、Azure Active Directory (Azure AD) ではなく ID プロバイダーとして Windows Server Active Directory Federation Services (AD FS) を使用するデプロイで当てはまります。

| **アカウント** | **Azure** | **AD FS** |
|---|---|---|
| ローカル管理者 (.\Administrator) |   |
| Azure AD 全体管理者 | インストール時に使用。 <br> 既定のプロバイダーの所有者 | 該当なし。 |
| 拡張ストレージのアカウント|   |   |
||

## <a name="what-tools-do-i-use-to-manage"></a>管理に使用するツールについて

[管理者ポータル](../../operator/azure-stack-manage-portals.md)または PowerShell を使用して、Azure Stack Hub を管理できます。 基本的概念を学ぶためには、ポータルを介するのが最も容易な方法です。 PowerShell を使用する場合は、準備の手順があります。 開始する前に、Azure Stack Hub での PowerShell の使用方法についてよく理解する必要がある場合があります。 詳細については、[Azure Stack Hub 上の PowerShell の概要](../../user/azure-stack-powershell-overview.md)に関するページを参照してください。

Azure Stack Hub ではデプロイ、管理、整理のための基礎となるメカニズムとして、Azure Resource Manager を使用します。 Azure Stack Hub の管理とユーザーのサポートを行う予定であれば、Resource Manager について学習する必要があります。 「[Azure Resource Manager の概要](https://download.microsoft.com/download/E/A/4/EA4017B5-F2ED-449A-897E-BD92E42479CE/Getting_Started_With_Azure_Resource_Manager_white_paper_EN_US.pdf)」ホワイトペーパーを参照してください。

## <a name="your-typical-responsibilities"></a>クラウド オペレーターの通常の担当範囲

ユーザーはサービスを使用する必要があります。 ユーザーの観点からは、クラウド オペレーターの主な役割は、ユーザーがこれらのサービスを使用できるようにすることです。 提供するサービスを決定し、プラン、オファー、クォータを作成することで、それらのサービスを使用できるようにします。 詳細については、[Azure Stack Hub でのサービス提供の概要](../../operator/service-plan-offer-subscription-overview.md)に関するページを参照してください。

[Azure Stack Hub Marketplace](../../operator/azure-stack-marketplace.md) に項目を追加する必要もあります。 [Azure から Azure Stack Hub に Marketplace の項目をダウンロードする](../../operator/azure-stack-download-azure-marketplace-item.md)のが最も簡単な方法です。

プラン、オファー、およびサービスをテストする場合は、管理者ポータルではなく、[ユーザー ポータル](../../operator/azure-stack-manage-portals.md)を使用することができます。

サービスを提供することに加えて、オペレーターとして Azure Stack Hub を稼働させ続けるという通常の業務を行う必要があります。 これらの業務には、次のタスクが含まれます。

- [Azure AD](../../operator/azure-stack-add-new-user-aad.md) のデプロイ用のユーザー アカウントを追加します。
- [ロールベースのアクセス制御を使用してアクセス許可を設定します](../../operator/azure-stack-manage-permissions.md)。 (このタスクは管理者に限定されません)。
- [インフラストラクチャの正常性を監視します](../../operator/azure-stack-monitor-health.md)。
- [ネットワーク](../../operator/azure-stack-viewing-public-ip-address-consumption.md)と[ストレージ](../../operator/azure-stack-manage-storage-accounts.md)のリソースを管理します。
- [Azure Stack Hub を開始および停止します](../../operator/azure-stack-start-and-stop.md)。
- [拡張ストレージを運用します](../../user/azure-stack-network-howto-extend-datacenter.md)。
- [IoT Hub を管理します](/azure/iot-hub/about-iot-hub)。
- [イベント ハブを管理します](../../operator/event-hubs-rp-overview.md?bc=%2fazure-stack%2fbreadcrumb%2ftoc.json&branch=release-tzl&toc=%2fazure-stack%2ftdc%2ftoc.json&view=azs-2002)。
- [App Service を管理します](../../operator/azure-stack-app-service-overview.md?bc=%2fazure-stack%2fbreadcrumb%2ftoc.json&branch=release-tzl&toc=%2fazure-stack%2ftdc%2ftoc.json&view=azs-2002)。
- ハードウェアを交換します。 こちらに、[交換可能な部分](../../operator/azure-stack-replace-component.md)の一覧があります。
- [サポートを受けます](../../operator/azure-stack-help-and-support-overview.md?bc=%2fazure-stack%2fbreadcrumb%2ftoc.json&branch=release-tzl&toc=%2fazure-stack%2ftdc%2ftoc.json&view=azs-2002)。

## <a name="operator-tasks"></a>オペレーターのタスク

オペレーターの日単位、週単位、および月単位のタスクの一覧を次に示します。

# <a name="daily"></a>[毎日](#tab/daily)

1. アラートを確認します。
2. バックアップの状態を確認します。
3. Defender シグネチャを更新します (切断されたシステム)。
4. OneFS で Isilon システムの正常性とイベントを確認します。
5. Isilon の容量を確認します。

# <a name="weekly"></a>[週次](#tab/weekly)

1. 容量を確認します。
2. Avocent の接続で `isi status –verbose` を実行します。

# <a name="monthly"></a>[毎月](#tab/monthly)

1. 毎月の更新プログラム パッケージ (Microsoft と OEM) を適用します。
2. ASDK を使用してバックアップを検証します。
3. Azure Stack Hub Marketplace を管理します (最新の状態を維持)。
4. スイッチのファームウェアと Avocent を更新します。
5. ストレージの容量を回収します。

# <a name="ondemand"></a>[オンデマンド](#tab/ondemand)

1. シークレットをローテーションします。
2. オファー、プラン、クォータを作成および更新します。
3. 修正プログラム パッケージを適用します。
4. 修正プログラム パッケージを適用します。
5. 容量を拡張します (ノードと IPSpace)。
6. Avocent の接続で `isi status –verbose` を実行します。
7. ストレージ アカウントを復元します。
8. システムを停止します。
9. 診断ログを収集します。

---

## <a name="what-to-tell-your-users"></a>ユーザーに伝えること

ユーザーには、Azure Stack Hub 内のサービスを操作する方法、環境に接続する方法、およびオファーをサブスクライブする方法を知らせる必要があります。 ユーザーに提供できるカスタム ドキュメントのほかに、ユーザーには [Azure Stack Hub ユーザー ドキュメント](../../user/index.yml)を案内できます。

### <a name="understand-how-to-work-with-services-in-azure-stack-hub"></a>Azure Stack Hub 内のサービスを操作する方法について

Azure Stack Hub 内のサービスを使用してアプリを構築する前に、ユーザーが理解しておく必要のある情報があります。 たとえば、PowerShell と API については、特定のバージョン要件があります。 また、Azure のサービスと Azure Stack Hub の同等のサービスの機能には、いくつかの違いがあります。 ユーザーには、以下の記事に必ず目を通してもらってください。

- [サービスを使用する場合やアプリを作成する場合の Azure Stack Hub と Azure の違い](../../user/azure-stack-considerations.md)
- [Azure Stack Hub VM の機能](../../user/azure-stack-vm-considerations.md)
- [Azure Stack Hub のストレージ: 相違点と考慮事項](../../user/azure-stack-acs-differences.md)

これらの記事には、Azure と Azure Stack Hub でのサービスの相違点に関する情報がまとめられています。 これらは、グローバルな Azure ドキュメントで Azure サービスについて提供されている情報を補完するものです。

### <a name="connect-to-azure-stack-hub-as-a-user"></a>ユーザーとして Azure Stack Hub に接続する

ユーザーは、[ユーザー ポータルへのアクセス方法](../../user/azure-stack-use-portal.md)や PowerShell を介した接続方法を知る必要があります。 統合システム環境では、ユーザーのポータル アドレスはデプロイごとに異なります。 ユーザーには正しい URL を提供する必要があります。

PowerShell を使用する場合、ユーザーはサービスを使用する前にリソース プロバイダーに登録する必要がある場合があります。 リソース プロバイダーがサービスを管理しています。 たとえば、ネットワーク リソース プロバイダーは仮想ネットワーク、ネットワーク インターフェイス、ロード バランサーなどのリソースを管理しています。 ユーザーは PowerShell を[インストール](../../operator/powershell-install-az-module.md)し、追加のモジュールを[ダウンロード](../../operator/azure-stack-powershell-download.md)して、PowerShell を[構成](../../user/azure-stack-powershell-configure-user.md)する (これにはリソース プロバイダーの登録が含まれます) 必要があります。

### <a name="subscribe-to-an-offer"></a>プランへのサブスクライブ

ユーザーはサービスを使用する前に、オペレーターが作成した[オファーをサブスクライブする](../../operator/azure-stack-subscribe-plan-provision-vm.md)必要があります。

## <a name="where-to-get-support"></a>サポートが受けられる場所

以前のリリースの Azure Stack Hub (1905 より前) に関するサポート情報を見つける場合は、「[Azure Stack Hub サービス ポリシー](../../operator/azure-stack-servicing-policy.md)」を参照してください。

統合システムについては、Microsoft と Microsoft の OEM (original equipment manufacturer) ハードウェア パートナーとの間に、統合されたエスカレーションと解決のプロセスがあります。

クラウド サービスに問題がある場合は、Microsoft カスタマー サポート サービス (CSS) を通じてサポートを提供します。 サポート リクエストを開くには、管理者ポータルの右上隅にあるヘルプとサポートのアイコン (疑問符) を選択します。 次に、 **[ヘルプとサポート]** を選択し、 **[サポート]** セクションの **[新しいサポート リクエスト]** を選択します。

デプロイ、パッチと更新プログラム、ハードウェア (現場交換可能ユニットを含む) や、ハードウェア ライフサイクル ホストで実行するソフトウェアなどのハードウェア ブランドのソフトウェアに問題がある場合は、最初に OEM ハードウェア ベンダーにお問い合わせください。

その他の問題については、Microsoft CSS にお問い合わせください。

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub でのリージョンの管理](../../operator/azure-stack-region-management.md)