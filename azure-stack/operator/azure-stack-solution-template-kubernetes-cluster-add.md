---
title: Kubernetes を Azure Stack Hub Marketplace に追加する
titleSuffix: Azure Stack Hub
description: Kubernetes を Azure Stack Hub Marketplace に追加する方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 07/24/2020
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 03/24/2020v
ms.openlocfilehash: e2edda983e6fd554b2ff76a526b476318b31a6f5
ms.sourcegitcommit: b2337a9309c52aac9f5a1ffd89f1426d6c178ad5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/28/2020
ms.locfileid: "87250182"
---
# <a name="add-kubernetes-to-azure-stack-hub-marketplace"></a>Kubernetes を Azure Stack Hub Marketplace に追加する

> [!note]  
> Kubernetes Azure Stack Hub Marketplace 項目のみを使用して、クラスターを概念実証としてデプロイします。 Azure Stack Hub でサポートされている Kubernetes クラスターの場合は、[AKS エンジン](azure-stack-aks-engine.md)を使用します。

Kubernetes は、マーケットプレース項目としてユーザーに提供できます。 ユーザーはその後、1 回の連携した操作で Kubernetes をデプロイできます。

この記事では、Azure Resource Manager テンプレートを使用して、スタンドアロンの Kubernetes クラスターのためにリソースのデプロイとプロビジョニングを行う方法を示しています。 開始する前に、Azure Stack Hub とグローバル Azure テナントの設定を確認します。 Azure Stack Hub ついて必要な情報を収集します。 テナントと Azure Stack Hub Marketplace に必要なリソースを追加します。 このクラスターは、Ubuntu サーバー、カスタム スクリプト、および Azure Stack Hub Marketplace 内に置かれる Kubernetes Cluster マーケットプレース項目に依存します。

## <a name="create-a-plan-an-offer-and-a-subscription"></a>プラン、オファー、サブスクリプションを作成する

Kubernetes のマーケットプレース項目のプラン、オファー、サブスクリプションを作成します。 既存のプランとオファーを使うこともできます。

1. 管理者ポータル `https://adminportal.local.azurestack.external` にサインインします。

1. プランは基本プランとして作成します。 手順については、「[Azure Stack Hub でのプランの作成](azure-stack-create-plan.md)」をご覧ください。

1. オファーを作成します。 手順については、「[Azure Stack Hub でのオファーの作成](azure-stack-create-offer.md)」をご覧ください。

1. **[オファー]** を選択し、作成したプランを見つけます。

1. [オファー] ブレードで **[概要]** を選択します。

1. **[状態の変更]** を選択します。 **[パブリック]** を選択します。

1. **[+ リソースの作成]**  >  **[Offers and Plans] (オファーとプラン)**  >  **[サブスクリプション]** と選択し、サブスクリプションを作成します。

    a. **表示名**を入力します。

    b. **ユーザー**を入力します。 テナントに関連付けられている Azure AD アカウントを使用します。

    c. **[プロバイダーの説明]**

    d. **[ディレクトリ テナント]** を Azure Stack Hub の Azure AD テナントに設定します。 

    e. **[オファー]** を選択します。 作成したオファーの名前を選択します。 サブスクリプション ID をメモします。

## <a name="create-a-service-principal-and-credentials-in-ad-fs"></a>AD FS でサービス プリンシパルと資格情報を作成する

ID 管理サービスのために Active Directory Federated Services (AD FS) を使用する場合は、Kubernetes クラスターを展開するユーザーのサービス プリンシパルを作成する必要があります。 クライアント シークレットを使用してサービス プリンシパルを作成します。 手順については、[クライアント シークレットを使用したサービス プリンシパルの作成](azure-stack-create-service-principals.md#create-a-service-principal-that-uses-client-secret-credentials)に関するセクションを参照してください。

## <a name="add-an-ubuntu-server-image"></a>Ubuntu サーバーのイメージを追加する

次の Ubuntu Server イメージを Azure Stack Hub Marketplace に追加します。

1. 管理者ポータル `https://adminportal.local.azurestack.external` にサインインします。

1. **[すべてのサービス]** を選択し、 **[管理]** カテゴリで **[Marketplace management] (Marketplace 管理)** を選択します。

1. **+ Add from Azure**(+ Azure から追加) を選択します。

1. 「`Ubuntu Server`」と入力します。

1. サーバーの最新バージョンを選択します。 通常版を確認して、最新バージョンがインストールされていることを確認してください。
    - **[発行者]** : Canonical
    - **[プラン]** : UbuntuServer
    - **バージョン**:16.04.201806120 (または最新バージョン)
    - **SKU**:16.04 LTS

1. **[ダウンロード]** を選択します。

## <a name="add-a-custom-script-for-linux"></a>Linux 用のカスタム スクリプトを追加する

Azure Stack Hub Marketplace から Kubernetes を追加します。

1. 管理者ポータル `https://adminportal.local.azurestack.external` を開きます。

1. **[すべてのサービス]** を選択し、 **[管理]** カテゴリで **[Marketplace management] (Marketplace 管理)** を選択します。

1. **+ Add from Azure**(+ Azure から追加) を選択します。

1. 「`Custom Script for Linux`」と入力します。

1. 次のプロファイルを持つスクリプトを選択します。
   - **[プラン]** : Linux 2.0 用のカスタム スクリプト
   - **バージョン**:2.0.6 (または最新バージョン)
   - **[発行者]** : Microsoft Corp

     > [!Note]  
     > 複数バージョンの Linux 用カスタム スクリプトが表示されることがあります。 最新バージョンの項目を追加する必要があります。

1. **[ダウンロード]** を選択します。

## <a name="add-kubernetes-to-the-marketplace"></a>Kubernetes をマーケットプレースに追加する

1. 管理者ポータル `https://adminportal.local.azurestack.external` を開きます。

1. **[すべてのサービス]** を選択し、 **[管理]** カテゴリで **[Marketplace management] (Marketplace 管理)** を選択します。

1. **+ Add from Azure**(+ Azure から追加) を選択します。

1. 「`Kubernetes`」と入力します。

1. [`Kubernetes Cluster`] を選択します。

1. **[ダウンロード]** を選択します。

    > [!note]  
    > マーケットプレース項目が Azure Stack Hub Marketplace に表示されるまで 5 分かかる場合があります。

    ![Azure Stack Hub Marketplace 内の Kubernetes 項目](../user/media/azure-stack-solution-template-kubernetes-deploy/marketplaceitem.png)

## <a name="update-or-remove-the-kubernetes"></a>Kubernetes を更新または削除する

Kubernetes 項目を更新する際、Azure Stack Hub Marketplace 内の以前の項目を削除します。 以下の指示に従って、Kubernetes 更新プログラムを Azure Stack Hub Marketplace に追加します。

Kubernetes を削除するには:

1. オペレーターとして、PowerShell を使用して Azure Stack Hub に接続します。 手順については、[オペレーターとして PowerShell を使用した Azure Stack Hub への接続](azure-stack-powershell-configure-admin.md)に関する記事を参照してください。

2. ギャラリーで、Kubernetes クラスターの現在の項目を検索します。

    ```powershell  
    Get-AzsGalleryItem | Select Name
    ```
    
3. `Microsoft.AzureStackKubernetesCluster.0.3.0` などの現在の項目の名前に注意してください。

4. 次の PowerShell コマンドレットを使用して項目を削除します。

    ```powershell  
    $Itemname="Microsoft.AzureStackKubernetesCluster.0.3.0"

    Remove-AzsGalleryItem -Name $Itemname
    ```

## <a name="next-steps"></a>次のステップ

[Kubernetes を Azure Stack Hub にデプロイする](../user/azure-stack-solution-template-kubernetes-deploy.md)

[Azure Stack Hub でのサービスの提供の概要](service-plan-offer-subscription-overview.md)
