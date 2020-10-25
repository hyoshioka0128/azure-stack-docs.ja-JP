---
title: Azure Stack Hub の VM に Ruby アプリをデプロイする
description: Azure Stack Hub 内の仮想マシンに Ruby アプリをデプロイします。
author: mattbriggs
ms.topic: overview
ms.date: 5/27/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 10/02/2019
ms.openlocfilehash: 7be8293180f03d9d82565aa27c74c8e8cbe7c9c1
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90573974"
---
# <a name="deploy-a-ruby-web-app-to-a-vm-in-azure-stack-hub"></a>Azure Stack Hub 内の VM に Ruby Web アプリをデプロイする

Azure Stack Hub でご自分の Ruby Web アプリをホストする VM を作成できます。 この記事では、サーバーを設定し、Ruby Web アプリをホストするようにサーバーを構成してから、アプリを Azure Stack Hub にデプロイします。

この記事では、Ruby および Ruby on Rails Web フレームワークを使用します。

## <a name="create-a-vm"></a>VM の作成

1. Azure Stack Hub でご自分の VM を設定します。 手順については、「[Web アプリをホストする Linux VM を Azure Stack Hub にデプロイする](azure-stack-dev-start-howto-deploy-linux.md)」を参照してください。

2. VM ネットワーク ウィンドウで、次のポートにアクセスできることを確認します。

    | Port | Protocol | 説明 |
    | --- | --- | --- |
    | 80 | HTTP | ハイパーテキスト転送プロトコル (HTTP) は、サーバーからの Web ページの配信に使用されるプロトコルです。 クライアントは、DNS 名または IP アドレスを使用して HTTP 経由で接続されます。 |
    | 443 | HTTPS | ハイパーテキスト転送プロトコル セキュア (HTTPS) は、セキュリティ証明書を要求し、情報の暗号化された転送を許可する、セキュリティで保護されたバージョンの HTTP です。 |
    | 22 | SSH | Secure Shell (SSH) は、セキュリティで保護された通信のための暗号化されたネットワーク プロトコルです。 SSH クライアントとのこの接続を使用して、VM を構成し、アプリをデプロイします。 |
    | 3389 | RDP | 省略可能。 リモート デスクトップ プロトコル (RDP) では、リモート デスクトップ接続を介して、ご利用のマシンでグラフィック ユーザー インターフェイスを使用できるようにします。   |
    | 3000 | Custom | 開発時に Ruby on Rails Web フレームワークによって使用されるポート。 運用サーバーでは、80 と 443 を介してトラフィックをルーティングします。 |

## <a name="install-ruby"></a>Ruby のインストール

1. SSH クライアントを使用して VM に接続します。 手順については、「[PuTTy を使用して SSH 経由で接続する](azure-stack-dev-start-howto-ssh-public-key.md#connect-with-ssh-by-using-putty)」を参照してください。

1. PPA リポジトリをインストールします。 ご自分の VM 上の bash プロンプトで、次のコマンドを入力します。

    ```bash  
    sudo apt -y install software-properties-common
    sudo apt-add-repository ppa:brightbox/ruby-ng

    sudo apt update
    ```

2. ご自分の VM に Ruby および Ruby on Rails をインストールします。 SSH セッション内でご自分の VM に接続したままで、次のコマンドを入力します。

    ```bash  
    sudo apt install ruby
    gem install rails -v 4.2.6
    ```

3. Ruby on Rails の依存関係をインストールします。 SSH セッション内でご自分の VM に接続したままで、次のコマンドを入力します。

    ```bash  
    sudo apt-get install make
    sudo apt-get install gcc
    sudo apt-get install sqlite3
    sudo apt-get install nodejs
    sudo gem install sqlite
    sudo gem install bundler
    ```

    > [!NOTE]  
    > Ruby on Rails の依存関係のインストール時に、`sudo gem install bundler` を繰り返し実行する必要がある場合があります。 インストールに失敗した場合は、エラー ログを確認して、問題を解決してください。

4. インストールを検証します。 SSH セッション内でご自分の VM に接続したままで、次のコマンドを入力します。

    ```bash  
        ruby -v
    ```

3. [Git をインストール](https://git-scm.com) します。これは、広域分散型のバージョン管理およびソース コード管理 (SCM) システムです。 SSH セッション内でご自分の VM に接続したままで、次のコマンドを入力します。

    ```bash  
       sudo apt-get -y install git
    ```

## <a name="create-and-run-an-app"></a>アプリの作成と実行

1. SSH セッション内でご自分の VM に接続したままで、次のコマンドを入力します。

    ```bash
        rails new myapp
        cd myapp
        rails server -b 0.0.0.0 -p 3000
    ```

2. ご自分の新しいサーバーに移動します。 実行中のご自分の Web アプリケーションを確認できます。

    ```HTTP  
       http://yourhostname.cloudapp.net:3000
    ```

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub 向けの開発](azure-stack-dev-start.md)方法について、さらに学習する。
- [IaaS としての Azure Stack Hub 向けの一般的なデプロイ](azure-stack-dev-start-deploy-app.md)を確認する。
- Ruby プログラミング言語の詳細および Ruby の他のリソースについては、[Ruby-lang.org](https://www.ruby-lang.org) を参照してください。
