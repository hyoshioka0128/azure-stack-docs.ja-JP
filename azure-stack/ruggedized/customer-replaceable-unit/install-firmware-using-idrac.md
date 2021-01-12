---
title: iDRAC を使用してファームウェアをインストールする
description: iDRAC を使用してファームウェアをインストールする方法について説明します
author: PatAltimore
ms.topic: how-to
ms.date: 11/13/2020
ms.author: patricka
ms.reviewer: ''
ms.lastreviewed: ''
ms.openlocfilehash: 3b6a257d5f5f42c951313c2220559306577de15d
ms.sourcegitcommit: d719f148005e904fa426a001a687e80730c91fda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/06/2021
ms.locfileid: "97910246"
---
# <a name="installing-firmware-using-the-idrac-interface"></a>iDRAC インターフェイスを使用してファームウェアをインストールする

統合されている Dell Remote Access Card (iDRAC) を使用すると、 **[Update and Rollback]\(更新およびロールバック\)** 機能で、ファームウェアを (1 つずつ) リモートから更新できます。 これは、サーバーを実行しているときも機能します。

サポートされているファームウェアのバージョンの一覧については、「[Integrated System for Microsoft Azure Stack Hub 14G](https://www.dell.com/support/home/product-support/product/cloud-for-microsoft-azure-stack14g/docs)」のページから最新のサポート マトリックスをダウンロードしてください。

1 つのデバイスのファームウェアを iDRAC Web インターフェイスを使用して更新するには、次の手順に従います。

**手順**

1.  **[Maintenance]\(メンテナンス\)** \*[*System Update**]\(システム更新\) に移動します。 **[Firmware Update]\(ファームウェアの更新\)** ページが表示されます。

    ![[Integrated Dell Remote Access Controller 9 | Enterprise] の [Maintenance]\(メンテナンス\) ページを示すスクリーンショット。](media/image-85.png)

2.  **[Update]\(更新\)** タブで **[ローカル]** をファイルの場所として選択します。

3.  **[Browse]\(参照\)** 、必要なコンポーネント用のファームウェア イメージ、 **[アップロード]** を順に選択します。

4.  アップロードが完了すると、 **[Update Details]\(更新の詳細\)** セクションに、iDRAC にアップロードされた各ファームウェア ファイルとその状態が表示されます。 ファームウェア イメージ ファイルが有効で、正常にアップロードされた場合、 **[Contents]\(コンテンツ\)** 列には、ファームウェア イメージ ファイル名の横に (+) アイコンが表示されます。 名前を展開し、 **[Device Name]\(デバイス名\)** 、 **[Current]\(現在\)** 、 **[Available firmware version]\(使用可能なファームウェア バージョン\)** 情報を参照します。

5.  必要なファームウェア ファイルを選択し、次のいずれかを行います。

    -   ホスト システムの再起動が不要なファームウェア イメージの場合は、 **[インストール]** を選択します。

    -   ホスト システムの再起動が必要なファームウェア イメージの場合は、 **[Install and Reboot]\(インストールおよび再起動\)** または **[Install Next Reboot]\(インストール後再起動\)** を選択します。

    -   ファームウェアの更新を取り消すには、 **[キャンセル]** を選択します。

6.  **[Job Queue]\(ジョブ キュー\)** ページを表示するには、 **[Job Queue]\(ジョブ キュー\)** を選択します。 このページは、ステージングされたファームウェアの更新プログラムの表示および管理に使用できます。または

    **[OK]** を選択すると、現在のページを更新して、ファームウェアの更新プログラムの状態を参照できます。
    
