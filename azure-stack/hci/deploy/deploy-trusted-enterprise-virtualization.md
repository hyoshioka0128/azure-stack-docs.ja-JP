---
title: Azure Stack HCI におけるエンタープライズ向け高信頼仮想化のデプロイ
description: このトピックでは、エンタープライズ向け高信頼仮想化を使用する高度なセキュリティで保護されたインフラストラクチャを Azure Stack HCI オペレーティング システムで計画、構成、およびデプロイする方法について説明します。
author: JohnCobb1
ms.author: v-johcob
ms.topic: how-to
ms.date: 01/07/2021
ms.openlocfilehash: 37a29bcfae28f1b1f01cf8e459df016a3b5c4daf
ms.sourcegitcommit: 330d04d39e0cf3e8965e2ccbc181c968cb71d9ad
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2021
ms.locfileid: "98052670"
---
# <a name="deploy-trusted-enterprise-virtualization-on-azure-stack-hci"></a>Azure Stack HCI におけるエンタープライズ向け高信頼仮想化のデプロイ

>適用対象:Azure Stack HCI バージョン 20H2

このトピックでは、エンタープライズ向け高信頼仮想化を使用する高度なセキュリティで保護されたインフラストラクチャを Azure Stack HCI オペレーティング システムで計画、構成、およびデプロイする方法について説明します。 Azure Stack HCI への投資を活用して、Windows Admin Center と Azure portal を経由して[仮想化ベースのセキュリティ (VBS)](https://docs.microsoft.com/windows-hardware/design/device-experiences/oem-vbs) およびハイブリッド クラウド サービスを使用するハードウェア上で安全なワークロードを実行します。

## <a name="overview"></a>概要
VBS は、[Azure Stack HCI におけるセキュリティ投資](/windows-server/get-started-19/whats-new-19#security)のキー コンポーネントであり、セキュリティの脅威からホストと仮想マシン (VM) を保護します。 たとえば、国防総省 (DoD) 情報システムのセキュリティを強化するためのツールとして公開されている[セキュリティ技術導入ガイド (STIG)](https://nvd.nist.gov/ncp/checklist/914) では、一般的なセキュリティ要件として、VBS と [ハイパーバイザーで保護されたコード整合性 (HVCI)](https://docs.microsoft.com/windows-hardware/drivers/bringup/device-guard-and-credential-guard) がリストに含まれています。 セキュリティが侵害されたホストでは VM の保護が保証されないため、VBS および HVCI が有効になっているホスト ハードウェアを使用して VM 上 のワークロードを保護することが不可欠です。

VBS ではハードウェア仮想化機能を使用して、メモリの安全な領域を作成し、オペレーティング システムから分離します。 Windows の [仮想保護モード (VSM)](https://docs.microsoft.com/virtualization/hyper-v-on-windows/tlfs/vsm) を使用すると、さまざまなセキュリティ ソリューションをホストして、オペレーティング システムの脆弱性や悪意のある攻撃からの保護を大幅に向上させることができます。

VBS では、Windows ハイパーバイザーを使用することで、オペレーティング システム ソフトウェアにセキュリティ境界を作成して管理し、制限を適用して重要なシステム リソースを保護して、認証されたユーザーの資格情報などのセキュリティ アセットを保護します。 VBS を利用することで、マルウェアがオペレーティング システムのカーネルにアクセスできたとしても、マルウェアによるコードの実行や、プラットフォームのシークレットへのアクセスがハイパーバイザーによって防止されるため、悪用の可能性を大幅に制限し、阻止できます。

ハイパーバイザーは、システム ソフトウェアの最も高い特権レベルであり、すべてのシステム メモリ全体でページのアクセス許可を設定して適用します。 一方、VSM では、コードの整合性チェックが渡された後にのみ、ページが実行されます。 バッファー オーバーフローなどの脆弱性が発生し、マルウェアによってメモリの変更が試行された場合でも、コード ページは変更されないため、変更されたメモリが実行されません。 VBS と HVCI は、コードの整合性ポリシーの適用を大幅に強化します。 すべてのカーネル モード ドライバーとバイナリは、開始前にチェックされ、署名されていないドライバーやシステム ファイルはシステム メモリに読み込まれません。

## <a name="deploy-trusted-enterprise-virtualization"></a>エンタープライズ向け高信頼仮想化のデプロイ
このセクションでは、エンタープライズ向け高信頼仮想化を使用する安全性の高いインフラストラクチャを管理目的で Azure Stack HCI と Windows Admin Center でデプロイするために、ハードウェアを取得する方法の概要について説明します。

### <a name="step-1-acquire-hardware-for-trusted-enterprise-virtualization-on-azure-stack-hci"></a>手順 1:Azure Stack HCI でのエンタープライズ向け高信頼仮想化のためにハードウェアを取得する
最初に、ハードウェアを調達する必要があります。 最も簡単な方法としては、[Azure Stack HCI カタログ](https://hcicatalog.azurewebsites.net)で適切な Microsoft ハードウェア パートナーを見つけて、Azure Stack HCI オペレーティング システムがプレインストールされている統合システムを購入します。 カタログでは、フィルターを適用して、この種類のワークロードに最適化されたベンダーのハードウェアを表示できます。

それ以外の場合は、Azure Stack HCI オペレーティング システムを独自のハードウェアにデプロイする必要があります。 Azure Stack HCI デプロイ オプションの詳細および Windows Admin Center のインストールについては、「[Azure Stack HCI オペレーティング システムのデプロイ](./operating-system.md)」を参照してください。

次に、Windows Admin Center を使用して、[Azure Stack HCI クラスターを作成します](./create-cluster.md)。

Azure Stack HCI のすべてのパートナー ハードウェアは、ハードウェア アシュアランスの追加の認定を受けています。 認定プロセスでは、必要なすべての [VBS](https://docs.microsoft.com/windows-hardware/design/device-experiences/oem-vbs) 機能がテストされます。 ただし、Azure Stack HCI では、VBS および HVCI は自動的には有効になりません。 ハードウェア アシュアランスの追加の認定の詳細については、[Windows Server カタログ](https://www.windowsservercatalog.com/content.aspx?ctf=AQinfo-systems.htm#:~:text=Hardware%20Assurance%20Windows%20Server%20systems%20that%20are%20awarded,of%20Windows%20Server%2C%20starting%20with%20Windows%20Server%202016)の **システム** に関するページのハードウェア アシュアランスに関するセクションを参照してください。

   >[!WARNING]
   > HVCI は、Azure Stack HCI カタログに記載されていないハードウェア デバイスと互換性がない可能性があります。 エンタープライズ向け高信頼仮想化インフラストラクチャについては、パートナーの Azure Stack HCI の検証済みハードウェアを使用することを強くお勧めします。

### <a name="step-2-enable-hvci"></a>手順 2:HVCI を有効にする
サーバーのハードウェアと VM で HVCI を有効にします。 詳細については、「[コード整合性に対する仮想化ベースの保護を有効にする](https://docs.microsoft.com/windows/security/threat-protection/device-guard/enable-virtualization-based-protection-of-code-integrity)」を参照してください。

### <a name="step-3-set-up-azure-security-center-in-windows-admin-center"></a>手順 3: Windows Admin Center で Azure Security Center を設定する
Windows Admin Center で、[Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-introduction) を設定して、脅威防止を追加し、ワークロードのセキュリティ体制を迅速に評価します。

詳細については、「[Security Center を使用した Windows Admin Center リソースの保護](https://docs.microsoft.com/azure/security-center/windows-admin-center-integration)」を参照してください。

Security Center の使用を開始するには:
- 使用を開始するには、 Microsoft Azureのサブスクリプションが必要です。 サブスクリプションがない場合は、[無料試用版](https://azure.microsoft.com/free)にサインアップできます。
- Azure portal の Azure Security Center ダッシュボードにアクセスしたとき、または API を介してプログラムで有効にした場合は、現在のすべての Azure サブスクリプションで Security Center の Free の価格レベルが有効になります。
高度なセキュリティ管理と脅威検出の機能を利用するには、Azure Defender を有効にする必要があります。 Azure Defender は 30 日間無料で試用いただけます。 詳しくは、「[Security Center の価格](https://azure.microsoft.com/pricing/details/security-center)」をご覧ください。
- Azure Defender を有効にする準備ができたら、「[クイック スタート:Azure Security Center を設定する](https://docs.microsoft.com/azure/security-center/security-center-get-started)」で手順を参照してください。

Windows Admin Center を使用して、Backup、File Sync、Site Recovery、Point-to-Site VPN、Update Management、Azure Monitor などの追加の Azure ハイブリッド サービスを設定することもできます。

## <a name="next-steps"></a>次のステップ
エンタープライズ向け高信頼仮想化に関する詳細については、以下を参照してください。
- [Windows Server 上の Hyper-V](/windows-server/virtualization/hyper-v/hyper-v-on-windows-server)
