---
title: Azure の登録の検証
titleSuffix: Azure Stack Hub
description: Azure Stack Hub 適合性チェッカーを使用して、Azure 登録を検証する方法について説明します。
author: PatAltimore
ms.topic: how-to
ms.date: 10/19/2020
ms.author: patricka
ms.reviewer: jerskine
ms.lastreviewed: 10/19/2020
ms.openlocfilehash: ffe992c4a2db39f5b2e29d80a002f1486099baaa
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97868796"
---
# <a name="validate-azure-registration"></a>Azure の登録の検証

Azure Stack Hub のデプロイを開始する前に、Azure Stack Hub 適合性チェッカー ツール (**AzsReadinessChecker**) を使用して、Azure Stack Hub で Azure サブスクリプションをすぐに使用できることを検証します。 適合性チェッカーは以下を検証します。

- 使用する Azure サブスクリプションの種類がサポート対象であること。 サブスクリプションは、クラウド ソリューション プロバイダー (CSP) またはマイクロソフトエンタープライズ契約 (EA) である必要があります。
- ご自身のサブスクリプションの登録に使用するアカウントが Azure にサインインでき、サブスクリプション所有者であること。

Azure Stack Hub 登録の詳細については、「[Azure を使用した Azure Stack Hub の登録](azure-stack-registration.md)」を参照してください。

## <a name="get-the-readiness-checker-tool"></a>適合性チェッカー ツールを取得する

最新バージョンの **AzsReadinessChecker** を [PowerShell ギャラリー](https://aka.ms/AzsReadinessChecker)からダウンロードします。  

## <a name="install-and-configure"></a>インストールと構成

### <a name="az-powershell"></a>[Az PowerShell](#tab/az)

### <a name="prerequisites"></a>前提条件

以下の前提条件が必要です。

#### <a name="az-powershell-modules"></a>Az PowerShell モジュール

Az PowerShell モジュールをインストールしておく必要があります。 手順については、「[PowerShell Az プレビュー モジュールをインストールする](powershell-install-az-module.md)」を参照してください。

#### <a name="azure-active-directory-aad-environment"></a>Azure Active Directory (AAD) 環境

- Azure Stack Hub で使用する Azure サブスクリプションの所有者であるアカウントのユーザー名とパスワードを識別します。  
- 使用する Azure サブスクリプションのサブスクリプション ID を識別します。

### <a name="steps-to-validate-the-azure-registration"></a>Azure の登録を検証する手順

1. 管理者特権の PowerShell プロンプトを開き、次のコマンドを実行して、**AzsReadinessChecker** をインストールします。

   ```powershell
   Install-Module -Name Az.BootStrapper -Force -AllowPrerelease
   Install-AzProfile -Profile 2019-03-01-hybrid -Force
   Install-Module -Name Microsoft.AzureStack.ReadinessChecker -AllowPrerelease
   ```

2. PowerShell プロンプトから次のコマンドを実行して、`$subscriptionID` を、使用する Azure サブスクリプションとして設定します。 `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` をお使いのサブスクリプション ID に置き換えます。

   ```powershell
   $subscriptionID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
   ```

3. PowerShell プロンプトから、次のコマンドを実行します。 

   ```powershell
   Connect-AzAccount -subscription $subscriptionID
   ```

4. PowerShell プロンプトから次のコマンドを実行して、お使いのサブスクリプションの検証を開始します。 Azure AD 管理者と Azure AD テナント名を指定します。

   ```powershell
   Invoke-AzsRegistrationValidation  -RegistrationSubscriptionID $subscriptionID
   ```

5. ツールの実行後、出力を確認します。 サインインと登録の両方の要件ついて、状態が適切であることを確認します。 検証が成功した場合の出力は、次の例のように表示されます。

   ```powershell
   Invoke-AzsRegistrationValidation v1.2005.1269 started.
   Checking Registration Requirements: OK

   Log location (contains PII): C:\Users\[*redacted*]\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessChecker.log
   Report location (contains PII): C:\Users\[*redacted*]\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessCheckerReport.json
   Invoke-AzsRegistrationValidation Completed
   ```

### <a name="azurerm-powershell"></a>[AzureRM PowerShell](#tab/rm)

### <a name="prerequisites"></a>前提条件

以下の前提条件が必要です。

#### <a name="azurerm-powershell-modules"></a>AzureRM PowerShell モジュール

Az PowerShell モジュールをインストールしておく必要があります。 手順については、[PowerShell AzureRM モジュールのインストール](powershell-install-az-module.md)に関するページを参照してください。

#### <a name="the-computer-on-which-the-tool-runs"></a>ツールを実行するコンピューター

- インターネットに接続された Windows 10 または Windows Server 2016。
- PowerShell 5.1 以降。 バージョンを確認するには、次の PowerShell コマンドレットを実行し、"**メジャー**" バージョンと "**マイナー**" バージョンを調べます。  
  ```powershell
  $PSVersionTable.PSVersion
  ```
- [Azure Stack Hub 用に構成された PowerShell](powershell-install-az-module.md)。
- 最新バージョンの [Microsoft Azure Stack Hub 適合性チェッカー](https://aka.ms/AzsReadinessChecker) ツール。  

#### <a name="azure-active-directory-azure-ad-environment"></a>Azure Active Directory (Azure AD) 環境

- Azure Stack Hub で使用する Azure サブスクリプションの所有者であるアカウントのユーザー名とパスワードを識別します。  
- 使用する Azure サブスクリプションのサブスクリプション ID を識別します。
- 使用する **AzureEnvironment** を識別します。 環境名のパラメーターとしてサポートされる値は、**AzureCloud**、**AzureChinaCloud**、または **AzureUSGovernment** であり、使用されている Azure サブスクリプションに応じて異なります。

### <a name="steps-to-validate-the-azure-registration"></a>Azure の登録を検証する手順

1. 前提条件を満たしているコンピューターで、管理者特権の PowerShell プロンプトを開き、次のコマンドを実行して、**AzsReadinessChecker** をインストールします。

   ```powershell
   Install-Module Microsoft.AzureStack.ReadinessChecker -Force -AllowPrerelease
   ```

2. PowerShell プロンプトから次のコマンドを実行して、`$registrationCredential` を、サブスクリプション所有者であるアカウントとして設定します。 `subscriptionowner@contoso.onmicrosoft.com` を、お使いのアカウントとテナント名に置き換えます。

   ```powershell
   $registrationCredential = Get-Credential subscriptionowner@contoso.onmicrosoft.com -Message "Enter Credentials for Subscription Owner"
   ```

   > [!NOTE]
   > CSP として、共有サービスまたは IUR サブスクリプションを使用する場合は、それぞれの Azure AD のユーザーの資格情報を指定する必要があります。 これは、通常、`subscriptionowner@iurcontoso.onmicrosoft.com` のようになります。 前の手順で説明したように、そのユーザーは適切な資格情報を持っている必要があります。

3. PowerShell プロンプトから次のコマンドを実行して、`$subscriptionID` を、使用する Azure サブスクリプションとして設定します。 `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` をお使いのサブスクリプション ID に置き換えます。

   ```powershell
   $subscriptionID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
   ```

4. PowerShell プロンプトから次のコマンドを実行して、お使いのサブスクリプションの検証を開始します。

   - `AzureEnvironment` の値を **AzureCloud**、**AzureGermanCloud**、または **AzureChinaCloud** として指定します。  
   - Azure AD 管理者と Azure AD テナント名を指定します。
      ```powershell
      Invoke-AzsRegistrationValidation -RegistrationAccount $registrationCredential -AzureEnvironment AzureCloud -RegistrationSubscriptionID $subscriptionID
      ```

5. ツールの実行後、出力を確認します。 サインインと登録の両方の要件ついて、状態が適切であることを確認します。 検証が成功した場合の出力は、次の例のように表示されます。

   ```powershell
   Invoke-AzsRegistrationValidation v1.1809.1005.1 started.
   Checking Registration Requirements: OK
   Log location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessChecker.log
   Report location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessCheckerReport.json
   Invoke-AzsRegistrationValidation Completed
   ```
---

## <a name="report-and-log-file"></a>レポートとログ ファイル

検証を実行するたびに、結果のログが **AzsReadinessChecker.log** と **AzsReadinessCheckerReport.json** に出力されます。 これらのファイルの場所は、PowerShell に検証結果と共に表示されます。

これらのファイルは、Azure Stack Hub をデプロイする前、または検証に関する問題を調査する前に、検証の状態を共有するときに役立ちます。 両方のファイルに、以降の各検証チェックの結果が保持されます。 デプロイ チームはこのレポートを使用して ID 構成を確認できます。 デプロイ チームやサポート チームは、検証の問題を調査する際に、このログ ファイルを役立たせることができます。

既定では、両方のファイルが `C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessCheckerReport.json` に書き込まれます  

- 別のレポートの場所を指定するには、実行コマンド ラインの末尾に `-OutputPath <path>` パラメーターを使用します。
- ツールの以前の実行に関する情報を **AzsReadinessCheckerReport.json** からクリアするには、実行コマンド ラインの末尾に `-CleanReport` パラメーターを使用します。

詳細については、「[Azure Stack Hub 検証レポート](azure-stack-validation-report.md)」を参照してください。

## <a name="validation-failures"></a>検証エラー

検証チェックに失敗した場合は、エラーの詳細が PowerShell ウィンドウに表示されます。 また、ツールによって、**AzsReadinessChecker.log** ファイルにログ情報が記録されます。

次の例は、一般的な検証エラーの詳細を示します。

### <a name="user-must-be-an-owner-of-the-subscription"></a>ユーザーがサブスクリプションの所有者でなければならない

```shell
Invoke-AzsRegistrationValidation v1.1809.1005.1 started.
Checking Registration Requirements: Fail
Error Details for registration account admin@contoso.onmicrosoft.com:
The user admin@contoso.onmicrosoft.com is role(s) Reader for subscription 3f961d1c-d1fb-40c3-99ba-44524b56df2d. User must be an owner of the subscription to be used for registration.
Additional help URL https://aka.ms/AzsRemediateRegistration

Log location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessChecker.log
Report location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessCheckerReport.json
Invoke-AzsRegistrationValidation Completed
```

**原因** - アカウントが Azure サブスクリプションの管理者ではありません。

**解決策** - Azure サブスクリプション管理者であるアカウントを使用します。これは、Azure Stack Hub デプロイから使用の請求対象となります。

### <a name="expired-or-temporary-password"></a>期限切れまたは一時パスワード

```shell
Invoke-AzsRegistrationValidation v1.1809.1005.1 started.
Checking Registration Requirements: Fail
Error Details for registration account admin@contoso.onmicrosoft.com:
Checking Registration failed with: Retrieving TenantId for subscription [subscription ID] using account admin@contoso.onmicrosoft.com failed with AADSTS50055: Force Change Password.
Trace ID: [Trace ID]
Correlation ID: [Correlation ID]
Timestamp: 2018-10-22 11:16:56Z: The remote server returned an error: (401) Unauthorized.

Log location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessChecker.log
Report location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessCheckerReport.json
Invoke-AzsRegistrationValidation Completed
```

**原因** - パスワードの有効期限が切れているか一時パスワードであるため、アカウントでサインインできません。

**解決策** - PowerShell で次のコマンドを実行し、プロンプトに従ってパスワードをリセットします。

```powershell
Login-AzureRMAccount
```

別の方法として、[Azure portal](https://portal.azure.com) にアカウント所有者としてサインインします。この場合、ユーザーはパスワードの変更を強制されます。

### <a name="unknown-user-type"></a>ユーザーの種類が不明  

```shell
Invoke-AzsRegistrationValidation v1.1809.1005.1 started.
Checking Registration Requirements: Fail
Error Details for registration account admin@contoso.onmicrosoft.com:
Checking Registration failed with: Retrieving TenantId for subscription <subscription ID> using <account> failed with unknown_user_type: Unknown User Type

Log location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessChecker.log
Report location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessCheckerReport.json
Invoke-AzsRegistrationValidation Completed
```

**原因** - 指定した Azure AD 環境にアカウントでサインインできません。 この例では、**AzureChinaCloud** が **AzureEnvironment** として指定されています。  

**解決策** - 指定した Azure 環境に対してアカウントが有効であることを確認します。 PowerShell で次のコマンドを実行して、特定の環境に対してアカウントが有効であることを確認します。

```powershell
Login-AzureRmAccount -EnvironmentName AzureChinaCloud
```

## <a name="next-steps"></a>次の手順

- [Azure ID の検証](azure-stack-validate-identity.md)
- [対応状況レポートを表示する](azure-stack-validation-report.md)
- [Azure Stack Hub の統合に関する一般的な考慮事項](azure-stack-datacenter-integration.md)
