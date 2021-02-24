---
title: Azure Stack HCI のセキュリティに関する考慮事項
description: このトピックでは、Azure Stack HCI オペレーティング システムのセキュリティに関する考慮事項について説明します。
author: JohnCobb1
ms.author: v-johcob
ms.topic: conceptual
ms.date: 09/10/2020
ms.openlocfilehash: 181eb53d4b0e5c95065371e6b87e470a5e413d06
ms.sourcegitcommit: 69cfff119ab425d0fbb71e38d1480d051fc91216
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/30/2020
ms.locfileid: "91572672"
---
# <a name="azure-stack-hci-security-considerations"></a>Azure Stack HCI のセキュリティに関する考慮事項

>適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

このトピックでは、Azure Stack HCI オペレーティング システムに関連するセキュリティの考慮事項と推奨事項について説明します。
- パート 1 では、オペレーティング システムを強化し、データと ID を保護して組織のセキュリティで保護された基盤を効率的に構築するための、基本的なセキュリティ ツールとテクノロジについて説明します。
- パート 2 では、Azure Security Center で使用できるリソースについて説明します。
- パート 3 では、これらの領域で組織のセキュリティ体制をさらに強化するための高度なセキュリティの考慮事項について説明します。

## <a name="why-are-security-considerations-important"></a>セキュリティに関する考慮事項が重要である理由
セキュリティは、上位レベルの管理者からインフォメーション ワーカーに至るまで、組織内のすべてのユーザーに影響を及ぼします。 セキュリティ侵害が発生すると、通常業務が中断され、組織が停止する可能性があるため、不適切なセキュリティは組織にとって現実的なリスクとなります。 攻撃の可能性の検出が早いほど、セキュリティの侵害を軽減できます。

攻撃者は、環境の弱点を調べてそれを悪用した後、通常は最初の侵害から 24 ないし 48 時間以内に、特権をエスカレートしてネットワーク上のシステムを制御できます。 優れたセキュリティ対策を講じると、環境内のシステムが強化され、攻撃者の移動をブロックすることによって、攻撃者が制御を獲得するまでの時間が、数時間から数週間、さらには数か月まで延びます。 このトピックのセキュリティに関する推奨事項を実装することで、組織はこのような攻撃を可能な限り速く検出して対応できるようになります。

## <a name="part-1-build-a-secure-foundation"></a>パート 1: セキュリティで保護された基盤を構築する
以下のセクションでは、環境内で Azure Stack HCI オペレーティング システムが実行されているサーバーのセキュリティ保護された基盤を構築するための、推奨されるセキュリティ ツールとテクノロジについて説明します。

### <a name="harden-the-environment"></a>環境を強化する
このセクションでは、オペレーティング システムで実行されているサービスと仮想マシン (VM) を保護する方法について説明します。
- **Azure Stack HCI 認定ハードウェア**では、セキュア ブート、UEFI、TPM の一貫した設定が既定で提供されます。 仮想化ベースのセキュリティと認定ハードウェアを組み合わせることにより、セキュリティが重視されるワークロードを保護できます。 また、この信頼されたインフラストラクチャを Azure Security Center に接続して、行動分析とレポートをアクティブ化し、急速に変化するワークロードと脅威に対応することもできます。

    - "*セキュア ブート*" は、PC 業界によって開発されたセキュリティ標準であり、相手先ブランド供給 (OEM) によって信頼されているソフトウェアのみを使用して、デバイスを起動できるようにします。 詳細については、「[セキュア ブート](/windows-hardware/design/device-experiences/oem-secure-boot)」を参照してください。
    - *United Extensible Firmware Interface (UEFI)* では、サーバーの起動プロセスが制御されます。その後制御は、Windows または別のオペレーティング システムに渡されます。 詳細については、[UEFI ファームウェアの要件](/windows-hardware/design/device-experiences/oem-uefi)に関する記事を参照してください。
    - "*トラステッド プラットフォーム モジュール (TPM)* " テクノロジでは、ハードウェア ベースのセキュリティ関連機能が提供されます。 TPM チップは、暗号化キーの生成、保存、使用の制限を行う、セキュリティで保護された暗号化プロセッサです。 詳細については、「[トラステッド プラットフォーム モジュール技術の概要](/windows/security/information-protection/tpm/trusted-platform-module-overview)」を参照してください。

    Azure Stack HCI 認定ハードウェア プロバイダーの詳細については、[Azure Stack HCI ソリューション](https://azure.microsoft.com/products/azure-stack/hci/)の Web サイトをご覧ください。

- **Device Guard** と **Credential Guard**。 Device Guard を使用すると、シグネチャがわかっていないマルウェア、署名されていないコード、機密情報をキャプチャしたりシステムを破壊したりするためにカーネルにアクセスするマルウェアから保護されます。 Windows Defender Credential Guard では、仮想化ベースのセキュリティを使用してシークレットが分離され、特権のあるシステム ソフトウェアのみがそれらにアクセスできます。

    詳細については、「[Windows Defender Credential Guard を管理する](/windows/security/identity-protection/credential-guard/credential-guard-manage)」を参照し、[Device Guard および Credential Guard ハードウェア準備ツール](https://www.microsoft.com/download/details.aspx?id=53337)をダウンロードしてください。

- **Windows** と**ファームウェア**の更新プログラムは、クラスター、サーバー (ゲスト VM を含む)、PC に不可欠であり、オペレーティング システムとシステム ハードウェアの両方を攻撃者から確実に保護するのに役立ちます。 Windows Admin Center の**更新プログラム** ツールを使用して、個々のシステムに更新プログラムを適用できます。 ハードウェア プロバイダーに、ドライバー、ファームウェア、ソリューションの更新プログラムを取得するための Windows Admin Center のサポートが含まれている場合は、Windows の更新プログラムと同時にこれらの更新プログラムを取得できます。それ以外の場合は、ベンダーから直接取得します。

    詳細については、[クラスターの更新](../manage/update-cluster.md)に関する記事を参照してください。

    一度に複数のクラスターおよびサーバーの更新プログラムを管理するには、Windows Admin Center と統合されているオプションの Azure Update Management サービスをサブスクライブすることを検討してください。 詳細については、「[Windows Admin Center を使用した Azure Update Management](https://www.thomasmaurer.ch/2018/11/azure-update-management-windows-admin-center)」を参照してください。

### <a name="protect-data"></a>データの保護
このセクションでは、Windows Admin Center を使用して、オペレーティング システムのデータとワークロードを保護する方法について説明します。

- **記憶域スペース用の BitLocker** では、保存データが保護されます。 BitLocker を使用して、オペレーティング システム上の記憶域スペースのデータ ボリュームの内容を暗号化できます。 BitLocker を使用してデータを保護すると、FIPS 140-2 や HIPAA など、政府機関、地域、業界固有の標準に対するコンプライアンスを維持できます。
 
    Windows Admin Center での BitLocker の使用の詳細については、[ボリュームの暗号化、重複除去、圧縮の有効化](../manage/volume-encryption-deduplication.md)に関する記事を参照してください。

- Windows ネットワークに対する **SMB** 暗号化では、転送中のデータが保護されます。 "*サーバー メッセージ ブロック (SMB)* " はネットワーク ファイル共有プロトコルの一種です。サーバー メッセージ ブロックを使用すると、コンピューター上のアプリケーションで、ファイルの読み込みや書き込みを実行したり、コンピューター ネットワークにあるサーバー プログラムに対してサービスを要求したりすることができます。

    SMB 暗号化を有効にする方法については、「[SMB セキュリティ拡張機能](/windows-server/storage/file-server/smb-security)」を参照してください。

- Windows Admin Center の **Windows Defender ウイルス対策**では、クライアントとサーバーのオペレーティング システムが、ウイルス、マルウェア、スパイウェア、その他の脅威から保護されます。 詳細については、「[Windows Server 2016 および 2019 での Microsoft Defender ウイルス対策](/windows/security/threat-protection/microsoft-defender-antivirus/microsoft-defender-antivirus-on-windows-server-2016)」を参照してください。

### <a name="protect-identities"></a>ID を保護する
このセクションでは、Windows Admin Center を使用して特権 ID を保護する方法について説明します。

- **アクセスの制御**を使用すると、管理ランドスケープのセキュリティを向上させることができます。 Windows Admin Center サーバー (Windows 10 PC で実行しているのではなく) を使用している場合は、Windows Admin Center 自体に対するアクセスを、ゲートウェイ ユーザーとゲートウェイ管理者の 2 つのレベルで制御できます。 ゲートウェイ管理者 ID プロバイダーのオプションには、次のものが含まれます。
    - スマートカード認証を適用するための Active Directory またはローカル コンピューター グループ。
    - 条件付きアクセスと多要素認証を適用するための Azure Active Directory。
 
    詳細については、「[Windows Admin Center でのユーザー アクセス オプション](/windows-server/manage/windows-admin-center/plan/user-access-options)」および「[ユーザー アクセス制御とアクセス許可を構成する](/windows-server/manage/windows-admin-center/configure/user-access-control)」を参照してください。

- Windows Admin Center への**ブラウザー トラフィック**では、HTTPS が使用されます。 Windows Admin Center からマネージド サーバーへのトラフィックでは、標準の PowerShell と、Windows リモート管理 (WinRM) 経由の Windows Management Instrumentation (WMI) が使用されます。 Windows Admin Center では、ターゲット サーバーを管理するために、ローカル管理者パスワード ソリューション (LAPS)、リソース ベースの制約付き委任、Active Directory (AD) または Microsoft Azure Active Directory (Azure AD) を使用したゲートウェイ アクセスの制御、ロールベースのアクセス制御 (RBAC) がサポートされています。

    Windows Admin Center では、Windows 10 上の Microsoft Edge (Windows 10 バージョン 1709 以降)、Google Chrome、Microsoft Edge Insider がサポートされています。 Windows Admin Center は、windows 10 PC 上または Windows サーバー上にインストールできます。

    サーバーにインストールされた Windows Admin Center は、ホスト サーバー上の UI を持たないゲートウェイとして実行されます。 このシナリオでは、管理者は、ホスト上の自己署名セキュリティ証明書によって保護された HTTPS セッションを介してサーバーにログオンできます。 ただし、サポートされているブラウザーでは、信頼された VPN 経由のローカル IP アドレスに対する接続であっても、自己署名接続は安全でないものとして扱われるため、信頼された証明機関からの適切な SSL 証明書を使用することをお勧めします。

    組織に対するインストール オプションの詳細については、「[適切なインストールの種類](/windows-server/manage/windows-admin-center/plan/installation-options)」を参照してください。

- **CredSSP** は認証プロバイダーであり、いくつかのケースで管理対象の特定のサーバーを超えてコンピューターに資格情報を渡すために、Windows Admin Center によって使用されます。 現在、Windows Admin Center では次の場合に CredSSP が必要です。
    - 新しいクラスターを作成します。
    - フェールオーバー クラスタリングまたはクラスター対応更新機能を使用するために、**更新プログラム** ツールにアクセスする。
    - VM 内の非集計 SMB 記憶域を管理する。

    詳細については、「[Windows Admin Center は CredSSP を使用しますか](/windows-server/manage/windows-admin-center/understand/faq#does-windows-admin-center-use-credssp)」を参照してください。

- Windows Admin Center で**ロールベースのアクセス制御 (RBAC)** を使用すると、ユーザーをローカルの完全な管理者にするのではなく、管理する必要があるサーバーへのアクセスを制限できます。 Windows Admin Center で RBAC を使用するには、PowerShell の Just Enough Administration エンドポイントで各マネージド サーバーを構成します。

    詳細については、「[ロールベースのアクセス制御](/windows-server/manage/windows-admin-center/plan/user-access-options#role-based-access-control)」と「[Just Enough Administration](/powershell/scripting/learn/remoting/jea/overview)」を参照してください。

- ID の管理と保護に使用できる Windows Admin Center の**セキュリティ ツール**には、Active Directory、証明書、ファイアウォール、ローカル ユーザーとグループなどがあります。

    詳細については、「[Windows Admin Center を使用してサーバーを管理する](/windows-server/manage/windows-admin-center/use/manage-servers)」を参照してください。

## <a name="part-2-use-azure-security-center"></a>パート 2: Azure Security Center の利用
*Azure Security Center* は、データ センターのセキュリティ体制を強化する統合インフラストラクチャ セキュリティ管理システムです。クラウドとオンプレミスのハイブリッド ワークロード全体を保護する高度な脅威防止機能が提供されます。 Security Center には、ネットワークのセキュリティ状態の評価、ワークロードの保護、セキュリティ アラートの生成、特定の推奨事項に従った攻撃の修復と将来の脅威への対処を行うためのツールが用意されています。 Security Center では、Azure サービスを使用した自動プロビジョニングと保護により、デプロイのオーバーヘッドなしに、これらのすべてのサービスがクラウドで高速に実行されます。

Security Center を使用すると、これらのリソースに Log Analytics エージェントをインストールすることによって、Windows サーバーと Linux サーバーの両方で VM を保護できます。 Azure では、エージェントによって収集されたイベントが、ワークロードのセキュリティ強化のために実行する推奨事項 (セキュリティ強化タスク) に関連付けられます。 セキュリティのベスト プラクティスに基づくセキュリティ強化タスクには、セキュリティ ポリシーの管理と適用が含まれます。 その後、結果を追跡し、Security Center の監視によって経時的にコンプライアンスとガバナンスを管理しながら、すべてのリソースの攻撃対象領域を減らすことができます。

Azure のリソースとサブスクリプションにアクセスできるユーザーを管理することは、Azure のガバナンス戦略の重要な部分を占めています。 Azure のロールベースのアクセス制御 (RBAC) は、Azure でアクセスを管理するための主要な方法です。 詳細については、「[ロールベースのアクセス制御を使用して Azure 環境へのアクセスを管理する](/azure/cloud-adoption-framework/ready/azure-setup-guide/manage-access)」を参照してください。

Windows Admin Center を使用して Security Center を操作するには、Azure サブスクリプションが必要です。 業を開始するには、「[Azure Security Center と Windows Admin Center の統合](/azure/security-center/windows-admin-center-integration)」を参照してください。

登録した後、Windows Admin Center で Security Center にアクセスします。 **[すべての接続]** ページで、サーバーまたは VM を選択し、 **[ツール]** で **[Azure Security Center]** を選択して、 **[Sign into Azure]\(Azure にサインインする\)** を選択します。

詳細については、「[Azure Security Center とは](/azure/security-center/security-center-intro)」を参照してください

## <a name="part-3-add-advanced-security"></a>パート 3: 高度なセキュリティを追加する
以下のセクションでは、環境内で Azure Stack HCI オペレーティング システムが実行されているサーバーをさらに強化するするための、推奨される高度なセキュリティ ツールとテクノロジについて説明します。

### <a name="harden-the-environment"></a>環境を強化する
- **Microsoft セキュリティ ベースライン**は、商業組織および米国政府 (国防総省など) とのパートナーシップによって得られた、Microsoft のセキュリティに関する推奨事項に基づいています。 セキュリティ ベースラインには、Windows ファイアウォール、Windows Defender、その他に対する推奨されるセキュリティ設定が含まれています。

    セキュリティ ベースラインはグループ ポリシー オブジェクト (GPO) のバックアップとして提供され、Active Directory Domain Services (AD DS) にインポートし、ドメインに参加しているサーバーに展開して環境を強化できます。 また、ローカル スクリプト ツールを使用し、セキュリティ ベースラインを適用してスタンドアロン (ドメインに参加していない) サーバーを構成することもできます。 セキュリティ ベースラインの使用を始めるには、[Microsoft Security Compliance Toolkit 1.0](https://www.microsoft.com/download/details.aspx?id=55319) をダウンロードしてください。

    詳細については、「[Microsoft セキュリティ ベースライン](https://techcommunity.microsoft.com/t5/microsoft-security-baselines/bg-p/Microsoft-Security-Baselines)」を参照してください。

### <a name="protect-data"></a>データの保護
- **Hyper-V 環境を強化する**には、物理サーバーで実行されているオペレーティング システムを強化するのと同じように、VM で実行されている Windows Server を強化する必要があります。 仮想環境では、通常、同じ物理ホストを複数の VM で共有しているため、物理ホストとそのホスト上で実行されている VM の両方を保護することが不可欠です。 ホストを侵害する攻撃者は、複数の VM に影響を与えて、ワークロードとサービスに大きな影響を与えることができます。 このセクションでは、Hyper-V 環境内の Windows Server を強化するために使用できる以下の方法について説明します。

    - **保護されたファブリックとシールドされた VM** を使用すると、攻撃者による VM ファイルの変更を阻止することで、Hyper-V 環境で実行されている VM のセキュリティが強化されます。 "*保護されたファブリック*" は、通常、3 つのノード、1 つ以上の保護されたホスト、シールドされた VM のセットのクラスターであるホスト ガーディアン サービス (HGS) で構成されています。 構成証明サービスではホストの要求の有効性が評価され、キー保護サービスでは保護されたホストがシールドされた VM を開始するために使用できるキーを解放するかどうかが決定されます。

        詳細については、「[保護されたファブリックとシールドされた VM の概要](/windows-server/security/guarded-fabric-shielded-vm/guarded-fabric-and-shielded-vms)」を参照してください。
     
     - Windows Server の**仮想トラステッド プラットフォーム モジュール (vTPM)** でサポートされている VM 向けの TPM を使用すると、VM での BitLocker などの高度なセキュリティ テクノロジを使用できます。 Hyper-V マネージャーまたは `Enable-VMTPM` Windows PowerShell コマンドレットを使用して、任意の第 2 世代 Hyper-V VM で TPM のサポートを有効にすることができます。
     
        詳細については、「[Enable-VMTPM](/powershell/module/hyper-v/enable-vmtpm)」を参照してください。
     
     - Azure Stack HCI および Windows Server の**ソフトウェアによるネットワーク制御 (SDN)** では、データセンター内のルーター、スイッチ、ゲートウェイなどの物理および仮想ネットワーク デバイスが、一元的に構成および管理されます。 Hyper-V 仮想スイッチ、Hyper-V ネットワーク仮想化、RAS ゲートウェイなどの仮想ネットワーク要素は、SDN インフラストラクチャの不可欠な要素として設計されています。

        詳細については、「[ソフトウェア定義ネットワーク (SDN)](/windows-server/networking/sdn/)」を参照してください。

### <a name="protect-identities"></a>ID を保護する
- **ローカル管理者パスワード ソリューション (LAPS)** は、各コンピューターのローカル管理者アカウントのパスワードを、ランダムで一意の新しい値に定期的に設定する、Active Directory ドメイン参加済みシステム用の軽量メカニズムです。 パスワードは、Active Directory 内の対応するコンピューター オブジェクトのセキュリティで保護された機密属性に格納され、特別に承認されたユーザーのみが取得できます。 LAPS では、ドメイン アカウントを使用するより利点がある方法で、リモート コンピューター管理にローカル アカウントが使用されます。 詳細については、「[ローカル アカウントのリモート使用: LAPS によってすべてが変更される](/archive/blogs/secguide/remote-use-of-local-accounts-laps-changes-everything)」を参照してください。

    LAPS を使い始めるには、[Local Administrator Password Solution (LAPS)](https://www.microsoft.com/download/details.aspx?id=46899) をダウンロードします。

- **Microsoft Advanced Threat Analytics (ATA)** は、特権 ID を侵害しようとしている攻撃者を検出するために使用できるオンプレミスの製品です。 ATA では、認証、承認、および情報収集プロトコル (Kerberos、DNS など) のためにネットワーク トラフィックが解析されます。 ATA により、そのデータを使用してネットワーク上のユーザーや他のエンティティの動作プロファイルが構築され、異常と既知の攻撃パターンが検出されます。
    
    詳細については、「[Advanced Threat Analytics とは](/advanced-threat-analytics/what-is-ata)」を参照してください。

- **Windows Defender Remote Credential Guard** では、接続を要求しているデバイスに Kerberos 要求をリダイレクトすることにより、リモート デスクトップ接続での資格情報が保護されます。 また、リモート デスクトップ セッションに対するシングル サインオン (SSO) も提供されます。 リモート デスクトップ セッションの間に、ターゲット デバイスが侵害された場合、資格情報と資格情報派生物の両方がネットワーク経由でターゲット デバイスに渡されないため、資格情報は公開されません。

    詳細については、「[Windows Defender Credential Guard を管理する](/windows/security/identity-protection/credential-guard/credential-guard-manage)」を参照してください。

## <a name="next-steps"></a>次のステップ
セキュリティと規制コンプライアンスの詳細については、以下も参照してください。
- [セキュリティおよび保証](/windows-server/security/security-and-assurance)
- [Azure ソリューションのセキュリティに関するベスト プラクティス](https://azure.microsoft.com/resources/security-best-practices-for-azure-solutions/)
