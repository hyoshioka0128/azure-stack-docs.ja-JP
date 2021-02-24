---
title: Azure Stack HCI 用にファイアウォールを構成する
description: このトピックでは、Azure Stack HCI オペレーティング システム用にファイアウォールを構成する方法に関するガイダンスを提供します。
author: JohnCobb1
ms.author: v-johcob
ms.topic: how-to
ms.date: 01/06/2020
ms.openlocfilehash: a67881f2dd4be5e4dce5fb967c88484c27025624
ms.sourcegitcommit: 9b0e1264ef006d2009bb549f21010c672c49b9de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98255233"
---
# <a name="configure-firewalls-for-azure-stack-hci"></a>Azure Stack HCI 用にファイアウォールを構成する

>適用対象:Azure Stack HCI バージョン 20H2

このトピックでは、Azure Stack HCI オペレーティング システム用にファイアウォールを構成する方法に関するガイダンスを提供します。 接続の要件が含まれており、オペレーティング システムからアクセスする必要がある Azure 内の IP アドレスをサービス タグでグループ化する方法について説明しています。 このトピックでは、Microsoft Defender ファイアウォールを更新する手順も提供します。

## <a name="connectivity-requirements"></a>接続の要件
Azure Stack HCI は、定期的に Azure に接続する必要があります。 アクセスは次のものにのみ制限されます。
- 既知の Azure IP
- 送信方向
- ポート 443 (HTTPS)

詳細については、「[Azure Stack HCI の FAQ](../faq.md)」の「Azure Stack HCI の接続」セクションを参照してください

このトピックでは、許可リストに含まれているものを除くすべての送信先へのすべてのトラフィックをブロックするために、頻繁にロックダウンされるファイアウォールの構成を必要に応じて使用する方法について説明します。

   >[!IMPORTANT]
   > 送信接続が外部の企業のファイアウォールやプロキシ サーバーによって制限されている場合は、必ず、以下の表に一覧表示されている URL がブロックされないようにしてください。 関連情報については、「[Azure Arc 対応サーバー エージェントの概要](/azure/azure-arc/servers/agent-overview#networking-configuration)」の「ネットワーク構成」セクションを参照してください。


以下に示すように、Azure Stack HCI の場合、複数のファイアウォールを使用して Azure にアクセスします。

:::image type="content" source="./media/configure-firewalls/firewalls-diagram.png" alt-text="図は、ファイアウォールのポート 443 (HTTPS) を介してサービス タグ エンドポイントにアクセスする Azure Stack HCI を示しています。" lightbox="./media/configure-firewalls/firewalls-diagram.png":::

## <a name="working-with-service-tags"></a>サービス タグの使用
"*サービス タグ*" は、指定された Azure サービスからの IP アドレスのグループを表します。 Microsoft では、サービス タグに含まれる IP アドレスを管理し、IP アドレスが変更されるとサービス タグを自動的に更新して更新を最小限に抑えるようにしています。 詳細については、「[仮想ネットワーク サービス タグ](/azure/virtual-network/service-tags-overview)」を参照してください。

## <a name="required-endpoint-daily-access-after-azure-registration"></a>必要なエンドポイントの毎日のアクセス (Azure の登録後)
Azure では、サービス タグを使用してまとめられた Azure サービスの既知の IP アドレスを保持します。 Azure では、あらゆるサービスのすべての IP アドレスの週単位の JSON ファイルを発行します。 IP アドレスは頻繁には変更されませんが、年に数回変更されます。 次の表は、オペレーティング システムからアクセスする必要があるサービス タグ エンドポイントを示しています。

| 説明                   | IP 範囲のサービス タグ  | [URL]                                                                                 |
| :-----------------------------| :-----------------------  | :---------------------------------------------------------------------------------- |
| Azure Active Directory        | AzureActiveDirectory      | `https://login.microsoftonline.com`<br> `https://graph.microsoft.com`               |
| Azure Resource Manager        | AzureResourceManager      | `https://management.azure.com`                        |
| Azure Stack HCI クラウド サービス | AzureFrontDoor.Frontend   | `https://azurestackhci.azurefd.net` |
| Azure Arc                     | AzureArcInfrastructure<br> AzureTrafficManager | 使用する機能によって異なります。<br> ハイブリッド ID サービス: `*.his.arc.azure.com`<br> ゲスト構成: `*.guestconfiguration.azure.com`<br> **注:** より多くの機能を有効にすると、URL が増えることが予想されます。 |

## <a name="update-microsoft-defender-firewall"></a>Microsoft Defender ファイアウォールを更新する
このセクションでは、サービス タグに関連付けられた IP アドレスでオペレーティング システムに接続できるように Microsoft Defender ファイアウォールを構成する方法を示します。

1. 次のリソースから、オペレーティング システムを実行しているターゲット コンピューターに JSON ファイルをダウンロードします: [Azure IP 範囲とサービス タグ – パブリック クラウド](https://www.microsoft.com/download/details.aspx?id=56519)。

1. 次の PowerShell コマンドを使用して JSON ファイルを開きます。

    ```powershell
    $json = Get-Content -Path .\ServiceTags_Public_20201012.json | ConvertFrom-Json
    ```

1. "AzureResourceManager" サービス タグなど、特定のサービス タグの IP アドレス範囲の一覧を取得します。

    ```powershell
    $IpList = ($json.values | where Name -Eq "AzureResourceManager").properties.addressPrefixes
    ```

1. 許可リストを使用する場合は、外部の企業のファイアウォールに IP アドレスの一覧をインポートします。

1. クラスター内の各サーバーに対してファイアウォール規則を作成し、IP アドレス範囲の一覧への送信 443 (HTTPS) トラフィックを許可します。

    ```powershell
    New-NetFirewallRule -DisplayName "Allow Azure Resource Manager" -RemoteAddress $IpList -Direction Outbound -LocalPort 443 -Protocol TCP -Action Allow -Profile Any -Enabled True
    ```

## <a name="additional-endpoint-for-one-time-azure-registration"></a>1 回限りの Azure 登録用の追加のエンドポイント
Azure の登録プロセス中に、`Register-AzStackHCI` を実行するか、Windows Admin Center を使用すると、Az や AzureAD などの必要な PowerShell モジュールの最新バージョンがあることを確認するために、コマンドレットによって PowerShell ギャラリーへの接続が試行されます。 PowerShell ギャラリーは Azure でホストされていますが、現在はそのためのサービス タグはありません。 インターネットにアクセスできないためにサーバー ノードから `Register-AzStackHCI` コマンドレットを実行できない場合は、管理コンピューターにモジュールをダウンロードしてから、コマンドレットを実行するサーバー ノードにそれらを手動で転送することをお勧めします。

## <a name="next-steps"></a>次のステップ
詳細については、次のトピックも参照してください。
- 「[Azure Stack HCI の FAQ](../faq.md)」の接続に関するセクション