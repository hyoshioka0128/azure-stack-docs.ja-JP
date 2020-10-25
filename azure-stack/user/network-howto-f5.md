---
title: 2 つの Azure Stack Hub インスタンスに F5 をデプロイする
description: 2 つの Azure Stack Hub インスタンスに F5 をデプロイする方法について説明します。
author: mattbriggs
ms.topic: how-to
ms.date: 08/24/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 11/06/2019
ms.openlocfilehash: 3512584e4109d86a9ee6a0d71924d560b9ecba92
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90574212"
---
# <a name="how-to-deploy-f5-across-two-azure-stack-hub-instances"></a>2 つの Azure Stack Hub インスタンスに F5 をデプロイする方法

この記事では、2 つの Azure Stack Hub 環境に外部ロード バランサーを設定する手順について説明します。 この構成を使用して、さまざまなワークロードを管理できます。 この記事では、2 つの独立した Azure Stack Hub インスタンスにグローバルな負荷分散ソリューションとして F5 をデプロイします。 また、2 つのインスタンスを対象に NGINX サーバーで実行される負荷分散された Web アプリをデプロイします。 それらは、F5 仮想アプライアンスの高可用性のフェールオーバー ペアの内側で実行されます。

Azure Resource Manager テンプレートは、[f5-azurestack-gslb](https://github.com/Mikej81/f5-azurestack-gslb) GitHub リポジトリで入手できます。

## <a name="overview-of-load-balancing-with-f5"></a>F5 を使用した負荷分散の概要

F5 ハードウェア (ロード バランサー) は、Azure Stack Hub の外部にあっても、Azure Stack Hub がホストされるデータセンターの内部にあってもかまいません。 Azure Stack Hub には、2 つの異なる Azure Stack Hub デプロイ間でワークロードを負荷分散するためのネイティブな機能はありません。 F5 の BIG-IP 仮想エディション (VE) は、両方のプラットフォーム上で実行されます。 このセットアップにより、サポートするアプリケーション サービスのレプリケーションによって、Azure アーキテクチャと Azure Stack Hub アーキテクチャの間のパリティがサポートされます。 1 つの環境でアプリを開発し、別の環境に移動することができます。 また、同じ BIG-IP 構成、ポリシー、アプリケーション サービスを含む、運用対応の Azure Stack Hub 全体をミラー化することもできます。 このアプローチにより、アプリケーションのリファクタリングとテストに何時間もかかることがなくなり、コードの作成を進めることができます。

アプリケーションとそのデータをセキュリティで保護することは、多くの場合、アプリをパブリック クラウドに移動する開発者にとって重要なことです。 これを問題にする必要はありません。 開発者が Azure Stack Hub 環境でアプリを構築する一方で、セキュリティ アーキテクトは F5 の Web アプリケーション ファイアウォール (WAF) で必要な設定を構成します。 アプリケーションが業界をリードする同じ WAF によって保護されることはわかっており、スタック全体を Azure Stack Hub にレプリケートすることができます。 ポリシーとルールセットは同じであり、異なる WAF を採用すると発生する可能性がある、セキュリティの抜け道や脆弱性が発生することはありません。

Azure Stack Hub には、Azure とは別のマーケットプレースがあります。 特定の項目だけが追加されます。 この場合、各 Azure Stack Hub に新しいリソース グループを作成し、既に使用可能な F5 仮想アプライアンスをデプロイします。 それ以降、両方の Azure Stack Hub インスタンス間をネットワーク接続するためには、**パブリック IP** アドレスが必要です。 基本的に、これらはどちらもアイランドであり、**パブリック IP** アドレスにより両方の場所で通信できるようになります。

## <a name="prerequisites-for-big-ip-ve"></a>BIG-IP VE の前提条件

-  **F5 BIG-IP VE - ALL (BYOL, 2 Boot Locations)** を各 Azure Stack Hub Marketplace にダウンロードします。 それらをポータルで使用できない場合は、クラウド オペレーターに問い合わせてください。

-  Azure Resource Manager テンプレートは、次の GitHub リポジトリにあります: https://github.com/Mikej81/f5-azurestack-gslb 。

## <a name="deploy-f5-big-ip-ve-on-each-instance"></a>F5 BIG-IP VE を各インスタンスにデプロイする

Azure Stack Hub インスタンス A とインスタンス B にデプロイします。

1. Azure Stack Hub ユーザー ポータルにサインインします。

2. **[+ リソースの作成]** を選択します。

3. 「`F5`」と入力して、マーケットプレースを検索します。

4. **F5 BIG-IP VE – ALL (BYOL, 2 Boot Locations)** を選択します。

    ![[ダッシュボード > 新規 > Marketplace > すべて > F5 BIG-IP VE - ALL (BYOL, 2 Boot Locations)] ダイアログ ボックスの検索ボックスに "f5" と示されています。 1 件の検索結果は "F5 BIG-IP VE - ALL (BYOL, 2 Boot Locations)" です。](./media/network-howto-f5/image1.png)

5. 次のページの下部にある **[作成]** を選択します。

    ![[F5 BIG-IP VE - ALL (BYOL, 2 Boot Locations)] ダイアログ ボックスに、BIG-IP VE に関する情報と、お使いのライセンスに応じてデプロイできるモジュールが表示されます。 [作成] ボタンがあります。](./media/network-howto-f5/image2.png)

6. **F5-GSLB** という名前で新しいリソース グループを作成します。

7. デプロイを完了します。次の値は例です。

    ![[Microsoft.Template] ダイアログ ボックスの [入力] ページに、デプロイ例の値を含む、[VIRTUALMACHINENAME] や [ADMINUSERNAME] などの 15 個のテキスト ボックスが表示されています。](./media/network-howto-f5/image3.png)

8. デプロイが正常に完了したことを確認します。

    ![[Microsoft.Template] ダイアログ ボックスの [概要] ページに "デプロイが完了しました" と表示され、デプロイに関する詳細が表示されています。](./media/network-howto-f5/image4.png)

    > [!NOTE]  
    > 各 BIG-IP のデプロイには約 20 分かかります。

## <a name="configure-big-ip-appliances"></a>BIG-IP アプライアンスを構成する

Azure Stack Hub A と B の両方で、次の手順に従う必要があります。

1. Azure Stack Hub インスタンス A で Azure Stack Hub ユーザー ポータルにサインインし、BIG-IP テンプレートのデプロイから作成されたリソースを確認します。

    ![[F5-GSLB] ダイアログ ボックスの [概要] ページに、デプロイされたリソースと関連情報が表示されています。](./media/network-howto-f5/image18.png)

2. [BIG-IP の構成項目](https://clouddocs.f5.com/training/community/dns/html/class1/class1.html)については F5 の指示に従います。 

3. Azure Stack Hub インスタンス A と B にデプロイされた両方のアプライアンスでリッスンするように、BIG-IP Wide IP List を構成します。手順については、「[BIG-IP GTM の構成](https://techdocs.f5.com/kb/en-us/products/big-ip_gtm/manuals/product/gtm-concepts-11-5-0/4.html)」を参照してください。


4. BIG-IP アプライアンスのフェールオーバーを検証します。 テスト システムで、次のものを使用するように DNS サーバーを構成します。
    - Azure Stack Hub インスタンス A = `f5stack1-ext` パブリック IP アドレス
    - Azure Stack Hub インスタンス B = `f5stack1-ext` パブリック IP アドレス

5. `www.contoso.com` を参照すると、ブラウザーに NGINX の既定ページが読み込まれます。

## <a name="create-a-dns-sync-group"></a>DNS 同期グループを作成する

1. ルート アカウントを有効にして、信頼を確立します。 「[システム メンテナンス アカウントのパスワードの変更 (11.x - 15.x)](https://support.f5.com/csp/article/K13121)」の手順に従います。 信頼 (証明書交換) を設定した後、ルート アカウントを無効にします。

1. BIG-IP にサインインし、DNS 同期グループを作成します。 手順については、「[BIG-IP DNS 同期グループの作成](https://f5-dns-automation-demo-12-1-x.readthedocs.io/en/latest/lab2/sync-group.html)」を参照してください。

    > [!NOTE]  
    > BIP-IP アプライアンスのローカル IP アドレスは、**F5-GSLB** リソース グループにあります。 ネットワーク インターフェイスは "f5stack1-ext" であり、(アクセスに応じて) パブリック IP アドレスまたはプライベート IP アドレスに接続します。

    ![[DNS >> GSLB : Data Centers : Data Center List]\(DNS >> GSLB : データ センター : データセンターの一覧\) ダイアログ ボックスに、データ センターと状態が表示されています。 選択したデータ センターに適用する [Enable]\(有効\)、[Disable]\(無効\)、[Delete]\(削除\) の各ボタンがあります。](./media/network-howto-f5/image5.png)
          
    ![[DNS >> GSLB : Servers : Server List]\(DNS >> GSLB : サーバー : サーバーの一覧\) ダイアログ ボックスに、サーバーと状態が表示されています。 選択したサーバーに適用する [Enable]\(有効\)、[Disable]\(無効\)、[Delete]\(削除\)、[Reconnect]\(再接続\) の各ボタンがあります。](./media/network-howto-f5/image6.png)

1. 新しいリソース グループ **F5-GSLB** を選択し、**f5stack1** 仮想マシンを選択して、 **[Settings]\(設定\)** で **[Networking]\(ネットワーク\)** を選択します。

## <a name="post-install-configurations"></a>インストール後の構成

インストールした後は、Azure Stack Hub の NSG を構成し、ソース IP アドレスをロック ダウンする必要があります。

1. 信頼が確立された後、ポート 22 を無効にします。

2. システムがオンラインになっている場合は、ソース NSG をブロックします。 管理ソースに対して管理 NSG をロックする必要があります。同期のために他のインスタンスに対して外部 (4353/TCP) NSG をロックする必要があります。仮想サーバーのアプリケーションがデプロイされるまでは、443 もロックする必要があります。

3. GTM_DNS のルールがポート 53 (DNS) の入力トラフィックを許可するように設定され、BIG-IP リゾルバーが 1 回動作を開始します。 リスナーが作成されます。

    ![[ネットワーク インターフェイス] ダイアログ ボックスの [fStack1-ext] ページに、fstack1-ext インターフェイスに関する情報と、その NSG である fstack1-ext-nsg に関する情報が表示されています。 受信ポートの規則または送信ポートの規則を表示するタブがあります。](./media/network-howto-f5/image7.png)

4. Azure Stack Hub 環境内に基本的な Web アプリケーション ワークロードをデプロイし、BIG-IP の内側の負荷を分散します。 NGNIX サーバーを使用する例については、「[NGINX と NGINX Plus を Docker にデプロイする](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-docker/)」を参照してください。

    > [!NOTE]  
    > Azure Stack Hub A と Azure Stack Hub B の両方に NGNIX のインスタンスをデプロイします。

5. 各 Azure Stack Hub インスタンス内の Ubuntu VM 上 Docker コンテナーに NGINX をデプロイした後、サーバー上の既定の Web ページに接続できることを確認します。

    ![[Welcome to nginx!]\(nginx へようこそ\) ページに、nginx Web サーバーが正常にインストールされたことと、追加の構成が必要であることが表示されています。 サポート情報へのリンクが 2 つあります。](./media/network-howto-f5/image8.png)

6. BIG-IP アプライアンスの管理インターフェイスにサインインします。 この例では、**f5-stack1-ext** のパブリック IP アドレスを使用します。

    ![ユーザー名とパスワードを求める BIG-IP 構成ユーティリティのログイン画面。](./media/network-howto-f5/image9.png)

7. BIG-IP 経由で NGINX へのアクセスを公開します。
    
    -  このタスクでは、WordPress アプリケーションへの受信インターネット アクセスを許可するように、仮想サーバーとプールで BIG-IP を構成します。 まず、NGINX インスタンスのプライベート IP アドレスを識別する必要があります。

8. Azure Stack Hub のユーザー ポータルにサインインします。 

9. お使いの NGINX ネットワーク インターフェイスを選択します。

    ![[ダッシュボード > リソース グループ > NGINX > ubuntu2673] ダイアログ ボックスの [概要] ページに、ubuntu2673 ネットワーク インターフェイスに関する情報が表示されています。](./media/network-howto-f5/image10.png)

10. BIG-IP コンソールで、 **[Local traffic]\(ローカル トラフィック\) > [Pools]\(プール\) > [Pool List]\(プール リスト\)** に移動し、 **[+]** を選択します。 次の表の値を使用して、プールを構成します。 他のすべてのフィールドは既定値のままにします。

    ![左側のペインに、新しいプールの作成をナビゲートする機能が用意されています。 右側のペインのタイトルは、[Local Traffic >> Pools : Pool List >> New Pool]\(ローカル トラフィック >> プール : プール一覧 >> 新しいプール\) であり、新しいプールに関する情報を指定する機能が用意されています。 [Finished]\(完了\) ボタンがあります。](./media/network-howto-f5/image11.png)
    
    | Key | 値 |
    | --- | --- |
    | 名前 | NGINX_Pool |
    | Health Monitor (正常性モニター) | HTTPS |
    | Node Name (ノード名) | NGINX |
    | Address | \<your NGINX private IP address> |
    | Service Port (サービス ポート) | 443 |

11. **[完了]** を選択します。 正しく構成すると、プールの状態が緑になります。

    ![右側のペインのタイトルは、[Local Traffic >> Pools : Pool List]\(ローカル トラフィック >> プール : プール一覧\) であり、新しく作成されたプールが一覧の唯一のエントリです。](./media/network-howto-f5/image12.png)

    次に、仮想サーバーを構成する必要があります。 そのためには、まず、F5 BIG-IP のプライベート IP アドレスを確認する必要があります。

12. BIG-IP コンソールで、**[Network]\(ネットワーク\) > [Self IPs]\(セルフ IP\)** に移動し、IP アドレスを記録します。

    ![左側のペインに、セルフ IP の表示をナビゲートする機能が用意されています。 右側のペインのタイトルは、[Network >> Self IPs]\(ネットワーク >> セルフ IP\) です。 2 つのセルフ IP が表示され、最初の self_2nic が強調表示されています。](./media/network-howto-f5/image13.png)

13. **[Local Traffic]\(ローカル トラフィック\)**  >  **[Virtual Servers]\(仮想サーバー\)**  >  **[Virtual Server List]\(仮想サーバー一覧\)** に移動し、 **[+]** を選択して、仮想サーバーを作成します。 次の表の値を使用して、プールを構成します。 他のすべてのフィールドは既定値のままにします。

    | Key | 値 |
    | --- | --- |
    |名前 | NGINX |
    |Destination Address (宛先アドレス) | \<Self IP address of the BIG-IP> |
    |Service Port (サービス ポート) | 443 |
    |SSL Profile (Client) (SSL プロファイル (クライアント)) | clientssl |
    |Source Address Translation (ソース アドレス変換) | Auto Map (自動マップ) |
        
    ![左側のペインは、右側のペインを [Local Traffic >> Virtual Servers : Virtual Server List >> NGINX]\(ローカル トラフィック >> 仮想サーバー >> 仮想サーバー一覧 >> NGINX\) にナビゲートするために使用されます。ここで、必要な情報を入力します。](./media/network-howto-f5/image14.png)

    ![このページには、追加情報を入力する機能が用意されています。 [Update]\(更新\) と [Delete]\(削除\) ボタンがあります。](./media/network-howto-f5/image15.png)

14. これで、NGINX アプリケーション用の BIG-IP の構成は完了です。 適切な機能を確認するには、サイトを参照し、F5 の統計を確認します。

15. ブラウザーで `https://<F5-public-VIP-IP>` を開き、NGINX の既定のページが表示されることを確認します。

    ![[Welcome to nginx!]\(nginx へようこそ\) ページは、nginx Web サーバーが正常にインストールされたことと、追加の構成が必要であることを示しています。 サポート情報へのリンクが 2 つあります。](./media/network-howto-f5/image16.png)

16. 次に、**[Statistics]\(統計\) > [Module Statistics]\(モジュールの統計\) > [Local Traffic]\(ローカル トラフィック\)** に移動し、仮想サーバーの統計情報を調べて、トラフィック フローを確認します。

17. **[Statistics Type]\(統計の種類\)** で、**[Virtual Servers]\(仮想サーバー\)** を選択します。

    ![左側のペインの操作によって、右側のペインが [Statistics >> Module Statistics : Local Traffic >> Virtual Servers]\(統計 >> モジュールの統計 : ローカル トラフィック >> 仮想サーバー\) にナビゲートされ、一覧に NGINX 仮想サーバーとその他の内容が表示されています。 NGINX が強調表示されています。](./media/network-howto-f5/image17.png)


## <a name="for-more-information"></a>詳細情報

F5 の使用に関するリファレンス記事がいくつかあります。

- [BIG-IP DNS を使用したデータセンターの可用性サービス](https://clouddocs.f5.com/training/community/dns/html/class3/class3.html)
- [HTTP アプリケーションを使用した BIG-IP システムの展開](https://www.f5.com/content/dam/f5/corp/global/pdf/deployment-guides/iapp-http-dg.pdf)
- [GSLB 用のワイド IP の作成](https://clouddocs.f5.com/training/community/big-iq-cloud-edition/html/class10/module2/lab1.html)

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub ネットワークの違いと考慮事項](azure-stack-network-differences.md) 
