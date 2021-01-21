---
title: Azure からマーケットプレースの項目をダウンロードして Azure Stack Hub に発行する
description: Azure から Marketplace の項目をダウンロードして Azure Stack Hub に発行する方法について学習します。
author: sethmanheim
ms.topic: conceptual
ms.date: 12/16/2020
ms.author: sethm
ms.reviewer: avishwan
ms.lastreviewed: 12/16/2020
zone_pivot_groups: state-connected-disconnected
ms.openlocfilehash: 9560e9e2fcf5d7b01ca44b409d14a7827cac4060
ms.sourcegitcommit: 0983c1f90734b7ea5e23ae614eeaed38f9cb3c9a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/19/2021
ms.locfileid: "98571401"
---
# <a name="download-marketplace-items-to-azure-stack-hub"></a>Azure Stack Hub に Marketplace の項目をダウンロードする

クラウド オペレーターは、Azure Marketplace から項目をダウンロードし、Azure Stack Hub 環境を使用して、すべてのユーザーがそれらを利用可能にすることができます。 選択できる項目は、Azure Marketplace の選別された一覧にある項目であり、Azure Stack Hub で動作するように、事前にテストされ、サポートされています。 一覧には新しいアイテムが頻繁に追加されるので、定期的に新しいコンテンツを確認してください。

Marketplace 製品をダウンロードする場合は、次の 2 つのシナリオがあります。

- **接続されないか、部分的に接続されるシナリオ**:Marketplace の項目をダウンロードには、Marketplace シンジケーション ツールを使用してインターネットにアクセスする必要があります。 その後、ダウンロードしたものを、接続されていない Azure Stack Hub のインストールに転送します。 このシナリオでは、PowerShell を使用します。
- **接続されているシナリオ**:Azure Stack Hub 環境がインターネットに接続されている必要があります。 Azure Stack Hub 管理者ポータルを使用して項目を見つけ、ダウンロードします。

ダウンロードできる Marketplace 項目の完全な一覧については、[Azure Stack Hub 用の Azure Marketplace 項目](azure-stack-marketplace-azure-items.md)に関するページを参照してください。 Azure Stack Hub Marketplace の最近の追加、削除、更新の一覧については、「[Azure Stack Hub Marketplace の変更](azure-stack-marketplace-changes.md)」の記事を参照してください。

> [!NOTE]
> カタログは、お使いの Azure Stack Hub システムの接続先クラウドによって異なります。 クラウド環境は、Azure Stack Hub の登録に使用する Azure サブスクリプションによって決まります。

[!INCLUDE [Azure Stack Hub Operator Access Workstation](../includes/operator-note-owa.md)]

::: zone pivot="state-connected"
接続されたデプロイでは、管理者ポータルを使用して Marketplace 項目をダウンロードできます。

## <a name="prerequisites"></a>前提条件

Azure Stack Hub デプロイは、インターネット接続を備えていて、Azure に登録されている必要があります。

## <a name="use-the-portal-to-download-marketplace-items"></a>ポータルを使用して Marketplace 項目をダウンロードする

1. Azure Stack Hub 管理者ポータルにサインインします。

2. Marketplace 項目をダウンロードする前に、使用可能なストレージ領域を確認してください。 後で、ダウンロードする項目を選択するときに、ダウンロード サイズと使用可能なストレージ容量を比較することができます。 容量が限られている場合は、[空き領域の管理](azure-stack-manage-storage-shares.md#manage-available-space)のためのオプションを検討してください。

   空き領域を確認するには、 **[Region Management]\(リージョン管理\)** で対象リージョンを選択し、 **[リソース プロバイダー]**  >  **[ストレージ]** の順に移動します。

   ![Azure Stack Hub 管理者ポータルでのストレージ スペースの確認](media/azure-stack-download-azure-marketplace-item/storage.png)

3. Azure Stack Hub Marketplace を開き、Azure に接続します。 そのためには、 **[Marketplace management]\(Marketplace 管理\)** サービスを選択し、 **[Marketplace の項目]** を選択して、 **[Add from Azure]\(Azure から追加\)** を選択します。

   ![Azure から Marketplace の項目を追加する](media/azure-stack-download-azure-marketplace-item/marketplace.png)

4. 各行項目には、現在使用可能なバージョンも表示されます。 Marketplace 項目の複数のバージョンを使用できる場合、 **[バージョン]** 列には **[複数]** が表示されます。 各項目をクリックして、その説明と、ダウンロード サイズなどの追加情報を表示できます。

   ![Marketplace 項目の使用可能なバージョンを示すスクリーンショット。](media/azure-stack-download-azure-marketplace-item/add-from-azure1.png)

5. 項目のバージョンが **[複数]** と表示されている場合は、その項目を選択した後表示されるバージョン セレクターのドロップダウンから特定のバージョンを選択できます。 Microsoft は、Azure Stack のバージョンや課金モデルなどのさまざまなプロパティが理由で、利用している Azure Stack と互換性のないマーケットプレース製品を管理者にダウンロードさせないようにする属性を追加することができるようになっています。 これらの属性を追加できるのは Microsoft だけです。

   [![Azure から追加する](media/azure-stack-download-azure-marketplace-item/add-from-azure3sm.png "複数のバージョン")](media/azure-stack-download-azure-marketplace-item/add-from-azure3.png#lightbox)

6. 項目を選択し、 **[ダウンロード]** を選択します。 ダウンロード時間はさまざまで、ネットワーク接続によって異なります。 ダウンロードが完了したら、Azure Stack Hub オペレーターか、ユーザーとして、その新しい Marketplace 項目をデプロイできます。

7. ダウンロードした項目をデプロイするには、 **[+ リソースの作成]** を選択し、カテゴリで新しいマーケットプレース項目を検索します。 次に、デプロイ処理を開始する項目を選択します。 処理は、Marketplace 項目ごとに異なります。
::: zone-end

::: zone pivot="state-disconnected"
Azure Stack Hub のインターネット接続が制限されている場合や接続がない場合は、PowerShell と *マーケットプレース シンジケーション ツール* を使用して、インターネット接続のあるマシンに Marketplace 項目をダウンロードします。 その後、Azure Stack Hub 環境に項目を転送します。 接続されていない環境では、Azure Stack Hub ポータルを使用して Marketplace 項目をダウンロードすることはできません。

マーケットプレース シンジケーション ツールは、接続されているシナリオでも使用することができます。

このシナリオは次の 2 つに分けられます。

- **パート 1**:Marketplace から項目をダウンロードします。 インターネットにアクセスできるコンピューターで PowerShell を構成し、シンジケーション ツールをダウンロードして、Azure Marketplace から項目をダウンロードします。
- **パート 2**:Azure Stack Hub Marketplace にアップロードして発行します。 ダウンロードしたファイルを Azure Stack Hub 環境に移動した後、Azure Stack Hub Marketplace に発行します。

## <a name="prerequisites"></a>前提条件

- 接続された環境 (Azure Stack Hub である必要はありません)。 Azure から製品の一覧をその詳細と共に取得し、すべてをローカルにダウンロードするには、接続が必要です。 これが実行されれば、手順の残りの部分ではインターネット接続は不要です。 これにより、切断された環境で使用するために以前にダウンロードした項目のカタログが作成されます。

- 切断された環境に接続し、必要なすべてのアーティファクトを転送するための リムーバブル メディア。

- 以下の前提条件がある、切断された Azure Stack Hub 環境:

  - お使いの Azure Stack Hub のデプロイが Azure に登録されている必要があります。

  - インターネットに接続できるコンピューターには、**Azure Stack Hub PowerShell モジュール バージョン 1.2.11** 以降がインストールされている必要があります。 まだない場合は、[Azure Stack Hub 固有の PowerShell モジュールをインストール](powershell-install-az-module.md)してください。

  - ダウンロードした Marketplace 項目のインポートを有効にするには、[Azure Stack Hub オペレーター用の PowerShell 環境](azure-stack-powershell-configure-admin.md)を構成する必要があります。
  - .NET Framework 4.7 以降。

次のコマンドを使用して、PowerShell ギャラリーから **Azs.Syndication.Admin** モジュールをダウンロードします。

### <a name="az-modules"></a>[Az モジュール](#tab/az1)

  ```powershell
  Install-Module -Name Azs.Syndication.Admin -AllowPrerelease -PassThru
  ```

### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm1)

  ```powershell
  Install-Module -Name Azs.Syndication.Admin -RequiredVersion 0.1.140
  ```

---

Azure Stack の登録の完了後、[Marketplace management]\(Marketplace 管理\) ブレードに表示される次のメッセージは、切断されたユース ケースには関係ないため無視できます。

![Marketplace Management](media/azure-stack-download-azure-marketplace-item/toolsmsg.png)

## <a name="use-the-marketplace-syndication-tool-to-download-marketplace-items"></a>マーケットプレース シンジケーション ツールを使用して Marketplace 項目をダウンロードする

> [!IMPORTANT]
> 切断されたシナリオで Marketplace 項目をダウンロードするたびに、必ず Marketplace シンジケーション ツールをダウンロードしてください。 このツールは頻繁に変更されるため、各ダウンロードに対して最新バージョンを使用する必要があります。

### <a name="az-modules"></a>[Az モジュール](#tab/az2)

1. インターネットに接続されているコンピューターで、PowerShell コンソールを管理者として開きます。

2. Azure Stack Hub を登録するために使用した Azure アカウントを使用して、適切な Azure クラウドと AzureAD ディレクトリ テナントにサインインします。 アカウントを追加するには、PowerShell で `Login-AzAccount` を実行します。

   ```powershell  
   Login-AzAccount -Environment AzureCloud -Tenant '<mydirectory>.onmicrosoft.com'
   ```

   Azure アカウントの資格情報の入力を求めるメッセージが表示されます。アカウントの構成によっては、2 要素認証を使用する必要があります。

   > [!NOTE]
   > セッションの有効期限が切れた、パスワードが変更された、またはアカウントを切り替えたい場合は、`Add-AzAccount` を使用してサインインする前に、コマンドレット `Remove-AzAccount -Scope Process` を実行します。

3. 複数のサブスクリプションがある場合は、次のコマンドを実行して、登録に使用したものを選択します。

   ```powershell  
   Get-AzSubscription -SubscriptionID 'Your Azure Subscription GUID' | Select-AzSubscription
   ```

4. 前提条件の手順でまだ完了していない場合は、Marketplace シンジケーション ツールの最新バージョンをダウンロードします。

   ```powershell
   Install-Module -Name Azs.Syndication.Admin -AllowPrerelease -PassThru
   ```

5. VM イメージ、拡張機能、ソリューション テンプレートなどのダウンロードする Marketplace 項目を選択するには、次のコマンドを実行します。

   ```powershell
   $products = Select-AzsMarketplaceItem
   ```

   これにより、選択したサブスクリプションで使用可能なすべての Azure Stack 登録を一覧表示するテーブルが表示されます。 Marketplace 項目をダウンロードする Azure Stack 環境と一致する登録を選択し、 **[OK]** を選択します。

     ![選択したサブスクリプションで使用可能なすべての Azure Stack 登録の一覧を示すスクリーンショット。](media/azure-stack-download-azure-marketplace-item/select-registration.png)

   これで、ダウンロード可能なすべての Marketplace 項目を一覧表示する 2 番目のテーブルが表示されます。 ダウンロードする項目を選択し、**バージョン** を書き留めておきます (**Ctrl** キーを押しながら、複数のイメージを選択することができます)。
     ![選択したサブスクリプションで使用可能なすべての Azure Stack 登録の別の一覧を示すスクリーンショット。](media/azure-stack-download-azure-marketplace-item/select-products.png)
  
   また、 **[Add criteria]\(条件の追加\)** オプションを使用して、イメージの一覧をフィルター処理することもできます。
   ![Azure Stack 登録を選択する](media/azure-stack-download-azure-marketplace-item/select-products-with-filter.png)

   必要な項目を選択したら、[OK] をクリックします。

6. ダウンロード対象として選択した Marketplace 項目の ID が `$products` 変数に保存されます。 次のコマンドを使用して、選択した項目のダウンロードを開始します。 対象のフォルダー パスは、Azure Marketplace からダウンロードしたファイルを保存する場所に置き換えます。

    ```powershell
    $products | Export-AzsMarketplaceItem  -RepositoryDir "Destination folder path in quotes"
    ```

7. ダウンロードにかかる時間は、項目のサイズによって異なります。 ダウンロードが完了したら、その項目をスクリプトで指定したフォルダーで使用できます。 ダウンロードには、VHD ファイル (仮想マシンの場合) または .zip ファイル (仮想マシン拡張機能やリソース プロバイダーの場合) が含まれています。 *.azpkg* 形式のギャラリー パッケージが含まれていることもあります (これは .zip ファイルです)。

8. ダウンロードに失敗した場合は、次の PowerShell コマンドレットを再実行してやり直すことができます。

    ```powershell
    $products | Export-AzsMarketplaceItem  -RepositoryDir "Destination folder path in quotes"
    ```

9. **Azs.Syndication.Admin** モジュールもローカルにインストールして、Azure Stack Hub への Marketplace 項目のインポート元となるコンピューターにコピーできるようにする必要があります。

   > [!NOTE]
   > このモジュールのエクスポート先フォルダーは、Marketplace 項目をエクスポートした場所とは別のものにする必要があります。

    ```powershell
    Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name Azs.Syndication.Admin -Path "Destination folder path in quotes" -Force
    ```

### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm2)

1. インターネットに接続されているコンピューターで、PowerShell コンソールを管理者として開きます。

2. Azure Stack Hub を登録するために使用した Azure アカウントを使用して、適切な Azure クラウドと AzureAD ディレクトリ テナントにサインインします。 アカウントを追加するには、PowerShell で `Add-AzureRMAccount` を実行します。

   ```powershell  
   Login-AzureRMAccount -Environment AzureCloud -Tenant '<mydirectory>.onmicrosoft.com'
   ```

   Azure アカウントの資格情報の入力を求めるメッセージが表示されます。アカウントの構成によっては、2 要素認証を使用する必要があります。

   > [!NOTE]
   > セッションの有効期限が切れた、パスワードが変更された、またはアカウントを切り替えたい場合は、`Add-AzureRMAccount` を使用してサインインする前に、コマンドレット `Remove-AzureRMAccount -Scope Process` を実行します。

3. 複数のサブスクリプションがある場合は、次のコマンドを実行して、登録に使用したものを選択します。

   ```powershell  
   Get-AzureRMSubscription -SubscriptionID 'Your Azure Subscription GUID' | Select-AzureRMSubscription
   ```

4. 前提条件の手順でまだ完了していない場合は、Marketplace シンジケーション ツールの最新バージョンをダウンロードします。

   ```powershell
   Install-Module -Name Azs.Syndication.Admin -RequiredVersion 0.1.140
   ```

5. VM イメージ、拡張機能、ソリューション テンプレートなどのダウンロードする Marketplace 項目を選択するには、次のコマンドを実行します。

   ```powershell
   $products = Select-AzsMarketplaceItem
   ```

   これにより、選択したサブスクリプションで使用可能なすべての Azure Stack 登録を一覧表示するテーブルが表示されます。 Marketplace 項目をダウンロードする Azure Stack 環境と一致する登録を選択し、 **[OK]** を選択します。

     ![選択したサブスクリプションで使用可能なすべての Azure Stack 登録の一覧を示すスクリーンショット。](media/azure-stack-download-azure-marketplace-item/select-registration.png)

   これで、ダウンロード可能なすべての Marketplace 項目を一覧表示する 2 番目のテーブルが表示されます。 ダウンロードする項目を選択し、**バージョン** を書き留めておきます (**Ctrl** キーを押しながら、複数のイメージを選択することができます)。
     ![選択したサブスクリプションで使用可能なすべての Azure Stack 登録の別の一覧を示すスクリーンショット。](media/azure-stack-download-azure-marketplace-item/select-products.png)
  
   また、 **[Add criteria]\(条件の追加\)** オプションを使用して、イメージの一覧をフィルター処理することもできます。
   ![Azure Stack 登録を選択する](media/azure-stack-download-azure-marketplace-item/select-products-with-filter.png)

   必要な項目を選択したら、[OK] をクリックします。

6. ダウンロード対象として選択した Marketplace 項目の ID が `$products` 変数に保存されます。 次のコマンドを使用して、選択した項目のダウンロードを開始します。 対象のフォルダー パスは、Azure Marketplace からダウンロードしたファイルを保存する場所に置き換えます。

    ```powershell
    $products | Export-AzsMarketplaceItem  -RepositoryDir "Destination folder path in quotes"
    ```

7. ダウンロードにかかる時間は、項目のサイズによって異なります。 ダウンロードが完了したら、その項目をスクリプトで指定したフォルダーで使用できます。 ダウンロードには、VHD ファイル (仮想マシンの場合) または .zip ファイル (仮想マシン拡張機能やリソース プロバイダーの場合) が含まれています。 *.azpkg* 形式のギャラリー パッケージが含まれていることもあります (これは .zip ファイルです)。

8. ダウンロードに失敗した場合は、次の PowerShell コマンドレットを再実行してやり直すことができます。

    ```powershell
    $products | Export-AzsMarketplaceItem  -RepositoryDir "Destination folder path in quotes"
    ```

9. **Azs.Syndication.Admin** モジュールもローカルにインストールして、Azure Stack Hub への Marketplace 項目のインポート元となるコンピューターにコピーできるようにする必要があります。

   > [!NOTE]
   > このモジュールのエクスポート先フォルダーは、Marketplace 項目をエクスポートした場所とは別のものにする必要があります。

    ```powershell
    Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name Azs.Syndication.Admin -Path "Destination folder path in quotes" -Force
    ```

---



## <a name="import-the-download-and-publish-to-azure-stack-hub-marketplace-using-powershell"></a>PowerShell を使用したダウンロードのインポートと Azure Stack Hub Marketplace への発行

1. ローカルに[ダウンロード済みの](#use-the-marketplace-syndication-tool-to-download-marketplace-items)ファイルを、Azure Stack Hub 環境に接続されているコンピューターに移動する必要があります。 マーケットプレース シンジケーション ツールを使用してインポート操作を実行する必要があるので、このツールも Azure Stack Hub 環境で利用できるようにする必要があります。

   次の図に、フォルダー構造の例を示します。 ダウンロードしたすべての Marketplace 項目は **D:\downloadfolder** に含まれます。 各サブフォルダーは、製品 ID で名前が指定された Marketplace 項目です (例: **microsoft.custom-script-linux-arm-2.0.3**)。 各サブ フォルダーの内容は、Marketplace 項目のダウンロードされたコンテンツです。

   ![Marketplace ダウンロード ディレクトリ構造](media/azure-stack-download-azure-marketplace-item/mp1.png)

2. [こちらの記事](azure-stack-powershell-configure-admin.md)の指示に従って、Azure Stack Hub オペレーターの PowerShell セッションを構成します。

3. "既定のプロバイダー サブスクリプション" に対する所有者アクセス権を持つ ID を使用して、Azure Stack Hub にサインインします。

4. 次のスクリプトを実行して、シンジケーション モジュールをインポートした後、マーケットプレース シンジケーション ツールを起動します。

    ```powershell
    Import-AzsMarketplaceItem -RepositoryDir "Source folder path in quotes"
    ```

5. スクリプトが正常に完了すると、Azure Stack Hub Marketplace で Marketplace 項目を利用できるようになります。
::: zone-end
