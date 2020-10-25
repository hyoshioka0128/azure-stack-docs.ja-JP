---
title: 特殊化 VM をオンプレミスから Azure Stack Hub に移動する
description: 特殊化 VM をオンプレミスから Azure Stack Hub に移動する方法について説明します。
author: mattbriggs
ms.topic: how-to
ms.date: 9/8/2020
ms.author: mabrigg
ms.reviewer: kivenkat
ms.lastreviewed: 9/8/2020
ms.openlocfilehash: 383e96889f742b05a5f4b25e91bab48e1fd4c075
ms.sourcegitcommit: 9a340b383dcf42c85bc6ec0d01ff3c9ae29dfe4c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/09/2020
ms.locfileid: "89609930"
---
# <a name="move-a-specialized-vm-from-on-premises-to-azure-stack-hub"></a>特殊化 VM をオンプレミスから Azure Stack Hub に移動する

オンプレミス環境から仮想マシン (VM) イメージを追加できます。 イメージを仮想ハード ディスク (VHD) として作成し、そのイメージを Azure Stack Hub インスタンスのストレージ アカウントにアップロードできます。 その VHD から VM を作成できます。

特殊化ディスク イメージとは、既存の VM の仮想ハードディスク (VHD) のコピーで、元の VM のユーザー アカウント、アプリケーション、その他の状態データが含まれます。 これは通常、VM を Azure Stack Hub に移行するときに使用する形式です。 オンプレミスから Azure Stack Hub に VM を移行する必要がある場合は、特殊化 VHD が適しています。

## <a name="how-to-move-an-image"></a>イメージを移動する方法

VHD を準備するときに、ニーズに合致するセクションを見つけます。

#### <a name="windows-vm"></a>[Windows VM](#tab/port-win)

- 「[Azure にアップロードする Windows VHD または VHDX を準備する](/azure/virtual-machines/windows/prepare-for-upload-vhd-image)」の手順に従い、VHD を正しく準備します。 Azure Stack Hub には VHD を使用する必要があります。
   > [!NOTE]  
   > Sysprep を使用して VM を一般化**しないでください**。
- VM にインストールされたゲストの仮想化ツールやエージェント (VMware ツールなど) を削除します。
- IP アドレスと DNS 設定を DHCP から取得するように VM が構成されていることを確認します。 これにより、サーバーが起動時に仮想ネットワーク内の IP アドレスを確実に取得します。
- RDP/SSH が有効になっていること、およびファイアウォールで通信が許可されていることを確認します。
- VM 拡張機能をデプロイするには、VM エージェント `.msi` が使用可能であることを確認します。 ガイダンスについては、「[Azure 仮想マシン エージェントの概要](/azure/virtual-machines/extensions/agent-windows)」を参照してください。 VM エージェントが VHD に存在しない場合、拡張機能のデプロイは失敗します。 プロビジョニング中に OS プロファイルを設定したり、`$vm.OSProfile.AllowExtensionOperations = $true` を設定したりする必要はありません。

#### <a name="linux-vm"></a>[Linux VM](#tab/port-linux)

#### <a name="generalize-the-vhd"></a>VHD を汎用化する

適切な指示に従って、Linux OS の VHD を準備します。

- [CentOS ベースのディストリビューション](/azure/virtual-machines/linux/create-upload-centos?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Debian Linux](/azure/virtual-machines/linux/debian-create-upload-vhd?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Red Hat Enterprise Linux](../operator/azure-stack-redhat-create-upload-vhd.md)
- [SLES または openSUSE](/azure/virtual-machines/linux/suse-create-upload-vhd?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Ubuntu Server](/azure/virtual-machines/linux/create-upload-ubuntu?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

> [!IMPORTANT]
> VHD が汎用化されるため、最後の手順 (`sudo waagent -force -deprovision`) は実行しないでください。

#### <a name="identify-the-version-of-the-linux-agent"></a>Linux エージェントのバージョンを特定する

ソース VM イメージにインストールされている Linux エージェントのバージョンを特定するには、次のコマンドを実行します。 プロビジョニング コードを説明するバージョン番号は、`Goal state agent` ではなく `WALinuxAgent-` です。

   ```bash  
   waagent -version
   ```
    
   次に例を示します。
    
   ```bash  
   waagent -version
   WALinuxAgent-2.2.45 running on centos 7.7.1908
   Python: 2.7.5
   Goal state agent: 2.2.46
   ```

#### <a name="linux-agent-224-and-earlier-disable-the-linux-agent-provisioning"></a>Linux エージェント 2.2.4 以前で、Linux エージェントのプロビジョニングを無効にする 

2\.2.4 よりも前の Linux エージェントで Linux エージェントのプロビジョニングを無効にするには、 **/etc/waagent.conf** に次のパラメーターを設定します: `Provisioning.Enabled=n, and Provisioning.UseCloudInit=n`。

#### <a name="linux-agent-2245-and-later-disable-the-linux-agent-provisioning"></a>Linux エージェント 2.2.45 以降で、Linux エージェントのプロビジョニングを無効にする

2\.2.45 以降の Linux エージェントでプロビジョニングを無効にするには、次の構成オプションを変更します。

`Provisioning.Enabled` と `Provisioning.UseCloudInit` が無視されるようにする。

このバージョンでは現在、プロビジョニングを完全に無効にする `Provisioning.Agent` オプションはありません。ただし、プロビジョニング マーカー ファイルを追加することはでき、次の設定を行うと、プロビジョニングが無視されます。

1. **/etc/waagent.conf** で、次の構成オプションを追加します: `Provisioning.Agent=Auto`。
2. walinuxagent のプロビジョニングを確実に無効にするには、次を実行します: `mkdir -p /var/lib/waagent && touch /var/lib/waagent/provisioned`。
3. 次を実行して、クラウドの初期化のインストールを無効にします。

   ```bash  
   touch /etc/cloud/cloud-init.disabled
   sudo sed -i '/azure_resource/d' /etc/fstab
   ```

4. ログアウトします。

#### <a name="run-an-extension"></a>拡張機能を実行する

1. **/etc/waagent.conf** に次のパラメーターを設定します。

   - `Provisioning.Enabled=n`
   - `Provisioning.UseCloudInit=n`

2. walinuxagent のプロビジョニングを確実に無効にするには、次を実行します: `mkdir -p /var/lib/waagent && touch /var/lib/waagent/provisioned`

3. イメージに cloud-init がある場合は、クラウドの初期化を無効にします。

    ```bash  
   touch /etc/cloud/cloud-init.disabled
   sudo sed -i '/azure_resource/d' /etc/fstab
   ```

4. ログアウトを実行します。

---

## <a name="verify-your-vhd"></a>VHD を検証する

[!INCLUDE [Verify VHD](../includes/user-compute-verify-vhd.md)]

## <a name="upload-to-a-storage-account"></a>ストレージ アカウントにアップロードする

[!INCLUDE [Upload to a storage account](../includes/user-compute-upload-vhd.md)]

## <a name="create-the-disk-in-azure-stack-hub"></a>Azure Stack Hub でディスクを作成する

[!INCLUDE [Create the disk in Azure Stack Hub](../includes/user-compute-create-disk.md)]

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub への VM の移動の概要](vm-move-overview.md)
