---
title: Azure Stack - Modular Data Center の BLOB ストレージ用の拡張ストレージでの正常性チェック
description: この記事では、Modular Data Center BLOB ストレージ用の拡張ストレージで正常性チェックを実行する方法について説明します。
services: azure-stack
documentationcenter: ''
author: PatAltimore
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: app-service
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/01/2020
ms.author: patricka
ms.reviewer: karlt
ms.lastreviewed: 10/01/2020
ms.openlocfilehash: 8dea13880e2703290e1528fca47b777f98fca7b2
ms.sourcegitcommit: 9b0e1264ef006d2009bb549f21010c672c49b9de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98255522"
---
# <a name="extended-storage-health-checks"></a>拡張ストレージの正常性チェック

この記事では、データセンターの拡張ストレージ ハードウェアと Azure Stack Hub のデプロイの正常性を確認するためのガイダンスを提供します。

始める前に、[拡張ストレージのファームウェアの更新]()に関するページの手順を確認して従ってください。

システムを出荷する前の最後のステップは、システムをシャットダウンすることです。 クイック スタート ガイドの手順に従ってください。

## <a name="extended-storage-health"></a>拡張ストレージの正常性

このセクションでは、拡張ストレージ ハードウェアの正常性を確認する方法について説明します。
イベントを確認し、解決すべき問題がある場合は管理者に通知します。 


クラスター記憶域プールの正常性を確認するには、次のコマンドを実行します。
```console
isi storagepool health
```

たとえば、正常な場合、コマンドの結果は次のようになります。
```console
All pools are healthy.
Unprovisioned drives: none
```

異常な場合、またはドライブがない場合、コマンドの結果は次のようになります。

```console
SmartPools Health
Name                  Health  Type Prot   Members          Down          Smartfailed
--------------------- ------- ---- ------ ---------------- ------------- -------------
a2000_200tb_800gb-    -M---
ssd-sed_16gb
 a2000_200tb_800gb-   -M---   HDD  3x     3,11,14,19,23,30 Nodes:        Nodes:
ssd-sed_16gb:24                           ,38,41,46:bay6,1 Drives:       Drives:
                                          0-11,16,19,
                                          25:bay6,10-11,16

OK = Ok, U = Too few nodes, M = Missing drives,
D = Some nodes or drives are down, S = Some nodes or drives are smartfailed,
R = Some nodes or drives need repair
Unprovisioned drives: none
```

問題については、Microsoft サポートにお問い合わせください。

次のコマンドを実行すると、簡易ビューで、クラスター全体の正常性を確認できます。
```console
isi status
```

すべてが正常な場合は、緑色の **[OK]** が表示されます。それ以外の場合は、 **[Cluster Health]\(クラスターの正常性\)** の行、または後の表の 1 つまたは複数のクラスター ノード ID の行に、黄色の警告または赤いエラーが表示されます。

クラスターの正常性に関する詳細情報が必要な場合は、展開ビューでより詳細なコマンドを実行できます。
```console
isi status -a
```

問題については、Microsoft サポートにお問い合わせください。

## <a name="azure-stack-hub-health"></a>Azure Stack Hub の正常性

このセクションでは、Azure Stack Hub のデプロイの正常性を確認する方法について説明します。

拡張ストレージ システムと統合された Azure Stack のデプロイの全体的なビューを取得するには、後で示すスクリプトを実行します。これは、以下に対するラッパーです。
- 提供された資格情報を使用した ERCS VM への接続
- Test-AzureStack -Debug コマンドの実行 (画面への出力として、詳細な正常性情報が直接出力されます)

このスクリプトの前提条件は以下のとおりです。
- ハードウェア ライフサイクル ホスト (HLH) の C:\OEMSoftware\ExtendedStorage\ フォルダーにある .\Invoke-ExtendedStorageConfiguration.ps1 スクリプト ファイル
- $AzScred 資格情報変数。この変数には、*DOMAIN*\cloudadmin の資格情報が設定されている必要があります。 *DOMAIN* は、実際のドメイン名 (CONTOSO など) に置き換えます。


```powershell
$AzScred = Get-Credential -Credential 'DOMAIN\cloudadmin'
.\Invoke-ExtendedStorageConfiguration.ps1 -TestAzureStack -AzureStackCred $AzScred
```

出力で、Azure Stack のデプロイの全体的な正常性状態を確認します。また、拡張ストレージと Azure Stack の統合の特定の正常性状態について、**NAS** の特定のセクションも確認します。

Azure Stack の診断の詳細については、「[Azure Stack Hub システムの状態を検証する](../operator/azure-stack-diagnostic-test.md)」を参照してください。

## <a name="technical-support"></a>テクニカル サポート

問題については、Microsoft サポートにお問い合わせください。

## <a name="next-steps"></a>次のステップ

- [ファームウェアを更新する]()