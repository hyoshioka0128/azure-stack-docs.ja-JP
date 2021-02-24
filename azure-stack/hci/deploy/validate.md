---
title: Azure Stack HCI クラスターの検証
description: クラスター検証の重要性、どのようなときに既存の Azure Stack HCI クラスターに対して検証を実行するかについて説明します。 更新されたサーバー クラスターをトラブルシューティングするためのシナリオについて説明します。
author: JohnCobb1
ms.author: v-johcob
ms.topic: article
ms.date: 10/16/2020
ms.openlocfilehash: fe49df76ccb2a90849587acd5d4df7a41e329efb
ms.sourcegitcommit: 301e571626f8e85556d9eabee3f385d0b81fdef4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/17/2020
ms.locfileid: "92157701"
---
# <a name="validate-an-azure-stack-hci-cluster"></a>Azure Stack HCI クラスターの検証

>適用対象:Azure Stack HCI バージョン v20H2、Windows Server 2019

このハウツー記事では、クラスターの検証が重要である理由と、どのようなときに既存の Azure Stack HCI クラスターに対してそれを実行するかについて説明します。 主に次のようなシナリオで、クラスターの検証を実行することをお勧めします。
- サーバー クラスターをデプロイした後、Validate-DCB ツールを実行してネットワークをテストします。
- サーバー クラスターを更新した後、シナリオによっては、両方の検証オプションを実行して、クラスターの問題のトラブルシューティングを行います。
- 記憶域レプリカを使用してレプリケーションを設定した後、いくつかの具体的なイベントを調べて、コマンドをいくつか実行することにより、レプリケーションが正常に進行していることを確認します。
- サーバー クラスターを作成したら、運用環境に配置する前に Validate-DCB ツールを実行します。

    Azure Stack HCI クラスターをデプロイする方法の詳細については、「[デプロイの概要](deployment-overview.md)」を参照してください。

## <a name="what-is-cluster-validation"></a>クラスターの検証とは
クラスターの検証の目的は、クラスターの運用を始める前に、ハードウェアまたは構成の問題を検出することです。 クラスターの検証を行うと、デプロイしようとしている Azure Stack HCI ソリューションが本当に信頼できることを確認するのに役立ちます。 また、構成済みのフェールオーバー クラスターで診断ツールとしてクラスターの検証を使用することもできます。

## <a name="specific-validation-scenarios"></a>具体的な検証シナリオ
ここでは、検証が必要または便利なシナリオについて説明します。

- **クラスターを構成する前の検証:**
  - **フェールオーバー クラスターにする準備ができているサーバーのセット:** これは、最も簡単な検証シナリオです。 ハードウェア コンポーネント (システム、ネットワーク、ストレージ) は接続されていますが、システムはまだクラスターとして機能していません。 この状況でテストを実行しても、可用性には影響しません。
 
  - **サーバー VM:** クラスター内の仮想化されたサーバーの場合、他の新しいクラスターの場合と同様に、クラスターの検証を実行します。 この機能を実行するための要件は、次のどちらの場合でも同じです。
    - 2 台の物理コンピューターの間でフェールオーバーが行われる "ホスト クラスター"。
    - 同じ物理コンピューター上のゲスト オペレーティング システムの間でフェールオーバーが行われる "ゲスト クラスター"。
 
- **クラスターが構成されて使用された後の検証:**

  - **クラスターにサーバーを追加する前:** クラスターにサーバーを追加するときは、クラスターを検証することを強くお勧めします。 クラスターの検証を実行するときに、既存のクラスター メンバーと新しいサーバーの両方を指定します。
  
  - **ドライブを追加するとき:** 新しいドライブをクラスターに追加するときは、障害が発生したドライブの交換や、既存のドライブに依存する仮想ディスクやボリュームの作成とは異なり、クラスターの検証を実行して、新しいストレージが正しく機能することを確認します。

  - **ファームウェアまたはドライバーに影響する変更を行うとき:** クラスターでファームウェアまたはドライバーに影響するアップグレードまたは変更を行う場合は、クラスターの検証を実行して、ハードウェア、ファームウェア、ドライバー、ソフトウェアの新しい組み合わせで、フェールオーバー クラスターの機能がサポートされることを確認する必要があります。

  - **バックアップからシステムを復元した後:** バックアップからシステムを復元した後は、クラスターの検証を実行し、システムがクラスターの一部として正しく機能していることを確認します。

## <a name="validate-networking"></a>ネットワークを検証する
Microsoft の Validate-DCB ツールは、クラスターでのデータ センター ブリッジング (DCB) の構成を検証するように設計されています。 これを行うには、ツールに入力として期待される構成を渡すと、クラスター内の各サーバーのテストが行われます。 このセクションでは、Validate-DCB ツールをインストールして実行する方法、結果を確認する方法、およびツールによって識別されたネットワーク エラーを解決する方法について説明します。

ネットワークで RDMA (リモート ダイレクト メモリ アクセス) over Converged Ethernet (RoCE) を使用すると、ネットワーク ファブリックを無損失にするために DCB テクノロジが必要です。 iWARP の場合、DCB は省略可能です。 ただし、DCB の構成は複雑な場合があり、以下のすべてで正確に構成する必要があります。
- クラスター内の各サーバー
- ファブリック上の RDMA トラフィックが通過する各ネットワーク ポート

### <a name="prerequisites"></a>前提条件
- 次のような、検証するサーバー クラスターのネットワーク セットアップ情報。
    - ホストまたはサーバー クラスターの名前
    - 仮想スイッチの名前
    - ネットワーク アダプターの名前
    - 優先順位フロー制御 (PFC) と Enhanced Transmission Selection (ETS) の設定
- Microsoft から Windows PowerShell のツール モジュールをダウンロードするためのインターネット接続。

### <a name="install-and-run-the-validate-dcb-tool"></a>Validate-DCB ツールをインストールして実行する
Validate-DCB ツールをインストールして実行するには:
1. 管理 PC で、管理者として Windows PowerShell セッションを開き、次のコマンドを使用してツールをインストールします。

    ```powershell
    Install-Module Validate-DCB
    ```

1. NuGet プロバイダーの使用に関する要求を受け入れ、リポジトリにアクセスしてツールをインストールします。
1. PowerShell により Microsoft ネットワークに接続してツールがダウンロードされた後、「`Validate-DCB`」と入力して **Enter** キーを押し、ツール ウィザードを開始します。

    > [!NOTE]
    > Validate-DCB ツールのスクリプトを実行できない場合は、PowerShell の実行ポリシーの調整が必要になることがあります。 現在のスクリプト実行ポリシーの設定を表示するには、Get-ExecutionPolicy コマンドレットを使用します。 PowerShell での実行ポリシーの設定の詳細については、「[実行ポリシーについて](/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7)」を参照してください。

1. [Welcome to the Validate-DCB]\(Validate-DCB へようこそ\) 構成ウィザード ページで、 **[次へ]** を選択します。
1. [Clusters and Nodes]\(クラスターとノード\) ページで、検証するサーバー クラスターの名前を入力し、 **[解決]** を選択してページにその一覧を表示した後、 **[次へ]** を選択します。

    :::image type="content" source="../media/validate/clusters-and-nodes.png" alt-text="Validate-DCB 構成ウィザードの [Clusters and Nodes]\(クラスターとノード\) ページ":::

1. [Adapters]\(アダプター\) ページで、次のようにします。
   1. **[vSwitch attached]\(接続されている vSwitch\)** チェック ボックスをオンにして、vSwitch の名前を入力します。
   1. **[Adapter Name]\(アダプター名\)** に各物理 NIC の名前を入力し、 **[Host vNIC Name]\(ホスト vNIC 名\)** に各仮想 NIC (vNIC) の名前を入力し、 **[VLAN]** に各アダプターで使用されている VLAN ID を入力します。
   1. **[RDMA Type]\(RDMA の種類\)** ドロップダウンの一覧を展開して、適切なプロトコルを選択します: **[RoCE]** または **[iWARP]** 。 また、 **[Jumbo Frames]\(ジャンボ フレーム\)** をネットワークに対する適切な値に設定し、 **[次へ]** を選択します。

    :::image type="content" source="../media/validate/adapters.png" alt-text="Validate-DCB 構成ウィザードの [Clusters and Nodes]\(クラスターとノード\) ページ" lightbox="../media/validate/adapters.png":::

    > [!NOTE]
    > - SR-IOV によるネットワーク パフォーマンスの向上の詳細については、「[シングル ルート I/O 仮想化 (SR-IOV) の概要](/windows-hardware/drivers/network/overview-of-single-root-i-o-virtualization--sr-iov-)」を参照してください。

1. [Data Center Bridging]\(データ センター ブリッジング\) ページで、 **[Priority]\(優先順位\)** 、 **[Policy Name]\(ポリシー名\)** 、 **[Bandwidth Reservation]\(帯域幅予約\)** の値を組織の設定と一致するように変更して、 **[次へ]** を選択します。

    :::image type="content" source="../media/validate/data-center-bridging.png" alt-text="Validate-DCB 構成ウィザードの [Clusters and Nodes]\(クラスターとノード\) ページ" lightbox="../media/validate/data-center-bridging.png":::

    > [!NOTE]
    > 前のウィザード ページで RDMA over RoCE を選択した場合は、すべての NIC とスイッチポートでネットワークの信頼性のために DCB が必要です。

1. [Save and Deploy]\(保存とデプロイ\) ページの **[Configuration File Path]\(構成ファイルのパス\)** ボックスで、後で必要な場合に再び使用できる場所に、拡張子 .ps1 を使用して構成ファイルを保存し、次に **[Export]\(エクスポート\)** を選択して Validate-DCB ツールの実行を開始します。

   - 必要に応じて、ページの **[Deploy Configuration to Nodes]\(ノードに構成をデプロイする\)** セクションを設定することで、構成ファイルをデプロイできます。これには、Azure Automation アカウントを使用して構成をデプロイしてから検証する機能が含まれます。 Azure Automation の使用を開始するには、「[Azure Automation アカウントを作成する](/azure/automation/automation-quickstart-create-account)」を参照してください。

    :::image type="content" source="../media/validate/save-and-deploy.png" alt-text="Validate-DCB 構成ウィザードの [Clusters and Nodes]\(クラスターとノード\) ページ":::

### <a name="review-results-and-fix-errors"></a>結果を確認してエラーを修正する
Validate-DCB ツールにより、2 つのユニットで結果が生成されます。
1. [Global Unit] の結果には、モーダル テストを実行するための前提条件と要件の一覧が表示されます。
1. [Modal Unit] の結果では、各クラスター ホストの構成とベスト プラクティスについてのフィードバックが提供されます。

この例では、失敗カウントが 0 なので、すべての前提条件とモーダルのユニットのテストで、単一サーバーのスキャン結果が成功であったことが示されています。

:::image type="content" source="../media/validate/global-unit-and-modal-unit-results.png" alt-text="Validate-DCB 構成ウィザードの [Clusters and Nodes]\(クラスターとノード\) ページ":::

次の手順では、vNIC SMB02 からのジャンボ パケット エラーを識別して修正する方法を示します。
1. Validate-DCB ツール スキャンの結果では、失敗カウントで 1 つのエラーが示されています。

    :::image type="content" source="../media/validate/failed-count-error-1.png" alt-text="Validate-DCB 構成ウィザードの [Clusters and Nodes]\(クラスターとノード\) ページ":::

1. 結果を上にスクロールするとエラーが赤で表示され、ホスト S046036 の vNIC SMB02 に対するジャンボ パケットが既定のサイズ 1514 に設定されてますが、9014 に設定する必要があることが示されています。

    :::image type="content" source="../media/validate/jumbo-packet-setting-error.png" alt-text="Validate-DCB 構成ウィザードの [Clusters and Nodes]\(クラスターとノード\) ページ":::

1. ホスト S046036 の vNIC SMB02 の **[詳細設定]** プロパティを確認すると、ジャンボ パケットが既定の **[無効]** に設定されていることがわかります。

    :::image type="content" source="../media/validate/hyper-v-advanced-properties-jumbo-packet-setting.png" alt-text="Validate-DCB 構成ウィザードの [Clusters and Nodes]\(クラスターとノード\) ページ":::

1. エラーを修正するには、ジャンボ パケット機能を有効にして、サイズを 9014 バイトに変更する必要があります。 ホスト S046036 でもう一度スキャンを実行すると、失敗カウントが 0 に戻っていることで、この変更が確認されます。

    :::image type="content" source="../media/validate/jumbo-packet-error-fix-confirmation.png" alt-text="Validate-DCB 構成ウィザードの [Clusters and Nodes]\(クラスターとノード\) ページ":::

Validate-DCB ツールで明らかになったエラーの解決方法の詳細については、次のビデオをご覧ください。

> [!VIDEO https://www.youtube.com/embed/cC1uACvhPBs]

## <a name="validate-the-cluster"></a>クラスターを検証する
Windows Admin Center で既存のクラスターのサーバーを検証するには、次の手順を使用します。

1. Windows Admin Center の **[すべての接続]** で検証する Azure Stack HCI クラスターを選択し、 **[接続]** を選択します。

    **クラスター マネージャー ダッシュボード** に、クラスターに関する概要情報が表示されます。

1. **クラスター マネージャー ダッシュボード** の **[ツール]** で、 **[サーバー]** を選択します。
1. **[インベントリ]** ページで、クラスター内のサーバーを選択し、 **[その他]** サブメニューを展開して、 **[クラスターの検証]** を選択します。
1. **[クラスターの検証]** ポップアップ ウィンドウで、 **[はい]** を選択します。

    :::image type="content" source="../media/validate/validate-cluster-pop-up.png" alt-text="Validate-DCB 構成ウィザードの [Clusters and Nodes]\(クラスターとノード\) ページ":::

1. **[Credential Security Service Provider (CredSSP)]** ポップアップ ウィンドウで、 **[はい]** を選択します。
1. 資格情報を入力して **CredSSP** を有効にした後、 **[続行]** を選択します。<br> クラスターの検証がバックグラウンドで実行され、完了すると通知が表示されます。その時点で、次のセクションで説明するように、検証レポートを表示できます。

> [!NOTE]
> クラスター サーバーの検証が済んだら、セキュリティのため CredSSP を無効にする必要があります。

### <a name="disable-credssp"></a>CredSSP を無効にする
サーバー クラスターが正常に検証されたら、セキュリティのため、各サーバーで資格情報セキュリティ サポート プロバイダー (CredSSP) プロトコルを無効にします。 詳細については、[CVE-2018-0886](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/CVE-2018-0886) を参照してください。

1. Windows Admin Center の **[すべての接続]** で、クラスター内の最初のサーバーを選択し、 **[接続]** を選択します。
1. **[概要]** ページで **[Disable CredSSP]\(CredSSP を無効にする\)** を選択し、次に **[Disable CredSSP]\(CredSSP を無効にする\)** ポップアップ ウィンドウで **[はい]** を選択します。

    ステップ 2 を実行すると、サーバーの **[概要]** ページの上部にある赤い **[CredSSP ENABLED]\(CredSSP が有効になっています\)** というバナーが表示されなくなり、他のサーバーの CredSSP が無効になります。

### <a name="view-validation-reports"></a>検証レポートを表示する
これで、クラスター検証レポートを表示する準備ができました。

検証レポートにアクセスするには、2 つの方法があります。
- **[インベントリ]** ページで **[その他]** サブメニューを展開し、 **[View validation reports]\(検証レポートの表示\)** を選択します。


- **Windows Admin Center** の右上にある **[通知]** ベル アイコンを選択して、 **[通知]** ウィンドウを表示します。
**[Successfully validated cluster]\(正常に検証されたクラスター\)** 通知を選択し、 **[Go to Failover Cluster validation report]\(フェールオーバー クラスター検証レポートに移動する\)** を選択します。

> [!NOTE]
> サーバー クラスター検証プロセスが完了するまでに時間がかかることがあります。 プロセスの実行中は、Windows Admin Center の別のツールに切り替えないでください。 **[通知]** ウィンドウで、 **[クラスターの検証]** 通知の下にあるステータス バーには、プロセスがいつ完了したかが示されます。

## <a name="validate-the-cluster-using-powershell"></a>PowerShell を使用してクラスターを検証する
Windows PowerShell を使用して、サーバー クラスターに対する検証テストを実行し、結果を表示することもできます。 テストは、クラスターのセットアップの前と後の両方で実行できます。

サーバー クラスターの検証テストを実行するには、管理 PC から **Get-Cluster** および **Test-Cluster** <server clustername> PowerShell コマンドレットを実行するか、クラスターで直接 **Test-Cluster** コマンドレットのみを実行します。

```PowerShell
$Cluster = Get-Cluster -Name 'server-cluster1'
Test-Cluster -InputObject $Cluster -Verbose
```

その他の例と使用方法に関する情報については、「[Test-Cluster](/powershell/module/failoverclusters/test-cluster?view=win10-ps)」のリファレンス ドキュメントを参照してください。

## <a name="validate-replication-for-storage-replica"></a>記憶域レプリカのレプリケーションを検証する
記憶域レプリカを使用して、ストレッチ クラスター内またはクラスター間でボリュームをレプリケートしている場合は、レプリケーションの状態を取得するために使用できるイベントとコマンドレットがいくつかあります。 

次のシナリオでは、2 つのサイトにレプリケーション グループ (RG) を作成することにより記憶域レプリカを構成した後、Site1 のソース サーバー ノード (Server1、Server2) と Site2 のターゲット (レプリケート先) サーバー ノード (Server3、Server4) の両方に対して、データ ボリュームとログ ボリュームを指定しました。

Site1 の Server1 でのレプリケーションの進行状況を確認するには、Get-WinEvent コマンドを実行し、イベント 5015、5002、5004、1237、5001、2200 を調べます。

```powershell
Get-WinEvent -ComputerName Server1 -ProviderName Microsoft-Windows-StorageReplica -max 20
```

Site2 の Server3 の場合は、次の `Get-WinEvent` コマンドを実行して、パートナーシップの作成を示す記憶域レプリカ イベントを確認します。 このイベントでは、コピーされたバイト数と所要時間が示されます。 次に例を示します。

```powershell
Get-WinEvent -ComputerName Server3 -ProviderName Microsoft-Windows-StorageReplica | Where-Object {$_.ID -eq "1215"} | FL
```

Site2 の Server3 の場合、`Get-WinEvent` コマンドを実行し、イベント 5009、1237、5001、5015、5005、2200 を調べて、処理の進行状況を把握します。 このシーケンスでは、エラーの警告は発生しないはずです。 多数の 1237 イベントが発生します。これらは進行状況を示します。

```powershell
Get-WinEvent -ComputerName Server3 -ProviderName Microsoft-Windows-StorageReplica | FL
```

または、レプリカのターゲット サーバー グループでは、常にコピーの残りバイト数が示され、PowerShell で`Get-SRGroup` を使用してクエリを実行できます。 次に例を示します。

```powershell
(Get-SRGroup).Replicas | Select-Object numofbytesremaining
```

Site2 のノード Server3 の場合は、次のコマンドを実行し、イベント 5009、1237、5001、5015、5005、2200 を調べて、レプリケーションの進行状況を把握します。 エラーの警告は表示されないはずです。 ただし、多くの "1237" イベントが存在します。これは単に進行状況を示します。

```powershell
Get-WinEvent -ComputerName Server3 -ProviderName Microsoft-Windows-StorageReplica | FL
```

終了しない進行状況スクリプトとして:

```powershell
while($true) {
$v = (Get-SRGroup -Name "Replication2").replicas | Select-Object numofbytesremaining
[System.Console]::Write("Number of bytes remaining: {0}`r", $v.numofbytesremaining)
Start-Sleep -s 5
}
```

ストレッチ クラスター内でレプリケーションの状態を取得するには、`Get-SRGroup` と `Get-SRPartnership` を使用します。

```powershell
Get-SRGroup -Cluster ClusterS1
```

```powershell
Get-SRPartnership -Cluster ClusterS1
```

```powershell
(Get-SRGroup).replicas -Cluster ClusterS1
```

サイト間でデータが正常にレプリケートされたことを確認したら、VM と他のワークロードを作成できます。

## <a name="see-also"></a>関連項目
- 新しく作成された記憶域スペースでの合成ワークロードに対する、DiskSpd.exe を使用したパフォーマンス テスト。 詳細については、「[Windows Server で合成ワークロードを使用して記憶域スペースのパフォーマンスをテストする](/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/dn894707(v=ws.11))」を参照してください。
- Windows Server Assessment は、Microsoft に Windows Server 2019 のインストールの確認を希望するお客様が利用できる Premier サービスです。 詳細については、Microsoft Premier サポートにお問い合わせください。 詳細については、「[Windows Server のオンデマンド評価 (サーバー、セキュリティ、Hyper-V、フェールオーバー クラスター、IIS) の概要](/services-hub/health/getting-started-windows-server)」を参照してください。
