---
title: Azure Stack Hub の更新プログラムをインストールする
description: Azure Stack Hub の更新プログラムをインストールする方法について学習します。
author: sethmanheim
ms.topic: how-to
ms.date: 07/22/2020
ms.author: sethm
ms.lastreviewed: 09/10/2019
ms.reviewer: sranthar
ms.openlocfilehash: d0222de6db809fcbb73d31ac1ef4298e18895643
ms.sourcegitcommit: 53b0dde60a6435936a5e0cb9e931245f262d637a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/23/2020
ms.locfileid: "91106572"
---
# <a name="install-azure-stack-hub-updates"></a>Azure Stack Hub の更新プログラムをインストールする

更新プログラム パッケージをインストールするには、Azure Stack Hub 管理者ポータルの **[更新]** ブレードを使用します。 この記事では、更新プロセスの更新、監視、およびトラブルシューティングの手順について説明します。 **[更新]** ブレードを使用して、更新情報の表示、更新プログラムのインストール、更新の進行状況の監視、更新履歴の確認、現在の Azure Stack Hub と OEM パッケージ バージョンの表示を行います。

管理者ポータルから更新プログラムを管理したり、ダッシュボードの **[更新プログラム]** セクションを使用して次のことを行ったりすることができます。

- 現在のバージョンなどの重要な情報を表示します。
- 更新プログラムをインストールし、進行状況を監視します。
- 以前にインストールされた更新プログラムの更新履歴を確認します。
- クラウドの現在の OEM パッケージ バージョンを表示します。

## <a name="determine-the-current-version"></a>現在のバージョンの判断

現在のバージョンの Azure Stack Hub は **[更新]** ブレードで表示できます。 開くには:

1. Azure Stack Hub 管理者ポータルを開きます。

2. **[ダッシュボード]** を選択します。 **[更新]** ブレードに現在のバージョンが表示されます。 たとえば、このイメージではバージョンは 1.1903.0.35 です。

    ![既定のダッシュボードの [更新] タイル](./media/azure-stack-update-apply/image1.png)

## <a name="install-updates-and-monitor-progress"></a>更新プログラムのインストールと進行状況の監視

> [!IMPORTANT]
> Azure Stack Hub に更新プログラムを適用する前に、[更新前のチェックリスト](release-notes-checklist.md)の手順をすべて完了し、適用する更新プログラムの種類に適切なメンテナンス期間をスケジュールしていることを確実にしてください。

1. Azure Stack Hub 管理者ポータルを開きます。

2. **[ダッシュボード]** を選択します。 **[Update]\(更新\)** を選択します。

3. インストールする使用可能な更新プログラムを選択します。 **[使用可能]** とマークされた更新プログラムがない場合は、[更新プログラム パッケージを準備します](azure-stack-update-prepare-package.md)。

4. **[今すぐ更新]** を選択します。

    ![Azure Stack Hub で更新を開始する方法を示すスクリーンショット。](./media/azure-stack-update-apply/image2.png)

5. Azure Stack Hub のさまざまなサブシステムを介して更新プロセスが反復処理されるときに詳細な状態を表示できます。 サブシステムの例として、物理ホスト、Service Fabric、インフラストラクチャ仮想マシン、管理者とユーザーの両方のポータルを提供するサービスなどがあります。 更新プロセスを通じて、更新リソース プロバイダーにより更新に関するその他の詳細 (成功した手順数、進行中の数など) が報告されます。

6. 更新実行の詳細ブレードから **[ダウンロードの概要]** を選択して、詳細ログをダウンロードします。

    更新プログラムの監視中に問題が発生する場合は、[特権エンドポイント](./azure-stack-privileged-endpoint.md)を使用して、Azure Stack Hub 更新プログラムの実行の進行状況を監視することができます。 また、特権エンドポイントを使用して、Azure Stack Hub ポータルが利用できなくなった場合に、障害が発生した更新プログラムの実行を、前回成功した手順から再開することもできます。 手順については、「[PowerShell を使用した Azure Stack Hub での更新の監視](azure-stack-update-monitor.md)」を参照してください。

    ![Azure Stack Hub 更新実行の詳細](./media/azure-stack-update-apply/image3.png)

7. 完了したら、更新リソース プロバイダーにより、**成功**の確認が表示され、更新プロセスが終了したことと、それにかかった時間が示されます。 そこから、フィルターを使用して、すべての更新、利用可能な更新、またはインストールされた更新に関する情報を表示できます。

    ![azure-stack-update-apply](./media/azure-stack-update-apply/image4.png)

    更新が失敗した場合、 **[更新]** ブレードで **[要注意]** と報告されます。 **[詳細ログのダウンロード]** オプションを使用して、更新失敗時の大まかな状態を取得します。 Azure Stack Hub のログ収集は、診断とトラブルシューティングで役立ちます。

## <a name="review-update-history"></a>更新履歴を確認する

1. 管理者ポータルを開きます。

2. **[ダッシュボード]** を選択します。 **[Update]\(更新\)** を選択します。

3. **[更新の履歴]** を選択します。

    ![Azure Stack Hub の更新の履歴](./media/azure-stack-update-apply/image7.png)

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub での更新プログラム管理の概要](./azure-stack-updates.md)  
- [Azure Stack Hub サービス ポリシー](./azure-stack-servicing-policy.md)  
