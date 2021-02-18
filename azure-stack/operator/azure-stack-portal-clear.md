---
title: Azure Stack Hub からオンデマンドでポータル ユーザー データをクリアする
description: Azure Stack Hub ユーザーからの要求時に、ポータル ユーザー データをクリアする方法を学習します。
author: sethmanheim
ms.topic: conceptual
ms.date: 02/08/2021
ms.author: sethm
ms.reviewer: troettinger
ms.lastreviewed: 09/10/2019
ms.openlocfilehash: f85f5a06699b11d5316a889c045a33376304a0f0
ms.sourcegitcommit: d542b68b299b73e045f30916afb6018e365e9db6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2021
ms.locfileid: "99975896"
---
# <a name="clear-portal-user-data-from-azure-stack-hub"></a>Azure Stack Hub からポータル ユーザー データをクリアする

Azure Stack Hub オペレーターは、Azure Stack Hub ユーザーが要求したときに、オンデマンドでポータル ユーザー データをクリアできます。 Azure Stack Hub ユーザーは、タイルをピン留めし、ダッシュボードのレイアウトを変更することによって、ポータルをカスタマイズできます。 また、ユーザーは個人の好みに合わせてテーマを変更し、既定の言語を調整することもできます。

ポータル ユーザー データには、Azure Stack Hub ユーザー ポータルでのお気に入りや最近アクセスしたリソースが含まれます。 この記事では、ポータル ユーザー データをクリアする方法について説明します。

ポータル ユーザー設定の削除は、ユーザー サブスクリプションを削除した後にのみ行う必要があります。

> [!NOTE]
> 一部のユーザー データは、この記事のガイダンスに従った後でも、イベント ログのシステム セクションにまだ存在していることがあります。 このデータは、ログが自動的にロール オーバーされるまで、数日間保持される場合があります。

## <a name="prerequisites"></a>前提条件

- [PowerShell for Azure Stack Hub をインストールします](powershell-install-az-module.md)。
- GitHub から[最新の Azure Stack Hub ツールをダウンロード](azure-stack-powershell-download.md)します。
- ユーザー アカウントが、ディレクトリ内にまだ存在している必要があります。
- 管理者の Resource Manager エンドポイントにアクセスするための Azure Stack Hub 管理者資格情報。

> [!NOTE]
> ゲスト ディレクトリ (マルチテナント) から招待されたユーザーからポータル ユーザー情報を削除しようとする場合、そのディレクトリの読み取りアクセス許可が必要です。 詳細については、[この記事で後述する CSP のシナリオ](#clear-portal-user-data-in-guest-directory)を参照してください。

## <a name="clear-portal-user-data-using-a-user-principal-name"></a>ユーザー プリンシパル名を使用してポータル ユーザー データをクリアする

このシナリオでは、既定のプロバイダー サブスクリプションとユーザーが同じディレクトリに属しているか、またはユーザーが存在するディレクトリに対する読み取りアクセス権を持っていることを前提としています。

続行する前に、GitHub から[最新バージョンの Azure Stack Hub ツールをダウンロード](azure-stack-powershell-download.md)してください。

この手順では、Azure Stack Hub の管理者 Resource Manager エンドポイントと通信できるコンピューターを使用します。

1. 管理者特権で Windows PowerShell セッションを開き (管理者として実行)、**AzureStack-Tools-az** ディレクトリ内のルート フォルダーに移動して、必要な PowerShell モジュールをインポートします。

   ```powershell
   Import-Module .\DatacenterIntegration\Portal\PortalUserDataUtilities.psm1
   ```

2. 次のコマンドを実行します。 必ず、プレースホルダーを実際の環境に一致する値に置き換えてください。

   ```powershell
   ## The following Azure Resource Manager endpoint is for the ASDK. If you are in a multinode environment, contact your operator or service provider to get the endpoint.

   $adminARMEndpoint = "https://adminmanagement.local.azurestack.external"

   ## Replace the following value with the Azure Stack Hub directory tenant ID.
   $azureStackDirectoryTenantId = "f5025bf2-547f-4b49-9693-6420c1d5e4ca"

   ## Replace the following value with the user directory tenant ID.
   $userDirectoryTenantId = " 7ddf3648-9671-47fd-b63d-eecd82ed040e"

   ## Replace the following value with name of the user principal whose portal user data is to be cleared.
   $userPrincipalName = "myaccount@contoso.onmicrosoft.com"

   Clear-AzsUserDataWithUserPrincipalName -AzsAdminArmEndpoint $adminARMEndpoint `
    -AzsAdminDirectoryTenantId $azureStackDirectoryTenantId `
    -UserPrincipalName $userPrincipalName `
    -DirectoryTenantId $userDirectoryTenantId
   ```

   > [!NOTE]
   > `azureStackDirectoryTenantId` はオプションです。 この値を指定しない場合、スクリプトによって、Azure Stack Hub に登録されているすべてのテナント ディレクトリでユーザー プリンシパル名が検索され、一致したすべてのユーザーのポータル データがクリアされます。

## <a name="clear-portal-user-data-in-guest-directory"></a>ゲスト ディレクトリ内のポータル ユーザー データをクリアする

このシナリオでは、Azure Stack Hub オペレーターは、ユーザーが存在するゲスト ディレクトリへのアクセス権を持っていません。 これは、クラウド ソリューション プロバイダー (CSP) である場合の一般的なシナリオです。

Azure Stack Hub オペレーターがポータル ユーザー データを削除するには、少なくともユーザーオブジェクト ID が必要です。

ユーザーは、オブジェクト ID のクエリを実行し、それを Azure Stack Hub オペレーターに提供する必要があります。 オペレーターは、ユーザーが存在するディレクトリへのアクセス権を持っていません。

### <a name="user-retrieves-the-user-object-id"></a>ユーザーがユーザー オブジェクト ID を取得する

1. 管理者特権で Windows PowerShell セッションを開き (管理者として実行)、**AzureStack-Tools-az** ディレクトリ内のルート フォルダーに移動して、必要な PowerShell モジュールをインポートします。

   ```powershell
   Import-Module .\DatacenterIntegration\Portal\PortalUserDataUtilities.psm1
   ```

2. 次のコマンドを実行します。 プレースホルダーを実際の環境に一致する値に置き換えてください。

   ```powershell
   ## The following Azure Resource Manager endpoint is for the ASDK. If you are in a multinode environment, contact your operator or service provider to get the endpoint.
   $userARMEndpoint = "https://management.local.azurestack.external"

   ## Replace the following value with the directory tenant ID, which contains the user account.
   $userDirectoryTenantId = "3160cbf5-c227-49dd-8654-86e924c0b72f"

   ## Replace the following value with the name of the user principal whose portal user data is to be cleared.
   $userPrincipleName = "myaccount@contoso.onmicrosoft.com"

   Get-UserObjectId -DirectoryTenantId $userDirectoryTenantId `
    -AzsArmEndpoint $userARMEndpoint `
    -UserPricinpalName $userPrincipleName
   ```

   > [!NOTE]
   > ユーザーは、前のスクリプトの出力であるユーザー オブジェクト ID を Azure Stack Hub オペレーターに提供する必要があります。

## <a name="azure-stack-hub-operator-removes-the-portal-user-data"></a>Azure Stack Hub オペレーターがポータル ユーザー データを削除する

Azure Stack Hub オペレーターは、ユーザー オブジェクト ID を受け取ったら、次のコマンドを実行してポータル ユーザー データを削除します。

1. 管理者特権で Windows PowerShell セッションを開き (管理者として実行)、**AzureStack-Tools-az** ディレクトリ内のルート フォルダーに移動して、必要な PowerShell モジュールをインポートします。

   ```powershell
   Import-Module .\DatacenterIntegration\Portal\PortalUserDataUtilities.psm1
   ```

2. 次のコマンドを実行して、環境に合わせて、パラメーターを調整してください。

   ```powershell
   ## The following Azure Resource Manager endpoint is for the ASDK. If you are in a multinode environment, contact your operator or service provider to get the endpoint.
   $AzsAdminARMEndpoint = "https://adminmanagement.local.azurestack.external"

   ## Replace the following value with the Azure Stack Hub directory tenant ID.
   $AzsAdminDirectoryTenantId = "f5025bf2-547f-4b49-9693-6420c1d5e4ca"

   ## Replace the following value with the directory tenant ID of the user to clear.
   $DirectoryTenantId = "3160cbf5-c227-49dd-8654-86e924c0b72f"

   ## Replace the following value with the name of the user principal whose portal user data is to be cleared.
   $userObjectID = "s-1-*******"
   Clear-AzsUserDataWithUserObject -AzsAdminArmEndpoint $AzsAdminARMEndpoint `
    -AzsAdminDirectoryTenantId $AzsAdminDirectoryTenantId `
    -DirectoryTenantID $DirectoryTenantId `
    -UserObjectID $userObjectID `
   ```

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub を Azure に登録](azure-stack-registration.md)し、ユーザーに提供する項目を [Azure Stack Hub Marketplace](azure-stack-marketplace.md) に設定します。
