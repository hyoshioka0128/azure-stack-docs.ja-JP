---
title: Azure Stack Hub サブスクリプションまたはストレージ アカウントにストレージ エクスプローラーを接続する
description: Azure Stack Hub サブスクリプションにストレージ エクスプローラーを接続する方法について説明します。
author: mattbriggs
ms.topic: conceptual
ms.date: 2/1/2021
ms.author: mabrigg
ms.reviewer: xiaofmao
ms.lastreviewed: 11/11/2019
ms.openlocfilehash: d4fac8bd262260802c386446190557b1333a14ed
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99247236"
---
# <a name="connect-storage-explorer-to-an-azure-stack-hub-subscription-or-a-storage-account"></a>Azure Stack Hub サブスクリプションまたはストレージ アカウントにストレージ エクスプローラーを接続する

この記事では、[Azure Storage Explorer](/azure/vs-azure-tools-storage-manage-with-storage-explorer) を使用して Azure Stack Hub サブスクリプションとストレージ アカウントに接続する方法について説明します。 Storage Explorer は、Windows、macOS、Linux で Azure Stack Hub ストレージのデータを簡単に操作できるようにするスタンドアロン アプリです。

> [!NOTE]  
> Azure Stack Hub ストレージからデータを移動するために利用できるツールは、いくつかあります。 詳細については、[Azure Stack Hub ストレージのデータ転送ツール](azure-stack-storage-transfer.md)に関するページを参照してください。

まだインストールされていない場合は、[Storage Explorer をダウンロード](https://www.storageexplorer.com/)してインストールします。

Azure Stack Hub サブスクリプションまたはストレージ アカウントに接続したら、[Azure Storage Explorer の記事](/azure/vs-azure-tools-storage-manage-with-storage-explorer)を使用して Azure Stack Hub データを操作することができます。 

## <a name="prepare-for-connecting-to-azure-stack-hub"></a>Azure Stack Hub への接続を準備する

Storage Explorer で Azure Stack Hub サブスクリプションにアクセスするには、Azure Stack Hub への直接アクセス、または VPN 接続が必要です。 Azure Stack Hub への VPN 接続を設定する方法の詳細については、[VPN を使用した Azure Stack Hub への接続](../asdk/asdk-connect.md#connect-to-azure-stack-using-vpn)に関するページを参照してください。

> [!NOTE]  
> ASDK では、VPN 経由で ASDK に接続している場合、VPN 設定プロセス中に作成されたルート証明書 (CA.cer) を使用しないでください。  これは DER でエンコードされた証明書であり、Storage Explorer で Azure Stack Hub サブスクリプションを取得することは許可されません。 以下の手順を使用して、Storage Explorer で使用するための Base-64 でエンコードされた証明書をエクスポートします。

接続されていない統合システムおよび ASDK の場合は、内部エンタープライズ証明機関を使用して、Base-64 形式でルート証明書をエクスポートした後、Azure Storage Explorer にインポートすることをお勧めします。  

### <a name="export-and-then-import-the-azure-stack-hub-certificate"></a>Azure Stack Hub の証明書をエクスポートしインポートする

接続されていない統合システムおよび ASDK の場合は、Azure Stack Hub 証明書をエクスポートしてインポートします。 接続されている統合システムの場合は、証明書はパブリック的に署名されており、この手順は必要ありません。

1. Azure Stack Hub に VPN 接続された、Azure Stack Hub のホスト コンピューターまたはローカル コンピューターで `mmc.exe` を開きます。 

2. **[ファイル]** の **[スナップインの追加と削除]** を選択します。 [利用できるスナップイン] で、 **[証明書]** を選択します。 

3. **[コンピューター アカウント]** を選択し、 **[次へ]** を選択します。 **[ローカル コンピューター]** を選択し、 **[完了]** を選択します。

4.  **Console Root\Certificated (Local Computer)\Trusted Root Certification Authorities\Certificates** の直下で **AzureStackSelfSignedRootCert** を探します。

    ![Azure Stack Hub のルート証明書を mmc.exe で読み込む](./media/azure-stack-storage-connect-se/add-certificate-azure-stack.png)

5. 証明書を右クリックし、 **[すべてのタスク]**  >  **[エクスポート]** の順に選択したら、表示される手順に従って **Base-64 encoded X.509 (.CER)** で証明書をエクスポートします。

    エクスポートした証明書は、次の手順で使用します。

6. Storage Explorer を開始します。 **[Azure Storage へ接続]** ダイアログ ボックスが表示されたらそれをキャンセルします。

7. **[編集]** メニューで **[SSL 証明書]** を選択し、次に **[証明書のインポート]** を選択します。 ファイル ピッカー ダイアログ ボックスを使用して、前の手順でエクスポートした証明書を検索し、開きます。

    証明書のインポート後は、Storage Explorer の再起動を求めるメッセージが表示されます。

    ![Storage Explorer に証明書をインポートする](./media/azure-stack-storage-connect-se/import-azure-stack-cert-storage-explorer.png)

8. Storage Explorer が再起動したら、 **[編集]** メニューを選択し、 **[Target Azure Stack Hub APIs]\(Azure Stack Hub API を対象にする\)** が選択されているかどうかを確認します。 **[Target Azure Stack Hub]\(Azure Stack Hub を対象にする\)** が選択されていない場合は、選択してから Storage Explorer を再起動して、この変更を反映させます。 この構成は、Azure Stack Hub 環境との互換性を確保するために必要です。

    ![[Target Azure Stack Hub]\(Azure Stack Hub を対象にする\) が確実に選択されているようにする](./media/azure-stack-storage-connect-se/target-azure-stack-new.png)

## <a name="connect-to-an-azure-stack-hub-subscription-with-azure-ad"></a>Azure AD を使用して Azure Stack Hub サブスクリプションに接続する

Azure Active Directory (Azure AD) アカウントに属している Azure Stack Hub サブスクリプションに Storage Explorer を接続するには、次の手順を使用します。

1. Storage Explorer の左側のウィンドウで、 **[アカウントの管理]** を選択します。  
    サインインしている Microsoft サブスクリプションがすべて表示されます。

2. Azure Stack Hub サブスクリプションに接続するには、 **[アカウントの追加]** を選択します。

    ![Azure Stack Hub アカウントの追加](./media/azure-stack-storage-connect-se/add-azure-stack-account.png)

3. [Azure Storage へ接続] ダイアログ ボックスの **[Azure 環境]** で、 **[Azure]** 、 **[Azure 中国 21Vianet]** 、 **[Azure ドイツ]** 、 **[Azure 米国政府機関]** 、または **[新しい環境を追加します]** を選択します。 これは、使用されている Azure Stack Hub アカウントによって決まります。 **[サインイン]** を選択し、少なくとも 1 つのアクティブな Azure Stack Hub サブスクリプションに関連付けられた Azure Stack Hub アカウントを使用してサインインします。

    ![Azure Storage へ接続](./media/azure-stack-storage-connect-se/azure-stack-connect-to-storage.png)

4. Azure Stack Hub アカウントでのサインインに成功すると、左側のウィンドウに、そのアカウントに関連付けられている Azure Stack Hub サブスクリプションが表示されます。 操作する Azure Stack Hub サブスクリプションを選択してから、 **[適用]** を選択します。 ( **[すべてのサブスクリプション]** チェック ボックスをオンまたはオフにすると、一覧の Azure Stack Hub サブスクリプションがすべて選択された状態と、1 つも選択されていない状態を切り替えることができます)

    ![カスタム クラウド環境のダイアログ ボックスに情報を入力した後、Azure Stack Hub サブスクリプションを選択する](./media/azure-stack-storage-connect-se/select-accounts-azure-stack.png)

    左側のウィンドウに、選択した Azure Stack Hub サブスクリプションに関連付けられているストレージ アカウントが表示されます。

    ![Azure Stack Hub サブスクリプション アカウントを含むストレージ アカウントの一覧](./media/azure-stack-storage-connect-se/azure-stack-storage-account-list.png)

## <a name="connect-to-an-azure-stack-hub-subscription-with-ad-fs-account"></a>AD FS アカウントを使用して Azure Stack Hub サブスクリプションに接続する

> [!NOTE]  
> Azure のフェデレーション サービス (AD FS) のサインイン エクスペリエンスでは、Azure Stack Hub 1804 以降の更新プログラムが適用されたストレージ エクスプローラー 1.2.0 以降のバージョンがサポートされます。

AD FS アカウントに属している Azure Stack Hub サブスクリプションに Storage Explorer を接続するには、次の手順を使用します。

1. **[アカウントの管理]** を選択します。 エクスプローラーには、サインインした Microsoft サブスクリプションが一覧表示されます。
2. Azure Stack Hub サブスクリプションに接続するには、 **[アカウントの追加]** を選択します。

    ![Storage Explorer でアカウントを追加する方法を示すスクリーンショット。](media/azure-stack-storage-connect-se/add-an-account.png)

3. **[次へ]** を選択します。 [Azure Storage へ接続] ダイアログ ボックスの **[Azure 環境]** で **[Use Custom Environment] (カスタム環境の使用)** を選択し、 **[次へ]** をクリックします。

    ![Azure Storage への接続](media/azure-stack-storage-connect-se/connect-to-azure-storage.png)

4. Azure Stack Hub のカスタム環境に必要な情報を入力します。 

    | フィールド | Notes |
    | ---   | ---   |
    | 環境名 | このフィールドはユーザーがカスタマイズできます。 |
    | Azure Resource Manager endpoint (Azure Resource Manager エンドポイント) | Azure Stack Development Kit の Azure Resource Manager リソース エンドポイントの例は次のとおりです。<br>オペレーター用: `https://adminmanagement.local.azurestack.external` <br> ユーザー用: `https://management.local.azurestack.external` |

    Azure Stack Hub 統合システムで作業していて、管理エンドポイントが不明な場合は、オペレーターに問い合わせてください。

    ![アカウントの追加 - カスタム環境](./media/azure-stack-storage-connect-se/custom-environments.png)

5. **[サインイン]** を選択し、少なくとも 1 つのアクティブな Azure Stack Hub サブスクリプションに関連付けられている Azure Stack Hub アカウントに接続します。



6. 操作する Azure Stack Hub サブスクリプションを選択してから、 **[適用]** を選択します。

    ![アカウント管理](./media/azure-stack-storage-connect-se/account-management.png)

    左側のウィンドウに、選択した Azure Stack Hub サブスクリプションに関連付けられているストレージ アカウントが表示されます。

    ![関連付けられているサブスクリプションの一覧](./media/azure-stack-storage-connect-se/list-of-associated-subscriptions.png)

## <a name="connect-to-an-azure-stack-hub-storage-account"></a>Azure Stack Hub ストレージ アカウントに接続する

ストレージ アカウント名とキーのペアを使用して Azure Stack Hub ストレージ アカウントに接続することもできます。

1. Storage Explorer の左側のウィンドウで、[アカウントの管理] を選択します。 サインインしたすべての Microsoft アカウントが表示されます。

    ![アカウントの追加 - Storage Explorer](./media/azure-stack-storage-connect-se/azure-stack-sub-add-an-account.png)

2. Azure Stack Hub サブスクリプションに接続するには、 **[アカウントの追加]** を選択します。

    ![アカウントの追加 - Azure Storage へ接続](./media/azure-stack-storage-connect-se/azure-stack-use-a-storage-and-key.png)

3. [Azure Storage へ接続] ダイアログ ボックスで、 **[Use a storage account name and key]\(ストレージ アカウント名とキーを使用\)** を選択します。

4. ご使用のアカウント名を **[アカウント名]** に入力し、アカウント キーを **[アカウント キー]** テキスト ボックスに貼り付けます。 次に、 **[ストレージ エンドポイントのドメイン]** で **[Other (enter below)]\(その他 (以下に入力)\)** を選択し、Azure Stack Hub エンドポイントを入力します。

    Azure Stack Hub のエンドポイントには、リージョン名と Azure Stack Hub ドメインの 2 つの部分があります。 Azure Stack Development Kit の既定のエンドポイントは **local.azurestack.external** です。 エンドポイントが不明の場合は、クラウド管理者に問い合わせてください。

    ![名前とキーを使用して接続する](./media/azure-stack-storage-connect-se/azure-stack-attach-name-and-key.png)

5. **[接続]** を選択します。
6. ストレージ アカウントが正常に接続されると、そのストレージ アカウントが、名前の後に "**External, Other (外部、その他)** " を伴って表示されます。

    ![VMWINDISK](./media/azure-stack-storage-connect-se/azure-stack-vmwindisk.png)

## <a name="next-steps"></a>次のステップ

* [Storage Explorer の概要](/azure/vs-azure-tools-storage-manage-with-storage-explorer)
* [Azure Stack Hub ストレージ: 違いと考慮事項](azure-stack-acs-differences.md)
* Azure Storage の詳細については、「[Microsoft Azure Storage の概要](/azure/storage/common/storage-introduction)」を参照してください。
