---
title: Azure Stack Hub の Windows に AKS エンジンをデプロイする - ラグド
description: Kubernetes クラスターをデプロイおよび管理するために、Azure Stack Hub の Windows マシンを使用して AKS エンジンをホストする方法について説明します。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 2/1/2021
ms.author: mabrigg
ms.reviewer: alfredop
ms.lastreviewed: 12/20/2019
ms.openlocfilehash: b9f3aae35c756375f54a7b643a68d3a75dd8cde9
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99246998"
---
# <a name="install-the-aks-engine-on-windows-in-azure-stack-hub---ruggedized"></a>Azure Stack Hub の Windows に AKS エンジンをインストールする - ラグド

Kubernetes クラスターをデプロイおよび管理するには、Azure Stack Hub の Windows マシンを使用して AKS エンジンをホストできます。 この記事では、接続環境の Azure Stack Hub インスタンスと非接続環境の Azure Stack Hub インスタンスのクラスターを管理するためのクライアント VM を準備し、インストールを検証します。
<!-- TZLASDKFIX2 Line above was previously:
In this article, we look at preparing the client VM to manage your cluster for both connected and disconnected Azure Stack Hub instances, check the install, and setting up the client VM on the ASDK.
-->
## <a name="prepare-the-client-vm"></a>クライアント VM の準備

AKS エンジンとは、お使いの Kubernetes クラスターをデプロイおよび管理するために使用するコマンドライン ツールです。 このエンジンは、お使いの Azure Stack Hub のコンピューターで実行できます。 このコンピューターから AKS エンジンを実行して、お使いのクラスターの実行に必要な IaaS リソースとソフトウェアをデプロイします。 その後、このエンジンを実行しているコンピューターを使用して、お使いのクラスターで管理タスクを実行できます。

お使いのクライアント コンピューターを選択する場合には、次の点を考慮してください。

1. 障害が発生した場合にクライアント コンピューターを回復可能にする必要があるか。
3. クライアント コンピューターにどのように接続するかと、コンピューターがどのようにご自分のクラスターと通信するか。

## <a name="install-in-a-connected-environment"></a>接続されている環境へのインストール

クライアント VM をインストールし、インターネットに接続されている Azure Stack Hub 上のお使いの Kubernetes クラスターを管理できます。

1. Azure Stack Hub で Windows VM を作成します。 手順については、「[クイック スタート:Azure Stack Hub ポータルを使用して Windows サーバー VM を作成する](../../user/azure-stack-quick-windows-portal.md)」を参照してください。
2. お使いの VM に接続します。
3. [PowerShell の手順に従って Chocolatey をインストールします](https://chocolatey.org/install#install-with-powershellexe)。 

    Chocolatey の Web サイトによると:Chocolatey は、Windows のパッケージ マネージャーです。例えば、Windows 用の apt-get や yum などです。 必要なアプリケーションとツールを迅速にインストールするための分散型フレームワークとして設計されています。 現在 PowerShell を使用している NuGet インフラストラクチャ上に構築されており、ディストリビューションからドア、err、コンピューターへのパッケージの配信に重点を置いています。
4. [サポート対象の Kubernetes バージョン](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions)の表で、その AKS エンジンのバージョンを確認します。 この AKS ベースのエンジンは、ご自分の Azure Stack Hub の Marketplace で入手できるようになっている必要があります。 コマンドを実行するときに、バージョン `--version v0.48.0` を指定する必要があります。 バージョンを指定しないと、このコマンドによって最新バージョンがインストールされ、最新バージョンに必要な VHD イメージがご自分の Marketplace にはない可能性があります。
5. 管理者特権でのプロンプトで次のコマンドを実行し、バージョン番号を含めます。

    ```PowerShell  
        choco install aks-engine --version 0.55.4 -y
    ```

> [!NOTE]  
> このインストール方法が失敗する場合は、[切断されている環境](#install-in-a-disconnected-environment)の手順を試すか、別のパッケージ マネージャーである [GoFish](../../user/azure-stack-kubernetes-aks-engine-troubleshoot.md#try-gofish) を試してください。

## <a name="install-in-a-disconnected-environment"></a>切断されている環境へのインストール

インターネットから切断されている Azure Stack Hub 上のお使いの Kubernetes クラスターは、クライアント VM をインストールして管理できます。

1.  インターネットにアクセスできるコンピューターから、GitHub [Azure/aks-engine](https://github.com/Azure/aks-engine/releases/latest) に移動します。 `aks-engine-v0.38.8-windows-amd64.tar.gz` など、Windows マシンのアーカイブ (*.tar.gz) をダウンロードします。

2.  お使いの Azure Stack Hub インスタンスにストレージ アカウントを作成し、AKS エンジン バイナリを使用してアーカイブ ファイル (*.tar.gz) をアップロードします。 Azure Storage Explorer の使用方法については、[Azure Stack Hub と Azure Storage Explorer](../../user/azure-stack-storage-connect-se.md) に関するページを参照してください。

3. Azure Stack Hub で Windows VM を作成します。 手順については、「[クイック スタート:Azure Stack Hub ポータルを使用して Windows サーバー VM を作成する](../../user/azure-stack-quick-windows-portal.md)」を参照してください

4.  アーカイブ ファイル (* tar.gz) をアップロードした Azure Stack Hub ストレージ アカウントの BLOB URL から、お使いの管理 VM にファイルをダウンロードします。 コマンド プロンプトから、アクセス権のあるディレクトリにアーカイブを抽出します。

5. お使いの VM に接続します。

6. [PowerShell の手順に従って Chocolatey をインストールします](https://chocolatey.org/install#install-with-powershellexe)。 

7.  管理者特権でのプロンプトで、次のコマンドを実行します。 適切なバージョン番号を含めます。

    ```PowerShell  
        choco install aks-engine --version 0.55.4 -y
    ```

## <a name="verify-the-installation"></a>インストールの確認

お使いのクライアント VM がセットアップされたら、AKS エンジンがインストールされていることを確認します。

1. お使いのクライアント VM に接続します。
2. 次のコマンドを実行します。

    ```PowerShell  
    aks-engine version
    ```

クライアント VM に AKS エンジンがインストールされていることを確認できない場合は、[AKS エンジンのインストールのトラブルシューティング](../../user/azure-stack-kubernetes-aks-engine-troubleshoot.md)に関するページを参照してください。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [AKS エンジンを使用して Azure Stack Hub に Kubernetes クラスターをデプロイする](../../user/azure-stack-kubernetes-aks-engine-deploy-cluster.md)
