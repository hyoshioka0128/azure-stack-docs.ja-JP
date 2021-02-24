---
title: テナントの論理ネットワークを管理する
description: このトピックでは、ネットワーク コントローラーのデプロイ後、Windows Admin Center を使用して論理ネットワークを作成、更新、および削除する方法を手順に沿って説明します。
author: AnirbanPaul
ms.author: anpaul
ms.topic: how-to
ms.date: 02/02/2021
ms.openlocfilehash: 5cf88e5befd551eb7789388807c9c1e4df671dc9
ms.sourcegitcommit: 34babe5abf1bceee718011b5c5c25f75e1b03b0c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/17/2021
ms.locfileid: "100562951"
---
# <a name="manage-tenant-logical-networks"></a>テナントの論理ネットワークを管理する

>適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019、Windows Server 2016

このトピックでは、ネットワーク コントローラーのデプロイ後、Windows Admin Center を使用して論理ネットワークを作成、更新、および削除する方法を手順に沿って説明します。 ソフトウェアによるネットワーク制御 (SDN) 論理ネットワークは、従来の VLAN ベースのネットワークです。

VLAN ベースのネットワークを SDN 論理ネットワークとしてモデル化することで、これらのネットワークに接続されているワークロードにネットワーク ポリシーを適用することができます。 たとえば、SDN 論理ネットワークに接続されているワークロードにセキュリティ アクセス制御リスト (ACL) を適用できます。 ACL を適用することで、 ご自身の VLAN ベースのワークロードが外部と内部の両方の攻撃から保護されます。

## <a name="create-a-logical-network"></a>論理ネットワークの作成
論理ネットワークを作成するには、Windows Admin Center で次の手順を実行します。

:::image type="content" source="./media/tenant-logical-networks/create-logical-network.png" alt-text="論理ネットワークの名前のボックスが表示されている Windows Admin Center ホーム画面のスクリーンショット。" lightbox="./media/tenant-logical-networks/create-logical-network.png":::

1. Windows Admin Center ホーム画面の **[すべての接続]** で、論理ネットワークを作成するクラスターを選択します。
1. **[ツール]** の下で **[ネットワーク]** 領域まで下にスクロールし、 **[論理ネットワーク]** を選択します。
1. **[論理ネットワーク]** で、 **[インベントリ]** タブを選択し、 **[新規作成]** を選択します。
1. **[論理ネットワーク]** ウィンドウで、論理ネットワークの名前を入力します。
1. **[論理サブネット]** で **[追加]** を選択します。

    :::image type="content" source="./media/tenant-logical-networks/create-logical-network-subnet.png" alt-text="論理サブネット ウィンドウが表示されている Windows Admin Center ホーム画面のスクリーンショット。" lightbox="./media/tenant-logical-networks/create-logical-network-subnet.png":::

1. **[論理サブネット]** ウィンドウで、サブネットの名前を入力し、次の情報を指定します。
    1. ネットワークの **VLAN ID**。
    1. クラスレス ドメイン間ルーティング (CIDR) 表記の **アドレス プレフィックス**。
    1. ネットワークの **既定のゲートウェイ**。
    1. **DNS** サーバー アドレス (必要な場合)。
    1. 外部クライアントへの接続を提供するための論理ネットワークの場合は、 **[Public Logical network]\(パブリック論理ネットワーク\)** チェックボックスをオンにします。
1. **[Logical subnet IP Pools]\(論理サブネット IP プール\)** で、 **[追加]** を選択し、次の情報を指定します。
    1. 論理 **IP プール名**。
    1. **開始 IP アドレス**。
    1. **終了 IP アドレス**。 開始 IP アドレスと終了 IP アドレスは、サブネットに対して指定されたアドレス プレフィックスの範囲内である必要があります。
    1. **[追加]** を選択します。
1. **[論理サブネット]** ページで **[追加]** を選択します。
1. **[論理ネットワーク]** ページで、 **[送信]** を選択します。
1. **[論理ネットワーク]** の一覧で、論理ネットワークの状態が **正常** であることを確認します。

## <a name="get-a-list-of-logical-networks"></a>論理ネットワークの一覧を取得する
クラスター内のすべての論理ネットワークを簡単にご確認いただけます。

:::image type="content" source="./media/tenant-logical-networks/list-logical-networks.png" alt-text="論理ネットワークのインベントリ ウィンドウが表示されている Windows Admin Center ホーム画面のスクリーンショット。" lightbox="./media/tenant-logical-networks/list-logical-networks.png":::

1. Windows Admin Center ホーム画面の **[すべての接続]** で、論理ネットワークを表示するクラスターを選択します。
1. **[ツール]** の下で **[ネットワーク]** 領域まで下にスクロールし、 **[論理ネットワーク]** を選択します。
1. **[インベントリ]** タブには、クラスターで使用可能なすべての論理ネットワークが一覧表示され、個々の論理ネットワークを管理するためのコマンドが用意されています。 次のようにすることができます。
    - 論理ネットワークの一覧を表示する。
    - 論理ネットワークの設定、各論理ネットワークの状態、および各論理ネットワークのネットワーク仮想化が有効になっているかどうかを表示する。 ネットワーク仮想化が有効になっている場合は、各論理ネットワークに関連付けられている仮想ネットワークの数を表示することもできます。
    - 論理ネットワークの設定を変更する。
    - 論理ネットワークを削除する。

## <a name="view-logical-network-details"></a>論理ネットワークの詳細を表示する
特定の論理ネットワークの詳細情報を、その専用ページから表示できます。

:::image type="content" source="./media/tenant-logical-networks/view-logical-network-details.png" alt-text="論理ネットワークの詳細ビューが表示されている Windows Admin Center のスクリーンショット。" lightbox="./media/tenant-logical-networks/view-logical-network-details.png":::

1. **[ツール]** の下で **[ネットワーク]** 領域まで下にスクロールし、 **[論理ネットワーク]** を選択します。
1. **[インベントリ]** タブを選択し、詳細を表示する論理ネットワークを選択します。 後続のページでは次を実行できます。
    - 論理ネットワークのプロビジョニングの状態 (成功、失敗) を表示する。
    - 論理ネットワークに対してネットワーク仮想化が有効になっているかどうかを表示する。
    - 論理ネットワーク内のサブネットを表示する。
    - 新しいサブネットを追加する、既存のサブネットを削除する、および論理ネットワーク サブネットの設定を変更する。
    - 各サブネットを選択して、その **[サブネット]** ページに移動する。このページでは、論理サブネット IP プールを追加、削除、および変更できます。
    - 論理ネットワークに関連付けられている仮想ネットワークおよび接続を表示する。

## <a name="change-a-logical-networks-virtualization-setting"></a>論理ネットワークの仮想化の設定を変更する
論理ネットワークのネットワーク仮想化の設定は変更することができます。

:::image type="content" source="./media/tenant-logical-networks/change-logical-network-setting.png" alt-text="論理ネットワークの [Enable network virtualization]\(ネットワーク仮想化を有効にする\) チェックボックスが表示されている Windows Admin Center のスクリーンショット。" lightbox="./media/tenant-logical-networks/change-logical-network-setting.png":::

1. **[ツール]** の下で **[ネットワーク]** 領域まで下にスクロールし、 **[論理ネットワーク]** を選択します。
1. **[インベントリ]** タブを選択し、論理ネットワークを選択して、 **[設定]** を選択します。
1. 論理ネットワーク上に仮想ネットワークをデプロイする場合は、論理ネットワーク名の下で **[Enable network virtualization]\(ネットワーク仮想化を有効にする\)** チェックボックスをオンにし、 **[送信]** を選択します。

## <a name="delete-a-logical-network"></a>論理ネットワークを削除する
不要になった論理ネットワークは削除できます。

:::image type="content" source="./media/tenant-logical-networks/delete-logical-network.png" alt-text="論理ネットワークを削除するための削除の確認プロンプトが表示されている Windows Admin Center のスクリーンショット。" lightbox="./media/tenant-logical-networks/delete-logical-network.png":::

1. **[ツール]** の下で **[ネットワーク]** 領域まで下にスクロールし、 **[論理ネットワーク]** を選択します。
1. **[インベントリ]** タブを選択し、論理ネットワークを選択して、 **[削除]** を選択します。
1. **削除** の確認プロンプトで **[はい]** を選択します。
1. **[論理ネットワーク]** 検索ボックスの横にある **[更新]** を選択して、論理ネットワークが削除されていることを確認します。

## <a name="next-steps"></a>次のステップ
詳細については、次のトピックも参照してください。
- [テナントの仮想ネットワークを管理する](tenant-virtual-networks.md)
- [Azure Stack HCI におけるソフトウェアによるネットワーク制御 (SDN)](../concepts/software-defined-networking.md)
