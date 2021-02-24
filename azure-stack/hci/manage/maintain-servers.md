---
title: メンテナンスのために Azure Stack HCI サーバーをオフラインにする
description: このトピックでは、Windows Admin Center と PowerShell を使用して、Azure Stack HCI オペレーティング システムを実行しているサーバーを適切に一時停止、ドレイン、再開する方法に関するガイダンスを示します。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 10/23/2020
ms.openlocfilehash: 3b2c462ad8e6db96be5968074ce5478f1ca9c870
ms.sourcegitcommit: 6a51687a98c417a004cd4295ad06ae813e1978cc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/27/2020
ms.locfileid: "92641044"
---
# <a name="taking-an-azure-stack-hci-server-offline-for-maintenance"></a>メンテナンスのために Azure Stack HCI サーバーをオフラインにする

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

Azure Stack HCI を使用してメンテナンスのためにサーバーをオフラインにするには、クラスター内のすべてのサーバー間で共有されている記憶域の一部をオフラインにする必要があります。 そのためには、オフラインにするサーバーを一時停止し、役割と仮想マシン (VM) をクラスター内の他のサーバーに移動して、クラスター内のその他のサーバーですべてのデータが使用できるようにする必要があります。 このプロセスにより、メンテナンス期間の最初から最後まで、データが安全でアクセス可能な状態に保たれます。

メンテナンスのためにサーバーをオフラインにするには、Windows Admin Center または PowerShell のどちらでも使用できます。 このトピックでは、両方の方法について説明します。

   > [!IMPORTANT]
   > このトピックでは、メンテナンスを実行するために物理サーバーを電源オフにすること、または他の何らかの理由で再起動することが必要であると想定しています。 Azure Stack HCI クラスターに更新プログラムをインストールするには、「[Azure Stack HCI クラスターを更新する](update-cluster.md)」を参照してください。クラスター対応更新 (CAU) を使用して、このトピックのすべての手順を自動的に実行するだけでなく、サーバーを更新し、必要に応じて再起動する方法について説明しています。

## <a name="take-a-server-offline-using-windows-admin-center"></a>Windows Admin Center を使用してサーバーをオフラインにする

Azure Stack HCI クラスター内のサーバーをオフラインにするための準備を行う最も簡単な方法は、Windows Admin Center を使用することです。

### <a name="verify-its-safe-to-take-the-server-offline"></a>サーバーをオフラインにしても安全であることを確認する

1. Windows Admin Center を使用して、オフラインにするサーバーに接続します。 **[ツール]** メニューから **[記憶域] > [ディスク]** を選択し、各仮想ディスクの **[状態]** 列が **[オンライン]** と表示されていることを確認します。

2. 次に、 **[記憶域] > [ボリューム]** を選択し、各ボリュームの **[正常性]** 列が **[正常]** と表示されていること、および各ボリュームの **[状態]** 列が **[OK]** と表示されていることを確認します。

### <a name="pause-and-drain-the-server"></a>サーバーの一時停止とドレイン

サーバーをシャットダウンまたは再起動する前に、サーバーを一時停止し、実行中の VM などのクラスター化された役割をドレイン (移動) して、サーバーのシャットダウンがアプリケーションの状態に影響しないようにする必要があります。 クラスター化されたサーバーをメンテナンスのためにオフラインにする前に、常に一時停止してドレインします。

1. Windows Admin Center を使用してクラスターに接続し、クラスター マネージャーの **[ツール]** メニューから **[コンピューティング] > [ノード]** を選択します。

2. 一時停止してドレインするサーバーの名前をクリックし、 **[一時停止]** を選択します。 次のプロンプトが表示されます。

   *If you pause this node, all clustered roles move to other nodes and no roles can be added to this node until it's resumed. Are you sure you want to pause cluster node?* (このノードを一時停止すると、すべてのクラスター化された役割が他のノードに移動します。再開されるまで、このノードに役割を追加することはできません。クラスター ノードを一時停止しますか?)

3. サーバーを一時停止してドレイン プロセスを開始するには、 **[はい]** を選択します。 クラスター ノードの状態は **[ドレイン中]** と表示され、Hyper-V や VM などの役割については、クラスター内の他のサーバーへのライブ マイグレーションが直ちに開始されます。 これには数分かかることがあります。

   > [!NOTE]
   > サーバーを正常に一時停止してドレインすると、Azure Stack HCI 上で自動的に安全性のチェックが実行され、処理を続行しても安全であることが確認されます。 正常でないボリュームがある場合は、停止して、安全に続行できないことを警告します。

### <a name="shut-down-the-server"></a>サーバーをシャットダウンする

サーバーのドレインが完了すると、Windows Admin Center に **[一時停止]** として表示されます。 これで、メンテナンスのためにサーバーを安全にシャットダウンしたり、再起動したりできるようになりました。

### <a name="resume-the-server"></a>サーバーを再開する

クラスター化された役割と VM のホストをサーバーで再び開始する準備ができたら、サーバーを電源オンにし、起動するのを待ってから、次の手順に従ってサーバーを再開します。

1. クラスター マネージャーで、左側にある **[ツール]** メニューから **[コンピューティング] > [ノード]** を選択します。

2. 再開するサーバーの名前を選択し、 **[再開]** をクリックします。 次のプロンプトが表示されます。

   *クラスター ノードを再開しますか?*

3. ほとんどの場合は、 *[Transfer clustered roles back to this node]\(クラスター化された役割をこのノードに転送する\)* チェックボックスをオンにします。 **[はい]** を選択してサーバーを再開します。

上記の手順 3 でチェックボックスをオンにした場合、クラスター化された役割と VM については、すぐにサーバーへのライブ マイグレーションが開始されます。 これには数分かかることがあります。

### <a name="wait-for-storage-to-resync"></a>記憶域が再同期されるまで待機する

サーバーが再開されると、利用できなかった間に発生した新しい書き込みについては再同期が必要になります。 これは、インテリジェントな変更の追跡を使用して自動的に行われます。 スキャンまたは同期する必要があるのは変更のみであり、" *すべての* " データではありません。 このプロセスは、実稼働ワークロードへの影響を軽減するために調整されます。 サーバーが一時停止されてからの時間と、書き込まれた新しいデータの量によっては、完了までに数分かかることがあります。

   > [!IMPORTANT]
   > クラスター内の他のサーバーをオフラインにする前に、再同期が完了するまで待機する必要があります。

再同期が完了したかどうかを確認するには、Windows Admin Center を使用してサーバーに接続し、左側にある **[ツール]** メニューから **[記憶域] > [ボリューム]** を選択します。次に、ページの上部にある **[ボリューム]** を選択します。 すべてのボリュームの **[正常性]** 列に **[正常]** が表示され、各ボリュームの **[状態]** 列に **[OK]** と表示されている場合は、再同期が完了しており、クラスター内の他のサーバーをオフラインにしても安全です。

## <a name="take-a-server-offline-using-powershell"></a>PowerShell を使用してサーバーをオフラインにする

PowerShell を使用して Azure Stack HCI クラスター内のサーバーを適切に一時停止、ドレイン、再開するには、次の手順を使用します。

### <a name="verify-its-safe-to-take-the-server-offline"></a>サーバーをオフラインにしても安全であることを確認する

すべてのボリュームが正常であることを確認するには、次のコマンドレットを管理者として実行します。

```PowerShell
Get-VirtualDisk
```

この出力の例を次に示します。

```
FriendlyName              ResiliencySettingName FaultDomainRedundancy OperationalStatus HealthStatus    Size FootprintOnPool StorageEfficiency
------------              --------------------- --------------------- ----------------- ------------    ---- --------------- -----------------
Mirror II                 Mirror                1                     OK                Healthy         4 TB         8.01 TB            49.99%
Mirror-accelerated parity                                             OK                Healthy      1002 GB         1.96 TB            49.98%
Mirror                    Mirror                1                     OK                Healthy         1 TB            2 TB            49.98%
ClusterPerformanceHistory Mirror                1                     OK                Healthy        24 GB           49 GB            48.98%
```

各ボリュームの **HealthStatus** プロパティが **Healthy** であり、 **OperationalStatus** が OK と表示されていることを確認します。

### <a name="pause-and-drain-the-server"></a>サーバーの一時停止とドレイン

サーバーの一時停止とドレインを行うには、次のコマンドレットを管理者として実行します。

```PowerShell
Suspend-ClusterNode -Drain
```

### <a name="shut-down-the-server"></a>サーバーをシャットダウンする

サーバーのドレインが完了すると、PowerShell で **Paused** として表示されます。

これで、`Stop-Computer` または `Restart-Computer` の PowerShell コマンドレットを使用して、サーバーを安全にシャットダウンまたは再起動できるようになりました。

   > [!NOTE]
   > シャットダウンするか、クラスター サービスを開始または停止するサーバーで `Get-VirtualDisk` コマンドを実行すると、サーバーの動作状態が不完全または低下として報告され、正常性状態の列に警告が表示されることがあります。 これは正常であり、問題は発生しません。 すべてのボリュームは、オンラインでアクセス可能なままです。

### <a name="resume-the-server"></a>サーバーを再開する

サーバーをクラスターに戻して再開するには、次のコマンドレットを管理者として実行します。 サーバーで以前に実行されていたクラスター化された役割と VM を戻すには、オプションの **-Failback** フラグを使用します。

```PowerShell
Resume-ClusterNode –Failback Immediate
```

サーバーが再開されると、PowerShell で **Up** として表示されます。

### <a name="wait-for-storage-to-resync"></a>記憶域が再同期されるまで待機する

サーバーが再開されたら、クラスター内の他のサーバーをオフラインにする前に、再同期が完了するまで待機する必要があります。

進行状況を監視するには、次のコマンドレットを管理者として実行します。

```PowerShell
Get-StorageJob
```

再同期が既に完了している場合、出力は何も表示されません。

実行中の再同期 (修復) ジョブを示す出力例を次に示します。

```
Name   IsBackgroundTask ElapsedTime JobState  PercentComplete BytesProcessed BytesTotal
----   ---------------- ----------- --------  --------------- -------------- ----------
Repair True             00:06:23    Running   65              11477975040    17448304640
Repair True             00:06:40    Running   66              15987900416    23890755584
Repair True             00:06:52    Running   68              20104802841    22104819713
```

**BytesTotal** 列には、再同期する必要がある記憶域の量が表示されます。 **PercentComplete** 列に進行状況が表示されます。

   > [!WARNING]
   > これらの修復ジョブが完了するまでは、他のサーバーを安全にオフラインにすることはできません。

この間、 **HealthStatus** にはボリュームが **Warning** として引き続き表示されます。これは正常です。

たとえば、記憶域の再同期中に `Get-VirtualDisk` コマンドレットを使用すると、次の出力が表示される場合があります。

```
FriendlyName ResiliencySettingName OperationalStatus HealthStatus IsManualAttach Size
------------ --------------------- ----------------- ------------ -------------- ----
MyVolume1    Mirror                InService         Warning      True           1 TB
MyVolume2    Mirror                InService         Warning      True           1 TB
MyVolume3    Mirror                InService         Warning      True           1 TB
```

ジョブが完了したら、再度 `Get-VirtualDisk` コマンドレットを使用して、ボリュームが **Healthy** と表示されることを確認します。 出力例を次に示します。

```
FriendlyName ResiliencySettingName OperationalStatus HealthStatus IsManualAttach Size
------------ --------------------- ----------------- ------------ -------------- ----
MyVolume1    Mirror                OK                Healthy      True           1 TB
MyVolume2    Mirror                OK                Healthy      True           1 TB
MyVolume3    Mirror                OK                Healthy      True           1 TB
```

これで、クラスター内の他のサーバーの一時停止や再起動を安全に行うことができます。

## <a name="next-steps"></a>次のステップ

関連情報については、以下もご覧ください。

- [記憶域スペース ダイレクトの概要](/windows-server/storage/storage-spaces/storage-spaces-direct-overview)
- [Azure Stack HCI クラスターを更新する](update-cluster.md)