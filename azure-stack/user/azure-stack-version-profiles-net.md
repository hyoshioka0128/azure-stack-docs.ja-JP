---
title: Azure Stack Hub での .NET による API バージョンのプロファイルの使用
description: Azure Stack Hub での .NET SDK による API バージョンのプロファイルの使用方法について説明します。
author: sethmanheim
ms.topic: article
ms.date: 12/2/2020
ms.author: sethm
ms.reviewer: sijuman
ms.lastreviewed: 12/2/2020
ms.openlocfilehash: 0b790eebb276c23f5204d3f6c178c9e6ba54480b
ms.sourcegitcommit: 52c934f5eeb5fcd8e8f2ce3380f9f03443d1e445
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/07/2021
ms.locfileid: "97973998"
---
# <a name="use-api-version-profiles-with-net-in-azure-stack-hub"></a>Azure Stack Hub での .NET による API バージョンのプロファイルの使用

Azure Stack Hub Resource Manager 向けの .NET SDK には、インフラストラクチャの構築と管理に役立つツールが用意されています。 SDK のリソース プロバイダーには、Compute、Networking、Storage、App Services、および [KeyVault](/azure/key-vault/key-vault-whatis) が含まれます。 .NET SDK には、14 個の NuGet パッケージが含まれています。 プロジェクトをコンパイルするたびに、これらのパッケージをソリューションにダウンロードする必要があります。 しかし、ご利用のアプリのメモリを最適化するために、**2019-03-01-hybrid** または **2018-03-01-hybrid** のバージョンに使用するリソース プロバイダーを指定してダウンロードすることができます。 各パッケージは、リソース プロバイダー、それぞれの API バージョン、それが属する API プロファイルで構成されます。 .NET SDK に含まれている API プロファイルを使用すると、グローバルな Azure リソースと Azure Stack Hub 上のリソースとを切り替えることで、ハイブリッド クラウド開発を行うことができます。

## <a name="net-and-api-version-profiles"></a>.NET と API バージョンのプロファイル

API プロファイルは、リソース プロバイダーと API バージョンを組み合わせたものです。 API プロファイルを使用して、リソース プロバイダーのパッケージに含まれる各リソースの種類の、最も安定した最新バージョンを取得します。

- すべてのサービスの最新バージョンを使用するには、パッケージの **latest** プロファイルを使用します。 このプロファイルは、**Microsoft.Azure.Management** NuGet パッケージの一部です。

- Azure Stack Hub と互換性のあるサービスを使用するには、次のいずれかのパッケージを使用します。
  - **Microsoft.Azure.Management.Profiles.hybrid\_2019\_03\_01.<*ResourceProvider*>.0.9.0-preview.nupkg**
  - **Microsoft.Azure.Management.Profiles.hybrid\_2018\_03\_01.<*ResourceProvider*>.0.9.0-preview.nupkg**

  上記の NuGet パッケージの **ResourceProvider** 部分が適切なプロバイダーに変更されていることを確認します。

- サービスの最新の API バージョンを使用するには、特定の NuGet パッケージの **Latest** プロファイルを使用します。 たとえば、Compute サービスの **latest-API** バージョンを単独で使用する場合は、**Compute** パッケージの **latest** プロファイルを使用します。 **latest** プロファイルは、**Microsoft.Azure.Management** NuGet パッケージの一部です。

- 特定のリソース プロバイダーのリソースの種類に合わせて特定の API バージョンを使用するには、パッケージ内で定義されている特定の API バージョンを使用します。

すべてのオプションを同じアプリケーション内で組み合わせることができます。

## <a name="install-the-azure-net-sdk"></a>Azure .NET SDK のインストール

- GIT をインストールします。 手順については、「[はじめに - Git のインストール][]」 (はじめに - Git をインストールする) をご覧ください。

- 適切な NuGet パッケージをインストールするには、[パッケージの検索とインストール][]に関するページを参照してください。

- インストールする必要があるパッケージは、使用するプロファイル バージョンによって異なります。 プロファイル バージョンのパッケージ名は次のとおりです。

  - **Microsoft.Azure.Management.Profiles.hybrid\_2019\_03\_01.<*ResourceProvider*>.0.9.0-preview.nupkg**

  - **Microsoft.Azure.Management.Profiles.hybrid\_2018\_03\_01.<*ResourceProvider*>.0.9.0-preview.nupkg**

- Visual Studio Code 用の適切な NuGet パッケージをインストールするには、次のリンクを参照して [NuGet パッケージ マネージャーについての指示][]をダウンロードしてください。

- 使用できない場合は、サブスクリプションを作成し、サブスクリプション ID を保存して後で使用します。 サブスクリプションの作成方法の詳細については、「[Azure Stack Hub でオファーのサブスクリプションを作成する][]」を参照してください。

- サービス プリンシパルを作成し、クライアント ID とクライアント シークレットを保存します。 Azure Stack Hub 用のサービス プリンシパルの作成方法については、[Azure Stack Hub へのアクセスをアプリケーションに提供する][]に関する記事を参照してください。 クライアント ID は、サービス プリンシパルの作成時にはアプリケーション ID とも呼ばれます。

- サブスクリプションでサービス プリンシパルのロールが共同作成者/所有者であることを確認します。 サービス プリンシパルにロールを割り当てる方法については、[Azure Stack Hub へのアクセスをアプリケーションに提供する][]に関する記事を参照してください。

## <a name="prerequisites"></a>前提条件

Azure Stack Hub で .NET Azure SDK を使用するには、次の値を指定した後、環境変数を使用して値を設定する必要があります。 環境変数を設定するには、使用している特定のオペレーティング システムの表の後にある手順を参照してください。

| 値                     | 環境変数   | 説明                                                                                                             |
|---------------------------|-------------------------|-------------------------------------------------------------------------------------------------------------------------|
| テナント ID                 | `AZURE_TENANT_ID `      | Azure Stack Hub の [*テナント ID*][] の値。                                                                          |
| クライアント ID                 | `AZURE_CLIENT_ID `      | この記事の前のセクションでサービス プリンシパルが作成されたときに保存した、サービス プリンシパル アプリ ID。 |
| サブスクリプション ID           | `AZURE_SUBSCRIPTION_ID` | [*サブスクリプション ID*][] は、Azure Stack Hub 内のオファーにアクセスするために必要です。                                                      |
| クライアント シークレット             | `AZURE_CLIENT_SECRET`   | サービス プリンシパルの作成時に保存した、サービス プリンシパル アプリ シークレット。                                      |
| Resource Manager エンドポイント | `ARM_ENDPOINT`          | [*Azure Stack Hub Resource Manager エンドポイント*][] に関する記事を参照してください。                                                                    |
| 場所                  | `RESOURCE_LOCATION`     | Azure Stack Hub の場所。

Azure Stack Hub のテナント ID を確認するには、[この記事](../operator/azure-stack-csp-ref-operations.md)の手順に従ってください。 環境変数を設定するには、次の手順に従います。

### <a name="windows"></a>Windows

環境変数を設定するには、Windows コマンド プロンプトで次の形式を使用します。

```shell
set Azure_Tenant_ID=Your_Tenant_ID
```

### <a name="macos-linux-and-unix-based-systems"></a>macOS、Linux、および Unix ベースのシステム

Unix ベースのシステムでは、次のコマンドを使用します。

```shell
export Azure_Tenant_ID=Your_Tenant_ID
```

### <a name="the-azure-stack-hub-resource-manager-endpoint"></a>Azure Stack Hub Resource Manager エンドポイント

Azure Resource Manager は、管理者が Azure リソースのデプロイ、管理、監視を行えるようにするための管理フレームワークです。 Azure Resource Manager では、これらのタスクを個別に処理するのではなく、グループとして単一の操作で処理することができます。

Resource Manager エンドポイントからメタデータ情報を取得できます。 エンドポイントは、コードを実行するために必要な情報と共に、JSON ファイルを返します。

次の考慮事項に注意してください。

- Azure Stack Development Kit (ASDK) の **ResourceManagerUrl** は `https://management.local.azurestack.external/` です。

- 統合システムの **ResourceManagerUrl** は `https://management.region.<fqdn>/` です。ここで、`<fqdn>` は完全修飾ドメイン名です。
必要なメタデータを取得するには、`<ResourceManagerUrl>/metadata/endpoints?api-version=1.0` になります。

サンプルの JSON ファイルは、次のようになります。

```json
{
   "galleryEndpoint": "https://portal.local.azurestack.external:30015/",
   "graphEndpoint": "https://graph.windows.net/",
   "portal Endpoint": "https://portal.local.azurestack.external/",
   "authentication": 
      {
         "loginEndpoint": "https://login.windows.net/",
         "audiences": ["https://management.yourtenant.onmicrosoft.com/3cc5febd-e4b7-4a85-a2ed-1d730e2f5928"]
      }
}
```

## <a name="existing-api-profiles"></a>既存の API プロファイル

- **Microsoft.Azure.Management.Profiles.hybrid\_2019\_03\_01.<*ResourceProvider*>.0.9.0-preview.nupkg**:Azure Stack Hub 用に作成された最新のプロファイル。 バージョン 1904 以降を使用している限り、Azure Stack Hub と最も互換性のあるサービスには、このプロファイルを使用します。

- **Microsoft.Azure.Management.Profiles.hybrid\_2018\_03\_01.<*ResourceProvider*>.0.9.0-preview.nupkg**: バージョン 1808 以降を使用する Azure Stack Hub と互換性のあるサービスには、このプロファイルを使用します。

- **Latest**:すべてのサービスの最新バージョンで構成されているプロンプトファイル。 すべてのサービスの最新バージョンを使用してください。 このプロファイルは、**Microsoft.Azure.Management** NuGet パッケージの一部です。

Azure Stack Hub および API プロファイルの詳細については、「[API プロファイルの概要][]」を参照してください。

## <a name="azure-net-sdk-api-profile-usage"></a>Azure .NET SDK API プロファイルの使用

リソース管理クライアントをインスタンス化するには、次のコードを使用します。 同様のコードを使用して、他のリソース プロバイダー クライアント (Compute、Network、Storage など) をインスタンス化することができます。

```csharp
var client = new ResourceManagementClient(armEndpoint, credentials)
{
    SubscriptionId = subscriptionId;
};
```

このコードの `credentials` パラメーターは、クライアントをインスタンス化するために必要です。 次のコードでは、テナント ID とサービス プリンシパルによって認証トークンを生成します。

```csharp
var azureStackSettings = getActiveDirectoryServiceSettings(armEndpoint);
var credentials = ApplicationTokenProvider.LoginSilentAsync(tenantId, servicePrincipalId, servicePrincipalSecret, azureStackSettings).GetAwaiter().GetResult();
```

コードの `getActiveDirectoryServiceSettings` 呼び出しによって、メタデータ エンドポイントから Azure Stack Hub エンドポイントが取得されます。 これは、この呼び出しからの環境変数を示しています。

```csharp
public static ActiveDirectoryServiceSettings getActiveDirectoryServiceSettings(string armEndpoint)
{
    var settings = new ActiveDirectoryServiceSettings();
    try
    {
        var request = (HttpWebRequest)HttpWebRequest.Create(string.Format("{0}/metadata/endpoints?api-version=1.0", armEndpoint));
        request.Method = "GET";
        request.UserAgent = ComponentName;
        request.Accept = "application/xml";
        using (HttpWebResponse response = (HttpWebResponse)request.GetResponse())
        {
            using (StreamReader sr = new StreamReader(response.GetResponseStream()))
            {
                var rawResponse = sr.ReadToEnd();
                var deserialized = JObject.Parse(rawResponse);
                var authenticationObj = deserialized.GetValue("authentication").Value<JObject>();
                var loginEndpoint = authenticationObj.GetValue("loginEndpoint").Value<string>();
                var audiencesObj = authenticationObj.GetValue("audiences").Value<JArray>();
                settings.AuthenticationEndpoint = new Uri(loginEndpoint);
                settings.TokenAudience = new Uri(audiencesObj[0].Value<string>());
                settings.ValidateAuthority = loginEndpoint.TrimEnd('/').EndsWith("/adfs", StringComparison.OrdinalIgnoreCase) ? false : true;
            }
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine(String.Format("Could not get AD service settings. Exception: {0}", ex.Message));
    }
    return settings;
}
```

これらの手順により、API プロファイルの NuGet パッケージを使用して、アプリケーションを Azure Stack Hub に正常にデプロイすることができます。

## <a name="samples-using-api-profiles"></a>API プロファイルを使用したサンプル

次のサンプルは、.NET と Azure Stack Hub の API プロファイルを使用してソリューションを作成するためのリファレンスとして使用できます。

- [リソース グループの管理](https://github.com/Azure-Samples/hybrid-resources-dotnet-manage-resource-group)
- [ストレージ アカウントの管理](https://github.com/Azure-Samples/hybird-storage-dotnet-manage-storage-accounts)
- [仮想マシンの管理](https://github.com/Azure-Samples/hybrid-compute-dotnet-manage-vm): このサンプルでは、Azure Stack Hub でサポートされている **2019-03-01-hybrid** プロファイルを使用しています。

## <a name="next-steps"></a>次のステップ

API プロファイルに関する詳細情報

- [Azure Stack Hub での API バージョンのプロファイルの管理](azure-stack-version-profiles.md)
- [プロファイルでサポートされているリソース プロバイダー API バージョン](azure-stack-profiles-azure-resource-manager-versions.md)

  [はじめに - Git のインストール]: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git
  [パッケージの検索とインストール]: /nuget/tools/package-manager-ui
  [NuGet パッケージ マネージャーについての指示]: https://marketplace.visualstudio.com/items?itemName=jmrog.vscode-nuget-package-manager
  [Azure Stack Hub でオファーのサブスクリプションを作成する]: ../operator/azure-stack-subscribe-plan-provision-vm.md
  [Azure Stack Hub へのアクセスをアプリケーションに提供する]: ../operator/azure-stack-create-service-principals.md
  [*テナント ID*]: ../operator/azure-stack-identity-overview.md
  [*サブスクリプション ID*]: ../operator/service-plan-offer-subscription-overview.md#subscriptions
  [*Azure Stack Hub Resource Manager エンドポイント*]: ../user/azure-stack-version-profiles-ruby.md#the-azure-stack-hub-resource-manager-endpoint
  [API プロファイルの概要]: ../user/azure-stack-version-profiles.md#summary-of-api-profiles
  [Test Project to Virtual Machine, vNet, resource groups, and storage account]: https://github.com/seyadava/azure-sdk-for-net-samples/tree/master/TestProject
  [Use Azure PowerShell to create a service principal with a certificate]: ../operator/azure-stack-create-service-principals.md
  [Run unit tests with Test Explorer.]: /visualstudio/test/run-unit-tests-with-test-explorer?view=vs-2017&preserve-view=true
