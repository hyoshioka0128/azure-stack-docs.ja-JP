---
title: クラウド ソリューション プロバイダーで Azure Stack Hub サブスクリプションを管理できるようにする
description: クラウド ソリューション プロバイダー (CSP) で Azure Stack Hub サブスクリプションを自動的に管理できるようにする方法について学習します。
author: sethmanheim
ms.topic: article
ms.date: 07/27/2020
ms.author: sethm
ms.reviewer: alfredop
ms.lastreviewed: 05/20/2019
ms.openlocfilehash: fd0caa7e841b960f4bdab6d5712fd3563710ef20
ms.sourcegitcommit: b2337a9309c52aac9f5a1ffd89f1426d6c178ad5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/28/2020
ms.locfileid: "87251049"
---
# <a name="let-your-cloud-solution-provider-manage-your-azure-stack-hub-subscription"></a>クラウド ソリューション プロバイダーで Azure Stack Hub サブスクリプションを管理できるようにする

クラウド ソリューション プロバイダー (CSP) で Azure Stack Hub を使用している場合は、Azure および Azure Stack Hub 内のリソースにアクセスするために独自のサブスクリプションを管理するように選択できます。 プロバイダーに自分のサブスクリプションの管理を委託することもできます。 この記事で取り上げるテクニック:

* 自分のサブスクリプションへのアクセス権をサービス プロバイダーに付与する。
* サービス プロバイダーが自分のサービスを管理できることを確認する。

> [!NOTE]
> 自分のアカウントが CSP で管理されておらず、次の手順をスキップした場合、CSP でご利用の Azure Stack Hub サブスクリプションを管理することはできません。

## <a name="manage-your-subscription-with-a-csp"></a>CSP でサブスクリプションを管理する

CSP を**ユーザー**としてサブスクリプションに追加します。

1. CSP を、**ユーザー** ロールを持つゲスト ユーザーとしてテナント ディレクトリに追加します。 ユーザーの追加に関するヘルプについては、[Azure Active Directory への新しいユーザーの追加](/azure/active-directory/add-users-azure-active-directory)に関するページを参照してください。

2. CSP では、ローカル Azure Stack Hub サブスクリプションが自動的に作成されます。 これで、Azure Stack Hub の使用を開始する準備ができました。

3. CSP は、サブスクリプション内にリソースを作成して、その CSP もリソースを管理できることを確認する必要があります。 たとえば、それらは [Azure Stack Hub ポータルで Windows 仮想マシンを作成する](azure-stack-quick-windows-portal.md)ことができます。

## <a name="let-the-csp-manage-your-subscription-using-rbac-rights"></a>CSP で RBAC 権限を使用してサブスクリプションを管理できるようにする

CSP を**所有者**としてサブスクリプションに追加します。

1. CSP を、ゲスト ユーザーとしてテナント ディレクトリに追加します。 ユーザーの追加の詳細については、「[Azure Active Directory に新しいユーザーを追加する](/azure/active-directory/add-users-azure-active-directory)」を参照してください。

2. CSP ゲスト ユーザーに**所有者**ロールを追加します。 サブスクリプションへの CSP ユーザーの追加に関する詳細については、[ロールベースのアクセス制御を使用した Azure サブスクリプション リソースへのアクセスの管理](/azure/role-based-access-control/role-assignments-portal)に関するページを参照してください。 CSP では、ローカル Azure Stack Hub サブスクリプションが自動的に作成されます。 これで、Azure Stack Hub の使用を開始する準備ができました。
3. CSP は、サブスクリプション内にリソースを作成して、その CSP がリソースを管理できることを確認する必要があります。

## <a name="next-steps"></a>次のステップ

* Azure Stack Hub からリソース使用量情報を取得する方法の詳細については、「[Azure Stack Hub での使用量と請求](../operator/azure-stack-billing-and-chargeback.md)」をご覧ください。
