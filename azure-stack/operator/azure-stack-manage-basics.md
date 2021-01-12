---
title: Azure Stack Hub の管理の基本
description: Azure Stack Hub の管理の基本について学習します。
author: PatAltimore
ms.topic: article
ms.date: 06/16/2020
ms.author: patricka
ms.lastreviewed: 06/16/2020
ms.openlocfilehash: da6e4d78d2464468b75ab2bd22d9bb81dd01e33b
ms.sourcegitcommit: c5d46662492887b70a599a60f3c3d27e3460a742
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/07/2021
ms.locfileid: "97965513"
---
# <a name="azure-stack-hub-administration-basics"></a>Azure Stack Hub の管理の基本

Azure Stack Hub の管理が初めての場合は、知っておく必要があることがいくつかあります。 この記事では、Azure Stack Hub オペレーターとしての役割の概要と、ユーザーにその生産性を高めるのに役立つように伝える必要がある内容について説明します。

## <a name="understand-the-builds"></a>ビルドを理解する

### <a name="integrated-systems"></a>統合システム

Azure Stack Hub 統合システムを使用している場合は、更新プログラム パッケージで更新バージョンの Azure Stack Hub が配布されます。 これらのパッケージをインポートし、管理者ポータルの **[更新]** タイルを使用して、これらを適用できます。
 
### <a name="development-kit"></a>開発キット

Azure Stack Development Kit (ASDK) を使用する場合は、[Azure Stack Hub の概要](../asdk/asdk-what-is.md)に関するページを確認し、ASDK の目的とその制限事項について学習します。 ASDK は "*サンドボックス*" として使用することができます。そこで Azure Stack Hub を評価し、非運用環境でアプリの開発とテストを行うことができます。 デプロイの情報については、[Azure Stack Development Kit のデプロイ](../asdk/asdk-install.md)に関するページを参照してください。

Azure のように、Microsoft は迅速にイノベーションを進めています。 新しいビルドは定期的にリリースされます。 ASDK を実行しており、最新のビルドに移行する場合は、[Azure Stack Hub を再デプロイ](../asdk/asdk-redeploy.md)する必要があります。 更新プログラム パッケージは適用できません。 このプロセスは時間がかかりますが、最新の機能を試すことができるというメリットがあります。 Web サイトの ASDK ドキュメントには、最新のリリース ビルドが反映されています。

## <a name="learn-about-available-services"></a>利用できるサービスの詳細

ユーザーにどのようなサービスを提供できるかを認識しておく必要があります。 Azure Stack Hub では Azure サービスのサブセットがサポートされます。 サポートされるサービスは引き続き増加してゆきます。

**基礎となるサービス**

Azure Stack Hub をデプロイすると、Azure Stack Hub に既定で次の "基礎となるサービス" が含まれます。

- Compute
- ストレージ
- ネットワーク
- Key Vault

これらの基礎となるサービスによって、最小限の構成でユーザーにサービスとしてのインフラストラクチャ (IaaS) を提供できます。

**その他のサービス**

現時点では、その他に以下の、サービスとしてのプラットフォーム (PaaS) サービスをサポートしています。

- App Service
- Azure Functions
- SQL および MySQL データベース
- Event Hubs
- IoT Hub (プレビュー段階)
- Kubernetes (プレビュー段階)

これらのサービスをユーザーに提供する前には、追加の構成が必要です。 詳細については、Azure Stack Hub オペレーター ドキュメントの「チュートリアル」と、「操作方法ガイド」の「サービスの提供」セクションを参照してください。

**サービスのロードマップ**

Azure Stack Hub には、引き続き Azure サービスのサポートが追加される予定です。 予想されるロードマップについては、「[Azure Stack Hub: Azure の拡張機能](https://go.microsoft.com/fwlink/?LinkId=842846&clcid=0x409)」ホワイトペーパーを参照してください。 新しい発表については、[Azure Stack Hub のブログ投稿](https://techcommunity.microsoft.com/t5/azure-stack-blog/bg-p/AzureStackBlog)を確認することもできます。

## <a name="what-account-should-i-use"></a>使用するアカウントについて

Azure Stack Hub を管理する場合に注意するいくつかのアカウントに関する考慮事項があります。 特に、Azure Active Directory (Azure AD) ではなく ID プロバイダーとして Windows Server Active Directory Federation Services (AD FS) を使用する展開では注意が必要です。 次のアカウントに関する考慮事項は、Azure Stack Hub 統合システムと ASDK デプロイの両方に適用されます。

|Account|Azure AD|AD FS|
|-----|-----|-----|
|ローカル管理者 (.\Administrator)|ASDK ホスト管理者。|ASDK ホスト管理者。|
|AzureStack\AzureStackAdmin|ASDK ホスト管理者。<br><br>Azure Stack Hub 管理者ポータルにサインインするために使用できます。<br><br>Service Fabric リングを表示および管理するためのアクセス権。|ASDK ホスト管理者。<br><br>Azure Stack Hub 管理者ポータルにはアクセスできません。<br><br>Service Fabric リングを表示および管理するためのアクセス権。<br><br>既定のプロバイダー サブスクリプション (DPS) の所有者ではなくなりました。|
|AzureStack\CloudAdmin|特権エンドポイント内で許可されたコマンドにアクセス、実行できます。|特権エンドポイント内で許可されたコマンドにアクセス、実行できます。<br><br>ASDK ホストにサインインできません。<br><br>既定のプロバイダー サブスクリプション (DPS) の所有者。|
|Azure AD 全体管理者|インストール時に使用。<br><br>既定のプロバイダー サブスクリプション (DPS) の所有者。|適用不可。|
|

## <a name="what-tools-do-i-use-to-manage"></a>管理に使用するツールについて
 
[管理者ポータル](azure-stack-manage-portals.md)または PowerShell を使用して、Azure Stack Hub を管理できます。 基本的概念を学ぶためには、ポータルを介するのが最も容易な方法です。 PowerShell を使用する場合は、準備の手順があります。 開始する前に、Azure Stack Hub での PowerShell の使用方法についてよく理解する必要がある場合があります。 詳細については、[Azure Stack Hub 上の PowerShell の概要](../user/azure-stack-powershell-overview.md)に関するページを参照してください。

Azure Stack Hub ではデプロイ、管理、整理のための基礎となるメカニズムとして、Azure Resource Manager を使用します。 Azure Stack Hub の管理とユーザーのサポートを行う予定の場合は、Resource Manager について学習できます。 「[Azure Resource Manager の概要](https://download.microsoft.com/download/E/A/4/EA4017B5-F2ED-449A-897E-BD92E42479CE/Getting_Started_With_Azure_Resource_Manager_white_paper_EN_US.pdf)」ホワイトペーパーを参照してください。

## <a name="your-typical-responsibilities"></a>クラウド オペレーターの通常の担当範囲

ユーザーはサービスを使用する必要があります。 ユーザーの観点からは、クラウド オペレーターの主な役割は、ユーザーがこれらのサービスを使用できるようにすることです。 提供するサービスを決定し、プラン、オファー、クォータを作成することで、それらのサービスを使用できるようにします。 詳細については、[Azure Stack Hub でのサービス提供の概要](service-plan-offer-subscription-overview.md)に関するページを参照してください。 

[Azure Stack Hub Marketplace](azure-stack-marketplace.md) に項目を追加する必要もあります。 [Azure から Azure Stack Hub に Marketplace の項目をダウンロードする](azure-stack-download-azure-marketplace-item.md)のが最も簡単な方法です。

> [!NOTE]
> プラン、オファー、およびサービスをテストする場合は、管理者ポータルではなく、[ユーザー ポータル](azure-stack-manage-portals.md)を使用することができます。

サービスを提供することに加えて、オペレーターとして Azure Stack Hub を稼働させ続けるという通常の業務を行う必要があります。 これらの業務には、次のタスクが含まれます。

- ユーザー アカウント ([Azure AD](azure-stack-add-new-user-aad.md) のデプロイ用または [AD FS](azure-stack-add-users-adfs.md) のデプロイ用) を追加します。
- [ロールベースのアクセス制御 (RBAC) のロールを割り当てます](azure-stack-manage-permissions.md) (このタスクは管理者に限られません)。
- [インフラストラクチャの正常性を監視します](azure-stack-monitor-health.md)。
- [ネットワーク](azure-stack-viewing-public-ip-address-consumption.md)と[ストレージ](azure-stack-manage-storage-accounts.md)のリソースを管理します。
- ハードウェアを交換します。 たとえば、[障害が発生したディスクを交換](azure-stack-replace-disk.md)します。

## <a name="what-to-tell-your-users"></a>ユーザーに伝えること

ユーザーには、Azure Stack Hub 内のサービスを操作する方法、環境に接続する方法、およびオファーをサブスクライブする方法を知らせる必要があります。 ユーザーに提供できるカスタム ドキュメントのほかに、ユーザーには [Azure Stack Hub ユーザー ドキュメント](../user/index.yml)を案内できます。

**Azure Stack Hub 内のサービスを操作する方法について**

Azure Stack Hub 内のサービスを使用してアプリを構築する前に、ユーザーが理解しておく必要のある情報があります。 たとえば、PowerShell と API については、特定のバージョン要件があります。 また、Azure のサービスと Azure Stack Hub の同等のサービスの機能には、いくつかの違いがあります。 ユーザーには、以下の記事に必ず目を通してもらってください。

- [重要な考慮事項: Azure Stack Hub でのサービスの使用またはアプリの作成](../user/azure-stack-considerations.md)
- [Azure Stack Hub の仮想マシンに関する考慮事項](../user/azure-stack-vm-considerations.md)
- [ストレージ: 相違点と考慮事項](../user/azure-stack-acs-differences.md)

これらの記事には、Azure と Azure Stack Hub でのサービスの相違点に関する情報がまとめられています。 これらは、グローバルな Azure ドキュメントで Azure サービスについて提供されている情報を補完するものです。

**ユーザーとして Azure Stack Hub に接続する**

ASDK 環境で、ユーザーは、リモート デスクトップを使用して ASDK ホストに接続しない場合、仮想プライベート ネットワーク (VPN) 接続を構成して Azure Stack Hub に接続できます。 [Azure Stack Hub への接続](../asdk/asdk-connect.md)に関するページを参照してください。

ユーザーは、[ユーザー ポータルへのアクセス方法](../user/azure-stack-use-portal.md)や PowerShell を介した接続方法を知る必要があります。 統合システム環境では、ユーザーのポータル アドレスはデプロイごとに異なります。 ユーザーには正しい URL を提供する必要があります。

PowerShell を使用する場合、ユーザーはサービスを使用する前にリソース プロバイダーに登録する必要がある場合があります。 リソース プロバイダーがサービスを管理しています。 たとえば、ネットワーク リソース プロバイダーは仮想ネットワーク、ネットワーク インターフェイス、ロード バランサーなどのリソースを管理しています。 ユーザーは PowerShell を[インストール](powershell-install-az-module.md)し、追加のモジュールを[ダウンロード](azure-stack-powershell-download.md)して、PowerShell を[構成](../user/azure-stack-powershell-configure-user.md)する (これにはリソース プロバイダーの登録が含まれます) 必要があります。

**オファーへのサブスクライブ**

ユーザーはサービスを使用する前に、オペレーターが作成した[オファーをサブスクライブする](azure-stack-subscribe-plan-provision-vm.md)必要があります。

## <a name="where-to-get-support"></a>サポートが受けられる場所

> [!Note]  
> 以前のリリースの Azure Stack Hub (1905 より前) に関するサポート情報を見つける場合は、[以前のリリースの Azure Stack Hub (1905 より前) のヘルプとサポート](azure-stack-servicing-policy.md)に関するページを参照してください。

### <a name="integrated-systems"></a>統合システム

統合システムについては、Microsoft と Microsoft の OEM (original equipment manufacturer) ハードウェア パートナーとの間に、統合されたエスカレーションと解決のプロセスがあります。

クラウド サービスに問題がある場合は、Microsoft サポートを通じてサポートが提供されます。 サポート リクエストを開くには、管理者ポータルの右上隅にあるヘルプとサポートのアイコン (疑問符) を選択します。 次に、 **[ヘルプとサポート]** を選択し、 **[サポート]** セクションの **[新しいサポート リクエスト]** を選択します。

デプロイ、パッチと更新プログラム、ハードウェア (現場交換可能ユニットを含む) や、ハードウェア ライフサイクル ホストで実行するソフトウェアなどのハードウェア ブランドのソフトウェアに問題がある場合は、最初に OEM ハードウェア ベンダーにお問い合わせください。

その他の問題については、Microsoft サポートにお問い合わせください。

### <a name="azure-stack-development-kit-asdk"></a>Azure Stack Development Kit (ASDK)

ASDK については、[Microsoft フォーラム](https://social.msdn.microsoft.com/Forums/azure/home?forum=azurestack)でサポート関連の質問をすることができます。 フォーラムにアクセスするには、管理者ポータルの右上隅にあるヘルプとサポートのアイコン (疑問符) を選択し、 **[ヘルプとサポート]** を選んでから、 **[サポート]** セクションで **[MSDN フォーラム]** を選択します。 これらのフォーラムは定期的にチェックされています。 ASDK は評価環境であるため、Microsoft サポートを通じた正式なサポートは提供されていません。

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub でのリージョンの管理](azure-stack-region-management.md)
