---
title: Key Vault に格納されたパスワードを使用して Azure Stack Hub VM をデプロイする
description: Azure Stack Hub キー コンテナーに格納されているパスワードを使用して VM をデプロイする方法を説明します
author: mattbriggs
ms.topic: conceptual
ms.date: 11/20/2020
ms.author: mabrigg
ms.reviewer: ppacent
ms.lastreviewed: 11/20/2020
ms.openlocfilehash: 963a4d7bc6cd61ae724e4f087861e19d918fea7d
ms.sourcegitcommit: 8c745b205ea5a7a82b73b7a9daf1a7880fd1bee9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/24/2020
ms.locfileid: "95518179"
---
# <a name="deploy-an-azure-stack-hub-vm-using-a-password-stored-in-key-vault"></a>Key Vault に格納されたパスワードを使用して Azure Stack Hub VM をデプロイする

この記事では、Azure Stack Hub Key Vault に格納されたパスワードを使用して、Windows Server 仮想マシン (VM) をデプロイする手順を説明します。 Key Vault パスワードを使用する方が、プレーンテキスト パスワードを渡すよりも安全性が高くなります。

## <a name="overview"></a>概要

Azure Stack Hub Key Vault にはパスワードなどの値をシークレットとして格納できます。 シークレットの作成後、そのシークレットは Azure Resource Manager テンプレートで参照できます。 Resource Manager でシークレットを使用する利点は次のとおりです。

* リソースをデプロイするたびにシークレットを手動で入力する必要がありません。
* シークレットにアクセスできるユーザーまたはサービス プリンシパルを指定できます。

## <a name="prerequisites"></a>前提条件

* ユーザーは、Key Vault サービスを含むプランをサブスクライブする必要があります。
* [PowerShell for Azure Stack Hub をインストールする。](../operator/powershell-install-az-module.md)
* [PowerShell 環境の構成。](azure-stack-powershell-configure-user.md)

次の手順では、キー コンテナーに格納されているパスワードを取得することによって VM を作成するために必要なプロセスについて説明します。

1. Key Vault シークレットを作成します。
2. `azuredeploy.parameters.json` ファイルを更新します。
3. テンプレートをデプロイします。

> [!NOTE]  
> この手順は、Azure Stack Development Kit (ASDK) から、または VPN 経由で接続している場合は外部クライアントから実行できます。

## <a name="create-a-key-vault-secret"></a>Key Vault シークレットを作成する

次のスクリプトは、キー コンテナーを作成し、パスワードをシークレットとしてキー コンテナーに格納します。 Key Vault を作成するときは、`-EnabledForDeployment` パラメーターを使います。 このパラメーターを指定すると、Azure Resource Manager テンプレートから Key Vault を参照できるようになります。

### <a name="az-modules"></a>[Az モジュール](#tab/az1)

```powershell

$vaultName = "contosovault"
$resourceGroup = "contosovaultrg"
$location = "local"
$secretName = "MySecret"

New-AzResourceGroup `
  -Name $resourceGroup `
  -Location $location

New-AzKeyVault `
  -VaultName $vaultName `
  -ResourceGroupName $resourceGroup `
  -Location $location
  -EnabledForTemplateDeployment

$secretValue = ConvertTo-SecureString -String '<Password for your virtual machine>' -AsPlainText -Force

Set-AzureKeyVaultSecret `
  -VaultName $vaultName `
  -Name $secretName `
  -SecretValue $secretValue

```

### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm1)

```powershell

$vaultName = "contosovault"
$resourceGroup = "contosovaultrg"
$location = "local"
$secretName = "MySecret"

New-AzureRMResourceGroup `
  -Name $resourceGroup `
  -Location $location

New-AzureRMKeyVault `
  -VaultName $vaultName `
  -ResourceGroupName $resourceGroup `
  -Location $location
  -EnabledForTemplateDeployment

$secretValue = ConvertTo-SecureString -String '<Password for your virtual machine>' -AsPlainText -Force

Set-AzureKeyVaultSecret `
  -VaultName $vaultName `
  -Name $secretName `
  -SecretValue $secretValue

```
---

前のスクリプトを実行すると、出力にはシークレットの URI (Uniform Resource Identifier) が含まれます。 この URI を書き留めておきます。 それを、[Key Vault のパスワードでの Windows VM のデプロイ テンプレート](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/101-vm-windows-create-passwordfromkv)で参照する必要があります。 開発用コンピューターに [101-vm-secure-password](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/101-vm-windows-create-passwordfromkv) フォルダーをダウンロードします。 このフォルダーには、次の手順で必要な `azuredeploy.json` ファイルと `azuredeploy.parameters.json` ファイルが含まれます。

環境の値に従って `azuredeploy.parameters.json` ファイルを変更します。 特に重要なパラメーターは、コンテナー名、コンテナー リソース グループ、およびシークレットの URI (前のスクリプトによって生成されたもの) です。 パラメーター ファイルの例を次に示します。

## <a name="update-the-azuredeployparametersjson-file"></a>azuredeploy.parameters.json ファイルを更新する

環境に応じて、VM 値の KeyVault URI、secretName、adminUsername を使用して `azuredeploy.parameters.json` ファイルを更新します。 テンプレート パラメーター ファイルの JSON ファイルの例を次に示します。

```json
{
    "$schema":  "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion":  "1.0.0.0",
    "parameters":  {
       "adminUsername":  {
         "value":  "demouser"
          },
       "adminPassword":  {
         "reference":  {
            "keyVault":  {
              "id":  "/subscriptions/xxxxxx/resourceGroups/RgKvPwd/providers/Microsoft.KeyVault/vaults/KvPwd"
              },
            "secretName":  "MySecret"
         }
       },
       "dnsLabelPrefix":  {
          "value":  "mydns123456"
        },
        "windowsOSVersion":  {
          "value":  "2016-Datacenter"
        }
    }
}

```

## <a name="template-deployment"></a>テンプレートのデプロイ

次の PowerShell スクリプトを使用して、テンプレートをデプロイします。

### <a name="az-modules"></a>[Az モジュール](#tab/az2)

```powershell  
New-AzResourceGroupDeployment `
  -Name KVPwdDeployment `
  -ResourceGroupName $resourceGroup `
  -TemplateFile "<Fully qualified path to the azuredeploy.json file>" `
  -TemplateParameterFile "<Fully qualified path to the azuredeploy.parameters.json file>"
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm2)

```powershell  
New-AzureRMResourceGroupDeployment `
  -Name KVPwdDeployment `
  -ResourceGroupName $resourceGroup `
  -TemplateFile "<Fully qualified path to the azuredeploy.json file>" `
  -TemplateParameterFile "<Fully qualified path to the azuredeploy.parameters.json file>"
```

---


テンプレートが正常にデプロイされると、次の出力が表示されます。

![デプロイの出力](media/azure-stack-key-vault-deploy-vm-with-secret/deployment-output.png)

## <a name="next-steps"></a>次のステップ

* [Key Vault を使ってサンプル アプリをデプロイする](azure-stack-key-vault-sample-app.md)
* [Key Vault 証明書を使って VM をデプロイする](azure-stack-key-vault-push-secret-into-vm.md)
