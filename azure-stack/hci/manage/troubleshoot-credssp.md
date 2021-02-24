---
title: CredSSP のトラブルシューティング
description: CredSSP をトラブルシューティングする方法について説明します
author: v-dasis
ms.topic: how-to
ms.date: 12/14/2020
ms.author: v-dasis
ms.reviewer: JasonGerend
ms.openlocfilehash: 472f0cf6103ac1559a9f8a0f757d66bc545f9a5d
ms.sourcegitcommit: f4a1a7e9d0b64ca84105d48170a23e1f473e976c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/23/2020
ms.locfileid: "97743541"
---
# <a name="troubleshoot-credssp"></a>CredSSP のトラブルシューティング

> 適用対象: Azure Stack HCI バージョン v20H2

Azure Stack HCI の一部の操作には、Windows リモート管理 (WinRM) が使用されます。WinRM では、資格情報の委任が既定ではサポートされません。 委任を許可するには、コンピューターで一時的に Credential Security Support Provider (CredSSP) を有効にする必要があります。 CredSSP は、リモート認証を目的にサーバーへの資格情報の委任をクライアントに許可するセキュリティ サポート プロバイダーです。

CredSSP を有効にすると、セキュリティの状態が低下します。ほとんどの場合、目的のタスクまたは操作が完了したら、無効にすることをお勧めします。

CredSSP を有効にする必要のあるタスクの例を次に示します。

- クラスターの作成ウィザードのワークフロー
- Active Directory のクエリまたは更新
- SQL Server のクエリまたは更新
- 別のドメインまたは非ドメイン参加環境でのアカウントまたはコンピューターの検索

## <a name="troubleshooting-tips"></a>トラブルシューティングのヒント

CredSSP に関する問題が発生した場合、トラブルシューティングのヒントとして、次の情報を参考にしてください。

- PC ではなくサーバー上で Windows Admin Center を実行しているときにクラスターの作成ウィザードを使用するには、Windows Admin Center サーバー上でゲートウェイ管理者グループのメンバーである必要があります。 詳細については、「[Windows Admin Center でのユーザー アクセス オプション](/windows-server/manage/windows-admin-center/plan/user-access-options)」を参照してください。

- Active Directory の信頼が確立されていないか信頼が破られた場合、クラスターの作成ウィザードを実行しているときに CredSSP から問題がレポートされることがあります。 これは、クラスターの作成にワークグループベースのサーバーが使用されると発生します。 この場合は、クラスター内の各サーバーを手動で再起動してみてください。

- Windows Admin Center をサーバーで実行するときは、ユーザー アカウントがゲートウェイ管理者グループのメンバーであることを確認してください。

- マネージド サーバーと同じドメインのメンバーであるコンピューター上で Windows Admin Center を実行することをお勧めします。

- サーバーで CredSSP を有効または無効にするには、そのコンピューターのゲートウェイ管理者グループに自分が属していることを確認してください。 詳細については、「[ユーザー アクセス制御とアクセス許可を構成する](/windows-server/manage/windows-admin-center/configure/user-access-control#gateway-access-role-definitions)」の最初の 2 つのセクションを参照してください。

- クラスター内のサーバー上で WinRM サービスを再起動すると、各クラスター サーバーと Windows Admin Center 間の WinRM 接続を再確立するように求められることがあります。

    これを行う方法の 1 つは、各クラスター サーバーに移動し、Windows Admin Center の **[ツール]** メニューの **[サービス]** 、 **[WinRM]** 、 **[再起動]** の順に選択し、 **[サービスの再起動]** プロンプトで **[はい]** を選択することです。

## <a name="manual-troubleshooting"></a>手動でのトラブルシューティング

次の WinRM エラー メッセージが表示された場合は、このセクションの手動による検証手順を使用してエラーを解決してください。 サンプルのエラー メッセージ:

`Connecting to remote <sever name> failed with the following error message: The WinRM client cannot process the request. A computer policy does not allow the delegation of the user credentials to the target computer because the computer is not trusted. The identity of the target computer can be verified if you configure the WSMAN service to use a valid certificate.`

このセクションの手動による検証手順では、次のコンピューターを構成する必要があります。
- Windows Admin Center を実行しているコンピューター
- エラー メッセージを受信したサーバー

このエラーを解決するには、必要に応じて次の救済手順を試してください。

**救済策 1:**
1. Windows Admin Center とサーバーを実行しているコンピューターを再起動します。
1. クラスターの作成ウィザードをもう一度実行してみます。

    ウィザードの実行の詳細については、「[Windows Admin Center を使用して Azure Stack HCI クラスターを作成する](../deploy/create-cluster.md)」を参照してください。

**救済策 2:**
1. Windows Admin Center を実行しているコンピューターで、管理者として Windows PowerShell を開き、次のコマンドを実行します。

    ```powershell
    Disable-WsmanCredSSP -Role Client  
    ```

    ```powershell  
    Enable-WsmanCredSSP -Role Client -DelagateComputer <Server FQDN Name>
    ```

1. RDP 機能を使用してサーバーに接続した後、次の PowerShell コマンドを実行します。

    ```powershell  
    Disable-WsmanCredSSP -Role Server  
    ```

    ```powershell  
    Enable-WsmanCredSSP -Role Server  
    ```
    
1. クラスターの作成ウィザードをもう一度実行してみます。

    ウィザードの実行の詳細については、「[Windows Admin Center を使用して Azure Stack HCI クラスターを作成する](../deploy/create-cluster.md)」を参照してください。

**救済策 3:**
1. Windows Admin Center を実行しているコンピューターで、次の PowerShell コマンドを実行してサービス プリンシパル名 (SPN) を確認します。

    ```powershell
    setspn -Q WSMAN/<Windows Admin Center Computer Name>  
    ```
    
    結果は次の出力のようになります。

    `WSMAN/<Windows Admin Center Computer Name>`

    `WSMAN/<Windows Admin Center Computer FQDN Name>`

1. 結果が表示されない場合は、次の PowerShell コマンドを実行して SPN を登録します。

    ```powershell
    setspn -S WSMAN/<Windows Admin Center Computer Name> <Windows Admin Center Computer Name>  
    ```

    ```powershell
    setspn -S WSMAN/<Windows Admin Center Computer Name> <Windows Admin Center Computer FQDN Name>  
    ```

1. RDP 機能を使用してサーバーに接続した後、次の PowerShell コマンドを実行して SPN を確認します。

    ```powershell
    setspn -Q WSMAN/<Server Name>  
    ```

    結果は次の出力のようになります。

    `WSMAN/<Server Name>`

    `WSMAN/<Server FQDN Name>`

1. 結果が表示されない場合は、次の PowerShell コマンドを実行して SPN を登録します。

    ```powershell
    setspn -S WSMAN/<Server Name> <Server Name>  
    ```

    ```powershell
    setspn -S WSMAN/<Server Name> <Server FQDN Name>  
    ```

1. クラスターの作成ウィザードをもう一度実行してみます。

    ウィザードの実行の詳細については、「[Windows Admin Center を使用して Azure Stack HCI クラスターを作成する](../deploy/create-cluster.md)」を参照してください。


**救済策 4:**

前述の救済策のいずれかが失敗したか完了しなかった場合は、Active Directory でレコードの競合が発生している可能性があります。 別のコンピューター名を使用して、Active Directory のレコードを新しいレコードとして再設定できます。

Active Directory のレコードを再設定するには、Azure Stack HCI オペレーティング システムを新しいコンピューター名で再インストールします。

## <a name="next-steps"></a>次のステップ

CredSSP の詳細については、「[Credential Security Support Provider](/windows/win32/secauthn/credential-security-support-provider)」を参照してください。