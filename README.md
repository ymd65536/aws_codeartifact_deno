# aws_codeartifact_deno

AWS CodeArtifactとdenoの連携

## Run these commands to get started

## Run the program

```bash
deno run main.ts
```

## Run the program and watch for file changes

```bash
deno task dev
```

## Run the tests

```bash
deno test
```

## Handson

### 環境変数をセットする

```bash
export PROFILE_NAME="artifacttest"
export AWS_DOMAIN="deno-art" && echo $AWS_DOMAIN
export REPOSITORY_NAME="deno-pack"

export AWS_ACCOUNT_ID=`aws sts get-caller-identity --profile $PROFILE_NAME --query 'Account' --output text` && echo $AWS_ACCOUNT_ID
export AWS_DEFAULT_REGION="ap-northeast-1" && echo $AWS_DEFAULT_REGION
```

### 補足：IAM Identity Centerを利用している場合

環境変数をセットした後にログインを忘れないようにしてください。
以下のコマンドでログインできます。

```sh
aws sso login --profile $PROFILE_NAME
```

### ドメインを作成する

AWS CLIでドメインを作成します。

```sh
aws codeartifact create-domain --domain $AWS_DOMAIN --profile $PROFILE_NAME
```

### リポジトリを作成する

ドメインではパッケージを格納できないため、リポジトリを作成します。

```sh
aws codeartifact create-repository --domain $AWS_DOMAIN --domain-owner $AWS_ACCOUNT_ID --repository $REPOSITORY_NAME --profile $PROFILE_NAME
```

### npm-storeを作成する

リポジトリにはnpmのパッケージをいれたいため、まずはnpm-storeを作成します。

```sh
aws codeartifact create-repository --domain $AWS_DOMAIN --domain-owner $AWS_ACCOUNT_ID --repository npm-store --profile $PROFILE_NAME
```

### リポジトリとnpm-store を接続する

npm-storeを作成したあとはリポジトリとnpm-storeを接続します。

```sh
aws codeartifact associate-external-connection --domain $AWS_DOMAIN  --domain-owner $AWS_ACCOUNT_ID --repository npm-store --external-connection "public:npmjs" --profile $PROFILE_NAME
```

### リポジトリを更新する

最後にリポジトリを更新します。

```sh
aws codeartifact update-repository --repository $REPOSITORY_NAME --domain $AWS_DOMAIN  --domain-owner $AWS_ACCOUNT_ID --upstreams repositoryName=npm-store --profile $PROFILE_NAME
```

### CodeArtifactにログイン

エンドポイントの取得やトークンの取得が必要となる為、CodeArtifactににログインします。

```sh
aws codeartifact login --tool npm --domain $AWS_DOMAIN --region $AWS_DEFAULT_REGION --domain-owner $AWS_ACCOUNT_ID --repository $REPOSITORY_NAME --profile $PROFILE_NAME
```

### エンドポイントURLの取得とCODEARTIFACT_AUTH_TOKENの発行

```sh
export CODEARTIFACT_URL=`aws codeartifact get-repository-endpoint --domain $AWS_DOMAIN --domain-owner $AWS_ACCOUNT_ID --repository $REPOSITORY_NAME --format npm --profile $PROFILE_NAME` && echo $CODEARTIFACT_URL
export CODEARTIFACT_AUTH_TOKEN=`aws codeartifact get-authorization-token --domain $AWS_DOMAIN --region $AWS_DEFAULT_REGION --domain-owner $AWS_ACCOUNT_ID --query authorizationToken --output text --profile $PROFILE_NAME` && echo $CODEARTIFACT_AUTH_TOKEN
```

## パッケージを登録

CodeArtifactにパッケージが登録されていないことを確認します。

```sh
aws codeartifact list-packages --domain $AWS_DOMAIN --repository $REPOSITORY_NAME --query 'packages' --output text --profile $PROFILE_NAME
```

```sh
cd ./lib
```

CodeArtifactにパッケージを登録します。

```sh
deno publish
```

登録されたパッケージの一覧を表示します。

```sh
aws codeartifact list-packages --domain $AWS_DOMAIN --repository $REPOSITORY_NAME --profile $PROFILE_NAME

```

## CodeArtifactに登録したパッケージをsample-appに読み込む

サンプルアプリケーションの`sample-app`に作成したパッケージを読み込みます。

```sh
cd ../sample-app
```

`npm install`を実行して`index.js`を実行します。

```sh
npm install sample-package@1.0.0 && deno run main.ts
```
