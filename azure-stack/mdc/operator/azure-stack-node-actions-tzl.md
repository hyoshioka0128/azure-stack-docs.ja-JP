---
title: MDC の Azure Stack Hub でのスケール ユニットのノード操作
description: 電源オン、電源オフ、無効化、再開を含むスケール ユニットのノード アクションを実行する方法と、Azure Stack Hub 統合システムでノードの状態を表示する方法について説明します。
author: PatAltimore
ms.topic: article
ms.date: 10/26/2020
ms.author: patricka
ms.reviewer: thoroet
ms.lastreviewed: 10/26/2020
ms.openlocfilehash: e07e06d2dbe90c846fc8ea96f18e74b0be3caf49
ms.sourcegitcommit: 9b0e1264ef006d2009bb549f21010c672c49b9de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98255352"
---
# <a name="scale-unit-node-actions-in-azure-stack-hub---modular-data-center-mdc"></a>Azure Stack Hub のスケール ユニットのノード操作 - Modular Data Center (MDC)

この記事では、スケール ユニットの状態を確認する方法について説明します。 ユニットのノードは、表示することができます。 電源オン、電源オフ、シャットダウン、ドレイン、再開、修復などのノード アクションを実行できます。 通常、これらのノード アクションは、パーツのフィールド交換時に使用されるほか、ノードの復旧の手段として使用されます。

> [!Important]  
> この記事で説明されているすべてのノード アクションは、一度に 1 つのノードを対象にするようにしてください。

## <a name="view-the-node-status"></a>ノードの状態を表示する

管理者ポータルで、スケール ユニットとその関連するノードの状態を表示できます。

スケール ユニットの状態を表示するには、以下のようにします。

1. **[Region management]\(リージョンの管理\)** タイルで、リージョン名をクリックします。
2. 左側の **[インフラストラクチャ リソース]** で、 **[スケール ユニット]** を選択します。
3. 結果画面で、スケール ユニットを選択します。
4. 左側の **[全般]** で、 **[ノード]** を選択します。

   次の情報を確認します。

   - 個々のノードの一覧
   - 動作状態 (以下の一覧を参照)
   - 電源の状態 (実行中または停止)
   - サーバー モデル
   - ベースボード管理コントローラー (BMC) の IP アドレス
   - コアの合計数
   - メモリの総量

     ![スケール ユニットの状態](media/azure-stack-node-actions/multinode-actions.png)

### <a name="node-operational-states"></a>ノードの動作状態

| Status | 説明 |
|----------------------|-------------------------------------------------------------------|
| 実行中 | ノードは、アクティブにスケール ユニットに参加しています。 |
| 停止済み | ノードは利用不可です。 |
| 追加中 | ノードは、アクティブにスケール ユニットに追加されています。 |
| 修復中 | ノードは現在、アクティブに修復されています。 |
| メンテナンス | ノードは一時停止され、アクティブなユーザー ワークロードは実行されていません。 |
| 修復が必要 | ノードの修復を必要とするエラーが検出されました。 |

## <a name="scale-unit-node-actions"></a>スケール ユニットのノード操作

スケール ユニットのノードに関する情報を表示しているときに、次のようなノード操作を行うこともできます。

 - 開始と停止 (現在の電源状態による)。
 - 無効化と再開 (動作状態による)。
 - 修復。
 - シャットダウン。

ノードの操作状態によって、使用可能なオプションが決まります。

Azure Stack Hub PowerShell モジュールをインストールする必要があります。 これらのコマンドレットは **Azs.Fabric.Admin** モジュールに存在します。 PowerShell for Azure Stack Hub のインストールまたはインストールの確認については、「[PowerShell for Azure Stack Hub をインストールする](../../operator/powershell-install-az-module.md)」を参照してください。

## <a name="stop"></a>Stop

**停止** アクションは、ノードをオフにします。 これは、電源ボタンを押した場合と同じです。 オペレーティング システムにシャットダウン信号は送られません。 計画されている停止操作の場合は、最初に必ずシャットダウン操作を行ってください。

この操作は通常、ノードが無応答状態になったときに使用されます。

停止アクションを実行するには、管理者特権の PowerShell プロンプトを開き、次のコマンドレットを実行します。

```powershell  
  Stop-AzsScaleUnitNode -Location <RegionName> -Name <NodeName>
```

まれなケースで、停止アクションが機能しない場合には、操作を再試行し、2 度目も失敗するようであれば、代わりに BMC Web インターフェイスを使用してください。

詳細については、「[Stop-AzsScaleUnitNode](/powershell/module/azs.fabric.admin/stop-azsscaleunitnode)」を参照してください。

## <a name="start"></a>[開始]

**開始** 操作は、ノードをオンにします。 これは、電源ボタンを押した場合と同じです。

開始アクションを実行するには、管理者特権の PowerShell プロンプトを開き、次のコマンドレットを実行します。

```powershell  
  Start-AzsScaleUnitNode -Location <RegionName> -Name <NodeName>
```

めったにありませんが、開始操作が機能しない場合は、操作を再試行します。 2 回目も失敗した場合は、代わりに BMC Web インターフェイスを使用します。

詳細については、「[Start-AzsScaleUnitNode](/powershell/module/azs.fabric.admin/start-azsscaleunitnode)」を参照してください。

## <a name="drain"></a>ドレイン

**ドレイン** 操作は、すべてのアクティブなワークロードを、その特定のスケール ユニット内の残りのノードに移動します。

この操作は通常、ノード全体の交換などの、パーツの現場交換中に使用されます。

> [!Important]
> ノードのドレイン操作は、必ず、ユーザーに通知済みの計画されたメンテナンス期間中に行うようにしてください。 状況によっては、アクティブなワークロードが中断されることがあります。

ドレイン アクションを実行するには、管理者特権の PowerShell プロンプトを開き、次のコマンドレットを実行します。

```powershell  
  Disable-AzsScaleUnitNode -Location <RegionName> -Name <NodeName>
```

詳細については、「[Disable-AzsScaleUnitNode](/powershell/module/azs.fabric.admin/disable-azsscaleunitnode)」を参照してください。

## <a name="resume"></a>Resume

**再開** アクションは、無効化されたノードを再開し、ワークロードのアクティブな配置対象としてマークします。 ノードで実行されていた以前のワークロードはフェールバックされません。 (ノードのドレイン操作を使用する場合は、必ず電源をオフにしてください。 ノードは、電源をオンに戻しても、ワークロードのアクティブな配置対象としてマークされることはありません。 準備ができたら、再開操作を使用しノードをアクティブとしてマークする必要があります。)

再開アクションを実行するには、管理者特権の PowerShell プロンプトを開き、次のコマンドレットを実行します。

```powershell  
  Enable-AzsScaleUnitNode -Location <RegionName> -Name <NodeName>
```

詳細については、「[Enable-AzsScaleUnitNode](/powershell/module/azs.fabric.admin/enable-azsscaleunitnode)」を参照してください。

## <a name="repair"></a>修復

**修復** アクションは、ノードを修復します。 次のシナリオのいずれかに対してのみ使用します。

- ノードの完全交換 (新しいデータ ディスクあり、またはなし)。
- ハードウェア コンポーネントの障害と交換の後 (フィールド交換可能装置 (FRU) ドキュメントで推奨されている場合)。

> [!Important]  
> ノードまたは個々のハードウェア コンポーネントを置き換える必要がある場合の正確な手順については、OEM ハードウェア ベンダーの FRU ドキュメントを参照してください。 FRU ドキュメントでは、ハードウェア コンポーネントを交換した後、修復操作を実行する必要があるかどうかを指定します。

修復操作を実行する場合、BMC の IP アドレスを指定する必要があります。

修復アクションを実行するには、管理者特権の PowerShell プロンプトを開き、次のコマンドレットを実行します。

  ```powershell
  Repair-AzsScaleUnitNode -Location <RegionName> -Name <NodeName> -BMCIPv4Address <BMCIPv4Address>
  ```

## <a name="shutdown"></a>Shutdown

**シャットダウン** 操作では最初に、すべてのアクティブなワークロードを、同じスケール ユニット内の残りのノードに移動します。 操作では次に、スケール ユニット ノードを適切にシャットダウンします。

シャットダウンされたノードの開始後は、[再開](#resume)操作を実行する必要があります。 ノードで実行されていた以前のワークロードはフェールバックされません。

シャットダウン操作に失敗する場合は、シャットダウン操作の前に[ドレイン](#drain)操作を試行します。

シャットダウン操作を実行するには、管理者特権の PowerShell プロンプトを開き、次のコマンドレットを実行します。

  ```powershell
  Stop-AzsScaleUnitNode -Location <RegionName> -Name <NodeName> -Shutdown
  ```

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub Fabric オペレーター モジュールについて](/powershell/module/azs.fabric.admin/?view=azurestackps-1.6.0)。