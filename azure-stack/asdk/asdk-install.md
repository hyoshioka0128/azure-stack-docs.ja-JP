---
title: ASDK のインストール
description: Azure Stack Development Kit (ASDK) をインストールする方法について説明します。
author: PatAltimore
ms.topic: article
ms.date: 05/06/2019
ms.author: patricka
ms.reviewer: misainat
ms.lastreviewed: 02/08/2019
ms.openlocfilehash: fe485a3eebee3e44e19173fdf68440057de7d783
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97873420"
---
# <a name="install-the-asdk"></a>ASDK のインストール
[ASDK ホスト コンピューターの準備](asdk-prepare-host.md)ができたら、この記事の次の手順に従って、Azure Stack Development Kit (ASDK) を CloudBuilder.vhdx イメージにデプロイできます。

## <a name="install-the-asdk"></a>ASDK のインストール
この記事の手順は、**asdk-installer.ps1** PowerShell スクリプトをダウンロードして実行することによって提供されるグラフィカル ユーザー インターフェイス (GUI) を使って ASDK をデプロイする方法を示しています。

> [!NOTE]
> ASDK のインストーラー ユーザー インターフェイスは、WCF および PowerShell に基づくオープン ソースのスクリプトです。


1. ホスト コンピューターが CloudBuilder.vhdx イメージで正常に起動した後、ASDK インストール用に [ASDK ホスト コンピューターを準備](asdk-prepare-host.md)したときに指定した管理者資格情報を使ってサインインします。 これは、ASDK ホストのローカル管理者の資格情報と同じである必要があります。
2. 管理者特権の PowerShell コンソールを開き、 **&lt;ドライブ文字>\AzureStack_Installer\asdk-installer.ps1** PowerShell スクリプトを実行します。 このスクリプトが現在、CloudBuilder.vhdx イメージの C:\ とは別のドライブに存在する可能性があることに注意してください。 **[インストール]** をクリックします。

    ![ASDK のインストール](media/asdk-install/1.PNG) 

3. ID プロバイダーの **[種類]** ドロップダウン ボックスで、 **[Azure China Cloud]** 、 **[Azure US Government]** 、 **[AD FS]** 、または **[Azure Cloud]** を選択します。 **[ローカル管理者のパスワード]** の **[パスワード]** ボックスに、(現在の構成済みのローカル管理者パスワードと同じ) ローカル管理者パスワードを入力し、 **[次へ]** をクリックします。

    ![ASDK の [ID プロバイダーの種類] ドロップダウン](media/asdk-install/2.PNG) 
  
    Azure サブスクリプション ID プロバイダーを選択する場合は、インターネット接続、Azure AD ディレクトリ テナントのフルネーム (*domainname*.onmicrosoft.com の形式) または Azure AD の検証済みのカスタム ドメイン名が必要です。 また、指定されたディレクトリの全体管理者の資格情報も必要です。

    デプロイの後、Azure Active Directory (Azure AD) の全体管理者のアクセス許可は必要ありません。 ただし、一部の操作では、全体管理者の資格情報が必要な場合があります。 たとえば、リソース プロバイダーのインストーラー スクリプトや、アクセス許可を付与する必要のある新機能などがあります。 アカウントの全体管理者のアクセス許可を一時的に復元するか、*既定のプロバイダー サブスクリプション* の所有者である別の全体管理者アカウントを使用します。

    ID プロバイダーとして AD FS を使用している場合は、既定のスタンプ ディレクトリ サービスが使用されます。 サインインに使用する既定のアカウントは azurestackadmin@azurestack.local であり、使用するパスワードは、セットアップの一部として指定したものです。

   > [!NOTE]
   > AD FS を ID プロバイダーとして使って、インターネット接続されていない Azure Stack 環境を使う場合でも、最適な結果を得るには、インターネットへの接続中に ASDK をインストールすることが最良です。 そうすることにより、ASDK インストールに含まれている Windows Server 2016 の評価バージョンがデプロイ時にアクティブ化されます。

4. ASDK に使用するネットワーク アダプターを選択して、 **[次へ]** をクリックします。

    ![ASDK 用のネットワーク アダプターを選択する](media/asdk-install/3.PNG)

5. **[ネットワーク構成]** ページで、有効な **タイム サーバー IP** アドレスを指定します。 この必須フィールドでは、ASDK によって使われるタイム サーバーを設定します。 このパラメーターは、有効なタイム サーバーの IP アドレスとして指定する必要があります。 サーバー名はサポートされていません。

      > [!TIP]
      > タイム サーバーの IP アドレスを検索するには、[ntppool.org](https://www.ntppool.org/) にアクセスするか、time.windows.com に ping を実行します。 

    **オプションで**、**DNS フォワーダー** IP アドレスを指定できます。 DNS サーバーが Azure Stack のデプロイの一部として作成されます。 ソリューション内のコンピューターにスタンプ外の名前解決を許可するには、既存のインフラストラクチャの DNS サーバーを提供します。 スタンプ内の DNS サーバーが、このサーバーに不明な名前解決の要求を送信します。

    ![ASDK での DNS フォワーダーとネットワーク構成](media/asdk-install/4.PNG)

6. **[Verifying network interface card properties]\(ネットワーク インターフェイス カードのプロパティを確認しています\)** ページに、進行状況バーが表示されます。 検証が完了したら、 **[次へ]** をクリックします。

    ![ASDK でのネットワーク インターフェイス カードのプロパティの確認](media/asdk-install/5.PNG)

7. **[概要]** ページで、 **[Deploy]\(デプロイ\)** をクリックして、ASDK ホスト コンピューターへの ASDK のインストールを開始します。

    ![ASDK にデプロイする前のスクリプトの概要](media/asdk-install/6.PNG)

    > [!TIP]
    > ここで、ASDK のインストールに使用される PowerShell のセットアップ コマンドをコピーすることもできます。 これは、[PowerShell を使って ASDK をホスト コンピューターに再デプロイする](asdk-deploy-powershell.md)必要がある場合に役立ちます。

8. Azure AD デプロイを使用している場合、セットアップを開始して数分後に、Azure AD の全体管理者アカウントの資格情報を入力するように求めるメッセージが表示されます。

9. デプロイ処理には数時間かかります。その間に、ホスト コンピューターは自動的に 1 回再起動されます。 デプロイの進行状況を監視する場合は、ASDK のホストが再起動された後に、azurestack\AzureStackAdmin としてサインインします。 デプロイが成功した場合、PowerShell コンソールには、 **[COMPLETE: Action 'Deployment']\(完了: アクション 'デプロイ'\)** と表示されます。 
    > [!IMPORTANT]
    > コンピューターが azurestack ドメインに参加した後にローカル管理者としてサインインした場合、デプロイの進行状況は表示されません。 デプロイを再実行せず、代わりに azurestack\AzureStackAdmin としてサインインし、それが実行中であることを確認します。

    ![ASDK デプロイの成功](media/asdk-install/7.PNG)

これで、ASDK を正常にインストールしました。

なんらかの理由でデプロイが失敗した場合は、最初から[デプロイし直す](asdk-redeploy.md)か、次の PowerShell コマンドを使って、前回の成功した手順からデプロイを再開することができます。 このコマンドは、同じ管理者特権の PowerShell ウィンドウから使用できます。

  ```powershell
  cd C:\CloudDeployment\Setup
  .\InstallAzureStackPOC.ps1 -Rerun
  ```

## <a name="next-steps"></a>次のステップ
[デプロイ後の構成](asdk-post-deploy.md)
