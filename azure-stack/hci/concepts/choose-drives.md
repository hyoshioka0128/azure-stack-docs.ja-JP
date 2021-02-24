---
title: Azure Stack HCI のドライブを選択する
description: Azure Stack HCI のドライブを選択する方法。
author: khdownie
ms.author: v-kedow
ms.topic: conceptual
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 09/01/2020
ms.openlocfilehash: e5069745a1ada76f37a9dea78eeeafa18a481cff
ms.sourcegitcommit: 296c95cad20ed62bdad0d27f1f5246bfc1c81d5e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2020
ms.locfileid: "93064652"
---
# <a name="choose-drives-for-azure-stack-hci"></a>Azure Stack HCI のドライブを選択する

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

このトピックでは、Azure Stack HCI のパフォーマンスおよび容量の要件を満たすドライブを選択する方法に関するガイダンスを提供します。

## <a name="drive-types"></a>ドライブの種類

Azure Stack HCI は現在、次の 4 種類のドライブで動作します。

| ドライブの種類 | 説明 |
|----------------------|--------------------------|
|![PMem](media/choose-drives/pmem-100px.png)|**PMem** とは永続メモリを意味します。これは、低待機時間かつ高パフォーマンスの新しい種類の記憶域です。|
|![NVMe](media/choose-drives/NVMe-100-px.png)|**NVMe** (Non-Volatile Memory Express) とは、PCIe バスに直接接続されたソリッドステート ドライブを指します。 一般的なフォーム ファクターは、2.5 インチU.2、PCIe Add-In-Card (AIC)、および M.2 です。 NVMe では、PMem を除いて、現在サポートされている他の種類のドライブよりも高い IOPS と IO スループットおよび低待機時間が実現します。|
|![SSD](media/choose-drives/SSD-100-px.png)|**SSD** は、従来の SATA または SAS 経由で接続されるソリッドステート ドライブを指します。|
|![HDD](media/choose-drives/HDD-100-px.png)|**HDD** は、大容量の記憶域容量を提供する、回転式の磁気ハード ディスク ドライブを指します。|

## <a name="built-in-cache"></a>ビルトイン キャッシュ

Azure Stack HCI は、組み込みのサーバー側キャッシュを特徴とします。 これは、大容量で永続的な、リアルタイムの読み取りおよび書き込みキャッシュです。 複数の種類のドライブを使用するデプロイでは、"最速" の種類のすべてのドライブを使用するように自動的に構成されます。 残りのドライブは、キャパシティとして使用されます。

詳細については、[Azure Stack HCI のキャッシュの概要](cache.md)に関する記事を参照してください。

## <a name="option-1--maximizing-performance"></a>オプション 1 – パフォーマンスの最大化

任意のデータに対するランダムな読み取りと書き込みで予測可能かつ一貫したミリ秒未満の待機時間を実現するか、非常に高い IOPS ([1300 万以上](https://techcommunity.microsoft.com/t5/storage-at-microsoft/the-new-hci-industry-record-13-7-million-iops-with-windows/ba-p/428314)を達成済み) または IO スループット (500 GB/秒以上の読み取りを達成済み) を実現するには、"オール フラッシュ" を選択する必要があります。

これを行うには複数の方法があります。

![この図は、容量用の NVMe のみ、キャッシュ用の NVMe と容量用の SSD、容量用の SSD のみなど、デプロイ オプションを示しています。](media/choose-drives/All-Flash-Deployment-Possibilities.png)

1. **NVMe のみ。** NVMe のみを使用すると、最も予測可能な低待機時間を含む、比類のないパフォーマンスが得られます。 すべてのドライブが同じモデルの場合、キャッシュはありません。 耐久性の高い NVMe モデルと耐久性の低い NVMe モデルを混在させ、後者に対する書き込みをキャッシュするように前者を構成することもできます ([セットアップが必要です](/windows-server/storage/storage-spaces/understand-the-cache#manual-configuration))。

2. **NVMe + SSD。** NVMe を SSD とともに使用すると、NVMe によって SSD への書き込みが自動的にキャッシュされます。 これにより、書き込みがキャッシュ内で結合され、必要に応じてのみデステージングされるため、SSD の消耗が軽減されます。 これにより、 NVMe に似た書き込み特性が得られる一方で、読み取りは同じく高速な SSD から直接提供されます。

3. **SSD のみ。** NVMe のみの場合と同様に、すべてのドライブが同じモデルの場合、キャッシュはありません。 耐久性の高いモデルと耐久性の低いモデルを混在させる場合は、後者に対する書き込みをキャッシュするように前者を構成することができます ([セットアップが必要です](/windows-server/storage/storage-spaces/understand-the-cache#manual-configuration))。

   >[!NOTE]
   > キャッシュなしで NVMe のみまたは SSD のみを使用する利点は、すべてのドライブから使用可能なストレージ容量を取得できることです。 キャッシュに "費やされる" 容量がないため、小規模のスケールで有用である場合があります。

## <a name="option-2--balancing-performance-and-capacity"></a>オプション 2 – パフォーマンスと容量のバランスを取る

さまざまなアプリケーションやワークロードがあり、パフォーマンス要件が厳しい環境や、非常に多くのストレージ容量を必要とする環境では、大容量の HDD に対して NVMe または SSD キャッシュを使用して "ハイブリッド" にする必要があります。

![この図は、キャッシュ用の NVMe と容量用の HDD、キャッシュ用の SSD と容量用の HDD、キャッシュ用の NVMe と容量用の SSD と HDD の混合など、可能なデプロイを示しています。](media/choose-drives/Hybrid-Deployment-Possibilities.png)

1. **NVMe + HDD** 。 NVMe ドライブは、読み取りと書き込みをキャッシュすることによって、その両方を高速化します。 読み取りのキャッシュにより、HDD は書き込みに集中することができます。 書き込みのキャッシュは、バーストを吸収し、必要な場合にのみ書き込みの結合とデステージングを可能にします。これは、HDD IOPS と IO スループットを最大化する、人為的にシリアル化された方法で行われます。 これにより、NVMe に似た書き込み特性が得られ、頻繁にまたは最近読み取られたデータに対して、NVMe に似た読み取り特性も得られます。

2. **SSD + HDD** 。 上記と同様に、SSD は、読み取りと書き込みをキャッシュすることによって、その両方を高速化します。 これにより、SSD に似た書き込み特性と、頻繁にまたは最近読み取られたデータに対して、SSD に似た読み取り特性が得られます。

    " *3 つすべて* " の種類のドライブ使用するという、もう 1 つ別の特殊なオプションがあります。

3. **NVMe + SSD + HDD。** 3 つすべての種類のドライブを使用する場合は、NVMe ドライブで SSD とHDD の両方に対してキャッシュが提供されます。 ここでの魅力的な点は、SSD 上にボリュームを作成し、HDD 上にボリュームを作成し、それらを同じクラスター内に並置して、すべて NVMe によって高速化できることです。 前者は "オール フラッシュ" デプロイとまったく同じであり、後者は上記の "ハイブリッド" デプロイとまったく同じです。 これは概念的には、大部分が独立した容量管理、障害および修復サイクルなどを備えた 2 つのプールを持つようなものです。

   >[!IMPORTANT]
   > SSD 階層を使用して、最もパフォーマンス要件の厳しいワークロードをオール フラッシュに配置することをお勧めします。

## <a name="option-3--maximizing-capacity"></a>オプション 3 – 容量の最大化

アーカイブ、バックアップ ターゲット、データ ウェアハウス、または "コールド" ストレージなど、大容量を必要とし、書き込みの頻度が低いワークロードの場合は、キャッシュ用の小数の SSD を、容量用の多数の大容量 HDD と組み合わせる必要があります。

![容量を最大化するためのデプロイ オプション](media/choose-drives/maximizing-capacity.png)

1. **SSD + HDD** 。 SSD は、読み取りと書き込みをキャッシュしてバーストを吸収し、SSD に似た書き込みパフォーマンスを提供し、後で HDD に対する最適化されたデステージングを行います。

>[!IMPORTANT]
>HDD のみを使用した構成はサポートされていません。 耐久性の低い SSD に対する耐久性の高い SSD のキャッシュはお勧めしません。

## <a name="sizing-considerations"></a>サイズに関する考慮事項

### <a name="cache"></a>キャッシュ

各サーバーには、少なくとも 2 つのキャッシュ ドライブ (冗長性のために必要な最小値) が必要です。 容量ドライブの数は、キャッシュ ドライブの数の倍数にすることをお勧めします。 たとえば、4 台のキャッシュ ドライブがある場合、7 台や 9 台よりも 8 台の容量ドライブ (1:2 の比率) を使用した方がより一貫性のあるパフォーマンスを得ることができます。

キャッシュは、お使いのアプリケーションとワークロードのワーキング セット、つまり、それらが常にアクティブに読み取りおよび書き込みを行っているすべてのデータに対応できるようにサイズ設定する必要があります。 それを超えるキャッシュ サイズの要件はありません。 HDD を使用したデプロイでは、適正な初期構成は容量の 10% です。たとえば、各サーバーに 16 TB (4 x 4 TB HDD) の容量がある場合、サーバーあたりのキャッシュは 1.6 TB (2 x 800 GB SSD) になります。 オール フラッシュのデプロイ、特に非常に[耐久性の高い](https://techcommunity.microsoft.com/t5/storage-at-microsoft/understanding-ssd-endurance-drive-writes-per-day-dwpd-terabytes/ba-p/426024) SSD を使用する場合は、容量の 5% 近くから開始するのが妥当な場合があります。たとえば、各サーバーに 28.8 TB (24 x 1.2 TB SSD) の容量がある場合、サーバーあたりのキャッシュは 1.5 TB (2 x 750 GB NVMe) になります。 キャッシュ ドライブは、後でいつでも追加または削除して調整できます。

### <a name="general"></a>全般

サーバーあたりの合計ストレージ容量は、約 400 テラバイト (TB) に制限することをお勧めします。 サーバーあたりのストレージ容量が大きいほど、ソフトウェア更新プログラムを適用する場合など、ダウンタイムや再起動後のデータの再同期に必要な時間が長くなります。 Windows Server 2019 の場合、記憶域プールあたりの現在の最大サイズは 4 ペタバイト (PB) (4,000 TB) です。

## <a name="next-steps"></a>次のステップ

詳細については、次のトピックも参照してください。

- [キャッシュについて](cache.md)
- [システム要件](system-requirements.md)
- [ドライブの対称性に関する考慮事項](drive-symmetry-considerations.md)
- [ボリュームの計画](plan-volumes.md)
- [フォールト トレランスとストレージの効率性](fault-tolerance.md)
- [永続メモリの理解と配置](/windows-server/storage/storage-spaces/deploy-pmem)
