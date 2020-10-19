## DynamoDB概要

Amazon DynamoDBの特徴

### 1. NoSQL

> Key-value およびドキュメントデータモデルのいずれもサポートされています。そのため、DynamoDB では柔軟なスキーマを適用して、任意の時点で任意の数の列を各行に設定することができます。

### 2. フルマネージド型

> 分散データベースの運用とスケーリングに伴う管理作業をまかせることができるため、ハードウェアのプロビジョニング、設定と構成、レプリケーション、ソフトウェアのパッチ適用、クラスタースケーリングなどを自分で行う必要はなくなります。

### 3. 高い可用性と耐久性

> DynamoDB は十分な数のサーバー間でデータとトラフィックを自動的に分散し、一貫した高速なパフォーマンスを維持したまま、スループットとストレージの要件に対応します。すべてのデータは SSD (Solid State Disk) に保存され、1 つの AWS リージョン内の複数のアベイラビリティーゾーン間で自動的にレプリケートすることによって、高い可用性とデータ堅牢性を実現します。

## 基本構成

基本構成は以下のような感じです。

 - table
 - items
 - attirubtes

### Table

> Similar to other database systems, DynamoDB stores data in tables. A table is a collection of data.

> 他のDBと同様に、DynamoDBもテーブルにデータを格納。テーブルは、データのコレクション。

### Item

> Each table contains zero or more items. An item is a group of attributes that is uniquely identifiable among all of the other items. In DynamoDB, there is no limit to the number of items you can store in a table.

> 各テーブルは、0以上の項目を持つ。項目は、すべての項目の中で一意に識別できる属性のグループ。DynamoDBには、保存できる項目の数に制限はない。

### Attribute

> Each item is composed of one or more attributes. An attribute is a fundamental data element, something that does not need to be broken down any further. Attributes in DynamoDB are similar in many ways to fields or columns in other database systems.

> 各項目は、1つ以上の属性で構成。属性は基本的なデータ要素であり、これ以上分解する必要はない。DynamoDBの属性は、他のデータベースシステムの、フィールド・列と似ている。

### その他テーブルに必要な概念

 - PrimaryKey（HASH, RANGE）
 - CapacityUnit
 - インデックス（GSI, LSI）

### PrimaryKey

 各プライマリキー属性はスカラー値 (単一値のみを保持できる) である必要
 DynamoDBはプライマリーキーは必須。
 プライマリキーはテーブルの各項目を一意に識別するため、テーブル内の 2 つの項目が同じキーを持つことはない。
 プライマリーキーは、以下の2種類をサポート。

#### 1. パーティションキーのみ（HASHキー）

> A simple primary key, composed of one attribute known as the partition key. DynamoDB uses the partition key's value as input to an internal hash function. The output from the hash function determines the partition (physical storage internal to DynamoDB) in which the item will be stored.
> In a table that has only a partition key, no two items can have the same partition key value.

> パーティションキーと呼ばれる1つの属性で構成される主キー。DynamoDBは、パーティションキー値を入力として内部ハッシュ関数を使用。その出力を利用し、項目が保存されるパーティション（DynamoDB内部の物理ストレージ）を決定。
> 単一パーティションキーのテーブルは、2つの項目で同じパーティションキー値を持つことできない

#### 2. パーティションキーとソートキーの複合キー（HASH+RANGEキー）

> Referred to as a composite primary key, this type of key is composed of two attributes. The first attribute is the partition key, and the second attribute is the sort key.
> DynamoDB uses the partition key value as input to an internal hash function. The output from the hash function determines the partition (physical storage internal to DynamoDB) in which the item will be stored. All items with the same partition key value are stored together, in sorted order by sort key value.
> In a table that has a partition key and a sort key, it's possible for two items to have the same partition key value. However, those two items must have different sort key values.

> 複合主キーと呼ばれるこのキーは、2つの属性で構成。最初の属性は、パーティションキー、2つめはソートキー。
> ... 同じパーティションキー値を持つすべての項目は、ソートキー値でソートされた順序で格納される。
> 複合主キーを持つテーブルでは、2つの項目が同じパーティションキーを持つ可能性がある。ただし、その場合ソートキーが異なる必要がある。

### CapacityUnit

> Amazon DynamoDB でプロビジョニングされたテーブルを新しく作成する場合は、プロビジョンドスループット性能を指定する必要があります。これは、テーブルがサポートできる読み取りおよび書き込みアクティビティの量です。DynamoDB はこの情報を使用して、スループット要件を満たすのに十分なシステムリソースを予約します。

 DynamoDBは、テーブルごとにCapacityUnitの設定が必要。PrimaryKey用のCapacityUnit、GSIがある場合は、そのGSIに対しても設定が必要。
 CapacityUnitは、ReadCapacityUnitsとWriteCapacityUnitsがある。

#### 1. ReadCapacityUnits（読み込みキャパシティーユニット）

  1RCUは、最大4KBの項目について、 1秒あたり 1回の強力な整合性のある読み込み、あるいは 1秒あたり 2回の結果整合性のある読み込みを表します。

#### 2. WriteCapacityUnits（書き込みキャパシティーユニット）

  1WCUは、最大サイズが 1 KB の項目について、1 秒あたり 1 回の書き込みを表します。

### インデックス

 > Amazon DynamoDB は、プライマリキーの値を指定して、テーブルの項目への高速なアクセスを可能にします。しかし多くのアプリケーションでは、プライマリキー以外の属性を使って、データに効率的にアクセスできるようにセカンダリ（または代替）キーを 1 つ以上設定することで、メリットが得られることがあります。これに対応するために、1 つのテーブルで 1 つ以上のセカンダリインデックスを作成して、それらのインデックスに対して Query または Scan リクエストを実行することができます。

 プライマリーキー以外の属性に対して、高速なアクセスを実現したい場合に、インデックスを仕様。
 インデックスの種類は2種類存在。

#### 1. GlobalSecondaryIndex

  ベーステーブルと異なるパーティションキーとソートキーを持つ。
  すべてのパーティションにアクセスするという意味で、「グローバル」
  ベーステーブルとは別に独自のパーティション領域に保存される。

#### 2. LocalSecondaryIndex

  ベーステーブルと同じパーテイションキー、別のソートキーを持つ。
  同じパーティションキーをもつパーティションにのみアクセスするという意味で「ローカル」

#### GSI, LSI の比較

| 特徴 | グローバルセカンダリインデックス |	ローカルセカンダリインデックス |
| ----- | ----- | -----|
| キースキーマ	| プライマリキーはシンプル (パーティションキー) または複合 (パーティションキーとソートキー) のいずれかとすることができます。 | 	プライマリキーは複合 (パーティションキーとソートキー) である必要があります。|
| オンラインインデックスオペレーション |	テーブルの作成と同時に作成できます。また、既存のテーブルに新しい グローバルセカンダリインデックス を追加したり、既存の グローバルセカンダリインデックス を削除したりできます。 |	テーブルの作成と同時に作成されます。既存のテーブルにローカルセカンダリインデックスを追加したり、現在存在するローカルセカンダリインデックスを削除したりすることはできません。|
| クエリとパーティション	| すべてのパーティションでテーブル全体に対してクエリを実行できます。| クエリのパーティションキー値で指定された 1 つのパーティションに対してクエリを実行できます。|
| 読み込み整合性 |	クエリでは、結果整合性のみがサポートされます。	| 結果整合性または強い整合性のどちらかを選択できます。 |
| プロビジョニングされたスループットの消費 |	各 グローバルセカンダリインデックス には、読み込みおよび書き込みアクティビティに対する独自のプロビジョニングされたスループット設定があります。グローバルセカンダリインデックスのクエリまたはスキャンでは、ベーステーブルからではなく、インデックスからキャパシティーユニットを消費します。同じことが、テーブルへの書き込みによる グローバルセカンダリインデックス の更新にも当てはまります。	| クエリまたはスキャンでは、ベーステーブルから読み込みキャパシティーユニットを消費します。テーブルに書き込むと、そのlocal secondary indexも更新されます。この更新では、ベーステーブルから書き込みキャパシティーユニットを消費します。|
|射影される属性	| クエリまたはスキャンでは、インデックスに射影された属性だけをリクエストできます。DynamoDB は、テーブルから属性をフェッチしません。	| クエリまたはスキャンする場合、インデックスに射影されていない属性をリクエストできます。DynamoDB は、テーブルからそれらの属性を自動的にフェッチしません。|

https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/SecondaryIndexes.html


### ここまでの例

#### 1. テーブル作成（String型のパーティションキーArtists、String型のソートキーSongTitleが主キーのテーブルを作成）

```
DYNAMODB_ENDPOINT_URL=http://localhost:18080
aws dynamodb --endpoint-url=${DYNAMODB_ENDPOINT_URL} create-table \
  --table-name Music \
  --attribute-definitions \
      AttributeName=Artist,AttributeType=S \
      AttributeName=SongTitle,AttributeType=S \
  --key-schema \
        AttributeName=Artist,KeyType=HASH \
        AttributeName=SongTitle,KeyType=RANGE \
  --provisioned-throughput \
        ReadCapacityUnits=10,WriteCapacityUnits=10
```

#### 2. 作成したテーブルの確認

```
aws dynamodb --endpoint-url=${DYNAMODB_ENDPOINT_URL} describe-table \
  --table-name Music
```

#### 3. 作成したテーブルにGSIを追加

```
aws dynamodb --endpoint-url=${DYNAMODB_ENDPOINT_URL} update-table \
  --table-name Music \
  --attribute-definitions AttributeName=AlbumTitle,AttributeType=S \
  --global-secondary-index-updates file://fixtures/global_secondary_index.json
```

### 4. サンプルデータ追加

```
aws dynamodb --endpoint-url=${DYNAMODB_ENDPOINT_URL} batch-write-item \
  --request-items file://fixtures/musics.json \
  --return-consumed-capacity INDEXES
```

## 項目操作API

### 読み込み

| Operation | 概要 | その他 |
| ----- | ----- | ----- |
| GetItem | 主キーで1項目取得 | |
| BatchGetItem | 1つ以上のテーブルから指定したキーの項目を取得 | 1度に100項目まで取得可能 |
| Query | 指定したパーティションキーの項目全て取得。<br>ソートキーに対しての条件で検索も可能 | 使い方によっては、BatchGetItemよりRCUを節約できるかも |
| Scan | 全ての項目を取得 | |

### 書き込み

| Operation | 概要 | その他 |
| ----- | ----- | ----- |
| PutItem | 項目の作成 | |
| BatchWriteItem | 1つのテーブルに複数項目を一度に作成 | 1度に25項目まで取得可能<br>BatchGetItemと違い1テーブル<br>削除も可能 |
| UpdateItem | 主キーを指定して、Attributeを修正 | |
| DeleteItem | 主キーで項目削除 | |

#### 1. 項目書き込み PutItem

> Creates a new item, or replaces an old item with a new item. If an item that has the same primary key as the new item already exists in the specified table, the new item completely replaces the existing item. You can perform a conditional put operation (add a new item if one with the specified primary key doesn't exist), or replace an existing item if it has certain attribute values.

> 新しい項目の作成、もしくはフリ項目の置き換え。新しい項目と同じプライマリーキーが存在する場合は、完全に置き換えされる。条件付きのプット操作を実行でき、項目が存在する場合、指定した属性の更新のみすることも可能

```
aws dynamodb --endpoint-url=${DYNAMODB_ENDPOINT_URL} put-item \
  --table-name Music \
  --item \
    '{"Artist": {"S": "No One You Know"}, "SongTitle": {"S": "Call Me Today"}, "AlbumTitle": {"S": "Somewhat Famous"}, "Awards": {"N": "1"}}' \
  --return-consumed-capacity INDEXES

### 条件付き書き込み

aws dynamodb --endpoint-url=${DYNAMODB_ENDPOINT_URL} put-item \
  --table-name Music \
  --item \
    '{"Artist": {"S": "No One You Know"}, "SongTitle": {"S": "Call Me Today"}, "AlbumTitle": {"S": "Somewhat Famous"}, "Awards": {"N": "2"}}' \
  --condition-expression "Awards = :awards" \
  --expression-attribute-values '{":awards":{"N": "1"}}' \
  --return-consumed-capacity INDEXES
```

#### 2. 項目更新 UpdateItem

> Edits an existing item's attributes, or adds a new item to the table if it does not already exist. You can put, delete, or add attribute values. You can also perform a conditional update on an existing item (insert a new attribute name-value pair if it doesn't exist, or replace an existing name-value pair if it has certain expected attribute values).

> 既存の項目の属性値を変更、もしくはその項目が存在しない場合に追加する。属性値の追加、削除が可能。既存の項目に対して条件付きの更新も可能。（属性のキー・バリューのペアが存在しなければ新たに挿入する、もしくは特定の想定される属性値が存在する場合に既存の属性を更新することが可能

```
aws dynamodb --endpoint-url=${DYNAMODB_ENDPOINT_URL} update-item \
  --table-name Music \
  --key '{"Artist": {"S": "Acme Band"}, "SongTitle": {"S": "Happy Day"}}' \
  --update-expression "SET #Y = :y" \
  --expression-attribute-names '{"#Y":"Year"}' \
  --expression-attribute-values '{":y":{"N":"2020"}}' \
  --return-consumed-capacity INDEXES

### 条件付き更新（更新されない
aws dynamodb --endpoint-url=${DYNAMODB_ENDPOINT_URL} update-item \
  --table-name Music \
  --key '{"Artist": {"S": "Acme Band"}, "SongTitle": {"S": "Happy Day"}}' \
  --update-expression "SET #Y = :y" \
  --expression-attribute-names '{"#Y":"Year"}' \
  --expression-attribute-values '{":y":{"N":"2021"}}' \
  --condition-expression "attribute_not_exists(#Y)" \
  --return-consumed-capacity INDEXES
```

#### 3. 1項目の取得 GetItem

> The GetItem operation returns a set of attributes for the item with the given primary key. If there is no matching item, GetItem does not return any data and there will be no Item element in the response.
> GetItem provides an eventually consistent read by default. If your application requires a strongly consistent read, set ConsistentRead to true. Although a strongly consistent read might take more time than an eventually consistent read, it always returns the last updated value.

> GetItemは主キーの属性のセットを取得。もし一致する項目がない場合は、何も消さない。
> GetItemは、結果整合性読み込みをデフォルトで提供。強い整合性を要求するときはconsistent readをtrueにする。強い整合性は読み込みは結果整合性読み込みよりも時間がかかる可能性があるが、常に最後に更新された値を取得出来る。

```
### 結果整合性
aws dynamodb --endpoint-url=${DYNAMODB_ENDPOINT_URL} get-item \
  --table-name Music \
  --key '{"Artist": {"S": "Acme Band"}, "SongTitle": {"S": "Happy Day"}}' \
  --no-consistent-read \
  --return-consumed-capacity INDEXES

### 強い整合性
aws dynamodb --endpoint-url=${DYNAMODB_ENDPOINT_URL} get-item \
  --table-name Music \
  --key '{"Artist": {"S": "Acme Band"}, "SongTitle": {"S": "Happy Day"}}' \
  --consistent-read \
  --return-consumed-capacity INDEXES
```

#### 4. 複数の取得 Query

> The Query operation finds items based on primary key values. You can query any table or secondary index that has a composite primary key (a partition key and a sort key).
> Use the KeyConditionExpression parameter to provide a specific value for the partition key. The Query operation will return all of the items from the table or index with that partition key value. You can optionally narrow the scope of the Query operation by specifying a sort key value and a comparison operator in KeyConditionExpression. To further refine the Query results, you can optionally provide a FilterExpression. A FilterExpression determines which items within the results should be returned to you. All of the other results are discarded.
> Queries that do not return results consume the minimum number of read capacity units for that type of read operation.

> クエリはプライマリーキーの値によって項目を検索。クエリは複合主キーを持つテーブル、セカンダリインデックスに対して使用可能。
> KeyConditionExpressionパラメータを使用し、パーティションキーに特定の値を指定。クエリは、そのパーティションキー値を持つすべての項目を取得。ソートキー値を使用し、クエリのスコープを狭めることも可能。さらにFilterExpressionを使用することで、クエリの結果のどの項目を返却するかを決定可能。他のすべての結果は破棄される。
> 結果を返却しないクエリは、その最小のRCUを消費。

```
aws dynamodb --endpoint-url=${DYNAMODB_ENDPOINT_URL} query \
  --table-name Music \
  --key-condition-expression "Artist = :artist" \
  --expression-attribute-values '{":artist": {"S":"No One You Know"}}' \
  --return-consumed-capacity INDEXES

### FilterExpression
aws dynamodb --endpoint-url=${DYNAMODB_ENDPOINT_URL} query \
  --table-name Music \
  --key-condition-expression "Artist = :artist" \
  --expression-attribute-values '{":artist": {"S":"No One You Know"}, ":album_title": {"S":"Blue Sky Blues"}}' \
  --filter-expression "AlbumTitle = :album_title"

### FilterExpression * Limit
aws dynamodb --endpoint-url=${DYNAMODB_ENDPOINT_URL} query \
  --table-name Music \
  --key-condition-expression "Artist = :artist" \
  --expression-attribute-values '{":artist": {"S":"No One You Know"}, ":album_title": {"S":"Blue Sky Blues"}}' \
  --filter-expression "AlbumTitle = :album_title" \
  --limit 1
```


## 参考

https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/Introduction.html
