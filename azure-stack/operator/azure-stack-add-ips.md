---
title: Azure Stack Hub にパブリック IP アドレスを追加する
description: Azure Stack Hub にパブリック IP アドレスを追加する方法について説明します。
author: PatAltimore
ms.topic: article
ms.date: 05/28/2020
ms.author: patricka
ms.reviewer: scottnap
ms.lastreviewed: 09/10/2019
ms.openlocfilehash: 5f083e507689e7d24fa02060e63a9e6302ef3568
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97871941"
---
# <a name="add-public-ip-addresses"></a>パブリック IP アドレスの追加

この記事では、外部アドレスをパブリック IP アドレスと呼びます。 Azure Stack Hub のコンテキストでは、パブリック IP アドレスは、Azure Stack Hub の外部からアクセスできる IP アドレスです。 この記事の目的では、外部ネットワークがルーティング可能なパブリック インターネットであるか、イントラネット上にあるか、またプライベート アドレス空間を使用するかはあまり重要ではなく、手順は同じです。 

複数の IP プールを設定することはできますが、どれを使用するかを選択することはできません。 Azure Stack Hub はすべての IP プールを 1 つとして利用します。 リソースを作成するときに、割り当ての IP を選択することはできません。

> [!IMPORTANT]
> この記事の手順は、パートナー ツールキット バージョン 1809 以降を使用してデプロイされたシステムにのみ適用されます。 バージョン 1809 より前にデプロイされたシステムでは、新しいパブリック VIP プール範囲を許可するために、Top-of-Rack (TOR) スイッチ アクセス制御リスト (ACL) を更新する必要があります。 古いスイッチ構成を実行している場合は、OEM と協力して、新しいパブリック IP プールに適切な許可 ACL を追加するか、最新のパートナー ツールキットを使用して、スイッチを再構成し、新しいパブリック IP アドレスがブロックされないようにします。

## <a name="add-a-public-ip-address-pool"></a>パブリック IP アドレス プールを追加する
Azure Stack Hub システムの初期デプロイ後はいつでも、Azure Stack Hub システムにパブリック IP アドレスを追加できます。 [パブリック IP アドレスの使用量を表示する](azure-stack-viewing-public-ip-address-consumption.md)方法を参照し、ご利用の Azure Stack Hub での現在の使用量とパブリック IP アドレスの可用性を確認してください。

Azure Stack Hub に新しいパブリック IP アドレス ブロックを追加するプロセスの概要は次のようになります。

 ![IP の追加フロー](media/azure-stack-add-ips/flow.svg)

## <a name="obtain-the-address-block-from-your-provider"></a>プロバイダーからアドレス ブロックを取得する
まず、Azure Stack Hub に追加するアドレス ブロックを取得する必要があります。 どこからアドレス ブロックを取得するかに応じて、リード タイムを考慮し、Azure Stack Hub でのパブリック IP アドレスの使用率と照らし合わせて管理します。

> [!IMPORTANT]
> Azure Stack Hub では、有効なアドレス ブロックであり、Azure Stack Hub の既存のアドレス範囲と重複していなければ、指定するすべてのアドレス ブロックが受け入れられます。 Azure Stack Hub が接続されている外部ネットワークにルーティング可能で、かつ重複していない、有効なアドレス ブロックを取得するようにしてください。 Azure Stack Hub に範囲を追加すると、削除できません。

## <a name="add-the-ip-address-range-to-azure-stack-hub"></a>Azure Stack Hub に IP アドレス範囲を追加する

1. ブラウザーで、管理者ポータル ダッシュボードにアクセスします。 この例では、`https://adminportal.local.azurestack.external` を使用します。
2. Azure Stack Hub 管理者ポータルにクラウド オペレーターとしてサインインします。
3. 既定のダッシュボードで、[Region management]\(リージョンの管理\) リストを見つけ、管理するリージョンを選択します。 この例では、ローカルを使用します。
4. [リソース プロバイダー] タイルを見つけて、ネットワーク リソース プロバイダーをクリックします。
5. [パブリック IP プールの使用量] タイルをクリックします。
6. [Add IP pool]\(IP プールの追加\) ボタンをクリックします。
7. IP プールの名前を指定します。 選択した名前によって、IP プールを簡単に識別できます。 このフィールドでは、"/" のような特殊文字は使用できません。 必須ではありませんが、アドレス範囲と同じ名前にすることをお勧めします。
8. CIDR 表記で追加するアドレス ブロックを入力します。 次に例を示します。192.168.203.0/24
9. [アドレス範囲 (CIDR ブロック)] フィールドに有効な CIDR 範囲を指定すると、[開始 IP アドレス]、[終了 IP アドレス] および [利用可能な IP アドレス] の各フィールドが自動的に設定されます。 これらは読み取り専用で、自動的に生成されるため、[アドレス範囲] フィールドで値を変更せずにこれらのフィールドを変更することはできません。
10. ブレード上の情報を確認し、すべて正しいことを確認したら、 **[OK]** を選択して変更をコミットし、アドレス範囲を Azure Stack Hub に追加します。


## <a name="next-steps"></a>次のステップ 
[スケール ユニットのノード操作を確認します](azure-stack-node-actions.md)。
