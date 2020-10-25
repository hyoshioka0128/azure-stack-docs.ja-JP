---
title: PowerShell を使用して Azure Stack Hub に接続する
description: PowerShell を使用して Azure Stack Hub に接続する方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 8/4/2020
ms.author: mabrigg
ms.reviewer: thoroet
ms.lastreviewed: 8/4/2020
ms.openlocfilehash: 3001d06deb81e275f3b62127cb555d3afceaff3c
ms.sourcegitcommit: c75e2cfd96f37a3497958eb87446888477f85bc9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/05/2020
ms.locfileid: "87810798"
---
# <a name="connect-to-azure-stack-hub-with-powershell"></a>PowerShell を使用して Azure Stack Hub に接続する

オファー、プラン、クォータ、アラートの作成などのリソースの管理を PowerShell を使用して行うように、Azure Stack Hub を構成できます。 このトピックは、オペレーター環境を構成するために役立ちます。

## <a name="prerequisites"></a>前提条件

[Azure Stack Development Kit (ASDK)](../asdk/asdk-connect.md#connect-with-rdp) から、または [VPN 経由で ASDK に接続](../asdk/asdk-connect.md#connect-with-vpn)している場合は Windows ベースの外部クライアントから、次の前提条件を実行します。

- [Azure Stack Hub と互換性のある Azure PowerShell モジュール](azure-stack-powershell-install.md)をインストールします。  
- [Azure Stack Hub の操作に必要なツール](azure-stack-powershell-download.md)をダウンロードします。  

## <a name="connect-with-azure-ad"></a>Azure AD との接続

PowerShell を使用する Azure Stack Hub オペレーター環境を構成するには、次のいずれかのスクリプトを実行します。 Azure Active Directory (Azure AD) tenantName と Azure Resource Manager エンドポイント値を独自の環境構成で置き換えます。

[!include[Remove Account](../../includes/remove-account.md)]

```powershell  
    # Register an Azure Resource Manager environment that targets your Azure Stack Hub instance. Get your Azure Resource Manager endpoint value from your service provider.
    Add-AzureRMEnvironment -Name "AzureStackAdmin" -ArmEndpoint "https://adminmanagement.local.azurestack.external" `
      -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
      -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external

    # Set your tenant name.
    $AuthEndpoint = (Get-AzureRmEnvironment -Name "AzureStackAdmin").ActiveDirectoryAuthority.TrimEnd('/')
    $AADTenantName = "<myDirectoryTenantName>.onmicrosoft.com"
    $TenantId = (invoke-restmethod "$($AuthEndpoint)/$($AADTenantName)/.well-known/openid-configuration").issuer.TrimEnd('/').Split('/')[-1]

    # After signing in to your environment, Azure Stack Hub cmdlets
    # can be easily targeted at your Azure Stack Hub instance.
    Add-AzureRmAccount -EnvironmentName "AzureStackAdmin" -TenantId $TenantId
```

## <a name="connect-with-ad-fs"></a>AD FS を使用した接続

Azure Stack Hub オペレーター環境に Azure Active Directory フェデレーション サービス (Azure AD FS) を使用した PowerShell で接続します。 ASDK では、この Azure Resource Manager エンドポイントは `https://adminmanagement.local.azurestack.external` に設定されています。 Azure Resource Manager エンドポイントを Azure Stack Hub 統合システム用に取得するには、サービス プロバイダーにお問い合わせください。

  ```powershell  
  # Register an Azure Resource Manager environment that targets your Azure Stack Hub instance. Get your Azure Resource Manager endpoint value from your service provider.
    Add-AzureRMEnvironment -Name "AzureStackAdmin" -ArmEndpoint "https://adminmanagement.local.azurestack.external" `
      -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
      -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external

  # Sign in to your environment.
  Login-AzureRmAccount -EnvironmentName "AzureStackAdmin"
  ```

[!Include [AD FS only supports interactive authentication with user identities](../includes/note-powershell-adfs.md)]

## <a name="test-the-connectivity"></a>接続のテスト

必要な設定がすべて整ったら、PowerShell を使用して Azure Stack Hub 内にリソースを作成してみましょう。 たとえば、アプリのリソース グループを作成して仮想マシンを追加できます。 次のコマンドを使用して、**MyResourceGroup** という名前のリソース グループを作成します。

```powershell  
New-AzureRmResourceGroup -Name "MyResourceGroup" -Location "Local"
```

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub のテンプレートを開発します](../user/azure-stack-develop-templates.md)。
- [PowerShell を使用したテンプレートのデプロイ](../user/azure-stack-deploy-template-powershell.md)
  - [Azure Stack Hub モジュールのリファレンス](/powershell/azure/azure-stack/overview)
