---
title: Azure Stack HCI クラスターのサーバーを追加または削除する
description: Azure Stack HCI のクラスターに対してサーバー ノードを追加または削除する方法について説明します
ms.topic: how-to
author: v-dasis
ms.author: v-dasis
ms.reviewer: jgerend
ms.date: 07/21/2020
ms.openlocfilehash: 9dfdbcab43694146c4190db8ef29905626a4d597
ms.sourcegitcommit: 0e52f460295255b799bac92b40122a22bf994e27
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/21/2020
ms.locfileid: "86866646"
---
# <a name="add-or-remove-servers-for-an-azure-stack-hci-cluster"></a>Azure Stack HCI クラスターのサーバーを追加または削除する

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

Azure Stack HCI のクラスターに対してサーバーを簡単に追加または削除できます。 新しい各物理サーバーは、CPU の種類、メモリ、ドライブの数、ドライブの種類とサイズに関して、クラスター内の他のサーバーと厳密に一致している必要があることに注意してください。

サーバーを追加または削除したときは必ず、後でクラスター検証を実行して、クラスターが正常に機能していることを確認することも必要です。

## <a name="obtain-oem-hardware"></a>OEM ハードウェアを入手する

最初の手順は、元の OEM から新しい HCI ハードウェアを入手することです。 クラスターで使用するための新しいサーバー ハードウェアを追加するときは、OEM 提供のドキュメントを必ず参照してください。

1. 新しい物理サーバーをラックに設置し、正しく配線します。
1. 物理スイッチ ポートを有効にし、アクセス制御リスト (ACL) と VLAN ID を調整します (該当する場合)。
1. OEM の指示に従って、ベースボード管理コントローラー (BMC) で正しい IP アドレスを構成し、すべての BIOS 設定を適用します。
1. OEM によって提供されるツールを使用して、最新のファームウェア ベースラインをすべてのコンポーネントに適用します。
1. OEM 検証テストを実行して、既存のクラスター サーバーとの同質性を確保します。

## <a name="add-a-server-to-the-cluster"></a>クラスターにサーバーを追加する

サーバーが正しく起動したら、Windows Admin Center を使用してサーバーをクラスターに参加させます。

:::image type="content" source="media/manage-cluster/add-server.png" alt-text="サーバーの追加画面" lightbox="media/manage-cluster/add-server.png":::

1. **Windows Admin Center** で、上部のドロップダウン矢印から **[Cluster Manager]\(クラスター マネージャー\)** を選択します。
1. **[Cluster connections]\(クラスター接続\)** の下で、クラスターを選択します。
1. **[ツール]** の下で、 **[サーバー]** を選択します。
1. **[サーバー]** の下で、 **[インベントリ]** タブを選択します。
1. **インベントリ** タブで、**追加］** を選択します。
1. **[サーバー名]** に、追加するサーバーの完全修飾ドメイン名を入力し、 **[追加]** をクリックして、下部にある **[追加]** をもう一度クリックします。
1. サーバーがクラスターに正常に追加されたことを確認します。

## <a name="remove-a-server-from-the-cluster"></a>クラスターからサーバーを削除する

クラスターからサーバーを削除する手順は、クラスターにサーバーを追加する手順と似ています。

サーバーを削除すると、そのサーバーに関連付けられている仮想マシン、ドライブ、およびワークロードもすべて削除されることに注意してください。

:::image type="content" source="media/manage-cluster/remove-server.png" alt-text="サーバーの削除のダイアログ" lightbox="media/manage-cluster/remove-server.png":::

1. **Windows Admin Center** で、上部のドロップダウン矢印から **[Cluster Manager]\(クラスター マネージャー\)** を選択します。
1. **[Cluster connections]\(クラスター接続\)** の下で、クラスターを選択します。
1. **[ツール]** の下で、 **[サーバー]** を選択します。
1. **[サーバー]** の下で、 **[インベントリ]** タブを選択します。
1. **[インベントリ]** タブで、削除するサーバーを選択し、 **[削除]** を選択します。
1. また、記憶域プールからサーバーのドライブをすべて削除するには、そのチェックボックスをオンにします。
1. サーバーがクラスターから正常に削除されたことを確認します。

クラスターのサーバー ノードを追加または削除する場合は常に、後でクラスター検証テストを実行してください。

## <a name="next-steps"></a>次のステップ

- ノードを追加または削除した後、クラスターを検証する必要があります。 詳細については、「[クラスターを検証する](../deploy/validate.md)」を参照してください。