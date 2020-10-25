---
title: Windows Admin Center を使用して、Azure Stack HCI 上に Azure Kubernetes Service を設定するためのクイックスタート
description: Windows Admin Center を使用して、Azure Stack HCI 上に Azure Kubernetes Service を設定する方法について学習します
author: davannaw-msft
ms.topic: quickstart
ms.date: 09/22/2020
ms.author: dawhite
ms.openlocfilehash: 2f442928c01c59bc85eb672153b2f5b21a653ef1
ms.sourcegitcommit: 362081a8c19e7674c3029c8a44d7ddbe2deb247b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/09/2020
ms.locfileid: "91899722"
---
# <a name="quickstart-set-up-azure-kubernetes-service-on-azure-stack-hci-using-windows-admin-center"></a>クイック スタート:Windows Admin Center を使用して、Azure Stack HCI 上に Azure Kubernetes Service を設定する

> 適用対象:Azure Stack HCI

このクイックスタートでは、Windows Admin Center を使用して、Azure Stack HCI 上に Azure Kubernetes Service を設定します。 代わりに PowerShell を使用する場合は、[PowerShell を使用した設定](setup-powershell.md)に関するページを参照してください。

設定には次のタスクが含まれます。

* Windows Admin Center を設定する (まだ行っていない場合)
* Windows Admin Center を使用するために Azure Stack HCI 用の Azure Kubernetes Service 拡張機能をインストールする
* Kubernetes クラスターのデプロイ先となるシステム上に Azure Kubernetes Service ホストを設定する

作業を開始する前に、[システム要件](.\system-requirements.md)に関するページのすべての前提条件を満たしていることを確認してください。

## <a name="setting-up-windows-admin-center"></a>Windows Admin Center の設定

Windows Admin Center をまだインストールしていない場合は、「[Windows Admin Center のインストール](/windows-server/manage/windows-admin-center/deploy/install)」を参照してください。 Azure Stack HCI 上の Azure Kubernetes Service のパブリック プレビューでは、Windows Admin Center をダウンロードし、Windows 10 コンピューターで実行する必要があります。 現在、Azure Stack HCI 上の Azure Kubernetes Service と互換性があるのは、Windows Admin Center のデスクトップ モードのみです。 Azure Stack HCI 上の Azure Kubernetes Service 機能は、Windows Admin Cente ビルド 2009 以降でのみ利用できます。

## <a name="installing-the-azure-kubernetes-service-extension"></a>Azure Kubernetes Service 拡張機能のインストール

Azure Stack HCI 上の Azure Kubernetes Service パブリック プレビュー ファイルを取得したら、`.nupkg` ファイルをローカルまたは SMB 共有に保存し、ファイル パスを Windows Admin Center 拡張機能マネージャーの [フィード] 一覧に追加する必要があります。 `.nupkg` ファイルは、Windows Admin Center 拡張機能を含む NuGet パッケージです。

既存の拡張機能フィードにアクセスするには、Windows Admin Center を開き、画面の右上隅にある歯車を選択します。 これにより、設定メニューが表示されます。 拡張機能フィードは、 **[拡張機能]** メニューの **[ゲートウェイ]** セクションにあります。 **[フィード]** タブに移動し、 **[追加]** を選択します。 このペインに、Azure Stack HCI 上の Azure Kubernetes Service 拡張機能のコピーへのファイル パスを貼り付け、 **[追加]** を選択します。 ファイル パスが正常に追加された場合は、成功通知を受け取ります。 

フィードを追加したので、利用可能な拡張機能の一覧で、Azure Stack HCI 上の Azure Kubernetes Service 拡張機能を使用できるようになります。 拡張機能を選択したら、テーブルの上部にある **[インストール]** を選択して、この拡張機能をインストールします。 インストールの完了後に、Windows Admin Center が再度読み込まれます。 

[ ![Windows Admin Center 拡張機能マネージャーで使用可能な拡張機能一覧の表示。](.\media\setup\extension-manager.png) ](.\media\setup\extension-manager.png#lightbox)

## <a name="setting-up-an-azure-kubernetes-service-host"></a>Azure Kubernetes Service ホストの設定

Kubernetes クラスターを作成する前に行う必要がある最後の手順が 1 つあります。 Kubernetes クラスターのデプロイ先となるシステム上に、Azure Kubernetes Service ホストを設定する必要があります。 このシステムは、Azure Stack HCI クラスターである必要があります。 

> [!NOTE] 
> Kubernetes クラスターの作成時にマージする目的で 2 つの独立したシステムに Azure Kubernetes Service ホストを設定することは、サポートされているシナリオではありません。 

この設定は、新しい Azure Kubernetes Service ツールを使用して行うことができます。 

このツールを使用すると、必要なパッケージをインストールしてダウンロードできるだけでなく、コア Kubernetes サービスを提供し、アプリケーションのワークロードを調整する管理クラスターを作成できます。 

このツールを使用する前に、PowerShell を開き、各ノードで次のコマンドを実行して、Azure のサインインがどのコンピューターの設定によってもブロックされないことを確認します。
```PowerShell
az login
```

システム設定の確認が済んだので始めましょう。 
1. **[設定]** を選択して設定ウィザードを起動します。
2. Windows Admin Center を実行するコンピューターの前提条件、接続している Azure Stack HCI クラスター、およびネットワークを確認します。 さらに、Windows Admin Center で Azure アカウントにサインインしていること、および使用する予定の Azure サブスクリプションの有効期限が切れていないことを確認します。 完了したら、**[次へ]** をクリックします。
3. ウィザードの **[システム チェック]** ページで、Windows Admin Center ゲートウェイを Azure に接続するなど、必要な操作を行います。 この手順では、Windows Admin Center と、Azure Kubernetes Service をホストするシステムに、続行するための適切な構成があることを確認します。 操作が完了したら、 **[次へ]** を選択します。
4. **[ホストの構成]** 手順で、Azure Kubernetes Service をホストするコンピューターを構成します。 このセクションでは、 **[Automatically download updates]\(更新プログラムを自動的にダウンロードする\)** チェックボックスをオンにすることをお勧めします。 完了後に、 **[次へ]** を選択します。 ウィザードのこの手順では、次の詳細を構成するように求められます。
    * ホストの詳細。管理クラスターの名前や VM イメージを格納するフォルダーなど
    * VM ネットワーク。コンテナーを実行し、コンテナー管理を調整するために作成されるすべての Linux および Windows VM (ノード) に適用されます。 
    * ロード バランサーの設定。外部サービスに使用されるアドレスのプールを定義します

    ![Azure Kubernetes Service ホスト ウィザードの [ホストの構成] 手順を示す画像。](.\media\setup\host-configuration.png)

5. Azure に登録し、 **[Azure registration]\(Azure の登録\)** 手順で Microsoft に診断データを送信することを選択します。 このページでは、Azure サブスクリプションとリソース グループを求められますが、パブリック プレビューでの Azure Kubernetes Service の設定と使用にはコストはかかりません。 Microsoft に送信する診断データは、サービスをセキュリティで保護された最新の状態に保ち、問題のトラブルシューティングを行い、製品の改善に役立てるために使用されます。 選択後に、 **[次へ]** を選択します。
6. **[確認と作成]** 手順で、これまでに選択した内容をすべて確認します。 選択内容に問題がなければ、 **[設定]** を選択してホストの設定を開始します。 
7. **[セットアップの進行状況]** ページで、ホストの設定の進行状況を確認できます。 この時点で、新しいタブで Windows Admin Center を開き、管理タスクを続行してかまいません。 
8. デプロイに成功すると、次の手順が示され、 **[完了]** ボタンが有効になります。 **[次の手順]** で **[Download management cluster kubeconfig]\(管理クラスター kubeconfig をダウンロードする\)** を選択すると、ダウンロードが開始され、ウィザードから離れないように指示にされます。 

## <a name="next-steps"></a>次の手順

このクイックスタートでは、Windows Admin Center と、Azure Stack HCI 用の Azure Kubernetes Service 拡張機能をインストールしました。 また、Kubernetes クラスターのデプロイ先となるシステム上に、Azure Kubernetes Service ホストを構成しました。

これで、[Windows Admin Center での Kubernetes クラスターの作成](create-kubernetes-cluster.md)を進める準備が整いました。