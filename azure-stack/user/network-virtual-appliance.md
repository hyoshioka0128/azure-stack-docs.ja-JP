---
title: Azure Stack Hub でのネットワーク仮想アプライアンスに関する問題をトラブルシューティングする
description: Microsoft Azure Stack Hub でのネットワーク仮想アプライアンス (NVA) の使用時に発生する、VM または VPN の接続の問題をトラブルシューティングします。
author: sethmanheim
ms.author: sethm
ms.date: 09/08/2020
ms.topic: article
ms.reviewer: sranthar
ms.lastreviewed: 05/12/2020
ms.openlocfilehash: 293e445343acfe13a0be2cabab6cb1577c3941a2
ms.sourcegitcommit: b147d617c32cea138b5bd4bab568109282e44317
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/11/2020
ms.locfileid: "90010885"
---
# <a name="troubleshoot-network-virtual-appliance-problems"></a>ネットワーク仮想アプライアンスに関する問題をトラブルシューティングする

Azure Stack Hub でネットワーク仮想アプライアンス (NVA) を使用する仮想マシンまたは VPN で、接続の問題が発生する場合があります。

この記事は、NVA 構成のための Azure Stack Hub の基本プラットフォーム要件を確認するのに役立ちます。

NVA と、Azure Stack Hub プラットフォームとの統合に対するテクニカル サポートは、NVA のベンダーによって提供されます。

> [!NOTE]
> NVA が関係する接続やルーティングの問題がある場合は、直接 [NVA のベンダーにお問い合わせ](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines)いただく必要があります。

Azure Stack Hub での NVA に関する問題をこの記事で対処できない場合は、[Azure Stack Hub サポート チケット](../operator/azure-stack-manage-basics.md#where-to-get-support)を作成してください。

## <a name="checklist-for-troubleshooting-with-an-nva-vendor"></a>NVA のベンダーによるトラブルシューティングを行うためのチェックリスト

- NVA の VM ソフトウェアの更新。
- サービス アカウントのセットアップと機能。
- トラフィックを NVA にダイレクトする仮想ネットワーク サブネット上のユーザー定義ルート (UDR)。
- NVA からトラフィックをダイレクトする仮想ネットワーク サブネット上の UDR。
- NVA 内のテーブルとルールのルーティング (例: NIC1 から NIC2)。
- ネットワーク トラフィックの送受信を確認するための NVA NIC のトレース。

## <a name="basic-troubleshooting-steps"></a>基本的なトラブルシューティングの手順

1. 基本構成の確認。
2. NVA のパフォーマンスの確認。
3. 高度なネットワークのトラブルシューティングの実行。

## <a name="check-the-minimum-configuration-requirements-for-nvas-on-azure"></a>Azure 上の NVA に対する最小構成要件の確認

各 NVA が Azure Stack Hub 上で正常に機能するには、基本的な構成要件を満たす必要があります。 このセクションでは、これらの基本構成を確認するための手順を示します。 詳細については、[NVA のベンダーにお問い合わせください](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines)。

> [!IMPORTANT]
> パケットで S2S トンネルが使用される場合は、追加のヘッダーでさらにカプセル化されます。 このカプセル化により、各パケットの全体的なサイズが増加します。
>
> このシナリオでは、TCP MSS を 1,350 バイトで固定する必要があります。 お使いの VPN デバイスで MSS クランプがサポートされていない場合は、代わりにトンネル インターフェイスの MTU を 1,400 バイトに設定できます。
>
> 詳細については、[仮想ネットワークの TCP/IP パフォーマンスのチューニング](/azure/virtual-network/virtual-network-tcpip-performance-tuning)に関する記事を参照してください。

### <a name="check-whether-ip-forwarding-is-enabled-on-the-nva"></a>NVA で IP 転送が有効かどうかを確認する

#### <a name="use-the-azure-stack-hub-portal"></a>Azure Stack Hub ポータルを使用する

1. Azure Stack Hub ポータルで NVA リソースを検索し、 **[ネットワーク]** を選択して、ネットワーク インターフェイスを選択します。
2. **[ネットワーク インターフェイス]** ページで、 **[IP 構成]** を選択します。
3. IP 転送が有効になっていることを確認します。

#### <a name="use-powershell"></a>PowerShell の使用

1. 次のコマンドを実行します。 山かっこ内の値をご自分の情報に置き換えてください。

   ```powershell
   Get-AzureRMNetworkInterface -ResourceGroupName <ResourceGroupName> -Name <NIC name>
   ```

2. **EnableIPForwarding** プロパティを確認します。

3. IP 転送が有効になっていない場合は、次のコマンドを実行して有効にします。

   ```powershell
   $nic2 = Get-AzureRMNetworkInterface -ResourceGroupName <ResourceGroupName> -Name <NIC name>
   $nic2.EnableIPForwarding = 1
   Set-AzureRMNetworkInterface -NetworkInterface $nic2
   Execute: $nic2 #and check for an expected output:
   EnableIPForwarding   : True
   NetworkSecurityGroup : null
   ```

### <a name="check-whether-traffic-can-be-routed-to-the-nva"></a>トラフィックを NVA にルーティングできるかどうかを確認する

1. NVA にトラフィックをリダイレクトするように構成されている VM を見つけます。
1. NVA が次ホップであることを確認するには、`Tracert <Private IP of NVA>` (Windows の場合) または `Traceroute <Private IP of NVA>` を実行します。
1. NVA が次ホップとして一覧表示されていない場合は、Azure Stack Hub のルート テーブルを確認して更新します。

ゲストレベルのオペレーティング システムによっては、ICMP トラフィックをブロックするためのファイアウォール ポリシーが配置されていることがあります。 これらのファイアウォール ルールを機能させるために、前のコマンドで更新する必要があります。

### <a name="check-whether-traffic-can-reach-the-nva"></a>トラフィックで NVA に到達できるかどうかを確認する

1. NVA に接続する必要がある VM を見つけます。
1. ネットワーク セキュリティ グループ (NSG) によってトラフィックがブロックされているかどうかを確認します。 Windows の場合、`ping` (ICMP) または `Test-NetConnection <Private IP of NVA>` (TCP) を実行します。 Linux の場合、`Tcpping <Private IP of NVA>` を実行します。
1. NSG によってトラフィックがブロックされている場合は、トラフィックを許可するように変更します。

### <a name="check-whether-the-nva-and-vms-are-listening-for-expected-traffic"></a>NVA と VM で予期されるトラフィックがリッスンされているかどうかを確認する

1. RDP または SSH を使用して NVA に接続した後、次のコマンドを実行します。

   **Windows**

   ```shell
   netstat -an
   ```

   **Linux**

   ```shell
   netstat -an | grep -i listen
   ```

1. 結果に示されている NVA ソフトウェアによって使用されている TCP ポートを探します。 それらを確認できない場合は、NVA と VM 上のアプリケーションを、これらのポートに到達するトラフィックをリッスンして応答するように構成してします。 [サポートが必要な場合は NVA のベンダーにお問い合わせください](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines)。

## <a name="check-nva-performance"></a>NVA のパフォーマンスの確認

### <a name="validate-vm-cpu-usage"></a>VM の CPU 使用率を確認する

CPU 使用率が 100 パーセントに近くなると、ネットワーク パケットの破棄に影響する問題が発生する可能性があります。

CPU のスパイク時に、高い CPU の原因となっているゲスト VM 上のプロセスを調べます。 可能であれば、使用率を軽減します。

また、場合によっては、VM の SKU サイズを増やすか、仮想マシン スケール セットであれば、インスタンス数を増やす必要があります。

サポートが必要な場合は、[NVA のベンダーにお問い合わせください](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines)。

### <a name="validate-vm-network-statistics"></a>VM ネットワークの統計を検証する

VM ネットワークでスパイクが使用される場合、または使用率が高い期間が示される場合は、VM の SKU サイズを大きくしてスループットを向上させることを検討してください。

## <a name="advanced-network-administrator-troubleshooting"></a>高度なネットワーク管理者のトラブルシューティング

### <a name="capture-a-network-trace"></a>ネットワーク トレースをキャプチャする

[`PsPing`](/sysinternals/downloads/psping) または `Nmap` の実行中に、ソース VM、宛先 VM、および NVA で、同時ネットワーク トレースをキャプチャします。 その後、トレースを停止します。

1. 同時ネットワーク追跡をキャプチャするには、次のコマンドを実行します。

   **Windows**

   ```shell
   netsh trace start capture=yes tracefile=c:\server_IP.etl scenario=netconnection
   ```

   **Linux**

   ```shell
   sudo tcpdump -s0 -i eth0 -X -w vmtrace.cap
   ```

2. ソース VM から宛先 VM に対して `PsPing` または `Nmap` を使用します。 たとえば、`PsPing 10.0.0.4:80` や `Nmap -p 80 10.0.0.4` などです。

3. **tcpdump** か自分で選んだパケット アナライザーを利用し、宛先 VM からネットワーク トレースを開きます。 `PsPing` または `Nmap` を実行できるソース VM の IP に表示フィルターを適用します。 Windows での **netmon** の例は `IPv4.address==10.0.0.4`です。 Linux での例は `tcpdump -nn -r vmtrace.cap src` や `dst host 10.0.0.4` などです。

### <a name="analyze-traces"></a>トレースの分析

バックエンド VM トレースにパケットが入ってこない場合は、NSG または UDR により妨害されているか、NVA ルーティング テーブルが正しくない可能性があります。

パケットを確認できるが、応答がない場合は、VM アプリケーションまたはファイアウォールの問題に対処する必要があります。

サポートが必要な場合は、[NVA のベンダーにお問い合わせください](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines)。

### <a name="create-a-support-ticket"></a>サポート チケットの作成

上記の手順で問題が解決しない場合は、[サポート チケット](../operator/azure-stack-manage-basics.md#where-to-get-support)を作成し、[オンデマンド ログ収集ツール](../operator/azure-stack-diagnostic-log-collection-overview.md)を使用してログを提供してください。
