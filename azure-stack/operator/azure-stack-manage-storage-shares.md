---
title: Azure Stack Hub でのストレージ容量を管理する
description: Azure Stack Hub のストレージ容量と可用性を監視および管理する方法について説明します。
author: PatAltimore
ms.topic: conceptual
ms.date: 10/16/2020
ms.author: patricka
ms.reviewer: xiaofmao
ms.lastreviewed: 10/16/2020
ms.openlocfilehash: c6f58a2364378180641a7b3ea2e3a38e4bb4f183
ms.sourcegitcommit: 330d04d39e0cf3e8965e2ccbc181c968cb71d9ad
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2021
ms.locfileid: "98052308"
---
# <a name="manage-storage-capacity-for-azure-stack-hub"></a>Azure Stack Hub のストレージ容量を管理する

この記事は、Azure Stack Hub クラウド オペレーターが、Azure Stack Hub のデプロイのストレージ容量を監視および管理するのに役立ちます。 Azure Stack Hub ストレージ インフラストラクチャでは、Azure Stack Hub のデプロイの合計ストレージ容量のサブセットがストレージ サービスとして割り当てられます。 ストレージ サービスは、デプロイのノードに対応するボリューム上の共有にテナント データを格納します。

クラウド オペレーターとして操作できるストレージの量には制限があります。 ストレージの量は、実装するソリューションによって定義されます。 ソリューションは、マルチノード ソリューションを使用する場合は OEM ベンダーによって提供され、Azure Stack Development Kit (ASDK) をインストールする場合はインストール先のハードウェアによって提供されます。

Azure Stack Hub でサポートされているのは、スケール ユニット ノードを追加することによるストレージ容量の拡張だけです。 詳細については、「[Azure Stack Hub のスケール ユニット ノードを追加する](azure-stack-add-scale-node.md)」を参照してください。 ノードに物理ディスクを追加しても、ストレージ容量は拡張されません。

効率的な運用が確実に維持されるように、使用可能なストレージを[監視](#monitor-shares)することが重要です。 ボリュームの残りの空き容量が少なくなった場合は、共有が容量不足になることを防ぐために[使用可能な領域の管理](#manage-available-space)を計画します。

容量の管理のためのオプションは次のとおりです。


- 容量の解放。
- ストレージ オブジェクトの移行。

オブジェクト ストアが 100% 使用されると、そのボリュームに対するストレージ サービスは機能しなくなります。 ボリュームに対する操作を復元するための支援を得るには、Microsoft サポートに問い合わせてください。

## <a name="understand-volumes-and-shares-containers-and-disks"></a>ボリュームと共有、コンテナー、およびディスクを理解する

### <a name="volumes-and-shares"></a>ボリュームと共有

*ストレージ サービス* は、使用可能なストレージを、テナント データを保持するために割り当てられている複数の同量のボリュームにパーティション分割します。 Azure Stack Hub のボリュームの詳細については、「[Azure Stack Hub 用のストレージ インフラストラクチャを管理する](azure-stack-manage-storage-infrastructure.md)」を参照してください。

オブジェクト ストア ボリュームでは、テナント データが保持されます。 テナント データには、ページ BLOB、ブロック BLOB、追加 BLOB、テーブル、キュー、データベース、および関連するメタデータ ストアが含まれます。 オブジェクト ストア ボリュームの数は、Azure Stack Hub のデプロイに含まれるノード数と同じになります。

- 4 ノード デプロイには、4 つのオブジェクト ストア ボリュームがあります。 マルチノード デプロイでは、ノードが削除されたり正常に機能しなくなったりした場合でも、ボリュームの数が減ることはありません。
- ASDK を使用する場合は、単一の共有を持つ単一のボリュームがあります。

オブジェクト ストア ボリュームは、ストレージ サービスを排他的に使用するためのものです。 ボリューム上のファイルを直接変更、追加、または削除しないでください。 これらのボリュームに格納されているファイルは、ストレージ サービスのみが操作する必要があります。

ストレージ オブジェクト (BLOB など) は、単一のボリューム内に個別に格納されるため、各オブジェクトの最大サイズは、ボリュームのサイズを超えることはできません。 新しいオブジェクトの最大サイズは、新しいオブジェクトの作成時にボリューム内に未使用の領域として残っている容量によって決まります。

オブジェクト ストア ボリュームの空き領域が少ないときに、領域を[回収](#reclaim-capacity)するためのアクションが成功しないか使用できない場合、Azure Stack Hub クラウド オペレーターは、あるボリュームから別のボリュームにストレージ オブジェクトを移行できます。

テナント ユーザーによる Azure Stack Hub での BLOB ストレージの操作方法については、[Azure Stack Hub ストレージ サービス](../user/azure-stack-storage-overview.md)に関する記事を参照してください。

### <a name="containers"></a>Containers

テナント ユーザーは、BLOB データを格納するために使用されるコンテナーを作成します。 BLOB を配置するコンテナーはユーザーが決定しますが、コンテナーを配置するボリュームは、ストレージ サービスがアルゴリズムを使用して決定します。 このアルゴリズムでは、通常は、使用可能な領域が最も多いボリュームが選択されます。  

コンテナーに BLOB が配置された後、その BLOB が増加してより多くの領域を使用する可能性があります。 新しい BLOB を追加するときに既存の BLOB が増加していると、そのコンテナーを保持しているボリュームの空き領域が少なくなります。 

コンテナーは、単一のボリュームに限定されません。 コンテナー内の結合された BLOB データが増加して使用可能な領域の 80% 以上を使用すると、コンテナーは "*オーバーフロー*" モードに入ります。 オーバーフロー モードに入ると、そのコンテナー内に作成される新しい BLOB は、十分な領域がある別のボリュームに割り当てられます。 時間の経過と共に、オーバーフロー モードのコンテナーは、複数のボリュームに分散された BLOB を持つ可能性があります。

ボリュームの使用可能領域の 80%、その後 90% が使用された時点で、システムによって Azure Stack Hub 管理者ポータルに[アラート](#storage-space-alerts)が生成されます。 クラウド オペレーターは、使用可能なストレージ容量を確認し、コンテンツを再バランス調整することを計画する必要があります。 ディスクが 100% 使用されると、ストレージ サービスは動作を停止しますが、この時点で他のアラートが生成されることはありません。

### <a name="disks"></a>ディスク

Azure Stack Hub では、マネージド ディスクとアンマネージド ディスクを、オペレーティング システム (OS) およびデータ ディスクの両方として VM で使用できます。

**マネージド ディスク** を使用すると、VM ディスクに関連付けられているストレージ アカウントを管理できるため、Azure IaaS VM のディスク管理が簡素化されます。 Azure Stack Hub では、必要なディスクのサイズを指定するだけで、ディスクの作成と管理が自動的に行われます。 詳細については、[マネージド ディスクの概要](/azure/virtual-machines/windows/managed-disks-overview)に関する記事を参照してください。

管理と容量のバランスを簡単に取れるように、VM にはマネージド ディスクを使用することをお勧めします。 マネージド ディスクを使用する前に、ストレージ アカウントとコンテナーを準備する必要はありません。 複数のマネージド ディスクを作成する場合、ディスクは複数のボリュームに分散されるため、ボリュームの容量のバランスを取ることができます。  

**アンマネージド ディスク** は、Azure ストレージ アカウントにページ BLOB として格納される VHD ファイルです。 テナントによって作成されるページ BLOB は VM ディスクと呼ばれ、ストレージ アカウントのコンテナーに格納されます。 アンマネージド ディスクは、Azure アンマネージド ディスクのみがサポートされているサード パーティ ツールとの互換性のために必要な VM でのみ使用することをお勧めします。

テナントへのガイダンスは、VM のパフォーマンスを向上させるために各ディスクを別のコンテナーに配置することです。

- VM のディスク (ページ BLOB) を保持する各コンテナーは、ディスクを所有している VM に接続されているコンテナーであるとみなされます。
- VM の任意のディスクを保持していないコンテナーは、空のコンテナーとみなされます。

接続されているコンテナーの領域を解放するためのオプションは制限されています。 詳細については、「[アンマネージド ディスクを分散させる](#distribute-unmanaged-disks)」を参照してください。

>[!TIP]  
> クラウド オペレーターは、テナントによってコンテナーに追加された VM に接続されているディスクを直接管理しません。 ただし、ストレージの共有の領域に対する管理を計画するときに、アンマネージド ディスクがコンテナーと共有にどのように関連しているかを理解しておくと役に立つ可能性があります。

::: moniker range="<azs-2002"
## <a name="monitor-shares"></a>共有を監視する

Azure PowerShell または管理者ポータルを使用して共有を監視することで、空き領域が少なくなった場合にそれを把握できます。 ポータルを使用しているときは、領域が少なくなっている共有に関するアラートを受信します。

### <a name="use-powershell"></a>PowerShell の使用

クラウド オペレーターは、PowerShell の `Get-AzsStorageShare` コマンドレットを使用することで、共有のストレージ容量を監視できます。 このコマンドレットは、各共有の合計領域、割り当て済み領域、および空き領域をバイト単位で返します。

![例:共有の空き領域を返す](media/azure-stack-manage-storage-shares/free-space.png)

- **合計容量**: 共有で使用できる合計容量 (バイト単位)。 この領域は、ストレージ サービスによって保持されるデータとメタデータ用に使用されます。
- **使用済み容量**: テナント データとそれに関連付けられているメタデータを格納するファイルのすべてのエクステントで使用されるデータの量 (バイト単位)。

### <a name="use-the-administrator-portal"></a>管理者ポータルを使用する

クラウド オペレーターは、管理者ポータルを使用して、すべての共有のストレージ容量を確認できます。

1. 管理者ポータル `https://adminportal.local.azurestack.external` にサインインします。
2. **[すべてのサービス]** > **[ストレージ]** > **[ファイル共有]** を選択してファイル共有の一覧を開きます。そこで使用状況情報を確認できます。

    ![例:Azure Stack Hub 管理者ポータルでのストレージ ファイル共有](media/azure-stack-manage-storage-shares/storage-file-shares.png)

   - **Total**: 共有で使用できる合計容量 (バイト単位)。 この領域は、ストレージ サービスによって保持されるデータとメタデータ用に使用されます。
   - **使用済み**: テナント データとそれに関連付けられているメタデータを格納するファイルのすべてのエクステントで使用されるデータの量 (バイト単位)。

::: moniker-end
::: moniker range=">=azs-2002"

## <a name="monitor-volumes"></a>ボリュームを監視する

PowerShell または管理者ポータルを使用してボリュームを監視することで、空き領域が少なくなった場合にそれを把握できます。 ポータルを使用しているときは、領域が少なくなっているボリュームに関するアラートを受信します。

### <a name="use-powershell"></a>PowerShell の使用

クラウド オペレーターは、PowerShell の `Get-AzsVolume` コマンドレットを使用して、ボリュームのストレージ容量を監視できます。 このコマンドレットでは、各ボリュームの合計領域と空き領域がギガバイト (GB) 単位で返されます。

![例:ボリュームの空き領域を返す](media/azure-stack-manage-storage-shares/listvolumespowershell.png)

- **合計容量**: 共有で使用できる合計領域 (GB 単位)。 この領域は、ストレージ サービスによって保持されるデータとメタデータ用に使用されます。
- **残りの容量**:テナント データとそれに関連付けられているメタデータを格納するための空き領域 (GB 単位)。

### <a name="use-the-administrator-portal"></a>管理者ポータルを使用する

クラウド オペレーターは、管理者ポータルを使用して、すべてのボリュームのストレージ容量を確認できます。

1. Azure Stack Hub 管理者ポータル (`https://adminportal.local.azurestack.external`) にサインインします。
2. **[すべてのサービス]** > **[ストレージ]** > **[ボリューム]** を選択してボリュームの一覧を開きます。そこで使用状況情報を確認できます。

    ![例:Azure Stack Hub 管理者ポータルでのストレージ ボリューム](media/azure-stack-manage-storage-shares/listvolumes.png)

   - **Total**: ボリューム上の使用可能な合計領域。 この領域は、ストレージ サービスによって保持されるデータとメタデータ用に使用されます。
   - **使用済み**: テナント データとそれに関連付けられているメタデータを格納するファイルのすべてのエクステントで使用されるデータの量。

::: moniker-end

### <a name="storage-space-alerts"></a>記憶域スペースのアラート

管理者ポータルを使用しているときは、領域が少なくなっているボリュームに関するアラートを受信します。

> [!IMPORTANT]
> クラウド オペレーターは、共有が完全に使用されないようにする必要があります。 共有が 100% 使用されると、その共有に対するストレージ サービスは機能しなくなります。 使用率が 100% である共有で空き領域を復旧して操作を復元するには、Microsoft サポートに問い合わせる必要があります。

* **警告**:ファイル共有の使用率が 80% を超えると、管理者ポータルで "*警告*" アラートを受信します。

  ![例:Azure Stack Hub 管理者ポータルの [警告] アラート](media/azure-stack-manage-storage-shares/alert-warning.png)

* **[高]** :ファイル共有の使用率が 90% を超えると、管理者ポータルで "*重大*" アラートを受信します。

  ![例:Azure Stack Hub 管理者ポータルの [重大] アラート](media/azure-stack-manage-storage-shares/alert-critical.png)

* **詳細の表示**: 管理者ポータルでアラートの詳細を開いて、軽減策を確認できます。

  ![例:Azure Stack Hub 管理者ポータルのアラートの詳細の表示](media/azure-stack-manage-storage-shares/alert-details.png)

## <a name="manage-available-space"></a>使用可能な領域を管理する

ボリュームの領域を解放する必要がある場合は、最も影響が少ない方法を先に使用してください。 たとえば、マネージド ディスクの移行を選択する前に領域の回収を試してください。  

### <a name="reclaim-capacity"></a>容量の回収

削除されているテナント アカウントによって使用されている容量を回収できます。 この容量は、データ[保有期間](azure-stack-manage-storage-accounts.md#set-the-retention-period)に達すると自動的に回収されますが、今すぐ回収する操作を実行できます。

詳細については、「[Azure Stack Hub ストレージ アカウントを管理する](azure-stack-manage-storage-accounts.md#reclaim)」の「容量の回収」セクションを参照してください。

::: moniker range="<azs-1910"

### <a name="migrate-a-container-between-volumes"></a>ボリューム間でコンテナーを移行する

"*このオプションは、Azure Stack Hub 統合システムにのみ適用されます。* "

テナントの使用パターンによっては、一部のテナントの共有が他の共有よりも多くの領域を使用することがあります。 これにより、一部の共有は、比較的使用されていない他の共有よりも前に領域が不足する可能性があります。

一部の BLOB コンテナーを別の共有に手動で移行することで、過剰使用されている共有の領域を解放できます。 複数の小さいコンテナーを、それらすべてを保持できる容量がある 1 つの共有に移行できます。 移行を使用して、"*空の*" コンテナーを移動できます。 空のコンテナーとは、VM 用のディスクが含まれていないコンテナーです。

移行すると、すべてのコンテナーの BLOB が新しい共有で統合されます。

- コンテナーがオーバーフロー モードに入り、BLOB が別のボリュームに配置されている場合、新しい共有には、移行するコンテナーのすべての BLOB を保持するのに十分な容量が必要です。 これには、別の共有に配置されている BLOB も含まれます。

- PowerShell コマンドレット `Get-AzsStorageContainer` は、コンテナーの初期ボリュームで使用されている領域のみを識別します。 このコマンドレットでは、別のボリューム上に配置されている BLOB によって使用されている領域は識別されません。 そのため、コンテナーのフル サイズがはっきりしないことがあります。 新しい共有でのコンテナーの統合によって、新しい共有がオーバーフロー状態になり、データが別の共有に配置される可能性があります。 その結果、共有を再調整する必要が生じる場合があります。

- 特定のリソース グループへのアクセス許可がないために、PowerShell を使用してオーバーフロー データが配置されている別のボリュームのクエリを実行できない場合は、それらのリソース グループとコンテナーの所有者と協力して、移行するデータの合計容量を把握したうえでデータを移行します。  

> [!IMPORTANT]
> コンテナーの BLOB の移行は、PowerShell の使用を要求するオフライン操作です。 移行が完了するまで、移行中のコンテナーのすべての BLOB はオフラインのままであり、使用することはできません。 また、進行中の移行作業がすべて完了するまで、Azure Stack Hub をアップグレードすることは避けてください。

#### <a name="migrate-containers-by-using-powershell"></a>PowerShell を使用してコンテナーを移行する

1. [Azure PowerShell のインストールと構成](/powershell/azure/)が行われていることを確認します。 詳細については、「[Azure PowerShell を使用した Azure リソースの管理](/azure/azure-resource-manager/management/manage-resources-powershell)」を参照してください。
2. コンテナーを調べて、移行する予定の共有にどのようなデータがあるかを把握します。 ボリューム内の移行に最適な候補コンテナーを識別するには、`Get-AzsStorageContainer` コマンドレットを使用します。

   ```powershell  
   $farm_name = (Get-AzsStorageFarm)[0].name
   $shares = Get-AzsStorageShare -FarmName $farm_name
   $containers = Get-AzsStorageContainer -ShareName $shares[0].ShareName -FarmName $farm_name
   ```
   $containers を確認します。

   ```powershell
   $containers
   ```

   ![例: $containers](media/azure-stack-manage-storage-shares/containers.png)

3. 移行するコンテナーを保持するのに最適な移行先の共有を特定します。

   ```powershell
   $destinationshare = ($shares | Sort-Object FreeCapacity -Descending)[0]
   ```

   $destinationshares を確認します。

   ```powershell 
   $destinationshares
   ```

   ![例: $destination 共有](media/azure-stack-manage-storage-shares/examine-destinationshares.png)

4. コンテナーの移行を開始します。 移行は非同期です。 最初の移行が完了する前に、別のコンテナーの移行を開始する場合は、ジョブ ID を使用してそれぞれの状態を追跡します。

   ```powershell
   $job_id = Start-AzsStorageContainerMigration -StorageAccountName $containers[0].Accountname -ContainerName $containers[0].Containername -ShareName $containers[0].Sharename -DestinationShareUncPath $destinationshares[0].UncPath -FarmName $farm_name
   ```

   $jobId を調べます。 次の例の *d62f8f7a-8b46-4f59-a8aa-5db96db4ebb0* を、調べるジョブ ID に置き換えてください。

   ```powershell
   $jobId
   d62f8f7a-8b46-4f59-a8aa-5db96db4ebb0
   ```

5. ジョブ ID を使用して、移行ジョブの状態を確認します。 コンテナーの移行が完了すると、**MigrationStatus** が *Completed* に設定されます。

   ```powershell 
   Get-AzsStorageContainerMigrationStatus -JobId $job_id -FarmName $farm_name
   ```

   ![移行の状態を示すスクリーンショット。](media/azure-stack-manage-storage-shares/migration-status1.png)

6. 実行中の移行ジョブを取り消すことができます。 取り消された移行ジョブは、非同期的に処理されます。 $jobid を使用して、取り消しを追跡できます。

   ```powershell
   Stop-AzsStorageContainerMigration -JobId $job_id -FarmName $farm_name
   ```

   ![例:ロールバック状態](media/azure-stack-manage-storage-shares/rollback.png)

7. 移行の状態が *取り消し済み* になるまで、手順 6 のコマンドをもう一度実行できます。  

    ![取り消された移行の状態の例を示すスクリーンショット。](media/azure-stack-manage-storage-shares/cancelled.png)

### <a name="move-vm-disks"></a>VM ディスクを移動する

"*このオプションは、Azure Stack Hub 統合システムにのみ適用されます。* "

領域を管理する最も極端な方法には、VM ディスクの移動が伴います。 接続されているコンテナー (VM ディスクを含むコンテナー) の移動は複雑であるため、この操作を実行する場合は Microsoft サポートに問い合わせてください。

::: moniker-end
::: moniker range=">=azs-1910"

### <a name="migrate-a-managed-disk-between-volumes"></a>ボリューム間でマネージド ディスクを移行する

"*このオプションは、Azure Stack Hub 統合システムにのみ適用されます。* "

テナントの使用パターンによっては、一部のテナントのボリュームで他のボリュームよりも多くの領域が使用されることがあります。 その結果、比較的使用されていないボリュームよりも前に領域が不足する可能性があります。

一部のマネージド ディスクを別のボリュームに手動で移行することで、過剰使用されているボリュームの領域を解放できます。 複数のマネージド ディスクを、それらすべてを保持できる容量がある単一の共有に移行できます。 移行を使用して、"*オフライン*" マネージド ディスクを移動します。 オフライン マネージド ディスクとは、VM に接続されていないディスクのことです。

> [!IMPORTANT]
> マネージド ディスクの移行は、PowerShell の使用が要求されるオフライン操作です。 移行ジョブを開始する前に、移行の候補ディスクを、それらを所有している VM から切断する必要があります (移行ジョブの完了後に再接続できます)。 移行が完了するまでは、移行するすべてのマネージド ディスクをオフラインのままにしておく必要があり、それらを使用することはできません。これに該当しない場合、移行ジョブは中止され、移行されなかったすべてのディスクは元のボリュームに残ります。 また、進行中の移行作業がすべて完了するまで、Azure Stack Hub をアップグレードすることは避けてください。

#### <a name="to-migrate-managed-disks-using-powershell"></a>PowerShell を使用してマネージド ディスクを移行する

1. Azure PowerShell がインストールおよび構成されていることを確認します。 PowerShell 環境の構成方法については、「[PowerShell for Azure Stack Hub をインストールする](azure-stack-powershell-install.md)」をご覧ください。 Azure Stack Hub にサインインする場合は、[オペレーター環境の構成と Azure Stack Hub へのサインイン](azure-stack-powershell-configure-admin.md)に関するページを参照してください。
2. マネージド ディスクを調べて、移行を計画するボリューム上のディスクを把握します。 ボリュームの移行に最適な候補ディスクを識別するには、`Get-AzsDisk` コマンドレットを使用します。

   ```powershell  
   $ScaleUnit = (Get-AzsScaleUnit)[0]
   $StorageSubSystem = (Get-AzsStorageSubSystem -ScaleUnit $ScaleUnit.Name)[0]
   $Volumes = (Get-AzsVolume -ScaleUnit $ScaleUnit.Name -StorageSubSystem $StorageSubSystem.Name | Where-Object {$_.VolumeLabel -Like "ObjStore_*"})
   $SourceVolume  = ($Volumes | Sort-Object RemainingCapacityGB)[0]
   $VolumeName = $SourceVolume.Name.Split("/")[2]
   $VolumeName = $VolumeName.Substring($VolumeName.IndexOf(".")+1)
   $MigrationSource = "\\SU1FileServer."+$VolumeName+"\SU1_"+$SourceVolume.VolumeLabel
   $Disks = Get-AzsDisk -Status All -SharePath $MigrationSource | Select-Object -First 10
   ```
   次に、$disks を調べます。

   ```powershell
   $Disks
   ```

   ![例: $Disks](media/azure-stack-manage-storage-shares/disks.png)

3. 移行するディスクが保持されている最善のボリュームを識別します。

   ```powershell
   $DestinationVolume  = ($Volumes | Sort-Object RemainingCapacityGB -Descending)[0]
   $VolumeName = $DestinationVolume.Name.Split("/")[2]
   $VolumeName = $VolumeName.Substring($VolumeName.IndexOf(".")+1)
   $MigrationTarget = "\\SU1FileServer."+$VolumeName+"\SU1_"+$DestinationVolume.VolumeLabel
   ```

4. マネージド ディスクの移行を開始します。 移行は非同期です。 最初の移行が完了する前に、別のディスクの移行を開始する場合は、ジョブ名を使用してそれぞれの状態を追跡します。

   ```powershell
   $jobName = "MigratingDisk"
   Start-AzsDiskMigrationJob -Disks $Disks -TargetShare $MigrationTarget -Name $jobName
   ```

5. ジョブ名を使用して、移行ジョブの状態を確認します。 ディスクの移行が完了すると、**MigrationStatus** が **Completed** に設定されます。

   ```powershell 
   $job = Get-AzsDiskMigrationJob -Name $jobName
   ```

   ![例:移行の状態](media/azure-stack-manage-storage-shares/diskmigrationjob.png)
   
   1 つの移行ジョブで複数のマネージド ディスクを移行する場合でも、ジョブのサブタスクを確認できます。

   ```powershell 
   $job.Subtasks
   ```

   ![例:移行サブタスクの状態](media/azure-stack-manage-storage-shares/diskmigrationsubtask.png)

6. 実行中の移行ジョブを取り消すことができます。 取り消された移行ジョブは、非同期的に処理されます。 移行ジョブの状態が **Canceled** であることを確認するまで、ジョブ名を使用して取り消しを追跡できます。

   ```powershell
   Stop-AzsDiskMigrationJob -Name $jobName
   ```

   ![例:取り消し済みの状態](media/azure-stack-manage-storage-shares/diskmigrationstop.png)

### <a name="distribute-unmanaged-disks"></a>アンマネージド ディスクを分散させる

"*このオプションは、Azure Stack Hub 統合システムにのみ適用されます。* "

領域を管理する最も極端な方法には、アンマネージド ディスクの移動が伴います。 テナントで多数のアンマネージド ディスクが 1 つのコンテナーに追加された場合、コンテナーが "*オーバーフロー*" モードに入る前に、コンテナーの使用済み容量の合計が、保持しているボリュームの使用可能容量を超える可能性があります。 単一のコンテナーによってボリュームの領域が使い果たされるのを防ぐために、テナントは、1 つのコンテナーの既存のアンマネージド ディスクを別のコンテナーに分散させることができます。 接続されているコンテナー (VM ディスクを含むコンテナー) の分散は複雑であるため、この操作を実行する場合は Microsoft サポートに問い合わせてください。

::: moniker-end

## <a name="next-steps"></a>次のステップ

ユーザーへの VM の提供の詳細については、「[Azure Stack Hub のストレージ容量を管理する](./tutorial-offer-services.md)」を参照してください。