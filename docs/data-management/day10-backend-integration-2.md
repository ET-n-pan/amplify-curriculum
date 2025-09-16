# Day 10: バックエンド連携② - CRUD続き

## ゴール
!!! success "Day 10 Goals"
    - Amplify DataにCRUD機能を追加

## ハンズオン 1: Amplify DataのCRUD機能の実装続き

### Step 1: `amplify/data/resource.ts`の修正続き
`/src/amplify/data/resource.ts`を修正し、`updateOrder`、`deleteOrder`関数を追加します。  
関数のレスポンスを処理するファイル`updateOrder.js`、`deleteOrder.js`を`amplify/data/`に作成します。

### Step 2: '/src/stores/form-store.ts'の修正続き
`/src/stores/form-store.ts`を修正し、Day9で作成したAmplify DataのCRUD関数を利用するように変更します。
```
// インポート追加
// Orders2Page.vueでのインポートを重複しないように削除
import type { Schema } from "@/amplify/data/resource";
import { generateClient } from "aws-amplify/data";

// Amplify Dataクライアント生成
const client = generateClient<Schema>();
```


### Step 2: Orders2Page.vueの修正続き
`/src/pages/Orders2Page.vue`を修正し、Day9で作成したAmplify DataのCRUD関数を利用するように変更します。