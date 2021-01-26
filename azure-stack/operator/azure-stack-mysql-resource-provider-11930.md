---
title: Azure Stack Hub MySQL リソース プロバイダー 1.1.93.0 のリリース ノート
description: このリリース ノートでは、Azure Stack Hub MySQL リソース プロバイダー 1.1.93.0 の更新プログラムの新機能を紹介しています。
author: caoyang
ms.topic: article
ms.date: 09/22/2020
ms.author: caoyang
ms.reviewer: xiaofmao
ms.lastreviewed: 09/22/2020
ms.openlocfilehash: 225c765642dc48b299c899d71eb0d0ce13d4852d
ms.sourcegitcommit: 0983c1f90734b7ea5e23ae614eeaed38f9cb3c9a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/19/2021
ms.locfileid: "98571367"
---
# <a name="mysql-resource-provider-1193x-release-notes"></a>MySQL リソース プロバイダー 1.1.93.x のリリース ノート

これらのリリース ノートでは、MySQL リソース プロバイダー バージョン 1.1.93.x の機能強化と既知の問題について説明します。

## <a name="build-reference"></a>ビルドのリファレンス
MySQL リソース プロバイダー バイナリをダウンロードした後、自己展開ツールを実行してコンテンツを一時ディレクトリに展開します。 リソース プロバイダーには、対応する最低限の Azure Stack Hub のビルドがあります。 MySQL リソース プロバイダーのこのバージョンをインストールするために必要な Azure Stack Hub の最小リリース バージョンを次に示します。

> |サポートされる Azure Stack Hub のバージョン|MySQL リソース プロバイダーのバージョン|
> |-----|-----|
> |バージョン 2008、2005|[MySQL RP バージョン 1.1.93.1](https://aka.ms/azshmysqlrp11931)|  
> |     |     |

> [!IMPORTANT]
> SQL リソース プロバイダーの最新バージョンをデプロイする前に、サポートされている最低限の Azure Stack Hub 更新プログラムを Azure Stack Hub 統合システムに適用します。

## <a name="new-features-and-fixes"></a>新機能と修正

このバージョンの Azure Stack Hub MySQL リソース プロバイダーには、次の機能強化と修正プログラムが含まれています。

- **ベース VM を特殊化された Windows Server に更新します。** この Windows Server バージョンは Azure Stack Hub Add-on RP インフラストラクチャ専用であり、テナント マーケットプレースには表示されません。 このバージョンの MySQL リソース プロバイダーのデプロイまたはこのバージョンへのアップグレードを行う前に、確実に **Microsoft AzureStack Add-On RP Windows Server** イメージをダウンロードしてください。
- **孤立したデータベース メタデータとホスティング サーバー メタデータの削除をサポートします。** ホスティング サーバーが接続できなくなった場合、テナントには、孤立したデータベース メタデータをポータルから削除するオプションがあります。 ホスティング サーバーにリンクされている孤立したデータベース メタデータが存在しない場合、オペレーターは孤立したホスティング サーバー メタデータを管理ポータルから削除できます。
- **シークレット ローテーションの実行時、KeyVaultPfxPassword を省略可能な引数にします。** 詳細については、[こちらのドキュメント](azure-stack-sql-resource-provider-maintain.md#secrets-rotation)を確認してください。
- **その他のバグの修正。**

Azure Stack Hub が 2005 リリースにアップグレードされた後に MySQL リソース プロバイダー 1.1.93.1 を適用することをお勧めします。

## <a name="known-issues"></a>既知の問題
不適切な AzureRmContext を使用すると、1.1.93.0 バージョンのデプロイが失敗する場合があります。 1\.1.93.1 バージョンに直接アップグレードすることをお勧めします。 1\.1.93.0 へのアップグレードが既に正常に完了している場合は、1.1.93.1 バージョンをスキップしても問題ありません。

MySQL リソース プロバイダーを再デプロイするときに同じバージョンが既にデプロイされている (たとえば、MySQL リソース プロバイダー 1.1.93.1 が既にデプロイされているときに同じバージョンが再度デプロイされる) と、MySQL リソース プロバイダーをホストしている VM が停止されます。 この問題を解決するには、管理者ポータルに移動し、system.\<region\>.mysqladapter というリソース グループで mysqlvm\<version\> という名前の VM を見つけて再起動します。

## <a name="next-steps"></a>次のステップ

- [MySQL リソース プロバイダーの詳細を確認します](azure-stack-mysql-resource-provider.md)。
- [MySQL リソースプロバイダーのデプロイを準備します](azure-stack-mysql-resource-provider-deploy.md#prerequisites)。
- [MySQL リソース プロバイダーを以前のバージョンからアップグレードします](azure-stack-mysql-resource-provider-update.md)。
