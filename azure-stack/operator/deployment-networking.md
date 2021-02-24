---
title: デプロイ ネットワーク トラフィック
titleSuffix: Azure Stack Hub
description: Azure Stack Hub デプロイ中のネットワーク トラフィックについて説明します。
author: PatAltimore
ms.topic: conceptual
ms.date: 04/10/2020
ms.author: patricka
ms.reviewer: wamota
ms.lastreviewed: 12/05/2019
ms.openlocfilehash: 545a81b1bbc34f0e03425e30162ca3fb69270e91
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97868677"
---
# <a name="deployment-network-traffic"></a>デプロイ ネットワーク トラフィック

Azure Stack Hub デプロイ中のネットワーク トラフィックを理解すると、デプロイを成功させるのに役立ちます。 この記事では、デプロイ プロセス中のネットワーク トラフィック フローについて段階的に説明して、何が予想されるかがわかるようにします。

この図には、デプロイ プロセスに関連するすべてのコンポーネントと接続が示されています。

![Azure Stack Hub デプロイ ネットワーク トポロジ](media/deployment-networking/figure1.svg)

> [!NOTE]
> この記事では、接続デプロイの要件について説明します。 他のデプロイ方法については、[Azure Stack Hub デプロイの接続モデル](azure-stack-connection-models.md)に関するページを参照してください。

## <a name="the-deployment-vm"></a>デプロイ VM

Azure Stack Hub ソリューションには、Azure Stack Hub コンポーネントをホストするためのサーバーのグループと、Hardware Lifecycle Host (HLH) という名前の特別なサーバーが含まれています。 このサーバーは、ソリューションをデプロイし、そのライフサイクルを管理するために使用されます。デプロイ中、デプロイ VM (DVM) をホストします。

Azure Stack Hub ソリューション プロバイダーが追加の管理 VM をプロビジョニングする場合があります。 ソリューション プロバイダーからの管理 VM を変更する前に、ソリューション プロバイダーに確認してください。

## <a name="deployment-requirements"></a>デプロイ要件

デプロイを正常に完了するための最小要件があります。デプロイを開始する前に、その要件を OEM で検証できます。

- [証明書](azure-stack-pki-certs.md)。
- [Azure サブスクリプション](azure-stack-validate-registration.md)。 サブスクリプションの確認が必要な場合があります。
- インターネット アクセス。
- DNS。
- NTP。

> [!NOTE]
> この記事では、最後の 3 つの要件について説明します。 最初の 2 つの詳細については、リンク先のページをご覧ください。

## <a name="about-deployment-network-traffic"></a>デプロイ ネットワーク トラフィックについて

DVM は BMC ネットワークからの IP で構成され、インターネットへのネットワーク アクセスを必要とします。 BMC ネットワークのすべてのコンポーネントで外部ルーティングまたはインターネットへのアクセスが必要になるわけではありませんが、このネットワークからの IP を使用する OEM 固有のコンポーネントでも必要になることがあります。

デプロイ中、DVM はお使いのサブスクリプションの Azure アカウントを利用し、Azure Active Directory (Azure AD) に対して認証を行います。 そのために、DVM は一覧にある[特定のポートや URL](azure-stack-integrate-endpoints.md) にインターネット アクセスする必要があります。 DVM は DNS サーバーを利用し、内部コンポーネントによって行われた DNS 要求を外部 URL に転送します。 内部 DNS は転送されてきた要求を DNS フォワーダー アドレスに転送します。このアドレスは、デプロイ前に OEM に指定します。 同じことは NTP サーバーにも当てはまります。すべての Azure Stack Hub コンポーネントで一貫性を維持し、時刻を同期するには、信頼性のあるタイム サーバーが必要です。

デプロイ中、DVM が必要とするインターネット アクセスは外向きのみです。デプロイ中、内向きの呼び出しは行われません。 DVM はその IP をソースとして利用すること、Azure Stack Hub はプロキシ構成に対応していないことにご注意ください。 そのため、必要であれば、インターネットにアクセスするために透過型プロキシまたは NAT を指定する必要があります。 デプロイ時に、一部の内部コンポーネントは、パブリック VIP を使用して外部ネットワーク経由でインターネットにアクセスし始めます。 デプロイの完了後、Azure と Azure Stack Hub の間の全通信はパブリック VIP を利用した外部ネットワーク経由で行われます。

Azure Stack Hub スイッチのネットワーク構成には、特定のネットワーク ソースと接続先の間のトラフィックを制限するアクセス制御リスト (ACL) が含まれます。 DVM は、アクセスに制限がない唯一のコンポーネントです。HLH でさえも制限があります。 ご利用のネットワークからの管理とアクセスを簡単にするためのカスタマイズ オプションについては、OEM にお問い合わせください。 このような ACL が存在するため、デプロイ時には、DNS サーバーと NTP サーバーのアドレスを変更しないことが重要です。 変更する場合、このソリューションのすべてのスイッチを再構成する必要があります。

デプロイの完了後、指定した DNS サーバー アドレスと NTP サーバー アドレスは、システムのコンポーネントによって、外部ネットワークを使用して SDN 経由で引き続き使用されます。 たとえば、デプロイの完了後に DNS 要求を確認する場合、DVM IP からパブリック VIP にソースが変わります。

## <a name="next-steps"></a>次のステップ

[Azure の登録を検証する](azure-stack-validate-registration.md)
