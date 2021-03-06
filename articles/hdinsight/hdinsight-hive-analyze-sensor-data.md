---
title: "Hive と Hadoop を使用したセンサー データの分析 | Microsoft Docs"
description: "Hive クエリ コンソールを HDInsight (Hadoop) と共に使用してセンサー データを分析してから、Microsoft Excel の Power View を使用してデータを視覚化する方法について説明します。"
services: hdinsight
documentationcenter: 
author: Blackmist
manager: jhubbard
editor: cgronlun
tags: azure-portal
ms.assetid: a8ac160c-1cef-45d9-bf36-7beb5a439105
ms.service: hdinsight
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/20/2016
ms.author: larryfr
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: ef2e4b9272389f4e027821e84df4499bf8987c39


---
# <a name="analyze-sensor-data-using-the-hive-query-console-on-hadoop-in-hdinsight"></a>HDInsight の Hadoop で Hive クエリ コンソールを使用したセンサー データの分析
Hive クエリ コンソールを HDInsight (Hadoop) と共に使用してセンサー データを分析してから、Microsoft Excel の Power View を使用してデータを視覚化する方法について説明します。

> [!NOTE]
> このドキュメントの手順は、Windows ベースの HDInsight クラスターに対してのみ機能します。
> 
> 

このサンプルでは、Hive を使用して、暖房、換気、および空調 (HVAC) システムによって生成された履歴データを処理し、設定した温度を正確に維持できないシステムを識別します。 学習内容:

* 値をコンマで区切った (CSV) 形式で格納されたデータを照会するための HIVE テーブルの作成
* データを分析するための HIVE クエリの作成
* 分析したデータを取得するための、Microsoft Excel を使用した HDInsight への接続 (Open Database Connectivity (ODBC) を使用)
* Power View を使用したデータの視覚化

![A diagram of the solution architecture](./media/hdinsight-hive-analyze-sensor-data/hvac-architecture.png)

## <a name="prerequisites"></a>前提条件
* HDInsight (Hadoop) クラスター - クラスター作成の詳細については、「[HDInsight での Hadoop クラスターのプロビジョニング](hdinsight-provision-clusters.md)」をご覧ください。
* Microsoft Excel 2013
  
  > [!NOTE]
  > Microsoft Excel は、 [Power View](https://support.office.com/Article/Power-View-Explore-visualize-and-present-your-data-98268d31-97e2-42aa-a52b-a68cf460472e?ui=en-US&rs=en-US&ad=US)を使用したデータ視覚化のために使用します。
  > 
  > 
* [Microsoft Hive ODBC ドライバー](http://www.microsoft.com/download/details.aspx?id=40886)

## <a name="to-run-the-sample"></a>サンプルを実行するには
1. Web ブラウザーで次の URL に移動します。 `<clustername>` を、使用する HDInsight クラスターの名前に置き換えます。
   
         https://<clustername>.azurehdinsight.net
   
    入力を要求されたら、このクラスターをプロビジョニングするときに使用した管理者ユーザー名とパスワードを使用して認証を実行します。
2. 表示された Web ページで **[概要ギャラリー]** タブをクリックし、**[Solutions with Sample Data (サンプル データを使用した解決策)]** カテゴリにある **[センサー データ分析]** サンプルを選択します。
   
    ![ギャラリー イメージの概要](./media/hdinsight-hive-analyze-sensor-data/getting-started-gallery.png)
3. Web ページに記載されている手順に従って、サンプルを完了します。




<!--HONumber=Nov16_HO3-->


