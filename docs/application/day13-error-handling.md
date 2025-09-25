# Day13 エラーハンドリング

## 目標
- エラーハンドリングの重要性を理解する
- Amplifyのエラーハンドリングのベストプラクティスを学ぶ
- 実際のコードにエラーハンドリングを実装

## エラーハンドリングとは？

エラーハンドリングとは、プログラムで発生するエラーを適切に処理することです。
初心者にとって重要なポイントを理解しましょう。

### なぜエラーハンドリングが必要？

1. **ユーザー体験の向上**
   - エラーが起きても画面が真っ白にならない
   - 分かりやすいメッセージでユーザーに状況を伝える

2. **アプリケーションの安定性**
   - 1つのエラーでアプリ全体が止まることを防ぐ
   - 予期しない動作を減らす

## エラーハンドリングの基本パターン

### 1. try-catch文の使い方

```javascript
// 基本的な形
try {
  // エラーが起きる可能性のあるコード
  const result = await someAsyncFunction();
} catch (error) {
  // エラーが起きた時の処理
  console.error('エラーが発生しました:', error);
}
```

### 2. Reactでのエラーハンドリング

```javascript
const MyComponent = () => {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const handleSubmit = async () => {
    try {
      setLoading(true);
      setError(null); // 前回のエラーをクリア

      const result = await apiCall();
      // 成功時の処理

    } catch (err) {
      setError('操作に失敗しました。もう一度お試しください。');
    } finally {
      setLoading(false); // 必ず実行される
    }
  };

  if (error) {
    return <div style={{color: 'red'}}>{error}</div>;
  }

  return (
    // 通常の表示
  );
};
```

## Amplifyでのよくあるエラーとその対処法

### 1. 認証エラー

```javascript
import { signIn } from 'aws-amplify/auth';

const handleSignIn = async (username, password) => {
  try {
    await signIn({ username, password });
  } catch (error) {
    switch (error.name) {
      case 'UserNotFoundException':
        setError('ユーザーが見つかりません');
        break;
      case 'NotAuthorizedException':
        setError('パスワードが間違っています');
        break;
      default:
        setError('ログインに失敗しました');
    }
  }
};
```

### 2. データ取得エラー

```javascript
import { generateClient } from 'aws-amplify/api';

const client = generateClient();

const fetchTodos = async () => {
  try {
    const result = await client.graphql({
      query: listTodos
    });
    setTodos(result.data.listTodos.items);
  } catch (error) {
    console.error('データ取得エラー:', error);
    setError('データの読み込みに失敗しました');
  }
};
```

## エラーハンドリングのベストプラクティス

### 1. 適切なエラーメッセージ

```javascript
// ❌ 悪い例
catch (error) {
  setError(error.message); // 技術的すぎるメッセージ
}

// ✅ 良い例
catch (error) {
  console.error('技術的な詳細:', error); // 開発者用
  setError('保存に失敗しました。もう一度お試しください。'); // ユーザー用
}
```

### 2. ローディング状態の管理

```javascript
const [isLoading, setIsLoading] = useState(false);

const handleAction = async () => {
  setIsLoading(true);
  try {
    await someAsyncOperation();
  } catch (error) {
    handleError(error);
  } finally {
    setIsLoading(false); // 必ず実行される
  }
};
```

### 3. エラーの分類

```javascript
const handleError = (error) => {
  if (error.name === 'NetworkError') {
    setError('インターネット接続を確認してください');
  } else if (error.name === 'ValidationError') {
    setError('入力内容を確認してください');
  } else {
    setError('予期しないエラーが発生しました');
  }
};
```

## 初心者向けのチェックリスト

- [ ] エラーが起きても画面が真っ白にならない
- [ ] ユーザーに分かりやすいメッセージを表示する
- [ ] ローディング状態を適切に管理する
- [ ] コンソールに技術的な詳細を記録する
- [ ] エラー後も操作を続けられるようにする

## まとめ

エラーハンドリングは以下の3つのステップで考えましょう：

1. **予測する** - どこでエラーが起きるか考える
2. **キャッチする** - try-catchでエラーを捕まえる
3. **対処する** - ユーザーに適切に伝える

最初は完璧を目指さず、基本的なtry-catchから始めて、徐々に詳細な処理を追加していきましょう。

## 実践的なエラーハンドリング

ここでは、Day11・Day12で作成したコードを例に、実際のアプリケーションでよくあるエラーハンドリングパターンを学びます。

### 1. バリデーション（入力検証）エラー

**なぜ必要？**
ユーザーが間違った値を入力した時に、事前にエラーを見つけて分かりやすく教えることができます。

**基本的なパターン：**
```javascript
const validateForm = () => {
  const errors = [];

  if (!customerCode) errors.push("顧客コードは必須です");
  if (quantity <= 0) errors.push("数量は1以上にしてください");

  return errors;
};
```

**ポイント：**
- エラーは配列に集めて一度に表示
- 分かりやすい日本語メッセージを使用
- 数値は`parseInt()`や`isNaN()`で検証

### 2. ファイル処理のエラー

**よくある問題：**
- ファイル形式が間違っている（.csvでない）
- ファイルサイズが大きすぎる
- アップロード中の通信エラー

**対処方法：**
```javascript
// アップロード前の検証
if (!file.name.endsWith('.csv')) {
  showError('CSVファイルを選択してください');
  return; // ここで処理を止める
}

try {
  await uploadFile();
} catch (error) {
  // エラーの内容に応じて分かりやすいメッセージに変換
  if (error.name === 'NetworkError') {
    showError('ネットワーク接続を確認してください');
  } else {
    showError('アップロードに失敗しました');
  }
}
```

**ポイント：**
- 事前検証で防げるエラーは早めに防ぐ
- ネットワークエラーなど、エラーの種類別に対応
- 技術的なエラーメッセージをユーザー向けに翻訳

### 3. データベース操作のエラー

**よくある問題：**
- データが見つからない
- 権限がない
- 数値の変換エラー

**対処方法：**
```javascript
const updateOrder = async (orderId, orderData) => {
  // 1. 入力チェック（事前に防ぐ）
  if (!orderId) {
    return { success: false, message: "注文IDが無効です" };
  }

  try {
    const result = await client.mutations.updateOrder(orderData);
    return { success: true, message: "更新しました" };
  } catch (error) {
    // 2. エラー内容を分析して適切なメッセージを返す
    if (error.message.includes('Not Found')) {
      return { success: false, message: "注文が見つかりません" };
    }
    return { success: false, message: "更新に失敗しました" };
  }
};
```

**ポイント：**
- 戻り値で成功・失敗を明確にする
- コンソールには技術詳細、ユーザーには分かりやすいメッセージ
- 部分的な失敗も考慮（10件中8件成功など）

### 4. 非同期処理の進捗とエラー

**問題：**
時間のかかる処理（CSVアップロードなど）で、途中でエラーが起きた時の対応

**対処方法：**
```javascript
// 状態を細かく管理
switch (job.status) {
  case 'PROCESSING':
    showStatus(`処理中: ${progress}%`);
    break;
  case 'COMPLETED':
    showStatus('完了しました');
    break;
  case 'FAILED':
    showError(`エラー: ${job.errorMessage}`);
    stopPolling(); // ポーリングを停止
    break;
}
```

**ポイント：**
- 処理状況をリアルタイムで表示
- エラー時は無限ループを防ぐため処理を停止
- ユーザーが処理状況を把握できるようにする

### 5. 一括処理のエラー

**問題：**
複数のデータを処理する時、一部が失敗した場合の対応

**考え方：**
```javascript
// 悪い例：1つ失敗したら全部止める
// 良い例：成功したものは処理を続け、結果をまとめて報告

let successCount = 0;
let failCount = 0;

for (const item of items) {
  try {
    await processItem(item);
    successCount++;
  } catch (error) {
    failCount++;
    // ログは残すが処理は続行
    console.error(`Item ${item.id} failed:`, error);
  }
}

// 結果をまとめて表示
showResult(`${successCount}件成功、${failCount}件失敗`);
```

**ポイント：**
- 部分的な成功も価値がある
- 詳細なエラー情報はコンソールに記録
- ユーザーには処理結果のサマリーを表示

## エラーハンドリングの考え方

### レベル1：エラーを隠さない
```javascript
// ❌ 悪い例
try {
  await riskyOperation();
} catch (error) {
  // エラーを無視（最悪）
}

// ✅ 良い例
try {
  await riskyOperation();
} catch (error) {
  console.error('詳細:', error);
  showError('操作に失敗しました');
}
```

### レベル2：エラーの種類を区別
```javascript
if (error.name === 'NetworkError') {
  showError('通信エラーです');
} else if (error.message.includes('404')) {
  showError('データが見つかりません');
} else {
  showError('予期しないエラーです');
}
```

### レベル3：ユーザーが次にできることを示す
```javascript
if (error.name === 'NetworkError') {
  showError('通信エラーです。接続を確認してから再試行してください');
  showRetryButton();
}
```

