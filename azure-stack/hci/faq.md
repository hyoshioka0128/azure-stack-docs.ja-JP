---
title: Azure Stack HCI の FAQ
description: Azure Stack HCI の FAQ。
ms.topic: conceptual
author: JohnCobb1
ms.author: v-johcob
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 1/21/2021
ms.openlocfilehash: 6c84bad0de72235c6ccde1b8c23016afeb956cba
ms.sourcegitcommit: c87d1e26a4f96be4651f63fbf5ea3d98d6f14832
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/21/2021
ms.locfileid: "98659359"
---
# <a name="azure-stack-hci-faq"></a>Azure Stack HCI の FAQ
この Azure Stack HCI の FAQ は、Azure Stack HCI の接続に関するセクションと一般的な FAQ のセクションで構成されています。

## <a name="azure-stack-hci-connectivity"></a>Azure Stack HCI の接続
Azure Stack HCI は、Azure のハイブリッド サービスとして提供されるオンプレミスのハイパーコンバージド インフラストラクチャ スタックです。 Azure Stack HCI ソフトウェアは、オンプレミスで管理する物理サーバーにインストールします。 その後、Azure に接続することで、クラウドベースの監視、サポート、課金のほか、必要に応じて管理機能やセキュリティ機能が利用できます。 この FAQ セクションでは、接続の要件や動作に関してよく寄せられる質問に答えることで、Azure Stack HCI でクラウドがどのように使用されているかを明らかにしていきます。

### <a name="your-data-stays-on-premises"></a>データはオンプレミスに留まる

**Azure Stack HCI に格納されたデータはクラウドに送信されますか?**

いいえ。 オンプレミス仮想マシン (VM) の名前、メタデータ、構成、コンテンツなどの顧客データがクラウドに送信されることは決してありません。ただし、それを目的として Azure Backup や Azure Site Recovery などの追加サービスを明示的に有効にした場合、またはそれらの VM を個別に Azure Arc などのクラウド管理サービスに登録した場合を除きます。

Azure Stack HCI のセキュリティを維持しながら最新状態に保ち、予期した通りに作動するために収集する診断データの詳細については、「[Azure Stack HCI のデータ コレクション](concepts/data-collection.md)」および「[Azure でのデータ所在地](https://azure.microsoft.com/global-infrastructure/data-residency/)」を参照してください。

### <a name="edge-local-management-and-control"></a>エッジローカルな管理と制御

**Azure Stack HCI のコントロール プレーンはクラウドを経由しますか?**

いいえ。 クラウドへのネットワーク接続がダウンしたり著しく制限されたりしていても、エッジローカルなツール (Windows Admin Center、PowerShell、System Center など) を使用して直接、ホスト インフラストラクチャと VM を管理できます。 ホスト間での VM の移動、故障したドライブの交換、IP アドレスの構成など、一般的な日常業務にはクラウドは使用されません。 ただし、無線でのソフトウェア更新プログラムの入手、Azure の登録の変更、(バックアップや監視目的などで) クラウド サービスを直接利用する機能の使用には、クラウド接続が必要になります。

**Azure Stack HCI とクラウドの間に帯域幅や待ち時間の要件はありますか?**

いいえ。 Azure Stack HCI の同期には、帯域幅が制限された接続 (地方の T1 回線、衛星接続、携帯ネットワーク接続など) で十分です。最低限必要な接続は、1 日あたり数キロバイトにすぎません。 その他のサービスでは、特に、VM 全体のレプリケートやバックアップ、大きなソフトウェア更新プログラムのダウンロード、クラウドでの分析と監視を目的とした詳細ログのアップロードのために、追加の帯域幅が必要になる場合があります。

### <a name="designed-for-intermittent-and-limited-connectivity"></a>間欠的な接続や制限された接続を視野に入れた設計

**Azure Stack HCI では、クラウドへの継続的な接続が必要ですか?**

いいえ。 Azure Stack HCI は、接続が限られている期間やまったく接続できない期間に対処するように設計されています。

**クラウドへのネットワーク接続が一時的にダウンした場合はどうなりますか?**

接続がダウンしている間も、ホスト インフラストラクチャと VM はすべて正常に動作し続け、エッジローカルなツールを使用して管理することができます。 クラウド サービスを直接利用する機能は使用できません。 Azure Stack HCI が再び同期できるようになるまでは、Azure portal の情報も古くなります。

**Azure Stack HCI は、接続がダウンした状態でどの程度の期間動作できますか?**

Azure Stack HCI は、最低でも 30 日間に 1 回は、Azure と正常に同期される必要があります。

**30 日間の制限を超えるとどうなりますか?**

Azure Stack HCI が Azure と同期されないまま 30 日が経過した場合は、Azure portal や他のツールで、クラスターの接続状態が "**Out of policy (ポリシー違反)** " と表示され、クラスターが機能制限モードに移行します。 このモードでも、ホスト インフラストラクチャは稼動状態を維持し、現在の VM はすべて正常に動作し続けます。 ただし、Azure Stack HCI が再度同期できるようになるまで、新しい VM は作成できません。 内部的かつ技術的な理由として、クラウドによって生成されるクラスターのライセンスが期限切れになっていて、Azure と同期することで更新する必要があることが挙げられます。

### <a name="understanding-sync"></a>同期とは

**Azure Stack HCI とクラウドとの間でどのような内容が同期されますか?**

これは、使用している機能によって異なります。 少なくとも、Azure Stack HCI では、Azure portal に表示される基本的なクラスター情報 (クラスター ノードのリスト、ハードウェア モデル、ソフトウェア バージョンなど)、前回の同期以降に生じたコア日数を集計した課金情報のほか、Microsoft が Azure Stack HCI を安全かつ最新の状態に保ち、正常に動作させ続けるために最低限必要な診断情報が同期されます。 その合計サイズは小さく、数キロバイトです。 有効にするサービスが追加されると、アップロードされるデータも増えます。たとえば、Azure Log Analytics では、監視目的でログとパフォーマンス カウンターがアップロードされます。

**Azure Stack HCI とクラウドが同期される頻度はどのくらいですか?**

これは、使用している機能によって異なります。 Azure Stack HCI は、少なくとも 12 時間おきに同期を試みます。 同期に失敗した場合、その内容はローカルに保持され、次回同期が成功したときに送信されます。この定期的な実行に加え、PowerShell コマンドレット `Sync-AzureStackHCI` または Windows Admin Center を使用していつでも手動で同期できます。 有効にするサービスが追加されると、アップロードの頻度も高くなります。たとえば、Azure Log Analytics では、監視目的でアップロードが 5 分おきに実行されます。

### <a name="data-residency"></a>データの保存場所

**同期された情報は、実際にはどこに送信されますか?**

Azure Stack HCI は Azure と同期され、データは Microsoft が運営するセキュリティで保護されたデータセンターに格納されます。 Azure Stack HCI のセキュリティを維持しながら最新状態に保ち、予期した通りに作動するために収集する診断データの詳細については、「[Azure Stack HCI のデータ コレクション](concepts/data-collection.md)」および「[Azure でのデータ所在地](https://azure.microsoft.com/global-infrastructure/data-residency/)」を参照してください。

### <a name="disconnected-or-air-gapped"></a>非接続 (エアギャップ) 環境

**Azure に一切接続せずに Azure Stack HCI を使用することはできますか?**

いいえ。 Azure Stack HCI は、30 日間に 1 回は、Azure との間で正常に同期される必要があります。

**"エアギャップ" 環境の Azure Stack HCI と Azure の間でデータをオフラインで転送することはできますか?**

いいえ。 現在、ネットワーク接続を使用せずに、オンプレミスと Azure の間で登録および同期するメカニズムはありません。 たとえば、リムーバブル記憶域を使用して、証明書や課金データを転送することはできません。 多くのお客様からご要望があれば、将来そのような機能を取り入れることも検討します。 [Azure Stack HCI のフィードバック フォーラム](https://feedback.azure.com/forums/929833-azure-stack-hci)にご意見をお寄せください。

## <a name="azure-stack-hci-general-faqs"></a>Azure Stack HCI の一般的な FAQ

**Azure Stack HCI は Windows Server とどのような関係になっていますか。**

Windows Server はほぼすべての Azure 製品の基盤であり、お持ちのすべての機能は、Windows Server でサポートされている状態で引き続きリリースされます。 Azure Stack HCI の初期オファリングは、Windows Server 2019 が基になっており、従来の Windows Server ライセンス モデルを使用していました。 現在の Azure Stack HCI には、独自のオペレーティング システムとサブスクリプション ベースのライセンス モデルがあります。 Azure Stack HCI は、弊社のパートナーから提供され、Microsoft による検証を受けたハードウェアを使用して HCI をオンプレミスでデプロイするためのお勧めの方法です。

**Windows Server 2019 から Azure Stack HCI にアップグレードできますか。**

現時点では、Windows Server から Azure Stack HCI へのインプレース アップグレードはありません。 Windows Server 2019 および 2016 に基づくハイパーコンバージド クラスターを実行しているお客様向けの具体的な移行ガイダンスについては、しばらくお待ちください。

**どの Azure サービスを Azure Stack HCI に接続できますか。**

Azure Stack HCI を接続できる Azure サービスの最新一覧については、「[Connecting Windows Server to Azure hybrid services (Windows Server と Azure ハイブリッド サービスの接続)](/windows-server/manage/windows-admin-center/azure/index)」をご覧ください。

**Azure Stack Hub と Azure Stack HCI ソリューションはどのような点が共通していますか。**

Azure Stack HCI には、Azure Stack Hub と同じ、Hyper-V ベースのソフトウェアによるコンピューティング、ストレージ、ネットワーク テクノロジが搭載されています。 どちらのオファリングも、厳しいテストおよび検証基準を満たしており、信頼性および基盤となるハードウェア プラットフォームとの互換性が確保されています。

**これらはどのように違うのでしょうか。**

Azure Stack Hub では、オンプレミスでクラウド サービスを運用します。 Azure IaaS および PaaS サービスをオンプレミスで実行して、一貫性を保ちながら任意の場所でクラウド アプリを構築および実行し、オンプレミスで Azure portal を使用して管理できます。

Azure Stack HCI では、仮想化されたワークロードをオンプレミスで実行し、Windows Admin Center と使い慣れた Windows Server ツールを使用して管理します。 また、クラウドベースのサイトの回復、監視などのハイブリッド シナリオのために Azure に接続できます。

**Azure Stack HCI から Azure Stack Hub にアップグレードできますか。**

いいえ。ただし、お客様は Azure Stack HCI から Azure Stack Hub または Azure にワークロードを移行できます。

**Azure Stack HCI サーバーを識別するにはどうすればよいですか。**

Windows Admin Center では、[すべての接続] の一覧や他のさまざまな場所に、オペレーティング システムの一覧が表示されます。または、次の PowerShell コマンドを使用して、オペレーティング システムの名前とバージョンを照会することもできます。

```PowerShell
Get-ComputerInfo -Property 'osName', 'osDisplayVersion'
```

出力例を次に示します。

```
OsName                    OSDisplayVersion
------                    ----------------
Microsoft Azure Stack HCI 20H2
```
