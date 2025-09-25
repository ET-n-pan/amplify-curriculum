# Day 3: Authentication (Cognito)


## ゴール
!!! success "Day 3 Goals"
    - フロントエンドにログイン機能を実装する
    - Vue Routerを使用したページ遷移の実装
    - デフォルト（ホーム）ページの設定
    - Amazon Cognitoユーザープールでのユーザー管理

## ルーターコンポーネントのインストール
ターミナルでプロジェクトフォルダーを開き、以下のコマンドを実行します。
```
npm install vue-router
```
依存関係に関するwarningメッセージが出力されますが、想定内のものであるため、そのまま先に進めてください。

```
#このままフロントエンドを起動
npm run dev
```

## プロジェクトフォルダー確認と作成
src/フォルダーの下にlayout/とrouter/を作成します。
```
amplify-vue-ts-project/
├── amplify/
│   ├── auth/                # 認証用設定
│   ├── data/                # データモデルとAPIスキーマ
│   ├── backend.ts           # バックエンド設定ファイル
│   └── package.json         # バックエンド依存
├── src/
│   ├── asset/               # 静的ファイル（画像、アイコン、フォント等）
│   ├── components/          # 再利用可能なVueコンポーネント（ボタン、フォーム等）
│   ├── layout/              # →フォルダーを作成。ページレイアウトコンポーネント
│   ├── router/              # →フォルダーを作成。ルーティング設定（ページ遷移、認証ガード）
│   ├── App.vue              # アプリケーションのルートコンポーネント
│   ├── main.ts              # アプリケーションエントリーポイント（Vue初期化）
│   ├── style.css            # グローバルスタイル（共通CSS）
│   └── (...)
└── amplify_outputs.json     # Amplify生成した設定ファイル（APIエンドポイント、認証情報）
```
## tsconfig.json更新
プロジェクトルート（amplify-vue-ts-appディレクトリ直下）にあるtsconfig.jsonを以下のように更新します。

```
{
  "files": [],
  "references": [
    { "path": "./tsconfig.app.json" },
    { "path": "./tsconfig.node.json" }
  ],
    "compilerOptions": {
    "allowJs": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@stores/*": ["src/stores/*"]
    }
  }
}
```

## vite.config.ts更新
プロジェクトルートのvite.config.tsを以下のコードに更新します。
```
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [
    vue({
      template: {
        compilerOptions: {
          isCustomElement: (tag) => tag.startsWith('ui5-')
        }
      }
    })
  ],
  resolve: {
    alias: {
      '@': '/src'
    }
  }
})
```

## ルーター、レイアウトファイル作成
![router](../images/screenshots/d3-create-file.png)

ファイル作成後、各ファイルに以下のコードを貼り付けてください。
```
<!-- layout/MainLayout.vue -->
<template>
    <!-- Authenticator: AWS Amplify UI コンポーネント - 自動でログイン画面とメイン画面を切り替え -->
    <authenticator>
        <template v-slot="{signOut}">
            <div>ログインが完了しました</div>
            <!-- signOut: ログアウト処理を実行する関数 -->
            <button @click="signOut">ログアウト</button>
        </template>
    </authenticator>
</template>

<script setup>
import { Authenticator } from "@aws-amplify/ui-vue";
import "@aws-amplify/ui-vue/styles.css";
</script>
```

```
// router/index.ts
import { createRouter, createWebHashHistory, type RouteRecordRaw } from 'vue-router'

const routes: RouteRecordRaw[] = [
    {
        path: '/', // ルートパス：ホームページ
        name: 'Home', // ルート名
        component: () => import("@/layout/MainLayout.vue"), 
    }
    // 他のルートをここに追加できます
];

const router = createRouter({
    history: createWebHashHistory(), // ハッシュモードの履歴管理
    routes, // ルート定義を設定
});

export default router; // ルーターをエクスポート
```


## main.tsファイル更新
```
import { createApp } from 'vue'
// TODO: style.cssをインポート

import App from './App.vue'
import router from "./router/index.ts"; 
import { Amplify } from "aws-amplify";
import { parseAmplifyConfig } from "aws-amplify/utils";
import outputs from "../amplify_outputs.json";

// Amplify設定: AWSサービスとの接続情報を設定
const amplifyConfig = parseAmplifyConfig(outputs);
Amplify.configure({
  Auth: amplifyConfig.Auth, // Cognito認証設定のみ適用
});

// Vueアプリケーション作成
const app = createApp(App);

// グローバルプロパティ: アプリ全体でCognitoクライアントIDにアクセス可能
app.config.globalProperties.$clientId = amplifyConfig.Auth?.Cognito.userPoolClientId;

// ルーター設定: ページ遷移機能を追加
app.use(router);

// DOM要素にマウント: アプリケーションを画面に表示
app.mount('#app');
```

### ハンズオン 1: Style.cssインポート
style.cssをインポートしましょう。
#### ヒント
- `import`を使用
- `./style.css`を指定

<details>
<summary>解答例</summary>
```
import { createApp } from 'vue'
import './style.css' // 追加
import App from './App.vue'
```
</details>
&nbsp;


## App.vueファイル更新
```
<template>
  <!-- router-view: 現在のURLに対応するコンポーネントを表示するエリア -->
  <!-- "/" → MainLayout.vueが表示される -->
  <router-view />
</template>

<script setup lang="ts">
// 現在は空ですが、アプリ全体で共通の処理がある場合はここに記述
// 例: グローバルな状態管理、エラーハンドリング、テーマ設定など
</script>
```

## フロントエンドの変更とサインアップの確認
以下のように、フロントエンドがログインページに変更されていること、およびサインアップが可能なことを確認します。  

![login](../images/screenshots/d3-sign-up.png)

![mail](../images/screenshots/d3-mail-check.png)

![logged-in](../images/screenshots/d3-logged-in.png)

## ホームページのカスタマイズ
プログラミング初心者の方向けに、基本的なHTMLエレメントとCSSスタイルを使用してホームページをカスタマイズしてみましょう。

### HTMLとCSSの基礎知識

#### HTMLエレメントとは
HTMLエレメントは、Webページの構造と内容を定義するためのタグです。以下が基本的なHTMLエレメントです：

```
<!-- 見出し（タイトル）- 文字の大きさが異なります -->
<h1>最も大きな見出し</h1>
<h2>中ぐらいの見出し</h2>
<h3>小さめの見出し</h3>

<!-- 段落（文章）- 通常のテキストを表示します -->
<p>これは段落です。通常の文章を書く時に使います。</p>

<!-- ボタン - クリックできる要素です -->
<button>クリックできるボタン</button>

<!-- フォーム - ユーザーから情報を入力してもらう仕組み -->
<form>
    <input type="text" placeholder="テキストを入力">
    <input type="email" placeholder="メールアドレス">
    <textarea placeholder="長い文章を入力"></textarea>
</form>

<!-- リスト - 項目を箇条書きで表示します -->
<ul>
    <li>項目1</li>
    <li>項目2</li>
    <li>項目3</li>
</ul>
```

#### CSSスタイルとは
CSS（Cascading Style Sheets）は、HTMLエレメントの見た目（色、サイズ、配置など）を設定するための言語です。

**インラインスタイル**という方法では、HTMLエレメントに直接`style`属性を追加してスタイルを適用できます：

```html
<!-- 基本的なCSSプロパティの例 -->
<h1 style="color: blue;">青色のタイトル</h1>
<p style="font-size: 16px;">16ピクセルのサイズの文字</p>
<button style="background: red; color: white;">赤い背景のボタン</button>
<div style="padding: 20px; margin: 10px;">内側と外側に余白のあるボックス</div>
```

#### よく使用されるCSSプロパティ
- **color**: 文字の色を変更します（例: `color: red`）
- **background**: 背景色を変更します（例: `background: blue`）
- **font-size**: 文字の大きさを変更します（例: `font-size: 20px`）
- **padding**: 要素の内側の余白を設定します（例: `padding: 10px`）
- **margin**: 要素の外側の余白を設定します（例: `margin: 15px`）
- **border**: 要素の境界線を設定します（例: `border: 1px solid black`）
- **text-align**: 文字の配置を設定します（例: `text-align: center`）

### 実践的なカスタマイズ例
`layout/MainLayout.vue`ファイルを以下のように更新して、HTMLエレメントとCSSスタイルを試してみましょう：

```
<template>
    <authenticator>
        <template v-slot="{signOut}">
            <!-- メインコンテナ: 全体に余白とフォントを設定 -->
            <div style="padding: 20px; font-family: Arial, sans-serif;">

                <!-- メインタイトル: 青色で中央揃え -->
                <h1 style="color: #2c3e50; text-align: center;">ホームページへようこそ</h1>

                <!-- サブタイトル: グレー色 -->
                <h2 style="color: #34495e;">あなたのWebアプリケーション</h2>

                <!-- 説明文: 文字サイズと行間を調整 -->
                <p style="font-size: 16px; line-height: 1.6;">
                    これはあなたが作成したWebアプリケーションのホームページです。
                    認証機能が正常に動作していることを確認できます。
                </p>

                <!-- 機能紹介セクション -->
                <h3 style="color: #27ae60; margin-top: 30px;">主な機能</h3>
                <ul style="font-size: 14px;">
                    <li>ユーザー認証（サインアップ・ログイン）</li>
                    <li>セキュアなセッション管理</li>
                    <li>レスポンシブデザイン対応</li>
                </ul>

                <!-- フォーム例: 背景色と余白を設定 -->
                <h3 style="color: #e74c3c; margin-top: 30px;">お問い合わせフォーム</h3>
                <form style="background: #f8f9fa; padding: 15px; border-radius: 5px; margin: 15px 0;">

                    <!-- 名前入力欄のグループ -->
                    <div style="margin-bottom: 10px;">
                        <label style="display: block; margin-bottom: 5px; font-weight: bold;">お名前:</label>
                        <input type="text" placeholder="山田太郎"
                               style="width: 100%; padding: 8px; border: 1px solid #ddd; border-radius: 4px;">
                    </div>

                    <!-- メール入力欄のグループ -->
                    <div style="margin-bottom: 10px;">
                        <label style="display: block; margin-bottom: 5px; font-weight: bold;">メールアドレス:</label>
                        <input type="email" placeholder="taro@example.com"
                               style="width: 100%; padding: 8px; border: 1px solid #ddd; border-radius: 4px;">
                    </div>

                    <!-- メッセージ入力欄のグループ -->
                    <div style="margin-bottom: 15px;">
                        <label style="display: block; margin-bottom: 5px; font-weight: bold;">メッセージ:</label>
                        <textarea placeholder="こちらにメッセージをご記入ください..." rows="4"
                                  style="width: 100%; padding: 8px; border: 1px solid #ddd; border-radius: 4px; resize: vertical;"></textarea>
                    </div>

                    <!-- 送信ボタン: 青色の背景と白い文字 -->
                    <button type="submit"
                            style="background: #3498db; color: white; padding: 10px 20px; border: none; border-radius: 4px; cursor: pointer;">
                        送信する
                    </button>
                </form>

                <!-- アクションボタンのセクション: 中央揃え -->
                <div style="text-align: center; margin: 30px 0;">
                    <button style="background: #2ecc71; color: white; padding: 12px 24px; border: none; border-radius: 5px; margin: 5px; cursor: pointer;">
                        新しい機能を試す
                    </button>
                    <button style="background: #f39c12; color: white; padding: 12px 24px; border: none; border-radius: 5px; margin: 5px; cursor: pointer;">
                        ダッシュボード
                    </button>
                    <!-- ログアウトボタン: 赤色で目立たせる -->
                    <button @click="signOut"
                            style="background: #e74c3c; color: white; padding: 12px 24px; border: none; border-radius: 5px; margin: 5px; cursor: pointer;">
                        ログアウト
                    </button>
                </div>

                <!-- フッター: 小さな文字で中央揃え -->
                <footer style="margin-top: 40px; text-align: center; color: #7f8c8d; font-size: 12px;">
                    <p>&copy; 2024 あなたのWebアプリケーション. All rights reserved.</p>
                </footer>
            </div>
        </template>
    </authenticator>
</template>
```

### 学習ポイント

#### HTMLエレメント
- **h1, h2, h3**: 見出しの階層を作る時に使用します（h1が最も重要、h3は最も小さい見出し）
- **p**: 段落や説明文を書く時に使用します
- **button**: ユーザーがクリックできるボタンを作成します
- **form**: ユーザーから情報を入力してもらうためのフォームを作成します
- **input**: 一行のテキスト入力欄を作成します（type属性で種類を指定）
- **textarea**: 複数行のテキストを入力できる欄を作成します
- **ul/li**: 箇条書きリスト（ul = unordered list、li = list item）
- **div**: コンテンツをグループ化するためのコンテナです
- **label**: フォーム入力欄の説明ラベルです

#### CSSスタイル
- **color**: 文字の色を指定します
- **background**: 背景色を指定します
- **font-size**: 文字の大きさを指定します
- **padding**: 要素の内側の余白を指定します
- **margin**: 要素の外側の余白を指定します
- **border**: 要素の境界線を指定します
- **text-align**: 文字の配置（left、center、right）を指定します
- **border-radius**: 角を丸くします

### 参考資料
- [HTML入門 - MDN](https://developer.mozilla.org/ja/docs/Learn/Getting_started_with_the_web/HTML_basics)