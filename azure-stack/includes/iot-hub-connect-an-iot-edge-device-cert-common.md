---
author: BryanLa
ms.author: bryanla
ms.service: azure-stack
ms.topic: include
ms.date: 11/20/2020
ms.reviewer: bryanla
ms.lastreviewed: 11/20/2020
ms.openlocfilehash: 6d4bb587ad28fb26adf60689989d0e1858519cfe
ms.sourcegitcommit: 0765de47f4a73e09192d34739e40c750b6e7abaf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/27/2021
ms.locfileid: "98920848"
---
### <a name="create-a-device-root-ca-certificate"></a>デバイス ルート CA 証明書を作成する

デバイスに必要なデバイス ルート CA 証明書とキー ファイルを作成します。 

1. PuTTY セッションで Bash コマンド プロンプトに戻ります。
2. 前のセクションで証明書生成スクリプトを置いたデータ ディレクトリに移動します。
3. 次のコマンドを実行します。

   ```bash
   ./certGen.sh create_root_and_intermediate
   ```

4. デバイス ルート CA 証明書は、`<DATA-DIR>/certs/azure-iot-test-only.root.ca.cert.pem` ファイルに格納されます。

### <a name="create-the-iot-edge-device-ca-certificate"></a>IoT Edge デバイス CA 証明書を作成する

運用環境の IoT Edge デバイスには、config.yaml ファイルから参照されるデバイス CA 証明書が必要です。 デバイス CA 証明書により、デバイスで実行されるモジュールのための証明書が作成されます。 また、デバイス CA 証明書は、IoT Edge デバイスによってダウンストリーム デバイスに対する ID の検証に使用されるため、ゲートウェイのシナリオにも必要です。

IoT Edge デバイス CA 証明書ファイルを作成するには:

1. PuTTY セッションで Bash コマンド プロンプトに戻ります。
2. 次のスクリプトを実行します。これにより、複数のデバイス CA 証明書とキー ファイルが作成されます。 

   ```bash
   # Navigate to data directory
   cd <DATA-DIR>
   
   # Generate IoT Edge device CA certificate 
   ./certGen.sh create_edge_device_ca_certificate <DEVICE-CA-CERT-NAME>
   ```

3.  次の証明書とキーのペアのファイルは、後で config.yaml ファイル内で参照されます。

    `<DATA-DIR>/certs/iot-edge-device-ca-<DEVICE-CA-CERT-NAME>-full-chain.cert.pem`  
    `<DATA-DIR>/private/iot-edge-device-ca-<DEVICE-CA-CERT-NAME>.key.pem`


