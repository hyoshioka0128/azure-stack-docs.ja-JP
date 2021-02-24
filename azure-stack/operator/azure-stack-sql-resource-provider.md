---
title: SQL データベースを使用する
titleSuffix: Azure Stack Hub
description: SQL Server リソース プロバイダーを使用して、SQL データベースを Azure Stack Hub 上のサービスとして提供する方法を説明します。
author: bryanla
ms.topic: article
ms.date: 8/19/2020
ms.author: bryanla
ms.reviewer: xiaofmao
ms.lastreviewed: 10/25/2019
ms.openlocfilehash: c0f599d8a63f0a1ea16e5a39303391cb1b0790a0
ms.sourcegitcommit: 8079220917523ab9ddb824e4bba3e9b091f38a9f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/20/2020
ms.locfileid: "88661590"
---
# <a name="use-sql-databases-on-azure-stack-hub"></a>Azure Stack Hub 上で SQL データベースを使用する

SQL リソース プロバイダーを使用して、[Azure Stack Hub](azure-stack-overview.md) 上で SQL データベースを提供します。 リソースプロバイダーをインストールして 1 つまたは複数の SQL Server インスタンスに接続すると、御社および御社のユーザーは次のものを作成できます。

- クラウドネイティブ アプリ向けの SQL データベース。
- Web アプリケーション向けの SQL データベース。

SQL リソース プロバイダーをインストールする前に、考慮すべき制限事項:

- ユーザーは、個々のデータベースの作成と管理のみを行うことができます。 エンド ユーザーは、データベース サーバー インスタンスにアクセスできません。 これにより、マスター、Temp DB にアクセスしたり、データベースを動的に管理したりすることが必要なオンプレミス データベース アプリとの互換性が制限される可能性があります。
- Azure Stack Hub オペレーターは、SQL データベース サーバーとホストのデプロイ、更新、セキュリティ保護、構成、保守を担当します。 RP サービスでは、ホストおよびデータベース サーバー インスタンスの管理機能は提供されません。
- 異なるサブスクリプションの異なるユーザーのデータベースは、同じデータベース サーバー インスタンスに配置できます。 RP は、データベースを別のホストまたはデータベース サーバー インスタンス上で分離するためのメカニズムを備えていません。
- RP では、データベースのテナント使用に関するレポートは提供されません。
- SQL ホスティング サーバーは、グローバル Azure 内の別のサブスクリプションにのみ移動できます。 Azure Stack Hub では、SQL ホスティング サーバーの別のサブスクリプションへの移動はサポートされていません。

オンプレミスの従来の SQL Server ワークロードには、Azure Stack Hub 上の SQL Server 仮想マシンをお勧めします。

## <a name="sql-resource-provider-adapter-architecture"></a>SQL リソースプロバイダー アダプターのアーキテクチャ

リソース プロバイダーは、次のコンポーネントで構成されています。

- **SQL リソース プロバイダーの仮想マシン (VM)**。プロバイダー サービスを実行する Windows Server VM です。
- **リソース プロバイダー**。要求を処理し、データベース リソースにアクセスします。
- **SQL Server をホストするサーバー**。これはデータベースに容量を提供し、ホスティング サーバーと呼ばれます。

少なくとも 1 つの SQL Server インスタンスを作成するか、外部 SQL Server インスタンスへのアクセスを提供する必要があります。

> [!NOTE]
> Azure Stack Hub 統合システムにインストールされたホスティング サーバーは、テナント サブスクリプションから作成する必要があります。 既定のプロバイダー サブスクリプションからは作成できません。 作成するには、適切にサインインしたユーザー ポータルまたは PowerShell を使用する必要があります。 すべてのホスティング サーバーは課金対象の VM であり、ライセンスが必要です。 サービス管理者は、テナント サブスクリプションの所有者になることができます。

## <a name="next-steps"></a>次のステップ

[SQL Server リソース プロバイダーのデプロイ](azure-stack-sql-resource-provider-deploy.md)
