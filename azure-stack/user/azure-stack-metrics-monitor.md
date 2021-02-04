---
title: Azure Stack Hub の監視データの使用
description: Azure Stack Hub からの監視データを使用するためのオプションについて説明します。
author: mattbriggs
ms.topic: article
ms.date: 2/1/2021
ms.author: mabrigg
ms.lastreviewed: 11/11/2019
ms.openlocfilehash: b4398423fa50c0c9ff90cd9d9c73a48760791a8a
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99247627"
---
# <a name="consume-monitoring-data-from-azure-stack-hub"></a>Azure Stack Hub の監視データの使用

グローバル Azure の Azure Monitor と同様に、Azure Monitor パイプラインを使用して、監視データを 1 か所で確認します。 ただし、グローバル Azure で使用できるすべての監視データを Azure Stack Hub で使用できるわけではありません。 この記事では、Azure Stack Hub で監視データを使用するさまざまな方法の概要について説明します。
 
## <a name="options-for-data-consumption"></a>データの使用のオプション

| データ型 | カテゴリ | サポートされているサービス | アクセスの方法 |
|-------------------------------------------------------------|----------|------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| Azure Monitor プラットフォーム レベルのメトリック | メトリック | [Azure Stack Hub 上の Azure Monitor でサポートされるメトリック](azure-stack-metrics-supported.md) | REST API |
| ゲスト OS メトリック (Perf カウントなど) の計算 | メトリック | Windows VM と Linux VM | ストレージ テーブルまたは BLOB:<br>Windows または Linux の Azure Diagnostics <br>イベント ハブ:<br>Windows の Azure Diagnostics |
| Storage のメトリック | メトリック | Azure Storage | ストレージ テーブル:<br>Storage Analytics |
| アクティビティ ログ | events | すべての Azure サービス | REST API:<br>Azure Monitor イベント API |
| ゲスト OS ログ (IIS、ETW、syslog など) の計算 | events | Windows VM と Linux VM | ストレージ テーブルまたは BLOB:<br>Windows または Linux の Azure Diagnostics <br>イベント ハブ:<br>Windows の Azure Diagnostics |
| ストレージ ログ | events | Azure Storage | ストレージ テーブル:<br>Storage Analytics |

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub 上の Azure Monitor](azure-stack-metrics-azure-data.md) に関する詳細情報。
