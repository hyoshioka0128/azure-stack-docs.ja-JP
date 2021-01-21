---
title: Azure Stack Hub 更新アクティビティのチェックリスト
description: 最新の Azure Stack Hub の更新に備えてシステムを準備するためのチェックリスト。
author: PatAltimore
ms.topic: article
ms.date: 08/10/2020
ms.author: patricka
ms.reviewer: TBD
ms.lastreviewed: 01/14/2020
ms.openlocfilehash: 740fa4fe450121f27211001974e91d47d3aed692
ms.sourcegitcommit: 9b0e1264ef006d2009bb549f21010c672c49b9de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98256083"
---
# <a name="azure-stack-hub-update-activity-checklist"></a>Azure Stack Hub 更新アクティビティのチェックリスト

Azure Stack Hub の更新を準備するために、このチェックリストを確認します。 この記事には、Azure Stack Hub オペレーターに向けた、更新関連のアクティビティのチェックリストが掲載されています。

## <a name="prepare-for-azure-stack-hub-update"></a>Azure Stack Hub の更新を準備する

| &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; アクティビティ &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;                   | 詳細                                                   |
|------------------------------|-----------------------------------------------------------|
| 既知の問題の確認     | [既知の問題の一覧](known-issues.md)                |
| セキュリティ更新プログラムの確認 | [セキュリティ更新プログラムの一覧](release-notes-security-updates.md)      |
| アドオン リソース プロバイダーの更新プログラムを確認する | [App Service](azure-stack-app-service-update.md)<br>[Event Hubs](resource-provider-apply-updates.md)<br> [MySQL](azure-stack-mysql-resource-provider-update.md)<br>[SQL](azure-stack-sql-resource-provider-update.md)<br>  |
| 最新の OEM パッケージの適用 | お使いのシステムがアップグレードされる予定の Azure Stack Hub バージョンに関して OEM に問い合わせて、システムが最小 OEM パッケージ要件を満たしていることを確認してください。 OEM パッケージが、更新する Azure Stack Hub バージョンと互換性があることを確認します。 OEM パッケージが更新後の Azure Stack Hub バージョンと互換性がない場合は、Azure Stack Hub 更新プログラムを実行する前に OEM パッケージの更新を実行する必要があります。 手順については、「Azure Stack Hub に OEM (相手先ブランド供給) の更新プログラムを適用する」を参照してください。 |
| 省略可能:自動ログ収集の構成 | サポート チケットを開く必要がある場合にシステム ログを収集するプロセスを効率化するため、Azure Stack Hub 環境で自動ログ収集を構成することをお勧めします。 自動ログ収集を構成するには、「[ログを事前に送信する](./diagnostic-log-collection.md#send-logs-proactively)」の手順を参照してください。 |
| 最新の修正プログラムの適用 | 現在インストールされているリリースに適用される最新の修正プログラムを適用します。 最新の修正プログラムの一覧については、[リリース ノートの修正プログラム](release-notes.md)のセクションを参照してください。 |
| 容量計画ツールの実行 | ワークロードの計画とサイズ設定を行うには、最新バージョンの [Azure Stack Hub キャパシティ プランニング ツール](azure-stack-capacity-planning-overview.md)を使用します。 最新バージョンにはバグの修正プログラムが含まれていて、Azure Stack Hub の各更新プログラムでリリースされる新機能が提供されます。 |
| **Test-AzureStack** の実行 | `Test-AzureStack -Group UpdateReadiness` を実行して、操作上の問題を特定します。 コマンドレットには、特権エンドポイント セッション (PEP) を使用してアクセスできます。 詳細については、「[Azure Stack Hub システムの状態を検証する](azure-stack-diagnostic-test.md)」を参照してください。 |
| 問題の解決 | `Test-AzureStack` によって特定された操作上の問題を解決します。 |
| 更新プログラムが利用可能 | Azure Stack Hub デプロイでは、接続されたシナリオでのみ、セキュリティで保護されたエンドポイントが定期的にチェックされ、お客様のクラウド用の更新プログラムが入手可能かどうかが自動的に通知されます。 接続していないお客様は、[ここで説明されているプロセス](azure-stack-apply-updates.md)を使用して、新しいパッケージをダウンロードしてインポートできます。 |
| メンテナンス期間の日程計画とユーザーへの通知 | メンテナンス作業についてユーザーに通知し、通常のメンテナンス期間を可能であれば勤務時間外にスケジュールする必要があります。 メンテナンス作業をポータルから開始するか、Azure Resource Manager API からプログラミングによって開始するかに関係なく、作業によって既存のテナント ワークロードに影響が出たり、新しいテナント作業 (VM の作成、再設定、または削除) が失敗したりすることがあります。 また、バックアップなどの他の操作は、更新が完了するまで使用できない場合があります。 Azure Stack Hub の Express 更新や完全更新については、[こちら](release-notes.md)のリリース ノートでご確認いただけます。ご利用のバージョンでは、更新にどのくらいの時間がかかるか予想されています。 |

## <a name="during-azure-stack-hub-update"></a>Azure Stack Hub の更新中

| アクティビティ | 詳細 |
|--------------------|------------------------------------------------------------------------------------------------------|
| 更新プログラムの管理 |[オペレーター ポータルを使用して Azure Stack Hub の更新プログラムを管理します](azure-stack-updates.md)。 |
|  |  |
| 更新プログラムの監視 | オペレーター ポータルを使用できない場合は、[特権エンドポイントを使用して Azure Stack Hub の更新プログラムを監視](azure-stack-monitor-update.md)します。 |
|  |  |
| 更新プログラムの再開 | 失敗した更新プログラムを修正したら、[特権エンドポイントを使用して Azure Stack Hub で更新プログラムを再開](azure-stack-monitor-update.md)します。 |

> [!IMPORTANT]  
> 更新中 **Test-AzureStack** を実行しないでください。更新プロセスが停止することがあります。
>
>更新中にノードの修復を実行することは、その状態にかかわらず避けてください。
>更新中にノードの修復が必要になった場合は、Microsoft サポートにお問い合わせください。

## <a name="after-azure-stack-hub-update"></a>Azure Stack Hub の更新後

| アクティビティ | 詳細 |
|--------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 最新の修正プログラムの適用 | 更新されたバージョンに適用可能な最新の修正プログラムを適用します。 |
| 暗号化キーの取得 | 保存データの暗号化キーを取得し、お使いの Azure Stack Hub デプロイの外部にそれらを安全に保管します。 [キーを取得する方法の手順](azure-stack-security-bitlocker.md)に関する記事に従ってください。 |
|  |  |
| マルチテナントの再有効化 | マルチテナント Azure Stack Hub の場合は、更新の成功後、[すべてのゲスト ディレクトリ テナントを構成していることを確認します](azure-stack-enable-multitenancy.md#configure-guest-directory)。 |

## <a name="next-steps"></a>次のステップ

- [既知の問題の一覧を確認する](known-issues.md)
- [セキュリティ更新プログラムの一覧を確認する](release-notes-security-updates.md)