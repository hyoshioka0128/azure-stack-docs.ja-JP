---
title: Azure Stack Hub での GO による API バージョンのプロファイルの使用
description: Azure Stack Hub での GO による API バージョンのプロファイルの使用方法について説明します。
author: sethmanheim
ms.topic: article
ms.date: 09/02/2020
ms.author: sethm
ms.reviewer: sijuman
ms.lastreviewed: 05/26/2019
ms.openlocfilehash: 3b8df9de2975c7ba0e6eefdb10a2731cd5d47ca6
ms.sourcegitcommit: 7c01ab4b2e2250a7acd67d1c5ba27d15c1e8bce0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2020
ms.locfileid: "89448675"
---
# <a name="use-api-version-profiles-with-go-in-azure-stack-hub"></a>Azure Stack Hub での GO による API バージョンのプロファイルの使用

## <a name="go-and-version-profiles"></a>GO と バージョン プロファイル

プロファイルとは、各種のサービスから異なるバージョンのさまざまなリソースの種類を組み合わせたものです。 プロファイルを使用すると、さまざまなリソースの種類を取り混ぜてマッチングできます。 プロファイルには次の利点があります。

- 特定の API バージョンをロックすることによる、アプリの安定性。
- Azure Stack Hub および地域の Azure データセンターでのアプリの互換性。

Go SDK では、このプロファイル パス以下のプロファイルを使用できます。 プロファイルのバージョン番号は、**YYYY-MM-DD** 形式でラベル付けされます。 Azure Stack Hub バージョン 1904 以降の場合、最新の Azure Stack Hub API プロファイル バージョンは **2019-03-01** です。 特定のサービスをプロファイルからインポートするには、プロファイルから該当するモジュールをインポートします。 たとえば、**Compute** サービスを **2019-03-01** プロファイルからインポートするには、次のコードを使用します。

```go
import "github.com/Azure/azure-sdk-for-go/profiles/2019-03-01/compute/mgmt/compute"
```

## <a name="install-the-azure-sdk-for-go"></a>Azure SDK for Go のインストール

1. GIT をインストールします。 手順については、「[Getting Started - Installing Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)」 (はじめに - Git をインストールする) をご覧ください。
2. [Go プログラミング言語](https://golang.org/dl)をインストールします。 Azure 用の API プロファイルでは、Go バージョン 1.9 以降が必要です。
3. 次の bash コマンドを実行して、Azure GO SDK とその依存関係をインストールします。

   ```bash
   go get -u -d github.com/Azure/azure-sdk-for-go/...
   ```

### <a name="the-go-sdk"></a>Go SDK

Azure Go SDK の詳細については、次のリンク先をご覧ください。

- 「[Azure SDK for Go のインストール](/go/azure/azure-sdk-go-install)」にある Azure での Go SDK。
- Azure Go SDK は、GitHub の [azure-sdk-for-go](https://github.com/Azure/azure-sdk-for-go) リポジトリに一般公開されています。

### <a name="go-autorest-dependencies"></a>Go-AutoRest 依存関係

Go SDK では、Azure **Go-AutoRest** モジュールに依存して、Azure Resource Manager エンドポイントに REST 要求を送信します。 Azure  **Go-AutoRest** モジュールの依存関係を [GitHub の Azure Go-AutoRest](https://github.com/Azure/go-autorest) からインポートする必要があります。 インストールの bash コマンドは、「**インストール**」セクションで検索できます。

## <a name="how-to-use-go-sdk-profiles-on-azure-stack-hub"></a>Azure Stack Hub での Go SDK プロファイルの使用方法

Azure Stack Hub でサンプルの Go コードを実行するには、次の手順に従います。

1. Azure SDK for Go とその依存関係をインストールします。 手順については、前のセクションの「[Azure SDK for Go をインストールする](#install-the-azure-sdk-for-go)」をご覧ください。
2. リソース マネージャー エンドポイントからメタデータ情報を取得します。 エンドポイントは、Go コードを実行するために必要な情報と共に、JSON ファイルを返します。

   > [!NOTE]  
   > Azure Stack Development Kit (ASDK) の **ResourceManagerUrl** は `https://management.local.azurestack.external/` になります。  
   > 統合システムの **ResourceManagerUrl** は `https://management.<region>.<fqdn>/` になります。  
   > 必要なメタデータを取得するには、`<ResourceManagerUrl>/metadata/endpoints?api-version=1.0` になります。
  
   サンプルの JSON ファイルは、次のようになります。

   ```json
   { "galleryEndpoint": "https://portal.local.azurestack.external:30015/",  
     "graphEndpoint": "https://graph.windows.net/",  
     "portal Endpoint": "https://portal.local.azurestack.external/",
     "authentication": {
       "loginEndpoint": "https://login.windows.net/",
       "audiences": ["https://management.<yourtenant>.onmicrosoft.com/3cc5febd-e4b7-4a85-a2ed-1d730e2f5928"]
     }
   }
   ```

3. 使用できない場合は、サブスクリプションを作成し、サブスクリプション ID を保存して後で使用します。 サブスクリプションの作成方法に関する詳細については、「[Azure Stack Hub でオファーのサブスクリプションを作成する](../operator/azure-stack-subscribe-plan-provision-vm.md)」を参照してください。

4. **サブスクリプション** スコープと**所有者**ロールを使用して、クライアント シークレットを使用するサービス プリンシパルを作成します。 サービス プリンシパルの ID とシークレットを保存します。 Azure Stack Hub 向けサービス プリンシパルの作成の詳細については、「[アプリ ID を使用してリソースにアクセスする](../operator/azure-stack-create-service-principals.md)」を参照してください。 これで、使用する Azure Stack Hub 環境が設定されました。

5. コード内で Go SDK プロファイルからサービス モジュールをインポートします。 Azure Stack Hub プロファイルの最新バージョンは **2019-03-01** です。 たとえば、**2019-03-01** プロファイルの種類からネットワーク モジュールをインポートするには、次のコードを使用します。

   ```go
   package main
    import "github.com/Azure/azure-sdk-for-go/profiles/2019-03-01/network/mgmt/network"
   ```

6. 関数内で、**New** クライアント関数呼び出しを使って、クライアントを作成して認証します。 仮想ネットワーク クライアントを作成するには、次のコードを使用します。  

   ```go
   package main

   import "github.com/Azure/azure-sdk-for-go/profiles/2019-03-01/network/mgmt/network"

   func main() {
      vnetClient := network.NewVirtualNetworksClientWithBaseURI("<baseURI>", "(subscriptionID>")
      vnetClient.Authorizer = autorest.NewBearerAuthorizer(token)
   ```

   `<baseURI>` に、手順 2 で使用されている **ResourceManagerUrl** 値を設定します。 `<subscriptionID>` に、手順 3 で保存した **SubscriptionID** 値を設定します。

   トークンを作成するには、次のセクションをご覧ください。  

7. 前の手順で作成したクライアントを使用して、API のメソッドを呼び出します。 たとえば、前の手順のクライアントを使用して仮想ネットワークを作成するには、次の例をご覧ください。

   ```go
   package main

   import "github.com/Azure/azure-sdk-for-go/profiles/2019-03-01/network/mgmt/network"
   func main() {
   vnetClient := network.NewVirtualNetworksClientWithBaseURI("<baseURI>", "(subscriptionID>")
   vnetClient .Authorizer = autorest.NewBearerAuthorizer(token)

   vnetClient .CreateOrUpdate( )
   ```

Go SDK プロファイルを使用して Azure Stack Hub に仮想ネットワークを作成する詳細な例については、「[例](#example)」をご覧ください。

## <a name="authentication"></a>認証

Go SDK を使って Azure Active Directory から **Authorizer** プロパティを取得するには、**Go-AutoRest** モジュールをインストールします。 これらのモジュールは、"Go SDK" のインストールで既にインストールされています。 そうでない場合は、[GitHub から認証パッケージ](https://github.com/Azure/go-autorest/tree/master/autorest/adal)をインストールします。

Authorizer には、リソース クライアントの承認者を設定する必要があります。 クライアント資格情報を使用して Azure Stack Hub に authorizer トークンを取得する方法は複数あります。

1. サブスクリプションの所有者ロールを備えたサービス プリンシパルが使用可能な場合は、この手順を省略します。 それ以外の場合は、クライアント シークレットを使用するサービス プリンシパルの作成に関する指示と、ご利用のサブスクリプションにスコープされた "所有者" ロールへの割り当て方法のヘルプについては、「[アプリ ID を使用してリソースにアクセスする](../operator/azure-stack-create-service-principals.md)」を参照してください。 サービス プリンシパル アプリケーション ID とシークレットを必ずキャプチャしてください。

2. コード内で **Go-AutoRest** から **adal** パッケージをインポートします。

   ```go
   package main
   import "github.com/Azure/go-autorest/autorest/adal"
   ```

3. NewOAuthConfig メソッドを使用して、**adal** モジュールから **oauthConfig** を作成します。

   ```go
   package main

   import "github.com/Azure/go-autorest/autorest/ada1"

   func CreateToken() (adal.OAuthTokenProvider, error) {
      var token adal.OAuthTokenProvider
      oauthConfig, err := adal.NewOAuthConfig(activeDirectoryEndpoint, tenantID)
   }
   ```

   `<activeDirectoryEndpoint>` に、このドキュメントの前のセクションで取得した `ResourceManagerUrl` メタデータの `loginEndpoint` プロパティの値を設定します。 `<tenantID>` 値に、お使いの Azure Stack Hub テナント ID を設定します。

4. 最後に、**adal** モジュールの `NewServicePrincipalToken` メソッドを使用して、サービス プリンシパルのトークンを作成します。

   ```go
   package main

   import "github.com/Azure/go-autorest/autorest/adal"

   func CreateToken() (adal.OAuthTokenProvider, error) {
       var token adal.OAuthTokenProvider
       oauthConfig, err := adal.NewOAuthConfig(activeDirectoryEndpoint, tenantID)
       token, err = adal.NewServicePrincipalToken(
           *oauthConfig,
           clientID,
           clientSecret,
           activeDirectoryResourceID)
       return token, err
   ```

    `<activeDirectoryResourceID>` に、このドキュメントの前のセクションで取得した **ResourceManagerUrl** メタデータの "audience" リストのいずれかの値を設定します。
    `<clientID>` を、この記事の前のセクションでサービス プリンシパルが作成されたときに保存した、サービス プリンシパル アプリケーション ID に設定します。
    `<clientSecret>` に、この記事の前のセクションでサービス プリンシパルが作成されたときに保存した、サービス プリンシパル アプリケーションのシークレットを設定します。

## <a name="example"></a>例

この例では、Azure Stack Hub で仮想ネットワークを作成する Go コードのサンプルを示します。 Go SDK の詳細な例については、[Azure Go SDK サンプル リポジトリ](https://github.com/Azure-Samples/azure-sdk-for-go-samples)をご覧ください。 リポジトリのサービス フォルダー内にある hybrid パスの Azure Stack Hub のサンプルを使用できます。

> [!NOTE]  
> この例のコードを実行するには、使用されるサブスクリプションが、**ネットワーク** リソース プロバイダーの一覧に**登録済み**として示されていることを確認します。 これを確認するには、Azure Stack Hub ポータルでサブスクリプションを検索し、 **[リソース プロバイダー]** を選択します。

1. コード内で必要なパッケージをインポートします。 ネットワーク モジュールをインポートするには、Azure Stack Hub で使用可能な最新のプロファイルを使用します。

   ```go
   package main

   import (
       "context"
       "fmt"
       "github.com/Azure/azure-sdk-for-go/profiles/2019-03-01/network/mgmt/network"
       "github.com/Azure/go-autorest/autorest"
       "github.com/Azure/go-autorest/autorest/adal"
       "github.com/Azure/go-autorest/autorest/to"
   )
   ```

2. 使用する環境変数を定義します。 仮想ネットワークを作成するには、リソース グループを保持している必要があります。

   ```go
   var (
       activeDirectoryEndpoint = "yourLoginEndpointFromResourceManagerUrlMetadata"
       tenantID = "yourAzureStackTenantID"
       clientID = "yourServicePrincipalApplicationID"
       clientSecret = "yourServicePrincipalSecret"
       activeDirectoryResourceID = "yourAudienceFromResourceManagerUrlMetadata"
       subscriptionID = "yourSubscriptionID"
       baseURI = "yourResourceManagerURL"
       resourceGroupName = "existingResourceGroupName"
   )
   ```

3. 使用する環境変数を定義したので、**adal** パッケージを使用して、認証トークンを作成するためのメソッドを追加します。 認証の詳細については、前のセクションを参照してください。

   ```go
   //CreateToken creates a service principal token
   func CreateToken() (adal.OAuthTokenProvider, error) {
      var token adal.OAuthTokenProvider
      oauthConfig, err := adal.NewOAuthConfig(activeDirectoryEndpoint, tenantID)
      token, err = adal.NewServicePrincipalToken(
          *oauthConfig,
          clientID,
          clientSecret,
          activeDirectoryResourceID)
      return token, err
   }
   ```

4. `main` メソッドを追加します。 `main` メソッドでは、最初に、前の手順で定義されたメソッドを使用してトークンを取得します。 次に、プロファイルのネットワーク モジュールを使用して、クライアントを作成します。 最後に、仮想ネットワークを作成します。

   ```go
   package main

   import (
      "context"
      "fmt"
      "github.com/Azure/azure-sdk-for-go/profiles/2019-03-01/network/mgmt/network"
      "github.com/Azure/go-autorest/autorest"
      "github.com/Azure/go-autorest/autorest/adal"
      "github.com/Azure/go-autorest/autorest/to"
   )

   var (
      activeDirectoryEndpoint = "yourLoginEndpointFromResourceManagerUrlMetadata"
      tenantID = "yourAzureStackTenantID"
      clientID = "yourServicePrincipalApplicationID"
      clientSecret = "yourServicePrincipalSecret"
      activeDirectoryResourceID = "yourAudienceFromResourceManagerUrlMetadata"
     subscriptionID = "yourSubscriptionID"
     baseURI = "yourResourceManagerURL"
     resourceGroupName = "existingResourceGroupName"
   )

   //CreateToken creates a service principal token
   func CreateToken() (adal.OAuthTokenProvider, error) {
      var token adal.OAuthTokenProvider
      oauthConfig, err := adal.NewOAuthConfig(activeDirectoryEndpoint, tenantID)
      token, err = adal.NewServicePrincipalToken(
          *oauthConfig,
          clientID,
          clientSecret,
          activeDirectoryResourceID)
      return token, err
   }

   func main() {
      token, _ := CreateToken()
      vnetClient := network.NewVirtualNetworksClientWithBaseURI(baseURI, subscriptionID)
      vnetClient.Authorizer = autorest.NewBearerAuthorizer(token)
      future, _ := vnetClient.CreateOrUpdate(
          context.Background(),
          resourceGroupName,
          "sampleVnetName",
          network.VirtualNetwork{
              Location: to.StringPtr("local"),
              VirtualNetworkPropertiesFormat: &network.VirtualNetworkPropertiesFormat{
                  AddressSpace: &network.AddressSpace{
                      AddressPrefixes: &[]string{"10.0.0.0/8"},
                  },
                  Subnets: &[]network.Subnet{
                      {
                          Name: to.StringPtr("subnetName"),
                          SubnetPropertiesFormat: &network.SubnetPropertiesFormat{
                              AddressPrefix: to.StringPtr("10.0.0.0/16"),
                          },
                      },
                  },
              },
          })
      err := future.WaitForCompletionRef(context.Background(), vnetClient.Client)
      if err != nil {
          fmt.Printf(err.Error())
          return
      }
   }
   ```

Go SDK を使用した Azure Stack Hub 向けの入手可能なコード サンプルの一部を次に示します。

- [仮想マシンの作成](https://github.com/Azure-Samples/Hybrid-Compute-Go-Create-VM)
- [ストレージ データプレーン](https://github.com/Azure-Samples/Hybrid-Storage-Go-Dataplane)
- [Managed Disks の使用](https://github.com/Azure-Samples/Hybrid-Compute-Go-ManagedDisks) (Azure Stack Hub によってサポートされている最新の API バージョンをターゲットとする 2019-03-01 プロファイルを使用するサンプル)。

## <a name="next-steps"></a>次のステップ

- [PowerShell for Azure Stack Hub をインストールする](../operator/azure-stack-powershell-install.md)
- [Azure Stack Hub ユーザーの PowerShell 環境の構成](azure-stack-powershell-configure-user.md)
