---
title: Azure Stack Hub Storage でのデータ転送ツールの使用
description: Azure Stack Hub Storage のデータ転送ツールについて説明します。
author: mattbriggs
ms.topic: conceptual
ms.date: 11/22/2020
ms.author: mabrigg
ms.reviewer: xiaofmao
ms.lastreviewed: 11/22/2020
ms.openlocfilehash: 209f42ce605f4c3a5abd5fc1ee7ee9df01a6bca9
ms.sourcegitcommit: e88f0a1f2f4ed3bb8442bfb7b754d8b3a51319b4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/04/2021
ms.locfileid: "99534200"
---
# <a name="use-data-transfer-tools-in-azure-stack-hub-storage"></a>Azure Stack Hub Storage でのデータ転送ツールの使用

Azure Stack Hub は、ディスク、BLOB、テーブル、キュー、およびアカウント管理機能のストレージ サービスのセットを提供します。 Azure Stack Hub Storage のデータを管理または移動する場合、いくつかの Azure Storage ツールを利用できます。 この記事では、使用可能なツールの概要について説明します。

お客様の要件によって、次のツールのうちどれが最適であるかが決まります。

* [AzCopy](#azcopy)

    ストレージ固有のコマンドライン ユーティリティであり、これをダウンロードして、ストレージ アカウント内のあるオブジェクトから別のオブジェクトに、またはストレージ アカウント間でデータをコピーすることができます。

* [Azure PowerShell](#azure-powershell)

    タスク ベースのコマンドライン シェルであり、特にシステム管理用に設計されたスクリプト言語です。

* [Azure CLI](#azure-cli)

    オープン ソースでクロスプラットフォームの Azure および Azure Stack Hub Platform で使用できるコマンド群が提供されます。

* [Microsoft Azure Storage Explorer](#microsoft-azure-storage-explorer)

    ユーザー インターフェイスを備えた使いやすいスタンドアロンのアプリ。

* [Blobfuse](#blobfuse)

    ストレージ アカウント内の既存のブロック BLOB データに Linux ファイル システム経由でアクセスできるようにする、Azure Blob Storage 用の仮想ファイル システム ドライバーです。

Azure と Azure Stack Hub の間のストレージ サービスの違いのため、次のセクションで説明されている各ツールにはいくつかの固有の要件があります。 Azure Stack Hub Storage と Azure Storage の比較については、「[Azure Stack Hub ストレージ:違いと考慮事項](azure-stack-acs-differences.md)」をご覧ください。

## <a name="azcopy"></a>AzCopy

AzCopy は、最適なパフォーマンスの単純なコマンドを使用して、Microsoft Azure の Blob Storage と Table Storage との間でデータをコピーするために設計されたコマンドライン ユーティリティです。 ストレージ アカウント内のあるオブジェクトから別のオブジェクトに、またはストレージ アカウント間でデータをコピーできます。

### <a name="download-and-install-azcopy"></a>AzCopy のダウンロードとインストール

::: moniker range=">=azs-1811"
* 1811 以降の更新プログラムの場合は、[AzCopy V10 以降をダウンロード](/azure/storage/common/storage-use-azcopy-v10#download-azcopy)します。
::: moniker-end

::: moniker range="<azs-1811"
* それより前のバージョン (1802 から 1809 までの更新プログラム) では、[AzCopy 7.1.0 をダウンロード](https://aka.ms/azcopyforazurestack20170417)します。
::: moniker-end

### <a name="azcopy-101-configuration-and-limits"></a>AzCopy 10.1 の構成と制限事項

AzCopy 10.1 を構成して、古い API バージョンを使用できるようになりました。 これにより、Azure Stack Hub のサポート (制限あり) が有効になります。
AzCopy の API バージョンを構成して Azure Stack Hub をサポートするには、`AZCOPY_DEFAULT_SERVICE_API_VERSION` 環境変数を `2017-11-09` に設定します。

| オペレーティング システム | command  |
|--------|-----------|
| **Windows** | コマンド プロンプトでは、`set AZCOPY_DEFAULT_SERVICE_API_VERSION=2017-11-09` を使用します<br> PowerShell では、`$env:AZCOPY_DEFAULT_SERVICE_API_VERSION="2017-11-09"` を使用します|
| **Linux** | `export AZCOPY_DEFAULT_SERVICE_API_VERSION=2017-11-09` |
| **MacOS** | `export AZCOPY_DEFAULT_SERVICE_API_VERSION=2017-11-09` |

AzCopy 10.1 では、Azure Stack Hub の次の機能がサポートされます。

| 機能 | サポートされているアクション |
| --- | --- |
|コンテナーを管理する|コンテナーを作成する<br>コンテナーの内容を一覧表示する
|ジョブを管理する|ジョブを表示する<br>ジョブを再開する
|BLOB を削除する|1 つの BLOB を削除する<br>全部または一部の仮想ディレクトリを削除する
|ファイルをアップロードする|ファイルをアップロードする<br>ディレクトリをアップロードする<br>ディレクトリの内容をアップロードする
|ファイルのダウンロード|ファイルをダウンロードする<br>ディレクトリをダウンロードする<br>ディレクトリの内容をダウンロードする
|ファイルを同期する|ローカル ファイル システムにコンテナーを同期する<br>コンテナーにローカル ファイル システムを同期する

   > [!NOTE]
   > * Azure Stack Hub では、Azure Active Directory (AD) の使用による承認資格情報の AzCopy への提供はサポートされていません。 Shared Access Signature (SAS) トークンを使用して、Azure Stack Hub 上にあるストレージ オブジェクトにアクセスする必要があります。
   > * Azure Stack Hub では、2 か所の Azure Stack Hub BLOB の間、および Azure Storage と Azure Stack Hub との間の同期データ転送はサポートされていません。 AzCopy 10.1 を直接使用して Azure Stack Hub から Azure Storage に (または逆方向に) データを移動させるために "azcopy cp" を使用することはできません。

### <a name="azcopy-command-examples-for-data-transfer"></a>データ転送するための AzCopy コマンドの例

以下の例では、Azure Stack Hub BLOB との間でデータをコピーする代表的なシナリオを紹介しています。 詳細については、「[AzCopy を使ってみる](/azure/storage/common/storage-use-azcopy-v10)」を参照してください。

### <a name="download-all-blobs-to-a-local-disk"></a>すべての BLOB をローカル ディスクにダウンロードする

```
azcopy cp "https://[account].blob.core.windows.net/[container]/[path/to/directory]?[SAS]" "/path/to/dir" --recursive=true
```

### <a name="upload-single-file-to-virtual-directory"></a>1 つのファイルを仮想ディレクトリにアップロードする

```
azcopy cp "/path/to/file.txt" "https://[account].blob.core.windows.net/[container]/[path/to/blob]?[SAS]"
```

### <a name="azcopy-known-issues"></a>AzCopy の既知の問題

 - ファイル ストレージはまだ Azure Stack Hub で使用できないため、ファイル ストアに対する AzCopy 操作は使用できません。
 - AzCopy 10.1 を使用して 2 か所の Azure Stack Hub BLOB の間、または Azure Stack Hub と Azure Storage との間でデータを転送する場合は、最初にデータをローカルの場所にダウンロードしてから Azure Stack Hub または Azure Storage 上にあるターゲット ディレクトリに再度アップロードする必要があります。 または、AzCopy 7.1 を使用し、 **/SyncCopy** オプションで転送を指定して、データをコピーすることができます。  
 - Linux バージョンの AzCopy では、1802 以降のバージョンの更新プログラムのみがサポートされており、Table service はサポートされていません。
 - Azure Table Storage サービスとの間でデータをコピーする場合、[AzCopy バージョン 7.3 0 をインストール](https://aka.ms/azcopyforazurestack20171109)してください。
 
## <a name="azure-powershell"></a>Azure PowerShell

Azure PowerShell は、Azure と Azure Stack Hub の両方でサービスを管理するためのコマンドレットを提供するモジュールです。 タスク ベースのコマンドライン シェルであり、特にシステム管理用に設計されたスクリプト言語です。

### <a name="install-and-configure-powershell-for-azure-stack-hub"></a>PowerShell for Azure Stack Hub のインストールと構成

Azure Stack Hub を使用するには、Azure Stack Hub と互換性のある Azure PowerShell モジュールが必要です。 詳細については、「[PowerShell for Azure Stack Hub をインストールする](../operator/powershell-install-az-module.md)」と、[Azure Stack Hub ユーザーの PowerShell 環境の構成](azure-stack-powershell-configure-user.md)に関する記事を参照してください。

### <a name="powershell-sample-script-for-azure-stack-hub"></a>Azure Stack Hub 用の PowerShell サンプル スクリプト 
### <a name="az-modules"></a>[Az モジュール](#tab/az1)

このサンプルでは、[PowerShell for Azure Stack Hub のインストール](../operator/powershell-install-az-module.md)が正常に完了していることを前提としています。 このスクリプトは、構成を完了し、ローカルの PowerShell 環境にアカウントを追加するために Azure Stack Hub テナントの資格情報を要求するために役立ちます。 次に、スクリプトは、既定の Azure サブスクリプションを設定して、Azure 内に新しいストレージ アカウントを作成し、この新しいストレージ アカウントに新しいコンテナーを作成して、既存の画像ファイル (BLOB) をこのコンテナーにアップロードします。 このスクリプトにより、コンテナー内のすべての BLOB がリストされると、ローカル コンピューターに新しい格納先ディレクトリが作成され、画像ファイルがダウンロードされます。

1. [Azure Stack Hub と互換性のある Azure PowerShell モジュール](../operator/powershell-install-az-module.md)をインストールします。
2. [Azure Stack Hub の操作に必要なツール](../operator/azure-stack-powershell-download.md)をダウンロードします。
3. **Windows PowerShell ISE** を開いて、**管理者として実行し**、 **[ファイル]**  >  **[新規作成]** の順にクリックして、新しいスクリプト ファイルを作成します。
4. 次のスクリプトをコピーして新しいスクリプト ファイルに貼り付けます。
5. 構成設定に基づいてスクリプト変数を更新します。
   > [!NOTE]
   > このスクリプトは、**AzureStack_Tools** のルート ディレクトリで実行する必要があります。

```powershell  
# begin

$ARMEvnName = "AzureStackUser" # set AzureStackUser as your Azure Stack Hub environment name
$ARMEndPoint = "https://management.local.azurestack.external" 
$GraphAudience = "https://graph.windows.net/" 
$AADTenantName = "<myDirectoryTenantName>.onmicrosoft.com" 

$SubscriptionName = "basic" # Update with the name of your subscription.
$ResourceGroupName = "myTestRG" # Give a name to your new resource group.
$StorageAccountName = "azsblobcontainer" # Give a name to your new storage account. It must be lowercase.
$Location = "Local" # Choose "Local" as an example.
$ContainerName = "photo" # Give a name to your new container.
$ImageToUpload = "C:\temp\Hello.jpg" # Prepare an image file and a source directory in your local computer.
$DestinationFolder = "C:\temp\download" # A destination directory in your local computer.

# Import the Connect PowerShell module"
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
Import-Module .\Connect\AzureStack.Connect.psm1

# Configure the PowerShell environment
# Register an Az environment that targets your Azure Stack Hub instance
Add-AzEnvironment -Name $ARMEvnName -ARMEndpoint $ARMEndPoint 

# Login
$TenantID = Get-AzsDirectoryTenantId -AADTenantName $AADTenantName -EnvironmentName $ARMEvnName
Connect-AzAccount -EnvironmentName $ARMEvnName -TenantId $TenantID 

# Set a default Azure subscription.
Select-AzSubscription -SubscriptionName $SubscriptionName

# Create a new Resource Group 
New-AzResourceGroup -Name $ResourceGroupName -Location $Location

# Create a new storage account.
New-AzStorageAccount -ResourceGroupName $ResourceGroupName -Name $StorageAccountName -Location $Location -Type Standard_LRS

# Set a default storage account.
Set-AzCurrentStorageAccount -StorageAccountName $StorageAccountName -ResourceGroupName $ResourceGroupName 

# Create a new container.
New-AzStorageContainer -Name $ContainerName -Permission Off

# Upload a blob into a container.
Set-AzStorageBlobContent -Container $ContainerName -File $ImageToUpload

# List all blobs in a container.
Get-AzStorageBlob -Container $ContainerName

# Download blobs from the container:
# Get a reference to a list of all blobs in a container.
$blobs = Get-AzStorageBlob -Container $ContainerName

# Create the destination directory.
New-Item -Path $DestinationFolder -ItemType Directory -Force  

# Download blobs into the local destination directory.
$blobs | Get-AzureStorageBlobContent -Destination $DestinationFolder

# end
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm1)

このサンプルでは、[PowerShell for Azure Stack Hub のインストール](../operator/azure-stack-powershell-install.md)が正常に完了していることを前提としています。 このスクリプトは、構成を完了し、ローカルの PowerShell 環境にアカウントを追加するために Azure Stack Hub テナントの資格情報を要求するために役立ちます。 次に、スクリプトは、既定の Azure サブスクリプションを設定して、Azure 内に新しいストレージ アカウントを作成し、この新しいストレージ アカウントに新しいコンテナーを作成して、既存の画像ファイル (BLOB) をこのコンテナーにアップロードします。 このスクリプトにより、コンテナー内のすべての BLOB がリストされると、ローカル コンピューターに新しい格納先ディレクトリが作成され、画像ファイルがダウンロードされます。

1. [Azure Stack Hub と互換性のある Azure PowerShell モジュール](../operator/azure-stack-powershell-install.md)をインストールします。
2. [Azure Stack Hub の操作に必要なツール](../operator/azure-stack-powershell-download.md)をダウンロードします。
3. **Windows PowerShell ISE** を開いて、**管理者として実行し**、 **[ファイル]**  >  **[新規作成]** の順にクリックして、新しいスクリプト ファイルを作成します。
4. 次のスクリプトをコピーして新しいスクリプト ファイルに貼り付けます。
5. 構成設定に基づいてスクリプト変数を更新します。
   > [!NOTE]
   > このスクリプトは、**AzureStack_Tools** のルート ディレクトリで実行する必要があります。

```powershell  
# begin

$ARMEvnName = "AzureStackUser" # set AzureStackUser as your Azure Stack Hub environment name
$ARMEndPoint = "https://management.local.azurestack.external" 
$GraphAudience = "https://graph.windows.net/" 
$AADTenantName = "<myDirectoryTenantName>.onmicrosoft.com" 

$SubscriptionName = "basic" # Update with the name of your subscription.
$ResourceGroupName = "myTestRG" # Give a name to your new resource group.
$StorageAccountName = "azsblobcontainer" # Give a name to your new storage account. It must be lowercase.
$Location = "Local" # Choose "Local" as an example.
$ContainerName = "photo" # Give a name to your new container.
$ImageToUpload = "C:\temp\Hello.jpg" # Prepare an image file and a source directory in your local computer.
$DestinationFolder = "C:\temp\download" # A destination directory in your local computer.

# Import the Connect PowerShell module"
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
Import-Module .\Connect\AzureStack.Connect.psm1

# Configure the PowerShell environment
# Register an AzureRM environment that targets your Azure Stack Hub instance
Add-AzureRMEnvironment -Name $ARMEvnName -ARMEndpoint $ARMEndPoint 

# Login
$TenantID = Get-AzsDirectoryTenantId -AADTenantName $AADTenantName -EnvironmentName $ARMEvnName
Add-AzureRMAccount -EnvironmentName $ARMEvnName -TenantId $TenantID 

# Set a default Azure subscription.
Select-AzureRMSubscription -SubscriptionName $SubscriptionName

# Create a new Resource Group 
New-AzureRMResourceGroup -Name $ResourceGroupName -Location $Location

# Create a new storage account.
New-AzureRMStorageAccount -ResourceGroupName $ResourceGroupName -Name $StorageAccountName -Location $Location -Type Standard_LRS

# Set a default storage account.
Set-AzureRMCurrentStorageAccount -StorageAccountName $StorageAccountName -ResourceGroupName $ResourceGroupName 

# Create a new container.
New-AzureContainer -Name $ContainerName -Permission Off

# Upload a blob into a container.
Set-AzureBlobContent -Container $ContainerName -File $ImageToUpload

# List all blobs in a container.
Get-AzureBlob -Container $ContainerName

# Download blobs from the container:
# Get a reference to a list of all blobs in a container.
$blobs = Get-AzureStorageBlob -Container $ContainerName

# Create the destination directory.
New-Item -Path $DestinationFolder -ItemType Directory -Force  

# Download blobs into the local destination directory.
$blobs | Get-AzureStorageBlobContent -Destination $DestinationFolder

# end
```

---



### <a name="powershell-known-issues"></a>PowerShell の既知の問題

Azure Stack Hub の現在の互換性のある Azure PowerShell モジュールのバージョンは、ユーザー操作の場合は 1.2.11 です。 これは Azure PowerShell の最新バージョンとは異なります。 この違いは、ストレージ サービスの操作に次のように影響します。

バージョン 1.2.11 では `Get-AzStorageAccountKey` の戻り値の形式に `Key1` と `Key2` という 2 つのプロパティがありますが、Azure 最新バージョンは、すべてのアカウント キーが含まれる配列を返します。

```powershell
# This command gets a specific key for a storage account, 
# and works for Azure PowerShell version 1.4, and later versions.
(Get-AzStorageAccountKey -ResourceGroupName "RG01" `
-AccountName "MyStorageAccount").Value[0]

# This command gets a specific key for a storage account, 
# and works for Azure PowerShell version 1.3.2, and previous versions.
(Get-AzStorageAccountKey -ResourceGroupName "RG01" `
-AccountName "MyStorageAccount").Key1
```

詳細については、[Get-AzureRMStorageAccountKey](/powershell/module/Az.storage/Get-AzStorageAccountKey) を参照してください。

## <a name="azure-cli"></a>Azure CLI

Azure CLI は、Azure リソースを管理するための、Azure のコマンド ライン エクスペリエンスです。 macOS、Linux、および Windows 上にインストールし、コマンド ラインから実行できます。

Azure CLI は、コマンド ラインから Azure リソースを管理したり、Azure Resource Manager を操作対象とする自動化スクリプトを作成したりするために最適化されています。 豊富なデータ アクセスを含む、Azure Stack Hub ポータルにあるものと同じ機能の多くを使用できます。

Azure Stack Hub には、Azure CLI バージョン 2.0 以降が必要です。 Azure Stack Hub と Azure CLI のインストールと構成の詳細については、[Azure Stack Hub CLI のインストールと構成](azure-stack-version-profiles-azurecli2.md)に関する記事を参照してください。 Azure CLI を使用して Azure Stack Hub ストレージ アカウント内のリソースを操作するいくつかのタスクを実行する方法の詳細については、「[Azure ストレージでの Azure CLI の使用](/azure/storage/storage-azure-cli)」を参照してください。

### <a name="azure-cli-sample-script-for-azure-stack-hub"></a>Azure Stack Hub 用の Azure CLI サンプル スクリプト

CLI のインストールと構成が完了したら、次の手順を試し、Azure Stack Hub ストレージ リソースと対話する小さなシェル サンプル スクリプトを操作することができます。 スクリプトを実行すると、次のアクションが行われます。

* ストレージ アカウントに新しいコンテナーを作成する。
* 既存のファイルを (BLOB として) コンテナーにアップロードする。
* コンテナー内のすべての BLOB を一覧表示する。
* 指定したローカル コンピューター上の宛先にファイルをダウンロードする。

このスクリプトを実行する前に、ターゲットの Azure Stack Hub に正常に接続およびサインインできるようにしておきます。

1. 好みのテキスト エディターを開き、前述のスクリプトをコピーしてエディターに貼り付けます。
2. 構成の設定を反映するようにスクリプトの変数を更新します。
3. 必要な変数を更新したら、スクリプトを保存してエディターを終了します。 スクリプト名を **my_storage_sample.sh** に指定したと仮定して、次のステップに移ります。
4. 必要に応じてスクリプトを実行可能ファイルとしてマークします。`chmod +x my_storage_sample.sh`
5. スクリプトを実行します。 たとえば Bash の場合は次のようになります。 `./my_storage_sample.sh`

```azurecli
#!/bin/bash
# A simple Azure Stack Hub storage example script

export AZURESTACK_RESOURCE_GROUP=<resource_group_name>
export AZURESTACK_RG_LOCATION="local"
export AZURESTACK_STORAGE_ACCOUNT_NAME=<storage_account_name>
export AZURESTACK_STORAGE_CONTAINER_NAME=<container_name>
export AZURESTACK_STORAGE_BLOB_NAME=<blob_name>
export FILE_TO_UPLOAD=<file_to_upload>
export DESTINATION_FILE=<destination_file>

echo "Creating the resource group..."
az group create --name $AZURESTACK_RESOURCE_GROUP --location $AZURESTACK_RG_LOCATION

echo "Creating the storage account..."
az storage account create --name $AZURESTACK_STORAGE_ACCOUNT_NAME --resource-group $AZURESTACK_RESOURCE_GROUP --account-type Standard_LRS

echo "Creating the blob container..."
az storage container create --name $AZURESTACK_STORAGE_CONTAINER_NAME --account-name $AZURESTACK_STORAGE_ACCOUNT_NAME

echo "Uploading the file..."
az storage blob upload --container-name $AZURESTACK_STORAGE_CONTAINER_NAME --file $FILE_TO_UPLOAD --name $AZURESTACK_STORAGE_BLOB_NAME --account-name $AZURESTACK_STORAGE_ACCOUNT_NAME

echo "Listing the blobs..."
az storage blob list --container-name $AZURESTACK_STORAGE_CONTAINER_NAME --account-name $AZURESTACK_STORAGE_ACCOUNT_NAME --output table

echo "Downloading the file..."
az storage blob download --container-name $AZURESTACK_STORAGE_CONTAINER_NAME --account-name $AZURESTACK_STORAGE_ACCOUNT_NAME --name $AZURESTACK_STORAGE_BLOB_NAME --file $DESTINATION_FILE --output table

echo "Done"
```

## <a name="microsoft-azure-storage-explorer"></a>Microsoft Azure Storage Explorer

Azure Storage Explorer は、Microsoft のスタンドアロン アプリです。 これを使用すると、Windows、macOS、Linux のコンピューターで Azure Storage と Azure Stack Hub Storage 両方のデータを簡単に操作できます。 簡単に、Azure Stack Hub Storage データを簡単に管理する方法が必要な場合は、Microsoft Azure Storage Explorer の使用を検討してください。

* Azure Stack Hub を操作する Azure Storage Explorer の構成の詳細については、[Azure Stack サブスクリプションへの Storage Explorer の接続](azure-stack-storage-connect-se.md)に関する記事を参照してください。
* Microsoft Azure Storage Explorer について詳しくは、「[Storage Explorer の概要](/azure/vs-azure-tools-storage-manage-with-storage-explorer)」をご覧ください

## <a name="blobfuse"></a>blobfuse 

[blobfuse](https://github.com/Azure/azure-storage-fuse) は、ストレージ アカウント内の既存のブロック BLOB データに Linux ファイル システム経由でアクセスできるようにする、Azure Blob Storage 用の仮想ファイル システム ドライバーです。 Azure Blob Storage はオブジェクト ストレージ サービスであるため、階層型名前空間を持っていません。 blobfuse は、フォワードスラッシュ `/` を区切り記号として使用して、仮想ディレクトリ スキームによってこの名前空間を提供します。 blobfuse は、Azure と Azure Stack Hub の両方で動作します。 

Linux 上で blobfuse を使用して Blob Storage をファイル システムとしてマウントする方法の詳細については、「[blobfuse を使用して Blob Storage をファイル システムとしてマウントする方法](/azure/storage/blobs/storage-how-to-mount-container-linux)」を参照してください。 

Azure Stack Hub の場合、ストレージ アカウントの資格情報を構成する際に、accountName、accountKey/sasToken、および containerName と共に *blobEndpoint* を指定する必要があります。

Azure Stack Development Kit (ASDK) では、*blobEndpoint* を `myaccount.blob.local.azurestack.external` にする必要があります。 Azure Stack Hub 統合システムでは、エンドポイントが不明の場合は、クラウド管理者に問い合わせてください。

*accountKey* と *sasToken* は一度に 1 つずつしか構成できません。 ストレージ アカウント キーを指定した資格情報構成ファイルの形式を次に示します。

```
accountName myaccount 
accountKey myaccesskey== 
containerName mycontainer 
blobEndpoint myaccount.blob.local.azurestack.external
```

共有アクセス トークンを指定した資格情報構成ファイルの形式を次に示します。

```  
accountName myaccount 
sasToken ?mysastoken 
containerName mycontainer 
blobEndpoint myaccount.blob.local.azurestack.external
```

## <a name="next-steps"></a>次のステップ

* [Azure Stack Hub サブスクリプションに Microsoft Azure Storage Explorer を接続する](azure-stack-storage-connect-se.md)
* [Storage Explorer の概要](/azure/vs-azure-tools-storage-manage-with-storage-explorer)
* [Azure 互換ストレージ: 違いと考慮事項](azure-stack-acs-differences.md)
* [Microsoft Azure ストレージの概要](/azure/storage/common/storage-introduction)
