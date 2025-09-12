# AWS Amplify Gen2 学習カリキュラム

## ゴール
* Amplify Gen2のコードファスト(TypeScript)で、Auth・Data・Functionsの基本を扱える
* Vue + TypescriptでUI5 Web Componentを使い、Fiori風の一覧→明細→更新の一連UIを構築する
* SAP ODataにAmplifyのバックエンド側(HTTPデータソース/Lambda)経由で安全に連携する
* Git連携のブランチごとのフルスタック自動デプロイ化、改修の正しい反映手順を理解する

## サンプルアプリケーション
本学習カリキュラムでは、受注業務を題材として、入力データ（単票登録およびファイルアップロードによる一括登録）をSAP S/4HANAへ連携する、アドオンアプリケーションを開発します。  
完成した画面のイメージは以下の図の通りです。  
![完成した画面のイメージ](../images/screenshots/xxxxxxxxxx.png)


## アーキテクチャ
本学習カリキュラムで題材とするアプリケーションのアークテクチャ構成は以下の図の通りです。  
![アーキテクチャ構成図](../images/screenshots/xxxxxxxxxx.png)

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

## スケジュール

**準備 (1-3日目)**
- 環境確認、Vue3+Typescriptの起動
- Amplify Gen2 CLI (Command Line Interface)、Sandboxの起動
- AWS Auth導入、Cognitoサインイン/アウト実装

**UI開発 (4-6日目)**
- SAP UI5 web component利用、Fiori風UIで、いくつかのコンポーネントの再現
- 簡単な計算式を埋め込んだUIをAgGridの利用で試作

**データ管理 (7-10日目)**
- Amplify Dataの基本
- SAP ODataの理解（SandBoxでの叩き方)
- バックエンド連携

**アプリケーション開発 (11-14日目)**
- ドメイン決定&API Design
- フロントエンド実装&更新処理実装①
- エラーハンドリングなど

**振り返り (15日目)**
- コードレビューと最適化
- 実装戦略
- パフォマンス監視

## 学習を開始するには
[Day 1: キックオフ & セットアップ](foundation/day01-kickoff-setup.md)より環境をセットアップし、順番にハンズオンを実施してください。  
問題発生時には各日のカリキュラムにあるトラブルシューティングの項を参照してください。
