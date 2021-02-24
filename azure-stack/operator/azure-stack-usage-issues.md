---
title: Azure Stack Hub での使用量接続の問題とエラー
description: Azure Stack Hub 使用量接続の問題とエラーのトラブルシューティング。
author: sethmanheim
ms.topic: article
ms.date: 12/15/2020
ms.author: sethm
ms.reviewer: avishwan
ms.lastreviewed: 06/27/2019
ms.openlocfilehash: 8b5522cc15972018be4130bf7901444a9e4cdf05
ms.sourcegitcommit: a53ea4a28e715c80a99fa89e9d364bc4556558de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/16/2020
ms.locfileid: "97577008"
---
# <a name="usage-connectivity-errors"></a>使用量接続エラー

Azure Stack Hub の使用状況データは、Azure Stack Hub 内の [*Azure Bridge*](azure-stack-usage-reporting.md) コンポーネントによって、Azure に送信されます。 Azure Stack Hub 内のブリッジでは、Azure 使用状況サービスに接続できない場合、次のエラーが表示されます。

![使用量ブリッジ エラー](media/azure-stack-usage-issues/usageerror2.png)

ウィンドウに、エラーや解決方法についての詳しい情報が表示されることがあります。

![エラーの解決方法](media/azure-stack-usage-issues/usageerror3.png)

## <a name="resolve-connectivity-issues"></a>接続に関する問題を解決する

問題を緩和するには、次の手順を試してください。

- ネットワーク構成によって Azure Bridge がリモート サービスに接続されることを確認します。

- [ **[Region Management]\(リージョン管理\)**  >  **[プロパティ]**](azure-stack-registration.md#verify-azure-stack-hub-registration) ブレードに移動して、登録に使用された Azure サブスクリプション ID、リソース グループ、登録リソースの名前を検索します。 Azure portal で、登録リソースが正しい Azure サブスクリプション ID の下に存在することを確認します。 これを行うには、Azure サブスクリプション ID で作成された **[すべてのリソース]** に移動して、 **[非表示の型の表示]** ボックスをオンにします。 登録リソースが見つからない場合は、「[登録を更新または変更する](azure-stack-registration.md#renew-or-change-registration)」の手順に従って、ご利用の Azure Stack Hub を再登録します。

  ![ポータル](media/azure-stack-usage-issues/stackres.png)

## <a name="error-codes"></a>エラー コード

このセクションでは、使用量エラー コードについて説明します。

| エラー コード                 | 問題                                                                                                                                             | Remediation                                                                                                                                                                                                                                                                                        |
|----------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| NetworkError               | Azure Stack Hub のブリッジで Azure の使用状況サービス エンドポイントへの要求を送信することができません。                                                            | プロキシによって使用状況サービス エンドポイントへのアクセスがブロックまたは遮断されていないか確認します。                                                                                                                                                                                                             |
| RequestTimedOut            | Azure Bridge から要求が送信されましたが、Azure の使用状況サービスがタイムアウト時間内に応答できませんでした。                             | プロキシによって使用状況サービス エンドポイントへのアクセスがブロックまたは遮断されていないか確認します。                                                                                                                                                                                                                        |
| LoginError                 | Microsoft Azure Active Directory で認証できませんでした。                                                                                                             | Azure AD ログイン エンドポイントが Azure Stack Hub のすべての XRP VM から確実にアクセスできるようにします。                                                                                                                                                                                                                     |
| CertificateValidationError | Azure サービスで認証できないため、Azure Bridge では要求を送信できません。                                    | Azure Stack Hub XRP マシンと使用状況ゲートウェイ エンドポイントとの間の HTTPS トラフィックを遮断しているプロキシがないか確認します。                                                                                                                                                                                      |
| 権限がありません               | Azure サービスでは Azure Stack Hub のブリッジを認証できないため、Azure Bridge ではデータを Azure 内の使用状況サービスにプッシュできません。 | 登録リソースが変更されていないか確認し、そうであれば Azure Stack Hub を再登録します。 <br><br> 場合によっては、Azure Stack Hub と Azure AD との間の時間同期の問題がこの失敗の原因となります。 この場合、Azure Stack Hub の XRP VM の時間が Azure AD と確実に同期しているようにします。 |
|                            |                                                                                                                                                   |                                                                                                                                                                                                                                                                                                    |

さらに、Azure Bridge、WAS、WASPublic コンポーネントのログ ファイルを提供することが求められる場合があります。

## <a name="next-steps"></a>次のステップ

- 詳しくは、「[Azure Stack Hub 使用状況データの Azure への報告](azure-stack-usage-reporting.md)」をご覧ください。
- 登録プロセス中にエラーが発生した場合にエラー メッセージを確認する方法については、[テナント登録時のエラー メッセージ](azure-stack-registration-errors.md)に関するページを参照してください。
- 詳細については、「[クラウド ソリューション プロバイダー向けの使用量レポート インフラストラクチャ](azure-stack-csp-ref-infrastructure.md)」を参照してください。
