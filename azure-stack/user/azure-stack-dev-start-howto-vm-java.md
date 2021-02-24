---
title: Azure Stack Hub 内の VM に Java WAR をデプロイする
description: Azure Stack Hub 内の仮想マシンに Java WAR をデプロイします。
author: mattbriggs
ms.topic: how-to
ms.date: 12/2/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 12/2/2020
ms.custom: conteperfq4
ms.openlocfilehash: 12aa4c0e666085b7692a55ff795c07d9b05aa96c
ms.sourcegitcommit: 9ef2cdc748cf00cd3c8de90705ea0542e29ada97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/02/2020
ms.locfileid: "96525415"
---
# <a name="deploy-a-java-web-app-to-a-vm-in-azure-stack-hub"></a>Azure Stack Hub 内の VM に Java Web アプリをデプロイする

Azure Stack Hub でご自分の Java Web アプリをホストする仮想マシン (VM) を作成できます。 この記事では、Azure Stack Hub の Linux VM で Apache Tomcat サーバーをインストールし、適切なポートを開いて構成します。 その後、サーバーに Java Web アプリケーション リソース (WAR) ファイルを読み込みます。 WAR ファイルは、一連の Java アーカイブ (JAR) ファイル、つまり、クラス、テキスト、イメージ、XML、HTML、および Web アプリケーションの配信に使用される他のリソースなどの Java リソースを含む圧縮ファイルを配布するために使用されます。

## <a name="create-a-vm"></a>VM の作成

1. 「[Web アプリをホストする Linux VM を Azure Stack Hub にデプロイする](azure-stack-dev-start-howto-deploy-linux.md)」の手順に従って、Azure Stack Hub で VM を設定します。

2. VM ネットワーク ウィンドウで、次のポートにアクセスできることを確認します。

    | Port | Protocol | 説明 |
    | --- | --- | --- |
    | 80 | HTTP | ハイパーテキスト転送プロトコル (HTTP) は、サーバーからの Web ページの配信に使用されるプロトコルです。 クライアントは、DNS 名または IP アドレスを使用して HTTP 経由で接続されます。 |
    | 443 | HTTPS | ハイパーテキスト転送プロトコル セキュア (HTTPS) は、セキュリティ証明書を要求し、情報の暗号化された転送を許可する、セキュリティで保護されたバージョンの HTTP です。 |
    | 22 | SSH | Secure Shell (SSH) は、セキュリティで保護された通信のための暗号化されたネットワーク プロトコルです。 SSH クライアントとのこの接続を使用して、VM を構成し、アプリをデプロイします。 |
    | 3389 | RDP | 省略可能。 リモート デスクトップ プロトコル (RDP) では、リモート デスクトップ接続を介して、ご利用のマシンでグラフィック ユーザー インターフェイスを使用できるようにします。   |
    | 8080 | Custom | Apache Tomcat サービスの既定のポート。 運用サーバーでは、80 と 443 を介してトラフィックをルーティングします。 |

## <a name="install-java"></a>Java のインストール

1. SSH クライアントを使用して VM に接続します。 手順については、[PuTTY を使用する SSH 経由での接続](azure-stack-dev-start-howto-ssh-public-key.md#connect-with-ssh-by-using-putty)に関する記述を参照してください。

2. ご自分の VM 上の Bash プロンプトで、次のコマンドを実行します。

    ```bash  
        sudo apt-get install default-jdk
    ```

3. インストールを検証します。 SSH セッション内でご自分の VM に接続したままで、次のコマンドを実行します。

    ```bash  
        java -version
    ```

## <a name="install-and-configure-tomcat"></a>Tomcat をインストールして構成する

1. SSH クライアントを使用して VM に接続します。 手順については、[PuTTY を使用する SSH 経由での接続](azure-stack-dev-start-howto-ssh-public-key.md#connect-with-ssh-by-using-putty)に関する記述を参照してください。

1. 次のようにして、Tomcat ユーザーを作成します。

    a. 次のコマンドを実行して、新しい Tomcat グループを作成します。

    ```bash  
        sudo groupadd tomcat
    ```
     
    b. 新しい Tomcat ユーザーを作成します。 */opt/tomcat* のホーム ディレクトリを使用して、このユーザーを Tomcat グループに追加します。 このディレクトリに Tomcat をデプロイします。

    ```bash  
        sudo useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat
    ```

1. 次のようにして、Tomcat をインストールします。

    a. [Tomcat 8 のダウンロード ページ](http://tomcat.apache.org/download-80.cgi)から、Tomcat 8 の最新バージョンの tar の URL を入手します。

    b. cURL を使用し、リンクを使って最新バージョンをダウンロードします。 次のコマンドを実行します。

    ```bash  
        cd /tmp 
        curl -O <URL for the tar for the latest version of Tomcat 8>
    ```

    c. Tomcat を */opt/tomcat* ディレクトリにインストールします。 フォルダーを作成してから、アーカイブを開きます。

    ```bash  
        sudo mkdir /opt/tomcat
        sudo tar xzvf apache-tomcat-8*tar.gz -C /opt/tomcat --strip-components=1
        sudo chown -R tomcat webapps/ work/ temp/ logs/
    ```

1. 次のコマンドを実行して、Tomcat のアクセス許可を更新します。

    ```bash  
        sudo chgrp -R tomcat /opt/tomcat
        sudo chmod -R g+r conf
        sudo chmod g+x conf
    ```

1. *systemd* サービス ファイルを作成します。これにより、Tomcat をサービスとして実行できます。

   a. Java をインストールした場所を Tomcat で把握する必要があります。 このパスは一般に *JAVA_HOME* と呼ばれます。 次を実行して、その場所を調べます。

    ```bash  
        sudo update-java-alternatives -l
    ```

    次のようなものが生成されます。

    ```Text  
        Output
        java-1.8.0-openjdk-amd64       1081       /usr/lib/jvm/java-1.8.0-openjdk-amd64
    ```

    *JAVA_HOME* 変数の値は、出力からパスを取り出し、 */jre* を追加することによって構築できます。 たとえば、上記の例の */usr/lib/jvm/java-1.8.0-openjdk-amd64/jre* を使用します。

    b. ご自分のサーバーからの値を使用して、systemd サービス ファイルを作成します。

    ```bash  
        sudo nano /etc/systemd/system/tomcat.service
    ```

    c. ご自分のサービス ファイルに次の内容を貼り付けます。 必要に応じて *JAVA_HOME* の値を変更し、ご使用のシステム上で見つけた値と一致させます。 CATALINA_OPTS で指定されているメモリ割り当ての設定を変更することもできます。

    ```Text  
        [Unit]
        Description=Apache Tomcat Web Application Container
        After=network.target
        
        [Service]
        Type=forking
        
        Environment=JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64/jre
        Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
        Environment=CATALINA_HOME=/opt/tomcat
        Environment=CATALINA_BASE=/opt/tomcat
        Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
        Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'
        
        ExecStart=/opt/tomcat/bin/startup.sh
        ExecStop=/opt/tomcat/bin/shutdown.sh
    
        User=tomcat
        Group=tomcat
        UMask=0007
        RestartSec=10
        Restart=always
        
        [Install]
        WantedBy=multi-user.target
    ```

    d. ファイルを保存して閉じます。

    e. systemd デーモンを再読み込みして、ご自分のサービス ファイルを認識させます。

    ```bash  
        sudo systemctl daemon-reload
    ```

    f. Tomcat サービスを開始します。 

    ```bash  
        sudo systemctl start tomcat
    ```

    g. 以下を入力し、エラーが発生することなく開始されたことを確認します。

    ```bash  
        sudo systemctl status tomcat
    ```

1. Tomcat サーバーを確認します。 Tomcat では、一般的な要求の受け入れにポート 8080 を使用します。 次のコマンドを実行して、そのポートへのトラフィックを許可します。

    ```bash  
        sudo ufw allow 8080
    ```

    ご自分の Azure Stack Hub VM 用の *受信ポート規則* を追加していない場合は、ここで追加します。 詳細については、「[VM の作成](#create-a-vm)」を参照してください。

1. Azure Stack Hub と同じネットワークでブラウザーを開いてから、サーバー *yourmachine.local.cloudapp.azurestack.external:8080* を開きます。

    ![Apache Tomcat ページを示すスクリーンショット。](media/azure-stack-dev-start-howto-vm-java/apache-tomcat.png)

    ご自分のサーバー上の Apache Tomcat ページが読み込まれます。 次に、Server Status、Manager App、Host Manager にアクセスできるようにサーバーを構成します。

1. サービス ファイルを有効にして、ご自分のサーバーを再起動したときに Tomcat が自動的に開始されるようにします。

    ```bash  
        sudo systemctl enable tomcat
    ```

1. Web 管理インターフェイスへのアクセスを自分自身に許可するには、Tomcat サーバーを構成します。 

   a. *tomcat-users.xml* ファイルを編集し、ロールとユーザーを定義してサインインできるようにします。 `manager-gui` と `admin-gui` にアクセスするユーザーを定義します。

    ```bash  
        sudo nano /opt/tomcat/conf/tomcat-users.xml
    ```

   b. 次の要素を `<tomcat-users>` セクションに追加します。

    ```XML  
        <role rolename="tomcat"/>
        <user username="<username>" password="<password>" roles="tomcat,manager-gui,admin-gui"/>
    ```

    たとえば、最終的なファイルは次のようになります。

    ```XML  
        <tomcat-users xmlns="http://tomcat.apache.org/xml"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
              version="1.0">
        <role rolename="tomcat"/>
        <user username="tomcatuser" password="changemepassword" roles="tomcat,manager-gui,admin-gui"/>
        </tomcat-users>
    ```

    c. ファイルを保存して閉じます。

1. Tomcat は *Manager* と *Host Manager* アプリへのアクセスをサーバーからの接続に制限します。 Azure Stack Hub 内の VM 上に Tomcat をインストールするため、この制限を解除する必要があります。 適切な *context.xml* ファイルを編集して、これらのアプリに対する IP アドレスの制限を変更します。

    a. Manager アプリで *context.xml* を更新します。

    ```bash  
        sudo nano /opt/tomcat/webapps/manager/META-INF/context.xml
    ```

    b. IP アドレスの制限をコメント アウトしてすべての接続を許可するか、Tomcat への接続に使用するマシンの IP アドレスを追加します。

    ```XML  
    <Context antiResourceLocking="false" privileged="true" >
        <!--<Valve className="org.apache.catalina.valves.RemoteAddrValve"
                allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />-->
    </Context>
    ```

    c. ファイルを保存して閉じます。

    d. Host Manager アプリの *context.xml* を同じように更新します。

    ```bash  
        sudo nano /opt/tomcat/webapps/host-manager/META-INF/context.xml
    ```

    e. ファイルを保存して閉じます。

1. この変更でサーバーを更新するには、Tomcat サービスを再起動します。

    ```bash  
        sudo systemctl restart tomcat
    ```

1. Azure Stack Hub と同じネットワークでブラウザーを開いてから、サーバー *yourmachine.local.cloudapp.azurestack.external:8080* を開きます。

    a. Tomcat サーバーの状態を確認し、アクセス権があることを確認するには、 **[サーバーの状態]** を選択します。

    b. ご自分の Tomcat の資格情報を使用してサインインします。

    ![Azure Stack Hub VM 上の Apache Tomcat](media/azure-stack-dev-start-howto-vm-java/apache-tomcat-management-app.png)

## <a name="create-an-app"></a>アプリを作成する

Tomcat にデプロイする WAR を作成する必要があります。 ご自分の環境を確認するだけの場合は、[Apache Tomcat サイト](https://tomcat.apache.org/tomcat-6.0-doc/appdev/sample/)で WAR の例を参照してください。

Azure 内での Java アプリの開発に関するガイダンスについては、「[Azure で Java アプリを構築、デプロイする](https://azure.microsoft.com/develop/java/)」を参照してください。

## <a name="deploy-and-run-the-app"></a>アプリのデプロイと実行

1. SSH クライアントを使用して VM に接続します。 手順については、[PuTTY を使用する SSH 経由での接続](azure-stack-dev-start-howto-ssh-public-key.md#connect-with-ssh-by-using-putty)に関する記述を参照してください。

1. ご自分のアプリ パッケージでサーバーを更新するには、Tomcat サービスを停止します。

    ```bash  
        sudo systemctl stop tomcat
    ```

1. webapps フォルダーに書き込めるようにするには、Tomcat グループにご使用の FTP ユーザーを追加します。 ご使用の FTP ユーザーとは、Azure Stack Hub 内に VM を作成するときに定義したユーザーのことです。

    ```bash  
        sudo usermod -a -G tomcat <VM-user>
    ```

1. webapps フォルダーをクリアし、新規または更新済みの WAR を読み込むには、FileZilla を使用してご自分の VM に接続します。 手順については、「[FileZilla を使用して SFTP で接続する](azure-stack-dev-start-howto-ssh-public-key.md#connect-with-sftp-with-filezilla)」を参照してください。

    a. *TOMCAT_HOME/webapps* をクリアします。

    b. WAR を *TOMCAT_HOME/webapps* ( */opt/tomcat/webapps/* など) に追加します。

1.  Tomcat は自動的に、アプリケーションを展開して配置します。 これは、先ほど作成した DNS 名を使用して確認できます。 次に例を示します。

    ```HTTP  
       http://yourmachine.local.cloudapp.azurestack.external:8080/sample
    ```
    
## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub 向けの開発](azure-stack-dev-start.md)方法について、さらに学習する。
- [IaaS としての Azure Stack Hub 向けの一般的なデプロイ](azure-stack-dev-start-deploy-app.md)を確認する。
- Java プログラミング言語の詳細および Java の他のリソースについては、[Java.com](https://www.java.com) を参照してください。
