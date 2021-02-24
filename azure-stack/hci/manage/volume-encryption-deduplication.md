---
title: ボリュームの暗号化、重複除去、圧縮を有効にする - Azure Stack HCI
description: このトピックでは、Windows Admin Center を使用して、Azure Stack HCI でボリュームの暗号化、重複除去、圧縮を使用する方法について説明します。
author: JohnCobb1
ms.author: v-johcob
ms.topic: how-to
ms.date: 09/03/2020
ms.openlocfilehash: 9599a4d24d93e545c964de91ad10b905c79b42a1
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90573498"
---
# <a name="enable-volume-encryption-deduplication-and-compression-in-azure-stack-hci"></a>Azure Stack HCI でボリュームの暗号化、重複除去、圧縮を有効にする

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

このトピックでは、Windows Admin Center を使用して、Azure Stack HCI でボリュームに対して BitLocker を使用した暗号化を有効にする方法について説明します。 また、ボリュームの重複除去と圧縮を有効にする方法についても説明します。 ボリュームの作成方法については、「[ボリュームの作成](create-volumes.md)」を参照してください。

## <a name="turn-on-bitlocker-to-protect-volumes"></a>BitLocker をオンにしてボリュームを保護する
Windows Admin Center で BitLocker をオンにするには、次のようにします。

1. 記憶域スペース ダイレクト クラスターに接続し、 **[ツール]** ペインで **[ボリューム]** を選択します。
1. **[ボリューム]** ページで **[インベントリ]** タブを選択し、 **[オプション機能]** で **[暗号化 (BitLocker)]** トグルをオンに切り替えます。

    :::image type="content" source="media/volume-encryption-deduplication/bitlocker-toggle-switch.png" alt-text="BitLocker を有効にするトグル スイッチ":::

1. **[暗号化 (BitLocker)]** ポップアップで、 **[開始]** を選択し、 **[Turn on Encryption]\(暗号化を有効にする\)** ページで、ワークフローを完了するための資格情報を入力します。

>[!NOTE]
   > **[Install BitLocker feature first]\(BitLocker 機能を最初にインストールしてください\)** ポップアップが表示される場合は、指示に従ってクラスター内の各サーバーに機能をインストールし、サーバーを再起動します。

## <a name="turn-on-deduplication-and-compression"></a>重複除去と圧縮を有効にする
重複除去と圧縮はボリュームごとに管理されます。 重複除去と圧縮では後処理モデルが使用されるため、実行されるまで節約は確認されません。 実行されると、以前から存在するファイルであっても、すべてのファイルが処理されます。

Windows Admin Center でボリュームの重複除去と圧縮を有効にするには、次のようにします。

1. 記憶域スペース ダイレクト クラスターに接続し、 **[ツール]** ペインで **[ボリューム]** を選択します。
1. **[ボリューム]** ページで、 **[インベントリ]** タブを選択します。
1. ボリュームの一覧で、管理するボリュームの名前を選択します。
1. ボリュームの詳細ページで、 **[Deduplication and compression]\(重複除去と圧縮\)** トグルをオンにします。
1. **[重複除去を有効にする]** ペインで、重複除去モードを選択します。

    Windows Admin Center では、複雑な設定の代わりに、さまざまなワークロードに対応した既製のプロファイルを選択できます。 わからない場合は、既定の設定を使用します。

1. **[重複除去を有効にする]** を選択します。

重複除去と圧縮を有効にする方法に関する簡単な動画をご覧ください。 この動画では、暗号化は説明されていません。

> [!VIDEO https://www.youtube-nocookie.com/embed/PRibTacyKko]

ボリュームの暗号化を有効にすると、ボリュームのパフォーマンスにわずかに影響します。通常は 10% 未満ですが、影響はハードウェアとワークロードによって異なります。 データの重複除去と圧縮もパフォーマンスに影響します。詳細については、「[データの重複除去の候補となるワークロードを特定する](/windows-server/storage/data-deduplication/install-enable#enable-dedup-candidate-workloads)」を参照してください。

<!---Add info on greyed out ReFS option? --->

以上で終わりです。 ボリューム内のデータを保護するために、必要に応じて繰り返してください。

## <a name="next-steps"></a>次のステップ
関連トピックおよびその他のストレージ管理タスクについては、次のトピックも参照してください。

- [記憶域スペース ダイレクトの概要](/windows-server/storage/storage-spaces/storage-spaces-direct-overview)
- [ボリュームの計画](../concepts/plan-volumes.md)
- [ボリュームの拡張](extend-volumes.md)
- [ボリュームの削除](delete-volumes.md)