---
title: Azure Stack Hub での Ruby による API バージョンのプロファイルの使用
description: Azure Stack Hub での Ruby による API バージョンのプロファイルの使用方法について説明します。
author: sethmanheim
ms.topic: article
ms.date: 09/03/2020
ms.author: sethm
ms.reviewer: sijuman
ms.lastreviewed: 05/16/2019
ms.openlocfilehash: 6813bf18656e034688255bcdb46b9b943359ce9c
ms.sourcegitcommit: 7c01ab4b2e2250a7acd67d1c5ba27d15c1e8bce0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2020
ms.locfileid: "89448624"
---
# <a name="use-api-version-profiles-with-ruby-in-azure-stack-hub"></a>Azure Stack Hub での Ruby による API バージョンのプロファイルの使用

## <a name="ruby-and-api-version-profiles"></a>Ruby と API バージョンのプロファイル

Azure Stack Hub Resource Manager 向けの Ruby SDK には、インフラストラクチャの構築と管理に役立つツールが用意されています。 SDK のリソース プロバイダーには､Ruby 言語を使用した Compute、Virtual Network､Storage があります。 Ruby SDK に含まれている API プロファイルを使用すると、グローバルな Azure リソースと Azure Stack Hub 上のリソース間を切り替えることで、ハイブリッド クラウド開発を行うことができます。

API プロファイルは、リソース プロバイダーとサービス バージョンを組み合わせたものです。 API プロファイルを使用すると、種類の異なる複数のリソースを組み合わせることができます。

- すべてのサービスの最新バージョンを使用するには、Azure SDK ロールアップ gem の **Latest** プロファイルを使用します。
- Azure Stack Hub と互換性のあるサービスを使用するには、Azure SDK ロールアップ gem の **V2019_03_01_Hybrid** または **V2018_03_01** プロファイルを使用します。
- サービスの最新の **api-version** を使用するには、特定の gem の **Latest** プロファイルを使用します。 たとえば、Compute サービスの最新の **api-version** を単独で使用する場合は、**Compute** gem の **Latest** プロファイルを使用します。
- サービスの特定の **api-version** を使用するには、gem 内で定義された特定の API バージョンを使用します。

> [!NOTE]
> すべてのオプションを同じアプリ内で組み合わせることができます。

## <a name="install-the-azure-ruby-sdk"></a>Azure Ruby SDK のインストール

- 正式な指示に従って、[Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) をインストールします。
- 正式な指示に従って、[Ruby](https://www.ruby-lang.org/en/documentation/installation/) をインストールします。
  - インストール時には、**PATH 変数に Ruby を追加する**ためのオプションを選択します。
  - Ruby のインストール時にプロンプトが表示された場合は、開発キットをインストールします。
  - 次に、下記のコマンドを使用して bundler をインストールします。 

       ```ruby
       Gem install bundler
       ```

- 使用できない場合は、サブスクリプションを作成し、サブスクリプション ID を保存して後で使用します。 サブスクリプションの作成手順については、「[Azure Stack Hub でオファーのサブスクリプションを作成する](../operator/azure-stack-subscribe-plan-provision-vm.md)」の記事で説明しています。
- サービス プリンシパルを作成し、その ID とシークレットを保存します。 Azure Stack Hub 向けサービス プリンシパルの作成手順については、「[アプリ ID を使用してリソースにアクセスする](../operator/azure-stack-create-service-principals.md)」の記事で説明しています。
- サブスクリプションでサービス プリンシパルに共同作成者/所有者ロールが割り当てられていることを確認します。 サービス プリンシパルへのロールの割り当て方法に関する手順は、「[アプリ ID を使用してリソースにアクセスする](../operator/azure-stack-create-service-principals.md)」の記事で説明されています。

## <a name="install-the-rubygem-packages"></a>RubyGem パッケージをインストールする

Azure RubyGem パッケージは直接インストールできます。

```ruby  
gem install azure_mgmt_compute
gem install azure_mgmt_storage
gem install azure_mgmt_resources
gem install azure_mgmt_network
```

または、Gemfile でそれらを使用します。

```ruby
gem 'azure_mgmt_storage'
gem 'azure_mgmt_compute'
gem 'azure_mgmt_resources'
gem 'azure_mgmt_network'
```

Azure Resource Manager Ruby SDK は現在プレビュー段階であり、今後のリリースで重大なインターフェイス変更が加えられる見込みです。 マイナー バージョンの番号が上がっただけでも、重大な変更を示している可能性があります。

## <a name="use-the-azure_sdk-gem"></a>azure_sdk gem を使用する

**azure_sdk** gem は、Ruby SDK でサポートされているすべての gem をまとめたロールアップです。 この gem は、すべてのサービスの最新バージョンをサポートする、 **Latest**  プロファイルで構成されています。 これには、Azure Stack Hub 用に構築された、バージョン管理されたプロファイル  **V2017_03_09** および **V2019_03_01_Hybrid** が含まれています。

azure_sdk ロールアップ gem は、次のコマンドを使用してインストールできます。  

```ruby  
gem install 'azure_sdk'
```

## <a name="prerequisites"></a>前提条件

Azure Stack Hub で Ruby Azure SDK を使用するには、次の値を指定した後、環境変数で値を設定する必要があります。 環境変数を設定するには、使用している特定のオペレーティング システムの表の後にある手順を参照してください。

| 値 | 環境変数 | 説明 |
| --- | --- | --- |
| テナント ID | `AZURE_TENANT_ID` | Azure Stack Hub の[テナント ID](../operator/azure-stack-identity-overview.md)。 |
| クライアント ID | `AZURE_CLIENT_ID` | この記事の前のセクションでサービス プリンシパルが作成されたときに保存した、サービス プリンシパル アプリ ID。  |
| サブスクリプション ID | `AZURE_SUBSCRIPTION_ID` | [サブスクリプション ID](../operator/service-plan-offer-subscription-overview.md#subscriptions) は Azure Stack Hub 内のオファーにアクセスするために使用します。 |
| クライアント シークレット | `AZURE_CLIENT_SECRET` | サービス プリンシパルの作成時に保存した、サービス プリンシパル アプリ シークレット。 |
| Resource Manager エンドポイント | `ARM_ENDPOINT` | [Azure Stack Hub Resource Manager エンドポイント](#the-azure-stack-hub-resource-manager-endpoint) に関する記事を参照してください。  |

### <a name="the-azure-stack-hub-resource-manager-endpoint"></a>Azure Stack Hub Resource Manager エンドポイント

Microsoft Azure Resource Manager は、管理者が Azure リソースのデプロイ、管理、監視を行うことができる管理フレームワークです。 Azure Resource Manager では、これらのタスクを個別に処理するのではなく、グループとして単一の操作で処理することができます。

Resource Manager エンドポイントからメタデータ情報を取得できます。 エンドポイントは、コードを実行するために必要な情報と共に、JSON ファイルを返します。

 > [!NOTE]  
 > Azure Stack Development Kit (ASDK) の **ResourceManagerUrl** は`https://management.local.azurestack.external/` です。統合システムの **ResourceManagerUrl** は `https://management.region.<fqdn>/` です。ここで、`<fqdn>` は完全修飾ドメイン名です。  
 > 必要なメタデータを取得するには、`<ResourceManagerUrl>/metadata/endpoints?api-version=1.0` になります。
  
 サンプルの JSON ファイルは、次のようになります。

 ```json
 {
   "galleryEndpoint": "https://portal.local.azurestack.external:30015/",  
   "graphEndpoint": "https://graph.windows.net/",  
   "portal Endpoint": "https://portal.local.azurestack.external/",
   "authentication": {
     "loginEndpoint": "https://login.windows.net/",
     "audiences": ["https://management.<yourtenant>.onmicrosoft.com/3cc5febd-e4b7-4a85-a2ed-1d730e2f5928"]
   }
 }
```

### <a name="set-environment-variables"></a>環境変数の設定

#### <a name="microsoft-windows"></a>Microsoft Windows

環境変数を設定するには、Windows コマンド プロンプトで次の形式を使用します。

```console
set AZURE_TENANT_ID=<YOUR_TENANT_ID>
```

#### <a name="macos-linux-and-unix-based-systems"></a>macOS、Linux、および Unix ベースのシステム

Unix ベースのシステムでは、次のコマンドを使用します。

```bash
export AZURE_TENANT_ID=<YOUR_TENANT_ID>
```

## <a name="existing-api-profiles"></a>既存の API プロファイル

**Azure_sdk** ロールアップ gem には、次の 3 つのプロファイルがあります。

- **V2019_03_01_Hybrid**:Azure Stack Hub 用に作成されたプロファイル。 このプロファイルは、Azure Stack Hub バージョン 1904 以降で使用可能なすべてのサービスの最新バージョンを使用する場合に使用します。
- **V2017_03_09**:Azure Stack Hub 用に作成されたプロファイル。 このプロファイルは、Azure Stack Hub バージョン 1808 以前との互換性に優れたサービスに使用します。
- **Latest**:このプロファイルは、すべてのサービスの最新バージョンで構成されます。 すべてのサービスの最新バージョンを使用してください。

Azure Stack Hub および API プロファイルについて詳しくは、「[API プロファイルの概要](azure-stack-version-profiles.md#summary-of-api-profiles)」をご覧ください。

## <a name="azure-ruby-sdk-api-profile-usage"></a>Azure Ruby SDK API プロファイルの使用

次のコードを使用して、プロファイル クライアントをインスタンス化します。 このパラメーターは、Azure Stack Hub またはまたはその他のプライベート クラウドにのみ必要です。 グローバル Azure では、既定でこれらの設定が使用されます。

```ruby  
active_directory_settings = get_active_directory_settings(ENV['ARM_ENDPOINT'])

provider = MsRestAzure::ApplicationTokenProvider.new(
  ENV['AZURE_TENANT_ID'],
  ENV['AZURE_CLIENT_ID'],
  ENV['AZURE_CLIENT_SECRET'],
  active_directory_settings
)
credentials = MsRest::TokenCredentials.new(provider)
options = {
  credentials: credentials,
  subscription_id: subscription_id,
  active_directory_settings: active_directory_settings,
  base_url: ENV['ARM_ENDPOINT']
}

# Target profile built for Azure Stack Hub
client = Azure::Resources::Profiles::V2019_03_01_Hybrid::Mgmt::Client.new(options)
```

プロファイル クライアントは、個別のリソース プロバイダー (Compute、Storage、Network など) にアクセスするために使用できます。

```ruby  
# To access the operations associated with Compute
profile_client.compute.virtual_machines.get 'RESOURCE_GROUP_NAME', 'VIRTUAL_MACHINE_NAME'

# Option 1: To access the models associated with Compute
purchase_plan_obj = profile_client.compute.model_classes.purchase_plan.new

# Option 2: To access the models associated with Compute
# Notice Namespace: Azure::Profiles::<Profile Name>::<Service Name>::Mgmt::Models::<Model Name>
purchase_plan_obj = Azure::Profiles::V2019_03_01_Hybrid::Compute::Mgmt::Models::PurchasePlan.new
```

## <a name="define-azure-stack-hub-environment-setting-functions"></a>Azure Stack Hub 環境設定関数の定義

サービス プリンシパルを Azure Stack Hub 環境に対して認証するには、`get_active_directory_settings()` を使用してエンドポイントを定義します。 このメソッドでは、前に設定した **ARM_Endpoint**環境変数を使用します。

```ruby  
# Get Authentication endpoints using Arm Metadata Endpoints
def get_active_directory_settings(armEndpoint)
  settings = MsRestAzure::ActiveDirectoryServiceSettings.new
  response = Net::HTTP.get_response(URI("#{armEndpoint}/metadata/endpoints?api-version=1.0"))
  status_code = response.code
  response_content = response.body
  unless status_code == "200"
    error_model = JSON.load(response_content)
    fail MsRestAzure::AzureOperationError.new("Getting Azure Stack Hub Metadata Endpoints", response, error_model)
  end
  result = JSON.load(response_content)
  settings.authentication_endpoint = result['authentication']['loginEndpoint'] unless result['authentication']['loginEndpoint'].nil?
  settings.token_audience = result['authentication']['audiences'][0] unless result['authentication']['audiences'][0].nil?
  settings
end
```

## <a name="samples-using-api-profiles"></a>API プロファイルを使用したサンプル

Ruby と Azure Stack Hub の API のプロファイルを使用してソリューションを作成する場合は、GitHub の次のサンプルを参考資料として使用します。

- [Azure のリソースとリソース グループを Ruby で管理する](https://github.com/Azure-Samples/Hybrid-Resource-Manager-Ruby-Resources-And-Groups)。
- [Ruby を使用して仮想マシンを管理する](https://github.com/Azure-Samples/Hybrid-Compute-Ruby-Manage-VM) (Azure Stack Hub でサポートされている最新の API バージョンをターゲットにするために 2019-03-01-hybrid プロファイルを使用するサンプル)。
- [Ruby からテンプレートを使用して SSH 対応 VM をデプロイする](https://github.com/Azure-Samples/Hybrid-Resource-Manager-Ruby-Template-Deployment)。

### <a name="sample-resource-manager-and-groups"></a>Resource Manager とグループのサンプル

このサンプルを実行するには、Ruby がインストールされていることを確認してください。 Visual Studio Code を使用している場合は、Ruby SDK 拡張機能もダウンロードしてください。

> [!NOTE]  
> サンプルのリポジトリは、[Hybrid-Resource-Manager-Ruby-Resources-And-Groups](https://github.com/Azure-Samples/Hybrid-Resource-Manager-Ruby-Resources-And-Groups)です。

1. リポジトリを複製します。

   ```console
   git clone https://github.com/Azure-Samples/Hybrid-Resource-Manager-Ruby-Resources-And-Groups.git
   ```

2. bundle を使用して依存関係をインストールします。

   ```console
   cd Hybrid-Resource-Manager-Ruby-Resources-And-Groups
   bundle install
   ```

3. PowerShell を使用して Azure サービス プリンシパルを作成し、必要な値を取得します。

   サービス プリンシパルの作成方法については、「[Azure PowerShell を使用して資格情報でのサービス プリンシパルを作成する](../operator/azure-stack-create-service-principals.md)」をご覧ください。

   必要な値は次のとおりです。

   - テナント ID
   - クライアント ID
   - クライアント シークレット
   - サブスクリプション ID
   - Resource Manager エンドポイント

   作成したサービス プリンシパルから取得した情報を使用して、次の環境変数を設定します。

   - `export AZURE_TENANT_ID={your tenant ID}`
   - `export AZURE_CLIENT_ID={your client ID}`
   - `export AZURE_CLIENT_SECRET={your client secret}`
   - `export AZURE_SUBSCRIPTION_ID={your subscription ID}`
   - `export ARM_ENDPOINT={your Azure Stack Hub Resource Manager URL}`

   > [!NOTE]  
   > Windows では、`export`ではなく、`set`を使用します。

4. location 変数が、たとえば `LOCAL="local"` のように確実に Azure Stack Hub の場所に設定されているようにします。

5. Azure Stack Hub またはその他のプライベート クラウドを使用している場合は、次のコード行を追加して、適切な Active Directory エンドポイントをターゲットにします。

   ```ruby  
   active_directory_settings = get_active_directory_settings(ENV['ARM_ENDPOINT'])
   ```

6. `options` 変数内で、Azure Stack Hub と連携するための Active Directory 設定とベース URL を追加します。

   ```ruby  
   options = {
   credentials: credentials,
   subscription_id: subscription_id,
   active_directory_settings: active_directory_settings,
   base_url: ENV['ARM_ENDPOINT']
   }
   ```

7. Azure Stack Hub プロファイルをターゲットとするプロファイル クライアントを作成します。

   ```ruby  
   client = Azure::Resources::Profiles::V2019_03_01_Hybrid::Mgmt::Client.new(options)
   ```

8. サービス プリンシパルを Azure Stack Hub に対して認証するには、**get_active_directory_settings()** を使用してエンドポイントを定義します。 このメソッドでは、前に設定した **ARM_Endpoint**環境変数を使用します。

   ```ruby  
   def get_active_directory_settings(armEndpoint)
     settings = MsRestAzure::ActiveDirectoryServiceSettings.new
     response = Net::HTTP.get_response(URI("#{armEndpoint}/metadata/endpoints?api-version=1.0"))
     status_code = response.code
     response_content = response.body
     unless status_code == "200"
       error_model = JSON.load(response_content)
       fail MsRestAzure::AzureOperationError.new("Getting Azure Stack Hub Metadata Endpoints", response, error_model)
     end
     result = JSON.load(response_content)
     settings.authentication_endpoint = result['authentication']['loginEndpoint'] unless result['authentication']['loginEndpoint'].nil?
     settings.token_audience = result['authentication']['audiences'][0] unless result['authentication']['audiences'][0].nil?
     settings
   end
   ```

9. サンプルを実行します。

   ```ruby
   bundle exec ruby example.rb
   ```

## <a name="next-steps"></a>次のステップ

- [PowerShell for Azure Stack Hub をインストールする](../operator/azure-stack-powershell-install.md)
- [Azure Stack Hub ユーザーの PowerShell 環境の構成](azure-stack-powershell-configure-user.md)  
