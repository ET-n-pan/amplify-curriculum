# AWS Amplify Gen2 学習カリキュラム

## ゴール
* Amplify Gen2のコードファスト(TypeScript)で、Auth・Data・Functionsの基本を扱える
* Vue + TypescriptでUI5 Web Componentを使い、Fiori風の一覧→明細→更新の一連UIを構築
* SAP ODataにAmplifyのバックエンド側(HTTPデータソース/Lambda)経由で安全に連携する
* Git連携のブランチごとのフルスタック自動デプロイ化、改修の正しい反映手順を理解

## スケジュール

**基礎 (1-3日目)**
- 環境確認、Vue3+Typescriptの起動
- Amplify Gen2 CLI (Command Line Interface)、Sandboxの起動
- AWS Auth導入、Cognitoサインイン/アウト実装

**UI 開発 (4-6日目)**
- SAP UI5 web component利用、Fiori風UIで、いくつかのコンポーネントの再現
- 簡単な計算式を埋め込んだUIをAgGridの利用で試作

**データ管理 (7-10日目)**
- Amplify Dataの基本
- SAP ODataの理解（SandBoxでの叩き方)
- バックエンド連携

**アプリケーション 開発 (11-14日目)**
- ドメイン決定&API Design
- フロントエンド実装&更新処理実装①
- エラーハンドリングなど

**仕上がり (15日目)**
- コードレビューと最適化
- 実装戦略
- パフォマンス監視

## テクノロジー
| Technology | Version | Purpose |
|------------|---------|----------|
| Vue.js | 3.x | フロントエンドフレームワーク |
| TypeScript | 5.x | Type-safe JavaScript 開発 |
| AWS Amplify | Gen2 | Backend-as-a-service platform |
| UI5 Web Components | Latest | Enterprise-grade UI components |
| AG Grid | Community/Enterprise | データ可視化 |
| AWS Cognito | - | 認証とユーザー管理 |
| SAP OData | v2/v4 | Enterprise data integration |

## 次に
1. [Day 1: Kickoff & Setup](foundation/day01-kickoff-setup.md)から環境をセットアップ
2. 順番にハンズオン実施
3. 問題発生時に付録のトラブルシューティングを参照
