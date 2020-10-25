---
title: Azure Stack Hub で GRE を使用して VPN トンネルを作成する
description: Azure Stack Hub で GRE を使用して VPN トンネルを作成する方法について説明します。
author: mattbriggs
ms.topic: how-to
ms.date: 5/27/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 09/19/2019
ms.openlocfilehash: cf384a161ce603b9e357c0efa989d6f648a68dda
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90572359"
---
# <a name="how-to-create-a-vpn-tunnel-using-gre-in-azure-stack-hub"></a>Azure Stack Hub で GRE を使用して VPN トンネルを作成する方法

このソリューションの Azure Stack Hub Resource Manager テンプレートを使用して、同じ Azure Stack Hub 環境内の 2 つの Azure Stack Hub VNet を接続できます。 組み込みの仮想ネットワーク ゲートウェイを使用して、[Azure Stack Hub VNet に接続することはできません](./azure-stack-network-differences.md)。 現時点では、ネットワーク仮想アプライアンス (NVA) を使用して、2 つの Azure Stack Hub VNet 間に VPN トンネルを作成する必要があります。 ソリューション テンプレートにより、RRAS がインストールされた 2 つの Windows Server 2016 VM がデプロイされます。 このソリューションでは、2 つの VNET 間に S2SVPN IKEv2 トンネルを使用するように 2 つの RRAS サーバーを構成します。 **内部**として指定された各 VNET のサブネット間のルーティングを許可する適切な NSG と UDR のルールが作成されています 

このデプロイ パターンは、Azure Stack Hub インスタンス内だけでなく、Windows RRAS S2S VPN トンネルを使用して Azure Stack Hub インスタンス間と他のリソース (オンプレミス ネットワークなど) への VPN トンネルの作成を可能にする基盤です。

テンプレートは [Azure Intelligent Edge Patterns](https://github.com/Azure-Samples/azure-intelligent-edge-patterns) GitHub リポジトリにあります。 このテンプレートは、**rras-gre-vnet-vnet** フォルダーにあります。 

![この図は、2 つの VNET 間に VPN トンネルを提供する実装を示しています。 各 VNET に、RRAS サーバー、内部サブネット、トンネル サブネットがあります。](./media/azure-stack-network-howto-vpn-tunnel-gre/overview.png)

## <a name="requirements"></a>必要条件

- 最新の更新プログラムが適用された状態でデプロイされたシステム。 
- 次の Azure Stack Hub Marketplace 必須項目。
    -  Windows Server 2016 Datacenter (最新ビルドを推奨)
    -  カスタム スクリプト拡張機能

## <a name="things-to-consider"></a>考慮事項

- ネットワーク セキュリティ グループは、テンプレートのトンネル サブネットに適用されます。 NSG を追加して、各 VNet の内部サブネットをセキュリティで保護します。
- RDP 拒否規則はトンネル NSG に適用され、パブリック IP アドレスを使用して VM にアクセスする場合は許可に設定する必要があります。
- このソリューションでは、DNS 解決を考慮していません。
- VNet 名と vmName の組み合わせは 15 文字未満にする必要があります。
- このテンプレートは、VNet1 および VNet2 用にカスタマイズされた VNet 名を使用するように設計されています。
- このテンプレートは BYOL Windows を使用しています
- 最新バージョンの (1907) では、リソース グループを削除する場合、リソース グループの削除を確実に完了するには、トンネル サブネットから NSG を手動でデタッチする必要があります
- このテンプレートでは、DS3v2 VM を使用しています。 RRAS サービスにより、Windows 内部 SQL Server がインストールされ、実行されます。 VM のサイズが小さすぎると、メモリの問題が発生する可能性があります。 VM サイズを小さくする前にパフォーマンスを確認してください。
- これは高可用なソリューションではありません。 さらに HA スタイルのソリューションが必要な場合は、2 つ目の VM を追加できます。また、ルート テーブルのルートをセカンダリ インターフェイスの内部 IP に手動で変更する必要があります。 また、複数のトンネルを相互接続するように構成する必要があります。

## <a name="options"></a>Options

- _artifactsLocation および_artifactsLocationSasToken パラメーターを使用して、独自の BLOB ストレージ アカウントと SAS トークンを使用できます
- このテンプレート INTERNALSUBNETREFVNET1 と INTERNALSUBNETREFVNET2 には 2 つの出力があります。これは、パイプライン スタイルのデプロイ パターンでこれを使用する場合、内部サブネットのリソース ID です。

このテンプレートには、VNet の名前付けと IP アドレス指定の既定値が指定されています。 管理者 (rrasadmin) のパスワードが必要です。また、SAS トークンで独自のストレージ BLOB を使用することもできます。 デプロイが失敗する可能性があるため、これらの値を適切な範囲内に保つように注意します。 PowerShell RDS パッケージは各 RRAS VM で実行され、ルーティングと必要なすべての依存サービスと機能がインストールされます。 この DSC は、必要に応じてさらにカスタマイズできます。 カスタム スクリプト拡張機能では、次のスクリプトが実行され、Add-Site2SiteGRE.ps1 によって 2 つの RRAS サーバー間の VPNS2S トンネルが共有キーを使用して構成されます。 カスタム スクリプト拡張機能からの詳細な出力を表示して、VPN トンネル構成の結果を確認できます。

!["S2SVPNTunnel" というタイトルの図に、サイト間 VPN トンネルによって接続された 2 つの VNET が示されています。](./media/azure-stack-network-howto-vpn-tunnel-gre/s2svpntunnel.png)

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub ネットワークの違いと考慮事項](azure-stack-network-differences.md)  
[複数サイト間 VPN トンネルを設定する方法](network-howto-vpn-tunnel.md)
