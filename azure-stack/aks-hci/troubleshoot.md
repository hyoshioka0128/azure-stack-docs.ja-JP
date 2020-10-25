---
title: AKS のトラブルシューティング
description: この記事では Azure Stack HCI での Azure Kubernetes Service (AKS) のトラブルシューティングに関する情報を提供します。
author: davannaw-msft
ms.topic: how-to
ms.date: 09/22/2020
ms.author: dawhite
ms.openlocfilehash: eaa5879f0e8cd00f43a37247f03f7ce09ecdb78c
ms.sourcegitcommit: 362081a8c19e7674c3029c8a44d7ddbe2deb247b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/09/2020
ms.locfileid: "91899620"
---
# <a name="troubleshooting-azure-kubernetes-service-on-azure-stack-hci"></a>Azure Stack HCI 上の Azure Kubernetes Service に関するトラブルシューティング

Azure Stack HCI 上の Azure Kubernetes Service を使用して Kubernetes クラスターを作成または管理するときに、問題が発生することがあります。 この記事では、これらの問題の解決に役立つトラブルシューティングのガイドラインを提供します。

## <a name="troubleshooting-azure-stack-hci"></a>Azure Stack HCI のトラブルシューティング
Azure Stack HCI クラスター内の複数のサーバーにわたる、ネットワークとストレージの QoS (サービスの品質) 設定に関するクラスター検証レポートについてトラブルシューティングを行い、重要なルールが定義されていることを確認するには、「[クラスター検証レポートのトラブルシューティング](../hci/manage/validate-qos.md)」を参照してください。

CredSSP に関する問題のトラブルシューティングについては、「[CredSSP のトラブルシューティング](../hci/manage/troubleshoot-credssp.md)」を参照してください。

## <a name="troubleshooting-windows-admin-center"></a>Windows Admin Center のトラブルシューティング
この製品はパブリック プレビュー段階であるため、まだ開発中です。 現時点では、Windows Admin Center の Azure Kubernetes Service 拡張機能にはいくつかの問題があります。 
* 現時点では、Azure Stack HCI 上の Azure Kubernetes Service を設定するために使用するシステムのクラスター内の各サーバーは、信頼されたサーバーである必要があります。 そのため、Windows Admin Center は、クラスター内の 1 つまたはいくつかだけではなく、すべてのサーバーで CredSSP 操作を実行できる必要があります。 
* `msft.sme.aks couldn't load` というエラーが表示され、チャンクの読み込みに失敗したというエラーが表示された場合は、最新バージョンの Microsoft Edge または Google Chrome を使用してもう一度お試しください。
* Azure Kubernetes Service ホストを設定するためのウィザード、または Kubernetes クラスターを作成するためのウィザードを開始する前に、Windows Admin Center を通して Azure にサインインする必要があります。 ワークフロー中にもう一度サインインすることが必要になる場合があります。 Windows Admin Center を通した Azure へのサインインで問題が生じている場合は、[Azure portal](https://portal.azure.com/) などの別の操作元から Azure アカウントへのサインインをお試しください。 引き続き問題が発生する場合は、サポートに連絡する前に [Windows Admin Center の既知の問題](/windows-server/manage/windows-admin-center/support/known-issues)に関する記事をご確認ください。
* Windows Admin Center に表示された Azure Stack HCI デプロイ上の Azure Kubernetes Service の現在のイテレーションでは、Azure Kubernetes Service ホストをセットアップしたユーザーのみがシステムに Kubernetes クラスターを作成できます。 この問題を回避するには、Azure Kubernetes Service ホストを設定したユーザーのプロファイルから、新しい Kubernetes クラスターを作成するユーザーのプロファイルに、.wssd フォルダーをコピーします。
* いずれかのウィザードで誤った構成に関するエラーが発生する場合は、クラスターのクリーンアップ操作を実行します。 これらの操作には、C:\Program Files\AksHci\mocctl.exe ファイルの削除が含まれる場合があります。
* クラスターの作成ウィザードで CredSSP を正常に機能させるには、Windows Admin Center のインストールと使用を同じアカウントで行う必要があります。 あるアカウントで Windows Admin Center をインストールし、別のアカウントでそれを使用しようとすると、エラーが発生します。
* クラスターのデプロイ中に、helm.zip ファイルの転送に関する問題が発生することがあります。 この問題により、helm.zip ファイルへのパスが存在しないか無効であるというエラーが発生することがよくあります。 この問題を解決するには、デプロイを再試行してみてください。
* 長時間にわたってデプロイが停止する場合は、CredSSP または接続の問題が発生しているおそれがあります。 デプロイのトラブルシューティングを行うには、以下の手順をお試しください。 
    1.  Windows Admin Center を実行しているマシンの PowerShell ウィンドウで、次のコマンドを実行します。 
          ```PowerShell
          Enter-PSSession <servername>
          ```
    2.  このコマンドが成功する場合は、サーバーに接続でき、接続に関する問題はありません。
    
    3.  CredSSP の問題が発生している場合は、次のコマンドを実行して、ゲートウェイ マシンとターゲット マシンの間の信頼をテストします。 
          ```PowerShell
          Enter-PSSession –ComputerName <server> –Credential company\administrator –Authentication CredSSP
          ``` 
        次のコマンドを実行して、ローカル ゲートウェイにアクセスする際の信頼をテストすることもできます。 
          ```PowerShell
          Enter-PSSession -computer localhost -credential (Get-Credential)
          ``` 
* Azure Arc を使用していて、複数のテナント ID がある場合は、次のコマンドを実行して、デプロイの前に目的のテナントを指定します。 そうしないと、デプロイが失敗するおそれがあります。

   ```Azure CLI
   az login –tenant <tenant>
   ```
* 新しい Azure アカウントを作成したばかりで、ゲートウェイ マシン上のアカウントにサインインしていない場合は、Windows Admin Center ゲートウェイを Azure に登録する場合に問題が発生することがあります。 この問題を軽減するには、ブラウザーの別のタブまたはウィンドウで Azure アカウントにサインインしてから、Windows Admin Center ゲートウェイを Azure に登録します。

### <a name="creating-windows-admin-center-logs"></a>Windows Admin Center ログの作成
Windows Admin Center を使用して問題を報告する場合は、開発チームが問題を診断するのに役立つログを添付することをお勧めします。 Windows Admin Center のエラーは、通常、次の 2 つのいずれかの形式で発生します。 
- Windows Admin Center を実行しているコンピューターのイベント ビューアーに表示されるイベント 
- ブラウザー コンソールに現れる JavaScript の問題 

Windows Admin Center のログを収集するには、パブリック プレビュー パッケージで提供されている Get-SMEUILogs.ps1 スクリプトを使用します。 
 
このスクリプトを使用するには、スクリプトが格納されているフォルダーで次のコマンドを実行します。 
 
```PowerShell
./Get-SMEUILogs.ps1 -ComputerNames [comp1, comp2, etc.] -Destination [comp3] -HoursAgo [48] -NoCredentialPrompt
```
 
コマンドには以下のパラメーターがあります。
 
* `-ComputerNames`:ログの収集対象とするマシンの一覧。
* `-Destination`: ログの集計先のマシン。
* `-HoursAgo`: ログを収集する開始時刻。スクリプトを実行する前の時間で表されます。
* `-NoCredentialPrompt`: 資格情報のプロンプトをオフにし、現在の環境で既定の資格情報を使用するためのスイッチ。
 
このスクリプトの実行に問題がある場合は、次のコマンドを実行してヘルプ テキストを表示できます。 
 
```PowerShell
GetHelp .\Get-SMEUILogs.ps1 -Examples
```

## <a name="troubleshooting-windows-worker-nodes"></a>Windows ワーカー ノードのトラブルシューティング 
Windows ワーカー ノードにサインインするには、まず、`kubectl get` を実行してノードの IP アドレスを取得します。 `EXTERNAL-IP` 値をメモします。

```PowerShell
kubectl get nodes -o wide
``` 
`ssh Administrator@ip` を使用して、SSH でノードに接続します。 SSH でノードに接続したら、`net user administrator *` を実行して管理者のパスワードを更新できます。 

## <a name="troubleshooting-linux-worker-nodes"></a>Linux ワーカー ノードのトラブルシューティング 
Linux ワーカー ノードにサインインするには、まず、`kubectl get` を実行してノードの IP アドレスを取得します。 `EXTERNAL-IP` 値をメモします。

```PowerShell
kubectl get nodes -o wide
``` 
`ssh clouduser@ip` を使用して、SSH でノードに接続します。 

## <a name="troubleshooting-azure-arc-kubernetes"></a>Azure Arc Kubernetes のトラブルシューティング
接続、アクセス許可、および Arc エージェントに関連するいくつかの一般的なシナリオのトラブルシューティングについては、[Azure Arc 対応 Kubernetes のトラブルシューティング](/azure/azure-arc/kubernetes/troubleshooting)に関するページを参照してください。

## <a name="next-steps"></a>次のステップ
Azure Stack HCI で Azure Kubernetes Service を使用しているときに問題が引き続き発生する場合は、[GitHub](https://aka.ms/aks-hci-issues) を通じてバグを報告できます。  
