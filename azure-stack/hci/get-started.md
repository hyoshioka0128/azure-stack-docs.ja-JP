---
title: Azure Stack HCI と Windows Admin Center の使用を開始する
description: 既存の Azure Stack HCI クラスターにすばやく接続し、Windows Admin Center を使用してクラスターとストレージのパフォーマンスを監視します。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 09/09/2020
ms.openlocfilehash: 19c04e7e7783403250c6e43de6912d1ca0780b56
ms.sourcegitcommit: 69cfff119ab425d0fbb71e38d1480d051fc91216
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/30/2020
ms.locfileid: "91572723"
---
# <a name="get-started-with-azure-stack-hci-and-windows-admin-center"></a>Azure Stack HCI と Windows Admin Center の使用を開始する

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

このトピックでは、Azure Stack HCI クラスターに接続して、クラスターとストレージのパフォーマンスを監視する手順について説明します。 クラスターをまだ設定していない場合は、[Azure Stack HCI をダウンロード](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)し、手順について「[デプロイの概要](deploy/deployment-overview.md)」を参照してください。

## <a name="install-windows-admin-center"></a>Windows Admin Center をインストールする

Windows Admin Center は、ローカルにデプロイされるブラウザーベースのアプリであり、Azure Stack HCI の管理に使用します。 [Windows Admin Center をインストールする](/windows-server/manage/windows-admin-center/deploy/install)方法として最も簡単なのはローカル管理 PC にインストールすること (デスクトップ モード) ですが、サーバーにインストールすること (サービス モード) もできます。

Windows Admin Center をサーバーにインストールする場合、CredSSP が必要なタスク (クラスター作成や更新プログラムのインストールなど) では、Windows Admin Center サーバー上のゲートウェイ管理者グループのメンバーであるアカウントを使用する必要があります。 詳細については、「[ユーザー アクセス制御とアクセス許可を構成する](/windows-server/manage/windows-admin-center/configure/user-access-control#gateway-access-role-definitions)」の最初の 2 つのセクションを参照してください。

## <a name="add-and-connect-to-an-azure-stack-hci-cluster"></a>Azure Stack HCI クラスターを追加して接続する

Windows Admin Center のインストールが完了したら、メインの概要ページから、管理するクラスターを追加できます。

1. **[すべての接続]** の下にある **[+ 追加]** をクリックします。

    :::image type="content" source="media/get-started/addcluster.png" alt-text="クラスターの追加のスクリーンショット":::

2. Windows Server クラスターの追加を選択します。

    :::image type="content" source="media/get-started/chooseconnectiontype.png" alt-text="クラスターの追加のスクリーンショット":::

3. 管理するクラスターの名前を入力し、 **[追加]** をクリックします。 クラスターが [概要] ページの接続リストに追加されます。

4. **[すべての接続]** の下で、追加したクラスターの名前をクリックします。 Windows Admin Center が**クラスター マネージャー**を開始し、そのクラスターの Windows Admin Center ダッシュボードに直接移動します。

## <a name="monitor-cluster-performance-with-the-windows-admin-center-dashboard"></a>Windows Admin Center ダッシュボードを使用してクラスターのパフォーマンスを監視する

Windows Admin Center ダッシュボードには、サーバー、ドライブ、およびボリュームに関するアラートと正常性情報のほか、CPU、メモリ、およびストレージの使用状況に関する詳細が表示されます。 ダッシュボードの下部には、IOPS や、時間、日、週、月、年ごとの待ち時間などのクラスターのパフォーマンス情報が表示されます。

:::image type="content" source="media/get-started/dashboard.png" alt-text="クラスターの追加のスクリーンショット":::

## <a name="monitor-performance-of-individual-components"></a>個々のコンポーネントのパフォーマンスを監視する

ダッシュボードの左側にある **[ツール]** メニューを使用すると、クラスターの任意のコンポーネントをドリルダウンして、仮想マシン、サーバー、ボリューム、ドライブの概要とインベントリを表示できます。

### <a name="virtual-machines"></a>仮想マシン

クラスターで実行されている仮想マシンの概要を表示するには、左側の **[ツール]** メニューから **[仮想マシン]** をクリックします。

:::image type="content" source="media/get-started/vms-summary.png" alt-text="クラスターの追加のスクリーンショット":::

クラスターで実行されている仮想マシンとその状態、ホスト サーバー、CPU 使用率、メモリ不足、必要なメモリ、割り当て済みメモリ、稼働時間の完全なインベントリについては、ページの上部にある **[インベントリ]** をクリックします。

:::image type="content" source="media/get-started/vms-inventory.png" alt-text="クラスターの追加のスクリーンショット":::

### <a name="servers"></a>サーバー

クラスター内のサーバーの概要を表示するには、左側の **[ツール]** メニューから **[サーバー]** をクリックします。

:::image type="content" source="media/get-started/servers-summary.png" alt-text="クラスターの追加のスクリーンショット":::

状態、稼働時間、製造元、モデル、シリアル番号を含むクラスター内のサーバーの完全なインベントリについては、ページの上部にある **[インベントリ]** をクリックします。

:::image type="content" source="media/get-started/servers-inventory.png" alt-text="クラスターの追加のスクリーンショット":::

### <a name="volumes"></a>ボリューム

クラスター上のボリュームの概要を表示するには、左側の **[ツール]** メニューから **[ボリューム]** をクリックします。

:::image type="content" source="media/get-started/volumes-summary.png" alt-text="クラスターの追加のスクリーンショット":::

状態、ファイル システム、回復性、サイズ、ストレージの使用率、IOPS などを含むクラスター上のボリュームの完全なインベントリについては、ページの上部にある **[インベントリ]** をクリックします。

:::image type="content" source="media/get-started/volumes-inventory.png" alt-text="クラスターの追加のスクリーンショット":::

### <a name="drives"></a>ドライブ

クラスター内のドライブの概要を表示するには、左側の **[ツール]** メニューから **[ドライブ]** をクリックします。

:::image type="content" source="media/get-started/drives-summary.png" alt-text="クラスターの追加のスクリーンショット":::

クラスター内のドライブとそのシリアル番号、状態、モデル、サイズ、種類、使用、場所、サーバー、容量の完全なインベントリについては、ページの上部にある **[インベントリ]** をクリックします。

:::image type="content" source="media/get-started/drives-inventory.png" alt-text="クラスターの追加のスクリーンショット":::

### <a name="virtual-switches"></a>仮想スイッチ

クラスター内の仮想スイッチの設定を表示するには、左側にある **[ツール]** メニューから **[仮想スイッチ]** をクリックし、次に、設定を表示する仮想スイッチの名前をクリックします。 Windows Admin Center には、仮想スイッチと関連付けられているネットワーク アダプターとその IP アドレス、接続状態、リンク速度、MAC アドレスが表示されます。

:::image type="content" source="media/get-started/virtual-switch-settings.png" alt-text="クラスターの追加のスクリーンショット":::

## <a name="add-counters-with-the-performance-monitor-tool"></a>パフォーマンス モニター ツールを使用してカウンターを追加する

Windows、アプリ、またはデバイスのパフォーマンス カウンターをリアルタイムで表示して比較するには、パフォーマンス モニター ツールを使用します。

1. 左側の **[ツール]** メニューから **[パフォーマンス モニター]** を選択します。
2. **[Blank workspace]\(空のワークスペース\)** をクリックして新しいワークスペースを開始するか、 **[Restore previous]\(以前のものを復元\)** をクリックして以前のワークスペースを復元します。
    :::image type="content" source="media/get-started/performance-monitor.png" alt-text="クラスターの追加のスクリーンショット":::
3. 新しいワークスペースを作成する場合は、 **[Add counter]\(カウンターの追加\)** ボタンをクリックし、監視する 1 つ以上のソース サーバーを選択するか、クラスター全体を選択します。
4. 監視するオブジェクトとインスタンス、およびカウンターとグラフの種類を選択して、動的なパフォーマンス情報を表示します。
    :::image type="content" source="media/get-started/example-counter.png" alt-text="クラスターの追加のスクリーンショット":::
5. 上部のメニューから **[保存] > [名前を付けて保存]** を選択して、ワークスペースを保存します。

## <a name="use-azure-monitor-for-monitoring-and-alerts"></a>監視とアラートに Azure Monitor を使用する

[Azure Monitor](manage/azure-monitor.md) を使用して、分析とレポート作成のためにイベントとパフォーマンスのカウンターを収集したり、特定の条件が検出されたときにアクションを実行したり、電子メールで通知を受信したりすることもできます。 Windows Admin Center から Azure に直接接続するには、 **[ツール]** メニューから **[Azure Monitor]** をクリックします。

## <a name="collect-diagnostics-information"></a>診断情報の収集

クラスターの問題をトラブルシューティングするための情報を収集するには、 **[ツール]** メニューから **[診断]** を選択します。 Microsoft サポートに電話した場合、この情報を要求されることがあります。

## <a name="next-steps"></a>次のステップ

Azure Stack HCI クラスターの監視の詳細については、以下も参照してください。

- [記憶域スペース ダイレクトのパフォーマンス履歴](/windows-server/storage/storage-spaces/performance-history)
- [Azure Monitor を使用して Azure Stack HCI を監視する](manage/azure-monitor.md)
