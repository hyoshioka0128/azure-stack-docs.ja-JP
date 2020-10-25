---
title: Azure Stack HCI で CSV インメモリ読み取りキャッシュを使用する
description: このトピックでは、システム メモリを使用してパフォーマンスを向上させる方法について説明します。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 09/04/2020
ms.openlocfilehash: 84d5292c3f812402027b310954a021752276a799
ms.sourcegitcommit: 01dcda15d88c8d44b4918e2f599daca462a8e3d9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/05/2020
ms.locfileid: "89493803"
---
# <a name="use-the-csv-in-memory-read-cache-with-azure-stack-hci"></a>Azure Stack HCI で CSV インメモリ読み取りキャッシュを使用する

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

このトピックでは、システム メモリを使用して、Azure Stack HCI のパフォーマンスを向上させる方法について説明します。

Azure Stack HCI は、クラスターの共有ボリューム (CSV) のインメモリ読み取りキャッシュと互換性があります。 システム メモリを使用して読み取りをキャッシュすると、バッファーなし I/O を使用して VHD または VHDX ファイルにアクセスする Hyper-V などのアプリケーションのパフォーマンスを向上させることができます。 (バッファなし I/O とは、Windows キャッシュ マネージャーによってキャッシュされないすべての操作です。)

インメモリ キャッシュはサーバーローカルであるため、データの局所性が向上します。最近の読み取りは、仮想マシンが実行されているのと同じホスト上のメモリにキャッシュされるため、ネットワーク経由での読み取りの頻度が減少します。 この結果、待機時間が短くなり、ストレージのパフォーマンスが向上します。

CSV インメモリ読み取りキャッシュは、Azure Stack HCI の[記憶域プール キャッシュ](../concepts/cache.md)とは異なるので注意してください。

## <a name="planning-considerations"></a>計画に関する考慮事項

インメモリ読み取りキャッシュは、仮想デスクトップ インフラストラクチャ (VDI) など、大量の読み取りが発生するワークロードで最も効果的です。 反対に、ワークロードで書き込みが極めて大量に発生する場合は、キャッシュによってその価値を上回るオーバーヘッドが発生する場合があるため、キャッシュは無効にする必要があります。

CSV インメモリ読み取りキャッシュ用に、物理メモリの合計の最大 80% を使用できます。 仮想マシンのために十分なメモリを確保するよう注意してください。

  > [!NOTE]
  > DISKSPD や [VM Fleet](https://github.com/Microsoft/diskspd/tree/master/Frameworks/VMFleet) など、一部のマイクロベンチマーク ツールでは、CSV インメモリ読み取りキャッシュがない場合より、有効になっている場合に、より悪い結果が出ることがあります。 既定では、VM Fleet では、仮想マシンごとに 10 GiB の VHDX が 1 つ作成され (100 の VM に対して合計で約 1 TiB)、それらに対して*一様にランダム*な読み取りと書き込みが実行されます。 実際のワークロードとは異なり、読み取りは予測可能なパターンまたは反復パターンに従わないため、インメモリ キャッシュは効果がなく、オーバーヘッドが発生します。

## <a name="configuring-the-in-memory-read-cache"></a>インメモリ読み取りキャッシュを構成する

CSV インメモリ読み取りキャッシュは、同じ機能を持つ Azure Stack HCI、Windows Server 2019、Windows Server 2016 で使用できます。 Azure Stack HCI と Windows Server 2019 では、既定ではオンになっており、1 ギビバイト (GiB) が割り当てられます。 Windows Server 2016 では、既定ではオフになっています。

| OS バージョン          | 既定の CSV キャッシュ サイズ |
|---------------------|------------------------|
| Azure Stack HCI     | 1 GiB                  |
| Windows Server 2019 | 1 GiB                  |
| Windows Server 2016 | 0 (無効)           |

PowerShell を使用して割り当てられているメモリの量を確認するには、以下を実行します。

```PowerShell
(Get-Cluster).BlockCacheSize
```

返される値は、サーバーあたりのメビバイト (MiB) 数です。 たとえば、`1024` は 1 GiB を表します。

割り当てられているメモリの量を変更するには、PowerShell を使用してこの値を変更します。 たとえば、サーバーあたり 2 GiB を割り当てるには、以下を実行します。

```PowerShell
(Get-Cluster).BlockCacheSize = 2048
```

変更をただちに有効にするために、CSV ボリュームを一時停止して再開するか、サーバー間で移動します。 たとえば、次の PowerShell フラグメントを使用して、各 CSV を別のサーバー ノードに移動し、再び戻します。

```PowerShell
Get-ClusterSharedVolume | ForEach {
    $Owner = $_.OwnerNode
    $_ | Move-ClusterSharedVolume
    $_ | Move-ClusterSharedVolume -Node $Owner
}
```

## <a name="next-steps"></a>次のステップ

関連情報については、以下も参照してください。

- [記憶域プール キャッシュについて](../concepts/cache.md)
- [フェールオーバー クラスターでクラスターの共有ボリュームを使用する](/windows-server/failover-clustering/failover-cluster-csvs#enable-the-csv-cache-for-read-intensive-workloads-optional)
