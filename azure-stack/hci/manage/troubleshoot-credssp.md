---
title: CredSSP のトラブルシューティング
description: CredSSP をトラブルシューティングする方法について説明します
author: v-dasis
ms.topic: how-to
ms.date: 08/06/2020
ms.author: v-dasis
ms.reviewer: JasonGerend
ms.openlocfilehash: 24e9483aa9658809adc9be7fbfa4a2cb13daab84
ms.sourcegitcommit: 952d26ad08fcc28ad3ad83e27644e61497623a44
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/06/2020
ms.locfileid: "87899928"
---
# <a name="troubleshoot-credssp"></a>CredSSP のトラブルシューティング

> 適用対象: Azure Stack HCI バージョン v20H2

Azure Stack HCI の一部の操作には、Windows リモート管理 (WinRM) が使用されます。WinRM では、資格情報の委任が既定ではサポートされません。 委任を許可するには、コンピューターで一時的に Credential Security Support Provider (CredSSP) を有効にする必要があります。 CredSSP は、リモート認証を目的にターゲット サーバーへの資格情報の委任をクライアントに許可するセキュリティ サポート プロバイダーです。 

CredSSP を有効にすると、セキュリティの状態が低下します。ほとんどの場合、目的のタスクまたは操作が完了したら、無効にすることをお勧めします。

CredSSP を有効にする必要のあるタスクの例を次に示します。

- クラスターの作成ウィザードのワークフロー
- Active Directory のクエリまたは更新
- SQL Server のクエリまたは更新
- 別のドメインまたは非ドメイン参加環境でのアカウントまたはコンピューターの検索

## <a name="troubleshooting-tips"></a>トラブルシューティングのヒント

CredSSP に関する問題が発生した場合、トラブルシューティングのヒントとして、次の情報を参考にしてください。

- Active Directory の信頼が確立されていないか信頼が破られた場合、クラスターの作成ウィザードを実行しているときに CredSSP から問題がレポートされることがあります。 これは、クラスターの作成にワークグループベースのサーバーが使用されると発生します。 この場合は、クラスター内の各サーバーを手動で再起動してみてください。

- Windows Admin Center をサーバー (サービス モード) で実行するときは、ユーザー アカウントがゲートウェイ管理者グループのメンバーであることを確認してください。

- サーバーで CredSSP を有効または無効にするには、そのコンピューターのゲートウェイ管理者グループに自分が属していることを確認してください。 詳細については、「[ユーザー アクセス制御とアクセス許可を構成する](/windows-server/manage/windows-admin-center/configure/user-access-control#gateway-access-role-definitions)」の最初の 2 つのセクションを参照してください。

## <a name="next-steps"></a>次のステップ

CredSSP の詳細については、「[Credential Security Support Provider](/windows/win32/secauthn/credential-security-support-provider)」を参照してください。