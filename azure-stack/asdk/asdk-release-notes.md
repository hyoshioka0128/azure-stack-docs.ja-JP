---
title: ASDK リリース ノート
description: Azure Stack Development Kit (ASDK) の機能強化、修正、既知の問題。
author: sethmanheim
ms.topic: article
ms.date: 08/10/2020
ms.author: sethm
ms.reviewer: misainat
ms.lastreviewed: 08/10/2020
ms.openlocfilehash: 6bbbc76acd38984924c454c26204f1edb0d68142
ms.sourcegitcommit: 52b33ea180c38a5ecce150f5a9ea4a026344cc3d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/11/2020
ms.locfileid: "88074130"
---
# <a name="asdk-release-notes"></a>ASDK リリース ノート

この記事では、Azure Stack Development Kit (ASDK) の変更、修正、および既知の問題に関する情報を提供します。 実行しているバージョンが不明な場合は、[ポータルを使用して確認](../operator/azure-stack-updates.md)します。

![RSS](./media/asdk-release-notes/feed-icon-14x14.png) [RSS フィード](https://docs.microsoft.com/api/search/rss?search=ASDK+release+notes&locale=en-us#)をサブスクライブして、ASDK の新着情報を常に把握するようにしてください。

::: moniker range="azs-2005"
## <a name="build-12005040"></a>ビルド 1.2005.0.40

### <a name="new-features"></a>新機能

- このリリースでの修正された問題、変更点、および新機能の一覧については、[Azure Stack リリースノート](../operator/release-notes.md)の関連セクションを参照してください。

### <a name="fixed-and-known-issues"></a>修正された問題と既知の問題

- 解読証明書のパスワードは、バックアップ データの解読に必要な秘密キーが含まれている自己署名証明書 (.pfx) のパスワードを指定するための新しいオプションです。 このパスワードは、バックアップが証明書を使用して暗号化されている場合にのみ必要です。
- マルチノード ソース システムで元の外部証明書のパスワードが変更された場合にクラウドの回復に失敗する問題を修正しました。 
- このリリースでの Azure Stack の既知の問題の一覧については、[既知の問題](../operator/known-issues.md)に関する記事を参照してください。
- 使用可能な Azure Stack 修正プログラムが ASDK には適用できないことに注意してください。

#### <a name="initial-configuration-fails-in-asdk"></a>ASDK で初期構成に失敗する

- ASDK をデプロイするときに、"**Status of 'Deployment-Phase0-DeployBareMetal' is 'Error' ('Deployment-Phase0-DeployBareMetal' の状態が 'エラー' です)** " や " **'Deployment-InitialSteps' の状態が 'エラー' です**" というエラー メッセージが表示される場合があります。

- 以下の回避策があります。

1. PowerShell ISE など、行カウンターが備わっている任意のエディターで、C:\CloudDeployment\Roles\PhysicalMachines\Tests\BareMetal.Tests.ps1 ファイルを開きます。

2. 822 行目を次のように置き換えます。

   ```powershell

   PartNumber = if($_.PartNumber) {$_.PartNumber.Trim()} else {""};

   ```  
::: moniker-end

::: moniker range="azs-2002"
## <a name="build-12002035"></a>ビルド 1.2002.0.35

### <a name="new-features"></a>新機能

- このリリースでの修正された問題、変更点、および新機能の一覧については、[Azure Stack リリースノート](../operator/release-notes.md)の関連セクションを参照してください。

### <a name="fixed-and-known-issues"></a>修正された問題と既知の問題

- 解読証明書のパスワードは、バックアップ データの解読に必要な秘密キーが含まれている自己署名証明書 (.pfx) のパスワードを指定するための新しいオプションです。 このパスワードは、バックアップが証明書を使用して暗号化されている場合にのみ必要です。

- このリリースでの Azure Stack の既知の問題の一覧については、[既知の問題](../operator/known-issues.md)に関する記事を参照してください。

- 使用可能な Azure Stack 修正プログラムが ASDK には適用できないことに注意してください。

#### <a name="sql-vm-provision-fails-in-asdk"></a>ASDK で SQL VM のプロビジョニングが失敗する

- 適用先:この問題は、ASDK 2002 に適用されます。
- 原因: ASDK 2002 で新しい SQL VM を作成する際に、「**Extension with publisher 'Microsoft.SqlServer.Management', type 'SqlIaaSAgent', and type handler version '2.0' could not be found in the extension repository. (発行元 'Microsoft.SqlServer.Management'、種類 'SqlIaaSAgent'、種類ハンドラー バージョン '2.0' の拡張機能が拡張機能リポジトリに見つかりません。)** 」というエラー メッセージが表示されることがあります。 Azure Stack Hub に **SqlIaaSAgent** 2.0 がありません。
::: moniker-end

::: moniker range="azs-1910"

## <a name="build-11910058"></a>ビルド 1.1910.0.58

### <a name="new-features"></a>新機能

- このリリースでの修正された問題、変更点、および新機能の一覧については、[Azure Stack リリースノート](../operator/release-notes.md)の関連セクションを参照してください。

### <a name="fixed-and-known-issues"></a>修正された問題と既知の問題

- ログを収集して Azure Storage BLOB コンテナーに格納する際の問題が修正されました。 この操作の構文は次のとおりです。

  ```powershell
  Get-AzureStackLog -OutputSasUri "<Blob service SAS Uri>"
  ``` 

- スプーラ サービスの読み込みに時間がかかることで一部の Windows 機能の削除が妨げられ、再起動が必要になるというデプロイの問題が修正されました。
- このリリースでの Azure Stack の既知の問題の一覧については、[既知の問題](../operator/known-issues.md)に関する記事を参照してください。
- 使用可能な Azure Stack 修正プログラムが ASDK には適用できないことに注意してください。
::: moniker-end
