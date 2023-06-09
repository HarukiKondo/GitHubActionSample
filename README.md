# GitHubActionSample
GitHubActionSample

### 今回作成したサンプル用の CI/CDファイル

```yml
name: CI

on: [push]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  # npm モジュールのインストール
  setup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - uses: actions/cache@v3
        id: npm-cache
        with:
          path: "**/node_modules"
          key: ${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}-{{ checksum "patches.hash" }}
      - name: Install packages
        run: cd frontend && npm ci

  # フロントエンドのテスト実施
  frontend-test:
    runs-on: ubuntu-latest
    needs: [setup]
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - uses: actions/cache@v3
        with:
          path: "**/node_modules"
          key: ${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
      - name: Frontend test
        run: cd frontend && npm run test
        env:
          CI: true

  # フロントエンドのビルド
  frontend-build:
    runs-on: ubuntu-latest
    needs: [frontend-test, setup]
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - uses: actions/cache@v3
        with:
          path: "**/node_modules"
          key: ${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
      - name: Frontend build
        run: cd frontend && npm run build
        env:
          CI: false
          PUBLIC_URL: /GitHubActionSample
        # ビルド成果物をアップロード
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: './frontend/build'

  # GitHub Pagesを使って成果物を公開する
  GiHub-pages:
    runs-on: ubuntu-latest
    needs: [frontend-build]
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```

### 実際に動作した GitHub Actionの例

- 動作記録

    ![](./assets/1.png)

- 実行されたワークフロー一覧
    npmモジュールのインストール⇒テスト⇒ビルド

    ![](./assets/2.png)

- npmモジュールのインストール Action

    sample.ymlファイルで定義されたコマンドが実行されてモジュールがインストールされている。

    ![](./assets/3.png)

- フロントエンドのテスト Action

    sample.ymlファイルで定義されたコマンドが実行されてApp.tsxのコンポーネントがレンダリングされているかのテストが実行されている。

    ![](./assets/4.png)

- フロントエンドのビルド Action

    sample.ymlファイルで定義されたコマンドが実行されてフロントエンドのビルドが行われている。

    ![](./assets/5.png)

- リポジトリの設定ファイルから、GitHub Actionsによる成果物の公開設定をONにする。

  ![](./assets/6.png)

- GitHub pagesでのデプロイ Action

  ![](./assets/7.png)