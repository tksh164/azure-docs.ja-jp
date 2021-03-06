---
title: "Azure Function App と Azure Storage アカウントの作成 | Microsoft Docs"
description: "Azure Function App は、Azure IoT ハブのイベントをリッスンして受信メッセージを処理し、それらを Azure Table Storage に書き込みます。"
services: iot-hub
documentationcenter: 
author: shizn
manager: timlt
tags: 
keywords: "クラウドへのデータの格納, クラウドに格納されているデータ, IoT クラウド サービス"
ms.assetid: 6c58de85-c5c4-4989-bb5e-08c45c549966
ms.service: iot-hub
ms.devlang: node
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/28/2016
ms.author: xshi
translationtype: Human Translation
ms.sourcegitcommit: ffcb9214b8fa645a8a2378c5e7054b9f984addbb
ms.openlocfilehash: df446b1a6b244761f39bfdcebb7e404435c2c35d


---
# <a name="create-an-azure-function-app-and-azure-storage-account"></a>Azure Function App と Azure Storage アカウントの作成
[Azure Functions](../azure-functions/functions-overview.md) は、"*関数*" (小規模なコード) をクラウドで手軽に実行するためのソリューションです。 Azure Function App は、Azure での関数の実行をホストします。

## <a name="what-you-will-do"></a>学習内容
Azure Resource Manager テンプレートを使用して、Azure Function App と Azure Storage アカウントを作成します。 Azure Function App は、Azure IoT ハブのイベントをリッスンして受信メッセージを処理し、それらを Azure Table Storage に書き込みます。 問題が発生した場合は、[トラブルシューティングのページ](iot-hub-raspberry-pi-kit-node-troubleshooting.md)で解決策を探してください。

## <a name="what-you-will-learn"></a>学習内容
この記事では、次のことについて説明します。

* [Azure Resource Manager](../azure-resource-manager/resource-group-overview.md) を使用して Azure リソースをデプロイする方法。
* Azure Function App を使用し、IoT ハブのメッセージを処理して Azure Table Storage のテーブルに書き込む方法。

## <a name="what-you-need"></a>必要なもの
次の作業を完了している必要があります。
* [Raspberry Pi 3 の使用](iot-hub-raspberry-pi-kit-node-get-started.md)
* [Azure IoT ハブの作成](iot-hub-raspberry-pi-kit-node-get-started.md)

## <a name="open-the-sample-app"></a>サンプル アプリを開く
次のコマンドを実行して、Visual Studio Code でサンプル プロジェクトを開きます。

```bash
cd Lesson3
code .
```

![リポジトリの構造](media/iot-hub-raspberry-pi-lessons/lesson3/repo_structure.png)

* `app` サブフォルダーの `app.js` ファイルは、キー ソース ファイルです。 このソース ファイルには、メッセージを IoT ハブに 20 回送信し、メッセージを送信するたびに LED を点滅させるコードが含まれています。
* `arm-template.json` ファイルは、Azure Function App と Azure Storage アカウントが含まれた Azure Resource Manager テンプレートです。
* `arm-template-param.json` ファイルは、Azure Resource Manager テンプレートによって使用される構成ファイルです。
* `ReceiveDeviceMessages` サブフォルダーは、Azure 関数の Node.js コードが含まれています。

## <a name="configure-azure-resource-manager-templates-and-create-resources-in-azure"></a>Azure Resource Manager テンプレートの構成と Azure でのリソースの作成
Visual Studio Code で `arm-template-param.json` ファイルを更新します。

![Azure Resource Manager テンプレートのパラメーター](media/iot-hub-raspberry-pi-lessons/lesson3/arm_para.png)

* **[your IoT Hub name]** を、[IoT ハブを作成して Raspberry Pi 3 を登録](iot-hub-raspberry-pi-kit-node-lesson2-prepare-azure-iot-hub.md)したときに指定した **{ハブ名}** で置き換えます。
* **[prefix string for new resources]** を任意のプレフィックスで置き換えます。 このプレフィックスによって、必ずリソース名をグローバルに一意にでき、競合を避けることができます。 ダッシュまたは番号のイニシャルは、プレフィックスに使用しないようにしてください。

`arm-template-param.json` ファイルの更新後、次のコマンドを実行してリソースを Azure にデプロイします。

```bash
az group deployment create --template-file arm-template.json --parameters @arm-template-param.json -g iot-sample
```

これらのリソースの作成には約 5 分かかります。 リソースの作成の進行中に、次の記事に進むことができます。

## <a name="summary"></a>概要
IoT ハブのメッセージを処理する Azure Function App と、それらのメッセージを格納する Azure Storage アカウントを作成しました。 デバイスからクラウドへのメッセージを送信するサンプルを Pi にデプロイして実行できます。

## <a name="next-steps"></a>次のステップ
[Raspberry Pi 3 での、デバイスからクラウドへのメッセージを送信するサンプル アプリケーションの実行](iot-hub-raspberry-pi-kit-node-lesson3-run-azure-blink.md)




<!--HONumber=Nov16_HO5-->


