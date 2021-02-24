---
title: Modular Datacenter のネットワーク統合
description: Azure Modular Datacenter の Azure Stack ネットワーク統合について説明します。
author: PatAltimore
ms.author: patricka
ms.service: azure-stack
ms.topic: conceptual
ms.date: 01/02/2020
ms.lastreviewed: 01/02/2020
ms.openlocfilehash: 09f24384710ddee8b987944a713c62112aef0264
ms.sourcegitcommit: 9b0e1264ef006d2009bb549f21010c672c49b9de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98255454"
---
# <a name="network-integration"></a>ネットワーク統合

この記事では、Azure Modular Datacenter の Azure Stack ネットワーク統合について説明します。

## <a name="border-connectivity-uplink"></a>境界接続 (アップリンク)

ネットワーク統合の計画は、Azure Stack 統合システムの展開、操作、および管理を正常に行うための重要な前提条件です。 境界接続の計画は、動的ルーティング (Border Gateway Protocol (BGP) による) と静的ルーティングのどちらを使用するか選択することから始まります。 動的ルーティングを使用するには、16 ビットの BGP 自律システム番号 (パブリックまたはプライベート) を割り当てる必要があります。 静的ルーティングには、境界デバイスに割り当てられた静的な既定のルートが使用されます。

エッジ スイッチでは、ポイントツーポイント IP (/30 ネットワーク) を持つレイヤー 3 アップリンクが物理インターフェイスに構成されている必要があります。 Azure Stack 操作をサポートするエッジ スイッチを持つレイヤー 2 アップリンクは、サポートされていません。

### <a name="bgp-routing"></a>BGP ルーティング

BGP のような動的ルーティング プロトコルを使用すると、システムが常にネットワークの変更を把握していることが保証され、管理が容易になります。 セキュリティを強化するために、エッジと境界間の BGP ピアリングにパスワードを設定できます。

次の図に示すように、トップオブラック (TOR) スイッチ上のプライベート IP 空間のアドバタイズは、プレフィックス一覧を使用してブロックされます。 このプレフィックス一覧により、プライベート ネットワークの広告が拒否され、TOR とエッジ間の接続でルート マップとして適用されます。

Azure Stack ソリューションの内部で実行されている ソフトウェア ロード バランサー (SLB) により、TOR デバイスをピアにして、VIP アドレスを動的にアドバタイズできるようになります。

ユーザー トラフィックが即時かつ透過的に障害から復旧できるようにするために、TOR デバイス間で構成された仮想プライベート クラウドまたはマルチシャーシ リンク アグリゲーション (MLAG) により、ホストと、IP ネットワークのネットワーク冗長性を提供する HSRP または VRRP に対して MLAG の使用が許可されます。

### <a name="static-routing"></a>静的ルーティング

静的ルーティングには、境界デバイスへの追加構成が必要です。 これにより、より多くの手動介入と管理が必要になり、変更を行う前に分析を行う必要もあります。 構成エラーによって発生した問題の場合、行われた変更によっては、ロールバックにより長い時間がかかる可能性があります。 このルーティング方法はお勧めしませんが、サポートされています。

静的ルーティングを使用してネットワーク環境に Azure Stack を統合するには、境界とエッジ デバイスの間の 4 つの物理的リンクすべてを接続する必要があります。 静的ルーティングの動作方法のため、高可用性は保証されません。

境界デバイスは、Azure Stack 内の任意のネットワーク宛てのトラフィックのエッジと境界の間に設定された 4 つのポイントツーポイント IP のそれぞれを指す静的ルートで構成する必要があります。 ただし、操作には、外部またはパブリック VIP ネットワークのみが必要です。 初期のデプロイには、BMC および外部ネットワークへの静的ルートが必要です。 オーケストレーターは、BMC およびインフラストラクチャ ネットワーク上に存在する管理リソースにアクセスするために境界内の静的ルートを残しておくことができます。 "スイッチ インフラストラクチャ" ネットワークと "スイッチ管理" ネットワークへの静的ルートの追加は省略可能です。

TOR デバイスには、すべてのトラフィックを境界デバイスに送信する既定の静的ルートが構成されています。 この既定のルールに対する 1 つのトラフィックの例外は、TOR から境界への接続に適用されたアクセス制御リストを使用してブロックされるプライベート空間の場合です。

静的ルーティングは、エッジと境界スイッチの間のアップリンクにのみ適用されます。 BGP 動的ルーティングは、ラックの内部で使用されます。これは SLB とその他のコンポーネントにとって不可欠なツールであり、無効にしたり削除したりすることはできないためです。

\* デプロイ後の BMC ネットワークは省略可能です。

\*\* スイッチ インフラストラクチャ ネットワークは、ネットワーク全体をスイッチ管理ネットワークに含めることができるため、省略可能です。

\*\*\* スイッチ管理ネットワークは必須であり、スイッチ インフラストラクチャ ネットワークとは別に追加できます。

### <a name="transparent-proxy"></a>透過的プロキシ

データセンターですべてのトラフィックがプロキシを使用する必要がある場合は、ラックからのすべてのトラフィックをポリシーに従って処理する透過プロキシを構成する必要があります。 トラフィックはネットワーク上のゾーン間で分離する必要があります。

Azure Stack ソリューションは、通常の Web プロキシをサポートしていません。

透過プロキシ (インターセプト、インライン、または強制プロキシとも呼ばれます) は、特殊なクライアント構成を必要とすることなく、通常の通信をネットワーク レイヤーでインターセプトします。 クライアントがプロキシの存在を意識する必要はありません。

SSL トラフィックのインターセプトはサポートされていないため、エンドポイントにアクセスするとサービス エラーが発生する場合があります。 ID に必要なエンドポイントとの通信に対してサポートされる最大タイムアウトは 60 秒で、再試行は 3 回です。

## <a name="dns"></a>DNS

このセクションでは、ドメイン ネーム システム (DNS) の構成について説明します。

### <a name="configure-conditional-dns-forwarding"></a>条件付き DNS フォワーダーの構成

このガイダンスは、Active Directory フェデレーション サービス (AD FS) のデプロイにのみ適用されます。

既存の DNS インフラストラクチャ を使用して名前解決を有効にするには、条件付きフォワーダーを構成します。

条件付きフォワーダーを追加するには、特権エンドポイントを使用する必要が あります。

この手順では、データセンター ネットワーク内の、Azure Stack の特権エンドポイントと通信できるコンピューターを使用します。

1. 管理者特権の Windows PowerShell セッション (管理者として実行) を開きます。 特権エンドポイントの IP アドレスに接続します。 CloudAdmin 認証の資格情報を使用します。

    ```powershell
    \$cred=Get-Credential Enter-PSSession -ComputerName \<IP Address of ERCS\> -ConfigurationName PrivilegedEndpoint -Credential \$cred 
    ```

1. 特権エンドポイントに接続したら、次の PowerShell コマンドを実行します。 サンプルの値を、使用する DNS サーバーのドメイン名とアドレスで置き換えてください。

    ```powershell
    Register-CustomDnsServer -CustomDomainName "contoso.com" -CustomDnsIPAddresses "192.168.1.1","192.168.1.2" 
    ```

### <a name="resolve-azure-stack-dns-names-from-outside-azure-stack"></a>Azure Stack 外部からの Azure Stack DNS 名を解決する

権限のあるサーバーは、外部の DNS ゾーンとユーザーが作成したゾーンの情報を保持しています。 このサーバーと統合することで、ゾーンの委任または条件付き転送を使用して Azure Stack 外部からの Azure Stack DNS 名を解決できるようになります。

### <a name="get-dns-server-external-endpoint-information"></a>DNS サーバーの外部エンドポイント情報の取得

Azure Stack のデプロイを DNS インフラストラクチャと統合するには、次の情報が必要です。

- DNS サーバーの完全修飾ドメイン名 (FQDN)
- DNS サーバーの IP アドレス

Azure Stack DNS サーバーの FQDN は次のような形式をとります。

- *\<NAMINGPREFIX\>-ns01.\<REGION\>.\<EXTERNALDOMAINNAME\>*
- *\<NAMINGPREFIX\>-ns02.\<REGION\>.\<EXTERNALDOMAINNAME\>*

サンプルの値を使用すると、DNS サーバーの FQDN は次のようになります。

- *azs-ns01.east.cloud.fabrikam.com*
- *azs-ns02.east.cloud.fabrikam.com*

この情報は管理ポータルで使用できますが、*AzureStackStampInformation.json* という名前のファイルで、すべての Azure Stack デプロイの最後にも作成されます。 このファイルは、デプロイ仮想マシンの *C:\\CloudDeployment\\logs* フォルダー内にあります。 Azure Stack のデプロイに使用された値がわからない場合は、ここから取得できます。

デプロイ仮想マシンが使用不可またはアクセス不可になっている場合は、特権エンドポイントに接続して *Get-AzureStackStampInformation* PowerShell コマンドレットを実行することで値を取得できます。 詳細については、特権エンドポイントに関するページを参照してください。

### <a name="set-up-conditional-forwarding-to-azure-stack"></a>Azure Stack への条件付き転送を設定する

Azure Stack と DNS インフラストラクチャを統合する最も簡単で安全な方法は、親ゾーンをホストするサーバーから、ゾーンの条件付き転送を行うことです。 Azure Stack の外部 DNS 名前空間の親ゾーンをホストする DNS サーバーを直接制御できる場合は、この方法をお勧めします。

DNS で条件付き転送を行う方法がわからない場合は、TechNet の記事「ドメイン名の条件付きフォワーダーの割り当て」またはお使いの DNS ソリューションに固有のドキュメントを参照してください。

会社のドメイン名の子ドメインのように見える Azure Stack 外部の DNS ゾーンを指定しているシナリオでは、条件付き転送は使用できません。 DNS 委任を構成する必要があります。

例:

- 会社の DNS ドメイン名: *contoso.com*
- Azure Stack 外部 DNS ドメイン名: *azurestack.contoso.com*

### <a name="edit-dns-forwarder-ips"></a>DNS フォワーダー IP を編集する

DNS フォワーダー IP は Azure Stack のデプロイ中に設定されます。 何らかの理由でフォワーダー IP を更新する必要がある場合は、特権エンドポイントに接続し、*Get-AzSDnsForwarder* および *Set-AzSDnsForwarder [[-IPAddress] \<IPAddress[]\>]* の PowerShell コマンドレットを実行することで値を編集することができます。 詳細については、特権エンドポイントに関するページを参照してください。

### <a name="delegate-the-external-dns-zone-to-azure-stack"></a>外部 DNS ゾーンを Azure Stack に委任する

DNS 名を Azure Stack デプロイの外部から解決できるようにするには、DNS 委任を設定する必要があります。

各レジストラーは独自の DNS 管理ツールを所有していて、ドメインのネーム サーバー レコードを変更します。 レジストラーの DNS 管理ページで、NS レコードを編集し、ゾーンの NS レコードを、Azure Stack の NS レコードに置き換えます。

ほとんどの DNS レジストラーでは、委任を実行するために 2 つ以上の DNS サーバーを指定する必要があります。

### <a name="firewall"></a>ファイアウォール

Azure Stack は、そのインフラストラクチャ ロールのために仮想 IP アドレス (VIP) を設定します。 この VIP はパブリック IP アドレス プールから割り当てられます。 各 VIP は、ソフトウェア定義のネットワーク レイヤーで、アクセス制御リスト (ACL) で保護されます。 ACL は、ソリューションをさらに強化するために、さまざまな物理スイッチ (TOR や BMC) でも使われます。 デプロイ時に指定された外部 DNS ゾーン内のエンドポイントごとに、DNS エントリが作成されます。 たとえば、ユーザー ポータルに portal. *\<region\>.\<fqdn\>* の DNS ホスト エントリが割り当てられます。

次のアーキテクチャ図は、さまざまなネットワーク レイヤーと ACL を示しています。

![アーキテクチャ図は、さまざまなネットワーク レイヤーと ACL を示しています。](media/network-deployment/network-architecture.png)

### <a name="ports-and-urls"></a>ポートと URL

Azure Stack サービス (ポータル、Azure Resource Manager、DNS など) を外部ネットワークに対して使用可能にするには、特定の URL、ポート、プロトコルに対して、これらのエンドポイントへの受信トラフィックを許可する必要があります。

透過プロキシから従来のプロキシ サーバーへのアップリンクが存在するか、ファイアウォールでソリューションを保護しているデプロイでは、特定のポートと URL に受信および送信の両方の通信を許可する必要があります。 たとえば、ID、Azure Stack Hub Marketplace、パッチと更新プログラム、登録、使用状況データに使用するポートと URL が該当します。

### <a name="outbound-communication"></a>送信方向の通信

Azure Stack は、透過的なプロキシ サーバーのみをサポートします。 透過プロキシから従来のプロキシ サーバーへのアップリンクが存在するデプロイ環境では、接続モードでデプロイするときに次の表のポートと URL に外部への通信を許可する必要があります。

SSL トラフィックのインターセプトはサポートされていないため、エンドポイントにアクセスするとサービス エラーが発生する場合があります。 ID に必要なエンドポイントとの通信に対してサポートされる最大タイムアウトは、60 秒です。

>[!NOTE]
>Azure Stack では、Azure ExpressRoute を使用して、次の表に示す Azure サービスに到達することはサポートされていません。ExpressRoute ではすべてのエンドポイントにトラフィックをルーティングできない場合があるからです。


|目的|接続先 URL|Protocol|Port|発信元ネットワーク|
|---------|---------|---------|---------|---------|
|ID|**Azure**<br>login.windows.net<br>login.microsoftonline.com<br>graph.windows.net<br>https:\//secure.aadcdn.microsoftonline-p.com<br>www.office.com<br>ManagementServiceUri = https:\//management.core.windows.net<br>ARMUri = https:\//management.azure.com<br>https:\//\*.msftauth.net<br>https:\//\*.msauth.net<br>https:\//\*.msocdn.com<br>**Azure Government**<br>https:\//login.microsoftonline.us/<br>https:\//graph.windows.net/<br>**Azure China 21Vianet**<br>https:\//login.chinacloudapi.cn/<br>https:\//graph.chinacloudapi.cn/<br>**Azure Germany**<br>https:\//login.microsoftonline.de/<br>https:\//graph.cloudapi.de/|HTTP<br>HTTPS|80<br>443|パブリック VIP - /27<br>パブリック インフラストラクチャ ネットワーク|
|Azure Stack Hub Marketplace シンジケーション|**Azure**<br>https:\//management.azure.com<br>https://&#42;.blob.core.windows.net<br>https://&#42;.azureedge.net<br>**Azure Government**<br>https:\//management.usgovcloudapi.net/<br>https://&#42;.blob.core.usgovcloudapi.net/<br>**Azure China 21Vianet**<br>https:\//management.chinacloudapi.cn/<br>http://&#42;.blob.core.chinacloudapi.cn|HTTPS|443|パブリック VIP - /27|
|パッチと更新プログラム|https://&#42;.azureedge.net<br>https:\//aka.ms/azurestackautomaticupdate|HTTPS|443|パブリック VIP - /27|
|登録|**Azure**<br>https:\//management.azure.com<br>**Azure Government**<br>https:\//management.usgovcloudapi.net/<br>**Azure China 21Vianet**<br>https:\//management.chinacloudapi.cn|HTTPS|443|パブリック VIP - /27|
|使用法|**Azure**<br>https://&#42;.trafficmanager.net<br>**Azure Government**<br>https://&#42;.usgovtrafficmanager.net<br>**Azure China 21Vianet**<br>https://&#42;.trafficmanager.cn|HTTPS|443|パブリック VIP - /27|
|Windows Defender|&#42;.wdcp.microsoft.com<br>&#42;.wdcpalt.microsoft.com<br>&#42;.wd.microsoft.com<br>&#42;.update.microsoft.com<br>&#42;.download.microsoft.com<br>https:\//www.microsoft.com/pkiops/crl<br>https:\//www.microsoft.com/pkiops/certs<br>https:\//crl.microsoft.com/pki/crl/products<br>https:\//www.microsoft.com/pki/certs<br>https:\//secure.aadcdn.microsoftonline-p.com<br>|HTTPS|80<br>443|パブリック VIP - /27<br>パブリック インフラストラクチャ ネットワーク|
|NTP|デプロイのために用意されている NTP サーバーの IP|UDP|123|パブリック VIP - /27|
|DNS|デプロイのために用意されている DNS サーバーの IP|TCP<br>UDP|53|パブリック VIP - /27|
|CRL|証明書上で CRL 配布ポイントの下にある URL|HTTP|80|パブリック VIP - /27|
|LDAP|Azure Graph 統合のために用意されている Active Directory フォレスト|TCP<br>UDP|389|パブリック VIP - /27|
|LDAP SSL|Graph 統合のために用意されている Active Directory フォレスト|TCP|636|パブリック VIP - /27|
|LDAP GC|Graph 統合のために用意されている Active Directory フォレスト|TCP|3268|パブリック VIP - /27|
|LDAP GC SSL|Graph 統合のために用意されている Active Directory フォレスト|TCP|3269|パブリック VIP - /27|
|AD FS|AD FS 統合のために用意されている AD FS メタデータ エンドポイント|TCP|443|パブリック VIP - /27|
|診断ログ収集サービス|Azure Blob Storage に用意されている共有アクセス署名 URL|HTTPS|443|パブリック VIP - /27|
|     |     |     |     |     |
                                                                                                                                                                
### <a name="inbound-communication"></a>受信方向の通信

Azure Stack エンドポイントを外部ネットワークに公開するには、一連のインフラストラクチャ VIP が必要です。 "エンドポイント (VIP) " の表は、各エンドポイント、必要なポート、およびプロトコルを示しています。 SQL リソース プロバイダーなど、追加のリソース プロバイダーを必要とするエンドポイントについては、特定のリソース プロバイダーのデプロイに関するドキュメントを参照してください。

社内インフラストラクチャの VIP は Azure Stack の発行には不要なため、記載されていません。 ユーザーの VIP は動的であり、ユーザー自身によって定義され、Azure Stack オペレーターによって管理されるわけではありません。

>[!NOTE]
>
>IKEv2 VPN は、標準ベースの IPsec VPN ソリューションであり、UDP ポート 500 と 4500、および TCP ポート 50 を使用します。 ファイアウォールでは、これらのポートが必ずしも開いているとは限りません。そのため、IKEv2 VPN ではプロキシとファイアウォールを通過できないことがあります。

|エンドポイント (VIP)|DNS ホスト A レコード|Protocol|Port|
|---------|---------|---------|---------|
|AD FS|Adfs. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|Azure portal (管理者)|Adminportal. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|AdminHosting | *.adminhosting.\<region>.\<fqdn> | HTTPS | 443 |
|Azure Resource Manager (管理者)|Adminmanagement. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|Azure portal (ユーザー)|Portal. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|Azure Resource Manager (ユーザー)|Management. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|Azure Graph|Graph. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|証明書の失効リスト|Crl. *&lt;region>.&lt;fqdn>*|HTTP|80|
|DNS|&#42;. *&lt;region>.&lt;fqdn>*|TCP と UDP|53|
|Hosting | *.hosting.\<region>.\<fqdn> | HTTPS | 443 |
|Azure Key Vault (ユーザー)|&#42;.vault. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|Azure Key Vault (管理者)|&#42;.adminvault. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|Azure Queue Storage |&#42;.queue. *&lt;region>.&lt;fqdn>*|HTTP<br>HTTPS|80<br>443|
|Azure Table Storage |&#42;.table. *&lt;region>.&lt;fqdn>*|HTTP<br>HTTPS|80<br>443|
|Azure Blob Storage |&#42;.blob. *&lt;region>.&lt;fqdn>*|HTTP<br>HTTPS|80<br>443|
|SQL リソース プロバイダー|sqladapter.dbadapter. *&lt;region>.&lt;fqdn>*|HTTPS|44300-44304|
|MySQL リソース プロバイダー|mysqladapter.dbadapter. *&lt;region>.&lt;fqdn>*|HTTPS|44300-44304|
|Azure App Service|&#42;.appservice. *&lt;region>.&lt;fqdn>*|TCP|80 (HTTP)<br>443 (HTTPS)<br>8172 (MSDeploy)|
|  |&#42;.scm.appservice. *&lt;region>.&lt;fqdn>*|TCP|443 (HTTPS)|
|  |api.appservice. *&lt;region>.&lt;fqdn>*|TCP|443 (HTTPS)<br>44300 (Azure Resource Manager)|
|  |ftp.appservice. *&lt;region>.&lt;fqdn>*|TCP、UDP|21、1021、10001-10100 (FTP)<br>990 (FTPS)|
|Azure VPN Gateway|     |     |[VPN Gateway に関する FAQ を参照してください](/azure/vpn-gateway/vpn-gateway-vpn-faq#can-i-traverse-proxies-and-firewalls-using-point-to-site-capability)。|
|     |     |     |     |