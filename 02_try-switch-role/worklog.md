## このファイルについて
02_try-switch-roleディレクトリのリソースを使用して試したことを説明するためのドキュメントです
本書と件のディレクトリのリソースで筆者と同じことが再現できます

### switcherとswitcheeの作成

* こちらを参考に作成します
    * [AWS CLI を使用して IAM ロールを引き受ける](https://aws.amazon.com/jp/premiumsupport/knowledge-center/iam-assume-role-cli/)

#### switcherのIAMユーザを作成

アカウントAで操作します

##### ユーザ作成
```
$ aws iam create-user --user-name Bob
```

##### ポリシー作成
```
$ vim example-policy.json
$ aws iam create-policy --policy-name example-policy --policy-document file://example-policy.json
```

##### ポリシーアタッチ
アタッチを実行します
```
$ aws iam attach-user-policy \
--user-name Bob \
--policy-arn "arn:aws:iam::xxxxxxxxxxxx:policy/example-policy"
```
※create-policyの表示結果を参考にarnを指定します。

アタッチ結果を確認します
```
$ aws iam list-attached-user-policies --user-name Bob
```
```
{
    "AttachedPolicies": [
        {
            "PolicyName": "example-policy",
            "PolicyArn": "arn:aws:iam::xxxxxxxxxxxx:policy/example-policy"
        }
    ]
}
```

##### 信頼関係定義用JSONを用意
```
$ vim example-role-trust-policy.json
```

#### switcheeのIAMロールを作成

アカウントBで操作します（CLIの環境変数を切り替えてください）

##### ロールを作成

```
aws iam create-role \
--role-name example-role \
--assume-role-policy-document file://example-role-trust-policy.json

aws iam attach-role-policy \
--role-name example-role \
--policy-arn "arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess"

aws iam list-attached-role-policies \
--role-name example-role
```

#### CLIを使ってswitch-roleして一時クレデンシャルを取得

アカウントAで操作します（CLIの環境変数を切り替えてください）

```
aws sts assume-role \
--role-arn arn:aws:iam::xxxxxxxxxxxx:role/Bob-role \
--role-session-name Bob
```
※アカウントBに作成したロールのarnを設定

### switch-roleでNoSQL Workbenchにアクセス確認

```
$ aws sts assume-role \
> --role-arn arn:aws:iam::xxxxxxxxxxxx:role/Bob-role \
> --role-session-name Bob
{
    "Credentials": {
        "AccessKeyId": "xxx",
        "SecretAccessKey": "xxx",
        "SessionToken": "xxx",
        "Expiration": "xxx"
    },
    "AssumedRoleUser": {
        "AssumedRoleId": "xxx:Bob",
        "Arn": "arn:aws:sts::xxxxxxxxxxxx:assumed-role/Bob-role/Bob"
    }
}
```
上記の結果をもとに。図のように指定してConnectionを作成します。
https://gyazo.com/0bcb2933330cfe49a3a2fb020f6f9db2
※ IAM role or ARNには指定なしです。