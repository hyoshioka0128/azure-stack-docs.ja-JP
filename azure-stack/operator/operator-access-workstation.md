---
title: Azure Stack Hub オペレーター アクセス ワークステーション
description: Azure Stack Hub オペレーター アクセス ワークステーションをダウンロードして構成する方法について説明します。
author: ashika789
ms.topic: article
ms.date: 1/15/2021
ms.author: patricka
ms.reviewer: asganesh
ms.lastreviewed: 1/15/2021
ms.openlocfilehash: 011a75dcdbe8818af568eb5cc6bb6123a4f11c4e
ms.sourcegitcommit: 48a46142ea7bccd6c8a609e188dd7f3f6444f3c4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/18/2021
ms.locfileid: "98561963"
---
# <a name="azure-stack-hub-operator-access-workstation"></a>Azure Stack Hub オペレーター アクセス ワークステーション 

オペレーター アクセス ワークステーション (OAW) は、バージョン 2005 以降を実行するハードウェア ライフサイクル ホスト (HLH) にジャンプ ボックス仮想マシン (VM) をデプロイするために使用されます。これにより、Azure Stack Hub オペレーターは、サポート シナリオで特権エンドポイント (PEP) と管理者ポータルにアクセスできるようになります。 

OAW VM は、オペレーターが新しいタスクを実行するときに作成する必要があります。 Azure Stack Hub では、常に VM を実行する必要があるわけではないため、VM 内で必要なタスクが完了したら、VM をシャットダウンして削除してください。  

## <a name="oaw-scenarios"></a>OAW のシナリオ

次の表は、OAW の一般的なシナリオの一覧ですが、これは両立しないものではありません。 リモート デスクトップを使用して OAW に接続することが推奨されます。 

|シナリオ                                                                                                                          |説明                 |
|----------------------------------------------------------------------------------------------------------------------------------|-----------------------------|
|[管理ポータルにアクセスする](./azure-stack-manage-portals.md)                     |管理操作を実行します                                                                           |
|[PEP にアクセスする](./azure-stack-privileged-endpoint.md)                                     |ログの収集とアップロード:<br>Azure Stack Hub からのファイル転送用 HLH 上に -[SMB 共有を作成します](#transfer-files-between-the-hlh-and-oaw)<br>-Azure Storage Explorer を使用して、保存されたログを SMB 共有にアップロードします |
|[Azure Stack Hub の登録](./azure-stack-registration.md#renew-or-change-registration) |再登録するには、管理ポータルから、以前の登録名とリソース グループを取得します                               |
|[Marketplace シンジケーション](./azure-stack-download-azure-marketplace-item.md)            |HLH 上に [SMB 共有を作成](#transfer-files-between-the-hlh-and-oaw)し、ダウンロードしたイメージまたは拡張機能を格納します                                                        |

## <a name="download-files"></a>ファイルのダウンロード

OAW VM を作成するためのファイルを入手するには、[**ここからダウンロード**](https://aka.ms/OAWDownload)してください。 ダウンロードする前に、[Microsoft プライバシー ステートメント](https://privacy.microsoft.com/privacystatement)と[法律条項](/legal/azure-stack-hub/azure-stack-operator-access-workstation-legal-terms)を必ず確認してください。

このソリューションのステートレスな性質により、OAW VM 向けの更新プログラムはありません。 マイルストーンごとに、VM イメージ ファイルの新しいバージョンがリリースされます。 新しい OAW VM を作成するには、最新バージョンを使用します。 このイメージ ファイルは、最新の Windows Server 2019 バージョンに基づいています。 インストール後、Windows Update を使用して、緊急更新プログラムを含む、更新プログラムを適用できます。 

ダウンロードした OAW.zip ファイルのハッシュを検証して、OAW VM の作成に使用する前にそれが変更されていないことを確認します。 次の PowerShell スクリプトを実行します。 戻り値が True の場合、ダウンロードした OAW.zip を使用できます。

```powershell
param(
    [Parameter(Mandatory=$True)]
    [ValidateNotNullOrEmpty()]
    [ValidateScript({Test-Path $_ -PathType Leaf})]
    [string]
    $DownloadedOAWZipFilePath
)

$expectedHash = '73E16995B79433E79F9EFA9A292443296D112B24B4D86A060FCB4C9403B8D014'
$actualHash = (Get-FileHash -Path $DownloadedOAWZipFilePath).Hash

Write-Host "Expected hash: $expectedHash"

if ($expectedHash -eq $actualHash)
{
    Write-Host 'SUCCESS: OAW.zip file hash matches.'
}
else
{
    Write-Error 'ERROR: OAW.zip file hash does not match! It is not safe to use it, please download it again.'
    Write-Error "Actual hash: $actualHash"
}
```

## <a name="user-account-policy"></a>ユーザー アカウント ポリシー 
OAW VM には、次のユーザー アカウント ポリシーが適用されます。

- ビルトイン Administrator のユーザー名: AdminUser
- MinimumPasswordLength = 14
- PasswordComplexity が有効
- MinimumPasswordAge = 1 (日)
- MaximumPasswordAge = 42 (日)
- NewGuestName = GUser (既定で無効)

## <a name="pre-installed-software"></a>プリインストールされているソフトウェア
次の表に、OAW VM にプレインストールされているソフトウェアを示します。

| ソフトウェア名           | 場所                                                                                       |
|--------------------------|------------------------------------------------------------------------------------------------|
| [ビジネス向け Microsoft Edge](https://www.microsoft.com/edge/business/)                                            | \[SystemDrive\]\Program Files (x86)\Microsoft\Edge\Application                                                                                        |
| [Az モジュール](./powershell-install-az-module.md)                         | \[SystemDrive\]\ProgramFiles\WindowsPowerShell\Modules                                         |  
| [PowerShell 7](https://devblogs.microsoft.com/powershell/announcing-PowerShell-7-0/)| \[SystemDrive\]\Program Files\PowerShell\7                                                                       |
| [Azure コマンド ライン インターフェイス (CLI)](/cli/azure/?view=azure-cli-latest) | \[SystemDrive\]\Program Files (x86)\Microsoft SDKs\Azure\CLI2 |
| [Microsoft Azure ストレージ エクスプローラー](https://azure.microsoft.com/features/storage-explorer/)   | \[SystemDrive\]\Program Files (x86)\Microsoft Azure Storage Explorer                                                                       |
| [AzCopy](/azure/storage/common/storage-use-azcopy-v10)                             | \[SystemDrive\]\VMSoftware\azcopy_windows_amd64_10.3.4                                         |
| [AzureStack-Tools](https://github.com/Azure/AzureStack-Tools/tree/az)                  | \[SystemDrive\]\VMSoftware\AzureStack-Tools                                                    |

## <a name="check-hlh-version"></a>HLH のバージョンを確認する

1. 自分の資格情報を使用して HLH にログオンします。
1. PowerShell ISE を開き、次のスクリプトを実行します。

   ```powershell
   'C:\Version\Get-Version.ps1'
   ```

   次に例を示します。

   ![OAW VM のバージョンを確認するための PowerShell コマンドレットのスクリーンショット](./media/operator-access-workstation/check-hardware-lifecycle-host-version.png)

## <a name="create-the-oaw-vm-using-a-script"></a>スクリプトを使用して OAW VM を作成する

次のスクリプトでは、オペレーター アクセス ワークステーション (OAW) として仮想マシンを準備します。これは、管理と診断のために Microsoft Azure Stack Hub にアクセスするために使用されます。

1. 自分の資格情報を使用して HLH にログオンします。
1. OAW.zip をダウンロードし、ファイルを抽出します。
1. 管理者特権の PowerShell セッションを開きます。
1. OAW.zip ファイルの抽出されたコンテンツに移動します。
1. New-OAW.ps1 スクリプトを実行します。 

たとえば、Azure Stack Hub バージョン 2005 以降を使用して、カスタマイズせずに HLH に OAW VM を作成するには、 **-LocalAdministratorPassword** パラメーターのみを指定して New-OAW.ps1 スクリプトを実行します。

```powershell
$securePassword = Read-Host -Prompt "Enter password for Azure Stack OAW's local administrator" -AsSecureString
New-OAW.ps1 -LocalAdministratorPassword $securePassword  
```

Azure Stack Hub へのネットワーク接続があるホスト上に OAW VM を作成するには:

```powershell
$securePassword = Read-Host -Prompt "Enter password for Azure Stack OAW's local administrator" -AsSecureString
New-OAW.ps1 -LocalAdministratorPassword $securePassword `
   -IPAddress '192.168.0.20' `
   -SubnetMask '255.255.255.0' `
   -DefaultGateway '192.168.0.1' `
   -DNS '192.168.0.10'
```

AzureStackStampInformation.json ファイルから ERCS VM の IP アドレスを取得するには:

```powershell
$securePassword = Read-Host -Prompt "Enter password for Azure Stack OAW's local administrator" -AsSecureString
New-OAW.ps1 -LocalAdministratorPassword $securePassword `
   -AzureStackCertificatePath 'F:\certroot.cer' `
   -DeploymentDataFilePath 'F:\DeploymentData.json' `
   -AzSStampInfoFilePath 'F:\AzureStackStampInformation.json'
```

DeploymentData.json を使用してHLH 上に OAW VM を作成するには:

```powershell
$securePassword = Read-Host -Prompt "Enter password for Azure Stack OAW's local administrator" -AsSecureString
New-OAW.ps1 -LocalAdministratorPassword $securePassword `
   -DeploymentDataFilePath 'D:\AzureStack\DeploymentData.json'
```

DeploymentData.json ファイルに OAW VM の名前付けプレフィックスが含まれている場合、**VirtualMachineName** パラメーターには、その値が使用されます。 その他の場合、既定の名前は **AzSOAW** またはユーザーが指定した任意の名前です。

OAW に対しては、2 つのパラメーター セットを使用できます。 省略可能なパラメーターは、角かっこ内に表示されます。

```powershell
New-OAW 
-LocalAdministratorPassword <Security.SecureString> `
[-AzureStackCertificatePath <String>] `
[-AzSStampInfoFilePath <String>] `
[-CertificatePassword <Security.SecureString>] `
[-ERCSVMIP <String[]>] `
[-DNS <String[]>] `
[-DeploymentDataFilePath <String>] `
[-SkipNetworkConfiguration] `
[-ImageFilePath <String>] `
[-VirtualMachineName <String>] `
[-VirtualMachineMemory <int64>] `
[-VirtualProcessorCount <int>] `
[-VirtualMachineDiffDiskPath <String>] `
[-PhysicalAdapterMACAddress <String>] `
[-VirtualSwitchName <String>] `
[-ReCreate] `
[-AsJob] `
[-Passthru] `
[-WhatIf] `
[-Confirm] `
[<CommonParameters>]
```

```powershell
New-OAW
-LocalAdministratorPassword <Security.SecureString> `
-IPAddress <String> `
-SubnetMask <String> `
-DefaultGateway <String> `
-DNS <String[]> `
[-AzureStackCertificatePath <String>] `
[-AzSStampInfoFilePath <String>] `
[-CertificatePassword <Security.SecureString>] `
[-ERCSVMIP <String[]>] `
[-ImageFilePath <String>] `
[-VirtualMachineName <String>] `
[-VirtualMachineMemory <int64>] `
[-VirtualProcessorCount <int>] `
[-VirtualMachineDiffDiskPath <String>] `
[-PhysicalAdapterMACAddress <String>] `
[-VirtualSwitchName <String>] `
[-ReCreate] `
[-AsJob] `
[-Passthru] `
[-WhatIf] `
[-Confirm] `
[<CommonParameters>]
```

次の表に、各パラメーターの定義を示します。

| パラメーター   | 必須/省略可能  | 説明       |
|-------------|--------------------|-------------------|
| LocalAdministratorPassword | 必須 | 仮想マシンのローカル管理者アカウント AdminUser のパスワード。 |
| IPAddress                  | 必須 | 仮想マシンで TCP/IP を構成するための静的 IPv4 アドレス。                                                |
| SubnetMask                 | 必須 | 仮想マシンで TCP/IP を構成するための IPv4 サブネット マスク。                                                   |
| DefaultGateway             | 必須 | 仮想マシンで TCP/IP を構成するための既定のゲートウェイの IPv4 アドレス。                                    |
| DNS                        | 必須 | 仮想マシンで TCP/IP を構成するための DNS サーバー。                                                          |
| ImageFilePath              | オプション | Microsoft が提供する OAW.vhdx のパス。 既定値は、このスクリプトと同じ親フォルダーにある **OAW.vhdx** です。 |
| VirtualMachineName         | オプション | 仮想マシンに割り当てる名前。 名前付けプレフィックスが DeploymentData.json ファイルで見つかった場合は、既定の名前として使用されます。 その他の場合、既定の名前として **AzSOAW** が使用されます。 別の名前を指定し、既定値を上書きできます。 |
| VirtualMachineMemory       | オプション | 仮想マシンに割り当てるメモリ。 既定値は **4 GB** です。                            |
| VirtualProcessorCount      | オプション | 仮想マシンに割り当てる仮想プロセッサの数。 既定値は **8** です。        |
| VirtualMachineDiffDiskPath | オプション | 管理 VM がアクティブだった間に一時差分ディスク ファイルを保存するパス。 既定値は、このスクリプトの同じ親フォルダーにある **DiffDisks** サブディレクトリです。 |
| AzureStackCertificatePath  | オプション | Azure Stack Hub にアクセスするために仮想マシンにインポートする証明書のパス。 |
| AzSStampInfoFilePath       | オプション | スクリプトで ERCS VM の IP を取得できる AzureStackStampInformation.json ファイルのパス。 |
| CertificatePassword        | オプション | Azure Stack Hub へのアクセスのために仮想マシンにインポートする証明書のパスワード。 |
| ERCSVMIP                   | オプション | 仮想マシンの信頼されたホストの一覧に追加する Azure Stack Hub ERCS VM の IP。 **-SkipNetworkConfiguration** が設定されている場合は、有効になりません。 |
SkipNetworkConfiguration     | オプション | ユーザーが後で構成できるように、仮想マシンのネットワーク構成をスキップします。 |
| DeploymentDataFilePath     | オプション | DeploymentData.json のパス。 **-SkipNetworkConfiguration** が設定されている場合は、有効になりません。            |
| PhysicalAdapterMACAddress  | オプション | 仮想マシンの接続に使用される、ホストのネットワーク アダプターの MAC アドレス。<br>- 物理ネットワーク アダプターが 1 つしかない場合、このパラメーターは不要で、その唯一のネットワーク アダプターが使用されます。<br>- 物理ネットワーク アダプターが複数ある場合は、使用するものを指定するために、このパラメーターが必須となります。<br> |
| VirtualSwitchName          | オプション | 仮想マシンの Hyper-V で構成する必要がある仮想スイッチの名前。<br>- 指定された名前の VMSwitch がある場合は、その VMSwitch が選択されます。<br>- 指定された名前の VMSwitch がない場合は、指定された名前で VMSwitch が作成されます。<br> |
| ReCreate                   | オプション | 同じ名前の仮想マシンが既に存在する場合は、その仮想マシンを削除して再作成します。 |

## <a name="check-the-oaw-vm-version"></a>OAW VM のバージョンを確認する

1. 自分の資格情報を使用して OAW VM にログオンします。
1. PowerShell ISE を開き、次のスクリプトを実行します。

   ```powershell
   'C:\Version\Get-Version.ps1'
   ```

   次に例を示します。

   ![ハードウェア ライフサイクル ホストのバージョンを確認するための PowerShell コマンドレットのスクリーンショット](./media/operator-access-workstation/check-operator-access-workstation-vm-version.png)

## <a name="transfer-files-between-the-hlh-and-oaw"></a>HLH と OAW の間でファイルを転送する

HLH と OAW の間でファイルを転送する必要がある場合は、[New-SmbShare](/powershell/module/smbshare/new-smbshare?view=win10-ps) コマンドレットを使用して SMB 共有を作成します。 New-SmbShare により、ファイル システム フォルダーがサーバー メッセージ ブロック (SMB) 共有として、リモート クライアントに公開されます。 次に例を示します。

このコマンドレットによって作成された共有を削除するには、[Remove-SmbShare](/powershell/module/smbshare/remove-smbshare?view=win10-ps) コマンドレットを使用します。 次に例を示します。

## <a name="remove-the-oaw-vm"></a>OAW VM を削除する

次のスクリプトでは、管理と診断のために Azure Stack Hub へのアクセスに使用される OAW VM を削除します。 また、このスクリプトでは、その VM に関連付けられているディスク ファイルとガーディアンも削除します。

1. 自分の資格情報を使用して HLH にログオンします。
1. 管理者特権の PowerShell セッションを開きます。 
1. インストールされている OAW.zip ファイルの抽出されたコンテンツに移動します。
1. Remove-OAW.ps1 スクリプトを実行して VM を削除します。 

   ```powershell
   Remove-OAW.ps1 -VirtualMachineName <name>
   ```

   \<name\> は、削除する仮想マシンの名前です。 既定では、この名前は **AzSOAW** です。

   次に例を示します。

   ```powershell
   Remove-OAW.ps1 -VirtualMachineName AzSOAW
   ```

## <a name="next-steps"></a>次のステップ

[Azure Stack 管理タスク](azure-stack-manage-basics.md)
