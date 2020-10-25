---
title: Azure Stack HCI の既知の問題
description: このトピックでは、Azure Stack HCI の既知の問題について説明します。
author: myoungerman
ms.author: v-myoung
ms.topic: troubleshooting
ms.date: 07/21/2020
ms.openlocfilehash: 21e56fbb34c89446b0dd0e395046a9c851f391dc
ms.sourcegitcommit: f2d16c3148da50d679940e024267995b85ce6332
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/28/2020
ms.locfileid: "91407754"
---
# <a name="known-issues-for-azure-stack-hci"></a>Azure Stack HCI の既知の問題

>適用対象:Azure Stack HCI バージョン 20H2

このトピックでは、Azure Stack HCI の既知の問題について説明します

- オンプレミス クラスターと Azure Stack HCI クラウド サービスの間の接続では、[Azure Private Link](https://azure.microsoft.com/services/private-link) はまだサポートされていません。
- Azure Stack HCI クラウド サービスは、現在、選択したリージョンでのみ使用できます。
- Azure Stack HCI は 2020 年 7 月 21 日に発表されました。 この Azure portal ユーザー インターフェイスの拡張機能が世界各地で表示されるまでに数日かかる場合があります。 この期間中は、クラスターが Azure portal の既定のリソース ページとして表示される場合があります。 ご不便をおかけして申し訳ございませんでした。
- オペレーティング システムに対話形式でログインしたときに、"Azure Stack HCI へようこそ" というようこそ画面は、チェコ語、ハンガリー語、オランダ語、ポーランド語、スウェーデン語、およびトルコ語 (ロケール コード cs-cz、hu-hu、nl-nl、pl-pl、pt-pt、sv-se、tr-tr) ではまだローカライズされていません。 さらに、英語以外のすべての言語では、"[Y]es/[N]o" などの入力プロンプトは値 "Y" と "N" のみを受け入れます。このプロンプト自体が、誤解を招くようなローカライズされた形 (たとえば、フランス語では [O]ui/[N]on、ドイツ語では [J]a/[N]ein) で表示される場合も同様です。
- 入れ子になった仮想化を使用して Azure Stack HCI を評価する場合、仮想化ベースのセキュリティに対する Azure Stack HCI の依存関係が原因で、"仮想化サポートが有効になっていない Hyper-V をインストールできません" というようなエラーが表示されることがあります。 次の 2 つの回避策を使用できます。(1) Hyper-V 第 1 世代の仮想マシンを代わりに使用します。または、(2) Hyper-V 機能を VM の VHDX にオフラインで挿入します。 Azure Stack HCI ノードとして機能する各 VM の電源がオフになっている状態で、ホストから次の PowerShell コマンドを実行します。**Install-WindowsFeature -Vhd \<path> -Name Hyper-V, RSAT-Hyper-V-Tools, Hyper-V-Powershell**
