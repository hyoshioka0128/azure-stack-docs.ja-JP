---
title: Kubernetes の Azure Stack Hub へのデプロイのトラブルシューティング
description: Kubernetes の Azure Stack Hub へのデプロイのトラブルシューティング方法について説明します。
author: mattbriggs
ms.topic: article
ms.author: mabrigg
ms.date: 2/1/2021
ms.reviewer: waltero
ms.lastreviewed: 11/14/2019
ms.openlocfilehash: b76bd5aa4e46eba8c7abf7f5beceae79ed5c025a
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99247746"
---
# <a name="troubleshoot-kubernetes-deployment-to-azure-stack-hub"></a>Kubernetes の Azure Stack Hub へのデプロイのトラブルシューティング

> [!NOTE]  
> Kubernetes Azure Stack Hub Marketplace 項目のみを使用して、概念実証としてクラスターをデプロイします。 Azure Stack Hub でサポートされている Kubernetes クラスターの場合は、[AKS エンジン](azure-stack-kubernetes-aks-engine-overview.md)を使用します。

この記事では、Kubernetes クラスターのトラブルシューティングを行う方法を確認します。 トラブルシューティングを開始するには、デプロイに必要な要素を確認します。 場合によっては、Azure Stack Hub または Kubernetes をホストする Linux VM からデプロイ ログを収集する必要があります。 管理エンドポイントからログを取得するには、Azure Stack Hub 管理者にお問い合わせください。

## <a name="overview-of-kubernetes-deployment"></a>Kubernetes のデプロイの概要

クラスターのトラブルシューティングを行う前に、Azure Stack Hub Kubernetes クラスターのデプロイ プロセスを確認してください。 デプロイは、Azure Resource Manager ソリューション テンプレートを使用して VM を作成し、クラスター用の AKS エンジンをインストールします。

### <a name="kubernetes-deployment-workflow"></a>Kubernetes のデプロイ ワークフロー

次の図は、クラスターをデプロイする一般的なプロセスを示しています。

![Kubernetes のデプロイ プロセス](media/azure-stack-solution-template-kubernetes-trouble/002-Kubernetes-Deploy-Flow.png)

### <a name="deployment-steps"></a>デプロイメントの手順

1. マーケットプレース項目から入力パラメーターを収集します。

    次のような Kubernetes クラスターの設定に必要な値を入力します。
    -  **ユーザー名**:Kubernetes クラスターと DVM の一部である Linux 仮想マシン (VM) のユーザー名。
    -  **[SSH 公開キー]** : Kubernetes クラスターと DVM の一部として作成されたすべての Linux マシンの承認に使用されるキー。
    -  **サービス プリンシパル**: Kubernetes Azure クラウド プロバイダーによって使用される ID。 サービス プリンシパルを作成したときにアプリケーション ID として識別されたクライアント ID。 
    -  **クライアント シークレット**: サービス プリンシパルを作成するときに作成したキー。

2. デプロイの VM とカスタム スクリプト拡張機能を作成します。
    -  マーケットプレースの Linux イメージ **Ubuntu Server 16.04-LTS** を使用して、デプロイの Linux VM を作成します。
    -  マーケットプレースからカスタム スクリプト拡張機能をダウンロードして実行します。 このスクリプトは **Custom Script for Linux 2.0** です。
    -  DVM カスタム スクリプトを実行します。 スクリプトでは次のタスクを実行します。
        1. Azure Resource Manager メタデータ エンドポイントからギャラリー エンドポイントを取得します。
        2. Azure Resource Manager メタデータ エンドポイントからアクティブなディレクトリ リソース ID を取得します。
        3. AKS エンジン用の API モデルを読み込みます。
        4. Kubernetes クラスターに AKS エンジンをデプロイし、Azure Stack Hub クラウド プロファイルを `/etc/kubernetes/azurestackcloud.json` に保存します。
3. マスター VM を作成します。

4. カスタム スクリプト拡張機能をダウンロードして実行します。

5. マスター スクリプトを実行します。

    スクリプトでは次のタスクを実行します。
    - kubelet などの etcd、Docker、および Kubernetes リソースをインストールします。 etcd は、複数のマシンのクラスターにデータを格納することができる分散型のキー値ストアです。 Docker は、コンテナーという最小限のオペレーティング システム レベルの仮想化をサポートしています。 Kubelet は、各 Kubernetes ノード上で動作するノード エージェントです。
    - **etcd** サービスを設定します。
    - **kubelet** サービスを設定します。
    - kubelet を開始します。 このタスクには、次の手順が含まれます。
        1. API サービスを開始します。
        2. コントローラー サービスを開始します。
        3. スケジューラ サービスを開始します。
6. エージェント VM を作成します。

7. カスタム スクリプト拡張機能をダウンロードして実行します。

7. エージェント スクリプトを実行します。 エージェントのカスタム スクリプトでは次のタスクを実行します。
    - **etcd** をインストールします。
    - **kubelet** サービスを設定します。
    - Kubernetes クラスターを結合します。

## <a name="steps-to-troubleshoot-kubernetes"></a>Kubernetes のトラブルシューティング手順

Kubernetes クラスターをサポートしている VM のデプロイ ログを収集して確認することができます。 Azure Stack Hub 管理者に相談して、使用する必要がある Azure Stack Hub のバージョンを確認し、デプロイに関連したログを Azure Stack Hub から取得してください。

1. クラスターをデプロイしたリソース グループの **[デプロイ]** ウィンドウで、ARM デプロイによって返されたエラー コードを確認します。 エラー コードについては、AKS エンジンの GitHub リポジトリにある「[トラブルシューティング](https://github.com/msazurestackworkloads/azurestack-gallery/blob/master/kubernetes/docs/troubleshooting.md)」の記事で説明されています。 エラーの説明で問題を解決できない場合は、次の手順を続行してください。
2. Kubernetes クラスターで、[デプロイの状態](#review-deployment-status)を確認し、マスター ノードのログを取得します。
3. Azure Stack Hub の最新バージョンを使用していることを確認します。 使用しているバージョンがわからない場合は、Azure Stack Hub の管理者にお問い合わせください。
4. VM 作成ファイルを確認します。 次の問題が発生している可能性があります。  
    - 公開キーが無効な場合があります。 作成したキーを確認してください。  
    - VM 作成によって内部エラーまたは作成エラーがトリガーされることがあります。 エラーは、Azure Stack Hub サブスクリプションの容量制限など、さまざまな要因で起こることがあります。
    - VM の完全修飾ドメイン名 (FQDN) は重複するプレフィックスで始まっていることを確認してください。
5.  VM が **[OK]** の場合は、DVM を評価します。 DVM にエラー メッセージがある場合:
    - 公開キーが無効な場合があります。 作成したキーを確認してください。  
    - Azure Stack Hub 管理者に連絡して、特権付きエンドポイントを使用して Azure Stack Hub のログを取得してください。 詳細については、[Azure Stack Hub の診断ツール](../operator/azure-stack-get-azurestacklog.md)に関するページを参照してください。
5. デプロイに関して質問がある場合は、[Azure Stack Hub フォーラム](https://social.msdn.microsoft.com/Forums/azure/home?forum=azurestack)で質問を投稿するか、他の人が既に回答を受け取っていないか確認することができます。 


## <a name="review-deployment-status"></a>デプロイの状態を確認する

Kubernetes クラスターをデプロイする際には、デプロイの状態を確認して、問題をチェックすることができます。

1. Azure Stack Hub ポータル `https://portal.local.azurestack.external` を開きます。
2. **[リソース グループ]** を選択し、Kubernetes クラスターのデプロイ時に使用したリソース グループの名前を選択します。
3. **[デプロイ]** を選択し、 **[デプロイ名]** を選択します。

    ![Kubernetes のトラブルシューティング:デプロイの選択](media/azure-stack-solution-template-kubernetes-trouble/azure-stack-kub-trouble-report.png)

4.  トラブルシューティングのウィンドウを確認します。 デプロイ済みの各リソースから、次の情報が提供されます。
    
    | プロパティ | 説明 |
    | ----     | ----        |
    | リソース | リソースの名前。 |
    | Type | リソース プロバイダーとリソースの種類。 |
    | Status | 項目の状態。 |
    | TimeStamp | 時刻のタイムスタンプ (UTC)。 |
    | 操作の詳細 | 操作に関係するリソース プロバイダー、リソースのエンド ポイント、リソースの名前などの操作の詳細。 |

    各項目には緑色または赤色の状態アイコンが表示されます。

## <a name="review-deployment-logs"></a>デプロイ ログを確認する

Azure Stack Hub ポータルに、デプロイ エラーのトラブルシューティングや対処を行うための十分な情報がない場合は、次の手順として、クラスター ログで詳細を確認します。 デプロイ ログを手動で取得するには、通常、クラスターのマスター VM のいずれかに接続する必要があります。 より簡単な方法として、Azure Stack Hub チームが提供する以下の [Bash スクリプト](https://aka.ms/AzsK8sLogCollectorScript)をダウンロードして実行することもできます。 このスクリプトは、DVM およびクラスターの VM に接続し、関連するシステム ログとクラスター ログを収集して、ワークステーションにそれらのログをダウンロードして戻します。

### <a name="prerequisites"></a>前提条件

Azure Stack Hub の管理に使用するマシンには Bash プロンプトが必要です。 Windows マシンでは、[Git for Windows](https://git-scm.com/downloads) をインストールすることで Bash プロンプトを取得できます。 インストールされたら、[スタート] メニューで _Git Bash_ を探します。

### <a name="retrieving-the-logs"></a>ログを取得する

クラスター ログを収集してダウンロードするには、次の手順に従います。

1. Bash プロンプトを開きます。 Windows マシンから、_Git Bash_ を開くか、次を実行します。`C:\Program Files\Git\git-bash.exe`

2. Bash プロンプトで次のコマンドを実行して、ログ コレクター スクリプトをダウンロードします。

    ```Bash  
    mkdir -p $HOME/kuberneteslogs
    cd $HOME/kuberneteslogs
    curl -O https://raw.githubusercontent.com/msazurestackworkloads/azurestack-gallery/master/diagnosis/getkuberneteslogs.sh
    chmod 744 getkuberneteslogs.sh
    ```

3. スクリプトに必要な情報を探して、実行します。

    | パラメーター           | 説明                                                                                                      | 例                                                                       |
    |---------------------|------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------|
    | -d、--vmd-host      | DVM のパブリック IP または完全修飾ドメイン名 (FQDN)。 VM 名は `vmd-` で始まります。 | IP:192.168.102.38<br>DNS: vmd-myk8s.local.cloudapp.azurestack.external |
    | -h, --help  | Print コマンドの使用方法。 | |
    | -i、--identity-file | Kubernetes クラスターの作成時に Marketplace 項目に渡される RSA 秘密キー ファイルのパス。 Kubernetes ノードにリモートからアクセスするために必要です。 | C:\data\id_rsa.pem (Putty)<br>~/.ssh/id_rsa (SSH)
    | -m、--master-host   | Kubernetes マスター ノードのパブリック IP または完全修飾ドメイン名 (FQDN)。 VM 名は `k8s-master-` で始まります。 | IP:192.168.102.37<br>FQDN: k8s-12345.local.cloudapp.azurestack.external      |
    | -u、--user          | Kubernetes クラスターの作成時にマーケットプレース項目に渡されるユーザー名。 Kubernetes ノードにリモートからアクセスするために必要です。 | azureuser (既定値) |


   パラメーター値を追加すると、コマンドは次の例のようになります。

    ```Bash  
    ./getkuberneteslogs.sh --identity-file "C:\id_rsa.pem" --user azureuser --vmd-host 192.168.102.37
     ```

4. 数分後に、スクリプトによって収集されたログが `KubernetesLogs_{{time-stamp}}` というディレクトリに出力されます。 そこに、クラスターに属している各 VM のディレクトリがあります。

    また、ログ コレクター スクリプトは、ログ ファイルでエラーを検索し、既知の問題を検出した場合はトラブルシューティング手順を含めます。 既知の問題が検出される可能性を高めるために、最新バージョンのスクリプトを実行してください。

> [!NOTE]  
> ログ コレクター スクリプトの詳細を確認するには、 こちらの GitHub [リポジトリ](https://github.com/msazurestackworkloads/azurestack-gallery/tree/master/diagnosis) を参照してください。

## <a name="next-steps"></a>次のステップ

[Kubernetes を Azure Stack Hub にデプロイする](azure-stack-solution-template-kubernetes-deploy.md)

[Kubernetes クラスターを Marketplace に追加する (Azure Stack Hub のオペレーター)](../operator/azure-stack-solution-template-kubernetes-cluster-add.md)

[Azure における Kubernetes](/azure/container-service/kubernetes/container-service-kubernetes-walkthrough)
