---
title: 開始と停止
titleSuffix: Azure Stack Hub
description: Azure Stack Hub を開始および停止する方法について説明します。
author: PatAltimore
ms.topic: how-to
ms.date: 03/04/2020
ms.author: patricka
ms.reviewer: misainat
ms.lastreviewed: 10/15/2019
ms.openlocfilehash: a79fcab4bb96eb7e32121095c521e4db0a3b02f9
ms.sourcegitcommit: 52c934f5eeb5fcd8e8f2ce3380f9f03443d1e445
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/07/2021
ms.locfileid: "97973267"
---
# <a name="start-and-stop-azure-stack-hub"></a>Azure Stack Hub の開始と停止

Azure Stack Hub サービスを正しくシャットダウンして再起動するには、この記事の手順に従ってください。 "*停止*" は、Azure Stack Hub 環境全体を物理的にシャットダウンして電源を切ります。 "*開始*" は、すべてのインフラストラクチャ ロールの電源が入れられ、テナントのリソースがシャットダウン前の電源状態に戻されます。

## <a name="stop-azure-stack-hub"></a>Azure Stack Hub の停止

Azure Stack Hub は次の手順で停止 (シャットダウン) します。

1. 将来のシャットダウンに備えて、Azure Stack Hub 環境のテナント リソースで実行しているすべてのワークロードを準備します。

2. Azure Stack Hub ERCS VM へのネットワーク アクセス権を持つマシンから特権エンドポイント セッション (PEP) を開きます。 手順については、「[Azure Stack Hub で特権エンドポイントを使用する](azure-stack-privileged-endpoint.md)」を参照してください。

3. PEP から次を実行します。

    ```powershell
      Stop-AzureStack
    ```

4. Azure Stack Hub のすべての物理ノードの電源がオフになるまで待ちます。

> [!Note]
> 物理ノードの電源の状態は、お使いの Azure Stack Hub ハードウェアを提供した相手先ブランド供給 (OEM) の手順に従って確認できます。

## <a name="start-azure-stack-hub"></a>Azure Stack Hub の開始

Azure Stack Hub は次の手順で開始します。 Azure Stack Hub を停止した方法に関係なくこれらの手順に従います。

1. Azure Stack Hub 環境内の各物理ノードの電源を入れます。 物理ノードの電源投入手順は、お使いの Azure Stack Hub のハードウェアを提供した OEM の手順に従って確認します。

2. Azure Stack Hub インフラストラクチャ サービスが開始されるまで待ちます。 Azure Stack Hub インフラストラクチャ サービスは、開始プロセスの終了に 2 時間かかることがあります。 Azure Stack Hub の開始状態は、[**Get-ActionStatus** コマンドレット](#get-the-startup-status-for-azure-stack-hub)を使って確認できます。

3. すべてのテナント リソースがシャットダウン前の状態に戻ったことを確認します。 ワークロード マネージャーによる起動後に、テナント リソースで実行しているワークロードの再構成が必要な場合があります。

## <a name="get-the-startup-status-for-azure-stack-hub"></a>Azure Stack Hub のスタートアップ状態の取得

Azure Stack Hub のスタートアップ ルーチンのスタートアップは次の手順で取得します。

1. Azure Stack Hub ERCS VM へのネットワーク アクセス権を持つマシンから特権エンドポイント セッションを開きます。

2. PEP から次を実行します。

    ```powershell
      Get-ActionStatus Start-AzureStack
    ```

## <a name="troubleshoot-startup-and-shutdown-of-azure-stack-hub"></a>Azure Stack Hub のスタートアップおよびシャットダウンのトラブルシューティング

Azure Stack Hub 環境の電源投入後 2 時間経ってもインフラストラクチャおよびテナントのサービスが正常に開始されない場合は、次の手順を実行します。

1. Azure Stack Hub ERCS VM へのネットワーク アクセス権を持つマシンから特権エンドポイント セッションを開きます。

2. 次のコマンドを実行します。

    ```powershell
      Test-AzureStack
      ```

3. 出力を確認し、正常性エラーを解決します。 詳細については、[Azure Stack Hub の検証テストの実行](azure-stack-diagnostic-test.md)に関するページを参照してください。

4. 次のコマンドを実行します。

    ```powershell
      Start-AzureStack
    ```

5. **Start-AzureStack** の実行に失敗した場合は、Microsoft サポートにお問い合わせください。

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub 診断ツール](./azure-stack-diagnostic-log-collection-overview.md)の詳細を確認する
