---
title: Azure Stack HCI のボリュームを削除する
description: Windows Admin Center と PowerShell を使用して Azure Stack HCI でボリュームを削除する方法について説明します。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.date: 07/21/2020
ms.openlocfilehash: 7a90948cccb75bb8bb1578101d530c46852e99d6
ms.sourcegitcommit: 0e52f460295255b799bac92b40122a22bf994e27
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/21/2020
ms.locfileid: "86866524"
---
# <a name="deleting-volumes-in-azure-stack-hci"></a>Azure Stack HCI のボリュームの削除

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

このトピックでは、Windows Admin Center を使用して Azure Stack HCI クラスターのボリュームを削除する手順について説明します。

Windows Admin Center を使用したボリュームの削除方法を説明する短いビデオをご覧ください。

> [!VIDEO https://www.youtube-nocookie.com/embed/DbjF8r2F6Jo]

## <a name="use-windows-admin-center-to-delete-a-volume"></a>Windows Admin Center を使用してボリュームを削除する

1. Windows Admin Center で、記憶域スペース ダイレクト クラスターに接続し、 **[ツール]** ペインで **[ボリューム]** を選択します。
2. **[ボリューム]** ページで **[インベントリ]** タブを選択し、削除するボリュームを選択します。
3. ボリュームの詳細ページの上部で、 **[削除]** を選択します。
4. 確認のダイアログで、ボリュームの削除を確認するチェック ボックスをオンにして、 **[削除]** を選択します。

## <a name="delete-volumes-using-powershell"></a>PowerShell を使用してボリュームを削除する

記憶域スペース ダイレクト内のボリュームを削除するには、**Remove-VirtualDisk** コマンドレットを使用します。 このコマンドレットは、**VirtualDisk** オブジェクトを削除し、使用した領域を、**VirtualDisk** オブジェクトを公開する記憶域プールに返します。

まず、管理 PC で PowerShell を起動し、**Get-VirtualDisk** コマンドレットを **CimSession** パラメーターを指定して実行します。このパラメーターは、記憶域スペース ダイレクト クラスターまたはサーバー ノードの名前で、たとえば *clustername.microsoft.com* となります。

```PowerShell
Get-VirtualDisk -CimSession clustername.microsoft.com
```

これにより、クラスター上のボリューム名に対応する、 **-FriendlyName** パラメーターに使用可能な値の一覧が返されます。

### <a name="example"></a>例

*Volume1* というミラー ボリュームを削除するには、PowerShell で次のコマンドを実行します。

```PowerShell
Remove-VirtualDisk -FriendlyName "Volume1"
```

操作を実行してボリュームに含まれるすべてのデータを消去するかどうかを確認するメッセージが表示されます。 Y または N を選択します。

   > [!WARNING]
   > これは回復可能な操作ではありません。 この例では、**VirtualDisk** Volume オブジェクトが完全に削除されます。

## <a name="next-steps"></a>次のステップ

その他の重要な記憶域管理タスクの詳細な手順については、以下も参照してください。

- [ボリュームを計画する](../concepts/plan-volumes.md)
- [ボリュームを作成する](create-volumes.md)
- [ボリュームを拡張する](extend-volumes.md)