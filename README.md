# DynamoDB概要

## 基本構成

 - tables       テーブルでデータ管理するよ
 - items        attributesのグループ
 - attirubtes   属性。主キー・インデックス以外はスキーマレス

テーブルに必要な概念
 - PrimaryKey（HASH, RANGE）
 - CapacityUnit
 - インデックス（GSI, LSI）

### PrimaryKey

 DynamoDBはプライマリーキーは必須。
 プライマリキーはテーブルの各項目を一意に識別するため、テーブル内の 2 つの項目が同じキーを持つことはありません。
 プライマリーキーは、2種類のインデックスをサポートします。

 1. パーティションキー（HASHキー）

  DynamoDBは、パーティションキーの値を内部ハッシュ関数を使用し、保存する物理ストレージを決定。
  パーティションキーの値は、一意。

 2. パーティションキーとソートキーの複合キー（HASH+RANGEキー）

  単一パーティションキーの場合と同様、パーティションキーによって、保存先を決定。
  すべての項目は、パーティションキーの値毎に、ソートキーの値でソートされた順序によって保存される。
  同一パーティションキー値を持つ項目が存在。
　パーティションキーとソートキーの組み合わせで、一意。

### CapacityUnit

 > Amazon DynamoDB でプロビジョニングされたテーブルを新しく作成する場合は、プロビジョンドスループット性能を指定する必要があります。これは、テーブルがサポートできる読み取りおよび書き込みアクティビティの量です。DynamoDB はこの情報を使用して、スループット要件を満たすのに十分なシステムリソースを予約します。

 DynamoDBは、テーブルごとにCapacityUnitの設定が必要。PrimaryKey用のCapacityUnit、GSIがある場合は、そのGSIに対しても設定が必要。
 CapacityUnitは、ReadCapacityUnitsとWriteCapacityUnitsがある。

 1. ReadCapacityUnits（読み込みキャパシティーユニット）

  1RCUは、最大4KBの項目について、 1秒あたり 1回の強力な整合性のある読み込み、あるいは 1秒あたり 2回の結果整合性のある読み込みを表します。

 2. WriteCapacityUnits（書き込みキャパシティーユニット）

  1WCUは、最大サイズが 1 KB の項目について、1 秒あたり 1 回の書き込みを表します。

### インデックス

 > Amazon DynamoDB は、プライマリキーの値を指定して、テーブルの項目への高速なアクセスを可能にします。しかし多くのアプリケーションでは、プライマリキー以外の属性を使って、データに効率的にアクセスできるようにセカンダリ（または代替）キーを 1 つ以上設定することで、メリットが得られることがあります。これに対応するために、1 つのテーブルで 1 つ以上のセカンダリインデックスを作成して、それらのインデックスに対して Query または Scan リクエストを実行することができます。

 プライマリーキー以外の属性に対して、高速なアクセスを実現したい場合に、インデックスを仕様。
 インデックスの種類は2種類存在。

 1. GlobalSecondaryIndex

  ベーステーブルと異なるパーティションキーとソートキーを持つ。
  すべてのパーティションにアクセスするという意味で、「グローバル」
  ベーステーブルとは別に独自のパーティション領域に保存される。

 2. LocalSecondaryIndex

  ベーステーブルと同じパーテイションキー、別のソートキーを持つ。
  同じパーティションキーをもつパーティションにのみアクセスするという意味で「ローカル」

| 特徴 | グローバルセカンダリインデックス |	ローカルセカンダリインデックス |
| ----- | ----- | -----|
| キースキーマ	| プライマリキーはシンプル (パーティションキー) または複合 (パーティションキーとソートキー) のいずれかとすることができます。 | 	プライマリキーは複合 (パーティションキーとソートキー) である必要があります。|
| オンラインインデックスオペレーション |	テーブルの作成と同時に作成できます。また、既存のテーブルに新しい グローバルセカンダリインデックス を追加したり、既存の グローバルセカンダリインデックス を削除したりできます。 |	テーブルの作成と同時に作成されます。既存のテーブルにローカルセカンダリインデックスを追加したり、現在存在するローカルセカンダリインデックスを削除したりすることはできません。|
| クエリとパーティション	| すべてのパーティションでテーブル全体に対してクエリを実行できます。| クエリのパーティションキー値で指定された 1 つのパーティションに対してクエリを実行できます。|
| 読み込み整合性 |	クエリでは、結果整合性のみがサポートされます。	| 結果整合性または強い整合性のどちらかを選択できます。 |
| プロビジョニングされたスループットの消費 |	各 グローバルセカンダリインデックス には、読み込みおよび書き込みアクティビティに対する独自のプロビジョニングされたスループット設定があります。グローバルセカンダリインデックスのクエリまたはスキャンでは、ベーステーブルからではなく、インデックスからキャパシティーユニットを消費します。同じことが、テーブルへの書き込みによる グローバルセカンダリインデックス の更新にも当てはまります。	| クエリまたはスキャンでは、ベーステーブルから読み込みキャパシティーユニットを消費します。テーブルに書き込むと、そのlocal secondary indexも更新されます。この更新では、ベーステーブルから書き込みキャパシティーユニットを消費します。|
|射影される属性	| クエリまたはスキャンでは、インデックスに射影された属性だけをリクエストできます。DynamoDB は、テーブルから属性をフェッチしません。	| クエリまたはスキャンする場合、インデックスに射影されていない属性をリクエストできます。DynamoDB は、テーブルからそれらの属性を自動的にフェッチしません。|

https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/SecondaryIndexes.html


## 項目操作API

 読み込み
 | Operation | 概要 | その他 |
 | ----- | ----- | ----- |
 | GetItem | 主キーで1項目取得 | |
 | BatchGetItem | 1つ以上のテーブルから指定したキーの項目を取得 | 1度に100項目まで取得可能 |
 | Query | 指定したパーティションキーの項目全て取得。<br>ソートキーに対しての条件で検索も可能 | 使い方によっては、BatchGetItemよりRCUを節約できるかも |
 | Scan | 全ての項目を取得 | |

 書き込み
 | Operation | 概要 | その他 |
 | ----- | ----- | ----- |
 | PutItem | 項目の作成 | |
 | BatchWriteItem | 1つのテーブルに複数項目を一度に作成 | 1度に25項目まで取得可能<br>BatchGetItemと違い1テーブル<br>削除も可能 |
 | UpdateItem | 主キーを指定して、Attributeを修正 | |
 | DeleteItem | 主キーで項目削除 | |

#### 例

0. テーブル作成（String型のパーティションキーArtists、String型のソートキーSongTitleが主キーのテーブルを作成）

```
DYNAMODB_ENDPOINT_URL=http://localhost:18080
docker-compose up -d

aws dynamodb --endpoint-url=${DYNAMODB_ENDOPOINT_URL} create-table \
  --table-name Music \
  --attribute-definitions \
      AttributeName=Artist,AttributeType=S \
      AttributeName=SongTitle,AttributeType=S \
  --key-schema \
        AttributeName=Artist,KeyType=HASH \
        AttributeName=SongTitle,KeyType=RANGE \
  --provisioned-throughput \
        ReadCapacityUnits=10,WriteCapacityUnits=10

aws dynamodb --endpoint-url=${DYNAMODB_ENDOPOINT_URL} describe-table \
  --table-name Music
```

1. 項目書き込み PutItem, BatchWriteItem

```
aws dynamodb --endpoint-url=${DYNAMODB_ENDPOINT_URL} put-item \
  --table-name Music \
  --item \
    '{"Artist": {"S": "No One You Know"}, "SongTitle": {"S": "Call Me Today"}, "AlbumTitle": {"S": "Somewhat Famous"}, "Awards": {"N": "1"}}' \
  --return-consumed-capacity TOTAL

aws dynamodb --endpoint-url=${DYNAMODB_ENDPOINT_URL} batch-write-item \
  --request-items file://fixtures/musics.json \
  --return-consumed-capacity INDEXES
```

2. 項目更新 UpdateItem

```
aws dynamodb --endpoint-url=${DYNAMODB_ENDPOINT_URL} update-item \
  --table-name Music \
  --key '{"Artist": {"S": "Acme Band"}, "SongTitle": {"S": "Happy Day"}}' \
  --update-expression "SET #Y = :y" \
  --expression-attribute-names '{"#Y":"Year"}' \
  --expression-attribute-values '{":y":{"N":"2020"}}' \
  --return-consumed-capacity INDEXES
```

3. 1項目の取得 GetItem

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

4. 複数の取得 Query (BatchGetItemは省略

```
aws dynamodb --endpoint-url=${DYNAMODB_ENDPOINT_URL} query \
  --table-name Music \
  --key-condition-expression "Artist = :artist" \
  --expression-attribute-values '{":artist": {"S":"No One You Know"}}' \
  --return-consumed-capacity INDEXES
```

5. 項目の削除 DeleteItem(省略)
