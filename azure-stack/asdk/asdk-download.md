---
title: ASDK のダウンロードと抽出
description: Azure Stack Development Kit (ASDK) をダウンロードおよび抽出する方法について説明します。
author: PatAltimore
ms.topic: article
ms.date: 05/06/2019
ms.author: patricka
ms.reviewer: misainat
ms.lastreviewed: 08/10/2019
ms.openlocfilehash: 3f8a0615f18ddf9ba4a98c2af2b947c49cfdd550
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97873267"
---
# <a name="download-and-extract-the-asdk"></a>ASDK のダウンロードと抽出
開発キットのホスト コンピューターが Azure Stack Development Kit (ASDK) の基本的なインストール要件を満たしていることを確認した後、次の手順は ASDK デプロイ パッケージをダウンロードおよび抽出して Cloudbuilder.vhdx を取得することです。

## <a name="download-the-asdk"></a>ASDK のダウンロード
1. ダウンロードを開始する前に、コンピューターが次の前提条件を満たしていることを確認します。

   - コンピューターには、オペレーティング システム ディスクに加えて、4 つの独立した同一の論理ハード ドライブ上に、少なくとも 60 GB の空きディスク領域が必要です。
   - [.NET framework 4.6 (またはそれ以降のバージョン)](https://dotnet.microsoft.com/download/dotnet-framework-runtime/net46) がインストールされています。

2. [利用開始のページに移動](https://azure.microsoft.com/overview/azure-stack/try/?v=try)し、ASDK のダウンロード、詳細の入力を行い、 **[送信]** をクリックします。
3. [ASDK 用のデプロイ チェッカー](https://go.microsoft.com/fwlink/?LinkId=828735&clcid=0x409)前提条件チェッカー スクリプトをダウンロードして実行します。 このスタンドアロン スクリプトでは、ASDK のセットアップで実行される前提条件チェックを実行します。 ASDK のより大きなパッケージをダウンロードする前に、ハードウェアとソフトウェアの要件を満たしているかどうかを確認できます。
4. **[Download the software]\(ソフトウェアをダウンロード\)** の **[Azure Stack Development Kit]** をクリックします。

   > [!NOTE]
   > ASDK ダウンロード (AzureStackDevelopmentKit.exe) は、約 10 GB です。

## <a name="extract-the-asdk"></a>ASDK の抽出
1. ダウンロードが完了したら、 **[実行]** をクリックして ASDK 自己展開ツール (AzureStackDevelopmentKit.exe) を起動します。
2. 自己展開ツール ウィザードの **[License Agreement]\(使用許諾契約書\)** ページで表示される使用許諾契約書を読んで同意してから、 **[次へ]** をクリックします。
3. 自己展開ツール ウィザードの **[重要な注意]** ページに表示されるプライバシーに関する声明を確認し、 **[次へ]** をクリックします。
4. 自己展開ツール ウィザードの **[Select Destination Location]\(インストール先の選択\)** ページで、Azure Stack セットアップ ファイルが展開される場所を選択し、 **[次へ]** をクリックします。 既定の場所は、"*現在のフォルダー*"\Azure Stack Development Kit です。 
5. 自己展開ツール ウィザードの **[Ready to Extract]\(展開の準備完了\)** ページで展開先の場所の概要を確認し、 **[Extract]\(展開\)** をクリックして、CloudBuilder.vhdx (約 28 GB) と ThirdPartyLicenses.rtf ファイルを展開します。 このプロセスは完了するまで時間がかかります。
6. CloudBuilder.vhdx ファイルを ASDK ホスト コンピューターの C:\ ドライブのルートにコピーまたは移動します (`C:\CloudBuilder.vhdx`)。

> [!NOTE]
> ファイルの展開後は、ハード ディスクの空き領域を増やすために、.EXE および .BIN ファイルを削除してかまいません。 または、ASDK を再デプロイする必要がある場合にファイルをもう一度ダウンロードしなくて済むように、これらのファイルをバックアップすることもできます。


## <a name="next-steps"></a>次のステップ
[ASDK ホスト コンピューターの準備](asdk-prepare-host.md)