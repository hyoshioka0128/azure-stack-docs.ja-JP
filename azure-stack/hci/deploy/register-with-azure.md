---
title: Azure Stack HCI を Azure に接続する
description: Azure Stack HCI を Azure に登録する方法。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 09/24/2020
ms.openlocfilehash: 621662110b2e842a432183dfb7415f3c8365907b
ms.sourcegitcommit: 034e61836038ca75199a0180337257189601cd12
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/25/2020
ms.locfileid: "91230497"
---
# <a name="connect-azure-stack-hci-to-azure"></a>Azure Stack HCI を Azure に接続する

> 適用対象:Azure Stack HCI v20H2

Azure Stack HCI は Azure サービスとして提供され、Azure オンライン サービス条件に従ってインストールから 30 日以内に登録する必要があります。 このトピックでは、監視、サポート、課金、およびハイブリッド サービスのために、Azure Stack HCI クラスターを [Azure Arc](https://azure.microsoft.com/services/azure-arc/) に登録する方法について説明します。 登録時に、オンプレミスの個々の Azure Stack HCI クラスターを表す Azure Resource Manager リソースが作成されます。これにより、Azure 管理プレーンが Azure Stack HCI に効果的に拡張されます。 Azure リソースとオンプレミス クラスターの間で情報が定期的に同期されます。 

## <a name="prerequisites-for-registration"></a>登録の前提条件

Azure Stack HCI クラスターを作成しないと、Azure に登録することはできません。 ノードは物理マシンまたは仮想マシンにすることができますが、Unified Extensible Firmware Interface (UEFI) である必要があります。つまり、Hyper-V 第 1 世代の仮想マシンを使用することはできません。 Azure Arc 登録は Azure Stack HCI のネイティブ機能であるため、エージェントは必要ありません。

### <a name="internet-access"></a>インターネットへのアクセス

Azure Stack HCI ノードは、Azure に接続するためにクラウドに接続する必要があります。 たとえば、次の送信 ping は成功します。

```PowerShell
C:\> ping bing.com
```

### <a name="azure-subscription"></a>Azure サブスクリプション

まだ Azure アカウントを持っていない場合は[作成します](https://azure.microsoft.com/)。 

以下のいずれかの種類の既存のサブスクリプションを使用できます。
- [学生](https://azure.microsoft.com/free/students/)または [Visual Studio サブスクライバー](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/)向けの Azure クレジットが付いた無料アカウント
- クレジット カードを使用した[従量課金制](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/)サブスクリプション
- マイクロソフトエンタープライズ契約 (EA) を通じて取得されたサブスクリプション
- クラウド ソリューション プロバイダー (CSP) プログラムを通じて取得されたサブスクリプション

### <a name="azure-active-directory-permissions"></a>Azure Active Directory のアクセス許可

登録プロセスを完了するには、Azure Active Directory のアクセス許可が必要です。 まだお持ちでない場合は、Azure AD 管理者に依頼して、アクセス許可を付与または委任してもらいます。 詳しくは、「[Azure の登録を管理する](../manage/manage-azure-registration.md#azure-active-directory-permissions)」をご覧ください。

## <a name="register-using-powershell"></a>PowerShell を使用して登録する

Azure Stack HCI クラスターを Azure に登録するには、次の手順を使用します。

1. PowerShell セッションを開き、次のコマンドを入力して、いずれかのクラスター ノードに接続します。

   ```PowerShell
   Enter-PSSession <server-name>
   ```

2. Azure Stack HCI 用の PowerShell モジュールをインストールします。

   ```PowerShell
   Install-WindowsFeature RSAT-Azure-Stack-HCI
   ```

3. 必要なコマンドレットをインストールします。

   ```PowerShell
   Install-Module Az.StackHCI
   ```

   > [!NOTE]
   > 1. "今すぐ PowerShellGet で NuGet プロバイダーをインストールしてインポートしますか?" などのプロンプトが表示される場合があります。 これには [はい] (Y) と答える必要があります。
   > 2. "'PSGallery' からモジュールをインストールしますか? " というメッセージが表示され、[はい] (Y) と答える必要があります。
   > 3. 最後に、**Az** モジュール全体をインストールすると **StackHCI** サブモジュールが含められ、それが長期的に正しいと思われるかもしれませんが、 標準の Azure PowerShell 規約に従い、プレビュー段階のサブモジュールは自動的には含められません。そうではなく、これらを明示的に指定する必要があります。 したがって、ここでは上記のように **Az.StackHCI** を明示的に要求する必要があります。

4. 実際の登録を実行します。

   ```PowerShell
   Register-AzStackHCI  -SubscriptionId "<subscription_ID>" [-ResourceName] [-ResourceGroupName]
   ```

   この構文では、(ローカル サーバーがメンバーである) ローカル クラスターを現在のユーザーとして既定の Azure リージョンとクラウド環境に登録し、Azure リソースとリソース グループにスマートな既定の名前を使用します。ただし、このコマンドにパラメーターを追加して、必要に応じてこれらの値を指定することもできます。

   `Register-AzStackHCI` コマンドレットを実行するユーザーは [Azure Active Directory のアクセス許可](../manage/manage-azure-registration.md#azure-active-directory-permissions)を持っている必要があることに注意してください。そうしないと、登録プロセスが完了しません。代わりに、終了して、登録が管理者の同意待ちのままになります。 アクセス許可が付与されたら、`Register-AzStackHCI` を再実行して登録を完了します。

5. Azure での認証

   登録プロセスを完了するには、Azure アカウントを使用して認証 (サインイン) する必要があります。 登録を続行するには、上記の手順 4 で指定した Azure サブスクリプションへのアクセス権がアカウントに必要です。 提供されたコードをコピーし、別のデバイス (ご自分の PC や携帯電話など) で microsoft.com/devicelogin に移動して、コードを入力し、そこにサインインします。 これは、入力方式が限定されている他のデバイス (Xbox など) に対して Microsoft が使用するのと同じエクスペリエンスです。

登録ワークフローではログインしたことが検出され、完了に進みます。 その後、Azure portal でクラスターを確認できるようになります。

## <a name="next-steps"></a>次のステップ

これで以下の準備が整いました。

- [クラスターの検証](validate.md)
- [ボリュームの作成](../manage/create-volumes.md)
