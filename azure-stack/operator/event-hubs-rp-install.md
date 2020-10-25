---
title: Azure Stack Hub に Event Hubs をインストールする方法
description: Azure Stack Hub に Event Hubs リソース プロバイダーをインストールする方法について説明します。
author: BryanLa
ms.author: bryanla
ms.service: azure-stack
ms.topic: how-to
ms.date: 12/09/2019
ms.reviewer: jfggdl
ms.lastreviewed: 12/09/2019
zone_pivot_groups: state-connected-disconnected
ms.openlocfilehash: f7eb790506c5dbbb7d49fc469dd61d394fb1f0fc
ms.sourcegitcommit: 53b0dde60a6435936a5e0cb9e931245f262d637a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/23/2020
ms.locfileid: "91106848"
---
# <a name="how-to-install-event-hubs-on-azure-stack-hub"></a>Azure Stack Hub に Event Hubs をインストールする方法

[!INCLUDE [preview banner](../includes/event-hubs-preview.md)]

この記事では、Event Hubs リソース プロバイダーをダウンロードしてインストールし、サブスクリプションの顧客に提供できるようにする方法について説明します。

## <a name="download-packages"></a>パッケージのダウンロード

Azure Stack Hub に Event Hubs をインストールする前に、Marketplace Management 機能を使用してリソース プロバイダーとその依存パッケージをダウンロードする必要があります。 Marketplace Management 機能に慣れていない場合は、[Azure から Marketplace 項目をダウンロードして Azure Stack Hub に発行する方法](azure-stack-download-azure-marketplace-item.md)に関するページを入念に確認してください。 この記事では、Azure から Azure Stack Hub Marketplace に項目をダウンロードする手順について説明しています。 接続されている場合と接続されていない場合、両方のシナリオについての説明があります。 

> [!NOTE]
> ネットワーク待ち時間と Azure Stack Hub インスタンス上の既存のパッケージによっては、ダウンロード プロセスに 30 分から 2 時間かかることがあります。 

::: zone pivot="state-connected"
接続されたシナリオについては、次の手順に従ってください。

1. Azure Stack Hub 管理者ポータルにサインインします。
2. 左側にある **[Marketplace Management]** を選択します。
3. **[リソース プロバイダー]** を選択します。
4. **+ Add from Azure**(+ Azure から追加) を選択します。
5. 検索バーを使用して　"Event Hubs" を検索します。
6. 検索結果で "Event Hubs" 行を選択します。 
7. "Event Hubs" ダウンロード ページで、インストールする Event Hubs のバージョンを選択し、ページの下部にある **[ダウンロード]** を選択します。 
   [![Marketplace Management のパッケージ](media/event-hubs-rp-install/1-marketplace-management-download.png)](media/event-hubs-rp-install/1-marketplace-management-download.png#lightbox)

Event Hubs と共に、次のような追加のソフトウェア パッケージがダウンロードされることに注意してください。

- Microsoft Azure Stack Hub Add-On RP Windows Server INTERNAL ONLY
- PowerShell Desired State Configuration
::: zone-end

::: zone pivot="state-disconnected"
接続されていないか、部分的に接続されているシナリオの場合は、パッケージをローカル コンピューターにダウンロードしてから、Azure Stack Hub インスタンスにインポートします。

1. まだ行っていない場合は、[Marketplace 項目のダウンロードに関するページの「接続されていないか、部分的に接続されているシナリオ」](azure-stack-download-azure-marketplace-item.md?pivots=state-disconnected)の手順に従ってください。 ここでは、Marketplace シンジケーション ツールをダウンロードして実行します。これにより、Event Hubs パッケージをダウンロードできるようになります。
2. シンジケーション ツールの [Azure Marketplace の項目] ウィンドウが開いたら、[Event Hubs] を見つけて選択し、必要なパッケージをローカル コンピューターにダウンロードします。
3. ダウンロードが完了したら、Azure Stack Hub インスタンスにパッケージをインポートし、Marketplace に発行します。 
::: zone-end

## <a name="installation"></a>インストール 

1. まだサインインしていない場合は、Azure Stack Hub 管理者ポータルにサインインし、左側の **[Marketplace Management]** 選択して、 **[リソース プロバイダー]** を選択します。
2. Event Hubs とその他の必要なソフトウェアがダウンロードされると、**Marketplace Management** に、"Event Hubs" パッケージの状態が "インストールされていません" と表示されます。 状態が "ダウンロード済み" の他のパッケージが存在する場合があります。 インストールする "Event Hubs" の行を選択します。
   [![Marketplace Management のダウンロード済みパッケージ](media/event-hubs-rp-install/2-marketplace-management-downloaded.png)](media/event-hubs-rp-install/2-marketplace-management-downloaded.png#lightbox)
 
3. Event Hubs のインストール パッケージのページの上部に青いバナーが表示されます。 バナーを選択して Event Hubs のインストールを開始します。
   [![Marketplace Management の Event Hubs でインストールを開始する方法を示すスクリーンショット。](media/event-hubs-rp-install/3-marketplace-management-install-ready.png)](media/event-hubs-rp-install/3-marketplace-management-install-ready.png#lightbox)

### <a name="install-prerequisites"></a>必須コンポーネントのインストール

1. 次に、インストール ページが表示されます。 インストール プロセスを開始するには、 **[必須コンポーネントのインストール]** を選択します。
   ![Marketplace Management の Event Hubs で必須コンポーネントのインストールを開始する方法を示すスクリーンショット。](media/event-hubs-rp-install/4-marketplace-management-install-prereqs-start.png)
 
2. 必須コンポーネントのインストールが成功するまで待ちます。 **[必須コンポーネントのインストール]** の横に緑色のチェックマークが表示されるのを確認してから、次の手順に進みます。

   ![Marketplace Management の Event Hubs - 必須コンポーネント](media/event-hubs-rp-install/5-marketplace-management-install-prereqs-succeeded.png)

### <a name="prepare-secrets"></a>シークレットの準備 

1. **2 [Prepare secrets]\(シークレットの準備\)** 手順で、 **[証明書の追加]** を選択すると、 **[証明書の追加]** パネルが表示されます。
   ![Marketplace Management の Event Hubs - シークレットの準備](media/event-hubs-rp-install/6-marketplace-management-install-prepare-secrets.png)

2. **[証明書の追加]** の証明書のファイル名フィールドの右側にある参照ボタンを選択します。
3. 必須コンポーネントを完了したときに生成した .pfx 証明書ファイルを選択します。 詳細については、[インストールの前提条件](event-hubs-rp-prerequisites.md)に関するページを参照してください。 

4. Event Hubs SSL 証明書用にセキュリティで保護された文字列を作成するために指定したパスワードを入力します。 その後、 **[追加]** を選択します。
   ![Marketplace Management の Event Hubs - 証明書の追加](media/event-hubs-rp-install/7-marketplace-management-install-prepare-secrets-add-cert.png)

### <a name="install-resource-provider"></a>リソース プロバイダーのインストール

1. 証明書のインストールが成功したら、 **[Prepare secrets]\(シークレットの準備\)** の横に緑色のチェックマークが表示されるのを確認してから、次の手順に進みます。 次に、**3 [Install resource provider]\(リソース プロバイダーのインストール\)** の横にある **[インストール]** ボタンを選択します。
   ![Marketplace Management の Event Hubs - インストールの開始](media/event-hubs-rp-install/8-marketplace-management-install-start.png)
 
2. 次に、以下のページが表示されます。これは Event Hubs リソース プロバイダーをインストール中であることを示しています。
   [![Marketplace Management の Event Hubs - インストール中](media/event-hubs-rp-install/9-marketplace-management-install-inprogress.png)](media/event-hubs-rp-install/9-marketplace-management-install-inprogress.png#lightbox)
 
3. インストール完了の通知が表示されるのを待ちます。 通常、このプロセスには、Azure Stack Hub の種類に応じて 1 時間以上かかります。 
   [![Marketplace Management の Event Hubs - インストールの完了](media/event-hubs-rp-install/10-marketplace-management-install-complete.png)](media/event-hubs-rp-install/10-marketplace-management-install-complete.png#lightbox)

4. Event Hubs のインストールが正常に完了したことを確認します。そのためには、**Marketplace Management** の **[リソース プロバイダー]** ページに戻ります。 Event Hubs の状態が "インストール済み" と表示されます。
   ![利用可能になった Marketplace Management の Event Hubs](media/event-hubs-rp-install/11-marketplace-management-rps-installed.png)

## <a name="register-event-hubs"></a>Event Hubs の登録

次に、Event Hubs リソース プロバイダーを登録する必要があります。 登録すると、Event Hubs の管理ページを使用してサービスを管理できます。

1. 管理者ポータルの左上にある **[すべてのサービス]** を選択します。
2. **[サブスクリプション]** を選択します。 サブスクリプションの一覧が表示されます。 
   > [!NOTE]
   > **[ユーザー サブスクリプション]** は選択しないでください
3. **[サブスクリプション]** ページで、 **[既定のプロバイダー サブスクリプション]** を選択します。
4. **[既定のプロバイダー サブスクリプション]** ページの左側にある **[リソース プロバイダー]** を選択します。
5. 上部にある **[名前でフィルター]** フィールドで、「EventHub」という文字列を検索します。
6. "Microsoft.EventHub" および "Microsoft.EventHub.Admin" リソース プロバイダーの行の **[状態]** 列を確認します。
7. 状態が "登録解除" になっている場合は、各プロバイダーを選択し、 **[登録]** を選択します。 
   ![登録解除になっているリソース プロバイダー](media/event-hubs-rp-install/12-default-subscription-rps-unregistered.png)
8. 数秒後に、 **[更新]** を選択します。 リソース プロバイダーの状態が登録済みになっていることを確認します。 
9. これで、Microsoft.EventHub と Microsoft.EventHub.Admin の状態が "登録済み" になります。
   ![登録済みのリソース プロバイダー](media/event-hubs-rp-install/13-default-subscription-rps-registered.png)

10. **[すべてのサービス]** ページに戻ります。
11. 「Event Hubs」を検索します。 "Event Hubs" が表示されます。これは、Event Hubs 管理ページのエントリ ポイントです。 
   ![使用可能なサービス - Event Hubs](media/event-hubs-rp-install/14-all-service-event-hubs.png)
 
## <a name="next-steps"></a>次のステップ

ユーザーが Event Hubs リソースをデプロイできるようになるには、1 つ以上のプラン、オファー、およびサブスクリプションを作成しておく必要があります。 

- 初めてサービスを提供する場合は、[ユーザーへのサービス提供](tutorial-offer-services.md)に関するチュートリアルを開始してください。 その後、次のチュートリアルに進んで、[サービス内容をテストします](tutorial-test-offer.md)。
- サービス提供の概念を理解したら、Event Hubs リソース プロバイダーが含まれるオファーとプランを作成します。 次に、ユーザー用のサブスクリプションを作成するか、ユーザーが自身のサブスクリプションを作成できるようにオファー情報を提供します。 参考情報については、[サービス、プラン、オファー、サブスクリプションの概要](service-plan-offer-subscription-overview.md)で、一連の記事をご覧いただくこともできます。

更新プログラムを確認するには、[Azure Stack Hub 上で Event Hubs を更新する方法](resource-provider-apply-updates.md)に関するページをご覧ください。

リソース プロバイダーを削除する必要がある場合は、[Event Hubs リソース プロバイダーの削除](event-hubs-rp-remove.md)に関するページをご覧ください

ユーザー エクスペリエンスの詳細については、ユーザー ドキュメントの「[Azure Stack Hub 上の Event Hubs の概要](../user/event-hubs-overview.md)」を参照してください。