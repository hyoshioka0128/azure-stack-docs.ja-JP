---
title: SDN Express を使用して SDN インフラストラクチャをデプロイする
description: SDN Express を使用して SDN インフラストラクチャをデプロイする方法を説明します
author: v-dasis
ms.topic: how-to
ms.date: 02/01/2021
ms.author: v-dasis
ms.reviewer: JasonGerend
ms.openlocfilehash: b8431b7e2cf2cad3d238386619839a760b722042
ms.sourcegitcommit: d74f6d8630783de49667956f39cac67e1968a89d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99473193"
---
# <a name="deploy-an-sdn-infrastructure-using-sdn-express"></a>SDN Express を使用して SDN インフラストラクチャをデプロイする

> 適用対象: Azure Stack HCI バージョン 20H2

このトピックでは、SDN Express PowerShell スクリプトを使用して、エンドツーエンドのソフトウェア定義ネットワーク (SDN) インフラストラクチャをデプロイします。 このインフラストラクチャには、高可用性 (HA) ネットワーク コントローラー (NC)、高可用性ソフトウェア ロード バランサー (SLB)、および高可用性ゲートウェイを含む場合があります。  スクリプトでは段階的デプロイがサポートされており、ネットワーク コントローラーのみをデプロイして、最小限のネットワーク要件に対応するコア セットの機能を実現できます。 

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

## <a name="requirements"></a>要件

SDN を正常にデプロイするには、次の要件が満たされている必要があります。

- すべてのホスト サーバーで Hyper-V が有効になっている
- すべてのホスト サーバーが Active Directory に参加している
- 仮想スイッチが作成されている
- 物理ネットワークが、構成ファイルで定義されているサブネットと VLAN に対して構成されている
- 構成ファイルで指定された VHDX ファイルが、SDN Express スクリプトが実行されているデプロイ コンピューターから到達可能である

## <a name="create-the-vdx-file"></a>VDX ファイルを作成する

SDN によって、Azure Stack HCI OS が含まれる VHDX ファイルが、SDN 仮想マシン (VM) を作成するためのソースとして使用されます。 VHDX 内の OS のバージョンは、Hyper-V ホストで使用されているバージョンと一致している必要があります。

ISO から Azure Stack HCI OS をダウンロードしてインストールしている場合は、[スクリプト センター](https://gallery.technet.microsoft.com/scriptcenter/Convert-WindowsImageps1-0fe23a8f)で説明されているように、`Convert-WindowsImage` ユーティリティを使用してこの VHDX を作成できます。

`Convert-WindowsImage` の使用例を次に示します。

 ```powershell
$wimpath = "d:\sources\install.wim"
$vhdpath = "c:\temp\WindowsServerDatacenter.vhdx"
$Edition = 4   # 4 = Full Desktop, 3 = Server Core

import-module ".\convert-windowsimage.ps1"

Convert-WindowsImage -SourcePath $wimpath -Edition $Edition -VHDPath $vhdpath -SizeBytes 500GB -DiskLayout UEFI
```

## <a name="download-the-github-repository"></a>GitHub リポジトリをダウンロードする

SDN Express スクリプト ファイルは GitHub 内にあります。 最初の手順として、必要なファイルとフォルダーをお使いのデプロイ コンピューターで取得します。

1. [SDN Express](https://github.com/microsoft/SDN) GitHub リポジトリにアクセスします。

1. 指定したデプロイ コンピューターにリポジトリからファイルをダウンロードします。 **[Clone]\(複製\)** または **[Download ZIP]\(ZIP のダウンロード\)** を選択します。

    > [!NOTE]
    > 指定したデプロイ コンピューターでは、Windows Server 2016 以降が実行されている必要があります。

1. ZIP ファイルを展開し、`SDNExpress` フォルダーをデプロイ コンピューターの `C:\` フォルダーにコピーします。

## <a name="edit-the-configuration-file"></a>構成ファイルを編集する

PowerShell `MultiNodeSampleConfig.psd1` 構成データ ファイルには、さまざまなパラメーターと構成設定の入力として SDN Express スクリプトに必要なすべてのパラメーターと設定が含まれています。 このファイルには、ネットワーク コントローラー コンポーネントだけをデプロイするのか、ソフトウェア ロード バランサーとゲートウェイ コンポーネントも同様にデプロイするのかに基づいて、何を入力する必要があるかについての具体的な情報が含まれています。

詳細については、「[ソフトウェア定義ネットワーク インフラストラクチャを計画する](../concepts/plan-software-defined-networking-infrastructure.md)」をご覧ください。

では、始めましょう。

1. `C:\SDNExpress\scripts` フォルダーに移動します。

1. 使い慣れたテキスト エディターで `MultiNodeSampleConfig.psd1` ファイルを開きます。

1. 特定のパラメーター値をご自身のインフラストラクチャに合わせて変更します。

## <a name="run-the-deployment-script"></a>展開スクリプトを実行する

SDN Express スクリプトによって、お使いの SDN インフラストラクチャがデプロイされます。 スクリプトが完了すると、インフラストラクチャはワークロードのデプロイに使用できるようになります。

1. `README.md` ファイルで、デプロイ スクリプトの実行方法に関する最新情報を確認します。  

1. クラスター ホスト サーバーの管理者資格情報を持つユーザー アカウントから、次のコマンドを実行します。

    ```powershell
    SDNExpress\scripts\SDNExpress.ps1 -ConfigurationDataFile MultiNodeSampleConfig.psd1 -Verbose
    ```

1. ネットワーク コントローラー VM を作成した後、DNS サーバー上にあるネットワーク コントローラーのクラスター名に対して動的 DNS 更新を構成します。 詳細については、「[ネットワーク コントローラーをデプロイするための要件](/windows-server/networking/sdn/plan/installation-and-preparation-requirements-for-deploying-network-controller#step-3-configure-dynamic-dns-registration-for-network-controller)」の手順 3 を参照してください。

## <a name="next-steps"></a>次のステップ

VM を管理します。 詳細については、[VM の管理](../manage/vm.md)に関する記事を参照してください。