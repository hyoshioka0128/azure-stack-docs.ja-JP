---
title: IoT Edge デバイスを Azure Stack Hub の IoT Hub に接続する
description: IoT Edge デバイスを Azure Stack Hub の IoT Hub に接続する方法について説明します。
author: BryanLa
ms.author: bryanla
ms.service: azure-stack
ms.topic: how-to
ms.date: 11/20/2020
ms.reviewer: dmolokanov
ms.lastreviewed: 11/20/2020
ms.openlocfilehash: 768952db6d93930f710dd80ea10e23ae6d8297a5
ms.sourcegitcommit: 0765de47f4a73e09192d34739e40c750b6e7abaf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/27/2021
ms.locfileid: "98920852"
---
# <a name="connect-an-iot-edge-device-to-iot-hub-on-azure-stack-hub"></a>IoT Edge デバイスを Azure Stack Hub の IoT Hub に接続する

[!INCLUDE [preview-banner](../includes/iot-hub-preview.md)]

この記事では、Azure Stack Hub で実行されている IoT Hub に仮想 IoT Edge デバイスを接続する方法について説明します。 同じ一般的なプロセスを使用して、物理デバイスを IoT Hub に接続することができます。

## <a name="prerequisites"></a>前提条件

先に進む前に、次の前提条件を満たしておく必要があります。

- 少なくとも ["共同作成者" アクセス許可](../user/azure-stack-manage-permissions.md)で [Azure Stack Hub ユーザー ポータル](../user/azure-stack-use-portal.md)にアクセスできるアカウントが必要です。

- 管理者と協力して、[Azure Stack Hub リソース プロバイダーに IoT Hub をインストール](../operator/iot-hub-rp-overview.md)します。 インストールの手順には、IoT Hub サービスを含むプランの作成も含まれます。 

- プランが利用できるようになったら、管理者は Azure Stack Hub サブスクリプションを作成または更新して IoT Hub を含めることができます。 [新しいオファーをサブスクライブして、独自のサブスクリプションを作成する](azure-stack-subscribe-services.md)こともできます。

- 公開キー暗号化 (PKI) の基本を理解しておくと役に立ちます。 具体的には、証明機関 (CA) と X509 証明書を使用して信頼チェーンを構築する方法です。 この信頼により、IoT Hub サービスや IoT Edge デバイスなどのネットワーク ノードは、相互に安全に認証と通信を行うことができます。 

## <a name="overview"></a>概要

ここでは、IoT Edge デバイスを Azure Stack Hub 上の IoT Hub に接続するために実行する手順の概要について説明します。

1. Azure Stack Hub インスタンスに IoT Hub リソースと Linux VM リソースを作成します。 Linux VM は、仮想 IoT Edge デバイスとして機能します。 
2. IoT Edge デバイスに必要な証明書を構成します。 必要なステップの数は、お使いの IoT Hub ルート CA 証明書を発行した CA の種類によって異なります。
3. IoT Edge デバイスに必要なソフトウェアとサービスを構成します。
4. IoT Edge デバイスをテストして、正常に動作し、IoT Hub と通信していることを確認します。

各セクションでスクリプトを実行する前に、お使いの環境の構成に合わせて、スクリプトのプレースホルダーを更新してください。 後のセクションで必要になるため、使用した値を記録しておきます。

| プレースホルダー | 例 | 説明 |
|-------------|---------|-------------|
| `<DEVICE-CA-CERT-NAME>` | `edged1cert`| IoT Edge デバイスの CA 証明書に指定した名前。 |
| `<IOT-HUB-HOSTNAME>`| `test-hub-1.mgmtiothub.region.mydomain.com`| IoT Hub のホスト名。 |
| `<IOT-HUB-ROOT-CA>`| `root.cer`| エクスポートした IoT Hub ルート CA に対して指定したファイル名 (プライベート CA を使用している場合)。 |
| `<DATA-DIR>`| `/home/edgeadmin/edged1`| Linux VM 上に作成したデータ ディレクトリへの完全なパス。 |
| `<USER-ACCOUNT>`| `edgeadmin`| Linux VM にサインインするために使用するアカウント。 |

## <a name="create-azure-stack-hub-resources"></a>Azure Stack Hub リソースを作成する

最初に、IoT Hub や Linux VM など、必要な Azure Stack Hub リソースを作成します。 これらのリソースを作成するには、[Azure Stack Hub ユーザー ポータル](../user/azure-stack-use-portal.md)を使用します。

### <a name="create-an-iot-hub-resource"></a>IoT Hub リソースを作成する

1. Azure Stack Hub インスタンスにアクセスできるコンピューターから、[Azure Stack Hub ユーザー ポータル](../user/azure-stack-use-portal.md)にサインインします。
2. IoT Hub リソースをまだ作成していない場合は、「**Azure portal を使用して IoT Hub を作成する**」の「[IoT Hub の作成](/azure/iot-hub/iot-hub-create-through-portal#create-an-iot-hub)」の手順に従って作成します。 

   > [!IMPORTANT]
   > その記事の手順に従うときは、Azure portal ではなく、必ず [Azure Stack Hub ユーザー ポータル](../user/azure-stack-use-portal.md)を使用してください。 また、Azure Stack Hub では利用できない Azure の機能があるため、一部のスクリーンショットと手順は若干異なる場合があることに注意してください。 

### <a name="deploy-and-configure-a-linux-vm"></a>Linux VM をデプロイして構成する 

このセクションでは、仮想 IoT Edge デバイスとして機能する新しい Linux VM をデプロイします。

1. 「[Azure Stack Hub ポータルを使用して Linux サーバー VM を作成する](azure-stack-quick-linux-portal.md)」クイック スタートを使用して、Azure Stack Hub インスタンスに Linux VM をインストールします。 **[設定]** ページの **[パブリック受信ポートを選択]** プロパティで、ポート **SSH (22)** を必ず有効にします。 

   > [!NOTE]
   > NGINX Web サーバーは必要ないので、 **[Connect to the VM]\(VMに接続する\)** までの最初の 5 つのセクションだけを完了する必要があります。 

2. VM を作成し、PuTTY を使用してそれに接続したら、ユーザー アカウントのホーム ディレクトリのデータ サブディレクトリを作成します。次に例を示します。

   ```bash
   mkdir edged1
   ```

3. 次のいずれかの方法を使用して、証明書生成ツールを設定します。 いずれの場合も、IoT Edge GitHub リポジトリからファイルをダウンロードします。これは、後でデバイス証明書を生成するために必要です。

   **cURL スクリプトを使用する:**
   ```bash
   # Navigate to the edge device data directory
   cd <DATA-DIR>
   
   # Download certGen.sh script file, and openssl_root_ca.cnf config file
   curl -Lo certGen.sh https://raw.githubusercontent.com/Azure/iotedge/master/tools/CACertificates/certGen.sh
   curl -Lo openssl_root_ca.cnf https://raw.githubusercontent.com/Azure/iotedge/master/tools/CACertificates/openssl_root_ca.cnf
   ```
   **Git を使用してリポジトリをクローンする:**
   
   ```bash
   # Navigate to the home directory
   cd /home/<USER-ACCOUNT>
   
   # Clone the iotedge repo, which contains the certGen.sh script file and openssl_root_ca.cnf config file
   git clone https://github.com/Azure/iotedge.git
   
   # Navigate to the edge device data directory
   cd <DATA-DIR>
   
   # Copy certGen.sh and openssl_root_ca.cnf into your working directory
   cp /home/<USER-ACCOUNT>/iotedge/tools/CACertificates/*.cnf .
   cp /home/<USER-ACCOUNT>/iotedge/tools/CACertificates/certGen.sh .
   ```

## <a name="configure-iot-edge-device-certificates"></a>IoT Edge デバイス証明書を構成する

このセクションでは、仮想 IoT Edge デバイスで必要とされる VM 証明書の構成を行います。 

IoT Hub サービスと IoT Edge デバイスは、X509 証明書を使用してセキュリティで保護されます。 どちらも、同じ CA によって発行されたルート CA 証明書を使用する必要があります。 IoT Hub によって使用されているルート CA の種類に基づいて、下の適切なタブを選択し、証明書の構成を完了します。

# <a name="public-ca"></a>[パブリック CA](#tab/public-ca)

[!INCLUDE [common-cert-config](../includes/iot-hub-connect-an-iot-edge-device-cert-common.md)]

# <a name="private-ca"></a>[プライベート CA](#tab/private-ca)

> [!IMPORTANT]
> プライベート CA を使用している場合は、IoT Hub のルート CA 証明書を IoT Edge デバイスに転送するために、追加の手順が必要になります。 次の手順は、証明書の生成に独自のプライベート CA を使用している場合に "***のみ** _" 実行してください。 お使いの IoT Hub ルート CA 証明書がパブリック CA によって発行された場合は、前の "_ *パブリック CA**" タブを選択します。 

### <a name="export-the-root-ca-certificate-from-your-iot-hub"></a>IoT Hub からルート CA 証明書をエクスポートする

Azure Stack Hub にアクセスできるマシンを使用して、IoT Hub のルート CA 証明書を PEM 形式でエクスポートします。 次の例では、[Microsoft Edge](https://www.microsoft.com/edge) または [Google Chrome](https://www.google.com/chrome/index.html) ブラウザーを使用して証明書をエクスポートする方法を示します。 

   1. IoT Hub の **[概要]** ページで、 **[ホスト名]** プロパティの右側にある **[コピー]** ボタンを使用して、IoT Hub のホスト名をクリップボードにコピーします。  

      [![IoT Hub のホスト名](media\iot-hub-connect-an-iot-edge-device\copy-iot-hub-host-name.png)](media\iot-hub-connect-an-iot-edge-device\copy-iot-hub-host-name.png#lightbox)
   1. Microsoft Edge または Google Chrome ブラウザーで新しいタブを開き、「`https://`」と入力し、前のステップでコピーした IoT Hub のホスト名を貼り付けて、**Enter** キーを押します。 

   1. エラー メッセージが返されたら、アドレス バーの左側にあるロック アイコンを選択し、 **[証明書]** を選択します。
      [![証明書のセキュリティで保護された接続](media\iot-hub-connect-an-iot-edge-device\iot-hub-root-ca-connection.png)](media\iot-hub-connect-an-iot-edge-device\iot-hub-root-ca-connection.png#lightbox)

   1. SSL/TLS 証明書が表示されたら、 **[証明のパス]** タブを選択します。次に、パスで先頭の証明書を選択し、 **[証明書の表示]** ボタンを選択します。  
      [![SSL 証明書の証明書のセキュリティで保護された接続](media\iot-hub-connect-an-iot-edge-device\iot-hub-root-ca-certificate-ssl.png)](media\iot-hub-connect-an-iot-edge-device\iot-hub-root-ca-certificate-ssl.png#lightbox) 

   1. ルート CA 証明書が表示されたら、 **[詳細]** タブを選択し、 **[ファイルにコピー...]** ボタンを選択します。
      [![ルート CA 証明書の証明書のセキュリティで保護された接続](media\iot-hub-connect-an-iot-edge-device\iot-hub-root-ca-certificate-ssl-root-ca.png)](media\iot-hub-connect-an-iot-edge-device\iot-hub-root-ca-certificate-ssl-root-ca.png#lightbox) 

   1. **[証明書のエクスポート ウィザード]** が表示されたら、証明書を **Base-64 エンコード X.509 形式** のファイルにエクスポートします (例: **root.cer**)。 次のセクションでこのファイルを使用するので、Linux VM からアクセスしたり、Linux VM にコピーしたりできる場所にエクスポートします。

   1. エクスポートが正常に完了したら、ブラウザーのタブを閉じてかまいません。

### <a name="install-the-iot-hub-root-ca-certificate-on-the-iot-edge-device"></a>IoT Edge デバイスに IoT Hub ルート CA 証明書をインストールする

次に、前のセクションでエクスポートした IoT Hub ルート CA 証明書を、IoT Edge デバイスの信頼されたストアにインストールします。 

1. 前のセクションでエクスポートした IoT Hub ルート CA ファイルを、IoT Edge デバイスに転送します。 IoT Edge デバイスとして Linux VM を使用しているため、Linux VM にコピーする必要があります。 環境によっては、PSCP コマンドで PuTTY を使用するか、WinSCP プログラムを使用して、Linux VM にファイルをコピーすることを検討します。
2. IoT Hub ルート CA ファイルを保存した Linux VM PuTTY セッションから、次のスクリプトを実行します。 スクリプトにより、TLS 接続が成功したことが確認され、IoT Edge デバイスの信頼されたストアにルート CA がインストールされます。

   ```bash
   # Verify connection failed first
   # Expected response: Verify return code: 2 (unable to get issuer certificate)
   openssl s_client -connect <IOT-HUB-HOSTNAME>:443
   
   # Verify connection succeeded when root CA provided
   # Expected response: Verify return code: 0 (ok)
   openssl s_client -connect <IOT-HUB-HOSTNAME>:443 -CAfile <IOT-HUB-ROOT-CA>
   
   # Install root CA in the trusted store on Edge device
   sudo cp <IOT-HUB-ROOT-CA> /usr/local/share/ca-certificates/
   sudo update-ca-certificates
   
   # Verify connection succeeded even when no root CA provided
   # Expected response: Verify return code: 0 (ok)
   openssl s_client -connect <IOT-HUB-HOSTNAME>:443
   ```

[!INCLUDE [common-cert-config](../includes/iot-hub-connect-an-iot-edge-device-cert-common.md)]

### <a name="append-the-iot-hub-root-ca-to-the-device-root-ca"></a>IoT Hub のルート CA をデバイスのルート CA に追加する

次に、エクスポートして IoT Edge デバイスにコピーした IoT Hub のルート CA を、前に生成したデバイスのルート CA に追加します。

```bash
# Navigate to home directory
cd /home/<USER-ACCOUNT>

# Append IoT Hub root CA to the device root CA
cat <IOT-HUB-ROOT-CA> >> certs/azure-iot-test-only.root.ca.cert.pem
```
-----

## <a name="configure-iot-edge-device-software-and-services"></a>IoT Edge デバイスのソフトウェアとサービスを構成する

このセクションでは、仮想 IoT Edge デバイスで必要とされる IoT Hub と VM の構成を行います。

### <a name="install-iot-edge-and-container-runtimes-on-the-vm"></a>IoT Edge とコンテナーのランタイムを VM にインストールする

1. VM PuTTY セッションに戻り、次のスクリプトを実行して、Microsoft のキーとソフトウェア リポジトリ フィードを登録します。

   ```bash
   # Navigate to the home directory
   cd /home/<USER-ACCOUNT>

   # Install the repository configuration. 
   curl https://packages.microsoft.com/config/ubuntu/16.04/multiarch/prod.list > ./microsoft-prod.list
   
   # Copy the generated list.
   sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/
   
   # Install Microsoft GPG public key.
   curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
   sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
   ```

2. コンテナー ランタイムをインストールします。

   ```bash
   sudo apt-get update
   sudo apt-get install moby-engine moby-cli
   ```

3. Azure IoT Edge セキュリティ デーモンをインストールします。

   ```bash
   sudo apt-get update
   sudo apt-get install iotedge
   ```

> [!NOTE]
> **IMPORTANT: Please update the configuration file (重要: 構成ファイルを更新してください)** という通知は、後のセクションで更新するので、ここでは無視して構いません。

### <a name="create-an-iot-edge-device-in-iot-hub"></a>IoT Hub で IoT Edge デバイスを作成する

1. [Azure Stack Hub ユーザー ポータル](../user/azure-stack-use-portal.md)に戻り、**IoT Hub** サービスに移動します。

2. 自分の IoT Hub リソースを選択し、 **[IoT Edge]** ページを選択して、 **[IoT Edge デバイスを追加する]** を選択します。

   [![IoT Hub リソース](media\iot-hub-connect-an-iot-edge-device\add-an-iot-edge-device.png)](media\iot-hub-connect-an-iot-edge-device\add-an-iot-edge-device.png#lightbox)

3. **[デバイスの作成]** ページで、 **[デバイス ID]** を入力します (例: "edged1")。
4. 終わったら、 **[保存]** を選択します。

   [![IoT Edge - デバイスを作成する](media\iot-hub-connect-an-iot-edge-device\create-iot-edge-device.png)](media\iot-hub-connect-an-iot-edge-device\create-iot-edge-device.png#lightbox)

5. ポータルが **[IoT Edge]** ページに戻ったら、デバイスの一覧から新しいデバイスを選択します。

   [![IoT Edge - デバイスを表示する](media\iot-hub-connect-an-iot-edge-device\view-iot-edge-devices.png)](media\iot-hub-connect-an-iot-edge-device\view-iot-edge-devices.png#lightbox)

6. デバイスの詳細ページで、 **[プライマリ接続文字列]** の右側にある **[コピー]** ボタンを使用して、文字列をクリップボードにコピーします。 この接続文字列は次のセクションで使用します。

   [![IoT Edge - デバイスの詳細](media\iot-hub-connect-an-iot-edge-device\iot-edge-device-details.png)](media\iot-hub-connect-an-iot-edge-device\iot-edge-device-details.png#lightbox)

### <a name="configure-the-virtual-iot-edge-device-on-the-vm"></a>VM で仮想 IoT Edge デバイスを構成する

1. VM PuTTY セッションに戻ります。 Bash を使用し、仮想 IoT Edge デバイスの Nano で構成ファイルを開きます。

   ```bash
   sudo nano /etc/iotedge/config.yaml
   ```

2. **Provisioning mode and settings** セクションのコメント **# Manual provisioning configuration using a connection string** の下で、`provisioning` プロパティを見つけます。 `<ADD DEVICE CONNECTION STRING HERE>` プレースホルダーを前のセクションでクリップボードにコピーした接続文字列に置き換えて、IoT Edge デバイスの接続文字列を更新します。

   > [!NOTE]
   > クリップボードの内容を Nano に貼り付けるには、**Shift** キーを押しながらマウスの右ボタンをクリックします。 または、**Shift** + **Insert** キーを押します。 貼り付け操作で、カーソルが接続文字列の右端に移動した場合は、**Home** キーを押して左端に戻します。

   ```yaml
   # Manual provisioning configuration using a connection string
   provisioning:
     source: "manual"
     device_connection_string: "<ADD DEVICE CONNECTION STRING HERE>"
   ```

3. **Certificate settings** セクションで `certificates` プロパティを見つけてコメントを解除します。 ファイルの URI をコメント解除し、前に作成した 3 つの証明書へのパスに置き換えます。次に例を示します。

   ```yaml
   certificates:
     device_ca_cert: "<DATA-DIR>/certs/iot-edge-device-ca-<DEVICE-CA-CERT-NAME>-full-chain.cert.pem"
     device_ca_pk: "<DATA-DIR>/private/iot-edge-device-ca-<DEVICE-CA-CERT-NAME>.key.pem"
     trusted_ca_certs: "<DATA-DIR>/certs/azure-iot-test-only.root.ca.cert.pem"
   ```

4. 完了すると、`provisioning` および `certificates` プロパティは次の応答のようになります。

   [![Nano エディター - provisioning プロパティ](media\iot-hub-connect-an-iot-edge-device\nano-edit-config-yml-connection-string.png)](media\iot-hub-connect-an-iot-edge-device\nano-edit-config-yml-connection-string.png#lightbox)

   [![Nano エディター - certificates プロパティ](media\iot-hub-connect-an-iot-edge-device\nano-edit-config-yml-certificates.png)](media\iot-hub-connect-an-iot-edge-device\nano-edit-config-yml-certificates.png#lightbox)


5. **Ctrl** + **X** キー、**Y** キー、**Enter** キーの順に押し、ファイルを保存して閉じます。

6. Bash に戻ったら、デーモンを再起動します。
   
   ```bash
   sudo systemctl restart iotedge
   ```

## <a name="verify-a-successful-installation"></a>インストールの成功を確認する

1. IoT Edge デーモンの状態を確認します。

   ```bash
   systemctl status iotedge
   ```

2. 次のような応答が表示され、IoT Edge サービスが実行中で "アクティブ" になっていることがわかります。 その場合は、ステップ #4 に進むことができます。

   [![正常に実行されている IoT Edge サービス](media\iot-hub-connect-an-iot-edge-device\iot-edge-service-running.png)](media\iot-hub-connect-an-iot-edge-device\iot-edge-service-running.png#lightbox)

3. IoT Edge サービスが失敗した場合:
   - 次の例のような応答が表示される場合は、"失敗" を示します。[![失敗した IoT Edge サービス](media\iot-hub-connect-an-iot-edge-device\iot-edge-service-failed.png)](media\iot-hub-connect-an-iot-edge-device\iot-edge-service-failed.png#lightbox)   
   - 次のようにしてトラブルシューティングを行うことができます。
     - デーモンのログを調べます。

         ```bash
         journalctl -u iotedge --no-pager --no-full
         ```
     - トラブルシューティング ツールを実行して、最も一般的な構成とネットワークのエラーを確認します。 この例では、不適切な形式の `/etc/iotedge/config.yaml` ファイルが検出されています。

         ```bash
         sudo iotedge check
         ```

         [![IoT Edge サービスの確認](media\iot-hub-connect-an-iot-edge-device\iot-edge-service-check.png)](media\iot-hub-connect-an-iot-edge-device\iot-edge-service-check.png#lightbox)

         > [!NOTE]
         > `$edgeHub` システム モジュールはオプションであり、最初の IoT Edge モジュールをデプロイするまで、デバイスにはデプロイされません。 そのため、ホスト接続チェックの間に `$edgeHub` をポートにバインドできないことを示す `iotedge check` エラーが返されることがあります。 デプロイで `$edgeHub` が必要ない場合、このエラーは無視してかまいません。           

   - 上の例のように .YML ファイルの形式が正しくないことがわかったら、次のようにします。
      - 「[VM で仮想 IoT Edge デバイスを構成する](#configure-the-virtual-iot-edge-device-on-the-vm)」の手順を繰り返して、.YML ファイルを修正します。
      - 「[インストールの成功を確認する](#verify-a-successful-installation)」の手順を繰り返します。

4. IoT Edge サービスが正常に開始されたことを確認したら、実行中のモジュールの一覧を表示します。 サービス `edgeAgent` の状態が `running` と表示されるはずです。

   ```bash
   sudo iotedge list
   ```

## <a name="clean-up-resources"></a>リソースをクリーンアップする

もう使わない場合は、[Azure Stack Hub ユーザー ポータル](../user/azure-stack-use-portal.md)に戻り、前に作成した VM と IoT Hub リソースを削除します。

## <a name="next-steps"></a>次のステップ

補足リソースを次に示します。

- [Debian ベースの Linux システムに Azure IoT Edge ランタイムをインストールする](/azure/iot-edge/how-to-install-iot-edge-linux)
- [透過的なゲートウェイとして機能するように IoT Edge デバイスを構成](/azure/iot-edge/how-to-create-transparent-gateway)します。
- [IoT Edge デバイスの機能をテストするためのデモ用の証明書を作成する](/azure/iot-edge/how-to-create-test-certificates)