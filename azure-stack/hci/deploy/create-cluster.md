---
title: Windows Admin Center を使用して Azure Stack HCI クラスターを作成する
description: Windows Admin Center を使用して Azure Stack HCI 用のサーバー クラスターを作成する方法について説明します
author: v-dasis
ms.topic: how-to
ms.date: 01/13/2021
ms.author: v-dasis
ms.reviewer: JasonGerend
ms.openlocfilehash: a81b684e86f9d13105c39607f9be1c6a1d56eaf0
ms.sourcegitcommit: 649540e30e1018b409f4b1142bf2cb392c9e8b0d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/14/2021
ms.locfileid: "98208057"
---
# <a name="create-an-azure-stack-hci-cluster-using-windows-admin-center"></a>Windows Admin Center を使用して Azure Stack HCI クラスターを作成する

> 適用対象: Azure Stack HCI バージョン v20H2

この記事では、Windows Admin Center を使用して、記憶域スペース ダイレクトを使用する Azure Stack HCI クラスターを作成する方法について説明します。 Windows Admin Center のクラスターの作成ウィザードでは、面倒な処理のほとんどが自動的に行われます。 PowerShell を使用して自分で行いたい場合は、[PowerShell を使用した Azure Stack HCI クラスターの作成](create-cluster-powershell.md)に関する記事を参照してください。 PowerShell 記事は、ウィザードの内部のしくみを把握したり、トラブルシューティングを行ったりするうえでも参考になります。

2 種類のクラスターのどちらかを選択して作成できます。

- 単一のサイトに存在する、少なくとも 2 つのサーバー ノードを持つ標準クラスター。
- 2 つのサイトにまたがり、少なくとも 4 つのサーバー ノードを持つストレッチ クラスター。サイトごとに少なくとも 2 つのノードがあります。

ストレッチ クラスターの詳細については、「[ストレッチ クラスターの概要](../concepts/stretched-clusters.md)」を参照してください。

Azure Stack HCI のテストには興味があるものの、ハードウェアに限りがある、または予備のハードウェアがない場合には、[Azure Stack HCI の評価ガイド](https://github.com/Azure/AzureStackHCI-EvalGuide/blob/main/README.md)をご覧ください。オンプレミスにある単一の物理システムまたは Azure で、入れ子になった仮想化を使用して Azure Stack HCI を体験することができます。

## <a name="before-you-run-the-wizard"></a>ウィザードを実行する前に

クラスターの作成ウィザードを実行する前に、次のことを済ましてください。

- [システム要件](../concepts/system-requirements.md)に関するページで、ハードウェアと関連する要件を読みます。
- Azure Stack HCI の[物理ネットワークの要件](../concepts/physical-network-requirements.md)と[ホスト ネットワークの要件](../concepts/host-network-requirements.md)に関するページを読みます。
- クラスター内の各サーバーに Azure Stack HCI OS をインストールします。 「[Azure Stack HCI オペレーティング システムのデプロイ](operating-system.md)」を参照してください。
- 各サーバーでローカルの Administrators グループのメンバーであるアカウントを用意します。
- 管理用 PC またはサーバーに Windows Admin Center をインストールします。 「[Windows Admin Center のインストール](/windows-server/manage/windows-admin-center/deploy/install)」を参照してください。
- ストレッチ クラスターの場合は、あらかじめ 2 つのサイトを Active Directory に設定します。 ただし、その設定はウィザードで自動的に行うことができるので、手間はかかりません。

Windows Admin Center を (ローカル PC ではなく) サーバーで実行する場合は、ゲートウェイ管理者グループに属しているアカウント、または Windows Admin Center サーバーのローカルの Administrators グループに属しているアカウントを使用します。

また、Windows Admin Center の管理コンピューターは、クラスターを作成するのと同じ Active Directory ドメイン、または完全に信頼されたドメインに参加している必要があります。 クラスター化するサーバーは、まだドメインに属している必要はありません。クラスターの作成中にドメインに追加することができます。

クラスターの作成ウィザードでの主な手順は次のとおりです。

1. **開始** - 各サーバーがクラスター参加に必要な前提条件および機能を満たしていることを確認します。
1. **ネットワーク** - ネットワーク アダプターを割り当てて構成し、各サーバー用の仮想スイッチを作成します。
1. **クラスタリング** - クラスターが正しくセットアップされていることを検証します。 ストレッチ クラスターの場合は、2 つのサイトもセットアップします。
1. **ストレージ** - 記憶域スペース ダイレクトを構成します。

ウィザードが完了した後、クラスター監視を設定し、Azure に登録し、ボリュームを作成します (また、ストレッチ クラスターを作成する場合は、サイト間のレプリケーションも設定します)。

ウィザードを開始する前に、最新の Windows Admin Center の拡張機能 (特にクラスターの作成拡張機能) がインストールされていることをご確認ください。 これを実行するには、次のようにします。

1. Windows Admin Center を開き、右上にある [設定] (歯車アイコン) をクリックします。
1. **[設定]** で **[拡張機能]** を選択します。
1. **[Cluster Creation]\(クラスターの作成\)** を選択し、 **[インストール]** をクリックします。
1. **[Cluster Manager]\(クラスター マネージャー\)** を選択し **[インストール]** をクリックします。

これで準備ができました。始めましょう。

1. Windows Admin Center で、 **[すべての接続]** の下にある **[追加]** をクリックします。
1. **[Add or create resources]\(リソースの追加または作成\)** パネルの **[Server clusters]\(サーバー クラスター\)** で **[Create new]\(新規作成\)** を選択します。
1. **[1. Choose cluster type]\(1. クラスターの種類の選択\)** で、 **[Azure Stack HCI]** を選択します。

    :::image type="content" source="media/cluster/create-cluster-type.png" alt-text="クラスターの作成ウィザード - HCI オプション" lightbox="media/cluster/create-cluster-type.png":::

1. **[Select server locations]\(サーバーの場所の選択\)** で、次のいずれかを選択します。

    - **All servers in one site (すべてのサーバーが 1 つのサイト内)**
    - **Servers in two sites (2 つのサイトのサーバー)** (ストレッチ クラスターの場合)

1. 完了したら、 **[作成]** をクリックします。 次に示すように、クラスターの作成ウィザードが表示されます。

    :::image type="content" source="media/cluster/create-cluster-wizard.png" alt-text="クラスターの作成ウィザード - はじめに" lightbox="media/cluster/create-cluster-wizard.png":::

## <a name="step-1-get-started"></a>手順 1:はじめに

ウィザードの手順 1 では、すべての前提条件が満たされていることを確認し、サーバー ノードを追加し、必要な機能をインストールしてから、必要に応じて各サーバーを再起動します。

1. ウィザードに一覧表示されている **[1.1 Check the prerequisites]\(1.1 前提条件の確認\)** を見て、各サーバー ノードがクラスター対応であることを確認します。 完了したら、 **[次へ]** をクリックします。
1. **[1.2 Add servers]\(1.2 サーバーの追加\)** で、アカウントのユーザー名とパスワードを入力し、 **[Next]\(次へ\)** をクリックします。 このアカウントは、各サーバーのローカルの Administrators グループのメンバーである必要があります。
1. 追加する最初のサーバーの名前を入力して、 **[追加]** をクリックします。
1. クラスターの一部にする各サーバーについて、ステップ 3 を繰り返します。 完了したら、 **[次へ]** をクリックします。
1. 必要に応じて、 **[1.3 Join a domain]\(1.3 ドメインに参加\)** で、サーバーに参加するドメインと使用するアカウントを指定します。 必要に応じて、サーバーの名前を変更することもできます。 続けて、 **[次へ]** をクリックします。
1. **[1.4 Install features]\(1.4 機能のインストール\)** で機能を確認し、必要に応じて追加します。 完了したら、 **[次へ]** をクリックします。

    ウィザードにより、次の必要な機能がインストールされます。

    - BitLocker
    - データ センター ブリッジング (RoCEv2 ネットワーク アダプター用)
    - フェールオーバー クラスタリング
    - ファイル サーバー
    - FS-Data-Deduplication モジュール
    - Hyper-V
    - RSAT-AD-PowerShell モジュール
    - ストレージ レプリカ (ストレッチ クラスターの場合にインストールされます)

1. **[1.5 Install updates]\(1.5 更新プログラムのインストール\)** で、必要に応じて **[Install updates]\(更新プログラムのインストール\)** をクリックして、オペレーティング システムの更新プログラムをインストールします。 完了したら、 **[次へ]** をクリックします。
1. **[1.6 Install hardware updates]\(1.6 ハードウェア更新プログラムのインストール\)** で、必要に応じて **[更新プログラムの入手]** をクリックして、使用可能なベンダーのハードウェア更新プログラムを入手します。
1. ベンダー固有の手順に従って、ハードウェアに更新プログラムをインストールします。 これらの手順には、更新プログラムが確実に成功するように、ハードウェア上で対称性とコンプライアンスのチェックを実行することも含まれます。 場合によっては、いくつかの手順を再実行する必要があります。
1. 必要に応じて、 **[1.7 Restart servers]\(1.7 サーバーの再起動\)** で **[Restart servers]\(サーバーの再起動\)** をクリックします。 各サーバーが正常に起動されたことを確認します。

## <a name="step-2-networking"></a>手順 2:ネットワーク

ウィザードの手順 2. では、クラスターの仮想スイッチ、ネットワーク アダプター、その他のネットワーク要素を構成します。 RDMA (iWARP と RoCE の両方) ネットワーク アダプターがサポートされています。

Azure Stack HCI の RDMA および Hyper-V ホスト ネットワークの詳細については、[ホスト ネットワークの要件](../concepts/host-network-requirements.md)に関する記事を参照してください。

> [!NOTE]
> ネットワークまたは仮想スイッチの手順でエラーが表示された場合は、 **[Apply and test]\(適用してテスト\)** をもう一度選択してください。

1. **[次へ: ネットワーク]** を選択します。
1. **[2.1 Check network adapters]\(2.1 ネットワーク アダプターの確認\)** で、各アダプターの横に緑色のチェックボックスが表示されるまで待ってから、 **[Next]\(次へ\)** を選択します。

1. **[2.2 Select management adapters]\(2.2 管理アダプターの選択\)** で、各サーバーに使用する 1 つまたは 2 つの管理アダプターを選択します。 ウィザードではクラスター管理専用に少なくとも 1 つの物理 NIC が必要であるため、管理のためのアダプターを少なくとも 1 つは選択することが必須です。  管理用として指定したアダプターは、ウィザードの残りのワークフローから除外されます。

    :::image type="content" source="media/cluster/create-cluster-management-adapters.png" alt-text="クラスターの作成ウィザード - 管理アダプターの選択" lightbox="media/cluster/create-cluster-management-adapters.png":::

    管理アダプターには、次の 2 つの構成オプションがあります。

    - **管理用の物理ネットワーク アダプター 1 つ**。 このオプションの場合、DHCP と静的 IP アドレス割り当ての両方がサポートされます。

    - **チーミングされた、管理用の物理ネットワーク アダプター 2 つ**。 2 つのアダプターがチーミングされている場合、サポートされるのは静的 IP アドレスの割り当てのみです。 選択したアダプター (一方または両方) に DHCP アドレスが使用されている場合、仮想スイッチの作成前に、DHCP IP アドレスが静的 IP アドレスに変換されます。

    チーミングされたアダプターを使用すると、複数のスイッチに対する 1 つの接続ができますが、使用されるのは 1 つの IP アドレスだけです。 負荷分散が可能になり、フォールト トレランスは DNS レコードの更新を待つことなくすぐに行われます。

    次に、それぞれのサーバーについて次の作業を行います。

    - **[説明]** チェック ボックスをオンにします。 すべてのアダプターが選択されていること、およびウィザードによって推奨事項が提示される場合があることに注意してください。
    - クラスター管理に使用しないアダプターのチェックボックスをオフにします。

    > [!NOTE]
    > 1 Gb のアダプターを管理アダプターとして使用することもできますが、ストレージとワークロード (VM) のトラフィックを伝送するには、10 Gb 以上の速さのアダプターを使用することをお勧めします。

1. 変更を行ったら、 **[Apply and test]\(適用してテスト\)** をクリックします。
1. **[Define networks]\(ネットワークの定義\)** で、各サーバーの各ネットワーク アダプターに一意の静的 IP アドレス、サブネット マスク、VLAN ID があることを確認します。 各テーブル要素をポイントし、必要に応じて値を入力または変更します。 終わったら、 **[Apply and test]\(適用してテスト\)** をクリックします。

    > [!NOTE]
    > クラスターで VLAN ID 構成をサポートする場合は、すべてのサーバーのすべてのネットワーク アダプターで VLANID プロパティがサポートされている必要があります。

1. 各サーバーの **[状態]** 列が **[Passed]\(合格\)** になるまで待ち、 **[次へ]** をクリックします。 このステップでは、同じサブネットと VLAN ID を持つすべてのアダプター間のネットワーク接続を検証します。 次のステップで仮想スイッチを作成すると、指定した IP アドレスが物理アダプターから仮想アダプターに転送されます。 構成したアダプターの数によっては、完了するまでに数分かかる場合があります。

1. **[2.3 Virtual switch]\(2.3 仮想スイッチ\)** で、必要に応じて次のいずれかのオプションを選択します。 ネットワーク アダプターの数によっては、一部のオプションを使用できない場合があります。

    - **[Skip virtual switch creation]\(仮想スイッチの作成をスキップする\)** - 後で仮想スイッチを設定するかどうかを選択します。
    - **[Create one virtual switch for compute and storage together]\(コンピューティングとストレージの両方のために仮想スイッチを 1 つ作成する\)** - VM と記憶域スペース ダイレクトに同じ仮想スイッチを使用するかどうかを選択します。 これは "コンバージド" オプションです。
    - **[Create one virtual switch for compute only]\(Hyper-V 専用の仮想スイッチを 1 つ作成する\)** - VM 専用の仮想スイッチを使用する場合に選択します。
    - **[Create two virtual switches]\(2 つの仮想スイッチを作成する\)** - VM 用と記憶域スペース ダイレクト用のそれぞれに専用の仮想スイッチを使用する場合に選択します。

        :::image type="content" source="media/cluster/create-cluster-virtual-switches.png" alt-text="クラスターの作成ウィザード - 仮想スイッチ" lightbox="media/cluster/create-cluster-virtual-switches.png":::

    > [!NOTE]
    > SDN 用のネットワーク コントローラーをデプロイしようとしている場合 (ウィザードの **[手順 5:SDN]** ) は、仮想スイッチが 1 つ必要になります。 そのため、ここで仮想スイッチを作成せず、ウィザードの外部で 1 つ作成しない場合、ウィザードによるネットワーク コントローラーのデプロイは行われません。

    次の表は、各種のネットワーク アダプター構成で、どの仮想スイッチ構成がサポートされ、有効であるかを示したものです。

    | オプション | 1 から 2 個のアダプター | 3 個以上のアダプター | チーミングされたアダプター |
    | :------------- | :--------- |:--------| :---------|
    | 1 つのスイッチ (コンピューティング + ストレージ) | enabled | enabled  | サポート外 |
    | 1 つのスイッチ (コンピューティングのみ) | サポート外| enabled | enabled |
    | 2 つのスイッチ | サポート外 | enabled | enabled |

1. スイッチの名前と他の構成設定を必要に応じて変更し、 **[Apply and test]\(適用してテスト\)** をクリックします。 仮想スイッチが作成された後、各サーバーの **[状態]** 列が **[Passed]\(合格\)** になります。

1. 手順 **[2.4 RDMA]** は省略可能です。 RDMA を使用している場合は、 **[Configure RDMA (Recommended)]\(RDMA の構成 (推奨)\)** チェックボックスをオンにして、 **[Next]\(次へ\)** をクリックします。

    :::image type="content" source="media/cluster/create-cluster-rdma.png" alt-text="クラスターの作成ウィザード - RDMA の構成" lightbox="media/cluster/create-cluster-rdma.png":::

    帯域幅予約の割り当てについては、[ホスト ネットワークの要件](../concepts/host-network-requirements.md)に関する記事の「[トラフィックの帯域幅の割り当て](../concepts/host-network-requirements.md#traffic-bandwidth-allocation)」セクションを参照してください。

1. **[Advanced]\(詳細設定\)** を選択し、 **[Data Center Bridging (DCB)]\(データセンター ブリッジング (DCB)\)** チェックボックスをオンにします。

1. **[Cluster heartbeat]\(クラスターのハートビート\)** で、優先度レベルと帯域幅予約の割合を割り当てます。

1. **[Storage]\(ストレージ\)** で、優先度レベルと帯域幅予約の割合を割り当てます。

1. 完了したら、 **[Apply changes]\(変更の適用\)** を選択し、 **[Next]\(次へ\)** をクリックします。

1. **[2.5 Define networks]\(2.5 ネットワークの定義\)** で、一覧表示されている各アダプターの名前、IP アドレス、サブネット マスク、VLAN ID、既定ゲートウェイのフィールドを確認し、編集します。

    :::image type="content" source="media/cluster/create-cluster-define-networks.png" alt-text="クラスターの作成ウィザード - ネットワークの定義" lightbox="media/cluster/create-cluster-define-networks.png":::

1. 終わったら、 **[Apply and test]\(適用してテスト\)** をクリックします。 アダプターの状態が OK でない場合は、必要に応じて **[Retry connectivity test]\(接続テストの再試行\)** を実行します。

## <a name="step-3-clustering"></a>手順 3:クラスタリング

ウィザードの手順 3 では、これまでのすべての設定が正しく行われていることを確認し、ストレッチ クラスター デプロイの場合は 2 つのサイトを自動的に設定してから、実際にクラスターを作成します。 また、前もって Active Directory にサイトを設定しておくこともできます。

1. **[Next:Clustering]\(次へ: クラスタリング\)** を選択します。
1. **[3.1 Validate the cluster]\(3.1 クラスターの検証\)** で、 **[Validate]\(検証\)** を選択します。 検証には数分かかる場合があります。

    **[Credential Security Service Provider (CredSSP)]\(資格情報セキュリティ サービス プロバイダー (CredSSP)\)** ポップアップが表示される場合は、 **[はい]** を選択し、ウィザードの CredSSP を一時的に有効にして続けます。 クラスターが作成されて、ウィザードが完了したら、CredSSP を無効にしてセキュリティを強化します。 CredSSP に関する問題が発生した場合は、「[CredSSP のトラブルシューティング](../manage/troubleshoot-credssp.md)」で詳しい情報を参照してください。

1. すべての検証状態を確認し、レポートをダウンロードしてエラーに関する詳細情報を取得し、変更を行ってから、必要に応じて **[Validate again]\(再検証\)** をクリックします。 **[Download report]\(レポートのダウンロード\)** を選択することもできます。 すべての検証チェックに合格するまで、必要に応じて繰り返します。 すべて問題なければ、 **[Next]\(次へ\)** をクリックします。
1. **[3.2 Create the cluster]\(3.2 クラスターの作成\)** で、クラスターの名前を入力します。

    :::image type="content" source="media/cluster/create-cluster.png" alt-text="クラスターの作成ウィザード - クラスターの作成" lightbox="media/cluster/create-cluster.png":::

1. **[IP address]\(IP アドレス\)** で、使用する静的または動的 IP アドレスを選択します。 IP アドレスは次の形式で入力する必要があります。"*IP アドレス/現在のサブネットの長さ*"。 次に例を示します。10.0.0.200/24。
1. **[Advanced] \(詳細設定)** を選択します。 いくつかのオプションがあります。

    - **Register the cluster with DNS and Active Directory (クラスターを DNS と Active Directory に登録する)**
    - **Add eligible storage to the cluster (recommended) (クラスターに適格なストレージを追加する (推奨))**

1. **[Networks]\(ネットワーク\)** で、 **[Use all networks (recommended)]\(すべてのネットワークを使用する (推奨)\)** または **[Specify one or more networks not to use]\(使用しない 1 つ以上のネットワークを指定する\)** を選択します。

1. 終わったら、 **[Create cluster]\(クラスターの作成\)** をクリックします。

1. ストレッチ クラスターの場合は、 **[3.3 Assign servers to sites]\(3.3 サーバーをサイトに割り当てる\)** で、使用する 2 つのサイトの名前を指定します。

1. 次に、各サーバーをサイトに割り当てます。 後でサイト間のレプリケーションを設定します。 完了したら、 **[Apply changes]\(変更の適用\)** をクリックします。

## <a name="step-4-storage"></a>手順 4:ストレージ

ウィザードの手順 4 では、クラスターの記憶域スペース ダイレクトを設定します。

1. **[Next:Storage]\(次へ: ストレージ\)** を選択します。
1. お使いのデプロイにとって意味がある場合は、 **[4.1 Clean drives]\(4.1 ドライブのクリーニング\)** で、必要に応じて **[Erase drives]\(ドライブの消去\)** を選択できます。
1. **[4.2 Check drives]\(4.2 ドライブの確認\)** で、各サーバーの横にある **>** アイコンをクリックして、ディスクが動作していて接続されていることを確認します。 すべて問題なければ、 **[Next]\(次へ\)** をクリックします。
1. **[4.3 Validate storage]\(4.3 ストレージの検証\)** で、 **[Next]\(次へ\)** をクリックします。
1. 検証レポートをダウンロードして確認します。 すべて問題がなければ、 **[次へ]** をクリックします。 そうでない場合は、 **[Validate again]\(再検証\)** を実行します。
1. **[4.4 Enable Storage Spaces Direct]\(4.4 記憶域スペース ダイレクトを有効にする\)** で、 **[Enable]\(有効\)** をクリックします。
1. レポートをダウンロードして確認します。 すべて問題がなければ、 **[完了]** をクリックします。 
1. **[Go to connections list]\(接続一覧に移動\)** を選択します。
1. 数分後に、一覧にクラスターが表示されます。 これを選択すると、クラスターの概要ページが表示されます。

ドメイン全体にクラスター名がレプリケートされるまで時間がかかることがあります。特に、Active Directory にワークグループ サーバーが新たに追加されている場合は時間がかかります。 Windows Admin Center にクラスターが表示される場合がありますが、まだ接続することはできません。

しばらくしてもクラスターの解決が成功しない場合は、通常、クラスター名ではなく、サーバー名を使用することができます。

## <a name="next-steps"></a>次のステップ

- クラスターを Azure に登録します。 「[Azure の登録を管理する](../manage/manage-azure-registration.md)」を参照してください。
- クラスターの最終検証を実行します。 「[Azure Stack HCI クラスターの検証](validate.md)」を参照してください
- VM をプロビジョニングします。 「[Windows Admin Center を使用して Azure Stack HCI 上の VM を管理する](../manage/vm.md)」を参照してください。
- PowerShell を使用してクラスターをデプロイすることもできます。 [PowerShell を使用した Azure Stack HCI クラスターの作成](create-cluster-powershell.md)に関する記事を参照してください。
