---
title: Docker を使用して Azure Stack Hub 内で PowerShell を実行する
description: Docker を使用して Azure Stack Hub 内で PowerShell を実行する
author: mattbriggs
ms.topic: how-to
ms.date: 12/16/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 12/16/2020
ms.openlocfilehash: d187058561f166084b672d941262c7979f7deec2
ms.sourcegitcommit: 52c934f5eeb5fcd8e8f2ce3380f9f03443d1e445
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/07/2021
ms.locfileid: "97974049"
---
# <a name="use-docker-to-run-powershell-for-azure-stack-hub"></a>Docker を使用して Azure Stack Hub に対して PowerShell を実行する

この記事では、Docker を使用してコンテナーを作成します。そのコンテナー上で、さまざまなインターフェイスの操作に必要な PowerShell のバージョンを実行できます。 AzureRM モジュールと最新の Az モジュールの両方を使用する手順がわかります。 AzureRM を使用するには、Windows ベースのコンテナーが必要です。 Az では、Linux ベースのコンテナーが使用されます。

## <a name="docker-prerequisites"></a>Docker の前提条件

### <a name="install-docker"></a>Docker をインストールする

1. [Docker](https://docs.docker.com/install/) をインストールします。

1. PowerShell や Bash などのコマンドライン ツールで、次を入力します。

    ```bash
    docker --version
    ```

### <a name="set-up-a-service-principal-for-using-powershell"></a>PowerShell を使用してサービス プリンシパルを設定する

PowerShell を使用して Azure Stack Hub 内のリソースにアクセスするには、お使いの Azure Active Directory (Azure AD) テナントのサービス プリンシパルが必要になります。 ユーザーのロールベースのアクセス制御 (RBAC) を使用してアクセス許可を委任します。 クラウド オペレーターにサービス プリンシパルを要求することが必要になる場合があります。

1. サービス プリンシパルを設定するには、[サービス プリンシパルを作成することによるアプリケーションへの Azure Stack Hub リソースに対するアクセスの付与](../operator/azure-stack-create-service-principals.md)に関するページの手順に従います。

2. 後で使用するので、アプリケーション ID、シークレット、テナント ID、オブジェクト ID を記録しておきます。

## <a name="run-powershell-in-docker"></a>Docker で PowerShell を実行する

### <a name="az-modules"></a>[Az モジュール](#tab/az)

以下の手順では、PowerShell と、Azure Stack Hub に必要なモジュールが含まれる Linux ベースのコンテナー イメージを実行します。

1. Docker は、Linux コンテナーを使用して実行する必要があります。 Docker を実行するときは、Linux コンテナーに切り替えます。

1. Azure Stack Hub と同じドメインに参加しているマシンから Docker を実行します。 Azure Stack Development Kit (ASDK) を使用している場合は、[お使いのリモート マシン上に VPN](azure-stack-connect-azure-stack.md#connect-to-azure-stack-hub-with-vpn) をインストールする必要があります。


## <a name="install-azure-stack-hub-az-module-on-a-linux-container"></a>Linux コンテナーに Azure Stack Hub Az モジュールをインストールする

1. コマンド ラインから、次の Docker コマンドを実行して、Ubuntu コンテナーで PowerShell を実行します。

    ```bash
    docker run -it mcr.microsoft.com/azurestack/powershell
    ```

    Ubuntu、Debian、または Centos を実行できます。 次の Docker ファイルは、GitHub リポジトリ [azurestack-powershell](https://github.com/Azure/azurestack-powershell) で見つけることができます。 Docker ファイルに対する最新の変更については、GitHub リポジトリを参照してください。 各 OS にタグが付けられています。 タグ (コロンの後のセクション) を目的の OS のタグに置き換えます。

    | Linux | Docker イメージ |
    | --- | --- |
    | Ubuntu | `docker run -it mcr.microsoft.com/azurestack/powershell:ubuntu-18.04` |
    | Debian | `docker run -it mcr.microsoft.com/azurestack/powershell:debian-9` |
    | Centos | `docker run -it mcr.microsoft.com/azurestack/powershell:centos-7` |

2. コマンドレットに対するシェルの準備が整いました。 サインインして `Test-AzureStack.ps1` を実行し、シェルの接続をテストします。

    最初に、サービス プリンシパルの資格情報を作成します。 **シークレット** と **アプリケーション ID** が必要になります。 また、`Test-AzureStack.ps1` を実行してコンテナーを確認するときは、**オブジェクト ID** も必要になります。 クラウド オペレーターにサービス プリンシパルを要求することが必要になる場合があります。

    次のコマンドレットを入力して、サービス プリンシパル オブジェクトを作成します。

    ```powershell  
    $passwd = ConvertTo-SecureString <Secret> -AsPlainText -Force
    $pscredential = New-Object System.Management.Automation.PSCredential('<ApplicationID>', $passwd)
    ```

5. Azure Stack Hub インスタンスから、次の値を使用して次のスクリプトを実行し、環境に接続します。

    | 値 | 説明 |
    | --- | --- |
    | 環境の名前。 | Azure Stack Hub 環境の名前。 |
    | Resource Manager エンドポイント | Resource Manager の URL。 不明な場合は、クラウド オペレーターに問い合わせてください。 これは、`https://management.region.domain.com` のように表示されます。 | 
    | ディレクトリ テナント ID | Azure Stack Hub テナント ディレクトリの ID。 | 
    | 資格情報 | サービス プリンシパルが含まれるオブジェクト。 この場合は、`$pscredential` です。  |

    ```powershell
    ./Login-Environment.ps1 -Name <String> -ResourceManagerEndpoint <resource manager endpoint> -DirectoryTenantId <String> -Credential $pscredential
    ```

   PowerShell からアカウント オブジェクトが返されます。

7. コンテナーで `Test-AzureStack.ps1` スクリプトを実行して、環境をテストします。 サービス プリンシパルの **オブジェクト ID** を指定します。 オブジェクト ID を指定しなくても、スクリプトは実行されますが、テナント (ユーザー) モジュールだけがテストされ、管理者特権を必要とするモジュールでは失敗します。

    ```powershell  
    ./Test-AzureStack.ps1 <Object ID>
    ```

### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/rm)

以下の手順では、Windows ベースのコンテナー イメージを実行し、PowerShell と、Azure Stack Hub に必要なモジュールをインストールします。 Windows で Docker を実行する場合は、物理マシンで Docker を実行する必要があります。 入れ子になった仮想化は、Hyper-V ではサポートされていません。

1. Windows 10 を必要とする Windows コンテナーを使って Docker を実行する必要があります。 Docker を実行しているときに、Windows コンテナーに切り替えます。 Az モジュールをサポートするイメージには、Docker 17.05 以降が必要です。

1. Azure Stack Hub と同じドメインに参加しているマシンから Docker を実行します。 Azure Stack Development Kit (ASDK) を使用している場合は、[お使いのリモート マシン上に VPN](azure-stack-connect-azure-stack.md#connect-to-azure-stack-hub-with-vpn) をインストールする必要があります。

### <a name="install-azure-stack-hub-azurerm-module-on-a-windows-container"></a>Windows コンテナーに Azure Stack Hub AzureRM モジュールをインストールする

Dockerfile は、Windows PowerShell 5.1 がインストールされている Microsoft イメージ *microsoft/windowsservercore* を開きます。 その後、そのファイルは、NuGet と Azure Stack Hub PowerShell モジュールを読み込み、Azure Stack Hub ツールからツールをダウンロードします。

1. ZIP ファイルとして [azure-stack-powershell リポジトリをダウンロードする](https://github.com/Azure-Samples/azure-stack-hub-powershell-in-docker.git)か、そのリポジトリを複製します。

2. ターミナルからリポジトリ フォルダーを開きます。

3. お使いのリポジトリでコマンド ライン インターフェイスを開き、次のコマンドを入力します。

    ```bash  
    docker build --tag azure-stack-powershell .
    ```

4. イメージがビルドされたら、次を入力して、対話型コンテナーを開始します。

    ```bash  
    docker run -it azure-stack-powershell powershell
    ```

    コンテナー名を書き留めます。 毎回新しいコンテナーを作成するのではなく、次の Docker コマンドを使用して同じコンテナーを使用できます。

    ```bash  
        docker exec -it "Container name" powershell
    ```

5. コマンドレットに対するシェルの準備が整いました。

    ```bash
    Windows PowerShell
    Copyright (C) 2016 Microsoft Corporation. All rights reserved.

    PS C:\>
    ```

6. サービス プリンシパルを使用して、お使いの Azure Stack Hub インスタンスに接続します。 現在、Docker で PowerShell プロンプトを使用しています。 

    ```powershell
    $passwd = ConvertTo-SecureString <Secret> -AsPlainText -Force
    $pscredential = New-Object System.Management.Automation.PSCredential('<ApplicationID>', $passwd)
    Add-AzureRMEnvironment -Name "AzureStackUser" -ArmEndpoint <Your Azure Resource Manager endoint>
    Add-AzureRmAccount -EnvironmentName "AzureStackUser" -TenantId <TenantID> -ServicePrincipal -Credential $pscredential
    ```

   PowerShell からアカウント オブジェクトが返されます。

    ```powershell  
    Account    SubscriptionName    TenantId    Environment
    -------    ----------------    --------    -----------
    <AccountID>    <SubName>       <TenantID>  AzureCloud
    ```

7. Azure Stack Hub でリソース グループを作成して、接続をテストします。

    ```powershell  
    New-AzureRmResourceGroup -Name "MyResourceGroup" -Location "Local"
    ```

---

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub の Azure Stack Hub PowerShell](azure-stack-powershell-overview.md) の概要をお読みください。
- Azure Stack Hub の [PowerShell の API プロファイル](azure-stack-version-profiles.md)についてお読みください。
- [Azure Stack Hub PowerShell](../operator/azure-stack-powershell-install.md) をインストールします。
- クラウドの一貫性のための [Azure Resource Manager テンプレート](azure-stack-develop-templates.md)の作成についてお読みください。
