---
title: Azure Stack HCI クラスターを管理する - Windows Admin Center
description: Azure Stack HCI 上のクラスターを Windows Admin Center を使用して管理することについて説明します。
ms.topic: how-to
author: v-dasis
ms.author: v-dasis
ms.reviewer: jgerend
ms.date: 01/22/2021
ms.openlocfilehash: 4b6e7537a1111ed2e38d1783acf7197a4249b2ac
ms.sourcegitcommit: e772df8ac78c86d834a68d1a8be83b7f738019b7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/26/2021
ms.locfileid: "98772104"
---
# <a name="manage-azure-stack-hci-clusters-using-windows-admin-center"></a>Windows Admin Center を使用して Azure Stack HCI クラスターを管理する

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

Windows Admin Center を使用して、Azure Stack HCI クラスターを管理できます。 具体的には、Windows Admin Center のクラスター マネージャー機能を使用して、クラスターを管理します。

## <a name="view-the-cluster-dashboard"></a>クラスター ダッシュボードを表示する

クラスター ダッシュボードには、クラスターの正常性とパフォーマンスに関する情報が表示されます。

:::image type="content" source="media/manage-cluster/cluster-dashboard.png" alt-text="クラスター ダッシュボード画面" lightbox="media/manage-cluster/cluster-dashboard.png":::

この情報を表示するには、 **[すべての接続]** の下のクラスター名を選択し、左側の **[ツール]** の下の **[ダッシュボード]** を選択します。 次の情報を表示できます。

- クラスター イベント アラート
- クラスターに参加しているサーバーの一覧
- クラスターで実行されている仮想マシンの一覧
- クラスターで使用可能なディスク ドライブの一覧
- クラスターで使用可能なボリュームの一覧
- クラスターの合計クラスター CPU 使用率
- クラスターの合計クラスター メモリ使用量
- クラスターの合計クラスター ストレージ使用率
- クラスター入力/出力操作の合計数/秒 (IOPS)
- クラスターの平均待機時間 (ミリ秒)

## <a name="change-storage-settings"></a>ストレージ設定を変更する

サーバー メモリを使用して頻度が高い読み取りをキャッシュし、サーバーごとに使用される最大メモリを指定できます。 詳細については、[Azure Stack HCI のキャッシュの概要](../concepts/cache.md)に関する記事を参照してください。

1. Windows Admin Center で、上部のドロップダウン矢印から **[Cluster Manager]\(クラスター マネージャー\)** を選択します。
1. **[ツール]** で、一番下にある **[設定]** を選択します。
1. **[インメモリ キャッシュ]** を選択し、新しい名前を入力します。

    :::image type="content" source="media/manage-cluster/cluster-settings-memory.png" alt-text="クラスターの [インメモリ キャッシュ] 画面" lightbox="media/manage-cluster/cluster-settings-memory.png":::

1. 記憶域スペース ダイレクトによって使用される記憶域プールの名前を変更できます。 **[記憶域プール]** を選択し、新しい名前を入力します。 これは、ストレッチ クラスターに適用できます。

    :::image type="content" source="media/manage-cluster/cluster-settings-storage-pools.png" alt-text="クラスターの [記憶域プール] 画面" lightbox="media/manage-cluster/cluster-settings-storage-pools.png":::

1. [記憶域スペース ダイレクト] の設定を変更できます。 **[記憶域スペース ダイレクト]** を選択し、次の設定を必要に応じて変更します。

    - **Persistent cache (永続的キャッシュ)** - 永続的キャッシュを有効または無効にします
    - **Cache mode for HDD (HDD のキャッシュ モード)** - HDD ドライブのキャッシュ モードを変更します
    - **Cache mode for SSD (SSD のキャッシュ モード)** - SSD ドライブのキャッシュを変更します

    :::image type="content" source="media/manage-cluster/cluster-settings-storage-spaces-direct.png" alt-text="クラスターの [記憶域スペース ダイレクト] 画面" lightbox="media/manage-cluster/cluster-settings-storage-spaces-direct.png":::

## <a name="change-cluster-settings"></a>クラスター設定を変更する

クラスターに適用できる全般設定がいくつかあります。 ここでは、アクセス ポイント、ノードのシャットダウン動作、トラフィックの暗号化、VM の負荷分散、クラスター監視を設定して管理できます。

1. Windows Admin Center で、上部のドロップダウン矢印から **[Cluster Manager]\(クラスター マネージャー\)** を選択します。
1. **[ツール]** の **[設定]** を選択します。
1. クラスター名を変更するには、 **[アクセス ポイント]** を選択し、新しい名前を入力します。

    :::image type="content" source="media/manage-cluster/cluster-settings-access.png" alt-text="アクティブ - アクティブ ストレッチ クラスターのシナリオ" lightbox="media/manage-cluster/cluster-settings-access.png":::

1. ノードのシャットダウン動作を制御するには、 **[ノードのシャットダウン動作]** を選択し、チェックボックスがオンになっていることを確認します。 これにより、ノードから任意の仮想マシンを先に移動して、ノードのグレースフル シャットダウンが可能になります。

    :::image type="content" source="media/manage-cluster/cluster-settings-shutdown.png" alt-text="クラスターの [ノードのシャットダウン動作] 画面" lightbox="media/manage-cluster/cluster-settings-shutdown.png":::

1. クラスター ノード間でデータを送信するために使用される SMB 接続を暗号化するには、 **[Cluster traffic encryption]\(クラスターのトラフィックの暗号化\)** を選択し、次のドロップダウン ボックスから **[暗号化]** を選択します。

   - **コア トラフィック** - NetFT (クラスター仮想アダプター) を介してポート 3343 で送信されるトラフィックを暗号化します。

   - **Storage traffic (ストレージ トラフィック)** - クラスター共有ボリューム (CSV) とストレージ バス層 (SBL) のトラフィックを暗号化します

        :::image type="content" source="media/manage-cluster/cluster-settings-encryption.png" alt-text="クラスターの [Cluster traffic encryption]\(クラスターのトラフィックの暗号化\) 画面" lightbox="media/manage-cluster/cluster-settings-encryption.png":::

1. クラスター全体で仮想マシンの負荷を自動的に分散するには、 **[Virtual machine load balancing]\(仮想マシンの負荷分散\)** を選択し、次の手順を実行します。

   - **[Balance virtual machines]\(仮想マシンのバランスを取る\)** で、適切なアクションを選択します
   - **[強度]** で、適切な動作を選択します。

     このしくみの詳細については、「[仮想マシンの負荷分散の概要](/windows-server/failover-clustering/vm-load-balancing-overview)」を参照してください。

        :::image type="content" source="media/manage-cluster/cluster-settings-vm-load.png" alt-text="クラスターの [Balance virtual machines]\(仮想マシンのバランスを取る\) 画面" lightbox="media/manage-cluster/cluster-settings-vm-load.png":::

1. クォーラム監視の種類を選択するには **[監視]** を選択した後、 **[監視の種類]** で次のいずれかを選択します。

   - **クラウド監視** - Azure クラウド リソースを監視として使用します。
   - **ディスク監視** - ディスク リソースを監視として使用します (ストレッチ クラスターには使用しないでください)
   - **ファイル共有監視** - ファイル共有を監視として使用します。

        監視を設定する方法の詳細については、「[クラスター監視のセットアップ](witness.md)」を参照してください。 「[Azure Stack HCI のクラスターとプールのクォーラムについて](../concepts/quorum.md)」も参照してください。

        :::image type="content" source="media/manage-cluster/cluster-settings-witness.png" alt-text="クラスターの [監視] 画面" lightbox="media/manage-cluster/cluster-settings-witness.png":::

1. アフィニティ ルールを使用してホスト サーバーとサイト間の仮想マシンの配置を制御するには、 **[アフィニティ ルール]** を選択し、 **[ルールの作成 ]** をクリックします。 ルールを設定する方法の詳細については、「[VM のサーバーとサイトのアフィニティ ルールを作成する](vm-affinity.md)」を参照してください。

    :::image type="content" source="media/manage-cluster/affinity-rules.png" alt-text="クラスターの [アフィニティ ルール] 画面" lightbox="media/manage-cluster/affinity-rules.png":::

1. 診断のために Microsoft に送信するデータの量を選択するには、 **[診断データ]** を選択して、次のいずれかを選択します。

    - **Diagnostic data off (Security) (診断データ オフ (セキュリティ))** - データは送信されません
    - **Required (Basic) (必須 (基本))** - セキュリティと最新の状態を維持するために最小限のデータが送信されます
    - **Optional (Full) (オプション (完全))** - 該当するすべてのデータが送信されます

    :::image type="content" source="media/manage-cluster/cluster-diagnostic-data.png" alt-text="クラスターの [データ診断] 画面" lightbox="media/manage-cluster/cluster-diagnostic-data.png":::

## <a name="change-hyper-v-settings"></a>Hyper-V の設定を変更する

クラスターに適用できる Hyper-V ホスト設定はいくつかあります。

1. Windows Admin Center で、上部のドロップダウン矢印から **[Cluster Manager]\(クラスター マネージャー\)** を選択します。
1. **[ツール]** の **[設定]** を選択します。
1. **[全般]** を選択した後、次の設定を使用します。

   - **仮想ハード ディスクのパス** - 仮想ハード ディスクのファイルを格納するための既定のフォルダーを指定します。

   - **仮想マシンのパス** - 仮想マシンの構成ファイルを格納するための既定のフォルダーを指定します。

        :::image type="content" source="media/manage-cluster/cluster-settings-hyperv.png" alt-text="クラスターの Hyper-V の [全般設定] 画面" lightbox="media/manage-cluster/cluster-settings-hyperv.png":::

1. 仮想マシンからローカル デバイスとリソースをリダイレクトできるようにするには、 **[拡張セッション モード]** を選択します。 拡張セッション モード接続では、サポートされているゲスト オペレーティング システムが必要であることに注意してください。

    :::image type="content" source="media/manage-cluster/cluster-settings-session.png" alt-text="クラスターの Hyper-V の [拡張セッション モード] 画面" lightbox="media/manage-cluster/cluster-settings-session.png":::

1. バーチャル マシンが物理 NUMA ノードにまたがることを許可するには、 **[NUMA Spanning]\(NUMA スパニング\)** を選択します。 不均一メモリ アーキテクチャ (NUMA) のスパニングでは、単一の NUMA ノードで使用可能なメモリよりも多くのメモリを持つ仮想マシンを提供できます。

    :::image type="content" source="media/manage-cluster/cluster-settings-numa.png" alt-text="クラスターの [NUMA Spanning]\(NUMA スパニング\) 画面" lightbox="media/manage-cluster/cluster-settings-numa.png":::

1. 実行中に同時に移動 (ライブ マイグレーション) できる VM の数を指定するには、 **[ライブ マイグレーション]** を選択し、数値を選択してから、次のように指定します。

   - **[認証プロトコル]** で、 **[CredSSP]** または **[Kerberos]** を選択します。

   - **[パフォーマンス オプション]** で、 **[圧縮]** または **[SMB]** を選択します。 圧縮データは、TCP/IP 接続を介して送信されます。

   - ノード上の使用可能な任意のネットワークを使用して移行を実行するには、 **[Use any network]\(任意のネットワークを使用する\)** チェックボックスをオンにします。

        :::image type="content" source="media/manage-cluster/cluster-settings-live-migration.png" alt-text="クラスターの [ライブ マイグレーション] 画面" lightbox="media/manage-cluster/cluster-settings-live-migration.png":::

1. 同時に実行できる記憶域の移行の数を指定するには、 **[記憶域の移行]** を選択してから、数値を選択します。

    :::image type="content" source="media/manage-cluster/cluster-settings-storage-migration.png" alt-text="クラスターの [記憶域の移行] 画面" lightbox="media/manage-cluster/cluster-settings-storage-migration.png":::

## <a name="register-the-cluster-with-azure"></a>クラスターを Azure に登録する

Azure でクラスターを登録または登録解除するには、 **[Azure Stack HCI registration]\(Azure Stack HCI の登録)** を選択します。 これを行う方法の詳細については、「[Azure Stack HCI を Azure に接続する](../deploy/register-with-azure.md)」を参照してください。

:::image type="content" source="media/manage-cluster/cluster-registration.png" alt-text="クラスターの [Azure 登録] 画面" lightbox="media/manage-cluster/cluster-registration.png":::

## <a name="next-steps"></a>次のステップ

クラスターを監視するには、「[Azure Monitor を使用して Azure Stack HCI を監視する](azure-monitor.md)」を参照してください。