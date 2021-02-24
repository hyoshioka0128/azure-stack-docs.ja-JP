---
title: Azure Stack Hub のプロファイルでサポートされているリソース プロバイダー API バージョン
description: Azure Stack Hub のプロファイルでサポートされている Azure Resource Manager API のバージョンについて説明します。
author: sethmanheim
ms.custom: contperf-fy20q4
ms.topic: article
ms.date: 12/2/2020
ms.author: sethm
ms.reviewer: sijuman
ms.lastreviewed: 12/2/2020
ms.openlocfilehash: 31916f9f68f69785eb1e6e16cd9abd3ca8bf84f8
ms.sourcegitcommit: e13f27291bab236aac5d8b05401056961e9cc1e9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/19/2020
ms.locfileid: "97697662"
---
# <a name="resource-provider-api-versions-supported-by-profiles-in-azure-stack-hub"></a>Azure Stack Hub のプロファイルでサポートされているリソース プロバイダー API バージョン

この記事では、Azure Stack Hub で使用される API プロファイルごとにリソース プロバイダーおよびバージョン番号を確認することができます。 この記事の表には、各リソース プロバイダーでサポートされているバージョンと、プロファイルの API バージョンが一覧表示されています。 各リソース プロバイダーには、一連リソースの種類と特定のバージョン番号が含まれています。

API プロファイルでは、次の 3 つの名前付け規則を使用します。

- **latest**
- **yyyy-mm-dd-hybrid**
- **yyyy-mm-dd-profile**

Azure Stack Hub の API プロファイルとバージョン リリース周期の詳細については、「[Azure Stack Hub での API バージョン プロファイルの管理](azure-stack-version-profiles.md)」を参照してください。

> [!NOTE]
> **最新** の API プロファイルには最新バージョンのリソース プロバイダーの API が含まれており、この記事には示されていません。

## <a name="overview-of-the-2019-03-01-hybrid-profile"></a>2019-03-01 ハイブリッド プロファイルの概要

| リソース プロバイダー | API バージョン |
|-----------------------------------------------|-----------------------------------------------------|
| Microsoft.Compute | 2017-12-01 |
| Microsoft.Network | 2017-10-01<br>VPN Gateway は 2017-10-01 |
| Microsoft.Storage (データ プレーン) | 2017-11-09 |
| Microsoft.Storage (コントロール プレーン) | 2017-10-01 |
| Microsoft.Web | 2018-02-01 |
| Microsoft.KeyVault | 2016-10-01 (変更なし) |
| Microsoft.Resources (Azure Resource Manager 自体) | 2016-06-01 |
| Microsoft.Authorization (ポリシーの操作) | 2016-09-01 |
| Microsoft.Insights | 2018-01-01 |

API プロファイルのプロバイダー用リソースの種類ごとのバージョン一覧については、「[2019-03-01 ハイブリッド プロファイルの詳細](#details-for-the-2019-03-01-hybrid-profile)」を参照してください。

## <a name="details-for-the-2019-03-01-hybrid-profile"></a>2019-03-01 ハイブリッド プロファイルの詳細

### <a name="microsoftauthorization"></a>Microsoft.Authorization

ロールベースのアクセス制御を使用して、所属する組織のユーザーがリソースに対して実行できるアクションを管理することができます。 ロールを定義し、ユーザーまたはグループにロールを割り当て、アクセス許可に関する情報を取得できます。 詳細については、「[承認](/rest/api/authorization/)」をご覧ください。

| リソースの種類 | API のバージョン |
|---------------------|--------------------|
| Locks | 2016-09-01 |
| 操作 | 2015-07-01 |
| アクセス許可 | 2015-07-01 |
| ポリシーの割り当て | 2016-12-01 |
| ポリシーの定義 | 2016-12-01 |
| プロバイダー操作 | 2015-07-01 |
| ロールの割り当て | 2015-07-01 |
| ロールの定義 | 2015-07-01 |

### <a name="microsoftcommerce"></a>Microsoft.Commerce

| リソースの種類 | API Version |
|----------------------------------|----------------------|
| 委任されたプロバイダー サブスクリプション | 2015-06-01 - プレビュー |
| 委任された利用状況の集計 | 2015-06-01 - プレビュー |
| リソース支出の見積もり | 2015-06-01 - プレビュー |
| 操作 | 2015-06-01 - プレビュー |
| サブスクライバーの利用状況の集計 | 2015-06-01 - プレビュー |
| 利用状況の集計 | 2015-06-01 - プレビュー |

### <a name="microsoftcompute"></a>Microsoft.Compute

Azure コンピューティング API によって、仮想マシンおよび仮想マシンでサポートされるリソースに、プログラムからアクセスできるようになります。 詳細については、[Azure Compute](/rest/api/compute/)」 (Azure コンピューティング) をご覧ください。

| リソースの種類 | API Version |
|---------------------------------------------------------------|-------------|
| 可用性セット | 2017-12-01 |
| 場所 | 2017-12-01 |
| 場所/操作 | 2017-12-01 |
| 場所/パブリッシャー | 2017-12-01 |
| 場所/使用法 | 2017-12-01 |
| 場所/vm サイズ | 2017-12-01 |
| 操作 | 2017-12-01 |
| Virtual Machines | 2017-12-01 |
| 仮想マシン/拡張機能 | 2017-12-01 |
| Virtual Machine Scale Sets | 2017-12-01 |
| 仮想マシン スケール セット/拡張機能 | 2017-12-01 |
| 仮想マシン スケール セット/ネットワーク インターフェイス | 2017-12-01 |
| 仮想マシン スケール セット/仮想マシン | 2017-12-01|
| 仮想マシン スケール セット/仮想マシン/ネットワーク インターフェイス | 2017-12-01 |

### <a name="microsoftgallery"></a>Microsoft.Gallery

| リソースの種類 | API Version |
|------------------|-------------|
| キュレーション | 2015-04-01 |
| キュレーション コンテンツ | 2015-04-01 |
| キュレーション抽出 | 2015-04-01 |
| ギャラリー項目 | 2015-04-01 |
| 操作 | 2015-04-01 |
| ポータル | 2015-04-01 |
| 検索 | 2015-04-01 |
| 候補 | 2015-04-01 |

### <a name="microsoftinsights"></a>Microsoft.Insights

| リソースの種類 | API のバージョン |
|--------------------|--------------------|
| 操作 | 2015-04-01 |
| イベントの種類 | 2015-04-01 |
| イベント カテゴリ | 2015-04-01 |
| メトリック定義 | 2018-01-01 |
| メトリック | 2018-01-01 |
| 診断設定 | 2017-05-01-preview |
| 診断設定カテゴリ | 2017-05-01-preview |

### <a name="microsoftkeyvault"></a>Microsoft.KeyVault

ご利用のキー コンテナー、およびそのキー コンテナー内のキー、シークレット、証明書を管理します。 詳細については、「[Azure Key Vault REST API reference](/rest/api/keyvault/)」(Azure Key Vault REST API リファレンス) をご覧ください。

| リソースの種類 | API のバージョン |
|-------------------------|--------------|
| 操作 | 2016-10-01 |
| 資格情報コンテナー | 2016-10-01 |
| コンテナー/アクセス ポリシー | 2016-10-01 |
| コンテナー/シークレット | 2016-10-01 |

### <a name="microsoftnetwork"></a>Microsoft.Network

操作の呼び出し結果は、使用可能なネットワークのクラウド操作一覧の表現になります。 詳細については、「[Operation REST API](/rest/api/operation/)」 (操作の REST API) をご覧ください。

| リソースの種類 | API のバージョン |
|---------------------------|--------------|
| 接続 | 2017-10-01 |
| DNS ゾーン数 | 2016-04-01 |
| ロード バランサー | 2017-10-01 |
| ローカル ネットワーク ゲートウェイ | 2017-10-01 |
| 場所 | 2017-10-01|
| 場所/操作の結果 | 2017-10-01 |
| 場所/操作 | 2017-10-01 |
| 場所/使用法 |2017-10-01 |
| ネットワーク インターフェイス | 2017-10-01 |
| ネットワーク セキュリティ グループ | 2017-10-01 |
| 操作 | 2017-10-01 |
| パブリック IP アドレス | 2017-10-01 |
| ルート テーブル | 2017-10-01 |
| Virtual Network ゲートウェイ | 2017-10-01 |
| 仮想ネットワーク | 2017-10-01 |

### <a name="microsoftresources"></a>Microsoft.Resources

Azure Resource Manager を使用して、Azure ソリューション用のインフラストラクチャをデプロイおよび管理することができます。 関連するリソースはリソース グループにまとめ、JSON テンプレートを使って必要なリソースをデプロイします。 Resource Manager によるリソースのデプロイと管理の概要については、「[Azure Resource Manager の概要](/azure/azure-resource-manager/resource-group-overview)」をご覧ください。

| リソースの種類 | API のバージョン |
|-----------------------------------------|-------------------|
| デプロイメント | 2018-05-01 |
| デプロイ/操作 | 2018-05-01 |
| リンク | 2018-05-01 |
| 場所 | 2018-05-01 |
| 操作 | 2018-05-01 |
| プロバイダー | 2018-05-01 |
| ResourceGroups| 2018-05-01 |
| リソース | 2018-05-01 |
| サブスクリプション | 2018-05-01 |
| Subscriptions/locations | 2016-06-01 |
| Subscriptions/operationresults | 2018-05-01 |
| サブスクリプション/プロバイダー | 2018-05-01 |
| Subscriptions/ResourceGroups | 2018-05-01 |
| サブスクリプション/リソース グループ/リソース | 2018-05-01 |
| サブスクリプション/リソース | 2018-05-01 |
| サブスクリプション/タグ名 | 2018-05-01 |
| サブスクリプション/タグ名/タグの値 | 2018-05-01 |
| テナント | 2016-06-01 |

### <a name="microsoftstorage"></a>Microsoft.Storage

ストレージ リソース プロバイダー (SRP) では、ストレージ アカウントとキーをプログラムで管理することができます。 詳細については、「[Azure Storage Resource Provider REST API Reference](/rest/api/storagerp/)」(Azure ストレージ リソース プロバイダー REST API リファレンス) をご覧ください。

| リソースの種類 | API のバージョン |
|-------------------------|--------------|
| CheckNameAvailability | 2017-10-01 |
| 場所 | 2017-10-01 |
| 場所/クォータ | 2017-10-01 |
| 操作 | 2017-10-01 |
| ストレージ アカウント | 2017-10-01 |
| 使用法 | 2017-10-01 |

## <a name="next-steps"></a>次のステップ

- [PowerShell for Azure Stack Hub をインストールする](../operator/powershell-install-az-module.md)
- [Azure Stack Hub の PowerShell 環境を構成する](azure-stack-powershell-configure-user.md)  
