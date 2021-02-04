---
title: Azure Stack Hub で VM の既知の問題をトラブルシューティングする
description: Azure Stack Hub で仮想マシンの既知の問題をトラブルシューティングする方法について説明します。
author: mattbriggs
ms.topic: troubleshooting
ms.date: 2/1/2021
ms.author: mabrigg
ms.reviewer: kivenkat
ms.lastreviewed: 07/09/2020
ms.openlocfilehash: 45fcd2857a7da8820f1f588d864873e0e5f41467
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99246590"
---
# <a name="known-issues-vms-on-azure-stack-hub"></a>既知の問題:Azure Stack Hub 上の VM

この記事では、仮想マシン (VM) とスケール セットを操作するときの、Azure Stack Hub コンピューティング リソース プロバイダーのトラブルシューティングに関する既知の問題について説明します。

## <a name="portal-doesnt-show-correct-vm-name"></a>ポータルに正しい VM 名が表示されない
- **適用条件**  
    この問題はすべてのリリースに適用されます。  
- **原因**  
    概要ブレードで VM の詳細を表示すると、コンピューター名に (使用できません) と表示されます。 この表示は、特殊化ディスクまたはディスク スナップショットから作成された VM に対する仕様です。  
- **修復**  
    ポータルで、 **[設定]**  >  **[プロパティ]** を選択します。
- **個数**  
    共通  

## <a name="nvv4-vm-size-on-portal"></a>ポータル上の NVv4 VM のサイズ
- **適用条件**  
    この問題は Azure Stack Hub リリース 2002 以降に適用されます。  
- **原因**  
    VM の作成エクスペリエンスを実行すると、次の VM サイズが表示されます:NV4as_v4。 AMD Mi25 ベースの Azure Stack Hub GPU プレビューに必要なハードウェアを所有しているお客様は、VM のデプロイを成功させることができます。 他のすべてのお客様は、この VM サイズでの VM のデプロイに失敗します。  
- **修復**  
    [なし] :  
- **個数**  
    共通  

## <a name="vm-boot-diagnostics"></a>VM ブート診断
- **適用条件**  
    この問題は、サポートされているすべてのリリースに適用されます。  
- **原因**  
    新しい仮想マシン (VM) を作成するときに、次のエラーが表示されることがあります。仮想マシン 'vm-name' を起動できませんでした。 エラー:Failed to update serial output settings for VM 'vm-name'. (VM 'vm-name' のシリアル出力設定を更新できませんでした。) このエラーは、VM でブート診断を有効にしても、ブート診断ストレージ アカウントを削除した場合に発生します。  
- **修復**  
    以前使用したものと同じ名前のストレージ アカウントを再作成します。
- **個数**  
    共通  

## <a name="vm-diagnostics-storage-account-not-found"></a>VM 診断ストレージ アカウントが見つからない
- **適用条件**  
    この問題は、サポートされているすべてのリリースに適用されます。  
- **原因**  
    停止して割り当てを解除された仮想マシンを起動しようとすると、次のエラーが表示されることがあります:VM diagnostics Storage account 'diagnosticstorageaccount' not found. (VM 診断ストレージ アカウント 'diagnosticstorageaccount' が見つかりません。) Ensure storage account is not deleted. (ストレージ アカウントが削除されていないことを確実にしてください。) ブート診断が有効になっている VM を起動しようとしたときに、参照されるブート診断ストレージ アカウントが削除されていると、このエラーが発生します。  
- **修復**  
    以前使用したものと同じ名前のストレージ アカウントを再作成します。  
- **発生頻度** 普通  

## <a name="consumed-compute-quota"></a>コンピューティング クォータの消費
- **適用条件**  
    この問題は、サポートされているすべてのリリースに適用されます。  
- **原因**   
    新しい仮想マシンを作成すると、"This subscription is at capacity for Total Regional vCPUs on this location. This subscription is using all 50 Total Regional vCPUs available." (このサブスクリプションはこの場所のリージョンの vCPU の合計の容量に達しています。このサブスクリプションは、リージョンの vCPU の合計の使用可能な 50 個をすべて使用しています。)" などのエラーを受け取ることがあります。 これは、使用可能な合計コア数のクォータに達したことを示します。  
- **修復**  
    クォータを追加したアドオン プランをオペレーターに依頼してください。 現在のプランのクォータを変更しても機能せず、クォータの増加は反映されません。
- **個数**  
    珍しい  

## <a name="virtual-machine-scale-set"></a>仮想マシン スケール セット

-  **適用条件**  
    この問題は、サポートされているすべてのリリースに適用されます。  
- **原因**  
    4 ノードの Azure Stack Hub 環境で、修正プログラムや更新プログラムの適用中に作成が失敗します。 4 ノードの Azure Stack Hub 環境では、障害ドメインが 3 つの可用性セット内での VM の作成および仮想マシン スケール セット インスタンスの作成が、更新プロセス中に FabricVmPlacementErrorUnsupportedFaultDomainSize エラーで失敗します。  
- **修復**  
    障害ドメインが 2 つの可用性セットには 1 つの VM を正常に作成できます。 しかしながら、4 ノードの Azure Stack Hub のデプロイでは、更新プロセス中にスケール セット インスタンスを作成することはまだできません。  
- **個数**  
    珍しい  

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub VM の機能](azure-stack-vm-considerations.md)を確認する。
