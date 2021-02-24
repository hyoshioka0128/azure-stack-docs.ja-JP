---
title: Azure Stack Hub 上の AKS エンジンのトラブルシューティング
description: この記事では Azure Stack Hub 上の AKS エンジンのトラブルシューティングの手順について説明します。
author: mattbriggs
ms.topic: article
ms.date: 10/07/2020
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 10/07/2020
ms.openlocfilehash: ae82bb1c07ec8f466eb29fe8c610af09e01e233a
ms.sourcegitcommit: 1621f2748b2059fd47ccacd48595a597c44ee63f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/08/2020
ms.locfileid: "91853178"
---
# <a name="troubleshoot-the-aks-engine-on-azure-stack-hub"></a>Azure Stack Hub 上の AKS エンジンのトラブルシューティング

Azure Stack Hub で AKS エンジンをデプロイまたは使用するときに問題が発生することがあります。 この記事では、AKS エンジンのデプロイのトラブルシューティング、ご使用の AKS エンジンに関する情報の収集、Kubernetes ログの収集、カスタム スクリプト拡張機能のエラー コードの確認、および AKS エンジンに関する GitHub イシューを開くための手順について説明します。

## <a name="troubleshoot-the-aks-engine-install"></a>AKS エンジンのインストールのトラブルシューティング

### <a name="try-gofish"></a>GoFish を試す

以前のインストール手順が失敗した場合は、GoFish パッケージ マネージャーを使用して AKS エンジンをインストールできます。 [GoFish](https://gofi.sh) では、自身がクロスプラットフォームの Homebrew として記述されます。

#### <a name="install-the-aks-engine-with-gofish-on-linux"></a>GoFish を使用して Linux に AKS エンジンをインストールする

[[インストール]](https://gofi.sh/#install) ページから GoFish をインストールします。

1. Bash プロンプトから、次のコマンドを実行します。

    ```bash
    curl -fsSL https://raw.githubusercontent.com/fishworks/gofish/master/scripts/install.sh | bash
    ```

2.  次のコマンドを実行して、GoFish を使用して AKS エンジンをインストールします。

    ```bash
    Run "gofish install aks-engine"
    ```

#### <a name="install-the-aks-engine-with-gofish-on-windows"></a>GoFish を使用して Windows に AKS エンジンをインストールする

[[インストール]](https://gofi.sh/#install) ページから GoFish をインストールします。

1. 管理者特権の PowerShell プロンプトから、次のコマンドを実行します。

    ```PowerShell
    Set-ExecutionPolicy Bypass -Scope Process -Force
    iex ((New-Object System.Net.WebClient).DownloadString('https://raw.githubusercontent.com/fishworks/gofish/master/scripts/install.ps1'))
    ```

2.  同じセッションで次のコマンドを実行して、GoFish を使用して AKS エンジンをインストールします。

    ```PowerShell
    gofish install aks-engine
    ```

### <a name="checklist-for-common-deployment-issues"></a>デプロイに関する一般的な問題のチェックリスト

AKS エンジンを使用して Kubernetes クラスターをデプロイしているときにエラーが発生した場合は、次のことを確認します。

1.  正しいサービス プリンシパル資格情報 (SPN) を使用していますか?
2.  SPN には Azure Stack Hub サブスクリプションに対する "共同作成者" ロールがありますか?
3. Azure Stack Hub プランに十分な大きさのクォータがありますか?
4.  Azure Stack Hub インスタンスには、適用される修正プログラムまたはアップグレードがありますが?

詳細については、**Azure/aks-engine GitHub** リポジトリにある「[トラブルシューティング](https://github.com/Azure/aks-engine/blob/master/docs/howto/troubleshooting.md)」を参照してください。

## <a name="collect-aks-engine-logs"></a>AKS エンジン ログの収集

AKS エンジンによって作成された情報を確認できます。 AKS エンジンでは、アプリケーションの実行時の状態とエラーが報告されます。 出力をテキスト ファイルにパイプすることも、コマンドライン コンソールから直接コピーすることもできます。 AKS エンジンによってトリガーされるエラー コードの一覧については、「[カスタム スクリプト拡張機能のエラー コードを確認する](#review-custom-script-extension-error-codes)」を参照してください。

1.  AKS エンジンのコマンドライン ツールに表示される情報から、標準出力とエラーを収集します。

2. ローカル ファイルからログを取得します。 出力ディレクトリは、 **--output-directory** フラグを設定して `get-logs` コマンドで設定できます。

    ログのローカル パスを設定するには:

    ```bash  
    aks-engine get-logs --output-directory <path to the directory>
    ```

## <a name="collect-kubernetes-logs"></a>Kubernetes のログの収集

AKS エンジン ログに加えて、Kubernetes コンポーネントでは、状態とエラー メッセージが生成されます。 これらのログは、Bash スクリプト [getkuberneteslogs.sh](https://github.com/msazurestackworkloads/azurestack-gallery/releases/tag/diagnosis-v0.1.5) を使用して収集できます。

このスクリプトにより、次のログの収集プロセスが自動化されます。 

- ディレクトリ `/var/log/azure/` 内のログ ファイル
- ディレクトリ `/var/log/kubeaudit` 内のログ ファイル (kube 監査ログ)
- ログ ファイル `/var/log/waagent.log` (waagent)
- ログ ファイル `/var/log/azure/deploy-script-dvm.log` (Azure Stack Hub の Kubernetes Cluster Marketplace の項目を使用してデプロイされている場合)
- ディレクトリ `/etc/kubernetes/manifests` 内の静的マニフェスト
- ディレクトリ ` /etc/kubernetes/addons` 内の静的アドオン
- kube-system コンテナーのメタデータとログ
- kubelet の状態と履歴
- etcd の状態と履歴
- Docker の状態と履歴
- kube-system スナップショット
- Azure CNI 構成ファイル

Windows ノード用に、いくつかの追加ログが取得されます。

- ログ ファイル `c:\Azure\CustomDataSetupScript.log`
- kube-proxy の状態と履歴
- containerd の状態と履歴
- azure-vnet ログと azure-vnet-telemetry ログ
- Docker の ETW イベント
- Hyper-V の ETW イベント

このスクリプトがない場合は、クラスター内の各ノードに接続して、手動でログを検索してダウンロードする必要があります。 また、このスクリプトでは、必要に応じて、収集したログをストレージ アカウントにアップロードして、他のユーザーとログを共有することもできます。

要件:

 - Linux VM、Git Bash、または Bash on Windows。
 - スクリプトを実行するマシンに [Azure CLI](azure-stack-version-profiles-azurecli2.md) がインストールされている。
 - Azure Stack Hub への Azure CLI セッションにサインインしたサービス プリンシパル ID。 このスクリプトは、作業を行うために Azure Stack Resource Manager リソースを検出して作成することができるため、Azure CLI とサービス プリンシパル ID が必要です。
 - 環境で Kubernetes クラスターが既に選択されているユーザー アカウント (サブスクリプション)。 
1. スクリプト tar ファイルの最新リリースをご自分のクライアント VM、Kubernetes クラスターにアクセスできるマシン、または AKS エンジンを使用してクラスターをデプロイするために使用したものと同じマシンにダウンロードします。

    次のコマンドを実行します。

    ```bash  
    mkdir -p $HOME/kuberneteslogs
    cd $HOME/kuberneteslogs
    wget https://github.com/msazurestackworkloads/azurestack-gallery/releases/download/diagnosis-v0.1.5/diagnosis-v0.1.5.tar.gz
    tar xvf diagnosis-v0.1.5.tar.gz -C ./
    ```

2. `getkuberneteslogs.sh` スクリプトに必要なパラメーターを探します。 このスクリプトでは、次のパラメーターが使用されます。

    | パラメーター | 説明 | 必須 | 例 |
    | --- | --- | --- | --- |
    | -h, --help | Print コマンドの使用方法。 | no | 
    -u,--user | クラスター VM の管理者ユーザー名 | はい | azureuser<br>(既定値) |
    | -i、--identity-file | Kubernetes クラスターの作成に使用される公開キーに関連付けられた RSA 秘密キー (' id_rsa ' と呼ばれることもあります)  | はい | `./rsa.pem` (Putty)<br>`~/.ssh/id_rsa` (SSH) |
    |   -g, --resource-group    | Kubernetes クラスター リソース グループ | はい | k8sresourcegroup |
    |   -n, --user-namespace               | 指定された名前空間のコンテナーからログが収集されます (kube-system ログは常に収集されます) | no |   monitoring |
    |       --api-model                    | Azure Stack Hub ストレージ アカウントで apimodel.json ファイルが保持されます。 --upload-logs パラメーターも指定すると、apimodel.json ファイルがストレージ アカウントにアップロードされます。 | no | `./apimodel.json` |
    | --all-namespaces               | すべての名前空間のコンテナーからログが収集されます。 --user-namespace がオーバーライドされます。 | no | |
    | --upload-logs                  | Azure Stack Hub ストレージ アカウントで取得したログが保持されます。 ログは KubernetesLogs リソース グループにあります。 | no | |
    --disable-host-key-checking    | スクリプトの実行中に、SSH の StrictHostKeyChecking オプションが "no" に設定されます。 安全な環境でのみ使用してください。 | no | |

3. ご自分の情報を使用して、次のサンプル コマンドのいずれかを実行します。

    ```bash
    ./getkuberneteslogs.sh -u azureuser -i private.key.1.pem -g k8s-rg
    ./getkuberneteslogs.sh -u azureuser -i ~/.ssh/id_rsa -g k8s-rg --disable-host-key-checking
    ./getkuberneteslogs.sh -u azureuser -i ~/.ssh/id_rsa -g k8s-rg -n default -n monitoring
    ./getkuberneteslogs.sh -u azureuser -i ~/.ssh/id_rsa -g k8s-rg --upload-logs --api-model clusterDefinition.json
    ./getkuberneteslogs.sh -u azureuser -i ~/.ssh/id_rsa -g k8s-rg --upload-logs
    ```

## <a name="review-custom-script-extension-error-codes"></a>カスタム スクリプト拡張機能のエラー コードを確認する

クラスターを実行しているカスタム スクリプト拡張機能 (CSE) によって作成されたエラー コードの一覧を参照できます。 CSE エラーは、問題の根本原因を診断するのに役立ちます。 Kubernetes クラスターで使用されている Ubuntu サーバーの CSE では、AKS エンジンの操作の多くがサポートされます。 CSE 終了コードの詳細については、「[cse_helpers.sh](https://github.com/Azure/aks-engine/blob/master/pkg/engine/cse.go)」を参照してください。

### <a name="providing-kubernetes-logs-to-a-microsoft-support-engineer"></a>Microsoft サポート エンジニアへの Kubernetes ログの提供

ログの収集と調査を行っても問題が解決しない場合は、サポート チケットの作成プロセスを開始し、`--upload-logs` パラメーターを設定して `getkuberneteslogs.sh` を実行して収集したログを提供することができます。 

お客様の Azure Stack Hub オペレーターに依頼してください。 オペレーターは、ログの情報を使用してサポート ケースを作成します。

サポートの問題に対処する過程で、Microsoft サポート エンジニアから Azure Stack Hub オペレーターに対して、Azure Stack Hub システム ログの収集が要請されることがあります。 必要に応じて、`getkuberneteslogs.sh` を実行して Kubernetes ログをアップロードしたストレージ アカウント情報をオペレーターに渡します。

オペレーターは **Get-AzureStackLog** PowerShell コマンドレットを実行するこもできます。 このコマンドには、Kubernetes ログを保存したストレージ アカウントを指定するパラメーター (`-InputSaSUri`) が使用されます。

オペレーターは、お客様が生成したログと、Microsoft サポートから要請される場合があるその他のシステム ログを組み合わせて Microsoft に渡すことができます。

## <a name="open-github-issues"></a>GitHub イシューを開く

デプロイ エラーを解決できない場合は、GitHub イシューを開くことができます。 

1. AKS エンジン リポジトリで [GitHub イシュー](https://github.com/Azure/aks-engine/issues/new)を開きます。
2. 次の形式を使用してタイトルを追加します。C`SE error: exit code <INSERT_YOUR_EXIT_CODE>`.
3. イシューには次の情報を含めます。

    - クラスターのデプロイに使用されるクラスター構成ファイル `apimodel json`。 GitHub に投稿する前に、すべてのシークレットとキーを削除します。  
     - 次の **kubectl** コマンド `get nodes` の出力。  
     - `/var/log/azure/cluster-provision.log` と `/var/log/cloud-init-output.log` の内容

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub 上の AKS エンジン](azure-stack-kubernetes-aks-engine-overview.md)を確認してください
