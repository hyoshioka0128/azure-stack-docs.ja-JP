---
title: Azure Stack HCI の Azure Kubernetes Service とは
description: Azure Stack HCI 上の Azure Kubernetes Service は、Azure Kubernetes Service (AKS) のオンプレミスの実装であり、コンテナー化されたアプリケーションの大規模な実行を自動化します。
ms.topic: overview
author: v-susbo
ms.author: v-susbo
ms.date: 12/02/2020
ms.openlocfilehash: a7d97d2454e9d77c1760180b46be9ce219d6af68
ms.sourcegitcommit: 0efffe1d04a54062a26d5c6ce31a417f511b9dbf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2020
ms.locfileid: "96612660"
---
# <a name="what-is-azure-kubernetes-service-on-azure-stack-hci"></a>Azure Stack HCI の Azure Kubernetes Service とは
> 適用対象:AKS on Azure Stack HCI、AKS runtime on Windows Server 2019 Datacenter

Azure Stack HCI 上の Azure Kubernetes Service は、Azure Kubernetes Service (AKS) のオンプレミスの実装であり、コンテナー化されたアプリケーションの大規模な実行を自動化します。 Azure Kubernetes Service が Azure Stack HCI および Windows Server 2019 Datacenter でプレビュー段階になりました。これにより、お客様のデータセンターで Linux および Windows コンテナーを簡単にホストできます。

オンプレミスの Azure Kubernetes Service の使用を開始するには、[プレビューに登録](https://aka.ms/AKS-HCI-Evaluate) (プレビュー期間中は追加料金は発生しません) してから、[Azure Kubernetes Service on Azure Stack HCI を設定](setup.md)します。 代わりに Azure Kubernetes Service を使用してクラウドベースのコンテナーを調整する場合は、[Azure の Azure Kubernetes Service](/azure/aks/intro-kubernetes) に関するページを参照してください。

以下のセクションでは、Azure Stack HCI 上の Azure Kubernetes Service を使用するいくつかの理由について説明し、サービスとその使用の開始方法に関する一般的ないくつかの質問にお答えします。 コンテナーの背景については、「[Windows とコンテナー](/virtualization/windowscontainers/about/)」を参照してください。Kubernetes の背景については、[Kubernetes の基本概念](kubernetes-concepts.md)に関するページまたは [Kubernetes.io](https://kubernetes.io) を参照してください。

## <a name="use-aks-to-automate-management-of-containerized-applications"></a>AKS を使用して、コンテナー化されたアプリケーションの管理を自動化する

Docker と Windows を使用して、いくつかのコンテナーを手動で管理することはできますが、多くの場合、アプリでは 5 個、10 個、または数百個ものコンテナーが利用されます。ここで、Kubernetes オーケストレーターの出番です。

Kubernetes は、大規模なコンテナー管理を自動化するためのオープンソースのオーケストレーターです。 Azure Kubernetes Service により、オンプレミスの Kubernetes デプロイが簡略化されます。その際に、Azure Stack HCI 上に Kubernetes と重要なアドオンを設定したり、ワークロードをホストするための Kubernetes クラスターを作成したりするためのウィザードが提供されます。

Azure Stack HCI 上のプレビュー段階にある Azure Kubernetes Service によって提供される機能をいくつか以下に示します。

- Azure Stack HCI クラスター全体で実行される Kubernetes クラスターに、コンテナー化されたアプリを大規模にデプロイする
- Linux と Windows ベースのコンテナー化されたアプリの両方をデプロイして管理する
- Windows Admin Center または PowerShell を使用して AKS on Azure Stack HCI をデプロイする
- Kubernetes クラスターに対してノードを追加または削除して、スケールアップまたはスケールダウンする
- Kubernetes クラスターでストレージとネットワークを管理する
- Kubernetes デプロイの自動更新を提供する
- 利用可能な最新バージョンの Kubernetes にアップグレードする
- Azure Arc for Kubernetes を介して一般的な Azure サービスを使用する

## <a name="simplify-setting-up-kubernetes"></a>Kubernetes の設定を簡略化する

Azure Kubernetes Service により、Azure Stack HCI および Windows Server 2019 Datacenter で Kubernetes を設定するプロセスが簡略化されます。このサービスには次の機能が含まれています。

- Kubernetes とその依存関係 (kubeadm、kubelet、kubectl、ポッド ネットワーク アドオンなど) を設定するための Windows Admin Center ウィザード
- コンテナー化されたアプリケーションを実行する Kubernetes クラスターを作成するための Windows Admin Center ウィザード
- Kubernetes を設定して、Kubernetes クラスターを作成するための PowerShell コマンドレット。これは、ホストの設定と Kubernetes クラスターの作成をスクリプト化する場合です

## <a name="view-and-manage-kubernetes-using-on-premises-tools-or-azure-arc"></a>オンプレミス ツールまたは Azure Arc を使用して、Kubernetes を表示および管理する

オンプレミスで Azure Kubernetes Service を設定し、Kubernetes クラスターを作成したら、Kubernetes インフラストラクチャを管理および監視するためのいくつかの方法が提供されます。

- **Kubectl および Kubernetes ダッシュボードなどの一般的なツールを使用してオンプレミスで** - オープンソースの Web ベース インターフェイスを使用して、Kubernetes クラスターへのアプリケーションのデプロイ、クラスター リソースの管理、トラブルシューティング、実行中のアプリケーションの表示を行います。
- **Azure Arc を使用して Azure portal で** - Azure Arc を使用して、クラウド環境とオンプレミス環境にわたって Kubernetes クラスター上にデプロイされたアプリケーションを管理します。 
<br>また、Azure Arc では、次のような他の Azure サービスで Kubernetes クラスターを管理することができます。

  - Azure Monitor
  - Azure Policy
  - ロールベースのアクセス制御

## <a name="run-linux-and-windows-containers"></a>Linux および Windows コンテナーを実行する

Azure Kubernetes Service では、Linux ベースと Windows ベースの両方のコンテナーが完全にサポートされます。 Azure Stack HCI 上に Kubernetes クラスターを作成するときに、Linux コンテナーまたは Windows コンテナー、あるいはその両方を実行するノード プール (同一の Kubernetes クラスター ノードのグループ) を作成するかどうかを選択できます。 

Azure Kubernetes Service によって、Linux と Windows のノードが作成されます。これにより、Linux や Windows のオペレーティング システムを直接管理する必要がなくなります。

## <a name="secure-your-container-infrastructure"></a>コンテナーのインフラストラクチャをセキュリティで保護する

Azure Kubernetes Service には、コンテナーのインフラストラクチャをセキュリティで保護するのに役立つ多数の機能が含まれています。

- **ワーカー ノードのハイパーバイザーベースの分離** - 各 Kubernetes クラスターは専用の分離された仮想マシン セットで実行されるため、テナントは同じ物理インフラストラクチャを共有できます。
- **ワーカー ノードの Microsoft によって管理されている Linux および Windows イメージ** - セキュリティのベスト プラクティスに従うために Microsoft によって作成された Linux および Windows 仮想マシン イメージが、ワーカー ノードにより実行されます。 また、Microsoft によって、これらのイメージは毎月、最新のセキュリティ更新プログラムで更新されます。

Azure Stack HCI 上の Azure Kubernetes Service プレビュー リリースに関するセキュリティは継続的な投資領域であるため、随時ご確認ください。

## <a name="where-can-i-run-azure-kubernetes-service"></a>Azure Kubernetes Service はどこで実行できますか?

Azure Kubernetes Service は、以下のプラットフォーム上で使用できます。

- Azure クラウド内。[Azure の Azure Container Service](/azure/aks/intro-kubernetes) を使用します
- オンプレミス。Azure Stack HCI の Azure Kubernetes Service を使用します (この記事の全容)
- Azure Kubernetes Service runtime on Windows Server を介したオンプレミス (この記事は AKSr on Windows Server にも適用されます)
- Azure Stack Hub 環境のオンプレミス。[Azure Stack Hub 上の AKS エンジン](../user/azure-stack-kubernetes-aks-engine-overview.md)を使用します。

## <a name="how-does-kubernetes-work-on-azure-stack-hci"></a>Azure Stack HCI では Kubernetes はどのように動作しますか?

Azure Kubernetes Service の動作は、Azure クラウドで使用する場合と、Azure Stack HCI で実行する場合では少し異なります。

- Azure の Kubernetes サービスは、Kubernetes 管理インフラストラクチャ (コントロール プレーン) の多くが自動的に管理されるホステッド サービスです。 コントロール プレーンとコンテナー化されたアプリケーションの両方が Azure Virtual Machines で実行されます。
- Azure Stack HCI 上で Azure Kubernetes Service を使用する場合、Azure Stack HCI クラスター上に直接サービスを設定します。つまり、コントロール プレーンを制御できるようになります。 コントロール プレーン、コンテナー化されたアプリケーション、および Azure Kubernetes Service 自体はすべて、ハイパーコンバージド クラスターによってホストされる仮想マシンで実行されます。

Azure Kubernetes Service が Azure Stack HCI クラスター上に設定されると、ホストされている Azure Kubernetes Service と同様に動作します。このサービスは、コンテナー化されたアプリケーションを実行する Kubernetes クラスターを作成するために使用します。 これらの Kubernetes クラスターは、アプリケーション コンテナーを実行する、ワーカー ノードとして機能する VM のグループです。 また、Kubernetes クラスターには、アプリケーション コンテナーの調整に使用される Kubernetes システム サービスで構成されるコントロール プレーンも含まれています。

次の 2 つの簡略図は、Azure Kubernetes Service が Azure で実行されている場合と Azure Stack HCI で実行されている場合のアーキテクチャを比較したものです。

:::image type="content" source="media\overview\aks-azure-architecture.png" alt-text="Azure でホストされている Azure Kubernetes Service のアーキテクチャ。プラットフォーム サービスとほとんどのコントロール プレーンが Azure によってどのように管理されており、一方、コンテナー化されたアプリケーションを実行する Kubernetes クラスターが顧客によってどのように管理されているかを示しています。" lightbox="media\overview\aks-azure-architecture.png":::

:::image type="content" source="media\overview\aks-hci-architecture.png" alt-text="Azure Stack HCI の Azure Kubernetes Service のアーキテクチャ。Azure Kubernetes Service プラットフォーム、コントロール プレーン、コンテナー化されたアプリケーションを実行する Kubernetes クラスターなど、すべてのものが Azure Stack HCI クラスター上でどのように実行されるかを示しています。" lightbox="media\overview\aks-hci-architecture.png":::

## <a name="what-you-need-to-get-started"></a>必要事項

以下のセクションでは、Azure Stack HCI 上で Azure Kubernetes Service を実行するために必要なものの概要を示します。 詳細については、[Azure Stack HCI 上に Azure Kubernetes Service をインストールする前](system-requirements.md)に関するページを参照してください。

### <a name="on-your-windows-admin-center-system"></a>Windows Admin Center システム上

Windows Admin Center ゲートウェイを実行しているマシンには、次の要件があります。

- Windows 10 マシン (現時点では、Azure Stack HCI または Windows Server 2019 Datacenter での Windows Admin Center の実行はサポートされていません)
- 60 GB の空き領域
- Azure に登録済み
- Azure Stack HCI クラスターまたは Windows Server 2019 Datacenter クラスターと同じドメイン内

### <a name="on-the-azure-stack-hci-cluster-or-windows-server-2019-datacenter-failover-cluster-that-hosts-azure-kubernetes-service"></a>Azure Kubernetes Service をホストする Azure Stack HCI クラスターまたは Windows Server 2019 Datacenter フェールオーバー クラスター上

Azure Stack HCI クラスターまたは Windows Server 2019 Datacenter フェールオーバー クラスターには、次の要件があります。

- このプレビュー リリースでは、クラスター内の最大サーバー数は 4 である
- Azure Kubernetes Service の記憶域プールで使用可能な容量は 1 TB である
- Azure Kubernetes Service VM の実行に使用可能なメモリは 30 GB 以上である
- このプレビュー リリースでは、クラスター内のすべてのサーバーで EN-US リージョンと言語の選択を使用する必要がある

一般的な Azure Stack HCI のシステム要件については、[Azure Stack HCI のシステム要件](../hci/concepts/system-requirements.md)に関するページを参照してください。

### <a name="the-network-configuration-for-azure-stack-hci"></a>Azure Stack HCI のネットワーク構成

Azure Stack HCI または Windows Server 2019 Datacenter クラスター上の VM に接続されているネットワークには、Azure Kubernetes Service で使用でき、Azure Stack HCI または Windows Server 2019 Datacenter クラスター上の VM からアクセスできる、専用スコープの DHCP IPv4 アドレスが必要です。

## <a name="next-steps"></a>次の手順

Azure Stack HCI 上で Azure Kubernetes Service の使用を開始する場合は、次の記事を参照してください。

- [要件を確認する](system-requirements.md)
- [Azure Stack HCI で Azure Kubernetes Service を設定する](create-kubernetes-cluster.md)
