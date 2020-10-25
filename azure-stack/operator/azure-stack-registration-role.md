---
title: Azure Stack Hub 登録用のカスタム ロールを作成する
titleSuffix: Azure Stack Hub
description: カスタム ロールを作成して Azure Stack Hub の登録にグローバル管理者の使用を回避する方法について説明します。
author: BryanLa
ms.topic: how-to
ms.date: 08/05/2020
ms.author: bryanla
ms.reviewer: rtiberiu
ms.lastreviewed: 06/10/2019
ms.openlocfilehash: cb2da0dc57069647ecefd8ccc278526f7f35f57d
ms.sourcegitcommit: b69c8334571094721b26e6bdebd639f4fd294dd0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/06/2020
ms.locfileid: "87839234"
---
# <a name="create-a-custom-role-for-azure-stack-hub-registration"></a>Azure Stack Hub 登録用のカスタム ロールを作成する

> [!WARNING]
> これは、セキュリティ体制の機能ではありません。 Azure サブスクリプションが誤って変更されないようにする制約が必要なシナリオでは、これを使用します。 ユーザーがこのカスタム ロールに対する権限を委任されている場合、そのユーザーはアクセス許可の編集と権利の昇格の権限を持ちます。 カスタム ロールには信頼できるユーザーのみを割り当ててください。

Azure Stack Hub の登録時には、Azure Active Directory (Azure AD) アカウントでサインインする必要があります。 アカウントには、次の Azure AD のアクセス許可と Azure サブスクリプションのアクセス許可が必要です。

* **Azure AD テナントでのアプリの登録のアクセス許可:** 管理者がアプリの登録のアクセス許可を持っています。 ユーザーのアクセス許可は、テナントのすべてのユーザーに対するグローバル設定です。 この設定を確認または変更するには、「[リソースにアクセスできる Azure AD アプリとサービス プリンシパルをポータルで作成する](/azure/active-directory/develop/howto-create-service-principal-portal#required-permissions)」を参照してください。

    "*ユーザーはアプリケーションを登録できる*" の設定は、ユーザー アカウントで Azure Stack Hub を登録できるようにするため、 **[はい]** に設定する必要があります。 アプリ登録設定が **[いいえ]** に設定されている場合は、Azure Stack Hub の登録にユーザー アカウントを使用することはできません。グローバル管理者アカウントを使用する必要があります。

* **Azure サブスクリプションの十分なアクセス許可のセット:** 所有者ロールに属しているユーザーには、十分なアクセス許可があります。 他のアカウントの場合は、次のセクションで説明するようにカスタム ロールを割り当てることで、アクセス許可セットを割り当てることができます。

Azure サブスクリプションの所有者アクセス許可を持つアカウントを使う代わりに、カスタム ロールを作成して、特権が低いユーザー アカウントにアクセス許可を割り当てることができます。 その後、このアカウントを使用して、Azure Stack Hub を登録できます。

## <a name="create-a-custom-role-using-powershell"></a>PowerShell を使用してカスタム ロールを作成する

カスタム ロールを作成するには、[所有者](/azure/role-based-access-control/built-in-roles#owner)や[ユーザー アクセス管理者](/azure/role-based-access-control/built-in-roles#user-access-administrator)など、すべての `AssignableScopes` に対する `Microsoft.Authorization/roleDefinitions/write` アクセス許可が必要になります。 カスタム ロールの作成を簡素化するには、次の JSON テンプレートを使います。 このテンプレートでは、Azure Stack Hub の登録に必要な読み取りアクセスおよび書き込みアクセスを許可するカスタム ロールを作成します。

1. JSON ファイルを作成します。 (例: `C:\CustomRoles\registrationrole.json`)。
2. 次の JSON をファイルに追加します。 `<SubscriptionID>` は、Azure サブスクリプション ID に置き換えてください。

    ```json
    {
      "Name": "Azure Stack Hub registration role",
      "Id": null,
      "IsCustom": true,
      "Description": "Allows access to register Azure Stack Hub",
      "Actions": [
        "Microsoft.Resources/subscriptions/resourceGroups/write",
        "Microsoft.Resources/subscriptions/resourceGroups/read",
        "Microsoft.AzureStack/registrations/*",
        "Microsoft.AzureStack/register/action",
        "Microsoft.Authorization/roleAssignments/read",
        "Microsoft.Authorization/roleAssignments/write",
        "Microsoft.Authorization/roleAssignments/delete",
        "Microsoft.Authorization/permissions/read",
        "Microsoft.Authorization/locks/read",
        "Microsoft.Authorization/locks/write"
      ],
      "NotActions": [
      ],
      "AssignableScopes": [
        "/subscriptions/<SubscriptionID>"
      ]
    }
    ```

3. PowerShell で、Azure Resource Manager を使用するために Azure に接続します。 メッセージが表示されたら、[所有者](/azure/role-based-access-control/built-in-roles#owner)または[ユーザー アクセス管理者](/azure/role-based-access-control/built-in-roles#user-access-administrator)などの十分なアクセス許可を持つアカウントを使用して認証します。

    ```azurepowershell
    Connect-AzureRmAccount
    ```

4. カスタム ロールを作成するには、**New-AzureRmRoleDefinition** を使用して JSON テンプレート ファイルを指定します。

    ``` azurepowershell
    New-AzureRmRoleDefinition -InputFile "C:\CustomRoles\registrationrole.json"
    ```

## <a name="assign-a-user-to-registration-role"></a>ユーザーを登録ロールに割り当てる

登録カスタム ロールを作成した後、Azure Stack Hub の登録に使用するユーザー アカウントにそのロールを割り当てます。

1. 権限を委任するのに十分な Azure サブスクリプションのアクセス許可 を持つアカウント ([所有者](/azure/role-based-access-control/built-in-roles#owner)や[ユーザー アクセス管理者](/azure/role-based-access-control/built-in-roles#user-access-administrator)など) でサインインします。
2. **[サブスクリプション]** で、 **[アクセス制御 (IAM)] > [ロール割り当ての追加]** の順に選択します。
3. **[ロール]** で、作成したカスタム ロール "*Azure Stack Hub 登録ロール*"。
4. ロールに割り当てるユーザーを選択します。
5. **[保存]** を選択して、選択したユーザーをロールに割り当てます。

    ![Azure portal でカスタム ロールに割り当てるユーザーを選択します。](media/azure-stack-registration-role/assign-role.png)

カスタム ロールの使用の詳細については、「[RBAC と Azure portal を使用してアクセスを管理する](/azure/role-based-access-control/role-assignments-portal)」を参照してください。

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub を Azure に登録する](azure-stack-registration.md)
