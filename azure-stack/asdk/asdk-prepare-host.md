---
title: ASDK ホスト コンピューターの準備
description: ASDK のインストール用に Azure Stack Development Kit (ASDK) ホスト コンピューターを準備する方法について説明します。
author: PatAltimore
ms.topic: article
ms.date: 08/28/2019
ms.author: patricka
ms.reviewer: misainat
ms.lastreviewed: 08/28/2019
ms.openlocfilehash: 5745d7efc5a953abaaa4e1d878677131c1c6e709
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97873335"
---
# <a name="prepare-the-asdk-host-computer"></a>ASDK ホスト コンピューターの準備
Azure Stack Development Kit (ASDK) をホスト コンピューターにインストールするには、ASDK ホストがインストール用に準備されている必要があります。 ホストが準備されると、CloudBuilder.vhdx 仮想マシン (VM) のハード ドライブから起動して ASDK のデプロイが開始されます。

## <a name="prepare-the-development-kit-host-computer"></a>開発キットのホスト コンピューターを準備する
ASDK をホスト コンピューターにインストールする前に、ASDK ホスト コンピューター環境を準備する必要があります。 その環境を準備するには、次の手順に従います。

1. ASDK ホスト コンピューターに、ローカル管理者としてサインインします。
2. CloudBuilder.vhdx ファイルが C:\ ドライブのルート (`C:\CloudBuilder.vhdx`) に移動されていることを確認します。
3. 次のスクリプトを実行して、ASDK インストーラー ファイル (asdk-installer.ps1) を [Azure Stack の GitHub ツール リポジトリ](https://github.com/Azure/AzureStack-Tools)から ASDK ホスト コンピューターの **C:\AzureStack_Installer** フォルダーにダウンロードします。

   > [!IMPORTANT]
   > ASDK をインストールするたびに、必ず asdk-installer.ps1 ファイルをダウンロードしてください。 このスクリプトは頻繁に変更されるため、各 ASDK デプロイに対して最新バージョンを使用する必要があります。 古いバージョンのスクリプトは最新リリースでは動作しない可能性があります。

   ```powershell
   # Variables
   $Uri = 'https://raw.githubusercontent.com/Azure/AzureStack-Tools/master/Deployment/asdk-installer.ps1'
   $LocalPath = 'C:\AzureStack_Installer'
   # Create folder
   New-Item $LocalPath -Type directory
   # Enforce usage of TLSv1.2 to download from GitHub
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   # Download file
   Invoke-WebRequest $uri -OutFile ($LocalPath + '\' + 'asdk-installer.ps1')
   ```

4. 管理者特権で PowerShell コンソールから **C:\AzureStack_Installer\asdk-installer.ps1** スクリプトを開始し、 **[Prepare Environment]\(環境の準備\)** をクリックします。

    ![ASDK で環境を準備する](media/asdk-prepare-host/1.PNG) 

5. インストーラーの **[Select Cloudbuilder vhdx]\(Cloudbuilder vhdx の選択\)** ページで、[前の手順](asdk-download.md)でダウンロードおよび展開した **cloudbuilder.vhdx** ファイルを探して選択します。 このページでは、ASDK キットのホスト コンピューターに他のドライバーを追加する必要がある場合に、 **[ドライバーの追加]** チェック ボックスをオンにすることもできます。 **[次へ]** をクリックします。  

    ![cloudbuilder.vhdx を選択して、ドライバーを ASDK に追加する](media/asdk-prepare-host/2.PNG)

6. **[オプション設定]** ページで、ASDK ホスト コンピューターのローカル管理者アカウント情報を指定し、 **[次へ]** をクリックします。

    この手順でローカル管理者の資格情報を指定しない場合は、ASDK のセットアップの一環としてコンピューターを再起動した後、ホストに直接または KVM でアクセスする必要があります。

   ![ASDK のオプション設定 - ローカルの管理者アカウント情報を指定する](media/asdk-prepare-host/3.PNG)

    次のオプションの設定の値を指定することもできます。
    - **[コンピューター名]** : このオプションでは、ASDK ホストの名前を設定します。 名前は FQDN の要件に準拠している必要があり、長さは 15 文字以下にする必要があります。 既定は、Windows によって生成されたランダムなコンピューター名です。

        - ネットワーク アダプターを選択します。 **[次へ]** をクリックする前に、アダプターに接続できることを確認します。

            ![ネットワーク アダプターの設定のスクリーンショット](media/asdk-prepare-host/step-four-network-adapter.png)

        - 表示されている **[IP アドレス]** 、 **[ゲートウェイ]** 、および **[DNS]** 値が正しいことを確認し、有効な **時刻サーバーの IP** アドレスを入力して、 **[次へ]** をクリックします。

            >[!TIP]
            >タイム サーバーの IP アドレスを検索するには、[ntppool.org](https://www.ntppool.org/) にアクセスするか、time.windows.com に ping を実行します。 

            ![IP 構成設定のスクリーンショット](media/asdk-prepare-host/step-five-host-ip-config.png)

7. **[次へ]** をクリックして、準備プロセスを開始します。
8. 準備が **[完了]** と示されたら、 **[次へ]** をクリックします。

    ![ASDK での環境の準備](media/asdk-prepare-host/4.PNG)

9. **[今すぐ再起動する]** をクリックして ASDK ホスト コンピューターを cloudbuilder.vhdx から起動し、[デプロイ プロセスを続行](asdk-install.md)します。

    ![ASDK を再起動する](media/asdk-prepare-host/5.PNG)


## <a name="next-steps"></a>次のステップ
[ASDK のインストール](asdk-install.md)
