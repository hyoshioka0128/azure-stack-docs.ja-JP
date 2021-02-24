---
title: PowerShell を使用した Azure Stack Hub での Key Vault の管理
description: PowerShell を使用して Azure Stack Hub で Key Vault を管理する方法について説明します。
author: sethmanheim
ms.topic: article
ms.date: 11/20/2020
ms.author: sethm
ms.lastreviewed: 11/20/2020
ms.openlocfilehash: 0d7ce96b1619c21c5f442ab4d5a240a9eed16397
ms.sourcegitcommit: 8c745b205ea5a7a82b73b7a9daf1a7880fd1bee9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/24/2020
ms.locfileid: "95518349"
---
# <a name="manage-key-vault-in-azure-stack-hub-using-powershell"></a>PowerShell を使用した Azure Stack Hub での Key Vault の管理

この記事では、Azure Stack Hub で PowerShell を使用してキー コンテナーを作成および管理する方法について説明します。 Key Vault の PowerShell コマンドレットを使用して以下を行う方法について説明します。

* Key Vault を作成します。
* 暗号化キーとシークレットを格納および管理する。
* ユーザーまたはアプリに対して、コンテナーで操作を呼び出すことを許可する。

>[!NOTE]
>この記事で説明する Key Vault の PowerShell コマンドレットは、Azure PowerShell SDK で提供されます。

## <a name="prerequisites"></a>前提条件

* Azure Key Vault サービスを含むプランをサブスクライブする必要があります。
* [PowerShell for Azure Stack Hub をインストールします](../operator/powershell-install-az-module.md)。
* [Azure Stack Hub の PowerShell 環境を構成します](azure-stack-powershell-configure-user.md)。

## <a name="enable-your-tenant-subscription-for-key-vault-operations"></a>Key Vault 操作のためのテナント サブスクリプションの有効化

キー コンテナーに対してなんらかの操作を発行するには、テナント サブスクリプションでコンテナー操作が有効になっていることを確認する必要があります。 キー コンテナー操作が有効になっていることを確認するには、次のコマンドを実行します。

### <a name="az-modules"></a>[Az モジュール](#tab/az1)

```powershell  
Get-AzResourceProvider -ProviderNamespace Microsoft.KeyVault | ft -Autosize
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm1)
 

 ```powershell  
Get-AzureRMResourceProvider -ProviderNamespace Microsoft.KeyVault | ft -Autosize
```
---


サブスクリプションでコンテナー操作が有効になっている場合は、出力のキー コンテナーのすべてのリソースの種類で **RegistrationState** が **Registered** と表示されます。

![PowerShell でのキー コンテナーの登録状態](media/azure-stack-key-vault-manage-powershell/image1.png)

コンテナー操作が有効になっていない場合は、次のコマンドを発行して、自分のサブスクリプションで Key Vault サービスを登録します。

### <a name="az-modules"></a>[Az モジュール](#tab/az2)

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.KeyVault
```

### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm2)
 
 ```powershell
Register-AzureRMResourceProvider -ProviderNamespace Microsoft.KeyVault
```

---


登録に成功した場合は、次の出力が返されます。

![Key vault registration in Powershell successful (PowerShell でのキー コンテナーの登録に成功しました)](media/azure-stack-key-vault-manage-powershell/image2.png)

キー コンテナー コマンドを呼び出すと、"サブスクリプションが名前空間 'Microsoft.KeyVault' を使用するように登録されていません" などのエラーが表示されることがあります。エラーが表示された場合は、前の手順に従って、Key Vault リソース プロバイダーを有効にしたことを確認してください。

## <a name="create-a-key-vault"></a>Key Vault を作成します

キー コンテナーを作成する前に、リソース グループを作成し、このリソース グループに、キー コンテナー関連のすべてのリソースが存在するようにしておきます。 次のコマンドを使用して、新しいリソース グループを作成します。

### <a name="az-modules"></a>[Az モジュール](#tab/az3)

```powershell
New-AzResourceGroup -Name "VaultRG" -Location local -verbose -Force
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm3)
 
 ```powershell
New-AzureRMResourceGroup -Name "VaultRG" -Location local -verbose -Force
```

---


![PowerShell で生成された新しいリソース グループ](media/azure-stack-key-vault-manage-powershell/image3.png)

次のコマンドレットを使用して、以前に作成したリソース グループ内にキー コンテナーを作成します。 このコマンドは、3 つの必須パラメーター (リソース グループ名、キー コンテナー名、地理的な場所) を読み取ります。

次のコマンドを実行して、キー コンテナーを作成します。

### <a name="az-modules"></a>[Az モジュール](#tab/az4)

```powershell
New-AzKeyVault -VaultName "Vault01" -ResourceGroupName "VaultRG" -Location local -verbose
```
   
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm4)
 
```powershell
New-AzureRMKeyVault -VaultName "Vault01" -ResourceGroupName "VaultRG" -Location local -verbose
```

---


![PowerShell で生成された新しいキー コンテナー](media/azure-stack-key-vault-manage-powershell/image4.png)

このコマンドの出力は、作成したキー コンテナーのプロパティを示します。 アプリでは、このコンテナーにアクセスするときに、**Vault URI** プロパティ (この例では `https://vault01.vault.local.azurestack.external`) を使用する必要があります。

### <a name="active-directory-federation-services-ad-fs-deployment"></a>Active Directory フェデレーション サービス (AD FS) のデプロイ

AD FS のデプロイでは、"アクセス ポリシーが設定されていません。 ユーザーまたはアプリケーションに、このコンテナーを使用するアクセス許可がありません" という警告が表示される場合があります。 この問題を解決するには、[**Set-AzKeyVaultAccessPolicy**](#authorize-an-app-to-use-a-key-or-secret) コマンドを使用して、コンテナーのアクセス ポリシーを設定します。

### <a name="az-modules"></a>[Az モジュール](#tab/az5)

```powershell
# Obtain the security identifier(SID) of the active directory user
$adUser = Get-ADUser -Filter "Name -eq '{Active directory user name}'"
$objectSID = $adUser.SID.Value

# Set the key vault access policy
Set-AzKeyVaultAccessPolicy -VaultName "{key vault name}" -ResourceGroupName "{resource group name}" -ObjectId "{object SID}" -PermissionsToKeys {permissionsToKeys} -PermissionsToSecrets {permissionsToSecrets} -BypassObjectIdValidation
```

### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm5)

```powershell
# Obtain the security identifier(SID) of the active directory user
$adUser = Get-ADUser -Filter "Name -eq '{Active directory user name}'"
$objectSID = $adUser.SID.Value

# Set the key vault access policy
Set-AzureRMKeyVaultAccessPolicy -VaultName "{key vault name}" -ResourceGroupName "{resource group name}" -ObjectId "{object SID}" -PermissionsToKeys {permissionsToKeys} -PermissionsToSecrets {permissionsToSecrets} -BypassObjectIdValidation
```

---


## <a name="manage-keys-and-secrets"></a>キーとシークレットの管理

コンテナーを作成したら、これらの手順を使用して、コンテナーでキーとシークレットを作成および管理します。

### <a name="create-a-key"></a>キーの作成

**Add-AzureKeyVaultKey** コマンドレットを使用して、キー コンテナーにソフトウェアで保護されたキーを作成またはインポートします。

```powershell
Add-AzureKeyVaultKey -VaultName "Vault01" -Name "Key01" -verbose -Destination Software
```


`-Destination` パラメーターは、そのキーがソフトウェアで保護されることを指定するために使用されます。 キーが正常に作成されると、作成されたキーの詳細が出力されます。

![PowerShell で生成された新しいキー コンテナー キー](media/azure-stack-key-vault-manage-powershell/image5.png)

これで、作成されたキーの URI を使用してそのキーを参照できます。 既存のキーと同じ名前のキーを作成またはインポートした場合、元のキーは、新しいキーで指定される値で更新されます。 前のバージョンにアクセスするには、キーのバージョン固有の URI を使用します。 次に例を示します。

* `https://vault10.vault.local.azurestack.external:443/keys/key01` を使用して、常に現在のバージョンを取得します。
* `https://vault010.vault.local.azurestack.external:443/keys/key01/d0b36ee2e3d14e9f967b8b6b1d38938a` を使用して、この特定のバージョンを取得します。

### <a name="get-a-key"></a>キーの取得

**Get-AzureKeyVaultKey** コマンドレットを使用して、キーとその詳細を読み取ります。

```powershell
Get-AzureKeyVaultKey -VaultName "Vault01" -Name "Key01"
```

### <a name="create-a-secret"></a>シークレットの作成

**Set-AzureKeyVaultSecret** コマンドレットを使用して、コンテナー内のシークレットを作成または更新します。 シークレットがまだ存在しない場合は、作成されます。 既に存在する場合は、新しいバージョンのシークレットが作成されます。

```powershell
$secretvalue = ConvertTo-SecureString "User@123" -AsPlainText -Force
Set-AzureKeyVaultSecret -VaultName "Vault01" -Name "Secret01" -SecretValue $secretvalue
```

![PowerShell でシークレットを作成する](media/azure-stack-key-vault-manage-powershell/image6.png)

### <a name="get-a-secret"></a>シークレットの取得

**Get-AzureKeyVaultSecret** コマンドレットを使用して、キー コンテナー内のシークレットを読み取ります。 このコマンドでは、シークレットのすべてのバージョンまたは特定のバージョンを返すことができます。

```powershell
Get-AzureKeyVaultSecret -VaultName "Vault01" -Name "Secret01"
```

キーとシークレットを作成したら、そのキーとシークレットを外部アプリが使用することを承認できます。

## <a name="authorize-an-app-to-use-a-key-or-secret"></a>アプリに対してキーまたはシークレットの使用を許可する

アプリがキー コンテナー内のキーまたはシークレットにアクセスすることを承認するには、次のコマンドレットを使用します。

次の例では、コンテナー名が **ContosoKeyVault** で、承認するアプリのクライアント ID が **8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed** です。 アプリを承認するには、次のコマンドを実行します。 また、**PermissionsToKeys** パラメーターを指定し、ユーザー、アプリ、またはセキュリティ グループに対してアクセス許可を設定することもできます。

AD FS が構成された Azure Stack Hub 環境に対してこのコマンドレットを使用する場合、パラメーター BypassObjectIdValidation を指定する必要があります

### <a name="az-modules"></a>[Az モジュール](#tab/az6)

```powershell
Set-AzKeyVaultAccessPolicy -VaultName 'ContosoKeyVault' -ServicePrincipalName 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed -PermissionsToKeys decrypt,sign -BypassObjectIdValidation
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm6)

```powershell
Set-AzureRMKeyVaultAccessPolicy -VaultName 'ContosoKeyVault' -ServicePrincipalName 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed -PermissionsToKeys decrypt,sign -BypassObjectIdValidation
```

---


その同じアプリがコンテナーのシークレットを読み取ることを承認する場合は、次のコマンドレットを実行します。

### <a name="az-modules"></a>[Az モジュール](#tab/az7)

```powershell
Set-AzKeyVaultAccessPolicy -VaultName 'ContosoKeyVault' -ServicePrincipalName 8f8c4bbd-485b-45fd-98f7-ec6300 -PermissionsToKeys Get -BypassObjectIdValidation
```

### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm7)

```powershell
Set-AzureRMKeyVaultAccessPolicy -VaultName 'ContosoKeyVault' -ServicePrincipalName 8f8c4bbd-485b-45fd-98f7-ec6300 -PermissionsToKeys Get -BypassObjectIdValidation
```

---


## <a name="next-steps"></a>次のステップ

* [Key Vault に格納されているパスワードを使用して VM をデプロイする](azure-stack-key-vault-deploy-vm-with-secret.md)
* [Key Vault に格納されている証明書を使用して VM をデプロイする](azure-stack-key-vault-push-secret-into-vm.md)
