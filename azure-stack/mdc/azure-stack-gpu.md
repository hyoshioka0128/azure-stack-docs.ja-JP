---
title: Azure Stack 上の GPU VM | Microsoft Docs
description: Azure Stack での GPU コンピューティングのリファレンスです。
services: azure-stack
documentationcenter: ''
author: PatAltimore
manager: femila
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/02/2020
ms.author: patricka
ms.reviewer: kivenkat
ms.lastreviewed: 01/02/2020
ms.openlocfilehash: 1fb2542680f0fa6d7c020db77ce5e59663d87f9c
ms.sourcegitcommit: 9b0e1264ef006d2009bb549f21010c672c49b9de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98255607"
---
# <a name="gpu-vms-on-azure-stack"></a>Azure Stack 上の GPU VM 

*適用対象:Azure Stack 統合システム* 

このトピックでは、Azure Stack Hub 上で GPU VM を管理する方法について説明します。


## <a name="partitioned-gpu-vm-size"></a>パーティション分割された GPU VM サイズ 

NVv4 シリーズの仮想マシンには [AMD Radeon Instinct MI25](https://www.amd.com/en/products/professional-graphics/instinct-mi25) GPU が搭載されています。 NVv4 シリーズでは、Azure Stack Hub に、部分的な GPU を備えた仮想マシンが導入されています。 このサイズは、GPU アクセラレータによるグラフィックス アプリケーションと仮想デスクトップに使用できます。 NVv4 仮想マシンでは現在、Windows ゲスト オペレーティング システムのみがサポートされています。 

| サイズ | vCPU | メモリ:GiB | 一時ストレージ (SSD) GiB | GPU | GPU メモリ: GiB | 最大データ ディスク数 | 最大 NIC 数 | 
| --- | --- | --- | --- | --- | --- | --- | --- |   
| Standard_NV4as_v4 |4 |14 |88 | 1/8 | 2 | 4 | 2 | 

## <a name="patch-and-update-fru-behavior-of-vms"></a>パッチ、更新プログラムの適用時、および FRU の際の VM の動作 

GPU VM では、修正プログラムと更新プログラム (PnU) の適用時や、Azure Stack Hub のハードウェア交換 (FRU) などの操作中にダウンタイムが発生することがあります。 次の表は、これらのアクティビティ中に見られる VM の状態と、それらの操作後に VM を再び使用できるようにするためにユーザーが実行できる手動のアクションを示しています。 

| 操作 | PnU - 簡易更新 | PnU - 完全更新、OEM 更新 | FRU | 
| --- | --- | --- | --- | 
| VM の状態  | 更新中および更新後は手動の起動操作を行わないと使用できません | 更新中は使用できません。 更新後に手動操作を行うと使用できるようになります | 更新中は使用できません。 更新後に手動操作を行うと使用できるようになります| 
| 手動操作 | 更新中に VM を使用できるようにする必要があるときは、使用可能な GPU パーティションがある場合は、ポータルで [再起動] ボタンをクリックすることで VM を再起動できます。 VM は、更新後にポータルから [再起動] ボタンを使用して再起動する必要があります | 更新中は VM を使用することはできません。 更新の完了後、[停止] ボタンを使用して VM の停止と割り当て解除を行い、[開始] ボタンを使用してバックアップを起動する必要があります | 更新中は VM を使用することはできません。更新の完了後、[停止] ボタンを使用して VM の停止と割り当て解除を行い、[開始] ボタンを使用してバックアップを起動する必要があります。| 

## <a name="guest-driver-installation"></a>ゲスト ドライバーのインストール 

[こちら](/azure/virtual-machines/windows/n-series-amd-driver-setup)のドキュメントでは、NVv4 GPU-P 対応 VM 内の AMD ゲスト ドライバーの設定と、ドライバーのインストールを確認する手順が説明されています。 

## <a name="next-steps"></a>次のステップ 

[Azure Stack VM の機能](azure-stack-vm-considerations.md)