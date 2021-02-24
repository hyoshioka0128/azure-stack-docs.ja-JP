---
title: サービスとしての検証のリリース ノート
titleSuffix: Azure Stack Hub
description: Azure Stack Hub のサービスとしての検証のリリース ノート。
author: mattbriggs
ms.topic: article
ms.date: 12/16/2020
ms.author: mabrigg
ms.reviewer: johnhas
ms.lastreviewed: 10/28/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 82e7820b49578139bbfb140e98fd740e6c1be8fb
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97868201"
---
# <a name="release-notes-for-validation-as-a-service"></a>サービスとしての検証のリリース ノート

[!INCLUDE [Azure_Stack_Partner](./includes/azure-stack-partner-appliesto.md)]

この記事には、Azure Stack Hub のサービスとしての検証のリリース ノートが含まれています。

## <a name="version-4432"></a>バージョン 4.4.3.2

2020 年 2 月 2 日

- サービス メンテナンスの更新。

- バグの修正
  - テスト Test101LinuxEmptyAttachedDiskManagedDisk、Test101WindowsEmptyAttachedDiskManagedDisk を修正しました。

## <a name="version-4442"></a>バージョン 4.4.4.2

2020 年 11 月 11 日

- CSE 検証ワークフローが更新され、AzureStack のフル アップデート後に、テスト署名済みの OEM 拡張パッケージを自動的にインストールできるようになりました。
  - この修正以前は、AzureStack のフル アップデート後に、VaaS を使用したテスト署名済みの OEM 拡張パッケージをスタンプにインストールする際に失敗することがありました。 VaaS によって AzureStack 更新プログラムが適用され、その後に実行が終了されます。
  - これはすでに修正済みで、CSE 検証ワークフローによって指定された AzureStack 更新プログラムとテスト署名済みの OEM 拡張パッケージがインストールされることを確認できます。
- OEM パッケージ検証拡張機能を 'OEM 検証ワークフロー' に追加しました
  - この拡張機能は、スタンプの更新が開始される前に実行されます。
  - この拡張機能によって、OEM 拡張機能パッケージの内容と oemMetadata.xml の要素が検証されます
  - OEM 拡張パッケージでエラーまたは問題が発生した場合は、VaaS テストが開始される前に検出されます。
  - これらの検証は、以前は VaaS のテスト実行後のパッケージの署名時に実行されていました。  
- 新しいバージョンの AzureStack および AzureRM PowerShell モジュールをインストールするように VaaS の前提要件が更新されました
  - AzureStack PS モジュール バージョン 1.8.2
  - AzureRM PS モジュール バージョン 2.5.0
- サービスのマイナー アップデート。

## <a name="version-443112"></a>バージョン 4.4.3.112

2020 年 8 月 23 日

- サービスの更新。
  - サービス展開の更新。
  - サービスの認証方法が更新されました。

## <a name="version-44368"></a>バージョン 4.4.3.68

2020 年 6 月 30 日

- サービスの更新。
  - Service Fabric で実行されるようにサービスを移動しました。

## <a name="version-4421"></a>バージョン 4.4.2.1

2020 年 1 月 9 日

- テスト コンテンツの更新:
  - OEM 検証ワークフロー (バージョン 5.1.52.0 -> 5.1.53.0):テスト スケジュール ペインの必要なパラメーター数を減らしました。
  - コンピューティング テスト - TestVMOperations のバグ修正

- 既知の問題:
  - OEM 検証ワークフロー中に次のテスト ケースを実行できない場合は、vaashelp@microsoft.com にお問い合わせください。
    - Test101LinuxEmptyAttachedDiskManagedDisk
    - Test101WindowsEmptyAttachedDiskManagedDisk

2019 年 12 月 3 日

- テスト コンテンツの更新:
  - 月次 Azure Stack Hub 更新プログラム ワークフローと OEM パッケージの検証ワークフローのオンライン ドキュメントが更新されました。 更新されたドキュメント「[OEM パッケージの検証](azure-stack-vaas-validate-oem-package.md)」と「[Microsoft のソフトウェア更新プログラムの検証](azure-stack-vaas-validate-microsoft-updates.md)」を確認してください。
  - VaaS パッケージ検証ワークフローの更新:月次 Azure Stack Hub 更新プログラムの検証と OEM パッケージの検証に必要なテストは、OEM 検証ワークフローだけです。 このテストでは、指定された AzureStack/OEM パッケージでスタンプを更新し、クラウド シミュレーション エンジンの検証テストを実行します。
  - VaaS PowerShell 拡張機能の更新:パッケージ検証ワークフローの自動化がサポートされるようになりました。 この拡張機能を使用する場所と具体的な手順について詳しくは、PowerShell を使用した Azure Stack Hub VaaS の自動化に関する記事を参照してください。

- 既知の問題:
  - OEM 検証ワークフロー中に次のテスト ケースを実行できない場合は、vaashelp@microsoft.com にお問い合わせください。
    - Test101LinuxEmptyAttachedDiskManagedDisk
    - Test101WindowsEmptyAttachedDiskManagedDisk

## <a name="version-4353"></a>バージョン 4.3.5.3

2019 月 11 月 7 日

- テスト コンテンツの更新:
  - Monthly Azure Stack Hub Update Verification (月次 Azure Stack Hub 更新プログラムの検証) (バージョン 5.1.46.0 -> 5.1.49.0)。
  - OEM Extension Package Verification (OEM 拡張機能パッケージの検証) (バージョン 5.1.46.0 -> 5.1.49.0)。
  - 5.1.46.0 の結果は保持されています。 5\.1.46.0 で正常に実行された場合は、結果を送信するときに vaashelp@microsoft.com に通知します。

- バグの修正
  - 更新プログラムの .zip に特殊文字が含まれていた場合に月次 Azure Stack Hub 更新プログラムの検証が実行に失敗する問題を修正しました。

- 既知の問題
  - mstest.exe が検出されない場合、VaaS テストが失敗します。 対応策 :
    1. PowerShell ウィンドウでエージェントの CTRL + C を実行します。
    1. 「mstest.exe」と入力して、mstest.exe が認識されるプログラムであることを確認します。
    1. mstest.exe が認識されない場合は、現在の PowerShell ウィンドウを閉じます。
    1. [スタート] (タスク バーの PowerShell ではない) をクリックし、PowerShell を見つけ、管理者として開きます。
    1. 「mstest.exe」と入力し、それがコマンドとして使用可能なことを確認します。
    1. エージェントを再起動し、テストを再実行します。
  - 場合によって、クラウド シミュレーション エンジンが \*vm テストでエラーを報告することがあります。 再実行を試みる前に vaashelp@microsoft.com にお問い合わせください。


2019 年 10 月 29 日

- 月次 Azure Stack Hub 更新プログラム ワークフローと OEM パッケージの検証ワークフローのオンライン ドキュメントが更新されました。

    更新されたドキュメント「[OEM パッケージの検証](azure-stack-vaas-validate-oem-package.md)」と「[Microsoft のソフトウェア更新プログラムの検証](azure-stack-vaas-validate-microsoft-updates.md)」を確認してください。
- VaaS ワークフローの更新:月次 Azure Stack Hub 更新プログラム (バージョン 5.1.30.0 -> 5.1.46.0) - 月次 Azure Stack Hub 更新プログラムの検証テスト フローが更新されました。

    ワークフローは手動での介入を必要とせず、シームレスに実行されるようにスケジュールできるようになりました。
- VaaS ワークフローの更新:OEM パッケージの検証  (バージョン 5.1.30.0 -> 5.1.46.0)  –  OEM パッケージの検証ワークフローが更新されました。

    ワークフローは手動での介入を必要とせず、シームレスに実行されるようにスケジュールできるようになりました。
- OEM パッケージの検証ワークフロー (バージョン 5.1.30.0 -> 5.1.46.0) のクラウド シミュレーション エンジンが更新され、検証時間が短縮されました。実行時間が 1 時間に短縮されました。
- OEM パッケージの検証ワークフローと Azure Stack Hub 更新プログラム ワークフロー (バージョン 5.1.30.0 -> 5.1.46.0) のクラウド シミュレーション エンジンでは、検証対象の更新プログラムは、子フォルダーに他の更新プログラムが含まれていない 2 つの別々の親フォルダーに入っている必要があります。
- OEM パッケージの検証ワークフローと Azure Stack Hub 更新プログラム ワークフロー (バージョン 5.1.30.0 -> 5.1.46.0) のクラウド シミュレーション エンジンでは、テストは、月次の Azure Stack Hub 更新プログラムの検証テスト、OEM 拡張機能パッケージの検証テスト、そして最後にクラウド シミュレーション エンジンの順序でスケジュールする必要があります。
- VaaS エージェントの更新:更新された VaaS エージェントは、Azure Stack Hub のクラウド管理者の資格情報を使ってスタンプのクエリを実行し、ワークフローに自動入力するためのスタンプ情報を取得するようになりました。

    この更新では、すべてのエージェントを更新して再起動する必要があります。 VaaS エージェントの更新方法に関する次の説明を参照してください: https://docs.microsoft.com/azure-stack/partner/azure-stack-vaas-local-agent
- VaaS ポータル UI の更新:テストを容易にするために、エージェント選択テーブルはテスト スケジュール ペインの上に移動されました。

    VaaS エージェントが正しく更新されていれば、ジョブをスケジュールするときに、スタンプ情報を入力する必要はなくなりました。

## <a name="version-405"></a>バージョン 4.0.5

2019 年 6 月 7 日

- パッケージの検証ワークフローのクラウド シミュレーション エンジンが更新され、検証時間が短縮されました。  
    実行時間: 6 時間に短縮  
    バージョン:5.1.13.0 -> 5.1.22.0  


2019 年 1 月 17 日

- Disk Identification Test (ディスク識別テスト) がストレージ プールの不整合を解決するために更新されました。 バージョン:5.1.14.0 -> 5.1.15.0
- Azure Stack Hub 月間更新検証が、承認済みソフトウェアおよびコンテンツの検証の不整合を解決するために更新されました。 バージョン:5.1.14.0 -> 5.1.17.0
- OEM Extension Package Verification (OEM 拡張機能パッケージの検証) が、Azure Stack Hub の更新手順の前に必要なチェックを実行するよう更新されました。 バージョン:5.1.14.0 -> 5.1.16.0
- 内部のバグ修正。

## <a name="version-403"></a>バージョン 4.0.3

2019 年 1 月 7 日

Azure Stack Hub 月間更新検証ワークフローを実行しているとき、ご利用の OEM 更新プログラム パッケージのバージョンが 1810 以降でない場合、OEM 更新手順に進んだときにエラーが表示されます。 このエラーはバグです。 修正プログラムが開発されています。 軽減手順は次のとおりです。

1. OEM の更新プログラムを通常どおり実行します。
2. パッケージが正常に適用されたら Test-AzureStack を実行し、出力を保存します。
3. テストを取り消します。
4. 保存した出力を VaaSHelp@microsoft.com に送信し、実行の合格結果を受け取ります。

## <a name="version-402"></a>バージョン 4.0.2

2018 年 11 月 30 日

- 内部のバグ修正。

## <a name="version-401"></a>バージョン 4.0.1

2018 年 10 月 8 日

- VaaS の前提条件:

    `Install-VaaSPrerequisites` に、クラウド管理者の資格情報は不要になりました。 このコマンドレットの最新バージョンを実行する場合、前提条件をインストールするための変更後のコマンドについては、「[ローカル エージェントのダウンロードとインストール](azure-stack-vaas-local-agent.md#download-and-install-the-local-agent)」を参照してください。 コマンドは次のとおりです。

    ```powershell
    $ServiceAdminCreds = New-Object System.Management.Automation.PSCredential "<aadServiceAdminUser>", (ConvertTo-SecureString "<aadServiceAdminPassword>" -AsPlainText -Force)
    Import-Module .\VaaSPreReqs.psm1 -Force
    Install-VaaSPrerequisites -AadTenantId $AadTenantId `
                              -ServiceAdminCreds $ServiceAdminCreds `
                              -ArmEndpoint https://adminmanagement.$ExternalFqdn `
                              -Region $Region
    ```

## <a name="version-400"></a>バージョン 4.0.0

2018 年 8 月 29 日

- VaaS の前提条件と VHD の更新:

    パッケージ検証時の問題に対処するために、`Install-VaaSPrerequisites` に、クラウド管理者の資格情報が必要になりました。 「[ローカル エージェントのダウンロードとインストール](azure-stack-vaas-local-agent.md#download-and-install-the-local-agent)」のドキュメントが次のコードで更新されました。

    ```powershell
    $ServiceAdminCreds = New-Object System.Management.Automation.PSCredential "<aadServiceAdminUser>", (ConvertTo-SecureString "<aadServiceAdminPassword>" -AsPlainText -Force)
    $CloudAdminCreds = New-Object System.Management.Automation.PSCredential "<cloudAdminDomain\username>", (ConvertTo-SecureString "<cloudAdminPassword>" -AsPlainText -Force)
    Import-Module .\VaaSPreReqs.psm1 -Force
    Install-VaaSPrerequisites -AadTenantId $AadTenantId `
                              -ServiceAdminCreds $ServiceAdminCreds `
                              -ArmEndpoint https://adminmanagement.$ExternalFqdn `
                              -Region $Region `
                              -CloudAdminCredentials $CloudAdminCreds
    ```
    > [!NOTE]
    > スクリプトで必要とされる `$CloudAdminCreds` は、検証対象の Azure Stack Hub インスタンス用です。 VaaS テナントが使用する Azure Active Directory の資格情報ではありません。

- ローカル エージェントの更新:

    以前のバージョンのローカル エージェントは、現在の 4.0.0 リリースのサービスと互換性がありません。 すべてのユーザーが、ローカル エージェントを更新する必要があります。 最新のエージェントをインストールする手順については、「[ローカル エージェントのダウンロードとインストール](azure-stack-vaas-local-agent.md#download-and-install-the-local-agent)」を参照してください。

- PowerShell オートメーションの更新:

    `LaunchVaaSTests` PowerShell スクリプトに、最新バージョンのスクリプト パッケージを必要とする変更が加えられました。 最新バージョンのスクリプト パッケージをインストールする手順については、「[テスト パス ワークフローの起動](azure-stack-vaas-automate-with-powershell.md)」をご覧ください。

- サービスとしての検証ポータル:

  - パッケージの署名の通知

    パッケージ検証ワークフローの一環として OEM カスタマイズ パッケージが提出されると、パッケージ形式が検証され、公開されている仕様に従っているかどうかが確認されます。 パッケージが準拠していない場合、実行は失敗します。 テナントの登録済みの Azure Active Directory 連絡先の電子メール アドレスに電子メール通知が送信されます。

  - 対話型テスト カテゴリ:

    **対話型** テスト カテゴリが追加されました。 これらのテストにより、自動化されていない対話型の Azure Stack Hub シナリオが実行されます。

  - 対話型の機能検証:

    テスト成功ワークフローで、特定の機能に焦点を合わせたフィードバックを提供する機能を使用できるようになりました。 `OEM Update on Azure Stack Hub 1806 RC Validation 5.1.4.0` テストでは、特定の更新プログラムが正しく適用されたかどうかを確認し、フィードバックを収集します。

## <a name="next-steps"></a>次のステップ

- [サービスとしての検証のトラブルシューティング](azure-stack-vaas-troubleshoot.md)
