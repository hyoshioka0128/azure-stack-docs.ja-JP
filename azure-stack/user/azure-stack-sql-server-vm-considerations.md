---
title: Azure Stack Hub で SQL Server のパフォーマンスを最適化する
description: この記事では、Azure Stack Hub VM における SQL Server のパフォーマンス向上と最適化に役立つ、SQL サーバーのベスト プラクティスを紹介します。
author: bryanla
ms.topic: article
ms.date: 04/02/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 01/14/2020
ms.openlocfilehash: b1aae77d4247c80ace7a94e2b034f29f10220b55
ms.sourcegitcommit: 52c934f5eeb5fcd8e8f2ce3380f9f03443d1e445
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/07/2021
ms.locfileid: "97974032"
---
# <a name="sql-server-best-practices-to-optimize-performance-in-azure-stack-hub"></a>Azure Stack Hub におけるパフォーマンスを最適化するための SQL Server のベスト プラクティス

この記事では、Microsoft Azure Stack Hub 仮想マシン (VM) における SQL Server の最適化とパフォーマンス向上を行うための、SQL サーバーのベスト プラクティスを紹介します。 Azure Stack Hub VM で SQL Server を実行するときは、オンプレミスのサーバー環境で SQL Server に適用されるデータベース パフォーマンス チューニング オプションと同じものを使用します。 Azure Stack Hub クラウド内のリレーショナル データベースのパフォーマンスは、VM のファミリ サイズやデータ ディスクの構成などの多くの要因に左右されます。

SQL Server イメージを作成するときは、[VM を Azure Stack Hub ポータルにプロビジョニングすることを検討してください](/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-server-provision)。 Azure Stack Hub 管理者ポータルの Marketplace 管理から SQL IaaS 拡張機能をダウンロードし、任意の SQL Server VM イメージをダウンロードします。 これには、SQL Server 2016 SP1、SQL Server 2016 SP2、SQL Server 2017 が含まれます。

> [!NOTE]  
> この記事では、グローバルな Azure portal を使用して SQL Server VM をプロビジョニングする方法について説明しますが、ガイダンスは次の点を除いて Azure Stack Hub にも適用されます。オペレーティング システム ディスクに SSD は利用できません。また、ストレージ構成に若干の違いがあります。

VM イメージでは、SQL Server の場合、ライセンス持ち込み (BYOL) のみを使用できます。 Windows Server の場合、既定のライセンス モデルは従量課金制 (PAYG) です。 VM の Windows Server ライセンス モデルの詳細については、「[Azure Stack Hub Marketplace 内の Windows Server に関する FAQ](../operator/azure-stack-windows-server-faq.md#what-about-other-vms-that-use-windows-server-such-as-sql-or-machine-learning-server)」記事を参照してください。  

この記事では、Azure Stack Hub VM で SQL Server の "*最適な*" パフォーマンスを得ることに焦点を絞っています。 ワークロードの要求が厳しくない場合は、推奨される最適化がすべて必要になるわけではありません。 各推奨事項を評価するときに、パフォーマンスのニーズとワークロードのパターンを考慮してください。

> [!NOTE]  
> Azure VM における SQL Server のパフォーマンスのガイダンスについては、[こちらの記事](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-performance)を参照してください。

## <a name="checklist-for-sql-server-best-practices"></a>SQL サーバーのベスト プラクティスに関するチェックリスト

Azure Stack Hub VM で SQL Server の最適なパフォーマンスを実現するためのチェックリストを以下に示します。


|領域|最適化|
|-----|-----|
|VM サイズ |[DS3](azure-stack-vm-sizes.md) 以上 (SQL Server Enterprise Edition の場合)。<br><br>[DS2](azure-stack-vm-sizes.md) 以上 (SQL Server Standard Edition および Web Edition の場合)。|
|ストレージ |[Premium Storage](azure-stack-acs-differences.md) をサポートする VM ファミリを使用します。|
|ディスク |少なくとも 2 つのデータ ディスク (1 つはログ ファイル用、もう 1 つはデータ ファイルと TempDB 用) を使用し、容量のニーズに基づいてディスク サイズを選択します。 SQL Server のインストール中に、これらのディスクに既定のデータ ファイルの場所を設定します。<br><br>データベース ストレージまたはログに、オペレーティング システム ディスクまたは一時ディスクを使用することは避けます。<br>複数の Azure データ ディスクをストライプし、記憶域スペースを使用して IO スループットを向上させます。<br><br>ドキュメントに記載されている割り当てサイズでフォーマットします。|
|I/O|データ ファイルの瞬時初期化を有効にします。<br><br>一定の増分値が比較的小さい (64 MB から 256 MB) データベースで自動拡張を制限します。<br><br>データベースで自動圧縮を無効にします。<br><br>オペレーティング システム ディスクではなく、データ ディスク上に既定のバックアップおよびデータベース ファイルの場所を設定します。<br><br>ロックされたページを有効にします。<br><br>SQL Server サービス パックと累積的な更新プログラムを適用します。|
|機能固有|Blob ストレージに直接バックアップします (使用している SQL Server のバージョンでサポートされている場合)。|
|||

これらの最適化を行う *方法* と *理由* については、以下のセクションに記載されている詳細とガイダンスをご確認ください。

## <a name="vm-size-guidance"></a>VM サイズのガイダンス

パフォーマンス重視のアプリケーションでは、次の [VM サイズ](azure-stack-vm-sizes.md)をお勧めします。

- **SQL Server Enterprise エディション:** DS3 以上

- **SQL Server Standard エディションおよび Web エディション:** DS2 以上

Azure Stack Hub の場合、DS と DS_v2 の VM ファミリ シリーズの間にパフォーマンスの違いはありません。

## <a name="storage-guidance"></a>ストレージのガイダンス

Azure Stack Hub の DS シリーズ (および DSv2 シリーズ) の VM では、オペレーティング システム ディスクおよびデータ ディスクの最大スループット (IOPS) が提供されます。 DS シリーズまたは DSv2 シリーズの VM では、選択したディスクの種類やサイズに関係なく、オペレーティング システム ディスク用に最大 1,000 IOPS、データ ディスクごとに最大 2,300 IOPS が提供されます。

データ ディスクのスループットは、VM のファミリ シリーズに基づいて一意に決定されます。 [こちらの記事を参照](azure-stack-vm-sizes.md)して、VM のファミリ シリーズごとのデータ ディスクのスループットを特定することができます。

> [!NOTE]  
> 実稼働ワークロードの場合は、DS シリーズまたは DSv2 シリーズの VM を選択し、オペレーティング システム ディスクとデータ ディスク上で使用可能な最大 IOPS が提供されるようにします。

Azure Stack Hub でストレージ アカウントを作成する場合、geo レプリケーション オプションは無効です。この機能は Azure Stack Hub で使用できないためです。

## <a name="disks-guidance"></a>ディスクのガイダンス

Azure Stack Hub VM には、次の 3 種類のメイン ディスクがあります。

- **オペレーティング システム ディスク:** Azure Stack Hub VM を作成すると、プラットフォームによって、オペレーティング システム ディスク用に少なくとも 1 つのディスク (**C** ドライブとしてラベル付けされる) が VM に接続されます。 このディスクは、ページ BLOB としてストレージに格納されている VHD です。

- **一時ディスク:** Azure Stack Hub VM には、一時ディスクと呼ばれる別のディスク (**D** ドライブとしてラベル付けされる) が含まれています。 これは、スクラッチ領域に使用できるノード上のディスクです。

- **データ ディスク:** 追加のディスクをデータ ディスクとして VM に接続することができます。これらのディスクは、ページ BLOB としてストレージに格納されます。

次のセクションでは、これらの異なるディスクの使用に関する推奨事項について説明します。

### <a name="operating-system-disk"></a>オペレーティング システム ディスク

オペレーティング システム ディスクは、実行中のバージョンのオペレーティング システムとして起動およびマウントできる VHD であり、 **C** ドライブとしてラベル付けされます。

### <a name="temporary-disk"></a>一時ディスク

**D** ドライブとしてラベル付けされている一時ストレージ ドライブは永続的ではありません。 失いたくないデータは、**D** ドライブに格納しないでください。 これには、ご使用のユーザー データベース ファイルとユーザー トランザクション ログ ファイルが含まれます。

各データ ディスクではデータ ディスクごとに最大 2,300 の IOPS が提供されるため、データ ディスクに TempDB を格納することをお勧めします。

### <a name="data-disks"></a>データ ディスク

- **データ ファイルとログ ファイル用のデータ ディスクの使用。** ディスク ストライピングを使用していない場合は、Premium Storage をサポートする VM の 2 つのデータ ディスクを使用します。1 つのディスクにはログ ファイルが含まれ、もう 1 つのディスクにはデータ ファイルと TempDB ファイルが含まれます。 各データ ディスクでは、VM ファミリに応じて、多くの IOPS が提供されます (「[Azure Stack Hub でサポートされている VM のサイズ](azure-stack-vm-sizes.md)」を参照してください)。 記憶域スペースなどのディスク ストライピング技法を使用している場合は、データ ファイルとログ ファイルをすべて同じドライブ上に配置します (TempDB を含む)。 この構成では、特定の時間にどのファイルで必要になるかに関係なく、SQL Server で最大数の IOPS を使用できます。

> [!NOTE]  
> SQL Server VM をポータルからプロビジョニングするとき、必要に応じてストレージの構成を編集することができます。 Azure Stack Hub では、実際の構成に応じて 1 つまたは複数のディスクが構成されます。 複数のディスクは、1 つの記憶域プールにまとめられます。 この構成では、データ ファイルとログ ファイルが一緒に格納されます。

- **ディスク ストライピング:** スループットを向上させるために、データ ディスクをさらに追加し、ディスク ストライピングを使用できます。 必要なデータ ディスク数を決定するには、ログ ファイルと、データおよび TempDB ファイルに必要な IOPS 数を分析します。 IOPS の制限は、VM のサイズではなく、VM シリーズ ファミリに基づくデータ ディスクあたりの制限であることに注意してください。 ただし、ネットワーク帯域幅の制限は、VM のサイズに基づきます。 詳細については、「[Azure Stack Hub でサポートされている仮想マシンのサイズ](azure-stack-vm-sizes.md)」に示されている表を参照してください。 次のガイドラインに従ってください。

  - Windows Server 2012 以降の場合は、次のガイドラインに従った[記憶域スペース](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh831739(v=ws.11))を使用します｡

    1. パーティションの配置不良を原因とするパフォーマンスの低下を回避するため、インターリーブ (ストライプ サイズ) をオンライン トランザクション処理 (OLTP) ワークロードに対しては 64 KB (65,536 バイト)、データ ウェアハウス ワークロードに対しては 256 KB (262,144 バイト) に設定します。 これは､PowerShell を使って設定する必要があります｡

    2. カラム数 = 物理ディスク数を設定します｡ 8 つを超えるディスクを構成する場合は、PowerShell を使用します (サーバー マネージャー UI ではない)。

       たとえば、次の PowerShell では、インターリーブ サイズが 64 KB、列数が 2 に設定されている新しい記憶域プールが作成されます｡

       ```powershell  
       $PoolCount = Get-PhysicalDisk -CanPool $True
       $PhysicalDisks = Get-PhysicalDisk | Where-Object {$_.FriendlyName -like "*2" -or $_.FriendlyName -like "*3"}

       New-StoragePool -FriendlyName "DataFiles" -StorageSubsystemFriendlyName "Storage Spaces*" -PhysicalDisks $PhysicalDisks | New-VirtualDisk -FriendlyName "DataFiles" -Interleave 65536 -NumberOfColumns 2 -ResiliencySettingName simple -UseMaximumSize |Initialize-Disk -PartitionStyle GPT -PassThru |New-Partition -AssignDriveLetter -UseMaximumSize |Format-Volume -FileSystem NTFS -NewFileSystemLabel "DataDisks" -AllocationUnitSize 65536 -Confirm:$false
       ```

- 負荷予測に基づいて、ご使用の記憶域プールに関連付けるディスク数を決定します。 接続できるデータ ディスクの数は VM サイズによって異なることに注意してください。 詳細については、「[Azure Stack Hub でサポートされている VM のサイズ](azure-stack-vm-sizes.md)」を参照してください。
- データ ディスクで利用可能な最大 IOPS を確保するには、[VM のサイズ](azure-stack-vm-sizes.md)でサポートされる最大数のデータ ディスクを追加し、ディスク ストライピングを使用することをお勧めします。
- **NTFS アロケーション ユニット サイズ:** データ ディスクをフォーマットするときは、データ ファイルとログ ファイルに加えて TempDB にも 64 KB アロケーション ユニット サイズを使用することをお勧めします。
- **ディスク管理プラクティス:** データ ディスクを削除する場合、変更中は SQL Server サービスを停止してください。 また、パフォーマンスが向上しないため、ディスク上のキャッシュ設定は変更しないでください。

> [!WARNING]  
> これらの操作中に SQL Server の停止を怠ると、データベースが破損するおそれがあります。

## <a name="io-guidance"></a>I/O のガイダンス

- 初期ファイル割り当てに必要な時間を短縮するために、ファイルの瞬時初期化を有効にすることを検討します。 ファイルの瞬時初期化を利用するには、SQL Server (MSSQLSERVER) サービス アカウントに **SE_MANAGE_VOLUME_NAME** を付与し、 **[ボリュームの保守タスクを実行]** セキュリティ ポリシーにそのサービス アカウントを追加します。 Azure の SQL Server プラットフォーム イメージを使用している場合、既定のサービス アカウント (**NT Service\MSSQLSERVER**) は、 **[ボリュームの保守タスクを実行]** セキュリティ ポリシーに追加されません。 つまり、SQL Server Azure プラットフォーム イメージでは、ファイルの瞬時初期化は有効になりません。 **[ボリュームの保守タスクを実行]** セキュリティ ポリシーに SQL Server サービス アカウントを追加したら、SQL Server サービスを再起動します。 この機能を使用する場合、セキュリティに関する考慮事項があります。 詳細については、「 [データベース ファイルの初期化](/sql/relational-databases/databases/database-instant-file-initialization?view=sql-server-ver15&preserve-view=true)」をご覧ください。
- **自動拡張** は、予想外の増加に付随するものです。 自動拡張を使用して、データやログの増加に日常的に対処しないでください。 自動拡張を使用する場合は、**Size** スイッチを使用してファイルを事前に拡張します。
- パフォーマンスに悪影響を及ぼすおそれのある不要なオーバーヘッドを回避するために、 **自動圧縮** が無効になっていることを確認します。
- 既定のバックアップ ファイルとデータベース ファイルの場所を設定します。 この記事では、推奨事項を使用し、[サーバーのプロパティ] ウィンドウで変更を行います。 手順については、「 [データ ファイルとログ ファイルの既定の場所の表示または変更 (SQL Server Management Studio)](/sql/database-engine/configure-windows/view-or-change-the-default-locations-for-data-and-log-files?view=sql-server-ver15&preserve-view=true)」をご覧ください。 次のスクリーンショットは、これらの変更を行う場所を示しています。

    > ![既定の場所を表示または変更する](./media/sql-server-vm-considerations/image1.png)

- ロックされたページを有効にして、IO とページング アクティビティを減らします。 詳細については、「 [Lock Pages in Memory オプションの有効化 (Windows)](/sql/database-engine/configure-windows/enable-the-lock-pages-in-memory-option-windows?view=sql-server-ver15&preserve-view=true)」をご覧ください。

- バックアップを含め、Azure Stack Hub との間での転送時にデータ ファイルを圧縮することを検討します。

## <a name="feature-specific-guidance"></a>機能固有のガイダンス

一部のデプロイでは、より高度な構成手法を使用することで、パフォーマンスがさらに向上する場合があります。 パフォーマンスの向上を実現する際に役立つ SQL Server 機能をいくつか以下にリストします。

- **Azure** **ストレージにバックアップします。** Azure Stack Hub VM で実行されている SQL Server のバックアップを実行する際は、SQL Server Backup to URL を使用できます。 SQL Server 2012 SP1 CU2 以降で使用できるこの機能は、接続されているデータ ディスクにバックアップする場合に推奨されます。

    Azure Storage を使用してバックアップまたは復元を行うときは、「[SQL Server Backup to URL に関するベスト プラクティスとトラブルシューティング](/sql/relational-databases/backup-restore/sql-server-backup-to-url-best-practices-and-troubleshooting?view=sql-server-ver15&preserve-view=true)」と「[Microsoft Azure に格納されたバックアップからの復元](/sql/relational-databases/backup-restore/restoring-from-backups-stored-in-microsoft-azure?view=sql-server-2017&preserve-view=true)」に記載されている推奨事項に従ってください。 [Azure VM での SQL Server の自動バックアップ](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-automated-backup)を使用して、バックアップを自動化することもできます。

-   **Azure Stack Hub ストレージにバックアップします。** Azure Storage へのバックアップと同様の方法で、Azure Stack Hub ストレージにバックアップできます。 SQL Server Management Studio (SSMS) 内でバックアップを作成する場合は、構成情報を手動で入力する必要があります。 SSMS を使用してストレージ コンテナーや Shared Access Signature を作成することはできません。 SSMS は、Azure Stack Hub サブスクリプションではなく、Azure サブスクリプションにのみ接続されます。 代わりに、ストレージ アカウント、コンテナー、Shared Access Signature を Azure Stack Hub ポータルで、あるいは PowerShell を使用して作成する必要があります。


    ![SQL Server のバックアップ](./media/sql-server-vm-considerations/image3.png)

    > [!NOTE]  
    > Shared Access Signature は Azure Stack Hub ポータルから取得する SAS トークンであり、文字列の先頭に "?" はありません。 ポータルからコピー関数を使用する場合は、トークンの先頭の "?" を削除し、 SQL Server 内で機能するようにする必要があります。

    バックアップ先を設定して SQL Server で構成したら、Azure Stack Hub Blob ストレージにバックアップすることができます。

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub でのサービスの使用またはアプリの作成](azure-stack-considerations.md)
