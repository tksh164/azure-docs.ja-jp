---
title: "Service Bus キューの使用方法 (PHP) | Microsoft Docs"
description: "Azure での Service Bus キューの使用方法を学習します。 コード サンプルは PHP で記述されています。"
services: service-bus-messaging
documentationcenter: php
author: sethmanheim
manager: timlt
editor: 
ms.assetid: e29c829b-44c5-4350-8f2e-39e0c380a9f2
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: PHP
ms.topic: article
ms.date: 10/04/2016
ms.author: sethm
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 1d89a2f0a7f36bf9e2052682e931ac5c3596721f


---
# <a name="how-to-use-service-bus-queues"></a>Service Bus キューの使用方法
[!INCLUDE [service-bus-selector-queues](../../includes/service-bus-selector-queues.md)]

このガイドでは、Service Bus キューの使用方法について説明します。 サンプルは PHP で記述され、[Azure SDK for PHP](../php-download-sdk.md) を利用しています。 紹介するシナリオは、**キューの作成**、**メッセージの送受信**、**キューの削除**です。

[!INCLUDE [howto-service-bus-queues](../../includes/howto-service-bus-queues.md)]

## <a name="create-a-php-application"></a>PHP アプリケーションの作成
Microsoft Azure BLOB Service にアクセスする PHP アプリケーションを作成するための要件は、コード内から [Microsoft Azure SDK for PHP ](../php-download-sdk.md)のクラスを参照することのみです。 アプリケーションの作成には任意の開発ツールまたはメモ帳を使用できます。

> [!NOTE]
> PHP のインストールでは、[OpenSSL 拡張機能](http://php.net/openssl)をインストールして有効にしておく必要もあります。
> 
> 

このガイドで使用するサービス機能は、PHP アプリケーション内からローカルで呼び出すことも、Azure の Web ロール、worker ロール、または Web サイト上で実行されるコード内で呼び出すこともできます。

## <a name="get-the-azure-client-libraries"></a>Azure クライアント ライブラリの入手
[!INCLUDE [get-client-libraries](../../includes/get-client-libraries.md)]

## <a name="configure-your-application-to-use-service-bus"></a>Service Bus を使用するようにアプリケーションを構成する
Service Bus キュー API を使用するには、次の操作を行います。

1. [require_once][require_once] ステートメントを使用してオートローダー ファイルを参照します。
2. 使用する可能性のあるクラスを参照する

次の例では、オートローダー ファイルをインクルードし、 **ServicesBuilder** クラスを参照する方法を示しています。

> [!NOTE]
> この例 (およびこの記事のその他の例) では、Composer を使用して Azure 向け PHP クライアント ライブラリがインストールされていることを前提としています。 ライブラリを手動でまたは PEAR パッケージとしてインストールした場合は、**WindowsAzure.php** オートローダー ファイルを参照する必要があります。
> 
> 

```
require_once 'vendor/autoload.php';
use WindowsAzure\Common\ServicesBuilder;
```

下のすべてのサンプルに `require_once` ステートメントが入っていますが、サンプルの実行に必要なクラスのみが参照されます。

## <a name="set-up-a-service-bus-connection"></a>Service Bus 接続の設定
Service Bus クライアントをインスタンス化するには、まず次の形式の有効な接続文字列が必要です。

```
Endpoint=[yourEndpoint];SharedSecretIssuer=[Default Issuer];SharedSecretValue=[Default Key]
```

ここで **Endpoint** の一般的な形式は `[yourNamespace].servicebus.windows.net` です。

いずれの Azure サービス クライアントを作成するにも、**ServicesBuilder** クラスを使用する必要があります。 そのための方法は次のとおりです。

* 接続文字列を直接渡す
* **CloudConfigurationManager (CCM)** を使用して複数の外部ソースに対して接続文字列を確認する
  * 既定では 1 つの外部ソース (環境変数) のみがサポートされています。
  * **ConnectionStringSource** クラスを継承して新しいソースを追加できます。

ここで概説している例では、接続文字列を直接渡します。

```
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;

$connectionString = "Endpoint=[yourEndpoint];SharedSecretIssuer=[Default Issuer];SharedSecretValue=[Default Key]";

$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);
```

## <a name="how-to-create-a-queue"></a>方法: キューを作成する
Service Bus キューの管理処理は **ServiceBusRestProxy** クラスを使用して実行できます。 **ServiceBusRestProxy** オブジェクトを作成するには、**ServicesBuilder::createServiceBusService** ファクトリ メソッドに、管理処理用のトークン アクセス許可をカプセル化した適切な接続文字列を渡します。

次の例では、**ServiceBusRestProxy** をインスタンス化し、**ServiceBusRestProxy->createQueue** を呼び出して、`MySBNamespace` サービス名前空間内で `myqueue` という名前のキューを作成する方法を示しています。

```
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\QueueInfo;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try    {
    $queueInfo = new QueueInfo("myqueue");

    // Create queue.
    $serviceBusRestProxy->createQueue($queueInfo);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // http://msdn.microsoft.com/library/windowsazure/dd179357
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

> [!NOTE]
> `ServiceBusRestProxy` オブジェクトの `listQueues` メソッドを使用すると、指定した名前のキューが名前空間に既に存在するかどうかを確認できます。
> 
> 

## <a name="how-to-send-messages-to-a-queue"></a>方法: キューにメッセージを送信する
メッセージを Service Bus キューに送信するには、アプリケーションで **ServiceBusRestProxy->sendQueueMessage** メソッドを呼び出します。 次のコードでは、上のコードで `MySBNamespace` サービス名前空間内で作成した `myqueue` キューにメッセージを送信する方法を示しています。

```
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\BrokeredMessage;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try    {
    // Create message.
    $message = new BrokeredMessage();
    $message->setBody("my message");

    // Send message.
    $serviceBusRestProxy->sendQueueMessage("myqueue", $message);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // http://msdn.microsoft.com/library/windowsazure/hh780775
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

Service Bus キューに送信されたメッセージ (および Service Bus キューから受信したメッセージ) は、**BrokeredMessage** クラスのインスタンスになります。 **BrokeredMessage** オブジェクトには (**getLabel**、**getTimeToLive**、**setLabel**、**setTimeToLive** など) 一連の標準メソッド、カスタム アプリケーション固有プロパティを保有するためのプロパティ、任意のアプリケーション データの本体があります。

Service Bus キューでサポートされているメッセージの最大サイズは、[Standard レベル](service-bus-premium-messaging.md)では 256 KB、[Premium レベル](service-bus-premium-messaging.md)では 1 MB です。 標準とカスタムのアプリケーション プロパティが含まれるヘッダーの最大サイズは 64 KB です。 キューで保持されるメッセージ数には上限がありませんが、キュー 1 つあたりが保持できるメッセージの合計サイズには上限があります。 このキュー サイズの上限は 5 GB です。

## <a name="how-to-receive-messages-from-a-queue"></a>キューからメッセージを受信する方法
キューからメッセージを受信する最良の方法は **ServiceBusRestProxy->receiveQueueMessage** メソッドを使用することです。 メッセージは 2 つの異なるモード (**ReceiveAndDelete** (既定) と **PeekLock**) で受信できます。

**ReceiveAndDelete** モードを使用する場合、受信が 1 回ずつの動作になります。つまり、Service Bus はキュー内のメッセージに対する読み取り要求を受け取ると、メッセージを読み取り中としてマークし、アプリケーションに返します。 **ReceiveAndDelete** モードは最もシンプルなモデルであり、障害発生時にアプリケーション側でメッセージを処理しないことを許容できるシナリオに最適です。 このことを理解するために、コンシューマーが受信要求を発行した後で、メッセージを処理する前にクラッシュしたというシナリオを考えてみましょう。 Service Bus はメッセージを読み取り済みとしてマークするため、アプリケーションが再起動してメッセージの読み取りを再開すると、クラッシュ前に読み取られていたメッセージは見落とされることになります。

**PeekLock** モードでは、メッセージの受信処理が 2 段階の動作になり、メッセージが失われることが許容できないアプリケーションに対応することができます。 Service Bus は要求を受け取ると、次に読み取られるメッセージを検索して、他のコンシューマーが受信できないようロックしてから、アプリケーションにメッセージを返します。 アプリケーションがメッセージの処理を終えた後 (または後で処理するために確実に保存した後)、受信したメッセージを **ServiceBusRestProxy->deleteMessage** に渡して受信処理の第 2 段階を完了します。 Service Bus が **deleteMessage** の呼び出しを確認すると、メッセージが読み取り中としてマークされ、キューから削除されます。

次の例では、**PeekLock** モード (既定ではないモード) を使用したメッセージの受信および処理の方法を示しています。

```
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\ReceiveMessageOptions;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try    {
    // Set the receive mode to PeekLock (default is ReceiveAndDelete).
    $options = new ReceiveMessageOptions();
    $options->setPeekLock();

    // Receive message.
    $message = $serviceBusRestProxy->receiveQueueMessage("myqueue", $options);
    echo "Body: ".$message->getBody()."<br />";
    echo "MessageID: ".$message->getMessageId()."<br />";

    /*---------------------------
        Process message here.
    ----------------------------*/

    // Delete message. Not necessary if peek lock is not set.
    echo "Message deleted.<br />";
    $serviceBusRestProxy->deleteMessage($message);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // http://msdn.microsoft.com/library/windowsazure/hh780735
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>方法: アプリケーションのクラッシュと読み取り不能のメッセージを処理する
Service Bus には、アプリケーションにエラーが発生した場合や、メッセージの処理に問題がある場合に復旧を支援する機能が備わっています。 受信側のアプリケーションが何らかの理由によってメッセージを処理できない場合には、受信したメッセージについて (**deleteMessage** メソッドの代わりに) **unlockMessage** メソッドを呼び出すことができます。 このメソッドが呼び出されると、Service Bus によってキュー内のメッセージのロックが解除され、メッセージが再度受信できる状態に変わります。メッセージを受信するアプリケーションは、以前と同じものでも、別のものでもかまいません。

キュー内でロックされているメッセージには、タイムアウトも設定されています。アプリケーションがクラッシュした場合など、ロックがタイムアウトになる前にアプリケーションがメッセージの処理に失敗した場合は、Service Bus によってメッセージのロックが自動的に解除され、再度受信できる状態に変わります。

メッセージが処理された後、**deleteMessage** 要求が発行される前にアプリケーションがクラッシュした場合は、アプリケーションが再起動する際にメッセージが再配信されます。 一般的に、この動作は "**1 回以上の処理**" と呼ばれます。つまり、すべてのメッセージが 1 回以上処理されますが、特定の状況では、同じメッセージが再配信される可能性があります。 重複処理が許されないシナリオの場合、重複メッセージの配信を扱うロジックをアプリケーションに追加することをお勧めします。 通常、この問題はメッセージの **getMessageId** メソッドを使用して対処します。このプロパティは、配信が試行された後も同じ値を保持します。

## <a name="next-steps"></a>次のステップ
これで、Service Bus キューの基本を学習できました。詳細については、[「キュー、トピック、サブスクリプション」][キュー、トピック、サブスクリプション]を参照してください。

詳細については、[PHP デベロッパー センター](/develop/php/)も参照してください。

[キュー、トピック、サブスクリプション]: service-bus-queues-topics-subscriptions.md
[require_once]: http://php.net/require_once





<!--HONumber=Nov16_HO3-->


