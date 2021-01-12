---
title: Azure Stack HCI を Azure に接続する
description: Azure Stack HCI クラスターを Azure に登録する方法。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 12/28/2020
ms.openlocfilehash: b2576bed615d6a65a5e0c61e5e3ccbc2c052132b
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97872706"
---
# <a name="connect-azure-stack-hci-to-azure"></a>Azure Stack HCI を Azure に接続する

> 適用対象:Azure Stack HCI v20H2

Azure Stack HCI は Azure サービスとして提供され、Azure オンライン サービス条件に従ってインストールから 30 日以内に登録する必要があります。 このトピックでは、監視、サポート、課金、およびハイブリッド サービスのために、Azure Stack HCI クラスターを [Azure Arc](https://azure.microsoft.com/services/azure-arc/) に登録する方法について説明します。 登録時に、オンプレミスの個々の Azure Stack HCI クラスターを表す Azure Resource Manager リソースが作成されます。これにより、Azure 管理プレーンが Azure Stack HCI に効果的に拡張されます。 Azure リソースとオンプレミス クラスターの間で情報が定期的に同期されます。

   > [!IMPORTANT]
   > Azure への登録が必要です。登録がアクティブになるまでクラスターは完全にはサポートされません。 デプロイ時にクラスターを Azure に登録しなかった場合、またはクラスターは登録したが 30 日以上 Azure に接続しなかった場合は、新しい仮想マシン (VM) を作成または追加することはできません。 これが発生した場合は、VM を作成しようとすると、次のエラー メッセージが表示されます。
   >
   > *'vmname' の仮想マシン ロールを構成中にエラーが発生しました。ジョブが失敗しました。"Vmname" のクラスター化されたロールを開くときにエラーが発生しました。アクセスされるサービスには、特定の数の接続のライセンスが付与されています。サービスで受け入れることができる数の接続が既に存在するため、現時点ではサービスにこれ以上接続することはできません。*
   >
   > 解決するには、Azure への送信接続を許可し、このトピックの説明に従ってクラスターが登録されていることを確認します。

## <a name="prerequisites-for-registration"></a>登録の前提条件

Azure Stack HCI クラスターを作成しないと、Azure に登録することはできません。 クラスターをサポートするには、クラスター ノードが物理サーバーである必要があります。 仮想マシンはテストに使用できますが、Unified Extensible Firmware Interface (UEFI) をサポートしている必要があります。つまり、Hyper-V Generation 1 仮想マシンは使用できません。 Azure Arc の登録は Azure Stack HCI オペレーティング システムのネイティブ機能であるため、登録にエージェントは必要ありません。

### <a name="internet-access"></a>インターネットへのアクセス

Azure Stack HCI は、定期的に Azure パブリック クラウドに接続する必要があります。 送信接続が外部の企業ファイアウォールまたはプロキシ サーバーによって制限されている場合は、限られた数の既知の Azure IP でポート 443 (HTTPS) への送信アクセスを許可するように構成する必要があります。 ファイアウォールを準備する方法の詳細については、「[Azure Stack HCI 用にファイアウォールを構成する](../concepts/configure-firewalls.md)」を参照してください。

   > [!NOTE]
   > Az や AzureAD などの必要な PowerShell モジュールの最新バージョンがあることを確認するために、登録プロセスによって PowerShell ギャラリーへの接続が試行されます。 PowerShell ギャラリーは Azure 上でホストされていますが、現在はサービス タグがありません。 送信インターネット アクセスを利用できる管理マシンから上記のコマンドレットを実行できない場合は、モジュールをダウンロードし、`Register-AzStackHCI` コマンドを実行するクラスター ノードに手動で転送することをお勧めします。 または、[切断されたシナリオでモジュールをインストールする](/powershell/scripting/gallery/how-to/working-with-local-psrepositories?view=powershell-7.1#installing-powershellget-on-a-disconnected-system)こともできます。

### <a name="azure-subscription-and-permissions"></a>Azure のサブスクリプションとアクセス許可

まだ Azure アカウントを持っていない場合は[作成します](https://azure.microsoft.com/)。

以下のいずれかの種類の既存のサブスクリプションを使用できます。
- [学生](https://azure.microsoft.com/free/students/)または [Visual Studio サブスクライバー](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/)向けの Azure クレジットが付いた無料アカウント
- クレジット カードを使用した[従量課金制](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/)サブスクリプション
- マイクロソフトエンタープライズ契約 (EA) を通じて取得されたサブスクリプション
- クラウド ソリューション プロバイダー (CSP) プログラムを通じて取得されたサブスクリプション

クラスターを登録するユーザーには、次の Azure サブスクリプション アクセス許可が必要です。

- リソース プロバイダーを登録する
- Azure リソースとリソース グループの作成、取得、削除

EA または CSP を介した Azure サブスクリプションの場合、最も簡単な方法は、組み込みの "所有者" または "共同作成者" Azure ロールをサブスクリプションに割り当てるように、Azure サブスクリプション管理者に依頼することです。 ただし、管理者によっては、より制限の厳しいオプションを好む場合があります。 この場合は、次の手順に従って、Azure Stack HCI の登録専用のカスタム Azure ロールを作成できます。

1. 次の内容で **customHCIRole.json** という json ファイルを作成します。 必ず <subscriptionID> を Azure サブスクリプション ID に変更してください。 サブスクリプション ID を取得するには、[portal.azure.com](https://portal.azure.com) にアクセスし、[サブスクリプション] に移動して、一覧から ID をコピーして貼り付けます。

   ```json
   {
     "Name": "Azure Stack HCI registration role”,
     "Id": null,
     "IsCustom": true,
     "Description": "Custom Azure role to allow subscription-level access to register Azure Stack HCI",
     "Actions": [
       "Microsoft.Resources/subscriptions/resourceGroups/write",
       "Microsoft.Resources/subscriptions/resourceGroups/read",
       "Microsoft.Resources/subscriptions/resourceGroups/delete",
       "Microsoft.AzureStackHCI/register/action",
       "Microsoft.AzureStackHCI/Unregister/Action",
       "Microsoft.AzureStackHCI/clusters/*"
     ],
     "NotActions": [
     ],
   "AssignableScopes": [
       "/subscriptions/<subscriptionId>"
     ]
   }
   ```

2. カスタム ロールを作成します。

   ```powershell
   New-AzRoleDefinition -InputFile <path to customHCIRole.json>
   ```

3. ユーザーにカスタム ロールを割り当てます。

   ```powershell
   $user = get-AzAdUser -DisplayName <userdisplayname>
   $role = Get-AzRoleDefinition -Name "Azure Stack HCI registration role"
   New-AzRoleAssignment -ObjectId $user.Id -RoleDefinitionId $role.Id -Scope /subscriptions/<subscriptionid>
   ```

### <a name="azure-active-directory-permissions"></a>Azure Active Directory のアクセス許可

登録プロセスを完了するには、適切な Azure Active Directory のアクセス許可も必要です。 まだお持ちでない場合は、Azure AD 管理者に依頼して、アクセス許可の同意を付与するか、委任してもらいます。 詳しくは、「[Azure の登録を管理する](../manage/manage-azure-registration.md#azure-active-directory-app-permissions)」をご覧ください。

## <a name="register-using-powershell"></a>PowerShell を使用して登録する

管理 PC を使用して Azure Stack HCI クラスターを Azure に登録するには、次の手順を使用します。

1. 管理 PC に必要なコマンドレットをインストールします。 Azure Stack HCI の最新の一般提供 (GA) イメージからデプロイされたクラスターを登録するには、次のコマンドを実行するだけです。 パブリック プレビュー イメージからデプロイしたクラスターの場合、Azure への登録を試行する前に、必ずクラスター内の各サーバーに 2020 年 11 月 23 日のプレビュー更新プログラム (KB4586852) を適用しておきます。

   ```PowerShell
   Install-Module -Name Az.StackHCI
   ```

   > [!NOTE]
   > - "今すぐ PowerShellGet で NuGet プロバイダーをインストールしてインポートしますか?" などのプロンプトが表示される場合があります。 これには [はい] (Y) と答える必要があります。
   > - "'PSGallery' からモジュールをインストールしますか? " というメッセージが表示され、[はい] (Y) と答える必要があります。

2. クラスター内の任意のサーバーの名前を使用して登録を実行します。 Azure サブスクリプション ID を取得するには、[portal.azure.com](https://portal.azure.com) にアクセスし、[サブスクリプション] に移動して、一覧から ID をコピーして貼り付けます。

   ```PowerShell
   Register-AzStackHCI  -SubscriptionId "<subscription_ID>" -ComputerName Server1 [–Credential] [-ResourceName] [-ResourceGroupName] [-Region]
   ```

   この構文では、(Server1 がメンバーである) クラスターを現在のユーザーとして既定の Azure リージョンとクラウド環境に登録し、Azure リソースとリソース グループにスマートな既定の名前を使用します。ただし、このコマンドにパラメーターを追加して、必要に応じてこれらの値を指定することもできます。

   `Register-AzStackHCI` コマンドレットを実行するユーザーは [Azure Active Directory のアクセス許可](../manage/manage-azure-registration.md#azure-active-directory-app-permissions)を持っている必要があることに注意してください。そうしないと、登録プロセスが完了しません。代わりに、終了して、登録が管理者の同意待ちのままになります。 アクセス許可が付与されたら、`Register-AzStackHCI` を再実行して登録を完了します。

3. Azure での認証

   登録プロセスを完了するには、Azure アカウントを使用して認証 (サインイン) する必要があります。 登録を続行するには、上記の手順 4 で指定した Azure サブスクリプションへのアクセス権がアカウントに必要です。 提供されたコードをコピーし、別のデバイス (ご自分の PC や携帯電話など) で microsoft.com/devicelogin に移動して、コードを入力し、そこにサインインします。 これは、入力方式が限定されている他のデバイス (Xbox など) に対して Microsoft が使用するのと同じエクスペリエンスです。

登録ワークフローではログインしたことが検出され、完了に進みます。 その後、Azure portal でクラスターを確認できるようになります。

## <a name="next-steps"></a>次のステップ

これで以下の準備が整いました。

- [クラスターの検証](validate.md)

