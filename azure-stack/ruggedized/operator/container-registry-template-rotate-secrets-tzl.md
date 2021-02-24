---
title: Azure Stack Hub 内でコンテナー レジストリのシークレットをローテーションする | Microsoft Docs
titleSuffix: Azure Stack Hub
description: Azure Stack Hub 内でコンテナー レジストリのシークレットをローテーションする方法について説明します。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 2/1/2021
ms.author: mabrigg
ms.reviewer: chasat
ms.lastreviewed: 12/17/2019
ms.openlocfilehash: 3bfde6e55883e124379353302e77a439c4de0505
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99245638"
---
# <a name="rotate-container-registry-secrets-in-azure-stack-hub"></a>Azure Stack Hub 内でコンテナー レジストリをローテーションする

Azure Stack Hub のユーザーは、コンテナー レジストリ テンプレート デプロイのシークレット (証明書、ユーザー名、パスワード) をローテーションできます。 スクリプトを実行して Microsoft Azure Key Vault 内で新しいシークレット値を設定し、既存のコンテナー レジストリ テンプレート インスタンスを **再デプロイ** することができます。 シークレットを単独でローテーションする場合、新しいデプロイは必要ありません。

## <a name="prerequisites-for-the-user"></a>ユーザーの前提条件

 - ユーザーは、Azure Stack Hub PowerShell モジュールをインストールしておく必要があります。 詳細については、「[PowerShell for Azure Stack のインストール](../../operator/azure-stack-powershell-install.md?toc=https%3A%2F%2Fdocs.microsoft.com%2Fazure-stack%2Fuser%2FTOC.json&bc=https%3A%2F%2Fdocs.microsoft.com%2Fazure-stack%2Fbreadcrumb%2Ftoc.json)」を参照してください。

 - コンテナー レジストリ テンプレート用の更新シークレットを取得します。 新しい SSL 証明書または新しいユーザー名とパスワードの組み合わせを使用して、Docker レジストリにアクセスできます。

 - [msazurestackworkloads/azurestack-gallery](https://github.com/msazurestackworkloads/azurestack-gallery/archive/master.zip) GitHub リポジトリから zip ファイルをダウンロードした後、`\registry\scripts` 内で見つかったスクリプトを取得します。

## <a name="import-new-secrets-into-key-vault"></a>新しいシークレットを Key Vault にインポートする

以下の手順に従って、Key Vault 内で新しいシークレットを設定します。

### <a name="set-updated-registry-user-password-for-existing-username"></a>既存のユーザー名に対して更新済みレジストリのユーザー パスワードを設定する

1.  管理者特権の PowerShell プロンプトを開き、scripts フォルダーから `Import-Module .\\pre-reqs.ps1` を実行します。

2.  既存のレジストリ ユーザーの値を更新するには、コマンドレットを実行します。

    ```powershell  
    Set-RegistryAccessSecret -KeyVaultName newregkv `
        -RegistryUserName <username> `
        -RegistryUserPassword <newpassword> `
        -SkipExistCheck $true
    ```
    たとえば、コマンドレットにより、次の出力が返されます。

    ```powershell  
    PS C:\azurestack-gallery-master\registry\Scripts> Set-RegistryAccessSecret -KeyVaultName newregkv `
        -RegistryUserName admin `
        -RegistryUserPassword password1 `
        -SkipExistCheck $true 
    
    Check if key vault secret name (admin) exists.
    Creating key vault secret name (admin) as it does not exist.
    ```

1.  このレコードに新しい値が入力されたことを検証するには、管理者特権の PowerShell プロンプトを開き、次のコマンドレットを実行します。

    ```powershell  
    Get-AzureKeyVaultSecret -VaultName newregkv -Name admin -IncludeVersions
    ```
    たとえば、コマンドレットにより、次の出力が返されます。
    ```powershell  
    PS C:\azurestack-gallery-master\registry\Scripts> Get-AzureKeyVaultSecret -VaultName newregkv -Name admin -IncludeVersions
    
    
    Vault Name   : newregkv
    Name         : admin
    Version      : 2a1495372c474cc890c888518f02b19f
    Id           : https://newregkv.vault.shanghai.azurestack.corp.microsoft.com:443/secrets/
                   admin/2a1495372c474cc890c888518f02b19f
    Enabled      : True
    Expires      : 
    Not Before   : 
    Created      : 12/18/2019 7:05:56 PM
    Updated      : 12/18/2019 7:05:56 PM
    Content Type : 
    Tags         : 
    
    Vault Name   : newregkv
    Name         : admin
    Version      : 3fd65c1719c74997984648de18a1fa0e
    Id           : https://newregkv.vault.shanghai.azurestack.corp.microsoft.com:443/secrets/
                   admin/3fd65c1719c74997984648de18a1fa0e
    Enabled      : True
    Expires      : 
    Not Before   : 
    Created      : 12/17/2019 5:05:56 AM
    Updated      : 12/17/2019 5:05:56 AM
    Content Type : user credentials
    Tags         : 
    ```

### <a name="set-new-registry-username-and-password"></a>新しいレジストリのユーザー名とパスワードを設定する

1.  管理者特権の PowerShell プロンプトを開き、scripts フォルダーから `Import-Module .\pre-reqs.ps1` を開きます。

2.  新しいユーザー名とパスワードに対して新しいシークレットを作成するには、管理者特権の PowerShell プロンプトを開き、次のコマンドレットを実行します。

    ```powershell  
    Set-RegistryAccessSecret -KeyVaultName newregkv `
        -RegistryUserName <newusername> `
        -RegistryUserPassword <newpassword> 
    ```
    たとえば、コマンドレットにより、次の出力が返されます。
    ```powershell  
    PS C:\azurestack-gallery-master\registry\Scripts> Set-RegistryAccessSecret -KeyVaultName newregkv `
        -RegistryUserName admin1 `
        -RegistryUserPassword password1
    
    Check if key vault secret name (admin1) exists.
    Creating key vault secret name (admin1) as it does not exist. 
    ```

1.  新しいシークレットが作成されたことを検証するには、管理者特権の PowerShell プロンプトを開き、次のコマンドレットを実行します。

    ```powershell  
    Get-AzureKeyVaultSecret -VaultName \<KeyVaultName> -Name \<username>
    ```

    たとえば、コマンドレットにより、次の出力が返されます。

    ```powershell  
    PS C:\azurestack-gallery-master\registry\Scripts> Get-AzureKeyVaultSecret -VaultName newregkv -Name admin1
    
    
    Vault Name   : newregkv
    Name         : admin1
    Version      : 2ae9a7239f4044be82ca9d1e9b80e85a
    Id           : https://newregkv.vault.shanghai.azurestack.corp.microsoft.com:443/secrets/admin1/2ae9a7239f4044be82ca9d1e9b80e85a
    Enabled      : True
    Expires      : 
    Not Before   : 
    Created      : 12/18/2019 11:28:18 PM
    Updated      : 12/18/2019 11:28:18 PM
    Content Type : user credentials
    Tags         : 
    ```

> [!Important]  
> 新しいシークレット (ユーザー名/パスワードの組み合わせ) を作成する場合は、古い Key Vault シークレットを削除する必要があります。 古いシークレットを削除せずに既存のコンテナー レジストリ テンプレートを再デプロイすると、新旧両方のユーザー名とパスワードの組み合わせが、レジストリへのログインに使用できるようになります。

### <a name="update-the-ssl-certificate-for-existing-key-vault-secret"></a>既存の Key Vault シークレットの SSL 証明書を更新する

1. 管理者特権の PowerShell プロンプトを開き、次のコマンドレットを実行します。

    ```powershell
    Set-CertificateSecret -KeyVaultName \<keyvaultname> `
     -CertificateSecretName \<originalsecretnameforcertificate> `
    Set-CertificateSecret -KeyVaultName <keyvaultname> `
        -CertificateSecretName <originalsecretnameforcertificate> `
        -CertificateFilePath <pathtonewcertificate> `
        -CertificatePassword <certificatepassword> `
        -SkipExistCheck $true
    ```

    たとえば、コマンドレットにより、次の出力が返されます。

    ```powershell  
    PS C:\azurestack-gallery-master\registry\Scripts> Set-CertificateSecret -KeyVaultName newregkv `
        -CertificateSecretName containersecret `
        -CertificateFilePath C:\crinstall\shanghairegcertnew.pfx `
        -CertificatePassword <certificatepassword> `
        -SkipExistCheck $true
    Check if key vault secret name (containersecret) exists.
    Creating key vault secret name (containersecret) as it does not exist.
    ----------------------------------------------------------------
    PFX KeyVaultResourceId       : /subscriptions/997da68a-xxxx-xxxx-ad3d-ffeac81b02dc/resourceGroups/newregreg/providers/Microsoft.KeyVault/vaults/newregkv
    PFX KeyVaultSecretUrl        : https://newregkv.vault.shanghai.azurestack.corp.microsoft.com:443/secrets/containersecret/a07ece6b9914408e8f20c516e15b66c9
    PFX Certificate Thumbprint   : 31810AA7FEF1173188691FB3F47208E5389FBA61
    ---------------------------------------------------------------- 
    ```

1.  既存のコンテナー レジストリ テンプレートを再デプロイするときに、この関数によって生成される値を使用します。

2.  既存のシークレットの新しいバージョンが作成されたことを検証するには、管理者特権の PowerShell プロンプトを開き、次のコマンドレットを実行します。

    ```powershell  
    Get-AzureKeyVaultSecret -VaultName <KeyVaultName> -Name <secretname>
    ```

    たとえば、コマンドレットにより、次の出力が返されます。

    ```powershell  
    PS C:\azurestack-gallery-master\registry\Scripts> Get-AzureKeyVaultSecret -VaultName newregkv -Name containersecret -IncludeVersions
    
    
    Vault Name   : newregkv
    Name         : containersecret
    Version      : a07ece6b9914408e8f20c516e15b66c9
    Id           : https://newregkv.vault.shanghai.azurestack.corp.microsoft.com:443/secrets/containersecret/a07ece6b9914408e8f20c516e15b66c9
    Enabled      : True
    Expires      : 
    Not Before   : 
    Created      : 12/18/2019 11:46:28 PM
    Updated      : 12/18/2019 11:46:28 PM
    Content Type : 
    Tags         : 
    
    Vault Name   : newregkv
    Name         : containersecret
    Version      : 0199c7ec1d8d41bb9ddff0f39dca9931
    Id           : https://newregkv.vault.shanghai.azurestack.corp.microsoft.com:443/secrets/containersecret/0199c7ec1d8d41bb9ddff0f39dca9931
    Enabled      : True
    Expires      : 
    Not Before   : 
    Created      : 12/17/2019 5:06:03 AM
    Updated      : 12/17/2019 5:06:03 AM
    Content Type : pfx
    Tags         : 
    ```

### <a name="set-a-new-ssl-certificate-for-the-container-registry-template"></a>コンテナー レジストリ テンプレート用の新しい SSL 証明書を設定する

1.  管理者特権の PowerShell プロンプトを開き、次のコマンドレットを実行します。

    ```powershell  
    Set-CertificateSecret -KeyVaultName <keyvaultname> `
        -CertificateSecretName <newsecretnameforcertificate> `
        -CertificateFilePath <pathtonewcertificate> `
        -CertificatePassword <certificatepassword>
    ```

    たとえば、コマンドレットにより、次の出力が返されます。

    ```powershell  
    PS C:\azurestack-gallery-master\registry\Scripts>    Set-CertificateSecret -KeyVaultName newregkv `
        -CertificateSecretName containersecret121719 `
        -CertificateFilePath C:\crinstall\shanghairegcertnew.pfx `
        -CertificatePassword <certificatepassword> 
    Check if key vault secret name (containersecret121719) exists.
    Creating key vault secret name (containersecret121719) as it does not exist.
    ----------------------------------------------------------------
    PFX KeyVaultResourceId       : /subscriptions/997da68a-xxxx-xxxx-ad3d-ffeac81b02dc/resourc
    eGroups/newregreg/providers/Microsoft.KeyVault/vaults/newregkv
    PFX KeyVaultSecretUrl        : https://newregkv.vault.shanghai.azurestack.corp.microsoft.c
    om:443/secrets/containersecret121719/bb2cfe4df7bc4fbe854a00799afa8566
    PFX Certificate Thumbprint   : 31810AA7FEF1173188691FB3F47208E5389FBA61 
    ```

## <a name="redeploy-existing-container-registry-template"></a>既存のコンテナー レジストリ テンプレートを再デプロイする

1. Azure Stack Hub ユーザー ポータルを開きます。

2.  コンテナー レジストリ テンプレート VM がデプロイされているリソース グループに移動します。

    ![コンテナー レジストリ テンプレート VM がデプロイされているリソース グループを示すスクリーンショット。](./media/container-registry-template-rotating-secrets-tzl/image1.png)

3. **[デプロイ]** の下のデプロイを選択します。

    ![[デプロイ] ページで選択されたデプロイを示すスクリーンショット。](./media/container-registry-template-rotating-secrets-tzl/image2.png)

4.  シークレットを初めてローテーションする場合は、元のデプロイを選択します。 シークレットをローテーションするのが初めてではない場合は、最新のデプロイを選択し、 **[再デプロイ]** を選択します。

    ![[再デプロイ] 操作が強調表示されている [概要] ページを示すスクリーンショット。](./media/container-registry-template-rotating-secrets-tzl/image3.png)

5.  **[ソリューション テンプレートのデプロイ]** で **[既存のものを使用]** を選択し、最初にコンテナー レジストリ テンプレートのデプロイに使用したリソース グループを選択します。 再デプロイを成功させるには、同じリソース グループを使用する必要があります。

    ![[ソリューション テンプレートのデプロイ] ページと [パラメーター] ページを示すスクリーンショット。](./media/container-registry-template-rotating-secrets-tzl/image4.png)

6.  **[パラメーター]** で、パラメーターが元のデプロイと一致することを確認します。 サービス プリンシパル クライアント ID とサービス プリンシパル シークレットを追加する必要があります。

    - レジストリ サービスのユーザー名とパスワードのみをローテーションする場合、必要なのは、サービス プリンシパルのパラメーターを追加することだけです。

    - 証明書をローテーションする場合は、新しいシークレットを設定した結果として出力された PFXKeyVaultSecretURL と PFXThumbprint に対して新しい値を入力する必要があります。

    ![コンテナー レジストリ テンプレート](./media/container-registry-template-rotating-secrets-tzl/image5.png)

7.  **[OK]** 、 **[作成]** の順に選択します。 再デプロイが続行されます。 レジストリ機能は、再デプロイ中も機能し続けます。

    - ユーザー名とパスワードをローテーションする場合、再デプロイが完了したら、レジストリに対してもう一度認証を行う必要があります。

    - 証明書をローテーションする場合は、レジストリへのアクセスが失われることはありません。 これは、信頼される証明書プロバイダーからの証明書を使用していることを前提としています。 プライベート証明書を使用する場合は、アクセスが失われないように、その証明書をクライアントにインストールする必要があります。

## <a name="next-steps"></a>次のステップ

[Azure Stack Marketplace の概要](../../operator/azure-stack-marketplace.md)
