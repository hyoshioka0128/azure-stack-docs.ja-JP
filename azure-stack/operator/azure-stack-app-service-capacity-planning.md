---
title: App Service サーバー ロールのキャパシティを計画する - Azure Stack Hub
description: Azure Stack Hub での Azure App Service サーバー ロールのキャパシティ プランニングについて説明します。
author: BryanLa
ms.topic: article
ms.date: 05/05/2020
ms.author: anwestg
ms.reviewer: anwestg
ms.lastreviewed: 04/13/2020
ms.openlocfilehash: ed29e43c7863b3bad3075ab97a9ce0085edb88c3
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90573294"
---
# <a name="capacity-planning-for-app-service-server-roles-in-azure-stack-hub"></a>Azure Stack Hub での App Service サーバー ロールのキャパシティ プランニング

稼働準備済みの Azure App Service on Azure Stack Hub のデプロイを設定するには、システムでサポートするキャパシティについて計画する必要があります。  

この記事では、実稼働環境のデプロイに使用すべき最小限のコンピューティング インスタンス数とコンピューティング SKU に関するガイダンスを示します。

> [!NOTE]
> 各ロールの推奨されるコンピューティング SKU に関するガイダンスは、Azure デプロイに沿った標準的なデプロイを導入できるように、Azure App Service on Azure Stack Hub の2020 Q2 リリースで更新されました。

これらのガイドラインを使用して、App Service のキャパシティ戦略を計画できます。

| App Service サーバー ロール | 推奨の最小インスタンス数 | 推奨のコンピューティング SKU|
| --- | --- | --- |
| コントローラー | 2 | A4v2 |
| フロントエンド | 2 | A4_v2 |
| 管理 | 2 | D3_v2 |
| Publisher | 2 | A2_v2 |
| Web Worker - 共有 | 2 | A4_v2 |
| Web Worker - 専用 - 小 | 層ごとに 2 個 | A1_v2 |
| Web Worker - 専用 - 中 | 層ごとに 2 個 | A2_v2 |
| Web Worker - 専用 - 大 | 層ごとに 2 個 | A4_v2 |

## <a name="controller-role"></a>コントローラー ロール

**推奨の最小構成**: A4v2 の 2 つのインスタンス

Azure App Service コントローラーでは通常、CPU、メモリ、ネットワーク リソースはそれほど消費されません。 ただし、高可用性のためには、2 つのコントローラーが必要です。 コントローラー 2 つは、許容される最大コント ローラー数でもあります。 2 番目の Web サイト コントローラーは、デプロイ中にインストーラーから直接作成できます。

## <a name="front-end-role"></a>フロントエンド ロール

**推奨の最小構成**: A4v_2 の 2 つのインスタンス

フロントエンドでは、Web worker の可用性に応じて、Web worker に要求をルーティングします。 高可用性のためには複数のフロントエンドが必要で、3 つ以上保有できます。 計画段階では、各コアが秒間約 100 リクエストを処理できると想定します。

## <a name="management-role"></a>管理ロール

**推奨の最小構成**: D3v2 の 2 つのインスタンス

Azure アプリケーション クラシック デプロイ モデル ロールでは、App Service Azure Resource Manager および API のエンドポイント、ポータル拡張機能 (管理、テナント、Functions ポータル)、データ サービスを担当します。 通常、運用環境では、管理サーバー ロールに約 4 GB の RAM のみが必要となります。 ただし、多くの管理タスク (Web サイトの作成など) が実行される場合は、CPU レベルが高くなる可能性があります。 高可用性のためには、このロールに複数のサーバーが割り当てられている必要があり、サーバーごとに少なくとも 2 つのコアが必要です。

## <a name="publisher-role"></a>パブリッシャー ロール

**推奨の最小構成**: A2v2 の 2 つのインスタンス

多くのユーザーが同時に公開する場合は、パブリッシャー ロールの CPU 使用率が高くなる可能性があります。 高可用性のためには、複数のパブリッシャー ロールを使用できるようにします。 パブリッシャーは、FTP または FTPS のトラフィックのみを処理します。

## <a name="web-worker-role"></a>Web Worker ロール

**推奨の最小構成**: A4_v2 の 2 つのインスタンス

高可用性のためには、少なくとも 4 つの Web worker ロールが必要です。つまり、共有 Web サイト モード用に 2 つと、提供する予定の各専用 worker 層用に 2 つです。 共有および専用のコンピューティング モードでは、さまざまなレベルのサービスがテナントに提供されます。 次のような顧客が多い場合は、より多くの Web worker が必要になる可能性があります。

- 専用コンピューティング モードの worker 層 (リソースを大量に使用) を使用している。
- 共有コンピューティング モードで稼働している。

ユーザーは、専用コンピューティング モード SKU 用の App Service プランを作成すると、その App Service プランで指定した数の Web worker を使用できなくなります。

ユーザーに従量課金プラン モデルで Azure Functions を提供するには、共有 Web worker をデプロイする必要があります。

使用する共有 Web worker ロールの数を決定する場合、次の考慮事項を検討します。

- **メモリ**:メモリは Web worker ロールの最も重要なリソースです。 メモリが不足していると、仮想メモリがディスクからスワップされるときの Web サイトのパフォーマンスに影響します。 各サーバーで、オペレーティング システム用に約 1.2 GB の RAM が必要となります。 このしきい値を超える RAM を Web サイトの実行で使用できます。
- **アクティブな Web サイトの割合**: 通常は、Azure App Service on Azure Stack Hub デプロイ内のアプリの約 5% がアクティブです。 ただし、特定の時点でのアクティブなアプリの割合は、それよりも高い場合も低い場合もあります。 アクティブなアプリの割合が 5% の場合、Azure App Service on Azure Stack Hub のデプロイに配置するアプリの最大数は、アクティブな Web サイトの数の 20 倍 (5 x 20 = 100) より少なくする必要があります。
- **平均メモリ占有領域**: 運用環境で計測されるアプリの平均メモリ占有領域は約 70 MB です。 この占有領域を使用して、すべての Web worker ロールのコンピューターまたは VM 全体で割り当てられるメモリは、次のように計算されます。

   `Number of provisioned applications * 70 MB * 5% - (number of web worker roles * 1044 MB)`

   たとえば、10 個の Web worker ロールを実行している環境に 5,000 個のアプリがある場合、各 Web worker ロール VM に 7,060 MB の RAM が必要です。

   `5,000 * 70 * 0.05 - (10 * 1044) = 7060 (= about 7 GB)`

   さらに worker インスタンスを追加する方法については、[worker ロールの追加](azure-stack-app-service-add-worker-roles.md)に関する記事をご覧ください。

### <a name="additional-considerations-for-dedicated-workers-during-upgrade-and-maintenance"></a>アップグレードおよびメンテナンス中の専用 worker に関する追加の考慮事項

worker のアップグレードおよびメンテナンス中、Azure App Service on Azure Stack Hub は常に各 worker 層の 20% に対してメンテナンスを実行します。  そのため、クラウド管理者は、アップグレード中およびメンテナンス中にテナントでサービスが停止しないように、worker 層ごとに常に 20% の未割り当て worker のプールを維持する必要があります。  たとえば、1 つの worker 層に 10 個の worker がある場合、アップグレードおよびメンテナンスを可能にするために 2 個を未割り当てにする必要があります。 10 個すべてが割り当て済みになった場合は、未割り当て worker のプールを維持するために、worker 層をスケールアップする必要があります。 

アップグレードおよびメンテナンス中、Azure App Service はワークロードが確実に動作し続けるようにワークロードを未割り当ての worker に移動します。 ただし、アップグレード中に未割り当ての worker を利用できない場合、テナント ワークロードのダウンタイムが発生する可能性があります。 共有 worker に関しては、利用可能な worker 内でテナント アプリがサービスによって自動的に割り当てられるため、お客様が追加の worker をプロビジョニングする必要はありません。 高可用性を実現するためには、この層に 2 個の worker があることが最小要件です。

クラウド管理者は、Azure Stack Hub 管理者ポータルの App Service 管理領域で worker 層の割り当てを監視できます。 App Service に移動し、左側のウィンドウにある [worker 層] を選択します。 [worker 層] テーブルには、worker 層の名前、サイズ、使用されているイメージ、利用可能な worker (未割り当て) の数、各層の worker の合計数、および worker 層の全体的な状態が表示されます。

![App Service 管理 - [worker 層]][1]

## <a name="file-server-role"></a>ファイル サーバー ロール

ファイル サーバー ロールでは、開発とテスト用にスタンドアロンのファイル サーバーを使用できます。 たとえば Azure Stack Development Kit (ASDK) で Azure App Service をデプロイするときに、この[テンプレート](https://aka.ms/appsvconmasdkfstemplate)を使用できます。  運用環境では、事前構成済みの Windows ファイル サーバーか、事前構成済みの Windows 以外のファイル サーバーを使用する必要があります。

運用環境では、ファイル サーバー ロールで集中的なディスク I/O が行なわれます。 ユーザー Web サイトのコンテンツとアプリ ファイルすべてが保存されるため、このロール用に次のリソースのいずれかを事前に構成しておく必要があります。

- Windows ファイル サーバー
- Windows ファイル サーバー クラスター
- Windows 以外のファイル サーバー
- Windows 以外のファイル サーバー クラスター
- NAS (ネットワーク接続ストレージ) デバイス

詳細については、[ファイル サーバーのプロビジョニング](azure-stack-app-service-before-you-get-started.md#prepare-the-file-server)に関する記述を参照してください。

## <a name="next-steps"></a>次のステップ

[App Service on Azure Stack Hub のデプロイの前提条件](azure-stack-app-service-before-you-get-started.md)

<!--Image references-->
[1]: ./media/azure-stack-app-service-capacity-planning/worker-tier-allocation.png