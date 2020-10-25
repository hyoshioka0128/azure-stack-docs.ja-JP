---
title: Azure Stack Hub での Java による API バージョンのプロファイルの使用
description: Azure Stack Hub での Java による API バージョンのプロファイルの使用方法について説明します。
author: sethmanheim
ms.topic: article
ms.date: 09/03/2020
ms.author: sethm
ms.reviewer: sijuman
ms.lastreviewed: 05/16/2019
ms.openlocfilehash: 2e37d8f977a89e9a0207ef094273e9838a54d23a
ms.sourcegitcommit: 7c01ab4b2e2250a7acd67d1c5ba27d15c1e8bce0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2020
ms.locfileid: "89448658"
---
# <a name="use-api-version-profiles-with-java-in-azure-stack-hub"></a>Azure Stack Hub での Java による API バージョンのプロファイルの使用

Azure Stack Hub Resource Manager 向けの Java SDK には、インフラストラクチャの構築と管理に役立つツールが用意されています。 SDK のリソース プロバイダーには、Compute、Networking、Storage、App Services、および [KeyVault](/azure/key-vault/key-vault-whatis) が含まれます。

Java SDK は、 **.java** ファイルに適切なモジュールを読み込む **Pom.xml** ファイルに依存関係を含めることで、API プロファイルを組み込みます。 ただし、依存関係として複数のプロファイル (**2019-03-01-hybrid** など) を追加したり、Azure プロファイルとして**最新のもの**を追加したりすることができます。 これらの依存関係を使用すると適切なモジュールが読み込まれるため、リソースの種類を作成するときに、それらのプロファイルから使用したい API バージョンを選択することができます。 これにより、Azure Stack Hub の最新の API バージョンに対する開発の際に、Azure で最新バージョンを使用することができます。

Java SDK を使用すると、真のハイブリッド クラウド開発者エクスペリエンスを実現できます。 Java SDK に含まれている API プロファイルを使用すると、グローバルな Azure リソースと Azure Stack Hub 上のリソース間を切り替えることで、ハイブリッド クラウド開発を行うことができます。

## <a name="java-and-api-version-profiles"></a>Java と API バージョンのプロファイル

API プロファイルは、リソース プロバイダーと API バージョンを組み合わせたものです。 API プロファイルを使用して、リソース プロバイダーのパッケージに含まれる各リソースの種類の、最も安定した最新バージョンを取得します。

- すべてのサービスの最新バージョンを使用するには、依存関係として **latest** プロファイルを使用します。

  - latest プロファイルを使用する場合、依存関係は **com.microsoft.azure** です。

  - Azure Stack Hub でサポートされている最新のサービスを使用するには、**com.microsoft.azure.profile\_2019\_03\_01\_hybrid** プロファイルを使用します。

    - このプロファイルは、**Pom.xml** ファイルに依存関係として指定されます。これにより、.NET と同様に、ドロップダウン リストから適切なクラスを選択すると、モジュールが自動的に読み込まれます。

  - 依存関係は次のようになります。

     ```xml
     <dependency>
     <groupId>com.microsoft.azure.profile_2019_03_01_hybrid</groupId>
     <artifactId>azure</artifactId>
     <version>1.0.0-beta-1</version>
     </dependency>
     ```

  - 特定のリソース プロバイダーのリソースの種類に合わせて特定の API バージョンを使用するには、Intellisense を使用して定義した特定の API バージョンを使用します。

すべてのオプションを同じアプリ内で組み合わせることができます。

## <a name="install-the-azure-java-sdk"></a>Azure Java SDK のインストール

次の手順に従って、Java SDK をインストールします。

1. 正式な指示に従って、Git をインストールします。 「[はじめに - Git のインストール](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)」を参照してください。

2. 指示に従って、[Java SDK](https://zulu.org/download/) と [Maven](https://maven.apache.org/) をインストールします。 適切なバージョンは、Java Developer Kit のバージョン 8 です。 Apache Maven の適切なバージョンは 3.0 以降です。 このクイック スタートを完了するには、`JAVA_HOME` 環境変数を Java Development Kit のインストール場所に設定する必要があります。 詳細については、「[Java と Maven を使用して初めての関数を作成する](/azure/azure-functions/functions-create-first-java-maven)」を参照してください。

3. 適切な依存関係パッケージをインストールするには、Java アプリで **Pom.xml** ファイルを開きます。 次のコードに示されているように、依存関係を追加します。

   ```xml  
   <dependency>
   <groupId>com.microsoft.azure.profile_2019_03_01_hybrid</groupId>
   <artifactId>azure</artifactId>
   <version>1.0.0-beta-1</version>
   </dependency>
   ```

4. インストールする必要があるパッケージのセットは、使用したいプロファイル バージョンによって異なります。 プロファイル バージョンのパッケージ名は次のとおりです。

   - **com.microsoft.azure.profile\_2019\_03\_01\_hybrid**
   - **com.microsoft.azure**
     - **latest**

5. 使用できない場合は、サブスクリプションを作成し、後で使用するためにサブスクリプション ID を保存します。 サブスクリプションの作成方法については、「[Azure Stack Hub でオファーのサブスクリプションを作成する](../operator/azure-stack-subscribe-plan-provision-vm.md)」を参照してください。

6. サービス プリンシパルを作成し、クライアント ID とクライアント シークレットを保存します。 Azure Stack Hub 用のサービス プリンシパルの作成方法については、[Azure Stack Hub へのアクセスのアプリケーションへの提供](../operator/azure-stack-create-service-principals.md)に関するページを参照してください。 クライアント ID は、サービス プリンシパルの作成時にはアプリケーション ID とも呼ばれます。

7. サブスクリプションでサービス プリンシパルのロールが共同作成者/所有者であることを確認します。 サービス プリンシパルへのロールの割り当て手順については、[Azure Stack Hub へのアクセスのアプリケーションへの提供](../operator/azure-stack-create-service-principals.md)に関するページを参照してください。

## <a name="prerequisites"></a>前提条件

Azure Stack Hub で Azure Java SDK を使用するには、次の値を指定した後、環境変数に値を設定する必要があります。 環境変数を設定するには、使用しているオペレーティング システムの表の後にある手順を参照してください。

| 値                     | 環境変数 | 説明                                                                                                                                                                                                          |
| ------------------------- | --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| テナント ID                 | `AZURE_TENANT_ID`            | Azure Stack Hub の[テナント ID](../operator/azure-stack-identity-overview.md)。                                                          |
| クライアント ID                 | `AZURE_CLIENT_ID`             | 前のセクションでサービス プリンシパルが作成されたときに保存した、サービス プリンシパル アプリケーション ID。                                                                                              |
| サブスクリプション ID           | `AZURE_SUBSCRIPTION_ID`      | [サブスクリプション ID](../operator/service-plan-offer-subscription-overview.md#subscriptions) は Azure Stack Hub 内のオファーにアクセスするために使用します。                |
| クライアント シークレット             | `AZURE_CLIENT_SECRET`        | サービス プリンシパルの作成時に保存した、サービス プリンシパル アプリケーション シークレット。                                                                                                                                   |
| Resource Manager エンドポイント | `ARM_ENDPOINT`              | 「[Azure Stack Hub Resource Manager エンドポイント](../user/azure-stack-version-profiles-ruby.md#the-azure-stack-hub-resource-manager-endpoint)」の記事を参照してください。 |
| 場所                  | `RESOURCE_LOCATION`    | Azure Stack Hub 用の**ローカル**。                                                                                                                                                                                                |

Azure Stack Hub のテナント ID を確認するには、[こちら](../operator/azure-stack-csp-ref-operations.md)の手順を参照してください。 環境変数を設定するには、以降のセクションの手順に従います。

### <a name="microsoft-windows"></a>Microsoft Windows

環境変数を設定するには、Windows コマンド プロンプトで次の形式を使用します。

```shell
Set AZURE_TENANT_ID=<Your_Tenant_ID>
```

### <a name="macos-linux-and-unix-based-systems"></a>macOS、Linux、および Unix ベースのシステム

Unix ベースのシステムでは、次のコマンドを使用します。

```shell
Export AZURE_TENANT_ID=<Your_Tenant_ID>
```

### <a name="trust-the-azure-stack-hub-ca-root-certificate"></a>Azure Stack Hub の CA ルート証明書を信頼する

Azure Stack Development Kit (ASDK) を使用する場合は、リモート マシン上で CA ルート証明書を信頼する必要があります。 Azure Stack Hub 統合システムで CA ルート証明書を信頼する必要はありません。

#### <a name="windows"></a>Windows

1. Azure Stack Hub の自己署名証明書を自分のデスクトップにエクスポートします。

1. コマンド プロンプトでディレクトリを `%JAVA_HOME%\bin` に変更します。

1. 次のコマンドを実行します。

   ```shell
   .\keytool.exe -importcert -noprompt -file <location of the exported certificate here> -alias root -keystore %JAVA_HOME%\lib\security\cacerts -trustcacerts -storepass changeit
   ```

### <a name="the-azure-stack-hub-resource-manager-endpoint"></a>Azure Stack Hub Resource Manager エンドポイント

Azure Resource Manager は、管理者が Azure リソースのデプロイ、管理、監視を行うことができる管理フレームワークです。 Azure Resource Manager では、これらのタスクを個別に処理するのではなく、グループとして単一の操作で処理することができます。

Resource Manager エンドポイントからメタデータ情報を取得できます。 エンドポイントは、コードを実行するために必要な情報と共に、JSON ファイルを返します。

次の考慮事項に注意してください。

- ASDK の **ResourceManagerUrl** は `https://management.local.azurestack.external/` です。

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
      "audiences": ["https://management.<yourtenant>.onmicrosoft.com/3cc5febd-e4b7-4a85-a2ed-1d730e2f5928"]
      }
}
```

## <a name="existing-api-profiles"></a>既存の API プロファイル

- **com.microsoft.azure.profile\_2019\_03\_01\_hybrid**: Azure Stack Hub 用に作成された最新のプロファイル。 1904 以降を使用している限り、Azure Stack Hub との互換性に優れたサービスには、このプロファイルを使用します。

- **com.microsoft.azure.profile\_2018\_03\_01\_hybrid**:Azure Stack Hub 用に作成されたプロファイル。 バージョン 1808 以降を使用する Azure Stack Hub と互換性のあるサービスには、このプロファイルを使用します。

- **com.microsoft.azure**:すべてのサービスの最新バージョンで構成されているプロンプトファイル。 すべてのサービスの最新バージョンを使用してください。

Azure Stack Hub および API プロファイルについて詳しくは、「[API プロファイルの概要](../user/azure-stack-version-profiles.md#summary-of-api-profiles)」をご覧ください。

## <a name="azure-java-sdk-api-profile-usage"></a>Azure Java SDK API プロファイルの使用

次のコードでは、Azure Stack Hub でサービス プリンシパルを認証します。 これにより、テナント ID と、Azure Stack Hub に固有の認証基準を使用して、トークンが作成されます。

```java
AzureTokenCredentials credentials = new ApplicationTokenCredentials(client, tenant, key, AZURE_STACK)
                    .withDefaultSubscriptionID(subscriptionID);
Azure azureStack = Azure.configure()
                    .withLogLevel(com.microsoft.rest.LogLevel.BASIC)
                    .authenticate(credentials, credentials.defaultSubscriptionID());
```

このコードにより、API プロファイルの依存関係を使用して、アプリを Azure Stack Hub に正常にデプロイすることができます。

## <a name="define-azure-stack-hub-environment-setting-functions"></a>Azure Stack Hub 環境設定関数の定義

適切なエンドポイントで Azure Stack Hub クラウドを登録するには、次のコードを使用します。

```java
// Get Azure Stack Hub cloud endpoints
final HashMap<String, String> settings = getActiveDirectorySettings(armEndpoint);

AzureEnvironment AZURE_STACK = new AzureEnvironment(new HashMap<String, String>() {
                {
                    put("managementEndpointUrl", settings.get("audience"));
                    put("resourceManagerEndpointUrl", armEndpoint);
                    put("galleryEndpointUrl", settings.get("galleryEndpoint"));
                    put("activeDirectoryEndpointUrl", settings.get("login_endpoint"));
                    put("activeDirectoryResourceID", settings.get("audience"));
                    put("activeDirectoryGraphResourceID", settings.get("graphEndpoint"));
                    put("storageEndpointSuffix", armEndpoint.substring(armEndpoint.indexOf('.')));
                    put("keyVaultDnsSuffix", ".vault" + armEndpoint.substring(armEndpoint.indexOf('.')));
                }
            });
```

前のコードの `getActiveDirectorySettings` 呼び出しは、メタデータ エンドポイントからエンドポイントを取得します。 これは、この呼び出しからの環境変数を示しています。

```java
public static HashMap<String, String> getActiveDirectorySettings(String armEndpoint) {

    HashMap<String, String> adSettings = new HashMap<String, String>();
    try {

        // create HTTP Client
        HttpClient httpClient = HttpClientBuilder.create().build();

        // Create new getRequest with below mentioned URL
        HttpGet getRequest = new HttpGet(String.format("%s/metadata/endpoints?api-version=1.0",
                             armEndpoint));

        // Add additional header to getRequest which accepts application/xml data
        getRequest.addHeader("accept", "application/xml");

        // Execute request and catch response
        HttpResponse response = httpClient.execute(getRequest);

        // Check for HTTP response code: 200 = success
        if (response.getStatusLine().getStatusCode() != 200) {
            throw new RuntimeException("Failed : HTTP error code : " + response.getStatusLine().getStatusCode());
        }

        String responseStr = EntityUtils.toString(response.getEntity());
        JSONObject responseJson = new JSONObject(responseStr);
        adSettings.put("galleryEndpoint", responseJson.getString("galleryEndpoint"));
        JSONObject authentication = (JSONObject) responseJson.get("authentication");
        String audience = authentication.get("audiences").toString().split("\"")[1];
        adSettings.put("login_endpoint", authentication.getString("loginEndpoint"));
        adSettings.put("audience", audience);
        adSettings.put("graphEndpoint", responseJson.getString("graphEndpoint"));

    } catch (ClientProtocolException cpe) {
        cpe.printStackTrace();
        throw new RuntimeException(cpe);
    } catch (IOException ioe) {
        ioe.printStackTrace();
        throw new RuntimeException(ioe);
    }
    return adSettings;
}
```

## <a name="samples-using-api-profiles"></a>API プロファイルを使用したサンプル

.NET と Azure Stack Hub の API プロファイルを使用してソリューションを作成する場合は、次の GitHub のサンプルを参考資料として使用します。

- [リソース グループの管理](https://github.com/Azure-Samples/Hybrid-resources-java-manage-resource-group)

- [ストレージ アカウントを管理する](https://github.com/Azure-Samples/hybrid-storage-java-manage-storage-accounts)

- [仮想マシンの管理](https://github.com/Azure-Samples/hybrid-compute-java-manage-vm) (2019-03-01-hybrid プロファイルで更新済み)

### <a name="sample-unit-test-project"></a>単体テスト プロジェクトのサンプル

1. 次のコマンドを使用して、リポジトリを複製します。

   ```shell
   git clone https://github.com/Azure-Samples/Hybrid-resources-java-manage-resource-group.git`
   ```

2. Azure サービス プリンシパルを作成し、サブスクリプションにアクセスするロールを割り当てます。 サービス プリンシパルの作成方法については、「[Azure PowerShell を使用して資格情報でのサービス プリンシパルを作成する](../operator/azure-stack-create-service-principals.md)」をご覧ください。

3. 次の必要な環境変数を取得します。

   - `AZURE_TENANT_ID`
   - `AZURE_CLIENT_ID`
   - `AZURE_CLIENT_SECRET`
   - `AZURE_SUBSCRIPTION_ID`
   - `ARM_ENDPOINT`
   - `RESOURCE_LOCATION`

4. コマンド プロンプトを使用して作成したサービス プリンシパルから取得した情報を使用して、次の環境変数を設定します。

   - `export AZURE_TENANT_ID={your tenant ID}`
   - `export AZURE_CLIENT_ID={your client ID}`
   - `export AZURE_CLIENT_SECRET={your client secret}`
   - `export AZURE_SUBSCRIPTION_ID={your subscription ID}`
   - `export ARM_ENDPOINT={your Azure Stack Hub Resource Manager URL}`
   - `export RESOURCE_LOCATION={location of Azure Stack Hub}`

   Windows では、**export** ではなく **set** を使用します。

5. Azure Resource Manager のメタデータ エンドポイントを取得するには、`getActiveDirectorySettings` の関数を使用します。

    ```java
    // Get Azure Stack Hub cloud endpoints
    final HashMap<String, String> settings = getActiveDirectorySettings(armEndpoint);
    ```

6. **Pom.xml** ファイルで、Azure Stack Hub 用の **2019-03-01-hybrid** プロファイルを使用するために次の依存関係を追加します。 この依存関係では、Compute、Networking、Storage、Key Vault、App Services のリソース プロバイダー用の、このプロファイルに関連付けられているモジュールがインストールされます。

    ```xml
    <dependency>
      <groupId>com.microsoft.azure.profile_2019_03_01_hybrid</groupId>
      <artifactId>azure</artifactId>
      <version>1.0.0-beta-1</version>
    </dependency>
    ```

7. 環境変数を設定するために開いていたコマンド プロンプトで、次のコマンドを入力します。

   ```shell
   mvn clean compile exec:java
   ```

## <a name="next-steps"></a>次のステップ

API プロファイルの詳細については、以下を参照してください。

- [Azure Stack Hub のバージョン プロファイル](azure-stack-version-profiles.md)
- [プロファイルでサポートされているリソース プロバイダー API バージョン](azure-stack-profiles-azure-resource-manager-versions.md)
