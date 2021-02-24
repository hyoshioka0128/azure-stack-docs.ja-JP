---
title: Windows Admin Center を使用して Kubernetes クラスターを作成するクイックスタート
description: Windows Admin Center を使用して Kubernetes クラスターを作成する方法について説明します
author: davannaw-msft
ms.topic: quickstart
ms.date: 12/02/2020
ms.author: dawhite
ms.openlocfilehash: 7a9c1bf5d89348175b7ac25a149fe01f384964a2
ms.sourcegitcommit: 0efffe1d04a54062a26d5c6ce31a417f511b9dbf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2020
ms.locfileid: "96612405"
---
# <a name="quickstart-create-a-kubernetes-cluster-on-azure-stack-hci-using-windows-admin-center"></a>クイック スタート:Windows Admin Center を使用して Azure Stack HCI 上に Kubernetes クラスターを作成する

> 適用対象:Azure Stack HCI、Windows Server 2019 Datacenter

Azure Kubernetes Service ホストを設定した後、Windows Admin Center を使用して Kubernetes クラスターを作成できます。 代わりに PowerShell を使用するには、[PowerShell を使用した Kubernetes クラスターの作成](create-kubernetes-cluster-powershell.md)に関するページを参照してください。

Kubernetes クラスターの作成ウィザードに進む前に、[Azure Kubernetes Service を設定](setup.md)していることを確認し、まだ確認していない場合は[システム要件](system-requirements.md)を確認してください。 Kubernetes クラスターの作成ウィザードには、[[すべての接続]](#creating-a-kubernetes-cluster-from-the-all-connections-page) ページまたは [Azure Kubernetes Service ホスト ダッシュボード](#creating-a-kubernetes-cluster-from-the-azure-kubernetes-service-host-dashboard)を使用してアクセスできます。

## <a name="creating-a-kubernetes-cluster-from-the-all-connections-page"></a>[すべての接続] ページでの Kubernetes クラスターの作成 

Windows Admin Center で Kubernetes クラスターを作成するには、次の 2 つの方法があります。 これらのオプションの 1 つは、 **[すべての接続]** ページで行います。 次の手順に従ってから、「[Kubernetes クラスターの作成ウィザード](#the-kubernetes-cluster-create-wizard)」セクションに進みます。 

1. Windows Admin Center で Kubernetes クラスターの作成を開始するには、ゲートウェイ画面で **[追加]** ボタンを押します。 
2. **[Add or create resources]\(リソースの追加または作成\)** パネルの **Kubernetes クラスター (プレビュー)** で、 **[新規作成]** を選択して Kubernetes クラスター ウィザードを起動します。 Kubernetes クラスターの [追加] ボタンはパブリック プレビューに表示されますが、機能しません。 Kubernetes クラスターの作成プロセス中はいつでもウィザードを終了できますが、進行状況は保存されないことに注意してください。 


    ![Windows Admin Center の [Add or create resources]\(リソースの追加または作成\) ブレードを示しています。Kubernetes クラスターの新しいタイルが含まれています。](.\media\create-kubernetes-cluster\add-connection.png)
  
## <a name="creating-a-kubernetes-cluster-from-the-azure-kubernetes-service-host-dashboard"></a>Azure Kubernetes Service ホスト ダッシュボードでの Kubernetes クラスターの作成  

Azure Kubernetes Service ホスト ダッシュボードを使用して Kubernetes クラスターを作成することもできます。 このダッシュボードは、Azure Kubernetes Service ホストを使用しているシステムに接続している場合、Azure Kubernetes Service ツールにあります。 次の手順に従ってから、「[Kubernetes クラスターの作成ウィザード](#the-kubernetes-cluster-create-wizard)」セクションに進みます。 

1. Kubernetes クラスターを作成するシステムに接続し、**Azure Kubernetes Service** ツールに移動します。 このシステムには、Azure Kubernetes Service ホストが既に設定されている必要があります。
2. **[Kubernetes クラスター]** の見出しの下にある **[クラスターの追加]** ボタンを選択します。

![Azure Kubernetes サービス ホストを設定した後に表示される Azure Kubernetes Service ツール ダッシュボードを示します。](.\media\setup\dashboard.png)
  
## <a name="the-kubernetes-cluster-create-wizard"></a>Kubernetes クラスターの作成ウィザード
**[すべての接続]** ページまたは Azure Kubernetes Service ツールを使用して、クラスターの作成ウィザードにアクセスしました。 次の手順で作業を開始します。  

1. Kubernetes クラスターをホストするシステムと Windows Admin Center の前提条件を確認します。 完了したら、 **[次へ]** を選択します。 
2. **[基本]** ページで、Azure Arc 統合、Azure Kubernetes Service ホスト情報、プライマリ ノード プール サイズなど、クラスターに関する情報を構成します。 Azure Kubernetes Service ホストのフィールドには、[設定](setup.md)ページでの作業時に使用した Azure Stack HCI または Windows Server 2019 Datacenter クラスターの完全修飾ドメイン名が必要です。 Azure Kubernetes Service ツールを使用して、このシステムのホスト設定を完了している必要があります。 パブリック プレビュー段階では、ノード数は編集不可であり、既定値は 2 ですが、より大きなワークロードに合わせてノード サイズを構成することができます。 完了したら、 **[次へ]** を選択します。

 [ ![Kubernetes クラスター ウィザードの [基本] ページを示しています。](.\media\create-kubernetes-cluster\basics.png) ](.\media\create-kubernetes-cluster\basics.png#lightbox)
 
3. **[ノード プール]** ページでワークロードを実行する追加のノード プールを構成することができます。 パブリック プレビュー段階では、(システム ノード プールに加えて) 最大 1 つの Windows ノード プールと 1 つの Linux ノード プールを追加できます。 後でこのウィザードを使用して Azure Arc 統合を有効にする場合は、プライマリ ノード プールではない Linux ノード プールが少なくとも 1 つ必要になります。 完了したら、**[次へ]** をクリックします。
4. **[ネットワーク]** ページでネットワーク構成を指定します。 [詳細設定] を選択すると、クラスター内のノードの仮想ネットワークを作成するときに使用するアドレス範囲をカスタマイズできます。 [基本] を選択すると、クラスター内のノードの仮想ネットワークが既定のアドレス範囲で作成されます。 パブリック プレビュー段階では、ネットワーク設定 (ロード バランサー、ネットワーク ポリシー、および HTTP アプリケーション ルーティング) を変更できません。 完了したら、 **[次へ]** を選択します。

    [ ![Kubernetes クラスター ウィザードの [ネットワーク] ページを示しています。](.\media\create-kubernetes-cluster\networking.png) ](\media\create-kubernetes-cluster\networking.png#lightbox)

5. **[統合]** ページで、クラスターを永続ストレージなどの追加サービスに接続します。 このページで、永続ストレージを構成する必要があります。 パブリック プレビュー段階では、永続ストレージの場所は、既定でホストの設定時に選択したストレージの場所になります。 このフィールドは現在編集できません。 完了したら、**[次へ]** をクリックします。
6. **[確認と作成]** ページで選択内容を確認します。 問題がなければ、 **[作成]** を選択してデプロイを開始します。 このページの上部にデプロイの進行状況が表示されます。 
7. デプロイが完了すると、 **[次の手順]** ページにクラスターの管理方法の詳細が表示されます。 このページには、SSH キーも含まれています。 前の手順で Azure Arc 統合を無効にすることを選択した場合、このページの情報と手順の一部が使用できないか、機能しない可能性があります。

> [!IMPORTANT] 
> セキュリティで保護されたアクセス可能な場所に SSH キーを保存することをお勧めします。

## <a name="next-steps"></a>次のステップ

このクイックスタートでは、Kubernetes クラスターをデプロイしました。 Azure Kubernetes Service on Azure Stack HCI の詳細と、AKS on Azure Stack HCI に Linux アプリケーションをデプロイして管理する方法については、次のチュートリアルに進んでください。

- [チュートリアル:Azure Stack HCI 上の Azure Kubernetes Service に Linux アプリケーションをデプロイする](deploy-linux-application.md)
