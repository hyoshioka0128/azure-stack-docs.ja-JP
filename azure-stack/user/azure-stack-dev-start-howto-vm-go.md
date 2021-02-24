---
title: Azure Stack Hub 内の VM に Go Web アプリをデプロイする
description: Azure Stack Hub 内の VM に Go Web アプリをデプロイする方法
author: mattbriggs
ms.topic: overview
ms.date: 12/2/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 12/2/2020
ms.openlocfilehash: a43dd6ccb766dfb7b67dad09c0fc9bba37d524b4
ms.sourcegitcommit: 9ef2cdc748cf00cd3c8de90705ea0542e29ada97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/02/2020
ms.locfileid: "96525321"
---
# <a name="deploy-a-go-web-app-to-a-vm-in-azure-stack-hub"></a>Azure Stack Hub 内の VM に Go Web アプリをデプロイする

Azure Stack Hub で Go Web アプリをホストする仮想マシン (VM) を作成することができます。 この記事では、サーバーを設定し、Go Web アプリをホストするようにサーバーを構成してから、アプリを Azure Stack Hub にデプロイします。

## <a name="create-a-vm"></a>VM の作成

1. 「[Web アプリをホストする Linux VM を Azure Stack Hub にデプロイする](azure-stack-dev-start-howto-deploy-linux.md)」の手順に従って、Azure Stack Hub で VM を設定します。

2. VM ネットワーク ウィンドウで、次のポートにアクセスできることを確認します。

    | Port | Protocol | 説明 |
    | --- | --- | --- |
    | 80 | HTTP | ハイパーテキスト転送プロトコル (HTTP) は、サーバーからの Web ページの配信に使用されるプロトコルです。 クライアントは、DNS 名または IP アドレスを使用して HTTP 経由で接続されます。 |
    | 443 | HTTPS | ハイパーテキスト転送プロトコル セキュア (HTTPS) は、セキュリティ証明書を要求し、情報の暗号化された転送を許可する、セキュリティで保護されたバージョンの HTTP です。 |
    | 22 | SSH | Secure Shell (SSH) は、セキュリティで保護された通信のための暗号化されたネットワーク プロトコルです。 SSH クライアントとのこの接続を使用して、VM を構成し、アプリをデプロイします。 |
    | 3389 | RDP | 省略可能。 リモート デスクトップ プロトコル (RDP) では、リモート デスクトップ接続を介して、ご利用のマシンでグラフィック ユーザー インターフェイスを使用できるようにします。   |
    | 3000 | Custom | ポート 3000 は、開発時に Go Web フレームワークによって使用されます。 運用サーバーでは、80 と 443 を介してトラフィックをルーティングします。 |

## <a name="install-go"></a>Go をインストールする

1. SSH クライアントを使用して VM に接続します。 手順については、「[PuTTY を使用して SSH 経由で接続する](azure-stack-dev-start-howto-ssh-public-key.md#connect-with-ssh-by-using-putty)」を参照してください。

1. ご自分の VM 上の bash プロンプトで、次のコマンドを入力します。

    ```bash  
    wget https://dl.google.com/go/go1.10.linux-amd64.tar.gz
    sudo tar -xvf go1.10.linux-amd64.tar.gz
    sudo mv go /usr/local
    ```

2. ご自分の VM 上に Go 環境を設定します。 SSH セッション内でご自分の VM に接続したままで、次のコマンドを入力します。

    ```bash  
    export GOROOT=/usr/local/go
    export GOPATH=$HOME/Projects/ADMFactory/Golang
    export PATH=$GOPATH/bin:$GOROOT/bin:$PATH

    vi ~/.profile
    ```

3. インストールを検証します。 SSH セッション内でご自分の VM に接続したままで、次のコマンドを入力します。

    ```bash  
        go version
    ```

3. [Git をインストール](https://git-scm.com) します。これは、広域分散型のバージョン管理およびソース コード管理 (SCM) システムです。 SSH セッション内でご自分の VM に接続したままで、次のコマンドを入力します。

    ```bash  
       sudo apt-get -y install git
    ```

## <a name="deploy-and-run-the-app"></a>アプリのデプロイと実行

1. VM 上でご自分の Git リポジトリを設定します。 SSH セッション内でご自分の VM に接続したままで、次のコマンドを入力します。

    ```bash  
       git clone https://github.com/appleboy/go-hello
    
       cd go-hello
       go get -d
    ```

2. アプリを起動します。 SSH セッション内でご自分の VM に接続したままで、次のコマンドを入力します。

    ```bash  
       go run hello-world.go
    ```

3. ご自分の新しいサーバーに移動します。 実行中のご自分の Web アプリケーションを確認できます。

    ```HTTP  
       http://yourhostname.cloudapp.net:3000
    ```

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub 向けの開発](azure-stack-dev-start.md)方法について、さらに学習する。
- [IaaS としての Azure Stack Hub 向けの一般的なデプロイ](azure-stack-dev-start-deploy-app.md)を確認する。
- Go プログラミング言語の詳細および Go の他のリソースについては、[Golang.org](https://golang.org) を参照してください。
