---
title: Azure Stack Hub での拡張機能ホストを準備する
description: Azure Stack Hub 内で、拡張機能ホストの準備を行う方法について学習します。これは、バージョン 1808 以降では、Azure Stack Hub 更新プログラム パッケージを通じて自動的に有効化されます。
author: PatAltimore
ms.author: patricka
ms.date: 1/22/2020
ms.topic: article
ms.reviewer: thoroet
ms.lastreviewed: 03/07/2019
ms.openlocfilehash: 43cb091b3076f26880599e35a770383182e944d0
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97870615"
---
# <a name="prepare-for-extension-host-in-azure-stack-hub"></a>Azure Stack Hub での拡張機能ホストを準備する

拡張機能ホストでは、必要とされる TCP/IP ポートの数を減らすことで Azure Stack Hub をセキュリティ保護します。 この記事では、Azure Stack Hub での拡張機能ホストの準備について確認します。これは、1808 更新プログラムより後の Azure Stack Hub 更新プログラム パッケージを通じて自動的に有効になります。 この記事は、Azure Stack Hub の更新プログラム、1808、1809、1811 に適用されます。

## <a name="certificate-requirements"></a>証明書の要件

拡張機能ホストでは、ポータル拡張機能ごとに一意のホスト エントリが保証されるように、新しい 2 つのドメイン名前空間が実装されます。 安全な通信を確保するために、新しいドメイン名前空間には追加の 2 つのワイルドカード証明書が必要です。

新しい名前空間と、関連する証明書を表に示します。

| デプロイ フォルダー | 必要な証明書のサブジェクト名とサブジェクトの別名 (SAN) | スコープ (リージョンごと) | サブドメインの名前空間 |
|-----------------------|------------------------------------------------------------------|-----------------------|------------------------------|
| 管理者拡張機能ホスト | *.adminhosting.\<region>.\<fqdn> (ワイルドカード SSL 証明書) | 管理者拡張機能ホスト | adminhosting.\<region>.\<fqdn> |
| パブリック拡張機能ホスト | *.hosting.\<region>.\<fqdn> (ワイルドカード SSL 証明書) | パブリック拡張機能ホスト | hosting.\<region>.\<fqdn> |

証明書の詳細な要件については、「[Azure Stack Hub 公開キー インフラストラクチャ証明書の要件](azure-stack-pki-certs.md)」を参照してください。

## <a name="create-certificate-signing-request"></a>証明書署名要求の作成

Azure Stack Hub 対応性チェッカー ツールを使用すると、必要とされる新しい 2 つの SSL 証明書への証明書署名要求を作成できます。 [Azure Stack Hub 証明書署名要求の生成](azure-stack-get-pki-certs.md)に関する記事の手順に従ってください。

> [!Note]  
> この手順は、SSL 証明書の要求内容によっては省略できます。

## <a name="validate-new-certificates"></a>新しい証明書の検証

1. ハードウェア ライフサイクル ホストまたは Azure Stack Hub 管理ワークステーションで、昇格された権限で PowerShell を開きます。
2. 次のコマンドレットを実行して、Azure Stack Hub 対応性チェッカー ツールをインストールします。

    ```powershell  
    Install-Module -Name Microsoft.AzureStack.ReadinessChecker
    ```

3. 次のスクリプトを実行して、必須のフォルダー構造を作成します。

    ```powershell  
    New-Item C:\Certificates -ItemType Directory

    $directories = 'ACSBlob','ACSQueue','ACSTable','Admin Portal','ARM Admin','ARM Public','KeyVault','KeyVaultInternal','Public Portal', 'Admin extension host', 'Public extension host'

    $destination = 'c:\certificates'

    $directories | % { New-Item -Path (Join-Path $destination $PSITEM) -ItemType Directory -Force}
    ```

    > [!Note]  
    > Active Directory フェデレーション サービス (AD FS) を使用してデプロイする場合、スクリプトの **$directories** に `ADFS` および `Graph` というディレクトリを追加する必要があります。

4. Azure Stack Hub で現在使用している既存の証明書を適切なディレクトリに配置します。 たとえば、**管理者の ARM** 証明書を `Arm Admin` フォルダーに配置します。 その後、新しく作成されたホスティング証明書を `Admin extension host` と `Public extension host` ディレクトリに配置します。
5. 次のコマンドレットを実行して、証明書の確認を開始します。

    ```powershell  
    $pfxPassword = Read-Host -Prompt "Enter PFX Password" -AsSecureString 

    Start-AzsReadinessChecker -CertificatePath c:\certificates -pfxPassword $pfxPassword -RegionName east -FQDN azurestack.contoso.com -IdentitySystem AAD
    ```

6. 出力を確認し、すべての証明書がすべてのテストに合格しているかを確認します。


## <a name="import-extension-host-certificates"></a>拡張機能ホストの証明書のインポート

次の手順では、Azure Stack Hub 特権エンドポイントに接続できるコンピューターを使用します。 そのコンピューターで新しい証明書ファイルにアクセスできるようにしておいてください。

1. 次の手順では、Azure Stack Hub 特権エンドポイントに接続できるコンピューターを使用します。 そのコンピューターで新しい証明書ファイルにアクセスできるようにしておいてください。
2. PowerShell ISE を開いて、次のスクリプト ブロックを実行します
3. 管理者ホスティング エンドポイント用の証明書をインポートします。

    ```powershell  

    $CertPassword = read-host -AsSecureString -prompt "Certificate Password"

    $CloudAdminCred = Get-Credential -UserName <Privileged endpoint credentials> -Message "Enter the cloud domain credentials to access the privileged endpoint."

    [Byte[]]$AdminHostingCertContent = [Byte[]](Get-Content c:\certificate\myadminhostingcertificate.pfx -Encoding Byte)

    Invoke-Command -ComputerName <PrivilegedEndpoint computer name> `
    -Credential $CloudAdminCred `
    -ConfigurationName "PrivilegedEndpoint" `
    -ArgumentList @($AdminHostingCertContent, $CertPassword) `
    -ScriptBlock {
            param($AdminHostingCertContent, $CertPassword)
            Import-AdminHostingServiceCert $AdminHostingCertContent $certPassword
    }
    ```
4. ホスティング エンドポイントの証明書をインポートします。
    ```powershell  
    $CertPassword = read-host -AsSecureString -prompt "Certificate Password"

    $CloudAdminCred = Get-Credential -UserName <Privileged endpoint credentials> -Message "Enter the cloud domain credentials to access the privileged endpoint."

    [Byte[]]$HostingCertContent = [Byte[]](Get-Content c:\certificate\myhostingcertificate.pfx  -Encoding Byte)

    Invoke-Command -ComputerName <PrivilegedEndpoint computer name> `
    -Credential $CloudAdminCred `
    -ConfigurationName "PrivilegedEndpoint" `
    -ArgumentList @($HostingCertContent, $CertPassword) `
    -ScriptBlock {
            param($HostingCertContent, $CertPassword)
            Import-UserHostingServiceCert $HostingCertContent $certPassword
    }
    ```

### <a name="update-dns-configuration"></a>DNS 構成の更新

> [!Note]  
> DNS 統合に DNS ゾーンの委任を使用した場合、この手順は必要ありません。
Azure Stack Hub エンドポイントを公開するよう個別のホスト A レコードが構成されている場合、追加のホスト A レコードを 2 つ作成する必要があります。

| IP | hostname | Type |
|----|------------------------------|------|
| \<IP> | *.Adminhosting.\<Region>.\<FQDN> | A |
| \<IP> | *.Hosting.\<Region>.\<FQDN> | A |

割り当てられた IP は、**Get-AzureStackStampInformation** コマンドレットを実行することで、特権エンドポイントを使用して取得できます。

### <a name="ports-and-protocols"></a>ポートとプロトコル

[Azure Stack Hub とデータセンターの統合 - エンドポイントの公開](azure-stack-integrate-endpoints.md)に関する記事では、拡張機能ホストのロールアウト前に Azure Stack Hub を公開するために受信通信を必要とするポートとプロトコルについて説明しています。

### <a name="publish-new-endpoints"></a>新しいエンドポイントの公開

ファイアウォール経由で 2 つの新しいエンドポイントを公開する必要があります。 パブリック VIP プールから割り当てられた IP は、Azure Stack Hub [環境の特権エンドポイント](azure-stack-privileged-endpoint.md)から実行する必要のある次のコードを使用して取得できます。

```powershell
# Create a PEP Session
winrm s winrm/config/client '@{TrustedHosts= "<IpOfERCSMachine>"}'
$PEPCreds = Get-Credential
$PEPSession = New-PSSession -ComputerName <IpOfERCSMachine> -Credential $PEPCreds -ConfigurationName "PrivilegedEndpoint"

# Obtain DNS Servers and extension host information from Azure Stack Hub Stamp Information and find the IPs for the Host Extension Endpoints
$StampInformation = Invoke-Command $PEPSession {Get-AzureStackStampInformation} | Select-Object -Property ExternalDNSIPAddress01, ExternalDNSIPAddress02, @{n="TenantHosting";e={($_.TenantExternalEndpoints.TenantHosting) -replace "https://*.","testdnsentry"-replace "/"}},  @{n="AdminHosting";e={($_.AdminExternalEndpoints.AdminHosting)-replace "https://*.","testdnsentry"-replace "/"}},@{n="TenantHostingDNS";e={($_.TenantExternalEndpoints.TenantHosting) -replace "https://",""-replace "/"}},  @{n="AdminHostingDNS";e={($_.AdminExternalEndpoints.AdminHosting)-replace "https://",""-replace "/"}}
If (Resolve-DnsName -Server $StampInformation.ExternalDNSIPAddress01 -Name $StampInformation.TenantHosting -ErrorAction SilentlyContinue) {
    Write-Host "Can access AZS DNS" -ForegroundColor Green
    $AdminIP = (Resolve-DnsName -Server $StampInformation.ExternalDNSIPAddress02 -Name $StampInformation.AdminHosting).IPAddress
    Write-Host "The IP for the Admin Extension Host is: $($StampInformation.AdminHostingDNS) - is: $($AdminIP)" -ForegroundColor Yellow
    Write-Host "The Record to be added in the DNS zone: Type A, Name: $($StampInformation.AdminHostingDNS), Value: $($AdminIP)" -ForegroundColor Green
    $TenantIP = (Resolve-DnsName -Server $StampInformation.ExternalDNSIPAddress01 -Name $StampInformation.TenantHosting).IPAddress
    Write-Host "The IP address for the Tenant Extension Host is $($StampInformation.TenantHostingDNS) - is: $($TenantIP)" -ForegroundColor Yellow
    Write-Host "The Record to be added in the DNS zone: Type A, Name: $($StampInformation.TenantHostingDNS), Value: $($TenantIP)" -ForegroundColor Green
}
Else {
    Write-Host "Cannot access AZS DNS" -ForegroundColor Yellow
    $AdminIP = (Resolve-DnsName -Name $StampInformation.AdminHosting).IPAddress
    Write-Host "The IP for the Admin Extension Host is: $($StampInformation.AdminHostingDNS) - is: $($AdminIP)" -ForegroundColor Yellow
    Write-Host "The Record to be added in the DNS zone: Type A, Name: $($StampInformation.AdminHostingDNS), Value: $($AdminIP)" -ForegroundColor Green
    $TenantIP = (Resolve-DnsName -Name $StampInformation.TenantHosting).IPAddress
    Write-Host "The IP address for the Tenant Extension Host is $($StampInformation.TenantHostingDNS) - is: $($TenantIP)" -ForegroundColor Yellow
    Write-Host "The Record to be added in the DNS zone: Type A, Name: $($StampInformation.TenantHostingDNS), Value: $($TenantIP)" -ForegroundColor Green
}
Remove-PSSession -Session $PEPSession
```

#### <a name="sample-output"></a>サンプル出力

```powershell
Can access AZS DNS
The IP for the Admin Extension Host is: *.adminhosting.\<region>.\<fqdn> - is: xxx.xxx.xxx.xxx
The Record to be added in the DNS zone: Type A, Name: *.adminhosting.\<region>.\<fqdn>, Value: xxx.xxx.xxx.xxx
The IP address for the Tenant Extension Host is *.hosting.\<region>.\<fqdn> - is: xxx.xxx.xxx.xxx
The Record to be added in the DNS zone: Type A, Name: *.hosting.\<region>.\<fqdn>, Value: xxx.xxx.xxx.xxx
```

> [!Note]  
> 拡張機能ホストを有効にする前に、この変更を行います。 これで、Azure Stack Hub ポータルに継続的にアクセスできるようになります。

| エンドポイント (VIP) | Protocol | Port |
|----------------|----------|-------|
| 管理者ホスティング | HTTPS | 443 |
| Hosting | HTTPS | 443 |

### <a name="update-existing-publishing-rules-post-enablement-of-extension-host"></a>既存の公開規則の更新 (拡張機能ホストの今後の有効化)

> [!Note]  
> 1808 Azure Stack Hub 更新プログラム パッケージでは、まだ拡張機能ホストは有効化 **されません**。 必須の証明書をインポートして、拡張機能ホストの準備を行うことができます。 1808 更新プログラム以降の Azure Stack Hub 更新プログラム パッケージを通じて拡張機能ホストが自動的に有効になる前に、ポートを閉じないようにしてください。

以下の既存のエンドポイント ポートは、既存のファイアウォール規則で閉じる必要があります。

> [!Note]  
> それらのポートは、検証に成功した後に閉じることをお勧めします。

| エンドポイント (VIP) | Protocol | Port |
|----------------------------------------|----------|-------------------------------------------------------------------------------------------------------------------------------------|
| ポータル (管理者) | HTTPS | 12495<br>12499<br>12646<br>12647<br>12648<br>12649<br>12650<br>13001<br>13003<br>13010<br>13011<br>13012<br>13020<br>13021<br>13026<br>30015 |
| ポータル (ユーザー) | HTTPS | 12495<br>12649<br>13001<br>13010<br>13011<br>13012<br>13020<br>13021<br>30015<br>13003 |
| Azure Resource Manager (管理者) | HTTPS | 30024 |
| Azure Resource Manager (ユーザー) | HTTPS | 30024 |

## <a name="next-steps"></a>次のステップ

- [ファイアウォールの統合](azure-stack-firewall.md)について確認する。
- [Azure Stack Hub 証明書署名要求の生成](azure-stack-get-pki-certs.md)について学習する。
