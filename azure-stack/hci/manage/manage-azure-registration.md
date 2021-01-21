---
title: Azure Stack HCI の Azure 登録を管理する
description: PowerShell を使用して、Azure Stack HCI の Azure 登録を管理し、登録状態を把握する方法。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.date: 12/10/2020
ms.openlocfilehash: a81a1973d7324371cb42b23ca7905d39492401cf
ms.sourcegitcommit: 9b0e1264ef006d2009bb549f21010c672c49b9de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98254434"
---
# <a name="manage-azure-registration"></a>Azure 登録の管理

> Azure Stack HCI v20H2 に適用されます

Azure Stack HCI クラスターを作成したら、[クラスターを Azure Arc](../deploy/register-with-azure.md) に登録する必要があります。クラスターが登録されると、オンプレミス クラスターとクラウドの間で定期的に情報が同期されます。 このトピックでは、登録状態を把握し、Azure Active Directory のアクセス許可を付与し、使用停止の準備ができたらクラスターの登録を解除する方法について説明します。

## <a name="understanding-registration-status"></a>登録状態の把握

登録状態を把握するには、`Get-AzureStackHCI` PowerShell コマンドレットと、`ClusterStatus`、`RegistrationStatus`、および `ConnectionStatus` の各プロパティを使用します。 たとえば、Azure Stack HCI オペレーティング システムをインストールした後、クラスターの作成または参加を行う前は、`ClusterStatus` プロパティには "not yet" (未完了) の状態が表示されます。

:::image type="content" source="media/manage-azure-registration/1-get-azurestackhci.png" alt-text="クラスターを作成する前の Azure の登録状態":::

クラスターが作成されると、`RegistrationStatus` にのみ "not yet" (未完了) の状態が表示されます。

:::image type="content" source="media/manage-azure-registration/2-get-azurestackhci.png" alt-text="クラスターを作成した後の Azure の登録状態":::

Azure Stack HCI では、Azure のオンライン サービス条件に従って、インストールから 30 日以内に登録する必要があります。 30 日間の経過後にクラスター化されていない場合、`ClusterStatus` には `OutOfPolicy` が表示され、30 日間の経過後に登録されていない場合、`RegistrationStatus` には `OutOfPolicy` が表示されます。

クラスターが登録されると、`ConnectionStatus` と `LastConnected` の時刻が表示されます。クラスターがインターネットから一時的に切断されていなければ、通常、これは最終日の値です。 Azure Stack HCI クラスターは、最大 30 日間、完全にオフラインで動作できます。

:::image type="content" source="media/manage-azure-registration/3-get-azurestackhci.png" alt-text="登録後の Azure の登録状態":::

その最大期間を超えた場合、`ConnectionStatus` には `OutOfPolicy` が表示されます。

## <a name="azure-active-directory-app-permissions"></a>Azure Active Directory アプリのアクセス許可

サブスクリプションで Azure リソースを作成することに加えて、Azure Stack HCI を登録すると、概念的にはユーザーに似たアプリ ID が Azure Active Directory テナントに作成されます。 アプリ ID にはクラスター名が継承されます。 この ID は、サブスクリプション内で、Azure Stack HCI クラウド サービスに代わって適切に機能します。

`Register-AzureStackHCI` を実行するユーザーが Azure Active Directory 管理者であるか、十分なアクセス許可が委任されている場合、これはすべて自動的に行われるため、追加の操作は必要ありません。 そうでない場合は、登録を完了するには、Azure Active Directory 管理者の承認が必要になる場合があります。 管理者は、アプリに明示的に同意するか、アプリに同意できるアクセス許可を委任することができます。

:::image type="content" source="media/manage-azure-registration/aad-permissions.png" alt-text="Azure Active Directory のアクセス許可と ID の図" border="false":::

同意するには、[portal.azure.com](https://portal.azure.com) を開き、Azure Active Directory に対して十分なアクセス許可を持つ Azure アカウントでサインインします。 **Azure Active Directory** に移動し、 **[アプリの登録]** を選択します。 クラスターにちなんで名付けられたアプリ ID を選択し、 **[API のアクセス許可]** に移動します。

Azure Stack HCI の一般提供 (GA) リリースの場合、アプリには次のアクセス許可が必要です。これは、パブリック プレビューの場合に必要なアプリのアクセス許可とは異なります。

```http
https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.Cluster.Read

https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.Cluster.ReadWrite

https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.ClusterNode.Read

https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.ClusterNode.ReadWrite
```

パブリック プレビューの場合、アプリのアクセス許可は次の内容でした (現在、これらは非推奨です)。

```http
https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.Census.Sync

https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.Billing.Sync
```

Azure Active Directory 管理者から承認を受けるには時間がかかる場合があるため、`Register-AzureStackHCI` コマンドレットは終了し、登録は "pending admin consent" (管理者の同意待ち)、つまり部分的に完了した状態のままになります。 同意が得られたら、`Register-AzureStackHCI` をもう一度実行するだけで登録は完了します。

## <a name="azure-active-directory-user-permissions"></a>Azure Active Directory ユーザーのアクセス許可

Register-AzStackHCI を実行するユーザーには、以下のために Azure AD のアクセス許可が必要です。

- Azure AD アプリケーションの作成、取得、設定、削除 (New/Get/Set/Remove-AzureADApplication)
- Azure AD サービス プリンシパルの作成、取得 (New/Get-New-AzureADServicePrincipal)
- AD アプリケーション シークレットの管理 (New/Get/Remove-AzureADApplicationKeyCredential)
- 特定のアプリケーションのアクセス許可を使用する同意の付与 (New/Get/Remove AzureADServiceAppRoleAssignments)

これを実現するには、次の 3 つの方法があります。

### <a name="option-1-allow-any-user-to-register-applications"></a>オプション 1: すべてのユーザーにアプリケーションの登録を許可する

Azure Active Directory で、 **[ユーザー設定] > [アプリの登録]** に移動します。 **[ユーザーはアプリケーションを登録できる]** で **[はい]** を選択します。

これにより、すべてのユーザーはアプリケーションを登録できるようになります。 ただし、ユーザーは、クラスターの登録時に、Azure AD 管理者に同意を付与してもらう必要があります。 これはテナント レベルの設定であるため、大企業のお客様には適さない場合があることに注意してください。

### <a name="option-2-assign-cloud-application-administration-role"></a>オプション 2:Cloud Application Administration ロールを割り当てる

組み込みの "Cloud Application Administration" Azure AD ロールをユーザーに割り当てます。 こうすることで、ユーザーは追加の AD 管理者の同意なしにクラスターを登録できます。

### <a name="option-3-create-a-custom-ad-role-and-consent-policy"></a>オプション 3:カスタムの AD ロールと同意ポリシーを作成する

最も制限の厳しいオプションは、Azure Stack HCI サービスに必要なアクセス許可について、テナント全体の管理者の同意を委任するカスタムの同意ポリシーを使用してカスタムの AD ロールを作成することです。 このカスタム ロールが割り当てられると、ユーザーは追加の AD 管理者の同意を必要とすることなく、登録と同意の付与の両方を行うことができます。

   > [!NOTE]
   > このオプションには Azure AD Premium ライセンスが必要であり、現在はパブリック プレビュー段階にあるカスタムの AD ロールとカスタムの同意ポリシー機能が使用されます。

   1. Azure AD に接続します。
   
      ```powershell
      Connect-AzureAD
      ```

   2. カスタムの同意ポリシーを作成します。

      ```powershell
      New-AzureADMSPermissionGrantPolicy -Id "AzSHCI-registration-consent-policy" -DisplayName "Azure Stack HCI registration admin app consent policy" -Description "Azure Stack HCI registration admin app consent policy"
      ```

   3. アプリ ID が 1322e676-dee7-41ee-a874-ac923822781c である Azure Stack HCI サービスに必要なアプリのアクセス許可を含む条件を追加します。 次のアクセス許可は Azure Stack HCI の GA リリース用です。クラスター内のすべてのサーバーに [2020 年 11 月 23 日のプレビュー更新プログラム (KB4586852)](https://support.microsoft.com/help/4595086/azure-stack-hci-release-notes-overview) を適用し、Az.StackHCI モジュール バージョン 0.4.1 以降をダウンロードしていない限り、パブリック プレビューでは機能しません。
   
      ```powershell
      New-AzureADMSPermissionGrantConditionSet -PolicyId "AzSHCI-registration-consent-policy" -ConditionSetType "includes" -PermissionType "application" -ResourceApplication "1322e676-dee7-41ee-a874-ac923822781c" -Permissions "bbe8afc9-f3ba-4955-bb5f-1cfb6960b242","8fa5445e-80fb-4c71-a3b1-9a16a81a1966","493bd689-9082-40db-a506-11f40b68128f","2344a320-6a09-4530-bed7-c90485b5e5e2"
      ```

   4. 手順 2 で作成したカスタムの同意ポリシーに注意して、Azure Stack HCI の登録を許可するアクセス許可を付与します。
   
      ```powershell
      $displayName = "Azure Stack HCI Registration Administrator "
      $description = "Custom AD role to allow registering Azure Stack HCI "
      $templateId = (New-Guid).Guid
      $allowedResourceAction =
      @(
             "microsoft.directory/applications/createAsOwner",
             "microsoft.directory/applications/delete",
             "microsoft.directory/applications/standard/read",
             "microsoft.directory/applications/credentials/update",
             "microsoft.directory/applications/permissions/update",
             "microsoft.directory/servicePrincipals/appRoleAssignedTo/update",
             "microsoft.directory/servicePrincipals/appRoleAssignedTo/read",
             "microsoft.directory/servicePrincipals/appRoleAssignments/read",
             "microsoft.directory/servicePrincipals/createAsOwner",
             "microsoft.directory/servicePrincipals/credentials/update",
             "microsoft.directory/servicePrincipals/permissions/update",
             "microsoft.directory/servicePrincipals/standard/read",
             "microsoft.directory/servicePrincipals/managePermissionGrantsForAll.AzSHCI-registration-consent-policy"
      )
      $rolePermissions = @{'allowedResourceActions'= $allowedResourceAction}
      ```

   5. 新しいカスタムの AD ロールを作成します。

      ```powershell
      $customADRole = New-AzureADMSRoleDefinition -RolePermissions $rolePermissions -DisplayName $displayName -Description $description -TemplateId $templateId -IsEnabled $true
      ```

   6. [これらの手順](/azure/active-directory/fundamentals/active-directory-users-assign-role-azure-portal?context=/azure/active-directory/roles/context/ugr-context)に従って、Azure Stack HCI クラスターを Azure に登録するユーザーに新しいカスタムの AD ロールを割り当てます。

## <a name="unregister-azure-stack-hci-with-azure"></a>Azure で Azure Stack HCI の登録を解除する

Azure Stack HCI クラスターの使用を停止する準備ができたら、`Unregister-AzStackHCI` コマンドレットを使用して登録を解除します。 これにより、Azure Arc による監視、サポート、課金のすべての機能が停止します。クラスターと Azure Active Directory アプリ ID を表す Azure リソースは削除されますが、リソース グループは削除されません。他の無関係なリソースが含まれている可能性があるためです。

クラスター ノードで `Unregister-AzStackHCI` コマンドレットを実行する場合、この構文を使用し、自分の Azure サブスクリプション ID と、登録を解除したい Azure Stack HCI クラスターのリソース名を指定します。

```PowerShell
Unregister-AzStackHCI -SubscriptionId "e569b8af-6ecc-47fd-a7d5-2ac7f23d8bfe" -ResourceName HCI001
```

別のデバイス (ご自分の PC や携帯電話など) で microsoft.com/devicelogin にアクセスするよう求められるので、コードを入力し、そこにサインインして Azure の認証を行います。

管理用 PC からコマンドレットを実行する場合、クラスター内のサーバーの名前も指定する必要があります。

```PowerShell
Unregister-AzStackHCI -ComputerName ClusterNode1 -SubscriptionId "e569b8af-6ecc-47fd-a7d5-2ac7f23d8bfe" -ResourceName HCI001
```

対話型の Azure ログイン ウィンドウがポップアップ表示されます。 表示される正確なプロンプトは、セキュリティ設定 (2 要素認証など) によって異なります。 画面の指示に従ってログインします。

## <a name="next-steps"></a>次のステップ

関連情報については、以下も参照してください。

- [Azure Stack HCI を Azure に接続する](../deploy/register-with-azure.md)
- [Azure Monitor を使用して Azure Stack HCI を監視する](azure-monitor.md)