---
title: 同じ Azure Stack 環境内の 2 つの仮想ネットワークを接続する
description: Fortinet FortiGate を使用して、同じ Azure Stack Hub 環境内の 2 つの仮想ネットワークを接続する方法について説明します。
author: mattbriggs
ms.topic: how-to
ms.date: 5/27/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 10/03/2019
ms.openlocfilehash: d4589e4ed9bc7850ce095456a27d3a9a642a1664
ms.sourcegitcommit: 8ffa29f71d69191534d42f86f49f719b4198a097
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/22/2020
ms.locfileid: "92354733"
---
# <a name="vnet-peering-with-fortigate"></a>FortiGate を使用した VNET ピアリング

この記事では、同じ環境内の 2 つの仮想ネットワーク間の接続を作成する方法について説明します。 接続を設定しながら、Azure Stack Hub での VPN ゲートウェイのしくみを学習します。 Fortinet FortiGate を使用して、同じ Azure Stack Hub 環境内で 2 つの VNET を接続します。 この手順では、各 VNET で FortiGate NVA (ネットワーク仮想アプライアンス) を使用して、2 つの VNET をそれぞれ別個のリソース グループ内にデプロイします。 また、2 つの VNET 間に IPSec VPN を設定するために必要な変更についても詳しく説明します。 VNET のデプロイごとに、この記事の手順を繰り返します。

## <a name="prerequisites"></a>前提条件

-   このソリューションで求められるコンピューティング要件、ネットワーク要件、リソース要件をデプロイするために必要とされる空き容量を持つシステムへのアクセス。

-  Azure Stack Hub Marketplace にダウンロードされ、発行されているネットワーク仮想アプライアンス (NVA) ソリューション。 NVA は、境界ネットワークから他のネットワークまたはサブネットへのネットワーク トラフィックのフローを制御します。 この手順では、「[Fortinet FortiGate の次世代ファイアウォールの単一の VM ソリューション](https://azuremarketplace.microsoft.com/marketplace/apps/fortinet.fortinet-FortiGate-singlevm)」を使用します。

-  FortiGate NVA をアクティブにするための、2 つ以上の有効な FortiGate ライセンス ファイル。 これらのライセンスを取得する方法については、Fortinet ドキュメント ライブラリの「[ライセンスの登録とダウンロード](https://docs2.fortinet.com/vm/azure/FortiGate/6.2/azure-cookbook/6.2.0/19071/registering-and-downloading-your-license)」の記事を参照してください。

    この手順では、「[単一の FortiGate-VM デプロイ](https://docs2.fortinet.com/vm/azure/FortiGate/6.2/azure-cookbook/6.2.0/632940/single-FortiGate-vm-deployment)」を使用します。 FortiGate NVA を Azure Stack Hub VNET に接続する手順については、オンプレミス ネットワーク内で確認できます。

    アクティブ/パッシブ (HA) 設定で FortiGate ソリューションをデプロイする方法の詳細については、Fortinet ドキュメント ライブラリの「[Azure 上の FortiGate-VM の HA](https://docs2.fortinet.com/vm/azure/FortiGate/6.2/azure-cookbook/6.2.0/983245/ha-for-FortiGate-vm-on-azure)」の記事を参照してください。

## <a name="deployment-parameters"></a>展開のパラメーター

次の表には、これらのデプロイで使用されているパラメーターが参照用にまとめられています。

### <a name="deployment-one-forti1"></a>デプロイ 1:Forti1

| FortiGate のインスタンス名 | Forti1 |
|-----------------------------------|---------------------------|
| BYOL ライセンス/バージョン | 6.0.3 |
| FortiGate の管理ユーザー名 | fortiadmin |
| リソース グループ名 | forti1-rg1 |
| 仮想ネットワーク名 | forti1vnet1 |
| VNET のアドレス空間 | 172.16.0.0/16* |
| パブリック VNET サブネット名 | forti1-PublicFacingSubnet |
| パブリック VNET アドレス プレフィックス | 172.16.0.0/24* |
| VNET 内部サブネット名 | forti1-InsideSubnet |
| VNET 内部サブネット プレフィックス | 172.16.1.0/24* |
| FortiGate NVA の VM サイズ | 標準 F2s_v2 |
| パブリック IP アドレス名 | forti1-publicip1 |
| パブリック IP アドレスの種類 | 静的 |

### <a name="deployment-two-forti2"></a>デプロイ 2:Forti2

| FortiGate のインスタンス名 | Forti2 |
|-----------------------------------|---------------------------|
| BYOL ライセンス/バージョン | 6.0.3 |
| FortiGate の管理ユーザー名 | fortiadmin |
| リソース グループ名 | forti2-rg1 |
| 仮想ネットワーク名 | forti2vnet1 |
| VNET のアドレス空間 | 172.17.0.0/16* |
| パブリック VNET サブネット名 | forti2-PublicFacingSubnet |
| パブリック VNET アドレス プレフィックス | 172.17.0.0/24* |
| VNET 内部サブネット名 | Forti2-InsideSubnet |
| VNET 内部サブネット プレフィックス | 172.17.1.0/24* |
| FortiGate NVA の VM サイズ | 標準 F2s_v2 |
| パブリック IP アドレス名 | Forti2-publicip1 |
| パブリック IP アドレスの種類 | 静的 |

> [!NOTE]
> \* 上記が何らかの形 (いずれかの Azure Stack Hub の VIP プールなど) でオンプレミス ネットワーク環境と重複する場合は、別のセットのアドレス空間とサブネット プレフィックスを選択してください。 また、アドレス範囲が互いに重複していないことを確認してください。

## <a name="deploy-the-fortigate-ngfw"></a>FortiGate NGFW をデプロイする

1.  Azure Stack Hub ユーザー ポータルを開きます。

2.  **[リソースの作成]** を選択し、`FortiGate` を検索します。

    ![検索結果の一覧に、"FortiGate NGFW - Single VM Deployment" が表示されています。](./media/azure-stack-network-howto-vnet-to-onprem/image6a.png)

3.  **[FortiGate NGFW]** を選択し、**[作成]** を選択します。

4.  「[デプロイで使用されるパラメーター](#deployment-parameters)」の表にあるパラメーターを使用して、**[基本]** を入力します。

    ![[基本] 画面に、リストおよびテキスト ボックスで選択および入力したデプロイ パラメーターの値が表示されています。](./media/azure-stack-network-howto-vnet-to-onprem/image7a.png)

5.  **[OK]** を選択します。

6.  「[デプロイで使用されるパラメーター](#deployment-parameters)」の表を使用して、仮想ネットワーク、サブネット、VM サイズの詳細を指定します。

    > [!Warning] 
    > オンプレミス ネットワークが IP 範囲 `172.16.0.0/16` とオーバーラップする場合は、別のネットワーク範囲とサブネットを選択して設定する必要があります。 「[デプロイで使用されるパラメーター](#deployment-parameters)」の表にあるものとは別の名前や範囲を使用する場合は、オンプレミス ネットワークと競合**しない**パラメーターを使用してください。 VNET 内の VNET IP 範囲やサブネットの範囲を設定するときは注意してください。 その範囲がオンプレミス ネットワークに存在する IP 範囲とオーバーラップすることは望ましくありません。

7.  **[OK]** を選択します。

8.  Fortigate NVA のパブリック IP を次のように構成します。

    ![[IP 割り当て] ダイアログ ボックスで、[パブリック IP アドレス名] に値 forti1-publicip1 が表示され、[パブリック IP アドレスの種類] に値 [静的] が表示されています。](./media/azure-stack-network-howto-vnet-to-onprem/image8a.png)

9.  **[OK]** を選択します。 **[OK]** を選択します。

10.  **［作成］** を選択します

デプロイには約 10 分かかります。

## <a name="configure-routes-udrs-for-each-vnet"></a>各 VNET のルート (UDR) を構成する

forti1-rg1 と forti2-rg1 の両方のデプロイで、次の手順を実行します。

1. Azure Stack Hub ユーザー ポータルを開きます。

1. リソース グループを選択します。 フィルターに「`forti1-rg1`」と入力し、forti1-rg1 リソース グループをダブルクリックします。

    ![forti1-rg1 リソース グループに対して 10 個のリソースが一覧表示されています。](./media/azure-stack-network-howto-vnet-to-onprem/image9a.png)

1. **forti1-forti1-InsideSubnet-routes-xxxx** リソースを選択します。

1. **[設定]** で、**[ルート]** を選択します。

    ![[設定] ダイアログ ボックスで [ルート] ボタンが選択されています。](./media/azure-stack-network-howto-vnet-to-onprem/image10a.png)

1. **[to-Internet]\(インターネットへ\)** ルートを削除します。

    ![[to-Internet]\(インターネットへ\) ルートは、一覧表示されている唯一のルートであり、選択されています。 [削除] ボタンがあります。](./media/azure-stack-network-howto-vnet-to-onprem/image11a.png)

1. *[はい]* を選択します。

1. **[追加]** を選択して、新しいルートを追加します。

1. ルートに `to-onprem` という名前を付けます。

1. VPN が接続されるオンプレミス ネットワークのネットワーク範囲を定義する、IP ネットワーク範囲を入力します。

1. **[次ホップの種類]** として **[仮想アプライアンス]** を選択し、`172.16.1.4` とします。 別の IP 範囲を使用している場合は、その IP 範囲を使用します。

    ![[ルートの追加] ダイアログ ボックスに、テキスト ボックスで選択および入力した 4 つの値が表示されています。](./media/azure-stack-network-howto-vnet-to-onprem/image12a.png)

1. **[保存]** を選択します。

各 FortiGate NVA をアクティブにするには、Fortinet が提供する有効なライセンス ファイルが必要です。 各 NVA がアクティブになるまで、NVA は機能**しません**。 ライセンス ファイルを取得する方法と NVA をアクティブにする手順については、Fortinet ドキュメント ライブラリの「[ライセンスの登録とダウンロード](https://docs2.fortinet.com/vm/azure/FortiGate/6.2/azure-cookbook/6.2.0/19071/registering-and-downloading-your-license)」の記事を参照してください。

2 つのライセンス ファイル (NVA ごとに 1 つ) を取得する必要があります。

## <a name="create-an-ipsec-vpn-between-the-two-nvas"></a>2 つの NVA 間に IPSec VPN を作成する

NVA がアクティブになったら、次の手順に従って、2 つの NVA 間に IPSec VPN を作成します。

forti1 NVA と forti2 NVA の両方について、次の手順に従います。

1. fortiX VM の概要ページに移動して、割り当てられたパブリック IP アドレスを取得します。

    ![forti1 仮想マシンの [概要] ページには、[リソース グループ] や [状態] などの forti1 の値が表示されています。](./media/azure-stack-network-howto-vnet-to-vnet/image13a.png)

1. 割り当てられた IP アドレスをコピーし、ブラウザーを開いてアドレス バーにそのアドレスを貼り付けます。 セキュリティ証明書が信頼されていないという警告がブラウザーに表示される場合があります。 そのまま続行します。

1. デプロイ中に指定した FortiGate の管理ユーザー名とパスワードを入力します。

    ![[ログイン] ダイアログ ボックスには、ユーザーとパスワード用のテキスト ボックスと [ログイン] ボタンがあります。](./media/azure-stack-network-howto-vnet-to-vnet/image14a.png)

1. **[System]\(システム\)**  >  **[Firmware]\(ファームウェア\)** を選択します。

1. 最新のファームウェアを示すボックスを選択します (例: `FortiOS v6.2.0 build0866`)。

    ![[ファームウェア] ダイアログ ボックスには、ファームウェアの識別子 "FortiOS v6.2.0 build0866"、リリース ノートへのリンク、2 つのボタン ([Backup config and upgrade]\(コンフィグをバックアップしてアップグレード\) と [アップグレード]) があります。](./media/azure-stack-network-howto-vnet-to-vnet/image15a.png)

1. **[Backup config and upgrade]\(コンフィグをバックアップしてアップグレード\)**  >  **[Continue]\(続行\)** を選択します。

1. NVA でファームウェアが最新のビルドに更新され、再起動されます。 このプロセスには約 5 分かかります。 FortiGate Web コンソールにもう一度ログインします。

1. **[VPN]**  >  **[IPSec Wizard]\(IPSec ウィザード\)** をクリックします。

1. VPN の名前を入力します。たとえば、**[VPN Creation Wizard]\(VPN 作成ウィザード\)** で「`conn1`」と入力します。

1. **[This site is behind NAT]\(このサイトは NAT の背後にある\)** を選択します。

    ![[VPN Creation Wizard]\(VPN 作成ウィザード\) のスクリーンショットは、最初の手順である [VPN Setup]\(VPN の設定\) を示しています。 [Template Type]\(テンプレートの種類\) には [Site to Site]\(サイト間\)、[Remote Device Type]\(リモート デバイスの種類\) には [FortiGate]、[NAT Configuration]\(NAT 構成\) には [This site is behind NAT]\(このサイトは NAT の背後にある\) の値が選択されています。](./media/azure-stack-network-howto-vnet-to-vnet/image16a.png)

1. **[次へ]** を選択します。

1. 接続先となるオンプレミスの VPN デバイスのリモート IP アドレスを入力します。

1. **[Outgoing Interface]\(送信インターフェイス\)** として **[port1]** を選択します。

1. **[Pre-shared Key]\(事前共有キー\)** を選択し、事前共有キーを入力 (および記録) します。 

    > [!NOTE]  
    > このキーは、オンプレミスの VPN デバイスで接続を設定するために必要であるため、"*正確に*" 一致する必要があります。

    ![[VPN Creation Wizard]\(VPN 作成ウィザード\) のスクリーンショットは、2 番目の手順である [Authentication]\(認証\) を示し、選択した値が強調表示されています。](./media/azure-stack-network-howto-vnet-to-vnet/image17a.png)

1. **[次へ]** を選択します。

1. **[Local Interface]\(ローカル インターフェイス\)** として **[port2]** を選択します。

1. 次のローカル サブネットの範囲を入力します。
    - forti1:172.16.0.0/16
    - forti2:172.17.0.0/16

    別の IP 範囲を使用している場合は、その IP 範囲を使用します。

1. オンプレミスの VPN デバイスを介して接続する、オンプレミス ネットワークを表す適切なリモート サブネットを入力します。
    - forti1:172.16.0.0/16
    - forti2:172.17.0.0/16

    別の IP 範囲を使用している場合は、その IP 範囲を使用します。

    ![[VPN Creation Wizard]\(VPN 作成ウィザード\) のスクリーンショットは、3 番目の手順である [Policy & Routing]\(ポリシーとルーティング\) を示しています。 選択した値と入力した値が表示されています。](./media/azure-stack-network-howto-vnet-to-vnet/image18a.png)

1. **[作成]**

1. **[Network]\(ネットワーク\)**  >  **[Interfaces]\(インターフェイス\)** を選択します。

    ![インターフェイスの一覧に、2 つのインターフェイス (構成済みの port1 と構成されていない port2) が表示されています。 インターフェイスを作成、編集、削除するためのボタンがあります。](./media/azure-stack-network-howto-vnet-to-vnet/image19a.png)

1. **[port2]** をダブルクリックします。

1. **[Role]\(ロール\)** の一覧から **[LAN]** を、アドレス指定モードとして **[DHCP]** を選択します。

1. **[OK]** を選択します。

他の NVA に対して同じ手順を繰り返します。

## <a name="bring-up-all-phase-2-selectors"></a>すべてのフェーズ 2 セレクターを表示する 

"*両方*" の NVA で上記が完了したら、次のようにします。

1.  forti2 の FortiGate Web コンソールで、 **[Monitor]\(モニター\)**  >  **[IPsec Monitor]\(IPsec モニター\)** を選択します。 

    ![VPN 接続 conn1 のモニターが表示されています。 これは、対応するフェーズ 2 セレクターと同様にダウン中として表示されています。](./media/azure-stack-network-howto-vnet-to-vnet/image20a.png)

2.  `conn1` を強調表示し、 **[Bring Up]\(表示\)**  >  **[All Phase 2 Selectors]\(すべてのフェーズ 2 セレクター\)** を選択します。

    ![モニターとフェーズ 2 セレクターは両方とも稼働中として表示されています。](./media/azure-stack-network-howto-vnet-to-vnet/image21a.png)

## <a name="test-and-validate-connectivity"></a>接続のテストと検証

これで、FortiGate NVA を介して各 VNET 間をルーティングできるようになりました。 接続を検証するには、各 VNET の InsideSubnet に Azure Stack Hub VM を作成します。 Azure Stack Hub VM の作成は、ポータル、CLI、または PowerShell を使用して行うことができます。 VM を作成する際は、次の手順に従います。

-   Azure Stack Hub VM は、各 VNET の **InsideSubnet** に配置されます。

-   VM の作成時には、NSG は適用**しません** (ポータルから VM を作成する場合は、既定で追加される NSG を削除してください)。

-   接続をテストするために使用する通信が VM のファイアウォール規則によって許可されていることを確認します。 テスト目的の場合は、可能な限り、OS 内でファイアウォールを完全に無効にすることをお勧めします。

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub ネットワークの違いと考慮事項](azure-stack-network-differences.md)  
[Fortinet FortiGate を使用して Azure Stack Hub にネットワーク ソリューションを提供する](../operator/azure-stack-network-solutions-enable.md)  
