---
title: Azure Stack Hub のヘルプとサポート
description: Microsoft Azure Stack Hub のサポートを受けます。
author: PatAltimore
ms.topic: article
ms.date: 02/26/2020
ms.author: patricka
ms.reviewer: shisab
ms.lastreviewed: 02/26/2020
ms.openlocfilehash: cb2fb8ce9ad1b24430cf545eba3f31ac9e754c43
ms.sourcegitcommit: 52c934f5eeb5fcd8e8f2ce3380f9f03443d1e445
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/07/2021
ms.locfileid: "97974321"
---
# <a name="azure-stack-hub-help-and-support"></a>Azure Stack Hub のヘルプとサポート

::: moniker range=">= azs-2002"

Azure Stack Hub のオペレーターは **[ヘルプとサポート]** を使用して診断ログを収集し、トラブルシューティングのために Microsoft に送信することができます。 Azure Stack Hub ポータルの **[ヘルプとサポート]** には、管理者ポータルからアクセスできます。 オペレーターが Azure Stack の詳細を参照し、サポート オプションを確認し、専門家の支援を得るためのリソースが用意されています。  

![Azure Stack Hub 管理者ポータルでヘルプとサポートにアクセスする方法](media/azure-stack-help-and-support/help-and-support.png)

## <a name="help-resources"></a>ヘルプ リソース

オペレーターは、 **[ヘルプとサポート]** を使用して、Azure Stack Hub の詳細を参照し、サポート オプションを確認し、専門家の支援を得ることができます。

### <a name="things-to-try-first"></a>最初に試すべきこと

**[ヘルプとサポート]** の上部には、最初に試すことが推奨される項目があります。たとえば、新しい概念について読む、課金のしくみを学ぶ、使用できるサポート オプションを確認する、などです。

![Azure Stack Hub のセルフサービス サポート](media/azure-stack-help-and-support/get-support-tiles.png)

- **ドキュメント**。 「[Azure Stack Hub のオペレーター ドキュメント](index.yml)」には、Azure Stack Hub サービスを提供する方法を示す概念、操作方法のトピック、およびチュートリアルが掲載されています。 これらのサービスには、仮想マシン、SQL Database、Web アプリなどがあります。

- **課金の詳細についてはこちらを参照してください**。 [使用量と課金](azure-stack-billing-and-chargeback.md)についてのヒントを得ます。

- **サポート オプション**。 Azure Stack Hub オペレーターは、あらゆる企業のニーズに対応できる幅広い [Azure サポート オプション](./azure-stack-manage-basics.md)から選択できます。

### <a name="get-expert-help"></a>専門家のサポートを受ける

統合システムについては、Microsoft と Microsoft の OEM (original equipment manufacturer) ハードウェア パートナーとの間に、統合されたエスカレーションと解決のプロセスがあります。

クラウド サービスに問題がある場合は、Microsoft サポートを通じてサポートが提供されます。 管理者ポータルの右上隅の **[ヘルプ]** (疑問符) を選択し、 **[ヘルプとサポート]** を選択すると、 **[ヘルプとサポートの概要]** が開き、新しいサポート リクエストを送信できます。 サポート要求を作成するときに、Azure Stack Hub サービスが事前に選択されます。 グローバルな Azure portal を使うのではなく、このエクスペリエンスを使ってチケットを送信することをお客様に強くお勧めします。

デプロイ、パッチと更新プログラム、ハードウェア (現場交換可能ユニットを含む) や、ハードウェア ライフサイクル ホストで実行するソフトウェアなどのハードウェア ブランドのソフトウェアに問題がある場合は、最初に OEM ハードウェア ベンダーにお問い合わせください。 その他の問題については、Microsoft サポートにお問い合わせください。

![統合システムの専門家のサポートを受ける](media/azure-stack-help-and-support/get-support-integrated.png)

Azure Stack Development Kit (ASDK) については、[Azure Stack Hub MSDN フォーラム](https://social.msdn.microsoft.com/Forums/azure/home?forum=azurestack)でサポート関連の質問をすることができます。

管理者ポータルの右上隅の **[ヘルプ]** (疑問符) を選択し、 **[ヘルプとサポート]** を選択すると、フォーラムへのリンクが含まれるヘルプとサポートの **概要** が開きます。 MSDN フォーラムは定期的にチェックされています。 ASDK は評価環境であるため、Microsoft サポートを通じた正式なサポートは提供されていません。

また、MSDN フォーラムにアクセスして問題について話し合ったり、オンライン トレーニングを受けて自分のスキルを向上させたりすることもできます。

![Azure Stack Hub の専門家のサポートを受ける](media/azure-stack-help-and-support/get-support-cards.png)

### <a name="get-up-to-speed-with-azure-stack-hub"></a>Azure Stack Hub をすぐに理解する

この一連のチュートリアルは、ASDK と統合システムのどちらを実行しているかに応じてカスタマイズされているので、自分の環境をすぐに理解することができます。

![Azure Stack Hub のサポート チュートリアルを利用する](media/azure-stack-help-and-support/get-support-tutorials.png)

## <a name="diagnostic-log-collection"></a>診断ログの収集

診断ログは、次の 2 つの方法で Microsoft に送信できます。

- [ログを事前に送信する](./azure-stack-diagnostic-log-collection-overview.md#send-logs-proactively):有効にすると、特定の正常性アラートによってログの収集がトリガーされます。
- [今すぐログを送信する](./azure-stack-diagnostic-log-collection-overview.md#send-logs-now):ログ収集の時間枠として、特定のスライディング ウィンドウを手動で選択できます。

![診断ログの収集を開始する方法を示すスクリーンショット。](media/azure-stack-help-and-support/banner-enable-automatic-log-collection.png)

::: moniker-end
::: moniker range="<= azs-1910"

## <a name="diagnostic-log-collection"></a>診断ログの収集

1907 リリース以降、**ヘルプとサポート** にはログを収集するために 2 つの新しい方法があります。

- **自動収集**:有効にすると、特定の正常性アラートによってログの収集がトリガーされます。
- **Collect logs now (今すぐログを収集する)** :過去 7 日間の 1 時間から 4 時間のスライディング ウィンドウを選択できます。

![診断ログの収集オプション](media/azure-stack-automatic-log-collection/azure-stack-log-collection-overview.png)

統合システムでは、Microsoft サポートと診断ログを共有できます。 Azure Stack Development Kit (ASDK) は評価環境であるため、Microsoft サポートではサポートされません。 詳細については、[Azure Stack Hub 診断ログの収集の概要](azure-stack-diagnostic-log-collection-overview.md)に関するページを参照してください。

## <a name="help-and-support-for-earlier-releases-azure-stack-hub-pre-1905"></a>以前のリリースの Azure Stack Hub (1905 より前) に関するヘルプとサポート

前の Azure Stack Hub のリリースにも、[Azure Stack Hub のオペレーター ドキュメント](./index.yml)にリダイレクトされる **[ヘルプとサポート]** へのリンクがあります。

![サポート利用のチュートリアル](media/azure-stack-help-and-support/get-support-previous.png)

クラウド サービスに問題がある場合は、Microsoft サポートを通じてサポートが提供されます。 管理者ポータルの右上隅の **[ヘルプ]** (疑問符) を選択し、 **[ヘルプとサポート]** を選択してから、 **[新しいサポート リクエスト]** を選択して、Microsoft サポートで新しいサポート リクエストを直接送信できます。

統合システムの場合は、Microsoft と OEM パートナーとの間に、調整されたエスカレーションと解決のプロセスがあります。 クラウド サービスに問題がある場合は、Microsoft サポートを通じてサポートが提供されます。

デプロイ、パッチと更新プログラム、ハードウェア (現場交換可能ユニットを含む) や、ハードウェア ライフサイクル ホストで実行するソフトウェアなどのハードウェア ブランドのソフトウェアに問題がある場合は、最初に OEM ハードウェア ベンダーにお問い合わせください。 その他の問題については、Microsoft サポートにお問い合わせください。

ASDK については、[Azure Stack Hub MSDN フォーラム](https://social.msdn.microsoft.com/Forums/azure/home?forum=azurestack)でサポート関連の質問をすることができます。

管理者ポータルの右上隅の **[ヘルプ]** (疑問符) を選択し、 **[新しいサポート要求]** を選択すると、Azure Stack Hub コミュニティの専門家からのヘルプを得られます。 ASDK は評価環境であるため、Microsoft サポートを通じた正式なサポートは提供されていません。

::: moniker-end

## <a name="next-steps"></a>次のステップ

- [診断ログの収集](./azure-stack-diagnostic-log-collection-overview.md)について学習する。
- [クラウド ID を検索する](azure-stack-find-cloud-id.md)方法について学習する。
- [Azure Stack Hub のトラブルシューティング](azure-stack-troubleshooting.md)について学習する。