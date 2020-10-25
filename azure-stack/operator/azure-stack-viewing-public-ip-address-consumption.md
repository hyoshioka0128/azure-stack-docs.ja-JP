---
title: ネットワーク リソースの管理
titleSuffix: Azure Stack Hub
description: MAC アドレス プールや、リージョン内のパブリック IP アドレスの使用など、ネットワーク リソースを管理する方法について説明します。
author: IngridAtMicrosoft
ms.topic: conceptual
ms.date: 1/22/2020
ms.author: inhenkel
ms.reviewer: scottnap
ms.lastreviewed: 09/17/2019
ms.openlocfilehash: b96485e74286cbbbd2ff20f2378dc7c452849b52
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86487398"
---
# <a name="manage-network-resources-in-azure-stack-hub"></a>Azure Stack Hub 上でのネットワーク リソースの管理

## <a name="mac-address-pool"></a>MAC アドレス プール

Azure Stack Hub では、MAC アドレスを自動的に生成して仮想マシン (VM) に割り当てるために、静的 MAC アドレス プールが使用されます。 この MAC アドレス プールは、デプロイ時に自動的に生成され、次の範囲を使用します。

- StartMacAddress:00-1D-D8-B7-00-00
- EndMacAddress:00-1D-D8-F4-FF-FF

> [!Note]  
> この MAC アドレス プールは、各 Azure Stack Hub システムで同じで、構成することはできません。

仮想ネットワークと既存の企業ネットワークとの接続方法によっては、VM の MAC アドレスが重複することがあります。

MAC アドレス プールの使用率の詳細については、Azure Stack Hub 管理者の PowerShell モジュール内にあるコマンドレット [Get AzsMacAddressPool](/powershell/module/azs.fabric.admin/get-azsmacaddresspool) を使用して確認できます。

## <a name="view-public-ip-address-consumption-in-azure-stack-hub"></a>Azure Stack Hub でパブリック IP アドレスの使用量を表示する

クラウド管理者は、以下を表示できます。
 - テナントに割り当てられたパブリック IP アドレスの数。
 - 現在割り当てに使用できるパブリック IP アドレスの数。
 - その場所に割り当てられたパブリック IP アドレスの割合。

**[Public IP pools usage]\(パブリック IP プールの使用量\)** タイルには、パブリック IP アドレス プール全体で使用されているパブリック IP アドレスの数が表示されます。 各 IP アドレスについて、テナント IaaS VM インスタンス、ファブリック インフラストラクチャ サービス、テナントによって明示的に作成されたパブリック IP アドレス リソースの使用量がタイルに表示されます。

タイルの目的は、その場所で使用されているパブリック IP アドレスの数を Azure Stack Hub オペレーターに直観的に理解してもらうことです。 数を理解することで、管理者はこのリソースをローコストで運用しているかどうかを判断できます。

**[テナント リソース]** の下にある **[パブリック IP アドレス]** メニュー項目は、"*テナントによって明示的に作成されている*" パブリック IP アドレスのみを表示します。 メニュー項目は、 **[リソース プロバイダー]**  ->  **[ネットワーク]** ウィンドウで簡単に見つかります。 **[Public IP pools usage]\(パブリック IP プールの使用量\)** タイルの **[使用済み]** パブリック IP アドレスの数は、 **[テナント リソース]** の **[パブリック IP アドレス]** タイルの数と一致することはなく、常にその数よりも大きくなります。

### <a name="view-the-public-ip-address-usage-information"></a>パブリック IP アドレスの使用量情報を表示する

リージョンで使用されているパブリック IP アドレスの合計数を表示するには:

1. Azure Stack Hub 管理者ポータルで、 **[すべてのサービス]** を選択します。 次に、 **[管理]** カテゴリで **[ネットワーク]** を選択します。
1. **[ネットワーク]** ウィンドウの **[概要]** セクションに、 **[Public IP pools usage]\(パブリック IP プールの使用量\)** タイルが表示されます。

    ![Azure Stack Hub 管理者ポータルの [ネットワーク リソース プロバイダー] ウィンドウ](media/azure-stack-viewing-public-ip-address-consumption/ip-address-consumption-01.png)

**[使用済み]** の数は、パブリック IP アドレス プールにおける割り当て済みのパブリック IP アドレスの数を表しています。 **[フリー]** の数は、パブリック IP アドレス プール内の、割り当て済みでないまだ使用可能なパブリック IP アドレスの数を表します。 **[% 使用]** の数は、使用済みまたは割り当て済みの数を、その場所にあるパブリック IP アドレス プール内のパブリック IP アドレスの合計数に対する割合として表わしています。

### <a name="view-the-public-ip-addresses-that-were-created-by-tenant-subscriptions"></a>テナントのサブスクリプションによって作成されたパブリック IP アドレスを表示する

**[テナント リソース]** の下にある **[パブリック IP アドレス]** を選択します。 特定のリージョンでテナントのサブスクリプションによって明示的に作成されたパブリック IP アドレスの一覧を確認します。

![Azure Stack Hub 管理者ポータルのテナントのパブリック IP アドレス](media/azure-stack-viewing-public-ip-address-consumption/ip-address-consumption-02.png)

動的に割り当て済みのパブリック IP アドレスが一覧に表示される場合があります。 ただし、アドレスはまだそれらに関連付けられていません。 アドレス リソースは、ネットワーク リソース プロバイダー内に作成されているものの、ネットワーク コントローラー内にはまだ作成されていません。

ネットワーク コントローラーは、リソースがインターフェイス、ネットワーク インターフェイス カード (NIC)、ロード バランサー、または仮想ネットワーク ゲートウェイにバインドされるまで、アドレスの割り当てを行いません。 パブリック IP アドレスがインターフェイスにバインドされると、ネットワーク コントローラーは IP アドレスの割り当てを行います。 アドレスは **[アドレス]** フィールドに表示されます。

### <a name="view-the-public-ip-address-information-summary-table"></a>パブリック IP アドレス情報の表示に関するまとめ

パブリック IP アドレスの割り当てはさまざまな状況で実行され、その状況に応じてどの一覧に表示されるかが決まります。

| **プライベート IP アドレスの割り当て状況** | **使用状況の概要に表示** | **テナントのパブリック IP アドレスの一覧に表示** |
| --- | --- | --- |
| NIC またはロード バランサーにまだ割り当てられていない動的パブリック IP アドレス (一時的)。 |いいえ |はい |
| NIC またはロード バランサーに割り当て済みの動的パブリック IP アドレス |はい |はい |
| テナントの NIC またはロード バランサーに割り当て済みの静的パブリック IP アドレス |はい |はい |
| ファブリック インフラストラクチャ サービス エンドポイントに割り当て済みの静的パブリック IP アドレス |はい |いいえ |
| IaaS VM インスタンス用に暗黙的に作成され、仮想ネットワークのアウトバウンド NAT で使用されるパブリック IP アドレス。 これらは、テナントが VM インスタンスを作成するたびにバック グラウンドで作成され、VM が情報をインターネットに送信できるようにします。 |はい |いいえ |

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub でストレージ アカウントを管理する](azure-stack-manage-storage-accounts.md)
