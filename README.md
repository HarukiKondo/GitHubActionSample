# GitHubActionSample
GitHubActionSample

### 今回作成したサンプル用の CI/CDファイル

```yml
name: CI

on: [push]

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
          key: ${{ runner.os }}-${{ hashFiles('**/yarn.lock') }}
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
          key: ${{ runner.os }}-${{ hashFiles('**/yarn.lock') }}
      - name: Frontend build
        run: cd frontend && npm run build
        env:
          CI: false
```
