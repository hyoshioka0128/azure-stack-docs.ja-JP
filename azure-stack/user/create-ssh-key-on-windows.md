---
title: Azure Stack Hub で Linux 用の SSH キーを作成する
description: Azure Stack Hub で Linux 用の SSH キーを作成する方法について説明します
author: mattbriggs
ms.topic: article
ms.date: 7/24/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 2/28/2020
ms.openlocfilehash: 8b6ddaa6dab41cccce5759622ec15f4e5e886956
ms.sourcegitcommit: b2337a9309c52aac9f5a1ffd89f1426d6c178ad5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/28/2020
ms.locfileid: "87250743"
---
# <a name="create-an-ssh-key-for-linux-on-azure-stack-hub"></a>Azure Stack Hub で Linux 用の SSH キーを作成する

Windows マシンで Linux マシンの SSH (Secure Shell) キーを作成できます。 この記事の手順で生成された公開キーを、VM での SSH 認証に使用します。 Windows マシンを使用している場合は、Ubuntu on Windows をインストールし、bash、ssh、git、apt などのユーティリティを使用してターミナルを取得します。 **ssh-keygen** を実行してキーを作成します。

## <a name="open-bash-on-windows"></a>Windows で bash を開く

1. マシンに Windows Subsystem for Linux がインストールされていない場合は、[Ubuntu on Windows](https://www.microsoft.com/en-us/p/ubuntu/9nblggh4msv6?activetab=pivot:overviewtab) をインストールします。  
    Windows Subsystem for Linux の使用方法の詳細については、「[Windows Subsystem for Linux のドキュメント](/windows/wsl/about)」を参照してください。

2. ツールバーに「**Ubuntu**」と入力し、 **[Open]\(開く\)** を選択します。

## <a name="create-a-key-with-ssh-keygen"></a>ssh-keygen を使用してキーを作成する

1. bash プロンプトから次のコマンドを入力します。

    ```bash  
    ssh-keygen -t rsa
    ```

    bash で次のプロンプトが表示されます。

    ```bash
    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/username/.ssh/id_rsa):
    ```

2. ファイル名とパスフレーズを入力します。 もう一度パスフレーズを入力します。

    bash で次のように表示されます。

    ```bash
    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/user/.ssh/id_rsa): key.txt
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in key.txt.
    Your public key has been saved in key.txt.pub.
    The key fingerprint is:
    SHA256:xanotrealoN6z1/KChqeah0CYVeyhL50/0rq37qgy6Ik username@machine
    The key's randomart image is:
    +---[RSA 2048]----+
    |   o.     .      |
    |  . o.   +       |
    | + o .+ o o      |
    |o o .  O +       |
    | . o .o S .      |
    |  o +. .         |
    |.  o +..o. .     |
    |= . ooB +o+ .    |
    |E=..*X=*.. +.    |
    +----[SHA256]-----+
    ```

3. SSH 公開キーを表示するには、次のようにします。

    ```bash
    cat /home/<username>/<filename>
    ```

    bash では、次のようなものが生成されます。

    ```bash
    ssh-rsa AAAAB3NzaC1ycTHISISANEXAMPLEDITqEJRNrf6tXy9c0vKnMhiol1BFzHFV3
    +suXk6NDeFcA9uI58VdD/CuvG826R+3OPnXutDdl2MLyH3DGG1fJAHObUWQxmDWluhSGb
    JMHiw2L9Wnf9klG6+qWLuZgjB3TQdus8sZI8YdB4EOIuftpMQ1zkAJRAilY0p4QxHhKbU
    IkvWqBNR+rd5FcQx33apIrB4LMkjd+RpDKOTuSL2qIM2+szhdL5Vp5Y6Z1Ut1EpOrkbg1
    cVw7oW0eP3ROPdyNqnbi9m1UVzB99aoNXaepmYviwJGMzXsTkiMmi8Qq+F8/qy7i4Jxl0
    aignia880qOtQrvNEvyhgZOM5oDhgE3IJ username@machine
    ```

4. テキスト `ssh-rsa [...]` を `username@machinename` までコピーします。 テキストにキャリッジ リターンが含まれていないことを確認します。 このテキストは、VM または Kubernetes クラスターを AKS エンジンを使用して作成するときに使用できます。

5. Windows マシンを使用している場合は、 **\\\\wsl$** を使用して Linux ファイルにアクセスできます。

    1. ツールバーに「`\\wsl$`」を入力します。 ディストリビューションの既定のウィンドウが開きます。

    2. `\\wsl$\Ubuntu\home\<username>` に移動し、公開キーと秘密キーを見つけて安全な場所に保存します。

## <a name="next-steps"></a>次のステップ

- [AKS エンジンを使用して Azure Stack Hub に Kubernetes クラスターをデプロイする](azure-stack-kubernetes-aks-engine-deploy-cluster.md)
- [クイック スタート: Azure Stack Hub ポータルを使用して Linux サーバー VM を作成する](azure-stack-quick-linux-portal.md)
