---
title: Windows Admin Center を使用して Kubernetes クラスターを作成するクイックスタート
description: Windows Admin Center を使用して Kubernetes クラスターを作成する方法について説明します
author: davannaw-msft
ms.topic: quickstart
ms.date: 09/22/2020
ms.author: dawhite
ms.openlocfilehash: 7ef2f0e0532ee342e8821b362b16433f755bc072
ms.sourcegitcommit: a1e2003fb9c6dacdc76f97614ff5a26a5b197b49
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/01/2020
ms.locfileid: "91623270"
---
# <a name="quickstart-create-a-kubernetes-cluster-on-azure-stack-hci-using-windows-admin-center"></a>クイック スタート:Windows Admin Center を使用して Azure Stack HCI 上に Kubernetes クラスターを作成する

> 適用対象:Azure Stack HCI

Azure Kubernetes Service ホストを設定した後、Windows Admin Center を使用して Kubernetes クラスターを作成できます。 代わりに PowerShell を使用するには、[PowerShell を使用した Kubernetes クラスターの作成](create-kubernetes-cluster-powershell.md)に関するページを参照してください。

次の手順で作業を開始します。

1. [Azure Kubernetes Service を設定し](setup.md)、[システム要件](system-requirements.md)をまだ確認していない場合は確認します。
1. Windows Admin Center で Kubernetes クラスターの作成を開始するには、ゲートウェイ画面で **[追加]** ボタンを押します。
2. **[Add or create resources]\(リソースの追加または作成\)** パネルの **Kubernetes クラスター (プレビュー)** で、 **[新規作成]** を選択して Kubernetes クラスター ウィザードを起動します。 Kubernetes クラスターの [追加] ボタンはパブリック プレビューに表示されますが、機能しません。 Kubernetes クラスターの作成プロセス中はいつでもウィザードを終了できますが、進行状況は保存されないことに注意してください。 

    ![Windows Admin Center の [Add or create resources]\(リソースの追加または作成\) ブレードを示しています。Kubernetes クラスターの新しいタイルが含まれています。](.\media\create-kubernetes-cluster\add-connection.png)

3. Kubernetes クラスターをホストするシステムと Windows Admin Center の前提条件を確認します。 完了したら、 **[次へ]** を選択します。 
4. **[基本]** ページで、Azure Arc 統合、Azure Kubernetes Service ホスト情報、プライマリ ノード プール サイズなど、クラスターに関する情報を構成します。  Azure Kubernetes Service ホストの情報、およびプライマリ ノードのプール サイズ。 Azure Kubernetes Service のホスト フィールドには、Kubernetes クラスターをデプロイする Azure Stack HCI クラスターの完全修飾ドメイン名が必要です。 Azure Kubernetes Service ツールを使用して、このシステムのホスト設定を完了している必要があります。 パブリック プレビュー段階では、ノード数は編集不可であり、既定値は 2 ですが、より大きなワークロードに合わせてノード サイズを構成することができます。 完了したら、 **[次へ]** を選択します。

    ![Kubernetes クラスター ウィザードの [基本] ページを示しています。](.\media\create-kubernetes-cluster\basics.png)

5. **[ノード プール]** ページでワークロードを実行する追加のノード プールを構成することができます。 パブリック プレビュー段階では、(システム ノード プールに加えて) 最大 1 つの Windows ノード プールと 1 つの Linux ノード プールを追加できます。 後でこのウィザードを使用して Azure Arc 統合を有効にする場合は、プライマリ ノード プールではない Linux ノード プールが少なくとも 1 つ必要になります。 完了したら、**[次へ]** をクリックします。
6. **[ネットワーク]** ページでネットワーク構成を指定します。 [詳細設定] を選択すると、クラスター内のノードの仮想ネットワークを作成するときに使用するアドレス範囲をカスタマイズできます。 [基本] を選択すると、クラスター内のノードの仮想ネットワークが既定のアドレス範囲で作成されます。 パブリック プレビュー段階では、ネットワーク設定 (ロード バランサー、ネットワーク ポリシー、および HTTP アプリケーション ルーティング) を変更できません。 完了したら、 **[次へ]** を選択します。

    ![Kubernetes クラスター ウィザードの [ネットワーク] ページを示しています。](.\media\create-kubernetes-cluster\networking.png)

7. **[統合]** ページで、クラスターを Kubernetes ダッシュボード、永続ストレージ、Azure Monitor などの追加サービスに接続します。 永続ストレージは、このページで構成する必要がある唯一のサービスです。 パブリック プレビュー段階では、永続ストレージの場所は、既定でホストの設定時に選択したストレージの場所になります。 このフィールドは現在編集できません。 完了したら、**[次へ]** をクリックします。
8. **[確認と作成]** ページで選択内容を確認します。 問題がなければ、 **[作成]** を選択してデプロイを開始します。 このページの上部にデプロイの進行状況が表示されます。 
9. デプロイが完了すると、 **[次の手順]** ページにクラスターの管理方法の詳細が表示されます。 このページには、SSH キーと Kubernetes ダッシュボード トークンも含まれています。 前の手順で Kubernetes ダッシュボードまたは Azure Arc 統合を無効にすることを選択した場合、このページの情報と手順の一部が使用できないか、機能しない可能性があります。

> [!IMPORTANT] 
> SSH キーと Kubernetes ダッシュボード トークンをセキュリティで保護されたアクセス可能な場所に保存することをお勧めします。

このクイックスタートでは、Azure Kubernetes Service ホストを設定し、Kubernetes クラスターをデプロイしました。 

Azure Stack HCI 上の Azure Kubernetes Service の詳細と、Azure Stack HCI 上の Azure Kubernetes Service に Linux アプリケーションをデプロイして管理する方法については、このチュートリアルを続行してください。
