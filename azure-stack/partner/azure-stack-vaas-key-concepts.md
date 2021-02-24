---
title: サービスとしての検証の主要概念
titleSuffix: Azure Stack Hub
description: Azure Stack Hub のサービスとしての検証の主要概念について説明します。
author: mattbriggs
ms.topic: article
ms.date: 12/16/2020
ms.author: mabrigg
ms.reviewer: johnhas
ms.lastreviewed: 10/28/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 7419cbe82e6bcd2eb79af486daf30975f6c8cdd0
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97874661"
---
# <a name="validation-as-a-service-key-concepts"></a>サービスとしての検証の主要概念

この記事では、サービスとしての検証 (VaaS) の主要概念について説明します。

## <a name="solutions"></a>ソリューション

VaaS ソリューションは、特定のハードウェア部品表 (BoM) を含む Azure Stack Hub ソリューションを表します。 VaaS ソリューションは、Azure Stack Hub ソリューションに対して実行されるワークフローのコンテナーとして機能します。

### <a name="create-a-solution-in-the-azure-stack-hub-validation-portal"></a>Azure Stack Hub 検証ポータルでソリューションを作成する

1. [Azure Stack Hub 検証ポータル](https://azurestackvalidation.com)にサインインします。
2. ソリューション ダッシュボードで、 **[新しいソリューション]** を選択します。
3. ソリューションの名前を入力します。 名前付けの推奨事項については、「[VaaS ソリューションの名前付け規則](azure-stack-vaas-best-practice.md#naming-convention-for-vaas-solutions)」をご覧ください。
4. **[保存]** を選択してソリューションを作成します。

## <a name="workflows"></a>Workflows

VaaS ワークフローは、VaaS ソリューションのコンテキスト内で動作します。 ワークフローは、Azure Stack Hub デプロイの機能を実行する一連のテスト スイートを表します。 Azure Stack Hub ソリューションのデプロイまたはソフトウェア更新プログラムごとにワークフローを作成する必要があります。

ワークフローは、テスト シナリオの種類によって分類されています。 非公式のテストでは、**テスト成功** ワークフローを使用して、使用可能なすべての VaaS 2 次テストからテストを選択できます。 公式のテストでは、**検証** ワークフローは Microsoft が選択した特定のテスト シナリオを対象としています。

![VaaS ワークフロー タイル](media/tile_all-workflows.png)

> [!NOTE]
> 現在、**パッケージ検証** ワークフローでは 2 つのシナリオがサポートされています。[OEM パッケージの検証](azure-stack-vaas-validate-oem-package.md)と [Microsoft のソフトウェア更新プログラムの検証](azure-stack-vaas-validate-microsoft-updates.md)です。

ワークフローの種類の詳細については、「[Azure Stack Hub のサービスとしての検証とは](azure-stack-vaas-overview.md)」をご覧ください。

### <a name="getting-started-with-vaas-workflows"></a>VaaS ワークフローの作業を開始する

1. ソリューション ダッシュボードで、新しいソリューションを作成するか、既存のソリューションを選択します。 これにより、ワークフロー タイルが最新の情報に更新され、アクティブになります。
2. 新しいワークフローを作成するには、任意のタイルで **[開始]** を選択します。 各ワークフローに固有の情報については、次の記事をご覧ください。
    - テスト成功:[クイック スタート: Azure Stack Hub 検証ポータルを使用して初めてのテストをスケジュールする](azure-stack-vaas-schedule-test-pass.md)
    - ソリューション検証:[新しい Azure Stack Hub ソリューションの検証](azure-stack-vaas-validate-solution-new.md)
    - パッケージ検証 (毎月の更新プログラム):[Microsoft のソフトウェア更新プログラムの検証](azure-stack-vaas-validate-microsoft-updates.md)
    - パッケージ検証 (パッケージの署名):[OEM パッケージの検証](azure-stack-vaas-validate-oem-package.md)

3. 既存のワークフローを管理または監視するには、ワークフロー タイルで **[管理]** を選択します。 ワークフローの名前を選択し、 **[編集]** ボタンを使用してプロパティを表示するか、共通のテスト パラメーターを変更します。

ワークフローのプロパティとパラメーターの詳細については、「[Azure Stack Hub のサービスとしての検証のためのワークフロー共通パラメーター](azure-stack-vaas-parameters.md)」をご覧ください。

## <a name="tests"></a>テスト

VaaS でのテストは、Azure Stack Hub ソリューションに対して実行される一連の操作で構成されます。 テストには、カテゴリ (機能や信頼性など) で識別されるさまざまな目的があり、Azure Stack Hub の 1 つ以上のサービスを対象としています。 各テストでは独自のパラメーター セットが定義されており、その一部はテストが含まれているワークフローの共通パラメーターによって指定されています。

テストの管理と監視の詳細については、[Azure Stack Hub 検証ポータルでのテストの監視と管理](azure-stack-vaas-monitor-test.md)に関する記事をご覧ください。

テスト パラメーターの詳細については、「[Azure Stack Hub のサービスとしての検証のためのワークフロー共通パラメーター](azure-stack-vaas-parameters.md)」をご覧ください。

## <a name="agents"></a>エージェント

テストは VaaS エージェントによって実行されます。 VaaS テストを実行するエージェントには、次の 2 種類があります。

- **クラウド エージェント**: これは、VaaS で使用できる既定のエージェントです。 セットアップは不要ですが、環境へのインバウンド接続が必要であり、Azure Stack Hub エンドポイントはインターネットから解決できる必要があります。 クラウド エージェントと互換性がないテストもあります。
- **ローカル エージェント**: これを使用すると、環境へのインバウンド接続を実現できないシナリオで検証を実行できます。 一部のテストは、ローカル エージェントを使用して実行する必要があります。

ローカル エージェントは、特定の Azure Stack Hub または VaaS ソリューションに関連付けられているわけではありません。 ベスト プラクティスとして、Azure Stack Hub 環境の外部で実行してください。

ローカル エージェントを追加する手順については、「[Deploy the local agent (ローカル エージェントのデプロイ)](azure-stack-vaas-local-agent.md)」をご覧ください。

## <a name="next-steps"></a>次のステップ

- [サービスとしての検証のベスト プラクティス](azure-stack-vaas-best-practice.md)