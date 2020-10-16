# DynamoDB概要

> Amazon DynamoDB は、フルマネージド型の NoSQL データベースサービスで、高速で予測可能なパフォーマンスとシームレスなスケーラビリティを特長としています。DynamoDB を使用すると、分散データベースの運用とスケーリングに伴う管理作業をまかせることができるため、ハードウェアのプロビジョニング、設定と構成、レプリケーション、ソフトウェアのパッチ適用、クラスタースケーリングなどを自分で行う必要はなくなります。また、DynamoDB も保管時の暗号化を提供し、機密データの保護における負担と複雑な作業を解消します。
> DynamoDB は十分な数のサーバー間でデータとトラフィックを自動的に分散し、一貫した高速なパフォーマンスを維持したまま、スループットとストレージの要件に対応します。すべてのデータは SSD (Solid State Disk) に保存され、1 つの AWS リージョン内の複数のアベイラビリティーゾーン間で自動的にレプリケートすることによって、高い可用性とデータ堅牢性を実現します。グローバルテーブルを使用すると、DynamoDB テーブルを AWS リージョン間で継続して同期することができます。

らしいです。

## 基本構成

 - tables       テーブルでデータ管理するよ
 - items        MySQLでいうレコード的感じ
 - attirubtes   属性。主キー以外はスキーマレス（インデックスも？

## 項目操作API

 読み込み
 | Operation | 概要 | その他 |
 | ----- | ----- | ----- |
 | GetItem | 主キーで1項目取得 | |
 | BatchGetItem | 1つ以上のテーブルから指定したキーの項目を取得 | 1度に100項目まで取得可能 |
 | Query | 指定したパーティションキーの項目全て取得。ソートキーに対しての条件で検索も可能 | 使い方によっては、BatchGetItemよりRCUを節約できるかも |
 | Scan | 全ての項目を取得 | |

 書き込み
 | Operation | 概要 | その他 |
 | ----- | ----- | ----- |
 | PutItem | 項目の作成 | |
 | BatchWriteItem | 1つのテーブルに複数項目を一度に作成 | 1度に25項目まで取得可能、BatchGetItemと違い1テーブル、削除も可能 |
 | UpdateItem | 主キーを指定して、Attributeを修正 | |
 | DeleteItem | 主キーで項目削除 | |

#### 例

0. テーブル作成（String型のパーティションキーArtists、String型のソートキーSongTitleが主キーのテーブルを作成）

```
DYNAMODB_ENDPOINT_URL=http://localhost:18080

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
