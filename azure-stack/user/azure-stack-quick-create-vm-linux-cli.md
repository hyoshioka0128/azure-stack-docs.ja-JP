---
title: Azure Stack Hub で Azure CLI を使用して Linux VM を作成する
description: Azure Stack Hub 内で Azure CLI を使用して Linux 仮想マシンを作成します。
author: mattbriggs
ms.topic: quickstart
ms.date: 5/27/2020
ms.author: mabrigg
ms.lastreviewed: 01/14/2019
ms.openlocfilehash: b7b016d377267c9032a79c1e0f04bd1ac65e4185
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90571538"
---
# <a name="quickstart-create-a-linux-server-vm-by-using-the-azure-cli-in-azure-stack-hub"></a>クイック スタート:Azure Stack Hub 内で Azure CLI を使用して Linux サーバー VM を作成する

Azure CLI を使用して、Ubuntu Server 16.04 LTS 仮想マシン (VM) を作成できます。 この記事では、仮想マシンを作成し、使用します。 この記事では、次の方法についても説明します。

* リモート クライアントを使用して仮想マシンに接続する。
* NGINX Web サーバーをインストールし、既定のホーム ページを表示する。
* 使用されていないリソースをクリーンアップします。

## <a name="prerequisites"></a>前提条件

* Azure Stack Hub Marketplace 内の Linux イメージ

   Azure Stack Hub Marketplace には、既定では Linux イメージが含まれていません。 必要な Ubuntu Server 16.04 LTS イメージを Azure Stack Hub オペレーターに提供してもらってください。 オペレーターは、「[Azure から Azure Stack Hub に Marketplace の項目をダウンロードする](../operator/azure-stack-download-azure-marketplace-item.md)」の手順を使用できます。

* Azure Stack Hub には、そのリソースを作成して管理するために、Azure CLI の特定のバージョンが必要です。 Azure Stack Hub 用に構成された Azure CLI がない場合は、[Azure Stack Development Kit](../asdk/asdk-connect.md#connect-to-azure-stack-using-rdp) (または [VPN 経由で接続](../asdk/asdk-connect.md#connect-to-azure-stack-using-vpn)している場合は、Windows ベースの外部クライアント) にサインインし、[Azure CLI のインストールと構成](azure-stack-version-profiles-azurecli2.md)の手順に従います。

* Windows ユーザー プロファイルの *.ssh* ディレクトリに保存された *id_rsa.pub* という名前の Secure Shell (SSH) 公開キー。 SSH キーの作成の詳細については、「[SSH 公開キーの使用](azure-stack-dev-start-howto-ssh-public-key.md)」を参照してください。

## <a name="create-a-resource-group"></a>リソース グループを作成する

リソース グループは、Azure Stack Hub リソースのデプロイと管理を行うことができる論理コンテナーです。 開発キットまたは Azure Stack Hub 統合システムから、[az group create](/cli/azure/group#az-group-create) コマンドを実行してリソース グループを作成します。

> [!NOTE]
> 次のコード例では、すべての変数に値が割り当てられています。 しかし、独自の値を割り当てることができます。

次の例では、myResourceGroup という名前のリソース グループをローカルの場所に作成します。 

```cli
az group create --name myResourceGroup --location local
```

## <a name="create-a-virtual-machine"></a>仮想マシンの作成

[az vm create](/cli/azure/vm#az-vm-create) コマンドを使用して仮想マシンを作成します。 次の例では、myVM という名前の VM を作成します。 この例では、管理者ユーザー名に *Demouser*、管理者パスワードとして *Demouser@123* を使用します。 これらの値を、お使いの環境に適した内容に変更します。

```cli
az vm create \
  --resource-group "myResourceGroup" \
  --name "myVM" \
  --image "UbuntuLTS" \
  --admin-username "Demouser" \
  --admin-password "Demouser@123" \
  --location local
```

**PublicIpAddress** パラメーターでパブリック IP アドレスが返されます。 後で仮想マシンで使うため、アドレスをメモします。

## <a name="open-port-80-for-web-traffic"></a>Web トラフィック用にポート 80 を開く

この仮想マシンでは IIS Web サーバーを実行することになるため、インターネット トラフィックに対してポート 80 を開く必要があります。 ポートを開くために、[az vm open-port](/cli/azure/vm) コマンドを使用します。 

```cli
az vm open-port --port 80 --resource-group myResourceGroup --name myVM
```

## <a name="use-ssh-to-connect-to-the-virtual-machine"></a>SSH を使用して仮想マシンに接続する

SSH がインストールされたクライアント コンピューターから、仮想マシンに接続します。 Windows クライアント上で作業している場合は、[PuTTY](https://www.putty.org/) を使用して接続を作成します。 仮想マシンに接続するには、次のコマンドを使用します。

```bash
ssh <publicIpAddress>
```

## <a name="install-the-nginx-web-server"></a>NGINX Web サーバーのインストール

パッケージ リソースを更新し、最新の NGINX パッケージをインストールするため、次のスクリプトを実行します。

```bash
#!/bin/bash

# update package source
apt-get -y update

# install NGINX
apt-get -y install nginx
```

## <a name="view-the-nginx-welcome-page"></a>NGINX のようこそページの表示

NGINX Web サーバーがインストールされ、仮想マシン上のポート 80 が開かれたので、その仮想マシンのパブリック IP アドレスを使用して Web サーバーにアクセスできます。 そのためには、ブラウザーを開き、```http://<public IP address>``` に移動します。

![NGINX Web サーバーのようこそページ](./media/azure-stack-quick-create-vm-linux-cli/nginx.png)

## <a name="clean-up-resources"></a>リソースをクリーンアップする

不要になったリソースをクリーンアップします。 [az group delete](/cli/azure/group#az-group-delete) コマンドを使用して、それらを削除できます。 次のコマンドを実行します。

```cli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>次のステップ

このクイック スタートでは、Web サーバーがインストールされた基本の Linux サーバー仮想マシンをデプロイしました。 Azure Stack Hub 仮想マシンの詳細については、[Azure Stack Hub の仮想マシンの考慮事項](azure-stack-vm-considerations.md)に関するページを参照してください。
