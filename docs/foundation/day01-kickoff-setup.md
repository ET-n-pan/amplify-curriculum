# Day 1: キックオフ & セットアップ

## ゴール
- 必要なツールのインストール
- Vue3.js + TypeScript開発環境のセットアップ
- プロジェクトフォルダー構成


## 環境セットアップ

### オプション１：インストーラーで環境構築
#### vscodeインストールページ
https://code.visualstudio.com/
![vscode-install](../images/screenshots/d1-install-vscode.png)

#### aws-cliインストールページ
https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
![aws-install](../images/screenshots/d1-install-aws-cli.png)

#### nodejsインストールページ
https://nodejs.org/ja/download
![nodejs-install](../images/screenshots/d1-install-nodejs.png)

#### Gitインストールページ
https://git-scm.com/downloads/win
![git-install](../images/screenshots/d1-install-git.png)

### オプション２：コマンドラインで環境構築
#### Powershellを管理者として実行
![powershell-admin](../images/screenshots/d1-powershell-admin.png)

#### Powershellにコマンドを実行
![choco-install](../images/screenshots/d1-install-choco.png)
```
# Chocolateyをダウンロードしてインストールする：
powershell -c "irm https://community.chocolatey.org/install.ps1|iex"

# Powershellを再起動、管理者として実行

# Chocolateyのバージョンを確認する,"v2.5.1"が表示される：
choco -v 
```

https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

#### 必要なツールをインストール
![env-install](../images/screenshots/d1-install-env.png)
```
# ツールインストール
choco install nodejs --version="22.19.0" -y; choco install vscode -y; choco install awscli -y

# Verify installations
node -v; npm -v; aws --version; code --version
# バージョンを確認
```
![env-version](../images/screenshots/d1-env-version.png)

## Vue3 + TypeScriptプロジェクト設定

```bash
# Vue3+Typescriptプロジェクトを作成
npm create vite@latest amplify-vue-ts-app
```
![install-vue](../images/screenshots/d1-install-vue.png)
![install-typescript](../images/screenshots/d1-install-Typescript.png)

```
# プロジェクトフォルダーに入る
cd amplify-vue-ts-app
# プロジェクトが必要なモジュールをインストール
npm install
```
![npm-install](../images/screenshots/d1-npm-install-real.png)

```
# ローカルでプロジェクトを起動
npm run dev

# ctrl+cでプロジェクトを終了
```

![npm-run](../images/screenshots/d1-npm-run.png)
![vue-success](../images/screenshots/d1-vue-success.png)

## トラブルシューティング

### Node.js バージョンコンフリクト
```bash
# 現在のノードバージョンを確認
node --version

# nvmで他のバージョンを使用
nvm use 18
nvm alias default 18
```

### ポートコンフリクト

ポート 5173 が既に使われている場合:
```bash
# 他のポートを使って起動
npm run dev --port 5174

# または vite.config.ts を更新して、デフォルトポートを変更
```

### ビルドエラー

```bash
# node_modulesフォルダーを削除して再インストール
rm -rf node_modules package-lock.json
npm install

# キャッシュクリア
npx vite --clearCache
```