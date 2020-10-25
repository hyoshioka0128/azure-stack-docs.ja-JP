---
title: ユーザー用にサービス オファリングを作成する
titleSuffix: Azure Stack Hub
description: Azure Stack Hub 内でオファー、プラン、サービスを使用して、サービス オファリングを作成する方法について説明します。
author: BryanLa
ms.author: bryanla
ms.topic: tutorial
ms.date: 10/16/2019
ms.reviewer: shriramnat
ms.lastreviewed: 10/16/2019
ms.openlocfilehash: f2716bfbdab58c90401e3c6b10908c35f774feed
ms.sourcegitcommit: 53b0dde60a6435936a5e0cb9e931245f262d637a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/23/2020
ms.locfileid: "91107188"
---
# <a name="create-a-service-offering-for-users-in-azure-stack-hub"></a>Azure Stack Hub でユーザー用にサービス オファリングを作成する

このチュートリアルでは、オペレーターにオファーを作成する方法を示します。 オファーを使用すると、サブスクリプションに基づいてユーザーがサービスを利用できるようになります。 オファーをサブスクライブすると、ユーザーは、オファーで指定されたサービス内でリソースを作成してデプロイすることができます。

このチュートリアルでは、以下の内容を学習します。

> [!div class="checklist"]
> * オファーを作成します。
> * プランを作成します。
> * プランにサービスとクォータを割り当てます。
> * オファーにプランを割り当てます。

## <a name="overview"></a>概要

オファーは、1 つ以上のプランで構成されます。 プランでは、各サービスの対応するリソース プロバイダーとクォータが指定されており、1 つ以上のサービスにアクセスできます。 プランは、基本プランとしてオファーに追加することも、アドオン プランとしてオファーを拡張することもできます。 詳細については、「[サービス、プラン、オファー、サブスクリプションの概要](service-plan-offer-subscription-overview.md)」を参照してください。

![Azure Stack Hub でのサブスクリプション、オファー、プラン](media/azure-stack-key-features/image4.png)

### <a name="resource-providers"></a>リソース プロバイダー

リソース プロバイダーでは、サービスとしてのリソースの作成、デプロイ、管理がサポートされます。 一般的な例として、Microsoft.Compute リソース プロバイダーでは仮想マシン (VM) を作成してデプロイする機能が提供されます。 Azure のリソース管理モデルの概要については、[Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview) に関する記事を参照してください。

Azure Stack Hub では、リソース プロバイダーに 2 つの一般的なカテゴリがあります。リソースを基本サービスとしてデプロイするものと、付加価値サービスとしてデプロイするものです。

### <a name="foundational-services"></a>基礎となるサービス

>[!NOTE]
> このチュートリアルでは、基本サービスに基づいてオファーを作成する方法について説明します。 

基本サービスは次のリソース プロバイダーによってサポートされており、これらは Azure Stack Hub のすべてのインストールでネイティブに使用できます。

| リソース プロバイダー | リソースの例 |
| ----------------- | ------------------|
| Microsoft.Compute | VM、ディスク、仮想マシン スケール セット |
| Microsoft.KeyVault | キー コンテナー、シークレット |
| Microsoft.Network | 仮想ネットワーク、パブリック IP アドレス、ロード バランサー |
| Microsoft.Storage | ストレージ アカウント、BLOB、キュー、テーブル |

### <a name="value-add-services"></a>付加価値サービス

>[!NOTE]
> 付加価値サービスを提供するには、最初に、対応するリソース プロバイダーを Azure Stack Hub Marketplace にインストールする必要があります。 インストールが済むと、そのリソースが基本サービスと同じ方法でユーザーに提供されます。 付加価値サービス オファリングがサポートされているリソース プロバイダーの最新のセットについては、目次の「**ハウツー ガイド**」セクションを参照してください。

付加価値サービスは、Azure Stack Hub がデプロイされた後でインストールされたリソース プロバイダーによってサポートされます。 たとえば、次のようになります。

| リソース プロバイダー | リソースの例 |
| ----------------- | ------------------------- |
| Microsoft.Web | App Service の関数アプリ、Web アプリ、API アプリ | 
| Microsoft.MySqlAdapter | MySQL ホスティング サーバー、MySQL データベース | 
| Microsoft.SqlAdapter | SQL Server ホスティング サーバー、SQL Server データベース |
| Microsoft.EventHub | Event Hubs |

::: moniker range=">=azs-1902"

## <a name="create-an-offer"></a>オファーの作成

オファー作成プロセスでは、オファーとプランの両方を作成します。 プランは、オファーの基本プランとして使用されます。 プラン作成の間に、プランで使用できるようにするサービスと、それぞれのクォータを指定します。

1. クラウド管理者アカウントを使用して管理者ポータルにサインインします。

    - 統合システムの場合、URL はオペレーターのリージョンと外部ドメイン名によって異なります。 URL は形式 `https://adminportal.<region>.<FQDN>` を使用します。
    - Azure Stack Development Kit を使用している場合は、URL は `https://adminportal.local.azurestack.external` です。

    続いて、 **[+ リソースの作成]** > **[オファー + プラン]** > **[オファー]** を選択します。

    ![Azure Stack Hub 管理ポータル内で新しいオファーを作成する方法を示すスクリーンショット。](media/tutorial-offer-services/1-create-resource-offer.png)

1. **[Create a new offer]\(新しいオファーの作成\)** の **[基本]** タブで、 **[表示名]** と **[リソース名]** を入力し、既存の **[リソース グループ]** を選択するか、新しいリソース グループを作成ます。 表示名は、オファーのフレンドリ名です。 クラウド オペレーターのみがリソース名を見ることができます。リソース名は、Azure Resource Manager リソースとしてオファーを操作するために、管理者が使用する名前です。

   ![Azure Stack Hub 管理ポータル内で基本情報を追加する場所を示すスクリーンショット。](media/tutorial-offer-services/2-create-new-offer.png)

1. **[基本プラン]** タブを選択し、 **[新しいプランの作成]** を選択して新しいプランを作成します。 プランは、基本プランとしてオファーにも追加されます。

   ![Azure Stack Hub 管理ポータル内でプランを追加する方法を示すスクリーンショット。](media/tutorial-offer-services/3-create-new-offer-base-plans.png)

1. **[新しいプラン]** の **[基本]** タブで、 **[表示名]** と **[リソース名]** を入力します。 表示名は、ユーザーに表示されるプランのフレンドリ名です。 クラウド オペレーターのみがリソース名を見ることができます。リソース名は、Azure Resource Manager リソースとしてプランを操作するために、クラウド オペレーターが使用する名前です。 **[リソース グループ]** は、オファーに対して指定されているものに設定されます。

   ![Azure Stack Hub 管理ポータル内のプランの表示名を示すスクリーンショット。](media/tutorial-offer-services/4-create-new-plan-basics.png)

1. **[サービス]** タブを選択すると、インストールされているリソース プロバイダーから利用可能なサービスの一覧が表示されます。 **Microsoft.Compute**、**Microsoft.Network**、**Microsoft.Storage** を選択します。

   ![Azure Stack Hub 管理ポータル内のプラン サービスを示すスクリーンショット。](media/tutorial-offer-services/5-create-new-plan-services.png)

1. **[クォータ]** タブを選択すると、このプランで有効にしたサービスの一覧が表示されます。 **[新規作成]** を選択して、**Microsoft. Compute** に対するカスタム クォータを指定します。 クォータの **[名前]** は必須です。各クォータ値は、そのままにすることも、変更することもできます。 終わったら **[OK]** を選択し、残りのサービスについても同じ手順を繰り返します。

   ![Azure Stack Hub 管理ポータル内でのコンピューティング クォータの作成](media/tutorial-offer-services/6-create-new-plan-quotas.png)

1. **[確認および作成]** タブを選択します。新しい基本プランを作成する準備ができたことを示す緑色の [検証に成功しました] バナーが、上部に表示されます。 **［作成］** を選択します また、プランが作成されたことを示す通知も表示されます。

   ![Azure Stack Hub 管理ポータル内での新しいプランの作成](media/tutorial-offer-services/7-create-new-plan-review-create.png)

1. **[Create a new offer]\(新しいオファーの作成\)** ページの **[基本プラン]** タブに戻ると、プランが作成されていることがわかります。 新しいプランを基本プランとしてオファーに含めることが選択されているのを確認し、 **[レビュー + 作成]** を選択します。

   ![Azure Stack Hub 管理ポータル内での基本プランの追加](media/tutorial-offer-services/8-create-new-offer-base-plans-done.png)

1. **[レビュー + 作成]** タブの上部に、緑色の [検証に成功しました] バナーが表示されます。 [基本] と [基本プラン] の情報を確認し、準備ができたら **[作成]** を選択します。

   ![Azure Stack Hub 管理ポータル内での新しいオファーの作成](media/tutorial-offer-services/9-create-new-offer-review-create.png)

1. 最初に [デプロイが進行中です] ページが表示された後、オファーがデプロイされると、[デプロイが完了しました] と表示されます。 **[リソース]** 列の下でオファーの名前を選択します。

   ![Azure Stack Hub 管理ポータルでのオファー デプロイ完了](media/tutorial-offer-services/10-offer-deployment-complete.png)

1. オファーがまだプライベートであることを示しているバナーに注意してください。そのため、ユーザーはオファーをサブスクライブできません。 **[状態の変更]** を選択して **[パブリック]** を選択することにより、それをパブリックに変更します。

    ![Azure Stack Hub 管理ポータル内の [パブリック] の状態](media/tutorial-offer-services/11-offer-change-state.png)

::: moniker-end

::: moniker range="<=azs-1901"

## <a name="create-an-offer-1901-and-earlier"></a>オファーの作成 (1901 以前)

オファー作成プロセスでは、オファーとプランの両方を作成します。 プランは、オファーの基本プランとして使用されます。 プラン作成の間に、プランで使用できるようにするサービスと、それぞれのクォータを指定します。

1. クラウド管理者アカウントを使用して管理者ポータルにサインインします。

    - 統合システムの場合、URL はオペレーターのリージョンと外部ドメイン名によって異なり、`https://adminportal.<region>.<FQDN>` という形式です。
    - Azure Stack Development Kit を使用している場合は、URL は `https://adminportal.local.azurestack.external` です。

    続いて、 **[+ リソースの作成]** > **[オファー + プラン]** > **[オファー]** を選択します。

    ![Azure Stack Hub 管理ポータル内の新しいオファー](media/tutorial-offer-services/image01.png)

1. **[新しいオファー]** で、 **[表示名]** と **[リソース名]** を入力し、新規または既存の **[リソース グループ]** を選択します。 表示名は、オファーのフレンドリ名です。 クラウド オペレーターのみがリソース名を見ることができます。リソース名は、Azure Resource Manager リソースとしてオファーを操作するために、管理者が使用する名前です。

   ![Azure Stack Hub 管理ポータル内の表示名](media/tutorial-offer-services/image02.png)

1. **[基本プラン]** を選択し、 **[プラン]** セクションで **[追加]** を選択して新しいプランをオファーに追加します。

   ![Azure Stack Hub 管理ポータルでのプランの追加](media/tutorial-offer-services/image03.png)

1. **[新しいプラン]** セクションで **[表示名]** と **[リソース名]** を入力します。 表示名は、ユーザーに表示されるプランのフレンドリ名です。 クラウド オペレーターのみがリソース名を見ることができます。リソース名は、Azure Resource Manager リソースとしてプランを操作するために、クラウド オペレーターが使用する名前です。

   ![Azure Stack Hub 管理ポータル内のプランの表示名](media/tutorial-offer-services/image04.png)

1. **[サービス]** を選択します。 サービスの一覧から、 **[Microsoft.Compute]** 、 **[Microsoft.Network]** 、および **[Microsoft.Storage]** を選びます。 **[選択]** を選択してそれらサービスをプランに追加します。

   ![Azure Stack Hub 管理ポータル内のプランのサービス](media/tutorial-offer-services/image05.png)

1. **[クォータ]** を選択してから、クォータを作成する最初のサービスを選択します。 IaaS クォータには、次の例をコンピューティング サービス、ネットワーク サービス、ストレージ サービスのそれぞれにクォータを構成するガイドとして使用します。

    - まず、コンピューティング サービスのクォータを作成します。 名前空間の一覧で、 **[Microsoft.Compute]** を選択し、 **[新しいクォータの作成]** を選択します。
    
      ![新しいクォータの作成](media/tutorial-offer-services/image06.png)

   - **[クォータの作成]** で、クォータの名前を入力します。 表示されているクォータのどの値も、変更することもそのまま使用することもできます。 この例では、既定の設定をそのまま使用し、 **[OK]** を選択します。
   
      ![クォータ名](media/tutorial-offer-services/image07.png)
       
    - 名前空間の一覧から **[Microsoft.Compute]** を選び、作成したクォータを選択します。 このステップでは、クォータがコンピューティング サービスにリンクされます。
    
      ![クォータの選択](media/tutorial-offer-services/image08.png)
        
        ネットワーク サービスとストレージ サービスでこれらの手順を繰り返します。 完了したら、 **[クォータ]** で **[OK]** を選択してすべてのクォータを保存します。

1. **[新しいプラン]** で、 **[OK]** を選択します。

1. **[プラン]** で、新しいプランを選択し、 **[選択]** を選択します。

1. **[新しいオファー]** で、 **[作成]** を選択します。 オファーが作成されると通知が表示されます。

1. ダッシュボード メニューで、 **[オファー]** を選択し、作成したオファーをクリックします。

1. **[状態の変更]** を選択し、 **[パブリック]** を選択します。

    ![[パブリック] の状態](media/tutorial-offer-services/image09.png)

::: moniker-end
 
## <a name="next-steps"></a>次のステップ

このチュートリアルで学習した内容は次のとおりです。

> [!div class="checklist"]
> * オファーを作成します。
> * プランを作成します。
> * プランにサービスとクォータを割り当てます。
> * オファーにプランを割り当てます。

次のチュートリアルに進み、次の操作方法を確認してください。
> [!div class="nextstepaction"]
> [このチュートリアルで提供されているサービスをテストする](tutorial-test-offer.md)
