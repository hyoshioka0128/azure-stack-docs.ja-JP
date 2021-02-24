---
title: AD FS ID を Azure Stack Hub データセンターに統合する
description: Azure Stack Hub の AD FS ID プロバイダーを、ご利用のデータセンターの AD FS と統合する方法を学習します。
author: BryanLa
ms.topic: article
ms.date: 04/10/2020
ms.author: bryanla
ms.reviewer: thoroet
ms.lastreviewed: 05/10/2019
ms.custom: conteperfq4
ms.openlocfilehash: 971bac83972664bbefe900f16aa2ab83c12aa3a1
ms.sourcegitcommit: 32d77de1a554315f53473407279e464a72aa9aa1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/15/2020
ms.locfileid: "97515049"
---
# <a name="integrate-ad-fs-identity-with-your-azure-stack-hub-datacenter"></a>AD FS ID を Azure Stack Hub データセンターに統合する

ID プロバイダーとして、Azure Active Directory (Azure AD) または Active Directory フェデレーション サービス (AD FS) を使用して、Azure Stack Hub をデプロイすることができます。 Azure Stack Hub をデプロイする前に、選択を行う必要があります。 接続されているシナリオでは、Azure AD または AD FS を選択できます。 切断されたシナリオでは、AD FS のみがサポートされます。 この記事では、Azure Stack Hub の AD FS をデータセンターの AD FS と統合する方法を示します。

> [!IMPORTANT]
> Azure Stack Hub ソリューション全体を再デプロイせずに、ID プロバイダーを切り替えることはできません。

## <a name="active-directory-federation-services-and-graph"></a>Active Directory フェデレーション サービス (AD FS) と Graph

AD FS を使用してデプロイすると、既存の Active Directory フォレスト内の ID を Azure Stack Hub 内のリソースに対して認証できます。 この既存の Active Directory フォレストでは、AD FS フェデレーション信頼の作成を許可するために、AD FS のデプロイが必要です。

認証は ID の一部です。 Azure Stack Hub 内でロールベースのアクセス制御 (RBAC) を管理するには、Graph コンポーネントを構成する必要があります。 リソースへのアクセスが委任されると、Graph のコンポーネントは LDAP プロトコルを使用して、既存の Active Directory フォレストでユーザー アカウントを検索します。

![Azure Stack Hub AD FS のアーキテクチャ](media/azure-stack-integrate-identity/azure-stack-adfs-architecture.svg)

既存の AD FS はアカウント セキュリティ トークン サービス (STS) であり、Azure Stack Hub AD FS (リソース STS) に要求を送信します。 Azure Stack Hub では、自動化によって、既存の AD FS のメタデータ エンドポイントとのクレーム プロバイダー信頼が作成されます。

既存の AD FS で、証明書利用者信頼を構成する必要があります。 この手順は、自動化によって実行されるわけではなく、オペレーターによって構成される必要があります。 AD FS の Azure Stack Hub VIP エンドポイントは、`https://adfs.<Region>.<ExternalFQDN>/` というパターンを使用して作成できます。

また、証明書利用者信頼構成には、マイクロソフトが提供する要求変換ルールを構成する必要があります。

Graph の構成には、既存の Active Directory での読み取りアクセス許可が付与されたサービス アカウントが提供されている必要があります。 このアカウントは RBAC シナリオを有効にする自動化の入力として必要です。

最後の手順では、既定のプロバイダー サブスクリプションに対して新しい所有者を構成する必要があります。 このアカウントで Azure Stack Hub 管理者ポータルにサインインすると、すべてのリソースに対してフル アクセスが付与されます。

要件:

|コンポーネント|要件|
|---------|---------|
|グラフ|Microsoft Active Directory 2012/2012 R2/2016 2019|
|AD FS|Windows Server 2012/2012 R2/2016 2019|

## <a name="setting-up-graph-integration"></a>Graph の統合を設定する

Graph は、単一の Active Directory フォレストとの統合のみをサポートしています。 複数のフォレストが存在する場合、ユーザーとグループのフェッチには、構成で指定されたフォレストのみが使用されます。

自動化パラメーターの入力として、次の情報が必要です。

|パラメーター|デプロイ ワークシート パラメーター|説明|例|
|---------|---------|---------|---------|
|`CustomADGlobalCatalog`|AD FS Forest FQDN|統合先のターゲット Active Directory フォレストの FQDN|Contoso.com|
|`CustomADAdminCredentials`| |LDAP の読み取りアクセス許可を持つユーザー|YOURDOMAIN\graphservice|

### <a name="configure-active-directory-sites"></a>Active Directory サイトを構成する

複数のサイトを持つ Active Directory のデプロイでは、Azure Stack Hub のデプロイに最も近い Active Directory サイトを構成します。 この構成により、Azure Stack Hub Graph サービスで、リモート サイトからグローバル カタログ サーバーを使用してクエリが解決されなくなります。

Azure Stack Hub の[パブリック VIP ネットワーク](azure-stack-network.md#public-vip-network) サブネットを、Azure Stack Hub に最も近い Active Directory サイトに追加します。 たとえば、Active Directory に 2 つのサイトがあるとします。シアトルとレドモントです。 シアトルのサイトに Azure Stack Hub がデプロイされている場合、Azure Stack Hub のパブリック VIP ネットワーク サブネットを、Active Directory のシアトルのサイトに追加します。

Active Directory サイトの詳細については、「[サイト トポロジの設計](/windows-server/identity/ad-ds/plan/designing-the-site-topology)」を参照してください。

> [!Note]  
> Active Directory が単一サイトで構成されている場合、この手順はスキップできます。 包括的なサブネットが構成されている場合、Azure Stack Hub のパブリック VIP ネットワーク サブネットがその一部ではないことを確認します。

### <a name="create-user-account-in-the-existing-active-directory-optional"></a>既存の Active Directory でユーザー アカウントを作成する (省略可能)

必要に応じて、既存の Active Directory で Graph サービスのアカウントを作成できます。 まだ使用するアカウントがない場合は、この手順を実行します。

1. 既存の Active Directory で、次のユーザー アカウントを作成します (推奨)。
   - **ユーザー名**: graphservice
   - **パスワード**:強力なパスワードを使用し、パスワードを無期限に設定します。

   特殊なアクセス許可やメンバーシップは不要

#### <a name="trigger-automation-to-configure-graph"></a>トリガーを自動化して Graph を構成する

この手順では、データセンター ネットワーク内の、Azure Stack Hub の特権エンドポイントと通信できるコンピューターを使用します。

1. 管理者特権での Windows PowerShell セッション (管理者として実行) を開き、特権エンドポイントの IP アドレスに接続します。 **CloudAdmin** の資格情報を使用して認証します。

   ```powershell  
   $creds = Get-Credential
   $pep = New-PSSession -ComputerName <IP Address of ERCS> -ConfigurationName PrivilegedEndpoint -Credential $creds
   ```

2. 特権エンドポイントを使用するセッションができたので、次のコマンドを実行します。 

   **Azure Stack Hub の 2008 以降のビルドについては、次のスクリプトを実行します** <br>

   ```powershell  
    $i = @(
           [pscustomobject]@{ 
                     CustomADGlobalCatalog="fabrikam.com"
                     CustomADAdminCredential= get-credential
                     SkipRootDomainValidation = $false 
                     ValidateParameters = $true
                   }) 

    Invoke-Command -Session $pep -ScriptBlock {Register-DirectoryService -customCatalog $using:i} 


   ```

   **Azure Stack Hub の 2008 より前のビルドについては、次のスクリプトを実行します** <br>

   ```powershell  
   Invoke-Command -Session $pep -ScriptBlock {Register-DirectoryService -CustomADGlobalCatalog contoso.com} 
   
   
   ```

   メッセージが表示されたら、Graph サービスに使用するユーザー アカウントの資格情報を指定します (graphservice など)。 Register-DirectoryService コマンドレットには、フォレスト内の他のドメインではなく、フォレスト内のフォレスト名/ルート ドメインを入力する必要があります。

   > [!IMPORTANT]
   > 資格情報のポップアップ表示を待機して (Get-Credential は特権エンドポイントではサポートされていません)、Graph サービス アカウントの資格情報を入力します。

3. **Register-DirectoryService** コマンドレットには、既存の Active Directory の検証が失敗する特定のシナリオで使用できる省略可能なパラメーターがあります。 このコマンドレットを実行すると、指定されたドメインがルート ドメインであること、グローバル カタログ サーバーに到達可能であること、指定されたアカウントでは読み取りアクセスが許可されていることが検証されます。

   |パラメーター|説明|
   |---------|---------|
   |`SkipRootDomainValidation`|推奨されるルート ドメインではなく、子ドメインを使用する必要があることを指定します。|
   |`ValidateParameters`|すべての検証チェックをバイパスします。|

#### <a name="graph-protocols-and-ports"></a>Graph のプロトコルとポート

Azure Stack Hub の Graph サービスでは、次のプロトコルとポートを使用して、書き込み可能なグローバル カタログ サーバー (GC) およびキー配布センター (KDC) と通信し、ターゲットの Active Directory フォレストのログイン要求を処理できます。

Azure Stack Hub の Graph サービスでは、次のプロトコルとポートを使用して、ターゲットの Active Directory と通信します。

|Type|Port|Protocol|
|---------|---------|---------|
|LDAP|389|TCP と UDP|
|LDAP SSL|636|TCP|
|LDAP GC|3268|TCP|
|LDAP GC SSL|3269|TCP|

## <a name="setting-up-ad-fs-integration-by-downloading-federation-metadata"></a>フェデレーション メタデータをダウンロードして AD FS の統合を設定する

自動化パラメーターの入力として、次の情報が必要です。

|パラメーター|デプロイ ワークシート パラメーター|説明|例|
|---------|---------|---------|---------|
|CustomAdfsName|AD FS Provider Name|クレーム プロバイダーの名前。<br>AD FS のランディング ページにそのように表示されます。|Contoso|
|CustomAD<br>FSFederationMetadataEndpointUri|AD FS Metadata URI|フェデレーション メタデータのリンク。| https:\//ad01.contoso.com/federationmetadata/2007-06/federationmetadata.xml |
|SigningCertificateRevocationCheck|NA|CRL チェックをスキップするオプション パラメーター。|なし|


### <a name="trigger-automation-to-configure-claims-provider-trust-in-azure-stack-hub"></a>Azure Stack Hub で自動化をトリガーしてクレーム プロバイダー信頼を構成する

この手順では、Azure Stack Hub の特権エンドポイントと通信できるコンピューターを使用します。 アカウント **STS AD FS** によって使用される証明書が、Azure Stack Hub によって信頼されている必要があります。

1. 管理者特権の Windows PowerShell セッションを開き、特権エンドポイントに接続します。

   ```powershell  
   $creds = Get-Credential
   Enter-PSSession -ComputerName <IP Address of ERCS> -ConfigurationName PrivilegedEndpoint -Credential $creds
   ```

2. これで特権エンドポイントに接続されました。お使いの環境に合わせてパラメーターを変更し、次のコマンドを実行します。

   ```powershell  
   Register-CustomAdfs -CustomAdfsName Contoso -CustomADFSFederationMetadataEndpointUri https://win-SQOOJN70SGL.contoso.com/federationmetadata/2007-06/federationmetadata.xml
   ```

3. お使いの環境に適したパラメーターを使用して、次のコマンドを実行して、既定のプロバイダー サブスクリプションの所有者を更新します。

   ```powershell  
   Set-ServiceAdminOwner -ServiceAdminOwnerUpn "administrator@contoso.com"
   ```

## <a name="setting-up-ad-fs-integration-by-providing-federation-metadata-file"></a>フェデレーション メタデータ ファイルを指定して AD FS の統合を設定する

バージョン 1807 から、このメソッドは、次の条件のいずれかに該当する場合に使用します。

- AD FS の証明書チェーンが、Azure Stack Hub の他のすべてのエンドポイントと比較して異なる。
- Azure Stack Hub の AD FS インスタンスと既存の AD FS サーバーがネットワークで接続されていない。

自動化パラメーターの入力として、次の情報が必要です。

|パラメーター|説明|例|
|---------|---------|---------|
|CustomAdfsName|クレーム プロバイダーの名前。 AD FS のランディング ページにそのように表示されます。|Contoso|
|CustomADFSFederationMetadataFileContent|メタデータの内容。|$using:federationMetadataFileContent|

### <a name="create-federation-metadata-file"></a>フェデレーション メタデータ ファイルを作成する

次の手順では、アカウント STS になる既存の AD FS デプロイとネットワークで接続されているコンピューターを使用する必要があります。 また、必要な証明書がインストールされている必要があります。

1. 管理者特権の Windows PowerShell セッションを開き、お使いの環境に適したパラメーターを使用して、次のコマンドを実行します。

   ```powershell  
    $url = "https://win-SQOOJN70SGL.contoso.com/FederationMetadata/2007-06/FederationMetadata.xml"
    $webclient = New-Object System.Net.WebClient
    $webclient.Encoding = [System.Text.Encoding]::UTF8
    $metadataAsString = $webclient.DownloadString($url)
    Set-Content -Path c:\metadata.xml -Encoding UTF8 -Value $metadataAsString
   ```

2. 特権エンドポイントと通信できるコンピューターに、メタデータ ファイルをコピーします。

### <a name="trigger-automation-to-configure-claims-provider-trust-in-azure-stack-hub"></a>Azure Stack Hub で自動化をトリガーしてクレーム プロバイダー信頼を構成する

この手順では、Azure Stack Hub で特権エンドポイントと通信可能で、前の手順で作成したメタデータ ファイルにアクセスできるコンピューターを使用します。

1. 管理者特権の Windows PowerShell セッションを開き、特権エンドポイントに接続します。

   ```powershell  
   $federationMetadataFileContent = get-content c:\metadata.xml
   $creds=Get-Credential
   Enter-PSSession -ComputerName <IP Address of ERCS> -ConfigurationName PrivilegedEndpoint -Credential $creds
   ```

2. これで特権エンドポイントに接続されました。お使いの環境に合わせてパラメーターを変更し、次のコマンドを実行します。

    ```powershell
    Register-CustomAdfs -CustomAdfsName Contoso -CustomADFSFederationMetadataFileContent $using:federationMetadataFileContent
    ```

3. 次のコマンドを実行して、既定のプロバイダー サブスクリプションの所有者を更新します。 お使いの環境に適したパラメータ―を使用します。

   ```powershell  
   Set-ServiceAdminOwner -ServiceAdminOwnerUpn "administrator@contoso.com"
   ```

   > [!Note]  
   > 既存の AD FS (アカウント STS) 上で証明書をローテーションする場合、AD FS 統合を再設定する必要があります。 メタデータ エンドポイントに到達できるか、メタデータ ファイルを指定することによって構成された場合でも、統合を設定する必要があります。

## <a name="configure-relying-party-on-existing-ad-fs-deployment-account-sts"></a>既存の AD FS デプロイ (アカウント STS) の証明書利用者を構成する

マイクロソフトは、要求変換ルールなど、証明書利用者信頼を構成するスクリプトを用意しています。 コマンドは手動で実行できるため、スクリプトの使用は任意です。

へルパー スクリプトは、GitHub の [Azure Stack Hub ツール](https://github.com/Azure/AzureStack-Tools/tree/vnext/DatacenterIntegration/Identity)に関するページからダウンロードできます。

コマンドを手動で実行する場合は、次の手順に従います。

1. データセンターの AD FS インスタンスまたはファーム メンバーの .txt ファイル (例: c:\ClaimIssuanceRules.txt) に次のコンテンツをコピーします。

   ```text
   @RuleTemplate = "LdapClaims"
   @RuleName = "Name claim"
   c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname", Issuer == "AD AUTHORITY"]
   => issue(store = "Active Directory", types = ("http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name"), query = ";userPrincipalName;{0}", param = c.Value);

   @RuleTemplate = "LdapClaims"
   @RuleName = "UPN claim"
   c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname", Issuer == "AD AUTHORITY"]
   => issue(store = "Active Directory", types = ("http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn"), query = ";userPrincipalName;{0}", param = c.Value);

   @RuleTemplate = "LdapClaims"
   @RuleName = "ObjectID claim"
   c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/primarysid"]
   => issue(Type = "http://schemas.microsoft.com/identity/claims/objectidentifier", Issuer = c.Issuer, OriginalIssuer = c.OriginalIssuer, Value = c.Value, ValueType = c.ValueType);

   @RuleName = "Family Name and Given claim"
   c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname", Issuer == "AD AUTHORITY"]
   => issue(store = "Active Directory", types = ("http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname", "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname"), query = ";sn,givenName;{0}", param = c.Value);

   @RuleTemplate = "PassThroughClaims"
   @RuleName = "Pass through all Group SID claims"
   c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid", Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"]
   => issue(claim = c);

   @RuleTemplate = "PassThroughClaims"
   @RuleName = "Pass through all windows account name claims"
   c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname"]
   => issue(claim = c);
   ```

2. エクストラネットおよびイントラネット用に Windows フォーム ベース認証が有効になっていることを確認します。 次のコマンドレットを実行して、既に有効になっているかどうかを確認できます。

   ```powershell  
   Get-AdfsAuthenticationProvider | where-object { $_.name -eq "FormsAuthentication" } | select Name, AllowedForPrimaryExtranet, AllowedForPrimaryIntranet
   ```

    > [!Note]  
    > Windows 統合認証 (WIA) サポートのユーザー エージェント文字列が、AD FS 展開に対して期限切れになっている場合があり、最新のクライアントをサポートするための更新プログラムが必要になる場合があります。 WIA によってサポートされるユーザー エージェント文字列の更新に関する詳細については、「[WIA をサポートしていないデバイスのイントラネットフォームベース認証の構成](/windows-server/identity/ad-fs/operations/configure-intranet-forms-based-authentication-for-devices-that-do-not-support-wia)」の記事をお読みください。<br><br>フォーム ベース認証ポリシーを有効にする手順については、「[認証ポリシーを構成する](/windows-server/identity/ad-fs/operations/configure-authentication-policies)」を参照してください。

3. 証明書利用者信頼を追加するには、AD FS インスタンスまたはファーム メンバーで、次の Windows PowerShell コマンドを実行します。 必ず AD FS エンドポイントを更新して、手順 1 で作成されたファイルを指定します。

   **AD FS 2016/2019 の場合**

   ```powershell  
   Add-ADFSRelyingPartyTrust -Name AzureStack -MetadataUrl "https://YourAzureStackADFSEndpoint/FederationMetadata/2007-06/FederationMetadata.xml" -IssuanceTransformRulesFile "C:\ClaimIssuanceRules.txt" -AutoUpdateEnabled:$true -MonitoringEnabled:$true -enabled:$true -AccessControlPolicyName "Permit everyone" -TokenLifeTime 1440
   ```

   **AD FS 2012/2012 R2 の場合**

   ```powershell  
   Add-ADFSRelyingPartyTrust -Name AzureStack -MetadataUrl "https://YourAzureStackADFSEndpoint/FederationMetadata/2007-06/FederationMetadata.xml" -IssuanceTransformRulesFile "C:\ClaimIssuanceRules.txt" -AutoUpdateEnabled:$true -MonitoringEnabled:$true -enabled:$true -TokenLifeTime 1440
   ```

   > [!IMPORTANT]  
   > Windows Server 2012 または 2012 R2 AD FS を使用している場合は、AD FS MMC スナップインを使用して発行承認規則を構成する必要があります。

4. Internet Explorer または Microsoft Edge ブラウザーを使用して Azure Stack Hub にアクセスするには、トークンのバインドを無視する必要があります。 無視しないと、サインインの試行が失敗します。 AD FS インスタンスまたはファーム メンバーで、次のコマンドを実行します。

   > [!note]  
   > Windows Server 2012 または 2012 R2 AD FS を使用する場合、この手順は該当しません。 その場合は、このコマンドをスキップして統合を続けてかまいません。

   ```powershell  
   Set-AdfsProperties -IgnoreTokenBinding $true
   ```

   **AD FS 2002 以上の場合**

   > [!NOTE]
   > お客様が所有する ADFS ホストまたはファーム上で `Add-ADFSRelyingPartyTrust` を実行する場合は、まず ADFS ホストまたはファームで TLS1.2 が適用されていることを確認してください。適用せずに実行すると、次のエラー メッセージが表示されます。

`Add-ADFSRelyingPartyTrust : The underlying connection was closed: An unexpected error occurred on a send.`

## <a name="spn-creation"></a>SPN の作成

多くのシナリオで、認証にサービス プリンシパル名 (SPN) の使用が要求されます。 次はその例の一部です。

- Azure Stack Hub の AD FS デプロイでの CLI の使用。
- AD FS を使用してデプロイされる場合の Azure Stack Hub 用の System Center 管理パック。
- AD FS を使用してデプロイされる場合の Azure Stack Hub でのリソース プロバイダー。
- 各種のアプリ。
- 非対話型サインインを要求する。

> [!Important]  
> AD FS は、対話型サインイン セッションのみをサポートしています。 自動化シナリオに非対話型サインインを必要とする場合は、SPN を使用する必要があります。

SPN の作成に関する詳細については、[AD FS でのサービス プリンシパルの作成](azure-stack-create-service-principals.md)に関するページを参照してください。


## <a name="troubleshooting"></a>トラブルシューティング

### <a name="configuration-rollback"></a>構成のロールバック

エラーが発生し、環境が認証できない状態になった場合は、ロールバック オプションを使用できます。

1. Windows PowerShell セッションを管理者特権で開き、次のコマンドを実行します。

   ```powershell  
   $creds = Get-Credential
   Enter-PSSession -ComputerName <IP Address of ERCS> -ConfigurationName PrivilegedEndpoint -Credential $creds
   ```

2. その後次のコマンドレットを実行します。

   ```powershell  
   Reset-DatacenterIntegrationConfiguration
   ```

   ロールバック アクションを実行すると、すべての構成の変更がロールバックされます。 組み込みの **CloudAdmin** ユーザーによる認証のみが有効です。

   > [!IMPORTANT]
   > 既定のプロバイダー サブスクリプションの元の所有者を構成する必要があります。

   ```powershell  
   Set-ServiceAdminOwner -ServiceAdminOwnerUpn "azurestackadmin@[Internal Domain]"
   ```

### <a name="collecting-additional-logs"></a>追加のログを収集する

コマンドレットの実行が失敗した場合、`Get-Azurestacklogs` コマンドレットを使用して追加のログを収集できます。

1. Windows PowerShell セッションを管理者特権で開き、次のコマンドを実行します。

   ```powershell  
   $creds = Get-Credential
   Enter-pssession -ComputerName <IP Address of ERCS> -ConfigurationName PrivilegedEndpoint -Credential $creds
   ```

2. その後次のコマンドレットを実行します。

   ```powershell  
   Get-AzureStackLog -OutputPath \\myworkstation\AzureStackLogs -FilterByRole ECE
   ```

## <a name="next-steps"></a>次のステップ

[外部の監視ソリューションの統合](azure-stack-integrate-monitor.md)
