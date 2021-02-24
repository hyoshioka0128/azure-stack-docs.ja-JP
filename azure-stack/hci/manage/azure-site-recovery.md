---
title: Azure Site Recovery を使用して Azure Stack HCI VM を保護する
description: Windows Admin Center を使用して、Azure Site Recovery で Azure Stack HCI VM を保護します。
ms.topic: how-to
author: davannaw-msft
ms.author: dawhite
ms.date: 04/30/2020
ms.localizationpriority: low
ms.openlocfilehash: d66948092d97ac42ef9484d47e9b584ed6a80a90
ms.sourcegitcommit: a15a0f955bac922cebb7bf90a72384fd84ddfe56
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/22/2020
ms.locfileid: "86947573"
---
# <a name="protect-azure-stack-hci-vms-using-azure-site-recovery"></a>Azure Site Recovery を使用して Azure Stack HCI VM を保護する

>適用先:Windows Server 2019

Windows Admin Center は、サーバーまたはクラスター上の仮想マシン (VM) をレプリケートするプロセスを効率化し、お客様自身のデータセンターから Azure を活用できるようにします。 セットアップを自動化するには、Windows Admin Center を Azure に接続します。

この記事では、Azure Site Recovery を使用してレプリケーションの設定を構成し、Azure portal で復旧計画を作成する方法について説明します。 これらのタスクを完了すると、Windows Admin Center で VM レプリケーションを開始して VM を保護できます。 Azure Site Recovery は、お客様の物理サーバーとクラスター ノードも保護します。

詳細については、「[Azure ハイブリッド サービスへの Windows Server の接続](/windows-server/manage/windows-admin-center/azure/)」を参照してください。

## <a name="how-azure-site-recovery-works-with-windows-admin-center"></a>Windows Admin Center での Azure Site Recovery のしくみ
*Azure Site Recovery* は、ビジネスに不可欠なインフラストラクチャが障害から保護されるように、VM 上で実行されているワークロードをレプリケートする Azure サービスです。 詳細については、「[Site Recovery について](/azure/site-recovery/site-recovery-overview)」を参照してください。

Azure Site Recovery は、"*レプリケーション*" と "*フェールオーバー*" という 2 つのコンポーネントで構成されています。 レプリケーションの部分では、ターゲット VM の VHD を Azure ストレージ アカウントにレプリケートすることによって、VM を障害から保護します。 その後、障害が発生した場合、Azure に VM をフェールオーバーして実行できます。 また、Azure で復旧プロセスをテストするために、プライマリ VM に影響を与えることなくテスト フェールオーバーを実行することもできます。

障害から VM を保護するには、レプリケーション コンポーネントのセットアップを完了するだけで十分です。 ただし、プロセスのフェールオーバーの部分を構成するまで、Azure で VM を起動することはできません。

フェールオーバーの部分は Azure VM にフェールオーバーする際に設定できるので、初期セットアップ時には必須ではありません。 ホスト サーバーがダウンした場合は、その時点でフェールオーバー コンポーネントを構成し、保護されている VM のワークロードにアクセスできます。 ただし、障害が発生する前にフェールオーバー関連の設定を構成しておくことをお勧めします。

## <a name="prerequisites-and-planning"></a>前提条件と計画
この記事の手順を完了するには、以下が必要です。

- 保護する VM をホストしているターゲット サーバーは、Azure にレプリケートするためにインターネットにアクセスできる必要があります。
- Windows Admin Center から Azure への接続。 詳細については、「[Azure との統合の構成](/windows-server/manage/windows-admin-center/azure/azure-integration)」を参照してください。
- 容量計画ツールを確認して、レプリケーションとフェールオーバーを成功させるための要件を満たしているかを評価します。 詳細については、「[Azure Site Recovery Deployment Planner による Hyper-V から Azure へのディザスター リカバリー](/azure/site-recovery/hyper-v-site-walkthrough-capacity)」を参照してください。

## <a name="step-1-set-up-vm-protection-on-your-target-host"></a>手順 1:ターゲット ホストで VM の保護を設定する
保護対象とする VM が含まれているホスト サーバーまたはクラスターごとに、次の手順を実行します。
1. Windows Admin Center の **[すべての接続]** ページで、保護する VM をホストしているサーバーまたはクラスターに接続します。
1. **[ツール]** で **[仮想マシン]** を選択し、 **[インベントリ]** タブを選択します。
1. 任意の VM を選択します (保護対象の VM である必要はありません)。
1. **[その他]** サブメニューを展開し、 **[Set up VM Protection]\(VM の保護の設定\)** を選択します。

    :::image type="content" source="media/azure-site-recovery/set-up-vm-protection.png" alt-text="Windows Admin Center の [Set up VM Protection]\(VM の保護の設定\) サブメニュー オプション。":::

1. Azure アカウントにサインインします。
1. [Setting up host with Azure Site Recovery]\(Azure Site Recovery でのホストの設定\) ページで、以下の必要な情報を入力し、 **[設定]** を選択します。

   - **サブスクリプション:** このホストの VM レプリケーションに使用する Azure サブスクリプション。
   - **[リソース グループ]:** 新しいリソース グループ名。
   - **Recovery Services コンテナー:** このホストで保護される VM の Azure Site Recovery コンテナーの名前。
   - **[場所]:** Azure Site Recovery リソースを作成する Azure リージョン。

    :::image type="content" source="media/azure-site-recovery/set-up-host-with-asr.png" alt-text="Windows Admin Center の [Setting up host with Azure Site Recovery]\(Azure Site Recovery でのホストの設定\) ページ。":::

1. 次の通知が表示されるまで待ちます: **Site Recovery Setting Completed (Site Recovery の設定が完了しました)** 。

このプロセスには最大 10 分かかることがあります。 進行状況を確認するには、 **[通知]** (Windows Admin Center の右上にあるベル アイコン) に移動します。

>[!NOTE]
> このプロセスでは、Azure Site Recovery エージェントが対象のサーバーまたはノード (クラスターを構成している場合) に自動的にインストールされ、指定した**場所**に、指定した**ストレージ アカウント**と**コンテナー**を含む**リソース グループ**が作成されます。 また、ターゲット ホストが Azure Site Recovery サービスに登録され、既定のレプリケーション ポリシーが構成されます。

## <a name="step-2-select-vms-to-protect"></a>手順 2:保護する VM を選択する
VM を保護するには、次の手順を実行します。
1. Windows Admin Center で、前の手順で構成したサーバーまたはクラスターに戻ります。
1. **[ツール]** で **[仮想マシン]** を選択し、 **[インベントリ]** タブを選択します。
1. 保護する VM を選択し、 **[その他]** サブメニューを展開して **[VM の保護]** を選択します。

    :::image type="content" source="media/azure-site-recovery/protect-vm-setting.png" alt-text="Windows Admin Center の [VM の保護] 設定オプション。":::

1. VM を保護するための容量要件を確認します。 詳細については、「[Hyper-V VM のディザスター リカバリーの容量を計画する](/azure/site-recovery/site-recovery-capacity-planner)」を参照してください。

    Premium ストレージ アカウントを使用する場合は、Azure portal で作成します。 詳細については、「[Azure で利用できるディスクの種類](/azure/storage/common/storage-premium-storage)」の「**Premium SSD**」セクションを参照してください。 Windows Admin Center に用意されている **[新規作成]** オプションを選択すると、Standard ストレージ アカウントが作成されます。

1. この VM のレプリケーションに使用する**ストレージ アカウント**の名前を入力し、 **[VM の保護]** を選択すると、VM のレプリケーションが有効になります。

    :::image type="content" source="media/azure-site-recovery/protect-vm-setting-asr.png" alt-text="Windows Admin Center で VM を保護する Azure Site Recovery のストレージ アカウントを定義する。":::

    Azure Site Recovery で、レプリケーション プロセスが開始されます。 **[仮想マシンのインベントリ]** グリッドの **[Protected]\(保護済み\)** 列の値が **[はい]** に変更されると、VM が保護されています。 このプロセスには数分かかることがあります。

## <a name="step-3-configure-and-run-a-test-failover-in-the-azure-portal"></a>手順 3:Azure portal でテスト フェールオーバーを構成して実行する
VM レプリケーションを開始する前に、この手順を完了する必要はありません。 VM は、レプリケーションのみで保護されます。 ただし、Azure Site Recovery を設定するときに、フェールオーバーの設定を構成しておくことをお勧めします。

Azure VM へのフェールオーバーを準備するには、次の手順を実行します。
1. フェールオーバーされた VM がこの VNET に接続する Azure ネットワークを設定します。 詳細については、「[Azure にオンプレミス Hyper-V VM のディザスター リカバリーを設定する](/azure/site-recovery/hyper-v-site-walkthrough-prepare-azure)」を参照してください。

    >[!NOTE]
    > Windows Admin Center によって、このリソースでの手順が自動的に実行されます。 必要なのは、Azure ネットワークの設定のみです。

1. テスト フェールオーバーを実行します。 詳細については、「[Azure へのディザスター リカバリー訓練を実行する](/azure/site-recovery/hyper-v-site-walkthrough-test-failover)」を参照してください。

## <a name="step-4-create-recovery-plans"></a>手順 4:復旧計画の作成
Azure Site Recovery の復旧計画を使用すると、アプリケーションの VM コレクション全体をフェールオーバーおよび復旧できます。 保護されている VM を個別に復旧することも可能です。 ただし、アプリケーションの VM を復旧計画に追加することをお勧めします。 こうすることで、復旧計画を通じてアプリケーション全体をフェールオーバーできます。 また、復旧計画のテスト フェールオーバー機能を使用して、アプリケーションの復旧をテストすることもできます。

復旧計画を使用すると、VM のグループ化、フェールオーバーを開始する順序の指定、追加の復旧手順の自動化が可能になります。 VM を保護したら、Azure portal の Azure Site Recovery コンテナーに移動して、VM の復旧計画を作成できます。 詳細については、「[復旧計画を作成してカスタマイズする](/azure/site-recovery/site-recovery-create-recovery-plans)」を参照してください。

## <a name="step-5-monitor-replicated-vms-in-azure"></a>手順 5:レプリケートされた VM を Azure で監視する
サーバーの登録プロセスでエラーが発生していないことを確認するには、**Azure portal** に移動して **[すべてのリソース]** を選択し、 **[Recovery Services コンテナー]** 、 **[ジョブ]** 、 **[Site Recovery ジョブ]** の順に選択します。 **[Recovery Services コンテナー]** の名前は、この記事の最初のタスクの手順 6. で指定したものです。

VM のレプリケーションを監視するには、 **[Recovery Services コンテナー]** 、 **[レプリケートされたアイテム]** の順に移動します。

コンテナーに登録されているすべてのサーバーを表示するには、 **[Recovery Services コンテナー]** 、 **[Site Recovery インフラストラクチャ]** 、 **[Hyper-V hosts]\(Hyper-V ホスト\)** ([Hyper-V サイト] セクションの下) の順に移動します。

## <a name="known-issue"></a>既知の問題 ##
Azure Site Recovery にクラスターを登録している場合に、ノードで Azure Site Recovery をインストールできなかったり、Azure Site Recovery サービスを登録できなかったりすると、VM が保護されない可能性があります。 クラスター内のすべてのノードが登録されていることを Azure portal で確認するには、 **[Recovery Services コンテナー]** 、 **[ジョブ]** 、 **[Site Recovery ジョブ]** の順に移動します。

## <a name="next-steps"></a>次のステップ
Azure Site Recovery の詳細については、次の記事も参照してください。

- [Azure Site Recovery に関する一般的な質問](/azure/site-recovery/site-recovery-faq)
