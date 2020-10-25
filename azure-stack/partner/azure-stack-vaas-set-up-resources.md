---
title: VaaS 向けの Azure AD とストレージ リソースの設定
titleSuffix: Azure Stack Hub
description: Azure Stack Hub のサービスとしての検証用に Azure AD とストレージ リソースを設定する方法について説明します。
author: mattbriggs
ms.topic: tutorial
ms.date: 5/27/2020
ms.author: mabrigg
ms.reviewer: johnhas
ms.lastreviewed: 11/26/2018
ROBOTS: NOINDEX
ms.openlocfilehash: 1076dc69190b79728d28be99c7b84629e80b5709
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86490169"
---
# <a name="tutorial-set-up-resources-for-validation-as-a-service"></a>チュートリアル:サービスとしての検証のためのリソースを設定する

[!INCLUDE [Azure_Stack_Partner](./includes/azure-stack-partner-appliesto.md)]

サービスとしての検証 (VaaS) は、マーケットで Azure Stack Hub ソリューションを検証およびサポートするために使用される Azure サービスです。 ソリューションを検証するサービスを使用する前に、この記事に従ってください。

このチュートリアルでは、以下の内容を学習します。

> [!div class="checklist"]
> * Azure Active Directory (AD) を設定して VaaS を使用する準備をします。
> * ストレージ アカウントを作成します。

## <a name="configure-an-azure-ad-tenant"></a>Azure AD テナントの構成

Azure AD テナントは、組織を登録し、VaaS を使用するユーザーを認証します。 パートナーは、テナントのロールベースのアクセス制御 (RBAC) 機能を使って、パートナーの組織内でだれが VaaS を使用できるかを管理します。 詳細については、「[Azure Active Directory とは](/azure/active-directory/fundamentals/active-directory-whatis)」を参照してください。

### <a name="create-a-tenant"></a>テナントの作成

組織が VaaS サービスへのアクセスに使用するテナントを作成します。 わかりやすい名前 (たとえば、`ContosoVaaS@onmicrosoft.com`) を付けます。

1. [Azure portal](https://portal.azure.com) で Azure AD テナントを作成するか、既存のテナントを使用します。 <!-- For instructions on creating new Azure AD tenants, see [Get started with Azure AD](/azure/active-directory/get-started-azure-ad). -->

2. 組織のメンバーをテナントに追加します。 これらのユーザーは、サービスを使用してテストを表示またはスケジュール設定する責任を負います。 登録が完了したら、ユーザーのアクセス レベルを定義します。

    次のいずれかのロールを割り当てることで、VaaS でアクションを実行する権限をテナント内のユーザーに付与します。

    | ロール名 | 説明 |
    |---------------------|------------------------------------------|
    | 所有者 | すべてのリソースへのフル アクセス権が与えられます。 |
    | Reader | すべてのリソースを表示できますが、作成および管理することはできません。 |
    | Test Contributor | テスト リソースを作成および管理できます。 |

    **Azure Stack Hub 検証サービス** アプリでロールを割り当てるには:

   1. [Azure portal](https://portal.azure.com) にサインインします。
   2. **[すべてのサービス]**  >  **[ID]** セクションの下の **[Azure Active Directory]** を選択します。
   3. **[エンタープライズ アプリケーション]**  >  **[Azure Stack Hub Validation Service]\(Azure Stack Hub 検証サービス\)** アプリを選択します。
   4. **[ユーザーとグループ]** を選択します。 **[Azure Stack Hub Validation Service - Users and group]\(Azure Stack Hub 検証サービス - ユーザーとグループ\)** ブレードに、アプリの使用を許可されたユーザーが一覧表示されます。
   5. **+ [ユーザーの追加]** を選択し、テナントからユーザーを追加してロールを割り当てます。

      組織内の異なるグループ間で VaaS リソースとアクションを分離したい場合は、複数の Azure AD テナント ディレクトリを作成できます。

### <a name="register-your-tenant"></a>テナントの登録

このプロセスでは、**Azure Stack Hub Validation Service** Azure AD アプリケーションを使用してテナントを承認します。

1. テナントについて次の情報を Microsoft ([vaashelp@microsoft.com](mailto:vaashelp@microsoft.com)) に送信します。

    | Data | 説明 |
    |--------------------------------|---------------------------------------------------------------------------------------------|
    | 組織名 | 組織の正式名称。 |
    | Azure AD テナント ディレクトリ名 | 登録する Azure AD テナント ディレクトリ名。 |
    | Azure AD テナント ディレクトリ ID | Azure AD テナント ディレクトリに関連付けられている GUID。 Azure AD テナント ディレクトリ ID の確認方法については、「[Get tenant ID (テナント ID を取得する)](/azure/azure-resource-manager/resource-group-create-service-principal-portal#get-values-for-signing-in)」を参照してください。 |

2. ご利用のテナントで Azure Stack Hub 検証ポータルを使用できるかどうかを Azure Stack Hub 検証チームがチェックし、確認するのを待ちます。

### <a name="consent-to-the-vaas-app"></a>VaaS アプリへの同意

Azure AD 管理者として、テナントのために必要なアクセス許可を VaaS Azure AD アプリケーションに付与します。

1. テナントの全体管理者の資格情報を使って [Azure Stack Hub 検証ポータル](https://azurestackvalidation.com/)にサインインします。

2. **[マイ アカウント]** を選択します。

3 表示された Azure AD アクセス許可を VaaS に付与するように求められたら、条件に同意して続行します。

## <a name="create-an-azure-storage-account"></a>Azure Storage アカウントの作成

テストの実行中、VaaS によって、Azure ストレージ アカウントに診断ログが出力されます。 テスト ログに加えて、ストレージ アカウントは、パッケージ検証ワークフローの OEM 拡張機能パッケージをアップロードするために使用することもできます。

Azure Storage アカウントは、お客様の Azure Stack Hub 環境ではなく Azure パブリック クラウドでホストされます。

1. Azure portal で、 **[すべてのサービス]**  >  **[ストレージ]**  >  **[ストレージ アカウント]** の順に選択します。 **[ストレージ アカウント]** ブレードで、 **[追加]** を選択します。

2. ストレージ アカウントを作成するサブスクリプションを選択します。

3. **[リソース グループ]** で、 **[新規作成]** を選択します。 新しいリソース グループの名前を入力します。

4. Azure Storage アカウントの [[名前付け規則]](/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging#storage) をレビューします。 ストレージ アカウントの名前を入力します。

5. ストレージ アカウントのリージョンとして **[米国西部]** を選択します。

    ログの格納に対してネットワーク料金が発生しないように、 **[米国西部]** リージョンのみを使用するように Azure Storage アカウントを構成できます。 データのレプリケーションとホット ストレージ層機能は、このデータには必要ありません。 いずれかの機能を有効にすると、コストが大幅に増加します。

6. **[アカウントの種類]** 以外の設定は既定値のままにしておきます。

    - **[デプロイ モデル]** フィールドは、既定では **[Resource Manager]** に設定されています。
    - **[パフォーマンス]** フィールドは、既定では **[Standard]** に設定されています。
    - **[アカウントの種類]** フィールドで **[BLOB ストレージ]** を選択します。
    - **[レプリケーション]** フィールドは、既定では **[ローカル冗長ストレージ (LRS)]** に設定されています。
    - **[アクセス レベル]** は、既定では **[ホット]** に設定されています。

7. **[確認および作成]** を選択して、ストレージ アカウントの設定を確認し、アカウントを作成します。

## <a name="next-steps"></a>次のステップ

お客様の環境で受信接続が許可されていない場合は、ローカル エージェントのデプロイに関するチュートリアルに従ってご使用のハードウェアでテストを実行します。

> [!div class="nextstepaction"]
> [ローカル エージェントをデプロイする](azure-stack-vaas-local-agent.md)
