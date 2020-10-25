---
title: Azure Stack Hub での使用量と請求の登録エラー コード
description: Azure Stack Hub での使用量と請求の登録エラーコードについて説明します。
author: sethmanheim
ms.topic: article
ms.date: 08/27/2020
ms.author: sethm
ms.reviewer: avishwan
ms.lastreviewed: 06/27/2019
ms.openlocfilehash: 0c143a2a997a044e102eef63018dba5a492a3e11
ms.sourcegitcommit: 03aad17afe8519536066c735c59ad1bdfe8de083
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/27/2020
ms.locfileid: "89041606"
---
# <a name="usage-and-billing-registration-error-codes"></a>使用量と請求の登録エラー コード

Cloud Solution Provider (CSP) の場合、顧客の Azure サブスクリプション ID に対して使用量をレポートするための登録に[テナントを追加](azure-stack-csp-ref-operations.md#add-tenant-to-registration)するときに、次のエラー メッセージが発生することがあります。

## <a name="list-of-registration-error-codes"></a>登録エラー コードの一覧

| エラー   | 詳細  | 説明  |
|---|---|---|
| RegistrationNotFound | 指定された登録が見つかりませんでした。 次の情報が正しく指定されていることを確認してください。<br>1.サブスクリプション識別子 (指定された値: "**サブスクリプション識別子**")、<br>2.リソース グループ (指定された値: "**リソース グループ**")、<br>3.登録名 (指定された値: "**登録名**")。 | 通常、このエラーは初期登録を示す情報が正しくない場合に発生します。 リソース グループと登録の名前を確認する必要がある場合は、Azure Portal ですべてのリソースを一覧表示して探すことができます。 複数の登録リソースを検索する場合は、プロパティで **CloudDeploymentID** を参照し、**CloudDeploymentID** がクラウドと一致する登録を選択します。 **CloudDeploymentID** を特定するには、Azure Stack Hub 上で次の PowerShell コマンドを使用します。<br>`$azureStackStampInfo = Invoke-Command -Session $session -ScriptBlock { Get-AzureStackStampInformation }` |
| BadCustomerSubscriptionId | 指定された "**顧客サブスクリプション識別子**" と "**登録名**" サブスクリプション識別子は、同じ Microsoft CSP によって所有されていません。 顧客サブスクリプション識別子が正しいことを確認してください。 顧客サブスクリプション ID は大文字と小文字の区別があります。 問題が解決しない場合は、サポートにお問い合わせください。 | このエラーは、顧客サブスクリプションが CSP サブスクリプションで、初期登録で使用されたサブスクリプションのロールアップ先とは異なる CSP パートナーにロールアップされる場合に発生します。 このチェックは、使用される Azure Stack Hub について担当外の CSP パートナーへの請求を避けるために実行されます。 |
| InvalidCustomerSubscriptionId  | "**顧客サブスクリプション識別子**" が無効です。 有効な Azure サブスクリプションが指定されていることを確認してください。 |   |
| CustomerSubscriptionNotFound  | "**登録名**" に "**顧客サブスクリプション識別子**" が見つかりませんでした。 有効な Azure サブスクリプションを使用中であることと、サブスクリプション ID が PUT 操作を使用して登録に追加されたことを確認してください。 | このエラーは、テナントがサブスクリプションに追加されたことを検証しようとしたが、登録に関連付けられた顧客サブスクリプションが見つからない場合に発生します。 顧客が登録に追加されていないか、サブスクリプション ID が正しく記述されていません。 |
| UnauthorizedCspRegistration | 指定された "**登録名**" はマルチ テナントの使用を承認されていません。 電子メールを azstCSP@microsoft.com に送信し、登録名、リソース グループ、および登録で使用されたサブスクリプション識別子を含めます。 | 登録へのテナントの追加を開始するには、その前にマルチテナントに関して Microsoft から登録の承認を受ける必要があります。 |
| CustomerSubscriptionsNotAllowed | 切断されている顧客に対して顧客サブスクリプション操作はサポートされていません。 この機能を使用するには、従量課金ライセンスで再登録します。 | テナントを追加しようとしている登録は容量登録です。 そのため、登録が作成されたとき、パラメーター `BillingModel Capacity` が使用されました。 テナントを追加できるのは従量課金制の登録のみです。 `BillingModel PayAsYouUse` パラメーターを使用して再登録する必要があります。 |
| InvalidCSPSubscription | 指定された "**顧客サブスクリプション識別子**" は有効な CSP サブスクリプションではありません。 有効な Azure サブスクリプションが指定されていることを確認してください。 | ほとんどの場合、このエラーは誤って入力された顧客サブスクリプションが原因です。 |
| MetadataResolverBadGatewayError | アップストリーム サーバーの 1 つが予期しないエラーを返しました。 後で再試行してください。 問題が解決しない場合は、サポートにお問い合わせください。 |

## <a name="next-steps"></a>次のステップ

- 詳細については、「[クラウド ソリューション プロバイダー向けの使用量レポート インフラストラクチャ](azure-stack-csp-ref-infrastructure.md)」を参照してください。
- CSP プログラムの詳細については、「[クラウド ソリューション](https://partner.microsoft.com/solutions/microsoft-cloud-solutions)」を参照してください。
- Azure Stack Hub からリソース使用量情報を取得する方法の詳細については、「[Azure Stack Hub での使用量と請求](azure-stack-billing-and-chargeback.md)」をご覧ください。
