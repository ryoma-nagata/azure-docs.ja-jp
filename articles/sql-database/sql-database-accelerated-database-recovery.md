---
title: 高速データベース復旧 - Azure SQL Database | Microsoft Docs
description: Azure SQL Database には、単一データベース、エラスティック プール、および Azure SQL Data Warehouse を対象に、高速で一貫性のあるデータベース復旧、瞬間的なトランザクションのロールバック、および積極的なログの切り捨て動作を提供する、新しい機能があります。
ms.service: sql-database
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: CarlRabeler
ms.author: carlrab
ms.reviewer: ''
manager: craigg
ms.date: 10/11/2018
ms.openlocfilehash: 20ad792ffa1c06e75cd39e4549b8d897a663db3e
ms.sourcegitcommit: dbfd977100b22699823ad8bf03e0b75e9796615f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2018
ms.locfileid: "50239969"
---
# <a name="accelerated-database-recovery-preview"></a>高速データベース復旧 (プレビュー)

**高速データベース復旧 (ADR)** は、SQL データベース エンジンの復旧プロセスを再設計することで、実行時間の長いトランザクションがある場合などにデータベースの可用性を大幅に向上させる、新しい SQL データベース エンジン機能です。 ADR は現在、単一データベース、エラスティック プール、および Azure SQL Data Warehouse で使用できます。 ADR の主な利点は次のとおりです。

- **高速で一貫性のあるデータベース復旧**

  ADR を使用すれば、実行時間の長いトランザクションによって全体的な復旧時間が影響を受けることがなくなるので、システム内のアクティブ トランザクション数やそのサイズに関係なく、高速で一貫性のあるデータベース復旧を実現できます。

- **瞬間的なトランザクションのロールバック**

  ADR では、トランザクションがアクティブになっている時間や、実行された更新プログラムの数に関係なく、トランザクションを瞬時にロールバックできます。

- **積極的なログの切り捨て**

  ADR では、実行時間の長いアクティブ トランザクションがある場合でも、トランザクション ログが積極的に切り捨てられるので、ログが制御できなる事態を防止できます。

## <a name="the-current-database-recovery-process"></a>現行のデータベース復旧プロセス

SQL Server のデータベース復旧は [ARIES](https://people.eecs.berkeley.edu/~brewer/cs262/Aries.pdf) 復旧モデルに従っており、3 つのフェーズで構成されています。次の図とその後の記述は、それらのフェーズについて詳しく説明したものです。

![現行の復旧プロセス](./media/sql-database-accelerated-database-recovery/current-recovery-process.png)

- **分析フェーズ**

  最後に成功したチェックポイント (または最も古いページ LSN) から末尾まで、トランザクション ログを前方スキャンして、SQL Server が停止した時点での各トランザクションの状態を判定します。

- **再実行フェーズ**

  最も古い未コミット トランザクションから末尾まで、トランザクション ログを前方スキャンし、すべての操作をやり直すことにより、データベースの状態をクラッシュが発生した時点の状態に戻します。

- **元に戻すフェーズ**

  クラッシュの発生時点でアクティブだった各トランザクションについて、ログを後方に走査し、そのトランザクションによって実行された操作を元に戻します。

この設計では、SQL データベース エンジンが予期しない再起動から復旧するのにかかる時間は、クラッシュの発生時にシステム内でアクティブ時間が最も長かったトランザクションのサイズに (ほぼ) 比例します。 復旧するには、すべての未完了トランザクションをロールバックする必要があります。 必要となる時間の長さは、トランザクションによって実行された作業の量と、トランザクションがアクティブになっている時間の長さに比例します。 そのため、SQL Server の復旧プロセスでは、実行時間の長いトランザクション (大規模な一括挿入操作や、サイズの大きなテーブルに対するインデックス作成操作など) がある場合に、長時間を要することがあります。

また、この設計では、先に述べたのと同じ元に戻す復旧フェーズが使用されるため、大規模なトランザクションを取り消したりロールバックしたりする際にも、長い時間がかかることがあります。

さらに、SQL データベース エンジンでは、復旧とロールバック プロセスのために、対応するログ レコードが必要となるため、実行時間の長いトランザクションがある場合に、トランザクション ログを切り捨ることができません。 SQL データベース エンジンがこのように設計されているため、お客様によっては、トランザクション ログのサイズが非常に大きくなり、膨大な量のログ領域が消費されるという問題が発生します。

## <a name="the-accelerated-database-recovery-process"></a>高速データベース復旧のプロセス

ADR では、SQL データベース エンジンの復旧プロセスを以下のように根本から再設計することで、上記の問題に対処しています。

- 最も古いアクティブ トランザクションの先頭を基点にログをスキャンする必要性をなくすことで、所要時間を一定化し、高速化を実現しています。 ADR では、最後に成功したチェックポイント (または最も古いダーティ ページのログ シーケンス番号 (LSN)) を基点に、それ以降のトランザクション ログだけを処理します。 そのため、実行時間の長いトランザクションによって復旧時間が影響を受けることはありません。
- トランザクション全体のログを処理する必要がなくなるので、必要なトランザクション ログ領域が最小限に抑えられます。 そのため、チェックポイントやバックアップごとにトランザクション ログを積極的に切り捨てることができます。

要するに、ADR では、物理的なデータベース変更をすべてバージョン管理し、論理的な操作だけを元に戻すようにすることで、処理の対象を減らし、瞬間的な処理を可能にして、高速なデータベース復旧を実現しているのです。 クラッシュが発生した時点でアクティブだったトランザクションは中止としてマークされるため、それらのトランザクションによって生成されたバージョンはすべて、同時ユーザー クエリで無視できます。

ADR の回復プロセスには、現行の復旧プロセスと同じ 3 つのフェーズがあります。 次の図とその後の記述は、それらのフェーズが ADR でどのように動作するかについて詳しく説明したものです。

![ADR の復旧プロセス](./media/sql-database-accelerated-database-recovery/adr-recovery-process.png)

- **分析フェーズ**

  sLog を再構築し、バージョン管理されない操作のログ レコードをコピーする以外は、現行のプロセスと同じです。
- **再実行**フェーズ

  2 つのフェーズ (P) に分けられます
  - フェーズ 1

      sLog から再実行します (最も古い未コミット トランザクションから最後のチェックポイントまで)。 sLog から数個のレコードを処理するだけで済むので、再実行操作は迅速に完了します。
  - フェーズ 2

     トランザクション ログからの再実行は、(最も古い未コミット トランザクションではなく) 最後のチェックポイントから開始されます
- **元に戻すフェーズ**

   ADR では、sLog を使用してバージョン管理のない操作を元に戻し、永続バージョン ストア (PVS) と論理復元によってバージョンベースの Undo 処理を行レベルで実行できるため、元に戻すフェーズをほぼ瞬時に完了できます。

## <a name="adr-recovery-components"></a>ADR の復旧コンポーネント

ADR には次の 4 つの主要コンポーネントがあります。

- **永続バージョン ストア (PVS)**

  永続バージョン ストアは、データベース自体で生成された行バージョンを保持するための新しい SQL データベース エンジン メカニズムであり、従来の `tempdb` バージョン ストアに代わるものです。 PVS を使用すると、リソース分離が可能になり、読み取り可能なセカンダリの可用性が向上します。

- **論理復元**

  論理復元は、バージョン ベースの復元を行レベルで実行するための非同期プロセスです。トランザクションのロールバックを瞬時に実行し、バージョン管理されているすべての操作を元に戻すことができます。

  - 中止されたすべてのトランザクションを追跡できます
  - すべてのユーザー トランザクションに対し、PVS を使ってロールバックを実行できます
  - トランザクションの中止後すぐに、すべてのロックを解除できます

- **sLog**

  sLog は、バージョン管理されない操作 (メタデータ キャッシュの無効化、ロックの取得など) のログ レコードを格納する、セカンダリのメモリ内ログ ストリームです。 sLog の特徴は次のとおりです。

  - サイズが小さく、メモリ内で保持されます
  - チェックポイント プロセス時にシリアル化され、ディスク上で永続化されます
  - トランザクションのコミットに応じて、定期的に切り捨てられます
  - バージョン管理されない操作だけが処理されるので、やり直しと復元が高速化します  
  - 必要なログ レコードだけが保持され、トランザクション ログが積極的に切り捨てられます

- **クリーナー**

  クリーナーは、定期的に起動し、不要なページ バージョンをクリーンアップする非同期プロセスです。

## <a name="who-should-consider-accelerated-database-recovery"></a>高速データベース復旧の使用を検討するべきお客様

次に該当するお客様は、ADR の有効化を検討してください。

- 実行時間の長いトランザクションによってワークロードが発生しているお客様。
- アクティブ トランザクションによってトランザクション ログが大幅に増えたことがあるお客様。  
- SQL Server の復元動作 (SQL Server の予期しない再起動や手動のトランザクション ロールバックなど) に時間がかかり、データベースが長時間使えなくなったことがあるお客様。

## <a name="to-enable-adr-during-this-preview-period"></a>プレビュー期間中に ADR を有効化するには

この機能のプレビュー期間中に高速データベース復元 (ADR) の詳細を確認したり、機能をお試しいただくには、[adr@microsoft.com](mailto:adr@microsoft.com) までメールでお問い合わせください。 メールには、(単一データベース、エラスティック プール、および Azure Data Warehouse の) 論理サーバーの名前を記載してください。 この機能はプレビュー版なので、運用サーバーで機能を試すことは避けてください。
