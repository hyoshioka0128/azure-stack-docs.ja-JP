---
title: Azure Stack Hub ストレージの相違点と考慮事項
titleSuffix: Azure Stack Hub
description: Azure Stack Hub のデプロイに関する考慮事項と併せて、Azure Stack Hub ストレージと Azure ストレージの相違点について理解します。
author: mattbriggs
ms.topic: conceptual
ms.date: 5/27/2020
ms.author: mabrigg
ms.reviwer: jiahan
ms.lastreviewed: 08/12/2020
ms.openlocfilehash: 3f4ed6bd83414d158ef80996cadf024653956d38
ms.sourcegitcommit: 7d518629bd55f24e7459404bb19b7db8a54f4b94
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/12/2020
ms.locfileid: "88145388"
---
# <a name="azure-stack-hub-storage-differences-and-considerations"></a>Azure Stack Hub のストレージ: 相違点と考慮事項

Azure Stack Hub ストレージは、Microsoft Azure Stack Hub 内のストレージ クラウド サービスのセットです。 Azure Stack Hub ストレージでは、BLOB、テーブル、キュー、アカウント管理機能に Azure と整合性のあるセマンティクスを提供します。

この記事では、Azure Stack Hub ストレージと Azure Storage サービス間における既知の違いをまとめています。 Azure Stack Hub をデプロイするときに考慮すべき事柄も一覧しています。 グローバル Azure と Azure Stack Hub との違いの概要については、[主な考慮事項](azure-stack-considerations.md)に関する記事をご覧ください。

## <a name="cheat-sheet-storage-differences"></a>チート シート:ストレージの相違点

| 特徴量 | Azure (グローバル) | Azure Stack Hub |
| --- | --- | --- |
|File Storage|クラウド ベースの SMB ファイル共有がサポートされています。 | まだサポートされていません。
|保存データに対する Azure Storage サービスの暗号化|256 ビット AES 暗号化。 Key Vault でユーザーが管理するキーを使用した暗号化をサポートしています。|BitLocker 128 ビット AES 暗号化。 ユーザーが管理するキーを使用した暗号化はサポートされていません。
|ストレージ アカウントの種類|汎用 V1、V2、および Blob Storage アカウント。 |汎用 V1 のみ。
|レプリケーション オプション|ローカル冗長ストレージ、geo 冗長ストレージ、読み取りアクセス geo 冗長ストレージ、およびゾーン冗長ストレージ。 |ローカル冗長ストレージ。
|Premium Storage|高パフォーマンスで低待機時間のストレージを提供します。 Premium Storage アカウントのページ BLOB のみをサポートします。|プロビジョニング可能ですがパフォーマンス制限や保証がありません。 Premium Storage アカウントでは、ブロック BLOB を使用したブロック、BLOB、テーブル、およびキューの追加は行われません。
|マネージド ディスク|Premium および標準がサポートされます。 |使用バージョンが 1808 以降の場合にサポートされます。
|BLOB 名|1,024 文字 (2,048 バイト)。 |880 文字 (1,760 バイト)。
|ブロック BLOB の最大サイズ|4.75 TB (100 MB X 50,000 ブロック)。 |1802 update 以降のバージョンでは、4.75 TB (100 MB x 50,000 ブロック)。 それより前のバージョンでは 50,000 X 4 MB (約 195 GB)。
|ページ BLOB のスナップショット コピー|実行中の VM にアタッチされている Azure の管理対象外 VM ディスクのバックアップはサポートされています。 |まだサポートされていません。
|ページ BLOB の増分スナップショットのコピー|Premium および標準の Azure ページ BLOB がサポートされます。 |まだサポートされていません。
|ページ BLOB の課金|一意のページに対する料金が発生します。そのページが BLOB とスナップショットのどちらに含まれているかは関係ありません。 BLOB に関連付けられているスナップショットについて追加料金が発生するのは、ベース BLOB が更新されたときです。|ベース BLOB と、関連付けられているスナップショットに対する料金が発生します。 個々のスナップショットごとに追加料金が発生します。
|Blob Storage のストレージ層|ホット ストレージ層、クール ストレージ層、アーカイブ ストレージ層。|まだサポートされていません。
|Blob Storage の論理的な削除|一般提供。 |まだサポートされていません。
|ページ BLOB の最大サイズ|8 TB。 |1 TB。 
|ページ BLOB のページ サイズ|512 バイト。 |4 KB。 
|テーブルのパーティション キーと行キーのサイズ|1,024 文字 (2,048 バイト)。|400 文字 (800 バイト)。
|BLOB スナップショット|1 つの BLOB の最大スナップショット数は制限されていません。|1 つの BLOB の最大スナップショット数は 1,000 です。
|ストレージの Azure AD Authentication|プレビュー段階です。 |まだサポートされていません。
|不変 BLOB|一般提供。 |まだサポートされていません。
|ストレージのファイアウォールおよび仮想ネットワーク規則|一般提供。 |まだサポートされていません。|

ストレージ メトリックにも相違点があります。

* ストレージ メトリックのトランザクション データでは、内部と外部のネットワーク帯域幅が区別されません。
* ストレージ メトリックのトランザクション データには、マウントされたディスクへの仮想マシンのアクセスは含まれません。

## <a name="api-version"></a>API バージョン

Azure Stack Hub ストレージでは、以下のバージョンがサポートされます。

Azure Storage サービスの API:

更新プログラム 2005 以降のバージョン:

- [2019-02-02](/rest/api/storageservices/version-2019-02-02)
- [2018-11-09](/rest/api/storageservices/version-2018-11-09)
- [2018-03-28](/rest/api/storageservices/version-2018-03-28)
- [2017-11-09](/rest/api/storageservices/version-2017-11-09)
- [2017-07-29](/rest/api/storageservices/version-2017-07-29)
- [2017-04-17](/rest/api/storageservices/version-2017-04-17)
- [2016-05-31](/rest/api/storageservices/version-2016-05-31)
- [2015-12-11](/rest/api/storageservices/version-2015-12-11)
- [2015-07-08](/rest/api/storageservices/version-2015-07-08)
- [2015-04-05](/rest/api/storageservices/version-2015-04-05)

以前のバージョン:

- [2017-11-09](/rest/api/storageservices/version-2017-11-09)
- [2017-07-29](/rest/api/storageservices/version-2017-07-29)
- [2017-04-17](/rest/api/storageservices/version-2017-04-17)
- [2016-05-31](/rest/api/storageservices/version-2016-05-31)
- [2015-12-11](/rest/api/storageservices/version-2015-12-11)
- [2015-07-08](/rest/api/storageservices/version-2015-07-08)
- [2015-04-05](/rest/api/storageservices/version-2015-04-05)

Azure Storage サービスの Management API:

1811 update 以降のバージョン:

- [2017-10-01](/rest/api/storagerp/)
- [2017-06-01](/rest/api/storagerp/)
- [2016-12-01](/rest/api/storagerp/)
- [2016-05-01](/rest/api/storagerp/)
- [2016-01-01](/rest/api/storagerp/)
- [2015-06-15](/rest/api/storagerp/)
- [2015-05-01-preview](/rest/api/storagerp/)

以前のバージョン:

- [2016-01-01](/rest/api/storagerp/)
- [2015-06-15](/rest/api/storagerp/)
- [2015-05-01-preview](/rest/api/storagerp/)

## <a name="powershell-version"></a>PowerShell バージョン

ストレージ モジュールの PowerShell については、REST API と互換性のあるバージョンを把握しておいてください。

| Module | サポートされているバージョン | 使用法 |
|---|---|---|
| Azure.Storage | [4.5.0](https://www.powershellgallery.com/packages/Azure.Storage/4.5.0) | Azure Stack Hub ストレージ アカウントの BLOB、キュー、テーブルを管理します。 |
| AzureRM.Storage | [5.0.4](https://www.powershellgallery.com/packages/AzureRM.Storage/5.0.4) | Azure Stack Hub でストレージ アカウントを作成および管理します。 |

Azure Stack Hub でサポートされるストレージ クライアント ライブラリの詳細については、「[Azure Stack Hub ストレージの開発ツールの概要](azure-stack-storage-dev.md)」を参照してください。

## <a name="next-steps"></a>次のステップ

* [Azure Stack Hub ストレージの開発ツールの概要](azure-stack-storage-dev.md)
* [Azure Stack Hub ストレージでのデータ転送ツールの使用](azure-stack-storage-transfer.md)
* [Azure Stack Hub ストレージの概要](azure-stack-storage-overview.md)
