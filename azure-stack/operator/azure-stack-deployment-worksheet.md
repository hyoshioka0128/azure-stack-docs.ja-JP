---
title: Azure Stack Hub のデプロイ ワークシート
description: Azure Stack Hub をデプロイするデプロイ ワークシート ツールのインストール方法と使用方法について説明します。
author: PatAltimore
ms.topic: article
ms.date: 04/19/2019
ms.author: patricka
ms.reviewer: wamota
ms.lastreviewed: 04/19/2019
ms.openlocfilehash: afdad104cad67e928e1d6d988138ab1c8555631c
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97870887"
---
# <a name="deployment-worksheet-for-azure-stack-hub-integrated-systems"></a>Azure Stack Hub 統合システムのデプロイ ワークシート

Azure Stack Hub デプロイ ワークシートは、必要なすべてのデプロイ情報と決定事項を 1 か所にまとめた Windows フォーム アプリです。 計画プロセス中にデプロイ ワークシートを完成し、デプロイの開始前にそれを確認することができます。

ワークシートに必要な情報は、ネットワーク、セキュリティ、および ID 情報を対象にしています。 この情報には特定の領域に関する具体的な知識が必要な場合があるため、ワークシートを完成させるには専門家に相談することをお勧めします。

ワークシートに記入しているときに、ネットワーク環境へのデプロイ前の構成変更がいくつか必要になることがあります。 これらの変更には、Azure Stack Hub ソリューションのための IP アドレス空間の予約や、新しい Azure Stack Hub ソリューションへの接続を準備するためのルーター、スイッチ、およびファイアウォールの構成が含まれる可能性があります。

> [!NOTE]
> デプロイ ワークシート ツールを完了する方法の詳細については、「[Azure Stack Hub 統合システムのデータセンター統合計画に関する考慮事項](azure-stack-datacenter-integration.md)」を参照してください。

[![Azure Stack Hub のデプロイのデプロイ ワークシート](media/azure-stack-deployment-worksheet/depworksheet.png "デプロイ ワークシート")](media/azure-stack-deployment-worksheet/depworksheet.png)

## <a name="installing-the-windows-powershell-module"></a>Windows PowerShell モジュールのインストール

デプロイ ワークシートのリリースごとに、デプロイ ワークシートを使用する各マシンに PowerShell モジュールの 1 回限りのインストールを実行する必要があります。

> [!NOTE]  
> この方法を実行するには、コンピューターがインターネットに接続されている必要があります。

1. 管理者特権の PowerShell プロンプトを開きます。

2. PowerShell ウィンドウで、[PowerShell ギャラリー](https://www.powershellgallery.com/packages/Azs.Deployment.Worksheet/)からモジュールをインストールします。

   ```PowerShell
   Install-Module -Name Azs.Deployment.Worksheet -Repository PSGallery
   ```

信頼されていないリポジトリからのインストールに関するメッセージを受け取る場合は、**Y** キーを押してインストールを続行します。

## <a name="use-the-deployment-worksheet-tool"></a>デプロイ ワークシート ツールの使用

デプロイ ワークシート PowerShell モジュールをインストールしたコンピューター上でデプロイ ワークシートを起動して使用するには、次の手順を実行します。

1. Windows PowerShell を起動します (予期しない結果が発生する可能性があるため、PowerShell ISE は使用しないでください)。 管理者として PowerShell を実行する必要はありません。

2. **AzS.Deployment.Worksheet** PowerShell モジュールをインポートします。

   ```PowerShell
   Import-Module AzS.Deployment.Worksheet
   ```

3. モジュールがインポートされたら、デプロイ ワークシートを起動します。

   ```PowerShell
   Start-DeploymentWorksheet
   ```

デプロイ ワークシートは、 **[Customer Settings]\(顧客設定\)** 、 **[ネットワーク設定]** 、 **[Scale Unit #]\(スケール ユニット番号\)** など、環境設定を収集するための個別のタブで構成されています。 構成データ ファイルを生成する前に、すべてのタブで (**省略可能** とマークされたものを除いて) すべての値を指定する必要があります。 必要な値をすべてツールに入力したら、 **[アクション]** メニューを使用して **[インポート]** 、 **[エクスポート]** 、および **[生成]** を実行できます。 デプロイに必要な JSON ファイルは次のとおりです。

**Import**:このツールによって生成された Azure Stack Hub 構成データ ファイル (ConfigurationData.json)、または以前のリリースのデプロイ ワークシートによって作成されたファイルをインポートできます。 インポートを実行するとフォームがリセットされ、以前に入力した設定や生成されたデータが削除されます。

**Export**:現在フォームに入力されているデータが検証され、IP サブネットと割り当てが生成され、コンテンツが JSON 形式の構成ファイルとして保存されます。 これらのファイルを使用すると、ネットワーク構成を生成し、Azure Stack Hub をインストールできます。

**生成**:現在入力されているデータが検証され、デプロイ JSON ファイルをエクスポートせずにネットワーク マップが生成されます。 **[生成]** が成功すると、2 つの新しいタブが作成されます。 **[Subnet Summary]\(サブネットの概要\)** と **[IP Assignments]\(IP 割り当て\)** です。 これらのタブでデータを分析して、ネットワーク割り当てが想定どおりであることを確認できます。

**すべてクリア**:現在フォームに入力されているすべてのデータがクリアされ、既定値に戻されます。

**Save or Open your work in-progress (進行中の作業を保存する、または開く)** :作業中に部分的に入力したデータを保存し、開く場合は、 **[ファイル] -> [保存]** および **[ファイル] -> [開く]** メニューを使用します。 この機能は、すべてのデータを入力して検証する必要がある **[インポート]** および **[エクスポート]** 機能とは異なります。 [開く] や [保存] では検証されないため、進行中の作業を保存するためにすべてのフィールドを入力する必要はありません。

**Logging and Warning messages (ログおよび警告メッセージ)** :フォームの使用中は、重大ではない警告メッセージが PowerShell ウィンドウに表示されることがあります。 重大なエラーはポップアップ メッセージとして表示されます。 ディスクに書き込まれたログを含め、省略可能な詳細ログを有効にして、問題のトラブルシューティングに役立てることができます。

詳細ログを使用してツールを起動するには:

   ```PowerShell
   Start-DeploymentWorksheet -EnableLogging
   ```

保存されたログは、現在のユーザーの **Temp** ディレクトリで確認できます。次に例を示します。**C:\Users\me\AppData\Local\Temp\Microsoft_AzureStack\DeploymentWorksheet_Log.txt**.

## <a name="next-steps"></a>次のステップ

* [Azure Stack Hub デプロイの接続モデル](azure-stack-connection-models.md)
