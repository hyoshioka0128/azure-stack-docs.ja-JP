---
title: Azure Stack Hub データセンターの DNS の統合
description: Azure Stack Hub の DNS をデータセンターの DNS と統合する方法を学習します。
author: IngridAtMicrosoft
ms.topic: article
ms.date: 04/10/2020
ms.author: inhenkel
ms.reviewer: wfayed
ms.lastreviewed: 08/21/2019
ms.openlocfilehash: be6ea1e8dbf8b17e02a4117f5f2d20cb9cfbbcfe
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86488724"
---
# <a name="azure-stack-hub-datacenter-dns-integration"></a>Azure Stack Hub データセンターの DNS の統合

Azure Stack Hub の外部から、**portal**、**adminportal**、**management**、**adminmanagement** などの Azure Stack Hub エンドポイントにアクセスできるようにするには、Azure Stack Hub DNS サービスを、Azure Stack Hub で使用したい DNS ゾーンをホストする DNS サーバーと統合する必要があります。

## <a name="azure-stack-hub-dns-namespace"></a>Azure Stack Hub の DNS 名前空間

Azure Stack Hub をデプロイするときに、DNS に関するいくつかの重要な情報を指定する必要があります。


|フィールド  |説明  |例|
|---------|---------|---------|
|リージョン|Azure Stack Hub のデプロイの地理的な場所。|`east`|
|外部ドメイン名|Azure Stack Hub のデプロイに使用したいゾーンの名前。|`cloud.fabrikam.com`|
|内部ドメイン名|Azure Stack Hub でのインフラストラクチャ サービスに使用される内部ゾーンの名前。 ディレクトリ サービスと統合されており、プライベートになっています (Azure Stack Hub デプロイの外部からは到達できません)。|`azurestack.local`|
|DNS フォワーダー|Azure Stack Hub の外部 (企業イントラネットまたはパブリック インターネット上のどちらか) でホストされている DNS クエリ、DNS ゾーンおよびレコードを転送するために使用される DNS サーバー。 デプロイ後に [**Set-AzSDnsForwarder** コマンドレット](#editing-dns-forwarder-ips) を使用して、DNS フォワーダーの値を編集できます。 
|名前付けのプレフィックス (省略可能)|Azure Stack Hub インフラストラクチャ ロール インスタンス マシン名に使用する、名前付けのプレフィックス。  指定されていない場合、既定値は`azs` です。|`azs`|

Azure Stack Hub のデプロイの完全修飾ドメイン名 (FQDN) とエンドポイントは、リージョン パラメーターと外部ドメイン名パラメーターの組み合わせです。 前の表に示した例の値を使用すると、この Azure Stack Hub のデプロイの FQDN は次の名前のようになります。

`east.cloud.fabrikam.com`

同様に、このデプロイのエンドポイントは次の URL のようになります。

`https://portal.east.cloud.fabrikam.com`

`https://adminportal.east.cloud.fabrikam.com`

この例の DNS 名前空間を Azure Stack Hub のデプロイに使用するには、次の条件が必要です。

- 使用している名前解決の要件に応じて、ゾーン `fabrikam.com` がドメイン レジストラーまたは社内の DNS サーバー (あるいはその両方) に登録されている。
- 子ドメイン `cloud.fabrikam.com` がゾーン`fabrikam.com` の下に存在する。
- ゾーン `fabrikam.com` および `cloud.fabrikam.com` をホストする DNS サーバーに、Azure Stack Hub のデプロイから到達できる。

Azure Stack Hub の外部から Azure Stack Hub エンドポイントおよびインスタンスの DNS 名を解決できるようにするには、Azure Stack Hub の外部 DNS ゾーンをホストする DNS サーバーと、使用したい親ゾーンをホストする DNS サーバーを統合する必要があります。

### <a name="dns-name-labels"></a>DNS 名ラベル

Azure Stack Hub では、パブリック IP アドレスへの DNS 名ラベルの追加がサポートされており、パブリック IP アドレスの名前解決が可能です。 DNS ラベルは、Azure Stack Hub でホストされているアプリとサービスにユーザーが名前でアクセスできる便利な方法です。 DNS 名ラベルでは、インフラストラクチャ エンドポイントとはわずかに異なる名前空間が使用されます。 前のサンプル名前空間に続いて、DNS 名ラベルの名前空間が次のように表示されます。

`*.east.cloudapp.cloud.fabrikam.com`

そのため、テナントがパブリック IP アドレス リソースの DNS 名ラベル フィールドで値 **Myapp** を示す場合、Azure Stack Hub の外部 DNS サーバー上にあるゾーン **east.cloudapp.cloud.fabrikam.com** で **myapp** の A レコードが作成されます。 結果として得られる完全修飾ドメイン名は次のようになります。

`myapp.east.cloudapp.cloud.fabrikam.com`

この機能を利用し、この名前空間を使用したい場合は、Azure Stack Hub 用の外部 DNS ゾーンをホストする DNS サーバーに、使用したい親ゾーンをホストする DNS サーバーも統合する必要があります。 これは、Azure Stack Hub サービス エンドポイント用の名前空間とは別の名前空間です。そのため、追加の委任または条件付き転送ルールを作成する必要があります。

DNS 名ラベルのしくみの詳細については、「[Azure Stack Hub での DNS の使用](../user/azure-stack-dns.md)」を参照してください。

## <a name="resolution-and-delegation"></a>解決と委任

DNS サーバーには次の 2 種類があります。

- 権限のある DNS サーバーは、DNS ゾーンをホストします。 このサーバーは、これらのゾーン内のレコードに対する DNS クエリのみに応答します。
- 再帰 DNS サーバーでは、DNS ゾーンはホストされません。 このサーバーは、権限のある DNS サーバーを呼び出して必要なデータを収集することで、すべての DNS クエリに応答します。

Azure Stack Hub には、権限のある DNS サーバーと再帰 DNS サーバーの両方が含まれます。 再帰サーバーは、その Azure Stack Hub のデプロイの内部プライベート ゾーンと外部パブリック DNS ゾーンを除くすべての名前の解決に使用されます。

![Azure Stack Hub DNS のアーキテクチャ](media/azure-stack-integrate-dns/Integrate-DNS-01.svg)

## <a name="resolving-external-dns-names-from-azure-stack-hub"></a>Azure Stack Hub からの外部 DNS 名の解決

Azure Stack Hub 外部のエンドポイントの DNS 名 (たとえば、www\.bing.com) を解決するには、Azure Stack Hub に権限のない DNS 要求を転送するために、その Azure Stack Hub で使用できる DNS サーバーを提供する必要があります。 デプロイでは、Azure Stack Hub の要求の転送先となる DNS サーバーをデプロイ ワークシート ([DNS フォワーダー] フィールド) に入力する必要があります。 フォールト トレランスのために、このフィールドには 2 つ以上のサーバーを入力します。 これらの値がない場合、Azure Stack Hub のデプロイは失敗します。 デプロイ後に [**Set-AzSDnsForwarder** コマンドレット](#editing-dns-forwarder-ips) を使用して、DNS フォワーダーの値を編集できます。 

### <a name="configure-conditional-dns-forwarding"></a>条件付き DNS フォワーダーの構成

> [!IMPORTANT]
> この要件が該当するのは AD FS デプロイのみです。

既存の DNS インフラストラクチャ を使用して名前解決を有効にするには、条件付きフォワーダーを構成します。

条件付きフォワーダーを追加するには、特権エンドポイントを使用する必要が あります。

この手順では、データセンター ネットワーク内の、Azure Stack Hub の特権エンドポイントと通信できるコンピューターを使用します。

1. 管理者特権での Windows PowerShell セッション (管理者として実行) を開き、特権エンドポイントの IP アドレスに接続します。 CloudAdmin 認証の資格情報を使用します。

   ```PowerShell
   $cred=Get-Credential 
   Enter-PSSession -ComputerName <IP Address of ERCS> -ConfigurationName PrivilegedEndpoint -Credential $cred
   ```

2. 特権エンドポイントに接続したら、次の PowerShell コマンドを実行します。 サンプルの値を、使用する DNS サーバーのドメイン名とアドレスで置き換えてください。

   ```PowerShell
   Register-CustomDnsServer -CustomDomainName "contoso.com" -CustomDnsIPAddresses "192.168.1.1","192.168.1.2"
   ```

## <a name="resolving-azure-stack-hub-dns-names-from-outside-azure-stack-hub"></a>Azure Stack Hub の外部からの DNS 名の解決
権限のあるサーバーは、外部の DNS ゾーンとユーザーが作成したゾーンの情報を保持しています。 これらのサーバーと統合することで、ゾーンの委任または条件付き転送で、Azure Stack Hub の外部から Azure Stack Hub DNS 名を解決できるようになります。

## <a name="get-dns-server-external-endpoint-information"></a>DNS サーバーの外部エンドポイント情報の取得

Azure Stack Hub のデプロイを DNS インフラストラクチャと統合するには、次の情報が必要です。

- DNS サーバーの FQDN
- DNS サーバーの IP アドレス

Azure Stack Hub DNS サーバーの FQDN は、次の形式になります。

`<NAMINGPREFIX>-ns01.<REGION>.<EXTERNALDOMAINNAME>`

`<NAMINGPREFIX>-ns02.<REGION>.<EXTERNALDOMAINNAME>`

サンプルの値を使用すると、DNS サーバーの FQDN は次のようになります。

`azs-ns01.east.cloud.fabrikam.com`

`azs-ns02.east.cloud.fabrikam.com`


この情報は、すべての Azure Stack Hub のデプロイの最後に `AzureStackStampInformation.json` という名前のファイルにも作成されます。 このファイルは、デプロイ仮想マシンの `C:\CloudDeployment\logs` フォルダー内にあります。 Azure Stack Hub のデプロイに使用された値がわからない場合は、ここからその値を取得できます。

デプロイ仮想マシンが使用不可またはアクセス不可になっている場合は、特権エンドポイントに接続して `Get-AzureStackStampInformation` PowerShell コマンドレットを実行することで値を取得できます。 詳細については、[特権エンドポイント](azure-stack-privileged-endpoint.md)に関するページを参照してください。

## <a name="setting-up-conditional-forwarding-to-azure-stack-hub"></a>Azure Stack Hub への条件付き転送の設定

Azure Stack Hub と DNS インフラストラクチャを統合する最も簡単で安全な方法は、親ゾーンをホストするサーバーから、ゾーンの条件付き転送を行うことです。 Azure Stack Hub の外部 DNS 名前空間の親ゾーンをホストする DNS サーバーを直接制御できる場合は、この方法をお勧めします。

DNS で条件付き転送を行う方法がわからない場合は、次の TechNet の記事を参照してください。[ドメイン名の条件付きフォワーダーの割り当て](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc794735(v=ws.10))に関する記事、またはお使いの DNS ソリューションに固有のドキュメントをご覧ください。

会社のドメイン名の子ドメインのように見える Azure Stack Hub の外部 DNS ゾーンを指定しているシナリオでは、条件付き転送は使用できません。 DNS 委任を構成する必要があります。

例:

- 会社の DNS ドメイン名: `contoso.com`
- Azure Stack Hub の外部 DNS ドメイン名: `azurestack.contoso.com`

## <a name="editing-dns-forwarder-ips"></a>DNS フォワーダー IP の編集

DNS フォワーダー IP は、Azure Stack Hub のデプロイ中に設定されます。 ただし、フォワーダー IP を何らかの理由で更新する必要がある場合は、特権エンドポイントに接続し、PowerShell コマンドレットの `Get-AzSDnsForwarder` と `Set-AzSDnsForwarder [[-IPAddress] <IPAddress[]>]` を実行することで、それらの値を編集できます。 詳細については、[特権エンドポイント](azure-stack-privileged-endpoint.md)に関するページを参照してください。

## <a name="delegating-the-external-dns-zone-to-azure-stack-hub"></a>Azure Stack Hub への外部 DNS ゾーンの委任

DNS 名を Azure Stack Hub デプロイの外部から解決できるようにするには、DNS 委任を設定する必要があります。

各レジストラーは独自の DNS 管理ツールを所有していて、ドメインのネーム サーバー レコードを変更します。 レジストラーの DNS 管理ページで、NS レコードを編集し、ゾーンの NS レコードを、Azure Stack Hub のものに置き換えます。

ほとんどの DNS レジストラーでは、委任を実行するために 2 つ以上の DNS サーバーを指定する必要があります。

## <a name="next-steps"></a>次のステップ

[ファイアウォールの統合](azure-stack-firewall.md)
