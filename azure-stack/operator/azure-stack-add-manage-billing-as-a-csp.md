---
title: クラウド ソリューション プロバイダーとして Azure Stack Hub の使用状況と課金を管理する
description: Azure Stack Hub をクラウド ソリューション プロバイダー (CSP) として登録し、課金に向けて顧客を追加する方法を説明します。
author: sethmanheim
ms.topic: article
ms.date: 07/27/2020
ms.author: sethm
ms.reviewer: alfredop
ms.lastreviewed: 10/15/2019
ms.openlocfilehash: 03206007c60d438149baad267117e447bd5b0a14
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90573481"
---
# <a name="manage-usage-and-billing-for-azure-stack-hub-as-a-cloud-solution-provider"></a>クラウド ソリューション プロバイダーとして Azure Stack Hub の使用状況と課金を管理する

この記事では、Azure Stack Hub をクラウド ソリューション プロバイダー (CSP) として登録する方法、および顧客を追加する方法について説明します。

CSP として、Azure Stack Hub を使用して、多様な顧客に対応します。 各顧客には Azure に CSP サブスクリプションがあります。 使用状況を Azure Stack Hub から各ユーザー サブスクリプションに転送する必要があります。

次の図では、共有サービス アカウントの選択、および Azure Stack Hub アカウントを使用した Azure アカウントの登録に必要な手順を示しています。 登録されると、エンド カスタマーによる利用を開始できます。

[![クラウド ソリューション プロバイダーとして使用状況と管理を有効にするためのプロセス](media/azure-stack-add-manage-billing-as-a-csp/process-add-useage-as-a-csp.svg "クラウド ソリューション プロバイダーとして使用状況と管理を有効にするためのプロセス")](media/azure-stack-add-manage-billing-as-a-csp/process-add-useage-as-a-csp.svg)

## <a name="create-a-csp-or-apss-subscription"></a>CSP または APSS サブスクリプションを作成する

### <a name="csp-subscription-types"></a>CSP サブスクリプションの種類

Azure Stack Hub で使用する共有サービス アカウントの種類を選択します。 マルチテナント Azure Stack Hub の登録に使用できるサブスクリプションの種類は、次のとおりです。

- クラウド ソリューション プロバイダー
- Partner Shared Services サブスクリプション

マルチテナントの権限を管理する方法を理解するのに役立つチュートリアル ビデオを作成しました。

> [!VIDEO https://www.youtube.com/embed/ZP6jkbLeS34]

#### <a name="azure-partner-shared-services"></a>Azure Partner Shared Services

Azure Partner Shared Services (APSS) サブスクリプションは、直接 CSP、または CSP ディストリビューターが Azure Stack Hub を運用する際の登録時の選択肢として推奨されています。

APSS サブスクリプションは、共有サービス テナントに関連付けられています。 Azure Stack Hub の登録時は、このサブスクリプションの所有者であるアカウントの資格情報を指定します。 Azure Stack Hub の登録に使用するアカウントは、デプロイに使用する管理者アカウントとは別のものにすることができます。 さらに、2 つのアカウントは同じドメインに属している必要はありません。既に使用しているテナントを使用してデプロイできます。 たとえば、`ContosoCSP.onmicrosoft.com` を使用するときに、別のテナント (例: `IURContosoCSP.onmicrosoft.com`) を使用して登録できます。 日常的な Azure Stack Hub 管理を実行するときは、`ContosoCSP.onmicrosoft.com` を使用してサインインする必要があります。 登録操作を実行する必要がある場合は、`IURContosoCSP.onmicrosoft.com` を使用して Azure にサインインします。

APSS サブスクリプションとその作成方法については、「[Add Azure Partner Shared Services (Azure Partner Shared Services の追加)](/partner-center/shared-services)」を参照してください。

#### <a name="csp-subscriptions"></a>CSP サブスクリプション

CSP サブスクリプションは、CSP リセラーまたはエンド カスタマーが Azure Stack Hub を運用する際の登録時の選択肢として推奨されています。

## <a name="register-azure-stack-hub"></a>Azure Stack Hub の登録

前のセクションの情報に従って作成した APSS サブスクリプションを使用して、Azure Stack Hub を Azure に登録します。 詳細については、[Azure サブスクリプションを使用した Azure Stack Hub の登録](azure-stack-registration.md)に関するページを参照してください。

## <a name="add-end-customer"></a>最終顧客を追加する

新しいテナントのリソース使用状況がその CSP サブスクリプションに報告されるように Azure Stack Hub を構成するには、「[Azure Stack Hub に使用量と課金のためのテナントを追加する](azure-stack-csp-howto-register-tenants.md)」を参照してください。

## <a name="charge-the-right-subscriptions"></a>正しいサブスクリプションに課金する

Azure Stack Hub では、"*登録*" と呼ばれる機能を使用します。 登録は、Azure に格納されているオブジェクトです。 特定の Azure Stack Hub への課金のために Azure サブスクリプションで使用される、登録オブジェクト ドキュメントです。 このセクションでは、登録の重要性について説明します。

登録を使用すると、Azure Stack Hub では次のことが可能になります。

- [Azure Stack Hub の使用状況データ](azure-stack-billing-and-chargeback.md)の Azure コマースへの転送、および Azure サブスクリプションへの課金。
- マルチテナント Azure Stack Hub デプロイを使用した、別のサブスクリプションでの各顧客の使用状況の報告。 マルチテナント機能を使用すると、同じ Azure Stack Hub インスタンス上のさまざまな組織を、Azure Stack Hub でサポートすることができるようになります。

Azure Stack Hub ごとに、1 つの既定のサブスクリプションと、複数のテナント サブスクリプションが存在します。 既定のサブスクリプションは、テナント固有のサブスクリプションが存在しない場合に課金される Azure サブスクリプションです。 これは登録される最初のサブスクリプションである必要があります。 マルチテナント使用状況レポートが機能するには、サブスクリプションは CSP または APSS サブスクリプションである必要があります。

その後、登録は、Azure Stack Hub を使用する各テナントの Azure サブスクリプションで更新されます。 テナント サブスクリプションは種類が CSP であり、かつ既定のサブスクリプションを所有するパートナーにロールアップされる必要があります。 別のパートナーの顧客を登録することはできません。

Azure Stack Hub によって使用状況の情報がグローバル Azure に転送されると、Azure 内のサービスによって登録が調べられ、各テナントの使用状況が適切なテナント サブスクリプションにマップされます。 テナントがまだ登録されていない場合、その使用状況は、発信元の Azure Stack Hub インスタンスの既定のサブスクリプションに転送されます。

テナント サブスクリプションは CSP サブスクリプションであるため、その課金は CSP パートナーに送信され、最終顧客には使用状況の情報は表示されません。

## <a name="next-steps"></a>次のステップ

- CSP プログラムについて詳しくは、「[クラウド ソリューション プロバイダー プログラム](https://partner.microsoft.com/solutions/microsoft-cloud-solutions)」をご覧ください。
- Azure Stack Hub からリソース使用量の情報を取得する方法の詳細については、「[Azure Stack Hub での使用量と請求](azure-stack-billing-and-chargeback.md)」を参照してください。
