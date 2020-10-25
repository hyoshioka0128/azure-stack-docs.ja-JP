---
title: Azure Stack Hub Marketplace のよくあるご質問
titleSuffix: Azure Stack Hub
description: Azure Stack Hub Marketplace の Windows Server に関する FAQ 一覧。
author: sethmanheim
ms.topic: article
ms.date: 07/23/2020
ms.author: sethm
ms.reviewer: avishwan
ms.lastreviewed: 08/29/2019
ms.openlocfilehash: fff299a0d537bb4190e66a57eb642db7e8b9824d
ms.sourcegitcommit: f2a5ce52fcf69e05fe89be8211b7360de46f4a94
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/24/2020
ms.locfileid: "87133641"
---
# <a name="azure-stack-hub-marketplace-faq"></a>Azure Stack Hub Marketplace のよくあるご質問

この記事では、[Azure Stack Hub Marketplace](azure-stack-marketplace.md) 内の Marketplace アイテムについてよく寄せられるいくつかの質問に回答します。

## <a name="marketplace-items"></a>Marketplace アイテム

### <a name="who-should-i-contact-for-support-issues-with-azure-stack-hub-marketplace-items"></a>Azure Stack Hub Marketplace アイテムのサポートに関する問題については、だれに問い合わせる必要がありますか?

Azure Marketplace のサポート ガイダンスは、Azure Stack Hub Marketplace アイテムにも適用されます。 発行元は、Azure Stack Hub Marketplace 上の製品のテクニカル サポートを提供する責任を負います。 Azure Marketplace アイテムのサポート ガイダンスの詳細については、Azure Marketplace FAQ 記事の[サポートに関するセクション](/azure/marketplace/marketplace-faq-publisher-guide#customer-support)を参照してください。

### <a name="how-do-i-update-to-a-newer-windows-image"></a>より新しい Windows イメージに更新するにはどうすればよいですか?

まず、Azure Resource Manager テンプレートが特定のバージョンを参照しているかどうかを確認します。 該当する場合は、それらのテンプレートを更新するか、以前のバージョンのイメージを維持します。 **version: latest** を使用することをお勧めします。

次に、仮想マシン スケール セットが特定のバージョンを参照している場合、これらを後でスケーリングするかどうかを検討し、さらに以前のバージョンを維持するかどうかを決める必要があります。 どの条件も該当しない場合は、新しいイメージをダウンロードする前に、Azure Stack Hub Marketplace 内の以前のイメージを削除してください。 元のイメージのダウンロードに Marketplace 管理を使用した場合は、それを使って削除してください。 そのうえで、新しいバージョンをダウンロードします。

### <a name="what-are-the-licensing-options-for-windows-server-images-on-azure-stack-hub-marketplace"></a>Azure Stack Hub Marketplace には、Windows Server イメージにどのようなライセンス オプションがありますか?

Microsoft は、2 つのバージョンの Windows Server イメージを Azure Stack Hub Marketplace で提供しています。 Azure Stack Hub 環境で使用できるのは、このイメージの 1 バージョンのみです。  

- **従量課金制 (PAYG)** : これらのイメージでは、正規の価格の Windows メーターが実行されます。
   このオプションの対象ユーザー: "*使用量課金モデル*" を利用するマイクロソフト エンタープライズ契約 (EA) のお客様や、SPLA ライセンスの使用を希望しない CSP にお勧めします。
- **ライセンス持ち込み (BYOL)** : これらのイメージでは、基本メーターが実行されます。
   このオプションの対象ユーザー: Windows Server ライセンスを所有する EA のお客様や、SPLA ライセンスを使用する CSP にお勧めします。

Azure Stack Hub では、Azure ハイブリッド使用特典 (AHUB) はサポートされません。 "容量" モデルを通じてライセンスを受けるお客様は、BYOL イメージを使用する必要があります。 Azure Stack Development Kit (ASDK) を使ってテストを行う場合は、どちらのオプションを使用してもかまいません。

### <a name="what-if-i-downloaded-the-wrong-version-to-offer-my-tenantsusers"></a>テナント/ユーザーに提供する場合に間違ったバージョンをダウンロードした場合はどうすればよいですか?

まず、[marketplace management]\(Marketplace の管理\) から、間違ったバージョンを削除します。 完了するまでお待ちください ( **[marketplace management]\(Marketplace の管理\)** ブレードではなく、通知を見て完了を確認してください)。 そのうえで、正しいバージョンをダウンロードします。

両方のバージョンのイメージをダウンロードした場合、エンド ユーザーの Azure Stack Hub Marketplace には最新バージョンのみが表示されます。

### <a name="what-if-my-user-incorrectly-checked-the-i-have-a-license-box-in-previous-windows-builds-and-they-dont-have-a-license"></a>実際にはライセンスを持っていないにもかかわらず、ユーザーが間違って以前の Windows ビルドの [I have a license]\(ライセンスを持っています\) チェック ボックスをオンにしてしまった場合はどうすればよいですか?

次のスクリプトを実行して、ライセンス モデル属性を変更して、BYOL から PAYG モデルに切り替えることができます。

```powershell
$vm= Get-Azurermvm -ResourceGroup "<your RG>" -Name "<your VM>"
$vm.LicenseType = "None"
Update-AzureRmVM -ResourceGroupName "<your RG>" -VM $vm
```

VM のライセンスの種類を確認するには、次のコマンドを実行します。 ライセンス モデルに、**Windows_Server** と示されている場合、BYOL 料金で課金されます。 それ以外の場合は、PAYG モデルごとに Windows の測定に対して課金されます。

```powershell
$vm | ft Name, VmId,LicenseType,ProvisioningState
```

### <a name="what-if-i-have-an-older-image-and-my-user-forgot-to-check-the-i-have-a-license-box-or-we-use-our-own-images-and-we-do-have-enterprise-agreement-entitlement"></a>私が以前のイメージを所有しているにもかかわらず、私のユーザーが [I have a license]\(ライセンスを持っています\) チェック ボックスをオンにするのを忘れてしまった場合は、どうすればよいですか。または、私たちが独自のイメージを使用していて、なおかつマイクロソフト エンタープライズ契約の権利がある場合はどうすればよいですか?

次のコマンドを実行して、ライセンス モデル属性を BYOL モデルに変更できます。

```powershell
$vm= Get-Azurermvm -ResourceGroup "<your RG>" -Name "<your VM>"
$vm.LicenseType = "Windows_Server"
Update-AzureRmVM -ResourceGroupName "<your RG>" -VM $vm
```

### <a name="what-about-other-vms-that-use-windows-server-such-as-sql-or-machine-learning-server"></a>Windows Server を使用する他の VM (SQL Server、Machine Learning Server など) についてはどうでしょうか?

これらのイメージは、**licenseType** パラメーターが適用されるため、PAYG となります。 このパラメーターは自分で設定することができます (前出の FAQ の回答を参照)。 これが適用されるのは、Windows Server ソフトウェアだけです。自分のライセンスの持ち込みが必要となる階層化された製品 (SQL など) には適用されません。 PAYG ライセンスは、階層化されたソフトウェア製品には適用されません。

Azure Stack Hub Marketplace から SQL Server イメージの **licenseType** プロパティを変更できるのは、バージョンが **XX.X.20190410** 以降の場合のみです。 Azure Stack Hub Marketplace から古いバージョンの SQL Server イメージを実行している場合は、**licenseType** 属性を変更することはできず、Azure Stack Hub Marketplace から最新の SQL Server イメージを使用して再デプロイする必要があります。

### <a name="i-have-an-enterprise-agreement-ea-and-will-be-using-my-ea-windows-server-license-how-do-i-make-sure-images-are-billed-correctly"></a>私はマイクロソフト エンタープライズ契約 (EA) を所有しており、EA Windows Server ライセンスを使う予定です。イメージに対して正しく課金されるようにするには、どうすればよいですか?

**licenseType: Windows_Server** を Azure Resource Manager テンプレートに追加してください。 この設定は、個々の仮想マシン (VM) のリソース ブロックに追加する必要があります。

## <a name="activation"></a>アクティブ化

Azure Stack Hub で Windows Server VM のライセンス認証を行うには、次の条件が満たされている必要があります。

- Azure Stack Hub のすべてのホスト システムに対し、OEM によって適切な BIOS マーカーが設定されていること。
- Windows Server 2012 R2 および Windows Server 2016 で [VM の自動ライセンス認証](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn303421(v=ws.11))が使用されていること。 キー管理サービス (KMS) などのライセンス認証サービスは、Azure Stack Hub ではサポートされていません。

### <a name="how-can-i-verify-that-my-vm-is-activated"></a>自分の VM がライセンス認証されたことは、どのようにして確認できますか?

管理者特権でのコマンド プロンプトで、次のコマンドを実行します。

```shell
slmgr /dlv
```

正しくライセンス認証されていれば、`slmgr` の出力にその旨が明記され、ホスト名が表示されます。 ディスプレイ上の透かしに頼らないようにしてください。これらは最新情報でなかったり、ご利用のマシンの背後にある異なる VM の表示であったりすることもあります。

### <a name="my-vm-isnt-set-up-to-use-avma-how-can-i-fix-it"></a>私の VM は AVMA を使用するように設定されていません。解決方法を教えてください。

管理者特権でのコマンド プロンプトで、次のコマンドを実行します。

```shell
slmgr /ipk <AVMA key>
```

イメージに使用するキーについては、記事「[仮想マシンの自動ライセンス認証](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn303421(v=ws.11))」を参照してください。

### <a name="i-create-my-own-windows-server-images-how-can-i-make-sure-they-use-avma"></a>独自の Windows Server イメージを作成しています。確実に AVMA が使用されるようにするには、どうすればよいですか?

`sysprep` コマンドを実行する前に、適切なキーを指定して `slmgr /ipk` コマンド ラインを実行することをお勧めします。 または、Unattend.exe セットアップ ファイルに AVMA キーを含めます。

### <a name="i-am-trying-to-use-my-windows-server-2016-image-created-on-azure-and-its-not-activating-or-using-kms-activation"></a>Azure 上で作成した Windows Server 2016 イメージを使おうとしているのですが、ライセンス認証が実行されません (または KMS ライセンス認証が使用されません)

`slmgr /ipk` コマンドを実行します。 Azure のイメージが正しく AVMA にフォールバックされないことがありますが、Azure KMS システムに到達できれば、ライセンス認証が行われます。 これらの VM が AVMA を使用する設定になっていることを確認するようお勧めします。

### <a name="i-have-performed-all-of-these-steps-but-my-vms-are-still-not-activating"></a>これらの手順はすべて実行しましたが、まだ VM がライセンス認証されません

ハードウェア サプライヤーに問い合わせて、正しい BIOS マーカーがインストールされていることを確認してください。

### <a name="what-about-earlier-versions-of-windows-server"></a>以前のバージョンの Windows Server についてはどうでしょうか?

以前のバージョンの Windows Server では、[VM の自動ライセンス認証](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn303421(v=ws.11))はサポートされていません。 VM を手動でアクティブにする必要があります。

## <a name="next-steps"></a>次のステップ

詳細については、次の記事を参照してください。

- [Azure Stack Hub Marketplace の概要](azure-stack-marketplace.md)
- [Azure から Azure Stack Hub に Marketplace の項目をダウンロードする](azure-stack-download-azure-marketplace-item.md)
