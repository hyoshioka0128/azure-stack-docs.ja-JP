---
title: Azure Stack Hub の VPN ゲートウェイを設定する
description: Azure Stack Hub の VPN ゲートウェイを設定する方法について説明します。
author: mattbriggs
ms.topic: how-to
ms.date: 12/2/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 12/2/2020
ms.openlocfilehash: bcc1b59795c0b2436f8b9556b35bb6f95dc7cf0e
ms.sourcegitcommit: 9ef2cdc748cf00cd3c8de90705ea0542e29ada97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/02/2020
ms.locfileid: "96525729"
---
# <a name="set-up-vpn-gateway-for-azure-stack-hub-using-fortigate-nva"></a>FortiGate NVA を使用して Azure Stack Hub の VPN ゲートウェイを設定する

この記事では、Azure Stack Hub に VPN 接続を作成する方法について説明します。 VPN ゲートウェイは、Azure Stack Hub の仮想ネットワークとリモート VPN ゲートウェイの間で暗号化されたトラフィックを送信する、仮想ネットワーク ゲートウェイの一種です。 次の手順では、FortiGate NVA (ネットワーク仮想アプライアンス) を使用して、リソース グループ内に 1 つの VNET をデプロイします。 また、FortiGate NVA で IPSec VPN を設定する手順も提供します。

## <a name="prerequisites"></a>前提条件

-  Azure Stack Hub 統合システムへのアクセスと、このソリューションで求められるコンピューティング要件、ネットワーク要件、リソース要件をデプロイするために必要とされる空き容量。 

    > [!NOTE]  
    > これらの手順は、Azure Stack Development Kit (ASDK) のネットワーク制限により、ASDK では使用 **できません**。 詳細については、「[ASDK の要件と考慮事項](../asdk/asdk-deploy-considerations.md)」を参照してください。

-  Azure Stack Hub 統合システムをホストするオンプレミス ネットワーク内の VPN デバイスへのアクセス。 デバイスでは、「[デプロイに使用されるパラメーター](#deployment-parameters)」で説明されているパラメーターを満たす IPSec トンネルを作成する必要があります。

-  Azure Stack Hub Marketplace でネットワーク仮想アプライアンス (NVA) ソリューションが利用できること。 NVA は、境界ネットワークから他のネットワークまたはサブネットへのネットワーク トラフィックのフローを制御します。 この手順では、「[Fortinet FortiGate の次世代ファイアウォールの単一の VM ソリューション](https://azuremarketplace.microsoft.com/marketplace/apps/fortinet.fortinet-FortiGate-singlevm)」を使用します。

    > [!NOTE]  
    > お使いの Azure Stack Hub Marketplace で **Fortinet FortiGate-VM For Azure BYOL** と **FortiGate NGFW - Single VM Deployment (BYOL)** が利用できない場合は、担当のクラウド オペレーターにお問い合わせください。

-  FortiGate NVA をアクティブにするには、有効な FortiGate ライセンス ファイルが 1 つ以上必要です。 これらのライセンスを取得する方法については、Fortinet ドキュメント ライブラリの「[ライセンスの登録とダウンロード](https://docs2.fortinet.com/vm/azure/FortiGate/6.2/azure-cookbook/6.2.0/19071/registering-and-downloading-your-license)」の記事を参照してください。

    この手順では、「[単一の FortiGate-VM デプロイ](https://docs2.fortinet.com/vm/azure/FortiGate/6.2/azure-cookbook/6.2.0/632940/single-FortiGate-vm-deployment)」を使用します。 FortiGate NVA を Azure Stack Hub VNET に接続する手順については、オンプレミス ネットワーク内で確認できます。

    アクティブ/パッシブ (HA) 設定で FortiGate ソリューションをデプロイする方法の詳細については、Fortinet ドキュメント ライブラリの「[Azure 上の FortiGate-VM の HA](https://docs2.fortinet.com/vm/azure/FortiGate/6.2/azure-cookbook/6.2.0/983245/ha-for-FortiGate-vm-on-azure)」の記事を参照してください。

## <a name="deployment-parameters"></a>展開のパラメーター

次の表には、これらのデプロイで使用されているパラメーターが参照用にまとめられています。

| パラメーター | 値 |
|-----------------------------------|---------------------------|
| FortiGate のインスタンス名 | forti1 |
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

> [!NOTE]
> \* `172.16.0.0/16` がオンプレミス ネットワークや Azure Stack Hub VIP プールと重複する場合は、異なるアドレス空間とサブネット プレフィックスを選択してください。

## <a name="deploy-the-fortigate-ngfw-marketplace-items"></a>FortiGate NGFW Marketplace 項目をデプロイする

1. Azure Stack Hub ユーザー ポータルを開きます。

1. **[リソースの作成]** を選択し、`FortiGate` を検索します。

    ![検索結果の一覧に、"FortiGate NGFW - Single VM Deployment" が表示されています。](./media/azure-stack-network-howto-vnet-to-onprem/image6a.png)

1. **[FortiGate NGFW]** を選択し、**[作成]** を選択します。

1. 「[デプロイで使用されるパラメーター](#deployment-parameters)」の表にあるパラメーターを使用して、**[基本]** を入力します。

    ![[基本] 画面に、リストおよびテキスト ボックスに入力したデプロイ パラメーター表の値が表示されています。](./media/azure-stack-network-howto-vnet-to-onprem/image7a.png)

1. **[OK]** を選択します。

1. 「[デプロイで使用されるパラメーター](#deployment-parameters)」の表を使用して、仮想ネットワーク、サブネット、VM サイズの詳細を指定します。

    > [!Warning] 
    > オンプレミス ネットワークが IP 範囲 `172.16.0.0/16` とオーバーラップする場合は、別のネットワーク範囲とサブネットを選択して設定する必要があります。 「[デプロイで使用されるパラメーター](#deployment-parameters)」の表にあるものとは別の名前や範囲を使用する場合は、オンプレミス ネットワークと競合 **しない** パラメーターを使用してください。 VNET 内の VNET IP 範囲やサブネットの範囲を設定するときは注意してください。 その範囲がオンプレミス ネットワークに存在する IP 範囲とオーバーラップすることは望ましくありません。

1. **[OK]** を選択します。

1. FortiGate NVA のパブリック IP を次のように構成します。

    ![[IP 割り当て] ダイアログ ボックスで、[パブリック IP アドレス名] に値 forti1-publicip1 が表示され、[パブリック IP アドレスの種類] に値 [静的] が表示されています。](./media/azure-stack-network-howto-vnet-to-onprem/image8a.png)

1. **[OK]** を選択します。 **[OK]** を選択します。

1. **［作成］** を選択します

    デプロイには約 10 分かかります。

## <a name="configure-routes-udr-for-the-vnet"></a>VNET のルート (UDR) を構成する

1. Azure Stack Hub ユーザー ポータルを開きます。

2. リソース グループを選択します。 フィルターに「`forti1-rg1`」と入力し、forti1-rg1 リソース グループをダブルクリックします。

    ![forti1-rg1 リソース グループに対して 10 個のリソースが一覧表示されています。](./media/azure-stack-network-howto-vnet-to-onprem/image9a.png)

2. "forti1-forti1-InsideSubnet-routes-xxxx" リソースを選択します。

3. **[設定]** で、**[ルート]** を選択します。

    ![[設定] ダイアログ ボックスで [ルート] ボタンが選択されています。](./media/azure-stack-network-howto-vnet-to-onprem/image10a.png)

4. **[to-Internet]\(インターネットへ\)** ルートを削除します。

    ![[to-Internet]\(インターネットへ\) ルートは、一覧表示されている唯一のルートであり、選択されています。 [削除] ボタンがあります。](./media/azure-stack-network-howto-vnet-to-onprem/image11a.png)

5. *[はい]* を選択します。

6. **[追加]** を選択して、新しいルートを追加します。

7. ルートに `to-onprem` という名前を付けます。

8. VPN が接続されるオンプレミス ネットワークのネットワーク範囲を定義する、IP ネットワーク範囲を入力します。

9. **[次ホップの種類]** として **[仮想アプライアンス]** を選択し、`172.16.1.4` とします。 別の IP 範囲を使用している場合は、その IP 範囲を使用します。

    ![[ルートの追加] ダイアログ ボックスに、テキスト ボックスに入力した 4 つの値が表示されています。](./media/azure-stack-network-howto-vnet-to-onprem/image12a.png)

10. **[保存]** を選択します。

## <a name="activate-the-fortigate-nva"></a>FortiGate NVA をアクティブにする

FortiGate NVA をアクティブにして各 NVA で IPSec VPN 接続を設定します。

各 FortiGate NVA をアクティブにするには、Fortinet が提供する有効なライセンス ファイルが必要です。 各 NVA がアクティブになるまで、NVA は機能 **しません**。 ライセンス ファイルを取得する方法と NVA をアクティブにする手順については、Fortinet ドキュメント ライブラリの「[ライセンスの登録とダウンロード](https://docs2.fortinet.com/vm/azure/FortiGate/6.2/azure-cookbook/6.2.0/19071/registering-and-downloading-your-license)」の記事を参照してください。

NVA をアクティブにしたら、NVA で IPSec VPN トンネルを作成します。

1. Azure Stack Hub ユーザー ポータルを開きます。

2. リソース グループを選択します。 フィルターに「`forti1`」と入力し、forti1 リソース グループをダブルクリックします。

3. リソース グループ ブレードのリソースの種類の一覧から、**forti1** 仮想マシンをダブルクリックします。

    ![forti1 仮想マシンの [概要] ページには、[リソース グループ] や [状態] などの forti1 の値が表示されています。](./media/azure-stack-network-howto-vnet-to-onprem/image13a.png)

4. 割り当てられた IP アドレスをコピーし、ブラウザーを開いてアドレス バーにその IP アドレスを貼り付けます。 サイトでは、セキュリティ証明書が信頼されていないという警告がトリガーされる場合があります。 そのまま続行します。

5. デプロイ中に指定した FortiGate の管理ユーザー名とパスワードを入力します。

    ![[ログイン] ダイアログ ボックスには、ユーザーとパスワード用のテキスト ボックスと [ログイン] ボタンがあります。](./media/azure-stack-network-howto-vnet-to-onprem/image14a.png)

6. **[System]\(システム\)**  >  **[Firmware]\(ファームウェア\)** を選択します。

7. 最新のファームウェアを示すボックスを選択します (例: `FortiOS v6.2.0 build0866`)。

    ![[ファームウェア] ダイアログ ボックスには、ファームウェアの識別子 "FortiOS v6.2.0 build0866" が表示されています。 リリース ノートへのリンクと 2 つのボタン ([Backup config and upgrade]\(コンフィグをバックアップしてアップグレード\) と [アップグレード]) があります。](./media/azure-stack-network-howto-vnet-to-onprem/image15a.png)

8. **[Backup config and upgrade]\(コンフィグをバックアップしてアップグレード\)**  >  **[Continue]\(続行\)** を選択します。

9. NVA でファームウェアが最新のビルドに更新され、再起動されます。 このプロセスには約 5 分かかります。 FortiGate Web コンソールにもう一度ログインします。

10. **[VPN]**  >  **[IPSec Wizard]\(IPSec ウィザード\)** をクリックします。

11. VPN の名前を入力します。たとえば、**[VPN Creation Wizard]\(VPN 作成ウィザード\)** で「`conn1`」と入力します。

12. **[This site is behind NAT]\(このサイトは NAT の背後にある\)** を選択します。

    ![[VPN Creation Wizard]\(VPN 作成ウィザード\) のスクリーンショットは、最初の手順である [VPN Setup]\(VPN の設定\) を示しています。 [Template Type]\(テンプレートの種類\) には [Site to Site]\(サイト間\)、[Remote Device Type]\(リモート デバイスの種類\) には [FortiGate]、[NAT Configuration]\(NAT 構成\) には [This site is behind NAT]\(このサイトは NAT の背後にある\) の値が選択されています。](./media/azure-stack-network-howto-vnet-to-onprem/image16a.png)

13. **[次へ]** を選択します。

14. 接続先となるオンプレミスの VPN デバイスのリモート IP アドレスを入力します。

15. **[Outgoing Interface]\(送信インターフェイス\)** として **[port1]** を選択します。

16. **[Pre-shared Key]\(事前共有キー\)** を選択し、事前共有キーを入力 (および記録) します。 

    > [!NOTE]  
    > このキーは、オンプレミスの VPN デバイスで接続を設定するために必要であるため、"*正確に*" 一致する必要があります。

    ![[VPN Creation Wizard]\(VPN 作成ウィザード\) のスクリーンショットは、2 番目の手順である [Authentication]\(認証\) を示し、選択した値が強調表示されています。](./media/azure-stack-network-howto-vnet-to-onprem/image17a.png)

17. **[次へ]** を選択します。

18. **[Local Interface]\(ローカル インターフェイス\)** として **[port2]** を選択します。

19. 次のローカル サブネットの範囲を入力します。
    - forti1:172.16.0.0/16
    - forti2:172.17.0.0/16

    別の IP 範囲を使用している場合は、その IP 範囲を使用します。

20. オンプレミスの VPN デバイスを介して接続する、オンプレミス ネットワークを表す適切なリモート サブネットを入力します。

    ![[VPN Creation Wizard]\(VPN 作成ウィザード\) のスクリーンショットは、3 番目の手順である [Policy & Routing]\(ポリシーとルーティング\) を示しています。 選択した値と入力した値が表示されています。](./media/azure-stack-network-howto-vnet-to-onprem/image18a.png)

21. **[作成]**

22. **[Network]\(ネットワーク\)**  >  **[Interfaces]\(インターフェイス\)** を選択します。

    ![インターフェイスの一覧に、2 つのインターフェイス (構成済みの port1 と構成されていない port2) が表示されています。 インターフェイスを作成、編集、削除するためのボタンがあります。](./media/azure-stack-network-howto-vnet-to-onprem/image19a.png)

23. **[port2]** をダブルクリックします。

24. **[Role]\(ロール\)** の一覧から **[LAN]** を、アドレス指定モードとして **[DHCP]** を選択します。

25. **[OK]** を選択します。

## <a name="configure-the-on-premises-vpn"></a>オンプレミスの VPNを構成する

IPSec VPN トンネルを作成するには、オンプレミスの VPN デバイスを構成する必要があります。 次の表は、オンプレミスの VPN デバイスを設定するために必要なパラメーターを示しています。 オンプレミスの VPN デバイスを構成する方法の詳細については、お使いのデバイスのドキュメントを参照してください。

| パラメーター | 値 |
| --- | --- |
| リモート ゲートウェイ IP | forti1 に割り当てられたパブリック IP アドレス (「[FortiGate NVA をアクティブにする](#activate-the-fortigate-nva)」を参照)。 |
| リモート IP ネットワーク | 172.16.0.0/16 (VNET にこの手順の IP 範囲を使用している場合)。 |
| 認証メソッド = 事前共有キー (PSK) | 手順 16 を参照。
| IKE のバージョン | 1 |
| IKE モード | メイン (ID 保護) |
| フェーズ 1 の提案アルゴリズム | AES128-SHA256、AES256-SHA256、AES128-SHA1、AES256-SHA1 |
| Diffie-hellman グループ | 14、5 |

## <a name="create-the-vpn-tunnel"></a>VPN トンネルを作成する

オンプレミスの VPN デバイスが適切に構成されると、VPN トンネルを確立できるようになります。

FortiGate NVA で、次のようにします。

1. forti1 の FortiGate Web コンソールで、 **[Monitor]\(モニター\)**  >  **[IPsec Monitor]\(IPsec モニター\)** に移動します。

    ![VPN 接続 conn1 のモニターが表示されています。 これは、対応するフェーズ 2 セレクターと同様にダウン中として表示されています。](./media/azure-stack-network-howto-vnet-to-onprem/image20a.png)

2. **conn1** を強調表示し、**[Bring Up]\(表示\)** > **[All Phase 2 Selectors]\(すべてのフェーズ 2 セレクター\)** を選択します。

    ![モニターとフェーズ 2 セレクターは両方とも稼働中として表示されています。](./media/azure-stack-network-howto-vnet-to-onprem/image21a.png)

## <a name="test-and-validate-connectivity"></a>接続のテストと検証

オンプレミスの VPN デバイスを介して、VNET ネットワークとオンプレミス ネットワークの間でルーティングを行うことができます。

次のように接続を検証します。

1. Azure Stack Hub の VNET とオンプレミス ネットワーク上のシステムに VM を作成します。 VM の作成手順については、「[クイック スタート:Azure Stack Hub ポータルを使用して Windows サーバー VM を作成します](./azure-stack-quick-windows-portal.md)。

2. Azure Stack Hub の VM を作成し、オンプレミスのシステムを準備する際には、次のことを確認してください。

-  Azure Stack Hub の VM が各 VNET の **InsideSubnet** に配置されていること。

-  オンプレミスのシステムが、IPSec 構成で定義されている定義済みの IP 範囲内のオンプレミス ネットワークに配置されていること。 また、オンプレミスの VPN デバイスのローカル インターフェイスの IP アドレスが、Azure Stack Hub の VNET ネットワークに到達できるルートとして、オンプレミスのシステムに確実に指定されているようにします (例: `172.16.0.0/16`)。

-  Azure Stack Hub の VM の作成時に NSG を適用 **しない** こと。 VM をポータルから作成している場合に、既定で追加される NSG を削除する必要がある場合があります。

-  オンプレミスのシステム OS と Azure Stack Hub の VM OS に、接続のテストに使用する通信を禁止する OS のファイアウォール規則が確実に含まれていないこと。 テスト目的の場合は、両方のシステムのオペレーティング システム内でファイアウォールを完全に無効にすることをお勧めします。

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub ネットワークの違いと考慮事項](azure-stack-network-differences.md)  
[Fortinet FortiGate を使用して Azure Stack Hub にネットワーク ソリューションを提供する](../operator/azure-stack-network-solutions-enable.md)  
