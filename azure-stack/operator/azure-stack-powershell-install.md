---
title: Azure Stack Hub 用の PowerShell AzureRM モジュールをインストールする
description: PowerShell for Azure Stack Hub をインストールする方法について説明します。 PowerShell AzureRM モジュールのインストール方法と必要な API プロファイルを確認します。
author: mattbriggs
ms.topic: article
ms.date: 08/04/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 08/04/2020
ms.openlocfilehash: bbf1a5d296ddbef554a4401e66eab4226ae38dd3
ms.sourcegitcommit: a1e2003fb9c6dacdc76f97614ff5a26a5b197b49
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/01/2020
ms.locfileid: "91623168"
---
# <a name="install-powershell-azurerm-module-for-azure-stack-hub"></a>Azure Stack Hub 用の PowerShell AzureRM モジュールをインストールする

Azure PowerShell AzureRM には、Azure Stack Hub リソースの管理に Azure Resource Manager モデルを使用する一連のコマンドレットが用意されています。

また、"*API プロファイル*" を使用して、互換性のある Azure Stack Hub リソース プロバイダーのエンドポイントを指定する必要もあります。

API プロファイルには、Azure と Azure Stack Hub のバージョンの違いを管理するための方法が用意されています。 API バージョンのプロファイルは、特定の API バージョンを持つ一連の Azure Resource Manager PowerShell モジュールです。 各クラウド プラットフォームでは、一連の API バージョンのプロファイルがサポートされています。 たとえば、Azure Stack Hub では、**2019-03-01-hybrid** などの特定のプロファイル バージョンがサポートされます。 プロファイルをインストールすると、指定されたプロファイルに対応する Azure Resource Manager PowerShell モジュールがインストールされます。

Azure Stack Hub と互換性のある PowerShell モジュールは、インターネットに接続されたシナリオ、部分的に接続されたシナリオ、または接続が切断されたシナリオでインストールできます。 この記事では、これらのシナリオの詳細な手順について説明します。

Docker コンテナーで Azure Stack Hub の AzureRM モジュールを実行することもできます。 手順については、[Docker を使用して Azure Stack Hub に対して PowerShell を実行する](../user/azure-stack-powershell-user-docker.md)方法に関するページを参照してください。

## <a name="1-verify-your-prerequisites"></a>1.前提条件を確認する

Azure Stack Hub と PowerShell AzureRM モジュールの使用を開始する前に、次の前提条件を満たしている必要あります。

- **PowerShell バージョン 5.1** <br>
バージョンを確認するには、 **$PSVersionTable.PSVersion** を実行して、**メジャー** バージョンを比較します。 PowerShell 5.1 を使用していない場合は、「[Windows PowerShell のインストール](/powershell/scripting/install/installing-windows-powershell#upgrading-existing-windows-powershell)」に従ってください。

  > [!Note]
  > PowerShell 5.1 には、Windows マシンが必要です。

- **PowerShell の実行 (管理者特権でのコマンド プロンプト)** 。

- **PowerShell ギャラリーへのアクセス** <br>
  [PowerShell ギャラリー](https://www.powershellgallery.com)へのアクセスが必要になります。 ギャラリーとは、PowerShell コンテンツのための中央のリポジトリです。 **PowerShellGet** モジュールには、PowerShell 成果物の検出、インストール、更新、および発行を行うためのコマンドレットが含まれています。 これらの成果物の例としては、モジュール、DSC リソース、ロール機能、PowerShell ギャラリーおよびその他のプライベート リポジトリからのスクリプトなどがあります。 切断されたシナリオで PowerShell を使用している場合は、インターネットに接続しているマシンからリソースを取得し、切断されたマシンがアクセスできる場所に格納する必要があります。

## <a name="2-validate-the-powershell-gallery-accessibility"></a>2.PowerShell ギャラリーにアクセスできるか検証する

PSGallery がリポジトリとして登録されているかどうかを検証します。

> [!Note]  
> この手順では、インターネットへのアクセスが必要です。

管理者特権の PowerShell プロンプトを開き、次のコマンドレットを実行します。

```powershell
Install-module -Name PowerShellGet -Force
Import-Module -Name PackageManagement -ErrorAction Stop
Get-PSRepository -Name "PSGallery"
```

リポジトリが登録されていない場合は、管理者特権の PowerShell セッションを開き、次のコマンドを実行します。

```powershell
Register-PSRepository -Default
Set-PSRepository -Name "PSGallery" -InstallationPolicy Trusted
```

## <a name="3-uninstall-existing-versions-of-the-azure-stack-hub-powershell-modules"></a>3.既存のバージョンの Azure Stack Hub PowerShell モジュールをアンインストールする

必要なバージョンをインストールする前に、必ず以前にインストールした Azure Stack Hub AzureRM PowerShell モジュールをアンインストールしてください。 モジュールをアンインストールするには、次の 2 つの方法のいずれかを使用します。

1. 既存の AzureRM と Az PowerShell モジュールをアンインストールするには、アクティブな PowerShell セッションをすべて閉じ、次のコマンドレットを実行します。

    ```powershell
    Get-Module -Name Azure* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    Get-Module -Name Azs.* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    Get-Module -Name Az.* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    ```

    「モジュールが既に使用されています」などのエラーが発生した場合には、そのモジュールを使用している PowerShell セッションを終了してから、上のスクリプトをもう一度実行してください。

2. `C:\Program Files\WindowsPowerShell\Modules` と `C:\Users\{yourusername}\Documents\WindowsPowerShell\Modules` の各フォルダーから、`Azure`、`Az`、または `Azs.` で始まるすべてのフォルダーを削除します。 これらのフォルダーを削除すると、既存の PowerShell モジュールがすべて削除されます。

## <a name="4-connected-install-powershell-for-azure-stack-hub-with-internet-connectivity"></a>4.接続済みの場合: インターネット接続を使用して PowerShell for Azure Stack Hub をインストールする

必要な API バージョン プロファイルおよび Azure Stack Hub PowerShell モジュールは、実行している Azure Stack Hub のバージョンによって異なります。

### <a name="install-azure-stack-hub-powershell"></a>Azure Stack Hub PowerShell のインストール

次の PowerShell スクリプトを実行して、これらのモジュールを開発用ワークステーションにインストールします。

::: moniker range=">=azs-2002"
Azure Stack Hub 2002 以降の場合:

AzureRm モジュールまたは Az プレビュー モジュールのいずれかを使用できます。 Az モジュールを使用するには Azure Stack Hub 2002 と最新の修正プログラムが必要です。

Az プレビュー モジュールを使用するには [PowerShell Az モジュールのインストール](powershell-install-az-module.md)に関する説明に従ってください。

```powershell  
# Install the AzureRM.BootStrapper module. Select Yes when prompted to install NuGet
Install-Module -Name AzureRM.BootStrapper

# Install and import the API Version Profile required by Azure Stack Hub into the current PowerShell session.
Use-AzureRmProfile -Profile 2019-03-01-hybrid -Force
Install-Module -Name AzureStack -RequiredVersion 1.8.2
```

::: moniker-end
::: moniker range="azs-1910"
Azure Stack Hub 1910 の場合:

```powershell  
# Install the AzureRM.BootStrapper module. Select Yes when prompted to install NuGet
Install-Module -Name AzureRM.BootStrapper

# Install and import the API Version Profile required by Azure Stack Hub into the current PowerShell session.
Use-AzureRmProfile -Profile 2019-03-01-hybrid -Force
Install-Module -Name AzureStack -RequiredVersion 1.8.0
```

> [!Note]  
> - Azure Stack Hub モジュール バージョン 1.8.0 は破壊的変更を伴うリリースです。 詳細については、[リリース ノート](release-notes.md)を参照してください。

::: moniker-end
::: moniker range="<=azs-1908"
Azure Stack Hub 1908 以前の場合:

```powershell  
# Install the AzureRM.BootStrapper module. Select Yes when prompted to install NuGet
Install-Module -Name AzureRM.BootStrapper

# Install and import the API Version Profile required by Azure Stack Hub into the current PowerShell session.
Use-AzureRmProfile -Profile 2019-03-01-hybrid -Force
Install-Module -Name AzureStack -RequiredVersion 1.7.2
```

> [!Note]  
> Azure Stack Hub モジュール バージョン 1.7.2 は破壊的変更を伴うリリースです。 Azure Stack Hub 1.6.0 から移行するには、[移行ガイド](https://aka.ms/azspshmigration171)を参照してください。

::: moniker-end

### <a name="confirm-the-installation-of-powershell"></a>PowerShell のインストールを確認する

次のコマンドを実行してインストールを確認します。

```powershell
Get-Module -Name "Azure*" -ListAvailable
Get-Module -Name "Azs*" -ListAvailable
```

インストールに成功すると、出力に `AzureRm` モジュールと `AzureStack` モジュールが表示されます。

## <a name="5-disconnected-install-powershell-without-an-internet-connection"></a>5.切断状態の場合: インターネット接続なしで PowerShell をインストールする

接続が切断されたシナリオでは、まずインターネット接続が確立されたマシンに PowerShell モジュールをダウンロードします。 次に、それらを Azure Stack Development Kit (ASDK) に転送してインストールします。

インターネット接続が確立されたコンピューターにサインインし、次のスクリプトを使用して、Azure Stack Hub のバージョンに応じた Azure Resource Manager および Azure Stack Hub パッケージをダウンロードします。

インストールには 5 つの手順があります。

1. 接続されているマシンに Azure Stack Hub PowerShell をインストールします。
2. 追加のストレージ機能を有効にします。
3. 接続が切断されたワークステーションに PowerShell パッケージを転送します。
4. 切断されたワークステーション上で NuGet プロバイダーを手動でブートストラップします。
5. PowerShell のインストールを確認します。

### <a name="install-azure-stack-hub-powershell"></a>Azure Stack Hub PowerShell のインストール

::: moniker range=">=azs-2002"
Azure Stack Hub 2002 以降。

AzureRM または Az プレビュー モジュールのいずれかを使用できます。 Az モジュールについては、[PowerShell Az モジュールのインストール](powershell-install-az-module.md)に関する記事の手順を参照してください。

```powershell

Install-module -Name PowerShellGet -Force 
Import-Module -Name PackageManagement -ErrorAction Stop

$Path = "<Path that is used to save the packages>"
Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureRM -Path $Path -Force -RequiredVersion 2.5.0
Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureStack -Path $Path -Force -RequiredVersion 1.8.2
```
::: moniker-end

::: moniker range="azs-1910"
Azure Stack Hub 1910。

```powershell
Install-module -Name PowerShellGet -Force 
Import-Module -Name PackageManagement -ErrorAction Stop

$Path = "<Path that is used to save the packages>"
Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureRM -Path $Path -Force -RequiredVersion 2.5.0
Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureStack -Path $Path -Force -RequiredVersion 1.8.0
```

> [!NOTE]  
> Azure Stack Hub モジュール バージョン 1.8.0 は破壊的変更を伴うリリースです。 詳細については、[リリース ノート](release-notes.md)を参照してください。

::: moniker-end
::: moniker range="<=azs-1908"
Azure Stack Hub 1908 以前の場合:

```powershell
Install-module -Name PowerShellGet -Force 
Import-Module -Name PackageManagement -ErrorAction Stop

$Path = "<Path that is used to save the packages>"
Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureRM -Path $Path -Force -RequiredVersion 2.5.0
Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureStack -Path $Path -Force -RequiredVersion 1.7.2
```

> [!NOTE]  
> Azure Stack Hub モジュール バージョン 1.7.1 は破壊的変更を伴うリリースです。 Azure Stack Hub 1.6.0 から移行するには、[移行ガイド](https://github.com/Azure/azure-powershell/tree/AzureRM/documentation/migration-guides/Stack)を参照してください。

::: moniker-end

> [!NOTE]  
> インターネット接続のないマシンでは、次のコマンドレットを実行して利用統計情報の収集を無効にすることをお勧めします。 利用統計情報の収集を無効にしないと、コマンドレットのパフォーマンスが低下することがあります。 これは、インターネット接続のないマシンのみ該当します。
> ```powershell
> Disable-AzureRmDataCollection
> ```

### <a name="add-your-packages-to-your-workstation"></a>パッケージをワークステーションに追加する

1. ダウンロードしたパッケージを USB デバイスにコピーします。

2. 接続が切断されたワークステーションにサインインし、パッケージを USB デバイスからワークステーション上の場所にコピーします。

3. 切断されたワークステーション上で NuGet プロバイダーを手動でブートストラップします。 手順については、「[インターネットに接続されていないマシンで NuGet プロバイダーを手動でブートストラップする](/powershell/scripting/gallery/how-to/getting-support/bootstrapping-nuget#manually-bootstrapping-the-nuget-provider-on-a-machine-that-is-not-connected-to-the-internet)」をご覧ください。

4. この場所を既定のレポジトリとして登録し、このレポジトリから AzureRM モジュールと `AzureStack` モジュールをインストールします。

   ```powershell
   # requires -Version 5
   # requires -RunAsAdministrator
   # requires -Module PowerShellGet
   # requires -Module PackageManagement

   $SourceLocation = "<Location on the development kit that contains the PowerShell packages>"
   $RepoName = "MyNuGetSource"

   Register-PSRepository -Name $RepoName -SourceLocation $SourceLocation -InstallationPolicy Trusted

   Install-Module -Name AzureRM -Repository $RepoName

   Install-Module -Name AzureStack -Repository $RepoName
   ```

### <a name="confirm-the-installation-of-powershell"></a>PowerShell のインストールを確認する

次のコマンドを実行してインストールを確認します。

```powershell
Get-Module -Name "Azure*" -ListAvailable
Get-Module -Name "Azs*" -ListAvailable
```

## <a name="6-configure-powershell-to-use-a-proxy-server"></a>6.プロキシ サーバーを使用するように PowerShell を構成する

インターネットへのアクセスにプロキシ サーバーを必要とするシナリオでは、まず既存のプロキシ サーバーを使用するように PowerShell を構成します。

1. 管理者特権の PowerShell プロンプトを開きます。
2. 次のコマンドを実行します。

   ```powershell
   #To use Windows credentials for proxy authentication
   [System.Net.WebRequest]::DefaultWebProxy.Credentials = [System.Net.CredentialCache]::DefaultCredentials

   #Alternatively, to prompt for separate credentials that can be used for #proxy authentication
   [System.Net.WebRequest]::DefaultWebProxy.Credentials = Get-Credential
   ```

## <a name="known-issue"></a>既知の問題

###  <a name="method-get_serializationsettings-error"></a>get_SerializationSettings メソッドのエラー 

- 原因: PowerShell Az モジュールと PowerShell AzureRM モジュールには互換性がありません。

    次のエラーは、AzureRM モジュールと Az モジュールが同じセッションに読み込まれたことを示しています。 

    ```powershell  
    >  Method 'get_SerializationSettings' in type 'Microsoft.Azure.Management.Internal.Resources.ResourceManagementClient' from assembly 'Microsoft.Azure.Commands.ResourceManager.Common, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35' does 
    not have an implementation.
    ```

- 修復: 競合しているモジュールをアンインストールします。 

  AzureRM モジュールを使用する場合は、Az モジュールをアンインストールします。 または、Az モジュールを使用する場合は、AzureRM をアンインストールします。 ご自分の PowerShell セッションを閉じ、Az または AzureRM モジュールのいずれかをアンインストールします。 
  
  手順については、「[既存のバージョンの Azure Stack Hub PowerShell モジュールをアンインストールする](#3-uninstall-existing-versions-of-the-azure-stack-hub-powershell-modules)」を参照してください。

## <a name="next-steps"></a>次のステップ

- [GitHub からの Azure Stack Hub ツールのダウンロード](azure-stack-powershell-download.md)
- [Azure Stack Hub ユーザーの PowerShell 環境の構成](../user/azure-stack-powershell-configure-user.md)
- [Azure Stack Hub オペレーターの PowerShell 環境の構成](azure-stack-powershell-configure-admin.md)
- [Azure Stack Hub での API バージョンのプロファイルの管理](../user/azure-stack-version-profiles.md)
