---
title: Azure Stack Hub ストレージの開発ツールの概要
description: Azure Stack Hub ストレージの開発ツールの使用方法の概要
author: mattbriggs
ms.author: mabrigg
ms.date: 5/27/2020
ms.topic: conceptual
ms.reviewer: jiahan
ms.lastreviewed: 08/12/2020
ms.openlocfilehash: 28b1d8ade7b56a767d436b918ad7d386ef4759bd
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90574144"
---
# <a name="get-started-with-azure-stack-hub-storage-development-tools"></a>Azure Stack Hub ストレージの開発ツールの概要

Microsoft Azure Stack Hub には、Blob Storage、Table Storage、Queue Storage など一連のストレージ サービスがあります。

この記事は、Azure Stack Hub ストレージの開発ツールを初めて使うときのガイドとしてご利用ください。 より詳細な情報とサンプル コードについては、該当する Azure ストレージのチュートリアルでご覧いただけます。

> [!NOTE]  
> Azure Stack Hub ストレージと Azure ストレージには、プラットフォームごとの固有の要件などの違いがあります。 たとえば、Azure Stack Hub には、クライアント ライブラリとエンドポイント サフィックスの特定の要件があります。 詳しくは、「[Azure Stack Hub ストレージ:違いと考慮事項](azure-stack-acs-differences.md)」をご覧ください。

## <a name="azure-client-libraries"></a>Azure のクライアント ライブラリ

したがって、ストレージのクライアント ライブラリについて、REST API と互換性のあるバージョンを把握しておいてください。 また、コード内で Azure Stack Hub エンドポイントも指定する必要があります。


::: moniker range=">=azs-2005"
### <a name="2005-update-or-newer-versions"></a>更新プログラム 2005 以降のバージョン

| クライアント ライブラリ | Azure Stack Hub でサポートされるバージョン | Link | エンドポイントの指定 |
|----------------|-------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------|
| .NET | 11.0.0 | NuGet パッケージ:<br>共通: <https://www.nuget.org/packages/Microsoft.Azure.Storage.Common/11.0.0><br>BLOB: <https://www.nuget.org/packages/Microsoft.Azure.Storage.Blob/11.0.0><br>キュー:<br><https://www.nuget.org/packages/Microsoft.Azure.Storage.Queue/11.0.0><br> <br>GitHub リリース:<br><https://github.com/Azure/azure-storage-net/releases/tag/v11.0.0> | app.config ファイル |
| Java | 12.0.0-preview.3 | Maven パッケージ:<br><https://mvnrepository.com/artifact/com.azure/azure-storage-file/12.0.0-preview.3><br> <br>GitHub リリース:<br><https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage> | 接続文字列の設定 |
| Node.js | 2.8.3 | NPM リンク:<br><https://www.npmjs.com/package/azure-storage><br>(次のコマンドを実行: `npm install azure-storage@2.8.3`)<br> <br>GitHub リリース:<br><https://github.com/Azure/azure-storage-node/releases/tag/v2.8.3> | サービス インスタンスの宣言 |
| C++ | 7.1.0 | GitHub リリース:<br><https://github.com/Azure/azure-storage-cpp/releases/tag/v7.1.0> | 接続文字列の設定 |
| PHP | 1.2.0 | GitHub リリース:<br>共通: <https://github.com/Azure/azure-storage-php/releases/tag/v1.2.0-common><br>BLOB: <https://github.com/Azure/azure-storage-php/releases/tag/v1.2.0-blob><br>キュー:<br><https://github.com/Azure/azure-storage-php/releases/tag/v1.1.1-queue><br>テーブル: <https://github.com/Azure/azure-storage-php/releases/tag/v1.1.0-table><br> <br>Composer 経由でインストールする (詳細については[下記参照](#install-php-client-via-composer---current)) | 接続文字列の設定 |
| Python | 2.1.0 | GitHub リリース:<br>共通:<br><https://github.com/Azure/azure-storage-python/releases/tag/v2.1.0-common><br>BLOB:<br><https://github.com/Azure/azure-storage-python/releases/tag/v2.1.0-blob><br>キュー:<br><https://github.com/Azure/azure-storage-python/releases/tag/v2.1.0-queue> | サービス インスタンスの宣言 |
| Ruby | 1.0.1 | RubyGems パッケージ:<br>共通:<br><https://rubygems.org/gems/azure-storage-common/versions/1.0.1><br>BLOB: <https://rubygems.org/gems/azure-storage-blob/versions/1.0.1><br>キュー: <https://rubygems.org/gems/azure-storage-queue/versions/1.0.1><br>テーブル: <https://rubygems.org/gems/azure-storage-table/versions/1.0.1><br> <br>GitHub リリース:<br>共通: <https://github.com/Azure/azure-storage-ruby/releases/tag/v1.0.1-common><br>BLOB: <https://github.com/Azure/azure-storage-ruby/releases/tag/v1.0.1-blob><br>キュー: <https://github.com/Azure/azure-storage-ruby/releases/tag/v1.0.1-queue><br>テーブル: <https://github.com/Azure/azure-storage-ruby/releases/tag/v1.0.1-table> | 接続文字列の設定 |

#### <a name="install-php-client-via-composer---current"></a>Composer 経由で PHP クライアントをインストールする (現在)

Composer 経由でインストールするには (BLOB の例):

1. プロジェクトのルートに **composer.json** という名前のファイルを次のコードを使用して作成します。

    ```json
    {
      "require": {
      "Microsoft/azure-storage-blob":"1.2.0"
      }
    }
    ```

2. [composer.phar](https://getcomposer.org/composer.phar) をプロジェクトのルートにダウンロードします。
3. `php composer.phar install` を実行します。
::: moniker-end

::: moniker range=">=azs-1811 <=azs-2005"
### <a name="previous-versions-1811-to-2002-update"></a>以前のバージョン (1811 から 2002 までの更新プログラム)

| クライアント ライブラリ | Azure Stack Hub でサポートされるバージョン | Link | エンドポイントの指定 |
|----------------|-------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------|
| .NET | 9.2.0 | NuGet パッケージ:<br><https://www.nuget.org/packages/WindowsAzure.Storage/9.2.0><br> <br>GitHub リリース:<br><https://github.com/Azure/azure-storage-net/releases/tag/v9.2.0> | app.config ファイル |
| Java | 7.0.0 | Maven パッケージ:<br><https://mvnrepository.com/artifact/com.microsoft.azure/azure-storage/7.0.0><br> <br>GitHub リリース:<br><https://github.com/Azure/azure-storage-java/releases/tag/v7.0.0> | 接続文字列の設定 |
| Node.js | 2.8.3 | NPM リンク:<br><https://www.npmjs.com/package/azure-storage><br>(次のコマンドを実行: `npm install azure-storage@2.8.3`)<br> <br>GitHub リリース:<br><https://github.com/Azure/azure-storage-node/releases/tag/v2.8.3> | サービス インスタンスの宣言 |
| C++ | 5.2.0 | NuGet パッケージ:<br><https://www.nuget.org/packages/Microsoft.Azure.Storage.CPP.v140/5.2.0><br> <br>GitHub リリース:<br><https://github.com/Azure/azure-storage-cpp/releases/tag/v5.2.0> | 接続文字列の設定 |
| PHP | 1.2.0 | GitHub リリース:<br>共通: <https://github.com/Azure/azure-storage-php/releases/tag/v1.2.0-common><br>BLOB: <https://github.com/Azure/azure-storage-php/releases/tag/v1.2.0-blob><br>キュー:<br><https://github.com/Azure/azure-storage-php/releases/tag/v1.1.1-queue><br>テーブル: <https://github.com/Azure/azure-storage-php/releases/tag/v1.1.0-table><br> <br>Composer 経由でインストールする (詳細については[下記参照](#install-php-client-via-composer---current)) | 接続文字列の設定 |
| Python | 1.1.0 | GitHub リリース:<br>共通:<br><https://github.com/Azure/azure-storage-python/releases/tag/v1.1.0-common><br>BLOB:<br><https://github.com/Azure/azure-storage-python/releases/tag/v1.1.0-blob><br>キュー:<br><https://github.com/Azure/azure-storage-python/releases/tag/v1.1.0-queue> | サービス インスタンスの宣言 |
| Ruby | 1.0.1 | RubyGems パッケージ:<br>共通:<br><https://rubygems.org/gems/azure-storage-common/versions/1.0.1><br>BLOB: <https://rubygems.org/gems/azure-storage-blob/versions/1.0.1><br>キュー: <https://rubygems.org/gems/azure-storage-queue/versions/1.0.1><br>テーブル: <https://rubygems.org/gems/azure-storage-table/versions/1.0.1><br> <br>GitHub リリース:<br>共通: <https://github.com/Azure/azure-storage-ruby/releases/tag/v1.0.1-common><br>BLOB: <https://github.com/Azure/azure-storage-ruby/releases/tag/v1.0.1-blob><br>キュー: <https://github.com/Azure/azure-storage-ruby/releases/tag/v1.0.1-queue><br>テーブル: <https://github.com/Azure/azure-storage-ruby/releases/tag/v1.0.1-table> | 接続文字列の設定 |

#### <a name="install-php-client-via-composer---current"></a>Composer 経由で PHP クライアントをインストールする (現在)

Composer 経由でインストールするには (BLOB の例):

1. プロジェクトのルートに **composer.json** という名前のファイルを次のコードを使用して作成します。

    ```json
    {
      "require": {
      "Microsoft/azure-storage-blob":"1.2.0"
      }
    }
    ```

2. [composer.phar](https://getcomposer.org/composer.phar) をプロジェクトのルートにダウンロードします。
3. `php composer.phar install` を実行します。
::: moniker-end

::: moniker range=">=azs-1802 <=azs-1809"
### <a name="previous-versions-1802-to-1809-update"></a>以前のバージョン (1802 から 1809 までの更新プログラム)

| クライアント ライブラリ | Azure Stack Hub でサポートされるバージョン | Link | エンドポイントの指定 |
|----------------|-------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------|
| .NET | 8.7.0 | NuGet パッケージ:<br><https://www.nuget.org/packages/WindowsAzure.Storage/8.7.0><br> <br>GitHub リリース:<br><https://github.com/Azure/azure-storage-net/releases/tag/v8.7.0> | app.config ファイル |
| Java | 6.1.0 | Maven パッケージ:<br><https://mvnrepository.com/artifact/com.microsoft.azure/azure-storage/6.1.0><br> <br>GitHub リリース:<br><https://github.com/Azure/azure-storage-java/releases/tag/v6.1.0> | 接続文字列の設定 |
| Node.js | 2.7.0 | NPM リンク:<br><https://www.npmjs.com/package/azure-storage><br>(次のコマンドを実行: `npm install azure-storage@2.7.0`)<br> <br>GitHub リリース:<br><https://github.com/Azure/azure-storage-node/releases/tag/v2.7.0> | サービス インスタンスの宣言 |
| C++ | 3.1.0 | NuGet パッケージ:<br><https://www.nuget.org/packages/wastorage.v140/3.1.0><br> <br>GitHub リリース:<br><https://github.com/Azure/azure-storage-cpp/releases/tag/v3.1.0> | 接続文字列の設定 |
| PHP | 1.0.0 | GitHub リリース:<br>共通: <https://github.com/Azure/azure-storage-php/releases/tag/v1.0.0-common><br>BLOB: <https://github.com/Azure/azure-storage-php/releases/tag/v1.0.0-blob><br>キュー:<br><https://github.com/Azure/azure-storage-php/releases/tag/v1.0.0-queue><br>テーブル: <https://github.com/Azure/azure-storage-php/releases/tag/v1.0.0-table><br> <br>Composer 経由でインストールする (下記参照) | 接続文字列の設定 |
| Python | 1.0.0 | GitHub リリース:<br>共通:<br><https://github.com/Azure/azure-storage-python/releases/tag/v1.0.0-common><br>BLOB:<br><https://github.com/Azure/azure-storage-python/releases/tag/v1.0.0-blob><br>キュー:<br><https://github.com/Azure/azure-storage-python/releases/tag/v1.0.0-queue> | サービス インスタンスの宣言 |
| Ruby | 1.0.1 | RubyGems パッケージ:<br>共通:<br><https://rubygems.org/gems/azure-storage-common/versions/1.0.1><br>BLOB: <https://rubygems.org/gems/azure-storage-blob/versions/1.0.1><br>キュー: <https://rubygems.org/gems/azure-storage-queue/versions/1.0.1><br>テーブル: <https://rubygems.org/gems/azure-storage-table/versions/1.0.1><br> <br>GitHub リリース:<br>共通: <https://github.com/Azure/azure-storage-ruby/releases/tag/v1.0.1-common><br>BLOB: <https://github.com/Azure/azure-storage-ruby/releases/tag/v1.0.1-blob><br>キュー: <https://github.com/Azure/azure-storage-ruby/releases/tag/v1.0.1-queue><br>テーブル: <https://github.com/Azure/azure-storage-ruby/releases/tag/v1.0.1-table> | 接続文字列の設定 |

#### <a name="install-php-client-via-composer---previous"></a>Composer 経由で PHP クライアントをインストールする (以前)

Composer 経由でインストールするには (BLOB の例):

1. プロジェクトのルートに **composer.json** という名前のファイルを次のコードを使用して作成します。

   ```json
    {
      "require": {
      "Microsoft/azure-storage-blob":"1.0.0"
      }
    }
   ```

2. [composer.phar](https://getcomposer.org/composer.phar) をプロジェクトのルートにダウンロードします。
3. `php composer.phar install` を実行します。
:::moniker-end

## <a name="endpoint-declaration"></a>エンドポイントの宣言

Azure Stack Hub のエンドポイントには、リージョン名と Azure Stack Hub ドメインの 2 つの部分があります。
Azure Stack Development Kit の既定のエンドポイントは **local.azurestack.external** です。
エンドポイントが不明の場合は、クラウド管理者に問い合わせてください。

## <a name="examples"></a>例

### <a name="net"></a>.NET

Azure Stack Hub のエンドポイントのサフィックスは、app.config ファイルに指定されています。

```xml
<add key="StorageConnectionString"
value="DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=mykey;
EndpointSuffix=local.azurestack.external;" />
```

### <a name="java"></a>Java

Azure Stack Hub のエンドポイントのサフィックスは、接続文字列の設定に指定されています。

```java
public static final String storageConnectionString =
    "DefaultEndpointsProtocol=http;" +
    "AccountName=your_storage_account;" +
    "AccountKey=your_storage_account_key;" +
    "EndpointSuffix=local.azurestack.external";
```

### <a name="nodejs"></a>Node.js

Azure Stack Hub のエンドポイントのサフィックスは、宣言のインスタンスに指定されています。

```nodejs
var blobSvc = azure.createBlobService('myaccount', 'mykey',
'myaccount.blob.local.azurestack.external');
```

### <a name="c"></a>C++

Azure Stack Hub のエンドポイントのサフィックスは、接続文字列の設定に指定されています。

```cpp
const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;
AccountName=your_storage_account;
AccountKey=your_storage_account_key;
EndpointSuffix=local.azurestack.external"));
```

### <a name="php"></a>PHP

Azure Stack Hub のエンドポイントのサフィックスは、接続文字列の設定に指定されています。

```php
$connectionString = 'BlobEndpoint=https://<storage account name>.blob.local.azurestack.external/;
QueueEndpoint=https:// <storage account name>.queue.local.azurestack.external/;
TableEndpoint=https:// <storage account name>.table.local.azurestack.external/;
AccountName=<storage account name>;AccountKey=<storage account key>'
```

### <a name="python"></a>Python

Azure Stack Hub のエンドポイントのサフィックスは、宣言のインスタンスに指定されています。

```python
block_blob_service = BlockBlobService(account_name='myaccount',
account_key='mykey',
endpoint_suffix='local.azurestack.external')
```

### <a name="ruby"></a>Ruby

Azure Stack Hub のエンドポイントのサフィックスは、接続文字列の設定に指定されています。

```ruby
set
AZURE_STORAGE_CONNECTION_STRING=DefaultEndpointsProtocol=https;
AccountName=myaccount;
AccountKey=mykey;
EndpointSuffix=local.azurestack.external
```

## <a name="blob-storage"></a>BLOB ストレージ

Azure Stack Hub に、次の Azure Blob ストレージのチュートリアルを応用できます。 Azure Stack Hub のエンドポイントのサフィックスの具体的な要件については、前の[例](#examples)のセクションを参照してください。

* [.NET を使用して Azure Blob Storage を使用する](/azure/storage/blobs/storage-dotnet-how-to-use-blobs)
* [Java から BLOB ストレージを使用する方法](/azure/storage/blobs/storage-java-how-to-use-blob-storage)
* [Node.js から BLOB ストレージを使用する方法](/azure/storage/blobs/storage-nodejs-how-to-use-blob-storage)
* [C++ から BLOB ストレージを使用する方法](/azure/storage/blobs/storage-c-plus-plus-how-to-use-blobs)
* [PHP から BLOB ストレージを使用する方法](/azure/storage/blobs/storage-php-how-to-use-blobs)
* [Python から Azure BLOB ストレージを使用する方法](/azure/storage/blobs/storage-python-how-to-use-blob-storage)
* [Ruby から BLOB ストレージを使用する方法](/azure/storage/blobs/storage-ruby-how-to-use-blob-storage)

## <a name="queue-storage"></a>ストレージ

Azure Stack Hub に、次の Azure Queue storage のチュートリアルを応用できます。 Azure Stack Hub のエンドポイントのサフィックスの具体的な要件については、前の[例](#examples)のセクションを参照してください。

* [.NET を使用して Azure Queue Storage を使用する](/azure/storage/queues/storage-dotnet-how-to-use-queues)
* [Java から Queue ストレージを使用する方法](/azure/storage/queues/storage-java-how-to-use-queue-storage)
* [Node.js から Queue ストレージを使用する方法](/azure/storage/queues/storage-nodejs-how-to-use-queues)
* [C++ から Queue ストレージを使用する方法](/azure/storage/queues/storage-c-plus-plus-how-to-use-queues)
* [PHP から Queue ストレージを使用する方法](/azure/storage/queues/storage-php-how-to-use-queues)
* [Python から Queue ストレージを使用する方法](/azure/storage/queues/storage-python-how-to-use-queue-storage)
* [Ruby から Queue ストレージを使用する方法](/azure/storage/queues/storage-ruby-how-to-use-queue-storage)

## <a name="table-storage"></a>テーブル ストレージ

Azure Stack Hub に、次の Azure Table storage のチュートリアルを応用できます Azure Stack Hub のエンドポイントのサフィックスの具体的な要件については、前の[例](#examples)のセクションを参照してください。

* [.NET を使用して Azure Table Storage を使用する](/azure/cosmos-db/table-storage-how-to-use-dotnet)
* [Java からテーブル ストレージを使用する方法](/azure/cosmos-db/table-storage-how-to-use-java)
* [Node.js から Azure Table Storage を使用する方法](/azure/cosmos-db/table-storage-how-to-use-nodejs)
* [C++ から Table ストレージを使用する方法](/azure/cosmos-db/table-storage-how-to-use-c-plus)
* [PHP から Table ストレージを使用する方法](/azure/cosmos-db/table-storage-how-to-use-php)
* [Python で Table Storage を使用する方法](/azure/cosmos-db/table-storage-how-to-use-python)
* [Ruby から Table ストレージを使用する方法](/azure/cosmos-db/table-storage-how-to-use-ruby)

## <a name="next-steps"></a>次のステップ

* [Microsoft Azure ストレージの概要](/azure/storage/common/storage-introduction)
