---
title: Azure Stack Hub でオファーを使用してサブスクリプションを作成する
description: Azure Stack Hub でオファーを使用して新しいサブスクリプションを作成し、テスト VM でオファーをテストする方法について説明します。
author: bryanla
ms.topic: tutorial
ms.date: 06/04/2019
ms.author: bryanla
ms.reviewer: efemmano
ms.lastreviewed: 11/13/2019
ms.openlocfilehash: ef1f26c9929292b235a79037b792c18d84a824b0
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90572563"
---
# <a name="tutorial-create-and-test-a-subscription-in-azure-stack-hub"></a>チュートリアル:Azure Stack Hub でサブスクリプションを作成してテストする

このチュートリアルでは、オファーが含まれているサブスクリプションを作成する方法とそのテスト方法を示します。 テストの場合は、クラウド管理者として Azure Stack Hub ユーザー ポータルにサインインし、オファーにサブスクライブしてから、仮想マシン (VM) を作成します。

> [!TIP]
> より高度な評価エクスペリエンスを使用するには、[特定のユーザーのサブスクリプションを作成](../operator/azure-stack-subscribe-plan-provision-vm.md#create-a-subscription-as-a-cloud-operator)し、ユーザー ポータルでそのユーザーとしてサインインします。

このチュートリアルでは、Azure Stack Hub のオファーにサブスクライブする方法について説明します。

ここでは、次の内容について学習します。

> [!div class="checklist"]
> * プランへのサブスクライブ 
> * オファーのテスト

## <a name="subscribe-to-an-offer"></a>プランへのサブスクライブ

ユーザーとしてオファーにサブスクライブするには、Azure Stack Hub ユーザー ポータルにサインインして、Azure Stack Hub オペレーターから提供される利用可能なサービスを確認します。

1. ユーザー ポータルにサインインし、 **[サブスクリプションの取得]** を選択します。

   ![サブスクリプションの取得](media/azure-stack-subscribe-services/get-subscription.png)

2. **[表示名]** フィールドに、お客様のサブスクリプションの名前を入力します。 次に、 **[Offer]\(オファー\)** を選択し、 **[Choose an offer]\(オファーの選択\)** セクションで利用可能なオファーの 1 つを選択します。 **[作成]** を選択します。

   ![オファーの作成](media/azure-stack-subscribe-services/create-subscription.png)

   > [!TIP]
   > サブスクリプションの使用を開始するには、ユーザー ポータルを更新します。

3. 作成したサブスクリプションを表示するには、 **[すべてのサービス]** を選択します。 その後、 **[一般]** カテゴリで **[サブスクリプション]** を選択し、新しいサブスクリプションを選択します。 オファーにサブスクライブしたら、ポータルを更新して、サービスが新しいサブスクリプションの一部として含まれているかどうかを確認します。 この例では、**仮想マシン**が追加されています。

   ![サブスクリプションの表示](media/azure-stack-subscribe-services/view-subscription.png)

## <a name="test-the-offer"></a>オファーのテスト

ユーザー ポータルにサインインしている間に、新しいサブスクリプション機能を使用して VM をプロビジョニングすることで、オファーをテストします。

> [!NOTE]
> このテストでは、まず Windows Server 2016 Datacenter VM を Azure Stack Hub Marketplace に追加する必要があります。

1. ユーザー ポータルにサインインします。

2. ユーザー ポータルで、 **[仮想マシン]** 、 **[追加]** 、 **[Windows Server 2016 Datacenter]** の順に選択し、 **[作成]** を選択します。

3. **[基本]** セクションで、 **[名前]** 、 **[ユーザー名]** 、 **[パスワード]** を入力し、 **[サブスクリプション]** を選択します。次に、 **[リソース グループ]** を作成し (または既存のものを選択し)、 **[OK]** を選択します。

4. **[サイズの選択]** セクションで、 **[Standard A1]** を選択してから **[選択]** を選びます。  

5. **[設定]** ブレードで、既定値のまま **[OK]** を選択します。

6. **[概要]** セクションで、 **[OK]** を選択して VM を作成します。  

7. 新しい VM を表示するには、 **[仮想マシン]** を選択してから新しい VM を検索し、その名前を選択します。

    ![すべてのリソース](media/azure-stack-subscribe-services/view-vm.png)

> [!NOTE]
> VM のデプロイが完了するまでに数分かかる場合があります。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [コミュニティ テンプレートからの VM の作成](azure-stack-create-vm-template.md)
