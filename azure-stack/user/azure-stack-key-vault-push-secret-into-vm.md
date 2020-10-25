---
title: Azure Stack Hub に安全に格納された証明書で VM をデプロイする
description: Azure Stack Hub の Key Vault を使って、仮想マシンをデプロイして証明書を仮想マシンにプッシュする方法について説明します。
author: sethmanheim
ms.topic: conceptual
ms.date: 09/01/2020
ms.author: sethm
ms.lastreviewed: 12/27/2019
ms.openlocfilehash: 5f99d816470649366703da5de4bf68ebdbe26a61
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90571832"
---
# <a name="deploy-a-vm-with-a-securely-stored-certificate-on-azure-stack-hub"></a>Azure Stack Hub に安全に格納された証明書で VM をデプロイする

この記事では、キー コンテナー証明書がインストールされた Azure Stack Hub 仮想マシン (VM) をデプロイする方法について説明します。

## <a name="overview"></a>概要

Active Directory への認証、Web トラフィックの暗号化など、多くのシナリオで証明書が使用されます。 証明書はシークレットとして Azure Stack Hub Key Vault に安全に格納できます。 Azure Stack Hub Key Vault を使用する利点は次のとおりです。

* 証明書がスクリプト、コマンド ラインの履歴、またはテンプレートに公開されません。
* 証明書管理プロセスが合理化されます。
* 証明書にアクセスするキーを制御できます。

## <a name="process-description"></a>プロセスの説明

次の手順では、VM に証明書をプッシュするために必要なプロセスについて説明します。

1. キー コンテナー シークレットを作成します。
2. **azuredeploy.parameters.json** ファイルを更新します。
3. テンプレートをデプロイします。

> [!NOTE]
> この手順は、Azure Stack Development Kit (ASDK) から、または VPN 経由で接続している場合は外部クライアントから実行できます。

## <a name="prerequisites"></a>前提条件

* ユーザーは、Key Vault サービスを含むプランをサブスクライブする必要があります。
* [PowerShell for Azure Stack Hub をインストールします](../operator/azure-stack-powershell-install.md)。
* [Azure Stack Hub ユーザーの PowerShell 環境の構成](azure-stack-powershell-configure-user.md)

## <a name="create-a-key-vault-secret"></a>キー コンテナー シークレットを作成する

次のスクリプトは、.pfx 形式で証明書を作成し、Key Vault を作成して、Key Vault にシークレットとして証明書を格納します。 シークレットの `contentType` は `pfx` に設定する必要があります。

> [!IMPORTANT]
> キー コンテナーを作成するときは、`-EnabledForDeployment` パラメーターを使う必要があります。 このパラメーターにより、Azure Resource Manager テンプレートから Key Vault を確実に参照できます。

```powershell
# Create a certificate in the .pfx format
New-SelfSignedCertificate `
  -certstorelocation cert:\LocalMachine\My `
  -dnsname contoso.microsoft.com

$pwd = ConvertTo-SecureString `
  -String "<Password used to export the certificate>" `
  -Force `
  -AsPlainText

Export-PfxCertificate `
  -cert "cert:\localMachine\my\<certificate thumbprint that was created in the previous step>" `
  -FilePath "<Fully qualified path to where the exported certificate can be stored>" `
  -Password $pwd

# Create a key vault and upload the certificate into the key vault as a secret
$vaultName = "contosovault"
$resourceGroup = "contosovaultrg"
$location = "local"
$secretName = "servicecert"
$fileName = "<Fully qualified path to where the exported certificate can be stored>"
$certPassword = "<Password used to export the certificate>"

$fileContentBytes = get-content $fileName `
  -Encoding Byte

$fileContentEncoded = [System.Convert]::ToBase64String($fileContentBytes)
$jsonObject = @"
{
"data": "$filecontentencoded",
"dataType" :"pfx",
"password": "$certPassword"
}
"@
$jsonObjectBytes = [System.Text.Encoding]::UTF8.GetBytes($jsonObject)
$jsonEncoded = [System.Convert]::ToBase64String($jsonObjectBytes)

New-AzureRmResourceGroup `
  -Name $resourceGroup `
  -Location $location

New-AzureRmKeyVault `
  -VaultName $vaultName `
  -ResourceGroupName $resourceGroup `
  -Location $location `
  -sku standard `
  -EnabledForDeployment

$secret = ConvertTo-SecureString `
  -String $jsonEncoded `
  -AsPlainText -Force

Set-AzureKeyVaultSecret `
  -VaultName $vaultName `
  -Name $secretName `
   -SecretValue $secret
```

このスクリプトを実行すると、出力にはシークレットの URI が含まれます。 この URI は、[Windows Resource Manager に証明書をプッシュするテンプレート](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/201-vm-windows-pushcertificate)で参照する必要があるため、メモしておいてください。 開発用コンピューターに [vm-push-certificate-windows](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/201-vm-windows-pushcertificate) テンプレート フォルダーをダウンロードします。 このフォルダーには、**azuredeploy.json** ファイルと **azuredeploy.parameters.json** ファイルが含まれています。これらは次の手順で必要になります。

ご自分の環境値に従って、**azuredeploy.parameters.json** ファイルを変更します。 重要なパラメーターは、コンテナー名、コンテナー リソース グループ、およびシークレットの URI (前のスクリプトによって生成されたもの) です。 パラメーター ファイルの例を次のセクションに示します。

## <a name="update-the-azuredeployparametersjson-file"></a>azuredeploy.parameters.json ファイルを更新する

ご自分の環境に従って、`vaultName`、シークレットの URI、`VmName`、他のパラメーターで **azuredeploy.parameters.json** ファイルを更新します。 テンプレート パラメーター ファイルの JSON ファイルの例を次に示します。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "newStorageAccountName": {
      "value": "kvstorage01"
    },
    "vmName": {
      "value": "VM1"
    },
    "vmSize": {
      "value": "Standard_D1_v2"
    },
    "adminUserName": {
      "value": "demouser"
    },
    "adminPassword": {
      "value": "demouser@123"
    },
    "vaultName": {
      "value": "contosovault"
    },
    "vaultResourceGroup": {
      "value": "contosovaultrg"
    },
    "secretUrlWithVersion": {
      "value": "https://testkv001.vault.local.azurestack.external/secrets/testcert002/82afeeb84f4442329ce06593502e7840"
    }
  }
}
```

## <a name="deploy-the-template"></a>テンプレートのデプロイ

次の PowerShell スクリプトを使用して、テンプレートをデプロイします。

```powershell
# Deploy a Resource Manager template to create a VM and push the secret to it
New-AzureRmResourceGroupDeployment `
  -Name KVDeployment `
  -ResourceGroupName $resourceGroup `
  -TemplateFile "<Fully qualified path to the azuredeploy.json file>" `
  -TemplateParameterFile "<Fully qualified path to the azuredeploy.parameters.json file>"
```

テンプレートが正常にデプロイされると、次の出力が表示されます。

![Template deployment の結果](media/azure-stack-key-vault-push-secret-into-vm/deployment-output.png)

証明書は、デプロイ中に Azure Stack Hub によって VM にプッシュされます。 証明書の場所は、VM のオペレーティング システムによって異なります。

* Windows では、証明書はユーザー指定の証明書ストアで **LocalMachine** の証明書の場所に追加されます。
* Linux では、証明書は、X509 証明書ファイルの場合は **UppercaseThumbprint.crt**、秘密キーの場合は **UppercaseThumbprint.prv** というファイル名で、**/var/lib/waagent** ディレクトリに配置されます。

## <a name="retire-certificates"></a>証明書の使用を終了する

証明書の使用は、証明書管理プロセスの一環として終了します。 以前のバージョンの証明書は削除できませんが、`Set-AzureKeyVaultSecretAttribute` コマンドレットを使って無効にすることはできます。

次の例は、証明書を無効にする方法を示しています。 `VaultName`、`Name`、`Version` パラメーターは、独自の値に置き換えてください。

```powershell
Set-AzureKeyVaultSecretAttribute -VaultName contosovault -Name servicecert -Version e3391a126b65414f93f6f9806743a1f7 -Enable 0
```

## <a name="next-steps"></a>次のステップ

* [Key Vault パスワードを使用して VM をデプロイする](azure-stack-key-vault-deploy-vm-with-secret.md)
* [アプリケーションが Key Vault にアクセスできるようにする](azure-stack-key-vault-sample-app.md)
