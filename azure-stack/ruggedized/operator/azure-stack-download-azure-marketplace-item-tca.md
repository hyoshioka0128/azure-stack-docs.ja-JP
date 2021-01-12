---
title: Azure から Marketplace の項目をダウンロードして Azure Stack に発行する | Microsoft Docs
description: Azure から Marketplace の項目をダウンロードして Azure Stack に発行する方法について説明します。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: tzl
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/02/2020
ms.author: sethm
ms.reviewer: avishwan
ms.lastreviewed: 01/02/2020
ms.openlocfilehash: a3d8fcc1c3ffa65539b233a8b89f55d57f61a5ef
ms.sourcegitcommit: d719f148005e904fa426a001a687e80730c91fda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/06/2021
ms.locfileid: "97910620"
---
# <a name="download-marketplace-items-to-azure-stack-hub-ruggedized"></a>Azure Stack Hub (ラグド) に Marketplace の項目をダウンロードする

クラウド オペレーターは、Azure Marketplace から項目をダウンロードし、Azure Stack Hub 環境を使用して、すべてのユーザーがそれらを利用可能にすることができます。 選択できる項目は、Azure Marketplace の精選された一覧にある項目です。これらの項目は、Azure Stack で動作するように、事前にテストされ、サポートされています。 一覧には新しいアイテムが頻繁に追加されるので、定期的に新しいコンテンツを確認してください。

Marketplace 製品をダウンロードする場合は、次の 2 つのシナリオがあります。

- **接続されているシナリオ**:Azure Stack Hub 環境がインターネットに接続されている必要があります。 Azure Stack Hub 管理者ポータルを使用して項目を見つけ、ダウンロードします。
- **接続されないか、部分的に接続されるシナリオ**:Marketplace の項目をダウンロードには、Marketplace シンジケーション ツールを使用してインターネットにアクセスする必要があります。 その後、ダウンロードしたファイルを、接続されていない Azure Stack インストールに転送します。 このシナリオでは、PowerShell を使用します。

ダウンロードできる Marketplace 項目の完全な一覧については、[Azure Stack 用の Azure Marketplace 項目](../../operator/azure-stack-marketplace-azure-items.md)に関するページを参照してください。 Azure Stack Marketplace の最近の追加、削除、更新の一覧については、「[Azure Stack Marketplace の変更](../../operator/azure-stack-marketplace-changes.md)」の記事を参照してください。

> [!NOTE]
> カタログは、お使いの Azure Stack Hub システムの接続先クラウドによって異なります。 クラウド環境は、Azure Stack Hub の登録に使用する Azure サブスクリプションによって決まります。

## <a name="connected-scenario"></a>接続されているシナリオ

Azure Stack Hub がインターネットに接続されている場合は、管理者ポータルを使用して Marketplace 項目をダウンロードできます。

### <a name="prerequisites"></a>前提条件

Azure Stack Hub デプロイは、インターネット接続を備えていて、[Azure に登録](registration-tzl.md)されている必要があります。

### <a name="use-the-portal-to-download-marketplace-items"></a>ポータルを使用して Marketplace 項目をダウンロードする

1. Azure Stack Hub 管理者ポータルにサインインします。

2. Marketplace 項目をダウンロードする前に、使用可能なストレージ領域を確認してください。 後で、ダウンロードする項目を選択するときに、ダウンロード サイズと使用可能なストレージ容量を比較することができます。 容量が限られている場合は、[空き領域の管理](../../operator/azure-stack-manage-storage-shares.md#manage-available-space)のためのオプションを検討してください。

   空き領域を確認するには、 **[Region Management]\(リージョン管理\)** で対象リージョンを選択し、 **[リソース プロバイダー]**  >  **[ストレージ]** の順に移動します。

   ![Azure Stack 管理者ポータルでのストレージ領域の確認](media/azure-stack-download-azure-marketplace-item-tzl/storage.png)

3. Azure Stack Hub Marketplace を開き、Azure に接続します。 そのためには、 **[Marketplace management]\(Marketplace 管理\)** サービスを選択し、 **[Marketplace の項目]** を選択して、 **[Add from Azure]\(Azure から追加\)** を選択します。

   ![Azure から Marketplace の項目を追加する](media/azure-stack-download-azure-marketplace-item-tzl/marketplace.png)

4. 各行項目には、現在使用可能なバージョンも表示されます。 Marketplace 項目の複数のバージョンを使用できる場合、 **[バージョン]** 列には **[複数]** が表示されます。 各項目をクリックして、その説明と、ダウンロード サイズなどの追加情報を表示できます。

   ![[複数] のバージョン エントリが強調表示されている [Azure から追加する] ページを示すスクリーンショット。](media/azure-stack-download-azure-marketplace-item-tzl/add-from-azure1.png)

5. 項目のバージョンが **[複数]** と表示されている場合は、その項目を選択した後表示されるバージョン セレクターのドロップダウンから特定のバージョンを選択できます。

   ![Azure から追加する](media/azure-stack-download-azure-marketplace-item-tzl/add-from-azure3.png)

6. 項目を選択し、 **[ダウンロード]** を選択します。 ダウンロード時間はさまざまで、ネットワーク接続によって異なります。 ダウンロードが完了したら、Azure Stack オペレーターか、ユーザーとして、その新しい Marketplace 項目をデプロイできます。

7. ダウンロードした項目をデプロイするには、 **[+ リソースの作成]** を選択し、カテゴリで新しいマーケットプレース項目を検索します。 次に、デプロイ処理を開始する項目を選択します。 処理は、Marketplace 項目ごとに異なります。

## <a name="disconnected-or-a-partially-connected-scenario"></a>接続されていないか、部分的に接続されているシナリオ

接続されていない環境では、Azure から Marketplace 項目をダウンロードすることはできません。 オフライン シンジケーション ツールを使用して、Marketplace 項目をローカル ドライブにダウンロードし、そこからそれらの項目をお使いの Azure Stack Hub にアップロードする必要があります。

[オフライン シンジケーション ツールはこちらからダウンロード](https://aka.ms/azsSyndicationtool)できます。

### <a name="download-marketplace-items-from-azure"></a>Azure から Marketplace 項目をダウンロードする

#### <a name="prerequisites"></a>前提条件

- コンピューターにインストールされた Azure PowerShell。

- ご自身の Azure Stack Hub の登録に使用した Azure サブスクリプション ID、登録リソース グループ、および登録名。 この情報は、Azure Stack Hub オペレーター ポータル内の **[リージョンの管理]** ブレードの **[プロパティ]** タブでご確認いただけます。

- Azure サブスクリプション ID に対する所有者/共同作成者の特権を持つアカウントのユーザー名とパスワード。

- オフライン ファイルを書き込むことができるローカル ドライブまたはネットワーク パス。

#### <a name="download-items"></a>項目をダウンロードする

1. PowerShell を開いて、抽出したフォルダーにアクセスします。

2. **Invoke-AzSMarketplaceDownload.ps1** PowerShell スクリプトを実行します。

    ```powershell
    .\Invoke-AzSMarketplaceDownload.ps1 -RegistrationSubscriptionId '<subscription ID>' ` 
       -RegistrationResourceGroup 'azurestack' -RegistrationName '<registration name>' `
       -TenantName mytenant.onmicrosoft.com -DownloadFolder 'F:\offlineSyndication'
    ```

    または、Azure PowerShell を使用して既にログインしている場合は、Azure コンテキストを渡すこともできます。

    ```powershell
    Add-AzureRmAccount -Environment AzureCloud -Tenant mytenant.onmicrosoft.com 
    .\Invoke-AzSMarketplaceDownload.ps1 -RegistrationResourceGroup 'azurestack' -RegistrationName '<registration name>' -DownloadFolder 'F:\offlineSyndication' -AzureContext $(Get-AzureRMContext)
    ```

    Azure コンテキストを渡さない場合は、サインインするように求めるメッセージが表示されます。

3. ダウンロードする製品を選択できるウィンドウが表示されます。 Ctrl キーを押しながらクリックすると、複数の項目を選択できます。

4. **[OK]** を選択します。 これにより、Marketplace 項目とその依存関係がダウンロードされます (存在する場合)。

### <a name="upload-marketplace-items-to-azure-stack-hub"></a>Marketplace 項目を Azure Stack Hub にアップロードする

#### <a name="prerequisites"></a>前提条件

- Azure Stack 管理の Resource Manager エンドポイントとディレクトリ テナント。

- オフライン Marketplace 項目へのアクセス。

#### <a name="upload-items"></a>項目をアップロードする

1. PowerShell を開いて、抽出したフォルダーにアクセスします。

2. **Invoke-AzSMarketplaceUpload.ps1** PowerShell スクリプトを実行します。

    ```powershell
    .\Invoke-AzsMarketplaceUpload.ps1 -AzureStackCloudName "AzureStack-Admin" -AzureStackAdminARMEndpoint https://adminmanagement.<region>.<fqdn> -TenantName mytenant.onmicrosoft.com -DownloadFolder F:\offlineSyndication
    ```

    または、Azure PowerShell でご自身で Azure Stack 環境を設定し、Resource Manager 管理エンドポイントに対して認証を行い、コンテキストをスクリプトに渡すことができます。

    ```powershell
    Add-AzureRmEnvironment -Name Redmond-Admin -ARMEndpoint https://adminmanagement.redmond.azurestack.corp.microsoft.com

    Add-AzureRmAccount -Environment Redmond-Admin

    .\Invoke-AzsMarketplaceUpload.ps1 -DownloadFolder F:\Downloads\offlining -AzureContext $(Get-AzureRmContext)
    ```

    この手順により、指定した Azure Stack Hub に Marketplace 項目がアップロードされます。
