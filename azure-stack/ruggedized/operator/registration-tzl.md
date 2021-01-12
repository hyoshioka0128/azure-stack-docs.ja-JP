---
title: Azure Stack Hub を Azure に登録する - ラグド
description: Azure Marketplace 項目をダウンロードし、データ レポートを設定できるように、Azure Stack Hub を Azure に登録する方法について説明します。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/21/2020
ms.author: sethm
ms.reviewer: alfredop
ms.lastreviewed: 12/20/2019
ms.openlocfilehash: 3d24ba3a714c8ea8f17189a7c63a134dab0f8d12
ms.sourcegitcommit: d719f148005e904fa426a001a687e80730c91fda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/06/2021
ms.locfileid: "97909805"
---
# <a name="register-azure-stack-hub-with-azure---azure-stack-hub-ruggedized"></a>Azure Stack Hub を Azure に登録する - Azure Stack Hub ラグド

Marketplace シンジケーションを設定し、PaaS サービスを使用するには、デプロイの完了後に、Azure Stack Hub に基づく Modular Data Center (MDC) または Azure Stack Hub ラグドを登録し、アクティブにする必要があります。 管理者は Marketplace シンジケーションを使用して、ローカルの Azure Stack Hub Marketplace に、Azure Marketplace からダウンロードしたイメージを設定します。

Azure クラウドに接続するシステムと切断されるシステムの登録が必要です。

## <a name="overview"></a>概要

登録プロセスは、次の手順で構成されます。

1. Azure Stack Hub から登録トークンをエクスポートします。
2. Azure Stack Hub を Azure に登録します。
3. Azure からアクティブ化キーをエクスポートします。
4. Azure Stack Hub をアクティブ化します。

システムが Azure クラウドに接続されている場合は、1 つのスクリプトで 4 つの手順をすべて実行できます。 システムが切断されている場合、管理者は手順を個別に実行する必要があります。

> [!NOTE]
> MDC の場合、各 Azure Stack Hub システムを個別に登録する必要があります。 そのため、以下の手順を 3 回実行する必要があります。

## <a name="prerequisites"></a>前提条件

登録する前に、次の前提条件を満たしておく必要があります。

- 資格情報を確認する。
- PowerShell 言語モードを設定する。
- PowerShell for Azure Stack Hub をインストールする。
- Azure Stack Hub ツール モジュールをインストールする
- 登録シナリオを決定する。

### <a name="verify-your-credentials"></a>資格情報を確認する

Azure を使用して Azure Stack Hub を登録する前に、以下のものが必要です。

- Azure サブスクリプションのサブスクリプション ID。  

    > [!NOTE]  
    > Azure サブスクリプションは、Azure クラウド環境 (Azure Commercial、Azure Government など) に関連付けられています。これにより、Marketplace コンテンツにアクセスするために接続するクラウドが決まります。

- サブスクリプションの所有者であるアカウントのユーザー名とパスワード。 
- ユーザー アカウントは Azure サブスクリプションにアクセスできなければならず、そのサブスクリプションに関連付けられているディレクトリで ID アプリとサービス プリンシパルを作成するアクセス許可を持つ必要があります。 最低限の特権による管理を使用し、Azure Stack Hub を Azure に登録することをお勧めします。 登録のためのサブスクリプションへのアクセスを制限するカスタム ロール定義を作成する方法の詳細については、[Azure Stack Hub の登録ロールの作成](../../operator/azure-stack-registration-role.md)に関する記事をご覧ください。
- Azure Stack Hub リソース プロバイダーを登録します (詳細については、以下のセクションをご覧ください)。

登録後、Azure Active Directory (Azure AD) グローバル管理者のアクセス許可は不要です。 ただし、一部の操作では、グローバル管理者の資格情報が必要な場合があります (リソース プロバイダーのインストーラー スクリプトや、アクセス許可の付与を必要とする新機能など)。 アカウントのグローバル管理者のアクセス許可を一時的に復帰させるか、"*既定のプロバイダー サブスクリプション*" の所有者である別のグローバル管理者アカウントを使用できます。

Azure Stack Hub を登録するユーザーは、Azure AD のサービス プリンシパルの所有者です。 Azure Stack Hub の登録を変更できるのは、Azure Stack Hub を登録したユーザーのみです。 登録サービス プリンシパルの所有者ではない管理者以外のユーザーが Azure Stack Hub を登録または再登録しようとすると、403 応答が発生する可能性があります。 403 応答は、操作を完了するための十分なアクセス許可がユーザーにないことを示します。

Azure Stack Hub を登録しても、Azure サブスクリプションに課金されることはありません。

### <a name="powershell-language-mode"></a>PowerShell 言語モード

Azure Stack Hub を正常に登録するには、PowerShell 言語モードを **FullLanguageMode** に設定する必要があります。 現在の言語モードが完全に設定されていることを確認するには、PowerShell ウィンドウを管理者特権で開いて、次の PowerShell コマンドレットを実行します。

```powershell  
$ExecutionContext.SessionState.LanguageMode
```

出力で **FullLanguageMode** が返されていることを確認します。 その他の言語モードが返されている場合は、別のマシン上で登録を実行するか、言語モードを **FullLanguageMode** に設定してから作業を続行する必要があります。

### <a name="install-powershell-for-azure-stack-hub"></a>PowerShell for Azure Stack Hub をインストールする

Azure に登録するために、最新の PowerShell for Azure Stack Hub を使用します。

最新バージョンがまだインストールされていない場合は、[PowerShell for Azure Stack Hub のインストール](../../operator/azure-stack-powershell-install.md)に関する記事をご覧ください。

### <a name="install-the-azure-stack-hub-tools-module"></a>Azure Stack Hub ツール モジュールをインストールする

[Azure Stack Hub ツール モジュール](https://www.powershellgallery.com/packages/azs.tools.admin/0.1.0)には、Azure Stack Hub 機能 (登録機能を含む) をサポートする PowerShell モジュールが含まれています。 登録プロセス中に、**RegisterWithAzure.psm1** サブモジュールをインポートし、これを使用して Azure Stack Hub インスタンスを Azure に登録する必要があります。

最新の Azure Stack Hub ツールをインストールするには、次の手順に従います。

1. 管理者特権の PowerShell プロンプトを開きます。
2. 次のコマンドレットを実行します。

   ```powershell  
   Install-Module -Name Azs.Tools.Admin
   ```

### <a name="determine-your-registration-scenario"></a>登録シナリオを決定する

Azure Stack Hub のデプロイは、"*接続*" デプロイまたは "*切断*" デプロイになります。

- **接続済み**: "接続" とは、インターネットと Azure に接続できるように Azure Stack Hub をデプロイしたことを意味します。 ID ストアには、Azure AD または Active Directory フェデレーション サービス (AD FS) を使用できます。

- **切断**: Azure からの切断デプロイ オプションを使用すると、インターネットへの接続なしで Azure Stack Hub をデプロイして使用できます。 切断されているシステムでは PaaS の使用状況を Azure に報告できないため、PaaS サービスを使用するためには、**容量** 課金モデルとして登録する必要があります。

### <a name="determine-a-unique-registration-name-to-use"></a>使用する一意の登録名の判別

Azure Stack Hub を Azure に登録するときに、一意の登録名を指定する必要があります。 Azure の登録に Azure Stack Hub サブスクリプションを関連付ける簡単な方法は、Azure Stack Hub の **クラウド ID** を使用することです。

Azure Stack Hub デプロイのクラウド ID を調べるには、特権エンドポイントにアクセスできるコンピューターで管理者として PowerShell を開き、次のコマンドを実行して、**CloudID** の値を書き留めます。

```PowerShell
Enter-PSSession -ComputerName <privileged endpoint computer name> -ConfigurationName PrivilegedEndpoint
Get-AzureStackStampInformation
```

## <a name="registration-and-activation-for-systems-connected-to-the-azure-cloud"></a>Azure クラウドに接続されているシステムの登録とアクティブ化

Azure に接続されている Azure Stack Hub システムを登録するには、次の手順を使用します。

> [!NOTE]  
> これらのすべての手順は、特権エンドポイント (PEP) にアクセスできるコンピューターから実行する必要があります。 PEP の詳細については、[Azure Stack Hub での特権エンドポイントの使用](../../operator/azure-stack-privileged-endpoint.md)に関するページを参照してください。

接続された環境では、インターネットと Azure にアクセスできます。 これらの環境では、Azure Stack Hub リソース プロバイダーを Azure に登録してから、課金モデルを構成します。

1. Azure Stack Hub リソース プロバイダーを Azure に登録するには、PowerShell ISE を管理者として起動し、**EnvironmentName** パラメーターを適切な Azure サブスクリプションの種類に設定して (下記のパラメーターを参照)、次の PowerShell コマンドレットを使用します。

2. 同じ PowerShell セッションで、正しい Azure PowerShell コンテキストにサインインしていることを確認します。 このコンテキストは、以前に Azure Stack Hub リソース プロバイダーを登録するために使用された Azure アカウントです。

   ```powershell
   Connect-AzureRmAccount -Environment "<environment name>"
   ```

   **AzureUSSec** の場合は、まず `CustomCloud` 環境を初期化してから、**Connect-AzureRmAccount** を呼び出す必要があります。

   ```powershell
   Initialize-AzureRmEnvironment -Name 'CustomCloud' -CloudManifestFilePath $CloudManifestFilePath
   Connect-AzureRmAccount -Environment 'CustomCloud'
   ```

   | パラメーター | 説明 |  
   |-----|-----|
   | EnvironmentName | Azure クラウド サブスクリプション環境名。 サポートされる環境名は、**AzureCloud**、**AzureUSGovernment**、または **AzureUSSec** です。   |

   > [!NOTE]
   > セッションの有効期限が切れた場合、パスワードが変更された場合、またはアカウントを切り替える場合は、**Add-AzureRmAccount** を使用してサインインする前に、次のコマンドレットを実行します: **Remove-AzureRmAccount-Scope Process**。

3. 複数のサブスクリプションがある場合は、次のコマンドを実行して、使用するものを選択します。

   ```powershell
   Get-AzureRmSubscription -SubscriptionID '<Your Azure Subscription GUID>' | Select-AzureRmSubscription
   ```

4. Azure サブスクリプションで Azure Stack Hub リソース プロバイダーを登録するには、次のコマンドを実行します。

   ```powershell  
   Register-AzureRmResourceProvider -ProviderNamespace Microsoft.AzureStack
   ```

   Azure portal を使用して、Azure サブスクリプションで Azure Stack Hub リソース プロバイダーを登録することもできます。 Azure サブスクリプションに関連付けられているアカウントを使用して Azure portal にログインします。 **[カテゴリ]**  >  **[一般]**  >  **[サブスクリプション]** に移動し、Azure Stack Hub リソース プロバイダーを登録するために必要なサブスクリプション ID を選択します。 左側のウィンドウで、 **[設定]**  >  **[リソース プロバイダー]** に移動します。 **[Microsoft.AzureStack]** リソース プロバイダーを選択し、 **[登録]** を選択します。

   ![Azure Stack Hub リソース プロバイダーの登録](./media/registration-tzl/register-azure-resource-provider-portal.png)

5. 同じ PowerShell セッションで **Set-AzsRegistration** コマンドレットを実行します。

   ```powershell  
   $CloudAdminCred = Get-Credential -UserName <Privileged endpoint credentials> -Message "Enter the cloud domain credentials to access the privileged endpoint."
   $msAssetTag = "Enter the value printed on the product"
   $RegistrationName = "<unique-registration-name>"

   Set-AzsRegistration `
      -PrivilegedEndpointCredential $CloudAdminCred `
      -PrivilegedEndpoint <PrivilegedEndPoint computer name> `
      -BillingModel Ruggedized `
      -RegistrationName $RegistrationName `
      -msAssetTag $msAssetTagName `
      -UsageReportingEnabled: $false
   ```

   MS Asset タグ (`msAssetTag`) はラグド課金モデルの登録に必須であり、製品に印刷されています。

   このプロセスには 10 - 15 分かかります。 コマンドが完了すると、 **「Your environment is now registered and activated using the provided parameters. (提供されたパラメーターを使用して環境が登録され、アクティブ化されました。)」** というメッセージが表示されます。

## <a name="registration-and-activation-for-systems-not-connected-to-the-azure-cloud"></a>Azure クラウドに接続されていないシステムの登録とアクティブ化 

(インターネットに接続されていない) 切断されている環境で Azure Stack Hub を登録する場合は、次の手順を実行します。

1. Azure Stack Hub 環境から登録トークンを取得します。

2. Azure に接続し、手順 1. の登録トークンを使用して登録します。 Azure に接続でき、PowerShell for Azure Stack Hub がインストールされているコンピューターを使用します。

3. Azure の登録からアクティブ化キーを取得します。

4. Azure のアクティブ化キーを使用して Azure Stack Hub をアクティブにします。

### <a name="get-a-registration-token-from-the-azure-stack-hub-environment"></a>Azure Stack Hub 環境から登録トークンを取得する

Azure Stack Hub 環境から登録トークンを取得します。 次に、Azure に接続でき、PowerShell for Azure Stack Hub と Azure Stack Hub ツールがインストールされているコンピューターでそのトークンを使用します。

1. 登録トークンを取得するには、管理者特権でのプロンプトから次の PowerShell コマンドレットを実行します。

   ```PowerShell
    $FilePathForRegistrationToken = "$env:SystemDrive\RegistrationToken.txt" 
    
    $RegistrationToken = Get-AzsRegistrationToken `
      -PrivilegedEndpointCredential $YourCloudAdminCredential `
      -UsageReportingEnabled:$False `
      -PrivilegedEndpoint $YourPrivilegedEndpoint `
      -BillingModel Capacity -AgreementNumber '<EA agreement number>' -msAssetTag '<MS Asset tag>' `
      -TokenOutputFilePath $FilePathForRegistrationToken 
   ```

   > [!TIP]  
   > 登録トークンは、`$FilePathForRegistrationToken` で指定されたファイルに保存されます。 必要に応じて、パスまたはファイル名を変更できます。

2. 接続された Azure マシンで使用するためにこの登録トークンを保存します。 `$FilePathForRegistrationToken` からファイルまたはテキストをコピーできます。

### <a name="connect-to-azure-and-register"></a>Azure に接続して登録する

インターネットに接続されているコンピューターで、正しい Azure PowerShell コンテキストにサインインします。 次に、**Register-AzsEnvironment** を呼び出します。 Azure に登録する登録トークンを指定します。 同じ Azure サブスクリプション ID を使用して Azure Stack Hub の複数のインスタンスを登録する場合は、一意の登録名を指定します。

自分の登録トークンと、一意のトークン名が必要です。

1. 管理者特権での PowerShell プロンプトから、次のコマンドレットを実行して、適切なサブスクリプションを使用していることを確認します。

    ```powershell  
    Get-AzureRmSubscription -SubscriptionID '<Your Azure Subscription GUID>' | Select-AzureRmSubscription
    ```

2. その後、次の PowerShell コマンドレットを実行します。

    ```powershell  
    $RegistrationToken = "<Your Registration Token>"
    $RegistrationName = "<unique-registration-name>"
    Register-AzsEnvironment -RegistrationToken $RegistrationToken -RegistrationName $RegistrationName
    ```

3. 必要に応じて、**Get-Content** コマンドレットを使用して、登録トークンが含まれているファイルを参照できます。

   自分の登録トークンと、一意のトークン名が必要です。

   1. 管理者として PowerShell ISE を起動し、Azure Stack Hub ツールをダウンロードしたときに作成された **AzureStack-Tools-master** ディレクトリ内の **Registration** フォルダーに移動します。 **RegisterWithAzure.psm1** モジュールをインポートします。

      ```powershell  
      Import-Module .\RegisterWithAzure.psm1
      ```

   2. その後、次の PowerShell コマンドレットを実行します。

      ```powershell  
      $RegistrationToken = Get-Content -Path '<Path>\<Registration Token File>'
      Register-AzsEnvironment -RegistrationToken $RegistrationToken -RegistrationName $RegistrationName
      ```

> [!NOTE]  
> 後で参照できるように登録リソース名および登録トークンを保存します。

### <a name="retrieve-an-activation-key-from-azure-registration-resource"></a>Azure の登録リソースからアクティブ化キーを取得する

次に、**Register-AzsEnvironment** 中に Azure に作成された登録リソースからアクティブ化キーを取得します。

アクティブ化キーを取得するには、次の PowerShell コマンドレットを実行します。

```PowerShell
$RegistrationResourceName = "<unique-registration-name>"
$KeyOutputFilePath = "$env:SystemDrive\ActivationKey.txt"
$ActivationKey = Get-AzsActivationKey -RegistrationName $RegistrationResourceName -KeyOutputFilePath $KeyOutputFilePath
```

> [!TIP]  
> アクティブ化キーは、`$KeyOutputFilePath` で指定されたファイルに保存されます。 必要に応じて、パスまたはファイル名を変更できます。

### <a name="create-an-activation-resource-in-azure-stack-hub"></a>Azure Stack Hub にアクティブ化リソースを作成する

**Get-AzsActivationKey** によって作成されたアクティブ化キーのファイルまたはテキストを使用して Azure Stack Hub 環境に戻ります。 次に、そのアクティブ化キーを使用して Azure Stack Hub にアクティブ化リソースを作成します。 アクティブ化リソースを作成するには、次の PowerShell コマンドレットを実行します。

```PowerShell
$ActivationKey = "<activation key>"
New-AzsActivationResource -PrivilegedEndpointCredential $YourCloudAdminCredential -PrivilegedEndpoint $YourPrivilegedEndpoint -ActivationKey $ActivationKey
```

必要に応じて、**Get-Content** コマンドレットを使用して、登録トークンが含まれているファイルを参照できます。

```PowerShell
$ActivationKey = Get-Content -Path '<Path>\<Activation Key File>'
New-AzsActivationResource -PrivilegedEndpointCredential $YourCloudAdminCredential -PrivilegedEndpoint $YourPrivilegedEndpoint -ActivationKey $ActivationKey
```

## <a name="verify-azure-stack-hub-registration"></a>Azure Stack Hub の登録を検証する

Azure Stack Hub の登録に成功したことは、 **[Region management]\(リージョン管理\)** タイルを使用して確認できます。 このタイルは、管理者ポータルの既定のダッシュボードにあります。 状態には、登録済みと未登録とがあります。 登録済みである場合は、Azure Stack Hub の登録に使用した Azure サブスクリプション ID が、登録のリソース グループおよび名前と共に表示されます。

1. Azure Stack Hub 管理者ポータルにサインインします。 URL はオペレーターのリージョンと外部ドメイン名によって異なり、`https://adminportal.<region>.<FQDN>` 形式になります。

2. ダッシュボードで、 **[Region management]\(リージョン管理\)** を選択します。

3. **[プロパティ]** を選択します。 このブレードには、環境の状態と詳細が表示されます。 状態には、 **[登録済み]** 、 **[未登録]** または **[有効期限切れ]** があります。

   [![Azure Stack Hub 管理者ポータルの [リージョンの管理] タイル](media/registration-tzl/admin1sm.png "[Region management]\(リージョン管理\) タイル")](media/registration-tzl/admin1.png#lightbox)

   登録した場合、プロパティには以下が含まれます。

    - **登録サブスクリプション ID**:登録され、Azure Stack Hub に関連付けられた Azure サブスクリプション ID。
    - **登録リソース グループ**:Azure Stack Hub リソースを含む関連付けられているサブスクリプション内の Azure リソース グループ。

4. Azure portal を使用して Azure Stack Hub 登録リソースを表示し、登録が成功したことを確認できます。 Azure Stack Hub の登録に使用したサブスクリプションに関連付けられているアカウントを使用して、[Azure portal](https://portal.azure.com/) にサインインします。 **[すべてのリソース]** を選択し、 **[非表示の型の表示]** チェックボックスをオンにして、登録名を選択します。

5. 登録が成功しなかった場合、この問題を解決するには、[使用するサブスクリプションを変更](https://docs.microsoft.com/azure-stack/operator/azure-stack-registration#change-the-subscription-you-use)して再登録する必要があります。

また、登録が成功したかどうかは、Marketplace の管理機能を使用して確認することもできます。 **[Marketplace management]\(Marketplace の管理\)** ブレードに Marketplace 項目の一覧が表示されたら、登録は成功しています。 ただし、切断されている環境では、[Marketplace management]\(Marketplace の管理\) に Marketplace 項目は表示されません。

> [!NOTE]
> 登録が完了すると、登録されていないことを示すアクティブな警告は表示されなくなります。

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub の管理の基本](../../operator/azure-stack-manage-basics.md)  
