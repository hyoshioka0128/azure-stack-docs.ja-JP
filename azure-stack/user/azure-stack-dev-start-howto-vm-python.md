---
title: Azure Stack Hub 内の VM に Python Web アプリをデプロイする
description: Azure Stack Hub 内の仮想マシンに Python Web アプリをデプロイします。
author: mattbriggs
ms.topic: overview
ms.date: 07/24/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 3/12/2020
ms.openlocfilehash: 59c6d473a3ac07d82ae40dedc0ff2b719ee51807
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90571815"
---
# <a name="deploy-a-python-web-app-to-a-vm-in-azure-stack-hub"></a>Azure Stack Hub 内の VM に Python Web アプリをデプロイする

Azure Stack Hub でご自分の Python Web アプリをホストする VM を作成することができます。 この記事では、サーバーを設定し、Python Web アプリをホストするようにサーバーを構成してから、アプリを Azure Stack Hub にデプロイします。

この記事では、Nginx サーバー上の仮想環境内で Flask を実行している Python 3.x を使用します。

## <a name="create-a-vm"></a>VM の作成

1. 「[Web アプリをホストする Linux VM を Azure Stack Hub にデプロイする](azure-stack-dev-start-howto-deploy-linux.md)」の手順に従って、Azure Stack Hub で VM を設定します。

2. VM ネットワーク ウィンドウで、次のポートにアクセスできることを確認します。

    | Port | Protocol | 説明 |
    | --- | --- | --- |
    | 80 | HTTP | ハイパーテキスト転送プロトコル (HTTP) は、サーバーからの Web ページの配信に使用されるプロトコルです。 クライアントは、DNS 名または IP アドレスを使用して HTTP 経由で接続されます。 |
    | 443 | HTTPS | ハイパーテキスト転送プロトコル セキュア (HTTPS) は、セキュリティ証明書を要求し、情報の暗号化された転送を許可する、セキュリティで保護されたバージョンの HTTP です。 |
    | 22 | SSH | Secure Shell (SSH) は、セキュリティで保護された通信のための暗号化されたネットワーク プロトコルです。 SSH クライアントとのこの接続を使用して、VM を構成し、アプリをデプロイします。 |
    | 3389 | RDP | 省略可能。 リモート デスクトップ プロトコル (RDP) では、リモート デスクトップ接続を介して、ご利用のマシンでグラフィック ユーザー インターフェイスを使用できるようにします。   |
    | 5000、8000 | Custom | 開発時に Flask Web フレームワークによって使用されるポート。 運用サーバーでは、80 と 443 を介してトラフィックをルーティングします。 |

## <a name="install-python"></a>Python のインストール

1. SSH クライアントを使用して VM に接続します。 手順については、「[PuTTy を使用して SSH 経由で接続する](azure-stack-dev-start-howto-ssh-public-key.md#connect-with-ssh-by-using-putty)」を参照してください。
2. ご自分の VM 上の bash プロンプトで、次のコマンドを入力します。

    ```bash  
    sudo apt-get -y install python3 python3-venv python3-dev
    ```

3. インストールを検証します。 SSH セッション内でご自分の VM に接続したままで、次のコマンドを入力します。

    ```bash  
        python -version
    ```

3. [Nginx をインストール](https://www.nginx.com/resources/wiki/)します。これは、軽量な Web サーバーです。 SSH セッション内でご自分の VM に接続したままで、次のコマンドを入力します。

    ```bash  
       sudo apt-get -y install nginx git
    ```

4. [Git をインストール](https://git-scm.com) します。これは、広域分散型のバージョン管理およびソース コード管理 (SCM) システムです。 SSH セッション内でご自分の VM に接続したままで、次のコマンドを入力します。

    ```bash  
       sudo apt-get -y install git
    ```

## <a name="deploy-and-run-the-app"></a>アプリのデプロイと実行

1. VM 上でご自分の Git リポジトリを設定します。 SSH セッション内でご自分の VM に接続したままで、次のコマンドを入力します。

    ```bash  
       git clone https://github.com/Azure-Samples/azure-stack-hub-flask-hello-world.git
    
       cd azure-stack-hub-flask-hello-world
    ```

2. 仮想環境を作成し、そこにパッケージの依存関係をすべて設定します。 SSH セッション内でご自分の VM に接続したままで、次のコマンドを入力します。

    ```bash  
    python3 -m venv venv
    source venv/bin/activate
    pip install -r requirements.txt
    
    export FLASK_APP=application.py
    # export FLASK_DEBUG=1 
    flask run -h 0.0.0.0
    ```

3. ご自分の新しいサーバーに移動します。 実行中のご自分の Web アプリケーションを確認できます。

    ```HTTP  
       http://yourhostname.cloudapp.net:5000
    ```

## <a name="update-your-server"></a>ご自分のサーバーを更新する

1. SSH セッション内でご自分の VM に接続します。 Ctrl + C キーを押して、サーバーを停止します。

2. 次のコマンドを入力します。

    ```bash  
    deactivate
    open the git repo
    git pull
    ```

3. 仮想環境をアクティブ化し、アプリを起動します。

    ```bash  
    source venv/bin/activate
    export FLASK_APP=application.py
    flask run -h 0.0.0.0
    ```

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub 向けの開発](azure-stack-dev-start.md)方法について、さらに学習する。
- [IaaS としての Azure Stack Hub 向けの一般的なデプロイ](azure-stack-dev-start-deploy-app.md)を確認する。
- Python プログラミング言語の詳細および Python の他のリソースについては、[Python.org](https://www.python.org) を参照してください。
