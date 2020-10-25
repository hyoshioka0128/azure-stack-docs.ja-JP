---
title: Azure Active Directory (Azure AD) を使用して Azure Stack Hub に Kubernetes をデプロイする
description: Azure Active Directory (Azure AD) を使用して Azure Stack Hub に Kubernetes をデプロイする方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 07/24/2020
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 3/12/2020
ms.openlocfilehash: 97b2cfc5a595c23bdd12d6ce4519606218bb4687
ms.sourcegitcommit: 53b0dde60a6435936a5e0cb9e931245f262d637a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/23/2020
ms.locfileid: "91106651"
---
# <a name="deploy-kubernetes-to-azure-stack-hub-using-azure-active-directory"></a>Azure Active Directory を使用して Azure Stack Hub に Kubernetes をデプロイする

> [!NOTE]  
> Kubernetes Azure Stack Marketplace 項目のみを使用して、概念実証としてクラスターをデプロイします。 Azure Stack でサポートされている Kubernetes クラスターの場合は、[AKS エンジン](azure-stack-kubernetes-aks-engine-overview.md)を使用します。

この記事の手順に従えば、Azure Active Directory (Azure AD) を ID 管理サービスとして 1 回の連携した操作で使用するときに、Kubernetes のリソースをデプロイおよび設定することができます。

## <a name="prerequisites"></a>前提条件

使用を開始するには、次の手順に従って、適切なアクセス許可があり、Azure Stack Hub の準備が整っていることを確認します。

1. Azure Active Directory (Azure AD) テナントにアプリケーションを作成できることを確認します。 Kubernetes のデプロイには所定のアクセス許可が必要です。

    アクセス許可をチェックする手順については、「[Azure Active Directory のアクセス許可を確認する](/azure/azure-resource-manager/resource-group-create-service-principal-portal)」を参照してください。

1. SSH 公開キーと秘密キーのペアを生成して、Azure Stack Hub 上の Linux VM にサインインします。 クラスターを作成するときに、公開キーが必要になります。

    キーを生成する手順については、[SSH キーの生成](azure-stack-dev-start-howto-ssh-public-key.md)に関するページを参照してください。

1. Azure Stack Hub テナント ポータル内で有効なサブスクリプションがあり、新しいアプリケーションの追加に十分使用できるパブリック IP アドレスがあることを確認します。

    クラスターを Azure Stack Hub **管理者**サブスクリプションにデプロイすることはできません。 **User** サブスクリプションを使用する必要があります。 

1. マーケットプレースに Kubernetes クラスターがない場合は、Azure Stack Hub 管理者に連絡してください。

## <a name="create-a-service-principal"></a>サービス プリンシパルの作成

Azure でサービス プリンシパルを設定します。 サービス プリンシパルは、Azure Stack Hub リソースへのアクセス権をアプリケーションに付与します。

1. グローバルな [Azure Portal](https://portal.azure.com) にサインインします。

1. Azure Stack Hub インスタンスに関連付けられた Azure AD テナントを使ってサインインしたことを確認します。 Azure ツールバーのフィルター アイコンをクリックすると、サインインを切り替えることができます。

    ![AD テナントを選択する](media/azure-stack-solution-template-kubernetes-deploy/tenantselector.png)

1. Azure AD アプリケーションを作成します。

    a. [Azure Portal](https://portal.azure.com) で Azure アカウントにサインインします。  
    b. **[Azure Active Directory]**  >  **[アプリの登録]**  >  **[新規登録]** の順に選択します。  
    c. アプリケーションの名前と URL を指定します。  
    d. **[サポートされているアカウントの種類]** を選択します。  
    e.  アプリケーションの URI の `http://localhost` を追加します。 作成するアプリケーションの種類として、 **[Web]** を選択します。 値を設定したら、 **[登録]** を選択します。

1. **アプリケーション ID** をメモします。 クラスターを作成するときに、ID が必要になります。 ID は、**サービス プリンシパル クライアント ID** として参照されます。

1. サービス プリンシパルのブレードで、 **[新しいクライアント シークレット]** を選択します。 **[設定]**  >  **[キー]** 。 サービス プリンシパルの認証キーを生成する必要があります。

    a. **[説明]** を入力します。

    b. **[有効期限]** で **[Never expires]\(有効期限なし\)** を選択します。

    c. **[追加]** を選択します。 キー文字列をメモします。 クラスターを作成するときに、キー文字列が必要になります。 キーは、**サービス プリンシパル クライアント シークレット**として参照されます。

## <a name="give-the-service-principal-access"></a>サービス プリンシパルへのアクセスの付与

サービス プリンシパルがリソースを作成できるように、そのプリンシパルにサブスクリプションへのアクセスを付与します。

1.  Azure Stack Hub ポータル `https://portal.local.azurestack.external/` にサインインします。

1. **[すべてのサービス]**  >  **[サブスクリプション]** を選択します。

1. オペレーターによって作成された、Kubernetes クラスターを使用するためのサブスクリプションを選択します。

1. **[アクセス制御 (IAM)]** を選択し、 **[ロール割り当ての追加]** を選択します。

1. **[共同作成者]** ロールを選択します。

1. サービス プリンシパル用に作成したアプリケーションの名前を選択します。 必要に応じて、検索ボックスに名前を入力します。

1. **[保存]** をクリックします。

## <a name="deploy-kubernetes"></a>Kubernetes のデプロイ

1. Azure Stack Hub ポータル `https://portal.local.azurestack.external` を開きます。

1. **[+ リソースの作成]**  >  **[コンピューティング]**  >  **[Kubernetes クラスター]** を選択します。 **Create** をクリックしてください。

    ![Kubernetes クラスターの作成方法を示すスクリーンショット。](media/azure-stack-solution-template-kubernetes-deploy/01_kub_market_item.png)

### <a name="1-basics"></a>1.基本

1. [Kubernetes クラスターを作成] で **[基本]** を選びます。

    ![Kubernetes クラスターに関する基本情報を追加する方法を示すスクリーンショット。](media/azure-stack-solution-template-kubernetes-deploy/02_kub_config_basic.png)

1. **[サブスクリプション]** を選択します。

1. 新しいリソース グループの名前を入力するか、既存のリソース グループを選択します。 リソース名は、英数字かつ小文字にする必要があります。

1. リソース グループの **[場所]** を選択します。 これは、Azure Stack Hub のインストールに向けて選択するリージョンです。

### <a name="2-kubernetes-cluster-settings"></a>2.Kubernetes クラスターの設定

1. [Kubernetes クラスターを作成] で **[Kubernetes Cluster Settings] (Kubernetes クラスターの設定)** を選択します。

    ![Kubernetes クラスターの設定に関する情報を指定する場所を示すスクリーンショット。](media/azure-stack-solution-template-kubernetes-deploy/03_kub_config_settings-aad.png)

1. **[Linux VM admin username]\(Linux VM 管理者ユーザー名\)** を入力します。 Kubernetes クラスターと DVM の一部である Linux Virtual Machines のユーザー名

1. Kubernetes クラスターと DVM の一部として作成されたすべての Linux マシンに対する承認に使用される **[SSH 公開キー]** を入力します。

1. リージョンで一意の **[Master Profile DNS Prefix]\(マスター プロファイル DNS プレフィックス\)** を入力します。 これは、`k8s-12345` など、リージョンで一意の名前になっている必要があります。 ベスト プラクティスとして、リソース グループ名と同じ名前を選択してみてください。

    > [!NOTE]  
    > 各クラスターに対して、新しい一意のマスター プロファイル DNS プレフィックスを使用してください。

1. **[Kubernetes master pool profile count]\(Kubernetes マスター プール プロファイル数\)** を選択します。 この数には、マスター プール内のノードの数が含まれます。 1 ～ 7 を使用できます。 この値は奇数である必要があります。

1. **[The VMSize of the Kubernetes master VMs] (Kubernetes マスター VM の VMSize)** を選択します。 これにより、Kubernetes マスター VM の VM サイズを指定します。 

1. **[Kubernetes node pool profile count]\(Kubernetes ノード プール プロファイル数\)** を選択します。 値には、クラスター内のエージェントの数が含まれます。 

1. **[VMSize of the Kubernetes node VMs]\(Kubernetes ノード VM の VM サイズ\)** を選択します。 これにより、Kubernetes ノード VM の VM サイズを指定します。 

1. Azure Stack Hub のインストールに向けて、**Azure Stack Hub の ID システム**用の **Azure AD** を選択します。

1. **[Service principal clientId]\(サービス プリンシパル クライアント ID\)** を入力します。これは、Kubernetes Azure クラウド プロバイダーによって使用されます。 Azure Stack Hub 管理者がサービス プリンシパルを作成したときに、アプリケーション ID として識別されたクライアント ID。

1. **[Service principal client secret]\(サービス プリンシパルのクライアント シークレット\)** を入力します。 これは、サービスの作成時に設定するクライアント シークレットです。

1. **[Kubernetes バージョン]** を入力します。 これは、Kubernetes Azure プロバイダーのバージョンです。 Azure Stack Hub では、各 Azure Stack Hub バージョンに対して、カスタムの Kubernetes ビルドがリリースされます。

### <a name="3-summary"></a>3.まとめ

1. [Summary] (概要) を選択します。 ブレードには、Kubernetes クラスターの構成設定の検証メッセージが表示されます。

    ![ソリューション テンプレートのデプロイ](media/azure-stack-solution-template-kubernetes-deploy/04_preview.png)

2. 設定を確認します。

3. **[OK]** を選択してクラスターをデプロイします。

> [!TIP]  
>  デプロイに関してご質問がある場合は、[Azure Stack Hub フォーラム](https://social.msdn.microsoft.com/Forums/azure/home?forum=azurestack)で質問を投稿するか、他の人が既に回答を受け取っていないかどうかを確認することができます。


## <a name="next-steps"></a>次のステップ

[クラスターへの接続](azure-stack-solution-template-kubernetes-deploy.md#connect-to-your-cluster)

[Kubernetes ダッシュボードの有効化](azure-stack-solution-template-kubernetes-dashboard.md)
