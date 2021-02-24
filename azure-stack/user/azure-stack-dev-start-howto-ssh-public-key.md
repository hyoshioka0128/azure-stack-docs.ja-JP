---
title: Azure Stack Hub で SSH 公開キーを使用する方法
description: SSH 公開キーの使用方法
author: mattbriggs
ms.topic: overview
ms.date: 12/2/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 12/2/2020
ms.openlocfilehash: 20e8afdae36f7a182f20aff1f4162d36378ca313
ms.sourcegitcommit: 9ef2cdc748cf00cd3c8de90705ea0542e29ada97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/02/2020
ms.locfileid: "96525457"
---
# <a name="use-an-ssh-public-key"></a>SSH 公開キーの使用

開発用マシンから、ご自分の Web アプリをホストする Azure Stack Hub インスタンス内のサーバー VM へのオープン SSH 接続を使用するには、SSH (Secure Shell) 公開キーと秘密キーのペアを作成する必要がある場合があります。 

この記事では、キーを作成してから、それらを使用してサーバーに接続します。 SSH クライアントを使用して Linux サーバー上の bash プロンプトを取得したり、SFTP (Secure FTP) クライアントを使用してサーバーとの間でファイルを移動したりすることができます。

## <a name="create-an-ssh-public-key-on-windows"></a>Windows 上で SSH 公開キーを作成する

このセクションでは、ご使用の Azure Stack Hub インスタンスで Linux マシンへのセキュリティで保護された接続を作成するときに使用する SSH 公開キーと秘密キーのペアを、PuTTY Key Generator を使って作成します。 PuTTY は、SSH や Telnet 経由でサーバーに接続できるようにする、無料のターミナル エミュレーターです。

1. [ご使用のマシン用の PuTTY をダウンロードしてインストールします。](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

1. PuTTY Key Generator を開きます。

    ![[キー] ボックスが空の状態の PuTTY Key Generator](media/azure-stack-dev-start-howto-ssh-public-key/001-putty-key-gen-start.png)

1. **[パラメーター]** で、 **[RSA]** を選択します。

1. **[Number of bits in a generated key]\(生成されるキーのビット数\)** ボックスに、「**2048**」と入力します。  

1. **[Generate] \(生成)** を選択します。

1. **[キー]** 領域で、空白領域の上にカーソルを動かし、いくつかのランダムな文字を生成します。

    ![[キー] ボックスが設定された状態の PuTTY Key Generator](media/azure-stack-dev-start-howto-ssh-public-key/002-putty-key-gen-result.png)

1. **[キーのパスフレーズ]** を入力し、 **[パスフレーズの確認入力]** ボックスでそれを確認します。 後で使用できるようにパスフレーズをメモしておきます。

1. **[Save public key]\(公開キーの保存\)** を選択し、アクセスできる場所に保存します。

1. **[Save private key]\(秘密キーの保存\)** を選択し、アクセスできる場所に保存します。 これが公開キーとペアになっていることを忘れないでください。

ご自分の公開キーは、保存したテキスト ファイル内に格納されます。 テキストは次のようになります。

```text  
---- BEGIN SSH2 PUBLIC KEY ----
Comment: "rsa-key-20190330"
THISISANEXAMPLEDONOTUSE AAAAB3NzaC1yc2EAAAABJQAAAQEAthW2CinpqhXq
9uSa8/lSH7tLelMXnFljSrJIcpxp3MlHlYVbjHHoKfpvQek8DwKdOUcFIEzuStfT
Z8eUI1s5ZXkACudML68qQT8R0cmcFBGNY20K9ZMz/kZkCEbN80DJ+UnWgjdXKLvD
Dwl9aQwNc7W/WCuZtWPazee95PzAShPefGZ87Jp0OCxKaGYZ7UXMrCethwfVumvU
aj+aPsSThXncgVQUhSf/1IoRtnGOiZoktVvt0TIlhxDrHKHU/aZueaFXYqpxDLIs
BvpmONCSR3YnyUtgWV27N6zC7U1OBdmv7TN6M7g01uOYQKI/GQ==
---- END SSH2 PUBLIC KEY ----
```

アプリケーションでキーが要求された場合は、テキスト ファイルのコンテンツ全体をコピーして貼り付けます。

## <a name="connect-with-ssh-by-using-putty"></a>PuTTY を使用して SSH で接続する

PuTTY をインストールした場合は、PuTTY Key Generator と SSH クライアントの両方があります。 このセクションでは、SSH クライアントである PuTTY を開き、接続値と SSH キーを構成します。 Azure Stack Hub インスタンスと同じネットワークで作業している場合は、ご利用の VM に接続します。

接続する前に、次が必要となります。
- PuTTY
- 認証の種類として SSH 公開キーを使用する、Azure Stack Hub インスタンス内の Linux マシンの IP アドレスとユーザー名。
- マシン用に開かれるポート 22。
- マシンの作成時に使用した公開 SSH キー。
- ご使用の Azure Stack Hub インスタンスと同じネットワーク上にある、PuTTY を実行しているクライアント マシン。

1. PuTTY を開きます。

    ![[PuTTY Configuration]\(PuTTY 構成\) ウィンドウ](media/azure-stack-dev-start-howto-ssh-public-key/002-putty-connect.png)

2. **[Host Name (or IP address)]\(ホスト名 (または IP アドレス)\)** ボックスに、マシンのユーザー名とパブリック IP アドレス ( **username@192.XXX.XXX.XX** など) を入力します。 
3. **[ポート]** が **[22]** で、 **[接続の種類]** が **[SSH]** であることを確認します。
4. **[カテゴリ]** ツリーで、 **[SSH]** と **[認証]** を展開します。

    ![[PuTTY Configuration]\(PuTTY 構成\) ウィンドウ - SSH 秘密キー](media/azure-stack-dev-start-howto-ssh-public-key/002-putty-set-private-key.png)

5. **[Private key file for authentication]\(認証のための秘密キー ファイル\)** ボックスの横にある **[参照]** を選択してから、公開キーと秘密キーのペアの秘密キー ファイル ( *\<filename>.ppk*) を検索します。
6. **[カテゴリ]** ツリーで、 **[セッション]** を選択します。

    ![[PuTTY Configuration]\(PuTTY 構成\) ウィンドウの [Saved Sessions]\(保存されたセッション\) ボックス](media/azure-stack-dev-start-howto-ssh-public-key/003-puTTY-save-session.png)

7. **[Saved Sessions]\(保存されたセッション\)** で、セッションの名前を入力してから **[保存]** を選択します。
8. **[Saved Sessions]\(保存されたセッション\)** リストで、セッションの名前を選択してから **[読み込み]** を選びます。
9. **[Open (開く)]** を選択します。 SSH セッションが開きます。

## <a name="connect-with-sftp-with-filezilla"></a>FileZilla を使用して SFTP で接続する

ご自分の Linux マシンとの間でファイルを移動する場合は、SFTP (Secure FTP) をサポートする FTP クライアントである、FileZilla を使用できます。 FileZilla は Windows 10、Linux、および macOS 上で実行されます。 FileZilla クライアントでは、FTP、FTPS (FTP over TLS)、および SFTP がサポートされます。 これは、GNU General Public License の条件の下、無料で配布されているオープンソース ソフトウェアです。

### <a name="set-your-connection"></a>ご自分の接続を設定する

1. [FileZilla をダウンロードしてインストールします](https://filezilla-project.org/download.php)。
1. FileZilla を開きます。
1. **[File]\(ファイル\)**  >  **[Site Manager]\(サイト マネージャー\)** の順に選択します。

    ![FileZilla の [Site Manager]\(サイト マネージャー\) ウィンドウ](media/azure-stack-dev-start-howto-ssh-public-key/005-filezilla-file-manager.png)

1. **[プロトコル]** ドロップダウン リストで、 **[SFTP - SSH File Transfer Protocol]\(SFTP - SSH ファイル転送プロトコル\)** を選択します。
1. **[ホスト]** ボックスに、ご自分のマシンのパブリック IP アドレスを入力します。
1. **[ログオンの種類]** ボックスで、 **[標準]** を選択します。
1. ユーザー名とパスワードを入力します。
1. **[OK]** を選択します。
1. **[Edit]\(編集\)**  >  **[Settings]\(設定\)** の順に選択します。

    ![FileZilla の [設定] ウィンドウ](media/azure-stack-dev-start-howto-ssh-public-key/006-filezilla-add-private-key.png)

1. **[ページの選択]** ツリーで、 **[接続]** を展開してから **[SFTP]** を選択します。
1. **[Add key file]\(鍵ファイルを追加\)** を選択し、ご自分の秘密キー ファイル ( *\<filename>.ppk* など) を入力します。
1. **[OK]** を選択します。

### <a name="open-your-connection"></a>ご自分の接続を開く

1. FileZilla を開きます。
1. **[File]\(ファイル\)**  >  **[Site Manager]\(サイト マネージャー\)** の順に選択します。
1. ご自分のサイトの名前を選択してから、 **[接続]** を選びます。

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub 内で開発環境を設定する](azure-stack-dev-start.md)方法について学習する。
