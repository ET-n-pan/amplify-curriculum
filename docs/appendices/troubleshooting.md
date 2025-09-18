# トラブルシューティングガイド

この包括的なトラブルシューティングガイドは、AWS Amplify開発トレーニングプログラム中に発生する一般的な問題をカバーします。

## 環境セットアップの問題

### Node.jsとnpmの問題

!!! error "Nodeバージョンの非互換性"
    **症状**: ビルド失敗、依存関係の競合、またはCLIコマンドが動作しない
    
    **解決策**:
    ```bash
    # 現在のNodeバージョンを確認
    node --version
    
    # Node Version Manager (nvm)をインストール
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
    
    # Node 18+をインストールして使用
    nvm install 18
    nvm use 18
    nvm alias default 18
    ```

!!! error "npmアクセス権エラー"
    **症状**: グローバルパッケージインストール時に`EACCES`エラー
    
    **解決策**:
    ```bash
    # npmが異なるディレクトリを使用するよう設定
    mkdir ~/.npm-global
    npm config set prefix '~/.npm-global'
    echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.profile
    source ~/.profile
    
    # 代替手段: アクセス権を修正
    sudo chown -R $(whoami) $(npm config get prefix)/{lib/node_modules,bin,share}
    ```

### AWS CLIと認証情報

!!! error "AWS CLIが見つからない"
    **症状**: `aws: command not found`
    
    **解決策**:
    ```bash
    # HomebrewでmacOS
    brew install awscli
    
    # Windows
    # こちらからダウンロード: https://awscli.amazonaws.com/AWSCLIV2.msi
    
    # Linux
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install
    ```

!!! error "無効な認証情報"
    **症状**: `Unable to locate credentials`またはアクセス拒否エラー
    
    **解決策**:
    ```bash
    # AWS CLIを再設定
    aws configure
    
    # 現在の設定を確認
    aws configure list
    aws sts get-caller-identity
    
    # 代替手段: 環境変数を使用
    export AWS_ACCESS_KEY_ID=your_access_key
    export AWS_SECRET_ACCESS_KEY=your_secret_key
    export AWS_DEFAULT_REGION=us-east-1
    ```

## Amplify CLIの問題

### インストールの問題

!!! error "Amplify CLIインストール失敗"
    **症状**: npm install失敗またはCLIコマンドが見つからない
    
    **解決策**:
    ```bash
    # npmキャッシュをクリア
    npm cache clean --force
    
    # アンインストールと再インストール
    npm uninstall -g @aws-amplify/cli
    npm install -g @aws-amplify/cli@latest
    
    # インストールを確認
    amplify --version
    which amplify
    ```

### サンドボックスの問題

!!! error "サンドボックスデプロイ失敗"
    **症状**: リソースのデプロイ失敗、タイムアウトエラー
    
    **解決策**:
    ```bash
    # AWSサービス制限を確認
    aws service-quotas get-service-quota --service-code amplify --quota-code L-123456
    
    # Amplifyキャッシュをクリア
    rm -rf ~/.amplify
    rm -rf .amplify
    
    # 異なるリージョンを試す
    aws configure set region us-west-2
    
    # 詳細ログでサンドボックスを再開
    amplify sandbox --debug
    ```

!!! error "サンドボックスがデプロイ中にスタック"
    **症状**: デプロイが無限にハング
    
    **解決策**:
    ```bash
    # 現在のデプロイを強制停止
    Ctrl+C
    
    # CloudFormationスタックを確認
    aws cloudformation list-stacks --stack-status-filter CREATE_IN_PROGRESS UPDATE_IN_PROGRESS
    
    # スタックしたリソースを削除
    amplify delete
    
    # 待機して再試行
    sleep 60
    amplify sandbox
    ```

## Vue.js連携の問題

### TypeScriptコンパイルエラー

!!! error "TypeScriptビルド失敗"
    **症状**: `tsc`エラー、型チェック失敗
    
    **解決策**:
    ```bash
    # TypeScriptを更新
    npm install -D typescript@latest
    
    # TypeScriptキャッシュをクリア
    rm -rf node_modules/.cache
    npx tsc --build --clean
    
    # tsconfig.json設定を確認
    npx tsc --showConfig
    ```

!!! error "モジュール解決の問題"
    **症状**: インポートを解決できない、パスマッピングが動作しない
    
    **解決策**:
    ```json
    // tsconfig.jsonを更新
    {
      "compilerOptions": {
        "baseUrl": ".",
        "paths": {
          "@/*": ["./src/*"]
        },
        "moduleResolution": "bundler"
      }
    }
    ```
    
    ```typescript
    // vite.config.tsを更新
    import path from 'path'
    
    export default defineConfig({
      resolve: {
        alias: {
          '@': path.resolve(__dirname, './src')
        }
      }
    })
    ```

### Amplify設定の問題

!!! error "amplify_outputs.jsonが見つからない"
    **症状**: amplify_outputs.jsonのモジュールが見つからないエラー
    
    **解決策**:
    ```bash
    # サンドボックスが実行中であることを確認
    amplify sandbox
    
    # ファイルが存在するか確認
    ls -la amplify_outputs.json
    
    # 設定を手動で生成
    amplify generate config
    
    # 必要に応じて型定義を作成
    echo 'declare module "*amplify_outputs.json"' > src/amplify_outputs.d.ts
    ```

!!! error "Amplify設定エラー"
    **症状**: ランタイム設定エラー、API呼び出し失敗
    
    **解決策**:
    ```typescript
    // main.tsで設定をデバッグ
    import outputs from '../amplify_outputs.json'
    console.log('Amplify config:', outputs)
    
    // 設定構造を検証
    if (!outputs.auth || !outputs.data) {
      console.error('無効なAmplify設定')
    }
    
    // Amplifyを再設定
    import { Amplify } from 'aws-amplify'
    Amplify.configure(outputs)
    ```

## 認証の問題

### サインアップ/サインインの問題

!!! error "ユーザーが既に存在"
    **症状**: SignUp操作がUserAlreadyExistsExceptionを返す
    
    **解決策**:
    ```typescript
    // サインアップ前にユーザーが存在するか確認
    import { signIn } from 'aws-amplify/auth'
    
    try {
      await signIn({ username, password })
      // ユーザーが存在、アプリにリダイレクト
    } catch (error) {
      if (error.name === 'UserNotFoundException') {
        // サインアップを続行
        await signUp({ username, password })
      }
    }
    ```

!!! error "確認コードの問題"
    **症状**: 無効な確認コード、コードの有効期限切れ
    
    **解決策**:
    ```typescript
    // 確認コードを再送
    import { resendSignUpCode } from 'aws-amplify/auth'
    
    await resendSignUpCode({ username })
    
    // 新しいコードのためにメール/SMSを確認
    // コードが正しく入力されていることを確認（スペースなし）
    ```

### セッション管理

!!! error "トークン有効期限切れエラー"
    **症状**: しばらくするとAPI呼び出しが許可されないを返す
    
    **解決策**:
    ```typescript
    // 自動トークン更新を実装
    import { fetchAuthSession, signOut } from 'aws-amplify/auth'
    
    try {
      const session = await fetchAuthSession()
      if (!session.tokens) {
        throw new Error('有効なトークンがありません')
      }
    } catch (error) {
      // ログインにリダイレクト
      await signOut()
      router.push('/login')
    }
    ```

## データ/APIの問題

### GraphQLスキーマの問題

!!! error "GraphQLスキーマ同期エラー"
    **症状**: クライアントクエリがバックエンドスキーマと一致しない
    
    **解決策**:
    ```bash
    # GraphQLクライアントコードを再生成
    amplify generate graphql-client-code --format typescript
    
    # スキーマを更新してサンドボックスを再開
    # amplify/data/resource.tsを編集
    # Ctrl+Cでサンドボックスを停止
    amplify sandbox
    ```

!!! error "許可エラー"
    **症状**: GraphQL操作への無許可アクセス
    
    **解決策**:
    ```typescript
    // スキーマの許可ルールを確認
    const schema = a.schema({
      Todo: a
        .model({
          content: a.string(),
        })
        .authorization((allow) => [
          allow.owner(),
          allow.authenticated().to(['read'])
        ]),
    })
    
    // API呼び出し前にユーザーが認証されていることを確認
    import { getCurrentUser } from 'aws-amplify/auth'
    
    try {
      await getCurrentUser()
      // ユーザーが認証されている、API呼び出しを続行
    } catch (error) {
      // ログインにリダイレクト
    }
    ```

### ネットワークとCORSの問題

!!! error "CORSポリシーエラー"
    **症状**: ブラウザコンソールでCORSポリシーによってブロックされる
    
    **解決策**:
    ```typescript
    // CORSはAmplifyによって自動的に処理されます
    // 正しいAPIエンドポイントを使用していることを確認
    
    // 正しいURLのためにamplify_outputs.jsonを確認
    import outputs from '../amplify_outputs.json'
    console.log('API URL:', outputs.data?.url)
    
    // サンドボックスが実行中でアクセス可能であることを確認
    curl -X POST outputs.data.url -H "Content-Type: application/json"
    ```

## ビルドとデプロイの問題

### ビルド失敗

!!! error "Viteビルドエラー"
    **症状**: ビルドコマンドが様々なエラーで失敗
    
    **解決策**:
    ```bash
    # ビルドキャッシュをクリア
    rm -rf dist
    rm -rf node_modules/.vite
    
    # 依存関係を更新
    npm update
    
    # 詳細出力でビルド
    npm run build -- --debug
    
    # 構文エラーを確認
    npx tsc --noEmit
    ```

!!! error "ビルド中のメモリ問題"
    **症状**: JavaScriptヒープのメモリ不足エラー
    
    **解決策**:
    ```bash
    # Node.jsメモリ制限を増やす
    export NODE_OPTIONS="--max-old-space-size=4096"
    npm run build
    
    # またはpackage.jsonを修正
    {
      "scripts": {
        "build": "NODE_OPTIONS=--max-old-space-size=4096 vite build"
      }
    }
    ```

## パフォーマンスの問題

### 開発サーバーの動作が遅い

!!! error "遅いホットモジュールリプレースメント (HMR)"
    **症状**: 変更がブラウザに反映されるまで時間がかかる
    
    **解決策**:
    ```typescript
    // Vite設定を最適化
    export default defineConfig({
      server: {
        hmr: {
          overlay: false
        },
        host: true
      },
      optimizeDeps: {
        include: ['aws-amplify', 'vue', 'vue-router', 'pinia']
      }
    })
    ```

### 大きなバンドルサイズ

!!! error "バンドルサイズが大きすぎる"
    **症状**: 読み込み時間が遅い、distファイルが大きい
    
    **解決策**:
    ```typescript
    // コード分割を実装
    export default defineConfig({
      build: {
        rollupOptions: {
          output: {
            manualChunks: {
              vendor: ['vue', 'vue-router', 'pinia'],
              amplify: ['aws-amplify']
            }
          }
        }
      }
    })
    
    // 大きなコンポーネントに動的インポートを使用
    const HeavyComponent = defineAsyncComponent(() => import('./HeavyComponent.vue'))
    ```

## デバッグツールとコマンド

### 便利なデバッグコマンド

```bash
# すべてのバージョンを確認
node --version
npm --version
aws --version
amplify --version

# AWSデバッグ
aws sts get-caller-identity
aws configure list
aws logs describe-log-groups

# Amplifyデバッグ
amplify status
amplify env list
amplify diagnose

# ネットワークデバッグ
curl -I https://api-endpoint
nslookup api-endpoint
ping api-endpoint
```

### ブラウザ開発者ツール

1. **Consoleタブ**: JavaScriptエラーと警告を確認
2. **Networkタブ**: APIリクエストとレスポンスを監視
3. **Applicationタブ**: ローカルストレージとセッションデータを検査
4. **Sourcesタブ**: ブレークポイントを設定してコード実行をデバッグ

### ログ分析

```bash
# Amplifyログを表示
cat ~/.amplify/logs/amplify-cli.log

# AWS CloudFormationイベントを表示
aws cloudformation describe-stack-events --stack-name amplify-*

# Lambda関数ログを表示
aws logs tail /aws/lambda/function-name --follow
```

## 追加サポートの入手

### コミュニティリソース

- **AWS Amplify Discord**: [discord.gg/amplify](https://discord.gg/amplify)
- **GitHub Issues**: [github.com/aws-amplify/amplify-cli/issues](https://github.com/aws-amplify/amplify-cli/issues)
- **Stack Overflow**: `aws-amplify`タグで質問
- **AWS Forums**: [forums.aws.amazon.com](https://forums.aws.amazon.com)

### AWSサポートオプション

- **AWSドキュメント**: [docs.amplify.aws](https://docs.amplify.aws)
- **AWSサポートセンター**: 有料サポートプラン用
- **AWS re:Post**: コミュニティ主導Q&Aプラットフォーム

!!! tip "サポートを求める前に"
    1. 既存の問題とドキュメントを検索する
    2. 完全なエラーメッセージとログを提供する
    3. 関連する設定ファイルを含める
    4. すべてのツールと依存関係のバージョンを指定する
    5. 問題を再現する手順を説明する

