---
title: SDN Express を使用して SDN インフラストラクチャをデプロイする
description: SDN Express を使用して SDN インフラストラクチャをデプロイする方法を説明します
author: v-dasis
ms.topic: how-to
ms.date: 01/22/2021
ms.author: v-dasis
ms.reviewer: JasonGerend
ms.openlocfilehash: ec4c348242910eaf78831b59659bd5943f9cb854
ms.sourcegitcommit: e772df8ac78c86d834a68d1a8be83b7f738019b7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/26/2021
ms.locfileid: "98782015"
---
# <a name="deploy-an-sdn-infrastructure-using-sdn-express"></a>SDN Express を使用して SDN インフラストラクチャをデプロイする

> 適用対象: Azure Stack HCI バージョン 20H2

このトピックでは、SDN Express PowerShell スクリプトを使用して、エンドツーエンドのソフトウェア定義ネットワーク (SDN) インフラストラクチャをデプロイします。 このインフラストラクチャには、高可用性 (HA) ネットワーク コントローラー、高可用性ソフトウェア ロード バランサー (SLB)、および高可用性ゲートウェイが含まれます。  

スクリプトでは段階的デプロイがサポートされており、ネットワーク コントローラーのみをデプロイして、最小限のネットワーク要件に対応するコア セットの機能を実現できます。 Windows Admin Center でクラスターの作成ウィザードを使用して、ネットワーク コントローラーを展開することもできます。 ただし、SLB やゲートウェイなどの他の SDN コンポーネントをデプロイするには、SDN Express スクリプトを使用する必要があります。

System Center Virtual Machine Manager (VMM) を使用して SDN インフラストラクチャをデプロイすることもできます。 詳細については、「[VMM ファブリックで SDN リソースを管理する](/system-center/vmm/network-sdn)」を参照してください。

## <a name="before-you-begin"></a>開始する前に

SDN のデプロイを開始する前に、物理およびホストのネットワーク インフラストラクチャを計画して構成してください。 次の記事をご覧ください。

- [物理ネットワークの要件](../concepts/physical-network-requirements.md)
- [ホスト ネットワークの要件](../concepts/host-network-requirements.md)
- [Windows Admin Center を使用してクラスターを作成する](../deploy/create-cluster.md)
- [Windows PowerShell を使用してクラスターを作成する](../deploy/create-cluster-powershell.md)
- [ソフトウェア定義ネットワーク インフラストラクチャを計画する](../concepts/plan-software-defined-networking-infrastructure.md)

すべての SDN コンポーネントをデプロイする必要はありません。 「[ソフトウェア定義ネットワーク インフラストラクチャを計画する](../concepts/plan-software-defined-networking-infrastructure.md)」の「[段階的デプロイ](../concepts/plan-software-defined-networking-infrastructure.md#phased-deployment)」セクションを参照して、必要なインフラストラクチャ コンポーネントを決定した後、それに従ってスクリプトを実行します。

すべてのホスト サーバーに Azure Stack HCI オペレーティング システムがインストールされていることを確認してください。 「[Azure Stack HCI オペレーティング システムのデプロイ](../deploy/operating-system.md)」で、これを実行する方法をご覧ください。

## <a name="run-the-sdn-express-scripts"></a>SDN Express スクリプトを実行する

1. [SDN Express](https://github.com/microsoft/SDN) GitHub リポジトリにアクセスします。

1. 指定したデプロイ コンピューターにリポジトリからファイルをダウンロードします。 **[Clone]\(複製\)** または **[Download ZIP]\(ZIP のダウンロード\)** を選択します。

    > [!NOTE]
    > 指定したデプロイ コンピューターでは、Windows Server 2016 以降が実行されている必要があります。

1. ZIP ファイルを展開し、`SDNExpress` フォルダーをデプロイ コンピューターの `C:\` フォルダーにコピーします。

1. `C:\SDNExpress\scripts` フォルダーに移動します。

1. `SDNExpress.ps1` スクリプト ファイルを実行します。 このスクリプトでは、さまざまな構成設定の入力として PowerShell デプロイ (PSD) ファイルが使用されます。 スクリプトの実行手順については、`README.md` ファイルを参照してください。  

1. SDN VM を作成するためのソースとして、Azure Stack HCI OS が含まれる VHDX を使用します。 ISO から Azure Stack HCI OS をダウンロードしてインストールしている場合は、ドキュメントで説明されているように、`convert-windowsimage` ユーティリティを使用してこの VHDX を作成できます。

1. インフラストラクチャに合うように特定の値 (ホスト名、ドメイン名、ユーザー名とパスワードなど) と「[ソフトウェア定義ネットワーク インフラストラクチャを計画する](../concepts/plan-software-defined-networking-infrastructure.md)」トピックで説明されているネットワークのネットワーク情報を変更することで、`SDNExpress\scripts\MultiNodeSampleConfig.psd1` ファイルをカスタマイズします。 このファイルには、ネットワーク コントローラー コンポーネントだけをデプロイするのか、ソフトウェア ロード バランサーとゲートウェイ コンポーネントも同様にデプロイするのかに基づいて、何を入力する必要があるかについての具体的な情報が含まれています。

1. Hyper-V ホスト上で、管理者の資格情報を持つユーザー アカウントから、次のスクリプトを実行します。

    ```powershell
    SDNExpress\scripts\SDNExpress.ps1 -ConfigurationDataFile MultiNodeSampleConfig.psd1 -Verbose
    ```

1. ネットワーク コントローラー VM を作成した後、DNS サーバー上にあるネットワーク コントローラーのクラスター名に対して動的 DNS 更新を構成します。 詳細については、「[ネットワーク コントローラーをデプロイするための要件](/windows-server/networking/sdn/plan/installation-and-preparation-requirements-for-deploying-network-controller#step-3-configure-dynamic-dns-registration-for-network-controller)」の手順 3 を参照してください。

## <a name="next-steps"></a>次のステップ

VM を管理します。 詳細については、[VM の管理](../manage/vm.md)に関する記事を参照してください。