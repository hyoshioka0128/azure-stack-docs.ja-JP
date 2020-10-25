---
title: Azure Stack Hub MySQL リソース プロバイダー 1.1.93.0 のリリース ノート
description: このリリース ノートでは、Azure Stack Hub MySQL リソース プロバイダー 1.1.93.0 の更新プログラムの新機能を紹介しています。
author: caoyang
ms.topic: article
ms.date: 09/22/2020
ms.author: caoyang
ms.reviewer: xiaofmao
ms.lastreviewed: 09/22/2020
ms.openlocfilehash: f298e2e9e55d8057a63264fb347069ef07204a16
ms.sourcegitcommit: 69cfff119ab425d0fbb71e38d1480d051fc91216
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/30/2020
ms.locfileid: "91585981"
---
# <a name="mysql-resource-provider-11930-release-notes"></a>MySQL リソース プロバイダー 1.1.93.0 のリリース ノート

これらのリリース ノートでは、MySQL リソース プロバイダー バージョン 1.1.93.0 の機能強化と既知の問題について説明します。

## <a name="build-reference"></a>ビルドのリファレンス
MySQL リソース プロバイダー バイナリをダウンロードした後、自己展開ツールを実行してコンテンツを一時ディレクトリに展開します。 リソース プロバイダーには、対応する最低限の Azure Stack Hub のビルドがあります。 MySQL リソース プロバイダーのこのバージョンをインストールするために必要な Azure Stack Hub の最小リリース バージョンを次に示します。

> |サポートされる Azure Stack Hub のバージョン|MySQL リソース プロバイダーのバージョン|
> |-----|-----|
> |バージョン 2005|[MySQL RP バージョン 1.1.93.0](https://aka.ms/azshmysqlrp11930)|  
> |     |     |

> [!IMPORTANT]
> MySQL リソース プロバイダーの最新バージョンをデプロイする前に、サポートされている最低限の Azure Stack Hub 更新プログラムを Azure Stack Hub 統合システムに適用します。

## <a name="new-features-and-fixes"></a>新機能と修正

このバージョンの Azure Stack Hub MySQL リソース プロバイダーには、次の機能強化と修正プログラムが含まれています。

- **ベース VM を特殊化された Windows Server に更新します。** この Windows Server バージョンは Azure Stack Hub Add-on RP インフラストラクチャ専用であり、テナント マーケットプレースには表示されません。 このバージョンの MySQL リソース プロバイダーのデプロイまたはこのバージョンへのアップグレードを行う前に、確実に **Microsoft AzureStack Add-On RP Windows Server INTERNAL ONLY** イメージをダウンロードしてください。
- **孤立したデータベース メタデータとホスティング サーバー メタデータの削除をサポートします。** ホスティング サーバーが接続できなくなった場合、テナントには、孤立したデータベース メタデータをポータルから削除するオプションがあります。 ホスティング サーバーにリンクされている孤立したデータベース メタデータが存在しない場合、オペレーターは孤立したホスティング サーバー メタデータを管理ポータルから削除できます。
- **シークレット ローテーションの実行時、KeyVaultPfxPassword を省略可能な引数にします。** 詳細については、[こちらのドキュメント](azure-stack-sql-resource-provider-maintain.md#secrets-rotation)を確認してください。
- **その他のバグの修正。**

Azure Stack Hub が 2005 リリースにアップグレードされた後に MySQL リソース プロバイダー 1.1.93.0 を適用することをお勧めします。

## <a name="known-issues"></a>既知の問題
[なし] :

## <a name="next-steps"></a>次のステップ

- [MySQL リソース プロバイダーの詳細を確認します](azure-stack-mysql-resource-provider.md)。
- [MySQL リソースプロバイダーのデプロイを準備します](azure-stack-mysql-resource-provider-deploy.md#prerequisites)。
- [MySQL リソース プロバイダーを以前のバージョンからアップグレードします](azure-stack-mysql-resource-provider-update.md)。
