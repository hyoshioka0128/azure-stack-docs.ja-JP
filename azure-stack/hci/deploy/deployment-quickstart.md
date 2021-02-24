---
title: Azure Stack HCI クラスターの作成と Azure への登録に関するクイックスタート
description: Azure Stack HCI をデプロイし、Windows Admin Center を使用してクラスターを作成して、それを Azure に登録する方法について説明します。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 01/22/2021
ms.openlocfilehash: 6527623ba5b42f40eb24cd6269f5d45bbccb0d25
ms.sourcegitcommit: e772df8ac78c86d834a68d1a8be83b7f738019b7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/26/2021
ms.locfileid: "98772233"
---
# <a name="quickstart-create-an-azure-stack-hci-cluster-and-register-it-with-azure"></a>クイックスタート: Azure Stack HCI クラスターを作成して Azure に登録する

> 適用対象:Azure Stack HCI バージョン 20H2

このクイックスタートでは、2 台のサーバーで構成される単一サイトの Azure Stack HCI クラスターをデプロイし、Azure にそれを登録する方法について説明します。 複数サイトのデプロイについては、「[ストレッチ クラスターの概要](../concepts/stretched-clusters.md)」を参照してください。

## <a name="before-you-start"></a>アップグレードを開始する前に

クラスターを作成する前に、次の手順を実行します。

* [Azure Stack HCI カタログ](https://hcicatalog.azurewebsites.net)から、Azure Stack HCI オペレーティング システムがプレインストールされている、好みの Microsoft ハードウェア パートナーのサーバーを 2 台購入します。 [システム要件](../concepts/system-requirements.md)を参照し、クラスターで実行する予定のワークロードが選択したハードウェアでサポートされていることを確認します。 簡易構成の場合、iWARP を使用する高速ネットワーク アダプターを搭載したシステムを使用することをお勧めします。
* 各サーバーでローカル Administrators グループのメンバーであるアカウントを作成します。
* まだない場合は、[Azure サブスクリプションを入手します](https://azure.microsoft.com/)。
* 管理用 PC に [Windows Admin Center をインストール](/windows-server/manage/windows-admin-center/deploy/install)し、[Windows Admin Center を Azure に登録](../manage/register-windows-admin-center.md)します。 管理コンピューターは、クラスターを作成するのと同じ Active Directory ドメイン、または完全に信頼されたドメインに参加している必要があることに注意してください。
* デプロイのために、サーバー名、ドメイン名、IP アドレス、VLAN ID を記録しておきます。

## <a name="create-the-cluster"></a>クラスターを作成する

以下の手順のようにして、簡単な 2 ノードの単一サイト クラスターを作成します。 詳細について、またはストレッチ クラスターの作成については、「[Windows Admin Center を使用して Azure Stack HCI クラスターを作成する](create-cluster.md)」を参照してください。

1. Windows Admin Center で、 **[すべての接続]** の下にある **[追加]** をクリックします。
1. **[リソースの追加]** パネルの **[Windows Server cluster]\(Windows Server クラスター\)** で、 **[Create new]\(新しく作成\)** を選択します。
1. **[Choose cluster type]\(クラスターの種類の選択\)** で、 **[Azure Stack HCI]** を選択します。
1. **[Select server locations]\(サーバーの場所の選択\)** で、 **[All servers in one site]\(すべてのサーバーが 1 つのサイト内\)** を選択します。
1. **Create** をクリックしてください。 クラスターの作成ウィザードが表示されます。 **[Credential Security Service Provider (CredSSP)]\(資格情報セキュリティ サービス プロバイダー (CredSSP)\)** ポップアップが表示される場合は、 **[はい]** を選択して一時的に有効にします。 

クラスターの作成ウィザードには 5 つのセクションがあり、それぞれにいくつかのステップがあります。

1. **[Get started]\(作業の開始\)** 。 このセクションでは、前提条件を確認し、サーバーを追加し、ドメインに参加し、必要な機能と更新プログラムをインストールして、サーバーを再起動します。 
2. **ネットワーク。** ウィザードのこのセクションでは、正しいネットワーク アダプターが有効になっていることを確認し、使用していないものを無効にします。 管理アダプターを選択し、仮想スイッチの構成を設定し、IP アドレスを指定することでネットワークを定義します。
3. **[Clustering]\(クラスタリング\)** 。 このセクションでは、サーバーの構成が一貫しており、クラスタリングに適していることを確認してから、実際のクラスターを作成します。
4. **ストレージ。** 次に、ドライブをクリーニングして確認し、ストレージを検証して、記憶域スペース ダイレクトを有効にします。
5. **[SDN]** 。 このクラスターではてソフトウェアによるネットワーク (SDN) を使用しないため、セクション 5 はスキップしてかまいません。

ウィザードで CredSSP プロトコルを有効にした場合は、セキュリティのため、各サーバーでそれを無効にすることをお勧めします。

1. Windows Admin Center の **[すべての接続]** で、作成したクラスターを選択します。
1. **[ツール]** の下で、 **[サーバー]** を選択します。
1. 右側のペインで、クラスター内の最初のサーバーを選択します。
1. **[概要]** で **[Disable CredSSP]\(CredSSP を無効にする\)** を選択します。 上部の赤い **[CredSSP ENABLED]\(CredSSP が有効になっています\)** というバナーが表示されなくなります。
1. クラスターの 2 台目のサーバーで、ステップ 3 と 4 を繰り返します。

## <a name="set-up-a-cluster-witness"></a>クラスター監視のセットアップ

クラスター内のいずれかのサーバーがオフラインになった場合に、他のノードも使用できなくなることがないように、監視リソースを設定する必要があります。 このクイックスタートでは、別のサーバーにある SMB ファイル共有を監視用に使用します。 クラスター内のすべてのサーバー ノードに信頼性の高いインターネット接続がある場合は、Azure クラウド監視を使用しても構いません。 監視オプションの詳細については、「[クラスター監視のセットアップ](../manage/witness.md)」を参照してください。

1. Windows Admin Center で、上部のドロップダウン矢印から **[Cluster Manager]\(クラスター マネージャー\)** を選択します。
1. **[Cluster connections]\(クラスター接続\)** の下で、クラスターを選択します。
1. **[ツール]** の **[設定]** を選択します。
1. 右側のペインで **[Witness]\(監視\)** を選択します。
1. **[監視の種類]** で、 **[ファイル共有の監視]** を選択します。
1. ファイル共有のパス (例: **\\servername.domain.com\Witness$** ) を指定し、必要に応じて資格情報を指定します。
1. **[保存]** をクリックします。

## <a name="register-with-azure"></a>Azure に登録する

Azure Stack HCI を使用するには Azure に接続する必要があり、登録を完了するには Azure Active Directory のアクセス許可が必要です。 まだ持っていない場合は、Azure AD 管理者に、アクセス許可の付与または委任のどちらかを依頼します。 詳細については、「[Azure Stack HCI を Azure に接続する](register-with-azure.md)」を参照してください。 登録が済むと、クラスターはバックグラウンドで自動的に接続されます。

## <a name="next-steps"></a>次のステップ

このクイックスタートでは、Azure Stack HCI クラスターを作成し、それを Azure に登録しました。 これで、[ボリュームを作成](../manage/create-volumes.md)してから、[仮想マシンを作成](../manage/vm.md)できるようになります。