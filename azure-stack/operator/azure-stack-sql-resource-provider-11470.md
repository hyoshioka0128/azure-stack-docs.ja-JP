---
title: Azure Stack Hub SQL リソース プロバイダー 1.1.47.0 のリリース ノート
description: 新しい機能、修正プログラム、既知の問題など、Azure Stack Hub SQL リソース プロバイダー更新プログラムの最新情報をご覧ください。
author: PatAltimore
ms.topic: article
ms.date: 11/26/2019
ms.author: patricka
ms.reviewer: xiaofmao
ms.lastreviewed: 11/26/2019
ms.openlocfilehash: d95be92f4cab37c41e1802fdcb18969e3276f459
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97869221"
---
# <a name="sql-resource-provider-11470-release-notes"></a>SQL リソース プロバイダー 1.1.47.0 のリリース ノート

これらのリリース ノートでは、SQL リソース プロバイダー バージョン 1.1.47.0 の機能強化と既知の問題について説明します。

## <a name="build-reference"></a>ビルドのリファレンス

SQL リソース プロバイダー バイナリをダウンロードした後、自己展開ツールを実行してコンテンツを一時ディレクトリに抽出します。 リソース プロバイダーには、対応する最低限の Azure Stack Hub のビルドがあります。 SQL リソース プロバイダーのこのバージョンをインストールするために必要な Azure Stack Hub の最小リリース バージョンを次に示します。

> |Azure Stack Hub の最小バージョン|SQL リソース プロバイダーのバージョン|
> |-----|-----|
> |バージョン 1910 (1.1910.0.58)|[SQL RP バージョン 1.1.47.0](https://aka.ms/azurestacksqlrp11470)|  
> |     |     |

> [!IMPORTANT]
> SQL リソース プロバイダーの最新バージョンをデプロイする前に、サポートされている最低限の Azure Stack Hub 更新プログラムを Azure Stack Hub 統合システムに適用します。

## <a name="new-features-and-fixes"></a>新機能と修正

このバージョンの Azure Stack Hub SQL リソース プロバイダーは、1910 更新プログラムでの最新のポータル変更との互換性をリソース プロバイダーに与えるための修正プログラム リリースです。 新しい機能はありません。

最新の Azure Stack Hub API バージョン プロファイル 2019-03-01-hybrid と Azure Stack Hub PowerShell モジュール 1.8.0 もサポートされています。 このため、デプロイおよび更新時にモジュールの特定の履歴バージョンをインストールする必要はありません。

Azure Stack Hub が 1910 更新プログラムにアップグレードされた後、リソース プロバイダーの更新プロセスに従って、SQL リソース プロバイダーの修正プログラム 1.1.47.0 を適用してください。 SQL リソース プロバイダーの容量監視で読み込み処理が続くという管理者ポータルの既知の問題に対処することができます。

## <a name="known-issues"></a>既知の問題

Azure Stack Hub 統合システムの[証明書をローテーションする](azure-stack-mysql-resource-provider-maintain.md#secrets-rotation)場合、Key Vault 証明書のパスワードを更新する意図がなくても、KeyVaultPfxPassword 引数は必須となります。

## <a name="next-steps"></a>次のステップ

- [SQL リソース プロバイダーの詳細を確認します](azure-stack-sql-resource-provider.md)。
- [SQL リソースプロバイダーのデプロイを準備します](azure-stack-sql-resource-provider-deploy.md#prerequisites)。
- [SQL リソース プロバイダーを以前のバージョンからアップグレードします](azure-stack-sql-resource-provider-update.md)。
