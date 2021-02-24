---
title: Azure Stack HCI で Azure portal を使用する
description: Azure portal を使用して Azure Stack HCI クラスターを表示および管理する方法。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 09/15/2020
ms.openlocfilehash: e261c54140ed62a7b2fd19588e2b3ffc3b3911b8
ms.sourcegitcommit: 0579942530284d63472fb346cf5ff55de328cea4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90560887"
---
# <a name="use-the-azure-portal-with-azure-stack-hci"></a>Azure Stack HCI で Azure portal を使用する

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

このトピックでは、Azure Stack HCI クラスターのグローバル ビューのために Azure portal の Azure Stack HCI 部分に接続する方法について説明します。 物理インフラストラクチャがオンプレミスでホストされている場合でも、Azure portal を使用してクラスターを管理および監視できます。 クラウド ベースの監視によって、オンプレミスの監視システムとデータベースを維持する必要がなくなり、インフラストラクチャの複雑さが軽減されます。 また、アラートやその他の情報を Azure に直接アップロードすることでスケーラビリティが向上します。そこでは、既に毎日何百万ものオブジェクトが管理されています。

## <a name="view-your-clusters-in-the-azure-portal"></a>Azure portal でクラスターを表示する

Azure Stack HCI クラスターが登録されると、その Azure リソースを Azure portal で表示できます。 これを表示するには、最初に [Azure portal](https://portal.azure.com) にサインインします。 既に[クラスターを Azure に登録](../deploy/register-with-azure.md)している場合、クラスター名に "-rg".が付加された名前を持つ新しいリソース グループが表示されます。 Azure Stack HCI リソース グループが表示されない場合は、"hci" を検索し、ドロップダウン メニューからクラスターを選択します。

:::image type="content" source="media/azure-portal/azure-portal-search.png" alt-text="Azure portal で hci を検索して Azure Stack HCI リソースを見つける":::

Azure Stack HCI サービスのホーム ページには、すべてのクラスターが、リソース グループ、場所、関連付けられているサブスクリプションと共に一覧表示されます。

:::image type="content" source="media/azure-portal/azure-portal-home.png" alt-text="Azure portal の Azure Stack HCI サービスのホーム ページ":::

Azure Stack HCI リソースをクリックして、そのリソースの概要ページを表示します。ここには、クラスターとサーバー ノードの概要が表示されます。

:::image type="content" source="media/azure-portal/azure-portal-overview.png" alt-text="Azure portal の Azure Stack HCI リソースの概要ページ":::

## <a name="view-the-activity-log"></a>アクティビティ ログを表示する

アクティビティ ログには、クラスターでの最近の操作とイベントの一覧が、その状態、時刻、関連付けられているサブスクリプション、開始ユーザーと共に表示されます。 サブスクリプション、重大度、期間、リソース グループ、リソースでイベントをフィルター処理できます。

:::image type="content" source="media/azure-portal/azure-portal-activity-log.png" alt-text="Azure portal の Azure Stack HCI リソースのアクティビティ ログ画面":::

## <a name="configure-access-control"></a>アクセスの制御の構成

ユーザー アクセスの確認、ロールの管理、ロール割り当ての追加と表示、割り当ての拒否を行うには、アクセス制御を使用します。

:::image type="content" source="media/azure-portal/azure-portal-iam.png" alt-text="Azure portal の Azure Stack HCI リソースのアクセス制御画面":::

## <a name="add-and-edit-tags"></a>タグを追加および編集する

タグは名前と値のペアで、同じタグを複数のリソースやリソース グループに適用することでリソースを分類したり、統合した請求を表示したりできるようにします。 タグ名では大文字と小文字が区別されず、タグ値では大文字と小文字が区別されます。 [タグの詳細を確認します](/azure/azure-resource-manager/management/tag-resources)。

:::image type="content" source="media/azure-portal/azure-portal-tags.png" alt-text="Azure portal での Azure Stack HCI リソースのタグの追加または編集":::

## <a name="compare-azure-portal-and-windows-admin-center"></a>Azure portal と Windows Admin Center の比較

Windows Admin Center と異なり、Azure Stack HCI の Azure portal エクスペリエンスは、グローバル規模のマルチクラスター監視向けに設計されています。 次の表を使用して、ニーズに合致する管理ツールを判別してください。 また、これらは一貫した設計を提供しており、補完的なシナリオで役立ちます。

| Windows Admin Center | Azure portal |
| --------------- | --------------- |
| Edge ローカル ハードウェアと仮想マシン (VM) の管理、常に使用可能 | 大規模な管理、追加機能 |
| Azure Stack HCI インフラストラクチャの管理 | その他の Azure サービスの管理 |
| 個々のクラスターの監視と更新 | 大規模な監視と更新 |
| 手動での VM のプロビジョニングと管理 | Azure Arc からのセルフサービス VM |

## <a name="next-steps"></a>次のステップ

関連情報については、以下も参照してください。

- [Azure Stack HCI を Azure に接続する](../deploy/register-with-azure.md)
- [Azure Monitor を使用して Azure Stack HCI を監視する](azure-monitor.md)
