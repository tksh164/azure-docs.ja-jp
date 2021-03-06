---
title: "Azure Site Recovery: 単一 VMM サーバー上で Hyper-V 仮想マシンをレプリケートする | Microsoft Docs"
description: "この記事では、VMM サーバーが 1 台のみの場合に Hyper-V 仮想マシンをレプリケートする方法について説明します。"
services: site-recovery
documentationcenter: 
author: rayne-wiselman
manager: jwhit
editor: 
ms.assetid: 1c0c4b99-2a6a-4fa2-89fa-d19c39869be6
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: backup-recovery
ms.date: 11/14/2016
ms.author: raynew
translationtype: Human Translation
ms.sourcegitcommit: 7455d6f99ed8ceb401224f98105f7b651f55c724
ms.openlocfilehash: 0f84f13093e359e0c4365848965d871c30f4eba3


---
# <a name="replicate-hyper-v-virtual-machines-on-a-single-vmm-server"></a>単一 VMM サーバー上で Hyper-V 仮想マシンをレプリケートする

この記事では、デプロイ内に 1 つの VMM サーバーしかない場合に、VMM クラウドの Hyper-V ホスト サーバー上にある Hyper-V 仮想マシンをレプリケートする方法について説明します。

Azure には、リソースの作成と操作に関して、Azure Resource Manager とクラシックの 2 種類の[デプロイメント モデル](../azure-resource-manager/resource-manager-deployment-model.md)があります。 また、Azure にも 2 つのポータルがあります。クラシック デプロイメント モデルをサポートする Azure クラシック ポータルと、両方のデプロイメント モデルをサポートする Azure Portal です。 この記事では、Azure Portal でレプリケーションを設定する手順について説明します。

この記事の内容について質問がある場合は、記事の下部にある Disqus コメント欄または [Azure Recovery Services フォーラム](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr)に投稿してください。


VMM サーバーが 1 つしかないときに、VMM の Hyper-V ホストに配置された Hyper-V VM をレプリケートする場合、[Azure にレプリケート](site-recovery-vmm-to-azure.md)するか、単一の VMM サーバー上のクラウド間でレプリケートします。

クラウド間でレプリケートする場合、フェールオーバーと復旧がシームレスではなく、手動での手順が多数必要となるため、Azure にレプリケートすることをお勧めします。 VMM サーバーのみを使用してレプリケートする場合、次のことが可能です。

* 単一のスタンドアロン VMM サーバーでレプリケートする
* ストレッチ Windows クラスターにデプロイされている単一の VMM サーバーでレプリケートする

## <a name="replicate-across-sites-with-a-single-standalone-vmm-server"></a>単一スタンドアロンの VMM サーバーでのサイト間レプリケート
![スタンドアロンの仮想 VMM サーバー](./media/site-recovery-single-vmm/single-vmm-standalone.png)

このシナリオでは、単一の VMM サーバーを仮想マシンとしてプライマリ サイトにデプロイし、Site Recovery と Hyper-V レプリカを使用して、この VM をセカンダリ サイトにレプリケートします。

1. HYPER-V 仮想マシン上の VMM を設定します。 時間を節約するために、VMM で使用される SQL Server インスタンスを同じ VM にインストールすることをお勧めします。 SQL Server のリモート インスタンスを使用したときに障害が発生した場合、VMM を回復するには、そのインスタンスを回復しておく必要があります。
2. VMM サーバーには少なくとも 2 つのクラウドを構成してください。 一方のクラウドにはレプリケートする VM が含まれ、もう一方のクラウドはセカンダリ ロケーションとして機能します。 保護する VM を含むクラウドには、以下が存在する必要があります。

   - 各グループに 1 つまたは複数の Hyper-V ホスト サーバーが登録されている 1 つまたは複数の VMM ホスト グループ。
   - 各 Hyper-V ホスト サーバー上の少なくとも 1 つの Hyper-V 仮想マシン。
3. Recovery Services コンテナーを作成し、コンテナー登録キーを生成、ダウンロードして、コンテナーに VMM サーバーを登録します。 登録時に、Azure Site Recovery Provider を VMM サーバーにインストールします。
4. VMM VM 上に 1 つ以上のクラウドをセットアップし、Hyper-V ホストをそれらのクラウドに追加します。
5. クラウドの保護設定を構成します。 ソースとターゲットの場所として、単一の VMM サーバーの名前を指定します。 ネットワーク マッピングを構成するには、保護する VM を含むクラウドの VM ネットワークを、レプリケート先のクラウドの VM ネットワークにマップします。
6. 両方のクラウドが同じサーバー上にあるため、ネットワーク経由で VM の初期レプリケーションを有効にできます。
7. Hyper-V マネージャー コンソールで、VMM VM を含む Hyper-V ホスト上での Hyper-V レプリカを有効にし、それから VM 上のレプリケーションを有効にします。 Site Recovery で保護されているクラウドには VMM VM を追加しないようにします。 これにより、Hyper-V レプリカの設定が Site Recovery によって上書きされないようにすることができます。
8. 復旧計画を作成する場合は、ソースとターゲットに同じ VMM サーバーを指定します。

[こちらの記事](site-recovery-vmm-to-vmm.md) に記載されている手順に従って、コンテナーを作成し、サーバーを登録して、保護を設定します。

### <a name="what-to-do-in-an-outage"></a>障害発生時の対処
システムが完全に停止し、セカンダリ サイトから操作する必要がある場合は、次の手順を実行します。

1. セカンダリ サイトの Hyper-V Manager コンソールで、計画されていないフェールオーバーを実行して、プライマリ サイトからセカンダリ サイトに VMM VM をフェールオーバーします。
2. VMM VM がセカンダリ サイトで稼働していることを確認します。
3. Recovery Services コンテナーで、計画されていないフェールオーバーを実行して、プライマリ クラウドからセカンダリ クラウドにワークロード VM をフェールオーバーします。 VM の計画されていないフェールオーバーを完了するには、フェールオーバーをコミットするか、必要に応じて別の復旧ポイントを選択します。
4. 計画されていないフェールオーバーが完了すると、ユーザーはセカンダリ サイトのワークロード リソースにアクセスできるようになります。

プライマリ サイトが再び正常に稼働するようになったら、次の手順を実行します。

1. Hyper-V Manager コンソールで、VMM VM のレプリケーションの反転を有効にして、セカンダリからプライマリへのレプリケーションを開始します。
2. Hyper-V Manager コンソールで、計画されたフェールオーバーを実行して、VMM VM をプライマリ サイトにフェールバックします。 フェールオーバーをコミットして完了します。 次に、レプリケーションの反転を有効にして、プライマリからセカンダリへの VMM のレプリケーションを開始します。
3. Recovery Services コンテナーで、ワークロード VM のレプリケーションの反転を有効にして、セカンダリからプライマリへのレプリケーションを開始します。
4. Recovery Services コンテナーで、計画されたフェールオーバーを実行して、ワークロード VM をプライマリ サイトにフェールバックします。 フェールオーバーをコミットして完了します。 次に、レプリケーションの反転を有効にして、プライマリからセカンダリへのワークロード VM のレプリケーションを開始します。

## <a name="replicate-across-sites-with-a-single-vmm-server-in-a-stretched-cluster"></a>ストレッチ クラスターの単一 VMM サーバーでのサイト間レプリケート
![クラスター化された仮想 VMM サーバー](./media/site-recovery-single-vmm/single-vmm-cluster.png)

スタンドアロン VMM サーバーを、セカンダリ サイトにレプリケートする VM としてデプロイするのではなく、Windows フェールオーバー クラスターに VM としてデプロイすることで、VMM の可用性を高めることができます。 これにより、ワークロードの回復力とハードウェア障害に対する保護が提供されます。 Site Recovery でデプロイするには、VMM VM を地理的に離れたサイトにまたがるストレッチ クラスターにデプロイする必要があります。 これを行うには、次の手順を実行します。

1. Windows フェールオーバー クラスターの仮想マシンに VMM をインストールし、セットアップ時に、サーバーを可用性が高い状態で実行するオプションを選択します。
2. VMM によって使用される SQL Server インスタンスは SQL Server AlwaysOn 可用性グループでレプリケートする必要があります。それにより、セカンダリ サイト内にデータベースのレプリカが作成されます。
3. [こちらの記事](site-recovery-vmm-to-vmm.md) に記載されている手順に従って、コンテナーを作成し、サーバーを登録して、保護を設定します。 クラスターの各 VMM サーバーを Recovery Services コンテナーに登録する必要があります。 そのためには、アクティブなノードに Provider をインストールし、VMM サーバーを登録します。 次に、プロバイダーを他のノードにインストールします。

### <a name="what-to-do-in-an-outage"></a>障害発生時の対処
障害が発生すると、VMM サーバーと対応する SQL Server データベースがフェールオーバーされ、セカンダリ サイトからアクセスされるようになります。

## <a name="next-steps"></a>次のステップ
[VMM から VMM にレプリケートするための Site Recovery のデプロイの詳細](site-recovery-vmm-to-vmm.md) を確認します。



<!--HONumber=Nov16_HO4-->


