---
title: Windows PowerShell を使用してネットワーク コントローラーを展開する
description: Windows PowerShell を使用してネットワーク コントローラーを展開する方法について説明します。
author: v-dasis
ms.topic: how-to
ms.date: 09/22/2020
ms.author: v-dasis
ms.reviewer: JasonGerend
ms.openlocfilehash: e217c8b3e2a67dafa121fe752b66af9f24f888a1
ms.sourcegitcommit: 362081a8c19e7674c3029c8a44d7ddbe2deb247b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/09/2020
ms.locfileid: "91899552"
---
# <a name="deploy-network-controller-using-windows-powershell"></a>Windows PowerShell を使用してネットワーク コントローラーを展開する

> 適用対象: Azure Stack HCI バージョン 20H2、Windows Server 2019

このトピックでは、Windows PowerShell を使用して、Azure Stack HCI クラスターで実行されている 1 つ以上の仮想マシン (VM) にネットワーク コントローラーを展開する方法について説明します。 ネットワーク コントローラーは、ソフトウェアによるネットワーク制御 (SDN) のコンポーネントです。

>[!NOTE]
>Windows Admin Center でクラスターの作成ウィザードを使用して、ネットワーク コントローラーを展開することもできます。 詳細については、「[Windows Admin Center を使用して Azure Stack HCI クラスターを作成する](create-cluster.md)」を参照してください。

## <a name="using-windows-powershell"></a>Windows PowerShell を使用する

PowerShell は、ホスト サーバー上のリモート デスクトップ (RDP) セッションでローカルで実行することも、管理コンピューターからリモートで実行することもできます。

管理コンピューターから PowerShell を実行する場合は、管理しているサーバーまたはクラスターの名前を指定した `-Name` パラメーターまたは `-Cluster` パラメーターを含めます。 また、サーバーの `-ComputerName` パラメーターを使用するときに、完全修飾ドメイン名 (FQDN) を指定することが必要な場合があります。

また、Hyper-V およびフェールオーバー クラスタリング用の、リモート サーバー管理ツール (RSAT) コマンドレットと PowerShell モジュールも必要になります。 管理コンピューターの PowerShell セッションでこれらをまだ使用できない場合は、次のコマンドを使用して追加できます: `Add-WindowsFeature RSAT-Clustering-PowerShell`。

## <a name="install-the-network-controller-server-role"></a>ネットワーク コントローラー サーバー ロールのインストール

この手順を使用して、ネットワーク コントローラー サーバー ロールを仮想マシン (VM) にインストールします。

>[!IMPORTANT]
>ネットワーク コントローラー サーバー ロールを物理ホストに展開しないでください。 ネットワーク コントローラーを展開するには、Hyper-V ホストにインストールされている Hyper-V VM にネットワーク コントローラー サーバー ロールをインストールする必要があります。 3 つの異なる Hyper-V ホスト上の VM にネットワーク コントローラーをインストールした後、ホストをネットワーク コントローラーに追加して、ソフトウェアによるネットワーク制御 (SDN) に対して Hyper-V ホストを有効にする必要があります。 これにより、SDN ソフトウェア ロード バランサーを機能させることができます。

この手順を実行するには、**Administrators** グループのメンバーシップ、またはそれと同等のメンバーシップが必要です。  

>[!NOTE]
>Windows PowerShell の代わりにサーバー マネージャーを使用してネットワーク コントローラーをインストールする場合は、「[サーバー マネージャーを使用してネットワーク コントローラー サーバー ロールをインストールする](/windows-server/networking/sdn/technologies/network-controller/install-the-network-controller-server-role-using-server-manager)」を参照してください。

ネットワーク コントローラーをインストールするには、次のコマンドを入力します。

```powershell
Install-WindowsFeature -Name NetworkController -IncludeManagementTools
```

ネットワーク コントローラーのインストールでは、コンピューターを再起動する必要があります。 これを行うには、次のコマンドを入力します。

```powershell
Restart-Computer
```

## <a name="configure-the-network-controller-cluster"></a>ネットワーク コントローラー クラスターの構成

ネットワーク コントローラー クラスターにより、ネットワーク コントローラー アプリケーションでは高可用性とスケーラビリティが実現します。このアプリケーションは、クラスターの作成後に構成することができ、クラスター上でホストされます。

>[!NOTE]
>以下のセクションの手順は、ネットワーク コントローラーをインストールした VM で直接実行することも、Windows Admin Center を実行しているリモート コンピューターから実行することもできます。 また、この手順を実行するには、**Administrators** グループのメンバーシップ、またはそれと同等のメンバーシップが必要です。 ネットワーク コントローラーをインストールしたコンピューターまたは VM がドメインに参加している場合は、ユーザー アカウントが **Domain Users** グループのメンバーである必要があります。

ネットワーク コントローラー クラスターを作成するには、ノード オブジェクトを作成し、クラスターを構成します。

### <a name="create-a-node-object"></a>ノード オブジェクトを作成する

ネットワーク コントローラー クラスターのメンバーである VM ごとに、ノード オブジェクトを作成する必要があります。

ノード オブジェクトを作成するには、次のコマンドを入力します。 各パラメーターには、実際の展開に適した値を使用してください。  

```powershell
New-NetworkControllerNodeObject -Name <string> -Server "ServerName" -FaultDomain "SiteName" -RestInterface "Name" [-NodeCertificate <X509Certificate2>]
```

次の表に、`New-NetworkControllerNodeObject` コマンドの各パラメーターの説明を示します。

|パラメーター|説明|
|-------------|---------------|
|名前|**Name** パラメーターでは、クラスターに追加するサーバーのフレンドリ名を指定します。|
|サーバー|**Server** パラメーターでは、クラスターに追加するサーバーのホスト名、完全修飾ドメイン名 (FQDN)、または IP アドレスを指定します。 ドメインに参加しているコンピューターの場合、FQDN が必要です。|
|FaultDomain|**FaultDomain** パラメーターでは、クラスターに追加するサーバーの障害ドメインを指定します。 このパラメーターは、クラスターに追加するサーバーと同時に障害が発生する可能性があるサーバーを定義します。 この障害は、電源やネットワーク ソースなど、共有の物理的な依存関係が原因と考えられます。 通常、障害ドメインは、これらの共有の依存関係に関連する階層を表します。障害ドメイン ツリーの上位から、同時に障害が発生する可能性の高いサーバーが増えます。 実行時に、ネットワーク コントローラーはクラスター内の障害ドメインを考慮し、ネットワーク コントローラー サービスが別々の障害ドメインに配置されるように分散を試みます。 このプロセスは、1 つのフォールト ドメインでの障害発生時に、そのサービスの可用性と状態が損なわれないようにするために役立ちます。 障害ドメインは階層形式で指定されます。 次に例を示します。"Fd:/DC1/Rack1/Host1" の場合、DC1 はデータセンター名、Rack1 はラック名、Host1 はノードが配置されているホストの名前です。|
|RestInterface|**RestInterface** パラメーターでは、Representational State Transfer (REST) 通信が終了するノード上のインターフェイスの名前を指定します。 このネットワーク コントローラー インターフェイスは、ネットワークの管理レイヤーから Northbound API 要求を受信します。|
|NodeCertificate|**NodeCertificate** パラメーターでは、ネットワーク コントローラーがコンピューターの認証に使用する証明書を指定します。 クラスター内の通信に証明書ベースの認証を使用する場合は証明書が必要です。また、証明書は、ネットワーク コントローラー サービス間のトラフィックの暗号化にも使用されます。 証明書のサブジェクト名は、ノードの DNS 名と同じである必要があります。|

### <a name="configure-the-cluster"></a>クラスターを構成する

クラスターを構成するには、次のコマンドを入力します。 各パラメーターには、実際の展開に適した値を使用してください。

```powershell
Install-NetworkControllerCluster -Node "NetworkControllerNodeName" -ClusterAuthentication "ClusterAuthenticationType" [-ManagementSecurityGroup <string>][-DiagnosticLogLocation <string>][-LogLocationCredential <PSCredential>] [-CredentialEncryptionCertificate <X509Certificate2>][-Credential <PSCredential>][-CertificateThumbprint <String>] [-UseSSL][-ComputerName <string>][-LogSizeLimitInMBs<UInt32>] [-LogTimeLimitInDays<UInt32>]
```

次の表に、`Install-NetworkControllerCluster` コマンドの各パラメーターの説明を示します。
  
|パラメーター|説明|
|-------------|---------------|
|ClusterAuthentication|**ClusterAuthentication** パラメーターでは、ノード間の通信をセキュリティで保護するために使用され、ネットワーク コントローラー サービス間のトラフィックの暗号化にも使用される認証の種類を指定します。 サポートされる値は、**Kerberos**、**X509**、**None** です。 Kerberos 認証はドメイン アカウントを使用し、ネットワーク コントローラー ノードがドメインに参加している場合にのみ使用できます。 X509 ベースの認証を指定する場合は、NetworkControllerNode オブジェクトで証明書を指定する必要があります。 さらに、このコマンドを実行する前に、証明書を手動でプロビジョニングする必要があります。|
|ManagementSecurityGroup|**ManagementSecurityGroup** パラメーターでは、リモート コンピューターから管理コマンドレットを実行することが許可されているユーザーを含むセキュリティ グループの名前を指定します。 これは、ClusterAuthentication が Kerberos の場合にのみ適用されます。 ローカル コンピューターのセキュリティ グループではなく、ドメイン セキュリティ グループを指定する必要があります。|
|Node|**Node** パラメーターでは、**New-NetworkControllerNodeObject** コマンドを使用して作成したネットワーク コントローラー ノードのリストを指定します。|
|DiagnosticLogLocation|**DiagnosticLogLocation** パラメーターでは、診断ログが定期的にアップロードされる共有の場所を指定します。 このパラメーターの値を指定しない場合、ログは各ノードでローカルに保存されます。 ログは、ローカルの %systemdrive%\Windows\tracing\SDNDiagnostics フォルダーに保存されます。 クラスター ログは、ローカルの %systemdrive%\ProgramData\Microsoft\Service Fabric\log\Traces フォルダーに保存されます。|
|LogLocationCredential|**LogLocationCredential** パラメーターでは、ログが保存される共有の場所にアクセスするために必要な資格情報を指定します。|
|CredentialEncryptionCertificate|**CredentialEncryptionCertificate** パラメーターでは、ネットワーク コントローラーのバイナリへのアクセスに使用される資格情報と **LogLocationCredential** (指定されている場合) を暗号化するためにネットワーク コントローラーで使用される証明書を指定します。 このコマンドを実行する前に、すべてのネットワーク コントローラー ノードで証明書をプロビジョニングする必要があります。また、同じ証明書をすべてのクラスター ノードに登録する必要があります。 運用環境では、このパラメーターを使用してネットワーク コントローラーのバイナリとログを保護することをお勧めします。 このパラメーターを使用しない場合、資格情報はクリア テキストで保存されるため、未承認ユーザーによって悪用される可能性があります。|
|資格情報|このパラメーターは、リモート コンピューターからこのコマンドを実行する場合にのみ必要です。 **Credential** パラメーターでは、ターゲット コンピューターでこのコマンドを実行する権限を持つユーザー アカウントを指定します。|
|CertificateThumbprint|このパラメーターは、リモート コンピューターからこのコマンドを実行する場合にのみ必要です。 **CertificateThumbprint** パラメーターでは、ターゲット コンピューターでこのコマンドを実行する権限を持つユーザー アカウントのデジタル公開キー証明書 (X509) を指定します。|
|UseSSL|このパラメーターは、リモート コンピューターからこのコマンドを実行する場合にのみ必要です。 **UseSSL** パラメーターでは、リモート コンピューターへの接続を確立するために使用される Secure Sockets Layer (SSL) プロトコルを指定します。 既定では、SSL は使用されません。|
|[ComputerName]|**ComputerName** パラメーターでは、このコマンドが実行されるネットワーク コントローラー ノードを指定します。 このパラメーターに値を指定しないと、ローカル コンピューターが既定で使用されます。|
|LogSizeLimitInMBs|このパラメーターでは、ネットワーク コントローラーが保存できる最大ログ サイズ (MB) を指定します。 ログは循環的に保存されます。 DiagnosticLogLocation が指定されている場合、このパラメーターの既定値は 40 GB です。 DiagnosticLogLocation が指定されていない場合、ログはネットワーク コントローラー ノードに保存され、このパラメーターの既定値は 15 GB になります。|
|LogTimeLimitInDays|このパラメーターでは、ログの保存期間の上限 (日数) を指定します。 ログは循環的に保存されます。 このパラメーターの既定値は 3 日です。|

## <a name="configure-the-network-controller-application"></a>ネットワーク コントローラー アプリケーションの構成

ネットワーク コントローラー アプリケーションを構成するには、次のコマンドを入力します。 各パラメーターには、実際の展開に適した値を使用してください。

```powershell
Install-NetworkController -Node <NetworkControllerNode[]> -ClientAuthentication <ClientAuthentication>  [-ClientCertificateThumbprint <string[]>]  [-ClientSecurityGroup <string>] -ServerCertificate <X509Certificate2> [-RESTIPAddress <String>] [-RESTName <String>] [-Credential <PSCredential>][-CertificateThumbprint <String> ] [-UseSSL]
```

次の表に、`Install-NetworkController` コマンドの各パラメーターの説明を示します。

|パラメーター|説明|
|-------------|---------------|
|ClientAuthentication|**ClientAuthentication** パラメーターでは、REST とネットワーク コントローラー間の通信をセキュリティで保護するために使用される認証の種類を指定します。 サポートされる値は、**Kerberos**、**X509**、**None** です。 Kerberos 認証はドメイン アカウントを使用し、ネットワーク コントローラー ノードがドメインに参加している場合にのみ使用できます。 X509 ベースの認証を指定する場合は、NetworkControllerNode オブジェクトで証明書を指定する必要があります。 さらに、このコマンドを実行する前に、証明書を手動でプロビジョニングする必要があります。|
|Node|**Node** パラメーターでは、**New-NetworkControllerNodeObject** コマンドを使用して作成したネットワーク コントローラー ノードのリストを指定します。|
|ClientCertificateThumbprint|このパラメーターは、ネットワーク コントローラー クライアントに証明書ベースの認証を使用している場合にのみ必要です。 **ClientCertificateThumbprint** パラメーターでは、Northbound レイヤーのクライアントに登録されている証明書の拇印を指定します。|
|ServerCertificate|**ServerCertificate** パラメーターでは、ネットワーク コントローラーがその ID をクライアントに対して証明するために使用する証明書を指定します。 サーバー証明書は、拡張キー使用法拡張機能にサーバー認証の目的を含める必要があります。また、クライアントによって信頼されている CA によってネットワーク コントローラーに発行される必要があります。|
|RESTIPAddress|ネットワーク コントローラーの単一ノードの展開では、**RESTIPAddress** の値を指定する必要はありません。 複数ノードの展開の場合、**RESTIPAddress** パラメーターでは、CIDR 表記で REST エンドポイントの IP アドレスを指定します。 たとえば、192.168.1.10/24 とします。 **ServerCertificate** の Subject Name 値は、**RESTIPAddress** パラメーターの値に解決される必要があります。 すべてのノードが同じサブネット上にある場合は、ネットワーク コントローラーのすべての複数ノードの展開にこのパラメーターを指定する必要があります。 ノードがさまざまなサブネット上にある場合は、**RESTIPAddress** を使用するのではなく、**RestName** パラメーターを使用する必要があります。|
|RestName|ネットワーク コントローラーの単一ノードの展開では、**RestName** の値を指定する必要はありません。 **RestName** の値を指定する必要があるのは、複数ノードの展開に含まれるノードが異なるサブネット上にある場合だけです。 複数ノードの展開の場合、**RestName** パラメーターでは、ネットワーク コントローラー クラスターの FQDN を指定します。|
|ClientSecurityGroup|**ClientSecurityGroup** パラメーターでは、メンバーがネットワーク コントローラー クライアントである Active Directory セキュリティ グループの名前を指定します。 このパラメーターは、**ClientAuthentication** に Kerberos 認証を使用する場合にのみ必要です。 このセキュリティ グループには、REST API へのアクセスに使用するアカウントが含まれている必要があります。また、このコマンドを実行する前に、セキュリティ グループを作成し、メンバーを追加する必要があります。|
|資格情報|このパラメーターは、リモート コンピューターからこのコマンドを実行する場合にのみ必要です。 **Credential** パラメーターでは、ターゲット コンピューターでこのコマンドを実行する権限を持つユーザー アカウントを指定します。|
|CertificateThumbprint|このパラメーターは、リモート コンピューターからこのコマンドを実行する場合にのみ必要です。 **CertificateThumbprint** パラメーターでは、ターゲット コンピューターでこのコマンドを実行する権限を持つユーザー アカウントのデジタル公開キー証明書 (X509) を指定します。|
|UseSSL|このパラメーターは、リモート コンピューターからこのコマンドを実行する場合にのみ必要です。 **UseSSL** パラメーターでは、リモート コンピューターへの接続を確立するために使用される Secure Sockets Layer (SSL) プロトコルを指定します。 既定では、SSL は使用されません。|

ネットワーク コントローラー アプリケーションの構成が完了したら、ネットワーク コントローラーの展開が完了します。

## <a name="network-controller-deployment-validation"></a>ネットワーク コントローラーの展開の検証

ネットワーク コントローラーの展開を検証するには、ネットワーク コントローラーに資格情報を追加し、その資格情報を取得します。

`ClientAuthentication` の種類として Kerberos を使用している場合、この手順を実行するには、作成した **ClientSecurityGroup** グループのメンバーシップが必要です。

1. クライアント コンピューターで、`ClientAuthentication` の種類として Kerberos を使用している場合は、**ClientSecurityGroup** グループのメンバーであるユーザー アカウントでログオンします。

1. PowerShell で次のコマンドを入力します。 各パラメーターには、実際の展開に適した値を使用してください。

    ```powershell
    $cred=New-Object Microsoft.Windows.Networkcontroller.credentialproperties
    $cred.type="usernamepassword"
    $cred.username="admin"
    $cred.value="abcd"

    New-NetworkControllerCredential -ConnectionUri "https://networkcontroller"-Properties $cred -ResourceId "cred1"
    ```

1. ネットワーク コントローラーに追加した資格情報を取得するには、次のコマンドを入力します。 各パラメーターには、実際の展開に適した値を使用してください。

    ```powershell
    Get-NetworkControllerCredential -ConnectionUri https://networkcontroller -ResourceId cred1  
    ```

1. コマンドの出力を確認します。次の出力例のようになります。

    ```powershell
    Tags                   :
    ResourceRef     : /credentials/cred1
    CreatedTime    : 1/1/0001 12:00:00 AM
    InstanceId        : e16ffe62-a701-4d31-915e-7234d4bc5a18
    Etag                  : W/"1ec59631-607f-4d3e-ac78-94b0822f3a9d"
    ResourceMetadata :
    ResourceId       : cred1
    Properties       : Microsoft.Windows.NetworkController.CredentialProperties
    ```

    > [!NOTE]
    > `Get-NetworkControllerCredential` コマンドを実行する場合、ドット演算子を使用して資格情報のプロパティを表示することで、コマンドの出力を変数に割り当てることができます (例: `$cred.Properties`)。

## <a name="additional-powershell-commands-for-network-controller"></a>ネットワーク コントローラーに関するその他の PowerShell コマンド

ネットワーク コントローラーを展開したら、PowerShell コマンドを使用して展開を管理および変更できます。 展開に加えることができる変更の一部を次に示します。

- ネットワーク コントローラー ノード、クラスター、およびアプリケーション設定を変更する

- ネットワーク コントローラー クラスターおよびアプリケーションを削除する

- ネットワーク コントローラー クラスター ノードを管理する (ノードの追加、削除、有効化、無効化など)

次の表に、これらのタスクを実行するために使用できる PowerShell コマンドの構文を示します。

|タスク|command|構文|
|--------|-------|----------|
|ネットワーク コントローラー クラスターの設定を変更する|Set-NetworkControllerCluster|`Set-NetworkControllerCluster [-ManagementSecurityGroup <string>][-Credential <PSCredential>] [-computerName <string>][-CertificateThumbprint <String> ] [-UseSSL]`
|ネットワーク コントローラー アプリケーションの設定を変更する|Set-NetworkController|`Set-NetworkController [-ClientAuthentication <ClientAuthentication>] [-Credential <PSCredential>] [-ClientCertificateThumbprint <string[]>] [-ClientSecurityGroup <string>] [-ServerCertificate <X509Certificate2>] [-RestIPAddress <String>] [-ComputerName <String>][-CertificateThumbprint <String> ] [-UseSSL]`
|ネットワーク コントローラー ノードの設定を変更する|Set-NetworkControllerNode|`Set-NetworkControllerNode -Name <string> > [-RestInterface <string>] [-NodeCertificate <X509Certificate2>] [-Credential <PSCredential>] [-ComputerName <string>][-CertificateThumbprint <String> ] [-UseSSL]`
|ネットワーク コントローラーの診断設定を変更する|Set-NetworkControllerDiagnostic|`Set-NetworkControllerDiagnostic [-LogScope <string>] [-DiagnosticLogLocation <string>] [-LogLocationCredential <PSCredential>] [-UseLocalLogLocation] >] [-LogLevel <loglevel>][-LogSizeLimitInMBs <uint32>] [-LogTimeLimitInDays <uint32>] [-Credential <PSCredential>] [-ComputerName <string>][-CertificateThumbprint <String> ] [-UseSSL]`
|ネットワーク コントローラー アプリケーションを削除する|Uninstall-NetworkController|`Uninstall-NetworkController [-Credential <PSCredential>][-ComputerName <string>] [-CertificateThumbprint <String> ] [-UseSSL]`
|ネットワーク コントローラー クラスターを削除する|Uninstall-NetworkControllerCluster|`Uninstall-NetworkControllerCluster [-Credential <PSCredential>][-ComputerName <string>][-CertificateThumbprint <String> ] [-UseSSL]`
|ネットワーク コントローラー クラスターにノードを追加する|Add-NetworkControllerNode|`Add-NetworkControllerNode -FaultDomain <String> -Name <String> -RestInterface <String> -Server <String> [-CertificateThumbprint <String> ] [-ComputerName <String> ] [-Credential <PSCredential> ] [-Force] [-NodeCertificate <X509Certificate2> ] [-PassThru] [-UseSsl]`
|ネットワーク コントローラー クラスター ノードを無効にする|Disable-NetworkControllerNode|`Disable-NetworkControllerNode -Name <String> [-CertificateThumbprint <String> ] [-ComputerName <String> ] [-Credential <PSCredential> ] [-PassThru] [-UseSsl]`
|ネットワーク コントローラー クラスター ノードを有効にする|Enable-NetworkControllerNode|`Enable-NetworkControllerNode -Name <String> [-CertificateThumbprint <String> ] [-ComputerName <String> ] [-Credential <PSCredential> ] [-PassThru] [-UseSsl]`
|ネットワーク コントローラー ノードをクラスターから削除する|Remove-NetworkControllerNode|`Remove-NetworkControllerNode [-CertificateThumbprint <String> ] [-ComputerName <String> ] [-Credential <PSCredential> ] [-Force] [-Name <String> ] [-PassThru] [-UseSsl]`

詳細については、「[NetworkController](/powershell/module/networkcontroller/?view=win10-ps)」のネットワーク コントローラーに関する Windows PowerShell リファレンス ドキュメントを参照してください。

## <a name="sample-network-controller-configuration-script"></a>ネットワーク コントローラーのサンプル構成スクリプト

次のサンプル構成スクリプトは、マルチノード ネットワーク コントローラー クラスターを作成し、ネットワーク コントローラー アプリケーションをインストールする方法を示しています。 また、`$cert` 変数は、サブジェクト名の文字列 "networkController.contoso.com" と一致するローカル コンピューター証明書ストアから証明書を選択します。

```powershell
$a = New-NetworkControllerNodeObject -Name "Node1" -Server "NCNode1.contoso.com" -FaultDomain "fd:/rack1/host1" -RestInterface Internal
$b = New-NetworkControllerNodeObject -Name "Node2" -Server "NCNode2.contoso.com" -FaultDomain "fd:/rack1/host2" -RestInterface Internal
$c = New-NetworkControllerNodeObject -Name "Node3" -Server "NCNode3.contoso.com" -FaultDomain "fd:/rack1/host3" -RestInterface Internal

$cert= get-item Cert:\LocalMachine\My | get-ChildItem | where {$_.Subject -imatch "networkController.contoso.com" }

Install-NetworkControllerCluster -Node @($a,$b,$c)  -ClusterAuthentication Kerberos -DiagnosticLogLocation \\share\Diagnostics - ManagementSecurityGroup Contoso\NCManagementAdmins -CredentialEncryptionCertificate $cert  
Install-NetworkController -Node @($a,$b,$c) -ClientAuthentication Kerberos -ClientSecurityGroup Contoso\NCRESTClients -ServerCertificate $cert -RestIpAddress 10.0.0.1/24
```

## <a name="next-steps"></a>次のステップ

ネットワーク コントローラーの展開で Kerberos を使用していない場合は、証明書を展開する必要があります。 詳細については、「[ネットワーク コントローラーの展開後の手順](/windows-server/networking/sdn/technologies/network-controller/post-deploy-steps-nc)」を参照してください。