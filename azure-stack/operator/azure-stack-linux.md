---
title: Linux イメージを Azure Stack Hub Marketplace に追加する
description: Azure Stack Hub Marketplace へ Linux イメージを追加する方法について説明します。
author: sethmanheim
ms.topic: article
ms.date: 11/18/2020
ms.author: sethm
ms.reviewer: thoroet
ms.lastreviewed: 11/18/2020
ms.openlocfilehash: 5fc9d8ba2cc12ddbb46156e091227ab2f47e0bd4
ms.sourcegitcommit: 8c745b205ea5a7a82b73b7a9daf1a7880fd1bee9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/24/2020
ms.locfileid: "95517635"
---
# <a name="add-linux-images-to-the-azure-stack-hub-marketplace"></a>Linux イメージを Azure Stack Hub Marketplace に追加する

Azure Stack Hub Marketplace に Linux ベースのイメージを追加することによって、Azure Stack Hub 上に Linux 仮想マシン (VM) をデプロイできます。 最も簡単に Linux イメージを Azure Stack Hub に追加する方法では、Marketplace Management を使用します。 これらのイメージは、Azure Stack Hub との互換性を確保できるよう、あらかじめ準備とテストが行われています。

## <a name="marketplace-management"></a>Marketplace Management

Azure Marketplace から Linux イメージをダウンロードするには、[Azure から Azure Stack Hub への Marketplace 項目のダウンロード](azure-stack-download-azure-marketplace-item.md)に関する記事を参照してください。 Azure Stack Hub で、ユーザーに提供する Linux イメージを選択します。

これらのイメージは頻繁に更新されるので、こまめに確認して最新の状態に保つようにしてください。

## <a name="prepare-your-own-image"></a>独自のイメージを準備する

可能な限り、使用可能なイメージは、Marketplace Management からダウンロードしてください。 これらのイメージは、Azure Stack Hub 用にあらかじめ準備とテストが行われています。

### <a name="azure-linux-agent"></a>Azure Linux エージェント

Azure Linux エージェント (一般に **WALinuxAgent** または **walinuxagent** と呼ばれています) が必要であり、エージェントのバージョンによっては Azure Stack Hub 上で動作しないものがあります。 2\.2.21 から 2.2.34 までのバージョン (2.2.21 と 2.2.34 を含みます) は、Azure Stack Hub ではサポートされません。 バージョン 2.2.35 より後の最新のエージェントを使用する場合は、1901 または 1902 の修正プログラムを適用するか、Azure Stack Hub を 1903 リリース (またはそれ以降) に更新してください。 [cloud-init](https://cloud-init.io/) は、Azure Stack Hub の 1910 より後のリリースでサポートされることに注意してください。

| Azure Stack Hub のビルド | Azure Linux エージェントのビルド |
| ------------- | ------------- |
| 1.1901.0.99 以前 | 2.2.20 |
| 1.1902.0.69  | 2.2.20  |
|  1.1901.3.105   | 2.2.35 以降 |
| 1.1902.2.73  | 2.2.35 以降 |
| 1.1903.0.35  | 2.2.35 以降 |
| 1903 移行のビルド | 2.2.35 以降 |
| サポートされていません | 2.2.21-2.2.34 |
| 1910 移行のビルド | Azure WALA エージェントのすべてのバージョン|

次の手順を使って、独自の Linux イメージを準備できます。

* [CentOS ベースのディストリビューション](/azure/virtual-machines/linux/create-upload-centos?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Debian Linux](/azure/virtual-machines/linux/debian-create-upload-vhd?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Red Hat Enterprise Linux](azure-stack-redhat-create-upload-vhd.md)
* [SLES と openSUSE](/azure/virtual-machines/linux/suse-create-upload-vhd?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Ubuntu Server](/azure/virtual-machines/linux/create-upload-ubuntu?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

## <a name="cloud-init"></a>cloud-init

[Cloud-init](https://cloud-init.io/) は、Azure Stack Hub の 1910 より後のリリースでサポートされます。 cloud-init を使用して Linux VM をカスタマイズするには、次の PowerShell の手順に従います。

### <a name="step-1-create-a-cloud-inittxt-file-with-your-cloud-config"></a>手順 1:cloud-config を使用して cloud-init.txt ファイルを作成する

cloud-init.txt というファイルを作成し、次の構成を貼り付けます。

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - nodejs
  - npm
write_files:
  - owner: www-data:www-data
    path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 80;
        location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection keep-alive;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }
      }
  - owner: azureuser:azureuser
    path: /home/azureuser/myapp/index.js
    content: |
      var express = require('express')
      var app = express()
      var os = require('os');
      app.get('/', function (req, res) {
        res.send('Hello World from host ' + os.hostname() + '!')
      })
      app.listen(3000, function () {
        console.log('Hello world app listening on port 3000!')
      })
runcmd:
  - service nginx restart
  - cd "/home/azureuser/myapp"
  - npm init
  - npm install express -y
  - nodejs index.js
  ```
  
### <a name="step-2-reference-cloud-inittxt-during-the-linux-vm-deployment"></a>手順 2:Linux VM のデプロイ中に cloud-init.txt を参照する

Azure Stack Hub Linux VM からアクセスできる Azure ストレージ アカウント、Azure Stack Hub ストレージ アカウント、または GitHub リポジトリにファイルをアップロードします。
現時点では、VM のデプロイでの cloud-init の使用は、REST、PowerShell、CLI でのみサポートされており、Azure Stack Hub に関連付けられたポータル UI はありません。

[こちら](../user/azure-stack-quick-create-vm-linux-powershell.md)の手順に従って、PowerShell を使用して Linux VM を作成できますが、`-CustomData` フラグの一部として cloud-init.txt を必ず参照してください。

### <a name="az-modules"></a>[Az モジュール](#tab/az)

```powershell
$VirtualMachine =Set-AzVMOperatingSystem -VM $VirtualMachine `
  -Linux `
  -ComputerName "MainComputer" `
  -Credential $cred -CustomData "#include https://cloudinitstrg.blob.core.windows.net/strg/cloud-init.txt"
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm)

```powershell
$VirtualMachine =Set-AzureRMVMOperatingSystem -VM $VirtualMachine `
  -Linux `
  -ComputerName "MainComputer" `
  -Credential $cred -CustomData "#include https://cloudinitstrg.blob.core.windows.net/strg/cloud-init.txt"
```
---

## <a name="add-your-image-to-marketplace"></a>Marketplace にイメージを追加する

[Marketplace へのイメージの追加](azure-stack-add-vm-image.md)に関するページに従ってください。 `OSType` パラメーターが `Linux` に設定されていることを確認してください。

Marketplace にイメージを追加した後は、Marketplace の項目が作成され、ユーザーが Linux VM をデプロイできます。

## <a name="next-steps"></a>次のステップ

* [Azure から Azure Stack Hub に Marketplace の項目をダウンロードする](azure-stack-download-azure-marketplace-item.md)
* [Azure Stack Hub Marketplace の概要](azure-stack-marketplace.md)
