---
title: コンテナー レジストリを追加する - Azure Stack Hub | Microsoft Docs
titleSuffix: Azure Stack
description: Azure Stack Hub Marketplace (ラグド) にコンテナー レジストリを追加する方法について説明します。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 2/1/2021
ms.author: mabrigg
ms.reviewer: chasat
ms.lastreviewed: 12/17/2019
ms.openlocfilehash: b94ae36daf6cd0ed69ded630a1cc1a7c96be80b2
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99247525"
---
# <a name="add-a-container-registry-to-azure-stack-hub-ruggedized"></a>Azure Stack Hub (ラグド) にコンテナー レジストリを追加する

Azure Stack Hub Marketplace には、自分のユーザーが各自のコンテナー レジストリをデプロイして管理できるようコンテナー レジストリを追加することができます。 このソリューション テンプレートを使用すると、AKS 基本 Ubuntu 16.04-LTS イメージ上で動作するオープンソースの Docker コンテナー レジストリがユーザー サブスクリプションにインストールされ、構成されます。 このテンプレートでは、接続型のデプロイと非接続型 (エアギャップ) のデプロイの両方がサポートされ、Azure Active Directory (AAD) と Active Directory フェデレーション サービス (AD FS) でデプロイされた Azure Stack Hubs の両方がサポートされます。

## <a name="get-the-marketplace-item"></a>Marketplace アイテムを入手する

Marketplace の **コンテナー レジストリ テンプレート** アイテムは、GitHub リポジトリ (https://github.com/msazurestackworkloads/azurestack-gallery/releases/download/registry-v1.0.2/Microsoft.AzureStackContainerRegistry.1.0.2.azpkg ) にあります。 一部のサブスクリプションでは、この Marketplace アイテムをポータルから入手できます。

また、Microsoft.AzureStackDockerContainerRegistry.1.0.2.azpkg を使用して、このアイテムを Marketplace に追加 (サイド ローディング) することもできます。 この記事のスクリプトを入手するには、 https://github.com/msazurestackworkloads/azurestack-gallery/archive/master.zip から (zip) パッケージとして Git リポジトリをダウンロードし、ファイルを抽出します。 スクリプトは、`azurestack-gallery-master\registry\scripts` フォルダーにあります。

## <a name="prerequisites"></a>前提条件

Marketplace アイテムであるコンテナー レジストリを Azure Stack Hub に追加する前に、次のアイテムを用意しておく必要があります。

| Item | Type | 詳細 |
| --- | --- | --- |
| Azure Stack Hub PowerShell モジュール (Azs.Gallery.Admin) | PowerShell モジュール | コンテナー レジストリ テンプレート ギャラリー アイテムをサイド ローディングする場合にのみ必要になります。Azure Stack Hub PowerShell モジュールは、ギャラリー アイテムを追加したり削除したりする際に使用します。<br>[Azure Stack PowerShell モジュールのインストール](../../operator/azure-stack-powershell-install.md) |
| コンテナー レジストリ テンプレート | Marketplace の項目 | Azure Stack Hub ユーザーとしてコンテナー レジストリをデプロイするには、Marketplace アイテムであるコンテナー レジストリ テンプレートをサブスクリプションから使用できることが必要です。または、これを Azure Stack Hub Marketplace に手動で追加 (サイド ローディング) する必要があります。 サイド ローディングする場合は、[GitHub リポジトリ](https://github.com/msazurestackworkloads/azurestack-gallery/releases/tag/registry-v1.0.1)の `readme.md` の手順に従って、パッケージをサイド ローディングしてください。 |
| AKS 基本 Ubuntu 16.04-LTS イメージ (2019 年 9 月以降にリリースされたバージョン) | Marketplace の項目 | Azure Stack Hub ユーザーがコンテナー レジストリをデプロイするには、Marketplace で AKS 基本イメージを入手できるようにする必要があります。 Docker コンテナー レジストリのバイナリを Ubuntu VM でホストするときに、このイメージがコンテナー レジストリ テンプレートによって使用されます。 |
| Linux カスタム スクリプト拡張機能 2.0 | Marketplace の項目 | Azure Stack Hub ユーザーがコンテナー レジストリをデプロイするには、Marketplace で Linux カスタム スクリプト拡張機能を入手できるようにする必要があります。 コンテナー レジストリ テンプレートのデプロイでは、この拡張機能を使用してレジストリが構成されます。 |
| SSL 証明書 | Certificate | コンテナー レジストリ テンプレートをデプロイするユーザーは、レジストリ サービスの SSL 暗号化を構成する際に使用された PFX 証明書を提示する必要があります。 このスクリプトを使用する場合は、管理者特権のプロンプトから PowerShell セッションを実行する必要があります。 これは DVM または HLH で実行しないでください。<br>パブリックまたはプライベート (エンタープライズ) 証明書を使用した Azure Stack Hub 用の PKI 証明書の要件に関する一般的なガイダンスについては、「[Azure Stack Hub 公開キー インフラストラクチャ (PKI) 証明書の要件](../../operator/azure-stack-pki-certs.md)」を参照してください。<br>エンドポイントにカスタム ドメインまたは DNS エントリを使用する場合を除き、証明書の FQDN は、`<vmname>.<location>.cloudapp.<fqdn>` というパターンに従っている必要があります。 その名前は、英字で始まること、2 文字以上の英字を含んでいること、小文字のみを使用すること、長さが 3 文字以上であることが必要です。 |
| サービス プリンシパル (SPN) | アプリケーションの登録 | コンテナー レジストリをデプロイして構成するには、アプリケーションの登録 (サービス プリンシパル (SPN) とも呼ばれます) を作成する必要があります。 VM とレジストリの構成時に、Marketplace アイテムのデプロイ前に作成された Microsoft Azure Key Vault およびストレージ アカウント リソースにアクセスする目的で、この SPN は使用されます。<br>この SPN は、Azure Stack Hub のユーザー ポータルでログインしているテナント内の AAD 内に作成する必要があります。 AD FS を使用している場合は、ローカル ディレクトリ内に作成されます。<br>AAD と AD FS 両方の認証方法に使用する SPN を作成する方法の詳細については、[こちらのガイダンス](../../operator/azure-stack-create-service-principals.md)を参照してください。<br> **重要**:更新プログラムをデプロイするためには、SPN アプリ ID とシークレットを保存する必要があります。<br> |
| レジストリのユーザー名とパスワード | 資格情報 | オープンソースの Docker コンテナー レジストリは、基本認証が有効な状態でデプロイされ、構成されます。 Docker のコマンドを使用してレジストリにアクセスし、イメージをプッシュしたりプルしたりするには、ユーザー名とパスワードが必要です。 ユーザー名とパスワードは、Key Vault ストアに安全に格納されています。<br>**重要**:レジストリにサインインしてイメージをプッシュまたはプルするには、レジストリのユーザー名とパスワードを保存する必要があります。 |
| SSH の公開キーと秘密キー | 資格情報 | VM のデプロイまたはランタイムの問題をトラブルシューティングするには、デプロイ用の SSH 公開キーが用意されていることと、それに対応する秘密キーがアクセスできる状態になっていることが必要です。 秘密キーと公開キーの組は、OpenSSH 形式 (ssh-keygen) を使用して生成することをお勧めします。ログを収集するための診断スクリプトには、この形式が必要となります。<br>**重要**:トラブルシューティングのために、デプロイされた VM にアクセスするには、公開キーと秘密キーにアクセスできる必要があります。 |
| 管理ポータルとユーザー ポータル、および管理エンドポイントへのアクセス | 接続 | このガイドでは、Azure Stack Hub システムに接続するシステムからレジストリのデプロイと構成を実行していることを前提としています。 |

Marketplace アイテムをデプロイするために必要なその他の入力は、`Pre-reqs` スクリプトによって作成されます。

## <a name="installation-steps"></a>インストール手順

コンテナー レジストリ テンプレートをインストールするには、デプロイ前にいくつかのリソースを作成しておく必要があります。

1. PowerShell を使用して Azure Stack Hub にユーザーとして接続し、コマンドレット `Select-AzureRmSubscription –Subscription <subscription guid>` を使用してサブスクリプションを選択します。 ユーザーとして Azure Stack Hub PowerShell に接続する方法の詳細については、[ユーザーとして PowerShell を使用して Azure Stack に接続する](../../user/azure-stack-powershell-configure-user.md)方法に関するページを参照してください。

2. `Import-Modules .\\pre-reqs.ps1` を実行して、`pre-reqs.ps1` スクリプト内のモジュールをインポートします。 このスクリプトによって、リソース グループ、ストレージ アカウント、BLOB コンテナー、Key Vault ストアが作成され、SPN にアクセス権限が割り当てられて、レジストリの証明書とユーザー名とパスワードが Key Vault ストアにコピーされます。

3. 管理者特権のプロンプトから次のコマンドレットを実行します。その際、パラメーターには、実際の環境の値を使用してください。

    ```powershell  
         Set-ContainerRegistryPrerequisites -Location Shanghai `
         -ServicePrincipalId <spn app id> `
         -ResourceGroupName newregreq1 `
         -StorageAccountName newregsa1 `
         -StorageAccountBlobContainer newregct1 `
         -KeyVaultName newregkv1 `
         -CertificateSecretName containersecret2 `
         -CertificateFilePath C:\crinstall\shanghairegcert.pfx `
         -CertificatePassword <cert password> `
         -RegistryUserName admin `
         -RegistryUserPassword <password> 
    ```

    | パラメーター | 詳細 |
    | --- | --- |
    | $Location | リージョン名と呼ばれることもあります。 |
    | $ResourceGroupName | ストレージ アカウントと Key Vault ストアの作成先となるリソース グループの名前を指定します。 Marketplace アイテムをデプロイするときは、別のリソース グループを指定することになります。 |
    | $StorageAccountName | コンテナー レジストリ用に作成するストレージ アカウントの名前を指定します。プッシュされたイメージを格納する際に使用されます。 |
    | $StorageAccountBlobContainer | イメージの保存に使用される、作成する BLOB コンテナーの名前を指定します。 |
    | $KeyVaultName | 証明書とユーザー名およびパスワードの値を格納する目的で作成する Key Vault ストアの名前を指定します。 |
    | $CertificateSecretName | PFX 証明書を格納するために Key Vault に作成されるシークレットの名前を指定します。 |
    | $CertificateFilePath | PFX 証明書のパスを指定します。 |
    | $CertificatePassword | PFX 証明書のパスワードを指定します。 |
    | $ServicePrincipalId | SPN の AppID を指定します。 |
    | $RegistryUserName | 基本認証を使用してレジストリ サービスにアクセスするためのユーザー名を指定します。 |
    | $RegistryUserPassword | レジストリ ユーザーのパスワードを指定します。 |

1. スクリプトが完成したら、そのスクリプトの末尾に、テンプレートのデプロイに使用するパラメーターが含まれていることに注目してください。 これらの値をコピーして貼り付ける際、その値が折り返されるとスペースが入る場合があります。

    ```powershell  
    ----------------------------------------------------------------
    PFX KeyVaultResourceId       : /subscriptions/<subcription id>/resourceGroups/newr
    egreg1/providers/Microsoft.KeyVault/vaults/newregkv1
    PFX KeyVaultSecretUrl        : https://newregkv1.vault.shanghai.azurestack.corp.microsoft.com:443/secr
    ets/containersecret1/37cc2f7ea1c44ad7b930e2c237a14949
    PFX Certificate Thumbprint   : 64BD5F3BC41DCBC6495998900ED322D8110DE25E
    ----------------------------------------------------------------
    StorageAccountResourceId     : /subscriptions/<subcription id>/resourcegroups/newr
    egreg1/providers/Microsoft.Storage/storageAccounts/newregsa1
    Blob Container               : newregct1
    ----------------------------------------------------------------
    
    Skus : aks-ubuntu-1604-201909
    
    
    Skus : aks-ubuntu-1604-201910
    
    ---------------------------------------------------------------- 
    
    ```

1. Azure Stack Hub ユーザー ポータルを開きます。

2. **[作成]**  >  **[コンピューティング]**  >  **[Container Registry Template]\(コンテナー レジストリ テンプレート\)** を選択します。

    ![[コンピューティング] が選択され、[Container Registry Template]\(コンテナー レジストリ テンプレート\) が選択して表示されている [ダッシュボード] > [新規] ページを示すスクリーンショット。](./media/container-registry-template-install-tzl/image1.png)

3. コンテナー レジストリ テンプレートをデプロイするサブスクリプション、リソース グループ、場所を選択します。

    ![[Create Container Registry Template]\(コンテナー レジストリ テンプレートの作成\) - [基本] ページを示すスクリーンショット。](./media/container-registry-template-install-tzl/image2.png)

4. 仮想マシンの構成の詳細を入力します。 このイメージ SKU は、既定で **aks-ubuntu-1604-201909** になりますが、`Set-ContainerRegistryPrerequisites` 関数の出力には、デプロイに使用可能な SKU のリストが含まれます。 複数の SKU が存在する場合は、デプロイに使用する最新の SKU を選択してください。

    ![[Create Container Registry Template]\(コンテナー レジストリ テンプレートの作成\) - [仮想マシンの構成] ページを示すスクリーンショット。](./media/container-registry-template-install-tzl/image3.png)

    | パラメーター | 詳細 |
    | --- | --- |
    | ユーザー名 | VM にログインするためのユーザー名を指定します。 |
    | SSH 公開キー | SSH プロトコルを使用した VM での認証に使用する SSH 公開キーを指定します。 |
    | サイズ | デプロイする VM のサイズを選択します。 |
    | パブリック IP アドレス | この VM の IP アドレス (動的または静的) の名前と種類を指定します。 ドメイン名は無効です。 使用できる文字は小文字、数字、およびハイフンのみです。 先頭の文字は英字にする必要があります。 末尾の文字は、文字または数字にする必要があります。 値は 3 から 63 文字の範囲でなければなりません。  |
    | ドメイン名のラベル | レジストリの DNS プレフィックスを指定します。 この FQDN 全体が、そのレジストリに対して作成された PFX 証明書の CN 値と一致する必要があります。 |
    | レプリカ | 開始するコンテナー レプリカの数を指定します。 |
    | イメージ SKU | デプロイに使用するイメージ SKU を指定します。 AKS 基本イメージに使用できる SKU は、`Set-ContainerRegistryPrerequisites` スクリプトによって一覧表示されます。 |
    | SPN クライアント ID | SPN アプリ ID を指定します。 |
    | SPN パスワードとパスワードの確認 | SPN アプリ ID のシークレットを指定します。 |

1. ストレージと Key Vault の構成を入力します。

    ![[Create Container Registry Template]\(コンテナー レジストリ テンプレートの作成\) - [ストレージと Key Vault の構成] ページを示すスクリーンショット。](./media/container-registry-template-install-tzl/image4.png)

    | パラメーター | 詳細 |
    | --- | --- |
    | 既存の拡張ストレージ アカウントのリソース ID | `pre-reqs` スクリプトによって返されたストレージ アカウントのリソース ID を指定します。 |
    | 既存のバックエンド BLOB コンテナー | pre-reqs スクリプトの出力に一覧表示されている BLOB コンテナー名を指定します。 |
    | PFX 証明書の Key Vault のリソース ID | `pre-reqs` スクリプトによって返された Key Vault のリソース ID を指定します。 |
    | PFX 証明書の Key Vault のシークレット URL | pre-reqs スクリプトによって返された証明書の URL を指定します。 |
    | PFX 証明書の拇印 | pre-reqs スクリプトによって返された証明書の拇印を指定します。 |

1. すべての値が指定されて、ソリューション テンプレートのデプロイが開始されると、VM がレジストリ サービスをデプロイして構成するまでに 10 分から 15 分かかります。

    ![コンテナー レジストリ テンプレート](./media/container-registry-template-install-tzl/image5.png)

2. レジストリをテストするために、レジストリの URL にアクセスできるマシンまたは VM から Docker の CLI インスタンスを開きます。

    > [!Note]
    >  自己署名証明書、またはレジストリへのアクセスに使用する VM によって認識されない証明書を使用した場合は、その証明書を VM にインストールして、Docker を再起動する必要があります。

## <a name="pushing-and-pulling-images-from-container-registry"></a>コンテナー レジストリでイメージをプッシュおよびプルする

1. `docker login –u \<username> -p \<password>` を使用してサインインします。
2. 既知のレジストリからイメージをプルします。
3. 新しくデプロイされた Docker コンテナー レジストリがターゲットとなるよう、そのイメージにタグを付けます。
4. 新しいターゲット レジストリにイメージをプッシュします。

次に例を示します。

```powershell  
PS C:\> docker pull mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0
1.0: Pulling from azureiotedge-simulated-temperature-sensor
5d20c808ce19: Already exists
656de8e592c3: Already exists
1e1868d1f676: Already exists
f3fb1b0d620f: Pulling fs layer
26224c4fc11a: Pulling fs layer
c459a69d65b2: Pulling fs layer
c459a69d65b2: Verifying Checksum
c459a69d65b2: Download complete
f3fb1b0d620f: Download complete
f3fb1b0d620f: Pull complete
26224c4fc11a: Verifying Checksum
26224c4fc11a: Pull complete
c459a69d65b2: Pull complete
Digest: sha256:dd64ff0918459184574e840ee97aa9f1bacd40aa37c972984ea10f0ecd719d5f
Status: Downloaded newer image for mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0
mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0

PS C:\> docker tag mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0    myreg.orlando.cloudapp.azurestack.corp.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0

PS C:\> docker login -u admin -p admin myreg.orlando.cloudapp.azurestack.corp.microsoft.com
docker : WARNING! Using --password via the CLI is insecure. Use --password-stdin.
At line:1 char:1
+ docker login -u admin -p admin myreg.orlando.cloudapp.azurestack.corp ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (WARNING! Using ...password-stdin.:String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
 
Login Succeeded

PS C:\> docker push myreg.orlando.cloudapp.azurestack.corp.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0
The push refers to repository [myreg.orlando.cloudapp.azurestack.corp.microsoft.com/azureiotedge-simulated-temperature-sensor]
d377c212e567: Preparing
0481685b758f: Preparing
15474c03a0b6: Preparing
8cdec5be5964: Preparing
79116d3fb0bf: Preparing
3fc64803ca2d: Preparing
3fc64803ca2d: Waiting
79116d3fb0bf: Mounted from azureiotedge-agent
8cdec5be5964: Mounted from azureiotedge-agent
15474c03a0b6: Pushed
d377c212e567: Pushed
3fc64803ca2d: Mounted from azureiotedge-agent
0481685b758f: Pushed
1.0: digest: sha256:f5fbc4a5c6806e12cafe1c363fea2b6cbd98a211b8153c5b19aca1386bfa6ecb size: 1576 

```

## <a name="known-issues"></a>既知の問題

このテンプレートでデプロイされる Docker Container Registry サービスのバージョンは 2.7 です。 このバージョンには、Windows コンテナー イメージのプッシュとプルを妨げる既知の問題があります。 この問題は、GitHub 項目 [https://github.com/docker/distribution-library-image/issues/89](https://github.com/docker/distribution-library-image/issues/89) で追跡されています。

## <a name="next-steps"></a>次のステップ

[Azure Stack Marketplace の概要](../../operator/azure-stack-marketplace.md)
