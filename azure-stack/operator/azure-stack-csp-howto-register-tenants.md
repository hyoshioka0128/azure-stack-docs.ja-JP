---
title: Azure Stack Hub に使用量と課金用のテナントを追加する
description: Azure Stack Hub に使用量と課金用のテナントを追加する方法について説明します。
author: sethmanheim
ms.topic: article
ms.date: 9/02/2020
ms.author: sethm
ms.reviewer: alfredop
ms.lastreviewed: 5/28/2020
ms.openlocfilehash: e0d426ac19645cbbc318f66b827946534539d125
ms.sourcegitcommit: 7c01ab4b2e2250a7acd67d1c5ba27d15c1e8bce0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2020
ms.locfileid: "89448573"
---
# <a name="add-tenant-for-usage-and-billing-to-azure-stack-hub"></a>Azure Stack Hub に使用量と課金用のテナントを追加する

この記事では、クラウド ソリューション プロバイダー (CSP) が管理する Azure Stack Hub デプロイにテナントを追加する方法について説明します。 新しいテナントがリソースを使用すると、Azure Stack Hub から CSP サブスクリプションに使用量が報告されます。

多くの場合、CSP は Azure Stack Hub デプロイで複数のエンド カスタマー (テナント) にサービスを提供しています。 Azure Stack Hub の登録にテナントを追加すると、各テナントの使用量が報告され、対応する CSP サブスクリプションが課金されます。 この記事の手順を完了しない場合、テナントの使用量は Azure Stack Hub の初期登録で使用したサブスクリプションに課金されます。 使用量の追跡とそのテナントの管理のために Azure Stack Hub にエンド カスタマーを追加する前に、Azure Stack Hub を CSP として構成する必要があります。 手順とリソースについては、「[クラウド ソリューション プロバイダーとして Azure Stack Hub の使用状況と課金を管理する](azure-stack-add-manage-billing-as-a-csp.md)」を参照してください。

次の図では、CSP が設定する必要のある、新しいエンド カスタマーの Azure Stack Hub の使用手順、およびそのカスタマーの使用量の追跡手順を示しています。 エンド カスタマーを追加すると、Azure Stack Hub 内のリソースも管理できるようになります。 リソースの管理には 2 つのオプションがあります。

- エンド カスタマーを管理し、ローカルの Azure Stack Hub サブスクリプション用の資格情報をそのエンド カスタマーに提供できます。  
- エンド カスタマーが自身のサブスクリプションをローカルで管理し、所有者アクセス許可を持つゲストとして CSP を追加できます。

## <a name="add-an-end-customer"></a>エンド カスタマーを追加する

エンド カスタマーを追加する前に、登録でマルチテナントの課金を有効にする必要があります。 マルチテナントの課金を有効にするには、登録サブスクリプション ID、リソース グループ名、および登録名を `azstcsp@microsoft.com` に送信します。 マルチテナントを有効にするには、通常 1 から 2 営業日かかります。

次の図に示すように、以下の手順を実行してエンド カスタマーを追加します。

![使用状況の追跡とエンド カスタマー アカウントの管理のためにクラウド ソリューション プロバイダーを設定する](media/azure-stack-csp-enable-billing-usage-tracking/process-csp-enable-billing.png)

### <a name="create-a-new-customer-in-partner-center"></a>パートナー センターに新しい顧客を作成する

パートナー センターで、顧客の新しい Azure サブスクリプションを作成します。 手順については、「[新しい顧客の追加](/partner-center/add-a-new-customer)」を参照してください。

### <a name="create-an-azure-subscription-for-the-end-customer"></a>エンド カスタマーの Azure サブスクリプションを作成する

パートナー センターで顧客のレコードを作成したら、その顧客にカタログ内の製品のサブスクリプションを販売できます。 手順については、「[顧客サブスクリプションの作成、停止、キャンセル](/partner-center/create-a-new-subscription)」を参照してください。

### <a name="create-a-guest-user-in-the-end-customer-directory"></a>エンド カスタマー ディレクトリにゲスト ユーザーを作成する

既定では、CSP (あなた) はエンド カスタマーの Azure Stack Hub サブスクリプションにアクセスできません。 ただし、顧客があなたにリソースの管理を望む場合、顧客があなたのアカウントを所有者/共同作成者として自分の Azure Stack Hub サブスクリプションに追加できます。 そのためには、Azure AD テナントにゲスト ユーザーとしてあなたのアカウントを追加してもらう必要があります。 ご自分の顧客の Azure サブスクリプションに引き続きアクセスできるよう、ご自分の Azure CSP アカウントとは別のアカウントを使用してご自分の顧客の Azure Stack Hub サブスクリプションを管理することをお勧めします。

### <a name="update-the-registration-with-the-end-customer-subscription"></a>エンド カスタマーのサブスクリプションで登録を更新する

エンド カスタマーの新しいサブスクリプションで登録を更新します。 Azure は、パートナー センターの顧客 ID を使用して顧客の使用量をレポートします。 この手順により、各顧客の使用量が、その顧客それぞれの CSP サブスクリプションごとに報告されるようになります。 これで、使用量の追跡と課金が簡単になります。 この手順を実行するためには、まず [Azure Stack Hub に登録する](azure-stack-registration.md)必要があります。

1. 管理者特権のプロンプトで Windows PowerShell を開き、次を実行します。  

   ```powershell
   Add-AzureRmAccount
   ```

   >[!NOTE]
   > セッションの有効期限が切れた、パスワードが変更された、または単にアカウントを切り替えたい場合は、**Add-AzureRmAccount** を使用してサインインする前に、次のコマンドレットを実行します。`Remove-AzureRmAccount-Scope Process`

2. Azure の資格情報を入力します。
3. PowerShell セッションで、次のコマンドを実行します。

   ```powershell
   New-AzureRmResource -ResourceId "subscriptions/{registrationSubscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.AzureStack/registrations/{registrationName}/customerSubscriptions/{customerSubscriptionId}" -ApiVersion 2017-06-01
   ```

### <a name="new-azurermresource-powershell-parameters"></a>PowerShell の New-AzureRmResource パラメーター

次のセクションでは、**New-AzureRmResource** コマンドレットのパラメーターについて説明します。

| パラメーター | 説明 |
| --- | --- |
|registrationSubscriptionID | Azure Stack Hub の初期登録に使用された Azure サブスクリプション。|
| customerSubscriptionID | 登録する顧客が所有する (Azure Stack Hub ではない) Azure サブスクリプション。 CSP のオファー内で作成する必要があります。 実際には、パートナー センターを介することを意味します。 顧客が複数の Azure Active Directory テナントを持っている場合は、Azure Stack Hub へのログインに使用するテナントでこのサブスクリプションを作成する必要があります。 顧客サブスクリプション ID は大文字と小文字の区別があります。 |
| resourceGroup | 登録が格納されている Azure 内のリソース グループ。 |
| registrationName | お使いの Azure Stack Hub の登録名。 Azure に格納されているオブジェクトです。

> [!NOTE]  
> テナントは、使用する各 Azure Stack Hub に登録されている必要があります。 Azure Stack Hub のデプロイが 2 つあり、テナントがその両方を使用している場合は、各デプロイの初期登録をそのテナントのサブスクリプションで更新する必要があります。

### <a name="onboard-tenant-to-azure-stack-hub"></a>Azure Stack Hub にテナントをオンボードする

Azure Stack Hub を構成して、複数の Azure AD テナントのユーザーが Azure Stack Hub のサービスを使用できるようにできます。 手順については、[Azure Stack Hub でのマルチテナントの有効化](azure-stack-enable-multitenancy.md)に関するページを参照してください。

### <a name="create-a-local-resource-in-the-end-customer-tenant-in-azure-stack-hub"></a>Azure Stack Hub のエンド カスタマーのテナントにローカル リソースを作成する

Azure Stack Hub に新しいカスタマーを追加するか、お使いのゲスト アカウントがエンド カスタマーのテナントで所有者特権で有効にされたら、彼らがそのテナントにリソースを作成できることを確認してください。 たとえば、それらは [Azure Stack Hub ポータルで Windows 仮想マシンを作成する](../user/azure-stack-quick-windows-portal.md)ことができます。

## <a name="next-steps"></a>次のステップ

- 登録プロセス中にエラーが発生した場合にエラー メッセージを確認する方法については、[テナント登録時のエラー メッセージ](azure-stack-registration-errors.md)に関するページを参照してください。
- Azure Stack Hub からリソース使用量情報を取得する方法の詳細については、「[Azure Stack Hub での使用量と請求](azure-stack-billing-and-chargeback.md)」を参照してください。
- エンド カスタマーが CSP を Azure Stack Hub テナントの管理者として追加する方法を確認するには、[クラウド ソリューション プロバイダーがお使いの Azure Stack Hub サブスクリプションを管理する](../user/azure-stack-csp-enable-billing-usage-tracking.md)方法に関するページを参照してください。
