---
title: クラウド ソリューション プロバイダー向けの使用量レポート インフラストラクチャ - Azure Stack Hub
description: クラウド ソリューション プロバイダー (CSP) がサービスを提供するテナントの使用量を追跡するために使用される使用量レポート インフラストラクチャについて説明します。
author: sethmanheim
ms.topic: article
ms.date: 08/27/2020
ms.author: sethm
ms.reviewer: alfredop
ms.lastreviewed: 05/09/2019
ms.openlocfilehash: 51e747b5670a3e7eb58316069567e83e9ed9fc82
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90573052"
---
# <a name="usage-reporting-infrastructure-for-cloud-solution-providers"></a>クラウド ソリューション プロバイダー向けの使用量レポート インフラストラクチャ

Azure Stack Hub には、発生した使用量を追跡して Azure に転送にするために必要なインフラストラクチャが含まれています。 Azure では、適切な Azure サブスクリプションに対して、Azure Commerce によって[使用量データが処理され使用量が課金](azure-stack-billing-and-chargeback.md)されます。 このプロセスは、グローバル Azure クラウドでの使用量の追跡と同様に機能します。

一部の概念はグローバル Azure と Azure Stack Hub の間で一貫性があります。 Azure Stack Hub には、Azure サブスクリプションと同等の役割を果たすローカル サブスクリプションがあります。 ローカル サブスクリプションは、ローカルでのみ有効です。 ローカル サブスクリプションは、使用量が Azure に転送されるときに、Azure サブスクリプションにマップされます。

Azure Stack Hub には、ローカル使用量メーターがあります。 ローカルの使用量は、Azure Commerce で使用されるメーターにマップされます。 ただし、メーター ID は異なります。 ローカルでは、Microsoft が請求に使用しているものを上回る数のメーターをご利用いただけます。

Azure Stack Hub と Azure でのサービスの課金方法にはいくつか違いがあります。 たとえば Azure Stack Hub では、Azure とは異なり、VM は時間あたりの仮想コアのみに基づき、すべての VM シリーズで同じレートです。 これは、グローバル Azure ではハードウェアの違いが価格の違いに反映されるためです。 Azure Stack Hub では、お客様がハードウェアを提供するため、異なる VM クラスに異なるレートを請求する理由がありません。

Commerce で使用される Azure Stack Hub メーターとその価格については、パートナー センターで確認できます。 そのプロセスは、Azure サービスの場合と同じです。

1. パートナー センターで、 **[ダッシュボード] メニュー**に移動し、 **[販売]** を選択してから、 **[料金とプラン]** を選択します。
2. **[使用量ベースのサービス]** で、 **[最新]** を選択します。
3. **Azure in Global CSP price list (グローバル CSP 価格表での Azure)** スプレッドシートを開きます。
4. **リージョン = Azure Stack Hub** でフィルター処理します。

## <a name="terms-used-for-billing-and-usage"></a>請求と使用量に使用される用語

次の用語と概念は、Azure Stack Hub での使用量と請求に使用されます。

| 期間 | 定義 |
| --- | --- |
| 直接 CSP パートナー | 直接 CSP パートナーは、Azure と Azure Stack Hub の使用量について Microsoft から直接請求書を受け取り、顧客に直接請求します。 |
| 間接 CSP | 間接リセラーは、間接プロバイダー (ディストリビューターとも呼ばれます) と連携します。 リセラーはエンド カスタマーを募集します。間接プロバイダーは、Microsoft と請求関係を保持し、顧客への請求を管理し、製品サポートのような追加サービスを提供します。 |
| エンド カスタマー | エンド カスタマーは、Azure Stack Hub で実行されるアプリやその他のワークロードを所有する企業および政府機関です。 |

## <a name="next-steps"></a>次のステップ

- CSP プログラムの詳細については、「[クラウド ソリューション](https://partner.microsoft.com/solutions/microsoft-cloud-solutions)」を参照してください。
- Azure Stack Hub からリソース使用量情報を取得する方法の詳細については、「[Azure Stack Hub での使用量と請求](azure-stack-billing-and-chargeback.md)」をご覧ください。
