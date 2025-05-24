# GitHub Actionsの基礎概念

GitHub Actionsは**汎用的なワークフローエンジン**

次のようなタスクを自動化する
- プルリクエストが作成されたら、ビルドとテストを実行する
- リリースタグが作成されたら、コンテナイメージをデプロイする
- Issueが作成されたらランダムにメンバーをアサインする

GitHub Actionsは**ワークフロー**という単位でタスクを自動化する

ワークフローは
- いつ
- どこで
- どのような

    処理を実行するのかを定義する

GitHub Actionsはリポジトリに**ワークフローファイル**を追加するだけで実行できる

ワークフローファイルの規約
- ファイル拡張子は`.yml`または`.yaml`
- `.github/workflows`ディレクトリ直下へ配置
- 配置先ディレクトリと拡張子以外は任意のファイル名が使用可能

**ワークフローファイルの作成**
```shell
% mkdir -p .github/workflows #-pでpやディレクトリもまとめて作る
% touch .github/workflows/hello.yml # ワークフローファイル作成
```
**ワークフロー構文**

```yml
name: Hello                             # ワークフロー名
on: push                                # イベント(プッシュ時に起動)
jobs:                                   # ジョブの定義
    hello:                              # ジョブのID
        runs-on: ubuntu-latest          # ランナー(Ubuntuで実行)
        steps:                          # ステップの定義
            - run: echo "Hello, world!" # シェルコマンドの実行
            - uses: actions/checkout@v4 # アクションの呼び出し
```
- `on`で設定した**イベント**をきっかけに、**ワークフロー**が起動する
- ワークフローは指定した**ランナー**で1つ以上の**ジョブ**を実行する
- ジョブは複数のステップで構成され、各ステップで具体的な処理を実行する

## GitHub Actionsの構成要素

ワークフロー名
- **name**キーで指定、省略可能だが常に記述するべき

イベント
- **on**キーで指定、配列のように記述すれば複数イベント指定できる
```yml
on: [push, pull_request]
```

ジョブ
- **job**キー配下へ記述する、複数定義でき、区別のためにジョブIDを指定する

ランナー
- **runs-on**キーで実行環境を指定
- GitHub Actionsはこの指定に基づいてジョブの実行環境をプロビジョニングする

ステップ
- **step**キーへ記述する
- ワークフローにおける処理の最小単位
- ステップの実装方法は2つ
    1. シェルコマンドの実行
        - **run**キーで任意のシェルコマンドを実行する
    2. アクションの呼び出し
        - **uses**キーでアクションを呼び出せる
        - アクションはGitHub Actionsにおけるモジュールの一種
        - **with**キーを使えば入力パラメータも指定できる
        ```yaml
        - uses: actions/checkout@v4
          with:
            ref: main
        ```

## GitHub Actionsの実行

```shell
% git add .github/workflows/hello.yml
git commit -m "add hello.yml(2.3 GitHub Actinsの実行)"
% git push # 今回はpushが指定したイベント
```
- GitHubのActionsタブからワークフロー実行のログを確認できる

### GitHub CLIによるワークフロー実行ログの確認

- ワークフロー実行中の場合、
```shell
% gh run watch
```
- ワークフローが終了している場合、
```shell
% gh run view
```

で確認できる

実際の確認結果
```shell
% gh run view
? Select a workflow run ✓ add hello.yml(2.3 GitHub Actinsの実行), Hello [main] 7m21s ago

✓ main Hello · 15222468711
Triggered via push about 7 minutes ago

JOBS
✓ hello in 3s (ID 42820126705)

For more information about the job, try: gh run view --job=42820126705
View this run on GitHub: https://github.com/asato425/GitHub_CICD/actions/runs/15222468711
```

## GitHub Actionsのエラー

### 構文エラー
構文エラーには２種類ある

1. 構文エラー
```yml
name: workflow error
on: push
jobs:
  run:
    run-on: ubuntu-latest # runs-onが正しい
    steps:
      - run: date
```

2. YAML構文エラー
```yml
name: workflow error
on: push
jobs:
  run:
    runs-on: ubuntu-latest
      steps: # インデントが2文字ずれている
      - run: date
```

### 実行時エラー
ワークフローの実行中に発生する
- GitHub Actionsでは実行時エラーをコマンドの**終了ステータス**で判断する

**終了ステータス**
- コマンド実行時にOSが成功時は0,失敗時は0以外を返す

```shell
% invalid-command # 存在しないコマンドを実行
zsh: command not found: invalid-command
% echo $?         # 終了ステータスを確認
127               # 0以外の値→失敗とGitHub Actionsは判断
```

**※GitHub Actionsは終了ステータスで実行時エラーかどうかを判断するため、意図していない挙動でも正常終了(終了ステータス0)となる可能性あり**

## ワークフローの起動方法

- GitHub Actionsはイベント駆動型のサービス、様々なイベントをトリガーできる

### 手動実行
- `on`キーに`workflow_dispatch`を指定する
- 入力パラメータも指定できる

```yml
name: Manual
on:
  workflow_dispatch:                       # 手動実行イベント
    inputs:
      greeting:                            # 入力パラメータ名
        type: string                       # データ型（文字列）
        default: Hello                     # 入力パラメータのデフォルト値
        required: true                     # 入力パラメータの必須フラグ
        description: A cheerful word       # 入力パラメータの概要
jobs:
  run:
    runs-on: ubuntu-latest
    steps:
      - run: echo "${{ inputs.greeting }}" # 入力パラメータ「greeting」の参照
```
- ブラウザでRun workflowボタンから実行できる
- GitHub CLIからでも実行できる
```shell
% gh workflow run manual.yml -f greeting=good # 入力パラメータは-fで指定
bye
✓ Created workflow_dispatch event for manual.yml at main

To see runs for this workflow, try: gh run list --workflow=manual.yml
```

#### choiceによる列挙値の指定
- optionキーで指定可能な値を定義できる
```yml
inputs:
  log-level:
    type: choice  # 入力パラメータを特定の値に制限
    options:      # 受け入れる値を列挙
      - info
      - warn
      - error
```

### 定期実行
- `on`キーに`schedule`を指定する
- ワークフローの起動タイミングはcron形式で記述する
- タイムゾーンはUTCでJST(日本標準時)とは9時間ずれているため注意

```yml
name: Schedule
on:
  schedule:                # 定期実行イベント
    - cron: '*/15 * * * *' # 15分ごとに起動するcron式
jobs:
  run:
    runs-on: ubuntu-latest
    steps:
      - run: date
```