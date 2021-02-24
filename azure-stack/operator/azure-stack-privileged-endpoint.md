---
title: Azure Stack Hub での特権エンドポイントの使用
description: オペレーターとして Azure Stack Hub 内で特権エンドポイント (PEP) を使用する方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 12/16/2020
ms.author: mabrigg
ms.reviewer: fiseraci
ms.lastreviewed: 04/28/2020
ms.custom: conteperfq4
ms.openlocfilehash: 4e1a00be9f4adadee5d1bf1e647c455773211b61
ms.sourcegitcommit: 5f3d37994b8cb63c76e54136c0cc05bc4f475950
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99495535"
---
# <a name="use-the-privileged-endpoint-in-azure-stack-hub"></a>Azure Stack Hub での特権エンドポイントの使用

Azure Stack Hub オペレーターは、管理ポータル、PowerShell、または Azure Resource Manager API を使用して、ほとんどの日常的な管理タスクを実行します。 ただし、あまり一般的でない一部の操作については、*特権エンドポイント* (PEP) を使用する必要があります。 この PEP は、あらかじめ構成されたリモート PowerShell コンソールであり、必要なタスクを実行するために十分な機能だけを提供します。 エンドポイントは [PowerShell JEA (Just Enough Administration)](/powershell/scripting/learn/remoting/jea/overview) を使用して、コマンドレットの限定的なセットのみを公開します。 PEP にアクセスしてコマンドレットの限定的なセットを起動するために、低権限のアカウントが使用されます。 管理者アカウントは必要ありません。 セキュリティ強化のため、スクリプトは許可されません。

PEP を使用すると、次のタスクを実行できます。

- [診断ログの収集](azure-stack-get-azurestacklog.md)などの低レベル タスク。
- デプロイ後のドメイン ネーム システム (DNS) フォワーダーの追加や、Microsoft Graph 統合、Active Directory フェデレーション サービス (AD FS) 統合、証明書ローテーションの設定など、統合システムのための多くのデプロイ後データセンター統合タスク。
- 統合システムの詳細なトラブルシューティングのために、サポート部門と連携して一時的な高レベル アクセスを取得。

PEP では、PowerShell セッションで実行するすべてのアクション (および、それに対応する出力) がログに記録されます。 これにより、完全な透明性と操作の完全な監査が提供されます。 これらのログ ファイルは将来の監査のために保持できます。

> [!NOTE]
> Azure Stack Development Kit (ASDK) では、PEP で利用可能なコマンドの一部を、開発キットのホスト上の PowerShell セッションから直接実行できます。 ただし、これは統合システム環境で特定の操作を実行するために利用可能な唯一の手段であるため、ログ収集など、PEP を使用した一部の操作をテストすることが必要な場合があります。

[!INCLUDE [Azure Stack Hub Operator Access Workstation](../includes/operator-note-owa.md)]

## <a name="access-the-privileged-endpoint"></a>特権エンドポイントへのアクセス

PEP には、PEP をホストする仮想マシン (VM) 上のリモート PowerShell セッションを介してアクセスします。 ASDK では、この VM の名前は **AzS-ERCS01** です。 統合システムを使用している場合、PEP の 3 つのインスタンスがあり、それぞれ異なるホスト上の VM (*Prefix*-ERCS01、*Prefix*-ERCS02、または *Prefix*-ERCS03) 内で動作することで、回復性を確保しています。

統合システムに対してこの手順を開始する前に、IP アドレスによって、または DNS を介して PEP にアクセスできることを確認してください。 Azure Stack Hub の初期デプロイ後は、DNS 統合がまだセットアップされていないため、IP アドレスでしか PEP にアクセスできません。 PEP の IP アドレス を含む **AzureStackStampDeploymentInfo** という名前の JSON ファイルが、OEM ハードウェア ベンダーから提供されます。

IP アドレスは Azure Stack Hub 管理者ポータルでも見つかります。 `https://adminportal.local.azurestack.external` などのポータルを開きます。 **[リージョンの管理]**  >  **[プロパティ]** を選択します。

特権エンドポイントの実行時には、現在のカルチャ設定を `en-US` に設定する必要があります。そうしないと、Test-AzureStack や Get-AzureStackLog などのコマンドレットが想定されているように機能しません。

> [!NOTE]
> セキュリティ上の理由から、PEP への接続は、ハードウェア ライフサイクル ホスト上で実行されているセキュリティ強化された VM から、または[特権アクセス ワークステーション](https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model)のような専用のセキュリティで保護されたコンピューターからに限定して行う必要があります。 ハードウェア ライフサイクル ホストは、元の構成から変更しないようにし (新しいソフトウェアのインストールするなど)、PEP への接続にも使わないようにする必要があります。

1. 信頼関係を確立します。

   - 統合システムで、管理者特権の Windows PowerShell セッションから次のコマンドを実行して、ハードウェア ライフサイクル ホストまたは特権アクセス ワークステーションで実行されているセキュリティ強化された VM の信頼されたホストとして PEP を追加します。

      ```powershell  
      Set-Item WSMan:\localhost\Client\TrustedHosts -Value '<IP Address of Privileged Endpoint>' -Concatenate
      ```
      
   - ASDK を実行している場合、開発キットのホストにサインインします。

1. ハードウェア ライフサイクル ホストまたは特権アクセス ワークステーションで実行されているセキュリティ強化された VM で、Windows PowerShell セッションを開きます。 次のコマンドを実行して、PEP をホストする VM 上でリモート セッションを確立します。
 
   - 統合システム上で:

      ```powershell  
      $cred = Get-Credential

      $pep = New-PSSession -ComputerName <IP_address_of_ERCS> -ConfigurationName PrivilegedEndpoint -Credential $cred -SessionOption (New-PSSessionOption -Culture en-US -UICulture en-US)
      Enter-PSSession $pep
      ```
    
      `ComputerName` パラメーターには、PEP をホストする 1 つの VM の IP アドレスまたは DNS 名を指定できます。

      > [!NOTE]  
      > Azure Stack Hub は、PEP 資格情報の検証時にリモート呼び出しを行いません。 ローカルに保存された RSA 公開キーに依存して、それを行います。

   - ASDK を実行している場合:

      ```powershell  
      $cred = Get-Credential
    
      $pep = New-PSSession -ComputerName azs-ercs01 -ConfigurationName PrivilegedEndpoint -Credential $cred -SessionOption (New-PSSessionOption -Culture en-US -UICulture en-US)
      Enter-PSSession $pep
      ```
    
   入力を求められたら、次の資格情報を使用します。
   
   -  **ユーザー名**: **&lt;"*Azure Stack Hub ドメイン*"&gt;\cloudadmin** の形式で CloudAdmin アカウントを指定します。 (ASDK の場合、ユーザー名は **azurestack\cloudadmin** です)
   - **パスワード**:インストール中に AzureStackAdmin ドメイン管理者アカウントのパスワードとして指定したものと同じパスワードを入力します。

   > [!NOTE]
   > ERCS エンドポイントに接続できない場合は、別の ERCS VM の IP アドレスを使用して、手順 1 と手順 2 を再試行してください。

1. 接続後、環境に応じて **[*IP アドレスまたは ERCS VM 名*]: PS>** または **[azs-ercs01]:PS>** プロンプトが変わります。 ここから `Get-Command` を実行して、利用可能なコマンドレットの一覧を表示します。

   コマンドレットのリファレンスについては、「[Azure Stack ハブの特権エンドポイント リファレンス](../reference/pep-2002/index.md)」を参照してください。

   これらのコマンドレットの多くは、統合システム環境での使用のみが意図されています (データセンター統合に関連するコマンドレットなど)。 ASDK では、次のコマンドレットが検証済みです。

   - Clear-Host
   - Close-PrivilegedEndpoint
   - Exit-PSSession
   - Get-AzureStackLog
   - Get-AzureStackStampInformation
   - Get-Command
   - Get-FormatData
   - Get-Help
   - Get-ThirdPartyNotices
   - Measure-Object
   - New-CloudAdminUser
   - Out-Default
   - Remove-CloudAdminUser
   - Select-Object
   - Set-CloudAdminUserPassword
   - Test-AzureStack
   - Stop-AzureStack
   - Get-ClusterLog

## <a name="how-to-use-the-privileged-endpoint"></a>特権エンドポイントの使用方法 

前述のとおり、PEP は、[PowerShell JEA](/powershell/scripting/learn/remoting/jea/overview) エンドポイントです。 JEA エンドポイントにより、強力なセキュリティ層が提供される一方で、スクリプトやタブ補完などの基本的な PowerShell の機能の一部が失われます。 何らかの種類のスクリプト操作を試みると、エラー **ScriptsNotAllowed** で操作は失敗します。 このエラーは予想される動作です。

たとえば、特定のコマンドレットについてパラメーターの一覧を取得するには、次のコマンドを実行します。

```powershell
    Get-Command <cmdlet_name> -Syntax
```

または、[**Import-PSSession**](/powershell/module/microsoft.powershell.utility/import-pssession?view=powershell-5.1) コマンドレット使用して、ローカル コンピューターの現在のセッションにすべての PEP コマンドレットをインポートすることもできます。 PEP のすべてコマンドレットと関数を、タブ補完や、より一般にはスクリプトと共に、ローカル コンピューターで利用できるようになります。 **[Get-Help](/powershell/module/microsoft.powershell.core/get-help)** モジュールを実行して、コマンドレットの手順を確認することもできます。

ローカル コンピューターに PEP セッションをインポートするには、次の手順を実行します。

1. 信頼関係を確立します。

   - 統合システムで、管理者特権の Windows PowerShell セッションから次のコマンドを実行して、ハードウェア ライフサイクル ホストまたは特権アクセス ワークステーションで実行されているセキュリティ強化された VM の信頼されたホストとして PEP を追加します。

      ```powershell
      winrm s winrm/config/client '@{TrustedHosts="<IP Address of Privileged Endpoint>"}'
      ```

   - ASDK を実行している場合、開発キットのホストにサインインします。

1. ハードウェア ライフサイクル ホストまたは特権アクセス ワークステーションで実行されているセキュリティ強化された VM で、Windows PowerShell セッションを開きます。 次のコマンドを実行して、PEP をホストする仮想マシン上でリモート セッションを確立します。

   - 統合システム上で:
    
      ```powershell  
      $cred = Get-Credential
      
      $session = New-PSSession -ComputerName <IP_address_of_ERCS> `
        -ConfigurationName PrivilegedEndpoint -Credential $cred
      ```
    
      `ComputerName` パラメーターには、PEP をホストする 1 つの VM の IP アドレスまたは DNS 名を指定できます。

   - ASDK を実行している場合:
     
      ```powershell  
      $cred = Get-Credential
    
      $session = New-PSSession -ComputerName azs-ercs01 `
        -ConfigurationName PrivilegedEndpoint -Credential $cred
      ```

   入力を求められたら、次の資格情報を使用します。

   - **ユーザー名**: **&lt;"*Azure Stack Hub ドメイン*"&gt;\cloudadmin** の形式で CloudAdmin アカウントを指定します。 (ASDK の場合、ユーザー名は **azurestack\cloudadmin** です。)
      
   - **パスワード**:インストール中に AzureStackAdmin ドメイン管理者アカウントのパスワードとして指定したものと同じパスワードを入力します。

1. ローカル コンピューターに PEP セッションをインポートします。

   ```powershell 
   Import-PSSession $session
   ```

1. これで、Azure Stack Hub のセキュリティの状態を低下させることなく、PEP のすべての関数およびコマンドレットと共に、ローカルの PowerShell セッションで通常どおりにタブ補完を使用し、スクリプトを実行できるようになりました。 機能を有効にご活用ください。


## <a name="close-the-privileged-endpoint-session"></a>特権エンドポイント セッションを閉じる

前述のとおり、PEP では、PowerShell セッションで実行するすべてのアクション (および、それに対応する出力) がログに記録されます。 `Close-PrivilegedEndpoint` コマンドレットを使ってセッションを閉じる必要があります。 このコマンドレットは、エンドポイントを正しく閉じて、ログ ファイルを保管用の外部ファイル共有に転送します。

エンドポイント セッションを閉じるには:

1. PEP からアクセス可能な外部ファイル共有を作成します。 開発キット環境では、開発キットのホスト上にファイル共有を作成することができます。
1. 次のコマンドレットを実行します。

   ```powershell  
   Close-PrivilegedEndpoint -TranscriptsPathDestination "\\fileshareIP\SharedFolder" -Credential Get-Credential
   ```

   このコマンドレットでは次の表のパラメーターを使用します。

   | パラメーター | 説明 | Type | 必須 |
   |---------|---------|---------|---------|
   | *TranscriptsPathDestination* | "fileshareIP\sharefoldername" として定義されている外部ファイル共有へのパス | String | はい|
   | *資格情報* | ファイル共有にアクセスするための資格情報 | SecureString |   はい |


トランスクリプト ログ ファイルがファイル共有に正常に転送された後、それらのファイルは PEP から自動的に削除されます。 

> [!NOTE]
> コマンドレット `Exit-PSSession` または `Exit` を使用して PEP セッションを閉じた、または単に PowerShell コンソールを閉じた場合、トランスクリプト ログはファイル共有に転送されません。 それらは PEP に残ります。 次に `Close-PrivilegedEndpoint` を実行してファイル共有をインクルードしたときに、以前のセッションのトランスクリプト ログも併せて転送されます。 `Exit-PSSession` または `Exit` を使って PEP セッションを閉じないでください。代わりに、`Close-PrivilegedEndpoint` を使ってください。

## <a name="unlocking-the-privileged-endpoint-for-support-scenarios"></a>サポート シナリオでの特権エンドポイントのロック解除

サポート シナリオでは、Microsoft サポート エンジニアが特権エンドポイント PowerShell セッションを昇格させて、Azure Stack Hub インフラストラクチャの内部にアクセスすることが必要になる場合があります。 このプロセスは、"ガラスを割る" または "PEP のロック解除" と呼ばれることもあります。 PEP セッションの昇格プロセスは、2 つの手順、2 人のユーザー、2 つの組織認証プロセスで構成されます。 ロック解除の手順は Azure Stack Hub オペレーターが開始し、オペレーターは常に環境の制御を保持します。 オペレーターは PEP にアクセスし、次のコマンドレットを実行します。
 
 ```powershell  
      Get-SupportSessionToken
 ```

コマンドレットにより、サポート セッション要求トークン (非常に長い英数字の文字列) が返されます。 オペレーターは、この要求トークンを任意のメディア (チャット、電子メールなど) から Microsoft サポート エンジニアに渡します。 Microsoft サポート エンジニアは、要求トークンを使用して、サポート セッション認証トークンを生成し (有効な場合)、それを Azure Stack Hub オペレーターに送り返します。 同じ PEP PowerShell セッションで、オペレーターは次のコマンドレットに入力として認証トークンを渡します。

```powershell  
      unlock-supportsession
      cmdlet Unlock-SupportSession at command pipeline position 1
      Supply values for the following parameters:
      ResponseToken:
 ```

認証トークンが有効な場合、PEP PowerShell セッションは、完全な管理機能とインフラストラクチャへの完全な到達可能性を提供することによって昇格されます。 

> [!NOTE]
> 管理者特権の PEP セッションで実行されるすべての操作とコマンドレットは、Microsoft サポート エンジニアの厳格な監督下で実行する必要があります。 そうしないと、深刻なダウンタイムが発生し、データが失われ、Azure Stack Hub 環境の完全な再デプロイが必要になる可能性があります。

サポート セッションが終了したら、前のセクションで説明したように、**PrivilegedEndpoint** コマンドレットを使用して管理者特権の PEP セッションを再度閉じることが非常に重要になります。 PEP セッションが終了すると、ロック解除トークンが無効になるため、これを再利用して PEP セッションのロックを解除することはできません。
管理者特権の PEP セッションの有効期間は 8 時間ですが、その後も終了しない場合、管理者特権の PEP セッションは通常の PEP セッションへと自動的に再ロックされます。

## <a name="content-of-the-privileged-endpoint-tokens"></a>特権エンドポイント トークンの内容

PEP サポート セッション要求および認証トークンにより、暗号化を利用してアクセスが保護され、許可されたトークンによってのみ PEP セッションのロックを解除できるようになります。 トークンは、要求トークンを生成した PEP セッションでのみ応答トークンを受け入れることができるように設計されています。 PEP トークンには、Azure Stack Hub 環境または顧客を一意に識別できるような情報は含まれていません。 完全に匿名です。 各トークンの内容の詳細を以下に示します。
 
### <a name="support-session-request-token"></a>サポート セッション要求トークン

PEP サポート セッション要求トークンは、次の 3 つのオブジェクトで構成されています。

- ランダムに生成されたセッション ID。
- 1 回限りの公開キーと秘密キーの組み合わせを与える目的で生成された自己署名証明書。 この証明書には、環境に関する情報は一切含まれていません。
- 要求トークンの有効期限を示すタイム スタンプ。

要求トークンは、Azure Stack Hub 環境が登録されている Azure クラウドの公開キーで暗号化されます。
 
### <a name="support-session-authorization-response-token"></a>サポート セッション認証応答トークン

PEP サポート認証応答トークンは、次の 2 つのオブジェクトで構成されています。

- 要求トークンから抽出された、ランダム生成セッション ID。
- 応答トークンの有効期限を示すタイム スタンプ。
      
応答トークンは、要求トークンに含まれている自己署名証明書を使用して暗号化されます。 自己署名証明書は、Azure Stack Hub 環境が登録されている Azure クラウドに関連付けられた秘密キーで暗号化解除されました。


## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub の診断ツール](./diagnostic-log-collection.md)
- [Azure Stack Hub の特権エンドポイント リファレンス](../reference/pep-2002/index.md)