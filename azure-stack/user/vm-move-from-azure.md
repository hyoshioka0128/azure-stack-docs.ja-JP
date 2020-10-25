---
title: VM を Azure から Azure Stack Hub に移動する
description: VM を Azure から Azure Stack Hub に移動する方法について説明します。
author: mattbriggs
ms.topic: how-to
ms.date: 9/8/2020
ms.author: mabrigg
ms.reviewer: kivenkat
ms.lastreviewed: 9/8/2020
ms.openlocfilehash: c8c68a64f7a05e03c70e138cb4d8c95da7417ec9
ms.sourcegitcommit: 3e225b30a54159b6b8dbeb2f843a2e5a721b746e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/29/2020
ms.locfileid: "91519425"
---
# <a name="move-a-vm-from-azure-to-azure-stack-hub"></a>VM を Azure から Azure Stack Hub に移動する

Azure で作成された仮想マシン (VM) から Azure Stack Hub インスタンスに仮想ハード ドライブ (VHD) をアップロードできます。

## <a name="prepare-and-download-your-vhd-from-azure"></a>Azure から VHD を準備してダウンロードする

VHD を準備するときに、ニーズに合致するセクションを見つけます。

#### <a name="windows---specialized"></a>[Windows - 特殊化](#tab/win-spec)

- 記事「[PowerShell を使用して特殊化されたディスクから Windows VM を作成する](/azure/virtual-machines/windows/create-vm-specialized#prepare-the-vm)」の手順に従い VHD を準備します。
- VM 拡張機能をデプロイするために、VM エージェント .msi が使用可能であることを確認します。  
  詳細および手順については、「[Azure 仮想マシン エージェントの概要](/azure/virtual-machines/extensions/agent-windows)」を参照してください。 VM を移動する前に、VM に拡張機能がインストールされていることを確認します。 VM エージェントが VHD に存在しない場合、拡張機能のデプロイは失敗します。 プロビジョニング中に OS プロファイルを設定したり、`$vm.OSProfile.AllowExtensionOperations = $true` を設定したりする必要はありません。

#### <a name="windows---generalized"></a>[Windows - 汎用化](#tab/win-gen)

::: moniker range="<=azs-1910"
- VHD を Azure Stack Hub に移動する前に、「[Azure から Windows VHD をダウンロードする](/azure/virtual-machines/windows/download-vhd)」の手順に従い VHD を正しく汎用化してダウンロードします。
- Azure で VM をプロビジョニングする場合は、PowerShell を使用します。 `-ProvisionVMAgent` フラグを指定せずに準備します。
- Azure で VM を汎用化する前に、VM から **Remove-AzureRmVMExtension** コマンドレットを使用してすべての VM 拡張機能を削除します。 `Windows (C:) > WindowsAzure > Logs > Plugins` に移動することにより、どの VM 拡張機能がインストールされているかを見つけることができます。

```powershell  
Remove-AzureRmVMExtension -ResourceGroupName winvmrg1 -VMName windowsvm -Name "CustomScriptExtension"
```
::: moniker-end
::: moniker range=">=azs-2002"

VHD を Azure Stack Hub に移動する前に、「[Azure から Windows VHD をダウンロードする](/azure/virtual-machines/windows/download-vhd)」の手順に従い VHD を正しく汎用化してダウンロードします。
::: moniker-end

#### <a name="linux---specialized"></a>[Linux - 特殊化](#tab/lin-spec)

- Linux VM をダウンロードする前に、記事「[Azure CLI を使用してカスタム ディスクから Linux VM を作成する](/azure/virtual-machines/linux/upload-vhd#prepare-the-vm)」の「VM を準備する」セクションのガイダンスに従います。
- 記事「[Azure から Linux VHD をダウンロードする](/azure//virtual-machines/windows/download-vhd)」の手順に従い VHD を準備してダウンロードします。
- 特殊化 VHD の場合は、`-CreateOption Attach` を使用した "アタッチ" セマンティクスを必ず使用してください。 記事「[PowerShell で既存のマネージド OS ディスクを使用して仮想マシンを作成する (Windows)](/azure/virtual-machines/scripts/virtual-machines-powershell-sample-create-vm-from-managed-os-disks)」で例を確認できます。

#### <a name="linux---generalized"></a>[Linux - 汎用化](#tab/lin-gen)

1. **waagent** サービスを停止します。

   ```bash
   sudo waagent -force -deprovision
   export HISTSIZE=0
   logout
   ```

   Azure Stack Hub で動作する Azure Linux エージェントのバージョンは[こちらに記載されています](../operator/azure-stack-linux.md#azure-linux-agent)。 sysprep を実行したイメージに、Azure Stack Hub と互換性のある Azure Linux エージェントのバージョンが含まれていることを確認します。

2. VM を停止 - 割り当て解除します。

3. VHD をダウンロードします。

   1. VHD ファイルをダウンロードするには、共有アクセス署名 (SAS) URL を生成します。 URL が生成されると、その URL に有効期限が割り当てられます。

   1. VM についてのブレードのメニューで、 **[ディスク]** を選択します。

   1. VM 用のオペレーティング システム ディスクを選択して、 **[ディスクのエクスポート]** を選択します。

   1. URL の有効期限を 36000 に設定します。

   1. **[URL の生成]** を選択します。

   1. URL を生成します。

   1. 生成された URL の下にある **[VHD ファイルのダウンロード]** を選択します。

   1. ダウンロードを開始するのに、ブラウザーで **[保存]** を選択する必要がある場合があります。 VHD ファイルの既定の名前は **abcd** です。

   1. この VHD を Azure Stack Hub に移動できるようになりました。

> [!IMPORTANT]  
> 「[VHD を Azure にアップロードし新しい VM を作成するサンプル スクリプト](/azure/virtual-machines/scripts/virtual-machines-windows-powershell-upload-generalized-script)」にあるスクリプトを使用して、Azure Stack Hub ユーザー ストレージ アカウントに VHD をアップロードして VM を作成できます。 Azure Stack Hub ストレージ アカウントとコンテナーの URL として `$urlOfUploadedImageVhd` を指定してください。 汎用化 VHD の場合は、`-CreateOption FromImage` を設定するときに `FromImage` 値を使用するようにしてください。

---

## <a name="verify-your-vhd"></a>VHD を検証する

[!INCLUDE [Verify VHD](../includes/user-compute-verify-vhd.md)]

## <a name="upload-to-a-storage-account"></a>ストレージ アカウントにアップロードする

[!INCLUDE [Upload to a storage account](../includes/user-compute-upload-vhd.md)]

## <a name="create-the-vm"></a>VM の作成

カスタム イメージには、**特殊化**と**汎用化**の 2 つの形式があります。

### <a name="specialized"></a>[専用イメージ](#tab/create-vm-spec)

[!INCLUDE [Create the disk in Azure Stack Hub](../includes/user-compute-create-disk.md)]

### <a name="generalized"></a>[一般化されたイメージ](#tab/create-vm-gen)

[!INCLUDE [Create the image in Azure Stack Hub](../includes/user-compute-create-image.md)]
---
## <a name="next-steps"></a>次のステップ

[Azure Stack Hub への VM の移動の概要](vm-move-overview.md)
