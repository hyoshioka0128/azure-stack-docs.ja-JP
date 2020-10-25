---
title: サービスとしての検証の概要
titleSuffix: Azure Stack Hub
description: Azure Stack Hub のサービスとしての検証の概要。
author: mattbriggs
ms.topic: article
ms.date: 08/24/2020
ms.author: mabrigg
ms.reviewer: johnhas
ms.lastreviewed: 11/11/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 777a87094f9e950da5a301ec328d253cb5f7a7c1
ms.sourcegitcommit: 4922a14fdbc8a3b67df065336e8a21a42f224867
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2020
ms.locfileid: "88764717"
---
# <a name="what-is-validation-as-a-service-for-azure-stack-hub"></a>Azure Stack Hub のサービスとしての検証とは

[!INCLUDE [Azure_Stack_Partner](./includes/azure-stack-partner-appliesto.md)]

サービスとしての検証 (VaaS) は、Azure Stack Hub のサービスを Microsoft と共同開発しているソリューション パートナー向けに設計されたネイティブの Azure サービスです。 ソリューション パートナーはサービスを使用して、ソリューションが Microsoft の要件を満たし、Azure Stack Hub で期待どおりに動作することを確認できます。

VaaS の主な用途は次のとおりです。

- 新しい Azure Stack Hub ソリューションの検証。
- Azure Stack Hub ソフトウェアに加えられた変更の検証。
- デプロイ時に使用されるソリューション パートナーのパッケージへのデジタル署名。
- VaaS の 2 次テストのプレビュー。

## <a name="validate-a-new-azure-stack-hub-solution"></a>新しい Azure Stack Hub ソリューションの検証

パートナーは、**ソリューション検証**ワークフローを使用して、新しい Azure Stack Hub ソリューションを検証します。 ソリューションは、必須の Hardware Lab Kit (HLK) Azure Stack Hub コンポーネント テストに合格する必要があります。 さまざまなハードウェア構成を認定するために、新しいソリューションごとに、このワークフローを 2 回 (最小構成と最大構成に対してそれぞれ 1 回) 実行する必要があります。

詳細については、[新しい Azure Stack Hub ソリューションの検証](azure-stack-vaas-validate-solution-new.md)に関するページを参照してください。

## <a name="validate-changes-to-the-azure-stack-hub-software"></a>Azure Stack Hub ソフトウェアに加えられた変更の検証

パートナーは、**パッケージ検証**ワークフローを使用して、最新の Azure Stack Hub ソフトウェア更新プログラムでソリューションが動作することを確認します。 パッケージ検証ワークフローは、修正プログラムと更新プログラム (P&U) を使用して更新が適用された、Microsoft が推奨するハードウェア環境で実行する必要があります。 また、ベースライン ビルドでもこのワークフローを実行することをお勧めします。

詳細については、「[Validate software updates from Microsoft (Microsoft のソフトウェア更新プログラムの検証)](azure-stack-vaas-validate-microsoft-updates.md)」を参照してください。

## <a name="get-digitally-signed-solution-partner-packages"></a>デジタル署名されたソリューション パートナーのパッケージの取得

パートナーは、Azure Stack Hub 更新プログラムの検証に加えて、**パッケージ検証**ワークフローを使用して OEM カスタマイズ パッケージに対する更新プログラムを検証します。 これらの更新プログラムには、Azure Stack Hub ソフトウェアのデプロイ時に使用される Azure Stack Hub パートナー固有のドライバー、ファームウェア、その他のソフトウェアが含まれています。 少なくとも、サポートが予定されている最小サイズのソリューションを使用して、Azure Stack Hub ソフトウェアの最新バージョンで検証するパッケージをデプロイします。 パッケージは、テストを実行する前に VaaS に送信されます。 テストが成功した場合は、パッケージのテストが完了し、Azure Stack Hub デジタル署名でデジタル署名する必要があることを [vaashelp@microsoft.com](mailto:vaashelp@microsoft.com) に通知します。 Microsoft がパッケージに署名し、Azure Stack Hub 検証ポータルでパッケージがダウンロード可能であることを Azure Stack Hub パートナーに通知します。

詳細については、「[Validate OEM packages (OEM パッケージの検証)](azure-stack-vaas-validate-oem-package.md)」を参照してください。

## <a name="preview-vaas-test-collateral"></a>VaaS の 2 次テストのプレビュー

Microsoft は、定期的に Azure Stack Hub で新機能を提供しています。 これらの機能を市場に投入するための開発プロセスの一環として、**テスト成功**ワークフローで新しい 2 次テストが利用できるようになりました。 テスト成功ワークフローには、他のワークフローからの 2 次テストが含まれ、非公式のテストを実行できます。 承認を求めて結果を提出するときには、テスト成功ワークフローを使用しないでください。 ソリューションの正式な承認を得るには、ソリューション検証ワークフローとパッケージ検証ワークフローを使用します。

詳細については、「[クイック スタート: Azure Stack Hub 検証ポータルを使用して初めてのテストをスケジュールする](azure-stack-vaas-schedule-test-pass.md)」を参照してください。

## <a name="validation-workflow-tests-summary"></a>検証ワークフロー テストの概要

| 検証ワークフロー | 必須のテスト |
|----|------------|
| [新しいソリューションの検証](azure-stack-vaas-validate-solution-new.md) | Cloud Simulation Engine (クラウド シミュレーション エンジン)<br>Compute SDK Operational Suite (コンピューティング SDK 操作スイート)<br>Disk Identification Test (ディスク識別テスト)<br>KeyVault Extension SDK Operational Suite (KeyVault 拡張機能 SDK 操作スイート)<br>KeyVault SDK Operational Suite (KeyVault SDK 操作スイート)<br>Network SDK Operational Suite (Network SDK 操作スイート)<br>Storage Account SDK Operational Suite (ストレージ アカウント SDK 操作スイート)<br> |
| [OEM パッケージの検証](azure-stack-vaas-validate-oem-package.md) | OEM Extension Package Verification (OEM 拡張機能パッケージの検証)<br>Cloud Simulation Engine (クラウド シミュレーション エンジン) |
| [毎月の更新プログラムの検証](azure-stack-vaas-validate-microsoft-updates.md) | Monthly Azure Stack Hub Update Verification (月次 Azure Stack Hub 更新プログラムの検証)<br>Cloud Simulation Engine (クラウド シミュレーション エンジン)<br> |

## <a name="next-steps"></a>次のステップ

- [サービスとしての検証のリソースを設定する](azure-stack-vaas-set-up-resources.md)
- [サービスとしての検証の主要概念](azure-stack-vaas-key-concepts.md)を確認する
