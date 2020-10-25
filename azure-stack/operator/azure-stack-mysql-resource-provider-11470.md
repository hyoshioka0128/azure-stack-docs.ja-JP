---
title: Azure Stack Hub MySQL リソース プロバイダー 1.1.47.0 のリリース ノート
description: リリース ノートを参照して、Azure Stack Hub MySQL リソース プロバイダー 1.1.47.0 更新プログラムの新機能を確認してください。
author: justinha
ms.topic: article
ms.date: 11/26/2019
ms.author: justinha
ms.reviewer: xiaofmao
ms.lastreviewed: 11/26/2019
ms.openlocfilehash: 5bc03134a859182909e62c60159049d639eb8ffe
ms.sourcegitcommit: 08a421ab5792ab19cc06b849763be22f051e6d78
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/02/2020
ms.locfileid: "89364679"
---
# <a name="mysql-resource-provider-11470-release-notes"></a>MySQL リソース プロバイダー 1.1.47.0 のリリース ノート

これらのリリース ノートでは、MySQL リソース プロバイダー バージョン 1.1.47.0 の機能強化と既知の問題について説明します。

## <a name="build-reference"></a>ビルドのリファレンス
MySQL リソース プロバイダー バイナリをダウンロードした後、自己展開ツールを実行してコンテンツを一時ディレクトリに展開します。 リソース プロバイダーには、対応する最低限の Azure Stack Hub のビルドがあります。 MySQL リソース プロバイダーのこのバージョンをインストールするために必要な Azure Stack Hub の最小リリース バージョンを次に示します。

> |Azure Stack Hub の最小バージョン|MySQL リソース プロバイダーのバージョン|
> |-----|-----|
> |バージョン 1910 (1.1910.0.58)|[MySQL RP バージョン 1.1.47.0](https://aka.ms/azurestackmysqlrp11470)|  
> |     |     |

> [!IMPORTANT]
> MySQL リソース プロバイダーの最新バージョンをデプロイする前に、サポートされている最低限の Azure Stack Hub 更新プログラムを Azure Stack Hub 統合システムに適用するか、最新の Azure Stack Development Kit (ASDK) をデプロイします。

## <a name="new-features-and-fixes"></a>新機能と修正

このバージョンの Azure Stack Hub MySQL リソース プロバイダーは、リソース プロバイダーが 1910 更新プログラムでの最新のポータル変更の一部との互換性を持つようにするための修正プログラム リリースです。 新しい機能はありません。

最新の Azure Stack Hub API バージョン プロファイル 2019-03-01-hybrid と Azure Stack Hub PowerShell モジュール 1.8.0 もサポートされています。 このため、デプロイおよび更新時にモジュールの特定の履歴バージョンをインストールする必要はありません。

Azure Stack Hub が 1910 リリースにアップグレードされた後に MySQL リソース プロバイダーの修正プログラム 1.1.47.0 を適用することをお勧めします。

## <a name="known-issues"></a>既知の問題

Azure Stack Hub 統合システムの[証明書をローテーションする](azure-stack-mysql-resource-provider-maintain.md#secrets-rotation)場合、Key Vault 証明書のパスワードを更新する意図がなくても、KeyVaultPfxPassword 引数は必須となります。

## <a name="next-steps"></a>次のステップ

- [MySQL リソース プロバイダーの詳細を確認します](azure-stack-mysql-resource-provider.md)。
- [MySQL リソースプロバイダーのデプロイを準備します](azure-stack-mysql-resource-provider-deploy.md#prerequisites)。
- [MySQL リソース プロバイダーを以前のバージョンからアップグレードします](azure-stack-mysql-resource-provider-update.md)。
