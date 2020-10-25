---
title: ポータルを使って Azure Stack Hub の Key Vault を管理する
description: Azure Stack Hub ポータルを使用して Azure Stack Hub の Key Vault を管理する方法について説明します。
author: sethmanheim
ms.topic: article
ms.date: 06/09/2020
ms.author: sethm
ms.lastreviewed: 1/10/2020
ms.openlocfilehash: 5413c37b0574e022716a1a0d333c18e78a818937
ms.sourcegitcommit: d91e47a51a02042f700c6a420f526f511a6db9a0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/10/2020
ms.locfileid: "84666390"
---
# <a name="manage-key-vault-in-azure-stack-hub-using-the-portal"></a>ポータルを使って Azure Stack Hub の Key Vault を管理する

この記事では、Azure Stack Hub ポータルを使用して Azure Stack Hub でキー コンテナーを作成および管理する方法について説明します。

## <a name="prerequisites"></a>前提条件

Azure Key Vault サービスを含むプランをサブスクライブする必要があります。

## <a name="create-a-key-vault"></a>Key Vault を作成します

1. ユーザー ポータル `https://portal.local.azurestack.external` にサインインします。

2. ダッシュボードから、 **[+ リソースの作成]** 、 **[セキュリティ + ID]** 、 **[Key Vault]** の順に選択します。

    ![Key Vault 画面](media/azure-stack-key-vault-manage-portal/image1.png)

3. **[Key Vault の作成]** ウィンドウで、Vault に**名前**を割り当てます。 Vault の名前には、英数字とハイフン (-) 文字のみを含めることができます。 これらを数字で始めることはできません。

4. 使用可能なサブスクリプションの一覧から**サブスクリプション**を選択します。 Key Vault サービスを提供するすべてのサブスクリプションがドロップダウン リストに表示されます。

5. 既存の**リソース グループ**を選択するか、新しいものを作成します。

6. **[価格レベル]** を選択します。 Azure Stack Development Kit (ASDK) では、キー コンテナーは **Standard** SKU のみをサポートします。

7. 既存の**アクセス ポリシー**のいずれかを選択するか、新しいアクセス ポリシーを作成します。 アクセス ポリシーでは、この Vault で操作を実行する権限を、ユーザー、アプリ、またはセキュリティ グループに付与できます。

8. 必要に応じて、 **[高度なアクセス ポリシー]** を選んで機能へのアクセスを有効にします。 例: デプロイのための仮想マシン (VM)、テンプレートのデプロイのための Resource Manager、ボリューム暗号化のための Azure Disk Encryption へのアクセス。

9. 設定を構成したら、 **[OK]** を選択し、 **[作成]** を選択します。 この手順で、キー コンテナーのデプロイが開始されます。

## <a name="manage-keys-and-secrets"></a>キーとシークレットの管理

キー コンテナーを作成した後、次の手順を実行し、キー コンテナー内でキーとシークレットを作成および管理します。

### <a name="create-a-key"></a>キーの作成

1. Azure Stack Hub のユーザー ポータル `https://portal.local.azurestack.external` にサインインします。

2. ダッシュボードで **[すべてのリソース]** をクリックし、先ほど作成した Key Vault を選択して、 **[キー]** タイルを選択します。

3. **[キー]** ウィンドウで **[生成/インポート]** を選択します。

4. **[キーの作成]** ウィンドウで、 **[オプション]** 一覧から、キーの作成に使用する方法を選択します。 新しいキーを**生成**したり、既存のキーを**アップロード**したり、キーのバックアップを選択して**バックアップを復元**したりできます。

5. キーの**名前**を入力します。 キーの名前には、英数字とハイフン (-) 文字のみを含めることができます。

6. 必要に応じて、キーの **[アクティブ化する日を設定する]** と **[有効期限を設定する]** の値を構成します。

7. **[作成]** を選択してデプロイを開始します。

キーが正常に作成されたら、 **[キー]** で選択してプロパティを表示したり変更したりできます。 プロパティ セクションには **[キー識別子]** が含まれます。これは、外部アプリがこのキーにアクセスするために使う Uniform Resource Identifier (URI) です。 このキーでの操作を制限するには、 **[許可された操作]** で設定を構成します。

![キーの URI](media/azure-stack-key-vault-manage-portal/image4.png)

### <a name="create-a-secret"></a>シークレットの作成

1. ユーザー ポータル `https://portal.local.azurestack.external` にサインインします。

2. ダッシュボードで **[すべてのリソース]** を選択し、先ほど作成した Key Vault を選択して、 **[シークレット]** タイルを選択します。

3. **[シークレット]** で **[追加]** を選択します。

4. **[シークレットの作成]** の下で、 **[アップロード オプション]** 一覧から、シークレットの作成に使用するオプションを選択します。 シークレットの値を入力して**手動で**シークレットを作成したり、お使いのローカル コンピューターから**証明書**をアップロードしてシークレットを作成したりできます。

5. シークレットの**名前**を入力します。 シークレットの名前には、英数字とハイフン (-) 文字のみを含めることができます。

6. 必要に応じて、**コンテンツの種類**を指定し、シークレットの **[アクティブ化する日を設定する]** と **[有効期限を設定する]** の値を構成します。

7. **[作成]** を選択してデプロイを開始します。

シークレットが正常に作成されたら、 **[シークレット]** で選択してプロパティを表示したり変更したりできます。 **[シークレット識別子]** は、外部アプリがこのシークレットへのアクセスに使用できる URI です。

![シークレットの URI](media/azure-stack-key-vault-manage-portal/image5.png)

## <a name="next-steps"></a>次のステップ

* [Key Vault に格納されているパスワードを取得して VM をデプロイする](azure-stack-key-vault-deploy-vm-with-secret.md)
* [キー コンテナーに格納されている証明書を使用した VM のデプロイ](azure-stack-key-vault-push-secret-into-vm.md)
