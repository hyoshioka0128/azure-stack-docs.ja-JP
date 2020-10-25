---
title: 高度な ASDK 評価タスク
description: 高度な Azure Stack Development Kit (ASDK) 評価タスクについて説明します。
author: justinha
ms.topic: article
ms.date: 02/12/2019
ms.author: justinha
ms.reviewer: misainat
ms.lastreviewed: 10/16/2019
ms.openlocfilehash: d8a97ea33b810c258e81ed42ace9a571f5de7bfd
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86489863"
---
# <a name="advanced-asdk-evaluation-tasks"></a>高度な ASDK 評価タスク
基本的な Azure Stack Development Kit (ASDK) サービスの機能について把握したら、より高度なシナリオをテストして Azure Stack の理解を深めることができます。 これらのより高度な評価タスクは、Azure Stack のオペレーター ドキュメントに詳しく記載されています。

> [!NOTE]
> 多くのオペレーター タスクは ASDK デプロイと運用環境のマルチノード Azure Stack デプロイの両方でサポートされますが、ASDK デプロイではすべての使用シナリオがサポートされるわけではありません。 詳しくは、「[ASDK とマルチノード Azure Stack の相違点](asdk-what-is.md#asdk-and-multi-node-azure-stack-hub-differences)」をご覧ください。

## <a name="delegate-offers-in-azure-stack"></a>Azure Stack でのオファーの委任
Azure Stack オペレーターとして、オファー作成とユーザー サインアップを他のユーザーに担当してもらいたいことがよくあります。 たとえば、あなたがサービス プロバイダーで、再販業者に顧客のサインアップと管理を担当してもらいたい場合があります。 あるいは、あなたが社内の中枢の IT グループにいて、子会社で直接、ユーザーのサインアップをして欲しい場合も同様です。

[Azure Stack でのオファーの委任](../operator/azure-stack-delegated-provider.md)はこうしたタスクの手助けをするもので、自分で直接行うよりも多くのユーザーに接触し管理することができます。

## <a name="make-sql-databases-available-to-your-azure-stack-users"></a>SQL データベースを Azure Stack ユーザーから使用可能にする
Azure Stack オペレーターとして、ユーザー (テナント) が自分のクラウド ネイティブなアプリ、Web サイト、およびワークロードで使用できる SQL データベースを作成できるようにするオファーを作成できます。 これらのカスタムの、オンデマンドで、クラウド ベースのデータベースをユーザーに提供することによって、ユーザーの時間とリソースの節約になります。

SQL Server リソースプロバイダー アダプターを使って、[SQL データベースを Azure Stack ユーザーが Azure Stack のサービスとして使用できるようにします](../operator/azure-stack-sql-resource-provider.md?view=azs-2002)。 リソース プロバイダーをインストールした後で、1 つまたは複数の SQL Server インスタンスに接続します。

## <a name="make-web-and-api-apps-available-to-your-azure-stack-users"></a>Web アプリおよび API アプリを Azure Stack ユーザーが使用できるようにする
Azure Stack オペレーターは、ユーザー (テナント) が Azure Functions、Web アプリ、API アプリを作成できるようにするオファーを作成できます。 これらのオンデマンドで、クラウドベースのアプリへのアクセスをユーザーに提供することによって、ユーザーの時間とリソースを節約できます。

App Service リソース プロバイダーをデプロイして、[Web アプリおよび API アプリを Azure Stack ユーザーが使用できるようにします](../operator/azure-stack-app-service-overview.md?view=azs-2002)。

## <a name="next-steps"></a>次のステップ

[Azure Stack 統合システムでのサービスの提供の詳細について学習する](../operator/service-plan-offer-subscription-overview.md)
