---
title: Azure Stack HCI の課金と支払い
description: Azure Stack HCI の課金と支払いのしくみ。
author: khdownie
ms.author: v-kedow
ms.topic: conceptual
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 09/01/2020
ms.openlocfilehash: 22758f9a6b905e2c948a47c5870a05edcb63acdf
ms.sourcegitcommit: 1c5e7d8419037c0f3ef6fe9d8e6bfb6a59659c84
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/03/2020
ms.locfileid: "89428483"
---
# <a name="azure-stack-hci-billing-and-payment"></a>Azure Stack HCI の課金と支払い

> 適用対象:Azure Stack HCI バージョン v20H2

Azure Stack HCI は、他の Azure サービスと同様に、Azure サブスクリプションの請求に課金される Azure サービスです。 ゲスト仮想マシン (VM) には個々のオペレーティング システムのライセンスが必要になる場合がありますが、従来のオンプレミス ソフトウェア ライセンスは必要ありません。 通貨と割引は、Azure コマースの請求プラットフォームによって一元的に処理されます。お客様には、月末に 1 件の統合された請求明細書が届きます。

## <a name="what-does-azure-stack-hci-charge-for"></a>Azure Stack HCI の課金のしくみ

Azure または他のクラウド サービスを既に使用しているお客様にとって、Azure Stack HCI のクラウドスタイルの課金モデルは一貫性があり、使い慣れた簡単なものです。 パブリック プレビューの期間中は無料です。 プレビュー終了後は、Azure Stack HCI クラスターの物理プロセッサ コアあたりの定額料金に基づいて課金されます (他の Azure サービスを使用している場合は、追加の使用料金が適用されます)。

この価格モデルは、VM の数に基づく使用量ベースまたは消費ベースの課金とは異なります。 仮想プロセッサ コア (VCPU) の数は、1 か月の間で変化する場合がありますが、Azure Stack HCI の料金には影響しません。課金はクラスター内に存在する物理コアの数に基づいて行われます。

## <a name="advantages-of-the-azure-stack-hci-billing-model"></a>Azure Stack HCI 課金モデルの利点

- 単純であること。最小値、最大値、およびメモリ、ストレージ、ネットワークのイングレス/エグレスに関連する計算はありません。
- 物理コアに対する仮想コアの密度が高まることで、お客様にとっては仮想化インフラストラクチャの実行効率が上がるという利点があります。
- オンプレミスの Azure Stack HCI デプロイのコストを簡単に把握し、ネットワーク エッジからデータセンターまでのコストの増減は予測どおりになります。

## <a name="how-the-number-of-processor-cores-is-assessed"></a>プロセッサ コアの数の評価方法

Azure Stack HCI を使用すると、クラスター内に存在するコアの数を確認するために、物理コアの数が定期的に確認され、Azure に報告されます。 常時接続されていない場合や、接続が中断されている場合は、後で再試行されます。 数日分または数週間分のコア データを一度にアップロードできます。 お客様は少なくとも 30 日ごとに 1 回、Azure に接続する必要があります (課金のため)。

コア データを Azure に手動でアップロードするには、 **`Sync-AzureStackHCI`** コマンドレットを使用します。

## <a name="faq"></a>よく寄せられる質問

- Azure サブスクリプションを既に所有している場合、それを Azure Stack HCI に使用できますか? **はい**
- 組織の財務部門が既に Azure での支出を承認している場合、Azure Stack HCI は対象になりますか? **はい**
- 支出対象の Azure コミットメントがある場合、それを Azure Stack HCI に使用できますか? **はい**
- Azure クレジットを所有している場合 (学生の場合や賞金として授与された場合)、それを Azure Stack HCI に使用できますか? **はい**
- 組織に対してマイクロソフトエンタープライズ契約による割引が適用されている場合、それは Azure Stack HCI に適用されますか? **はい**
- Azure portal コスト管理ツールは Azure Stack HCI で動作しますか? **はい**
- Azure Billing Api を使用して構築されたサードパーティまたはカスタム ツールは Azure Stack HCI で動作しますか? **はい**

## <a name="next-steps"></a>次のステップ

関連情報については、以下もご覧ください。

- [料金の概要—Azure の料金体系について](https://azure.microsoft.com/pricing/)
- [Azure Cost Management と Billing の概要](/azure/cost-management-billing/cost-management-billing-overview)
