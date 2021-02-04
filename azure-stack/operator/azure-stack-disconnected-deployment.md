---
title: Azure Stack Hub 統合システムの Azure から切断されたデプロイの決定
description: Azure Stack Hub 統合システムの Azure から切断されたデプロイと考慮すべき計画の決定について説明します。
author: PatAltimore
ms.topic: conceptual
ms.date: 03/04/2020
ms.author: patricka
ms.reviewer: wfayed
ms.lastreviewed: 11/01/2019
ms.openlocfilehash: 53f46732b799e7a638ce7fb61e236ba4a55f2437
ms.sourcegitcommit: 5f3d37994b8cb63c76e54136c0cc05bc4f475950
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99495460"
---
# <a name="azure-disconnected-deployment-planning-decisions-for-azure-stack-hub-integrated-systems"></a>Azure Stack Hub 統合システムの Azure から切断されたデプロイ計画の決定
[Azure Stack Hub をご利用のハイブリッド クラウド環境に統合する方法](azure-stack-connection-models.md)を決定したら、Azure Stack Hub デプロイの決定を完了することができます。

インターネットに接続していなくても、Azure Stack Hub をデプロイして使用することができます。 ただし、切断されたデプロイでは、Active Directory フェデレーション サービス (AD FS) ID ストアおよび容量ベースの課金モデルに制限されます。 マルチテナントには Azure Active Directory (Azure AD) の使用が必要であるため、切断されたデプロイではマルチテナントはサポートされません。

このオプションは、次の場合に選択します。
- インターネットに接続されていない環境で Azure Stack Hub をデプロイすることが必要なセキュリティまたはその他の制限がある。
- データ (使用状況データを含む) を Azure に送信されないようにブロックしたい。
- Azure Stack Hub を純粋に、社内イントラネットにデプロイされるプライベート クラウド ソリューションとして使用したい。ハイブリッド シナリオには関心がない。

> [!TIP]
> この種類の環境は "*サブマリン シナリオ*" と呼ばれる場合もあります。

切断されたデプロイは、後でハイブリッド テナント VM シナリオのためにご利用の Azure Stack Hub インスタンスを Azure に接続することを制限するものではありません。 これは、デプロイ中に Azure への接続がないか、または ID ストアとして Azure AD を使用したくないことを示しています。

## <a name="features-that-are-impaired-or-unavailable-in-disconnected-deployments"></a>切断されたデプロイで損なわれるか、または使用できない機能 
Azure Stack Hub は Azure に接続されているときに最適に機能するように設計されているため、切断モードでは損なわれるか、または完全に使用できない機能がいくつか存在することに注意することが重要です。

|機能|切断モードでの影響|
|-----|-----|
|VM のデプロイ後を構成するための DSC 拡張機能を備えた VM デプロイ|損なわれる - DSC 拡張機能は、最新の WMF がないかどうかインターネットを参照します。|
|Docker コマンドを実行するための Docker 拡張機能を備えた VM デプロイ|損なわれる - Docker は最新バージョンがないかどうかインターネットをチェックするため、このチェックは失敗します。|
|Azure Stack Hub ポータルでのドキュメント リンク|使用できない - インターネット URL を使用するリンク ([フィードバックの送信]、[ヘルプ]、[クイック スタート] など) は機能しません。|
|オンライン修復ガイドを参照する、アラートによる修復/軽減|使用できない - インターネット URL を使用するアラート修復リンクはすべて機能しません。|
|Marketplace - Azure Marketplace から直接ギャラリー パッケージを選択して追加する機能|損なわれる - Azure Stack Hub を非接続モードでデプロイする場合、Azure Stack Hub ポータルを使用して Marketplace 項目をダウンロードすることはできません。 ただし、[マーケットプレース シンジケーション ツール](azure-stack-download-azure-marketplace-item.md)を使用して、インターネットに接続されたマシンに Marketplace アイテムをダウンロードしてから、ご利用の Azure Stack Hub 環境に転送することができます。|
|Azure Stack Hub デプロイを管理するための Azure AD フェデレーション アカウントの使用|使用できない - この機能には Azure への接続が必要です。 代わりに、ローカルの Active Directory インスタンスによる AD FS を使用する必要があります。|
|アプリケーション サービス|損なわれる - WebApps では、コンテンツの更新のためにインターネットへのアクセスが必要な場合があります。|
|コマンド ライン インターフェイス (CLI)|損なわれる - CLI では、サービス プリンシパルの認証およびプロビジョニングの機能が削減されます。|
|Visual Studio - クラウド検出|損なわれる - Cloud Discovery は別のクラウドを検出するか、まったく機能しないかのどちらかです。|
|Visual Studio - AD FS|損なわれる - AD FS 認証をサポートするのは、Visual Studio Enterprise と Visual Studio Code のみです。
テレメトリ|使用できない - Azure Stack Hub のテレメトリ データや、テレメトリ データに依存するすべてのサード パーティ製ギャラリー パッケージ。|
|証明書|使用できない - HTTPS のコンテキストでの証明書失効リスト (CRL) およびオンライン証明書ステータス プロトコル (OSCP) サービスにはインターネット接続が必要です。|
|Key Vault|損なわれる - Key Vault の一般的なユースケースでは、実行時にアプリにシークレットを読み取らせます。 このユース ケースの場合、アプリはそのディレクトリ内にサービス プリンシパルを必要とします。 Azure AD では、通常のユーザー (非管理者) は、既定ではサービス プリンシパルの追加を許可されます。 Azure AD (AD FS を使用) では、そうではありません。 アプリの追加には必ずディレクトリ管理者を経由する必要があるため、この欠損により、エンド ツー エンドのエクスペリエンスに問題が発生します。

## <a name="learn-more"></a>詳細情報
- ユース ケース、購入、パートナー、OEM ハードウェア ベンダーの詳細については、[Azure Stack Hub](https://azure.microsoft.com/overview/azure-stack/) の製品ページを参照してください。
- Azure Stack Hub 統合システムのロードマップと地理的な可用性については、ホワイト ペーパー「[Azure Stack Hub: An extension of Azure](https://azure.microsoft.com/resources/videos/azure-friday-azure-stack-an-extension-of-azure/)」 (Azure Stack: Azure の拡張機能) を参照してください。 
- Microsoft Azure Stack Hub のパッケージと価格の詳細については、[.pdf をダウンロードしてください](https://azure.microsoft.com/mediahandler/files/resourcefiles/5bc3f30c-cd57-4513-989e-056325eb95e1/Azure-Stack-packaging-and-pricing-datasheet.pdf)。 

## <a name="next-steps"></a>次のステップ
[データセンターのネットワーク統合](azure-stack-network.md)
