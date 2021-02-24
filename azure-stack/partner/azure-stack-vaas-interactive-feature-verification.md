---
title: 対話型機能検証テスト
titleSuffix: Azure Stack Hub
description: サービスとしての検証を使用して Azure Stack Hub 用の対話型機能検証テストを作成する方法について説明します。
author: mattbriggs
ms.topic: tutorial
ms.date: 08/24/2020
ms.author: mabrigg
ms.reviewer: johnhas
ms.lastreviewed: 10/28/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 1e7e87073518ab5a040cc96ec1af5b7db5f808d9
ms.sourcegitcommit: 53b0dde60a6435936a5e0cb9e931245f262d637a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/23/2020
ms.locfileid: "91107255"
---
# <a name="interactive-feature-verification-testing"></a>対話型機能検証テスト  

[!INCLUDE [Azure_Stack_Partner](./includes/azure-stack-partner-appliesto.md)]

お客様は、対話型機能検証テスト フレームワークを使用して、ご自分のシステムに対するテストを要求できます。 お客様がテストを要求すると、Microsoft はフレームワークを使用して、対話型の手動ステップが必要なテストを準備します。 Microsoft は、フレームワークを使用していくつかの独立した自動テストをつなぎ合わせることができます。

この記事では、シンプルな手動シナリオについて説明します。 Azure Stack Hub におけるディスクの交換がテストでチェックされます。 フレームワークによって、各ステップの診断ログが収集されます。 問題が見つかったら、それらをデバッグできます。 このフレームワークでは、他のツールやプロセスによって生成されたログの共有もできるほか、シナリオに関するフィードバックを送ることができます。

> [!Important]  
> この記事では、ディスク識別テストを実行する手順を参照します。 これは単なるデモンストレーションであり、テスト成功ワークフローから収集された結果は、新しいソリューションの検証には使用できません。

## <a name="overview-of-interactive-testing"></a>対話型テストの概要

ディスク交換に関するテストは一般的なシナリオです。 この例では、テストに 5 つのステップが含まれています。

1. 新しい**テスト パス** ワークフローを作成する。
2. **Disk Identification Test** を選択する。
3. メッセージが表示されたら、手動の手順を完了する。
4. シナリオの結果をチェックする。
5. テスト結果を Microsoft に送信する。

## <a name="create-a-new-test-pass"></a>新しいテスト パスの作成

既存のテスト成功が利用できない場合は、[テストのスケジュール設定](azure-stack-vaas-schedule-test-pass.md)の指示に従ってください。

## <a name="schedule-the-test"></a>テストのスケジュール設定

1. **[Disk Identification Test]\(ディスク識別テスト\)** を選択します。

    > [!Note]  
    > テストのバージョン番号は、2 次テストの改良が行われると増加します。 Microsoft による別段の指示がない限り、常に最新バージョンを使用することが推奨されます。

    ![ディスク識別テスト - Azure Stack Hub での対話型テストを示すスクリーンショット。](media/azure-stack-vaas-interactive-feature-verification/image4.png)

2. **[編集]** を選択して、ドメイン管理者のユーザー名とパスワードを指定します。

3. テストの起動先として適切なテスト実行エージェントまたは DVM を選択します。

    ![テスト実行エージェントの選択 - Azure Stack Hub での対話型テスト](media/azure-stack-vaas-interactive-feature-verification/image5.png)

4. **[送信]** を選択してテストを開始します。

    ![テストの確認と送信 - Azure Stack Hub での対話型テスト](media/azure-stack-vaas-interactive-feature-verification/image6.png)

5. 前の手順で選択したエージェントから対話型テストの UI にアクセスします。

    ![ディスク識別テスト - Azure Stack Hub での対話型テスト](media/azure-stack-vaas-interactive-feature-verification/image8.png)

6. **[ドキュメント]** と **[検証]** のリンク先に移動して、このシナリオの実施方法に関する Microsoft からの指示を確認します。

    ![ディスク識別テストのドキュメントと検証のリンク](media/azure-stack-vaas-interactive-feature-verification/image9.png)

7. **[次へ]** を選択します。

    ![[次へ] を選択 - Azure Stack Hub での対話型テスト](media/azure-stack-vaas-interactive-feature-verification/image10.png)

8. 手順に従って事前チェック スクリプトを実行します。

    ![事前チェック スクリプトを実行 - Azure Stack Hub での対話型テスト](media/azure-stack-vaas-interactive-feature-verification/image11.png)

9. 事前チェック スクリプトが正常に完了したら、 **[情報]** タブの **[ドキュメント]** リンクと **[検証]** リンクの説明に従って、手動シナリオ (ディスク交換) を実行します。

    ![手動シナリオの実行 - Azure Stack Hub での対話型テスト](media/azure-stack-vaas-interactive-feature-verification/image12.png)

    > [!Important]  
    > 手動シナリオを実行している間は、ダイアログ ボックスを閉じないでください。

10. 手動シナリオの実行が終了したら、手順に従って事後チェック スクリプトを実行します。

    ![事後チェック スクリプトを実行 - Azure Stack Hub での対話型テスト](media/azure-stack-vaas-interactive-feature-verification/image13.png)

11. 手動シナリオ (ディスク交換) が正常に完了した後、 **[送信]** を選択します。

    ![ディスク識別テストの送信 - Azure Stack Hub での対話型テスト](media/azure-stack-vaas-interactive-feature-verification/image14.png)

    > [!Important]  
    > ウィンドウを閉じると、テストは完了する前に停止します。

12. 実行したテストについてのフィードバックを送信します。 これらの質問は、Microsoft がシナリオの成功率とリリース品質を評価するのに役立てられます。

    ![Azure Stack Hub での対話型テストのエクスペリエンスに関するフィードバックを提供する](media/azure-stack-vaas-interactive-feature-verification/image15.png)

13. Microsoft に送信したいログ ファイルを添付します。

    ![ログ ファイルの添付 - Azure Stack Hub での対話型テスト](media/azure-stack-vaas-interactive-feature-verification/image16.png)

14. フィードバック送信の EULA に同意します。

15. **[送信]** を選択して、結果を Microsoft に送信します。

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub 検証ポータルでのテストの監視と管理](azure-stack-vaas-monitor-test.md)
