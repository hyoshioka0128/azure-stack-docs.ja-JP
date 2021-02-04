---
title: ユーザーとして PowerShell を使用して Azure Stack Hub に接続する
description: 対話形式のプロンプトを使用するため、またはスクリプトを記述するために、PowerShell を使用して Azure Stack Hub に接続する方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 11/22/2020
ms.author: mabrigg
ms.reviewer: thoroet
ms.lastreviewed: 11/22/2020
ms.openlocfilehash: f89fa7e7e4e8e35b209f0b22a5994c45ede6a17c
ms.sourcegitcommit: e88f0a1f2f4ed3bb8442bfb7b754d8b3a51319b4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/04/2021
ms.locfileid: "99533979"
---
# <a name="connect-to-azure-stack-hub-with-powershell-as-a-user"></a>ユーザーとして PowerShell を使用して Azure Stack Hub に接続する

PowerShell を使用して Azure Stack Hub に接続し、Azure Stack Hub リソースを管理することができます。 たとえば、PowerShell を使用し、オファーをサブスクライブしたり、仮想マシン (VM) を作成したり、Azure Resource Manager テンプレートをデプロイしたりできます。

セットアップするには:
  - 要件を満たしていることをご確認ください。
  - Azure Active Directory (Azure AD) または Active Directory フェデレーション サービス (AD FS) と接続します。 
  - リソース プロバイダーを登録します。
  - 接続をテストします。

## <a name="prerequisites-to-connecting-with-powershell"></a>PowerShell と接続するための前提条件

次の前提条件は[開発キット](../asdk/asdk-connect.md#connect-to-azure-stack-using-rdp)から構成するか、[VPN 経由で接続](../asdk/asdk-connect.md#connect-to-azure-stack-using-vpn)している場合は Windows ベースの外部クライアントから構成します。

* [Azure Stack Hub と互換性のある Azure PowerShell モジュール](../operator/powershell-install-az-module.md)をインストールします。
* [Azure Stack Hub の操作に必要なツール](../operator/azure-stack-powershell-download.md)をダウンロードします。

次のスクリプト変数を自分の Azure Stack Hub 構成の値に必ず変更してください。

- **Azure AD テナントの名前**  
  Azure Stack Hub の管理に使用される Azure AD テナントの名前。 たとえば、yourdirectory.onmicrosoft.com です。
- **Azure Resource Manager エンドポイント**  
  Azure Stack 開発キットの場合、この値は `https://management.local.azurestack.external` に設定されます。 Azure Stack Hub 統合システムのこの値を取得するには、サービス プロバイダーにお問い合わせください。

## <a name="connect-to-azure-stack-hub-with-azure-ad"></a>Azure AD を使用して Azure Stack Hub に接続する

### <a name="az-modules"></a>[Az モジュール](#tab/az1)

```powershell  
    Add-AzEnvironment -Name "AzureStackUser" -ArmEndpoint "https://management.local.azurestack.external"
    # Set your tenant name
    $AuthEndpoint = (Get-AzEnvironment -Name "AzureStackUser").ActiveDirectoryAuthority.TrimEnd('/')
    $AADTenantName = "<myDirectoryTenantName>.onmicrosoft.com"
    $TenantId = (invoke-restmethod "$($AuthEndpoint)/$($AADTenantName)/.well-known/openid-configuration").issuer.TrimEnd('/').Split('/')[-1]

    # After signing in to your environment, Azure Stack Hub cmdlets
    # can be easily targeted at your Azure Stack Hub instance.
    Connect-AzAccount -EnvironmentName "AzureStackUser" -TenantId $TenantId
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm1)
 
```powershell  
    Add-AzureRMEnvironment -Name "AzureStackUser" -ArmEndpoint "https://management.local.azurestack.external"
    # Set your tenant name
    $AuthEndpoint = (Get-AzureRMEnvironment -Name "AzureStackUser").ActiveDirectoryAuthority.TrimEnd('/')
    $AADTenantName = "<myDirectoryTenantName>.onmicrosoft.com"
    $TenantId = (invoke-restmethod "$($AuthEndpoint)/$($AADTenantName)/.well-known/openid-configuration").issuer.TrimEnd('/').Split('/')[-1]

    # After signing in to your environment, Azure Stack Hub cmdlets
    # can be easily targeted at your Azure Stack Hub instance.
    Add-AzureRMAccount -EnvironmentName "AzureStackUser" -TenantId $TenantId
```

---


## <a name="connect-to-azure-stack-hub-with-ad-fs"></a>AD FS を使用して Azure Stack Hub に接続する

### <a name="az-modules"></a>[Az モジュール](#tab/az2)

  ```powershell  
  # Register an Azure Resource Manager environment that targets your Azure Stack Hub instance
  Add-AzEnvironment -Name "AzureStackUser" -ArmEndpoint "https://management.local.azurestack.external"

  # Sign in to your environment
  Connect-AzAccount -EnvironmentName "AzureStackUser"
  ```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm2)
 
  ```powershell  
  # Register an Azure Resource Manager environment that targets your Azure Stack Hub instance
  Add-AzureRMEnvironment -Name "AzureStackUser" -ArmEndpoint "https://management.local.azurestack.external"

  # Sign in to your environment
  Login-AzureRMAccount -EnvironmentName "AzureStackUser"
  ```

---


## <a name="register-resource-providers"></a>リソース プロバイダーを登録する

ポータル経由でリソースがデプロイされていない新しいユーザー サブスクリプションの場合、リソース プロバイダーが自動登録されません。 次のスクリプトを実行し、リソース プロバイダーを明示的に登録できます。

### <a name="az-modules"></a>[Az モジュール](#tab/az3)

```powershell  
foreach($s in (Get-AzSubscription)) {
        Select-AzSubscription -SubscriptionId $s.SubscriptionId | Out-Null
        Write-Progress $($s.SubscriptionId + " : " + $s.SubscriptionName)
Get-AzResourceProvider -ListAvailable | Register-AzResourceProvider
    }
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm3)
 
```powershell  
foreach($s in (Get-AzureRMSubscription)) {
        Select-AzureRMSubscription -SubscriptionId $s.SubscriptionId | Out-Null
        Write-Progress $($s.SubscriptionId + " : " + $s.SubscriptionName)
Get-AzureRMResourceProvider -ListAvailable | Register-AzureRMResourceProvider
    }
```

---


[!Include [AD FS only supports interactive authentication with user identities](../includes/note-powershell-adfs.md)]

## <a name="test-the-connectivity"></a>接続のテスト

すべての設定が完了したら、PowerShell を使って接続をテストし、Azure Stack Hub でリソースを作成します。 テストとして、アプリケーションのリソース グループを作成し、VM を追加します。 次のコマンドを実行し、"MyResourceGroup" という名前のリソース グループを作成します。

### <a name="az-modules"></a>[Az モジュール](#tab/az4)
```powershell  
New-AzResourceGroup -Name "MyResourceGroup" -Location "Local"
```

### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm4)
 
```powershell  
New-AzureRMResourceGroup -Name "MyResourceGroup" -Location "Local"
```

---


## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub のテンプレートの開発](azure-stack-develop-templates.md)
- [PowerShell を使用したテンプレートのデプロイ](azure-stack-deploy-template-powershell.md)
- [Azure Stack Hub PowerShell Module リファレンス](/powershell/azure/azure-stack/overview)
- クラウド オペレーター環境用に PowerShell を設定する場合、[Azure Stack Hub オペレーターの PowerShell 環境の構成](../operator/azure-stack-powershell-configure-admin.md)に関する記事をご覧ください。
