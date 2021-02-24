---
title: Azure Stack Hub 上の AKS エンジンに Marketplace アイテムのクラスターを移動する
description: Azure Stack Hub 上の AKS エンジンに Marketplace アイテムのクラスターを移動する方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 2/1/2021
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 09/02/2020
ms.openlocfilehash: db9d1657760b558bf592bcf84a407855aeb5adcd
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99246930"
---
# <a name="move-your-marketplace-item-cluster-to-the-aks-engine-on-azure-stack-hub"></a>Azure Stack Hub 上の AKS エンジンに Marketplace アイテムのクラスターを移動する

Kubernetes Azure Stack Hub Marketplace アイテムでは、Azure Resource Manager テンプレートを使用してデプロイ仮想マシン (VM) をデプロイし、AKS エンジンをダウンロードしてインストールし、クラスターの説明に使用する入力 API モデルを生成します。その後、AKS エンジンが VM で実行され、クラスターがデプロイされます。 この記事では、AKS エンジンとそれに対応するファイルにアクセスして、Kubernetes クラスターで更新およびスケール操作を実行できるようにする方法について説明します。

## <a name="access-aks-engine-in-the-dvm"></a>DVM の AKS エンジンへのアクセス

Kubernetes Azure Stack Hub Marketplace アイテムによって開始されたデプロイが正常に終了すると、クラスター用に指定したリソース グループ内に作成されたデプロイ VM に、クラスターのデプロイに使用された AKS エンジンがインストールされていることを確認できます。この VM は Kubernetes クラスターの一部ではなく、独自の VNet に作成されます。 VM を見つけて、その中に AKS エンジンを配置する手順を次に示します。

1.  Azure Stack Hub ユーザー ポータルを開き、Kubernetes クラスター用に指定したリソース グループを検索します。
2.  リソース グループ内のデプロイ VM を見つけます。 名前は **vmd-** というプレフィックスで始まります。
3.  デプロイ VM を選択します。 [概要] で、**パブリック IP アドレスを見つけます。 このアドレスと、Putty などのコンソール アプリを使用して、VM への SSH セッションを確立します。
4.  デプロイ VM のセッションでは、AKS エンジンは、パス `./var/lib/waagent/custom-script/download/0/bin/aks-engine` にあります。
5.  aks-engine に入力として使用したクラスターを記述した `.json` ファイルを見つけます。 ファイルは `/var/lib/waagent/custom-script/download/0/bin/azurestack.json` にあります。 ファイルには、クラスターのデプロイに使用されるサービス プリンシパルの資格情報があることに注意してください。 ファイルを保存する場合は、ファイルを保護されたストアに転送してください。
6.  `/var/lib/waagent/custom-script/download/0/_output/<resource group name>` で AKS エンジンによって生成された出力ディレクトリを見つけます。 このディレクトリで、パス `/var/lib/waagent/custom-script/download/0/bin/apimodel.json` の出力 `apimodel.json` を見つけます。 ディレクトリと `apimodel.json` ファイルには、Kubernetes クラスターのデプロイに必要なすべての証明書、キー、および資格情報が含まれています。 これらのリソースは安全な場所に保管してください。
7.  Kubernetes 構成ファイル (多くの場合は **kubeconfig** というファイル名) をパス `/var/lib/waagent/custom-script/download/0/_output/k8smpi00/kubeconfig/kubeconfig.<location>.json` で探します。ここで、 **\<location>** は Azure Stack Hub の場所の識別子に相当します。 このファイルは、Kubernetes クラスターにアクセスするように **kubectl** を設定する場合に役立ちます。


## <a name="use-the-aks-engine-with-your-newly-created-cluster"></a>新しく作成したクラスターで AKS エンジンを使用する

aks-engine、入力 apimodel.json ファイル、出力ディレクトリ、および出力 apimodel.json ファイルを見つけ、それらを安全な場所に保存したら、任意の Linux VM で AKS エンジン バイナリと出力 `apimodel.json` を使用できます。

1.  AKS エンジンを引き続き使用し、**アップグレード** や **スケール** などの操作を実行するには、**aks-engine** バイナリ ファイルをターゲット マシンにコピーします。 同じ **vmd-** マシンをディレクトリに使用している場合。

2.  クラスターの名前、または新しいクラスターを参照する他の記憶しやすい名前でディレクトリを作成し、出力 apimodel.json ファイルをその中に保存します。 このファイルには資格情報が含まれているため、保護された場所であることを確認してください。 その後、aks-engine を実行して、[スケール](azure-stack-kubernetes-aks-engine-scale.md)や[アップグレード](azure-stack-kubernetes-aks-engine-upgrade.md)などの操作を実行できます

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub 上の AKS エンジン](azure-stack-kubernetes-aks-engine-overview.md)を確認してください  
- [Azure Stack Hub 上の AKS エンジンのトラブルシューティング](azure-stack-kubernetes-aks-engine-troubleshoot.md)  
