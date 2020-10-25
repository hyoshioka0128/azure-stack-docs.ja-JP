---
title: Azure Stack HCI の Azure 登録を管理する
description: PowerShell を使用して、Azure Stack HCI の Azure 登録を管理し、登録状態を把握する方法。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.date: 07/29/2020
ms.openlocfilehash: 696ef552dcf49f31fb613a22393617e653f7e10d
ms.sourcegitcommit: eb91a28a19a74f799b093ae2a705f7f6e4c5cd49
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/30/2020
ms.locfileid: "87436471"
---
# <a name="manage-azure-registration"></a>Azure 登録の管理

> Azure Stack HCI v20H2 に適用されます

Azure Stack HCI クラスターを作成したら、[クラスターを Azure Arc](../deploy/register-with-azure.md) に登録する必要があります。クラスターが登録されると、オンプレミス クラスターとクラウドの間で定期的に情報が同期されます。 このトピックでは、登録状態を把握し、使用停止の準備ができたときにクラスターの登録を解除する方法について説明します。

## <a name="understanding-registration-status"></a>登録状態の把握

登録状態を把握するには、`Get-AzureStackHCI` PowerShell コマンドレットと、`ClusterStatus`、`RegistrationStatus`、および `ConnectionStatus` の各プロパティを使用します。 たとえば、Azure Stack HCI オペレーティング システムをインストールした後、クラスターの作成または参加を行う前は、`ClusterStatus` プロパティには "not yet" (未完了) の状態が表示されます。

:::image type="content" source="media/manage-azure-registration/1-get-azurestackhci.png" alt-text="クラスターを作成する前の Azure の登録状態":::

クラスターが作成されると、`RegistrationStatus` にのみ "not yet" (未完了) の状態が表示されます。

:::image type="content" source="media/manage-azure-registration/2-get-azurestackhci.png" alt-text="クラスターを作成した後の Azure の登録状態":::

Azure Stack HCI では、Azure のオンライン サービス条件に従って、インストールから 30 日以内に登録する必要があります。 30 日間の経過後にクラスター化されていない場合、`ClusterStatus` には `OutOfPolicy` が表示され、30 日間の経過後に登録されていない場合、`RegistrationStatus` には `OutOfPolicy` が表示されます。

クラスターが登録されると、`ConnectionStatus` と `LastConnected` の時刻が表示されます。クラスターがインターネットから一時的に切断されていなければ、通常、これは最終日の値です。 Azure Stack HCI クラスターは、最大 30 日間、完全にオフラインで動作できます。

:::image type="content" source="media/manage-azure-registration/3-get-azurestackhci.png" alt-text="登録後の Azure の登録状態":::

その最大期間を超えた場合、`ConnectionStatus` には `OutOfPolicy` が表示されます。

## <a name="azure-active-directory-permissions"></a>Azure Active Directory のアクセス許可

サブスクリプションで Azure リソースを作成することに加えて、Azure Stack HCI を登録すると、概念的にはユーザーに似たアプリ ID が Azure Active Directory テナントに作成されます。 アプリ ID にはクラスター名が継承されます。 この ID は、サブスクリプション内で、Azure Stack HCI クラウド サービスに代わって適切に機能します。

`Register-AzureStackHCI` を実行するユーザーが Azure Active Directory 管理者であるか、十分なアクセス許可が委任されている場合、これはすべて自動的に行われるため、追加の操作は必要ありません。 そうでない場合は、登録を完了するには、Azure Active Directory 管理者の承認が必要になる場合があります。 管理者は、アプリに明示的に同意するか、アプリに同意できるアクセス許可を委任することができます。

:::image type="content" source="media/manage-azure-registration/aad-permissions.png" alt-text="Azure Active Directory のアクセス許可と ID の図" border="false":::

同意するには、portal.azure.com を開き、Azure Active Directory に対して十分なアクセス許可を持つ Azure アカウントでサインインします。 **Azure Active Directory** に移動し、 **[アプリの登録]** を選択します。 クラスターにちなんで名付けられたアプリ ID を選択し、 **[API のアクセス許可]** に移動します。

アプリには、次の 2 つのアクセス許可が必要です。

```http
https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.Census.Sync

https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.Billing.Sync
```

Azure Active Directory 管理者から承認を受けるには時間がかかる場合があるため、`Register-AzureStackHCI` コマンドレットは終了し、登録は "pending admin consent" (管理者の同意待ち)、つまり部分的に完了した状態のままになります。 同意が得られたら、`Register-AzureStackHCI` をもう一度実行するだけで登録は完了します。

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
