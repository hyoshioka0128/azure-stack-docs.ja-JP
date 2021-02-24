---
title: Azure Stack Development Kit (ASDK) での Azure CLI 用の証明書の設定 | Microsoft Docs
description: Azure Stack Development Kit 上で Azure CLI 用の証明書を設定する方法について説明します。
author: mattbriggs
ms.topic: how-to
ms.date: 12/2/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 12/2/2020
ms.openlocfilehash: 32a09b833e2e5a76abf0c9a6c67540ca124bfe94
ms.sourcegitcommit: 9b0e1264ef006d2009bb549f21010c672c49b9de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98254587"
---
# <a name="setting-up-certificates-for-azure-cli-on-azure-stack-development-kit"></a>Azure Stack Development Kit での Azure CLI 用の証明書の設定

この記事の手順に従って、Linux、Mac、Windows クライアントのプラットフォームから Azure Stack Development Kit (ASDK) のリソースを管理するように Azure コマンド ライン インターフェイス (CLI) を設定します。

この記事では、リモート管理コンピューターで証明書を取得して信頼する方法について説明します。 Azure CLI をインストールして、お使いの環境に接続するには、[Azure Stack Hub への Azure CLI のインストール](../user/azure-stack-version-profiles-azurecli2.md)に関するページをご覧ください。

## <a name="prepare-for-azure-cli"></a>Azure CLI の準備

ASDK の場合、開発用マシン上で Azure CLI を使用するには、Azure Stack Hub の CA ルート証明書が必要です。 CLI を使用してリソースを管理する場合に、この証明書を使用します。

 - ASDK の外部のワークステーションから CLI を使用する場合は、**Azure Stack Hub の CA ルート証明書** が必要です。  

 - **仮想マシンのエイリアス エンドポイント** は、"UbuntuLTS" や "Win2012Datacenter" などのエイリアスを提供します。 このエイリアスは、VM をデプロイするときに、1 つのパラメーターとしてイメージの発行元、オファー、SKU、およびバージョンを参照します。  

これらの値の取得方法については、以下のセクションで説明します。

## <a name="export-the-azure-stack-hub-ca-root-certificate"></a>Azure Stack Hub の CA ルート証明書をエクスポートする

ASDK で Azure CLI を使用するには、CA ルート証明書をエクスポートします。

ASDK ルート証明書を PEM 形式でエクスポートするには:

1. Azure Stack Hub のルート証明書の名前を取得します。
    1. Azure Stack Hub ユーザーまたは管理者ポータルにサインインします。
    2. アドレス バーの近くにある **[セキュア]** を選択します。
    3. ポップアップ ウィンドウで、 **[有効]** を選択します。
    4. [証明書] ウィンドウで、 **[証明のパス]** タブを選択します。
    5. Azure Stack Hub のルート証明書の名前を書き留めます。

    ![Azure Stack Hub のルート証明書](../user/media/azure-stack-version-profiles-azurecli2/root-cert-name.png)

2. [Azure Stack Hub に Windows VM を作成](../user/azure-stack-quick-windows-portal.md)します。

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

## <a name="set-up-the-virtual-machine-alias-endpoint"></a>仮想マシンのエイリアス エンドポイントを設定する

VM のエイリアス ファイルをホストする、パブリックにアクセス可能なエンドポイントを設定できます。 VM のエイリアス ファイルは、イメージの共通名が指定された JSON ファイルです。 この名前は、Azure CLI パラメーターとして VM をデプロイするときに使用します。

1. カスタム イメージを発行する場合は、発行時に指定した発行者、プラン、SKU、およびバージョン情報をメモしておいてください。 Marketplace のイメージである場合は、```Get-AzureVMImage``` コマンドレットを使用して情報を表示できます。  

2. GitHub から[サンプル ファイル](https://raw.githubusercontent.com/Azure/azure-rest-api-specs/master/arm-compute/quickstart-templates/aliases.json)をダウンロードします。

3. Azure Stack Hub にストレージ アカウントを作成します。 完了したら、BLOB コンテナーを作成します。 アクセス ポリシーを [パブリック] に設定します。  

4. その新しいコンテナーに JSON ファイルをアップロードします。 完了すると、BLOB の URL を表示できるようになります。 BLOB 名を選択してから、BLOB プロパティから URL を選択します。


## <a name="trust-the-certificate"></a>証明書を信頼する

ASDK で Azure CLI を使用する場合は、リモート マシン上で CA ルート証明書を信頼する必要があります。

### <a name="windows"></a>[Windows](#tab/win)

1. マシンで証明書の場所を探します。 この場所は、Python をインストールした場所に応じて異なる場合があります。 コマンド プロンプトまたは管理者特権の PowerShell プロンプトを開き、次のコマンドを入力します。

    ```powershell  
      python -c "import certifi; print(certifi.where())"
    ```

    証明書の場所を書き留めておきます。 たとえば、`~/lib/python3.5/site-packages/certifi/cacert.pem` のようにします。 特定のパスは、お使いの OS やインストールした Python のバージョンによって異なります。

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

Azure CLI を使用してインストールおよび接続する手順については、[Azure Stack Hub への Azure CLI のインストール](../user/azure-stack-version-profiles-azurecli2.md)に関するページをご覧ください。

### <a name="linux"></a>[Linux](#tab/lin)

CLI を設定するときに、Azure Stack Hub の CA ルート証明書を Python の既存の証明書に追加して信頼します。

1. マシンで証明書の場所を探します。 この場所は、Python をインストールした場所に応じて異なる場合があります。 pip と certifi モジュールをインストールしておく必要があります。 Bash プロンプトから次の Python コマンドを使用します。

    ```bash  
    az --version
    ```

    証明書の場所を書き留めておきます。 たとえば、`~/lib/python3.5/site-packages/certifi/cacert.pem` のようにします。 このパスは、オペレーティング システムと、インストールされている Python のバージョンによって異なります。

2. 証明書のパスを指定して、次の bash コマンドを実行します。

   - リモート Linux マシンの場合:

     ```bash  
     sudo cat PATH_TO_PEM_FILE >> ~/<yourpath>/cacert.pem
     ```

   - Azure Stack Hub 環境内の Linux マシンの場合:

     ```bash  
     sudo cat /var/lib/waagent/Certificates.pem >> ~/<yourpath>/cacert.pem
     ```

Azure CLI を使用してインストールおよび接続する手順については、[Azure Stack Hub への Azure CLI のインストール](../user/azure-stack-version-profiles-azurecli2.md)に関するページをご覧ください。

---

## <a name="next-steps"></a>次のステップ

[Azure CLI を Azure Stack Hub にインストールする](../user/azure-stack-version-profiles-azurecli2.md)