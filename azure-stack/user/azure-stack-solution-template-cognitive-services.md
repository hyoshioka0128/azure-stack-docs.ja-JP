---
title: Azure Cognitive Services を Azure Stack Hub にデプロイする
description: Azure Cognitive Services を Azure Stack Hub にデプロイする方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 10/09/2020
ms.author: mabrigg
ms.reviewer: guanghu
ms.lastreviewed: 10/09/2020
ms.openlocfilehash: 4dc6ad30d54b61e05f54be4a4f1301472794db60
ms.sourcegitcommit: e13f27291bab236aac5d8b05401056961e9cc1e9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/19/2020
ms.locfileid: "97697686"
---
# <a name="deploy-azure-cognitive-services-to-azure-stack-hub"></a>Azure Cognitive Services を Azure Stack Hub にデプロイする

Azure Cognitive Services は、Azure Stack Hub のコンテナー サポートとともに使用できます。 Azure Cognitive Services でのコンテナーのサポートにより、Azure で使用できる同じ豊富な API を使用できます。 コンテナーを使用することで、[Docker コンテナー](https://www.docker.com/what-container)で提供されるサービスのデプロイ先やホスト先に柔軟に対応できるようになります。 

コンテナー化とは、アプリまたはサービスとその依存関係や構成をコンテナー イメージとしてパッケージ化する、ソフトウェア配布のアプローチです。 ほとんどまたはまったく変更せずに、コンテナー ホストにイメージをデプロイできます。 各コンテナーは、他のコンテナーや、基になるオペレーティング システムから分離されます。 システム自体には、イメージを実行するために必要なコンポーネントしかありません。 コンテナー ホストは、仮想マシンよりも小さいフット プリントです。 短期的なタスクではイメージからコンテナーを作成することもでき、不要になったときには削除できます。

現在、Azure Cognitive Services のサブセットに対するコンテナー サポートを利用できます。

- **Speech**
    - 音声テキスト変換 (標準)
    - 音声テキスト変換 (カスタム)
    - テキスト読み上げ (標準)
    - テキスト読み上げ (ニューラル)
- **Language**
    - 言語の理解
    - テキスト分析 (センチメント分析)
- **Decision**
    - Anomaly Detector


> [!IMPORTANT]
> Azure Stack Hub 用の Azure Cognitive Services のサブセットは、現在パブリック プレビューの段階にあります。
> プレビュー バージョンはサービス レベル アグリーメントなしで提供されています。運用環境のワークロードに使用することはお勧めできません。 特定の機能はサポート対象ではなく、機能が制限されることがあります。 詳しくは、[Microsoft Azure プレビューの追加使用条件](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)に関するページをご覧ください。

次の Azure Cognitive Services のサブセットに対するコンテナー サポートは、現在プレビュー段階にあります。

- **Computer Vision**
    - Read 3.0、Read 3.1
    - 空間分析 (新機能)
- **Language**
    - キー フレーズの抽出
    - 言語検出
    - Text Analytics for Health
- **Speech**
    - 音声言語検出
    - ニューラル TTS
    - テキスト読み上げ (カスタム)

## <a name="use-containers-with-cognitive-services-on-azure-stack-hub"></a>Azure Stack Hub の Cognitive Services でコンテナーを使用する

- **データの制御**  
  Cognitive Services を使用しているときに、データを制御することをアプリ ユーザーに許可します。 Cognitive Services は、グローバル Azure またはパブリック クラウドにデータを送信できないアプリ ユーザーに配信できます。

- **モデルの更新プログラムの制御**  
  アプリ ユーザーに、ソリューションにデプロイされたモデルのバージョン更新プログラムを提供します。

- **ポータブル アーキテクチャ**  
  ポータブル アプリ アーキテクチャを作成できるようにして、ソリューションをパブリック クラウドや、オンプレミスまたはエッジのプライベート クラウドにデプロイできるようにします。 コンテナーは、Azure Kubernetes Service、Azure Container Instances、または Azure Stack Hub 内の Kubernetes クラスターにデプロイできます。 詳細については、[Azure Stack Hub への Kubernetes のデプロイ](azure-stack-solution-template-kubernetes-deploy.md)に関する記事を参照してください。

- **高スループットで待機時間の短いクエリ**  
   高いスループットと低待機時間のために、アプリ ユーザーにトラフィックの急増に合わせて拡大縮小する機能を提供します。 Cognitive Services を、アプリのロジックとデータに物理的に近い Azure Kubernetes Service で実行できるようにします。

Azure Stack Hub では、高可用性およびエラスティック スケーリングに向けて、アプリ コンテナーとともに Cognitive Services コンテナーを Kubernetes クラスターにデプロイします。 Cognitive Services を、App Services、Functions、BLOB ストレージ、SQL、または MySQL データベースで作成されたコンポーネントと組み合わせることで、アプリを開発できます。

Cognitive Services のコンテナーの詳細については、「[Azure Cognitive Services でのコンテナーのサポート](/azure/cognitive-services/cognitive-services-container-support)」を参照してください。

## <a name="deploy-the-azure-face-api"></a>Azure Face API をデプロイする

この記事では、Azure Stack Hub の Kubernetes クラスターに Azure Face API をデプロイする方法について説明します。 同じアプローチを使用して、他の Cognitive Services コンテナーを Azure Stack Hub の Kubernetes クラスターにデプロイすることができます。

## <a name="prerequisites"></a>前提条件

手順を開始する前に次が必要となります。

1.  Azure Cognitive Services コンテナー レジストリから Face コンテナー イメージをプルするには、コンテナー レジストリへのアクセスを要求します。 詳細については、「[プライベート コンテナー レジストリへのアクセスの要求](/azure/cognitive-services/face/face-how-to-install-containers#request-access-to-the-private-container-registry)」を参照してください。

2.  Azure Stack Hub で Kubernetes クラスターを準備します。 [Azure Stack Hub への Kubernetes のデプロイ](azure-stack-solution-template-kubernetes-deploy.md)に関する記事の手順に従うことができます。

## <a name="create-azure-resources"></a>Azure リソースを作成する

Face、Language Understanding (LUIS)、またはテキスト認識コンテナーをプレビューする Cognitive Service リソースを Azure で作成します。 Cognitive Services コンテナーをインスタンス化するために、リソースからのサブスクリプション キーとエンドポイント URL を使用する必要があります。

1. Azure portal で Azure リソースを作成します。 Face コンテナーをプレビューする場合、まず Azure portal で対応する Face リソースを作成する必要があります。 詳細については、「[クイック スタート: Azure portal で Cognitive Services アカウントを作成する](/azure/cognitive-services/cognitive-services-apis-create-account)」を参照してください。

   > [!NOTE]
   >  Face または Computer Vision リソースでは、F0 価格レベルを使用する必要があります。

2. Azure リソースのエンドポイント URL とサブスクリプション キーを取得します。 Azure リソースを作成したら、そのリソースからのサブスクリプション キーとエンドポイント URL を使用して、対応する Face、LUIS、またはテキスト認識コンテナーをプレビュー用にインスタンス化します。

## <a name="create-a-kubernetes-secret"></a>Kubernetes シークレットを作成する 

プライベート コンテナー レジストリにアクセスするには、Kubectl シークレット作成コマンドを使用します。 `<username>` と `<password>` を Azure Cognitive Services チームから受け取った資格情報に指定されているユーザー名とパスワードにそれぞれ置き換えます。

```bash  
    kubectl create secret docker-registry <secretName> \
        --docker-server='containerpreview.azurecr.io' \
        --docker-username='<username>' \
        --docker-password='<password>' 
```

## <a name="prepare-a-yaml-configure-file"></a>YAML 構成ファイルの準備

YAML 構成ファイルを使用して Kubernetes クラスターの Cognitive Service のデプロイを簡略化します。

これは、Face サービスを Azure Stack Hub にデプロイするためのサンプル YAML 構成ファイルです。

```Yaml  
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: <deploymentName>
spec:
  replicas: <replicaNumber>
  template:
    metadata:
      labels:
        app: <appName>
    spec:
      containers:
      - name: <containersName>
        image: <ImageLocation>
        env: 
        - name: EULA
          value: accept 
        - name: Billing
          value: <billingURL> 
        - name: apikey
          value: <apiKey>
        tty: true
        stdin: true
        ports:
        - containerPort: 5000 
      imagePullSecrets:
        - name: <secretName>
---
apiVersion: v1
kind: Service
metadata:
  name: <LBName>
spec:
  type: LoadBalancer
  ports:
  - port: 5000
    targetPort : 5000
    name: <PortsName>
  selector:
    app: <appName>
```

この YAML 構成ファイルで、Azure Container Registry から Cognitive Service コンテナー イメージを取得するために使うシークレットを使用します。 シークレット ファイルを使用して、コンテナーの特定のレプリカをデプロイします。 ユーザーが外部からこのサービスにアクセスできるようにするために、ロード バランサーも作成できます。

キー フィールドに関する詳細:

| フィールド | Notes |
| --- | --- |
| replicaNumber | 作成するインスタンスの初期レプリカを定義します。 スケーリングはデプロイ後に可能になります。 |
| ImageLocation | ACR での特定の Cognitive Service コンテナー イメージの場所を示します。 たとえば、顔サービスの場合は次のようになります: `aicpppe.azurecr.io/microsoft/cognitive-services-face` |
| BillingURL |「[Azure リソースを作成する](#create-azure-resources)」の手順で書き留めたエンドポイント URL |
| ApiKey | 「[Azure リソースを作成する](#create-azure-resources)」の手順で書き留めたサブスクリプション キー |
| SecretName | 「[Kubernetes シークレットを作成する](#create-a-kubernetes-secret)」の手順で作成したシークレットの名前 |

## <a name="deploy-the-cognitive-service"></a>Cognitive Services のデプロイ

Cognitive Services コンテナーをデプロイするには、次のコマンドを使用します。

```bash  
    Kubectl apply -f <yamlFileName>
```
デプロイ方法を監視するには、次のコマンドを使用します。 
```bash  
    Kubectl get pod - watch
```

## <a name="configure-http-proxy-settings"></a>HTTP プロキシ設定を構成する

ワーカー ノードにはプロキシと SSL が必要です。 送信要求を行うために HTTP プロキシを構成する必要がある場合は、次の 2 つの引数を使用します。

- **HTTP_PROXY** - 使用するプロキシ。例: `https://proxy:8888`
- **HTTP_PROXY_CREDS** - プロキシで認証されるために必要な資格情報。例: `username:password`

### <a name="set-up-the-proxy"></a>プロキシを設定する

1. `http-proxy.conf` ファイルを両方の場所に追加します。
    - `/etc/system/systemd/docker.service.d/`
    - `/cat/etc/environment/`

2. Cognitive Services チームから提供された資格情報を使用してコンテナーにサインオンして次のコンテナーで `docker pull` を実行できることを検証します。 

    `docker pull containerpreview.azurecr.io/microsoft/cognitive-services-read:latest`

    次のコマンドを実行します。

    `docker run hello-world pull`

### <a name="ssl-interception-setup"></a>SSL インターセプトの設定

1. **https インターセプション** 証明書を `/usr/local/share/ca-certificates` に追加し `update-ca-certificates`でストアを更新します。 

## <a name="test-the-cognitive-service"></a>Cognitive Service をテストする

[Open API 仕様](https://swagger.io/docs/specification/about/)には、そのコンテナーの相対 URI **/swagger** からアクセスします。 以前は Swagger 仕様と呼ばれていたこの仕様では、インスタンス化されたコンテナーによってサポートされる操作について説明しています。 たとえば、次の URI から、前の例でインスタンス化された感情分析コンテナーの OpenAPI 仕様にアクセスできます。

```HTTP  
http:<External IP>:5000/swagger
```

外部 IP アドレスは、次のコマンドで取得できます。 

```bash  
    Kubectl get svc <LoadBalancerName>
```

## <a name="try-the-services-with-python"></a>Python を使用してサービスを試す

いくつかの単純な Python スクリプトを実行して、Azure Stack Hub で Cognitive Services の検証を試みることができます。 参照用に [Computer Vision](/azure/cognitive-services/computer-vision/home)、[Face](/azure/cognitive-services/face/overview)、[Text Analytics](/azure/cognitive-services/text-analytics/overview)、および [Language Understanding](/azure/cognitive-services/luis/luis-container-howto) (LUIS) の Python 公式クイックスタート サンプルがあります。

Python アプリを使用してコンテナーで実行されているサービスを検証する場合は、次の 2 つのことを念頭に置いておく必要があります。 
1. コンテナー内の Cognitive Services の認証にはサブ キーは必要ありませんが、SDK を満たすために、プレースホルダーとして何らかの文字列が必要です。 
2. base_URL を実際のサービス エンドポイントの IP アドレスに置き換えます。

ここでは、イメージの中で顔を検出してフレームするために、サンプルの Python スクリプトが Face サービスの Python SDK を使用しています。

```Python  
import cognitive_face as CF

# Cognitive Services in container do not need sub keys for authentication
# Keep the invalid key to satisfy the SDK inputs requirement. 
KEY = '0'  #  (keeping the quotes in place).
CF.Key.set(KEY)

# Get your actual Ip Address of service endpoint of your cognitive service on Azure Stack Hub
BASE_URL = 'http://<svc IP Address>:5000/face/v1.0/'  
CF.BaseUrl.set(BASE_URL)

# You can use this example JPG or replace the URL below with your own URL to a JPEG image.
img_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
faces = CF.face.detect(img_url)
print(faces)

```

## <a name="next-steps"></a>次のステップ

[Computer Vision API コンテナーをインストールして実行する方法](/azure/cognitive-services/computer-vision/computer-vision-how-to-install-containers)

[Face API コンテナーをインストールして実行する方法](/azure/cognitive-services/face/face-how-to-install-containers)

[Text Analytics API コンテナーをインストールして実行する方法](/azure/cognitive-services/text-analytics/how-tos/text-analytics-how-to-install-containers)

[Language Understanding (LIUS) コンテナーをインストールして実行する方法](/azure/cognitive-services/luis/luis-container-howto)
