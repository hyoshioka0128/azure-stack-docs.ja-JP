---
title: Azure ID を検証する
titleSuffix: Azure Stack Hub
description: Azure Stack Hub 適合性チェッカーを使用して、Azure ID を検証します。
author: BryanLa
ms.topic: how-to
ms.date: 06/25/2020
ms.author: bryanla
ms.reviewer: jerskine
ms.lastreviewed: 06/25/2020
ms.openlocfilehash: 63f071b50b2e0dac09b5a8ae628001734457d7b8
ms.sourcegitcommit: 28850ae18844213ee410cfe96fc936655b5f6412
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/08/2020
ms.locfileid: "86148116"
---
# <a name="validate-azure-identity"></a>Azure ID の検証

Azure Stack Hub 適合性チェッカー ツール (**AzsReadinessChecker**) を使用して、対象の Azure Active Directory (Azure AD) を Azure Stack Hub で使用する準備が整っていることを検証します。 Azure Stack Hub のデプロイを開始する前に、Azure ID ソリューションを検証します。  

適合性チェッカーは以下を検証します。

- Azure Stack Hub の ID プロバイダーとしての Azure AD。
- 使用予定の Azure AD アカウントで、Azure AD のグローバル管理者としてサインインできます。

Azure Stack Hub のユーザー、アプリケーション、グループ、およびサービス プリンシパルに関する情報を Azure AD に格納する準備が、お使いの環境に整っていることを検証によって確保することができます。

## <a name="get-the-readiness-checker-tool"></a>適合性チェッカー ツールを取得する

最新バージョンの Azure Stack Hub 適合性チェッカー ツール (AzsReadinessChecker) を [PowerShell ギャラリー](https://aka.ms/AzsReadinessChecker)からダウンロードします。

## <a name="install-and-configure"></a>インストールと構成

### <a name="azurerm-powershell"></a>[AzureRM PowerShell](#tab/rm)

## <a name="prerequisites"></a>前提条件

以下の前提条件が必要です。

#### <a name="azurerm-powershell-modules"></a>AzureRM PowerShell モジュール

Az PowerShell モジュールをインストールしておく必要があります。 手順については、[PowerShell AzureRM モジュールのインストール](azure-stack-powershell-install.md)に関するページを参照してください。

#### <a name="the-computer-on-which-the-tool-runs"></a>ツールを実行するコンピューター

- インターネットに接続された Windows 10 または Windows Server 2016。
- PowerShell 5.1 以降。 お使いのバージョンを確認するには、次の PowerShell コマンドを実行し、"**メジャー**" バージョンと "**マイナー**" バージョンを確かめます。  
  ```powershell
  $PSVersionTable.PSVersion
  ```
- [Azure Stack Hub 用に構成された PowerShell](azure-stack-powershell-install.md)。
- 最新バージョンの [Microsoft Azure Stack Hub 適合性チェッカー](https://aka.ms/AzsReadinessChecker) ツール。

#### <a name="azure-ad-environment"></a>Azure AD 環境

- Azure Stack Hub に使用する Azure AD アカウントを特定し、それが確実に Azure AD グローバル管理者であるようにします。
- Azure AD テナントの名前を特定します。 テナント名は、Azure AD のプライマリ ドメイン名である必要があります (例: **contoso.onmicrosoft.com**)。
- 使用する Azure 環境を特定します。 環境名のパラメーターとしてサポートされる値は、**AzureCloud**、**AzureChinaCloud**、または **AzureUSGovernment** です。使用している Azure サブスクリプションに応じて異なります。

## <a name="steps-to-validate-azure-identity"></a>Azure ID を検証する手順

1. 前提条件を満たしているコンピューターで、管理者特権の PowerShell コマンド プロンプトを開き、次のコマンドを実行して、**AzsReadinessChecker** をインストールします。  

   ```powershell
   Install-Module Microsoft.AzureStack.ReadinessChecker -Force
   ```

2. PowerShell コマンド プロンプトから次を実行して、`$serviceAdminCredential` を、お使いの Azure AD テナントのサービス管理者として設定します。  `serviceadmin\@contoso.onmicrosoft.com` を、お使いのアカウントとテナント名に置き換えます。

   ```powershell
   $serviceAdminCredential = Get-Credential serviceadmin@contoso.onmicrosoft.com -Message "Enter credentials for service administrator of Azure Active Directory tenant"
   ```

3. PowerShell プロンプトから次のコマンドを実行して、Azure AD の検証を開始します。

   - **AzureEnvironment** の環境名の値を指定します。 環境名のパラメーターとしてサポートされる値は、**AzureCloud**、**AzureChinaCloud**、または **AzureUSGovernment** です。使用している Azure サブスクリプションに応じて異なります。
   - `contoso.onmicrosoft.com` を Azure AD テナント名に置き換えます。

   ```powershell
   Invoke-AzsAzureIdentityValidation -AADServiceAdministrator $serviceAdminCredential -AzureEnvironment <environment name> -AADDirectoryTenantName contoso.onmicrosoft.com
   ```

4. ツールの実行後、出力を確認します。 インストールの要件について、状態が **OK** であることを確認します。 次の図のように、検証が成功したことが表示されます。

   ```powershell
   Invoke-AzsAzureIdentityValidation v1.1809.1005.1 started.
   Starting Azure Identity Validation

   Checking Installation Requirements: OK

   Finished Azure Identity Validation

   Log location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessChecker.log
   Report location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessCheckerReport.json
   Invoke-AzsAzureIdentityValidation Completed
   ```


### <a name="az-powershell"></a>[Az PowerShell](#tab/az)

### <a name="prerequisites"></a>前提条件

以下の前提条件が必要です。

#### <a name="az-powershell-modules"></a>Az PowerShell モジュール

Az PowerShell モジュールをインストールしておく必要があります。 手順については、「[PowerShell Az プレビュー モジュールをインストールする](powershell-install-az-module.md)」を参照してください。

#### <a name="azure-active-directory-azure-ad-environment"></a>Azure Active Directory (Azure AD) 環境

- Azure Stack Hub に使用する Azure AD アカウントを特定し、それが確実に Azure AD グローバル管理者であるようにします。
- Azure AD テナントの名前を特定します。 テナント名は、Azure AD のプライマリ ドメイン名である必要があります (例: **contoso.onmicrosoft.com**)。

### <a name="steps-to-validate-azure-identity"></a>Azure ID を検証する手順

1. 前提条件を満たしているコンピューターで、管理者特権の PowerShell コマンド プロンプトを開き、次のコマンドを実行して、**AzsReadinessChecker** をインストールします。  

   ```powershell
   Install-Module -Name Az.BootStrapper -Force -AllowPrerelease
   Install-AzProfile -Profile 2019-03-01-hybrid -Force
   Install-Module -Name Microsoft.AzureStack.ReadinessChecker -AllowPrerelease
   ```

2. PowerShell プロンプトから、次のコマンドを実行します。 `contoso.onmicrosoft.com` を Azure AD テナント名に置き換えます。

   ```powershell
   Connect-AzAccount -tenant contoso.onmicrosoft.com
   ```

3. PowerShell プロンプトから次のコマンドを実行して、Azure AD の検証を開始します。 `contoso.onmicrosoft.com` を Azure AD テナント名に置き換えます。

   ```powershell
   Invoke-AzsAzureIdentityValidation -AADDirectoryTenantName contoso.onmicrosoft.com 
   ```

4. ツールの実行後、出力を確認します。 インストールの要件について、状態が **OK** であることを確認します。 次の図のように、検証が成功したことが表示されます。

   ```powershell
   Invoke-AzsAzureIdentityValidation v1.2005.1269 started.
   Starting Azure Identity Validation

   Checking Installation Requirements: OK

   Finished Azure Identity Validation

   Log location (contains PII): C:\Users\[*redacted*]\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessChecker.log
   Report location (contains PII): C:\Users\[*redacted*]\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessCheckerReport.json
   Invoke-AzsAzureIdentityValidation Completed
   ```

--- 


## <a name="report-and-log-file"></a>レポートとログ ファイル

検証を実行するたびに、結果のログが **AzsReadinessChecker.log** と **AzsReadinessCheckerReport.json** に出力されます。 これらのファイルの場所は、PowerShell に検証結果と共に表示されます。

これらのファイルは、Azure Stack Hub をデプロイする前、または検証に関する問題を調査する前に、検証の状態を共有するときに役立ちます。 両方のファイルに、以降の各検証チェックの結果が保持されます。 デプロイ チームはこのレポートを使用して ID 構成を確認できます。 デプロイ チームやサポート チームは、検証の問題を調査する際に、このログ ファイルを役立たせることができます。

既定では、両方のファイルが `C:\Users\<username>\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessCheckerReport.json` に書き込まれます  

- 別のレポートの場所を指定するには、実行コマンド ラインの末尾に `-OutputPath <path>` パラメーターを使用します。
- ツールの以前の実行に関する情報を **AzsReadinessCheckerReport.json** からクリアするには、実行コマンド ラインの末尾に `-CleanReport` パラメーターを使用します。

詳細については、「[Azure Stack Hub 検証レポート](azure-stack-validation-report.md)」を参照してください。

## <a name="validation-failures"></a>検証エラー

検証チェックに失敗した場合は、エラーの詳細が PowerShell ウィンドウに表示されます。 また、ツールによって、AzsReadinessChecker.log ファイルにログ情報が記録されます。

次の例は、一般的な検証エラーに関するガイダンスです。

### <a name="expired-or-temporary-password"></a>期限切れまたは一時パスワード

```powershell
Invoke-AzsAzureIdentityValidation v1.1809.1005.1 started.
Starting Azure Identity Validation

Checking Installation Requirements: Fail
Error Details for Service Administrator Account admin@contoso.onmicrosoft.com
The password for account  has expired or is a temporary password that needs to be reset before continuing. Run Login-AzureRMAccount, login with  credentials and follow the prompts to reset.
Additional help URL https://aka.ms/AzsRemediateAzureIdentity

Finished Azure Identity Validation

Log location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessChecker.log
Report location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessCheckerReport.json
Invoke-AzsAzureIdentityValidation Completed
```

**原因** - パスワードの有効期限が切れているか一時パスワードであるため、アカウントでサインインできません。

**解決策** - PowerShell で次のコマンドを実行し、プロンプトに従ってパスワードをリセットします。

```powershell
Login-AzureRMAccount
```

別の方法として、[Azure portal](https://portal.azure.com) にアカウント所有者としてサインインすることがあります。この場合、ユーザーはパスワードの変更を強制されます。

### <a name="unknown-user-type"></a>ユーザーの種類が不明 
 
```powershell
Invoke-AzsAzureIdentityValidation v1.1809.1005.1 started.
Starting Azure Identity Validation

Checking Installation Requirements: Fail
Error Details for Service Administrator Account admin@contoso.onmicrosoft.com
Unknown user type detected. Check the account  is valid for AzureChinaCloud
Additional help URL https://aka.ms/AzsRemediateAzureIdentity

Finished Azure Identity Validation

Log location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessChecker.log
Report location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessCheckerReport.json
Invoke-AzsAzureIdentityValidation Completed
```

**原因** - 指定した Azure AD (**AADDirectoryTenantName**) にアカウントでサインインできません。 この例では、**AzureChinaCloud** が **AzureEnvironment** として指定されています。

**解決策** - 指定した Azure 環境に対してアカウントが有効であることを確認します。 PowerShell で次のコマンドを実行して、特定の環境に対してアカウントが有効であることを確認します。

```powershell
Login-AzureRmAccount -EnvironmentName AzureChinaCloud
```

### <a name="account-is-not-an-administrator"></a>アカウントが管理者ではない

```powershell
Invoke-AzsAzureIdentityValidation v1.1809.1005.1 started.
Starting Azure Identity Validation

Checking Installation Requirements: Fail
Error Details for Service Administrator Account admin@contoso.onmicrosoft.com
The Service Admin account you entered 'admin@contoso.onmicrosoft.com' is not an administrator of the Azure Active Directory tenant 'contoso.onmicrosoft.com'.
Additional help URL https://aka.ms/AzsRemediateAzureIdentity

Finished Azure Identity Validation

Log location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessChecker.log
Report location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessCheckerReport.json
Invoke-AzsAzureIdentityValidation Completed
```

**原因** - アカウントで正常にサインインできますが、そのアカウントは Azure AD (**AADDirectoryTenantName**) の管理者ではありません。  

**解決策** - [Azure portal](https://portal.azure.com) にアカウント所有者としてサインインし、 **[Azure Active Directory]** 、 **[ユーザー]** 、 **[ユーザーの選択]** の順に移動します。 次に、 **[ディレクトリ ロール]** を選択して、ユーザーが**グローバル管理者**であることを確認します。 アカウントが**ユーザー**である場合は、 **[Azure Active Directory]**  >  **[カスタム ドメイン名]** に移動し、**AADDirectoryTenantName** に対して指定した名前が、このディレクトリのプライマリ ドメイン名としてマークされていることを確認します。 この例では、**contoso.onmicrosoft.com** です。

Azure Stack Hub では、ドメイン名がプライマリ ドメイン名である必要があります。

## <a name="next-steps"></a>次の手順

[Azure の登録を検証する](azure-stack-validate-registration.md)  
[対応状況レポートを表示する](azure-stack-validation-report.md)  
[Azure Stack Hub の統合に関する一般的な考慮事項](azure-stack-datacenter-integration.md)  
