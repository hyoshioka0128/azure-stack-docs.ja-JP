---
title: Azure Stack Hub 上の AKS エンジンに関する既知の問題
description: Azure Stack Hub 上で AKS エンジンを使用する場合の既知の問題について説明します。
author: mattbriggs
ms.topic: article
ms.date: 2/1/2021
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 09/11/2020
ms.openlocfilehash: b65fb410cd18e058f4e0ed7299eeff2d9daeb7f6
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99246879"
---
# <a name="known-issues-with-the-aks-engine-on-azure-stack-hub"></a>Azure Stack Hub 上の AKS エンジンに関する既知の問題

このトピックでは Azure Stack Hub 上の AKS エンジンに関する既知の問題について説明します。

## <a name="unable-to-resize-cluster-vms-with-the-compute-service"></a>コンピューティング サービスでクラスター VM のサイズを変更できません

- **適用先**: Azure Stack Hub、AKS エンジン (すべて)
- **説明**:AKS エンジンでは、コンピューティング サービスを利用してクラスター VM のサイズを変更できません。 AKS エンジンの場合、API モデル json ファイルでクラスターの状態が維持されます。 AKS エンジンで行われた作成、更新、スケール操作で目的の VM サイズが反映されるようにするには、それらの操作を実行する前に API モデルを更新する必要があります。 たとえば、コンピューティング サービスを利用し、既にデプロイされているクラスター上で VM のサイズを別のサイズに変更する場合、`aks-engine update` の実行時に状態が失われます。
- **対応策**: 正しく機能させるには、クラスターの API モデルを見つけ、そこでサイズを変更し、それから `aks-engine update` を実行します。
- **発生頻度**: コンピューティング サービスを使用してサイズを変更しようとしたとき。

## <a name="disk-detach-operation-fails-in-aks-engine-0550"></a>AKS エンジン 0.55.0 でディスクのデタッチ操作が失敗する

- **適用先**: Azure Stack Hub (更新プログラム 2005)、AKS エンジン 0.55.0
- **説明**:永続化ボリュームを含むデプロイを削除しようとすると、削除操作によって一連のアタッチ/デタッチ エラーがトリガーされます。 この原因は、AKS エンジン v 0.55.0 クラウド プロバイダーのバグです。 クラウド プロバイダーは、Azure Stack Hub (更新プログラム 2005) で Azure Resource Manager が現在サポートするよりも新しいバージョンの API を使用して Azure Resource Manager を呼び出します。
- **対応策**: [AKS エンジンの GitHub リポジトリ (問題 3817)](https://github.com/Azure/aks-engine/issues/3817#issuecomment-691329443) に関するページで詳細および軽減手順を確認できます。 AKS エンジンと対応するイメージの新しいビルドが利用可能になったら、ただちにアップグレードてください。
- **発生頻度**: 永続化ボリュームを含むデプロイの削除時。



## <a name="upgrade-issues-in-aks-engine-0510"></a>AKS エンジン 0.51.0 でのアップグレードの問題

* Kubernetes クラスターのバージョン 1.15.x から 1.16.x へのアップグレード (AKS アップグレード) 中に、次の Kubernetes コンポーネントをアップグレードするには、追加の手順が必要になります: **kube-proxy**、**azure-cni-networkmonitor**、**csi-secrets-store**、**kubernetes-dashboard**。 次に、発生する可能性のある問題とその対処方法について説明します。

  * 接続された環境では、影響を受けるコンポーネントがアップグレードされなかったという表示がクラスターで出されないため、この問題に気付かない場合があります。 すべてが予期したとおりに動作しているように見えます。
  <!-- * In disconnected environments, you can see this problem when you run a query for the system pods status and see that the pods for the components mentioned below are not in "Ready" state: -->

    ```bash  
    kubectl get pods -n kube-system
    ```

  * これらのコンポーネントのそれぞれでこの問題を解決するための回避策として、次の表の [回避策] 列のコマンドを実行します。

    |[コンポーネント名] |回避策 |影響を受けるシナリオ|
    |---------------|-----------|------------------|
    |**kube-proxy**     | `kubectl delete ds kube-proxy -n kube-system` |接続、切断 |
    |**azure-cni-networkmonitor**   | `kubectl delete ds azure-cni-networkmonitor -n kube-system`   | 接続、切断 |
    |**csi-secrets-store**  |`sudo sed -i s/Always/IfNotPresent/g /etc/kubernetes/addons/secrets-store-csi-driver.yaml`<br>`kubectl delete ds csi-secrets-store -n kube-system` | [Disconnected]\(切断済み\) |
    |**kubernetes-dashboard** |各マスター ノードで次のコマンドを実行します。<br>`sudo sed -i s/Always/IfNotPresent/g /etc/kubernetes/addons/kubernetes-dashboard.yaml` |[Disconnected]\(切断済み\) |

* Kubernetes 1.17 はこのリリースではサポートされていません。 1\.17 を参照している GitHub pull request (PR) がありますが、サポートされていません。

## <a name="aks-engine-get-versions-command-limitations"></a>aks-engine get-versions コマンドの制限

**aks-engine** `get-versions` コマンドの出力は、Azure Stack Hub ではなくグローバル Azure にのみ関連します。 さまざまなアップグレード パスの詳細については、「[新しい Kubernetes バージョンにアップグレードする手順](azure-stack-kubernetes-aks-engine-upgrade.md#steps-to-upgrade-to-a-newer-kubernetes-version)」を参照してください。

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub 上の AKS エンジンの概要](azure-stack-kubernetes-aks-engine-overview.md)
