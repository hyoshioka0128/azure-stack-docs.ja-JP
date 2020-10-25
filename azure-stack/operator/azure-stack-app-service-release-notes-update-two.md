---
title: App Service on Azure Stack Hub Update 2 のリリース ノート
description: App Service on Azure Stack Hub Update 2 の機能強化、修正、既知の問題の詳細を説明します。
author: bryanla
manager: stefsch
ms.topic: article
ms.date: 03/25/2019
ms.author: anwestg
ms.reviewer: anwestg
ms.lastreviewed: 05/18/2019
ms.openlocfilehash: b06ab0601616f8c089d7bcf90526dae97a8c6e5f
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86489540"
---
# <a name="app-service-on-azure-stack-hub-update-2-release-notes"></a>App Service on Azure Stack Hub Update 2 のリリース ノート

これらのリリース ノートでは、Azure App Service on Azure Stack Hub Update 2 における機能強化、修正点、および既知の問題について説明します。 既知の問題は、デプロイに直接関係する問題、更新プロセスの問題、ビルド (インストール後) の問題の 3 つのセクションに分けられています。

> [!IMPORTANT]
> Azure App Service 1.2 をデプロイする前に、Azure Stack Hub 統合システムに 1804 更新プログラムを適用するか、最新の Azure Stack Development Kit (ASDK) をデプロイします。

## <a name="build-reference"></a>ビルドのリファレンス

App Service on Azure Stack Hub Update 2 のビルド番号は **72.0.13698.10** です。

## <a name="prerequisites"></a>前提条件

> [!IMPORTANT]
> Azure App Service で Kudu の SSO の処理方法が改善されたため、Azure App Service on Azure Stack Hub の新しいデプロイには、3 つのサブジェクトのワイルドカード証明書が必要になります。 新しいサブジェクトは次のとおりです: **\*.sso.appservice.\<region\>.\<domainname\>.\<extension\>**

デプロイを開始する前に、[App Service on Azure Stack Hub の前提条件](azure-stack-app-service-before-you-get-started.md)に関するページを参照してください。

## <a name="new-features-and-fixes"></a>新機能と修正

Azure App Service on Azure Stack Hub Update 2 には、次の機能強化と修正が含まれています。

- **App Service のテナント ポータル、管理ポータル、Functions ポータル、Kudu ツール**の更新。 Azure Stack Hub Portal SDK バージョンと一貫性があります。

- **Azure Functions ランタイム**が **v1.0.11612** に更新されました。

- 信頼性を高めるためのコア サービスと、一般的な問題を簡単に診断できるようにするエラー メッセージの更新。

- **次のアプリケーション フレームワークとツールの更新**:
  - .NET Framework 4.7.1 の追加
  - **Node.js** の次のバージョンが追加されました。
    - NodeJS 6.12.3
    - NodeJS 8.9.4
    - NodeJS 8.10.0
    - NodeJS 8.11.1
  - **NPM** の次のバージョンが追加されました。
    - 5.6.0
  - パブリック クラウドで Azure App Service と一致するように更新された .NET Core コンポーネント。
  - 更新された Kudu

- デプロイ スロット機能の自動スワップを有効化: [自動スワップの構成](/azure/app-service/deploy-staging-slots#configure-auto-swap)。

- 実稼働環境で機能のテストを有効化: [実稼働環境でのテストの概要](https://azure.microsoft.com/resources/videos/introduction-to-azure-websites-testing-in-production-with-galin-iliev/)。

- Azure Functions プロキシの有効化: [Azure Functions プロキシの操作](/azure/azure-functions/functions-proxies)。

- App Service 管理者の拡張機能 UX のサポートが以下に追加されました。
  - シークレット ローテーション
  - 証明書ローテーション
  - システム資格情報のローテーション
  - 接続文字列のローテーション

## <a name="known-issues-post-installation"></a>既知の問題 (インストール後)

- App Service が既存の仮想ネットワークにデプロイされ、ファイル サーバーがプライベート ネットワークでしか使用できない場合、worker はファイル サーバーに到達することができません。

ファイル サーバーに接続するために既存の仮想ネットワークと内部 IP アドレスへデプロイする場合は、worker サブネットとファイル サーバー間の SMB トラフィックを有効にする送信セキュリティ規則を追加する必要があります。 管理者ポータルで WorkersNsg に移動し、次のプロパティを持つ送信セキュリティ規則を追加します。

* ソース:Any
* 送信元ポート範囲: *
* 変換先:IP アドレス
* 宛先 IP アドレス範囲:ファイル サーバーの IP の範囲
* 送信先ポート範囲:445
* プロトコル:TCP
* アクション:Allow
* 優先順位:700
* 名前:Outbound_Allow_SMB445

## <a name="known-issues-for-cloud-admins-operating-azure-app-service-on-azure-stack-hub"></a>Azure App Service on Azure Stack Hub を運用するクラウド管理者に対する既知の問題

[Azure Stack Hub 1804 リリース ノート](./release-notes.md?view=azs-2002)内のドキュメントを参照してください。

## <a name="next-steps"></a>次のステップ

- Azure App Service の概要については、「[Azure Stack Hub 上の App Service の概要](azure-stack-app-service-overview.md)」を参照してください。
- App Service on Azure Stack Hub のデプロイを準備する方法の詳細については、「[App Service on Azure Stack Hub のデプロイの前提条件](azure-stack-app-service-before-you-get-started.md)」をご覧ください。
