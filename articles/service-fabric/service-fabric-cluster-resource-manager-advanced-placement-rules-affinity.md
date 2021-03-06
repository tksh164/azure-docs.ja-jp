---
title: "Service Fabric クラスター リソース マネージャー - アフィニティ | Microsoft Docs"
description: "Service Fabric サービスのアフィニティ設定の概要"
services: service-fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: 
ms.assetid: 678073e1-d08d-46c4-a811-826e70aba6c4
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/19/2016
ms.author: masnider
translationtype: Human Translation
ms.sourcegitcommit: 219dcbfdca145bedb570eb9ef747ee00cc0342eb
ms.openlocfilehash: ef1b4c5ab23365b62c3c1061061aec5a9155c3f4


---
# <a name="configuring-and-using-service-affinity-in-service-fabric"></a>Service Fabric でアフィニティを構成し、使用する
アフィニティは主に、大規模なモノリシック アプリケーションをクラウドとマイクロサービスの世界に移行しやすくするために提供されるコントロールです。 特定のケースでは、アフィニティをサービスのパフォーマンスを向上させるための正当な最適化として使用できますが、副作用が伴う場合があります。

たとえば、大規模なアプリや、マイクロサービスでの使用を想定していないアプリを Service Fabric に移行するとしましょう。 実際、このような状況は多く、これまでに複数の顧客 (内部、外部とも) が経験してきました。 まずは、アプリ全体を環境に移行し、パッケージ化して実行するところから始めます。 次に、すべてが相互に通信する小規模なサービスにアプリを分解します。

ここで、困ったことが起こります。 “困ったこと” は、通常、次のいずれかに分類されます。

1. モノリシック アプリのコンポーネント X とコンポーネント Y の間にドキュメントに未記載の依存関係があることが判明したが、これらを複数のサービスに分解したばかりです。 これらは現在、クラスター内の個々のノードで実行されているため壊れてしまいました。
2. 通信は (ローカルの名前付きパイプ | 共有メモリー | ディスク上のファイル) を介して行われるが、処理スピードを速めるため、個別に更新できるようにしたい。 ハードの依存関係は後で削除します。
3. すべて問題ないが、これら 2 つのコンポーネント間の通信が非常に多いこと、またパフォーマンスに影響しやすいことが判明しました。 別々のサービスに分解すると、アプリケーション パフォーマンス全体が低下し、また待機時間が増大しました。 その結果、アプリケーション全体で要件を満たせなくなりました。

これらのケースでは、リファクタリング作業を断念することや、モノリシックに戻すことは望ましくありません。しかし、あたかもローカルであると感じられるようにする必要があると考えています。 サービスとして自然に動作するようにコンポーネントを再設計するか、可能な場合、他の何らかの方法でパフォーマンス予測値を解決できるまで、この問題は解決されません。

どうすればよいでしょうか。 そこで、アフィニティを有効にします。

## <a name="how-to-configure-affinity"></a>アフィニティの構成方法
アフィニティを設定するには、2 つのサービスの間にアフィニティの関係を定義します。 アフィニティは一方のサービスから別のサービスを “ポイント” し、“このサービスはそのサービスの実行中にだけ実行できる“ と示すことと考えることができます。 アフィニティを親子関係と呼ぶ場合もあります (親から子をポイントする)。 アフィニティでは、サービスのレプリカまたはインスタンスが、別のレプリカまたはインスタンスと同じノード上に配置されるようになります。

``` csharp
ServiceCorrelationDescription affinityDescription = new ServiceCorrelationDescription();
affinityDescription.Scheme = ServiceCorrelationScheme.Affinity;
affinityDescription.ServiceName = new Uri("fabric:/otherApplication/parentService");
serviceDescription.Correlations.Add(affinityDescription);
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

## <a name="different-affinity-options"></a>アフィニティのさまざまなオプション
アフィニティは、いくつかの相関スキームのいずれかで表され、2 種類のモードがあります。 アフィニティの最も一般的なモードは、NonAlignedAffinity と呼ばれるものです。 NonAlignedAffinity では、異なるサービスのレプリカまたはインスタンスが同じノード上に配置されます。 もう 1 つのモードは、AlignedAffinity です。 Aligned Affinity を使用できるのはステートフル サービスのみです。 2 つのステートフル サービスを整列したアフィニティが含まれるように構成すると、これらのサービスのプライマリがペアで同じノード上に配置されます。 また、これらのサービスのセカンダリの各ペアも同じノードに配置されます。 ステートフル サービスに対して NonAlignedAffinity を構成することもできます (ただし、あまり一般的ではありません)。 NonAlignedAffinity では、2 つのステートフル サービスの異なるレプリカが同じノード上に配置されますが、プライマリまたはセカンダリの整列は試行されません。

![アフィニティのモードとその影響][Image1]

### <a name="best-effort-desired-state"></a>ベスト エフォートの望まれる状態
アフィニティとモノリシックなアーキテクチャにはいくつかの違いがあります。 これらの多くは、アフィニティの関係がベスト エフォートであることに起因します。 アフィニティの関係内のサービスは根本的に、失敗し、個別に移動する可能性のある異なるエンティティです。 また、アフィニティの関係が損なわれるのにも原因があります。 たとえば、アフィニティ関係の一部のサービス オブジェクトのみを特定のノードに配置できる場合の容量制限が挙げられます。 このような場合、アフィニティの関係が存在する場合でも、その他の制約によりその関係を適用できません。 後でその他のすべての制約とのアフィニティを適用することが可能になっている場合、アフィニティ制約違反は自動的に修正されます。  

### <a name="chains-vs-stars"></a>チェーンと星
現在、アフィニティの関係のチェーンをモデル化することはできません。 これは、あるアフィニティの関係で子になっているサービスは、別のアフィニティの関係では親になれないことを意味します。 この種の関係をモデル化するには、実質的にはチェーンではなく、星としてモデル化する必要があります。 これを行うには、最下位の子を子の "中央" にいる親にします。 サービスの配置によっては、複数の子要素の親として機能する "プレースホルダー" サービスを作成する必要がある場合があります。

![アフィニティの関係のコンテキストにおけるチェーンと星][Image2]

アフィニティの関係について注目すべきもう 1 つの点は、方向があるということです。 これは、"アフィニティ" ルールでは、子が親と同じ場所にあるということのみが強制されることを意味します。 たとえば、親が突然別のノードにフェール オーバーした場合、クラスター リソース マネージャーは子が親と同じ場所にないことが通知されるまで、異常に気付きません。つまり、関係はすぐには適用されません。

### <a name="partitioning-support"></a>パーティション分割のサポート
アフィニティについて注目すべき最後の点は、親がパーティション分割されている場合、アフィニティの関係がサポートされないということです。 これは、最終的にはサポートされる可能性がありますが、今のところ許可されていません。

## <a name="next-steps"></a>次のステップ
* サービスの構成に利用できるその他のオプションの詳細については、「 [サービスの構成について学習する](service-fabric-cluster-resource-manager-configure-services.md)
* サービスを少数のマシンに制限してサービスのコレクションの負荷を集約する場合など、アフィニティを使おうとする理由の多くは、アプリケーション グループによってより強力にサポートされます。 詳しくは、「 [アプリケーション グループ](service-fabric-cluster-resource-manager-application-groups.md)

[Image1]:./media/service-fabric-cluster-resource-manager-advanced-placement-rules-affinity/cluster-resrouce-manager-affinity-modes.png
[Image2]:./media/service-fabric-cluster-resource-manager-advanced-placement-rules-affinity/cluster-resource-manager-chains-vs-stars.png



<!--HONumber=Nov16_HO3-->


