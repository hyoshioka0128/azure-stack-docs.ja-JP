---
title: Azure Stack Hub でのスケール ユニットのノード操作
description: 電源オン、電源オフ、無効化、再開を含むスケール ユニットのノード アクションと、Azure Stack Hub 統合システムでノードの状態を表示する方法について説明します。
author: PatAltimore
ms.topic: how-to
ms.date: 1/19/2021
ms.author: patricka
ms.reviewer: thoroet
ms.lastreviewed: 1/19/2021
ms.openlocfilehash: 5bececf48222c9dc7401df7cb84d83f375c93d75
ms.sourcegitcommit: 01abc9d81ced31bd727626195148b4e00cc2d62e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/20/2021
ms.locfileid: "98584294"
---
# <a name="scale-unit-node-actions-in-azure-stack-hub"></a>Azure Stack Hub でのスケール ユニットのノード操作

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

   - 個々のノードの一覧。
   - 動作状態 (以下の一覧を参照)。
   - 電源の状態 (実行中または停止)。
   - サーバー モデル。
   - ベースボード管理コントローラー (BMC) の IP アドレス。
   - コアの合計数。
   - メモリの総量。
   
    ノード操作によって、予期されるアラートが管理者ポータルで発生することもあります。 

![スケール ユニットの状態](media/azure-stack-node-actions/multinodeactions.png)

### <a name="node-operational-states"></a>ノードの動作状態

| Status | 説明 |
|----------------------|-------------------------------------------------------------------|
| 実行中 | ノードは、アクティブにスケール ユニットに参加しています。 |
| 停止済み | ノードは利用不可です。 |
| 追加中 | ノードは、アクティブにスケール ユニットに追加されています。 |
| 修復中 | ノードは現在、アクティブに修復されています。 |
| メンテナンス | ノードは一時停止され、アクティブなユーザー ワークロードは実行されていません。 |
| 修復が必要 | ノードの修復を必要とするエラーが検出されました。 |

### <a name="azure-stack-hub-shows-adding-status-after-an-operation"></a>Azure Stack Hub に [追加中] という状態が表示される

Azure Stack Hub では、ドレイン、再開、修復、シャットダウン、開始などの操作が実行された後に、操作ノードの状態が **[追加中]** と表示されることがあります。
これは、操作後に、ファブリック リソース プロバイダー ロールのキャッシュが更新されなかった場合に発生する可能性があります。 

次の手順を適用する前に、現在進行中の操作がないことを確認してください。 お使いの環境に合わせてエンドポイントを更新します。

### <a name="az-modules"></a>[Az モジュール](#tab/az1)

1. PowerShell を開き、Azure Stack Hub 環境を追加します。 これを行うには、お使いのコンピューターに [Azure Stack Hub PowerShell がインストールされている](./powershell-install-az-module.md)必要があります。

    ```powershell
    Add-AzEnvironment -Name AzureStack -ARMEndpoint https://adminmanagement.local.azurestack.external
    Add-AzAccount -Environment AzureStack
    ```

2. 次のコマンドを実行して、ファブリック リソース プロバイダー ロールを再起動します。

   ```powershell
   Restart-AzsInfrastructureRole -Name FabricResourceProvider
   ```

3. 影響を受けているスケール ユニット ノードの動作状態が、 **[実行中]** に変更されたことを確認します。 管理者ポータルまたは次の PowerShell コマンドを使用できます。

   ```powershell
   Get-AzsScaleUnitNode |ft name,scaleunitnodestatus,powerstate
   ```

4. ノードの動作状態がまだ **[追加中]** と表示される場合は、続いてサポート インシデントを開いてください。

### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm1)

1. PowerShell を開き、Azure Stack Hub 環境を追加します。 これを行うには、お使いのコンピューターに [Azure Stack Hub PowerShell がインストールされている](./powershell-install-az-module.md)必要があります。

    ```powershell
    Add-AzureRMEnvironment -Name AzureStack -ARMEndpoint https://adminmanagement.local.azurestack.external
    Add-AzureRMAccount -Environment AzureStack
    ```

2. 次のコマンドを実行して、ファブリック リソース プロバイダー ロールを再起動します。

   ```powershell
   Restart-AzsInfrastructureRole -Name FabricResourceProvider
   ```

3. 影響を受けているスケール ユニット ノードの動作状態が、 **[実行中]** に変更されたことを確認します。 管理者ポータルまたは次の PowerShell コマンドを使用できます。

   ```powershell
   Get-AzsScaleUnitNode |ft name,scaleunitnodestatus,powerstate
   ```

4. ノードの動作状態がまだ **[追加中]** と表示される場合は、続いてサポート インシデントを開いてください。

---



## <a name="scale-unit-node-actions"></a>スケール ユニットのノード操作

スケール ユニットのノードに関する情報を表示しているときに、次のようなノード操作を行うこともできます。

 - 開始と停止 (現在の電源状態による)。
 - 無効化と再開 (動作状態による)。
 - 修復。
 - シャットダウン。

ノードの操作状態によって、使用可能なオプションが決まります。

Azure Stack Hub PowerShell モジュールをインストールする必要があります。 これらのコマンドレットは **Azs.Fabric.Admin** モジュールに存在します。 PowerShell for Azure Stack Hub のインストールまたはインストールの確認については、「[PowerShell for Azure Stack Hub をインストールする](powershell-install-az-module.md)」を参照してください。

## <a name="stop"></a>Stop

**停止** アクションは、ノードをオフにします。 これは、電源ボタンを押した場合と同じです。 オペレーティング システムにシャットダウン信号は送られません。 計画されている停止操作の場合は、最初に必ずシャットダウン操作を行ってください。

この操作は通常、ノードが要求に応答しなくなったときに使用されます。

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

> [!CAUTION]  
> この記事で説明している操作を成功させるには、ファームウェアの平準化が重要です。 この手順を行わないと、システムの不安定化、パフォーマンスの低下、セキュリティ上の脅威、または Azure Stack Hub オートメーションによるオペレーティング システムのデプロイ時に失敗を引き起こす可能性があります。 ハードウェアを交換する場合は、ハードウェア パートナーのドキュメントを必ず参照して、適用されるファームウェアが、[Azure Stack Hub 管理者ポータル](azure-stack-updates.md)に表示されている OEM バージョンと一致していることを確認してください。<br><br>
詳細およびパートナー ドキュメントへのリンクについては、[ハードウェア コンポーネントの交換](azure-stack-replace-component.md)に関する記事を参照してください。

| ハードウェア パートナー | リージョン | URL |
|------------------|--------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Cisco | All | [Cisco Integrated System for Microsoft Azure Stack Hub Operations Guide (Cisco Integrated System for Microsoft Azure Stack Hub 運用ガイド)](https://www.cisco.com/c/en/us/td/docs/unified_computing/ucs/azure-stack/b_Azure_Stack_Operations_Guide_4-0/b_Azure_Stack_Operations_Guide_4-0_chapter_00.html#concept_wks_t1q_wbb)<br><br>[Release Notes for Cisco Integrated System for Microsoft Azure Stack Hub (Cisco Integrated System for Microsoft Azure Stack Hub のリリース ノート)](https://www.cisco.com/c/en/us/support/servers-unified-computing/ucs-c-series-rack-mount-ucs-managed-server-software/products-release-notes-list.html) |
| Dell EMC | All | [Cloud for Microsoft Azure Stack Hub 14G (アカウントとログインが必要)](https://support.emc.com/downloads/44615_Cloud-for-Microsoft-Azure-Stack-14G)<br><br>[Cloud for Microsoft Azure Stack Hub 13G (アカウントとログインが必要)](https://support.emc.com/downloads/42238_Cloud-for-Microsoft-Azure-Stack-13G) |
| Fujitsu | 日本 | [富士通マネージド サービス サポート デスク (アカウントとログインが必要)](https://eservice.fujitsu.com/supportdesk-web/) |
|  | ヨーロッパ、中東およびアフリカ | [Fujitsu サポート: IT 製品およびシステム](https://support.ts.fujitsu.com/IndexContact.asp?lng=COM&ln=no&LC=del) (英語) |
|  |  | [Fujitsu MySupport (アカウントとログインが必要)](https://support.ts.fujitsu.com/IndexMySupport.asp) (英語) |
| HPE | All | [HPE ProLiant for Microsoft Azure Stack Hub](http://www.hpe.com/info/MASupdates) |
| Lenovo | All | [ThinkAgile SXM Best Recipes](https://datacentersupport.lenovo.com/us/en/solutions/ht505122) |

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

- [Azure Stack PowerShell をインストールする](./powershell-install-az-module.md)
- [Azure Stack Hub Fabric オペレーター モジュールについて確認する](/powershell/module/azs.fabric.admin/?view=azurestackps-1.6.0&preserve-view=true)
- [ノードの追加操作を監視する](./azure-stack-add-scale-node.md#monitor-add-node-operations)
