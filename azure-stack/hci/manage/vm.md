---
title: Windows Admin Center で VM を管理する
description: Windows Admin Center を使用して Azure Stack HCI 上のクラスターで仮想マシンを作成および管理する方法について説明します。
author: v-dasis
ms.topic: how-to
ms.date: 11/06/2020
ms.author: v-dasis
ms.reviewer: JasonGerend
ms.openlocfilehash: 73d705bf5b36509b3aed31afb09105f2da91862f
ms.sourcegitcommit: 08ef9545316798c9a21c2f9bc1da8c15cb648982
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/07/2020
ms.locfileid: "94360361"
---
# <a name="manage-vms-with-windows-admin-center"></a>Windows Admin Center で VM を管理する

> 適用対象: Azure Stack HCI バージョン 20H2、Windows Server 2019

Windows Admin Center を使用して、Azure Stack HCI 上の仮想マシン (VM) を作成および管理できます。

## <a name="create-a-new-vm"></a>新しい仮想マシンを作成する

Windows Admin Center を使用して、新しい VM を簡単に作成できます。

:::image type="content" source="media/manage-vm/new-vm.png" alt-text="新しい VM 画面" lightbox="media/manage-vm/new-vm.png":::

1. Windows Admin Center のホームの **[すべての接続]** で、VM を作成するサーバーまたはクラスターを選択します。
1. **[ツール]** で下方向にスクロールし、 **[仮想マシン]** を選択します。
1. **[仮想マシン]** で、 **[インベントリ]** タブを選択し、 **[追加]** および **[新規]** を選択します。
1. **[新しい仮想マシン]** で、VM の名前を入力します。
1. **[第 2 世代]\(推奨)** を選択します。
1. **[ホスト]** で、VM を配置するサーバーを選択します。
1. **[パス]** で、事前に割り当てられたファイル パスをドロップダウン リストから選択するか、 **[参照]** をクリックして VM 構成と仮想ハード ディスク (VHD) ファイルを保存するフォルダーを選択します。 ネットワーク上にある使用可能な任意の SMB 共有を参照するには、パスを *\\server\share* として入力します。

1. **[仮想プロセッサ]** で、仮想プロセッサの数と、VM で入れ子になった仮想化を有効にするかどうかを選択します。
1. **[メモリ]** で、起動メモリの量 (最小の推奨値は 4 GB) と、必要に応じて、VM に割り当てる動的メモリの最小範囲と最大範囲を選択します。
1. **[ネットワーク]** で、ドロップダウン リストからネットワーク アダプターを選択します。
1. **[ストレージ]** で、 **[追加]** をクリックし、新しい空の仮想ハード ディスクを作成するか、既存の仮想ハード ディスクを使用するかを選択します。 既存の仮想ハード ディスクを使用する場合は、 **[参照]** をクリックして、該当するファイル パスを選択します。  
1. **[オペレーティング システム]** で、次のいずれかの操作を行います。
   - VM が作成された後で VM にオペレーティング システムをインストールする場合は、 **[後でオペレーティング システムをインストールする]** を選択します。
   - **[Install an operating system from an image file (*.iso)]\(イメージ ファイル (*.iso) からオペレーティング システムをインストールする\)** を選択し、 **[参照]** をクリックして、使用する適用可能な .iso イメージ ファイルを選択します。
1. 完了したら、 **[作成]** をクリックして VM を作成します。
1. VM を起動するには、 **[仮想マシン]** の一覧で、新しい VM にマウス ポインターを移動し、左側のチェック ボックスをオンにして、 **[開始]** を選択します。
1. **[状態]** で、VM の状態が **[実行中]** になっていることを確認します。

## <a name="get-a-list-of-vms"></a>VM の一覧の取得

サーバー上またはクラスター内のすべての VM を簡単に確認できます。

:::image type="content" source="media/manage-vm/vm-inventory.png" alt-text="仮想マシンの画面" lightbox="media/manage-vm/vm-inventory.png":::

1. Windows Admin Center で、 **[ツール]** で下方向にスクロールし、 **[仮想マシン]** を選択します。
1. 右側の **[インベントリ]** タブには、現在のサーバーまたはクラスターで使用可能なすべての VM が一覧表示され、個々の VM を管理するためのコマンドが用意されています。 次のようにすることができます。
    - 現在のサーバーまたはクラスターで実行されている VM の一覧を表示します。
    - クラスターの VM を表示している場合、VM の状態とホスト サーバーを表示します。 また、メモリ負荷、メモリの需要と割り当てられたメモリ、VM のアップタイム、ハートビートの状態、保護の状態 (Azure Site Recovery を使用) など、ホストの観点から CPU とメモリの使用率を確認します。
    - 新しい VM を作成します。
    - VM の削除、開始、無効化、シャットダウン、一時停止、再開、リセット、または名前の変更を行います。 VM の保存、保存された状態の削除、またはチェックポイントの作成も行います。
    - VM の設定を変更します。
    - Hyper-V ホストを介して VM コンソールに接続します。
    - Azure Site Recovery を使用して VM をレプリケートします。
    - 複数の VM で実行できる操作 (開始、シャットダウン、保存、一時停止、削除、リセットなど) の場合、複数の VM を選択して一度に操作を実行できます。

## <a name="view-vm-details"></a>VM の詳細を表示する

専用のページから、特定の VM の詳細情報とパフォーマンス グラフを表示できます。

:::image type="content" source="media/manage-vm/vm-details.png" alt-text="仮想マシンの詳細情報画面" lightbox="media/manage-vm/vm-details.png":::

1. **[ツール]** で下方向にスクロールし、 **[仮想マシン]** を選択します。
1. 右側の **[インベントリ]** タブをクリックし、VM を選択します。 後続のページでは、次の操作を実行できます。

   - CPU、メモリ、ネットワーク、IOPS、IO スループットのライブ データと履歴データの折れ線グラフを表示する (履歴データは、ハイパーコンバージド クラスターでのみ使用できます)
   - チェックポイントを表示、作成、適用、名前変更、および削除する。
   - 仮想ハード ディスク (.vhd) ファイル、ネットワーク アダプター、およびホスト サーバーの詳細を表示する。
   - VM の状態を表示する。
   - VM の保存、保存された状態の削除、エクスポート、または VM の複製を行う。
   - VM の設定を変更する。
   - Hyper-V ホスト経由で VMConnect を使用して VM コンソールに接続する。
   - Azure Site Recovery を使用して VM をレプリケートする。

## <a name="view-aggregate-vm-metrics"></a>集計 VM メトリックの表示

クラスター内のすべての VM のリソース使用率とパフォーマンス メトリックを表示できます。

:::image type="content" source="media/manage-vm/host-metrics.png" alt-text="ホスト メトリック画面" lightbox="media/manage-vm/host-metrics.png":::

1. **[ツール]** で下方向にスクロールし、 **[仮想マシン]** を選択します。
1. 右側の **[概要]** タブには、次のような、選択したサーバーまたはクラスターの Hyper-v ホストのリソースとパフォーマンスの総合的なビューが表示されます。
    - 実行中、停止、一時停止、および保存されている VM の数
    - クラスターの最近の正常性アラートまたは Hyper-V イベント ログのイベント
    - ホストとゲストの CPU とメモリの使用率の内訳
    - クラスターの IOPS と I/O スループットのライブ データと履歴データの折れ線グラフ

## <a name="change-vm-settings"></a>VM 設定の変更

VM では、さまざまな設定を変更できます。

> [!NOTE]
> 実行中の VM では一部の設定を変更できないため、最初に VM を停止する必要があります。

1. **[ツール]** で下方向にスクロールし、 **[仮想マシン]** を選択します。
1. 右側の **[インベントリ]** タブをクリックし、VM を選択して、 **[設定]** をクリックします。

1. VM の開始/停止アクションと全般設定を変更するには **[全般]** を選択し、次の操作を行います。
    - VM 名を変更するには、 **[名前]** フィールドに VM 名を入力します。
    - 既定の VM の開始/停止アクションを変更するには、ドロップダウン ボックスから適切な設定を選択します。
    - VM を一時停止または開始するための時間間隔を変更するには、表示されたフィールドに適切な値を入力します。

        :::image type="content" source="media/manage-vm/vm-settings-general.png" alt-text="VM の全般設定画面" lightbox="media/manage-vm/vm-settings-general.png":::

1. VM の起動メモリ、動的メモリの範囲、メモリ バッファーの割合、およびメモリの優先度を変更するには、 **[メモリ]** を選択します。

    :::image type="content" source="media/manage-vm/vm-settings-memory.png" alt-text="VM メモリ設定の変更画面" lightbox="media/manage-vm/vm-settings-memory.png":::

1. 仮想プロセッサの数を変更したり、入れ子になった仮想化を有効にしたり、同時マルチスレッド (SMT) を有効にしたりするには、 **[プロセッサ]** を選択します。

    :::image type="content" source="media/manage-vm/vm-settings-processor.png" alt-text="VM のプロセッサ設定の変更画面" lightbox="media/manage-vm/vm-settings-processor.png":::

1. 既存のディスクのサイズを変更するには、 **[サイズ (GB)]** の値を変更します。 新しい仮想ディスクを追加するには、 **[ディスク]** を選択し、空の仮想ディスクを作成するか、既存の仮想ディスクまたは ISO (.iso) イメージ ファイルを使用するかを選択します。 **[参照]** をクリックして、仮想ディスクまたはイメージ ファイルへのパスを選択します。

    :::image type="content" source="media/manage-vm/vm-settings-disk.png" alt-text="VM ディスク設定の変更画面" lightbox="media/manage-vm/vm-settings-disk.png":::

1. ネットワーク アダプターの設定を追加、削除、または変更するには、 **[ネットワーク]** を選択し、次の操作を行います。
    - 使用する仮想スイッチを指定し、仮想 LAN 識別を有効にするかどうかを指定します (VLAN ID も指定する必要があります)。
    - ネットワーク アダプターの追加の設定を変更する場合は、 **[詳細]** をクリックすると、次のことができます。
        - MAC アドレスの種類として、動的または静的のどちらかを選択する
        - MAC アドレスのスプーフィングを有効にする
        - 帯域幅管理を有効にし、最大/最小範囲を指定する

        :::image type="content" source="media/manage-vm/vm-settings-network.png" alt-text="VM ネットワーク設定の変更画面" lightbox="media/manage-vm/vm-settings-network.png":::

1. ブート デバイスを追加したり、VM のブート シーケンスを変更したりするには、 **[Boot order]\(ブート順序\)** を選択します。

    :::image type="content" source="media/manage-vm/vm-settings-boot.png" alt-text="VM ブート順序の変更画面" lightbox="media/manage-vm/vm-settings-boot.png":::

1. **[チェックポイント]** を選択して、VM チェックポイントを有効にし、チェックポイントの種類を選択して、チェックポイント ファイルの場所を指定します。

    > [!NOTE]
    > **運用** チェックポイント設定が推奨されます。この設定では、ゲスト オペレーティング システムでバックアップ テクノロジを使用して、データ整合性チェックポイントが作成されます。 **標準** 設定では、VHD スナップショットを使用して、アプリケーションとサービスの状態によってチェックポイントが作成されます。

     :::image type="content" source="media/manage-vm/vm-settings-checkpoint.png" alt-text="VM チェックポイントの変更画面" lightbox="media/manage-vm/vm-settings-checkpoint.png":::

1. VM のアフィニティ ルールを作成するには、 **[Affinity rules]\(アフィニティ ルール\)** を選択します。 アフィニティ ルールの作成の詳細については、「[VM のサーバーとサイトのアフィニティ ルールを作成する](vm-affinity.md)」を参照してください。

    :::image type="content" source="media/manage-vm/vm-affinity.png" alt-text="VM のアフィニティ ルールの画面" lightbox="media/manage-vm/vm-affinity.png":::

1. VM のセキュリティ設定を変更するには、 **[セキュリティ]** を選択し、次の手順を実行します。
    - 未承認のコードが起動時に実行されないようにするには、 **[セキュア ブートを有効にする]** を選択します (推奨)。 また、ドロップダウン ボックスから Microsoft またはオープンソースのテンプレートを選択します。
    - **[テンプレート]** で、使用するセキュリティテンプレートを選択します

    - **[暗号化のサポート]** では、以下のことができます。

        - **[トラステッド プラットフォーム モジュールを有効にする]** を選択して、ハードウェア暗号化サービス モジュールを使用できるようにする

        - 状態と仮想マシンの移行トラフィックの暗号化を有効にする

        > [!NOTE]
        > 暗号化のサポートには、VM のキー保護機能 (KP) が必要です。 これらのオプションのいずれかを選択すると、このホストで VM を実行できるようにする KP が生成されます (まだ存在していない場合)。

    - **[セキュリティ ポリシー]** で、VM の追加の保護オプションとして **[シールドを有効にする]** を選択します。

        :::image type="content" source="media/manage-vm/vm-settings-security.png" alt-text="VM のセキュリティ設定の変更" lightbox="media/manage-vm/vm-settings-security.png":::

## <a name="move-a-vm-to-another-server-or-cluster"></a>VM を別のサーバーまたはクラスターに移動する

次のようにして、仮想マシンを別のサーバーまたは別のクラスターに簡単に移動できます。

1. **[ツール]** で下方向にスクロールし、 **[仮想マシン]** を選択します。
1. **[インベントリ]** タブで、一覧から VM を選択し、 **[管理] > [移動]** を選択します。
1. 一覧からサーバーを選択し、 **[移動]** を選択します。
1. VM とそのストレージの両方を移動する場合は、別のクラスターまたは同じクラスター内の別のサーバーのどちらに移動するかを選択します。

    :::image type="content" source="media/manage-vm/vm-more-move.png" alt-text="VM の移動画面" lightbox="media/manage-vm/vm-more-move.png":::

1. VM のストレージのみを移動する場合は、同じパスへの移動を選択するか、または構成、チェックポイント、スマート ページングの別のパスを選択します。

    :::image type="content" source="media/manage-vm/vm-move-storage.png" alt-text="VM ストレージの移動画面" lightbox="media/manage-vm/vm-move-storage.png":::

## <a name="join-a-vm-to-a-domain"></a>VM をドメインに参加させる

次のようにして、VM をドメインに簡単に参加させることができます。

:::image type="content" source="media/manage-vm/vm-domain-join.png" alt-text="VM 移動のドメイン参加画面" lightbox="media/manage-vm/vm-domain-join.png":::

1. **[ツール]** で下方向にスクロールし、 **[仮想マシン]** を選択します。
1. **[インベントリ]** タブで、一覧から VM を選択し、 **[管理] > [ドメインへの参加]** を選択します。
1. 参加するドメインの名前およびドメインのユーザー名とパスワードを入力します。
1. VM のユーザー名とパスワードを入力します。
1. 終わったら、 **[参加]** をクリックします。

## <a name="clone-a-vm"></a>VM を複製する

次のようにして、VM を簡単に複製できます。

1. **[ツール]** で下方向にスクロールし、 **[仮想マシン]** を選択します。
1. 右側の **[インベントリ]** タブを選択します。 一覧から VM を選択し、 **[管理] > [複製]** を選択します。
1. 複製後の VM の名前とパスを指定します。
1. まだ行っていない場合は、VM で Sysprep を実行します。

:::image type="content" source="media/manage-vm/vm-clone.png" alt-text="VM の複製画面" lightbox="media/manage-vm/vm-clone.png":::

## <a name="import-or-export-a-vm"></a>VM をインポートまたはエクスポートする

VM は簡単にインポートまたはエクスポートできます。 次の手順では、インポート プロセスについて説明します。

:::image type="content" source="media/manage-vm/vm-more-import.png" alt-text="VM のインポート画面" lightbox="media/manage-vm/vm-more-import.png":::

1. **[ツール]** で下方向にスクロールし、 **[仮想マシン]** を選択します。
1. **[インベントリ]** タブで、 **[追加] > [インポート]** を選択します。
1. VM が格納されているフォルダー名を入力するか、 **[参照]** をクリックしてフォルダーを選択します。
1. インポートする VM を選択します。
1. 必要に応じて、VM の一意の ID を作成します。
1. 完了したら、 **[インポート]** を選択します。

VM をエクスポートする場合のプロセスは似ています。

1. **[ツール]** で下方向にスクロールし、 **[仮想マシン]** を選択します。
1. **[インベントリ]** タブで、一覧からエクスポートする VM を選択します。
1. **[管理] > [エクスポート]** を選択します。
1. VM のエクスポート先のパスを入力します。

:::image type="content" source="media/manage-vm/vm-export.png" alt-text="VM のエクスポート画面" lightbox="media/manage-vm/vm-export.png":::

## <a name="view-vm-event-logs"></a>VM イベント ログを表示する

VM イベント ログは、次のようにして表示できます。

1. **[ツール]** で下方向にスクロールし、 **[仮想マシン]** を選択します。
1. 右側の **[概要]** タブで、 **[すべてのイベントを表示]** を選択します。
1. イベント カテゴリを選択し、ビューを展開します。

## <a name="connect-to-a-vm-by-using-remote-desktop"></a>リモート デスクトップを使用して VM に接続する

Windows Admin Center を使用する代わりに、リモート デスクトップ プロトコル (RDP) 接続を使用して Hyper-V ホストを介して VM を管理することもできます。

1. **[ツール]** で下方向にスクロールし、 **[仮想マシン]** を選択します。
1. **[インベントリ]** タブで、一覧から [仮想マシンの選択] を選択し、 **[接続] > [接続]** または **[接続] > [RDP ファイルのダウンロード]** オプションを選択します。 どちらのオプションでも、VMConnect ツールを使用して Hyper-V ホストを介してゲスト VM に接続し、Hyper-V ホストの管理者のユーザー名とパスワードの資格情報を入力する必要があります。

    - **[接続]** オプションでは、Web ブラウザーでリモート デスクトップを使用して VM に接続します。

    - **[RDP ファイルのダウンロード]** オプションでは、.rdp ファイルがダウンロードされます。これを開いて、Remote Desktop Connection アプリ (mstsc.exe) に接続できます。

## <a name="protect-vms-with-azure-site-recovery"></a>Azure Site Recovery を使用して VM を保護する

Windows Admin Center を使用して Azure Site Recovery を構成し、オンプレミスの VM を Azure にレプリケートできます。 これはオプションの付加価値サービスです。 開始するには、[Azure Site Recovery を使用した VM の保護](azure-site-recovery.md)に関する記事を参照してください。

:::image type="content" source="media/manage-vm/vm-more-azure.png" alt-text="Azure Site Recovery のセットアップ画面" lightbox="media/manage-vm/vm-more-azure.png":::

## <a name="next-steps"></a>次のステップ

Windows PowerShell を使用して、VM を作成および管理することもできます。 詳しくは、「[Windows PowerShell を使用して Azure Stack HCI 上の VM を管理する](vm-powershell.md)」をご覧ください。