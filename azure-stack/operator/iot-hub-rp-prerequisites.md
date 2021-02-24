---
title: Azure Stack Hub に IoT Hub をインストールするための前提条件
description: Azure Stack Hub に IoT Hub リソース プロバイダーをインストールするにあたり、必要な前提条件について説明します。
author: yiyiguo
ms.author: yiygu
ms.service: azure-stack
ms.topic: how-to
ms.date: 1/6/2020
ms.openlocfilehash: f194ef78a31722a05742b14b312ea4aad58d3ed3
ms.sourcegitcommit: 0e3296fb27b9dabbc2569bf85656c4c7b1d58ba9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2020
ms.locfileid: "93050339"
---
# <a name="prerequisites-for-installing-iot-hub-on-azure-stack-hub"></a>Azure Stack Hub に IoT Hub をインストールするための前提条件

[!INCLUDE [preview-banner](../includes/iot-hub-preview.md)]

Azure Stack Hub に IoT Hub をインストールするには、次の前提条件を満たしておく必要があります。 すべての手順を完了するには、 **数日から数週間のリード タイムが必要になる場合があります** 。

> [!IMPORTANT]
> これらの前提条件は、少なくとも 4 ノードの Azure Stack Hub 統合システム ( **ビルド番号 1.2005.6.53** 以降) が既にデプロイされていることを前提としています。 Azure Stack Hub Development Kit (ASDK) では、IoT Hub リソース プロバイダーはサポートされません。

## <a name="common-prerequisites"></a>一般的な前提条件

[!INCLUDE [Common RP prerequisites](../includes/resource-provider-prerequisites.md)]

## <a name="dependency-prerequisites"></a>依存関係の前提条件

1. Marketplace から [Event Hubs をダウンロードしてインストール](event-hubs-rp-install.md)します。 Event Hubs のデプロイは、IoT Hub のデプロイを開始する前に行う必要があります。
2. IoT Hub をより高速にダウンロードしてインストールしたい場合は、IoT Hub パッケージをダウンロードする前に、以下の依存アイテムを Marketplace からダウンロードします。 それ以外の場合、IoT Hub のデプロイによって、依存パッケージのダウンロードが試行されます。
    * カスタム スクリプト拡張機能
    * PowerShell Desired State Configuration
    * 無料のライセンス:Windows Server 2016 の SQL Server 2016 SP2 Express
    * SQL IaaS 拡張機能
    * Azure Stack Add-On RP Windows Server
3. Event Hubs のインストールが正常に完了したら、10 分以上待って IoT Hub のデプロイに進みます。

## <a name="certificate-requirements"></a>証明書の要件

1. Event Hubs 用の公開キー基盤 (PKI) TLS/SSL 証明書を入手します。 サブジェクトの別名 (SAN) は、`CN=*.mgmtiothub.<region>.<fqdn>` の名前付けパターンに従っている必要があります。

   サブジェクト名を指定することもできますが、証明書を処理する際に IoT Hub によってこれが使用されることはありません。 サブジェクトの別名のみが使用されます。 詳細な要件の完全な一覧については、[PKI 証明書の要件](azure-stack-pki-certs.md)に関するページをご覧ください。

   ![IoT Hub の証明書の例](media\iot-hub-rp-prerequisites\certificate.png)

2. 必ず、[証明書の検証](azure-stack-validate-pki-certs.md)に関する記事に目を通してください。 この記事には、IoT Hub リソース プロバイダーに使用する証明書を準備して検証する方法が示されています。 

## <a name="dns-configuration-requirements"></a>DNS の構成要件
 
Azure Stack Hub 上の IoT Hub がネットワーク上で適切に動作するためには、ネットワーク管理者が DNS を構成する必要があります。 DNS 管理ツールで DNS の条件付き転送の設定を見つけて、`<region>.cloudapp.<externaldomainname>` のトラフィックを許可する条件付き転送ルールを追加します。 たとえば、「 `ussouth.cloudapp.contoso.com` 」のように入力します。

## <a name="next-steps"></a>次のステップ

次に、[接続環境の Azure Stack](iot-hub-rp-install.md) に IoT Hub リソース プロバイダーをインストールします。
