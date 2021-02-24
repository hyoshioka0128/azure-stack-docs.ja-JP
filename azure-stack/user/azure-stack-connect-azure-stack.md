---
title: Azure Stack Hub に接続する
description: Azure Stack Hub に接続する方法について説明します。
author: mattbriggs
ms.topic: conceptual
ms.date: 2/1/2021
ms.author: mabrigg
ms.reviewer: thoroet
ms.lastreviewed: 01/14/2020
ms.openlocfilehash: b8a9c565cf618ea97714574a8f4d662afadaa749
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99247831"
---
# <a name="connect-to-azure-stack-hub"></a>Azure Stack Hub に接続する

リソースを管理するには、Azure Stack Development Kit に接続する必要があります。 この記事では、Development Kit に接続するために必要な手順について詳しく説明します。 次の接続方法のうちどちらかを使用できます。

* リモート デスクトップ: Development Kit から、1 人の同時実行ユーザーのみがすばやく接続できます。
* 仮想プライベート ネットワーク (VPN): Azure Stack Hub インフラストラクチャ外のクライアントから複数のユーザーが同時接続できます (構成が必要です)。

## <a name="connect-to-azure-stack-hub-with-remote-desktop"></a>リモート デスクトップを使用して Azure Stack Hub に接続する
リモート デスクトップ接続では、1 人の同時実行ユーザーがポータルを使用してリソースを管理できます。

1. リモート デスクトップ接続を開き、Development Kit に接続します。 「**AzureStack\AzureStackAdmin**」というユーザー名と、Azure Stack Hub セットアップ時に指定した管理者パスワードを入力します。  

2. Development Kit コンピューターからサーバー マネージャーを開き、 **[ローカル サーバー]** をクリックし、Internet Explorer セキュリティ強化を無効にして、サーバー マネージャーを閉じます。

3. ポータルを開くには、`https://portal.local.azurestack.external/` に移動し、ユーザーの資格情報を使用してサインインします。


## <a name="connect-to-azure-stack-hub-with-vpn"></a>VPN を使用して Azure Stack Hub に接続する

Azure Stack Development Kit に対して、分割トンネルの VPN 接続を確立することができます。 VPN 接続を介して、管理者ポータル、ユーザー ポータル、ローカルにインストールされているツール (Visual Studio、PowerShell など) にアクセスして、Azure Stack Hub リソースを管理できます。 VPN 接続は、Azure Active Directory (Azure AD) ベースおよび Active Directory フェデレーション サービス (AD FS) ベースのデプロイの両方でサポートされています。 VPN 接続を使用すると、同時に複数のクライアントが Azure Stack Hub に接続できます。 

> [!NOTE] 
> この VPN 接続では、Azure Stack Hub インフラストラクチャ VM には接続できません。 

### <a name="prerequisites"></a>前提条件

* [Azure Stack Hub 互換の Azure PowerShell](../operator/powershell-install-az-module.md) をローカル コンピューターにインストールします。  
* [Azure Stack Hub の操作に必要なツール](../operator/azure-stack-powershell-download.md)をダウンロードします。 

### <a name="configure-vpn-connectivity"></a>VPN 接続の構成

Development Kit に対する VPN 接続を作成するには、ローカル Windows ベースのコンピューターから管理者特権の PowerShell セッションを開き、次のスクリプトを実行します (実際の環境に合わせて IP アドレスとパスワードは変更してください)。

```powershell 
# Configure winrm if it's not already configured
winrm quickconfig  

Set-ExecutionPolicy RemoteSigned

# Import the Connect module
Import-Module .\Connect\AzureStack.Connect.psm1 

# Add the development kit computer's host IP address & certificate authority (CA) to the list of trusted hosts. Make sure to update the IP address and password values for your environment. 

$hostIP = "<Azure Stack Hub host IP address>"

$Password = ConvertTo-SecureString `
  "<Administrator password provided when deploying Azure Stack Hub>" `
  -AsPlainText `
  -Force

Set-Item wsman:\localhost\Client\TrustedHosts `
  -Value $hostIP `
  -Concatenate

# Create a VPN connection entry for the local user
Add-AzsVpnConnection `
  -ServerAddress $hostIP `
  -Password $Password

```

セットアップが成功すると、VPN 接続の一覧に `azurestack` と表示されます。

![ネットワーク接続](media/azure-stack-connect-azure-stack/image3.png)  

### <a name="connect-to-azure-stack-hub"></a>Azure Stack Hub に接続する

次の 2 つの方法のいずれかを使用して、Azure Stack Hub インスタンスに接続します。  

* `Connect-AzsVpn` コマンドを使用する: 
    
  ```powershell
  Connect-AzsVpn `
    -Password $Password
  ```

  プロンプトに従って Azure Stack Hub ホストを信頼し、**AzureStackCertificateAuthority** の証明書をローカル コンピューターの証明書ストアにインストールします。 プロンプトは、PowerShell セッション ウィンドウの背後に表示される可能性があります。 

* ローカル コンピューターの **[ネットワーク設定]**  >  **[VPN]** に移動し、[`azurestack`] >  **[接続]** を選択します。 サインイン プロンプトで、ユーザー名 (AzureStack\AzureStackAdmin) とパスワードを入力します。

### <a name="test-the-vpn-connectivity"></a>VPN 接続のテスト

ポータル接続をテストするには、ブラウザーを開き、ユーザー ポータル `https://portal.local.azurestack.external/` に移動し、サインインしてリソースを作成します。  

## <a name="next-steps"></a>次のステップ



