---
title: Azure Stack の Windows Server に関する FAQ - MDC | Microsoft Docs
description: Modular Data Center (MDC) で操作されている場合の Azure Stack Marketplace の Windows Server に関する FAQ の一覧をお読みください。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: tzl
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/26/2020
ms.author: sethm
ms.reviewer: avishwan
ms.lastreviewed: 10/26/2020
ms.openlocfilehash: 64ccefb0a6644d2cd613a39d8beaaf1f82ee9fc0
ms.sourcegitcommit: d719f148005e904fa426a001a687e80730c91fda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/06/2021
ms.locfileid: "97910790"
---
# <a name="windows-server-in-azure-stack-marketplace-faq---modular-data-center-mdc"></a>Azure Stack Marketplace 内の Windows Server に関する FAQ - Modular Data Center (MDC)

この記事では、[Azure Stack Hub Marketplace](../../operator/azure-stack-marketplace.md) 内の Windows Server イメージについてよく寄せられるいくつかの質問に回答します。

## <a name="faqs"></a>FAQ

**Azure Stack における Windows Server Marketplace イメージのライセンス オプションを教えてください。**

Azure Stack Hub ラグドと MDC のユーザーは、Windows Server をゲスト オペレーティング システムとして無料で利用する資格を持っています。

Microsoft は、2 つのバージョンの Windows Server イメージを Azure Stack Marketplace で提供しています。 特定の Azure Stack 環境内で一度に使用できるのは、このイメージの 1 バージョンのみです。

- **従量課金制**: これらのイメージを、Azure Stack Hub ラグドまたは MDC 上で使用することはできません。

- **ライセンス持ち込み (BYOL)** : これらのイメージは、Azure Stack Hub ラグドおよび MDC で使用できます。

**Windows Server を使用する他の VM (SQL Server など) についてはどうでしょうか?**

Windows Server ソフトウェア ライセンスが適用されるのは、Windows オペレーティング システムだけです。自分のライセンスの持ち込みが必要となる階層化された製品 (SQL など) には適用されません。

**より新しい Windows イメージに更新するにはどうすればよいですか?**

まず、Azure Resource Manager テンプレートが特定のバージョンを参照しているかどうかを確認します。 該当する場合は、それらのテンプレートを更新するか、以前のバージョンのイメージを維持します。 **version: latest** を使用することをお勧めします。

次に、仮想マシン スケール セットが特定のバージョンを参照している場合、これらを後でスケーリングするかどうかを検討し、さらに以前のバージョンを維持するかどうかを決める必要があります。 どの条件も該当しない場合は、新しいイメージをダウンロードする前に、Marketplace 内の以前のイメージを削除してください。 元のイメージのダウンロードに Marketplace 管理を使用した場合は、それを使って行ってください。 そのうえで、新しいバージョンをダウンロードします。

**ユーザーに提供する場合に間違ったバージョンをダウンロードした場合はどうすればよいですか?**

まず、[marketplace management]\(Marketplace の管理\) から、間違ったバージョンを削除します。 完了するまでお待ちください ( **[marketplace management]\(Marketplace の管理\)** ブレードではなく、通知を見て完了を確認してください)。 そのうえで、正しいバージョンをダウンロードします。

両方のバージョンのイメージをダウンロードした場合、エンド ユーザーのマーケットプレース ギャラリーには最新バージョンのみが表示されます。

**Windows Server 仮想マシンをアクティブにするにはどうすればよいですか?**

Azure Stack で Windows Server 仮想マシンのライセンス認証を行うには、次の条件が満たされている必要があります。

- Windows Server 2012 R2 および Windows Server 2016 で[仮想マシンの自動ライセンス認証](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn303421(v=ws.11))が使用されていること。 キー管理サービス (KMS) などのライセンス認証サービスは、Azure Stack ではサポートされていません。

**自分の仮想マシンがライセンス認証されたことは、どのようにして確認できますか?**

管理者特権でのコマンド プロンプトで、次のコマンドを実行します。

```shell
slmgr /dlv
```

正しくライセンス認証されていれば、slmgr の出力にその旨が明記され、ホスト名が表示されます。 ディスプレイ上の透かしに頼らないようにしてください。これらは最新情報でなかったり、ご利用のマシンの背後にある異なる仮想マシンの表示であったりすることもあります。

**私の VM は AVMA を使用するように設定されていません。解決方法を教えてください。**

管理者特権でのコマンド プロンプトで、次のコマンドを実行します。

```shell
slmgr /ipk <AVMA key>
```

イメージに使用するキーについては、記事「[Automatic Virtual Machine Activation (仮想マシンの自動ライセンス認証)](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn303421(v=ws.11))」を参照してください。

**独自の Windows Server イメージを作成しています。確実に AVMA が使用されるようにするには、どうすればよいですか?**

`sysprep` コマンドを実行する前に、適切なキーを指定して `slmgr /ipk` コマンドを実行することをお勧めします。 または、Unattend.exe セットアップ ファイルに AVMA キーを含めます。

**Azure 上で作成した Windows Server 2016 イメージを使おうとしているのですが、ライセンス認証が実行されません (または KMS ライセンス認証が使用されません)。**

`slmgr /ipk` コマンドを実行します。 Azure のイメージが正しく AVMA にフォールバックされないことがありますが、Azure KMS システムに到達できれば、ライセンス認証が行われます。 これらの VM が AVMA を使用する設定になっていることを確認するようお勧めします。

**これらの手順はすべて実行しましたが、まだ仮想マシンがライセンス認証されません。**

Microsoft サポートに問い合わせて、正しい BIOS マーカーがインストールされていることをご確認ください。

**以前のバージョンの Windows Server についてはどうでしょうか?**

2012 R2 より前のバージョンの Windows Server では、[仮想マシンの自動ライセンス認証](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn303421(v=ws.11))はサポートされていません。 MAK キーを使用して、VM を手動でアクティブにする必要があります。

## <a name="next-steps"></a>次のステップ

詳細については、次の記事を参照してください。

- [Azure Stack Marketplace の概要](../../operator/azure-stack-marketplace.md)
- [Azure から Azure Stack に Marketplace の項目をダウンロードする](azure-stack-download-azure-marketplace-item-tca.md)
