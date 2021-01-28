---
title: Azure CLI を使用して Azure Stack Hub を管理する
description: クロスプラットフォーム コマンドライン インターフェイス (CLI) を使用して、Azure Stack Hub でリソースを管理およびデプロイする方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 12/16/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 12/16/2020
ms.openlocfilehash: 8ffab5fa1ffff5012edfbce3da69f35a753ff72e
ms.sourcegitcommit: e772df8ac78c86d834a68d1a8be83b7f738019b7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/26/2021
ms.locfileid: "98772291"
---
# <a name="install-azure-cli-on-azure-stack-hub"></a>Azure CLI を Azure Stack Hub にインストールする

Azure CLI をインストールすると、Windows マシンまたは Linux マシンで Azure Stack Hub を管理することができます。 この記事では、Azure CLI のインストールと設定の手順について説明します。

## <a name="install-azure-cli"></a>Azure CLI のインストール

1. 開発ワークステーションにサインインし、CLI をインストールします。 Azure Stack Hub には、Azure CLI のバージョン 2.0 以降が必要です。 

2. CLI は、「[Azure CLI のインストール](/cli/azure/install-azure-cli)」で説明されている手順を使用してインストールできます。 

3. インストールが正常に完了したことを確認するには、ターミナルまたはコマンド プロンプト ウィンドウを開いて次のコマンドを実行します。

    ```shell
    az --version
    ```

    お使いのコンピューターにインストールされている Azure CLI と依存するその他のライブラリのバージョンが表示されます。

    ![Azure Stack Hub 上の Azure CLI の Python の場所](media/azure-stack-version-profiles-azurecli2/cli-python-location.png)

2. CLI の Python の場所を書き留めておきます。 ASDK を実行する場合には、この場所を使用して証明書を追加する必要があります。 ASDK で CLI をインストールするための証明書を設定する手順については、「[Azure Stack Development Kit での Azure CLI 用の証明書の設定](../asdk/asdk-cli.md)」を参照してください。

## <a name="connect-with-azure-cli"></a>Azure CLI との接続

### <a name="azure-ad-on-windows"></a>[Azure AD と Windows](#tab/ad-win)

このセクションでは、ID 管理サービスとして Azure AD を使用していて、Windows マシン上で CLI を使用する場合の CLI の設定について説明します。

#### <a name="connect-to-azure-stack-hub"></a>Azure Stack Hub に接続する

1. ASDK を使用している場合、Azure Stack Hub の CA ルート証明書を信頼します。 手順については、「[証明書を信頼する](../asdk/asdk-cli.md#trust-the-certificate)」を参照してください。

2. `az cloud register` コマンドを実行して Azure Stack Hub 環境を登録します。

3. お客様の環境を登録します。 `az cloud register` を実行するときに、次のパラメーターを使用します。

      | 値 | 例 | 説明 |
      | --- | --- | --- |
      | 環境名 | AzureStackUser | ユーザー環境の場合は、`AzureStackUser` を使用します。 オペレーターの場合は、`AzureStackAdmin` を指定します。 |
      | Resource Manager エンドポイント | `https://management.contoso.onmicrosoft.com` | ASDK の **ResourceManagerUrl** は次のとおりです。`https://management.local.azurestack.external/` になります。統合システムの **ResourceManagerUrl** は`https://management.<region>.<fqdn>/` になります。統合システム エンドポイントに関する質問がある場合は、お客様のクラウド オペレーターにお問い合わせください。 |
      | ストレージ エンドポイント | local.contoso.onmicrosoft.com | `local.azurestack.external` は、ASDK の場合です。 統合システムの場合は、システムのエンドポイントを使用します。  |
      | Keyvault のサフィックス | .vault.contoso.onmicrosoft.com | `.vault.local.azurestack.external` は、ASDK の場合です。 統合システムの場合は、システムのエンドポイントを使用します。  |
      | エンドポイント Azure Active Directory Graph リソース ID | https://graph.windows.net/ | Active Directory リソース ID。 |
    
      ```azurecli  
      az cloud register `
          -n <environmentname> `
          --endpoint-resource-manager "https://management.<region>.<fqdn>" `
          --suffix-storage-endpoint "<fqdn>" `
          --suffix-keyvault-dns ".vault.<fqdn>" `
          --endpoint-active-directory-graph-resource-id "https://graph.windows.net/"
      ```

    [register コマンド](/cli/azure/cloud?view=azure-cli-latest#az_cloud_register)のリファレンスは、Azure CLI リファレンス ドキュメントにあります。


4. 次のコマンドを使用して、アクティブな環境を設定します。

      ```azurecli
      az cloud set -n <environmentname>
      ```

5. Azure Stack Hub 固有の API バージョンのプロファイルを使用するようにお使いの環境の構成を更新します。 構成を更新するには、次のコマンドを実行します。

    ```azurecli
    az cloud update --profile 2019-03-01-hybrid
   ```
 
6. `az login` コマンドを使用して Azure Stack Hub 環境にサインインします。

    Azure Stack Hub 環境にサインインするには、ユーザー資格情報を使用するか、クラウド オペレーターによって提供される[サービス プリンシパル](/azure/active-directory/develop/app-objects-and-service-principals) (SPN) を使用します。 

   - *ユーザー* としてサインインする場合: 

     `az login` コマンド内で直接ユーザー名とパスワードを指定するか、ブラウザーを使用して認証できます。 多要素認証が有効になっているアカウントの場合は、後者を実行する必要があります。

     ```azurecli
     az login -u "user@contoso.onmicrosoft.com" -p 'Password123!' --tenant contoso.onmicrosoft.com
     ```

     > [!NOTE]
     > お使いのユーザー アカウントで多要素認証が有効になっている場合は、`-u` パラメーターを指定しないで、`az login` コマンドを使用します。 このコマンドを実行すると、認証で使用する必要がある URL とコードを取得できます。

   - *サービス プリンシパル* を使ってサインインする｡ 
    
        サインインする前に、CLI または [Azure Portal でサービス プリンシパルを作成](../operator/azure-stack-create-service-principals.md)してロールに割り当てます。 次のコマンドを使用してサインインします。
    
        ```azurecli  
        az login `
          --tenant <Azure Active Directory Tenant name. `
                    For example: myazurestack.onmicrosoft.com> `
        --service-principal `
          -u <Application Id of the Service Principal> `
          -p <Key generated for the Service Principal>
        ```
    
7. 環境が正しく設定されていること、環境がアクティブなクラウドであることを確認します。

      ```azurecli
          az cloud list --output table
      ```

環境が一覧表示され、**IsActive** が `true` になっていることを確認します。 例:

```azurecli  
IsActive    Name               Profile
----------  -----------------  -----------------
False       AzureCloud         2019-03-01-hybrid
False       AzureChinaCloud    latest
False       AzureUSGovernment  latest
False       AzureGermanCloud   latest
True        AzureStackUser     2019-03-01-hybrid
```

#### <a name="test-the-connectivity"></a>接続のテスト

すべての設定が完了したら、CLI を使用し、Azure Stack Hub 内でリソースを作成します。 たとえば、アプリのリソース グループを作成し、VM を追加できます。 次のコマンドを使用して、"MyResourceGroup" という名前のリソース グループを作成します。

```azurecli
az group create -n MyResourceGroup -l local
```

リソース グループが正常に作成されると、上記のコマンドによって、新たに作成されたリソースの次のプロパティが出力されます。

![リソース グループ作成の出力](media/azure-stack-connect-cli/image1.png)

### <a name="ad-fs-on-windows"></a>[AD FS と Windows](#tab/adfs-win)

このセクションでは、ID 管理サービスとして Active Directory フェデレーション サービス (AD FS) を使用していて、Windows マシン上で CLI を使用する場合の CLI の設定について説明します。

#### <a name="connect-to-azure-stack-hub"></a>Azure Stack Hub に接続する


1. ASDK を使用している場合、Azure Stack Hub の CA ルート証明書を信頼します。 手順については、「[証明書を信頼する](../asdk/asdk-cli.md#trust-the-certificate)」を参照してください。

2. `az cloud register` コマンドを実行して Azure Stack Hub 環境を登録します。

3. お客様の環境を登録します。 `az cloud register` を実行するときに、次のパラメーターを使用します。

    | 値 | 例 | 説明 |
    | --- | --- | --- |
    | 環境名 | AzureStackUser | ユーザー環境の場合は、`AzureStackUser` を使用します。 オペレーターの場合は、`AzureStackAdmin` を指定します。 |
    | Resource Manager エンドポイント | `https://management.local.azurestack.external` | ASDK の **ResourceManagerUrl** は次のとおりです。`https://management.local.azurestack.external/` になります。統合システムの **ResourceManagerUrl** は`https://management.<region>.<fqdn>/` になります。統合システム エンドポイントに関する質問がある場合は、お客様のクラウド オペレーターにお問い合わせください。 |
    | ストレージ エンドポイント | local.azurestack.external | `local.azurestack.external` は、ASDK の場合です。 統合システムの場合は、システムのエンドポイントを使用します。  |
    | Keyvault のサフィックス | .vault.local.azurestack.external | `.vault.local.azurestack.external` は、ASDK の場合です。 統合システムの場合は、システムのエンドポイントを使用します。  |
    | VM イメージのエイリアスのドキュメント エンドポイント | https://raw.githubusercontent.com/Azure/azure-rest-api-specs/master/arm-compute/quickstart-templates/aliases.json | VM イメージのエイリアスを含むドキュメントの URI。 詳細については、「[仮想マシンのエイリアス エンドポイントを設定する](../asdk/asdk-cli.md#set-up-the-virtual-machine-alias-endpoint)」を参照してください。 |

    ```azurecli  
    az cloud register -n <environmentname> --endpoint-resource-manager "https://management.local.azurestack.external" --suffix-storage-endpoint "local.azurestack.external" --suffix-keyvault-dns ".vault.local.azurestack.external" --endpoint-vm-image-alias-doc <URI of the document which contains VM image aliases>
    ```

4. 次のコマンドを使用して、アクティブな環境を設定します。

      ```azurecli
      az cloud set -n <environmentname>
      ```

5. Azure Stack Hub 固有の API バージョンのプロファイルを使用するようにお使いの環境の構成を更新します。 構成を更新するには、次のコマンドを実行します。

    ```azurecli
    az cloud update --profile 2019-03-01-hybrid
   ```

    >[!NOTE]  
    >1808 ビルドより前のバージョンの Azure Stack Hub を実行している場合は、**2019-03-01-hybrid** の API バージョンのプロファイルではなく、**2017-03-09-profile** の API バージョンのプロファイルを使用する必要があります。 また、Azure CLI の最新バージョンを使用する必要もあります。

6. `az login` コマンドを使用して Azure Stack Hub 環境にサインインします。 Azure Stack Hub 環境には、ユーザーまたは[サービス プリンシパル](/azure/active-directory/develop/app-objects-and-service-principals)としてサインインできます。 

   - *ユーザー* としてサインインする場合:

     `az login` コマンド内で直接ユーザー名とパスワードを指定するか、ブラウザーを使用して認証できます。 多要素認証が有効になっているアカウントの場合は、後者を実行する必要があります。

     ```azurecli
     az cloud register  -n <environmentname>   --endpoint-resource-manager "https://management.local.azurestack.external"  --suffix-storage-endpoint "local.azurestack.external" --suffix-keyvault-dns ".vault.local.azurestack.external" --endpoint-vm-image-alias-doc <URI of the document which contains VM image aliases>   --profile "2019-03-01-hybrid"
     ```

     > [!NOTE]
     > お使いのユーザー アカウントで多要素認証が有効になっている場合は、`-u` パラメーターを指定しないで、`az login` コマンドを使用します。 このコマンドを実行すると、認証で使用する必要がある URL とコードを取得できます。

   - *サービス プリンシパル* を使ってサインインする｡ 
    
     サービス プリンシパルのログインに使用する .pem ファイルを用意します。

     プリンシパルが作成されたクライアント マシン上で、サービス プリンシパル証明書を、`cert:\CurrentUser\My` にある秘密キーを持つ pfx としてエクスポートします。 証明書名にはプリンシパルと同じ名前が付けられています。

     pfx を pem に変換します (OpenSSL ユーティリティ を使用)。

     CLI にサインインします。
  
     ```azurecli  
     az login --service-principal \
      -u <Client ID from the Service Principal details> \
      -p <Certificate's fully qualified name, such as, C:\certs\spn.pem>
      --tenant <Tenant ID> \
      --debug 
     ```

#### <a name="test-the-connectivity"></a>接続のテスト

すべての設定が完了したら、CLI を使用し、Azure Stack Hub 内でリソースを作成します。 たとえば、アプリのリソース グループを作成し、VM を追加できます。 次のコマンドを使用して、"MyResourceGroup" という名前のリソース グループを作成します。

```azurecli
az group create -n MyResourceGroup -l local
```

リソース グループが正常に作成されると、上記のコマンドによって、新たに作成されたリソースの次のプロパティが出力されます。

![リソース グループ作成の出力](media/azure-stack-connect-cli/image1.png)

### <a name="azure-ad-on-linux"></a>[Azure AD と Linux](#tab/ad-lin)

このセクションでは、ID 管理サービスとして Azure AD を使用していて、Linux マシン上で CLI を使用する場合の CLI の設定について説明します。

#### <a name="connect-to-azure-stack-hub"></a>Azure Stack Hub に接続する

次の手順を使用して Azure Stack Hub に接続します。


1. ASDK を使用している場合、Azure Stack Hub の CA ルート証明書を信頼します。 手順については、「[証明書を信頼する](../asdk/asdk-cli.md#trust-the-certificate)」を参照してください。

2. `az cloud register` コマンドを実行して Azure Stack Hub 環境を登録します。

3. お客様の環境を登録します。 `az cloud register` を実行するときに、次のパラメーターを使用します。

    | 値 | 例 | 説明 |
    | --- | --- | --- |
    | 環境名 | AzureStackUser | ユーザー環境の場合は、`AzureStackUser` を使用します。 オペレーターの場合は、`AzureStackAdmin` を指定します。 |
    | Resource Manager エンドポイント | `https://management.local.azurestack.external` | ASDK の **ResourceManagerUrl** は次のとおりです。`https://management.local.azurestack.external/` になります。統合システムの **ResourceManagerUrl** は`https://management.<region>.<fqdn>/` になります。統合システム エンドポイントに関する質問がある場合は、お客様のクラウド オペレーターにお問い合わせください。 |
    | ストレージ エンドポイント | local.azurestack.external | `local.azurestack.external` は、ASDK の場合です。 統合システムの場合は、システムのエンドポイントを使用します。  |
    | Keyvault のサフィックス | .vault.local.azurestack.external | `.vault.local.azurestack.external` は、ASDK の場合です。 統合システムの場合は、システムのエンドポイントを使用します。  |
    | VM イメージのエイリアスのドキュメント エンドポイント | https://raw.githubusercontent.com/Azure/azure-rest-api-specs/master/arm-compute/quickstart-templates/aliases.json | VM イメージのエイリアスを含むドキュメントの URI。 詳細については、「[仮想マシンのエイリアス エンドポイントを設定する](../asdk/asdk-cli.md#set-up-the-virtual-machine-alias-endpoint)」を参照してください。 |

    ```azurecli  
    az cloud register -n <environmentname> --endpoint-resource-manager "https://management.local.azurestack.external" --suffix-storage-endpoint "local.azurestack.external" --suffix-keyvault-dns ".vault.local.azurestack.external" --endpoint-vm-image-alias-doc <URI of the document which contains VM image aliases>
    ```

4. アクティブな環境を設定します。 

      ```azurecli
        az cloud set -n <environmentname>
      ```

5. Azure Stack Hub 固有の API バージョンのプロファイルを使用するようにお使いの環境の構成を更新します。 構成を更新するには、次のコマンドを実行します。

    ```azurecli
      az cloud update --profile 2019-03-01-hybrid
   ```

    >[!NOTE]  
    >1808 ビルドより前のバージョンの Azure Stack Hub を実行している場合は、**2019-03-01-hybrid** の API バージョンのプロファイルではなく、**2017-03-09-profile** の API バージョンのプロファイルを使用する必要があります。 また、Azure CLI の最新バージョンを使用する必要もあります。

6. `az login` コマンドを使用して Azure Stack Hub 環境にサインインします。 Azure Stack Hub 環境には、ユーザーまたは[サービス プリンシパル](/azure/active-directory/develop/app-objects-and-service-principals)としてサインインできます。 

   * *ユーザー* としてサインインする場合:

     `az login` コマンド内で直接ユーザー名とパスワードを指定するか、ブラウザーを使用して認証できます。 多要素認証が有効になっているアカウントの場合は、後者を実行する必要があります。

     ```azurecli
     az login \
       -u <Active directory global administrator or user account. For example: username@<aadtenant>.onmicrosoft.com> \
       --tenant <Azure Active Directory Tenant name. For example: myazurestack.onmicrosoft.com>
     ```

     > [!NOTE]
     > お使いのユーザー アカウントで多要素認証が有効になっている場合は、`-u` パラメーターを指定しないで、`az login` コマンドを使用できます。 このコマンドを実行すると、認証で使用する必要がある URL とコードを取得できます。
   
   * *サービス プリンシパル* としてサインインする場合:
    
     サインインする前に、CLI または [Azure Portal でサービス プリンシパルを作成](../operator/azure-stack-create-service-principals.md)してロールに割り当てます。 次のコマンドを使用してサインインします。

     ```azurecli  
     az login \
       --tenant <Azure Active Directory Tenant name. For example: myazurestack.onmicrosoft.com> \
       --service-principal \
       -u <Application Id of the Service Principal> \
       -p <Key generated for the Service Principal>
     ```

#### <a name="test-the-connectivity"></a>接続のテスト

すべての設定が完了したら、CLI を使用し、Azure Stack Hub 内でリソースを作成します。 たとえば、アプリのリソース グループを作成し、VM を追加できます。 次のコマンドを使用して、"MyResourceGroup" という名前のリソース グループを作成します。

```azurecli
    az group create -n MyResourceGroup -l local
```

リソース グループが正常に作成されると、上記のコマンドによって、新たに作成されたリソースの次のプロパティが出力されます。

![リソース グループ作成の出力](media/azure-stack-connect-cli/image1.png)

### <a name="ad-fs-linux"></a>[AD FS と Linux](#tab/adfs-lin)

このセクションでは、管理サービスとして Active Directory フェデレーション サービス (AD FS) を使用していて、Linux マシン上で CLI を使用する場合の CLI の設定について説明します。

#### <a name="connect-to-azure-stack-hub"></a>Azure Stack Hub に接続する

次の手順を使用して Azure Stack Hub に接続します。

1. ASDK を使用している場合、Azure Stack Hub の CA ルート証明書を信頼します。 手順については、「[証明書を信頼する](../asdk/asdk-cli.md#trust-the-certificate)」を参照してください。

2. `az cloud register` コマンドを実行して Azure Stack Hub 環境を登録します。

3. お客様の環境を登録します。 `az cloud register` を実行するときに、次のパラメーターを使用します。

    | 値 | 例 | 説明 |
    | --- | --- | --- |
    | 環境名 | AzureStackUser | ユーザー環境の場合は、`AzureStackUser` を使用します。 オペレーターの場合は、`AzureStackAdmin` を指定します。 |
    | Resource Manager エンドポイント | `https://management.local.azurestack.external` | ASDK の **ResourceManagerUrl** は次のとおりです。`https://management.local.azurestack.external/` になります。統合システムの **ResourceManagerUrl** は`https://management.<region>.<fqdn>/` になります。統合システム エンドポイントに関する質問がある場合は、お客様のクラウド オペレーターにお問い合わせください。 |
    | ストレージ エンドポイント | local.azurestack.external | `local.azurestack.external` は、ASDK の場合です。 統合システムの場合は、システムのエンドポイントを使用します。  |
    | Keyvault のサフィックス | .vault.local.azurestack.external | `.vault.local.azurestack.external` は、ASDK の場合です。 統合システムの場合は、システムのエンドポイントを使用します。  |
    | VM イメージのエイリアスのドキュメント エンドポイント | https://raw.githubusercontent.com/Azure/azure-rest-api-specs/master/arm-compute/quickstart-templates/aliases.json | VM イメージのエイリアスを含むドキュメントの URI。 詳細については、「[仮想マシンのエイリアス エンドポイントを設定する](../asdk/asdk-cli.md#set-up-the-virtual-machine-alias-endpoint)」を参照してください。 |

    ```azurecli  
    az cloud register -n <environmentname> --endpoint-resource-manager "https://management.local.azurestack.external" --suffix-storage-endpoint "local.azurestack.external" --suffix-keyvault-dns ".vault.local.azurestack.external" --endpoint-vm-image-alias-doc <URI of the document which contains VM image aliases>
    ```

4. アクティブな環境を設定します。 

      ```azurecli
        az cloud set -n <environmentname>
      ```

5. Azure Stack Hub 固有の API バージョンのプロファイルを使用するようにお使いの環境の構成を更新します。 構成を更新するには、次のコマンドを実行します。

    ```azurecli
      az cloud update --profile 2019-03-01-hybrid
   ```

    >[!NOTE]  
    >1808 ビルドより前のバージョンの Azure Stack Hub を実行している場合は、**2019-03-01-hybrid** の API バージョンのプロファイルではなく、**2017-03-09-profile** の API バージョンのプロファイルを使用する必要があります。 また、Azure CLI の最新バージョンを使用する必要もあります。

6. `az login` コマンドを使用して Azure Stack Hub 環境にサインインします。 Azure Stack Hub 環境には、ユーザーまたは[サービス プリンシパル](/azure/active-directory/develop/app-objects-and-service-principals)としてサインインできます。 

7. サインインします。 

   *  Web ブラウザーとデバイス コードを使用して **ユーザー** として:  

   ```azurecli  
    az login --use-device-code
   ```

   > [!NOTE]  
   >コマンドを実行すると、認証で使用する必要がある URL とコードを取得できます。

   * サービス プリンシパルとして:
        
     サービス プリンシパルのログインに使用する .pem ファイルを用意します。

      * プリンシパルが作成されたクライアント マシン上で、サービス プリンシパル証明書を、`cert:\CurrentUser\My` にある秘密キーを持つ pfx としてエクスポートします。 証明書名にはプリンシパルと同じ名前が付けられています。
  
      * pfx を pem に変換します (OpenSSL ユーティリティ を使用)。

     CLI にサインインします。

      ```azurecli  
      az login --service-principal \
        -u <Client ID from the Service Principal details> \
        -p <Certificate's fully qualified name, such as, C:\certs\spn.pem>
        --tenant <Tenant ID> \
        --debug 
      ```

#### <a name="test-the-connectivity"></a>接続のテスト

すべての設定が完了したら、CLI を使用し、Azure Stack Hub 内でリソースを作成します。 たとえば、アプリのリソース グループを作成し、VM を追加できます。 次のコマンドを使用して、"MyResourceGroup" という名前のリソース グループを作成します。

```azurecli
  az group create -n MyResourceGroup -l local
```

リソース グループが正常に作成されると、上記のコマンドによって、新たに作成されたリソースの次のプロパティが出力されます。

![リソース グループ作成の出力](media/azure-stack-connect-cli/image1.png)

### <a name="known-issues"></a>既知の問題

Azure Stack Hub 内で CLI を使用する場合、次のような既知の問題があります。

 - CLI 対話モード。 たとえば、`az interactive` コマンドは Azure Stack Hub ではまだサポートされていません。
 - Azure Stack Hub で使用できる VM イメージの一覧を取得するには、`az vm image list` コマンドの代わりに、`az vm image list --all` コマンドを使用します。 `--all` オプションを指定すると、Azure Stack Hub 環境内で使用できるイメージのみが応答として返されます。
 - Azure で使用できる仮想マシン イメージのエイリアスは、Azure Stack Hub に適用できない場合があります。 仮想マシン イメージを使用する場合は、イメージのエイリアスの代わりに、URN パラメーター全体 (Canonical:UbuntuServer:14.04.3-LTS:1.0.0) を使用する必要がありますします。 この URN は、`az vm images list` コマンドから派生したイメージ仕様と一致している必要があります。

---

## <a name="next-steps"></a>次のステップ

- [Azure CLI を使用したテンプレートのデプロイ](azure-stack-deploy-template-command-line.md)
- [Azure Stack Hub ユーザー (オペレーター) に対する Azure CLI の有効化](../operator/azure-stack-cli-admin.md)
- [ユーザー アクセス許可の管理](azure-stack-manage-permissions.md)
