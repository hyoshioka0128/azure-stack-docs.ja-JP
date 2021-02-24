---
title: Active Directory フェデレーション サービス (AD FS) を使用して Azure Stack Hub に Kubernetes をデプロイする
description: Active Directory フェデレーション サービス (AD FS) を使用して Azure Stack Hub に Kubernetes をデプロイする方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 2/1/2021
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 3/12/2020
ms.openlocfilehash: c04d877009e7deaffa20c9e629062ec3fec517c1
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99247780"
---
# <a name="deploy-kubernetes-to-azure-stack-hub-using-active-directory-federated-services"></a>Active Directory フェデレーション サービスを使用して Azure Stack Hub に Kubernetes をデプロイする

> [!NOTE]  
> Kubernetes Azure Stack Marketplace 項目のみを使用して、概念実証としてクラスターをデプロイします。 Azure Stack でサポートされている Kubernetes クラスターの場合は、[AKS エンジン](azure-stack-kubernetes-aks-engine-overview.md)を使用します。

この記事の手順に従えば、Kubernetes のリソースをデプロイおよび設定することができます。 これらの手順は、Active Directory フェデレーション サービス (AD FS) が ID 管理サービスである場合に使用します。

## <a name="prerequisites"></a>前提条件 

使用を開始するには、次の手順に従って、適切なアクセス許可を保持し、Azure Stack Hub の準備が整っていることを確認します。

1. SSH 公開および秘密キー ペアを生成して Azure Stack Hub 上の Linux VM にサインインします。 クラスターを作成するときは、公開キーが必要です。

    キーを生成する手順については、[SSH キーの生成](azure-stack-dev-start-howto-ssh-public-key.md)に関するページを参照してください。

1. Azure Stack Hub テナント ポータル内で有効なサブスクリプションがあり、新しいアプリケーションの追加に十分使用できるパブリック IP アドレスがあることを確認します。

    クラスターを Azure Stack Hub **管理者** サブスクリプションにデプロイすることはできません。 **User** サブスクリプションを使用する必要があります。 

1. マーケットプレースに Kubernetes クラスターがない場合は、Azure Stack Hub 管理者に連絡してください。

## <a name="create-a-service-principal"></a>サービス プリンシパルの作成

AD FS を ID ソリューションとして使用する場合は、Azure Stack Hub 管理者と協力してサービス プリンシパルを設定する必要があります。 サービス プリンシパルは、アプリケーションに Azure Stack Hub リソースへのアクセス権を付与します。

1. Azure Stack Hub の管理者から、サービス プリンシパルの情報が提供されます。 サービス プリンシパルの情報は次のようになります。

     ```Text  
       ApplicationIdentifier : S-1-5-21-1512385356-3796245103-1243299919-1356
       ClientId              : 3c87e710-9f91-420b-b009-31fa9e430145
       ClientSecret          : <your client secret>
       Thumbprint            : <often this value is empty>
       ApplicationName       : Azurestack-MyApp-c30febe7-1311-4fd8-9077-3d869db28342
       PSComputerName        : 192.168.200.224
       RunspaceId            : a78c76bb-8cae-4db4-a45a-c1420613e01b
     ```

2. 新しいサービス プリンシパルにサブスクリプションの共同作成者としてのロールを割り当てます。 手順については、「[ロールの割り当て](../operator/azure-stack-add-users-adfs.md)」を参照してください。

## <a name="deploy-kubernetes"></a>Kubernetes のデプロイ

1. Azure Stack Hub ポータル `https://portal.local.azurestack.external` を開きます。

1. **[+ リソースの作成]**  >  **[コンピューティング]**  >  **[Kubernetes クラスター]** を選択します。 **［作成］** を選択します

    ![Kubernetes クラスターの作成場所のページを示すスクリーンショット。](media/azure-stack-solution-template-kubernetes-deploy/01_kub_market_item.png)

### <a name="1-basics"></a>1.基本

1. [Kubernetes クラスターを作成] で **[基本]** を選びます。

    ![Kubernetes クラスターに関する基本情報を追加する場所を示すスクリーンショット。](media/azure-stack-solution-template-kubernetes-deploy/02_kub_config_basic.png)

1. **[サブスクリプション]** を選択します。

1. 新しいリソース グループの名前を入力するか、既存のリソース グループを選択します。 リソース名は、英数字かつ小文字にする必要があります。

1. リソース グループの **[場所]** を選択します。 これは、Azure Stack Hub のインストールに向けて選択するリージョンです。

### <a name="2-kubernetes-cluster-settings"></a>2.Kubernetes クラスターの設定

1. [Kubernetes クラスターを作成] で **[Kubernetes Cluster Settings] (Kubernetes クラスターの設定)** を選択します。

    ![Kubernetes クラスター設定の構成手順を示すスクリーンショット。](media/azure-stack-solution-template-kubernetes-deploy/03_kub_config_settings-adfs.png)

1. **[Linux VM admin username]\(Linux VM 管理者ユーザー名\)** を入力します。 Kubernetes クラスターと DVM の一部である Linux Virtual Machines のユーザー名

1. Kubernetes クラスターと DVM の一部として作成されたすべての Linux マシンに対する承認に使用される **[SSH 公開キー]** を入力します。

1. リージョンで一意の **[Master Profile DNS Prefix]\(マスター プロファイル DNS プレフィックス\)** を入力します。 これは、`k8s-12345` など、リージョンで一意の名前になっている必要があります。 ベスト プラクティスとして、リソース グループ名と同じ名前を選択してみてください。

    > [!NOTE]  
    > 各クラスターに対して、新しい一意のマスター プロファイル DNS プレフィックスを使用してください。

1. **[Kubernetes master pool profile count]\(Kubernetes マスター プール プロファイル数\)** を選択します。 この数には、マスター プール内のノードの数が含まれます。 1 ～ 7 を使用できます。 この値は奇数である必要があります。

1. **[The VMSize of the Kubernetes master VMs] (Kubernetes マスター VM の VMSize)** を選択します。

1. **[Kubernetes node pool profile count]\(Kubernetes ノード プール プロファイル数\)** を選択します。 値には、クラスター内のエージェントの数が含まれます。 

1. **[VMSize of the Kubernetes node VMs]\(Kubernetes ノード VM の VM サイズ\)** を選択します。 これにより、Kubernetes ノード VM の VM サイズを指定します。 

1. Azure Stack Hub のインストールに対して、**Azure Stack Hub の ID システム** 用の **ADFS** を選択します。

1. **[Service principal clientId]\(サービス プリンシパル クライアント ID\)** を入力します。これは、Kubernetes Azure クラウド プロバイダーによって使用されます。 Azure Stack Hub 管理者がサービス プリンシパルを作成したときに、アプリケーション ID として識別されたクライアント ID。

1. **[Service principal client secret]\(サービス プリンシパルのクライアント シークレット\)** を入力します。 これは、Azure Stack Hub 管理者から付与された、AD FS サービス プリンシパルのクライアント シークレットです。

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