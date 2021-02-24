---
title: Azure Stack Hub Capacity Planner | Microsoft Docs
description: Azure Stack Hub Capacity Planner を使用し、Azure Stack Hub デプロイのコンピューティング リソースの割り当てを表示する方法について説明します。
services: azure-stack
documentationcenter: ''
author: prchint
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/31/2019
ms.author: patricka
ms.reviewer: prchint
ms.lastreviewed: 05/31/2019
ms.openlocfilehash: eabc000ad42ae8cb25853fb06f6cfba517e36af3
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97872672"
---
# <a name="azure-stack-hub-capacity-planner---modular-data-center-mdc"></a>Azure Stack Hub Capacity Planner - Modular Data Center (MDC)

Azure Stack Hub Capacity Planner は、コンピューティング リソースのさまざまな割り当てがハードウェア オファリングのさまざまな選択に対してどのように適合するかを示すスプレッドシートです。 

## <a name="worksheet-descriptions"></a>ワークシートの説明
次の表では、Azure Stack Hub Capacity Planner の各ワークシートについて説明します。Azure Stack Capacity Planner は [https://aka.ms/azstackcapacityplanner](https://aka.ms/azstackcapacityplanner) からダウンロードできます。 

|ワークシート名|説明|
|-----|-----|
|Version-Disclaimer|この計算ツールの目的、バージョン番号、およびリリース日。|
|Instructions|仮想マシン (VM) のコレクションのキャパシティ プランニングをモデル化するための詳細な手順。|
|DefinedSolutionSKUs|最大 5 つのハードウェア定義を含む表。 エントリは例です。 検討中のシステム構成に合わせて詳細を変更してください。|
|DefineByVMFootprint|VM のサイズと数が異なる構成を比較して適切なハードウェア SKU を見つけます。|
|DefineByWorkloadFootprint|Azure Stack Hub ワークロードのコレクションを作成して、適切なハードウェア SKU を検索します。|
|  |  |

## <a name="definedsolutionskus-instructions"></a>DefinedSolutionSKUs の手順
このワークシートには、最大 5 つのハードウェア定義の例が含まれています。 検討中のシステム構成に合わせて詳細を変更してください。

### <a name="hardware-selections-provided-by-authorized-hardware-partners"></a>ハードウェアの認定パートナーによって提供されるハードウェアの選択
Azure Stack Hub は、ソリューション パートナーによってインストールされたソフトウェアが統合されたシステムとして提供されます。 ソリューション パートナーは、自身が権限を持つバージョンの Azure Stack Hub キャパシティ プランニング ツールを提供します。 ソリューション容量の最終的な検討には、それらのツールを使用してください。

### <a name="multiple-ways-to-model-computing-resources"></a>コンピューティング リソースをモデル化する複数の方法
Azure Stack Hub Capacity Planner 内のリソース モデリングは、Azure Stack Hub VM のサイズによって異なります。 VM のサイズ範囲は、最小の Basic 0 から最大の Standard_Fsv2 までです。 2 つの異なる方法でコンピューティング リソースの割り当てをモデル化できます。

- 特定のハードウェア オファリングを選択し、さまざまなリソースのどの組み合わせが適合するかを確認します。 

- VM 割り当ての特定の組み合わせを作成し、使用可能などのハードウェア SKU がこの VM 構成をサポートできるかを Azure Resource 計算ツールに表示させます。

このツールには、VM リソースを割り当てる 2 つの方法 (VM リソース割り当ての 1 つのコレクションとして、または最大 6 つの異なるワークロード構成のコレクションとして) が用意されています。 各ワークロード構成は、使用可能な VM リソースの異なる割り当てを含めることができます。 次のセクションでは、これらの各割り当てモデルを作成して使用するための詳細な手順を説明します。 非バック グラウンドのシェードされたセルに格納されている値、またはこのワークシートの SKU プルダウン リスト内の値のみ変更する必要があります。 シェードされたセル内で行われた変更によってリソースの計算が損なわれる場合があります。


## <a name="definebyvmfootprint-instructions"></a>DefineByVMFootprint の手順
さまざまなサイズと数の VM の 1 つのコレクションを使用してモデルを作成するには、 **[DefineByVMFootprint]** タブを選択し、次の手順に従います。

1. このワークシートの右上隅で、指定されたプルダウン リスト ボックス コントロールを使用して、各ハードウェア システム (SKU) にインストールするサーバーの初期の数 (4 から 16) を選択します。 このサーバーの数は、リソース割り当てモデルの使用可能なリソース全体への影響を確認するために、モデリング プロセス中にいつでも変更できます。
2. 1 つの特定のハードウェア構成に対してさまざまな VM リソース割り当てをモデル化する場合、ページの右上隅の **[現在の SKU]** ラベルのすぐ下にある青のプルダウン リスト ボックスを探します。 このリスト ボックスをプルダウンし、目的のハードウェア SKU を選択します。
3. さまざまなサイズの VM のモデルへの追加を開始する準備が整いました。 特定の VM タイプを含めるには、その VM エントリの左側の青枠で囲まれたボックスに数量の値を入力します。

   > [!NOTE]
   > VM ストレージの合計とは、VM のデータ ディスクの総容量 (サポートされているディスク数に単一ディスクの最大容量 [1 TB] を乗算したもの) のことです。 使用可能なストレージ構成テーブルは、構成評価指標に基づいて設定済みであるため、各 Azure Stack Hub VM に必要なレベルのストレージ リソースを選択できます。 ただし重要な点として、使用可能なストレージ構成テーブルは必要に応じて追加または変更できることに注意することが重要です。<br><br>各 VM は、最初に割り当てられたローカルの一時ストレージから始まります。 一時ストレージのシン プロビジョニングを反映するために、可能な最大一時ストレージ容量など、ドロップダウン メニューのいずれかでローカルの一時的な数他の数に変更できます。

4. VM を追加すると、使用可能な SKU リソースの変更を示すグラフが表示されます。 これにより、モデリング プロセス中にさまざまなサイズと VM の数量を追加することの効果を確認できます。 変更の影響を把握するもう 1 つの方法は、使用可能な VM のすぐ下に一覧表示される **[使用済み]** と **[空き容量]** の数を確認することです。 これらの数値は、現在選択されているハードウェア SKU に基づく推定値を反映しています。
5. 一連の VM を作成した後は、ページの右上隅の **[現在の SKU]** ラベルのすぐ下にある **[Suggested SKU]\(推奨 SKU\)** をクリックして、推奨ハードウェア SKU を検索できます。 このボタンを使用すると、VM 構成を変更し、どのハードウェアがそれぞれの構成をサポートするかを確認できます。


## <a name="definebyworkloadfootprint-instructions"></a>DefineByWorkloadFootprint の手順
Azure Stack Hub ワークロードのコレクションを使用してモデルを作成するには、 **[DefineByWorkloadFootprint]** タブを選択し、次の一連の手順に従います。 Azure Stack Hub のワークロードの作成には、使用可能な VM リソースを使用します。   

> [!TIP]
> Azure Stack Hub VM に指定されたストレージ サイズを変更するには、前のセクションの手順 3 の「注意」を参照してください。

1. このワークシートの右上隅で、指定されたプルダウン リスト ボックス コントロールを使用して、各ハードウェア システム (SKU) にインストールするサーバーの初期の数 (4 から 16) を選択します。
2. 1 つの特定のハードウェア構成に対してさまざまな VM リソース割り当てをモデル化する場合、ページの右上隅の **[現在の SKU]** ラベルのすぐ下にある青のプルダウン リスト ボックスを探します。 このリスト ボックスをプルダウンし、目的のハードウェア SKU を選択します。
3. 前のセクションの手順 3 で説明したように、**[DefineByVMFootprint]** ページの目的の Azure Stack Hub VM それぞれに対して適切なストレージ サイズを選択します。 VM あたりのストレージ サイズは、DefineByVMFootprint シートで定義されます。
4. **[DefineByWorkloadFootprint]** ページの左上から順に、最大 6 つの異なる種類のワークロードの構成を作成します。 そのワークロードに含まれる VM の種類それぞれについて数量を入力します。 このためには、そのワークロードの名前のすぐ下にある列に数値を入力します。 この特定の構成でサポートされるワークロードの種類を反映するために、ワークロード名を変更することができます。
5. **[数量]** ラベルのすぐ下にある、該当する列の下部に値を入力することによって、ワークロードの種類それぞれの特定の数量を含めることができます。
6. ワークロードの種類と数量を作成したら、ページの右上隅の **[現在の SKU]** ラベルのすぐ下にある **[Suggested SKU]\(推奨 SKU\)** を選択します。 これにより、このワークロード構成全体をサポートするために十分なリソースを含む最小の SKU が表示されます。
7. ハードウェア SKU に選択したサーバーの数を変更するか、VM の割り当てまたはワークロード構成内の数量を変更することで、さらなるモデリングを達成することができます。 関連付けられているグラフに、変更が全体的なリソースの消費量にどのように影響するかを示すフィードバックが即座に表示されます。
8. 目的の変更が完了したら、**[Suggested SKU]\(推奨 SKU\)** ボタンをもう一度クリックします。新しい構成に推奨される SKU が表示されます。 ドロップダウン メニューを選択して目的の SKU を選択することもできます。

## <a name="next-steps"></a>次のステップ
[Azure Stack Hub のデータ センター統合に関する考慮事項](../operator/azure-stack-datacenter-integration.md)。
