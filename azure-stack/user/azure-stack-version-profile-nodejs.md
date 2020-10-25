---
title: Azure Stack Hub の Node.js で API バージョンのプロファイルを使用する
description: Azure Stack Hub での Node.js による API バージョンのプロファイルの使用について説明します。
author: mattbriggs
ms.topic: article
ms.date: 04/30/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 04/30/2020
ms.openlocfilehash: 93c37e67af83dd7b6d735916cfacc7dab5c0cdbf
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90574093"
---
# <a name="use-api-version-profiles-with-nodejs-software-development-kit-sdk-in-azure-stack-hub"></a>Azure Stack Hub の Node.js ソフトウェア開発キット (SDK) で API バージョン プロファイルを使用する

## <a name="nodejs-and-api-version-profiles"></a>Node.js と API バージョンのプロファイル

Node.js SDK を使用して、アプリのインフラストラクチャのビルドと管理を行うことができます。 Node.js SDK の API プロファイルは、グローバルな Azure リソースと Azure Stack Hub リソースを切り替えできることによって、ハイブリッド クラウド ソリューションに役立ちます。 コードを 1 回記述すれば、グローバル Azure と Azure Stack Hub の両方をターゲットにすることができます。 

この記事では、開発ツールとして [Visual Studio Code](https://code.visualstudio.com/) を使用できます。 Visual Studio Code では、Node.js SDK をデバッグし、アプリを実行して Azure Stack Hub インスタンスにアプリをプッシュできます。 Visual Studio Code またはコマンド `node <nodefile.js>` を実行してターミナル ウィンドウからデバッグできます。

## <a name="the-nodejs-sdk"></a>Node.js SDK

Node.js SDK には、Azure Stack Hub Resource Manager ツールが用意されています。 SDK のリソース プロバイダーには、コンピューティング、ネットワーク、ストレージ、アプリ サービス、および KeyVault が含まれます。 Node.js アプリケーションにインストールできる 10 個のリソース プロバイダー クライアント ライブラリがあります。 アプリケーションのメモリを最適化するために、**2018-03-01-hybrid** または **2019-03-01-profile** に使用するリソース プロバイダーを指定してダウンロードすることができます。 各モジュールは、リソース プロバイダー、それぞれの API バージョン、API プロファイルで構成されます。 

API プロファイルは、リソース プロバイダーと API バージョンを組み合わせたものです。 API プロファイルを使用すると、リソース プロバイダーのパッケージに含まれる各リソースの種類の、最も安定した最新バージョンを取得できます。

  -   すべてのサービスの最新バージョンを使用するには、パッケージの **latest** プロファイルを使用します。

  -   Azure Stack Hub と互換性のあるサービスを使用するには、 **\@azure/arm-resources-profile-hybrid-2019-03-01** または **\@azure/arm-storage-profile-2019-03-01-hybrid** を使用します

### <a name="packages-in-npm"></a>npm のパッケージ

各リソース プロバイダーには独自のパッケージがあります。 パッケージは、[npm レジストリ](https://www.npmjs.com/package/@azure/arm-storage-profile-2019-03-01-hybrid)から取得できます。

次のパッケージを見つけることができます。

| リソース プロバイダー | Package |
| --- | --- |
| [App Service](https://www.npmjs.com/package/@azure/arm-appservice-profile-2019-03-01-hybrid) | @azure/arm-appservice-profile-2019-03-01-hybrid |
| [Azure Resource Manager サブスクリプション](https://www.npmjs.com/package/@azure/arm-subscriptions-profile-hybrid-2019-03-01) | @azure/arm-subscriptions-profile-hybrid-2019-03-01  |
| [Azure Resource Manager ポリシー](https://www.npmjs.com/package/@azure/arm-policy-profile-hybrid-2019-03-01) | @azure/arm-policy-profile-hybrid-2019-03-01
| [Azure Resource Manager DNS](https://www.npmjs.com/package/@azure/arm-dns-profile-2019-03-01-hybrid) | @azure/arm-dns-profile-2019-03-01-hybrid  |
| [承認](https://www.npmjs.com/package/@azure/arm-authorization-profile-2019-03-01-hybrid) | @azure/arm-authorization-profile-2019-03-01-hybrid  |
| [Compute](https://www.npmjs.com/package/@azure/arm-compute-profile-2019-03-01-hybrid) | @azure/arm-compute-profile-2019-03-01-hybrid |
| [Storage](https://www.npmjs.com/package/@azure/arm-storage-profile-2019-03-01-hybrid) | @azure/arm-storage-profile-2019-03-01-hybrid |
| [Network](https://www.npmjs.com/package/@azure/arm-network-profile-2019-03-01-hybrid) | @azure/arm-network-profile-2019-03-01-hybrid |
| [リソース](https://www.npmjs.com/package/@azure/arm-resources-profile-hybrid-2019-03-01) | @azure/arm-resources-profile-hybrid-2019-03-01 |
 | [Keyvault](https://www.npmjs.com/package/@azure/arm-keyvault-profile-2019-03-01-hybrid) | @azure/arm-keyvault-profile-2019-03-01-hybrid |

サービスの最新の API バージョンを使用するには、特定のクライアント ライブラリの **Latest** プロファイルを使用します。 たとえば、リソース サービスの最新の API バージョンを単独で使用する場合は、**リソース管理クライアント ライブラリ** パッケージの `azure-arm-resource` プロファイルを使用します。 。

リソース プロバイダーの特定の API バージョンには、パッケージ内で定義されている特定の API バージョンを使用します。

  > [!NOTE]  
  > すべてのオプションを同じアプリケーション内で組み合わせることができます。

## <a name="install-the-nodejs-sdk"></a>Node.js SDK のインストール

1. GIT をインストールします。 手順については、「[Getting Started - Installing Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)」 (はじめに - Git をインストールする) をご覧ください。

2. [Node.js](https://nodejs.org/en/download/) の現在のバージョンをインストールまたはアップグレードします。 Node.js には、[npm](https://www.npmjs.com/) JavaScript パッケージ マネージャーも含まれています。

3. [Visual Studio Code](https://code.visualstudio.com/) をインストールまたはアップグレードし、Visual Studio Code 用の [Node.js 拡張機能](https://code.visualstudio.com/docs/nodejs/nodejs-debugging)をインストールします。

2.  Azure Stack Hub Resource Manager のクライアントパッケージをインストールします。 詳細については、[クライアント ライブラリをインストールする方法](https://www.npmjs.com/package/@azure/arm-keyvault-profile-2019-03-01-hybrid)に関するページを参照してください。

3.  インストールする必要があるパッケージは、使用したいプロファイル バージョンによって異なります。 リソース プロバイダーの一覧については、「 [npm のパッケージ](#packages-in-npm)」セクションを参照してください。

4. npm を使用してリソース プロバイダー クライアント ライブラリをインストールします。 コマンド ラインから `npm install <package-name>` を実行します。 たとえば、`npm install @azure/arm-authorization-profile-2019-03-01-hybrid` を実行して、承認リソース プロバイダー ライブラリをインストールできます。

5.  SDK を使用する場合は、サブスクリプションを作成し、サブスクリプション ID をメモしておきます。 手順については、[Azure Stack Hub でオファーのサブスクリプションを作成する](/azure/azure-stack/azure-stack-subscribe-plan-provision-vm)方法に関するページを参照してください。

6.  サービス プリンシパルを作成し、クライアント ID とクライアント シークレットを保存します。 クライアント ID は、サービス プリンシパルの作成時にはアプリケーション ID とも呼ばれます。 手順については、「[Azure Stack Hub へのアクセスをアプリケーションに提供する](../operator/azure-stack-create-service-principals.md)」を参照してください。

7.  サブスクリプションでサービス プリンシパルのロールが共同作成者/所有者であることを確認します。 サービス プリンシパルへのロールの割り当て手順については、「[Azure Stack Hub へのアクセスをアプリケーションに提供する](../operator/azure-stack-create-service-principals.md)」を参照してください。

### <a name="nodejs-prerequisites"></a>Node.js の前提条件 

Azure Stack Hub で Node.js Azure SDK を使用するには、次の値を指定した後、環境変数に値を設定する必要があります。 環境変数を設定するには、使用しているオペレーティング システムの表の後にある手順を参照してください。

| 値 | 環境変数 | 説明 |
| --- | --- | --- |
| テナント ID | TENANT\_ID | Azure Stack Hub の[テナント ID](/azure/azure-stack/azure-stack-identity-overview) の値。 |
| クライアント ID | CLIENT\_ID | このドキュメントの前のセクションでサービス プリンシパルが作成されたときに保存した、サービス プリンシパル アプリケーション ID。  |
| サブスクリプション ID | AZURE\_SUBSCRIPTION\_ID   [サブスクリプション ID](../operator/service-plan-offer-subscription-overview.md#subscriptions) は Azure Stack Hub 内のオファーにアクセスする方法です。  |
| クライアント シークレット | APPLICATION\_SECRET | サービス プリンシパルの作成時に保存した、サービス プリンシパル アプリケーション シークレット 。 |
| Resource Manager エンドポイント | ARM\_ENDPOINT | [Azure Stack Hub Resource Manager エンドポイント](/azure/azure-stack/user/azure-stack-version-profiles-ruby#the-azure-stack-hub-resource-manager-endpoint) に関する記事を参照してください。 |

#### <a name="set-your-environmental-variables-for-nodejs"></a>Node.js の環境変数を設定する

環境変数を設定するには:

  - **Microsoft Windows**  

    環境変数を設定するには、Windows コマンド プロンプトで次の形式を使用します。
      
      `set Tenant_ID=<Your_Tenant_ID>`

  - **macOS、LinuxおよびUnixベース システム**  

    環境変数を設定するには、bash プロンプトで次の形式を使用します。

    `export Azure_Tenant_ID=<Your_Tenant_ID>`

**Azure Stack Hub Resource Manager エンドポイント**

Microsoft Azure Resource Manager は、管理者が Azure リソースのデプロイ、管理、監視を行えるようにするための管理フレームワークです。 Azure Resource Manager では、これらのタスクを個別に処理するのではなく、グループとして単一の操作で処理することができます。

メタデータ情報は、Resource Manager エンドポイントから取得できます。 エンドポイントは、コードを実行するために必要な情報と共に、JSON ファイルを返します。

> [!NOTE]  
> Azure Stack Development Kit (ASDK) の **ResourceManagerUrl** は`https://management.local.azurestack.external` です。統合システムの **ResourceManagerUrl** は `https://management.region.<fqdn>/` です。ここで、`<fqdn>` は完全修飾ドメイン名です。
必要なメタデータを取得するには、`<ResourceManagerUrl>/metadata/endpoints?api-version=1.0` になります。

サンプルの JSON ファイルは、次のようになります。

```JSON  
{
    "galleryEndpoint": "https://portal.local.azurestack.external/",

    "graphEndpoint": "https://graph.windowsNode.js/",

    "portal Endpoint": "https://portal.local.azurestack.external/",

    "authentication": {

        "loginEndpoint": "https://login.windowsNode.js/",

        "audiences": ["https://management.<yourtenant>.onmicrosoft.com/"]

    }

}
```

### <a name="existing-api-profiles"></a>既存の API プロファイル

-  **\@azure/arm-resourceprovider-profile-2019-03-01-hybrid**

    Azure Stack Hub 用に作成された最新のプロファイル。 1808 以降のスタンプを使用している限り、Azure Stack Hub との互換性に優れたサービスには、このプロファイルを使用します。

-  **\@azure-arm-resource**

    このプロファイルは、すべてのサービスの最新バージョンで構成されます。 Azure ですべてのサービスの最新バージョンを使用してください。

Azure Stack Hub および API プロファイルについて詳しくは、「[API プロファイルの概要](/azure/azure-stack/user/azure-stack-version-profiles#summary-of-api-profiles)」をご覧ください。

### <a name="azure-nodejs-sdk-api-profile-usage"></a>Azure Node.js SDK API プロファイルの使用

プロファイル クライアントをインスタンス化するには、次のコードを使用します。 このパラメーターは、Azure Stack Hub またはまたはその他のプライベート クラウドにのみ必要です。 グローバル Azure では、@azure-arm-resource または @azure-arm-storage によって、既定ですでにこれらの設定が使用されます。

```Node.js  
var ResourceManagementClient = require('@azure/arm-resources-profile-hybrid-2019-03-01').ResourceManagementClient;

var StorageManagementClient = require('@azure/arm-storage-profile-2019-03-01-hybrid').StorageManagementClient;
````

Azure Stack Hub でサービス プリンシパルを認証するには、次のコードが必要です。 これにより、テナント ID と、Azure Stack Hub に固有の認証基準によって、トークンが作成されます。

```Node.js  
var clientId = process.env['AZURE_CLIENT_ID'];
var tenantId = process.env['AZURE_TENANT_ID']; //"adfs"
var secret = process.env['AZURE_CLIENT_SECRET'];
var subscriptionId = process.env['AZURE_SUBSCRIPTION_ID'];
var base_url = process.env['ARM_ENDPOINT'];
var resourceClient, storageClient;
```

これにより、API プロファイル クライアント ライブラリを使用して、アプリケーションを Azure Stack Hub に正常にデプロイすることができます。

次のコード スニペットでは、Azure Stack Hub インスタンスに指定した Azure Resource Manager エンドポイントを使用して、ギャラリー エンドポイント、グラフ エンドポイント、対象ユーザー、ポータル エンドポイントなどの上記のデータを収集します。

```Node.js  
var map = {};
const fetchUrl = base_url + 'metadata/endpoints?api-version=1.0'
```

## <a name="environment-settings"></a>環境設定

サービス プリンシパルを Azure Stack Hub 環境に対して認証するには、次のコードを使用します。コマンド プロンプトでこのコードを使用し、環境変数を設定すると、開発者に対してこのマッピングが自動的に生成されます。

```Node.js  
function main() {
  var initializePromise = initialize();
  initializePromise.then(function (result) {
    var userDetails = result;
    console.log("Initialized user details");
    // Use user details from here
    console.log(userDetails)
    map["name"] = "AzureStack"
    map["portalUrl"] = userDetails.portalEndpoint 
    map["resourceManagerEndpointUrl"] = base_url 
    map["galleryEndpointUrl"] = userDetails.galleryEndpoint 
    map["activeDirectoryEndpointUrl"] = userDetails.authentication.loginEndpoint.slice(0, userDetails.authentication.loginEndpoint.lastIndexOf("/") + 1) 
    map["activeDirectoryResourceId"] = userDetails.authentication.audiences[0] 
    map["activeDirectoryGraphResourceId"] = userDetails.graphEndpoint 
    map["storageEndpointSuffix"] = "." + base_url.substring(base_url.indexOf('.'))  
    map["keyVaultDnsSuffix"] = ".vault" + base_url.substring(base_url.indexOf('.')) 
    map["managementEndpointUrl"] = userDetails.authentication.audiences[0] 
    map["validateAuthority"] = "false"
    Environment.Environment.add(map);
```

## <a name="samples-using-api-profiles"></a>API プロファイルを使用したサンプル

次のサンプルは、Node.js と Azure Stack Hub API のプロファイルを使用してソリューションを作成するための参考資料として使用できます。 GitHub から、次のリポジトリでサンプルを取得できます。

- [ストレージ ノード リソース プロバイダーの概要](https://github.com/Azure-Samples/hybrid-storage-nodejs-create-storageaccount)
- [コンピューティング ノードの管理](https://github.com/Azure-Samples/Hybrid-compute-nodejs-create-vm)
- [リソースマネージャー ノードのリソースとグループ](https://github.com/Azure-Samples/Hybrid-resourcegroups-nodejs-manageresources)

### <a name="sample-create-storage-account"></a>ストレージ アカウントの作成のサンプル 

1.  リポジトリを複製します。

    ```bash  
    git clone https://github.com/sijuman/storage-node-resource-provider-getting-started.git
    ```

1. `cd` でリポジトリの複製に移動します。

2.  Azure サービス プリンシパルを作成し、サブスクリプションにアクセスするロールを割り当てます。 手順については、[Azure PowerShell を使用して資格情報でサービス プリンシパルを作成する](/azure/azure-stack/azure-stack-create-service-principals)方法に関するページをご覧ください。

3.  次の必要な値を取得します。
    - テナント ID
    - クライアント ID (アプリケーション ID)
    - クライアント シークレット
    - Azure サブスクリプション ID
    - Azure Stack Hub Resource Manager エンドポイント

4.  コマンド プロンプトを使用して作成したサービス プリンシパルから取得した情報を使用して、次の環境変数を設定します。

    > [!NOTE]  
    > Windows では、**export** ではなく **set** を使用します。

    ```bash  
    export TENANT_ID=<your tenant id>
    export CLIENT_ID=<your client id>
    export APPLICATION_SECRET=<your client secret>K
    export AZURE_SUBSCRIPTION_ID=<your subscription id>
    export ARM_ENDPOINT=<your Azure Stack Hub Resource manager URL>
    ```

5.  サンプル アプリケーションの `index.js` ファイルを開きます。

6.  location 変数を Azure Stack Hub の場所に設定します。 たとえば、「 `LOCAL = "local"` 」のように入力します。

7.  Azure Stack Hub に対して認証できるようにする資格情報を設定します。 コードのこの部分は、index.js ファイルのこのサンプルに含まれています。

    ```Node.js  
    var clientId = process.env['CLIENT_ID'];
    var tenantId = process.env['TENANT_ID']; //"adfs"
    var secret = process.env['APPLICATION_SECRET'];
    var subscriptionId = process.env['AZURE_SUBSCRIPTION_ID'];
    var base_url = process.env['ARM_ENDPOINT'];
    var resourceClient, storageClient;
    ```

8.  上で規定されているクライアント ライブラリの組み合わせを使用して、正しいクライアント ライブラリを設定していることを確認します。 このサンプルでは、次のものを使用しています。

    ```Node.js
    var ResourceManagementClient = require('@azure/arm-resources-profile-hybrid-2019-03-01').ResourceManagementClient;
    var StorageManagementClient = require('@azure/arm-storage-profile-2019-03-01-hybrid').StorageManagementClient;
    ```

9.  [npm モジュール検索](https://www.npmjs.com/package/@azure/arm-keyvault-profile-2019-03-01-hybrid)を使用して、**2019-03-01-hybrid** を検索し、コンピューティング、ネットワーク、ストレージ、KeyVault、App Services のリソース プロバイダー用の、このプロファイルに関連付けられているパッケージをインストールします。

    これを行うには、コマンド プロンプトを開き、リポジトリのルート フォルダーにリダイレクトして、使用するリソースプロバイダーごとに `npm install @azure/arm-keyvault-profile-2019-03-01-hybrid` を実行します。

10.  コマンド プロンプトでコマンド `npm install` を実行して、すべての node.js モジュールをインストールします。

11.  サンプルを実行します。

        ```Node.js  
        node index.js
        ```

12.  index.js の後にクリーンアップするには、クリーンアップ スクリプトを実行します。

        ```Node.js  
        Node cleanup.js <resourceGroupName> <storageAccountName>
        ```

13.  サンプルが正常に完了しました。 サンプルの詳細については、以下を参照してください。

### <a name="what-does-indexjs-do"></a>index.js の機能

サンプルでは、新しいストレージ アカウントの作成、サブスクリプションまたはリソースグループ内のストレージ アカウントの一覧表示、ストレージ アカウントキーの一覧表示、ストレージ アカウントキーの再生成、ストレージ アカウントのプロパティの取得、ストレージ アカウント SKU の更新、およびストレージ アカウント名の使用可能性のチェックを行います。

サンプルを起動するには、サービス プリンシパルを使用してログインし、資格情報とサブスクリプション ID を使用して **ResourceManagementClient** オブジェクトと **StorageManagementClient** オブジェクトを作成します。

これにより、サンプルで、新しいストレージ アカウントを作成するリソース グループが設定されます。

```Node.js  
function createResourceGroup(callback) {
  var groupParameters = { location: location, tags: { sampletag: 'sampleValue' } };
  console.log('\nCreating resource group: ' + resourceGroupName);
  return resourceClient.resourceGroups.createOrUpdate(resourceGroupName, groupParameters, callback);
}
```

### <a name="create-a-new-storage-account"></a>新しいストレージ アカウントの作成

次に、サンプルでは、前の手順で作成したリソース グループに関連付けられた新しいストレージ アカウントが作成されます。

この場合、ストレージ アカウント名は、一意性を確保するためにランダムに生成されます。 ただし、サブスクリプションに同じ名前のアカウントがすでに存在する場合、新しいストレージ アカウントを作成する呼び出しは成功します。

```Node.js  
var createParameters = {
    location: location,
    sku: {
        name: accType,
    },
    kind: 'Storage',
    tags: {
        tag1: 'val1',
        tag2: 'val2'
    }
};


console.log('\\n--&gt;Creating storage account: ' + storageAccountName + ' with parameters:\\n' + util.inspect(createParameters));

return storageClient.storageAccounts.create(resourceGroupName, storageAccountName, createParameters, callback);
```

### <a name="list-storage-accounts-in-the-subscription-or-resource-group"></a>サブスクリプションまたはリソース グループ内のストレージ アカウントの一覧表示

このサンプルでは、指定されたサブスクリプションのすべてのストレージ アカウントが一覧表示されます。

```Node.js  
console.log('\\n--&gt;Listing storage accounts in the current subscription.');

return storageClient.storageAccounts.list(callback);

It also lists storage accounts in the resource group:

    console.log('\\n--&gt;Listing storage accounts in the resourceGroup : ' + resourceGroupName);

return storageClient.storageAccounts.listByResourceGroup(resourceGroupName, callback);
```

### <a name="read-and-regenerate-storage-account-keys"></a>ストレージ アカウントキーの読み取りと再生成

このサンプルでは、新しく作成されたストレージ アカウントとリソース グループのストレージ アカウント キーが一覧表示されます。

```Node.js  
console.log('\\n--&gt;Listing storage account keys for account: ' + storageAccountName);

return storageClient.storageAccounts.listKeys(resourceGroupName, storageAccountName, callback);
```

また、アカウント アクセス キーも再生成されます。

```Node.js  
console.log('\\n--&gt;Regenerating storage account keys for account: ' + storageAccountName);

return storageClient.storageAccounts.regenerateKey(resourceGroupName, storageAccountName, 'key1', callback);
```

### <a name="get-storage-account-properties"></a>ストレージ アカウントのプロパティの取得

このサンプルでは、ストレージ アカウントのプロパティが読み取られます。

```Node.js  
console.log('\\n--&gt;Getting info of storage account: ' + storageAccountName);

return storageClient.storageAccounts.getProperties(resourceGroupName, storageAccountName, callback);
```

### <a name="check-storage-account-name-availability"></a>ストレージ アカウント名の使用可能性のチェック

このサンプルでは、指定されたストレージアカウント名が Azure で使用できるかどうかがチェックされます。

```Node.js  
console.log('\\n--&gt;Checking if the storage account name : ' + storageAccountName + ' is available.');

return storageClient.storageAccounts.checkNameAvailability(storageAccountName, callback);
```

### <a name="delete-the-storage-account-and-resource-group"></a>ストレージ アカウントとリソース グループの削除

cleanup.js を実行すると、サンプルで作成されたストレージ アカウントが削除されます。

```Node.js  
console.log('\\nDeleting storage account : ' + storageAccountName);
return storageClient.storageAccounts.deleteMethod(resourceGroupName, storageAccountName, callback);
```

また、サンプルで作成されたリソース グループも削除されます。

```Node.js  
console.log('\\nDeleting resource group: ' + resourceGroupName);

return resourceClient.resourceGroups.deleteMethod(resourceGroupName, callback);
```

## <a name="next-steps"></a>次のステップ

API プロファイルの詳細については、以下を参照してください。

- [Azure Stack Hub での API バージョンのプロファイルの管理](azure-stack-version-profiles.md)
- [プロファイルでサポートされているリソース プロバイダー API バージョン](azure-stack-profiles-azure-resource-manager-versions.md)
