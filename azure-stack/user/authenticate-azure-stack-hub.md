---
title: Azure Stack Hub の認証情報を取得する
description: Azure Stack Hub の認証情報を取得する方法について説明します
author: mattbriggs
ms.topic: how-to
ms.date: 12/2/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 12/2/2020
ms.openlocfilehash: 1e762ab25250a6414087c1b22e911bb7a8d151ed
ms.sourcegitcommit: 52c934f5eeb5fcd8e8f2ce3380f9f03443d1e445
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/07/2021
ms.locfileid: "97974112"
---
# <a name="get-authentication-information-for-azure-stack-hub"></a>Azure Stack Hub の認証情報を取得する

Azure Stack Hub に対して認証を行うには、サブスクリプション ID、テナント ID、場所、および Azure Stack Hub Resource Manager エンドポイントを指定する必要があります。 これらの値は、Azure Stack Hub の [Azure Stack Hub Resource Manager エンドポイント](./azure-stack-version-profiles-ruby.md#the-azure-stack-hub-resource-manager-endpoint)から取得できます。 または、この記事の手順を使用して取得することもできます。

## <a name="values-needed-to-authenticate"></a>認証に必要な値

次の情報が必要です。

-   **サブスクリプション ID**  

    サブスクリプション ID は Azure Stack Hub 内のオファーにアクセスするために使用します。

-   **テナント ID**

    ディレクトリとは、ユーザー、アプリケーション、グループ、およびサービス プリンシパルに関する情報を保持するコンテナーのことです。 ディレクトリ テナントとは、Microsoft や自分の会社のなどの組織のことです。

-   **場所**

    場所 (またはリージョン) は、待ち時間で定義された境界内にデプロイされ、待ち時間の短い専用リージョン ネットワークを介して接続される一連のデータセンターです。 Azure Stack Hub では、場所に Azure リージョンではなくローカル データセンターが含まれる場合があります。

-   **Azure Stack Hub Resource Manager エンドポイント**

    Microsoft Azure Resource Manager は、管理者が Azure リソースのデプロイ、管理、監視を行うことができる管理フレームワークです。 Azure Resource Manager では、これらのタスクを個別に処理するのではなく、グループとして単一の操作で処理することができます。

## <a name="get-the-subscription-id"></a>サブスクリプション ID を取得する

サブスクリプション ID を取得するには、次のようにします。

1.  Azure Stack Hub のユーザー ポータルにサインインします。

2.  **[すべてのサービス]** を選択します。

    > ![Azure Stack Hub の認証情報のサブスクリプション ID、テナント ID](./media/authenticate-azure-stack-hub/azure-stack-hub-auth-infoa.png)

3.  **[サブスクリプション]** を選択します。

4.  使用するサブスクリプションを選択します。

5.  **[概要]** から **サブスクリプション ID** をコピーします。

## <a name="get-the-tenant-id"></a>テナント ID を取得する

テナント ID を取得するには、次のようにします。

1.  Azure Stack Hub のユーザー ポータルにサインインします。

2.  ブレードの右上にあるユーザー名の上にマウス ポインターを移動します。

3.  **[ディレクトリ ID]** はテナント ID です。

## <a name="get-the-azure-resource-manager-endpoint"></a>Azure Resource Manager エンドポイントを取得する

Azure Resource Manager エンドポイントは、Azure Stack Hub のデプロイと管理サービスのメタデータ エンドポイントです。 Azure サブスクリプション内のリソースを作成、更新、および削除できる管理レイヤーを提供します。

統合システムの場合、Azure Resource Manager エンドポイントの URL は次のようになります。<br>`https://management.<location>.<fqdn>`

ギャラリー エンドポイント、グラフ エンドポイント、ポータル エンドポイント、サインイン エンドポイント、および対象ユーザーなどのプロパティを指すメタデータ エンドポイントを取得するには、URL は次のようになります。`<ResourceManager>/metadata/endpoints?api-version=1.0`

## <a name="next-steps"></a>次のステップ

Azure Stack Hub での [Azure Stack Resource Manager](./azure-stack-version-profiles.md) の使用について詳細に説明します。
