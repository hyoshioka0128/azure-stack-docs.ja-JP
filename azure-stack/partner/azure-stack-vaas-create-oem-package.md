---
title: OEM 拡張機能パッケージを作成する
titleSuffix: Azure Stack Hub
description: Azure Stack Hub でOEM 拡張機能パッケージを作成する方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 12/16/2020
ms.author: mabrigg
ms.reviewer: johnhas
ms.lastreviewed: 10/28/2019
ROBOTS: NOINDEX
ms.openlocfilehash: dcc1bb0ce68b0c18363bc1caa196d084a54a6c0b
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97874695"
---
# <a name="create-an-oem-package"></a>OEM パッケージの作成

[!INCLUDE [Azure_Stack_Partner](./includes/azure-stack-partner-appliesto.md)]

Azure Stack Hub OEM 拡張機能パッケージは、OEM 固有のコンテンツを Azure Stack Hub インフラストラクチャに追加するメカニズムです。 このコンテンツは、更新、拡張、フィールドの置換などのデプロイと運用のプロセスで使用されます。

## <a name="creating-the-package"></a>パッケージの作成

OEM 拡張機能パッケージを作成して検証したら、VaaS で使用できます。 続行する前に、[OEM パッケージの作成](https://microsoft.sharepoint.com/:w:/r/teams/cloudsolutions/Sacramento/_layouts/15/Doc.aspx?sourcedoc=%7BD7406069-7661-419C-B3B1-B6A727AB3972%7D&file=Azure%20Stack%20OEM%20Extension%20Package.docx&action=default&mobileredirect=true)の手順を完了していることを確認してください。 その後、パッケージは VaaS のテスト結果と共に Microsoft に提出され、パッケージ検証ワークフローで署名されます。 次の手順は、生成されたファイルを、VaaS で使用できる単一の ZIP ファイルにバンドルする方法の詳細を示しています。

1. パッケージの次のコンテンツを確認します。
    - パッケージのコンテンツが含まれている ZIP ファイル。
    - `oemMetadata.xml` という名前のマニフェスト ファイル。パッケージ コンテンツのルートにある `metadata.xml` ファイルと同じ内容である必要があります。

2. コンテンツ ファイルを選択し、ZIP ファイルを作成します。

    ![OEM 拡張パッケージ作成時の ZIP ファイルのコンテンツ](media/vaas-create-oem-package-1.png) ![OEM 拡張パッケージ作成時に項目のコンテンツを圧縮する](media/vaas-create-oem-package-2.png)

3. ファイルを識別できるように、生成されたファイルの名前をわかりやすい名前に変更します。

## <a name="verifying-the-contents"></a>コンテンツの確認

ZIP ファイルの構造を検証するために、ZIP ファイルを検査し、サブフォルダーがないことを確認します。 TLD には圧縮済みのコンテンツがあります。 有効なパッケージ構造を次に示します。

> [!IMPORTANT]
> コンテンツではなく、親フォルダーを圧縮すると、パッケージの署名が失敗します。

![OEM 拡張パッケージの作成時に適切に圧縮されたパッケージ コンテンツ](media/vaas-create-oem-package-3.png)

これで、ZIP ファイルを VaaS にアップロードし、パッケージ検証ワークフローで Microsoft が署名できるようになりました。

## <a name="next-steps"></a>次のステップ

- [OEM パッケージを検証する](azure-stack-vaas-validate-oem-package.md)
