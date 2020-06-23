---
title: Azure Synapse Analytics の共有メタデータ テーブル
description: Azure Synapse Analytics には共有メタデータ モデルが用意されていて、Apache Spark でテーブルを作成することにより、データを複製しなくても、SQL オンデマンド (プレビュー) と SQL プール エンジンからそのテーブルにアクセスできるようになります。
services: sql-data-warehouse
author: MikeRys
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: ''
ms.date: 04/15/2020
ms.author: mrys
ms.reviewer: jrasnick
ms.openlocfilehash: 7c1951c772dcd2f49f4f7c09021f69193af0a87e
ms.sourcegitcommit: b80aafd2c71d7366838811e92bd234ddbab507b6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "81420836"
---
# <a name="azure-synapse-analytics-shared-metadata-tables"></a>Azure Synapse Analytics の共有メタデータ テーブル

[!INCLUDE [synapse-analytics-preview-terms](../../../includes/synapse-analytics-preview-terms.md)]

Azure Synapse Analytics では、さまざまなワークスペース計算エンジンが、Apache Spark プール (プレビュー)、SQL オンデマンド (プレビュー) エンジン、SQL プールの間でデータベースと Parquet ベースのテーブルを共有できます。

Spark ジョブによってデータベースが作成されると、Spark を使用してその内部にテーブルを作成できます (ストレージ形式として Parquet を使用)。 これらのテーブルではすぐに、任意の Azure Synapse ワークスペース Spark プールによってクエリを実行できるようになります。 これらは、アクセス許可の対象となる任意の Spark ジョブから使用することもできます。

Spark で作成、管理される外部テーブルは、SQL オンデマンドの対応する同期済みデータベースと、メタデータ同期が有効な SQL プールの対応する `$` プレフィックス付きスキーマで、名前が同じ外部テーブルとして使用することもできます。 テーブルの同期については、「[SQL での Spark テーブルの公開](#exposing-a-spark-table-in-sql)」で詳しく説明します。

テーブルは SQL オンデマンドと SQL プールに非同期的に同期されるため、表示されるまでに遅延が発生します。

外部テーブル、データ ソース、ファイル形式へのテーブルのマッピング。

## <a name="manage-a-spark-created-table"></a>Spark で作成されたテーブルの管理

Spark を使用して、Spark で作成されたデータベースを管理します。 たとえば、Spark プール ジョブを使用してそれを削除したり、Spark からその内部にテーブルを作成したりします。

SQL オンデマンドでそのようなデータベースにオブジェクトを作成したり、データベースの削除を試みたりすると、操作は成功しますが、元の Spark データベースは変更されません。

SQL プール内の同期されたスキーマを削除しようとしたり、その内部にテーブルを作成しようとしたりすると、Azure からエラーが返されます。

## <a name="exposing-a-spark-table-in-sql"></a>SQL での Spark テーブルの公開

### <a name="which-spark-tables-are-shared"></a>共有される Spark テーブル

Spark には、Azure Synapse によって自動的に SQL で公開されるテーブルが 2 種類あります。

- マネージド テーブル

  Spark には、テキスト、CSV、JSON、JDBC、PARQUET、ORC、HIVE、DELTA、LIBSVM など、マネージド テーブルにデータを格納するためのオプションが多数用意されています。 これらのファイルは通常、マネージド テーブル データが格納される `warehouse` ディレクトリに格納されます。

- 外部テーブル

  Spark では、`LOCATION` オプションを指定するか、Hive 形式を使用することにより、既存のデータに対して外部テーブルを作成することもできます。 このような外部テーブルは、Parquet などのさまざまなデータ形式にできます。

Azure Synapse では現在、SQL エンジンを使用して Parquet 形式でデータを格納する、「マネージドおよび外部 Spark テーブル」のみが共有されます。 他の形式に基づくテーブルは自動的に同期されません。 SQL エンジンがテーブルの基になる形式をサポートしている場合は、自分の SQL データベースの外部テーブルとして、このようなテーブルを明示的に同期することができます。

### <a name="how-are-spark-tables-shared"></a>Spark テーブルの共有方法

次のプロパティを使用して、SQL エンジンで外部テーブルとして公開される共有可能なマネージドおよび外部 Spark テーブル:

- SQL 外部テーブルのデータ ソースは、Spark テーブルの場所フォルダーを表すデータ ソースです。
- SQL 外部テーブルのファイル形式は Parquet です。
- SQL 外部テーブルのアクセス資格情報はパススルーです。

すべての Spark テーブル名は有効な SQL テーブル名であり、すべての Spark 列名は有効な SQL 列名であるため、SQL 外部テーブルには Spark テーブルおよび列の名前が使用されます。

Spark テーブルのデータ型は、Synapse SQL エンジンのものとは異なります。 次の表は、Spark テーブルのデータ型を SQL 型に対応させた対照表です。

| Spark データ型 | SQL データ型 | 説明 |
|---|---|---|
| `byte`      | `smallint`       ||
| `short`     | `smallint`       ||
| `integer`   |    `int`            ||
| `long`      |    `bigint`         ||
| `float`     | `real`           |<!-- need precision and scale-->|
| `double`    | `float`          |<!-- need precision and scale-->|
| `decimal`      | `decimal`        |<!-- need precision and scale-->|
| `timestamp` |    `datetime2`      |<!-- need precision and scale-->|
| `date`      | `date`           ||
| `string`    |    `varchar(max)`   | 照合順序 `Latin1_General_CP1_CI_AS_UTF8` を使用 |
| `binary`    |    `varbinary(max)` ||
| `boolean`   |    `bit`            ||
| `array`     |    `varchar(max)`   | 照合順序 `Latin1_General_CP1_CI_AS_UTF8` を使用して JSON にシリアル化 |
| `map`       |    `varchar(max)`   | 照合順序 `Latin1_General_CP1_CI_AS_UTF8` を使用して JSON にシリアル化 |
| `struct`    |    `varchar(max)`   | 照合順序 `Latin1_General_CP1_CI_AS_UTF8` を使用して JSON にシリアル化 |

<!-- TODO: Add precision and scale to the types mentioned above -->

## <a name="security-model"></a>セキュリティ モデル

Spark データベースおよびテーブルは、SQL エンジン内のそれらの同期された表現と同様に、基になるストレージ レベルで保護されます。 現在、それらにはオブジェクト自体に対するアクセス許可がないため、オブジェクト エクスプローラーでオブジェクトを表示できます。

マネージド テーブルを作成するセキュリティ プリンシパルは、そのテーブルの所有者と見なされ、テーブルと基になるフォルダーおよびファイルに対するすべての権限を付与されます。 さらに、データベースの所有者は、自動的にテーブルの共同所有者になります。

認証パススルーを使用して Spark または SQL の外部テーブルを作成する場合、データはフォルダーとファイルのレベルでのみ保護されます。 だれかがこの種類の外部テーブルに対してクエリを実行する場合は、クエリ送信者のセキュリティ ID がファイル システムに渡され、アクセス権が確認されます。

フォルダーとファイルに対するアクセス許可を設定する方法の詳細については、「[Azure Synapse Analytics の共有データベース](database.md)」を参照してください。

## <a name="examples"></a>例

### <a name="create-a-managed-table-backed-by-parquet-in-spark-and-query-from-sql-on-demand"></a>Spark で Parquet がベースのマネージド テーブルを作成し、SQL オンデマンドでクエリを実行する

このシナリオでは、`mytestdb` という名前の Spark データベースを用意します。 「[Spark データベースの作成と接続 - SQL オンデマンド](database.md#create--connect-to-spark-database---sql-on-demand)」を参照してください。

次のコマンドを実行し、SparkSQL を使用してマネージド Spark テーブルを作成します。

```sql
    CREATE TABLE mytestdb.myParquetTable(id int, name string, birthdate date) USING Parquet
```

これにより、データベース `mytestdb` にテーブル `myParquetTable` が作成されます。 少し時間が経過すると、SQL オンデマンドでテーブルを確認できるようになります。 たとえば、SQL オンデマンドで次のステートメントを実行します。

```sql
    USE mytestdb;
    SELECT * FROM sys.tables;
```

結果に `myParquetTable` が含まれていることを確認します。

>[!NOTE]
>ストレージ形式として Parquet が使用されていないテーブルは同期されません。

次に、Spark からテーブルに値を挿入します。たとえば、C# ノートブックで次の C# Spark ステートメントを使用します。

```csharp
using Microsoft.Spark.Sql.Types;

var data = new List<GenericRow>();

data.Add(new GenericRow(new object[] { 1, "Alice", new Date(2010, 1, 1)}));
data.Add(new GenericRow(new object[] { 2, "Bob", new Date(1990, 1, 1)}));

var schema = new StructType
    (new List<StructField>()
        {
            new StructField("id", new IntegerType()),
            new StructField("name", new StringType()),
            new StructField("birthdate", new DateType())
        }
    );

var df = spark.CreateDataFrame(data, schema);
df.Write().Mode(SaveMode.Append).InsertInto("mytestdb.myParquetTable");
```

これで、次のように SQL オンデマンドでデータを読み取ることができます。

```sql
SELECT * FROM mytestdb.dbo.myParquetTable WHERE name = 'Alice';
```

次の行が結果として表示されます。

```
id | name | birthdate
---+-------+-----------
1 | Alice | 2010-01-01
```

### <a name="creating-an-external-table-backed-by-parquet-in-spark-and-querying-it-from-sql-on-demand"></a>Spark で Parquet がベースの外部テーブルを作成し、SQL オンデマンドでクエリを実行する

この例では、前の例でマネージド テーブル用に作成した Parquet データ ファイルに対して外部 Spark テーブルを作成します。

たとえば、SparkSQL を使用して次を実行します。

```sql
CREATE TABLE mytestdb.myExternalParquetTable
    USING Parquet
    LOCATION "abfss://<fs>@arcadialake.dfs.core.windows.net/synapse/workspaces/<synapse_ws>/warehouse/mytestdb.db/myparquettable/"
```

プレースホルダー `<fs>` は、ワークスペースの既定のファイル システムであるファイル システム名に、プレースホルダー `<synapse_ws>` は、この例を実行するために使用している Synapse ワークスペースの名前に置き換えてください。

前の例では、`mytestdb` データベースに `myExtneralParquetTable` テーブルが作成されます。 少し時間が経過すると、SQL オンデマンドでテーブルを確認できるようになります。 たとえば、SQL オンデマンドで次のステートメントを実行します。

```sql
USE mytestdb;
SELECT * FROM sys.tables;
```

結果に `myExternalParquetTable` が含まれていることを確認します。

これで、次のように SQL オンデマンドでデータを読み取ることができます。

```sql
SELECT * FROM mytestdb.dbo.myExternalParquetTable WHERE name = 'Alice';
```

次の行が結果として表示されます。

```
id | name | birthdate
---+-------+-----------
1 | Alice | 2010-01-01
```

### <a name="querying-spark-tables-in-a-sql-pool"></a>SQL プールで Spark テーブルに対するクエリを実行する

前の例で作成したテーブルを使って、今度はメタデータ同期を可能にする `mysqlpool` という名前の SQL プールを自分のワークスペースに作成します (または「[SQL プールでの Spark データベースの公開](database.md#exposing-a-spark-database-in-a-sql-pool)」で作成した既存のプールを使用します)。

`mysqlpool` SQL プールに対して次のステートメントを実行します。

```sql
SELECT * FROM sys.tables;
```

テーブル `myParquetTable` および `myExternalParquetTable` がスキーマ `$mytestdb` に表示されていることを確認します。

これで、次のように SQL オンデマンドでデータを読み取ることができます。

```sql
SELECT * FROM [$mytestdb].myParquetTable WHERE name = 'Alice';
SELECT * FROM [$mytestdb].myExternalParquetTable WHERE name = 'Alice';
```

上記の SQL オンデマンドの場合と同じ結果が得られます。

## <a name="next-steps"></a>次のステップ

- [Azure Synapse Analytics の共有メタデータについての詳細情報](overview.md)
- [Azure Synapse Analytics の共有メタデータ テーブルについての詳細情報](table.md)


