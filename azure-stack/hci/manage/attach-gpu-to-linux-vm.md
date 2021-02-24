---
title: Azure Stack HCI で Linux VM に GPU をアタッチする
description: Azure Stack HCI 上の Ubuntu Linux VM で実行されている AI ワークロードに GPU を使用する方法について説明します。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.date: 07/01/2020
ms.openlocfilehash: 422f6984fad6218387673d2dc9292f0ae7cb1739
ms.sourcegitcommit: 7b189e5317b8fe5f8ad825565da3607a39a1b899
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/11/2020
ms.locfileid: "94493654"
---
# <a name="attaching-a-gpu-to-an-ubuntu-linux-vm-on-azure-stack-hci"></a>Azure Stack HCI 上の Ubuntu Linux VM に GPU をアタッチする

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

このトピックでは、Azure Stack HCI と Discrete Device Assignment (DDA) テクノロジを使用して、Ubuntu 仮想マシン (VM) に NVIDIA グラフィックス処理装置 (GPU) をインストールして構成する詳細な手順を紹介します。
このドキュメントは、Azure Stack HCI クラスターがデプロイされ、VM がインストールされていることを前提としています。

## <a name="install-the-gpu-and-then-dismount-it-in-powershell"></a>PowerShell で GPU をインストールし、マウントを解除する

1. OEM のインストラクションと BIOS の推奨事項に従って、適切なサーバーに NVIDIA GPU を物理的にインストールします。
2. 各サーバーの電源をオンにします。
3. NVIDIA GPU がインストールされているサーバーに、管理特権があるアカウントを使用してサインインします。
4. **デバイス マネージャー** を開いて *[その他のデバイス]* セクションに移動します。 デバイスが "3D ビデオ コントローラー" として表示されているのがわかります。
5. [3D ビデオ コントローラー] を右クリックして **[プロパティ]** ページを表示します。 **[詳細]** をクリックします。 **[プロパティ]** ボックスの一覧から [Location paths]\(場所のパス\) を選択します。
6. 下のスクリーンショットで強調表示されている PCIRoot という文字列を含む値に注目してください。 **[値]** を右クリックし、コピーして保存します。
    :::image type="content" source="media/attach-gpu-to-linux-vm/pciroot.png" alt-text="[Location paths]\(場所のパス\) のスクリーンショット":::
7. VM への DDA を行うため、昇格された特権で Windows PowerShell を開き、`Dismount-VMHostAssignableDevice` コマンドレットを実行して、GPU デバイスのマウントを解除します。 *LocationPath* の値は、手順 6. で取得した実際のデバイスの値に置き換えてください。
    ```PowerShell
    Dismount-VMHostAssignableDevice -LocationPath "PCIROOT(16)#PCI(0000)#PCI(0000)" -force
    ```
8. **デバイス マネージャー** のシステム デバイスに、このデバイスが "マウント解除済み" として表示されていることを確認します。
    :::image type="content" source="media/attach-gpu-to-linux-vm/dismounted.png" alt-text="マウント解除済みデバイスのスクリーンショット":::

## <a name="create-and-configure-an-ubuntu-virtual-machine"></a>Ubuntu 仮想マシンを作成して構成する

1. [Ubuntu デスクトップ リリース 18.04.02 ISO](http://old-releases.ubuntu.com/releases/18.04.2/) をダウンロードします。
2. GPU をインストールしたシステムのノードで **Hyper-V マネージャー** を開きます。
   > [!NOTE]
   > [DDA では、フェールオーバーがサポートされません](/windows-server/virtualization/hyper-v/plan/plan-for-deploying-devices-using-discrete-device-assignment)。 これは DDA に関する仮想マシンの制限です。 したがって、**フェールオーバー クラスター マネージャー** ではなく、**Hyper-V マネージャー** を使用してノードに VM をデプロイすることをお勧めします。 DDA で **フェールオーバー クラスター マネージャー** を使用すると、高可用性をサポートしていないデバイスが VM に存在することを示すエラー メッセージが表示されて失敗します。
3. 手順 1. でダウンロードした Ubuntu ISO を使用し、**Hyper-V マネージャー** の **仮想マシンの新規作成ウィザード** を使って新しい仮想マシンを作成します。2 GB のメモリを搭載し、ネットワーク カードがアタッチされた Ubuntu Gen 1 VM を作成してください。
4. PowerShell で下記のコマンドレットを使用して、マウント解除済みの GPU デバイスを VM に割り当てます。*LocationPath* の値は、実際のデバイスの値に置き換えてください。
    ```PowerShell
    # Confirm that there are no DDA devices assigned to the VM
    Get-VMAssignableDevice -VMName Ubuntu

    # Assign the GPU to the VM
    Add-VMAssignableDevice -LocationPath "PCIROOT(16)#PCI(0000)#PCI(0000)" -VMName Ubuntu

    # Confirm that the GPU is assigned to the VM
    Get-VMAssignableDevice -VMName Ubuntu
    ```

    VM への GPU の割り当てに成功すると、次の出力が表示されます。:::image type="content" source="media/attach-gpu-to-linux-vm/assign-gpu.png" alt-text="GPU 割り当てのスクリーンショット":::

    GPU に関する[こちら](/windows-server/virtualization/hyper-v/deploy/deploying-graphics-devices-using-dda)のドキュメントに従って、その他の値を構成します。

   ```PowerShell
    # Enable Write-Combining on the CPU
    Set-VM -GuestControlledCacheTypes $true -VMName VMName

    # Configure the 32 bit MMIO space
    Set-VM -LowMemoryMappedIoSpace 3Gb -VMName VMName

    # Configure greater than 32 bit MMIO space
    Set-VM -HighMemoryMappedIoSpace 33280Mb -VMName VMName
   ```

   > [!NOTE]
   > 33280Mb の値は、ほとんどの GPU で問題ないはずですが、実際の GPU メモリよりも大きい値に置き換える必要があります。

5. Hyper-V マネージャーを使用して、VM に接続し、Ubuntu OS のインストールを開始します。 既定値を選択すると、Ubuntu OS が VM にインストールされます。

6. インストールの完了後、**Hyper-V マネージャー** を使用して VM をシャットダウンし、ゲスト オペレーティング システムをシャットダウンするように VM の **[Automatic Stop Action]\(自動停止アクション\)** を構成します (以下のスクリーンショットを参照)。:::image type="content" source="media/attach-gpu-to-linux-vm/guest-shutdown.png" alt-text="ゲスト OS のシャットダウンのスクリーンショット":::

7. Ubuntu にログインし、ターミナルを開いて SSH をインストールします。

   ```shell
    $ sudo apt install openssh-server
   ```

8. **ifconfig** コマンドを使用して Ubuntu インストール用の TCP/IP アドレスを探し、**eth0** インターフェイスの IP アドレスをコピーします。

9. OpenSSH などの SSH クライアントを使用するか (Windows 10 には既定で ssh.exe がインストールされています) [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/) を使用して、Ubuntu VM に接続して、さらに構成を行います。

10. SSH クライアント経由でログインしたら、**lspci** コマンドを発行して、NVIDIA GPU が "3D コントローラー" として表示されていることを確認します。

    > [!IMPORTANT]
    > NVIDIA GPU が "3D コントローラー" として表示されない場合は、先に進まないでください。 必ず上記の手順に従ったうえで、次の手順に進むようにしてください。

11. VM 内で、 **[Software & Updates]\(ソフトウェアと更新プログラム\)** を探して開きます。 **[追加ドライバー]** に移動し、最新の NVIDIA GPU ドライバーを一覧から選択します。 **[変更の適用]** をクリックして、ドライバーのインストールを完了します。
    :::image type="content" source="media/attach-gpu-to-linux-vm/driver-install.png" alt-text="ドライバー インストールのスクリーンショット":::

12. ドライバーのインストールの完了後、Ubuntu VM を再起動します。 VM の起動後、SSH クライアント経由で接続し、**nvidia-smi** コマンドを発行して、NVIDIA GPU ドライバーのインストールが正常に完了したことを確認します。 出力は次のスクリーンショットのようになるはずです。:::image type="content" source="media/attach-gpu-to-linux-vm/nvidia-smi.png" alt-text="nvidia-smi コマンドからの出力を示すスクリーンショット。":::

13. SSH クライアントを使用してリポジトリを設定し、Docker CE エンジンをインストールします。

    ```shell
    $ sudo apt-get update
    $ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
    ```
    Docker の公式 GPG キーを追加します。

    ```shell
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```

    フィンガープリント 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88 の最後の 8 文字を検索して、このフィンガープリントを含んだキーがあることを確認します。

    ```shell
    $ sudo apt-key fingerprint 0EBFCD88
    ```

    出力は次のようになります。

    ```shell
    pub   rsa4096 2017-02-22 [SCEA]
    9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
    uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
    sub   rsa4096 2017-02-22 [S]
    ```

    Ubuntu AMD64 アーキテクチャの安定版リポジトリを設定します。

    ```shell
    $ sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
    ```

    パッケージを更新し、Docker CE をインストールします。

    ```shell
    $ sudo apt-get update
    $ sudo apt-get install docker-ce docker-ce-cli containerd.io
    ```

    Docker CE のインストールを確認します。

    ```shell
    $ sudo docker run hello-world
    ```

## <a name="configure-azure-iot-edge"></a>Azure IoT Edge を構成する

この構成に備えて、[NVIDIA-Deepstream-Azure-IoT-Edge-on-a-NVIDIA-Jetson-Nano](https://github.com/Azure-Samples/NVIDIA-Deepstream-Azure-IoT-Edge-on-a-NVIDIA-Jetson-Nano) GitHub リポジトリにある FAQ をご確認ください。Moby ではなく Docker をインストールする必要性について説明されています。 確認後、以下の手順に進んでください。

### <a name="install-nvidia-docker"></a>NVIDIA Docker をインストールする

1. SSH クライアントから、パッケージのリポジトリを追加します。

    ```shell
    curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
    sudo apt-key add -
    distribution=$(. /etc/os-release;echo $ID$VERSION_ID)

    curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
    sudo tee /etc/apt/sources.list.d/nvidia-docker.list

    sudo apt-get update
    ```

2. nvidia-docker2 をインストールし、Docker デーモン構成を再度読み込みます。

    ```shell
    sudo apt-get install -y nvidia-docker2
    sudo pkill -SIGHUP dockerd
    ```

3. VM を再起動します。

    ```shell
    sudo /sbin/shutdown -r now
    ```

4. 再起動時に、NVIDIA Docker が正常にインストールされていることを確認します。

    ```shell
    sudo docker run --runtime=nvidia --rm nvidia/cuda:9.0-base nvidia-smi
    ```

    インストールに成功した場合の出力は、次のスクリーンショットのようになります。:::image type="content" source="media/attach-gpu-to-linux-vm/docker.png" alt-text="Docker のインストールに成功した場合のスクリーンショット":::

5. 以下の手順に従って、Azure IoT Edge のインストールに進みます。ランタイムのインストールはスキップしてください。

    ```shell
    curl https://packages.microsoft.com/config/ubuntu/18.04/multiarch/prod.list > ./microsoft-prod.list

    sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/

    curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
    sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
    sudo apt-get update

    sudo apt-get install iotedge
    ```

    > [!NOTE]
    > Azure IoT Edge のインストール後、Ubuntu VM の /etc/iotedge/config.yaml に config.yaml が存在することを確認します。

6. Azure portal で、[こちら](/azure/iot-edge/how-to-register-device#register-in-the-azure-portal)のガイダンスに従って IoT Edge デバイス ID を作成します。 次に、新しく作成した IoT Edge 用のデバイス接続文字列をコピーします。

7. SSH クライアントを使用して、Ubuntu VM 上の config.yaml にあるデバイス接続文字列を更新します。

    ```shell
    sudo nano /etc/iotedge/config.yaml
    ```

    ファイルのプロビジョニング構成を見つけ、"Manual provisioning configuration" (手動プロビジョニングの構成) セクションをコメント解除します。 device_connection_string の値をご自分の IoT Edge デバイスの接続文字列で更新します。 他のプロビジョニング セクションがすべてコメント アウトされていることを確認します。provisioning: の行に先行する空白文字がなく、入れ子になった項目が 2 つの空白でインデントされていることを確認します。

    :::image type="content" source="media/attach-gpu-to-linux-vm/manual-provisioning.png" alt-text="手動プロビジョニング構成のスクリーンショット":::

    クリップボードの内容を Nano に貼り付けるには、Shift キーを押しながら右クリックするか、Shift キーを押しながら Ins キーを押します。 ファイルを保存して閉じます (Ctrl + X、Y、Enter)。

8. SSH クライアントを使用して、IoT Edge デーモンを再起動します。

    ```shell
    sudo systemctl restart iotedge
    ```

    インストールを確認して、IoT Edge デーモンの状態をチェックします。

    ```shell
    systemctl status iotedge

    journalctl -u iotedge --no-pager --no-full
    ```

9. SSH クライアントを使用して、Ubuntu VM に次のディレクトリ構造を作成します。

    ```shell
    cd /var
    sudo mkdir deepstream
    mkdir ./deepstream/custom_configs
    cd /var/deepstream
    sudo mkdir custom_streams
    sudo chmod -R 777 /var/deepstream
    cd ./custom_streams
    ```

10. 作業ディレクトリが /var/deepstream/custom_streams であることを確認し、SSH クライアントから次のコマンドを実行して、[デモ ビデオ ファイル](https://github.com/Azure-Samples/NVIDIA-Deepstream-Azure-IoT-Edge-on-a-NVIDIA-Jetson-Nano)をダウンロードします。

    ```shell
    wget -O cars-streams.tar.gz --no-check-certificate https://onedrive.live.com/download?cid=0C0A4A69A0CDCB4C&resid=0C0A4A69A0CDCB4C%21588371&authkey=AAavgrxG95v9gu0
    ```

    ビデオ ファイルの圧縮を解除します。

    ```shell
    tar -xzvf cars-streams.tar.gz
    ```

    ディレクトリ /var/deepstream/custom_streams は、次のスクリーンショットに示したような内容になっています。

    :::image type="content" source="media/attach-gpu-to-linux-vm/custom-streams.png" alt-text="custom_streams のスクリーンショット":::

11. /var/deepstream/custom_configs ディレクトリに、test5_config_file_src_infer_azure_iotedge_edited.txt という新しいファイルを作成します。 テキスト エディターを使用してファイルを開き、次のコードを貼り付けてから、ファイルを保存して閉じます。

    ```shell
    # Copyright (c) 2018 NVIDIA Corporation.  All rights reserved.
    #
    # NVIDIA Corporation and its licensors retain all intellectual property
    # and proprietary rights in and to this software, related documentation
    # and any modifications thereto.  Any use, reproduction, disclosure or
    # distribution of this software and related documentation without an express
    # license agreement from NVIDIA Corporation is strictly prohibited.

    [application]
    enable-perf-measurement=1
    perf-measurement-interval-sec=5
    #gie-kitti-output-dir=streamscl

    [tiled-display]
    enable=1
    rows=2
    columns=2
    width=1280
    height=720
    gpu-id=0
    #(0): nvbuf-mem-default - Default memory allocated, specific to particular platform
    #(1): nvbuf-mem-cuda-pinned - Allocate Pinned/Host cuda memory, applicable for Tesla
    #(2): nvbuf-mem-cuda-device - Allocate Device cuda memory, applicable for Tesla
    #(3): nvbuf-mem-cuda-unified - Allocate Unified cuda memory, applicable for Tesla
    #(4): nvbuf-mem-surface-array - Allocate Surface Array memory, applicable for Jetson
    nvbuf-memory-type=0

    [source0]
    enable=1
    #Type - 1=CameraV4L2 2=URI 3=MultiURI
    type=3
    uri=file://../../../../../samples/streams/sample_1080p_h264.mp4
    num-sources=2
    gpu-id=0
    nvbuf-memory-type=0

    [source1]
    enable=1
    #Type - 1=CameraV4L2 2=URI 3=MultiURI
    type=3
    uri=file://../../../../../samples/streams/sample_1080p_h264.mp4
    num-sources=2
    gpu-id=0
    nvbuf-memory-type=0

    [sink0]
    enable=0

    [sink3]
    enable=1
    #Type - 1=FakeSink 2=EglSink 3=File 4=RTSPStreaming
    type=4
    #1=h264 2=h265
    codec=1
    sync=0
    bitrate=4000000
    # set below properties in case of RTSPStreaming
    rtsp-port=8554
    udp-port=5400

    [sink1]
    enable=1
    #Type - 1=FakeSink 2=EglSink 3=File 4=UDPSink 5=nvoverlaysink 6=MsgConvBroker
    type=6
    msg-conv-config=../configs/dstest5_msgconv_sample_config.txt
    #(0): PAYLOAD_DEEPSTREAM - Deepstream schema payload
    #(1): PAYLOAD_DEEPSTREAM_MINIMAL - Deepstream schema payload minimal
    #(256): PAYLOAD_RESERVED - Reserved type
    #(257): PAYLOAD_CUSTOM   - Custom schema payload
    msg-conv-payload-type=1
    msg-broker-proto-lib=/opt/nvidia/deepstream/deepstream-4.0/lib/libnvds_azure_edge_proto.so
    topic=mytopic
    #Optional:
    #msg-broker-config=../../../../libs/azure_protocol_adaptor/module_client/cfg_azure.txt

    [sink2]
    enable=0
    type=3
    #1=mp4 2=mkv
    container=1
    #1=h264 2=h265 3=mpeg4
    ## only SW mpeg4 is supported right now.
    codec=3
    sync=1
    bitrate=2000000
    output-file=out.mp4
    source-id=0

    [osd]
    enable=1
    gpu-id=0
    border-width=1
    text-size=15
    text-color=1;1;1;1;
    text-bg-color=0.3;0.3;0.3;1
    font=Arial
    show-clock=0
    clock-x-offset=800
    clock-y-offset=820
    clock-text-size=12
    clock-color=1;0;0;0
    nvbuf-memory-type=0

    [streammux]
    gpu-id=0
    ##Boolean property to inform muxer that sources are live
    live-source=0
    batch-size=4
    ##time out in usec, to wait after the first buffer is available
    ##to push the batch even if the complete batch is not formed
    batched-push-timeout=40000
    ## Set muxer output width and height
    width=1920
    height=1080
    ##Enable to maintain aspect ratio wrt source, and allow black borders, works
    ##along with width, height properties
    enable-padding=0
    nvbuf-memory-type=0

    [primary-gie]
    enable=1
    gpu-id=0
    batch-size=4
    ## 0=FP32, 1=INT8, 2=FP16 mode
    bbox-border-color0=1;0;0;1
    bbox-border-color1=0;1;1;1
    bbox-border-color2=0;1;1;1
    bbox-border-color3=0;1;0;1
    nvbuf-memory-type=0
    interval=0
    gie-unique-id=1
    model-engine-file=../../../../../samples/models/Primary_Detector/resnet10.caffemodel_b4_int8.engine
    labelfile-path=../../../../../samples/models/Primary_Detector/labels.txt
    config-file=../../../../../samples/configs/deepstream-app/config_infer_primary.txt
    #infer-raw-output-dir=../../../../../samples/primary_detector_raw_output/

    [tracker]
    enable=1
    tracker-width=600
    tracker-height=300
    ll-lib-file=/opt/nvidia/deepstream/deepstream-4.0/lib/libnvds_mot_klt.so
    #ll-config-file required for DCF/IOU only
    #ll-config-file=tracker_config.yml
    #ll-config-file=iou_config.txt
    gpu-id=0
    #enable-batch-process applicable to DCF only
    enable-batch-process=0

    [tests]
    file-loop=1
    ```

12. Azure Portal に移動します。 **[IoT Hub Provisioned]\(プロビジョニングされている IoT Hub\)** を選択し、 **[デバイスの自動管理]** をクリックして、 **[IoT Edge]** をクリックします。

    :::image type="content" source="media/attach-gpu-to-linux-vm/iot-edge.png" alt-text="[デバイスの自動管理] のスクリーンショット":::

13. 右側のペインで、先ほど使用したデバイス接続文字列に対応するデバイス ID を選択します。 [モジュールの設定] をクリックします。

    :::image type="content" source="media/attach-gpu-to-linux-vm/set-modules.png" alt-text="[モジュールの設定] のスクリーンショット":::

14. [IoT Edge モジュール] で、[IoT Edge モジュール] をクリックして選択します。

    :::image type="content" source="media/attach-gpu-to-linux-vm/marketplace-module.png" alt-text="IoT Edge モジュールの追加のスクリーンショット":::

15. **[IoT Edge モジュールを追加する]** ペインで **[モジュールの設定]** タブを選択し、次の値を入力または選択します。

    - **[IoT Edge モジュール名]** : NVIDIADeepStreamSDK

    - **[イメージの URI]** : marketplace.azurecr.io/nvidia/deepstream-iot2

    - **[再起動ポリシー]** : [常に]

    - **[必要な状態]** : [実行中]

    - **[Image Pull Policy]\(イメージ プル ポリシー\)** : "*空白*"
    
    **[追加]** を選択します。

    :::image type="content" source="media/attach-gpu-to-linux-vm/deepstream-module-settings.png" alt-text="DeepStream SDK のスクリーンショット":::

16. [IoT Edge モジュール] に NvidiaDeepStreamSDK モジュールが一覧表示されます。

    :::image type="content" source="media/attach-gpu-to-linux-vm/edge-modules.png" alt-text="[IoT Edge モジュール] のスクリーンショット":::

17. "NVIDIADeepStreamSDK" モジュールをクリックし、[コンテナーの作成オプション] を選択します。 既定の構成は次のとおりです。

    :::image type="content" source="media/attach-gpu-to-linux-vm/container-create-options.png" alt-text="[コンテナーの作成オプション] のスクリーンショット":::

    上記の構成を下記の構成に置き換えます。

    ```shell
    {
      "ExposedPorts": {
        "8554/tcp": {}
      },
      "Entrypoint": [
        "/usr/bin/deepstream-test5-app",
        "-c",
        "test5_config_file_src_infer_azure_iotedge_edited.txt",
        "-p",
        "1",
        "-m",
        "1"
      ],
      "HostConfig": {
        "runtime": "nvidia",
        "Binds": [
          "/var/deepstream/custom_configs:/root/deepstream_sdk_v4.0.2_x86_64/sources/apps/sample_apps/deepstream-test5/custom_configs/",
          "/var/deepstream/custom_streams:/root/deepstream_sdk_v4.0.2_x86_64/sources/apps/sample_apps/deepstream-test5/custom_streams/"
        ],
        "PortBindings": {
          "8554/tcp": [
            {
              "HostPort": "8554"
            }
          ]
        }
      },
      "WorkingDir": "/root/deepstream_sdk_v4.0.2_x86_64/sources/apps/sample_apps/deepstream-test5/custom_configs/"
    }
    ```

18. **[確認と作成]** をクリックし、次のページで **[作成]** をクリックします。 これで Azure portal には、IoT Edge デバイスの 3 つのモジュールが表示されます。

    :::image type="content" source="media/attach-gpu-to-linux-vm/edge-hub-connections.png" alt-text="[モジュール] と [IoT Edge ハブの接続] のスクリーンショット":::

19. SSH クライアントを使用して Ubuntu VM に接続し、適切なモジュールが実行されていることを確認します。

    ```shell
    sudo iotedge list
    ```

    :::image type="content" source="media/attach-gpu-to-linux-vm/verify-modules-sudo.png" alt-text="iotedge list の出力を示すスクリーンショット。":::

    ```shell
    nvidia-smi
    ```

    :::image type="content" source="media/attach-gpu-to-linux-vm/verify-modules-nvidia-smi.png" alt-text="nvidia-smi のスクリーンショット":::

    > [!NOTE]
    > NvidiaDeepstream コンテナーがダウンロードされるまでに数分かかります。 "journalctl -u iotedge --no-pager --no-full" コマンドを使用して iotedge デーモンのログを確認することで、ダウンロードを検証することができます。

20. NvdiaDeepStreem コンテナーが稼動していることを確認します。 次のスクリーンショットのコマンド出力から、正常に実行されていることがわかります。

    ```shell
    sudo iotedge list
    ```

    :::image type="content" source="media/attach-gpu-to-linux-vm/verify1.png" alt-text="NvdiaDeepStreem コンテナーが稼動していることを示す出力のスクリーンショット。":::

    ```shell
    sudo iotedge logs -f NVIDIADeepStreamSDK
    ```

    :::image type="content" source="media/attach-gpu-to-linux-vm/verify2.png" alt-text="iotedge logs -f NVIDIADeepStreamSDK コマンドの出力を示すスクリーンショット。":::

    ```shell
    nvidia-smi
    ```

    :::image type="content" source="media/attach-gpu-to-linux-vm/verify3.png" alt-text="nvidia-smi コマンドの追加の出力を示すスクリーンショット。":::

21. **ifconfig** コマンドを使用して Ubuntu VM の TCP/IP アドレスを確認し、**eth0** インターフェイスの横の TCP/IP アドレスを探します。

22. VLC Player をワークステーションにインストールします。 VLC Player 内で **[Media]\(メディア\)、[open network stream]\(ネットワーク ストリームを開く\)** の順にクリックし、次の形式を使用してアドレスを入力します。

    rtsp://ipaddress:8554/ds-test

    ipaddress には、自分の VM の TCP/IP アドレスを指定します。

    :::image type="content" source="media/attach-gpu-to-linux-vm/vlc-player.png" alt-text="VLC Player のスクリーンショット":::

## <a name="next-steps"></a>次のステップ

GPU と DDA の詳細については、以下も参照してください。

- [Discrete Device Assignment を使用したデバイスのデプロイの計画](/windows-server/virtualization/hyper-v/plan/plan-for-deploying-devices-using-discrete-device-assignment)
- [Discrete Device Assignment を使用したグラフィックス デバイスのデプロイ](/windows-server/virtualization/hyper-v/deploy/deploying-graphics-devices-using-dda)
