---
title: Azure Stack Hub のタイム サーバーを構成する
description: Azure Stack Hub のタイム サーバーを構成する方法について説明します。
author: PatAltimore
ms.topic: article
ms.date: 2/19/2020
ms.author: patricka
ms.reviewer: thoroet
ms.lastreviewed: 10/10/2019
ms.openlocfilehash: e50bb123e7f52c17e5d180d0dd33cc84669882ab
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97871414"
---
# <a name="configure-the-time-server-for-azure-stack-hub"></a>Azure Stack Hub のタイム サーバーを構成する

Azure Stack Hub でのタイム サーバーを更新するには、特権エンドポイント (PEP) を使用します。 2 つ以上の NTP (ネットワーク タイム プロトコル) サーバーの IP アドレスに解決されるホスト名を使用します。

Azure Stack Hub では、NTP を使用して、インターネット上のタイム サーバーに接続します。 NTP サーバーからは、正確なシステム時刻が提供されます。 時刻は、Azure Stack Hub の物理ネットワーク スイッチ、ハードウェア ライフサイクル ホスト、インフラストラクチャ サービス、および仮想マシン間で使用されます。 クロックが同期されていない場合、ネットワークと認証の重大な問題が Azure Stack Hub で発生する可能性があります。 ログ ファイル、ドキュメント、およびその他のファイルが、不正確なタイムスタンプによって作成される可能性があります。

Azure Stack Hub で時刻を同期するには、タイム サーバー (NTP) を 1 つ指定する必要があります。 Azure Stack Hub のデプロイ時には、NTP サーバーのアドレスを指定します。 時刻は、重要なデータセンター インフラストラクチャ サービスです。 サービスが変更された場合は、時刻を更新する必要があります。

> [!NOTE]
> Azure Stack Hub で時刻を同期できるタイム サーバー (NTP) は 1 つのみです。 時刻を同期するために Azure Stack Hub に NTP を複数指定することはできません。

## <a name="configure-time"></a>時刻の構成

1. [PEP に接続します](azure-stack-privileged-endpoint.md)。
    > [!Note]  
    > サポート チケットを開いて、特権エンドポイントのロックを解除する必要はありません。

2. 次のコマンドを実行して、現在構成されている NTP サーバーを確認します。

    ```PowerShell
    Get-AzsTimeSource
    ```

3. 次のコマンドを実行して、新しい NTP サーバーを使用して直ちに時刻を同期するように Azure Stack Hub を更新します。

    > [!Note]  
    > この手順では、物理スイッチ上のタイム サーバーは更新されません。 タイム サーバーが Windows ベースの NTP サーバーではない場合、フラグ `0x8` を追加する必要があります。

    ```PowerShell
    Set-AzsTimeSource -TimeServer NEWTIMESERVERIP -resync
    ```

    Windows ベースのタイム サーバー以外のサーバーの場合:

    ```PowerShell
    Set-AzsTimeSource -TimeServer "NEWTIMESERVERIP,0x8" -resync
    ```

4. エラーが発生した場合は、コマンドの出力を確認してください。


## <a name="next-steps"></a>次のステップ

[対応状況レポートを表示する](azure-stack-validation-report.md)  
[Azure Stack Hub の統合に関する一般的な考慮事項](azure-stack-datacenter-integration.md)  
