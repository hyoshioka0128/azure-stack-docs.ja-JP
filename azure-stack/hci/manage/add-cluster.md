---
title: Azure Stack HCI クラスターのサーバーを追加または削除する
description: Azure Stack HCI のクラスターに対してサーバー ノードを追加または削除する方法について説明します
ms.topic: how-to
author: v-dasis
ms.author: v-dasis
ms.reviewer: jgerend
ms.date: 01/29/2021
ms.openlocfilehash: 0d8aa3cc6418c8e64b7a8dc97290757c8be41856
ms.sourcegitcommit: e56b0eaf92c633d5d782bfdf17ce521fa88a7256
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/01/2021
ms.locfileid: "99227432"
---
# <a name="add-or-remove-servers-for-an-azure-stack-hci-cluster"></a>Azure Stack HCI クラスターのサーバーを追加または削除する

> 適用対象:Azure Stack HCI バージョン 20H2

Azure Stack HCI のクラスターに対してサーバーを簡単に追加または削除できます。 新しい各物理サーバーは、CPU の種類、メモリ、ドライブの数、ドライブの種類とサイズに関して、クラスター内の他のサーバーと厳密に一致している必要があることに注意してください。

サーバーを追加または削除したときは必ず、後でクラスター検証を実行して、クラスターが正常に機能していることを確認することも必要です。 これは、非ストレッチ クラスターとストレッチ クラスターの両方に適用されます。

## <a name="before-you-begin"></a>始める前に

最初の手順は、元の OEM から新しい HCI ハードウェアを入手することです。 クラスターで使用するための新しいサーバー ハードウェアを追加するときは、OEM 提供のドキュメントを必ず参照してください。

1. 新しい物理サーバーをラックに設置し、正しく配線します。
1. 物理スイッチ ポートを有効にし、アクセス制御リスト (ACL) と VLAN ID を調整します (該当する場合)。
1. OEM の指示に従って、ベースボード管理コントローラー (BMC) で正しい IP アドレスを構成し、すべての BIOS 設定を適用します。
1. OEM によって提供されるツールを使用して、最新のファームウェア ベースラインをすべてのコンポーネントに適用します。
1. OEM 検証テストを実行して、既存のクラスター サーバーとのハードウェアの同質性を確保します。
1. Azure Stack HCI オペレーティング システムを新しいサーバーにインストールします。 詳細については、[Azure Stack HCI のデプロイ](../deploy/operating-system.md)に関するページをご覧ください。
1. サーバーをクラスター ドメインに参加させます。

## <a name="add-a-server-to-a-cluster"></a>クラスターにサーバーを追加する

Windows Admin Center を使用して、サーバーをクラスターに参加させます。

:::image type="content" source="media/manage-cluster/add-server.png" alt-text="サーバーの追加画面" lightbox="media/manage-cluster/add-server.png":::

1. **Windows Admin Center** で、上部のドロップダウン矢印から **[Cluster Manager]\(クラスター マネージャー\)** を選択します。
1. **[Cluster connections]\(クラスター接続\)** の下で、クラスターを選択します。
1. **[ツール]** の下で、 **[サーバー]** を選択します。
1. **[サーバー]** の下で、 **[インベントリ]** タブを選択します。
1. **インベントリ** タブで、**追加］** を選択します。
1. **[サーバー名]** に、追加するサーバーの完全修飾ドメイン名を入力し、 **[追加]** をクリックして、下部にある **[追加]** をもう一度クリックします。
1. サーバーがクラスターに正常に追加されたことを確認します。

## <a name="remove-a-server-from-a-cluster"></a>クラスターからサーバーを削除する

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

クラスターのサーバーを追加または削除する場合は常に、後でクラスター検証テストを実行してください。

## <a name="add-server-pairs-to-a-stretched-cluster"></a>ストレッチ クラスターにサーバー ペアを追加する

ストレッチ クラスターでは、各サイトに同じ数のサーバー ノードと同じ数のドライブが必要です。 ストレッチ クラスターにサーバー ペアを追加すると、そのドライブは、ストレッチ クラスターの両方のサイトの記憶域プールにすぐに追加されます。 追加時に各サイトの記憶域プールのサイズが同じでない場合は、拒否されます。 これは、記憶域プールのサイズは、サイト間で同じである必要があるためです。

ストレッチ クラスターへのサーバー ノードの追加について、少し時間を取ってビデオをご覧ください。

> [!VIDEO https://www.youtube.com/embed/AVHPkRmsZ5Y]

ストレッチ クラスターのサーバーの追加または削除は、Windows PowerShell を使用して行います。 [Get-ClusterFaultDomainXML](/powershell/module/failoverclusters/get-clusterfaultdomainxml) および [Set-ClusterFaultDomainXML](/powershell/module/failoverclusters/set-clusterfaultdomainxml) コマンドレットを使用して、サーバーを追加する前に、まずサイト (障害ドメイン) の情報を変更します。

次に、[Add-ClusterNode](/powershell/module/failoverclusters/add-clusternode) コマンドレットを使用して、各サイトにサーバー ペアを同時に追加できます。これにより、新しいサーバーの各ドライブも同時に追加できます。

通常、クラスターは、クラスター内のサーバーではなく、リモート コンピューターから管理します。 このリモート コンピューターは、管理コンピューターと呼ばれます。

> [!NOTE]
> 管理コンピューターから PowerShell コマンドを実行する場合は、管理しているクラスターの名前を指定した `-Cluster` パラメーターを含めます。

では、始めましょう。

1. 次の PowerShell コマンドレットを使用して、クラスターの状態を確認します。

    クラスター内のアクティブなサーバーの一覧を取得します。

     ```powershell
    Get-ClusterNode
    ```

    クラスターの記憶域プールの統計情報を取得します。

    ```powershell
    Get-StoragePool pool*
    ```

    サイト (障害ドメイン) ごとにサーバーの一覧を取得します。

    ```powershell
    Get-ClusterFaultDomain
    ```

1. メモ帳または他のテキスト エディターで `Sites.xml` ファイルを開きます。

    ```powershell
    Get-ClusterFaultDomainXML | out-file sites.xml
    ```
 
    ```powershell
    notepad
    ```

1. 管理 PC で `Sites.xml` ファイルがローカルに配置されている場所に移動し、ファイルを開きます。 `Sites.xml` ファイルは次のような内容です。

    ```
    <Topology>
        <Site Name="Site1" Description="" Location="">
            <Node Name="Server1" Description="" Location="">
            <Node Name="Server2" Description="" Location="">
        </Site>
        <Site Name="Site2" Description="" Location="">
            <Node Name="Server3" Description="" Location="">
            <Node Name="Server4" Description="" Location="">
        </Site>
    <Topology>
    ```

1. この例を使用して、次のように各サイト (`Server5`、`Server6`) にサーバーを追加します。

    ```
    <Topology>
        <Site Name="Site1" Description="" Location="">
            <Node Name="Server1" Description="" Location="">
            <Node Name="Server2" Description="" Location="">
            <Node Name="Server5" Description="" Location="">
        </Site>
        <Site Name="Site2" Description="" Location="">
            <Node Name="Server3" Description="" Location="">
            <Node Name="Server4" Description="" Location="">
            <Node Name="Server6" Description="" Location="">
        </Site>
    <Topology>
    ```

1. 現在のサイト (障害ドメイン) の情報を変更します。  最初のコマンドで、変数を設定して `Sites.xml` ファイルの内容を取得し、それを出力します。 2 番目のコマンドで、変数 `$XML` に基づいて変更を設定します。

    ```
    $XML = Get-Content .\sites.xml | out-string
    Set-ClusterFaultDomainXML -xml $XML
    ```

1. 行った変更が正しいことを確認します。

    ```
    Get-ClusterFaultDomain
    ```

1. `Add-ClusterNode` コマンドレットを使用して、クラスターにサーバー ペアを追加します。

    ```
    Add-ClusterNode -Name Server5,Server6
    ```

サーバーが正常に追加されると、関連付けられているドライブが各サイトの記憶域プールに自動的に追加されます。 最後に、ヘルス サービスによって、新しいドライブを組み込むためのストレージ ジョブが作成されます。

## <a name="remove-server-pairs-from-a-stretched-cluster"></a>ストレッチ クラスターからサーバー ペアを削除する

ストレッチ クラスターからのサーバー ペアの削除は、サーバー ペアの追加と同様のプロセスですが、代わりに [Remove-ClusterNode](/powershell/module/failoverclusters/remove-clusternode) コマンドレットを使用します。

1. 次の PowerShell コマンドレットを使用して、クラスターの状態を確認します。

    クラスター内のアクティブなサーバーの一覧を取得します。

     ```powershell
    Get-ClusterNode
    ```

    クラスターの記憶域プールの統計情報を取得します。

    ```powershell
    Get-StoragePool pool*
    ```

    サイト (障害ドメイン) ごとにサーバーの一覧を取得します。

    ```powershell
    Get-ClusterFaultDomain
    ```

1. メモ帳または他のテキスト エディターで `Sites.xml` ファイルを開きます。

    ```powershell
    Get-ClusterFaultDomainXML | out-file sites.xml
    ```
 
    ```powershell
    notepad
    ```

1. 前の例を使用し、`Sites.xml` ファイルで、各サイトの `<Node Name="Server5" Description="" Location="">` と `<Node Name="Server6" Description="" Location="">` XML エントリを削除します。
1. 次の 2 つのコマンドレットを使用して、現在のサイト (障害ドメイン) の情報を変更します。

    ```
    $XML = Get-Content .\sites.xml | out-string
    Set-ClusterFaultDomainXML -xml $XML
    ```

1. 行った変更が正しいことを確認します。

    ```
    Get-ClusterFaultDomain
    ```
1. `Remove-ClusterNode` コマンドレットを使用して、クラスターからサーバー ペアを削除します。

    ```
    Remove-ClusterNode -Name Server5,Server6
    ```

サーバーが正常に削除されると、関連付けられているドライブがサイト プールから自動的に削除されます。 最後に、ヘルス サービスによって、これらのドライブを削除するためのストレージ ジョブが作成されます。

## <a name="next-steps"></a>次のステップ

- サーバーを追加または削除した後は、クラスターを検証する必要があります。 詳細については、[クラスターの検証](../deploy/validate.md)に関するページを参照してください。