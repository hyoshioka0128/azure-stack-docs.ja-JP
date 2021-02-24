---
title: アドオン プランを作成する
titleSuffix: Azure Stack Hub
description: Azure Stack Hub でアドオン プランを作成する方法について説明します。
author: bryanla
ms.topic: conceptual
ms.date: 10/05/2019
ms.author: bryanla
ms.reviewer: efemmano
ms.lastreviewed: 11/3/2020
ms.openlocfilehash: a3cc9436ec0e3ec0c939760454b194bfec4a8c58
ms.sourcegitcommit: ecd98662194d2cdb15c22f8b1f99812fc5f4c15a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/04/2020
ms.locfileid: "93344883"
---
# <a name="create-add-on-plans-in-azure-stack-hub"></a>Azure Stack Hub でアドオン プランを作成する

基本プランの初期オファーを超えて、" *コンピューター* "、" *ストレージ* "、" *ネットワーク* " のクォータを拡張したり、追加のサービスを提供したりするには、Azure Stack Hub のオペレーターがアドオン プランを作成して [基本プラン](azure-stack-create-plan.md)に変更を加えます。 アドオン プランは、基本プランに変更を加えるものであり、ユーザーが各自のサブスクリプションで任意に有効にできる拡張機能です。

1 つのプランにすべてをまとめることが最適な場合もありますが、 基本プランを用意し、追加のサービスはアドオン プランを使用して提供する方がよい場合もあります。 たとえば、IaaS サービスを基本プランの一部として提供し、すべての PaaS サービスをアドオン プランとして扱うことができます。

アドオン プランを使用するもう 1 つの理由は、リソース使用量の監視に役立つからです。 そのためには、必要なサービスに応じて、比較的小さなクォータを含んだ基本プランから開始します。 ユーザーが上限に達すると、指定されたプランに基づく割り当てリソースを使い切ったという警告が表示されます。 その後、ユーザーは、追加のリソースが用意されているアドオン プランを選択することができます。

> [!NOTE]
> アドオン プランを使ってクォータを拡張することが望ましくない場合には、[クォータの元の構成を編集](azure-stack-quota-types.md#edit-a-quota)することもできます。

アドオン プランは、基本プランと[同じように作成](azure-stack-create-plan.md)されます。 この 2 つの違いは、プランをオファーに追加するときに決定されます。 これは、基本プランとアドオン プランのどちらかとして指定されます。 既存のオファーにアドオン プランを追加すると、サブスクリプションに追加のリソースが表示されるまでに最大で 1 時間かかることがあります。

::: moniker range=">=azs-1902"
## <a name="create-an-add-on-plan-1902-and-later"></a>アドオン プランを作成する (1902 以降)

1. Azure Stack Hub 管理者ポータルにクラウド管理者としてサインインします。
2. [新しい基本プランを作成](azure-stack-create-plan.md)したときと同じ手順で、これまでは提供されていなかったサービスを提供する新しいプランを作成します。
3. 管理者ポータルで **[オファー]** を選択し、アドオン プランで更新するオファーを選択します。

   ![Azure Stack 管理者ポータルで、アドオン プランで更新するオファーを選択する方法を示すスクリーンショット。](media/create-add-on-plan/add-on1.png)

4. オファーのプロパティの一番下で、 **[アドオン プラン]** を選択します。 **[追加]** を選択します。

    ![Azure Stack 管理者ポータルでアドオン プランを選択する方法を示すスクリーンショット。](media/create-add-on-plan/add-on2.png)

5. 追加するプランを選択します。 この例では、プランの名前は **20-storageaccounts** です。 プランの選択後、 **[選択]** をクリックして、オファーにプランを追加します。 プランがオファーに正常に追加されたという通知が届きます。

    ![Azure Stack 管理者ポータルで追加するアドオン プランを選択する方法を示すスクリーンショット。](media/create-add-on-plan/add-on3.png)

6. オファーに含まれているアドオン プランの一覧を見て、新しいアドオン プランが一覧に表示されていることを確認します。

    [![Azure Stack 管理者ポータル内で確認するアドオン プランの一覧を示すスクリーンショット。](media/create-add-on-plan/add-on4.png)](media/create-add-on-plan/add-on4lg.png#lightbox)
::: moniker-end

::: moniker range="<=azs-1901"

## <a name="create-an-add-on-plan-1901-and-earlier"></a>アドオン プランを作成する (1901 以前)

1. Azure Stack Hub 管理者ポータルにクラウド管理者としてサインインします。
2. [新しい基本プランを作成](azure-stack-create-plan.md)したときと同じ手順で、これまでは提供されていなかったサービスを提供する新しいプランを作成します。 この例では、Key Vault ( **Microsoft.KeyVault** ) サービスが新しいプランに含められます。
3. 管理者ポータルで **[オファー]** を選択し、アドオン プランで更新するオファーを選択します。

   ![Azure Stack 管理者ポータルで、アドオン プランで更新するオファーを選択する](media/create-add-on-plan/1.PNG)

4. オファーのプロパティの一番下までスクロールし、 **[アドオン プラン]** を選択します。 **[追加]** を選択します。

    ![Azure Stack 管理者ポータルでアドオン プランを選択する](media/create-add-on-plan/2.PNG)

5. 追加するプランを選択します。 この例では、プランの名前は **[Key vault plan]** です。 プランの選択後、 **[選択]** をクリックして、オファーにプランを追加します。 プランがオファーに正常に追加されたという通知が届きます。

    ![Azure Stack 管理者ポータルで追加するアドオン プランを選択する](media/create-add-on-plan/3.PNG)

6. オファーに含まれているアドオン プランの一覧を見て、新しいアドオン プランが一覧に表示されていることを確認します。

    ![Azure Stack 管理者ポータルでアドオン プランのリストを確認する](media/create-add-on-plan/4.PNG)

::: moniker-end

## <a name="next-steps"></a>次のステップ

* [オファーの作成](azure-stack-create-offer.md)
