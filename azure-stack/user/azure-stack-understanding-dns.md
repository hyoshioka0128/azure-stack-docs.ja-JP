---
title: Azure Stack Hub での iDNS の使用
description: Azure Stack Hub での iDNS 機能の使用方法について説明します。
author: Justinha
ms.topic: conceptual
ms.date: 09/16/2019
ms.author: Justinha
ms.reviewer: scottnap
ms.lastreviewed: 01/14/2019
ms.openlocfilehash: 466e44156181e53a92181ecb462b8b4ff0074317
ms.sourcegitcommit: 0aa5f7f20690839661c8bb3bfdbe32f82bec0c64
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/21/2020
ms.locfileid: "86566534"
---
# <a name="use-idns-in-azure-stack-hub"></a>Azure Stack Hub での iDNS の使用 

iDNS は、外部 DNS 名 (例: http:\//www.bing.com) を解決することができる、Azure Stack Hub のネットワーク機能です。内部仮想ネットワーク名を登録することもできます。 これを行うことで、同じ仮想ネットワーク内の仮想マシン (VM) を IP アドレスではなく名前で解決することができます。 このアプローチでは、カスタム DNS サーバー エントリを用意する必要がありません。 DNS の詳細については、[Azure DNS の概要](/azure/dns/dns-overview)に関するページを参照してください。

## <a name="what-does-idns-do"></a>iDNS が実行すること

Azure Stack Hub の iDNS を使用して、カスタム DNS サーバー エントリの指定なしで次の機能を取得できます。

- テナントのワークロードに対する共有 DNS 名前解決サービス。
- テナントの仮想ネットワーク内で名前解決と DNS 登録を行う権限がある DNS サービス。
- テナントの VM からインターネット名を解決する再帰的な DNS サービス。 テナントは、インターネット名 (例: www\.bing.com) を解決するためにカスタム DNS エントリを指定する必要がなくなりました。

独自の DNS とカスタム DNS サーバーを使用することもできます。 しかし iDNS を使用すれば、インターネットの DNS 名を解決することに加え、同じ仮想ネットワーク内の他の VM に接続することもでき、カスタム DNS エントリを作成する必要がありません。

## <a name="what-doesnt-idns-do"></a>iDNS が実行しないこと

iDNS では、仮想ネットワークの外部から解決できる名前の DNS レコードを作成することは認められていません。

Azure には、パブリック IP アドレスに関連付けられる DNS 名ラベルを指定するオプションがあります。 ラベル (プレフィックス) は自分で選択できますが、サフィックスは、パブリック IP アドレスを作成するリージョンに基づいて Azure が選択します。

![DNS 名ラベルの例](media/azure-stack-understanding-dns-in-tp2/image3.png)

前の画像が示すように、Azure は、指定された DNS の名前ラベルを持つ **westus.cloudapp.azure.com** ゾーンの DNS に "A" レコードを作成します。 プレフィックスとサフィックスが組み合わされて、パブリック インターネットの任意の場所で解決できる[完全修飾ドメイン名](https://en.wikipedia.org/wiki/Fully_qualified_domain_name) (FQDN) を構成します。

Azure Stack Hub では、内部名の登録でのみ iDNS をサポートするため、以下を行うことはできません。

- 既存のホストされている DNS ゾーン (例: local.azurestack.external) に DNS レコードを作成する。
- DNS ゾーン (Contoso.com など) を作成する。
- 独自のカスタム DNS ゾーンにレコードを作成する。
- ドメイン名の購入をサポートする。

## <a name="demo-of-how-idns-works"></a>iDNS のしくみのデモ

Virtual Network 上の VM のすべてのホスト名は、同じゾーンの DNS リソース レコードとして格納されます。ただし、その VM が展開された SDN インフラストラクチャ内の VNET ID に関連付けられた GUID として定義された独自の一意のコンパートメントに格納されます。 テナント VM の完全修飾ドメイン名 (FQDN) は、コンピューター名と Virtual Network の DNS サフィックス文字列 (GUID 形式) で構成されます。

<!--- what does compartment mean? Add a screenshot? can we clarify what we mean by host name and computer name. the description doesn't match the example in the table.--->
 
次に、これのしくみを示す簡単なラボを示します。 1 つの VNet に 3 つの VM を、また別の VNet に別の VM を作成しました。

<!--- Is DNS Label the right term? If so, we should define it. The column lists FQDNs, afaik. Where does the domain suffix come from? --->
 
|VM    |vNet    |プライベート IP   |パブリック IP    | DNS Label                                |
|------|--------|-------------|-------------|------------------------------------------|
|VM-A1 |VNetA   | 10.0.0.5    |172.31.12.68 |VM-A1-Label.lnv1.cloudapp.azscss.external |
|VM-A2 |VNetA   | 10.0.0.6    |172.31.12.76 |VM-A2-Label.lnv1.cloudapp.azscss.external |
|VM-A3 |VNetA   | 10.0.0.7    |172.31.12.49 |VM-A3-Label.lnv1.cloudapp.azscss.external |
|VM-B1 |VNetB   | 10.0.0.4    |172.31.12.57 |VM-B1-Label.lnv1.cloudapp.azscss.external |
 
 
|VNet  |GUID                                 |DNS サフィックス文字列                                                  |
|------|-------------------------------------|-------------------------------------------------------------------|
|VNetA |e71e1db5-0a38-460d-8539-705457a4cf75 |e71e1db5-0a38-460d-8539-705457a4cf75.internal.lnv1.azurestack.local|
|VNetB |e8a6e386-bc7a-43e1-a640-61591b5c76dd |e8a6e386-bc7a-43e1-a640-61591b5c76dd.internal.lnv1.azurestack.local|
 
 
iDNS のしくみを理解するには、次のいくつかの名前解決のテストを行ってください。

<!--- why Linux?--->

VM-A1 (Linux VM) からVM-A2 を検索します。 VNetA の DNS サフィックスが追加され、名前がプライベート IP に解決されていることがわかります。
 
```console
carlos@VM-A1:~$ nslookup VM-A2
Server:         127.0.0.53
Address:        127.0.0.53#53
 
Non-authoritative answer:
Name:   VM-A2.e71e1db5-0a38-460d-8539-705457a4cf75.internal.lnv1.azurestack.local
Address: 10.0.0.6
```
 
FQDN を指定せずに VM-A2 ラベルを検索すると、予期したとおりに失敗します。

```console 
carlos@VM-A1:~$ nslookup VM-A2-Label
Server:         127.0.0.53
Address:        127.0.0.53#53
 
** server can't find VM-A2-Label: SERVFAIL
```

DNS ラベルに FQDN を指定した場合、名前はパブリック IP に解決されます。

```console
carlos@VM-A1:~$ nslookup VM-A2-Label.lnv1.cloudapp.azscss.external
Server:         127.0.0.53
Address:        127.0.0.53#53
 
Non-authoritative answer:
Name:   VM-A2-Label.lnv1.cloudapp.azscss.external
Address: 172.31.12.76
```
 
(別の VNet の) VM-B1 を解決しようとした場合、このゾーンにはこのレコードが存在しないため失敗します。

```console
carlos@caalcobi-vm4:~$ nslookup VM-B1
Server:         127.0.0.53
Address:        127.0.0.53#53
 
** server can't find VM-B1: SERVFAIL
```

このレコードは別のゾーンのものであるため、VM-B1 の FQDN は使用できません。

```console 
carlos@VM-A1:~$ nslookup VM-B1.e8a6e386-bc7a-43e1-a640-61591b5c76dd.internal.lnv1.azurestack.local
Server:         127.0.0.53
Address:        127.0.0.53#53
 
** server can't find VM-B1.e8a6e386-bc7a-43e1-a640-61591b5c76dd.internal.lnv1.azurestack.local: SERVFAIL
```
 
DNS ラベルに FQDN を使用すると、次のように正常に解決されます。

``` 
carlos@VM-A1:~$ nslookup VM-B1-Label.lnv1.cloudapp.azscss.external
Server:         127.0.0.53
Address:        127.0.0.53#53
 
Non-authoritative answer:
Name:   VM-B1-Label.lnv1.cloudapp.azscss.external
Address: 172.31.12.57
```
 
VM-A3 (Windows VM) からです。 権限のある場合の回答と権限のない場合の回答の違いに注目してください。

内部レコード:

```console
C:\Users\carlos>nslookup
Default Server:  UnKnown
Address:  168.63.129.16
 
> VM-A2
Server:  UnKnown
Address:  168.63.129.16
 
Name:    VM-A2.e71e1db5-0a38-460d-8539-705457ª4cf75.internal.lnv1.azurestack.local
Address:  10.0.0.6
```

外部レコード:

```console
> VM-A2-Label.lnv1.cloudapp.azscss.external
Server:  UnKnown
Address:  168.63.129.16
 
Non-authoritative answer:
Name:    VM-A2-Label.lnv1.cloudapp.azscss.external
Address:  172.31.12.76
``` 
 
つまり、上記からは次がわかります。
 
*   各 VNet には独自のゾーンがあります。これには、VM 名と VNet の DNS サフィックス (GUID) で構成されるすべてのプライベート IP アドレスの A レコードが含まれます。
    *   \<vmname>.\<vnetGUID\>.internal.\<region>.\<stackinternalFQDN>
    *   これは自動的に行われます
*   パブリック IP アドレスを使用した場合、それらに DNS ラベルを作成することも可能です。 これらは、他の外部アドレスと同様に解決されます。
 
 
- iDNS サーバーは、それの内部 DNS ゾーンの権限を持つサーバーであり、テナント VM が外部リソースに接続するときのパブリック名のリゾルバーとして機能します。 外部リソースに対しクエリがある場合、iDNS サーバーは解決のために解決する権限を持つ DNS サーバーにその要求を送信します。
 
ラボの結果からわかるように、どの IP が使用されるか制御できます。 VM 名を使用すると、プライベート IP アドレスが取得され、DNS ラベルを使用すると、パブリック IP アドレスを取得できます。

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub での DNS の使用](azure-stack-dns.md)
