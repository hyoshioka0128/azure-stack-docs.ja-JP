---
title: Azure CLI を使用して Azure Stack Hub を管理する - MDC | Microsoft Docs
description: クロスプラットフォーム コマンドライン インターフェイス (CLI) を使用して、Modular Data Center (MDC) の Azure Stack Hub でリソースを管理およびデプロイする方法について説明します。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/2/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 12/2/2020
ms.openlocfilehash: 2296d1228819d32f77ccc0dd35e4f2228908e969
ms.sourcegitcommit: d719f148005e904fa426a001a687e80730c91fda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/06/2021
ms.locfileid: "97910263"
---
# <a name="manage-and-deploy-resources-to-azure-stack-hub-with-azure-cli---modular-data-center-mdc"></a>Azure CLI を使用した Azure Stack Hub へのリソースの管理とデプロイ - Modular Data Center (MDC)

*適用対象:Azure Stack Hub 統合システム*

この記事の手順に従って、Linux、Mac、Windows クライアントのプラットフォームから Azure Stack Hub のリソースを管理するように Azure コマンド ライン インターフェイス (CLI) を設定します。

## <a name="prepare-for-azure-cli"></a>Azure CLI の準備

**仮想マシンのエイリアス エンドポイント** は、"UbuntuLTS" や "Win2012Datacenter" などのエイリアスを提供します。 このエイリアスは、VM をデプロイするときに、1 つのパラメーターとしてイメージの発行元、オファー、SKU、およびバージョンを参照します。  

次のセクションでは、仮想マシンのエイリアス エンドポイントを設定する方法について説明します。

### <a name="set-up-the-virtual-machine-aliases-endpoint"></a>仮想マシンのエイリアス エンドポイントを設定する

VM のエイリアス ファイルをホストする、パブリックにアクセス可能なエンドポイントを設定できます。 VM のエイリアス ファイルは、イメージの共通名が指定された JSON ファイルです。 この名前は、Azure CLI パラメーターとして VM をデプロイするときに使用します。

1. カスタム イメージを発行する場合は、発行時に指定した発行者、プラン、SKU、およびバージョン情報をメモしておいてください。 Marketplace のイメージである場合は、```Get-AzureVMImage``` コマンドレットを使用して情報を表示できます。  

2. GitHub から[サンプル ファイル](https://raw.githubusercontent.com/Azure/azure-rest-api-specs/master/arm-compute/quickstart-templates/aliases.json)をダウンロードします。

3. Azure Stack Hub にストレージ アカウントを作成します。 完了したら、BLOB コンテナーを作成します。 アクセス ポリシーを [パブリック] に設定します。  

4. その新しいコンテナーに JSON ファイルをアップロードします。 完了すると、BLOB の URL を表示できるようになります。 BLOB 名を選択してから、BLOB プロパティから URL を選択します。

### <a name="install-or-upgrade-cli"></a>CLI をインストールまたはアップグレードする

開発ワークステーションにサインインし、CLI をインストールします。 Azure Stack Hub には、Azure CLI のバージョン 2.0 以降が必要です。 API プロファイルの最新バージョンには、CLI の最新バージョンが必要です。 CLI をインストールするには、「[Azure CLI のインストール](https://docs.microsoft.com/cli/azure/install-azure-cli)」で説明されている手順を使用します。 

インストールが正常に完了したことを確認するには、ターミナルまたはコマンド プロンプト ウィンドウを開いて次のコマンドを実行します。

```shell
az --version
```

お使いのコンピューターにインストールされている Azure CLI と依存するその他のライブラリのバージョンが表示されます。
    
![Azure Stack Hub 上の Azure CLI の Python の場所](media/azure-stack-version-profiles-azurecli2/cli-python-location.png)


## <a name="windowslinux-azure-ad"></a>Windows または Linux (Azure AD)

このセクションでは、ID 管理サービスとして Azure AD を使用していて、Windows マシンまたは Linux マシン上で CLI を使用する場合の CLI の設定について説明します。

### <a name="connect-to-azure-stack-hub"></a>Azure Stack Hub に接続する

1. `az cloud register` コマンドを実行して Azure Stack Hub 環境を登録します。

2. お客様の環境を登録します。 `az cloud register` を実行するときに、次のパラメーターを使用します。

    | 値 | 例 | 説明 |
    | --- | --- | --- |
    | 環境名 | AzureStackUser | ユーザー環境の場合は、`AzureStackUser` を使用します。 オペレーターの場合は、`AzureStackAdmin` を指定します。 |
    | Resource Manager エンドポイント | `https://management.local.azurestack.external` | <!-- TZLASDKFIX The **ResourceManagerUrl** in the ASDK is: `https://management.local.azurestack.external/`--> 統合システムの **ResourceManagerUrl** は `https://management.<region>.<fqdn>/` になります。統合システム エンドポイントに関する質問がある場合は、お客様のクラウド オペレーターにお問い合わせください。 |
    | ストレージ エンドポイント | local.azurestack.external | <!-- TZLASDKFIX `local.azurestack.external` is for the ASDK.--> 統合システムの場合は、システムのエンドポイントを使用します。  |
    | Keyvault のサフィックス | .vault.local.azurestack.external | <!-- TZLASDKFIX `.vault.local.azurestack.external` is for the ASDK.--> 統合システムの場合は、システムのエンドポイントを使用します。  |
    | VM イメージのエイリアスのドキュメント エンドポイント | https://raw.githubusercontent.com/Azure/azure-rest-api-specs/master/arm-compute/quickstart-templates/aliases.json | VM イメージのエイリアスを含むドキュメントの URI。 詳細については、「[仮想マシンのエイリアス エンドポイントを設定する](#set-up-the-virtual-machine-aliases-endpoint)」を参照してください。 |

    ```azurecli  
    az cloud register -n <environmentname> --endpoint-resource-manager "https://management.local.azurestack.external" --suffix-storage-endpoint "local.azurestack.external" --suffix-keyvault-dns ".vault.local.azurestack.external" --endpoint-vm-image-alias-doc <URI of the document which contains VM image aliases>
    ```

1. 次のコマンドを使用して、アクティブな環境を設定します。

      ```azurecli
      az cloud set -n <environmentname>
      ```

1. Azure Stack Hub 固有の API バージョンのプロファイルを使用するようにお使いの環境の構成を更新します。 構成を更新するには、次のコマンドを実行します。

    ```azurecli
    az cloud update --profile 2019-03-01-hybrid
   ```
 
1. `az login` コマンドを使用して Azure Stack Hub 環境にサインインします。 Azure Stack Hub 環境には、ユーザーまたはサービス プリンシパルとしてサインインします。 

   - *ユーザー* としてサインインする場合: 

     `az login` コマンド内で直接ユーザー名とパスワードを指定するか、ブラウザーを使用して認証できます。 多要素認証が有効になっているアカウントの場合は、後者を実行する必要があります。

     ```azurecli
     az login -u <Active directory global administrator or user account. For example: username@<aadtenant>.onmicrosoft.com> --tenant <Azure Active Directory Tenant name. For example: myazurestack.onmicrosoft.com>
     ```

     > [!NOTE]
     > お使いのユーザー アカウントで多要素認証が有効になっている場合は、`-u` パラメーターを指定しないで、`az login` コマンドを使用します。 このコマンドを実行すると、認証で使用する必要がある URL とコードを取得できます。

   - *サービス プリンシパル* を使ってサインインする｡ 
    
     サインインする前に、CLI または [Azure Portal でサービス プリンシパルを作成](../operator/azure-stack-create-service-principals.md)してロールに割り当てます。 次のコマンドを使用してサインインします。

     ```azurecli  
     az login --tenant <Azure Active Directory Tenant name. For example: myazurestack.onmicrosoft.com> --service-principal -u <Application Id of the Service Principal> -p <Key generated for the Service Principal>
     ```

### <a name="test-the-connectivity"></a>接続のテスト

すべての設定が完了したら、CLI を使用し、Azure Stack Hub 内でリソースを作成します。 たとえば、アプリのリソース グループを作成し、VM を追加できます。 次のコマンドを使用して、"MyResourceGroup" という名前のリソース グループを作成します。

```azurecli
az group create -n MyResourceGroup -l local
```

リソース グループが正常に作成されると、上記のコマンドによって、新たに作成されたリソースの次のプロパティが出力されます。

![リソース グループ作成の出力](media/azure-stack-connect-cli/output.png)

## <a name="windowslinux-ad-fs"></a>Windows または Linux (AD FS)

このセクションでは、ID 管理サービスとして Active Directory フェデレーション サービス (AD FS) を使用していて、Windows マシンまたは Linux マシン上で CLI を使用する場合の CLI の設定について説明します。


### <a name="connect-to-azure-stack-hub"></a>Azure Stack Hub に接続する

1. `az cloud register` コマンドを実行して Azure Stack Hub 環境を登録します。

2. お客様の環境を登録します。 `az cloud register` を実行するときに、次のパラメーターを使用します。

    | 値 | 例 | 説明 |
    | --- | --- | --- |
    | 環境名 | AzureStackUser | ユーザー環境の場合は、`AzureStackUser` を使用します。 オペレーターの場合は、`AzureStackAdmin` を指定します。 |
    | Resource Manager エンドポイント | `https://management.local.azurestack.external` | <!-- TZLASDKFIX The **ResourceManagerUrl** in the ASDK is: `https://management.local.azurestack.external/`--> 統合システムの **ResourceManagerUrl** は `https://management.<region>.<fqdn>/` になります。統合システム エンドポイントに関する質問がある場合は、お客様のクラウド オペレーターにお問い合わせください。 |
    | ストレージ エンドポイント | local.azurestack.external | <!-- TZLASDKFIX `local.azurestack.external` is for the ASDK.--> 統合システムの場合は、システムのエンドポイントを使用します。  |
    | Keyvault のサフィックス | .vault.local.azurestack.external | <!-- TZLASDKFIX `.vault.local.azurestack.external` is for the ASDK.--> 統合システムの場合は、システムのエンドポイントを使用します。  |
    | VM イメージのエイリアスのドキュメント エンドポイント | https://raw.githubusercontent.com/Azure/azure-rest-api-specs/master/arm-compute/quickstart-templates/aliases.json | VM イメージのエイリアスを含むドキュメントの URI。 詳細については、「[仮想マシンのエイリアス エンドポイントを設定する](#set-up-the-virtual-machine-aliases-endpoint)」を参照してください。 |

    ```azurecli  
    az cloud register -n <environmentname> --endpoint-resource-manager "https://management.local.azurestack.external" --suffix-storage-endpoint "local.azurestack.external" --suffix-keyvault-dns ".vault.local.azurestack.external" --endpoint-vm-image-alias-doc <URI of the document which contains VM image aliases>
    ```

1. 次のコマンドを使用して、アクティブな環境を設定します。

      ```azurecli
      az cloud set -n <environmentname>
      ```

1. Azure Stack Hub 固有の API バージョンのプロファイルを使用するようにお使いの環境の構成を更新します。 構成を更新するには、次のコマンドを実行します。

    ```azurecli
    az cloud update --profile 2019-03-01-hybrid
   ```

1. `az login` コマンドを使用して Azure Stack Hub 環境にサインインします。 Azure Stack Hub 環境には、ユーザーまたはサービス プリンシパルとしてサインインできます。 

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

### <a name="test-the-connectivity"></a>接続のテスト

すべての設定が完了したら、CLI を使用し、Azure Stack Hub 内でリソースを作成します。 たとえば、アプリのリソース グループを作成し、VM を追加できます。 次のコマンドを使用して、"MyResourceGroup" という名前のリソース グループを作成します。

```azurecli
az group create -n MyResourceGroup -l local
```

リソース グループが正常に作成されると、上記のコマンドによって、新たに作成されたリソースの次のプロパティが出力されます。

![リソース グループ作成の出力](media/azure-stack-connect-cli/output.png)


## <a name="known-issues"></a>既知の問題

Azure Stack Hub 内で CLI を使用する場合、次のような既知の問題があります。

 - CLI 対話モード。 たとえば、`az interactive` コマンドは Azure Stack Hub ではまだサポートされていません。
 - Azure Stack Hub で使用できる VM イメージの一覧を取得するには、`az vm image list` コマンドの代わりに、`az vm image list --all` コマンドを使用します。 `--all` オプションを指定すると、Azure Stack Hub 環境内で使用できるイメージのみが応答として返されます。
 - Azure で使用できる仮想マシン イメージのエイリアスは、Azure Stack Hub に適用できない場合があります。 仮想マシン イメージを使用する場合は、イメージのエイリアスの代わりに、URN パラメーター全体 (Canonical:UbuntuServer:14.04.3-LTS:1.0.0) を使用する必要がありますします。 この URN は、`az vm images list` コマンドから派生したイメージ仕様と一致している必要があります。

## <a name="next-steps"></a>次のステップ

- [Azure CLI を使用したテンプレートのデプロイ](../user/azure-stack-deploy-template-command-line.md)
- [Azure Stack Hub ユーザー (オペレーター) に対する Azure CLI の有効化](../operator/azure-stack-cli-admin.md)
- [ユーザー アクセス許可の管理](../operator/azure-stack-manage-permissions.md) 