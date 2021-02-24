---
title: Commvault を使用して Azure Stack Hub で VM をバックアップする
description: Commvault を使用して Azure Stack Hub で VM をバックアップする方法について説明します。
author: mattbriggs
ms.topic: how-to
ms.date: 12/2/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 12/2/2020
ms.openlocfilehash: 0d5b48c04e546d3b8776f436e5562786b8e77f33
ms.sourcegitcommit: 52c934f5eeb5fcd8e8f2ce3380f9f03443d1e445
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/07/2021
ms.locfileid: "97974134"
---
# <a name="back-up-your-vm-on-azure-stack-hub-with-commvault"></a>Commvault を使用して Azure Stack Hub で VM をバックアップする

## <a name="overview-of-backing-up-a-vm-with-commvault"></a>Commvault を使用した VM のバックアップの概要

この記事では、別の Azure Stack Hub スケール ユニットにある復旧 VM を更新する Commvault Live Sync の構成について説明します。 この記事では、Azure Stack Hub にデプロイされた仮想マシンのデータとシステム状態を保護および復旧する一般的なパートナー ソリューションの構成方法について詳しく説明します。

次の図は、Commvault を使用して VM をバックアップする場合の全体的なソリューションを示しています。

![この図は、Commvault を使用して、Azure Stack から別のスタックまたは Azure Cloud にデータをレプリケートする方法を示しています。](./media/azure-stack-network-howto-backup-commvault/bcdr-commvault-overall-arc.png)

この記事では、次のことについて説明します。

1. ソース Azure Stack Hub インスタンスで Commvault ソフトウェアを実行する VM を作成します。

2. セカンダリ ロケーションにストレージ アカウントを作成します。 この記事では、別の Azure Stack Hub インスタンス (ターゲット) のストレージ アカウントに BLOB コンテナーを作成すること、およびソース Azure Stack Hub からターゲット Azure Stack Hub に到達できることを前提としています。

3. ソース Azure Stack Hub インスタンスで Commvault を構成し、ソース Azure Stack Hub の VM を VM グループに追加します。

4. Commvault の Live Sync を構成します。

また、互換性のあるパートナー VM イメージを、Azure クラウドまたは別の Azure Stack Hub にダウンロードして提供し、Azure Stack Hub VM を保護することもできます。 この記事では、Commvault Live Sync を使用した VM の保護について説明します。

このアプローチのトポロジは、次の図のようになります。

![この図は、Azure Stack Hub 1 上の COMMVAULT VSA Proxy から Azure Stack Hub 2 (Hub 1 のバックアップが必要になったときにオンラインに移行できるリカバリー VM がある) へのデータ パスを示しています。](./media/azure-stack-network-howto-backup-commvault/backup-vm-commvault-diagram.svg)

## <a name="create-the-commvault-vm-from-the-commvault-marketplace-item"></a>Commvault Marketplace の項目から Commvault VM を作成する

1. Azure Stack Hub ユーザー ポータルを開きます。

2. **[リソースの作成]**  >  **[Compute]**  >  **[Commvault]** の順に選択します。

    > [!NOTE]  
    > Commvault を使用できない場合は、クラウド オペレーターに問い合わせてください。

    ![VM を作成する](./media/azure-stack-network-howto-backup-commvault/commvault-create-vm-01.png)

3. **[仮想マシンの作成] の [1 基本]** で基本設定を構成します。

    a. **[名前]** を入力します。

    b. **[Standard HDD]** を選択します。
    
    c. **[ユーザー名]** を入力します。
    
    d. **[パスワード]** を入力します。
    
    e. パスワードを確認入力します。
    
    f. バックアップ用の **[サブスクリプション]** を選択します。
    
    g. **リソース グループ** を選択します。
    
    h. Azure Stack Hub の **[場所]** を選択します。 ASDK を使用する場合は、 **[ローカル]** を選択します。
    
    i. **[OK]** を選択します。

    ![[ダッシュボード] > [新規] > [仮想マシンの作成] > [サイズの選択] ダイアログ ボックスに、仮想マシンに使用可能なサイズの一覧が表示されています。](./media/azure-stack-network-howto-backup-commvault/commvault-create-vm-02.png)

4. Commvault VM のサイズを選択します。 バックアップ用の VM サイズは、少なくとも 10 GB の RAM と 100 GB のストレージにする必要があります。

    ![[ダッシュボード] > [新規] > [仮想マシンの作成] > [設定] ダイアログ ボックスに、仮想マシンの作成に使用する設定が表示されています。](./media/azure-stack-network-howto-backup-commvault/commvault-create-vm-03.png).

5. Commvault VM の設定を選択します。

    a. 可用性を **[なし]** に設定します。
    
    b. マネージド ディスクの使用に **[はい]** を選択します。
    
    c. **[仮想ネットワーク]** の既定の VNet を選択します。
    
    d. 既定の **[サブネット]** を選択します。
    
    e. 既定の **[パブリック IP アドレス]** を選択します。
    
    f. **[基本]** のネットワーク セキュリティ グループの VM はそのままにします。
    
    g. HTTP (80)、HTTPS (443)、SSH (22)、および RDP (3389) のポートを開きます。
    
    h. **[拡張機能なし]** を選択します。
    
    i. **[ブート診断]** に対して **[有効]** を選択します。
    
    j. **[ゲスト OS の診断]** の設定を **[無効]** のままにします。
    
    k. 既定の **[診断ストレージ アカウント]** はそのままにします。
    
    l. **[OK]** を選択します。

6. 検証に合格した後に、Commvault VM の概要を確認します。 **[OK]** を選択します。

## <a name="get-your-service-principal"></a>サービス プリンシパルを取得する

ID マネージャーが Azure AD か ADFS かを把握する必要があります。 次の表には、Azure Stack Hub で Commvault を設定するために必要な情報がまとめられています。

| 要素 | 説明 | source |
|--------------------------|--------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------|
| Azure Resource Manager URL | Azure Stack Hub Resource Manager エンドポイント。 | https://docs.microsoft.com/azure-stack/user/azure-stack-version-profiles-ruby#the-azure-stack-hub-resource-manager-endpoint |
| アプリケーション名 |  |  |
| アプリケーション ID | この記事の前のセクションでサービス プリンシパルが作成されたときに保存した、サービス プリンシパル アプリ ID。 | https://docs.microsoft.com/azure-stack/operator/azure-stack-create-service-principals |
| サブスクリプション ID | サブスクリプション ID は、Azure Stack Hub 内のオファーにアクセスするために使用します。 | https://docs.microsoft.com/azure-stack/operator/service-plan-offer-subscription-overview#subscriptions |
| テナント ID (ディレクトリ ID) | Azure Stack Hub のテナント ID。 | https://docs.microsoft.com/azure-stack/operator/azure-stack-identity-overview |
| アプリケーションのパスワード | サービス プリンシパルの作成時に保存した、サービス プリンシパル アプリ シークレット。 | https://docs.microsoft.com/azure-stack/operator/azure-stack-create-service-principals |

## <a name="configure-backup-using-the-commvault-console"></a>Commvault Console を使用してバックアップを構成する

1. RDP クライアントを開き、Azure Stack Hub の Commavult VM に接続します。 資格情報を入力します。

2. Commvault VM に Azure Stack Hub PowerShell と Azure Stack Hub ツールをインストールします。

    a. Azure Stack Hub PowerShell のインストール手順については、「[PowerShell for Azure Stack Hub をインストールする](../operator/powershell-install-az-module.md?toc=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fazure-stack%2Fuser%2FTOC.json&bc=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fazure-stack%2Fbreadcrumb%2Ftoc.json)」を参照してください。  
    b. Azure Stack Hub ツールのインストール手順については、「[GitHub からの Azure Stack Hub ツールのダウンロード](../operator/azure-stack-powershell-download.md?toc=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fazure-stack%2Fuser%2FTOC.json%3Fview%3Dazs-1908&bc=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fazure-stack%2Fbreadcrumb%2Ftoc.json%3Fview%3Dazs-1908&view=azs-1908)」を参照してください。

3. Commvault VM に Commvault がインストールされたら、Commcell Console を開きます。 [スタート] から **[Commvault]**  >  **[Commvault Commcell Console]** の順に選択します。

    ![Commcell Console の左側には、CommCell Browser というタイトルのナビゲーション ペインがあります。 右側のペインには、[Getting Started]\(作業の開始\) タブ ページが表示されています。](./media/azure-stack-network-howto-backup-commvault/commcell-console.png)

4. Commvault Commcell Console で Azure Stack Hub の外部ストレージを使用するように、バックアップ リポジトリを構成します。 CommCell Browser で、[Storage Resources]\(ストレージ リソース\) > [Storage Pools]\(記憶域プール\) の順に選択します。 右クリックして **[Add Storage Pool]\(記憶域プールの追加\)** を選択します。 **[クラウド]** を選択します

5. 記憶域プールの名前を追加します。 **[次へ]** を選択します。

6. **[Create]\(作成\)**  >  **[Cloud Storage]\(クラウド ストレージ\)** を選択します。

    ![[StorageDevice#]\(ストレージ デバイス #\) ダイアログ ボックスに、[全般] タブ ページが表示されています。ここには、作成するストレージ デバイスを指定するための各種一覧とテキスト ボックスがあります。](./media/azure-stack-network-howto-backup-commvault/commcell-storage-add-storage-device.png)

7. クラウド サービス プロバイダーを選択します。 この手順では、別の場所にある 2 つ目の Azure Stack Hub を使用します。 Microsoft Azure Storage を選択します。

8. Commvault VM を MediaAgent として選択します。

9. ストレージ アカウントのアクセス情報を入力します。 Azure Storage アカウントを設定する手順については、こちらを参照してください。 アクセス情報:

    -  **サービス ホスト**: リソース内の BLOB コンテナーのプロパティから URL の名前を取得します。 たとえば、URL が https:\//backuptest.blob.westus.stackpoc.com/mybackups でした。サービス ホストで blob.westus.stackpoc.com を使用しました。
    
    -   **アカウント名**: ストレージ アカウント名を使用します。 これは、ストレージ リソースの [アクセス キー] ブレードで確認できます。
    
    -   **アクセス キー**:ストレージ リソースの [アクセス キー] ブレードからアクセス キーを取得します。
    
    -   **コンテナー**:コンテナーの名前。 このケースでは、mybackups です。
    
    -   **ストレージ クラス**:ユーザー コンテナーの既定のストレージ クラスのままにします。

10. [Microsoft Azure Stack Hub クライアントの作成](https://documentation.commvault.com/commvault/v11_sp13/article?p=86495.htm)に関するページの手順に従って、Microsoft Azure Stack Hub クライアントを作成します。

    ![[Create Azure Stack Client]\(Azure Stack クライアントの作成\) ダイアログ ボックスには、クライアントの特性を指定するための一覧とテキスト ボックスがあります。](./media/azure-stack-network-howto-backup-commvault/commcell-ceate-client.png)

11. バックアップ ポリシーを保護およびアタッチする VM またはリソース グループを選択します。

12. 復旧の RPO 要件に合わせてバックアップ スケジュールを構成します。

13. 最初の完全バックアップを実行します。

## <a name="configure-commvault-live-sync"></a>Commvault Live Sync を構成する 

2 つのオプションを使用できます。 バックアップのプライマリ コピーから変更をレプリケートするか、セカンダリ コピーから復旧 VM に変更をレプリケートするかを選択できます。 バックアップ セットからレプリケーションを行うと、ソース マシンに対する読み取り IO の影響がなくなります。

1. Live Sync の構成時に、ソース Azure Stack Hub (仮想サーバー エージェント) とターゲット Azure Stack Hub の詳細を指定する必要があります。

2. Commvault Live Sync を構成する手順については、[Microsoft Azure Stack Hub のライブ同期レプリケーション](https://documentation.commvault.com/commvault/v11_sp13/article?p=94386.htm)に関するページを参照してください。

    ![Commcell Console に、タブ ページ [vm-kr-cvlt] > [クライアント コンピューター] > [ASIC Azure Stack] > [仮想サーバー] > [Azure Stack] > [defaultBackupSet] が表示されています。 このページの [Off Stack Protection]\(Stack 外保護\) のコンテキスト メニューに、[Live Sync] > [構成] オプションがあります。](./media/azure-stack-network-howto-backup-commvault/live-sync-1.png)
 
3. Live Sync の構成時に、ターゲット Azure Stack Hub と仮想サーバー エージェントの詳細を指定する必要があります。

    ![[Live Sync Options for Subclient Off Stack Protection]\(サブクライアントの Stack 外保護の Live Sync オプション\) ウィザードの [Destination]\(宛先\) ステップには、Virtualization Client とプロキシ クライアントを指定するためのリスト ボックスがあります。](./media/azure-stack-network-howto-backup-commvault/live-sync-2.png)

4. 構成を続行し、レプリカ ディスクがホストされるターゲット ストレージ アカウント、レプリカ VM が配置されるリソース グループ、およびレプリカ VM に接続する名前を追加します。

    ![[Live Sync Options for Subclient Off Stack Protection]\(サブクライアントの Stack 外保護の Live Sync オプション\) ウィザードの [仮想マシン] ステップでは、VM を追加および削除できます。](./media/azure-stack-network-howto-backup-commvault/live-sync-3.png)

5. また、各 VM の横にある **[構成]** を選択して、VM サイズを変更し、ネットワーク設定を構成することもできます。

6. レプリケーションの頻度を、ターゲット Azure Stack Hub に設定します

    ![[Live Sync Options for Subclient Off Stack Protection]\(サブクライアントの Stack 外保護の Live Sync オプション\) ウィザードの [ジョブ オプション] ステップでは、バックアップ スケジュールを指定できます。](./media/azure-stack-network-howto-backup-commvault/live-sync-5.png)

7. 設定を確認して構成を保存します。 これで復旧環境が作成され、選択した間隔でレプリケーションが開始されます。


## <a name="set-up-failover-behavior-using-live-sync"></a>Live Sync を使用してフェールオーバー動作を設定する

Commvault Live Sync を使用すると、ある Azure Stack Hub から別の Azure Stack Hub にマシンをフェールオーバーしてフェールバックし、元の Azure Stack Hub で操作を再開できます。 ワークフローは自動化され、ログに記録されます。

![管理コンソールの [レプリケーション モニター] ページの [Replication RPO]\(レプリケーション RPO\) ペインの各サブペインには、使用可能なデータが表示されていません。 [レプリケーション モニター] ペインには、2 つの VM が表示されています。 それぞれに、レプリケーション情報の行があります。](./media/azure-stack-network-howto-backup-commvault/back-up-live-sync-panel.png)

復旧 Azure Stack Hub にフェールオーバーする VM を選択し、計画的な、または計画外のフェールオーバーを選択します。 計画的なフェールオーバーは、復旧サイトで操作を再開する前に運用環境を適切にシャットダウンする時間がある場合に適しています。 計画的なフェールオーバーでは、運用 VM をシャットダウンし、最終変更を復旧サイトにレプリケートし、最新のデータで復旧 VM をオンラインにし、Live Sync 構成時に指定した VM サイズとネットワーク構成を適用します。 計画外のフェールオーバーでは、運用 VM のシャットダウンを試みますが、運用環境が使用不能な場合は次に進み、VM に適用された最後に受信したレプリケーション データ セットと以前に選択したサイズとネットワーク構成を使用して、復旧 VM をオンラインにします。 次の画像は、復旧 VM が Commvault Live Sync によってオンラインにされた計画外のフェールオーバーを示しています。

![[ジョブの概要] には、[種類]、[Priority]\(優先度\)、[開始時刻]、[終了時刻] など、ディザスター リカバリー イベントに関する情報が表示されます。](./media/azure-stack-network-howto-backup-commvault/unplanned-failover.png)

![[イベント] というタイトルの一覧に、[DR Orchestration job has completed.]\(DR オーケストレーション ジョブが完了しました\) という説明の単一のイベントが表示されています。 このイベントにはその他の情報があります。](./media/azure-stack-network-howto-backup-commvault/fail-over-2.png)

![[Phase Details]\(フェーズの詳細\) というタイトルの一覧には、4 台のマシンの 6 つのイベントが表示されています。 それぞれに、フェーズ名、状態、開始時刻、終了時刻があります。 フェーズ名は、[電源オフ]、[電源オン]、[Disable Synch]\(同期の無効化\)、および [Post Operation]\(POST 操作\) です。](./media/azure-stack-network-howto-backup-commvault/fail-over-3.png)

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub ネットワークの違いと考慮事項](azure-stack-network-differences.md)  
