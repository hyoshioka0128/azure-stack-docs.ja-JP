---
title: Azure Stack Hub でアプリに Key Vault に格納されているシークレットへのアクセスを許可する
description: Azure Stack Hub で Key Vault からキーとシークレットを取得するサンプル アプリを実行する方法を説明します。
author: sethmanheim
ms.topic: conceptual
ms.date: 11/20/2020
ms.author: sethm
ms.lastreviewed: 11/20/2020
ms.openlocfilehash: e503bb3124e0b85ceb2816c6cd18af8580eaa61c
ms.sourcegitcommit: e88f0a1f2f4ed3bb8442bfb7b754d8b3a51319b4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/04/2021
ms.locfileid: "99533996"
---
# <a name="allow-apps-to-access-azure-stack-hub-key-vault-secrets"></a>Azure Stack Hub でアプリに Key Vault に格納されているシークレットへのアクセスを許可する

この記事の手順は、Azure Stack Hub のキー コンテナーからキーとシークレットを取得するサンプル アプリ **HelloKeyVault** を実行する方法を説明するものです。

## <a name="prerequisites"></a>前提条件

[Azure Stack Development Kit](../asdk/asdk-connect.md#connect-to-azure-stack-using-rdp) から、または [VPN 経由で接続](../asdk/asdk-connect.md#connect-to-azure-stack-using-vpn)している場合は Windows ベースの外部クライアントから、次の前提条件をインストールできます。

* [Azure Stack Hub と互換性のある Azure PowerShell モジュール](../operator/powershell-install-az-module.md)をインストールします。
* [Azure Stack Hub の操作に必要なツール](../operator/azure-stack-powershell-download.md)をダウンロードします。

## <a name="create-a-key-vault-and-register-an-app"></a>Key Vault の作成とアプリの登録

サンプル アプリケーションを準備するには:

* Azure Stack Hub でキー コンテナーを作成します。
* Azure Active Directory (Azure AD) でアプリを登録します。

Azure portal または PowerShell を使用して、サンプル アプリを準備します。

> [!NOTE]
> 既定では、この PowerShell スクリプトによって、Active Directory に新しいアプリが作成されます。 ただし、既存のアプリケーションのいずれかを登録できます。

次のスクリプトを実行する前に、必ず `aadTenantName` 変数と `applicationPassword` 変数の値を指定します。 `applicationPassword` の値を指定しないと、このスクリプトによりランダムなパスワードが生成されます。

### <a name="az-modules"></a>[Az モジュール](#tab/az)

```powershell
$vaultName           = 'myVault'
$resourceGroupName   = 'myResourceGroup'
$applicationName     = 'myApp'
$location            = 'local'

# Password for the application. If not specified, this script generates a random password during app creation.
$applicationPassword = ''

# Function to generate a random password for the application.
Function GenerateSymmetricKey()
{
    $key = New-Object byte[](32)
    $rng = [System.Security.Cryptography.RNGCryptoServiceProvider]::Create()
    $rng.GetBytes($key)
    return [System.Convert]::ToBase64String($key)
}

Write-Host 'Please log into your Azure Stack Hub user environment' -foregroundcolor Green

$tenantARM = "https://management.local.azurestack.external"
$aadTenantName = "FILL THIS IN WITH YOUR AAD TENANT NAME. FOR EXAMPLE: myazurestack.onmicrosoft.com"

# Configure the Azure Stack Hub operator's PowerShell environment.
Add-AzEnvironment `
  -Name "AzureStackUser" `
  -ArmEndpoint $tenantARM

$TenantID = Get-AzsDirectoryTenantId `
  -AADTenantName $aadTenantName `
  -EnvironmentName AzureStackUser

# Sign in to the user portal.
Connect-AzAccount `
  -EnvironmentName "AzureStackUser" `
  -TenantId $TenantID `

$now = [System.DateTime]::Now
$oneYearFromNow = $now.AddYears(1)

$applicationPassword = GenerateSymmetricKey

# Create a new Azure AD application.
$identifierUri = [string]::Format("http://localhost:8080/{0}",[Guid]::NewGuid().ToString("N"))
$homePage = "https://contoso.com"

Write-Host "Creating a new AAD Application"
$ADApp = New-AzADApplication `
  -DisplayName $applicationName `
  -HomePage $homePage `
  -IdentifierUris $identifierUri `
  -StartDate $now `
  -EndDate $oneYearFromNow `
  -Password $applicationPassword

Write-Host "Creating a new AAD service principal"
$servicePrincipal = New-AzADServicePrincipal `
  -ApplicationId $ADApp.ApplicationId

# Create a new resource group and a key vault in that resource group.
New-AzResourceGroup `
  -Name $resourceGroupName `
  -Location $location

Write-Host "Creating vault $vaultName"
$vault = New-AzKeyVault -VaultName $vaultName `
  -ResourceGroupName $resourceGroupName `
  -Sku standard `
  -Location $location

# Specify full privileges to the vault for the application.
Write-Host "Setting access policy"
Set-AzKeyVaultAccessPolicy -VaultName $vaultName `
  -ObjectId $servicePrincipal.Id `
  -PermissionsToKeys all `
  -PermissionsToSecrets all

Write-Host "Paste the following settings into the app.config file for the HelloKeyVault project:"
'<add key="VaultUrl" value="' + $vault.VaultUri + '"/>'
'<add key="AuthClientId" value="' + $servicePrincipal.ApplicationId + '"/>'
'<add key="AuthClientSecret" value="' + $applicationPassword + '"/>'
Write-Host
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm)

```powershell
$vaultName           = 'myVault'
$resourceGroupName   = 'myResourceGroup'
$applicationName     = 'myApp'
$location            = 'local'

# Password for the application. If not specified, this script generates a random password during app creation.
$applicationPassword = ''

# Function to generate a random password for the application.
Function GenerateSymmetricKey()
{
    $key = New-Object byte[](32)
    $rng = [System.Security.Cryptography.RNGCryptoServiceProvider]::Create()
    $rng.GetBytes($key)
    return [System.Convert]::ToBase64String($key)
}

Write-Host 'Please log into your Azure Stack Hub user environment' -foregroundcolor Green

$tenantARM = "https://management.local.azurestack.external"
$aadTenantName = "FILL THIS IN WITH YOUR AAD TENANT NAME. FOR EXAMPLE: myazurestack.onmicrosoft.com"

# Configure the Azure Stack Hub operator's PowerShell environment.
Add-AzureRMEnvironment `
  -Name "AzureStackUser" `
  -ArmEndpoint $tenantARM

$TenantID = Get-AzsDirectoryTenantId `
  -AADTenantName $aadTenantName `
  -EnvironmentName AzureStackUser

# Sign in to the user portal.
Add-AzureRMAccount `
  -EnvironmentName "AzureStackUser" `
  -TenantId $TenantID `

$now = [System.DateTime]::Now
$oneYearFromNow = $now.AddYears(1)

$applicationPassword = GenerateSymmetricKey

# Create a new Azure AD application.
$identifierUri = [string]::Format("http://localhost:8080/{0}",[Guid]::NewGuid().ToString("N"))
$homePage = "https://contoso.com"

Write-Host "Creating a new AAD Application"
$ADApp = New-AzureRMADApplication `
  -DisplayName $applicationName `
  -HomePage $homePage `
  -IdentifierUris $identifierUri `
  -StartDate $now `
  -EndDate $oneYearFromNow `
  -Password $applicationPassword

Write-Host "Creating a new AAD service principal"
$servicePrincipal = New-AzureRMADServicePrincipal `
  -ApplicationId $ADApp.ApplicationId

# Create a new resource group and a key vault in that resource group.
New-AzureRMResourceGroup `
  -Name $resourceGroupName `
  -Location $location

Write-Host "Creating vault $vaultName"
$vault = New-AzureRMKeyVault -VaultName $vaultName `
  -ResourceGroupName $resourceGroupName `
  -Sku standard `
  -Location $location

# Specify full privileges to the vault for the application.
Write-Host "Setting access policy"
Set-AzureRMKeyVaultAccessPolicy -VaultName $vaultName `
  -ObjectId $servicePrincipal.Id `
  -PermissionsToKeys all `
  -PermissionsToSecrets all

Write-Host "Paste the following settings into the app.config file for the HelloKeyVault project:"
'<add key="VaultUrl" value="' + $vault.VaultUri + '"/>'
'<add key="AuthClientId" value="' + $servicePrincipal.ApplicationId + '"/>'
'<add key="AuthClientSecret" value="' + $applicationPassword + '"/>'
Write-Host
```

---



次の画像は、キー コンテナーの作成に使用されたスクリプトからの出力を示しています。

![キー コンテナーとアクセス キー](media/azure-stack-key-vault-sample-app/settingsoutput.png)

上のスクリプトによって返される **VaultUrl**、**AuthClientId**、**AuthClientSecret** の値をメモしておきます。 これらの値は、**HelloKeyVault** アプリケーションを実行するために使用します。

## <a name="download-and-configure-the-sample-application"></a>サンプル アプリケーションのダウンロードと構成

「[Azure Key Vault client samples (Azure Key Vault クライアントのサンプル)](https://www.microsoft.com/download/details.aspx?id=45343)」ページから、キー コンテナーのサンプルをダウンロードします。 .zip ファイルの内容を自分の開発ワークステーションに抽出します。 samples フォルダーには 2 つのアプリがあります。この記事では、**HelloKeyVault** を使用します。

**HelloKeyVault** サンプルを読み込むには:

1. **Microsoft.Azure.KeyVault.Samples** > **samples** > **HelloKeyVault** フォルダーを参照します。
2. Visual Studio で **HelloKeyVault** アプリを開きます。

### <a name="configure-the-sample-application"></a>サンプル アプリケーションを構成する

Visual Studio で次の操作を行います。

1. HelloKeyVault\App.config ファイルを開き、`<appSettings>` 要素を見つけます。
2. キー コンテナーの作成時に返された値を使用して **VaultUrl**、**AuthClientId**、および **AuthCertThumbprint** キーを更新します。 既定では、App.config ファイルには `AuthCertThumbprint` のプレースホルダーが含まれます。 このプレースホルダーを `AuthClientSecret` に置き換えます。

   ```xml
   <appSettings>
    <!-- Update these settings for your test environment -->
    <add key="VaultUrl" value="URL to your Vault" />
    <add key="AuthClientId" value="Client Id of your Service Principal" />
    <add key="AuthCertThumbprint" value="Thumbprint of the certificate used for authentication" />
    <add key="TracingEnabled" value="false" />
   </appSettings>
   ```

3. ソリューションをリビルドします。

## <a name="run-the-app"></a>アプリを実行する

**HelloKeyVault** を実行すると、このアプリは Azure AD にサインインし、`AuthClientSecret` トークンを使用して、Azure Stack Hub の Key Vault に対する認証を行います。

**HelloKeyVault** サンプルを使用して、以下の操作を行います。

* キーおよびシークレットで作成、暗号化、ラップ、削除などの基本操作を行います。
* `encrypt`、`decrypt` などのパラメーターを **HelloKeyVault** に渡して、指定した変更をキー コンテナーに適用します。

## <a name="next-steps"></a>次のステップ

* [Key Vault パスワードを使用して VM をデプロイする](azure-stack-key-vault-deploy-vm-with-secret.md)
* [Key Vault 証明書を使って VM をデプロイする](azure-stack-key-vault-push-secret-into-vm.md)
