## このファイルについて
01_try-basic-operationディレクトリのリソースを使用して試したことを説明するためのドキュメントです
本書と件のディレクトリのリソースで筆者と同じことが再現できます

## DDBにテストデータを作成
こちらの公式記事を参考に、DDBにテストデータを作成します。
[AWS CLI の使用 - Amazon DynamoDB](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/Tools.CLI.html)

### Musicテーブルを作成します

```
$ aws dynamodb create-table \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema AttributeName=Artist,KeyType=HASH AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1
```

### テストデータを入れます    

```
$ aws dynamodb put-item \
--table-name Music  \
--item \
    '{"Artist": {"S": "No One You Know"}, "SongTitle": {"S": "Call Me Today"}, "AlbumTitle": {"S": "Somewhat Famous"}}' \
--return-consumed-capacity TOTAL
```

```
$aws dynamodb put-item \
    --table-name Music \
    --item '{ "Artist": {"S": "Acme Band"}, "SongTitle": {"S": "Happy Day"}, "AlbumTitle": {"S": "Songs About Life"} }' \
    --return-consumed-capacity TOTAL
```

#### エラー
後者のコマンドは、ドキュメントそのままの記述だとエラーになりましたので、変えています。
内容は変えずに一行にまとめただけです。

* コマンド
```
$ aws dynamodb put-item \
    --table-name Music \
    --item '{ \
        "Artist": {"S": "Acme Band"}, \
        "SongTitle": {"S": "Happy Day"}, \
        "AlbumTitle": {"S": "Songs About Life"} }' \
    --return-consumed-capacity TOTAL
```
* 結果
```
Error parsing parameter '--item': Invalid JSON: Expecting property name enclosed in double quotes: line 1 column 3 (char 2)
JSON received: { \
        "Artist": {"S": "Acme Band"}, \
        "SongTitle": {"S": "Happy Day"}, \
        "AlbumTitle": {"S": "Songs About Life"} }
```



### マネジメントコンソールではこのような状態になります
https://gyazo.com/7d2bea7c7b7827ab2154b328f26df1f8

### jsonファイルを読み込んでクエリします

```
$ aws dynamodb query --table-name Music --key-conditions file://key-conditions.json
```

結果が正常に返ってきました
```
$ aws dynamodb query --table-name Music --key-conditions file://key-conditions.json
{
    "Items": [
        {
            "AlbumTitle": {
                "S": "Somewhat Famous"
            },
            "Artist": {
                "S": "No One You Know"
            },
            "SongTitle": {
                "S": "Call Me Today"
            }
        }
    ],
    "Count": 1,
    "ScannedCount": 1,
    "ConsumedCapacity": null
}
```

### NoSQL Workbenchでアクセスしてみます

* こちらの公式記事を参考に、先ほど作成したDDBのテーブルにアクセスしてみます。
    * [データセットの探索 - Amazon DynamoDB](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/workbench.querybuilder.connect.html)

#### インストール

* まず、こちらのサイトからインストールします
    * https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/workbench.settingup.html

#### 起動

* [Operation Builder] - [Add connection]で接続を作成し、作成した接続をOpenします
* [Tables] - [Music]を選択し、先程CLIで行ったのと同じ操作をしてみます
    - AlbumTitle=Somewhat Famous
    - Artist=No One You Know"
    - SongTitle=Call Me Today"
* と、思ったけどあれ！一つしかconditionを指定できないな
    - https://gyazo.com/3a3cc1e052cdd49bdcd010249d3e7440
* ・・と思ったけど、右上のBuild operationsから指定できました。こちらでやってみます
* CLIと同じ結果が出ました。
    - https://gyazo.com/ae62d48b6f833b7e9f1d4a9780ebd578
