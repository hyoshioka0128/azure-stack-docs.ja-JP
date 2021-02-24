---
title: Modular Datacenter の概要 |
description: Azure Modular Datacenter は、一時的または固定の指令拠点における大規模な戦闘活動を支援するのに適した、ポータブルで迅速にデプロイ可能なデータセンターです。
author: BryanLa
ms.author: bryanla
ms.service: azure-stack
ms.topic: overview
ms.date: 11/06/2020
ms.reviewer: prchint
ms.lastreviewed: 11/06/2020
ms.openlocfilehash: eabafd31d25a59cd43daa1a0779afed0a80f8354
ms.sourcegitcommit: 25b234330df4e753ed2dd480c208ec88cd90234c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/19/2020
ms.locfileid: "94924318"
---
# <a name="modular-datacenter-overview"></a>Modular Datacenter の概要

Azure Modular Datacenter (MDC) は Azure Stack Hub に基づいています。 MDC は、一時的または固定の指令拠点における大規模な戦闘活動を支援するのに適した、ポータプルで迅速にデプロイ可能なデータセンターです。

Azure Stack Hub は水平および垂直にスケーラブルなソリューションであり、これによって、サービスとしてのインフラストラクチャ (IaaS) とサービスとしてのプラットフォーム (PaaS) サービスを実現する、マルチテナントのネイティブ ハイブリッド クラウド機能を、エッジ上の環境に提供できます。 Azure Stack Hub では、一時的および固定の指令拠点と遠征部隊のためにさまざまなモジュール式のシナリオがサポートされています。

Azure Stack Hub はハードウェアおよびソフトウェアの統合されたアプライアンスで、ノードをスケール単位とするスケーリングに基づいてさまざまな容量のものを購入できます。 Azure Stack Hub では、拡張機能を使用できます。これには、汎用のグラフィックス処理装置 (GPU) 対応の構成や、拡張可能な外部ストレージが含まれます。

## <a name="use-mdc"></a>MDC の使用

Azure Stack は、モジュール式エッジのシナリオ向けの Azure の機能に整合し、それを拡張する 4 つの原則を実現します。

### <a name="develop-and-deliver-apps-with-a-common-devops-model-including-api-symmetry-with-azure"></a>Azure との API の対称性を含む、共通の DevOps モデルを使用してアプリを開発および提供する

Azure と Azure Stack の間の一貫性により、ソリューションをいったん開発した後は、さまざまなユース ケースをサポートするようにデプロイでき、共通のツール一式を使用してセキュリティで保護し、維持することができます。 ツールの例としては、キー管理用の Azure Key Vault や、リソースの監視および管理用の Azure Monitor があります。 Azure Stack は、オンプレミス データ、アプリ、DevOps ツール、セキュリティで保護された操作 (キー管理など) と相互運用できます。

### <a name="deliver-azure-services-on-premises"></a>Azure サービスをオンプレミスで提供する

Azure Stack は、不利な状況 (競合、輻輳、またはアクセス拒否) にある通信環境でも、堅牢な通信環境でも実行できます。 Azure Stack では Azure への接続に頼らずに作戦アプリを実行でき、ローカルでの操作が利用可能になります。

### <a name="use-integrated-hardware-and-software-delivery-experience"></a>統合されたハードウェアおよびソフトウェアの提供エクスペリエンスを使用する

モジュール式エッジには、ベースライン コンピューティングとストレージのサービスに加え、機械学習、AI、分析用の高度な機能も提供する、幅広い機能が必要です。 可能な場合にモジュール式エッジからセキュアなクラウドに接続する機能や、それとは逆に、制限された環境ではそれと独立して操作できる機能は、意思決定に必要なデータへのアクセスを提供する上で必要不可欠です。

### <a name="keep-your-datacenter-secure-and-available-with-hybrid-cloud-security-operations"></a>ハイブリッド クラウド セキュリティ運用によりデータセンターの安全性と可用性を維持する

Azure Stack のクラウドネイティブな設計のおかげで、従来の仮想化環境に必要な運用上の複雑さが解消されます。 インストール プロセスと、システムにネイティブに組み込まれている Azure Stack 管理ファブリックとにより、管理者はいつ修正プログラムを適用して、操作全体のオーケストレーションを行うかを選択できます。

現場の Azure Stack デバイスに対する Microsoft による、または Microsoft が管理するオファリングによるリモート支援管理は、管理ポータル、特権管理エンドポイント、またはコマンド ライン インターフェイス (CLI) を使用してアクションを実行できるよう、ユーザーに対してロールベースのアクセス制御 (RBAC) を通して適切なアクセス権を付与することで有効にできます。 この機能により、修正プログラムの適用やその他の管理および監視アクティビティをすべて Microsoft が実行できるようになります。 Azure Stack は、セキュリティで保護された管理ポータルまたは CLI コマンドを使用して現場でアップグレードおよびアップデートが可能です。両者を使って、必要に応じてローカルまたはリモート ネットワークから、セキュリティ更新プログラムや IaaS および PaaS 機能の更新プログラムを適用し、Azure との商業的な均衡を維持します。

## <a name="benefits-of-using-mdc"></a>MDC を使用する利点

MDC は、条件の悪い通信環境において Azure を使用して一貫した環境をサポートします。

 - モジュール式の指令センターで大規模な分析アプリを実行できるようにする、Azure クラウド サービスを使用して迅速にデプロイ可能な静的でモジュール式のデータセンターです。
 - Azure のモジュール式エッジ オファリングを使用すると、仮想マシン、ストレージ、仮想ネットワークなどの IaaS プリミティブをサポートするための 1 つのアプローチにより、クラウドとエッジの間の整合性を維持できます。
 - Azure Active Directory および RBAC のサポート。
 - 共通の管理インターフェイス。
 - API の対称性と Microsoft、サード パーティ、オープンソースの DevOps ツールのサポート。
 - Azure Log Analytics および Azure Security Center を使用した管理と監視。
 - ハイブリッド クラウドを有効にすることにより、クラウド コンピューティングのアジリティをオンプレミス環境とエッジにもたらします。<br>次のようにすることができます。
     - コードを再利用し、Azure とオンプレミス環境全体でクラウド ネイティブ アプリを一貫した方法で実行する。
     - Azure サービスへのオプション接続を使用して、従来の仮想化されたワークロードを実行する。
     - データをクラウドに転送するか、専用のデータセンターに保管してコンプライアンスを維持する。
     - ハードウェア アクセラレータによる機械学習や、コンテナー化または仮想化されたワークロードをすべてインテリジェント エッジで実行する。

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub のキャパシティ プランニング](../operator/azure-stack-capacity-planning-overview.md)
