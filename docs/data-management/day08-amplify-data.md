# Day 8: Amplify Dataの基本

## ゴール
!!! success "Day 8 Goals"
    - Amplify Dataのセットアップと理解
    - データモデルの定義
    - クライアント生成
    - 基本的なCRUD操作の実装

## Amplify Dataとは
Amplify DataはAWS AppSyncを基盤とした、リアルタイム対応のGraphQL APIサービスです。

### 主な特徴
- **リアルタイム更新**: WebSocketを使用したリアルタイム通信
- **型安全性**: TypeScriptサポートとコード生成機能
- **認証連携**: Amplify Authとのシームレスな統合
- **スケーラビリティ**: GraphQLクエリを最適化したAPIとクライアント

## データスキーマの設定
### Step 1: データモデルの定義
`amplify/data/resource.ts`ファイルを作成してスキーマを定義します


### Step 3: サンドボックスの起動
```bash
npx ampx sandbox
```

## クライアント連携の実装


## トラブルシューティング
### よくあるエラーと対処法

1. **GraphQL API エラー**
   ```
   Error: No current user
   ```
   - 認証設定を確認してください
   - `authorization`の設定を確認してください

2. **モジュールエラー**
   ```
   Cannot find module '../amplify_outputs.json'
   ```
   - `npx ampx sandbox`でサンドボックスが起動していることを確認
   - `amplify_outputs.json`が生成されていることを確認

3. **データが表示されない**
   - ネットワークタブでAPI呼び出しが成功しているかを確認
   - コンソールエラーを確認
   - Amplify設定が正しく読み込まれているかを確認

## 参考資料
- [Amplify Data 公式ドキュメント](https://docs.amplify.aws/vue/build-a-backend/data/)
- [GraphQL API クエリ](https://docs.amplify.aws/vue/build-a-backend/data/query-data/)
- [リアルタイム購読](https://docs.amplify.aws/vue/build-a-backend/data/subscribe-data/)