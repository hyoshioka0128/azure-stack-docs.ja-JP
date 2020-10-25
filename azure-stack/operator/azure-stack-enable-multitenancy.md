---
title: Azure Stack Hub でマルチテナントを構成する
description: Azure Stack Hub で複数の Azure Active Directory テナントを有効および無効にする方法について説明します。
author: BryanLa
ms.topic: how-to
ms.date: 06/18/2020
ms.author: bryanla
ms.reviewer: bryanr
ms.lastreviewed: 06/10/2019
ms.openlocfilehash: e99c1cc09f3dc6b0a04ff22f5b5dc96004ba305e
ms.sourcegitcommit: d73637146daaba0ef0ab1729683bb52c65466099
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/25/2020
ms.locfileid: "88857505"
---
# <a name="configure-multi-tenancy-in-azure-stack-hub"></a>Azure Stack Hub でマルチテナントを構成する

Azure Stack Hub を構成して、複数の Azure Active Directory (Azure AD) テナントのユーザーが Azure Stack Hub のサービスを使用できるようにできます。 ここでは、次のシナリオを例に説明します。

- あなたは、Azure Stack Hub がインストールされている contoso.onmicrosoft.com のサービス管理者です。
- メアリーは、ゲスト ユーザーがいる fabrikam.onmicrosoft.com のディレクトリ管理者です。
- あなたの会社から IaaS および PaaS サービスを受けているメアリーの会社では、ゲスト ディレクトリ (fabrikam.onmicrosoft.com) のユーザーが、contoso.onmicrosoft.com にある Azure Stack Hub のリソースにサインインして使用できるようにする必要があります。

このガイドでは、このシナリオの内容に基づいて、Azure Stack Hub でマルチテナントを構成するために必要な手順を説明します。 このシナリオでは、Fabrikam のユーザーが Contoso の Azure Stack Hub デプロイのサービスにサインインして使用できるようにする手順を、あなたとメアリーが完了する必要があります。

クラウド ソリューション プロバイダー (CSP) の場合は、[マルチテナント Azure Stack Hub を構成および管理](azure-stack-add-manage-billing-as-a-csp.md)できる追加の方法があります。 

## <a name="enable-multi-tenancy"></a>マルチテナントの有効化

Azure Stack Hub でマルチテナントを構成する前に、対応すべき前提条件がいくつかあります。
  
 - あなたとメアリーは連携して、Azure Stack Hub がインストールされているディレクトリ (Contoso) とゲスト ディレクトリ (Fabrikam) の両方に管理手順を実行する必要があります。
 - Azure Stack Hub 用の PowerShell が[インストール](azure-stack-powershell-install.md)および[構成](azure-stack-powershell-configure-admin.md)済みであることを確認します。
 - [Azure Stack Hub ツールをダウンロード](azure-stack-powershell-download.md)し、Identity モジュールをインポートします。

    ```powershell
    Import-Module .\Identity\AzureStack.Identity.psm1
    ```

### <a name="configure-azure-stack-hub-directory"></a>Azure Stack Hub ディレクトリの構成

このセクションでは、Fabrikam の Azure AD ディレクトリのテナントからのサインインを許可するように Azure Stack Hub を構成します。

ゲスト ディレクトリ テナント (Fabrikam) のユーザーとサービス プリンシパルを許可するよう Azure Resource Manager を構成して、ゲスト ディレクトリ テナントを Azure Stack Hub にオンボードします。

contoso.onmicrosoft.com のサービス管理者は、次のコマンドを実行します。

```powershell  
## The following Azure Resource Manager endpoint is for the ASDK. If you're in a multinode environment, contact your operator or service provider to get the endpoint.
$adminARMEndpoint = "https://adminmanagement.local.azurestack.external"

## Replace the value below with the Azure Stack Hub directory
$azureStackDirectoryTenant = "contoso.onmicrosoft.com"

## Replace the value below with the guest tenant directory. 
$guestDirectoryTenantToBeOnboarded = "fabrikam.onmicrosoft.com"

## Replace the value below with the name of the resource group in which the directory tenant registration resource should be created (resource group must already exist).
$ResourceGroupName = "system.local"

## Replace the value below with the region location of the resource group.
$location = "local"

# Subscription Name
$SubscriptionName = "Default Provider Subscription"

Register-AzSGuestDirectoryTenant -AdminResourceManagerEndpoint $adminARMEndpoint `
 -DirectoryTenantName $azureStackDirectoryTenant `
 -GuestDirectoryTenantName $guestDirectoryTenantToBeOnboarded `
 -Location $location `
 -ResourceGroupName $ResourceGroupName `
 -SubscriptionName $SubscriptionName
```

### <a name="configure-guest-directory"></a>ゲスト ディレクトリの構成

Azure Stack Hub のオペレーターが Azure Stack Hub で使用する Fabrikam ディレクトリを有効にしたら、メアリーは Azure Stack Hub を Fabrikam のディレクトリ テナントに登録する必要があります。

#### <a name="register-azure-stack-hub-with-the-guest-directory"></a>ゲスト ディレクトリへの Azure Stack Hub の登録

メアリー (Fabrikam のディレクトリ管理者) は、ゲスト ディレクトリ fabrikam.onmicrosoft.com で次のコマンドを実行します。

```powershell
## The following Azure Resource Manager endpoint is for the ASDK. If you're in a multinode environment, contact your operator or service provider to get the endpoint.
$tenantARMEndpoint = "https://management.local.azurestack.external"
    
## Replace the value below with the guest tenant directory.
$guestDirectoryTenantName = "fabrikam.onmicrosoft.com"

Register-AzSWithMyDirectoryTenant `
 -TenantResourceManagerEndpoint $tenantARMEndpoint `
 -DirectoryTenantName $guestDirectoryTenantName `
 -Verbose
```

> [!IMPORTANT]
> Azure Stack Hub 管理者が今後新しいサービスや更新プログラムをインストールした場合には、このスクリプトをもう一度実行する必要があります。
>
> このスクリプトを再実行すると、お使いのディレクトリ内の Azure Stack Hub アプリの状態をいつでも確認できます。
>
> VM をマネージド ディスク内に作成する (1808 更新プログラムで導入) 際に問題が起きた場合には、このスクリプトの再実行を求める新しい**ディスク リソース プロバイダー**が追加されました。

### <a name="direct-users-to-sign-in"></a>ユーザーをサインインに誘導する

あなたとメアリーの両方が Fabrikam ディレクトリのオンボード手順を完了したので、メアリーは Fabrikam のユーザーをサインインに誘導することができます。 Fabrikam ユーザー (fabrikam.onmicrosoft.com のサフィックスを持つユーザー) は、https\://portal.local.azurestack.external にアクセスしてサインインします。

メアリーは、Fabrikam ディレクトリの[外部プリンシパル](/azure/role-based-access-control/rbac-and-directory-admin-roles) (fabrikam.onmicrosoft.com のサフィックスを持たない Fabrikam ディレクトリ内のユーザー) はすべて、https\://portal.local.azurestack.external/fabrikam.onmicrosoft.com を使用してサインインするように誘導します。 この URL を使用しない場合、これらのユーザーは自分の既定のディレクトリ (Fabrikam) に送信され、管理者が同意していないことを示すエラーを受信します。

## <a name="disable-multi-tenancy"></a>マルチテナントの無効化

Azure Stack Hub に複数のテナントが不要になった場合は、次の手順を順番に実行してマルチ テナントを無効にできます。

1. ゲスト ディレクトリの管理者 (このシナリオでは Mary) として、*Unregister-AzsWithMyDirectoryTenant* を実行します。 このコマンドレットは、新しいディレクトリからすべての Azure Stack Hub アプリをアンインストールします。

    ``` PowerShell
    ## The following Azure Resource Manager endpoint is for the ASDK. If you're in a multinode environment, contact your operator or service provider to get the endpoint.
    $tenantARMEndpoint = "https://management.local.azurestack.external"
        
    ## Replace the value below with the guest tenant directory.
    $guestDirectoryTenantName = "fabrikam.onmicrosoft.com"
    
    Unregister-AzsWithMyDirectoryTenant `
     -TenantResourceManagerEndpoint $tenantARMEndpoint `
     -DirectoryTenantName $guestDirectoryTenantName `
     -Verbose 
    ```

2. Azure Stack Hub のサービス管理者 (このシナリオではあなた) として、*Unregister-AzSGuestDirectoryTenant* を実行します。

    ``` PowerShell
    ## The following Azure Resource Manager endpoint is for the ASDK. If you're in a multinode environment, contact your operator or service provider to get the endpoint.
    $adminARMEndpoint = "https://adminmanagement.local.azurestack.external"
    
    ## Replace the value below with the Azure Stack Hub directory
    $azureStackDirectoryTenant = "contoso.onmicrosoft.com"
    
    ## Replace the value below with the guest tenant directory. 
    $guestDirectoryTenantToBeDecommissioned = "fabrikam.onmicrosoft.com"
    
    ## Replace the value below with the name of the resource group in which the directory tenant registration resource should be created (resource group must already exist).
    $ResourceGroupName = "system.local"
    
    Unregister-AzSGuestDirectoryTenant -AdminResourceManagerEndpoint $adminARMEndpoint `
     -DirectoryTenantName $azureStackDirectoryTenant `
     -GuestDirectoryTenantName $guestDirectoryTenantToBeDecommissioned `
     -ResourceGroupName $ResourceGroupName
    ```

    > [!WARNING]
    > マルチ テナントを無効にする手順は、正しい順序で実行する必要があります。 手順 2 を先に実行した場合、手順 1 は失敗します。

## <a name="retrieve-azure-stack-hub-identity-health-report"></a>Azure Stack Hub の ID 正常性レポートの取得 

プレースホルダーの `<region>`、`<domain>`、`<homeDirectoryTenant>` を置き換えてから、次のコマンドレットを Azure Stack Hub 管理者として実行します。

```powershell

$AdminResourceManagerEndpoint = "https://adminmanagement.<region>.<domain>"
$DirectoryName = "<homeDirectoryTenant>.onmicrosoft.com"
$healthReport = Get-AzsHealthReport -AdminResourceManagerEndpoint $AdminResourceManagerEndpoint -DirectoryTenantName $DirectoryName
Write-Host "Healthy directories: "
$healthReport.directoryTenants | Where status -EQ 'Healthy' | Select -Property tenantName,tenantId,status | ft


Write-Host "Unhealthy directories: "
$healthReport.directoryTenants | Where status -NE 'Healthy' | Select -Property tenantName,tenantId,status | ft
```

### <a name="update-azure-ad-tenant-permissions"></a>Azure AD テナントのアクセス許可の更新

このアクションにより、ディレクトリの更新が必要であることを示す Azure Stack Hub のアラートがクリアされます。 **Azurestack-tools-master/identity** フォルダーから次のコマンドを実行します。

```powershell
Import-Module ..\Identity\AzureStack.Identity.psm1

$adminResourceManagerEndpoint = "https://adminmanagement.<region>.<domain>"

# This is the primary tenant Azure Stack is registered to:
$homeDirectoryTenantName = "<homeDirectoryTenant>.onmicrosoft.com"

Update-AzsHomeDirectoryTenant -AdminResourceManagerEndpoint $adminResourceManagerEndpoint `
   -DirectoryTenantName $homeDirectoryTenantName -Verbose
```

このスクリプトにより、Azure AD テナントの管理者資格情報の入力が求められます。実行には数分かかります。 このコマンドレットを実行すると、アラートがクリアされます。

## <a name="next-steps"></a>次のステップ

- [委任されたプロバイダーの管理](azure-stack-delegated-provider.md)
- [Azure Stack Hub の主要概念](azure-stack-overview.md)
- [クラウド ソリューション プロバイダーとして Azure Stack Hub の使用状況と課金を管理する](azure-stack-add-manage-billing-as-a-csp.md)
- [Azure Stack Hub に使用量と課金用のテナントを追加する](azure-stack-csp-howto-register-tenants.md)
