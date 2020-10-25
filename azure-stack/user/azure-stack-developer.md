---
title: Azure Stack Hub 向けのアプリを開発する
description: Azure サービスを使用して Azure Stack Hub でアプリのプロトタイプを作成する際の開発の考慮事項。
author: sethmanheim
ms.topic: article
ms.date: 06/12/2020
ms.author: sethm
ms.reviewer: unknown
ms.lastreviewed: 05/21/2019
ms.openlocfilehash: 9aea9074b490a9d6e99ad4da0967d88ed63db04a
ms.sourcegitcommit: dd140b3a2ac8e558eae9f5f422711d2ba560da16
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/12/2020
ms.locfileid: "84744846"
---
# <a name="develop-for-azure-stack-hub"></a>Azure Stack Hub 向けの開発

Azure Stack Hub 環境へのアクセス権がなくても、アプリの開発にすぐに取り掛かることができます。 Azure Stack Hub では、お使いのデータセンターで実行されている Microsoft Azure サービスが提供されます。つまり、Azure Stack Hub で同じ Azure ツールとプロセスを使用して、開発できるということです。

## <a name="development-considerations"></a>開発の考慮事項

いくつかの準備をして次のトピックのガイダンスに従えば、Azure を使用して Azure Stack Hub 環境をエミュレートできます。

* Azure で、Azure Stack Hub にデプロイできる Azure Resource Manager テンプレートを作成できます。 移植性が確保されたテンプレートの作成について詳しくは、[テンプレートに関する考慮事項](azure-stack-develop-templates.md)をご覧ください。
* Azure と Azure Stack Hub 間には、サービスの可用性とサービスのバージョン管理に違いがあります。 [Azure Stack Hub ポリシー モジュール](azure-stack-policy-module.md)を使用して、Azure サービスの可用性やリソースの種類を Azure Stack Hub で使用できるものに制限できます。 サービスを制限することで、Azure Stack Hub で使用できるサービスにアプリが依存することを確実にします。
* [Azure Stack Hub クイックスタート テンプレート](https://github.com/Azure/AzureStack-QuickStart-Templates)は、Azure と Azure Stack Hub の両方にデプロイできるテンプレートを開発する方法を示す一般的なシナリオ例です。

## <a name="next-steps"></a>次のステップ

Azure Stack 開発の詳細については、次の記事をご覧ください。

* [Azure Resource Manager テンプレートのベスト プラクティス](azure-stack-develop-templates.md)
* [GitHub 上の Azure Stack Hub クイックスタート テンプレート](https://github.com/Azure/AzureStack-QuickStart-Templates)
