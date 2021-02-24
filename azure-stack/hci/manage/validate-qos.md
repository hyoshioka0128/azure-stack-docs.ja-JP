---
title: クラスター検証レポートのトラブルシューティング
description: クラスター検証レポートのトラブルシューティングを行い、Azure Stack HCI クラスターに対する QoS 設定の構成を検証します
author: khdownie
ms.topic: troubleshooting
ms.date: 01/05/2021
ms.author: v-kedow
ms.reviewer: JasonGerend
ms.openlocfilehash: a5b6ef03701daf1c1f4115f88a2a4e44bac1bd61
ms.sourcegitcommit: 0e2c814cf2c154ea530a4e51d71aaf0835fb2b5a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/06/2021
ms.locfileid: "97918631"
---
# <a name="troubleshoot-cluster-validation-reporting"></a>クラスター検証レポートのトラブルシューティング

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

このトピックは、Azure Stack HCI クラスター内のサーバー間でのネットワークとストレージの QoS (サービスの品質) 設定に関するクラスター検証レポートのトラブルシューティングを行い、重要なルールが定義されていることを確認するのに役立ちます。 最適な接続とパフォーマンスのため、クラスター検証プロセスでは、データ センター ブリッジング (DCB) の QoS 構成が一貫していること、そして定義されている場合は、フェールオーバー クラスタリングと SMB/SMB ダイレクト トラフィック クラスに適した規則が含まれていることが確認されます。

DCB は RDMA over Converged Ethernet (RoCE) ネットワークに必須であり、Internet Wide Area RDMA Protocol (iWARP) ネットワークでは省略可能です (ただし推奨されます)。

## <a name="install-data-center-bridging"></a>データ センター ブリッジングをインストールする

QoS 固有のコマンドレットを使用するには、データ センター ブリッジングをインストールする必要があります。 サーバーにデータ センター ブリッジング機能が既にインストールされているかどうかを確認するには、PowerShell で次のコマンドレットを実行します。

```PowerShell
Get-WindowsFeature -Name Data-Center-Bridging -ComputerName Server1
```

データ センター ブリッジングがインストールされていない場合は、クラスター内の各サーバーで次のコマンドレットを実行してインストールします。

```PowerShell
Install-WindowsFeature –Name Data-Center-Bridging -ComputerName Server1
```

## <a name="run-a-cluster-validation-test"></a>クラスター検証テストを実行する

Windows Admin Center で **[ツール] > [サーバー] > [インベントリ] > [クラスターの検証]** を選択して検証機能を使用するか、または次の PowerShell コマンドを実行します。

```PowerShell
Test-Cluster –Node Server1, Server2
```

テストでは、特に、DCB QoS の構成が一貫していることと、クラスター内のすべてのサーバーに同じ数のトラフィック クラスと QoS ルールがあることが検証されます。 また、すべてのサーバーで、フェールオーバー クラスタリングと SMB/SMB ダイレクト トラフィック クラスに対して QoS ルールが定義されていることも確認されます。

検証レポートは、Windows Admin Centerで表示するか、現在の作業ディレクトリのログ ファイルにアクセスして表示することができます。 次に例を示します。C:\Users\<username>\AppData\Local\Temp\

レポートの下部には、[Validate QoS Settings Configuration]\(QoS 設定の構成を検証する\) と、クラスター内の各サーバーの対応するレポートが表示されます。

サーバーで既に設定されているトラフィック クラスを把握するには、`Get-NetQosTrafficClass` コマンドレットを使用します。

詳細については、「[Azure Stack HCI クラスターの検証](../deploy/validate.md)」を参照してください。

## <a name="validate-networking-qos-rules"></a>ネットワーク QoS ルールを検証する

クラスター内のサーバー間で、DCB 許容状態と優先順位フロー制御状態の設定の整合性を検証します。

### <a name="dcb-willing-status"></a>DCB 許容状態

Data Center Bridging Capability Exchange プロトコル (DCBX) をサポートするネットワーク アダプターでは、リモート デバイスから構成を受け入れることができます。 この機能を有効にするには、ネットワーク アダプターで DCB 許容ビットを true に設定する必要があります。 許容ビットが false に設定されている場合、デバイスではリモート デバイスからのすべての構成試行が拒否され、ローカル構成のみが適用されます。 RDMA over Converged Ethernet (RoCE) アダプターを使用している場合は、すべてのサーバーで許容ビットを false に設定する必要があります。

Azure Stack HCI クラスター内のすべてのサーバーで、DCB 許容ビットが同じように設定されている必要があります。

次の例のように、DCB 許容ビットを true または false に設定するには、`Set-NetQosDcbxSetting` コマンドレットを使用します。

```PowerShell
Set-NetQosDcbxSetting –Willing $false
```

### <a name="dcb-flow-control-status"></a>DCB フロー制御状態

ファイバー チャネルなどの上位層プロトコルで、基になる転送が無損失であることが想定されている場合は、優先順位に基づくフロー制御が不可欠です。 DCB フロー制御は、グローバルに、または個々のネットワーク アダプターに対して、有効または無効にすることができます。 有効にすると、特定のアプリケーション トラフィックに優先順位を付ける QoS ポリシーを作成できるようになります。

フェールオーバーの間に QoS ポリシーがシームレスに機能するには、Azure Stack HCI クラスター内のすべてのサーバーに、同じフロー制御状態が設定されている必要があります。 RoCE アダプターを使用している場合は、すべてのサーバーで優先順位フロー制御を有効にする必要があります。

現在のフロー制御の構成を取得するには、`Get-NetQosFlowControl` コマンドレットを使用します。 既定では、すべての優先順位は無効になっています。

優先順位フロー制御をオンまたはオフにするには、-priority パラメーターを指定して `Enable-NetQosFlowControl` および `Disable-NetQosFlowControl` コマンドレットを使用します。 たとえば、次のコマンドでは、優先順位 3 のタグが付けられたトラフィックに対するフロー制御が有効になります。

```PowerShell
Enable-NetQosFlowControl –Priority 3
```

## <a name="validate-storage-qos-rules"></a>ストレージの QoS ルールを検証する

すべてのノードに、フェールオーバー クラスタリングと SMB または SMB ダイレクトに対する QoS ルールがあることを確認します。 ない場合、接続の問題とパフォーマンスの問題が発生する可能性があります。

### <a name="qos-rule-for-failover-clustering"></a>フェールオーバー クラスタリングの QoS ルール

クラスターで **何らかの** ストレージ QoS ルールが定義されている場合は、フェールオーバー クラスタリングに対する QoS ルールが存在する必要があります。ないと、接続の問題が発生する可能性があります。 フェールオーバー クラスタリングに新しい QoS ルールを追加するには、次の例のように `New-NetQosPolicy` コマンドレットを使用します。

```PowerShell
New-NetQosPolicy "Cluster" -IPDstPort 3343 -Priority 6
```

### <a name="qos-rule-for-smb"></a>SMB の QoS ルール

一部またはすべてのノードで QoS ルールが定義されているのに、SMB の QoS ルールが設定されていない場合は、SMB で接続とパフォーマンスの問題が発生する可能性があります。 SMB に新しいネットワーク QoS ルールを追加するには、次の例のように `New-NetQosPolicy` コマンドレットを使用します。

```PowerShell
New-NetQosPolicy -Name "SMB" -SMB -PriorityValue8021Action 3
```

### <a name="qos-rule-for-smb-direct"></a>SMB ダイレクトの QoS ルール

SMB ダイレクトでは、RDMA の方法を使用してデータが転送される代わりに、ネットワーク スタックがバイパスされます。 一部またはすべてのノードで QoS ルールが定義されているのに、SMB ダイレクトの QoS ルールが設定されていない場合、SMB ダイレクトで接続とパフォーマンスの問題が発生する可能性があります。 SMB ダイレクトに対する新しい QoS ポリシーを作成するには、次のコマンドを実行します。

```PowerShell
New-NetQosPolicy "SMB Direct" –NetDirectPort 445 –Priority 3
```

## <a name="next-steps"></a>次のステップ

関連情報については、以下も参照してください。

- [データ センター ブリッジング](/windows-server/networking/technologies/dcb/dcb-top)
- [データ センター ブリッジングを管理する](/windows-server/networking/technologies/dcb/dcb-manage)
- [QoS の一般的な構成](/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/jj735302(v=ws.11))