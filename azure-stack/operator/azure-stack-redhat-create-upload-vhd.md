---
title: Azure Stack Hub 用の Red Hat ベースの仮想マシンの準備
titleSuffix: Azure Stack Hub
description: Red Hat Linux オペレーティング システムを格納した Azure 仮想ハード ディスク (VHD) を作成してアップロードする方法について説明します。
author: sethmanheim
ms.topic: article
ms.date: 08/28/2020
ms.author: sethm
ms.reviewer: kivenkat
ms.lastreviewed: 12/11/2019
ms.openlocfilehash: 3ddc8a44d59a373f5dfe340860a5dcf195668cac
ms.sourcegitcommit: 4af79f4fa2598d57c81e994192c10f8c6be5a445
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/10/2020
ms.locfileid: "89742571"
---
# <a name="prepare-a-red-hat-based-virtual-machine-for-azure-stack-hub"></a>Azure Stack Hub 用の Red Hat ベースの仮想マシンの準備

この記事では、Red Hat Enterprise Linux (RHEL) の仮想マシン (VM) を Azure Stack Hub で使用できるように準備する方法について説明します。 この記事で取り上げる RHEL のバージョンは 7.1 以降です。 準備対象のハイパーバイザーは Hyper-V、Kernel-based Virtual Machine (KVM)、VMware です。

Red Hat Enterprise Linux のサポート情報については、「[Red Hat and Azure Stack: Frequently Asked Questions](https://access.redhat.com/articles/3413531)」(Red Hat と Azure Stack: よく寄せられる質問) を参照してください。

## <a name="prepare-a-red-hat-based-vm-from-hyper-v-manager"></a>Hyper-V マネージャーからの Red Hat ベースの VM の準備

このセクションは、Red Hat の Web サイトから取得した ISO ファイルの RHEL イメージが仮想ハード ディスク (VHD) にインストール済みであることを前提としています。 Hyper-V マネージャーを使用してオペレーティング システム イメージをインストールする方法の詳細については、[Hyper-V ロールのインストールと VM の構成](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11))に関するページを参照してください。

### <a name="rhel-installation-notes"></a>RHEL のインストールに関する注記

* Azure Stack Hub では、VHDX 形式はサポートされません。 Azure でサポートされるのは、容量固定の VHD のみです。 Hyper-V マネージャーを使ってディスクの形式を VHD に変換するか、または **convert-vhd コマンドレット**を使用してください。 VirtualBox を使用する場合は、ディスクの作成時に、既定の動的割り当てオプションではなく、 **[容量固定]** を選択します。
* Azure Stack Hub でサポートされるのは、第 1 世代の VM のみです。 第 1 世代の VM を、VHDX ファイル形式から VHD ファイル形式に、および容量可変から容量固定ディスクに変換できます。 VM の世代を変更することはできません。 詳細については、[Hyper-V で第 1 世代と第 2 世代のどちらの VM を作成する必要があるか](/windows-server/virtualization/hyper-v/plan/Should-I-create-a-generation-1-or-2-virtual-machine-in-Hyper-V)に関するページを参照してください。
* VHD のサイズの上限は、1,023 GB です。
* Linux オペレーティング システムをインストールする場合は、Logical Volume Manager (LVM) (通常、多くのインストールで既定) ではなく標準パーティションを使用することをお勧めします。 これにより、特にオペレーティング システム ディスクをトラブルシューティングのために別の同じ VM に接続する必要がある場合に、複製された VM との LVM 名の競合を回避することができます。
* ユニバーサル ディスク フォーマット (UDF) ファイル システムをマウントするためのカーネル サポートが必要です。 最初の起動時に、ゲストに接続されている UDF 形式のメディアにより、プロビジョニング構成が Linux VM に渡されます。 Azure Linux エージェントは、その構成を読み取り、VM をプロビジョニングするために、UDF ファイル システムをマウントする必要があります。
* オペレーティング システム ディスクでスワップ パーティションを構成しないでください。 Linux エージェントは、一時的なリソース ディスク上にスワップ ファイルを作成するよう構成できます。 このことに関する詳細については、次の手順を参照してください。
* Azure の VHD の仮想サイズはすべて、1 MB にアラインメントさせる必要があります。 未フォーマット ディスクから VHD に変換するときに、変換する前の未フォーマット ディスクのサイズが 1 MB の倍数であることを確認する必要があります。 詳細については、後述の手順を参照してください。
* Azure Stack Hub では、cloud-init がサポートされています。 [cloud-Init](/azure/virtual-machines/linux/using-cloud-init) は、Linux VM を初回起動時にカスタマイズするために広く使用されているアプローチです。 cloud-init を使って、パッケージをインストールしてファイルを書き込んだり、ユーザーとセキュリティを構成したりすることができます。 cloud-init は初回起動プロセスの間に呼び出されるので、構成を適用するために追加の手順や必要なエージェントはありません。 イメージに cloud-init を追加する手順については、「[cloud-init で使用するための既存の Linux Azure VM イメージの準備](/azure/virtual-machines/linux/cloudinit-prepare-custom-image)」を参照してください。

### <a name="prepare-an-rhel-7-vm-from-hyper-v-manager"></a>Hyper-V マネージャーからの RHEL 7 VM の準備

1. Hyper-V マネージャーで VM を選択します。

1. **[接続]** を選択すると、VM のコンソール ウィンドウが開きます。

1. `/etc/sysconfig/network` ファイルを作成または編集して次のテキストを追加します。

    ```shell
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

1. 必要に応じて `/etc/sysconfig/network-scripts/ifcfg-eth0` ファイルを作成または編集して次のテキストを追加します。

    ```shell
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    NM_CONTROLLED=no
    ```

1. 次のコマンドを実行して、起動時にネットワーク サービスが開始されるようにします。

    ```bash
    sudo systemctl enable network
    ```

1. RHEL リポジトリからパッケージをインストールできるように、次のコマンドを実行して Red Hat のサブスクリプションを登録します。

    ```bash
    sudo subscription-manager register --auto-attach --username=XXX --password=XXX
    ```

1. GRUB 構成でカーネルのブート行を変更して Azure の追加のカーネル パラメーターを含めます。 この変更を行うには、テキスト エディターで `/etc/default/grub` を開き、`GRUB_CMDLINE_LINUX` パラメーターを変更します。 次に例を示します。

    ```shell
    GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
    ```

   この変更により、すべてのコンソール メッセージが最初のシリアル ポートに送信され、Azure サポートが問題をデバッグするのに役立ちます。 NIC の新しい RHEL 7 名前付け規則もオフになります。

   クラウド環境では、すべてのログをシリアル ポートに送信するため、グラフィカル ブートおよびクワイエット ブートは役立ちません。 `crashkernel` オプションの構成は、必要であればそのままにしてかまいません。 ただし、このパラメーターにより、VM 内の使用可能なメモリ量が 128 MB 以上減少します。VM のサイズが小さいと、このことが問題になる可能性があります。 次のパラメーターを削除することをお勧めします。

    ```shell
    rhgb quiet crashkernel=auto
    ```

1. `/etc/default/grub` の編集を終了したら、次のコマンドを実行して GRUB 構成を再構築します。

    ```bash
    sudo grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

1. [1910 リリースより後のオプション] cloud-init を停止してアンインストールします。

    ```bash
    systemctl stop cloud-init
    yum remove cloud-init
    ```

1. SSH サーバーがインストールされており、起動時に開始するように構成されていることを確認します。通常は、既定でそのように構成されています。 `/etc/ssh/sshd_config` を変更して、次の行を含めます。

    ```shell
    ClientAliveInterval 180
    ```

1. Azure Stack Hub 用のカスタム VHD を作成する際には、バージョンが 2.2.20 と 2.2.35 の間 (両バージョンは除く) の WALinuxAgent は、1910 リリースより前の Azure Stack Hub 環境では機能しないことに留意してください。 バージョン 2.2.20/2.2.35 を使用してイメージを準備できます。 2\.2.35 より後のバージョンを使用してカスタム イメージを準備するには、Azure Stack Hub を 1903 リリース以降に更新するか、1901 および 1902 修正プログラムを適用します。
    
    [1910 リリースより前] 次の手順に従って、互換性のある WALinuxAgent をダウンロードします。
    
    1. setuptools をダウンロードします。
        
        ```bash
        wget https://pypi.python.org/packages/source/s/setuptools/setuptools-7.0.tar.gz --no-check-certificate
        tar xzf setuptools-7.0.tar.gz
        cd setuptools-7.0
        ```
    
    1. 2\.2.20 バージョンのエージェントを GitHub からダウンロードして解凍します。

        ```bash
        wget https://github.com/Azure/WALinuxAgent/archive/v2.2.20.zip
        unzip v2.2.20.zip
        cd WALinuxAgent-2.2.20
        ```

    1. setup.py をインストールします。

        ```bash
        sudo python setup.py install
        ```

    1. waagent を再起動します。
    
        ```bash
        sudo systemctl restart waagent
        ```

    1. エージェントのバージョンがダウンロードしたものと一致するかどうかをテストします。 この例では、2.2.20 となります。

        ```bash
        waagent -version
        ```

    1910 リリースより後では、次の手順に従って、互換性のある WALinuxAgent をダウンロードします。
    
    1. WALinuxAgent パッケージ `WALinuxAgent-<version>` が Red Hat extras リポジトリにプッシュされました。 次のコマンドを実行して extras リポジトリを有効にします。

        ```bash
        subscription-manager repos --enable=rhel-7-server-extras-rpms
        ```

    1. 次のコマンドを実行して Azure Linux エージェントをインストールします。

        ```bash
        sudo yum install WALinuxAgent
        sudo systemctl enable waagent.service
        ```

1. オペレーティング システム ディスクにスワップ領域を作成しないでください。

    Azure Linux エージェントは、VM が Azure にプロビジョニングされた後に VM に接続されたローカルのリソース ディスクを使用してスワップ領域を自動的に構成できます。 ローカル リソース ディスクは一時ディスクであるため、VM のプロビジョニングが解除されると空になることがあります。 前の手順で Azure Linux エージェントのインストール後に、`/etc/waagent.conf` にある次のパラメーターを適切に変更します。

    ```shell
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    #NOTE: set this to whatever you need it to be.
    ```

1. サブスクリプションを登録解除する場合は、次のコマンドを実行します。

    ```bash
    sudo subscription-manager unregister
    ```

1. エンタープライズ証明機関を使用してデプロイされたシステムを使用している場合、RHEL VM は Azure Stack Hub のルート証明書を信頼しません。 この証明書は、信頼されたルート ストアに配置する必要があります。 詳細については、「[信頼できるルート証明書のサーバーへの追加](https://manuals.gfi.com/en/kerio/connect/content/server-configuration/ssl-certificates/adding-trusted-root-certificates-to-the-server-1605.html)」を参照してください。

1. 次のコマンドを実行して VM をプロビジョニング解除し、Azure でのプロビジョニング用に準備します。

    ```bash
    sudo waagent -force -deprovision
    export HISTSIZE=0
    logout
    ```

1. Hyper-V マネージャーで **[アクション]** 、 **[シャットダウン]** の順に選択します。

1. Hyper-V Manager の "ディスクの編集" 機能または Convert-VHD PowerShell コマンドを使用して、VHD を固定サイズの VHD に変換します。 これで、Linux VHD を Azure にアップロードする準備が整いました。

## <a name="prepare-a-red-hat-based-virtual-machine-from-kvm"></a>KVM からの Red Hat ベースの仮想マシンの準備

1. Red Hat の Web サイトから、RHEL 7 の KVM イメージをダウンロードします。 この手順では、例として RHEL 7 を使用します。

1. ルート パスワードを設定します。

    暗号化されたパスワードを生成し、コマンドの出力をコピーします。

    ```bash
    openssl passwd -1 changeme
    ```

   guestfish でルート パスワードを設定します。

    ```shell
    guestfish --rw -a <image-name>
    > <fs> run
    > <fs> list-filesystems
    > <fs> mount /dev/sda1 /
    > <fs> vi /etc/shadow
    > <fs> exit
    ```

   ルート ユーザーの 2 番目のフィールドを、"!!" から 暗号化されたパスワードに変更します。

1. qcow2 イメージから KVM に VM を作成します。 ディスクの種類を **qcow2** に設定して、仮想ネットワーク インターフェイスのデバイス モデルを **virtio** に設定します。 その後、VM を起動し、root としてサインインします。

1. `/etc/sysconfig/network` ファイルを作成または編集して次のテキストを追加します。

    ```shell
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

1. `/etc/sysconfig/network-scripts/ifcfg-eth0` ファイルを作成または編集して次のテキストを追加します。

    ```shell
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    NM_CONTROLLED=no
    ```

1. 次のコマンドを実行して、起動時にネットワーク サービスが開始されるようにします。

    ```bash
    sudo systemctl enable network
    ```

1. RHEL リポジトリからパッケージをインストールできるように、次のコマンドを実行して Red Hat のサブスクリプションを登録します。

    ```bash
    subscription-manager register --auto-attach --username=XXX --password=XXX
    ```

1. GRUB 構成でカーネルのブート行を変更して Azure の追加のカーネル パラメーターを含めます。 この構成を行うには、テキスト エディターで `/etc/default/grub` を開き、`GRUB_CMDLINE_LINUX` パラメーターを変更します。 次に例を示します。

    ```shell
    GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
    ```

   このコマンドにより、すべてのコンソール メッセージが最初のシリアル ポートに送信され、メッセージを Azure での問題のデバッグに利用できるようになります。 NIC の新しい RHEL 7 名前付け規則も、このコマンドでオフになります。

   クラウド環境では、すべてのログをシリアル ポートに送信するため、グラフィカル ブートおよびクワイエット ブートは役立ちません。 `crashkernel` オプションの構成は、必要であればそのままにしてかまいません。 ただし、このパラメーターにより、VM 内の使用可能なメモリ量が 128 MB 以上減少します。VM のサイズが小さいと、このことが問題になる可能性があります。 次のパラメーターを削除することをお勧めします。

    ```shell
    rhgb quiet crashkernel=auto
    ```

1. `/etc/default/grub` の編集を終了したら、次のコマンドを実行して GRUB 構成を再構築します。

    ```bash
    grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

1. Hyper-V モジュールを initramfs に追加します。

    `/etc/dracut.conf` を編集して、コンテンツを追加します。

    ```shell
    add_drivers+="hv_vmbus hv_netvsc hv_storvsc"
    ```

    initramfs を再構築します。

    ```bash
    dracut -f -v
    ```

1. [1910 リリースより後のオプション] cloud-init を停止してアンインストールします。

    ```bash
    systemctl stop cloud-init
    yum remove cloud-init
    ```

1. SSH サーバーがインストールされており、起動時に開始するように構成されていることを確認します。

    ```bash
    systemctl enable sshd
    ```

    /etc/ssh/sshd_config を変更して、次の行を含めます。

    ```shell
    PasswordAuthentication yes
    ClientAliveInterval 180
    ```

1. Azure Stack Hub 用のカスタム VHD を作成する際には、バージョンが 2.2.20 と 2.2.35 の間 (両バージョンは除く) の WALinuxAgent は、1910 リリースより前の Azure Stack Hub 環境では機能しないことに留意してください。 バージョン 2.2.20/2.2.35 を使用してイメージを準備できます。 2\.2.35 より後のバージョンを使用してカスタム イメージを準備するには、Azure Stack Hub を 1903 リリース以降に更新するか、1901 および 1902 修正プログラムを適用します。

    [1910 リリースより前] 次の手順に従って、互換性のある WALinuxAgent をダウンロードします。

    1. setuptools をダウンロードします。
        
        ```bash
        wget https://pypi.python.org/packages/source/s/setuptools/setuptools-7.0.tar.gz --no-check-certificate
        tar xzf setuptools-7.0.tar.gz
        cd setuptools-7.0
        ```
        
    1. 2\.2.20 バージョンのエージェントを GitHub からダウンロードして解凍します。
        
        ```bash
        wget https://github.com/Azure/WALinuxAgent/archive/v2.2.20.zip
        unzip v2.2.20.zip
        cd WALinuxAgent-2.2.20
        ```
        
    1. setup.py をインストールします。
        
        ```bash
        sudo python setup.py install
        ```
        
    1. waagent を再起動します。
        
        ```bash
        sudo systemctl restart waagent
        ```
        
    1. エージェントのバージョンがダウンロードしたものと一致するかどうかをテストします。 この例では、2.2.20 となります。
        
        ```bash
        waagent -version
        ```
        
    [1910 リリースより後] 次の手順に従って、互換性のある WALinuxAgent をダウンロードします。
    
    1. WALinuxAgent パッケージ `WALinuxAgent-<version>` が Red Hat extras リポジトリにプッシュされました。 次のコマンドを実行して extras リポジトリを有効にします。

        ```bash
        subscription-manager repos --enable=rhel-7-server-extras-rpms
        ```

    1. 次のコマンドを実行して Azure Linux エージェントをインストールします。

        ```bash
        sudo yum install WALinuxAgent
        sudo systemctl enable waagent.service
        ```

1. オペレーティング システム ディスクにスワップ領域を作成しないでください。

    Azure Linux エージェントは、VM が Azure にプロビジョニングされた後に VM に接続されたローカルのリソース ディスクを使用してスワップ領域を自動的に構成できます。 ローカル リソース ディスクは一時ディスクであるため、VM のプロビジョニングが解除されると空になることがあります。 前の手順で Azure Linux エージェントのインストール後に、`/etc/waagent.conf` にある次のパラメーターを適切に変更します。

    ```shell
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    #NOTE: set this to whatever you need it to be.
    ```

1. 次のコマンドを実行して、サブスクリプションを登録解除します (必要な場合)。

    ```bash
    subscription-manager unregister
    ```

1. エンタープライズ証明機関を使用してデプロイされたシステムを使用している場合、RHEL VM は Azure Stack Hub のルート証明書を信頼しません。 この証明書は、信頼されたルート ストアに配置する必要があります。 詳細については、「[信頼できるルート証明書のサーバーへの追加](https://manuals.gfi.com/en/kerio/connect/content/server-configuration/ssl-certificates/adding-trusted-root-certificates-to-the-server-1605.html)」を参照してください。

1. 次のコマンドを実行して VM をプロビジョニング解除し、Azure でのプロビジョニング用に準備します。

    ```bash
    sudo waagent -force -deprovision
    export HISTSIZE=0
    logout
    ```

1. KVM で VM をシャット ダウンします。

1. qcow2 イメージを VHD 形式に変換します。

    > [!NOTE]
    > qemu-img のバージョン 2.2.1 以降には VHD が適切にフォーマットされないというバグがあることがわかっています。 この問題は QEMU 2.6 で修正されています。 qemu-img 2.2.0 以前を使用するか、2.6 以降に更新することをお勧めします。 [https://bugs.launchpad.net/qemu/+bug/1490611](https://bugs.launchpad.net/qemu/+bug/1490611 ) を参照してください。

    まず、イメージを未加工の形式に変換します。

    ```bash
    qemu-img convert -f qcow2 -O raw rhel-7.4.qcow2 rhel-7.4.raw
    ```

    未加工のイメージのサイズが 1 MB になっていることを確認します。 そうでない場合は、1 MB になるようにサイズの端数を切り上げます。

    ```bash
    MB=$((1024*1024))
    size=$(qemu-img info -f raw --output json "rhel-7.4.raw" | \
    gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')
    rounded_size=$((($size/$MB + 1)*$MB))
    qemu-img resize rhel-7.4.raw $rounded_size
    ```

    未フォーマット ディスクを固定サイズの VHD に変換します。

    ```bash
    qemu-img convert -f raw -o subformat=fixed -O vpc rhel-7.4.raw rhel-7.4.vhd
    ```

    または、qemu バージョン **2.6 以降**を使用して `force_size` オプションを含めます。

    ```bash
    qemu-img convert -f raw -o subformat=fixed,force_size -O vpc rhel-7.4.raw rhel-7.4.vhd
    ```

## <a name="prepare-a-red-hat-based-vm-from-vmware"></a>VMware からの Red Hat ベース VM の準備

このセクションは、VMware に RHEL VM が既にインストールされていることを前提としています。 VMware にオペレーティング システムをインストールする方法の詳細については、[VMware のゲスト オペレーティング システムのインストール ガイド](https://aka.ms/aa6z600)を参照してください。

* Linux オペレーティング システムをインストールする場合は、LVM (通常、多くのインストールで既定) ではなく標準パーティションを使用することをお勧めします。 この方法により、特にオペレーティング システム ディスクをトラブルシューティングのために別の VM に接続する必要がある場合に、LVM 名と複製された VM の競合が回避されます。 必要な場合は、LVM または RAID をデータ ディスク上で使用できます。
* オペレーティング システム ディスクでスワップ パーティションを構成しないでください。 一時的なリソース ディスク上にスワップ ファイルを作成するよう Linux エージェントを構成できます。 この構成の詳細については、以下の手順を参照してください。
* 仮想ハード ディスクを作成する場合は、 **[Store virtual disk as a single file (仮想ディスクを 1 つのファイルとして格納する)]** を選択します。

### <a name="prepare-an-rhel-7-vm-from-vmware"></a>VMware からの RHEL 7 VM の準備

1. `/etc/sysconfig/network` ファイルを作成または編集して次のテキストを追加します。

    ```shell
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

1. `/etc/sysconfig/network-scripts/ifcfg-eth0` ファイルを作成または編集して次のテキストを追加します。

    ```shell
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    NM_CONTROLLED=no
    ```

1. 次のコマンドを実行して、起動時にネットワーク サービスが開始されるようにします。

    ```bash
    sudo chkconfig network on
    ```

1. RHEL リポジトリからパッケージをインストールできるように、次のコマンドを実行して Red Hat のサブスクリプションを登録します。

    ```bash
    sudo subscription-manager register --auto-attach --username=XXX --password=XXX
    ```

1. GRUB 構成でカーネルのブート行を変更して Azure の追加のカーネル パラメーターを含めます。 この変更を行うには、テキスト エディターで `/etc/default/grub` を開きます。 `GRUB_CMDLINE_LINUX` パラメーターを変更します。 次に例を示します。

    ```shell
    GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
    ```

    この構成により、すべてのコンソール メッセージが最初のシリアル ポートに送信され、メッセージを Azure での問題のデバッグに利用できるようになります。 NIC の新しい RHEL 7 名前付け規則もオフになります。 次のパラメーターを削除することをお勧めします。

    ```shell
    rhgb quiet crashkernel=auto
    ```

    クラウド環境では、すべてのログをシリアル ポートに送信するため、グラフィカル ブートおよびクワイエット ブートは役立ちません。 `crashkernel` オプションの構成は、必要であればそのままにしてかまいません。 ただし、このパラメーターにより、VM 内の使用可能なメモリ量が 128 MB 以上減少します。VM のサイズが小さいと、このことが問題になる可能性があります。

1. `/etc/default/grub` の編集を終了したら、次のコマンドを実行して GRUB 構成を再構築します。

    ```bash
    sudo grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

1. Hyper-V モジュールを initramfs に追加します。

    `/etc/dracut.conf`を編集して、コンテンツを追加します。

    ```shell
    add_drivers+="hv_vmbus hv_netvsc hv_storvsc"
    ```

    initramfs を再構築します。

    ```bash
    dracut -f -v
    ```

1. [1910 リリースより後のオプション] cloud-init を停止してアンインストールします。

    ```bash
    systemctl stop cloud-init
    yum remove cloud-init
    ```

1. SSH サーバーがインストールされており、起動時に開始するように構成されていることを確認します。 通常これが既定の設定です。 `/etc/ssh/sshd_config` を変更して、次の行を含めます。

    ```shell
    ClientAliveInterval 180
    ```

1. Azure Stack Hub 用のカスタム VHD を作成する際には、バージョンが 2.2.20 と 2.2.35 の間 (両バージョンは除く) の WALinuxAgent は、1910 リリースより前の Azure Stack Hub 環境では機能しないことに留意してください。 バージョン 2.2.20/2.2.35 を使用してイメージを準備できます。 2\.2.35 より後のバージョンを使用してカスタム イメージを準備するには、Azure Stack Hub を 1903 リリース以降に更新するか、1901 および 1902 修正プログラムを適用します。

    [1910 リリースより前] 次の手順に従って、互換性のある WALinuxAgent をダウンロードします。

    1. setuptools をダウンロードします。
    
        ```bash
        wget https://pypi.python.org/packages/source/s/setuptools/setuptools-7.0.tar.gz --no-check-certificate
        tar xzf setuptools-7.0.tar.gz
        cd setuptools-7.0
        ```
        
    1. 2\.2.20 バージョンのエージェントを GitHub からダウンロードして解凍します。
        
        ```bash
        wget https://github.com/Azure/WALinuxAgent/archive/v2.2.20.zip
        unzip v2.2.20.zip
        cd WALinuxAgent-2.2.20
        ```
        
    1. setup.py をインストールします。
        
        ```bash
        sudo python setup.py install
        ```
        
    1. waagent を再起動します。
        
        ```bash
        sudo systemctl restart waagent
        ```
        
    1. エージェントのバージョンがダウンロードしたものと一致するかどうかをテストします。 この例では、2.2.20 となります。
        
        ```bash
        waagent -version
        ```
        
    [1910 リリースより後] 次の手順に従って、互換性のある WALinuxAgent をダウンロードします。
    
    1. WALinuxAgent パッケージ `WALinuxAgent-<version>` が Red Hat extras リポジトリにプッシュされました。 次のコマンドを実行して extras リポジトリを有効にします。

        ```bash
        subscription-manager repos --enable=rhel-7-server-extras-rpms
        ```

    1. 次のコマンドを実行して Azure Linux エージェントをインストールします。
    
        ```bash
        sudo yum install WALinuxAgent
        sudo systemctl enable waagent.service
        ```
git        
1. オペレーティング システム ディスクにスワップ領域を作成しないでください。

    Azure Linux エージェントは、VM が Azure にプロビジョニングされた後に VM に接続されたローカルのリソース ディスクを使用してスワップ領域を自動的に構成できます。 ローカル リソース ディスクは一時ディスクであるため、VM のプロビジョニングが解除されると空になることに注意してください。 前の手順で Azure Linux エージェントのインストール後に、`/etc/waagent.conf` にある次のパラメーターを適切に変更します。

    ```shell
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    NOTE: set this to whatever you need it to be.
    ```

1. サブスクリプションを登録解除する場合は、次のコマンドを実行します。

    ```bash
    sudo subscription-manager unregister
    ```

1. エンタープライズ証明機関を使用してデプロイされたシステムを使用している場合、RHEL VM は Azure Stack Hub のルート証明書を信頼しません。 これは、信頼されたルート ストアに配置する必要があります。 詳細については、「[信頼できるルート証明書のサーバーへの追加](https://manuals.gfi.com/en/kerio/connect/content/server-configuration/ssl-certificates/adding-trusted-root-certificates-to-the-server-1605.html)」を参照してください。

1. 次のコマンドを実行して VM をプロビジョニング解除し、Azure でのプロビジョニング用に準備します。

    ```bash
    sudo waagent -force -deprovision
    export HISTSIZE=0
    logout
    ```

1. VM をシャットダウンし、VMDK ファイルを VHD 形式に変換します。

    > [!NOTE]
    > qemu-img のバージョン 2.2.1 以降には VHD が適切にフォーマットされないというバグがあることがわかっています。 この問題は QEMU 2.6 で修正されています。 qemu-img 2.2.0 以前を使用するか、2.6 以降に更新することをお勧めします。

    まず、イメージを未加工の形式に変換します。

    ```bash
    qemu-img convert -f qcow2 -O raw rhel-7.4.qcow2 rhel-7.4.raw
    ```

    未加工のイメージのサイズが 1 MB になっていることを確認します。 そうでない場合は、1 MB になるようにサイズの端数を切り上げます。

    ```bash
    MB=$((1024*1024))
    size=$(qemu-img info -f raw --output json "rhel-7.4.raw" | \
    gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')
    rounded_size=$((($size/$MB + 1)*$MB))
    qemu-img resize rhel-7.4.raw $rounded_size
    ```

    未フォーマット ディスクを固定サイズの VHD に変換します。

    ```bash
    qemu-img convert -f raw -o subformat=fixed -O vpc rhel-7.4.raw rhel-7.4.vhd
    ```

    または、qemu バージョン **2.6 以降**を使用して `force_size` オプションを含めます。

    ```bash
    qemu-img convert -f raw -o subformat=fixed,force_size -O vpc rhel-7.4.raw rhel-7.4.vhd
    ```

## <a name="prepare-a-red-hat-based-vm-from-an-iso-by-using-a-kickstart-file-automatically"></a>kickstart ファイルを使用して ISO から Red Hat ベースの VM を自動的に準備する

1. 次の内容を含んだ kickstart ファイルを作成し、そのファイルを保存します。 cloud init の停止とアンインストールは省略可能です (cloud init は Azure Stack Hub 1910 リリースより後でサポートされています)。 1910 リリースより後の場合のみ、redhat リポジトリからエージェントをインストールします。 1910 より前では、前のセクションで行ったように Azure リポジトリを使用します。 kickstart のインストールの詳細については、 [kickstart インストール ガイド](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/chap-kickstart-installations.html)を参照してください。

    ```shell
    Kickstart for provisioning a RHEL 7 Azure VM

    System authorization information
    auth --enableshadow --passalgo=sha512

    Use graphical install
    text

    Do not run the Setup Agent on first boot
    firstboot --disable

    Keyboard layouts
    keyboard --vckeymap=us --xlayouts='us'

    System language
    lang en_US.UTF-8

    Network information
    network  --bootproto=dhcp

    Root password
    rootpw --plaintext "to_be_disabled"

    System services
    services --enabled="sshd,waagent,NetworkManager"

    System timezone
    timezone Etc/UTC --isUtc --ntpservers 0.rhel.pool.ntp.org,1.rhel.pool.ntp.org,2.rhel.pool.ntp.org,3.rhel.pool.ntp.org

    Partition clearing information
    clearpart --all --initlabel

    Clear the MBR
    zerombr

    Disk partitioning information
    part /boot --fstype="xfs" --size=500
    part / --fstyp="xfs" --size=1 --grow --asprimary

    System bootloader configuration
    bootloader --location=mbr

    Firewall configuration
    firewall --disabled

    Enable SELinux
    selinux --enforcing

    Don't configure X
    skipx

    Power down the machine after install
    poweroff

    %packages
    @base
    @console-internet
    chrony
    sudo
    parted
    -dracut-config-rescue

    %end

    %post --log=/var/log/anaconda/post-install.log

    #!/bin/bash

    Register Red Hat Subscription
    subscription-manager register --username=XXX --password=XXX --auto-attach --force

    Install latest repo update
    yum update -y

    Stop and Uninstall cloud-init
    systemctl stop cloud-init
    yum remove cloud-init

    Enable extras repo
    subscription-manager repos --enable=rhel-7-server-extras-rpms

    Install WALinuxAgent
    yum install -y WALinuxAgent

    Unregister Red Hat subscription
    subscription-manager unregister

    Enable waaagent at boot-up
    systemctl enable waagent

    Disable the root account
    usermod root -p '!!'

    Configure swap in WALinuxAgent
    sed -i 's/^\(ResourceDisk\.EnableSwap\)=[Nn]$/\1=y/g' /etc/waagent.conf
    sed -i 's/^\(ResourceDisk\.SwapSizeMB\)=[0-9]*$/\1=2048/g' /etc/waagent.conf

    Set the cmdline
    sed -i 's/^\(GRUB_CMDLINE_LINUX\)=".*"$/\1="console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300"/g' /etc/default/grub

    Enable SSH keepalive
    sed -i 's/^#\(ClientAliveInterval\).*$/\1 180/g' /etc/ssh/sshd_config

    Build the grub cfg
    grub2-mkconfig -o /boot/grub2/grub.cfg

    Configure network
    cat << EOF > /etc/sysconfig/network-scripts/ifcfg-eth0
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    NM_CONTROLLED=no
    EOF

    Deprovision and prepare for Azure
    waagent -force -deprovision

    %end
    ```

1. インストール システムからアクセスできる場所に kickstart ファイルを置きます。

1. Hyper-V マネージャーで新しい VM を作成します。 **[仮想ハード ディスクの接続]** ページで、 **[後で仮想ハード ディスクを接続する]** を選択し、仮想マシンの新規作成ウィザードを完了します。

1. VM 設定を開きます。

    a. 新しい仮想ハード ディスクを VM に接続します。 **[VHD 形式]** と **[固定サイズ]** を選択します。

    b. インストール ISO を DVD ドライブに接続します。

    c. CD から起動するように BIOS を設定します。

1. VM を起動します。 インストール ガイドが表示されたら、 **Tab** キーを押してブート オプションを構成します。

1. ブート オプションの最後に `inst.ks=<the location of the kickstart file>` を入力し、 **Enter**キーを押します。

1. インストールが完了するのを待ちます。 完了すると、VM は自動的にシャットダウンされます。 これで、Linux VHD を Azure にアップロードする準備が整いました。

## <a name="known-issues"></a>既知の問題

### <a name="the-hyper-v-driver-couldnt-be-included-in-the-initial-ram-disk-when-using-a-non-hyper-v-hypervisor"></a>非 Hyper-V ハイパーバイザーを使用しているとき、Hyper-V ドライバーを初期 RAM ディスクに含めることができない

一部のケースにおいて、Linux インストーラーは、Hyper-V 環境で実行されていることを Linux が検出しない限り、初期 RAM ディスク (initrd または initramfs) に Hyper-V 用のドライバーを含めないことがあります。

別の仮想化システム (Oracle VM VirtualBox、Xen Project など) を使用して Linux イメージを準備する場合は、少なくとも hv_vmbus と hv_storvsc のカーネル モジュールを初期 RAM ディスクで使用できるように initrd の再構築が必要になる場合があります。 これは少なくとも、アップストリームの Red Hat ディストリビューションに基づくシステムの既知の問題です。

この問題を解決するには、initramfs に Hyper-V モジュールを追加して再構築する必要があります。

`/etc/dracut.conf` を編集し、次の内容を追加します。

```shell
add_drivers+="hv_vmbus hv_netvsc hv_storvsc"
```

initramfs を再構築します。

```bash
dracut -f -v
```

詳細については、[initramfs の再構築](https://access.redhat.com/solutions/1958)に関するページを参照してください。

## <a name="next-steps"></a>次のステップ

これで、Red Hat Enterprise Linux 仮想ハード ディスクを使用して、Azure Stack Hub に新しい VM を作成する準備が整いました。 初めて Azure Stack Hub に VHD ファイルをアップロードする場合は、「[Marketplace アイテムを作成および発行する](azure-stack-create-and-publish-marketplace-item.md)」を参照してください。

Red Hat Enterprise Linux の実行が認定されているハイパーバイザーの詳細については、[Red Hat の Web サイト](https://access.redhat.com/certified-hypervisors)を参照してください。
