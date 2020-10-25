---
title: Azure Stack Hub でのプランの作成
description: サブスクライバーが仮想マシンをプロビジョニングできるプランの Azure Stack Hub での作成方法を説明します。
author: bryanla
ms.topic: conceptual
ms.date: 06/11/2019
ms.author: bryanla
ms.reviewer: efemmano
ms.lastreviewed: 06/11/2019
ms.openlocfilehash: 12de836d74382f26efd49ec8873aaa99a919ecd5
ms.sourcegitcommit: 53b0dde60a6435936a5e0cb9e931245f262d637a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/23/2020
ms.locfileid: "91107092"
---
# <a name="create-a-plan-in-azure-stack-hub"></a>Azure Stack Hub でのプランの作成

[Azure Stack Hub プラン](azure-stack-overview.md)は、1 つまたは複数のサービスとそのクォータをグループ化したものです。 プロバイダーは、ユーザーに提供するプランを作成できます。 次に、ユーザーがオファーをサブスクライブし、そこに含まれるプラン、サービス、クォータを使用します。 この例では、コンピューティング、ネットワーク、およびストレージの各リソース プロバイダーを含むプランを作成する方法を示します。 このプランのサブスクライバーは仮想マシンをプロビジョニングすることができます。

::: moniker range=">=azs-1902"
## <a name="create-a-plan-1902-and-later"></a>プランを作成する (1902 以降)

1. Azure Stack Hub 管理者ポータル `https://adminportal.local.azurestack.external` にサインインします。

2. ユーザーがサブスクライブできるプランやオファーを作成するには、 **[+ リソースの作成]** 、 **[オファー + プラン]** 、 **[プラン]** の順に選択します。
  
   ![Azure Stack Hub 管理者ポータルでプランを選択する方法を示すスクリーンショット。](media/azure-stack-create-plan/select-plan.png)

3. タブ付きのユーザー インターフェイスが表示され、プラン名の指定、サービスの追加、選択した各サービスのクォータの定義を行うことができます。 最も重要なのは、作成を決定する前に、作成するオファーの詳細を確認できることです。

   **[新しいプラン]** ウィンドウの **[基本]** タブで、 **[表示名]** と **[リソース名]** を入力します。 表示名は、オペレーターに表示されるプランのフレンドリ名です。 管理者ポータル内では、プランの詳細はオペレーターにのみ表示されます。

   ![Azure Stack Hub の新しいプランの詳細を指定する方法を示すスクリーンショット。](media/azure-stack-create-plan/plan-name.png)

4. プランのコンテナーとして、新しい**リソース グループ**を作成するか、既存のリソース グループを選択します。

   ![Azure Stack Hub の新しいプランのリソース グループを指定する方法を示すスクリーンショット。](media/azure-stack-create-plan/resource-group.png)

5. **[サービス]** タブを選択するか、または **[次へ: サービス >]** ボタンをクリックしてから、 **[Microsoft.Compute]** 、 **[Microsoft.Network]** 、および **[Microsoft.Storage]** のチェックボックスをオンにします。
  
   ![Azure Stack Hub の新しいプランのサービスを選択する方法を示すスクリーンショット。](media/azure-stack-create-plan/services.png)

6. **[クォータ]** タブを選択するか、または **[次へ: クォータ >]** ボタンをクリックします。 **[Microsoft.Storage]** の横にあるドロップダウン ボックスから既定のクォータを選択するか、 **[新規作成]** を選択してカスタマイズされたクォータを作成します。
  
   ![Azure Stack Hub の新しいプランのクォータを指定する方法を示すスクリーンショット](media/azure-stack-create-plan/quotas.png)

7. 新しいクォータを作成する場合は、クォータの **[名前]** を入力し、クォータ値を指定します。 **[OK]** を選択してクォータを作成します。

   ![Azure Stack Hub の新しいプランの新しいクォータを作成する方法を示すスクリーンショット。](media/azure-stack-create-plan/new-quota.png)

8. 手順 6 と 7 を繰り返して、**Microsoft.Network** と **Microsoft.Compute** のクォータを作成して割り当てます。 3 つのサービスすべてにクォータが割り当てられると、次の例のようになります。

   ![Azure Stack Hub の新しいプランに対するクォータ割り当てを完了する方法を示すスクリーンショット。](media/azure-stack-create-plan/all-quotas-assigned.png)

9. **[確認および作成]** を選択して、プランを確認します。 すべての値とクォータに目を通し、正しいことを確認します。 このインターフェイスでは、選択したプランのクォータを一度に 1 つずつ展開して、プランの各クォータの詳細を表示できます。 また、戻って必要な編集を加えることもできます。

   ![Azure Stack Hub のプランを作成する方法を示すスクリーンショット。](media/azure-stack-create-plan/create.png)

10. 準備ができたら、 **[作成]** を選択してプランを作成します。

11. 新しいプランを表示するには、左側で **[すべてのサービス]** をクリックし、 **[プラン]** を選択してから、プランを検索して、その名前を選択します。 リソースの一覧が長い場合は、 **[検索]** を使用して名前でプランを探すことができます。
::: moniker-end

::: moniker range="<=azs-1901"
## <a name="create-a-plan-1901-and-earlier"></a>プランを作成する (1901 以前)

1. Azure Stack Hub 管理者ポータル `https://adminportal.local.azurestack.external` にサインインします。

2. ユーザーがサブスクライブできるプランやオファーを作成するには、 **[+ 新規]** 、 **[オファー + プラン]** 、 **[プラン]** の順に選択します。
  
   ![Azure Stack Hub 管理者ポータルでプランを選択する](media/azure-stack-create-plan/select-plan1901.png)

3. **[新しいプラン]** で、 **[表示名]** と **[リソース名]** を入力します。 表示名は、ユーザーに表示されるプランのフレンドリ名です。 リソース名は管理者にのみ表示されます。リソース名は、Azure Resource Manager リソースとしてプランを操作するために管理者が使用します。

   ![Azure Stack Hub の新しいプランの詳細を指定する](media/azure-stack-create-plan/plan-name1901.png)

4. プランのコンテナーとして、新しい**リソース グループ**を作成するか、既存のリソース グループを選択します。

   ![Azure Stack Hub の新しいプランのリソース グループを指定する](media/azure-stack-create-plan/resource-group1901.png)

5. **[サービス]** を選択し、 **[Microsoft.Compute]** 、 **[Microsoft.Network]** 、および **[Microsoft.Storage]** のチェックボックスをオンにします。 次に、 **[選択]** を選択して構成を保存します。 マウスで各オプションをポイントすると、チェックボックスが表示されます。
  
   ![Azure Stack Hub の新しいプランのサービスを指定する](media/azure-stack-create-plan/services1901.png)

6. **[クォータ]** 、 **[Microsoft.Storage (ローカル)]** の順に選択し、既定のクォータを選択するか **[新しいクォータの作成]** を選択してカスタマイズされたクォータを作成します。
  
   ![Azure Stack Hub の新しいプランのクォータを指定する](media/azure-stack-create-plan/quotas1901.png)

7. 新しいクォータを作成する場合は、クォータの **[名前]** を入力し、クォータ値を指定して **[OK]** を選択します。 **[クォータの作成]** ダイアログが閉じられます。

   ![Azure Stack Hub の新しいプランの新しいクォータを作成する](media/azure-stack-create-plan/new-quota1901.png)

   次に、作成した新しいクォータを選択します。 クォータを選択するとそれが割り当てられ、選択ダイアログが閉じられます。
  
   ![Azure Stack Hub の新しいプランのクォータを割り当てる](media/azure-stack-create-plan/assign-quota1901.png)

8. 手順 6 と 7 を繰り返して、**Microsoft.Network (ローカル)** と **Microsoft.Compute (ローカル)** のクォータを作成して割り当てます。 3 つのサービスすべてにクォータが割り当てられると、次の例のようになります。

   ![Azure Stack Hub の新しいプランに対するクォータ割り当てを完了する](media/azure-stack-create-plan/all-quotas-assigned1901.png)

9. **[クォータ]** で **[OK]** を選択し、 **[新しいプラン]** で **[作成]** を選択してプランを作成します。

    ![Azure Stack Hub のプランを作成する](media/azure-stack-create-plan/create1901.png)

10. 新しいプランを表示するには、 **[すべてのリソース]** を選択してプランを検索し、プラン名を選択します。 リソースの一覧が長い場合は、 **[検索]** を使用して名前でプランを探すことができます。

    ![Azure Stack Hub の新しいプランを確認する](media/azure-stack-create-plan/plan-overview1901.png)
::: moniker-end

## <a name="next-steps"></a>次のステップ

* [オファーの作成](azure-stack-create-offer.md)
