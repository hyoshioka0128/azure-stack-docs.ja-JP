---
title: MDC 向けの Azure Stack のネットワーク デプロイ
description: MDC デバイス向けの Azure Stack のネットワーク デプロイについて説明します。
author: PatAltimore
ms.service: azure-stack
ms.topic: conceptual
ms.date: 01/17/2020
ms.author: patricka
ms.reviewer: shisab
ms.lastreviewed: 01/17/2020
ms.openlocfilehash: a0de01ce240248767ebfd8e558a83408c84282be
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97872213"
---
# <a name="network-deployment"></a>ネットワーク展開

このトピックでは、TOR スイッチへのアクセス許可、IP アドレスの割り当て、およびその他のネットワーク デプロイ タスクについて説明します。

## <a name="plan-configuration-deployment"></a>構成のデプロイを計画する

以下のセクションでは、アクセス許可と IP アドレスの割り当てについて説明します。

### <a name="physical-switch-access-control-list"></a>物理スイッチのアクセス制御リスト

Microsoft では、Azure Stack ソリューションを保護するために、TOR スイッチにアクセス制御リスト (ACL) を実装しました。 このセクションでは、このセキュリティの実装方法について説明します。 次の表は、Azure Stack ソリューション内のすべてのネットワークのソースと宛先を示しています。

![TOR スイッチのアクセス制御リストの図](media/network-deployment/acls.png)

次の表では、ACL の記載と Azure Stack ネットワークを関連付けています。

| BMC Mgmt Internal                            | トラフィックは内部のみに制限されます。                                                                                                                                      |
|----------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| BMC Mgmt External                            | ACL によって境界デバイスを超えるアクセスが許可されます。                                                                                                                            |   |   |   |   |   |
| Extended Storage Mgmt                        | 拡張ストレージ システムの専用管理インターフェイス                                                                                                           |   |   |   |   |   |
| Switch Mgmt                                  | 専用のスイッチ管理インターフェイス。                                                                                                                                   |   |   |   |   |   |
| "Azure Stack Infrastructure"                 | Azure Stack Infrastructure サービスと VM の制限付きネットワーク                                                                                                          |   |   |   |   |   |
| Azure Stack Infrastructure Public (PEP/ERCS) | Azure Stack Protected Endpoint、Emergency Recovery Console Server。 お客様は、ACL を開いて、データセンター管理ネットワークへのトラフィックを許可することができます。                        |   |   |   |   |   |
| Tor1、Tor2 RouterIP                           | SLB とスイッチまたはルーター間の BGP ピアリングに使用されるスイッチのループバック インターフェイス。 お客様は、境界でこれらの IP をファイアウォールで保護することを選択できます。 |   |   |   |   |   |
| ストレージ                                      | リージョン外にルーティングされないプライベート IP                                                                                                                             |   |   |   |   |   |
| Internal VIPs                                | リージョン外にルーティングされないプライベート IP                                                                                                                             |   |   |   |   |   |
| Public VIPs                                  | ネットワーク コントローラーによって管理されるテナント ネットワーク アドレス空間。                                                                                                           |   |   |   |   |   |
| Public Admin VIPs                            | 内部 VIP および Azure Stack インフラストラクチャとの通信に必要なテナント プール内のアドレスの少数のサブセット                                                    |   |   |   |   |   |
| Permitted Networks                           | お客様が定義したネットワーク。                                                                                                                                                 |   |   |   |   |   |
| 0.0.0.0                                      | Azure Stack の観点からは、0.0.0.0 が境界デバイスです。                                                                                                         |   |   |   |   |   |
| **Permit**                                   | トラフィックの許可は有効になっていますが、SSH アクセスは既定でブロックされています。                                                                                                           |   |   |   |   |   |
| **No Route**                                     | ルートが Azure Stack 環境の外部に伝達されることはありません。                                                                                                         |   |   |   |   |   |
| **MUX ACL**                                      | Azure Stack MUX ACL が使用されます。                                                                                                                                       |   |   |   |   |   |
| **N/A**                                          | VLAN ACL の一部ではありません。                                                                                                                                                 |   |   |   |   |   |
|                                              |                                                                                                                                                                           |   |   |   |   |   |

### <a name="ip-address-assignments"></a>IP アドレスの割り当て

[Deployment Worksheet]\(デプロイ ワークシート\) では、Azure Stack のデプロイ プロセスをサポートするために次のネットワーク アドレスを指定するように求められます。 デプロイ チームは、[Deployment Worksheet]\(デプロイ ワークシート\) ツールを使用して、IP ネットワークをシステムに必要なすべての小さなネットワークに分割します。

この例では、[Deployment Worksheet]\(デプロイ ワークシート\) の [ネットワーク設定] タブに次の値を入力します。

-   BMC Network (BMC ネットワーク):10.193.132.0 /27

-   Private Network Storage Network & Internal VIPs (プライベート ネットワーク ストレージ ネットワークと内部 VIP):11.11.128.0 /20

-   Infrastructure Network (インフラストラクチャ ネットワーク):12.193.130.0 /24

-   Public Virtual IP (VIP) Network (パブリック仮想 IP (VIP) ネットワーク):13.200.132.0 /24

-   Switch Infrastructure Network (スイッチ インフラストラクチャ ネットワーク):10.193.132.128 /26

[Deployment Worksheet]\(デプロイ ワークシート\) ツールで Generate 関数を実行すると、スプレッドシートに 2 つの新しいタブが作成されます。 1 つ目のタブは [Subnet Summary]\(サブネットの概要\) であり、システムに必要なすべてのネットワークを作成するためにスーパーネットがどのように分割されたかが示されます。 以下の例は、このタブに標準される列のごく一部です。実際の結果には、次の各ネットワークの詳細が表示されます。

| **ラック** | **サブネットの種類** | **名前**                                   | **IPv4 サブネット**   | **IPv4 アドレス** |
|----------|-----------------|--------------------------------------------|-------------------|--------------------|
| 境界線   | P2P リンク        | P2P_Border/Border1_To_Rack1/TOR1           | 10.193.132.128/30 | 4                  |
| 境界線   | P2P リンク        | P2P_Border/Border1_To_Rack1/TOR2           | 10.193.132.132/30 | 4                  |
| 境界線   | P2P リンク        | P2P_Border/Border2_To_Rack1/TOR1           | 10.193.132.136/30 | 4                  |
| 境界線   | P2P リンク        | P2P_Border/Border2_To_Rack1/TOR2           | 10.193.132.140/30 | 4                  |
| 境界線   | P2P リンク        | P2P_Rack1/TOR1_To_Rack1/BMC                | 10.193.132.144/30 | 4                  |
| 境界線   | P2P リンク        | P2P_Rack1/TOR2_To_Rack1/BMC                | 10.193.132.148/30 | 4                  |
| Rack1    | ループバック        | Loopback0_Rack1_TOR1                       | 10.193.132.152/32 | 1                  |
| Rack1    | ループバック        | Loopback0_Rack1_TOR2                       | 10.193.132.153/32 | 1                  |
| Rack1    | ループバック        | Loopback0_Rack1_BMC                        | 10.193.132.154/32 | 1                  |
| Rack1    | P2P リンク        | P2P_Rack1/TOR1-ibgp-1_To_Rack1/TOR2-ibgp-1 | 10.193.132.156/30 | 4                  |
| Rack1    | P2P リンク        | P2P_Rack1/TOR1-ibgp-2_To_Rack1/TOR2-ibgp-2 | 10.193.132.160/30 | 4                  |
| Rack1    | VLAN            | BMCMgmt                                    | 10.193.132.0/27   | 32                 |
| Rack1    | VLAN            | SwitchMgmt                                 | 10.193.132.168/29 | 8                  |
| Rack1    | VLAN            | CL01-RG01-SU01-Storage                     | 11.11.128.0/25    | 128                |
| Rack1    | VLAN            | CL01-RG01-SU01-Infra                       | 12.193.130.0/24   | 256                |
| Rack1    | その他           | CL01-RG01-SU01-VIPS                        | 13.200.132.0/24   | 256                |
| Rack1    | その他           | CL01-RG01-SU01-InternalVIPS                | 11.11.128.128/25  | 128                |

2 つ目のタブは **[IP Address Usage]\(IP アドレスの使用状況\)** であり、IP がどのように消費されているかが示されます。

#### <a name="bmc-network"></a>BMC ネットワーク

BMC ネットワークのスーパーネットには、少なくとも /26 ネットワークが必要です。 ゲートウェイには、まずネットワーク内の最初の IP、次にラック内の BMC デバイスが使用されます。 ハードウェア ライフサイクル ホストには、このネットワークに割り当てられた複数のアドレスがあり、ラックの配置、監視、およびサポートに使用できます。 これらの IP は、3 つのグループに配布されます。DVM、InternalAccessible、および ExternalAccessible です。

- Rack (ラック):Rack1         
- 名前:BMCMgmt      

| **割当先**      | **[IPv4 アドレス]** |
|----------------------|------------------|
| ネットワーク              | 10.193.132.0     |
| Gateway              | 10.193.132.1     |
| HLH-BMC              | 10.193.132.2     |
| AzS-Node01           | 10.193.132.3     |
| AzS-Node02           | 10.193.132.4     |
| AzS-Node03           | 10.193.132.5     |
| AzS-Node04           | 10.193.132.6     |
| ExternalAccessible-1 | 10.193.132.19    |
| ExternalAccessible-2 | 10.193.132.20    |
| ExternalAccessible-3 | 10.193.132.21    |
| ExternalAccessible-4 | 10.193.132.22    |
| ExternalAccessible-5 | 10.193.132.23    |
| InternalAccessible-1 | 10.193.132.24    |
| InternalAccessible-2 | 10.193.132.25    |
| InternalAccessible-3 | 10.193.132.26    |
| InternalAccessible-4 | 10.193.132.27    |
| InternalAccessible-5 | 10.193.132.28    |
| CL01-RG01-SU01-DVM00 | 10.193.132.29    |
| HLH-OS               | 10.193.132.30    |
| ブロードキャスト            | 10.193.132.31    |

#### <a name="storage-network"></a>記憶域ネットワーク

ストレージ ネットワークはプライベート ネットワークであり、ラックを超えてルーティングするためのものではありません。 これはプライベート ネットワーク スーパーネットの前半であり、次の表に示すように分散されたスイッチによって使用されます。 このゲートウェイは、サブネット内の最初の IP です。 内部 VIP に使用される後半は、Azure Stack SLB によって管理されるアドレスのプライベート プールであり、[IP Address Usage]\(IP アドレスの使用状況\) タブには表示されません。これらのネットワークは Azure Stack をサポートしており、TOR スイッチには ACL があり、これらのネットワークがソリューションの外部でアドバタイズまたはアクセスされないように防いでいます。

- Rack (ラック):Rack1                                    
- 名前:CL01-RG01-SU01-Storage 

| **割当先**              | **[IPv4 アドレス]** |
|------------------------------|------------------|
| ネットワーク                      | 11.11.128.0      |
| Gateway                      | 11.11.128.1      |
| TOR1                         | 11.11.128.2      |
| TOR2                         | 11.11.128.3      |
| ブロードキャスト                    | 11.11.128.127    |

#### <a name="azure-stack-infrastructure-network"></a>Azure Stack インフラストラクチャ ネットワーク

インフラストラクチャ ネットワーク スーパーネットには /24 ネットワークが必要であり、これは [Deployment Worksheet]\(デプロイ ワークシート\) ツールの実行後も引き続き /24 です。 このゲートウェイは、サブネットの最初の IP になります。

- Rack (ラック):Rack1               
- 名前:CL01-RG01-SU01-Infra 

| **割当先**            | **[IPv4 アドレス]** |
|----------------------------|------------------|
| ネットワーク                    | 12.193.130.0     |
| Gateway                    | 12.193.130.1     |
| TOR1                       | 12.193.130.2     |
| TOR2                       | 12.193.130.3     |
| ブロードキャスト                  | 12.193.130.255   |

#### <a name="switch-infrastructure-network"></a>スイッチ インフラストラクチャ ネットワーク

インフラストラクチャ ネットワークは、物理スイッチ インフラストラクチャによって使用される複数のネットワークに分割されます。 これは、Azure Stack ソフトウェアのみをサポートする Azure Stack インフラストラクチャとは異なります。 Switch Infra Network は、物理スイッチ インフラストラクチャのみをサポートします。 インフラストラクチャでサポートされるネットワークは次のとおりです。

| **名前**                                   | **IPv4 サブネット**   |
|--------------------------------------------|-------------------|
| P2P_Border/Border1_To_Rack1/TOR1           | 10.193.132.128/30 |
| P2P_Border/Border1_To_Rack1/TOR2           | 10.193.132.132/30 |
| P2P_Border/Border2_To_Rack1/TOR1           | 10.193.132.136/30 |
| P2P_Border/Border2_To_Rack1/TOR2           | 10.193.132.140/30 |
| P2P_Rack1/TOR1_To_Rack1/BMC                | 10.193.132.144/30 |
| P2P_Rack1/TOR2_To_Rack1/BMC                | 10.193.132.148/30 |
| Loopback0_Rack1_TOR1                       | 10.193.132.152/32 |
| Loopback0_Rack1_TOR2                       | 10.193.132.153/32 |
| Loopback0_Rack1_BMC                        | 10.193.132.154/32 |
| P2P_Rack1/TOR1-ibgp-1_To_Rack1/TOR2-ibgp-1 | 10.193.132.156/30 |
| P2P_Rack1/TOR1-ibgp-2_To_Rack1/TOR2-ibgp-2 | 10.193.132.160/30 |
| SwitchMgmt                                 | 10.193.132.168/29 |
|                                            |                   |

-   Point-to-point (P2P) (ポイントツーポイント (P2P)):これらのネットワークにより、すべてのスイッチ間の接続が可能になります。 サブネットのサイズは、P2P あたり /30 ネットワークです。 最下位の IP は、スタック上のアップストリーム (North) デバイスに常に割り当てられます。

-   Loopback (ループバック):これらのアドレスは、ラックで使用される各スイッチに割り当てられた /32 ネットワークです。 境界デバイスは、Azure Stack ソリューションの一部であるとは想定されていないため、ループバックは割り当てられません。

-   Switch Mgmt or Switch Management (スイッチ管理またはスイッチの管理):この /29 ネットワークでは、ラック内のスイッチの専用管理インターフェイスがサポートされています。 IP は次のように割り当てられます。この表は、[Deployment Worksheet]\(デプロイ ワークシート\) の [IP Address Usage]\(IP アドレスの使用状況\) タブにも表示されます。

- Rack (ラック):Rack1      
- 名前:SwitchMgmt

| **割当先**  | **[IPv4 アドレス]** |
|------------------|------------------|
| ネットワーク          | 10.193.132.168   |
| Gateway          | 10.193.132.169   |
| TOR1             | 10.193.132.170   |
| TOR2             | 10.193.132.171   |
| ブロードキャスト        | 10.193.132.175   |

## <a name="prepare-environment"></a>環境を準備する

ハードウェア ライフサイクル ホスト イメージには、物理ネットワーク スイッチ構成の生成に使用される必要な Linux コンテナーが含まれています。

最新のパートナー デプロイ ツールキットには、最新のコンテナー イメージが含まれています。
更新されたスイッチ構成を生成する必要がある場合は、ハードウェア ライフサイクル ホスト上のコンテナー イメージを置き換えることができます。

コンテナー イメージを更新する手順は次のとおりです。

1.  コンテナー イメージをダウンロードします

2.  次の場所にあるコンテナー イメージを置き換えます

## <a name="generate-configuration"></a>構成を生成する

ここでは、JSON ファイルとネットワーク スイッチ構成ファイルを生成する手順について説明します。

1.  [Deployment Worksheet]\(デプロイ ワークシート\) を開きます

2.  すべてのタブのすべての必須フィールドに入力します

3.  [Deployment Worksheet]\(デプロイ ワークシート\) 上で "Generate" 関数を呼び出します。  
    生成された IP サブネットと割り当てが表示される 2 つの追加のタブが作成されます。

4.  データを確認し、確認したら、"Export" 関数を呼び出します。  
    JSON ファイルを保存するフォルダーを指定するように求められます。

5.  Invoke-SwitchConfigGenerator.ps1 を使用してコンテナーを実行します。 このスクリプトを実行するには、管理者特権の PowerShell コンソールが必要であり、実行するには次のパラメーターが必要です。

    -   ContainerName - スイッチ構成を生成するコンテナーの名前。

    -   ConfigurationData - [Deployment Worksheet]\(デプロイ ワークシート\) からエクスポートされた ConfigurationData.json ファイルのパス。

    -   OutputDirectory - 出力ディレクトリのパス。

    -   Offline - スクリプトがオフライン モードで実行されることを通知します。

    ```powershell
    C:\WINDOWS\system32> .\Invoke-SwitchConfigGenerate.ps1 -ContainerName generalonrampacr.azurecr.io/master -ConfigurationData .\ConfigurationData.json -OutputDirectory c:\temp -Offline
    ```

スクリプトが完了すると、ワークシートで使用されているプレフィックスを含む zip ファイルが生成されます。 

```console
C:\WINDOWS\system32> .\Invoke-SwitchConfigGenerate.ps1 -ContainerName generalonrampacr.azurecr.io/master -ConfigurationData .\ConfigurationData.json -OutputDirectory c:\temp -Offline                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         
Seconds : 2
Section : Validation
Step    : WindowsRequirement
Status  : True
Detail  : @{CurrentImage=10.0.18363.0}


Seconds : 2
Section : Validation
Step    : DockerService
Status  : True
Detail  : @{Status=Running}


Seconds : 9
Section : Validation
Step    : DockerSetup
Status  : True
Detail  : @{CPU=4; Memory=4139085824; OS=Docker Desktop; OSType=linux}


Seconds : 9
Section : Validation
Step    : DockerImage
Status  : True
Detail  : @{Container=generalonrampacr.azurecr.io/master:1.1910.78.1}


Seconds : 10
Section : Run
Step    : Container
Status  : True
Detail  : @{ID=2a20ba622ef9f58f9bcd069c3b9af7ec076bae36f12c5653f9469b988c01706c; ExternalPort=32768}


Seconds : 38
Section : Generate
Step    : Config
Status  : True
Detail  : @{OutputFile=c:\temp\N22R19.zip}


Seconds : 38
Section : Exit
Step    : StopContainer
Status  : True
Detail  : @{ID=2a20ba622ef9f58f9bcd069c3b9af7ec076bae36f12c5653f9469b988c01706c}
```

### <a name="custom-configuration"></a>カスタム構成

Azure Stack スイッチ構成について、いくつかの環境設定を変更できます。 テンプレートで変更できる設定を特定できます。 この記事では、それらのカスタマイズ可能な各設定と、変更が Azure Stack に与える可能性のある影響について説明します。 これらの設定には、パスワード更新、syslog サーバー、SNMP 監視、認証、およびアクセス制御リストが含まれます。

Azure Stack ソリューションのデプロイ時に、OEM (相手先ブランド供給) によって、TOR と BMC の両方のスイッチ構成が作成され、適用されます。 OEM は、Azure Stack オートメーション ツールを使用して、これらのデバイスで必要な構成が正しく設定されていることを検証します。 構成は、Azure Stack デプロイ ワークシートの情報に基づきます。 

>[!NOTE]
>OEM または Microsoft Azure Stack エンジニアリング チームからの同意なしに構成を **変更しない** でください。 ネットワーク デバイスの構成を変更すると、Azure Stack インスタンスのネットワークの問題の操作やトラブルシューティングに大きな影響を与える可能性があります。 ネットワーク デバイスのこれらの機能の詳細、これらの変更を行う方法については、OEM ハードウェアプロバイダーまたは Microsoft サポートにお問い合わせください。 OEM には、Azure Stack デプロイ ワークシートに基づいた、オートメーション ツールによって作成された構成ファイルがあります。 

しかし、ネットワーク スイッチの構成で追加、削除、または変更できる値がいくつかあります。

#### <a name="password-update"></a>パスワードの更新

オペレーターは、ネットワークスイッチ上の任意のユーザーのパスワードをいつでも更新できます。 Azure Stack システムの情報を変更したり、Azure Stack でシークレットをローテーションする手順を使用したりする必要はありません。

#### <a name="syslog-server"></a>Syslog サーバー

オペレーターは、データセンターの syslog サーバーにスイッチ ログをリダイレクトできます。
この構成を使用することで、特定の時点のログをトラブルシューティングに使用できるようにします。 既定で、ログはスイッチに格納されます。ログを格納するためのそれらの容量は限られています。 スイッチ管理アクセスのアクセス許可を構成する方法の概要については、「アクセス制御リストの更新」セクションを参照してください。

#### <a name="snmp-monitoring"></a>SNMP の監視

オペレーターは、ネットワーク デバイスを監視し、データセンターのネットワーク監視アプリケーションにトラップを送信するように簡易ネットワーク管理プロトコル (SNMP) v2 または v3 を構成できます。 セキュリティ上の理由から、v2 より安全であるため、SNMPv3 を使用してください。 必要な MIB と構成については、OEM ハードウェア プロバイダーに問い合わせてください。
スイッチ管理アクセスのアクセス許可を構成する方法の概要については、「アクセス制御リストの更新」セクションを参照してください。

#### <a name="authentication"></a>認証

オペレーターは、RADIUS または TACACS を構成して、ネットワークデバイスの認証を管理できます。 サポートされている方法と必要な構成については、OEM ハードウェア プロバイダーに問い合わせてください。 スイッチ管理アクセスのアクセス許可を構成する方法の概要については、「アクセス制御リストの更新」セクションを参照してください。

#### <a name="access-control-list-updates"></a>アクセス制御リストの更新

オペレーターは、いくつかのアクセス制御リスト (ACL) を変更して、信頼されたデータセンター ネットワーク範囲から、ネットワークデバイス管理インターフェイスおよびハードウェア ライフサイクル ホスト (HLH) にアクセスできるようにすることができます。 オペレーターは、到達可能にするコンポーネントおよびどこから到達可能にするかを選択できます。 アクセス制御リストによって、オペレーターは、特定のネットワーク範囲内の管理ジャンプボックス VM に、スイッチ管理インターフェイス、HLH OS、HLH BMC へのアクセスを許可することができます。

詳細については、「[物理スイッチのアクセス制御リスト](#physical-switch-access-control-list)」を参照してください。

#### <a name="tacacs-radius-and-syslog"></a>TACACS、RADIUS、Syslog

Azure Stack ソリューションには、スイッチやルーターなどのデバイスのアクセス制御に対応する TACACS または RADIUS ソリューションや、スイッチ ログをキャプチャする Syslog ソリューションは含まれていませんが、このようなすべてのデバイスでは、これらのサービスがサポートされています。
お使いの環境にある既存の TACACS、RADIUS、Syslog サーバーとの統合を支援するために、ネットワーク スイッチ構成を含む追加のファイルが用意されています。オンサイトのエンジニアはこれを利用して、お客様のニーズに合わせてスイッチをカスタマイズすることができます。

## <a name="next-steps"></a>次の手順

[ネットワーク統合](network-integration.md)
