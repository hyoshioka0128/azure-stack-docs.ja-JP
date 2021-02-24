---
title: Azure Stack Hub の DNS フォワーダーを更新する
description: Azure Stack Hub の DNS フォワーダーを更新する方法を学習します。
author: PatAltimore
ms.topic: conceptual
ms.date: 11/21/2019
ms.author: patricka
ms.reviewer: thoroet
ms.lastreviewed: 11/21/2019
ms.openlocfilehash: 09c827d3b00470dd8e241181fc243bace62fc38b
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97871176"
---
# <a name="update-the-dns-forwarder-in-azure-stack-hub"></a>Azure Stack Hub の DNS フォワーダーを更新する

Azure Stack Hub インフラストラクチャで外部名を解決するには、少なくとも 1 つの到達可能な DNS フォワーダーが必要です。 Azure Stack Hub のデプロイには、DNS フォワーダーを用意する必要があります。 その入力が Azure Stack Hub の内部 DNS サーバーでフォワーダーとして使用され、認証、マーケットプレース管理、使用などのサービスの外部名解決が可能になります。

DNS は、変更可能な重要なデータセンター インフラストラクチャ サービスです。 このように機能するためには、Azure Stack Hub を更新する必要があります。

この記事では、特権エンドポイント (PEP) を使用して Azure Stack Hub の DNS フォワーダーを更新する方法について説明します。 2 つの信頼できる DNS フォワーダー IP アドレスを使用することをお勧めします。

## <a name="steps-to-update-the-dns-forwarder"></a>DNS フォワーダーを更新する手順

1. [特権エンドポイント](azure-stack-privileged-endpoint.md)にアクセスします。 サポート チケットを開いて、特権エンドポイントのロックを解除する必要はありません。

2. 次のコマンドを実行して、現在構成されている DNS フォワーダーを確認します。 別の方法として、管理者ポータルのリージョンのプロパティを使用することもできます。

   ```powershell
   Get-AzsDnsForwarder
   ```

3. 次のコマンドを実行して、新しい DNS フォワーダーを使用するように Azure Stack Hub を更新します。

   ```powershell
    Set-AzsDnsForwarder -IPAddress "IPAddress 1","IPAddress 2"
   ```

4. エラーが発生した場合は、コマンドの出力を確認してください。

## <a name="next-steps"></a>次のステップ

[ファイアウォールの統合](azure-stack-firewall.md)
