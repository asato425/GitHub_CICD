# GitHub AActionsの基礎概念

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