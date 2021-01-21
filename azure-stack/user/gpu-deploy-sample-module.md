---
title: グラフィックス処理装置 (GPU) 対応 IoT モジュールを Azure Stack Hub にデプロイする
description: グラフィックス処理装置 (GPU) 対応 IoT モジュールを Azure Stack Hub にデプロイする方法
author: mattbriggs
ms.author: mabrigg
ms.service: azure-stack
ms.topic: how-to
ms.date: 11/13/2020
ms.reviewer: gara
ms.lastreviewed: 11/13/2020
ms.openlocfilehash: 7a5a29c48533874cf2c859780ab0332b8410811f
ms.sourcegitcommit: 8526f642ef859b0006c3991d966f93608a87288a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/15/2021
ms.locfileid: "98243478"
---
# <a name="deploy-a-gpu-enabled-iot-module-on-azure-stack-hub"></a>GPU 対応 IoT モジュールを Azure Stack Hub デバイスにデプロイする

GPU 対応 Azure Stack Hub を使用すると、プロセッサへの負荷が高いモジュールを、IoT Edge で実行されている Linux デバイスに展開できます。 GPU に最適化された VM サイズは、1 つまたは複数の NVIDIA GPU を備えた、特殊な用途に特化した VM です。 この記事では、GPU に最適化された VM を使用して、コンピューティング処理やグラフィック処理の負荷が高い視覚化ワークロードを実行します。

開始する前に、グローバル Azure と Azure Stack Hub、Azure Container Registry (ACR)、および IoT ハブにアクセスできる Azure Active Directory (Azure AD) サブスクリプションが必要です。

この記事では、次の内容について説明します。
  - GPU 対応の Linux VM をインストールし、正しいドライバーをインストールします。
  - Docker をインストールし、ランタイムで GPU を有効にします。
  - Iot デバイスを IoT Hub に接続し、次のモデルを IoT マーケットプレースからインストールします: `Getting started with GPUs`。
  - Azure IoT エクスプローラーを使用して、ローカル コンピューターからデバイスをインストールして監視します。
  - また、必要に応じて Visual Studio Code の Azure IoT 拡張機能を使用してデバイスをインストールおよび監視します。

## <a name="prerequisites"></a>前提条件

Azure Stack Hub インスタンス、グローバル Azure、およびローカルの開発用コンピューターに、次のリソースが配置されている必要があります。

### <a name="azure-stack-hub-and-azure"></a>Azure Stack Hub と Azure

  - NVIDA GPU が搭載されている Azure Stack Hub 統合システムで Azure Active Directory (Azure AD) を使用するユーザーのサブスクリプション。 IoT Hub では、次のチップが動作します。
    - NCv3
    - NCas_v4

    Azure Stack Hub の GPU の詳細については、[「Azure Stack Hub 上のグラフィックス処理装置 (GPU) VM」](gpu-vms-about.md)を参照してください。
  - グローバル Azure サブスクリプション。 グローバル Azure サブスクリプションをお持ちでない場合は、開始する前に[無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)を作成してください。
- [Azure Container Registry (ACR)](/)。 ACR サインイン サーバー、ユーザー名とパスワードをメモしておきます。
-   グローバル Azure の Free レベルまたは Standard レベルの [IoT Hub](/azure/iot-hub/iot-hub-create-through-portal)。

### <a name="a-development-machine"></a>開発用コンピューター

-   開発設定に応じて、独自のコンピューターまたは仮想マシンを使用できます。 開発用コンピューターでは、入れ子になった仮想化がサポートされている必要があります。 この機能は、この記事で使用されているコンテナー エンジンである Docker を実行するために必要です。

  - 開発用コンピューターには、次のリソースが必要です。
      - [Python 3.x](https://www.python.org/downloads/)
      - Python パッケージをインストールするための [Pip](https://pypi.org/project/pip/)。 これは、Python をインストールする際に同時にインストールされます。 PIP がインストールされている場合は、最新バージョンにアップグレードしておくことをお勧めします。 PIP 自体を使用してアップグレードできます。 次のコマンドを入力します: `pip install --upgrade pip`
      - [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest)
      - [Git](https://git-scm.com/downloads)
      - [Docker](https://docs.docker.com/get-docker/)
      - [Visual Studio Code](https://code.visualstudio.com/)
      - Visual Studio Code 用 [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
      - Visual Studio Code 向け [Python 拡張機能パック](https://marketplace.visualstudio.com/items?itemName=donjayamanne.python-extension-pack)

## <a name="register-an-iot-edge-device"></a>IoT Edge デバイスを登録する

別のデバイスを使用して IoT Edge デバイスをホストします。 別のデバイスを使用することで開発用コンピューターと分離され、IoT Edge デバイスにデプロイ シナリオがより正確に反映されるようになります。 

Linux VM を使用して Azure で IoT Edge デバイスを作成します。

1.  Azure Stack Hub で [N シリーズの Linux サーバー VM](./azure-stack-quick-linux-portal.md) を作成します。 サーバーのコンポーネントをインストールするときに、SSH を使用してサーバーと通信します。 詳細については、「[SSH 公開キーの使用方法](./azure-stack-dev-start-howto-ssh-public-key.md)」を参照してください。

2.  [IoT Edge デバイスの作成と登録](/azure/iot-edge/how-to-register-device)

## <a name="prepare-a-gpu-enabled-vm"></a>GPU 対応 VM の準備

1. [「Linux を実行している N シリーズ VM に NVIDIA GPU ドライバーをインストールする」](/azure/virtual-machines/linux/n-series-driver-setup)の記事の手順に従って NVIDA GPU ドライバーを N シリーズ Linux サーバーにインストールします。

    > [!NOTE]  
    > ソフトウェアのインストールには、Bash コマンド ラインを使用します。 この同じコマンドを使用して GPU 対応 VM 上の Docker で実行されているコンテナーにドライバーをインストールするため、コマンドをメモしておきます

2.  Azure Stack Hub の N シリーズ Linux サーバーに最新の IoT Edge ランタイムをインストールします。 手順については、「[Debian ベースの Linux システムに Azure IoT Edge ランタイムをインストールする](/azure/iot-edge/how-to-install-iot-edge-linux#install-the-latest-runtime-version)」を参照してください

## <a name="install-docker"></a>Docker のインストール

GPU 対応 VM に Docker をインストールします。 VM 上のコンテナーにある IoT Edge マーケットプレースからモジュールを実行します。

Docker 19.02 以上をインストールする必要があります。 Docker ランタイムで NVIDIA GPU がサポートされるようになりました。 Docker の GPU の詳細については、Docker ドキュメントの[メモリ、CPU、GPU を含むランタイム オプション](https://docs.docker.com/config/containers/resource_constraints/#gpu)に関するページを参照してください。

### <a name="install-docker"></a>Docker のインストール

Docker コンテナーは、オンプレミスのカスタマー データセンター内、外部サービス プロバイダー内、Azure 上のクラウド内など、どこでも実行できます。 Docker イメージ コンテナーは、Linux と Windows 上でネイティブに実行できます。 ただし、Windows イメージは Windows ホスト上でのみ実行でき、Linux イメージは (現在のところ Hyper-V Linux VM を使用して) Linux ホストと Windows ホスト上で実行できます (ここで言うホストとは、サーバーまたは VM です)。 詳細については、「[Docker について](/dotnet/architecture/microservices/container-docker-introduction/docker-defined)」を参照してください。

1. SSH クライアントを使用して N シリーズ Linux サーバーに接続します。

1.  apt インデックスとリストを更新します。

    ```bash  
    sudo apt-get update
    ```

1.  コンピューター上の既存のパッケージの新しいバージョンを取り込みます。

    ```bash  
    sudo apt-get upgrade
    ```

2.  Docker の apt リポジトリを追加するために必要な依存関係をインストールします。

    ```bash  
    sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
    ```

3.  Docker の GPG キーを追加します。

    ```bash  
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```

4.  Docker の apt リポジトリを追加します。

    ```bash  
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    ```

5.  apt インデックスとリストを更新し、Docker Community Edition をインストールします。

    ```bash  
    sudo apt-get update 
    sudo apt-get install docker-ce
    ```

6.  Docker のバージョンを確認して、インストールを確認します。

    ```bash  
    docker -v
    ```

7. Docker で使用可能な GPU リソースを公開します。

    ```bash
    docker run -it --rm --gpus all ubuntu nvidia-smi
    ```

## <a name="get-the-item-from-the-marketplace"></a>マーケットプレースから項目を取得する

Azure portal に戻り、IoT マーケットプレースを使用してエッジ デバイスにモデルを追加します。 メニューから **[Marketplace モジュール]** を選択します。 `Getting started with GPUs` を検索し、指示に従ってモジュールを追加します。

手順については、「[デバイスを選択してモジュールを追加する](/azure/iot-edge/how-to-deploy-modules-portal#select-device-and-add-modules)」を参照してください

## <a name="enable-monitoring"></a>監視を有効にする

1. [Azure IoT エクスプローラー](/azure/iot-pnp/howto-use-iot-explorer)をダウンロードし、アプリケーションを IoT Hub に接続します。

2. IoT デバイスを選択し、ナビゲーション メニューからテレメトリに移動します。

3. IoT Edge デバイスからの出力の監視を開始するには **[開始]** を選択します。

![有効なインストール](media/gpu-deploy-sample-module/user-azure-iot-explorer-gpu.png)

## <a name="monitor-the-module-optional"></a>モジュールを監視する (省略可能)  

1. VS Code コマンド パレットで、**[Azure IoT Hub: Select IoT Hub]\(Azure IoT Hub: IoT ハブの選択\)** を実行します。

2. 構成する IoT Edge デバイスが含まれているサブスクリプションと IoT ハブを選択します。 この場合、Azure Stack Edge デバイスのデプロイに使用するサブスクリプションを選択し、Azure Stack Edge デバイス用に作成された IoT Edge デバイスを選択します。 これは、前の手順で Azure portal からコンピューティングを構成した場合に発生します。

3. VS Code エクスプローラーで、[Azure IoT Hub] セクションを展開します。 **[デバイス]** に、Azure Stack Edge デバイスに対応する IoT Edge デバイスが表示されます。 

    1. そのデバイスを選択して右クリックし、 **[組み込みイベント エンドポイントの監視を開始する]** を選択します。

    2. **[デバイス]、[モジュール]** の順にアクセスして、**GPU モジュール** が実行中であることを確認します。

    3. VS Code ターミナルでも Azure Stack Edge デバイスの監視出力に IoT Hub イベントが含まれていることを確認します。

    ![有効なインストール](media/gpu-deploy-sample-module/gpu-monitor-events-output.png)

    CPU よりも GPU の方が、同じ操作セット (5000 回の図形変換) の実行にかかった時間がずっと短いことがわかります。

## <a name="next-steps"></a>次の手順

  - 「[Azure Stack Hub 上の GPU (グラフィックス処理装置) 仮想マシン (VM)](gpu-vms-about.md)」を確認する

  - 「[コンピューティングの未来: インテリジェント クラウドとインテリジェント エッジ](https://azure.microsoft.com/overview/future-of-cloud)」を参照して Azure Stack Hub、Data Box Edge とインテリジェント エッジの詳細について確認する

  - [ハイブリッド クラウド ソリューション](/hybrid/app-solutions/)に関するページを参照してハイブリッド クラウド アプリケーションの詳細について確認する