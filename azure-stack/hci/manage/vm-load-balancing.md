---
title: 仮想マシンの負荷分散
description: このトピックでは、Azure Stack HCI および Windows Server で VM の負荷分散機能を構成する方法について説明します。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 1/14/2021
ms.openlocfilehash: 9caf8efdd8db46479cce3c5c4299fb5588c7d37a
ms.sourcegitcommit: 51ce5ba6cf0a377378d25dac63f6f2925339c23d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/14/2021
ms.locfileid: "98224763"
---
# <a name="virtual-machine-load-balancing"></a>仮想マシンの負荷分散

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019、Windows Server 2016

HCI デプロイに関する重要な考慮事項は、運用環境に移行するために必要な資本支出 (CapEx) です。 運用環境ではピーク トラフィック時の容量不足を防ぐために冗長性を追加することが一般的ですが、これによって CapEx が増加します。 多くの場合、クラスター内の一部のサーバーでホストされる仮想マシン (VM) が多くなり、他のサーバーは過少使用になることがあるため、冗長性が必要になります。

VM 負荷分散は、Azure Stack HCI、Windows Server 2019、および Windows Server 2016 では既定で有効になっていますが、クラスター内のサーバー使用率を最適化するための機能です。 過剰コミットされたサーバーが識別され、そのサーバーから過少コミットされたサーバーに VM がライブ移行されます。 障害ポリシー (アンチアフィニティ、障害ドメイン (サイト)、実行可能な所有者など) が尊重されます。

VM 負荷分散により、次のヒューリスティックに基づいてサーバーの負荷が評価されます。

- **現在のメモリ負荷:** メモリは、Hyper-V ホスト上の最も一般的なリソース制約です。
- **5 分間の平均 CPU 使用率:** クラスター内のどのサーバーも過剰コミット状態になりにくいようにします。

## <a name="how-does-vm-load-balancing-work"></a>VM 負荷分散はどのように動作するか

新しいサーバーをクラスターに追加すると、VM 負荷分散が自動的に行われます。定期的な負荷分散を繰り返し実行するように構成することもできます。

### <a name="when-a-new-server-is-added-to-a-cluster"></a>新しいサーバーがクラスターに追加されたとき

新しいサーバーをクラスターに参加させると、次の順序で、VM 負荷分散機能により、既存サーバーから新しく追加されたサーバーに容量が自動的に分散されます。

1. クラスター内の既存サーバーのメモリ負荷と CPU 使用率が評価されます。
2. しきい値を超えるすべてのサーバーが識別されます。
3. メモリ負荷と CPU 使用率が最も高いサーバーが識別され、分散の優先順位が決められます。
4. VM は、しきい値を超えるサーバーから、クラスターに新しく追加されたサーバーにライブ移行されます (ダウンタイムは発生しません)。

:::image type="content" source="media/vm-load-balancing/server-added.png" alt-text="クラスターに追加される新しいサーバーを示す画像" border="false"::: 

### <a name="recurring-load-balancing"></a>定期的な負荷分散

既定では、VM 負荷分散は定期的な分散として構成されます。分散を行うために、クラスター内の各サーバーのメモリ負荷と CPU 使用率が 30 分ごとに評価されます。 手順は次のとおりです。

1. クラスター内のすべてのサーバーのメモリ負荷と CPU 使用率が評価されます。
2. しきい値を超えるすべてのサーバーおよびしきい値を下回るすべてのサーバーが識別されます。
3. メモリ負荷と CPU 使用率が最も高いサーバーが識別され、分散の優先順位が決まります。
4. VM は、しきい値を超えるサーバーから、最小しきい値を下回る別のサーバーにライブ移行されます (ダウンタイムは発生しません)。

:::image type="content" source="media/vm-load-balancing/periodic-balancing.png" alt-text="自動的に再調整されるライブ クラスターを示す画像" border="false"::: 

## <a name="configure-vm-load-balancing-using-windows-admin-center"></a>Windows Admin Center を使用して VM 負荷分散を構成する

VM 負荷分散を構成する最も簡単な方法は、Windows Admin Center を使用することです。 

:::image type="content" source="media/vm-load-balancing/vm-load-balancing.png" alt-text="Windows Admin Center を使用した VM 負荷分散の構成" lightbox="media/vm-load-balancing/vm-load-balancing.png":::

1. クラスターに接続し、 **[ツール] > [設定]** に移動します。

2. **[設定]** で **[Virtual machine load balancing]\(仮想マシンの負荷分散\)** を選択します。

3. **[Balance virtual machines]\(仮想マシンのバランス\)** で、 **[常に]** を選択してサーバーの参加時と 30 分ごとに負荷分散するか、 **[Server joins]\(サーバーの参加\)** を選択してサーバーの参加時のみ負荷分散するか、 **[Never]\(行わない\)** を選択して VM 負荷分散機能を無効にします。 既定の設定は **[常に]** です。

4. **[強度]** で、 **[低]** を選択してサーバーの負荷が 80% を超えたときに VM をライブ移行するか、 **[中]** を選択してサーバーの負荷が 70% を超えたときに移行するか、 **[高]** を選択してクラスター内のサーバーを平均化し、平均から 5% 上回ると移行するようにします。 既定の設定は **[低]** です。

## <a name="configure-vm-load-balancing-using-windows-powershell"></a>Windows PowerShell を使用して VM 負荷分散を構成する

クラスターの共通プロパティ `AutoBalancerMode` を使用して負荷分散を行うかどうかやいつ行われるかを構成できます。 クラスターの負荷を分散するタイミングを制御するために、PowerShell で次のように実行します (後にある表の値で置き換えます)。

```PowerShell
(Get-Cluster).AutoBalancerMode = <value>
```

|AutoBalancerMode |動作|
|-----------------|-----------|
| 0 | 無効 |
| 1 | サーバー参加時の負荷分散 |
| 2 (既定値) | サーバー参加時と 30 分ごとの負荷分散 |

クラスターの共通プロパティ `AutoBalancerLevel` を使用して、分散の強度を構成することもできます。 強度のしきい値を制御するために、PowerShell で次のように実行します (後にある表の値で置き換えます)。

```PowerShell
(Get-Cluster).AutoBalancerLevel = <value>
```

| AutoBalancerLevel | 強度 | 動作 |
|-------------------|----------------|----------|
| 1 (既定) | 低 | ホストの負荷が 80% を超えると移行 |
| 2 | Medium | ホストの負荷が 70% を超えると移行 |
| 3 | 高 | クラスター内のサーバーを平均化、ホストが平均を 5% 上回ると移行 |

`AutoBalancerLevel` および `AutoBalancerMode` のプロパティがどのように設定されているかを確認するには、PowerShell で次のように実行します。

```PowerShell
Get-Cluster | fl AutoBalancer*
```

## <a name="next-steps"></a>次のステップ

関連情報については、以下も参照してください。

- [VM の管理](vm.md)
- [PowerShell で VM を管理する](vm-powershell.md)
- [VM アフィニティ ルールの作成](vm-affinity.md)
