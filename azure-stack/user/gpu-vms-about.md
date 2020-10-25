---
title: Azure Stack Hub 上のグラフィックス処理装置 (GPU) VM
description: Azure Stack Hub での GPU コンピューティングのリファレンスです。
author: mattbriggs
ms.author: mabrigg
ms.service: azure-stack
ms.topic: reference
ms.date: 07/07/2020
ms.reviewer: kivenkat
ms.lastreviewed: 07/07/2020
ms.openlocfilehash: bb945f69a972214823f366456967b71a013d98bf
ms.sourcegitcommit: 362081a8c19e7674c3029c8a44d7ddbe2deb247b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/09/2020
ms.locfileid: "91899841"
---
# <a name="graphics-processing-unit-gpu-virtual-machine-vm-on-azure-stack-hub"></a>Azure Stack Hub 上の GPU (グラフィックス処理装置) 仮想マシン (VM)

*適用対象:Azure Stack 統合システム*

この記事では、Azure Stack Hub のマルチノード システムでサポートされているグラフィックス処理装置 (GPU) のモデルについて説明します。 また、GPU で使用されるドライバーのインストール手順についてもご確認いただけます。 Azure Stack Hub で GPU がサポートされることにより、人工知能、トレーニング、推論、データ視覚化などのソリューションが可能になります。 AMD Radeon Instinct MI25 を使用することで、グラフィックを多用するアプリケーション (Autodesk AutoCAD など) をサポートできます。

パブリック プレビュー期間中は、3 つの GPU モデルから選択できます。 NVIDIA V100、NVIDIA T4、AMD MI25 の各 GPU で利用できます。 これらの物理 GPU は、次のように、Azure N-Series の仮想マシン (VM) の種類に対応しています。
- [NCv3](/azure/virtual-machines/ncv3-series)
- [NVv4 (AMD MI25)](/azure/virtual-machines/nvv4-series)
- NCas_v4

> [!IMPORTANT]  
> Azure Stack Hub での GPU のサポートは、現在パブリック プレビュー段階にあります。 プレビューに参加するには、[aka.ms/azurestackhubgpupreview](https://aka.ms/azurestackhubgpupreview) にあるフォームにご記入ください。
> このプレビュー バージョンはサービス レベル アグリーメントなしで提供されています。運用環境のワークロードに使用することはお勧めできません。 特定の機能はサポート対象ではなく、機能が制限されることがあります。 詳しくは、[Microsoft Azure プレビューの追加使用条件](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)に関するページをご覧ください。

## <a name="ncv3"></a>NCv3

NCv3 シリーズ VM は NVIDIA Tesla V100 GPU を備えています。 貯留層モデリング、DNA シーケンシング、タンパク質解析、モンテ カルロ シミュレーションをはじめとする従来の HPC ワークロードに、これらの最新の GPU を活用することができます。 

| サイズ | vCPU | メモリ:GiB | 一時ストレージ (SSD) GiB | GPU | GPU メモリ: GiB | 最大データ ディスク数 | 最大 NIC 数 |
|---|---|---|---|---|---|---|---|---|
| Standard_NC6s_v3    | 6  | 112 | 736  | 1 | 16 | 12 | 4 |
| Standard_NC12s_v3   | 12 | 224 | 1474 | 2 | 32 | 24 | 8 |
| Standard_NC24s_v3   | 24 | 448 | 2948 | 4 | 64 | 32 | 8 |

## <a name="nvv4"></a>NVv4

NVv4 シリーズの仮想マシンには [AMD Radeon Instinct MI25](https://www.amd.com/en/products/professional-graphics/instinct-MI25) GPU が搭載されています。 NVv4 シリーズでは、Azure Stack Hub に、部分的な GPU を備えた仮想マシンが導入されています。 このサイズは、GPU アクセラレータによるグラフィックス アプリケーションと仮想デスクトップに使用できます。 NVv4 仮想マシンでは現在、Windows ゲスト オペレーティング システムのみがサポートされています。 

| サイズ | vCPU | メモリ:GiB | 一時ストレージ (SSD) GiB | GPU | GPU メモリ: GiB | 最大データ ディスク数 | 最大 NIC 数 | 
| --- | --- | --- | --- | --- | --- | --- | --- |   
| Standard_NV4as_v4 |4 |14 |88 | 1/8 | 2 | 4 | 2 | 

## <a name="ncas_v4"></a>NCas_v4

この新しい NVIDIA T4 VM サイズを使用すると、比較的低負荷の ML、推論、および視覚化のワークロードを Azure Stack Hub で実行できます。 現在、この VM サイズはポータルからはデプロイ "*できません*"。代わりに、PowerShell または CLI を使用する必要があります。


| サイズ | vCPU | メモリ:GiB | GPU | GPU メモリ: GiB | 最大データ ディスク数 | 最大 NIC 数 | 
| --- | --- | --- | --- | --- | --- | --- |
| Standard_NC4as_v4 |4 |28 | 1 | 16 | 8 | 4 | 
| Standard_NC8as_v4 |8 |56 | 1 | 16 | 16 | 8 | 
| Standard_NC16as_v4 |16 |112 | 1 | 16 | 32 | 8 | 
| Standard_NC64as_v4 |64 |448 | 4 | 64 | 32 | 8 | 


## <a name="patch-and-update-fru-behavior-of-vms"></a>修正プログラム、更新プログラムの適用時、および FRU の際の VM の動作 

GPU VM では、修正プログラムと更新プログラム (PnU) の適用時や、Azure Stack Hub のハードウェア交換 (FRU) などの操作中にダウンタイムが発生することがあります。 次の表は、これらのアクティビティ中に見られる VM の状態と、それらの操作後に VM を再び使用できるようにするためにユーザーが実行できる手動のアクションを示しています。 

| 操作 | PnU - 簡易更新 | PnU - 完全更新、OEM 更新 | FRU | 
| --- | --- | --- | --- | 
| VM の状態  | 更新中および更新後は手動の起動操作を行わないと使用できません | 更新中は使用できません。 更新後に手動操作を行うと使用できるようになります | 更新中は使用できません。 更新後に手動操作を行うと使用できるようになります| 
| 手動操作 | 更新中に VM を使用できるようにする必要があるときは、使用可能な GPU パーティションがある場合は、ポータルで **[再起動]** ボタンをクリックすることで VM を再起動できます。 更新後はポータルで **[再起動]** ボタンを使用して VM を再起動します | 更新中は VM を使用することはできません。 更新の完了後、 **[停止]** ボタンを使用して VM の停止と割り当て解除を行い、[開始] ボタンを使用してバックアップを起動する必要があります | 更新中は VM を使用することはできません。更新の完了後、 **[停止]** ボタンを使用して VM の停止と割り当て解除を行い、 **[開始]** ボタンを使用してバックアップを起動する必要があります。| 

## <a name="guest-driver-installation"></a>ゲスト ドライバーのインストール 

### <a name="amd-mi25"></a>AMD MI25
「[Windows を実行している N シリーズ VM に AMD GPU ドライバーをインストールする](/azure/virtual-machines/windows/n-series-amd-driver-setup)」の記事では、NVv4 GPU-P 対応の VM 内に AMD Radeon Instinct MI25 のドライバーをインストールする手順と、ドライバーのインストールを検証する手順について説明しています。 この拡張機能は接続モードでのみ動作します。

### <a name="nvidia"></a>NVIDIA

VM で CUDA または GRID ワークロードを実行するには、NVIDIA ドライバーが必要です。 拡張機能を使用して VM に GRID ドライバーをインストールする前に、適切な GRID ライセンスをお持ちであること、また、ライセンス サーバーが設定されていることをご確認ください。 ライセンス サーバーを設定する方法については[こちら](https://docs.nvidia.com/grid/ls/latest/grid-license-server-user-guide/index.html)をご覧ください。 CUDA ドライバーにはライセンス サーバーは不要です。

NVIDIA CUDA ドライバーと GRID ドライバーは、手動で VM にインストールする必要があります。 Tesla CUDA ドライバーは、NVIDIA の[ダウンロード Web サイト](https://www.nvidia.com/Download/index.aspx)から入手できます。 必要なライセンスをお持ちの場合は、NVIDIA アプリケーション ハブから GRID ドライバーをダウンロードできます。

## <a name="next-steps"></a>次のステップ 

[Azure Stack VM の機能](azure-stack-vm-considerations.md)