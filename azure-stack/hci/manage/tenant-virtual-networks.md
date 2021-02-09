---
title: テナントの仮想ネットワークを管理する
description: このトピックでは、ソフトウェアによるネットワーク制御 (SDN) のデプロイ後、Windows Admin Center を使用して Hyper-V ネットワーク仮想化 (HNV) 仮想ネットワークを作成、更新、および削除する方法を手順に沿って説明します。
author: AnirbanPaul
ms.author: anpaul
ms.topic: how-to
ms.date: 02/01/2021
ms.openlocfilehash: 6af08615a25ed93f62f526d92ead7511699c8439
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99248684"
---
# <a name="manage-tenant-virtual-networks"></a>テナントの仮想ネットワークを管理する

>適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019、Windows Server 2016

このトピックでは、ソフトウェアによるネットワーク制御 (SDN) のデプロイ後、Windows Admin Center を使用して Hyper-V ネットワーク仮想化 (HNV) 仮想ネットワークを作成、更新、および削除する方法を手順に沿って説明します。

HNV は、各テナント ネットワークが個別のエンティティになるように、テナント ネットワークを切り離すのに役立ちます。 パブリック アクセス ワークロードを構成するか、仮想ネットワーク間でピアリングする場合を除き、各エンティティがクロス接続する可能性はありません。

## <a name="create-a-virtual-network"></a>仮想ネットワークの作成
仮想ネットワークを作成するには、Windows Admin Center で次の手順を実行します。

:::image type="content" source="./media/tenant-virtual-networks/create-virtual-network.png" alt-text="仮想ネットワーク名とアドレス プレフィックスを作成するウィンドウが表示されている Windows Admin Center ホーム画面のスクリーンショット。" lightbox="./media/tenant-virtual-networks/create-virtual-network.png":::

1. Windows Admin Center ホーム画面の **[すべての接続]** で、仮想ネットワークを作成するクラスターを選択します。
1. **[ツール]** の下で **[ネットワーク]** 領域まで下にスクロールし、 **[仮想ネットワーク]** を選択します。
1. **[仮想ネットワーク]** で、 **[インベントリ]** タブを選択し、 **[新規作成]** を選択します。
1. **[仮想ネットワーク]** ウィンドウで、仮想ネットワークの名前を入力します。
1. **[アドレス プレフィックス]** で、 **[追加]** を選択し、アドレス プレフィックスをクラスレス ドメイン間ルーティング (CIDR) 表記で入力します。 必要に応じて、さらにアドレス プレフィックスを追加することもできます。
1. **[サブネット]** で、 **[追加]** を選択し、サブネットの名前を入力して、アドレス プレフィックスを CIDR 表記で指定します。

   >[!NOTE]
   > サブネット アドレス プレフィックスは、仮想ネットワークの **[アドレス プレフィックス]** で定義したアドレス プレフィックスの範囲内である必要があります。

1. **[送信]** を選択するか、必要に応じて、さらにサブネットを追加し、 **[送信]** を選択します。
1. **[仮想ネットワーク]** の一覧で、仮想ネットワークの状態が **正常** であることを確認します。

## <a name="get-a-list-of-virtual-networks"></a>仮想ネットワークの一覧を取得する
クラスター内のすべての仮想ネットワークを簡単にご確認いただけます。

:::image type="content" source="./media/tenant-virtual-networks/list-virtual-networks.png" alt-text="仮想ネットワークの一覧が表示されている Windows Admin Center のスクリーンショット。" lightbox="./media/tenant-virtual-networks/list-virtual-networks.png":::

1. Windows Admin Center ホーム画面の **[すべての接続]** で、仮想ネットワークを表示するクラスターを選択します。
1. **[ツール]** の下で **[ネットワーク]** 領域まで下にスクロールし、 **[仮想ネットワーク]** を選択します。
1. **[インベントリ]** タブには、クラスターで使用可能なすべての仮想ネットワークが一覧表示され、個々の仮想ネットワークを管理するためのコマンドが用意されています。 次のようにすることができます。
    - 仮想ネットワークの一覧を表示する。
    - 仮想ネットワークの設定、各仮想ネットワークの状態、および各仮想ネットワークに接続されている仮想マシン (VM) の数を表示します。
    - 仮想ネットワークの設定を変更する。
    - 仮想ネットワークを削除します。

## <a name="view-virtual-network-details"></a>仮想ネットワークの詳細を表示する
特定の仮想ネットワークの詳細情報を、その専用ページから表示できます。

:::image type="content" source="./media/tenant-virtual-networks/view-virtual-network-details.png" alt-text="仮想ネットワークの詳細ビューが表示されている Windows Admin Center のスクリーンショット。" lightbox="./media/tenant-virtual-networks/view-virtual-network-details.png":::

1. **[ツール]** の下で **[ネットワーク]** 領域まで下にスクロールし、 **[仮想ネットワーク]** を選択します。
1. **[インベントリ]** タブを選択し、詳細を表示する仮想ネットワークを選択します。 後続のページでは次を実行できます。
    - 仮想ネットワークの **プロビジョニングの状態** (成功、失敗) を表示する。
    - 仮想ネットワークの **構成の状態** (正常、エラー、警告、不明) を表示する。
    - 仮想ネットワークの基になる **論理ネットワーク** を表示する。
    - 仮想ネットワークの **アドレス空間** を表示する。
    - 新しいサブネットを追加する、既存のサブネットを削除する、および仮想ネットワーク サブネットの設定を変更する。
    - 仮想ネットワークへの **仮想マシンの接続** を表示する。

## <a name="change-virtual-network-settings"></a>仮想ネットワークの設定を変更する
仮想ネットワークのアドレス プレフィックスの更新、仮想ネットワーク ピアリングの管理、および仮想ネットワークの Border Gateway Protocol (BGP) ルーターとピアの構成を行うことができます。

:::image type="content" source="./media/tenant-virtual-networks/change-virtual-network-settings.png" alt-text="仮想ネットワークの設定ビューが表示されている Windows Admin Center のスクリーンショット。" lightbox="./media/tenant-virtual-networks/change-virtual-network-settings.png":::

1. **[ツール]** の下で **[ネットワーク]** 領域まで下にスクロールし、 **[仮想ネットワーク]** を選択します。
1. **[インベントリ]** タブを選択し、仮想ネットワークを選択して、 **[設定]** を選択します。
1. **[全般]** タブでは次を実行できます。
    - 既存のアドレス プレフィックスを削除するか、新しいものを追加する。
    - 他の仮想ネットワークとのピアリングを構成する。
    - 仮想ネットワークに BGP ルーターを追加する。 これを行うには、BGP ルーター名と自律システム番号 (ASN) 番号を指定する必要があります。
    - BGP ルーターに対して 1 つまたは複数の BGP ピアを追加する。 これを行うには、各 BGP ピア名と、各 BGP ピアの ASN 番号を指定する必要があります。

## <a name="delete-a-virtual-network"></a>仮想ネットワークの削除
不要になった仮想ネットワークは削除できます。

:::image type="content" source="./media/tenant-virtual-networks/delete-virtual-network.png" alt-text="仮想ネットワーク削除の確認プロンプトが表示されている Windows Admin Center のスクリーンショット。" lightbox="./media/tenant-virtual-networks/delete-virtual-network.png":::

1. **[ツール]** の下で **[ネットワーク]** 領域まで下にスクロールし、 **[仮想ネットワーク]** を選択します。
1. **[インベントリ]** タブを選択し、仮想ネットワークを選択して、 **[削除]** を選択します。
1. **[仮想ネットワークの削除]** 確認プロンプトで、 **[削除]** を選択します。
1. [仮想ネットワーク] 検索ボックスの横にある **[更新]** を選択して、仮想ネットワークが削除されていることを確認します。

## <a name="next-steps"></a>次のステップ
詳細については、次のトピックも参照してください。
- [Azure Stack HCI におけるソフトウェアによるネットワーク制御 (SDN)](../concepts/software-defined-networking.md)
