---
title: Azure Stack HCI の言語を変更する
description: このトピックでは、Azure Stack HCI オペレーティング システム、Windows 10、Windows Admin Center、および Microsoft Edge の言語を変更する方法に関するガイダンスを提供します。
author: JohnCobb1
ms.author: v-johcob
ms.topic: how-to
ms.date: 07/21/2020
ms.openlocfilehash: 09245e4498e87896b95df993e23693ae1ed23eaf
ms.sourcegitcommit: a15a0f955bac922cebb7bf90a72384fd84ddfe56
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/22/2020
ms.locfileid: "86947335"
---
# <a name="change-languages-in-azure-stack-hci"></a>Azure Stack HCI の言語を変更する

>適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

管理しているサーバーで、管理 PC、Windows Admin Center、Microsoft Edge、および Azure Stack HCI オペレーティング システムの優先言語を変更できます。 Windows Admin Center は、これらの他のリソースでの言語の変更に関係なく、選択した言語のままになります。

この記事では、以下において言語を変更する方法について説明します。
- Windows 10
- Windows Admin Center
- Microsoft Edge
- Azure Stack HCI オペレーティング システムの Server Core

## <a name="change-the-language-in-the-management-pc"></a>管理 PC の言語を変更する
管理 PC で Windows 10 を使用している場合は、Windows 10、使用するアプリ、およびアクセスする Web サイトで使用する言語パックをインストールできます。 また、キーボードの言語を変更したり、Web サイトとアプリの言語の優先順位で入力言語を設定したりすることもできます。

Windows 10 で使用する言語またはキーボード言語を変更しても、Windows Admin Center の表示言語には影響しません。

   >[!IMPORTANT]
   > Windows 10 で言語を変更した後、キーボード入力に関する問題が発生しないようにするために、サインアウトする前に、優先するキーボード言語を既定値として設定することをお勧めします。

詳細については、「[Windows 10 で入力および表示の言語設定を管理する](https://support.microsoft.com/help/4496404/windows-10-manage-the-input-and-display-language)」を参照してください。

## <a name="change-the-language-in-windows-admin-center"></a>Windows Admin Center で言語を変更する
Windows Admin Center の言語と、必要な場合は場所に応じて地域設定を変更できます。 Windows Admin Center でこれらのオプションのいずれかを変更しても、Windows 10 を実行している管理 PC の言語設定には影響しません。

Windows Admin Center で言語を変更するには:
1. **[すべての接続]** ペインで、 **[設定]** の歯車アイコンを選択し、 **[設定]** で **[言語/地域]** を選択します。
1. **[言語]** ドロップダウン リストを展開して優先する言語を選択し、 **[地域設定]** ドロップダウン リストを展開して、必要に応じて別の地域を選択します。
1. **[Save and reload]\(保存して再読み込み\)** を選択します。

    :::image type="content" source="media/languages/language-region.png" alt-text="Windows Admin Center の [言語/地域] ページ":::

詳細については、「[Windows Admin Center の設定](/windows-server/manage/windows-admin-center/configure/settings)」を参照してください。

## <a name="change-the-language-in-microsoft-edge"></a>Microsoft Edge で言語を変更する
サポートされる言語を Microsoft Edge に追加し、ブラウザーで言語設定の順序を変更することができます。 また、外国語翻訳拡張機能をブラウザーに追加して、翻訳を取得することもできます。

詳細については、「[Microsoft Edge 言語サポート](/deployedge/microsoft-edge-supported-languages)」を参照してください。

## <a name="change-the-language-in-server-core"></a>Server Core の言語を変更する
Azure Stack HCI オペレーティング システムの Server Core の言語を変更する必要がある場合は、サーバーをクラスター化した後にそうすることをお勧めします。 サポートされている言語パックを Server Core に追加し、使用する言語とキーボード レイアウトに変更できます。 Windows PowerShell コマンドを使用して、現在の言語とキーボードの入力方法をオーバーライドすることもできます。

各言語パックは、 *%SystemRoot%\System32\\%Language-ID%* ディレクトリにインストールされます。 たとえば、*C:\Windows\System32\es-ES* はスペイン語の言語パックの場所です。 各言語パックは約 50 MB です。 38 個のすべての言語パックをインストールする場合、作成する必要なイメージのサイズは約 2 GB です。

詳細については、「[Windows で使用可能な言語](/windows-hardware/manufacture/desktop/available-language-packs-for-windows)」を参照してください。

言語パックを手動で取得し、オペレーティング システムに追加するには、次の操作を行います。

1. **DISM / Add-WindowsPackage** ツールを使用して、Server Core に言語パックを追加します。 DISM 実行可能ファイルに相当するのは、`Add-WindowsPackage` PowerShell コマンドです。

    詳細については、「[Windows イメージへの言語の追加](/windows-hardware/manufacture/desktop/add-language-packs-to-windows)」を参照してください。

1. 次の例に示すように、言語のオンデマンド機能 (FOD) を追加して、追加した言語の追加機能を有効にすることをお勧めします。

    ```DOS
    dism /online /add-capability /capabilityname:Language.Basic~~~de-de~0.0.1.0
    ```

    詳細については、「[言語とリージョンのオンデマンド機能 (FOD)](/windows-hardware/manufacture/desktop/features-on-demand-language-fod)」を参照してください。

1. PowerShell の **Set-WinUILanguageOverride** コマンドレットを使用して、現在のユーザー アカウントのオペレーティング システムでの Windows UI 言語を変更できます。 次の例で、`de-DE` は、オペレーティング システムの現在の言語設定としてドイツ語を指定しています。

    ```PowerShell
    Set-WinUILanguageOverride de-DE
    ```

    詳細については、「[Set-WinUILanguageOverride](/powershell/module/international/set-winuilanguageoverride?view=win10-ps)」を参照してください。

## <a name="next-steps"></a>次のステップ
詳細については、次のトピックも参照してください。

- [Azure Stack HCI クラスターのサーバーを追加または削除する](./add-cluster.md)
