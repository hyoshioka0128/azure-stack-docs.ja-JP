---
title: Azure CLI を使用して Azure Stack Hub を管理する
description: クロスプラットフォーム コマンドライン インターフェイス (CLI) を使用して、Azure Stack Hub でリソースを管理およびデプロイする方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 08/24/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 12/10/2019
ms.openlocfilehash: 1fb17516e5ef0b4e3a670703a34e1e895c847b52
ms.sourcegitcommit: 65a115d1499b5fe16b6fe1c31cce43be21d05ef8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/25/2020
ms.locfileid: "88818846"
---
# <a name="manage-and-deploy-resources-to-azure-stack-hub-with-azure-cli"></a>Azure CLI を使用した Azure Stack Hub へのリソースの管理とデプロイ

この記事の手順に従って、Linux、Mac、Windows クライアントのプラットフォームから Azure Stack Development Kit (ASDK) のリソースを管理するように Azure コマンド ライン インターフェイス (CLI) を設定します。

## <a name="prepare-for-azure-cli"></a>Azure CLI の準備

ASDK を使用している場合、開発用マシン上で Azure CLI を使用するには、Azure Stack Hub の CA ルート証明書が必要です。 CLI を使用してリソースを管理する場合に、この証明書を使用します。

 - ASDK の外部のワークステーションから CLI を使用する場合は、**Azure Stack Hub の CA ルート証明書**が必要です。  

 - **仮想マシンのエイリアス エンドポイント**は、"UbuntuLTS" や "Win2012Datacenter" などのエイリアスを提供します。 このエイリアスは、VM をデプロイするときに、1 つのパラメーターとしてイメージの発行元、オファー、SKU、およびバージョンを参照します。  

これらの値の取得方法については、以下のセクションで説明します。

### <a name="export-the-azure-stack-hub-ca-root-certificate"></a>Azure Stack Hub の CA ルート証明書をエクスポートする

統合システムを使用している場合は、CA ルート証明書をエクスポートする必要はありません。 ASDK を使用している場合は、ASDK 上で CA のルート証明書をエクスポートします。

ASDK ルート証明書を PEM 形式でエクスポートするには:

1. Azure Stack Hub のルート証明書の名前を取得します。
    - Azure Stack Hub ユーザーまたは管理者ポータルにサインインします。
    - アドレス バーの近くにある **[セキュア]** をクリックします。
    - ポップアップ ウィンドウで、 **[有効]** をクリックします。
    - [証明書] ウィンドウで、 **[証明のパス]** タブをクリックします。
    - Azure Stack Hub のルート証明書の名前を書き留めます。

    ![Azure Stack Hub のルート証明書](media/azure-stack-version-profiles-azurecli2/root-cert-name.png)

2. [Azure Stack Hub に Windows VM を作成](azure-stack-quick-windows-portal.md)します。

3. この VM にサインインし、管理者特権の PowerShell プロンプトを開き、次のスクリプトを実行します。

    ```powershell  
      $label = "<the name of your Azure Stack Hub root cert from Step 1>"
      Write-Host "Getting certificate from the current user trusted store with subject CN=$label"
      $root = Get-ChildItem Cert:\CurrentUser\Root | Where-Object Subject -eq "CN=$label" | select -First 1
      if (-not $root)
      {
          Write-Error "Certificate with subject CN=$label not found"
          return
      }

    Write-Host "Exporting certificate"
    Export-Certificate -Type CERT -FilePath root.cer -Cert $root

    Write-Host "Converting certificate to PEM format"
    certutil -encode root.cer root.pem
    ```

4. ローカル マシンに証明書をコピーします。


### <a name="set-up-the-virtual-machine-aliases-endpoint"></a>仮想マシンのエイリアス エンドポイントを設定する

VM のエイリアス ファイルをホストする、パブリックにアクセス可能なエンドポイントを設定できます。 VM のエイリアス ファイルは、イメージの共通名が指定された JSON ファイルです。 この名前は、Azure CLI パラメーターとして VM をデプロイするときに使用します。

1. カスタム イメージを発行する場合は、発行時に指定した発行者、プラン、SKU、およびバージョン情報をメモしておいてください。 Marketplace のイメージである場合は、```Get-AzureVMImage``` コマンドレットを使用して情報を表示できます。  

2. GitHub から[サンプル ファイル](https://raw.githubusercontent.com/Azure/azure-rest-api-specs/master/arm-compute/quickstart-templates/aliases.json)をダウンロードします。

3. Azure Stack Hub にストレージ アカウントを作成します。 完了したら、BLOB コンテナーを作成します。 アクセス ポリシーを [パブリック] に設定します。  

4. その新しいコンテナーに JSON ファイルをアップロードします。 完了すると、BLOB の URL を表示できるようになります。 BLOB 名を選択してから、BLOB プロパティから URL を選択します。

### <a name="install-or-upgrade-cli"></a>CLI をインストールまたはアップグレードする

開発ワークステーションにサインインし、CLI をインストールします。 Azure Stack Hub には、Azure CLI のバージョン 2.0 以降が必要です。 API プロファイルの最新バージョンには、CLI の最新バージョンが必要です。 CLI をインストールするには、「[Azure CLI のインストール](/cli/azure/install-azure-cli)」で説明されている手順を使用します。 

1. インストールが正常に完了したことを確認するには、ターミナルまたはコマンド プロンプト ウィンドウを開いて次のコマンドを実行します。

    ```shell
    az --version
    ```

    お使いのコンピューターにインストールされている Azure CLI と依存するその他のライブラリのバージョンが表示されます。

    ![Azure Stack Hub 上の Azure CLI の Python の場所](media/azure-stack-version-profiles-azurecli2/cli-python-location.png)

2. CLI の Python の場所を書き留めておきます。 ASDK を実行する場合には、この場所を使用して証明書を追加する必要があります。


## <a name="windows-azure-ad"></a>Windows (Azure AD)

このセクションでは、ID 管理サービスとして Azure AD を使用していて、Windows マシン上で CLI を使用する場合の CLI の設定について説明します。

### <a name="trust-the-azure-stack-hub-ca-root-certificate"></a>Azure Stack Hub の CA ルート証明書を信頼する

ASDK を使用する場合は、リモート マシン上で CA ルート証明書を信頼する必要があります。 統合システムでは、この手順は必要ありません。

Azure Stack Hub の CA ルート証明書を信頼するには、そのルート証明書を、Azure CLI と一緒にインストールされたバージョンの Python 用の既存の Python 証明書ストアに追加します。 人によっては、独自の Python インスタンスを実行している可能性があります。 Azure CLI には、独自のバージョンの Python が含まれています。

1. マシン上の証明書ストアの場所を探します。  この場所は、コマンド `az --version` を実行することによって調べることができます。

2. CLI の Python アプリが格納されているフォルダーに移動します。 実行する必要があるのは、このバージョンの Python です。 Python をシステム パスに設定している場合には、お使いのバージョンの Python が実行されます。 代わりに、CLI で使用しているバージョンを実行すると共に、そのバージョンに証明書を追加する必要があります。 たとえば、CLI の Python が `C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\` にあるとします。

    次のコマンドを使用します。

    ```powershell  
    cd "c:\pathtoyourcliversionofpython"
    .\python -c "import certifi; print(certifi.where())"
    ```

    証明書の場所を書き留めておきます。 たとえば、「 `C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\lib\site-packages\certifi\cacert.pem` 」のように入力します。 具体的なパスは、お使いの OS やインストールした CLI によって異なります。

2. Azure Stack Hub の CA ルート証明書を Python の既存の証明書に追加して信頼します。

    ```powershell
    $pemFile = "<Fully qualified path to the PEM certificate Ex: C:\Users\user1\Downloads\root.pem>"

    $root = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2
    $root.Import($pemFile)

    Write-Host "Extracting required information from the cert file"
    $md5Hash    = (Get-FileHash -Path $pemFile -Algorithm MD5).Hash.ToLower()
    $sha1Hash   = (Get-FileHash -Path $pemFile -Algorithm SHA1).Hash.ToLower()
    $sha256Hash = (Get-FileHash -Path $pemFile -Algorithm SHA256).Hash.ToLower()

    $issuerEntry  = [string]::Format("# Issuer: {0}", $root.Issuer)
    $subjectEntry = [string]::Format("# Subject: {0}", $root.Subject)
    $labelEntry   = [string]::Format("# Label: {0}", $root.Subject.Split('=')[-1])
    $serialEntry  = [string]::Format("# Serial: {0}", $root.GetSerialNumberString().ToLower())
    $md5Entry     = [string]::Format("# MD5 Fingerprint: {0}", $md5Hash)
    $sha1Entry    = [string]::Format("# SHA1 Fingerprint: {0}", $sha1Hash)
    $sha256Entry  = [string]::Format("# SHA256 Fingerprint: {0}", $sha256Hash)
    $certText = (Get-Content -Path $pemFile -Raw).ToString().Replace("`r`n","`n")

    $rootCertEntry = "`n" + $issuerEntry + "`n" + $subjectEntry + "`n" + $labelEntry + "`n" + `
    $serialEntry + "`n" + $md5Entry + "`n" + $sha1Entry + "`n" + $sha256Entry + "`n" + $certText

    Write-Host "Adding the certificate content to Python Cert store"
    Add-Content "${env:ProgramFiles(x86)}\Microsoft SDKs\Azure\CLI2\Lib\site-packages\certifi\cacert.pem" $rootCertEntry

    Write-Host "Python Cert store was updated to allow the Azure Stack Hub CA root certificate"
    ```

### <a name="connect-to-azure-stack-hub"></a>Azure Stack Hub に接続する

1. `az cloud register` コマンドを実行して Azure Stack Hub 環境を登録します。

2. お客様の環境を登録します。 `az cloud register` を実行するときに、次のパラメーターを使用します。

    | 値 | 例 | 説明 |
    | --- | --- | --- |
    | 環境名 | AzureStackUser | ユーザー環境の場合は、`AzureStackUser` を使用します。 オペレーターの場合は、`AzureStackAdmin` を指定します。 |
    | Resource Manager エンドポイント | `https://management.local.azurestack.external` | ASDK の **ResourceManagerUrl** は次のとおりです。`https://management.local.azurestack.external/` になります。統合システムの **ResourceManagerUrl** は`https://management.<region>.<fqdn>/` になります。統合システム エンドポイントに関する質問がある場合は、お客様のクラウド オペレーターにお問い合わせください。 |
    | ストレージ エンドポイント | local.azurestack.external | `local.azurestack.external` は、ASDK の場合です。 統合システムの場合は、システムのエンドポイントを使用します。  |
    | Keyvault のサフィックス | .vault.local.azurestack.external | `.vault.local.azurestack.external` は、ASDK の場合です。 統合システムの場合は、システムのエンドポイントを使用します。  |
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

    >[!NOTE]  
    >1808 ビルドより前のバージョンの Azure Stack Hub を実行している場合は、**2019-03-01-hybrid** の API バージョンのプロファイルではなく、**2017-03-09-profile** の API バージョンのプロファイルを使用する必要があります。 また、Azure CLI の最新バージョンを使用する必要もあります。
 
1. `az login` コマンドを使用して Azure Stack Hub 環境にサインインします。 Azure Stack Hub 環境には、ユーザーまたは[サービス プリンシパル](/azure/active-directory/develop/app-objects-and-service-principals)としてサインインします。 

   - *ユーザー*としてサインインする場合: 

     `az login` コマンド内で直接ユーザー名とパスワードを指定するか、ブラウザーを使用して認証できます。 多要素認証が有効になっているアカウントの場合は、後者を実行する必要があります。

     ```azurecli
     az login -u <Active directory global administrator or user account. For example: username@<aadtenant>.onmicrosoft.com> --tenant <Azure Active Directory Tenant name. For example: myazurestack.onmicrosoft.com>
     ```

     > [!NOTE]
     > お使いのユーザー アカウントで多要素認証が有効になっている場合は、`-u` パラメーターを指定しないで、`az login` コマンドを使用します。 このコマンドを実行すると、認証で使用する必要がある URL とコードを取得できます。

   - *サービス プリンシパル*を使ってサインインする｡ 
    
     サインインする前に、CLI または [Azure Portal でサービス プリンシパルを作成](../operator/azure-stack-create-service-principals.md?view=azs-2002)してロールに割り当てます。 次のコマンドを使用してサインインします。

     ```azurecli  
     az login --tenant <Azure Active Directory Tenant name. For example: myazurestack.onmicrosoft.com> --service-principal -u <Application Id of the Service Principal> -p <Key generated for the Service Principal>
     ```

### <a name="test-the-connectivity"></a>接続のテスト

すべての設定が完了したら、CLI を使用し、Azure Stack Hub 内でリソースを作成します。 たとえば、アプリのリソース グループを作成し、VM を追加できます。 次のコマンドを使用して、"MyResourceGroup" という名前のリソース グループを作成します。

```azurecli
az group create -n MyResourceGroup -l local
```

リソース グループが正常に作成されると、上記のコマンドによって、新たに作成されたリソースの次のプロパティが出力されます。

![リソース グループ作成の出力](media/azure-stack-connect-cli/image1.png)

## <a name="windows-ad-fs"></a>Windows (AD FS)

このセクションでは、ID 管理サービスとして Active Directory フェデレーション サービス (AD FS) を使用していて、Windows マシン上で CLI を使用する場合の CLI の設定について説明します。

### <a name="trust-the-azure-stack-hub-ca-root-certificate"></a>Azure Stack Hub の CA ルート証明書を信頼する

ASDK を使用する場合は、リモート マシン上で CA ルート証明書を信頼する必要があります。 統合システムでは、この手順は必要ありません。

1. マシンで証明書の場所を探します。 この場所は、Python をインストールした場所に応じて異なる場合があります。 コマンド プロンプトまたは管理者特権の PowerShell プロンプトを開き、次のコマンドを入力します。

    ```powershell  
      python -c "import certifi; print(certifi.where())"
    ```

    証明書の場所を書き留めておきます。 たとえば、「 `~/lib/python3.5/site-packages/certifi/cacert.pem` 」のように入力します。 特定のパスは、お使いの OS やインストールした Python のバージョンによって異なります。

2. Azure Stack Hub の CA ルート証明書を Python の既存の証明書に追加して信頼します。

    ```powershell
    $pemFile = "<Fully qualified path to the PEM certificate Ex: C:\Users\user1\Downloads\root.pem>"

    $root = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2
    $root.Import($pemFile)

    Write-Host "Extracting required information from the cert file"
    $md5Hash    = (Get-FileHash -Path $pemFile -Algorithm MD5).Hash.ToLower()
    $sha1Hash   = (Get-FileHash -Path $pemFile -Algorithm SHA1).Hash.ToLower()
    $sha256Hash = (Get-FileHash -Path $pemFile -Algorithm SHA256).Hash.ToLower()

    $issuerEntry  = [string]::Format("# Issuer: {0}", $root.Issuer)
    $subjectEntry = [string]::Format("# Subject: {0}", $root.Subject)
    $labelEntry   = [string]::Format("# Label: {0}", $root.Subject.Split('=')[-1])
    $serialEntry  = [string]::Format("# Serial: {0}", $root.GetSerialNumberString().ToLower())
    $md5Entry     = [string]::Format("# MD5 Fingerprint: {0}", $md5Hash)
    $sha1Entry    = [string]::Format("# SHA1 Fingerprint: {0}", $sha1Hash)
    $sha256Entry  = [string]::Format("# SHA256 Fingerprint: {0}", $sha256Hash)
    $certText = (Get-Content -Path $pemFile -Raw).ToString().Replace("`r`n","`n")

    $rootCertEntry = "`n" + $issuerEntry + "`n" + $subjectEntry + "`n" + $labelEntry + "`n" + `
    $serialEntry + "`n" + $md5Entry + "`n" + $sha1Entry + "`n" + $sha256Entry + "`n" + $certText

    Write-Host "Adding the certificate content to Python Cert store"
    Add-Content "${env:ProgramFiles(x86)}\Microsoft SDKs\Azure\CLI2\Lib\site-packages\certifi\cacert.pem" $rootCertEntry

    Write-Host "Python Cert store was updated to allow the Azure Stack Hub CA root certificate"
    ```

### <a name="connect-to-azure-stack-hub"></a>Azure Stack Hub に接続する

1. `az cloud register` コマンドを実行して Azure Stack Hub 環境を登録します。

2. お客様の環境を登録します。 `az cloud register` を実行するときに、次のパラメーターを使用します。

    | 値 | 例 | 説明 |
    | --- | --- | --- |
    | 環境名 | AzureStackUser | ユーザー環境の場合は、`AzureStackUser` を使用します。 オペレーターの場合は、`AzureStackAdmin` を指定します。 |
    | Resource Manager エンドポイント | `https://management.local.azurestack.external` | ASDK の **ResourceManagerUrl** は次のとおりです。`https://management.local.azurestack.external/` になります。統合システムの **ResourceManagerUrl** は`https://management.<region>.<fqdn>/` になります。統合システム エンドポイントに関する質問がある場合は、お客様のクラウド オペレーターにお問い合わせください。 |
    | ストレージ エンドポイント | local.azurestack.external | `local.azurestack.external` は、ASDK の場合です。 統合システムの場合は、システムのエンドポイントを使用します。  |
    | Keyvault のサフィックス | .vault.local.azurestack.external | `.vault.local.azurestack.external` は、ASDK の場合です。 統合システムの場合は、システムのエンドポイントを使用します。  |
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

    >[!NOTE]  
    >1808 ビルドより前のバージョンの Azure Stack Hub を実行している場合は、**2019-03-01-hybrid** の API バージョンのプロファイルではなく、**2017-03-09-profile** の API バージョンのプロファイルを使用する必要があります。 また、Azure CLI の最新バージョンを使用する必要もあります。

1. `az login` コマンドを使用して Azure Stack Hub 環境にサインインします。 Azure Stack Hub 環境には、ユーザーまたは[サービス プリンシパル](/azure/active-directory/develop/app-objects-and-service-principals)としてサインインできます。 

   - *ユーザー*としてサインインする場合:

     `az login` コマンド内で直接ユーザー名とパスワードを指定するか、ブラウザーを使用して認証できます。 多要素認証が有効になっているアカウントの場合は、後者を実行する必要があります。

     ```azurecli
     az cloud register  -n <environmentname>   --endpoint-resource-manager "https://management.local.azurestack.external"  --suffix-storage-endpoint "local.azurestack.external" --suffix-keyvault-dns ".vault.local.azurestack.external" --endpoint-vm-image-alias-doc <URI of the document which contains VM image aliases>   --profile "2019-03-01-hybrid"
     ```

     > [!NOTE]
     > お使いのユーザー アカウントで多要素認証が有効になっている場合は、`-u` パラメーターを指定しないで、`az login` コマンドを使用します。 このコマンドを実行すると、認証で使用する必要がある URL とコードを取得できます。

   - *サービス プリンシパル*を使ってサインインする｡ 
    
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

![リソース グループ作成の出力](media/azure-stack-connect-cli/image1.png)


## <a name="linux-azure-ad"></a>Linux (Azure AD)

このセクションでは、ID 管理サービスとして Azure AD を使用していて、Linux マシン上で CLI を使用する場合の CLI の設定について説明します。

### <a name="trust-the-azure-stack-hub-ca-root-certificate"></a>Azure Stack Hub の CA ルート証明書を信頼する

ASDK を使用する場合は、リモート マシン上で CA ルート証明書を信頼する必要があります。 統合システムでは、この手順は必要ありません。

Azure Stack Hub の CA ルート証明書を Python の既存の証明書に追加して信頼します。

1. マシンで証明書の場所を探します。 この場所は、Python をインストールした場所に応じて異なる場合があります。 pip と certifi モジュールをインストールしておく必要があります。 Bash プロンプトから次の Python コマンドを使用します。

    ```bash  
    az --version
    ```

    証明書の場所を書き留めておきます。 たとえば、「 `~/lib/python3.5/site-packages/certifi/cacert.pem` 」のように入力します。 このパスは、オペレーティング システムと、インストールされている Python のバージョンによって異なります。

2. 証明書のパスを指定して、次の bash コマンドを実行します。

   - リモート Linux マシンの場合:

     ```bash  
     sudo cat PATH_TO_PEM_FILE >> ~/<yourpath>/cacert.pem
     ```

   - Azure Stack Hub 環境内の Linux マシンの場合:

     ```bash  
     sudo cat /var/lib/waagent/Certificates.pem >> ~/<yourpath>/cacert.pem
     ```

### <a name="connect-to-azure-stack-hub"></a>Azure Stack Hub に接続する

次の手順を使用して Azure Stack Hub に接続します。

1. `az cloud register` コマンドを実行して Azure Stack Hub 環境を登録します。

2. お客様の環境を登録します。 `az cloud register` を実行するときに、次のパラメーターを使用します。

    | 値 | 例 | 説明 |
    | --- | --- | --- |
    | 環境名 | AzureStackUser | ユーザー環境の場合は、`AzureStackUser` を使用します。 オペレーターの場合は、`AzureStackAdmin` を指定します。 |
    | Resource Manager エンドポイント | `https://management.local.azurestack.external` | ASDK の **ResourceManagerUrl** は次のとおりです。`https://management.local.azurestack.external/` になります。統合システムの **ResourceManagerUrl** は`https://management.<region>.<fqdn>/` になります。統合システム エンドポイントに関する質問がある場合は、お客様のクラウド オペレーターにお問い合わせください。 |
    | ストレージ エンドポイント | local.azurestack.external | `local.azurestack.external` は、ASDK の場合です。 統合システムの場合は、システムのエンドポイントを使用します。  |
    | Keyvault のサフィックス | .vault.local.azurestack.external | `.vault.local.azurestack.external` は、ASDK の場合です。 統合システムの場合は、システムのエンドポイントを使用します。  |
    | VM イメージのエイリアスのドキュメント エンドポイント | https://raw.githubusercontent.com/Azure/azure-rest-api-specs/master/arm-compute/quickstart-templates/aliases.json | VM イメージのエイリアスを含むドキュメントの URI。 詳細については、「[仮想マシンのエイリアス エンドポイントを設定する](#set-up-the-virtual-machine-aliases-endpoint)」を参照してください。 |

    ```azurecli  
    az cloud register -n <environmentname> --endpoint-resource-manager "https://management.local.azurestack.external" --suffix-storage-endpoint "local.azurestack.external" --suffix-keyvault-dns ".vault.local.azurestack.external" --endpoint-vm-image-alias-doc <URI of the document which contains VM image aliases>
    ```

3. アクティブな環境を設定します。 

      ```azurecli
        az cloud set -n <environmentname>
      ```

4. Azure Stack Hub 固有の API バージョンのプロファイルを使用するようにお使いの環境の構成を更新します。 構成を更新するには、次のコマンドを実行します。

    ```azurecli
      az cloud update --profile 2019-03-01-hybrid
   ```

    >[!NOTE]  
    >1808 ビルドより前のバージョンの Azure Stack Hub を実行している場合は、**2019-03-01-hybrid** の API バージョンのプロファイルではなく、**2017-03-09-profile** の API バージョンのプロファイルを使用する必要があります。 また、Azure CLI の最新バージョンを使用する必要もあります。

5. `az login` コマンドを使用して Azure Stack Hub 環境にサインインします。 Azure Stack Hub 環境には、ユーザーまたは[サービス プリンシパル](/azure/active-directory/develop/app-objects-and-service-principals)としてサインインできます。 

   * *ユーザー*としてサインインする場合:

     `az login` コマンド内で直接ユーザー名とパスワードを指定するか、ブラウザーを使用して認証できます。 多要素認証が有効になっているアカウントの場合は、後者を実行する必要があります。

     ```azurecli
     az login \
       -u <Active directory global administrator or user account. For example: username@<aadtenant>.onmicrosoft.com> \
       --tenant <Azure Active Directory Tenant name. For example: myazurestack.onmicrosoft.com>
     ```

     > [!NOTE]
     > お使いのユーザー アカウントで多要素認証が有効になっている場合は、`-u` パラメーターを指定しないで、`az login` コマンドを使用できます。 このコマンドを実行すると、認証で使用する必要がある URL とコードを取得できます。
   
   * *サービス プリンシパル*としてサインインする場合:
    
     サインインする前に、CLI または [Azure Portal でサービス プリンシパルを作成](../operator/azure-stack-create-service-principals.md?view=azs-2002)してロールに割り当てます。 次のコマンドを使用してサインインします。

     ```azurecli  
     az login \
       --tenant <Azure Active Directory Tenant name. For example: myazurestack.onmicrosoft.com> \
       --service-principal \
       -u <Application Id of the Service Principal> \
       -p <Key generated for the Service Principal>
     ```

### <a name="test-the-connectivity"></a>接続のテスト

すべての設定が完了したら、CLI を使用し、Azure Stack Hub 内でリソースを作成します。 たとえば、アプリのリソース グループを作成し、VM を追加できます。 次のコマンドを使用して、"MyResourceGroup" という名前のリソース グループを作成します。

```azurecli
    az group create -n MyResourceGroup -l local
```

リソース グループが正常に作成されると、上記のコマンドによって、新たに作成されたリソースの次のプロパティが出力されます。

![リソース グループ作成の出力](media/azure-stack-connect-cli/image1.png)

## <a name="linux-ad-fs"></a>Linux (AD FS)

このセクションでは、管理サービスとして Active Directory フェデレーション サービス (AD FS) を使用していて、Linux マシン上で CLI を使用する場合の CLI の設定について説明します。

### <a name="trust-the-azure-stack-hub-ca-root-certificate"></a>Azure Stack Hub の CA ルート証明書を信頼する

ASDK を使用する場合は、リモート マシン上で CA ルート証明書を信頼する必要があります。 統合システムでは、この手順は必要ありません。

Azure Stack Hub の CA ルート証明書を Python の既存の証明書に追加して信頼します。

1. マシンで証明書の場所を探します。 この場所は、Python をインストールした場所に応じて異なる場合があります。 pip と certifi モジュールをインストールしておく必要があります。 Bash プロンプトから次の Python コマンドを使用します。

    ```bash  
    az --version 
    ```

    証明書の場所を書き留めておきます。 たとえば、「 `~/lib/python3.5/site-packages/certifi/cacert.pem` 」のように入力します。 このパスは、オペレーティング システムと、インストールされている Python のバージョンによって異なります。

2. 証明書のパスを指定して、次の bash コマンドを実行します。

   - リモート Linux マシンの場合:

     ```bash  
     sudo cat PATH_TO_PEM_FILE >> ~/<yourpath>/cacert.pem
     ```

   - Azure Stack Hub 環境内の Linux マシンの場合:

     ```bash  
     sudo cat /var/lib/waagent/Certificates.pem >> ~/<yourpath>/cacert.pem
     ```

### <a name="connect-to-azure-stack-hub"></a>Azure Stack Hub に接続する

次の手順を使用して Azure Stack Hub に接続します。

1. `az cloud register` コマンドを実行して Azure Stack Hub 環境を登録します。

2. お客様の環境を登録します。 `az cloud register` を実行するときに、次のパラメーターを使用します。

    | 値 | 例 | 説明 |
    | --- | --- | --- |
    | 環境名 | AzureStackUser | ユーザー環境の場合は、`AzureStackUser` を使用します。 オペレーターの場合は、`AzureStackAdmin` を指定します。 |
    | Resource Manager エンドポイント | `https://management.local.azurestack.external` | ASDK の **ResourceManagerUrl** は次のとおりです。`https://management.local.azurestack.external/` になります。統合システムの **ResourceManagerUrl** は`https://management.<region>.<fqdn>/` になります。統合システム エンドポイントに関する質問がある場合は、お客様のクラウド オペレーターにお問い合わせください。 |
    | ストレージ エンドポイント | local.azurestack.external | `local.azurestack.external` は、ASDK の場合です。 統合システムの場合は、システムのエンドポイントを使用します。  |
    | Keyvault のサフィックス | .vault.local.azurestack.external | `.vault.local.azurestack.external` は、ASDK の場合です。 統合システムの場合は、システムのエンドポイントを使用します。  |
    | VM イメージのエイリアスのドキュメント エンドポイント | https://raw.githubusercontent.com/Azure/azure-rest-api-specs/master/arm-compute/quickstart-templates/aliases.json | VM イメージのエイリアスを含むドキュメントの URI。 詳細については、「[仮想マシンのエイリアス エンドポイントを設定する](#set-up-the-virtual-machine-aliases-endpoint)」を参照してください。 |

    ```azurecli  
    az cloud register -n <environmentname> --endpoint-resource-manager "https://management.local.azurestack.external" --suffix-storage-endpoint "local.azurestack.external" --suffix-keyvault-dns ".vault.local.azurestack.external" --endpoint-vm-image-alias-doc <URI of the document which contains VM image aliases>
    ```

3. アクティブな環境を設定します。 

      ```azurecli
        az cloud set -n <environmentname>
      ```

4. Azure Stack Hub 固有の API バージョンのプロファイルを使用するようにお使いの環境の構成を更新します。 構成を更新するには、次のコマンドを実行します。

    ```azurecli
      az cloud update --profile 2019-03-01-hybrid
   ```

    >[!NOTE]  
    >1808 ビルドより前のバージョンの Azure Stack Hub を実行している場合は、**2019-03-01-hybrid** の API バージョンのプロファイルではなく、**2017-03-09-profile** の API バージョンのプロファイルを使用する必要があります。 また、Azure CLI の最新バージョンを使用する必要もあります。

5. `az login` コマンドを使用して Azure Stack Hub 環境にサインインします。 Azure Stack Hub 環境には、ユーザーまたは[サービス プリンシパル](/azure/active-directory/develop/app-objects-and-service-principals)としてサインインできます。 

6. サインインします。 

   *  Web ブラウザーとデバイス コードを使用して**ユーザー**として:  

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

### <a name="test-the-connectivity"></a>接続のテスト

すべての設定が完了したら、CLI を使用し、Azure Stack Hub 内でリソースを作成します。 たとえば、アプリのリソース グループを作成し、VM を追加できます。 次のコマンドを使用して、"MyResourceGroup" という名前のリソース グループを作成します。

```azurecli
  az group create -n MyResourceGroup -l local
```

リソース グループが正常に作成されると、上記のコマンドによって、新たに作成されたリソースの次のプロパティが出力されます。

![リソース グループ作成の出力](media/azure-stack-connect-cli/image1.png)

## <a name="known-issues"></a>既知の問題

Azure Stack Hub 内で CLI を使用する場合、次のような既知の問題があります。

 - CLI 対話モード。 たとえば、`az interactive` コマンドは Azure Stack Hub ではまだサポートされていません。
 - Azure Stack Hub で使用できる VM イメージの一覧を取得するには、`az vm image list` コマンドの代わりに、`az vm image list --all` コマンドを使用します。 `--all` オプションを指定すると、Azure Stack Hub 環境内で使用できるイメージのみが応答として返されます。
 - Azure で使用できる仮想マシン イメージのエイリアスは、Azure Stack Hub に適用できない場合があります。 仮想マシン イメージを使用する場合は、イメージのエイリアスの代わりに、URN パラメーター全体 (Canonical:UbuntuServer:14.04.3-LTS:1.0.0) を使用する必要がありますします。 この URN は、`az vm images list` コマンドから派生したイメージ仕様と一致している必要があります。

## <a name="next-steps"></a>次のステップ

- [Azure CLI を使用したテンプレートのデプロイ](azure-stack-deploy-template-command-line.md)
- [Azure Stack Hub ユーザー (オペレーター) に対する Azure CLI の有効化](../operator/azure-stack-cli-admin.md)
- [ユーザー アクセス許可の管理](azure-stack-manage-permissions.md) 
