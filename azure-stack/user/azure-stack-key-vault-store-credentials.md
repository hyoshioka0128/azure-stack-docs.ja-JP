---
title: Azure Stack Hub でサービス プリンシパルの資格情報を Key Vault に格納する
description: Azure Stack Hub でサービス プリンシパルの資格情報が Key Vault にどのように格納されるかについて説明します
author: sethmanheim
ms.topic: article
ms.date: 06/09/2020
ms.author: sethm
ms.lastreviewed: 01/16/2020
ms.openlocfilehash: c17ba69fedfd51681c5038efde591115ce54bddf
ms.sourcegitcommit: 0aa5f7f20690839661c8bb3bfdbe32f82bec0c64
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/21/2020
ms.locfileid: "86567520"
---
# <a name="store-service-principal-credentials-in-azure-stack-hub-key-vault"></a>Azure Stack Hub でサービス プリンシパルの資格情報を Key Vault に格納する

通常、Azure Stack Hub におけるアプリの開発では、デプロイ前にサービス プリンシパルを作成し、その資格情報を使用して認証を行う必要があります。 しかし、保存したサービス プリンシパルの資格情報を失ってしまう場合があります。 この記事では、サービス プリンシパルを作成し、その値を後で取り出せるよう Azure Key Vault に格納する方法について説明します。

Key Vault の詳細については、「[Azure Stack Hub での Key Vault の概要](azure-stack-key-vault-intro.md)」を参照してください。

## <a name="prerequisites"></a>前提条件

- Azure Key Vault サービスを含むオファーをサブスクライブします。
- PowerShell をインストールし、Azure Stack Hub で使うように構成します。

## <a name="key-vault-in-azure-stack-hub"></a>Azure Stack Hub の Key Vault

Azure Stack Hub の Key Vault は、クラウド アプリやクラウド サービスで使用される暗号化キーとシークレットを保護するのに役立ちます。 Key Vault を使用することにより、キーとシークレットを暗号化することができます。

キー コンテナーを作成するには、次の手順に従います。

1. Azure Stack Hub ポータルにサインインします。

2. ダッシュボードから、 **[+ リソースの作成]** 、 **[セキュリティ + ID]** 、 **[Key Vault]** の順に選択します。

   ![キー コンテナーの作成](media/azure-stack-key-vault-store-credentials/create-key-vault.png)

3. **[Key Vault の作成]** ウィンドウで、Vault に**名前**を割り当てます。 Vault の名前には、英数字とハイフン (-) 文字のみを含めることができます。 名前を数字で始めることはできません。

4. 使用可能なサブスクリプションの一覧から選択します。

5. 既存のリソース グループを選択するか、新しいものを作成します。

6. [価格レベル] を選択します。

7. 既存のアクセス ポリシーのいずれかを選択するか、新しいアクセス ポリシーを作成します。 この Vault に対して操作を実行するための権限を、アクセス ポリシーを通じて、ユーザー、アプリケーション、またはセキュリティ グループに付与できます。

8. 必要に応じて、[高度なアクセス ポリシー] を選んで機能へのアクセスを有効にします。

9. 設定を構成したら、 **[OK]** を選択し、 **[作成]** を選択します。 キー コンテナーのデプロイが開始されます。

## <a name="create-a-service-principal"></a>サービス プリンシパルの作成

1. Azure portal で Azure アカウントにサインインします。

2. **[Azure Active Directory]** 、 **[アプリの登録]** 、 **[追加]** の順に選択します。

3. アプリの名前と URL を指定します。 作成するアプリの種類として、 **[Web アプリ/API]** または **[ネイティブ]** を選択します。 値を設定したら、 **[作成]** をクリックします。

4. **[Active Directory]** 、 **[アプリの登録]** を順に選択し、目的のアプリケーションを選択します。

5. **アプリケーション ID** をコピーし、アプリ コードに保存します。 サンプル アプリでは、**アプリケーション ID** を参照するときに**クライアント ID** を使用します。

6. 認証キーを生成するには、 **[キー]** を選択します。

7. キーの説明を入力し、期間を指定します。

8. **[保存]** を選択します。

9. **[保存]** をクリックした後、利用可能になった**キー**をコピーします。

## <a name="store-the-service-principal-inside-key-vault"></a>Key Vault 内にサービス プリンシパルを格納する

1. Azure Stack Hub のユーザー ポータルにサインインした後、先ほど作成したキー コンテナーを選択して、 **[シークレット]** タイルを選択します。

2. **[シークレット]** ウィンドウで **[生成/インポート]** を選択します。

3. **[シークレットの作成]** ウィンドウで、オプションの一覧から **[手動]** を選択します。 証明書を使用してサービス プリンシパルを作成した場合、ドロップダウン リストから証明書を選択し、ファイルをアップロードします。

4. キーの名前として、サービス プリンシパルからコピーした**アプリケーション ID** を入力します。 キーの名前には、英数字とハイフン (-) 文字のみを含めることができます。

5. サービス プリンシパルから **[値]** タブにキーの値を貼り付けます。

6. **[コンテンツの種類]** に **[サービス プリンシパル]** を選択します。

7. キーの **[アクティブ化した日]** と **[有効期限]** の値を設定します。

8. **[作成]** を選択してデプロイを開始します。

シークレットが正常に作成された後、そこにサービス プリンシパルの情報が格納されます。 いつでも **[シークレット]** からそれを選択し、そのプロパティを表示したり変更を加えたりすることができます。 **[プロパティ]** セクションにはシークレット識別子が含まれます。これは、外部アプリがこのシークレットにアクセスするために使う Uniform Resource Identifier (URI) です。

## <a name="next-steps"></a>次のステップ

- [サービス プリンシパルの使用](../operator/azure-stack-create-service-principals.md?view=azs-2002)
- [ポータルを使用して Azure Stack Hub の Key Vault を管理する](azure-stack-key-vault-manage-portal.md)  
- [PowerShell を使用して Azure Stack Hub の Key Vault を管理する](azure-stack-key-vault-manage-powershell.md)
