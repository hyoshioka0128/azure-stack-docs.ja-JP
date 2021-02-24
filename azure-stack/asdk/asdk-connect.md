---
title: ASDK に接続する
description: Azure Stack Development Kit (ASDK) に接続する方法について説明します。
author: PatAltimore
ms.topic: article
ms.date: 11/14/2020
ms.author: patricka
ms.reviewer: knithinc
ms.lastreviewed: 11/14/2020
ms.openlocfilehash: 0bf880cdbb181ae923a16932ba5522cfadc74be0
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97873522"
---
# <a name="connect-to-the-asdk"></a>ASDK に接続する

リソースを管理するにはまず、Azure Stack Development Kit (ASDK) に接続する必要があります。 この記事では、次の接続オプションを使用して ASDK に接続するために必要な手順を説明します。

* [リモート デスクトップ接続 (RDP)](#connect-with-rdp):リモート デスクトップ接続を使用して接続する場合、単一のユーザーが ASDK にすばやく接続できます。
* [仮想プライベート ネットワーク (VPN)](#connect-with-vpn):VPN を使用して接続する場合、Azure Stack インフラストラクチャの外部のクライアントから複数のユーザーが同時に Azure Stack ポータルに接続できます。 VPN 接続には、いくつかのセットアップが必要です。

<a name="connect-with-rdp"></a>
## <a name="connect-to-azure-stack-using-rdp"></a>RDP を使用して Azure Stack に接続する

同時に 1 人のユーザーが、ASDK ホスト コンピューターから直接リモート デスクトップ接続を使用して、Azure Stack 管理者ポータルまたはユーザー ポータルでリソースを管理できます。

> [!TIP]
> このオプションでは、ASDK ホスト コンピューターにサインインしているときに、RDP を再度使用して、ASDK ホスト コンピューター上に作成された仮想マシン (VM) にサインインできます。

1. リモート デスクトップ接続 (mstc.exe) を開き、ASDK ホスト コンピューターの IP アドレスに接続します。 ASDK ホスト コンピューターにリモートでサインインする権限のあるアカウントを使用してください。 既定では、**AzureStack\AzureStackAdmin** に ASDK ホスト コンピューターにリモートでアクセス許可があります。  

2. ASDK ホスト コンピューターでサーバー マネージャー (ServerManager.exe) を開きます。 **[ローカル サーバー]** を選択し、 **[IE セキュリティ強化の構成]** をオフにして、サーバー マネージャーを閉じます。

3. 管理者ポータルに **AzureStack\CloudAdmin** としてサインインするか、他の Azure Stack オペレーターの資格情報を使用します。 ASDK 管理者ポータルのアドレスは `https://adminportal.local.azurestack.external` です。

4. ユーザー ポータルに **AzureStack\CloudAdmin** としてサインインするか、他の Azure Stack ユーザーの資格情報を使用します。 ASDK ユーザー ポータルのアドレスは `https://portal.local.azurestack.external` です。

> [!NOTE]
> どのアカウントをいつ使用するかについては、「[ASDK 管理の基礎](asdk-admin-basics.md#what-account-should-i-use)」をご覧ください。

<a name="connect-with-vpn"></a>
## <a name="connect-to-azure-stack-using-vpn"></a>VPN を使用して Azure Stack に接続する

ASDK ホスト コンピューターに対する分割トンネルの VPN 接続を確立して、Azure Stack ポータルと、Visual Studio や PowerShell などのローカルにインストールされたツールにアクセスできます。 VPN 接続を使うと、複数のユーザーは、ASDK でホストされている Azure Stack リソースに同時に接続できます。

VPN 接続は、Azure AD と Active Directory フェデレーション サービス (AD FS) の両方のデプロイでサポートされています。

> [!NOTE]
> VPN 接続では、Azure Stack VM には接続 *できません*。 VPN 経由で接続している間は、Azure Stack VM に RDP 接続することはできません。

### <a name="prerequisites"></a>前提条件
ASDK への VPN 接続を設定する前に、次の前提条件を満たしていることを確認してください。

- [Azure Stack 互換の Azure PowerShell](asdk-post-deploy.md#install-azure-stack-powershell) をローカル コンピューターにインストールします。  
- [Azure Stack を操作するために必要なツール](asdk-post-deploy.md#download-the-azure-stack-tools)をダウンロードします。

### <a name="set-up-vpn-connectivity"></a>VPN 接続の設定

ASDK への VPN 接続を作成するには、Windows ベースのローカル コンピューター上で管理者として PowerShell を開きます。 その後、次のスクリプトを実行します。IP アドレスとパスワードの値は実際の環境に合わせて更新してください。

### <a name="az-modules"></a>[Az モジュール](#tab/az)

```powershell
# Change directories to the default Azure Stack tools directory
cd C:\AzureStack-Tools-az

# Configure Windows Remote Management (WinRM), if it's not already configured.
winrm quickconfig  

Set-ExecutionPolicy RemoteSigned

# Import the Connect module.
Import-Module .\Connect\AzureStack.Connect.psm1

# Add the ASDK host computer's IP address as the ASDK certificate authority (CA) to the list of trusted hosts. Make sure you update the IP address and password values for your environment.

$hostIP = "<Azure Stack Hub host IP address>"

$Password = ConvertTo-SecureString `
  "<operator's password provided when deploying Azure Stack>" `
  -AsPlainText `
  -Force

Set-Item wsman:\localhost\Client\TrustedHosts `
  -Value $hostIP `
  -Concatenate

# Create a VPN connection entry for the local user.
Add-AzsVpnConnection `
  -ServerAddress $hostIP `
  -Password $Password

```

### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm)

```powershell
# Change directories to the default Azure Stack tools directory
cd C:\AzureStack-Tools-master

# Configure Windows Remote Management (WinRM), if it's not already configured.
winrm quickconfig  

Set-ExecutionPolicy RemoteSigned

# Import the Connect module.
Import-Module .\Connect\AzureStack.Connect.psm1

# Add the ASDK host computer's IP address as the ASDK certificate authority (CA) to the list of trusted hosts. Make sure you update the IP address and password values for your environment.

$hostIP = "<Azure Stack Hub host IP address>"

$Password = ConvertTo-SecureString `
  "<operator's password provided when deploying Azure Stack>" `
  -AsPlainText `
  -Force

Set-Item wsman:\localhost\Client\TrustedHosts `
  -Value $hostIP `
  -Concatenate

# Create a VPN connection entry for the local user.
Add-AzsVpnConnection `
  -ServerAddress $hostIP `
  -Password $Password

```
---
設定が成功すると、VPN 接続の一覧に **Azure Stack** と表示されます。

![ネットワーク接続](media/asdk-connect/vpn.png)  

### <a name="connect-to-azure-stack"></a>Azure Stack への接続

  次のどちらかの方法で、Azure Stack インスタンスに接続します。  

  * `Connect-AzsVpn` コマンドを使用する。
      
    ```powershell
    Connect-AzsVpn `
      -Password $Password
    ```

  * ローカル コンピューターの **[ネットワーク設定]**  >  **[VPN]**  >  **[Azure Stack]**  >  **[接続]** の順に選択します。 サインイン プロンプトで、ユーザー名 (**AzureStack\AzureStackAdmin**) とパスワードを入力します。

初回の接続では、Azure Stack ルート証明書を **AzureStackCertificateAuthority** からローカル コンピューターの証明書ストアにインストールすることを求めるメッセージが表示されます。 この手順により、信頼されたホストの一覧に、ASDK 証明機関 (CA) が追加されます。 **[はい]** をクリックして証明書をインストールします。

![ルート証明書](media/asdk-connect/cert.png)  
  
  > [!IMPORTANT]
  > プロンプトが PowerShell ウィンドウや他のアプリで隠れている場合があります。

### <a name="test-vpn-connectivity"></a>VPN 接続をテストする

ポータルの接続をテストするには、ブラウザーを開き、ユーザー ポータル (`https://portal.local.azurestack.external/`) または管理者ポータル (`https://adminportal.local.azurestack.external/`) のいずれかに移動します。

適切なサブスクリプションの資格情報でサインインし、リソースを作成して管理します。  

## <a name="next-steps"></a>次のステップ

[トラブルシューティング](asdk-troubleshooting.md)
