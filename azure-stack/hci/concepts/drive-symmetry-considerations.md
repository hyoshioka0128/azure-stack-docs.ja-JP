---
title: Azure Stack HCI のドライブの対称性に関する考慮事項
description: このトピックでは、ドライブの対称性の制約について説明し、サポート対象およびサポート対象外の構成の例を示します。
author: khdownie
ms.author: v-kedow
ms.topic: conceptual
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 09/01/2020
ms.openlocfilehash: 545a0b90ad938a172a184748780974ba7403f19f
ms.sourcegitcommit: 4af79f4fa2598d57c81e994192c10f8c6be5a445
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/10/2020
ms.locfileid: "89742401"
---
# <a name="drive-symmetry-considerations-for-azure-stack-hci"></a>Azure Stack HCI のドライブの対称性に関する考慮事項

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

Azure Stack HCI は、すべてのサーバーにまったく同じドライブがある場合に最も効果的に機能します。

実際には、Azure Stack HCI は長年にわたって実行され、組織のニーズの拡大に合わせてスケーリングするように設計されているため、これは現実的ではない場合があることを Microsoft では認識しています。 現在、十分な容量の 3 TB のハード ドライブを購入できても、来年にはそれほど小さい容量のドライブを見つけることができなくなる可能性もあります。 そのため、ある程度うまく組み合わせることが想定され、サポートされています。 ただし、対称である方が常に望ましいことに留意してください。

このトピックでは、その制約について説明し、サポート対象およびサポート対象外の構成の例を示します。

## <a name="constraints"></a>制約

### <a name="type"></a>種類

すべてのサーバーは、[ドライブの種類](choose-drives.md#drive-types)が同じである必要があります。

たとえば、1 台のサーバーに NVMe が存在する場合は、"*すべて*" に NVMe が存在する必要があります。

### <a name="number"></a>Number

すべてのサーバーは、種類別のドライブの数が同じである必要があります。

たとえば、1 台のサーバーに 6 台の SSD が存在する場合は、"*すべて*" に 6 台の SSD が存在する必要があります。

   > [!NOTE]
   > 障害発生時またはドライブの追加や削除時に、ドライブ数が一時的に異なることはかまいません。

### <a name="model"></a>モデル

可能な限り、同じモデルおよびファームウェアのバージョンのドライブを使用することをお勧めします。 できない場合は、可能な限り類似するドライブを慎重に選択してください。 Azure Stack HCI は IO を均等に分散し、モデルに基づいて区別しないため、全く異なるパフォーマンスまたは耐久性の特性を持つ同じ種類のドライブを組み合わせることはお勧めしていません (一方がキャッシュであり、もう一方が容量である場合を除きます)。

   > [!NOTE]
   > 類似の SATA と SAS のドライブを組み合わせることはかまいません。

### <a name="size"></a>サイズ

可能な限り、同じサイズのドライブを使用することをお勧めします。 サイズが異なる容量ドライブを使用すると、一部の容量が使用できなくなる場合があり、サイズが異なるキャッシュ ドライブを使用すると、キャッシュのパフォーマンスが向上しない場合があります。 詳細については、次のセクションを参照してください。

   > [!WARNING]
   > サーバー間で容量ドライブのサイズが異なると、容量が残される可能性があります。

## <a name="understand-capacity-imbalance"></a>理解: 容量の不均衡

Azure Stack HCI は、ドライブ間とサーバー間での容量の不均衡に対して堅牢です。 不均衡が深刻な場合でも、すべてが継続して機能します。 ただし、いくつかの要因によっては、すべてのサーバーでは使用できない容量については、使用できなくなる場合があります。

これが発生する理由を確認するために、次の簡略化された図で検討します。 色付きのボックスはそれぞれ、ミラー化されたデータの 1 つのコピーを表します。 たとえば、A、A'、A'' とマークされているボックスは、同じデータの 3 つのコピーです。 サーバーのフォールト トレランスを利用するには、これらのコピーを異なるサーバーに格納する*必要があります*。

### <a name="stranded-capacity"></a>残される容量

図に示すように、サーバー 1 (10 TB) とサーバー 2 (10 TB) はいっぱいです。 サーバー 3 には大きいドライブがあるため、その合計容量はより大きいです (15 TB)。 ただし、サーバー 3 にさらに多くの 3 方向ミラー データを格納するには、既にいっぱいになっているサーバー 1 とサーバー 2 にもコピーが必要になります。 サーバー 3 の残りの 5 TB 容量は使用できません。これが *"残される"* 容量です。

![3 方向ミラー、3 台のサーバー、残される容量](media/drive-symmetry-considerations/Size-Asymmetry-3N-Stranded.png)

### <a name="optimal-placement"></a>最適な配置

逆に、3 方向ミラーの回復性を備えた 10 TB、10 TB、10 TB、15 TB の容量の 4 台のサーバーでは、図に示すように、使用可能なすべての容量を使用するような方法でコピーを有効に配置*できます*。 これが可能である場合は常に、記憶域スペース ダイレクト アロケーターは最適な配置を見つけて使用するので、残される容量がありません。

![3 方向ミラー、4 台のサーバー、残される容量なし](media/drive-symmetry-considerations/Size-Asymmetry-4N-No-Stranded.png)

サーバーの数、回復性、容量の不均衡の深刻度、およびその他の要因は、残される容量があるかどうかに影響します。 **最も賢明で一般的な規則は、すべてのサーバーで使用できる容量のみが使用可能であることが保証されると想定することです。**

## <a name="understand-cache-imbalance"></a>理解: キャッシュの不均衡

Azure Stack HCI は、ドライブ間とサーバー間でのキャッシュの不均衡に対して堅牢です。 不均衡が深刻な場合でも、すべてが継続して機能します。 さらに、Azure Stack HCI では、使用可能なすべてのキャッシュが常に最大限まで使用されます。

ただし、異なるサイズのキャッシュ ドライブを使用すると、キャッシュのパフォーマンスの向上が一様にも予測どおりにもならない場合があります。キャッシュ ドライブが大きい[ドライブのバインド](cache.md#server-side-architecture)への IO のみがパフォーマンスの向上を示す可能性があります。 Azure Stack HCI は、IO をバインド間で均等に分散し、キャッシュと容量の比率に基づいて区別しません。

![キャッシュの不均衡](media/drive-symmetry-considerations/Cache-Asymmetry.png)

   > [!TIP]
   > キャッシュ バインドの詳細については、[キャッシュの概要](cache.md)に関する記事を参照してください。

## <a name="example-configurations"></a>構成例

サポート対象およびサポート対象外の構成を次に示します。

### <a name="image-typeicon-sourcemediadrive-symmetry-considerationssupportedpng-borderfalse-supported-different-models-between-servers"></a>:::image type="icon" source="media/drive-symmetry-considerations/supported.png" border="false"::: サポート対象: サーバー間で異なるモデル

最初の 2 台のサーバーでは、NVMe モデル "X" が使用されていますが、3 台目のサーバーでは、非常に類似する NVMe モデル "Z" が使用されています。

| サーバー 1                    | サーバー 2                    | サーバー 3                    |
|-----------------------------|-----------------------------|-----------------------------|
| 2 台 x NVMe モデル X (キャッシュ)    | 2 台 x NVMe モデル X (キャッシュ)    | 2 台 x NVMe モデル Z (キャッシュ)    |
| 10 台 x SSD モデル Y (容量) | 10 台 x SSD モデル Y (容量) | 10 台 x SSD モデル Y (容量) |

これはサポートされています。

### <a name="image-typeicon-sourcemediadrive-symmetry-considerationssupportedpng-borderfalse-supported-different-models-within-server"></a>:::image type="icon" source="media/drive-symmetry-considerations/supported.png" border="false"::: サポート対象: サーバー内で異なるモデル

すべてのサーバーで、非常に類似する HDD モデル "Y" と "Z" の異なる組み合わせが使用されています。 どのサーバーにも合計 10 台の HDD があります。

| サーバー 1                   | サーバー 2                   | サーバー 3                   |
|----------------------------|----------------------------|----------------------------|
| 2 台 x SDD モデル X (キャッシュ)    | 2 台 x SDD モデル X (キャッシュ)    | 2 台 x SDD モデル X (キャッシュ)    |
| 7 台 x HDD モデル Y (容量) | 5 台 x HDD モデル Y (容量) | 1 台 x HDD モデル Y (容量) |
| 3 台 x HDD モデル Z (容量) | 5 台 x HDD モデル Z (容量) | 9 台 x HDD モデル Z (容量) |

これはサポートされています。

### <a name="image-typeicon-sourcemediadrive-symmetry-considerationssupportedpng-borderfalse-supported-different-sizes-across-servers"></a>:::image type="icon" source="media/drive-symmetry-considerations/supported.png" border="false"::: サポート対象: サーバー間で異なるサイズ

最初の 2 台のサーバーでは 4 TB の HDD が使用されていますが、3 台目のサーバーでは非常に類似した 6 TB の HDD が使用されています。

| サーバー 1                | サーバー 2                | サーバー 3                |
|-------------------------|-------------------------|-------------------------|
| 2 台 x 800 GB NVMe (キャッシュ) | 2 台 x 800 GB NVMe (キャッシュ) | 2 台 x 800 GB NVMe (キャッシュ) |
| 4 台 x 4 TB HDD (容量) | 4 台 x 4 TB HDD (容量) | 4 台 x 6 TB HDD (容量) |

これはサポートされていますが、残される容量が発生します。

### <a name="image-typeicon-sourcemediadrive-symmetry-considerationssupportedpng-borderfalse-supported-different-sizes-within-server"></a>:::image type="icon" source="media/drive-symmetry-considerations/supported.png" border="false"::: サポート対象: サーバー内で異なるサイズ

すべてのサーバーで、1.2 TB の SSD と、非常に類似する 1.6 TB SSD の異なる組み合わせが使用されています。 どのサーバーにも合計 4 台の SSD があります。

| サーバー 1                 | サーバー 2                 | サーバー 3                 |
|--------------------------|--------------------------|--------------------------|
| 3 台 x 1.2 TB SSD (キャッシュ)   | 2 台 x 1.2 TB SSD (キャッシュ)   | 4 台 x 1.2 TB SSD (キャッシュ)   |
| 1 台 x 1.6 TB SSD (キャッシュ)   | 2 台 x 1.6 TB SSD (キャッシュ)   | -                        |
| 20 台 x 4 TB HDD (容量) | 20 台 x 4 TB HDD (容量) | 20 台 x 4 TB HDD (容量) |

これはサポートされています。

### <a name="image-typeicon-sourcemediadrive-symmetry-considerationsunsupportedpng-borderfalse-not-supported-different-types-of-drives-across-servers"></a>:::image type="icon" source="media/drive-symmetry-considerations/unsupported.png" border="false"::: サポート対象外: サーバー間で異なる種類のドライブ

サーバー 1 には NVMe が存在しますが、他のサーバーには存在しません。

| サーバー 1            | サーバー 2            | サーバー 3            |
|---------------------|---------------------|---------------------|
| 6 台 x NVMe (キャッシュ)    | -                   | -                   |
| -                   | 6 台 x SSD (キャッシュ)     | 6 台 x SSD (キャッシュ)     |
| 18 台 x HDD (容量) | 18 台 x HDD (容量) | 18 台 x HDD (容量) |

これはサポートされていません。 ドライブの種類は、すべてのサーバーで同じである必要があります。

### <a name="image-typeicon-sourcemediadrive-symmetry-considerationsunsupportedpng-borderfalse-not-supported-different-number-of-each-type-across-servers"></a>:::image type="icon" source="media/drive-symmetry-considerations/unsupported.png" border="false"::: サポート対象外: サーバー間で種類別の数が異なる

サーバー 3 には他より多くのドライブがあります。

| サーバー 1            | サーバー 2            | サーバー 3            |
|---------------------|---------------------|---------------------|
| 2 台 x NVMe (キャッシュ)    | 2 台 x NVMe (キャッシュ)    | 4 台 x NVMe (キャッシュ)    |
| 10 台 x HDD (容量) | 10 台 x HDD (容量) | 20 台 x HDD (容量) |

これはサポートされていません。 種類別のドライブの数は、すべてのサーバーで同じである必要があります。

### <a name="image-typeicon-sourcemediadrive-symmetry-considerationsunsupportedpng-borderfalse-not-supported-only-hdd-drives"></a>:::image type="icon" source="media/drive-symmetry-considerations/unsupported.png" border="false"::: サポート対象外: HDD ドライブのみ

すべてのサーバーに HDD ドライブのみが接続されています。

|サーバー 1|サーバー 2|サーバー 3|
|-|-|-|
|18 台 x HDD (容量) |18 台 x HDD (容量)|18 台 x HDD (容量)|

これはサポートされていません。 各サーバーに接続されている 2 台以上のキャッシュ ドライブ (NvME または SSD) を追加する必要があります。

## <a name="summary"></a>まとめ

要約すると、クラスター内のすべてのサーバーには同じ種類のドライブが存在し、種類別の数が同じである必要があります。 上記の考慮事項を使用して、必要に応じてドライブ モデルとドライブ サイズをうまく組み合わせることがサポートされています。

| 制約 | State |
|--|--|
| すべてのサーバーで同じ種類のドライブ | **必須** |
| すべてのサーバーで種類別の数が同じ | **必須** |
| すべてのサーバーで同じドライブ モデル | 推奨 |
| すべてのサーバーで同じドライブ サイズ | 推奨 |

## <a name="next-steps"></a>次のステップ

関連情報については、以下も参照してください。

- [Azure Stack HCI をデプロイする前に](../deploy/before-you-start.md)
- [ドライブの選択](choose-drives.md)
