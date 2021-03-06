---
title: "Logic Apps でのオンプレミス データ ゲートウェイによるファイル システム コネクタの使用 | Microsoft Docs"
description: "Logic Apps では、オンプレミス ファイル システムにワークフローの一部として簡単に接続することができます。"
services: logic-apps
documentationcenter: dev-center-name
author: derek1ee
manager: anneta
ms.service: logic-apps
ms.devlang: wdl
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/15/2016
ms.author: deli
translationtype: Human Translation
ms.sourcegitcommit: dc8c9eac941f133bcb3a9807334075bfba15de46
ms.openlocfilehash: f2ac0e44f6083d59fc58bc332bfeb5ff39223cce


---
# <a name="use-file-system-connector-with-on-premises-data-gateway"></a>オンプレミス データ ゲートウェイを使用したファイル システム コネクタの使用

ハイブリッド クラウドの接続は、Logic Apps の中核です。 オンプレミス データ ゲートウェイを使用すると、データを管理し、Logic Apps からオンプレミスのリソースに安全にアクセスできます。 この記事では、Dropbox、ファイル共有にアップロードされたファイルをコピーし、電子メールを送信するというシンプルなシナリオで、オンプレミスのファイル システムに接続する方法について説明します。

## <a name="prerequisites"></a>前提条件
- 最新の[オンプレミス データ ゲートウェイ](https://www.microsoft.com/en-us/download/details.aspx?id=53127)をインストールして構成します。

 まだインストールしていない場合は、最新のオンプレミス データ ゲートウェイのバージョン 1.15.6150.1 以降をインストールします。 詳しくは、[こちらの記事](http://aka.ms/logicapps-gateway)をご覧ください。 残りの手順を続行する前に、オンプレミス コンピューターにゲートウェイをインストールする必要があります。

## <a name="use-file-system-connector"></a>ファイル システム コネクタの使用

1. Dropbox の "ファイルが作成されたとき" トリガーを作成し、すべてのサポートされたファイル コネクタ アクションが、検索機能に "File System" と入力するのと同じくらいシンプルであることを簡単に確認してみましょう。

 ![ファイル コネクタの検索](media/logic-apps-using-file-connector/search-file-connector.png)

2. 「ファイルの作成」を選択し、接続を作成します。
 - 既存の接続がない場合は、作成するよう求めるメッセージが表示されます。
 - [Connect via on-premises data gateway (オンプレミス データ ゲートウェイ経由で接続する)] オプションをオンにします。チェックボックスをオンにすると、追加のフィールドが表示されます。
 - ルート フォルダーを指定します。これには、オンプレミスのデータ ゲートウェイがインストールされているコンピューター上のローカル フォルダー、またはコンピューターがアクセスできるネットワーク共有を指定できます。
 - ゲートウェイのユーザー名とパスワードを入力します。
 - 前の手順でインストールしたゲートウェイを選択します。
    
 > [!NOTE]
 > ルート フォルダーは、すべてのファイル関連のアクションの相対パスに使用されるメインの親フォルダーです。

 ![接続の構成](media/logic-apps-using-file-connector/create-file.png)

3. すべての詳細を指定したら、[作成] をクリックします。 Logic Apps により、接続が構成およびテストされ、適切に機能していることが確認されます。 すべてが確認されたら、以前に選択したカードのオプションが表示され、ファイル コネクタが使用できるようになります。

4. Dropbox にアップロードされたファイルを取得し、オンプレミスのファイル共有のルート フォルダーにコピーします。

 ![ファイル アクションの作成](media/logic-apps-using-file-connector/create-file-filled.png)

5. ファイルがコピーされたら、それを対象のユーザーに知らせる電子メールを送信しましょう。 他のコネクタのように、前のアクションの出力が "動的なコンテンツ" セレクターで利用できるようになります。
 - 受信者、タイトル、および電子メールの本文を指定します。 "動的なコンテンツ" セレクターを使用して、ファイル コネクタの出力を選択して電子メールを拡張します。

 ![電子メール送信アクション](media/logic-apps-using-file-connector/send-email.png)

6. Logic App を保存し、Dropbox にファイルをアップロードしてテストします。 オンプレミス ファイル共有にコピーされるファイルが表示され、アクションに関する電子メール通知が届きます。
 > [!TIP]
 > [Logic Apps を監視する](../logic-apps/logic-apps-monitor-your-logic-apps.md)方法を確認してください。

7. これで完了です。ファイル システム コネクタを使用した Logic App が機能するようになりました。 提供されるその他の機能の探索を開始できます。
    - ファイルを作成する
    - フォルダー内のファイルを一覧表示する
    - ファイルを追加する
    - ファイルを削除する
    - ファイルの内容を取得する
    - パスを使用してファイルの内容を取得する
    - ファイルのメタデータを取得する
    - パスを使用してファイルのメタデータを取得する
    - ルート フォルダー内のファイルを一覧表示する
    - ファイルを更新する

## <a name="next-steps"></a>次のステップ
- [Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md) についての詳細情報をご覧ください。 
- Logic Apps への[オンプレミス接続](../logic-apps/logic-apps-gateway-connection.md)を作成します。



<!--HONumber=Jan17_HO3-->


