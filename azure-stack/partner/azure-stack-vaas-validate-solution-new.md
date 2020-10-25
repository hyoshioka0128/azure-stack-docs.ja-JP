---
title: Azure Stack Hub でのソリューションの検証
description: ソリューション検証ワークフローを使用して、新しい Azure Stack Hub ソリューションを検証する方法について説明します。
author: mattbriggs
ms.topic: quickstart
ms.date: 08/24/2020
ms.author: mabrigg
ms.reviewer: johnhas
ms.lastreviewed: 11/11/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 65984da8615d9146db98800c710efee004f021c2
ms.sourcegitcommit: 4922a14fdbc8a3b67df065336e8a21a42f224867
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2020
ms.locfileid: "88764887"
---
# <a name="validate-a-solution-in-azure-stack-hub"></a>Azure Stack Hub でのソリューションの検証

[!INCLUDE [Azure_Stack_Partner](./includes/azure-stack-partner-appliesto.md)]

**ソリューション検証**ワークフローを使用して、新しい Azure Stack Hub ソリューションを検証する方法について説明します。

Azure Stack Hub ソリューションは、Windows Server ロゴ認定要件に合格した後に Microsoft とパートナーとの間で共同で合意されたハードウェア部品表 (BoM) です。 ハードウェア BoM に変更があった場合は、ソリューションの再認定が必要になります。 ソリューションを再検証するタイミングについては、チーム [vaashelp@microsoft.com](mailto:vaashelp@microsoft.com) までお問い合わせください。

ソリューションを検証するために、ソリューションの検証ワークフローは 2 回実行してください。 1 回目は、サポートされる "*最小*" 構成で実行します。 2 回目は、サポートされる "*最大*" 構成で実行します。 両方の構成ですべてのテストに合格したソリューションが Microsoft によって検証されます。

[!INCLUDE [azure-stack-vaas-workflow-validation-completion](includes/azure-stack-vaas-workflow-validation-completion.md)]

## <a name="create-a-solution-validation-workflow"></a>ソリューションの検証ワークフローの作成

1. [!INCLUDE [azure-stack-vaas-workflow-step_select-solution](includes/azure-stack-vaas-workflow-step_select-solution.md)]

2. **[Solution Validations]\(ソリューションの検証\)** タイルの **[開始]** を選択します。

    ![ソリューションの検証ワークフローのタイル](media/tile_validation-solution.png)

3. [!INCLUDE [azure-stack-vaas-workflow-step_naming](includes/azure-stack-vaas-workflow-step_naming.md)]

4. **ソリューション構成**を選択します。
    - **[最小]** : ソリューションは、サポートされる最小ノード数で構成されます。
    - **[最大]** : ソリューションは、サポートされる最大ノード数で構成されます。
5. [!INCLUDE [azure-stack-vaas-workflow-step_upload-stampinfo](includes/azure-stack-vaas-workflow-step_upload-stampinfo.md)]

    ![ソリューションの検証の情報](media/workflow_validation-solution_info.png)

6. [!INCLUDE [azure-stack-vaas-workflow-step_test-params](includes/azure-stack-vaas-workflow-step_test-params.md)]

    > [!NOTE]
    > 環境パラメーターは、ワークフローを作成した後は変更できません。

7. [!INCLUDE [azure-stack-vaas-workflow-step_tags](includes/azure-stack-vaas-workflow-step_tags.md)]
8. [!INCLUDE [azure-stack-vaas-workflow-step_submit](includes/azure-stack-vaas-workflow-step_submit.md)]
    テストの概要ページにリダイレクトされます。

## <a name="run-solution-validation-tests"></a>ソリューションの検証テストの実行

**ソリューションの検証テストの概要**ページには、検証を完了するために必要なテストの一覧が表示されます。

検証ワークフローでは、テストを**スケジュール設定**するときに、ワークフローの作成時に指定したワークフロー レベルの一般的なパラメーターを使用します (「[Azure Stack Hub のサービスとしての検証のためのワークフロー共通パラメーター](azure-stack-vaas-parameters.md)」を参照してください)。 テスト パラメーター値のいずれかが無効になった場合は、[ワークフロー パラメーターの変更](azure-stack-vaas-monitor-test.md#change-workflow-parameters)に関するセクションの手順に従ってパラメーター値を再度指定する必要があります。

> [!NOTE]
> 既存のインスタンスに対して検証テストをスケジュール設定すると、ポータルの古いインスタンスに代わる新しいインスタンスが作成されます。 古いインスタンスのログは保持されますが、ポータルからアクセスできません。<br>
テストが正常に完了すると、 **[スケジュール]** アクションが無効になります。

1. [!INCLUDE [azure-stack-vaas-workflow-step_select-agent](includes/azure-stack-vaas-workflow-step_select-agent.md)]

2. 以下のテストを選択します。
    - Cloud Simulation Engine (クラウド シミュレーション エンジン)
    - Disk Identification Test (ディスク識別テスト)

3. テスト インスタンスをスケジュール設定するためのプロンプトを開くには、コンテキスト メニューの **[スケジュール]** を選択します。

4. テスト パラメーターを確認し、 **[送信]** を選択してテストの実行をスケジュール設定します。

![ソリューションの検証テストをスケジュール設定する](media/workflow_validation-solution_schedule-test.png)

## <a name="next-steps"></a>次のステップ

- [VaaS ポータルでのテストの監視と管理](azure-stack-vaas-monitor-test.md)