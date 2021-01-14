---
title: Azure Stack Hub での Python による API バージョンのプロファイルの使用
description: Azure Stack Hub での Python による API バージョンのプロファイルの使用方法について説明します。
author: sethmanheim
ms.topic: article
ms.date: 12/2/2020
ms.author: sethm
ms.reviewer: sijuman
ms.lastreviewed: 12/2/2020
ms.openlocfilehash: 1359ce67220040a326251802b35ad72ec526d09e
ms.sourcegitcommit: 52c934f5eeb5fcd8e8f2ce3380f9f03443d1e445
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/07/2021
ms.locfileid: "97973981"
---
# <a name="use-api-version-profiles-with-python-in-azure-stack-hub"></a>Azure Stack Hub での Python による API バージョンのプロファイルの使用

Python SDK では、Azure Stack Hub やグローバル Azure などの異なるクラウド プラットフォームをターゲットとする API バージョン プロファイルをサポートします。 ハイブリッド クラウド向けのソリューションの作成時に API プロファイルを使用します。

この記事の手順では、Microsoft Azure サブスクリプションが必要です。 お持ちでない場合は、[無料試用版アカウントを入手](https://go.microsoft.com/fwlink/?LinkId=330212)できます。

## <a name="python-and-api-version-profiles"></a>Python と API バージョン プロファイル

Python SDK では、以下の API プロファイルをサポートします。

- **latest**  
    このプロファイルでは、Azure プラットフォーム内にある全サービス プロバイダー向けの最新の API バージョンをターゲットにします。
- **2019-03-01-hybrid**  
    このプロファイルでは、バージョン 1904 以降の Azure Stack Hub プラットフォーム内にある全リソース プロバイダー向けの最新の API バージョンをターゲットにします。
- **2018-03-01-hybrid**  
    このプロファイルでは、Azure Stack Hub プラットフォーム内にある全リソース プロバイダー向けの最も互換性の高い API バージョンをターゲットにします。
- **2017-03-09-profile**  
    このプロファイルでは、Azure Stack Hub がサポートするリソース プロバイダーの最も互換性の高い API バージョンをターゲットにします。

   API プロファイルと Azure Stack Hub の詳細については、「[Azure Stack Hub での API バージョンのプロファイルの管理](azure-stack-version-profiles.md)」を参照してください。

## <a name="install-the-azure-python-sdk"></a>Azure Python SDK をインストールする

1. [Git をインストールします](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)。
2. Python SDK のインストール方法については、[Python 開発者向け Azure](/python/azure/python-sdk-azure-install?view=azure-python&preserve-view=true) に関する記事を参照してください。
3. 使用できない場合は、サブスクリプションを作成し、サブスクリプション ID を保存して後で使用します。 サブスクリプションの作成の詳細については、「[Azure Stack Hub でオファーのサブスクリプションを作成する](../operator/azure-stack-subscribe-plan-provision-vm.md)」を参照してください。
4. サービス プリンシパルを作成し、その ID とシークレットを保存します。 Azure Stack Hub 用のサービス プリンシパルの作成方法については、[Azure Stack Hub へのアクセスのアプリケーションへの提供](../operator/azure-stack-create-service-principals.md)に関するページを参照してください。
5. サブスクリプションでサービス プリンシパルのロールが共同作成者/所有者であることを確認します。 サービス プリンシパルへのロールの割り当て手順については、[Azure Stack Hub へのアクセスのアプリケーションへの提供](../operator/azure-stack-create-service-principals.md)に関するページを参照してください。

## <a name="prerequisites"></a>前提条件

Azure Stack Hub で Python Azure SDK を使用するには、次の値を指定した後、環境変数に値を設定する必要があります。 環境変数を設定するには、次の表の後にある、お使いの特定のオペレーティング システム用の手順を参照してください。

| 値 | 環境変数 | 説明 |
|---------------------------|-----------------------|-------------------------------------------------------------------------------------------------------------------------|
| テナント ID | `AZURE_TENANT_ID` | Azure Stack Hub の[テナント ID](../operator/azure-stack-identity-overview.md)。 |
| クライアント ID | `AZURE_CLIENT_ID` | この記事の前のセクションでサービス プリンシパルが作成されたときに保存した、サービス プリンシパル アプリ ID。 |
| サブスクリプション ID | `AZURE_SUBSCRIPTION_ID` | [サブスクリプション ID](../operator/service-plan-offer-subscription-overview.md#subscriptions) は Azure Stack Hub 内のオファーにアクセスするために使用します。 |
| クライアント シークレット | `AZURE_CLIENT_SECRET` | サービス プリンシパルの作成時に保存した、サービス プリンシパル アプリ シークレット。 |
| Resource Manager エンドポイント | `ARM_ENDPOINT` | 「[Azure Stack Hub Resource Manager エンドポイント](azure-stack-version-profiles-ruby.md#the-azure-stack-hub-resource-manager-endpoint)」の記事を参照してください。 |
| リソースの場所 | `AZURE_RESOURCE_LOCATION` | Azure Stack Hub 環境のリソースの場所。

### <a name="trust-the-azure-stack-hub-ca-root-certificate"></a>Azure Stack Hub の CA ルート証明書を信頼する

ASDK を使用する場合は、リモート マシン上で CA ルート証明書を明示的に信頼する必要があります。 Azure Stack Hub 統合システムで CA ルート証明書を信頼する必要はありません。

#### <a name="windows"></a>Windows

1. お使いのマシン上で Python 証明書ストアの場所を探します。 この場所は、Python をインストールした場所に応じて異なる場合があります。 コマンド プロンプトまたは管理者特権の PowerShell プロンプトを開き、次のコマンドを入力します。

    ```PowerShell  
      python -c "import certifi; print(certifi.where())"
    ```

    証明書ストアの場所をメモします (例: **~/lib/python3.5/site-packages/certifi/cacert.pem**)。 このパスは、オペレーティング システムと、インストールされている Python のバージョンによって異なります。

2. Azure Stack Hub の CA ルート証明書を Python の既存の証明書に追加して信頼します。

    ```powershell
    $pemFile = "<Fully qualified path to the PEM certificate; for ex: C:\Users\user1\Downloads\root.pem>"

    $root = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2
    $root.Import($pemFile)

    Write-Host "Extracting required information from the cert file"
    $md5Hash    = (Get-FileHash -Path $pemFile -Algorithm MD5).Hash.ToLower()
    $sha1Hash   = (Get-FileHash -Path $pemFile -Algorithm SHA1).Hash.ToLower()
    $sha256Hash = (Get-FileHash -Path $pemFile -Algorithm SHA256).Hash.ToLower()

    $issuerEntry  = [string]::Format("# Issuer: {0}", $root.Issuer)
    $subjectEntry = [string]::Format("# Subject: {0}", $root.Subject)
    $labelEntry   = [string]::Format("# Label: {0}", $root.Subject.Split('=')[-1])
    $serialEntry  = [string]::Format("# Serial: {0}", $root.GetSerialNumberString().ToLower())
    $md5Entry     = [string]::Format("# MD5 Fingerprint: {0}", $md5Hash)
    $sha1Entry    = [string]::Format("# SHA1 Fingerprint: {0}", $sha1Hash)
    $sha256Entry  = [string]::Format("# SHA256 Fingerprint: {0}", $sha256Hash)
    $certText = (Get-Content -Path $pemFile -Raw).ToString().Replace("`r`n","`n")

    $rootCertEntry = "`n" + $issuerEntry + "`n" + $subjectEntry + "`n" + $labelEntry + "`n" + `
    $serialEntry + "`n" + $md5Entry + "`n" + $sha1Entry + "`n" + $sha256Entry + "`n" + $certText

    Write-Host "Adding the certificate content to Python Cert store"
    Add-Content "${env:ProgramFiles(x86)}\Python35\Lib\site-packages\certifi\cacert.pem" $rootCertEntry

    Write-Host "Python Cert store was updated to allow the Azure Stack Hub CA root certificate"
    ```

> [!NOTE]  
> 次の「[Python のサンプルコードを実行する](#run-the-python-sample)」セクションで説明されているように、Python SDK で開発するために **virtualenv** を使用する場合は、前述の証明書を仮想環境の証明書ストアに追加する必要があります。 パスは、`..\mytestenv\Lib\site-packages\certifi\cacert.pem` のようになります。

## <a name="python-samples-for-azure-stack-hub"></a>Azure Stack Hub 向けの Python 例

Python SDK を使用した Azure Stack Hub 向けの入手可能なコード サンプルの一部を次に示します。

- [リソースとリソース グループを管理する](https://azure.microsoft.com/resources/samples/hybrid-resourcemanager-python-manage-resources/)
- [ストレージ アカウントを管理する](https://azure.microsoft.com/resources/samples/hybrid-storage-python-manage-storage-account/)
- [仮想マシンを管理する](https://azure.microsoft.com/resources/samples/hybrid-compute-python-manage-vm/)。このサンプルでは、Azure Stack Hub でサポートされている最新の API バージョンをターゲットとする **2019-03-01-hybrid** プロファイルを使用します。

## <a name="manage-virtual-machine-sample"></a>仮想マシンを管理するサンプル

次の Python コード サンプルを使用して、Azure Stack Hub 内で仮想マシン (VM) に対する一般的な管理タスクを行います。 コード サンプルでは、次の操作方法を示しています。

- VM の作成:
  - Linux VM の作成
  - Windows VM の作成
- VM の更新:
  - ドライブの拡張
  - VM にタグを付ける
  - データ ディスクを接続する
  - データ ディスクをデタッチする
- VM の操作:
  - VM の起動
  - VM の停止
  - VM の再起動
- VM の一覧表示
- VM の削除

これらの操作を実行するコードを確認するには、GitHub リポジトリの [Hybrid-Compute-Python-Manage-VM](https://github.com/Azure-Samples/Hybrid-Compute-Python-Manage-VM) にある Python スクリプト **example.py** の **run_example()** 関数を参照してください。

それぞれの操作には､それと分かるコメントと print 関数が付いています｡ これらの例は、この一覧の順序どおりではない場合があります。

## <a name="run-the-python-sample"></a>Python のサンプルコードを実行する

1. まだ [Python をインストールしていない場合はインストールします](https://www.python.org/downloads/)。 このサンプル (と SDK) は､Python 2.7、3.4､3.5､3.6 に対応しています｡

2. Python 開発では、一般的に、仮想環境を使用することをお勧めします。 詳細については、[Python のドキュメント](https://docs.python.org/3/tutorial/venv.html)を参照してください。

3. Python 3 の **venv** モジュールを使用して､仮想環境をインストールして初期化します (Python 2.7 の場合は [virtualenv](https://pypi.python.org/pypi/virtualenv) をインストールする必要があります)。

    ```bash
    python -m venv mytestenv # Might be "python3" or "py -3.6" depending on your Python installation
    cd mytestenv
    source bin/activate      # Linux shell (Bash, ZSH, etc.) only
    ./scripts/activate       # PowerShell only
    ./scripts/activate.bat   # Windows CMD only
    ```

4. リポジトリを複製します。

    ```bash
    git clone https://github.com/Azure-Samples/Hybrid-Compute-Python-Manage-VM.git
    ```

5. **pip** を使用して依存関係をインストールします｡

    ```bash
    cd Hybrid-Compute-Python-Manage-VM
    pip install -r requirements.txt
    ```

6. Azure Stack Hub と連携させる[サービス プリンシパル](../operator/azure-stack-create-service-principals.md)を作成します。 サブスクリプションでサービス プリンシパルのロールが[共同作成者/所有者](../operator/azure-stack-create-service-principals.md#assign-a-role)であることを確認します。

7. 次の変数を設定して､これらの環境変数を現在のシェルにエクスポートします｡

    ```bash
    export AZURE_TENANT_ID={your tenant id}
    export AZURE_CLIENT_ID={your client id}
    export AZURE_CLIENT_SECRET={your client secret}
    export AZURE_SUBSCRIPTION_ID={your subscription id}
    export ARM_ENDPOINT={your AzureStack Resource Manager Endpoint}
    export AZURE_RESOURCE_LOCATION={your AzureStack Resource location}
    ```

8. このサンプルを実行するには、Azure Stack Hub Marketplace に Ubuntu 16.04-LTS と WindowsServer 2012-R2-DataCenter のイメージが存在している必要があります。 これらのイメージは [Azure からダウンロード](../operator/azure-stack-download-azure-marketplace-item.md)するか､[Platform Image Repository](../operator/azure-stack-add-vm-image.md) に追加することができます｡

9. サンプルを実行します。

    ```python
    python example.py
    ```

## <a name="next-steps"></a>次のステップ

- [Azure Python デベロッパー センター](https://azure.microsoft.com/develop/python/)
- [Azure Virtual Machines 関係のドキュメント](https://azure.microsoft.com/services/virtual-machines/)
- [Virtual Machines のラーニング パス](/learn/paths/deploy-a-website-with-azure-virtual-machines/)
