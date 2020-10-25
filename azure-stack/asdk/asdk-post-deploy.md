---
title: ASDK のデプロイ後の構成
description: Azure Stack Development Kit (ASDK) をインストールした後に行うことをお勧めする構成変更について説明します。
author: justinha
ms.topic: article
ms.date: 07/31/2019
ms.author: justinha
ms.reviewer: misainat
ms.lastreviewed: 07/31/2019
ms.openlocfilehash: 9417d3b691e3394d254ce1d599748233bf78213a
ms.sourcegitcommit: af7f169c7e204ffdf344f47c07ab8426e2afbd1d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/06/2020
ms.locfileid: "87865134"
---
# <a name="post-deployment-configurations-for-asdk"></a>ASDK のデプロイ後の構成

[Azure Stack Development Kit (ASDK) をインストール](asdk-install.md)した後、ASDK ホスト コンピューターで AzureStack\AzureStackAdmin としてサインイン中に、いくつかの推奨されるデプロイ後の構成変更を行う必要があります。

## <a name="install-azure-stack-powershell"></a>Azure Stack PowerShell のインストール

Azure Stack を使用するには、Azure Stack と互換性のある Azure PowerShell モジュールが必要です。

Azure Stack 用の PowerShell コマンドは、PowerShell ギャラリーを介してインストールされます。 PSGallery リポジトリを登録するには、PowerShell セッションを管理者特権で開き、次のコマンドを実行します。

``` Powershell
Set-PSRepository -Name "PSGallery" -InstallationPolicy Trusted
```

API バージョン プロファイルを使用して、Azure Stack と互換性のある AzureRM モジュールを指定します。  API バージョンのプロファイルは、Azure と Azure Stack の間のバージョンの違いを管理するための方法を提供します。 API バージョンのプロファイルは、特定の API バージョンを持つ一連の AzureRM PowerShell モジュールです。 PowerShell ギャラリーから入手できる **AzureRM.BootStrapper** モジュールは、API バージョンのプロファイルを操作するために必要な PowerShell コマンドレットを提供します。

ASDK ホスト コンピューターへのインターネット接続の有無にかかわらず、最新の Azure Stack PowerShell モジュールをインストールすることができます。

> [!IMPORTANT]
> 必要なバージョンをインストールする前に、必ず[既存の Azure PowerShell モジュールをアンインストールしてください](../operator/azure-stack-powershell-install.md#3-uninstall-existing-versions-of-the-azure-stack-hub-powershell-modules)。

- ASDK ホスト コンピューターからの**インターネット接続がある場合**:次の PowerShell スクリプトを実行して、これらのモジュールを ASDK インストールにインストールします。


  ```powershell  
  Get-Module -Name Azs.* -ListAvailable | Uninstall-Module -Force -Verbose
  Get-Module -Name Azure* -ListAvailable | Uninstall-Module -Force -Verbose

  # Install the AzureRM.BootStrapper module. Select Yes when prompted to install NuGet
  Install-Module -Name AzureRM.BootStrapper

  # Install and import the API Version Profile required by Azure Stack into the current PowerShell session.
  Use-AzureRmProfile -Profile 2019-03-01-hybrid -Force
  Install-Module -Name AzureStack -RequiredVersion 1.8.2
  ```

  インストールに成功した場合、出力に AzureRM および AzureStack モジュールが表示されます。

- ASDK ホスト コンピューターからの**インターネット接続がない場合**:接続が切断されたシナリオでは、まず次の PowerShell コマンドを使って、インターネット接続が確立されたマシンに PowerShell モジュールをダウンロードします。

  ```powershell
  $Path = "<Path that is used to save the packages>"

  Save-Package `
    -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureRM -Path $Path -Force -RequiredVersion 2.3.0
  
  Save-Package `
    -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureStack -Path $Path -Force -RequiredVersion 1.5.0
  ```

  ここで、ダウンロードしたパッケージを ASDK コンピューターにコピーし、その場所を既定のリポジトリとして登録し、そのリポジトリから AzureRM および AzureStack モジュールをインストールします。

    ```powershell  
    $SourceLocation = "<Location on the development kit that contains the PowerShell packages>"
    $RepoName = "MyNuGetSource"

    Register-PSRepository -Name $RepoName -SourceLocation $SourceLocation -InstallationPolicy Trusted

    Install-Module AzureRM -Repository $RepoName

    Install-Module AzureStack -Repository $RepoName
    ```

## <a name="download-the-azure-stack-tools"></a>Azure Stack ツールをダウンロードする

[AzureStack-Tools](https://github.com/Azure/AzureStack-Tools) は PowerShell モジュールをホストする GitHub リポジトリで、リソースの管理と Azure Stack へのデプロイに使用できます。 これらのツールを入手するには、GitHub リポジトリを複製するか、次のスクリプトを実行して AzureStack-Tools フォルダーをダウンロードします。

  ```powershell
  # Change directory to the root directory.
  cd \

  # Enforce usage of TLSv1.2 to download the Azure Stack tools archive from GitHub
  [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
  Invoke-WebRequest `
    -Uri https://github.com/Azure/AzureStack-Tools/archive/master.zip `
    -OutFile master.zip

  # Expand the downloaded files.
  Expand-Archive -Path master.zip -DestinationPath . -Force

  # Change to the tools directory.
  cd AzureStack-Tools-master
  ```

## <a name="validate-the-asdk-installation"></a>ASDK インストールを検証する

ASDK のデプロイが成功したことを確認するには、次の手順に従って Test-AzureStack コマンドレットを使用します。

1. ASDK ホスト コンピューターで AzureStack\AzureStackAdmin としてサインインします。
2. PowerShell を (PowerShell ISE ではなく) 管理者として開きます。
3. `Enter-PSSession -ComputerName AzS-ERCS01 -ConfigurationName PrivilegedEndpoint` を実行します。
4. `Test-AzureStack` を実行します。

テストが完了するまでに数分かかります。 インストールが成功した場合、出力は次のようになります。

![Test Azure Stack - Installation successful (Azure Stack のテスト - インストールに成功しました)](media/asdk-post-deploy/test-azurestack.png)

エラーがあった場合は、トラブルシューティング手順に従ってヘルプを取得します。

## <a name="enable-multi-tenancy"></a>マルチテナントの有効化

Azure AD を使用したデプロイでは、ASDK のインストール用に[マルチ テナント機能を有効にする](../operator/azure-stack-enable-multitenancy.md#enable-multi-tenancy)必要があります。

> [!NOTE]
> Azure Stack の登録に使用したドメイン以外のドメインからの管理者またはユーザー アカウントを使用して Azure Stack ポータルにログインする場合は、Azure Stack の登録に使用したドメイン名をポータルの URL に追加する必要があります。 たとえば、Azure Stack が fabrikam.onmicrosoft.com に登録されていて、ログインするユーザー アカウントが admin@contoso.com である場合、ユーザー ポータルへのログインに使用する URL は https\://portal.local.azurestack.external/fabrikam.onmicrosoft.com になります。

## <a name="next-steps"></a>次のステップ

[ASDK の Azure への登録](asdk-register.md)
