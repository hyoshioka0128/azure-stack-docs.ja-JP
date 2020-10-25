---
title: シークレットの切り替え
titleSuffix: Azure Stack Hub
description: Azure Stack Hub でシークレットをローテーションする方法について説明します。
author: BryanLa
ms.topic: how-to
ms.date: 06/29/2020
ms.reviewer: ppacent
ms.author: bryanla
ms.lastreviewed: 08/15/2020
monikerRange: '>=azs-1803'
ms.openlocfilehash: aca163df1026193933ffb9d09dbdf4a854638a75
ms.sourcegitcommit: 362081a8c19e7674c3029c8a44d7ddbe2deb247b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/09/2020
ms.locfileid: "91899807"
---
# <a name="rotate-secrets-in-azure-stack-hub"></a>Azure Stack Hub でシークレットをローテーションする

"*この記事の説明は、Azure Stack Hub 統合システムのバージョン 1803 以降に対してのみ適用されます。1803 より前のバージョンではシークレットのローテーションを試行しないでください。* "

この記事では、Azure Stack Hub インフラストラクチャのリソースおよびサービスとのセキュリティで保護された通信の維持に役立つ、シークレットのローテーションに関するガイダンスと PowerShell スクリプトを提供します。 

## <a name="overview"></a>概要

Azure Stack Hub はシークレットを使用して、インフラストラクチャのリソースおよびサービスとのセキュリティで保護された通信を維持します。 Azure Stack Hub インフラストラクチャの整合性を維持するために、オペレーターは、組織のセキュリティ要件と一致する頻度でシークレットをローテーションできる必要があります。

### <a name="internal-vs-external-secrets"></a>内部および外部シークレット

リリース 1811 以降、シークレットのローテーションは、内部と外部の証明書で分けられています。

- **内部シークレット**: Azure Stack Hub オペレーターの介入なしに Azure Stack Hub インフラストラクチャによって使用される証明書、パスワード、セキュリティで保護された文字列、およびキー。

- **外部シークレット**: Azure Stack Hub オペレーターによって提供される、外部向けサービス用のインフラストラクチャ サービス証明書。 外部シークレットには、次のサービスの証明書が含まれます。

    - 管理者ポータル
    - パブリック ポータル
    - 管理者の Azure Resource Manager
    - グローバルな Azure Resource Manager
    - 管理者の Key Vault
    - Key Vault
    - 管理者拡張機能ホスト
    - ACS (Blob、Table、Queue Storage を含みます)
    - ADFS*
    - Graph*
    
    \* 環境の ID プロバイダーが Active Directory フェデレーション サービス (AD FS) の場合にのみ適用されます。

> [!Important]
> 他のすべてのセキュリティで保護されたキーと文字列は、管理者によって手動で更新されます。 これには、ユーザーと管理者アカウントのパスワード、[ネットワーク スイッチのパスワードとアクセス許可](azure-stack-customer-defined.md)、および[この記事の後半で説明する](#update-the-bmc-credential)ベースボード管理コントローラー (BMC) の資格情報が含まれます。 
>
>また、この記事では、付加価値リソース プロバイダーのシークレットのローテーションについては説明しません。 これらのシークレットをローテーションするには、代わりに次の記事を参照してください。
>
> - [App Service on Azure Stack Hub のシークレットと証明書を交換する](app-service-rotate-certificates.md)
> - [MySQL リソース プロバイダー - シークレットのローテーション](azure-stack-mysql-resource-provider-maintain.md#secrets-rotation)
> - [SQL リソース プロバイダー - シークレットのローテーション](azure-stack-sql-resource-provider-maintain.md#secrets-rotation)

### <a name="expiration-alerts"></a>有効期限のアラート

シークレットの有効期限まで 30 日以内になると、管理者ポータルで次のアラートが生成されます。

- 保留中のサービス アカウントのパスワードの有効期限
- 保留中の内部証明書の有効期限
- 保留中の外部証明書の有効期限

次のセクションでシークレットのローテーション手順を完了すると、これらのアラートが解決されます。

> [!Note]
> 1811 より前の Azure Stack Hub 環境では、保留中の内部証明書やシークレットの有効期限のアラートが表示される場合があります。 これらのアラートは正確ではないため、内部シークレットのローテーションを実行せずに無視する必要があります。 正しくない内部シークレットの有効期限切れアラートは、1811 で解決される既知の問題です。 環境が 2 年間アクティブになっていない限り、内部シークレットの有効期限が切れることはありません。

### <a name="external-certificates-from-a-new-certificate-authority"></a>新しい証明書機関からの外部証明書

Azure Stack Hub では、次のようなコンテキストで、新しい証明書機関 (CA) からの外部証明書を使用したシークレットのローテーションがサポートされます。

|インストール済みの証明書 CA|ローテーション先の CA|サポートされています|サポートされる Azure Stack Hub のバージョン|
|-----|-----|-----|-----|
|自己署名済みから|Enterprise へ|サポートされています|1903 以降|
|自己署名済みから|自己署名済みへ|サポートされていません||
|自己署名済みから|公開へ<sup>*</sup>|サポートされています|1803 以降|
|Enterprise から|Enterprise へ|サポートされています。 1803 から 1903: 顧客がデプロイ時に使用したのと同じエンタープライズ CA を使用している限りサポートされます|1803 以降|
|Enterprise から|自己署名済みへ|サポートされていません||
|Enterprise から|公開へ<sup>*</sup>|サポートされています|1803 以降|
|公開から<sup>*</sup>|Enterprise へ|サポートされています|1903 以降|
|公開から<sup>*</sup>|自己署名済みへ|サポートされていません||
|公開から<sup>*</sup>|公開へ<sup>*</sup>|サポートされています|1803 以降|

<sup>*</sup>公開証明機関が、Windows の信頼されたルート プログラムに含まれていることを示しています。 完全な一覧は、「[参加者の一覧 - Microsoft の信頼されたルート プログラム](/security/trusted-root/participants-list)」という記事にあります。

## <a name="prerequisites"></a>前提条件

内部および外部シークレットのローテーションの場合:

1. 最初に Azure Stack Hub インスタンスを最新バージョンに更新することを強くお勧めします。

    ::: moniker range="<azs-1811"  
    >[!IMPORTANT]
    > 1811 より前のバージョンの場合:
    > - シークレットのローテーションが既に実行されている場合、バージョン 1811 以降に更新してから、シークレットのローテーションを再び実行する必要があります。 シークレットのローテーションは、[特権エンドポイント](azure-stack-privileged-endpoint.md)を使用して実行する必要があり、Azure Stack Hub オペレーターの資格情報が必要です。 ご使用の環境でシークレットのローテーションが実行されているかどうかがわからない場合は、シークレットのローテーションを実行する前に 1811 に更新してください。
    > - 拡張機能ホストの証明書を追加するためにシークレットのローテーションを行う必要はありません。 拡張機能ホストの証明書を追加するには、「[Azure Stack Hub の拡張機能ホストを準備する](azure-stack-extension-host-prepare.md)」の手順に従ってください。
    ::: moniker-end

2. 計画メンテナンス操作をユーザーに通知します。 可能な限り非営業時間中に、通常のメンテナンス期間をスケジュールします。 メンテナンス操作は、ユーザーのワークロードとポータル操作の両方に影響を及ぼす可能性があります。

3. シークレットのローテーション中に、アラートが開いて自動的に閉じることにオペレーターが気付くことがあります。 これは予期される動作であり、アラートは無視してもかまいません。 オペレーターは、[PowerShell コマンドレットの Test-AzureStack](azure-stack-diagnostic-test.md) を使用して、これらのアラートの有効性を確認できます。 System Center Operations Manager を使用して Azure Stack Hub システムを監視しているオペレーターの場合、システムをメンテナンス モードにすると、これらのアラートは ITSM システムに到達しなくなりますが、Azure Stack Hub システムに到達できなくなっても引き続きアラートは生成されます。

外部シークレットのローテーションの場合には、次の追加の前提条件を満たす必要があります。

1. シークレットをローテーションする前に、`-group SecretRotationReadiness` パラメーターを使用して **[`Test-AzureStack`](azure-stack-diagnostic-test.md)** PowerShell コマンドレットを実行し、すべてのテスト出力が正常であることを確認します。
2. 代わりの外部証明書の新しいセットを準備します。
    - 新しいセットは、[Azure Stack Hub PKI 証明書の要件](azure-stack-pki-certs.md)に関する記事に記載されている証明書の仕様に一致している必要があります。 
    - 証明書署名要求 (CSR) を生成して証明機関 (CA) に送信するには、[証明書署名要求の生成](azure-stack-get-pki-certs.md)に関する記事に記載された手順を使用します。また、それらを Azure Stack Hub 環境で使用するために準備するには、[PKI 証明書の準備](azure-stack-prepare-pki-certs.md)に関する記事の手順を使用します。 
    - 準備する証明書は、[PKI 証明書の検証](azure-stack-validate-pki-certs.md)に関する記事に記載された手順を使用して必ず検証してください。
    - パスワードに `*` や `)` などの特殊文字が含まれないことを確認します。
    - PFX 暗号化が **TripleDES-SHA1** であることを確認します。 問題が発生した場合は、「[Azure Stack Hub PKI 証明書に関する一般的な問題を修復する](azure-stack-remediate-certs.md#pfx-encryption)」を参照してください。
3. ローテーションに使われる証明書のバックアップを安全なバックアップ場所に格納します。 ローテーションを実行して失敗した場合は、ローテーションを再実行する前に、ファイル共有内の証明書をバックアップ コピーに置き換えます。 バックアップ コピーはセキュリティで保護されたバックアップ場所に保存してください。
4. ERCS VM からアクセスできるファイル共有を作成します。 ファイル共有は、**CloudAdmin** ID で読み書きできる必要があります。
5. ファイル共有にアクセスできるコンピューターから PowerShell ISE コンソールを開きます。 ファイル共有に移動します。ここで、外部証明書を配置するディレクトリを作成します。
6. ローテーション中にアクセスできるネットワーク ファイル共有に **[CertDirectoryMaker.ps1](https://www.aka.ms/azssecretrotationhelper)** をダウンロードし、スクリプトを実行します。 このスクリプトにより、ID プロバイダーに応じて ***.\Certificates\AAD*** または ***.\Certificates\ADFS*** に準拠するフォルダー構造が作成されます。 フォルダー構造は、 **\\Certificates** フォルダーで始まり、その後に、 **\\AAD** または **\\ADFS** フォルダーのみが続く必要があります。 その他のすべてのサブディレクトリは、上記の構造内に含まれます。 次に例を示します。
    - ファイル共有 = **\\\\\<IPAddress>\\\<ShareName>**
    - Azure AD プロバイダーの証明書ルート フォルダー = **\\Certificates\AAD**
    - 完全なパス = **\\\\\<IPAddress>\\\<ShareName>\Certificates\AAD**

    > [!IMPORTANT]
    > 後で `Start-SecretRotation` を実行すると、フォルダー構造が検証されます。 準拠していないフォルダー構造の場合、次のエラーがスローされます。
    >
    > ```powershell
    > Cannot bind argument to parameter 'Path' because it is null.
    > + CategoryInfo          : InvalidData: (:) [Test-Certificate], ParameterBindingValidationException
    > + FullyQualifiedErrorId : ParameterArgumentValidationErrorNullNotAllowed,Test-Certificate
    > + PSComputerName        : xxx.xxx.xxx.xxx
    > ```

7. 手順 2 で作成した代替の外部証明書の新しいセットを、手順 6 で作成した **\Certificates\\\<IdentityProvider>** ディレクトリにコピーします。 \<CertName\> の `cert.<regionName>.<externalFQDN>` 形式に必ず従ってください。 

    Azure AD ID プロバイダーのフォルダー構造の例を次に示します。
    ```powershell
        <ShareName>
            │
            └───Certificates
                  └───AAD
                      ├───ACSBlob
                      │       <CertName>.pfx
                      │
                      ├───ACSQueue
                      │       <CertName>.pfx
                      │
                      ├───ACSTable
                      │       <CertName>.pfx
                      │
                      ├───Admin Extension Host
                      │       <CertName>.pfx
                      │
                      ├───Admin Portal
                      │       <CertName>.pfx
                      │
                      ├───ARM Admin
                      │       <CertName>.pfx
                      │
                      ├───ARM Public
                      │       <CertName>.pfx
                      │
                      ├───KeyVault
                      │       <CertName>.pfx
                      │
                      ├───KeyVaultInternal
                      │       <CertName>.pfx
                      │
                      ├───Public Extension Host
                      │       <CertName>.pfx
                      │
                      └───Public Portal
                              <CertName>.pfx

    ```

## <a name="rotate-external-secrets"></a>外部シークレットをローテーションする

外部シークレットをローテーションするには、次の手順を実行します。

1. 次の PowerShell スクリプトを使用して、シークレットをローテーションします。 このスクリプトには、特権エンドポイント (PEP) セッションへのアクセスが必要です。 PEP には、PEP をホストする仮想マシン (VM) 上のリモート PowerShell セッションを介してアクセスします。 統合システムを使用している場合、PEP の 3 つのインスタンスがあり、それぞれ異なるホスト上の VM (Prefix-ERCS01、Prefix-ERCS02、または Prefix-ERCS03) 内で動作しています。 ASDK を使用している場合、この VM は AzS-ERCS01 という名前になります。 次を実行する前に、`<placeholder>` の値を更新します。

    ```powershell
    # Create a PEP Session
    winrm s winrm/config/client '@{TrustedHosts= "<IP_address_of_ERCS>"}'
    $PEPCreds = Get-Credential
    $PEPSession = New-PSSession -ComputerName <IP_address_of_ERCS_Machine> -Credential $PEPCreds -ConfigurationName "PrivilegedEndpoint"

    # Run Secret Rotation
    $CertPassword = ConvertTo-SecureString "<Cert_Password>" -AsPlainText -Force
    $CertShareCreds = Get-Credential
    $CertSharePath = "<Network_Path_Of_CertShare>"
    Invoke-Command -Session $PEPSession -ScriptBlock {
        Start-SecretRotation -PfxFilesPath $using:CertSharePath -PathAccessCredential $using:CertShareCreds -CertificatePassword $using:CertPassword
    }
    Remove-PSSession -Session $PEPSession
    ```

    このスクリプトは、次のステップを実行します。

    - **CloudAdmin** アカウントを使用して[特権エンドポイント](azure-stack-privileged-endpoint.md)で PowerShell セッションを作成し、セッションを変数として格納します。 この変数は、次の手順でパラメーターとして使用されます。 

    - [Invoke-Command](/powershell/module/microsoft.powershell.core/Invoke-Command) を実行し、PEP セッション変数を `-Session` パラメーターとして渡します。

    - 次のパラメーターを使用して、PEP セッションで `Start-SecretRotation` を実行します。
        - `-PfxFilesPath`:前に作成した Certificates ディレクトリへのネットワーク パス。  
        - `-PathAccessCredential`:共有に対する資格情報の PSCredential オブジェクト。
        - `-CertificatePassword`:作成されるすべての pfx 証明書ファイルに使われるパスワードのセキュリティで保護された文字列です。

2. 外部シークレットのローテーションには、約 1 時間かかります。 正常に完了すると、コンソールに `ActionPlanInstanceID ... CurrentStatus: Completed` が表示され、その後に `DONE` が表示されます。 「前提条件」セクションで作成した共有から証明書を削除し、セキュリティで保護されたバックアップ場所に保存します。

    > [!Note]
    > シークレットのローテーションが失敗した場合は、エラー メッセージの指示に従い、 `-ReRun` パラメーターを指定して `Start-SecretRotation` を再実行します。
    >
    >```powershell
    >Start-SecretRotation -ReRun
    >```  
    >
    >シークレットのローテーションの失敗が繰り返される場合は、サポートにお問い合わせください。

## <a name="rotate-internal-secrets"></a>内部シークレットをローテーションする

内部シークレットのローテーションは、それが侵害された可能性がある場合、または有効期限アラートを受け取った場合にのみ必要です。 1811 より前のバージョンでは、保留中の内部証明書やシークレットの有効期限に関するアラートが表示される場合があります。 これらのアラートは正確ではないため、無視してください。これらは既知の問題であり、1811 で解決されています。 環境が 2 年間アクティブになっていない限り、内部シークレットの有効期限が切れることはありません。

「[外部シークレットをローテーションする](#rotate-external-secrets)」の手順 2 にある PowerShell スクリプトを参照してください。 このスクリプトは、次の手順を実行するためにいくつかの変更を加えることによって、内部シークレットのローテーションに適応できる例を提供します。

1. "Run Secret Rotation" セクションで、`-Internal` パラメーターを [Start-SecretRotation コマンドレット](../reference/pep-2002/start-secretrotation.md)に追加します。次に例を示します。

    ```powershell
    # Run Secret Rotation
    ...
    Invoke-Command -Session $PEPSession -ScriptBlock {
        Start-SecretRotation -Internal
    }
    ...
    ```

    ::: moniker range="<azs-1811"
    > [!Note]
    > 1811 より前のバージョンでは、`-Internal` フラグは必要ありません。 
    ::: moniker-end

3. 正常に完了すると、コンソールに `ActionPlanInstanceID ... CurrentStatus: Completed` が表示され、その後に `DONE` が表示されます。

    > [!Note]
    > シークレット ローテーションが失敗した場合は、エラー メッセージ内の指示に従い、 `-Internal` パラメーターと `-ReRun` パラメーターを付けて `Start-SecretRotation` を再実行します。  
    >
    >```powershell
    >Start-SecretRotation -Internal -ReRun
    >```
    >
    > シークレットのローテーションの失敗が繰り返される場合は、サポートにお問い合わせください。

## <a name="update-the-bmc-credential"></a>BMC 資格情報を更新する

ベースボード管理コントローラーは、サーバーの物理的な状態を監視します。 BMC のユーザー アカウント名とパスワードを更新する手順については、OEM (Original Equipment Manufacturer) ハードウェア ベンダーを参照してください。

>[!NOTE]
> OEM から追加の管理アプリが提供されている場合があります。 他の管理アプリのユーザー名またはパスワードを更新しても、BMC のユーザー名またはパスワードには影響しません。 

::: moniker range="<azs-1910"
1. OEM の指示に従って、Azure Stack Hub の物理サーバー上で BMC を更新します。 環境内の各 BMC のユーザー名とパスワードは同じである必要があります。 BMC のユーザー名は、16 文字を超えることはできません。
::: moniker-end

::: moniker range=">=azs-1910"
1. OEM の指示に従って Azure Stack Hub 物理サーバーの BMC 資格情報を最初に更新する必要がなくなりました。 環境内の各 BMC のユーザー名とパスワードは同じである必要があり、16 文字以内で指定する必要があります。 
::: moniker-end

2. Azure Stack Hub セッションで特権エンドポイントを開きます。 手順については、「[Azure Stack Hub で特権エンドポイントを使用する](azure-stack-privileged-endpoint.md)」を参照してください。 

3. 特権エンドポイント セッションを開いたら、次の PowerShell スクリプトのいずれかを実行します。これは、Invoke-Command を使用して、Set-BmcCredential を実行します。 省略可能な -BypassBMCUpdate パラメーターを Set-BMCCredential と共に使用すると、BMC の資格情報は更新されません。 Azure Stack Hub の内部データストアだけが更新されます。特権エンドポイントのセッション変数をパラメーターとして渡します。

    ユーザー名とパスワードの入力を求める PowerShell スクリプトの例を次に示します。 

    ```powershell
    # Interactive Version
    $PEPIp = "<Privileged Endpoint IP or Name>" # You can also use the machine name instead of IP here.
    $PEPCreds = Get-Credential "<Domain>\CloudAdmin" -Message "PEP Credentials"
    $NewBmcPwd = Read-Host -Prompt "Enter New BMC password" -AsSecureString
    $NewBmcUser = Read-Host -Prompt "Enter New BMC user name"

    $PEPSession = New-PSSession -ComputerName $PEPIp -Credential $PEPCreds -ConfigurationName "PrivilegedEndpoint"

    Invoke-Command -Session $PEPSession -ScriptBlock {
        # Parameter BmcPassword is mandatory, while the BmcUser parameter is optional.
        Set-BmcCredential -BmcPassword $using:NewBmcPwd -BmcUser $using:NewBmcUser
    }
    Remove-PSSession -Session $PEPSession
    ```

    ユーザー名とパスワードを変数にエンコードすることもできますが、これは安全性が低くなる可能性があります。

    ```powershell
    # Static Version
    $PEPIp = "<Privileged Endpoint IP or Name>" # You can also use the machine name instead of IP here.
    $PEPUser = "<Privileged Endpoint user for example Domain\CloudAdmin>"
    $PEPPwd = ConvertTo-SecureString "<Privileged Endpoint Password>" -AsPlainText -Force
    $PEPCreds = New-Object System.Management.Automation.PSCredential ($PEPUser, $PEPPwd)
    $NewBmcPwd = ConvertTo-SecureString "<New BMC Password>" -AsPlainText -Force
    $NewBmcUser = "<New BMC User name>"

    $PEPSession = New-PSSession -ComputerName $PEPIp -Credential $PEPCreds -ConfigurationName "PrivilegedEndpoint"

    Invoke-Command -Session $PEPSession -ScriptBlock {
        # Parameter BmcPassword is mandatory, while the BmcUser parameter is optional.
        Set-BmcCredential -BmcPassword $using:NewBmcPwd -BmcUser $using:NewBmcUser
    }
    Remove-PSSession -Session $PEPSession
    ```

## <a name="reference-start-secretrotation-cmdlet"></a>リファレンス: Start-SecretRotation コマンドレット

[Start-SecretRotation コマンドレット](../reference/pep-2002/start-secretrotation.md)は、Azure Stack Hub システムのインフラストラクチャ シークレットのローテーションを行います。 このコマンドレットは、`-Session` パラメーターで PEP セッションを渡す `Invoke-Command` スクリプト ブロックを使用することによって、Azure Stack Hub の特権エンドポイントに対してのみ実行できます。 既定では、すべての外部ネットワーク インフラストラクチャ エンドポイントの証明書のみのローテーションを行います。

| パラメーター | 種類 | 必須 | [位置] | Default | 説明 |
|--|--|--|--|--|--|
| `PfxFilesPath` | String  | False  | named  | なし  | すべての外部ネットワーク エンドポイント証明書を含む **\Certificates** ディレクトリへのファイル共有パスです。 外部シークレットのローテーションを行う場合にのみ必要です。 最後のディレクトリは **\Certificates** にする必要があります。 |
| `CertificatePassword` | SecureString | False  | named  | なし  | -PfXFilesPath で提供されているすべての証明書のパスワード。 外部のシークレットのローテーションを行うときに PfxFilesPath を指定する場合は、必須の値です。 |
| `Internal` | String | False | named | なし | Azure Stack Hub オペレーターが内部インフラストラクチャ シークレットのローテーションを行うときは、常に Internal フラグを使用する必要があります。 |
| `PathAccessCredential` | PSCredential | False  | named  | なし  | すべての外部ネットワーク エンドポイント証明書を含む **\Certificates** ディレクトリへのファイル共有の PowerShell 資格情報。 外部シークレットのローテーションを行う場合にのみ必要です。  |
| `ReRun` | SwitchParameter | False  | named  | なし  | 試行が失敗した後でシークレットのローテーションが再試行されるときに、常に使用する必要があります。 |

### <a name="syntax"></a>構文

#### <a name="for-external-secret-rotation"></a>外部シークレットのローテーションの場合

```powershell
Start-SecretRotation [-PfxFilesPath <string>] [-PathAccessCredential <PSCredential>] [-CertificatePassword <SecureString>]  
```

#### <a name="for-internal-secret-rotation"></a>内部シークレットのローテーションの場合

```powershell
Start-SecretRotation [-Internal]  
```

#### <a name="for-external-secret-rotation-rerun"></a>外部シークレットのローテーションの再実行の場合

```powershell
Start-SecretRotation [-ReRun]
```

#### <a name="for-internal-secret-rotation-rerun"></a>内部シークレットのローテーションの再実行の場合

```powershell
Start-SecretRotation [-ReRun] [-Internal]
```

### <a name="examples"></a>例

#### <a name="rotate-only-internal-infrastructure-secrets"></a>内部インフラストラクチャ シークレットのみのローテーション

このコマンドは、Azure Stack Hub [環境の特権エンドポイント](azure-stack-privileged-endpoint.md)を使用して実行する必要があります。

```powershell
PS C:\> Start-SecretRotation -Internal
```

このコマンドを使用すると、Azure Stack Hub 内部ネットワークに公開されているすべてのインフラストラクチャ シークレットのローテーションが行われます。

#### <a name="rotate-only-external-infrastructure-secrets"></a>外部インフラストラクチャ シークレットのみのローテーション  

```powershell
# Create a PEP Session
winrm s winrm/config/client '@{TrustedHosts= "<IP_address_of_ERCS>"}'
$PEPCreds = Get-Credential
$PEPSession = New-PSSession -ComputerName <IP_address_of_ERCS> -Credential $PEPCreds -ConfigurationName "PrivilegedEndpoint"

# Create Credentials for the fileshare
$CertPassword = ConvertTo-SecureString "<CertPasswordHere>" -AsPlainText -Force
$CertShareCreds = Get-Credential
$CertSharePath = "<NetworkPathOfCertShare>"
# Run Secret Rotation
Invoke-Command -Session $PEPSession -ScriptBlock {  
    Start-SecretRotation -PfxFilesPath $using:CertSharePath -PathAccessCredential $using:CertShareCreds -CertificatePassword $using:CertPassword
}
Remove-PSSession -Session $PEPSession
```

このコマンドは、Azure Stack Hub の外部ネットワーク インフラストラクチャ エンドポイントで使用する TLS 証明書のローテーションを行います。

::: moniker range="<azs-1811"
#### <a name="rotate-internal-and-external-infrastructure-secrets-pre-1811-only"></a>内部と外部のインフラストラクチャ シークレットのローテーション (**1811 より前**のみ)

> [!IMPORTANT]
> ローテーションは内部証明書と外部証明書に分けられたので、このコマンドは **1811 より前**の Azure Stack Hub にのみ適用されます。
>
> **"*1811 以降*" 、内部証明書と外部証明書の両方をローテーションすることはできなくなりました。**

```powershell
# Create a PEP Session
winrm s winrm/config/client '@{TrustedHosts= "<IP_address_of_ERCS>"}'
$PEPCreds = Get-Credential
$PEPSession = New-PSSession -ComputerName <IP_address_of_ERCS> -Credential $PEPCreds -ConfigurationName "PrivilegedEndpoint"

# Create Credentials for the fileshare
$CertPassword = ConvertTo-SecureString "<CertPasswordHere>" -AsPlainText -Force
$CertShareCreds = Get-Credential
$CertSharePath = "<NetworkPathOfCertShare>"
# Run Secret Rotation
Invoke-Command -Session $PEPSession -ScriptBlock {
    Start-SecretRotation -PfxFilesPath $using:CertSharePath -PathAccessCredential $using:CertShareCreds -CertificatePassword $using:CertPassword
}
Remove-PSSession -Session $PEPSession
```

このコマンドは、Azure Stack Hub 内部ネットワークに公開されているインフラストラクチャ シークレットと、Azure Stack Hub の外部ネットワーク インフラストラクチャ エンドポイントに使用される TLS 証明書をローテーションします。 Start-SecretRotation は、スタックで生成されたすべてのシークレットのローテーションを行い、証明書が指定されているため、外部エンドポイント証明書のローテーションも行われます。  
::: moniker-end

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub のセキュリティについて詳しく学習する](azure-stack-security-foundations.md)