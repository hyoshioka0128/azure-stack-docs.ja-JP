---
title: ASDK のトラブルシューティング
description: Azure Stack Development Kit (ASDK) のトラブルシューティング方法について説明します。
author: PatAltimore
ms.topic: article
ms.date: 01/06/2021
ms.author: patricka
ms.reviewer: misainat
ms.lastreviewed: 01/06/2021
ms.openlocfilehash: ab4a7166f0a6ae43259f3f1f919c3b24af017c1e
ms.sourcegitcommit: 9b0e1264ef006d2009bb549f21010c672c49b9de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98255216"
---
# <a name="troubleshoot-the-asdk"></a>ASDK のトラブルシューティング
この記事では、Azure Stack Development Kit (ASDK) の一般的なトラブルシューティング情報を提供します。 Azure Stack 統合システムに関するヘルプについては、「[Microsoft Azure Stack のトラブルシューティング](../operator/azure-stack-troubleshooting.md)」を参照してください。 

ASDK は評価環境であるため、Microsoft サポートによるサポートは提供されません。 記載されていない問題が発生している場合は、[Azure Stack 用の Microsoft Q&A](/answers/questions/topics/azure-stack-hub.html) で専門家からのヘルプを得られます。

## <a name="deployment"></a>デプロイ
### <a name="deployment-failure"></a>デプロイの失敗
インストール時に障害が発生した場合、デプロイ スクリプトの -rerun オプションを使用して、失敗した手順からデプロイを再開できます。 次に例を示します。

  ```powershell
  cd C:\CloudDeployment\Setup
  .\InstallAzureStackPOC.ps1 -Rerun
  ```

### <a name="at-the-end-of-the-deployment-the-powershell-session-is-still-open-and-doesnt-show-any-output"></a>デプロイの最後に、PowerShell セッションがまだ開いており、出力が表示されません
この動作は、PowerShell コマンド ウィンドウが選択されている場合の既定の動作の結果に過ぎない可能性があります。 ASDK のデプロイは成功しましたが、ウィンドウを選択したときに、スクリプトが一時停止しました。 設定が完了したことを確認するには、コマンド ウィンドウのタイトル バーで、"select" という単語を探します。 ESC キーを押してその選択を解除すると、その後に完了メッセージが表示されるはずです。

### <a name="template-validation-error-parameter-osprofile-is-not-allowed"></a>パラメーター osProfile が許可されないというテンプレート検証エラー

テンプレートの検証時に、パラメーター "osProfile" が許可されないというエラー メッセージが表示される場合は、以下のコンポーネントの正しいバージョンが使用されていることを確認してください。

- [Compute](../user/azure-stack-profiles-azure-resource-manager-versions.md#microsoftcompute)
- [Network](../user/azure-stack-profiles-azure-resource-manager-versions.md#microsoftnetwork)

Azure から Azure Stack に VHD をコピーするには、[AzCopy 7.3.0](../user/azure-stack-storage-transfer.md#download-and-install-azcopy)を使用します。 イメージ自体の問題については、ベンダーと協力して解決してください。 Azure Stack の WALinuxAgent の要件の詳細については、「[Azure LinuX エージェント](../operator/azure-stack-linux.md#azure-linux-agent)」を参照してください。

### <a name="deployment-fails-due-to-lack-of-external-access"></a>外部アクセスがないことが原因でデプロイが失敗する
外部アクセスが必要な段階でデプロイが失敗する場合は、次の例のような例外が返されます。

```
An error occurred while trying to test identity provider endpoints: System.Net.WebException: The operation has timed out.
   at Microsoft.PowerShell.Commands.WebRequestPSCmdlet.GetResponse(WebRequest request)
   at Microsoft.PowerShell.Commands.WebRequestPSCmdlet.ProcessRecord()at, <No file>: line 48 - 8/12/2018 2:40:08 AM
```
このエラーが発生する場合は、[デプロイ ネットワークのトラフィックに関するドキュメント](../operator/deployment-networking.md)を見直して、最小限のネットワーク要件がすべて満たされていることを確認してください。 パートナー ツールキットの一部として、ネットワーク チェッカー ツールも提供されています。

その他のデプロイ エラーは、一般に、インターネット上でのリソースへの接続の問題が原因です。

インターネット上でのリソースへの接続を確認するには、次の手順を実行します。

1. PowerShell を開きます。
2. WAS01 またはいずれかの ERCS VM に対して Enter-PSSession を実行します。
3. 次のコマンドレットを実行します。 
   ```powershell
   Test-NetConnection login.windows.net -port 443
   ```

このコマンドが失敗する場合は、TOR スイッチおよびその他のネットワーク デバイスがすべて、[ネットワーク トラフィックを許可する](../operator/azure-stack-network.md)ように構成されていることを確認します。


## <a name="virtual-machines"></a>仮想マシン
### <a name="default-image-and-gallery-item"></a>既定のイメージとギャラリー アイテム
Azure Stack に VM をデプロイする前に、まず Windows Server イメージとギャラリー アイテムを追加する必要があります。

### <a name="after-restarting-my-azure-stack-host-some-vms-dont-automatically-start"></a>Azure Stack ホストの再起動後、一部の VM が自動的に起動しない
ホストの再起動後、Azure Stack サービスがすぐに使用できないことに気付く場合があります。 これは、Azure Stack [インフラストラクチャ VM](asdk-architecture.md#virtual-machine-roles) および RP が一貫性をチェックするために少し時間がかかるためですが、最終的に自動的に起動します。

また、ASDK ホストの再起動後に、そのテナント VM が自動的に起動しないことに気付く場合もあります。 手動でいくつかの手順を実行して、オンラインにすることができます。

1.  ASDK ホストで、[スタート] メニューから **フェールオーバー クラスター マネージャー** を起動します。
2.  クラスター **S-Cluster.azurestack.local** を選択します。
3.  **[役割]** を選びます。
4.  テナント VM が *保存済み* 状態で表示されます。 すべてのインフラストラクチャ VM が実行されたら、テナント VM を右クリックし、 **[開始]** を選択して、VM を再開します。

### <a name="ive-deleted-some-vms-but-still-see-the-vhd-files-on-disk"></a>一部の VM の削除後に、まだディスクに VHD ファイルが表示される 
この動作は仕様によるものです。

* VM を削除しても VHD は削除されません。 ディスクはリソース グループ内の個別のリソースです。
* ストレージ アカウントが削除されると、削除は Azure Resource Manager にただちに表示されますが、それを格納している可能性のあるディスクは、ガベージ コレクションが実行されるまで、記憶域に維持されます。

"孤立" VHD が表示された場合、それが削除されたストレージ アカウントのフォルダーの一部であるかどうかを知ることが重要です。 ストレージ アカウントが削除されていない場合は、VHD が残っていても正常です。

リテンション期間しきい値と、オンデマンドの再利用の設定について詳しくは、[ストレージ アカウントの管理](../operator/azure-stack-manage-storage-accounts.md)に関するページを参照してください。

## <a name="storage"></a>ストレージ
### <a name="storage-reclamation"></a>記憶域の再利用
ポータルに再利用された容量が表示されるまで、最大で 14 時間かかる場合があります。 領域の再利用は、ブロック BLOB ストア内の内部コンテナー ファイルの使用率をなど、さまざまな要因に依存します。 そのため、削除されるデータの量によって、ガベージ コレクターの実行時に再利用可能になる領域の量に対する保証はありません。

## <a name="next-steps"></a>次のステップ
[Azure Stack サポート フォーラムを参照する](https://social.msdn.microsoft.com/Forums/azure/home?forum=azurestack)