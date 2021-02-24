---
title: クラウド ID を検索する
description: Azure Stack Hub の [ヘルプとサポート] でクラウド ID を検索する方法について説明します。
author: PatAltimore
ms.topic: article
ms.date: 10/08/2019
ms.author: patricka
ms.reviewer: shisab
ms.lastreviewed: 10/08/2019
ms.openlocfilehash: 6da70761e8a37967d3debdff28b67447596c632a
ms.sourcegitcommit: 9b0e1264ef006d2009bb549f21010c672c49b9de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98255964"
---
# <a name="find-your-cloud-id"></a>クラウド ID を検索する

このトピックでは、管理者ポータルまたは特権エンドポイント (PEP) を使用して、クラウド ID を取得する方法について説明します。 クラウド ID は、特定のスケール ユニットからアップロードされたサポート データを追跡するための一意の ID です。 サポートでの分析のために診断ログがアップロードされた場合に、クラウド ID は、ログがそのスケール ユニットにどのように関連付けられているかを示すものになります。

## <a name="use-the-administrator-portal"></a>管理者ポータルを使用する

1. 管理者ポータルを開きます。 
1. **[リージョンの管理]** を選択します。

   ![ダッシュボードのスクリーンショット](./media/azure-stack-automatic-log-collection/dashboard.png)

1. **[プロパティ]** を選択し、**スタンプ クラウド ID** をコピーします。

   ![スタンプ クラウド ID を示すリージョン プロパティのスクリーンショット](media/azure-stack-automatic-log-collection/region-properties-blade-with-stamp-cloud-id.png)


## <a name="use-the-privileged-endpoint"></a>特権エンドポイントを使用する

1. 管理者特権の PowerShell セッションを開き、次のスクリプトを実行します。 必要に応じて、お使いの環境に合わせて PEP VM の IP アドレスとクラウド管理者の資格情報を置き換えます。 

   ```powershell
   $ipAddress = "<IP ADDRESS OF THE PEP VM>" # You can also use the machine name instead of IP here.

   $password = ConvertTo-SecureString "<CLOUD ADMIN PASSWORD>" -AsPlainText -Force
   $cred = New-Object -TypeName System.Management.Automation.PSCredential ("<DOMAIN NAME>\CloudAdmin", $password)
   $session = New-PSSession -ComputerName $ipAddress -ConfigurationName PrivilegedEndpoint -Credential $cred

   $stampInfo = Invoke-Command -Session $session { Get-AzureStackStampInformation }
   if ($session) {
       Remove-PSSession -Session $session
   }

   $stampInfo.CloudID
   ```

## <a name="next-steps"></a>次のステップ

* [ログを事前に送信する](./diagnostic-log-collection.md#send-logs-proactively)
* [今すぐログを送信する](./diagnostic-log-collection.md#send-logs-now)