# ベストプラクティス

このガイドでは、AWS Amplify開発におけるコード品質、セキュリティ、パフォーマンス、および保守性に重点を置いた業界のベストプラクティスを概説します。

## プロジェクト構造と組織化

### フロントエンド構造

```
src/
├── components/           # 再利用可能なUIコンポーネント
│   ├── common/          # 汎用コンポーネント（Button、Modalなど）
│   ├── forms/           # フォーム専用コンポーネント
│   └── layout/          # レイアウトコンポーネント（Header、Sidebar）
├── views/               # ページレベルコンポーネント
├── stores/              # Pinia状態管理
├── composables/         # Vue 3 Composition関数
├── services/            # APIとビジネスロジック
├── types/               # TypeScript型定義
├── utils/               # ユーティリティ関数
├── assets/              # 静的アセット
└── router/              # Vue Routerの設定
```

### バックエンド構造

```
amplify/
├── auth/                # 認証リソース
├── data/                # GraphQLスキーマとリソルバー
├── functions/           # Lambda関数
├── storage/             # ファイルストレージ設定
├── custom/              # カスタムCloudFormationリソース
└── backend.ts           # メインバックエンド設定
```

!!! tip "組織化の原則"
    - **機能ベースのグループ化**: 関連する機能をまとめる
    - **関心の明確な分離**: UI、ビジネスロジック、データを分離する
    - **一貫した命名規則**: ファイルにはkebab-case、コンポーネントにはPascalCaseを使用
    - **論理的階層**: 一般的なものから具体的なものへと整理

## TypeScriptベストプラクティス

### 型定義

```typescript
// 包括的なインターフェースを定義
interface User {
  readonly id: string
  email: string
  name: string
  role: 'admin' | 'user' | 'viewer'
  createdAt: Date
  preferences?: UserPreferences
}

interface UserPreferences {
  theme: 'light' | 'dark'
  language: string
  notifications: {
    email: boolean
    push: boolean
  }
}

// 柔軟性のためにユーティリティ型を使用
type CreateUserInput = Omit<User, 'id' | 'createdAt'>
type UpdateUserInput = Partial<Pick<User, 'name' | 'preferences'>>
```

### APIレスポンス型

```typescript
// GraphQLスキーマから型を生成
import type { Schema } from '../amplify/data/resource'

type TodoType = Schema['Todo']['type']
type CreateTodoInput = Schema['Todo']['createType']
type UpdateTodoInput = Schema['Todo']['updateType']

// APIレスポンス用のラッパー型を作成
interface ApiResponse<T> {
  data: T
  errors?: Array<{
    message: string
    code: string
  }>
}

// 状態管理に判別可能ユニオンを使用
type LoadingState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string }
```

### コンポーネントProps

```vue
<script setup lang="ts">
// 再利用可能なコンポーネントにジェネリックインターフェースを使用
interface TableColumn<T> {
  key: keyof T
  label: string
  sortable?: boolean
  formatter?: (value: T[keyof T]) => string
}

interface Props<T = any> {
  data: T[]
  columns: TableColumn<T>[]
  loading?: boolean
  onRowClick?: (row: T) => void
}

// デフォルトと制約を提供
const props = withDefaults(defineProps<Props>(), {
  loading: false
})

// 派生状態に算出プロパティを使用
const sortedData = computed(() => {
  // ソートロジックをここに
  return props.data
})
</script>
```

## Vue.js 3ベストプラクティス

### Composition APIパターン

```vue
<script setup lang="ts">
// 再利用可能なロジックにはcomposablesを使用
import { useAuthenticatedUser } from '@/composables/useAuth'
import { usePagination } from '@/composables/usePagination'
import { useAsyncData } from '@/composables/useAsyncData'

// 必要なものだけを分割代入
const { user, isAuthenticated } = useAuthenticatedUser()
const { currentPage, pageSize, totalPages } = usePagination()

// リアクティブ参照を適切に使用
const searchTerm = ref('')
const isLoading = ref(false)
const todos = ref<TodoType[]>([])

// 可能な限りwatcherよりcomputedを優先
const filteredTodos = computed(() =>
  todos.value.filter(todo =>
    todo.content.toLowerCase().includes(searchTerm.value.toLowerCase())
  )
)

// 副作用にはwatcherを使用
watch(searchTerm, async (newTerm) => {
  if (newTerm.length > 2) {
    await fetchTodos(newTerm)
  }
}, { debounce: 300 })

// onUnmountedでクリーンアップ
onUnmounted(() => {
  // サブスクリプションのキャンセル、タイマーのクリアなど
})
</script>
```

### Piniaを使った状態管理

```typescript
// stores/todos.ts
export const useTodosStore = defineStore('todos', () => {
  // 状態
  const todos = ref<TodoType[]>([])
  const loading = ref(false)
  const error = ref<string | null>(null)

  // ゲッター (computed)
  const completedTodos = computed(() =>
    todos.value.filter(todo => todo.completed)
  )

  const todoCount = computed(() => todos.value.length)

  // アクション
  const fetchTodos = async (): Promise<void> => {
    loading.value = true
    error.value = null

    try {
      const response = await client.models.Todo.list()
      todos.value = response.data
    } catch (err) {
      error.value = err instanceof Error ? err.message : '不明なエラー'
    } finally {
      loading.value = false
    }
  }

  const addTodo = async (content: string): Promise<void> => {
    try {
      const response = await client.models.Todo.create({ content })
      if (response.data) {
        todos.value.push(response.data)
      }
    } catch (err) {
      error.value = err instanceof Error ? err.message : 'Todoの作成に失敗しました'
      throw err
    }
  }

  return {
    // 状態
    todos: readonly(todos),
    loading: readonly(loading),
    error: readonly(error),
    
    // ゲッター
    completedTodos,
    todoCount,
    
    // アクション
    fetchTodos,
    addTodo
  }
})
```

## Amplifyバックエンドベストプラクティス

### GraphQLスキーマ設計

```typescript
// amplify/data/resource.ts
import { type ClientSchema, a, defineData } from '@aws-amplify/backend'

const schema = a.schema({
  // 説明的なモデル名を使用
  BlogPost: a.model({
    title: a.string().required(),
    content: a.string().required(),
    status: a.enum(['DRAFT', 'PUBLISHED', 'ARCHIVED']),
    publishedAt: a.datetime(),
    authorId: a.id().required(),
    categoryId: a.id(),
    tags: a.string().array(),
    viewCount: a.integer().default(0),
    
    // 関係
    author: a.belongsTo('Author', 'authorId'),
    category: a.belongsTo('Category', 'categoryId'),
    comments: a.hasMany('Comment', 'postId'),
  })
  .authorization((allow) => [
    // 細かい許可設定
    allow.owner().to(['create', 'update', 'delete']),
    allow.authenticated().to(['read']),
    allow.groups(['admin']).to(['create', 'update', 'delete']),
  ])
  .secondaryIndexes((index) => [
    index('authorId').sortKeys(['publishedAt']),
    index('categoryId'),
    index('status').sortKeys(['publishedAt']),
  ]),

  Author: a.model({
    name: a.string().required(),
    email: a.email().required(),
    bio: a.string(),
    avatarUrl: a.url(),
    
    // 関係
    posts: a.hasMany('BlogPost', 'authorId'),
  })
  .authorization((allow) => [
    allow.owner(),
    allow.authenticated().to(['read']),
  ]),

  Category: a.model({
    name: a.string().required(),
    description: a.string(),
    
    // 関係
    posts: a.hasMany('BlogPost', 'categoryId'),
  })
  .authorization((allow) => [
    allow.authenticated().to(['read']),
    allow.groups(['admin']).to(['create', 'update', 'delete']),
  ]),

  Comment: a.model({
    content: a.string().required(),
    postId: a.id().required(),
    authorId: a.id().required(),
    
    // 関係
    post: a.belongsTo('BlogPost', 'postId'),
    author: a.belongsTo('Author', 'authorId'),
  })
  .authorization((allow) => [
    allow.owner().to(['create', 'update', 'delete']),
    allow.authenticated().to(['read']),
  ]),
})
```

### 認証設定

```typescript
// amplify/auth/resource.ts
import { defineAuth } from '@aws-amplify/backend'

export const auth = defineAuth({
  loginWith: {
    email: {
      verificationEmailStyle: 'CODE',
      verificationEmailSubject: 'Welcome to MyApp!',
    },
    // Optional: Social providers
    externalProviders: {
      google: {
        clientId: process.env.GOOGLE_CLIENT_ID!,
        clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
      },
      loginWithAmazon: {
        clientId: process.env.AMAZON_CLIENT_ID!,
        clientSecret: process.env.AMAZON_CLIENT_SECRET!,
      },
      signInWithApple: {
        clientId: process.env.APPLE_CLIENT_ID!,
        keyId: process.env.APPLE_KEY_ID!,
        privateKey: process.env.APPLE_PRIVATE_KEY!,
        teamId: process.env.APPLE_TEAM_ID!,
      },
      callbackUrls: ['http://localhost:3000/', 'https://myapp.com/'],
      logoutUrls: ['http://localhost:3000/', 'https://myapp.com/'],
    },
  },
  userAttributes: {
    email: {
      required: true,
      mutable: true,
    },
    givenName: {
      required: true,
      mutable: true,
    },
    familyName: {
      required: true,
      mutable: true,
    },
    phoneNumber: {
      required: false,
      mutable: true,
    },
  },
  passwordPolicy: {
    minLength: 12,
    requireLowercase: true,
    requireUppercase: true,
    requireNumbers: true,
    requireSymbols: true,
  },
})
```

## セキュリティベストプラクティス

### フロントエンドセキュリティ

```typescript
// 環境変数の検証
const config = {
  apiUrl: import.meta.env.VITE_API_URL,
  region: import.meta.env.VITE_AWS_REGION,
  // フロントエンドで機密キーを公開しない
}

// 必要な環境変数を検証
Object.entries(config).forEach(([key, value]) => {
  if (!value) {
    throw new Error(`必要な環境変数が不足しています: ${key}`)
  }
})

// 入力検証とサニタイゼーション
import { z } from 'zod'

const CreateTodoSchema = z.object({
  content: z.string()
    .min(1, 'コンテンツが必要です')
    .max(500, 'コンテンツが長すぎます')
    .trim(),
  priority: z.enum(['low', 'medium', 'high']).default('medium'),
  dueDate: z.date().optional(),
})

const validateTodoInput = (input: unknown) => {
  try {
    return CreateTodoSchema.parse(input)
  } catch (error) {
    throw new Error(`無効な入力: ${error.message}`)
  }
}
```

### バックエンドセキュリティ

```typescript
// 適切なエラーハンドリングを持つLambda関数
export const handler: AppSyncResolverHandler<any, any> = async (event) => {
  console.log('Event: ', JSON.stringify(event, null, 2))

  try {
    // ユーザーが認証されていることを検証
    if (!event.identity?.sub) {
      throw new Error('許可されていません')
    }

    // 入力検証
    const { content } = event.arguments.input
    if (!content || content.trim().length === 0) {
      throw new Error('コンテンツが必要です')
    }

    // ビジネスロジックをここに
    const result = await processRequest(event.arguments.input, event.identity.sub)

    return {
      statusCode: 200,
      body: JSON.stringify(result),
    }
  } catch (error) {
    console.error('Error:', error)
    
    // 内部エラーの詳細を公開しない
    const message = error instanceof Error 
      ? error.message 
      : '内部サーバーエラー'
    
    throw new Error(message)
  }
}
```


## パフォーマンスベストプラクティス

### フロントエンドパフォーマンス

```vue
<script setup lang="ts">
// 重いコンポーネントに遅延読み込みを使用
const HeavyChart = defineAsyncComponent({
  loader: () => import('@/components/HeavyChart.vue'),
  loadingComponent: LoadingSpinner,
  errorComponent: ErrorDisplay,
  delay: 200,
  timeout: 3000,
})

// 大きなリストに仮想スクロールを実装
import { RecycleScroller } from 'vue-virtual-scroller'

// すべてのデータを読み込む代わりにページネーションを使用
const { data: todos, loading, error } = usePagination({
  queryFn: (page, limit) => client.models.Todo.list({
    limit,
    nextToken: page > 1 ? getNextToken(page) : undefined
  }),
  pageSize: 20
})

// 検索入力のデバウンス
import { debounce } from 'lodash-es'

const debouncedSearch = debounce(async (term: string) => {
  await searchTodos(term)
}, 300)

// バンドルサイズの最適化
import { defineAsyncComponent } from 'vue'
// 次の代わりに: import SomeHeavyComponent from './SomeHeavyComponent.vue'
const SomeHeavyComponent = defineAsyncComponent(() => 
  import('./SomeHeavyComponent.vue')
)
</script>
```

### バックエンドパフォーマンス

```typescript
// Use batch operations
const batchCreateTodos = async (todos: CreateTodoInput[]) => {
  // Process in batches to avoid timeouts
  const BATCH_SIZE = 25
  const results = []

  for (let i = 0; i < todos.length; i += BATCH_SIZE) {
    const batch = todos.slice(i, i + BATCH_SIZE)
    const batchPromises = batch.map(todo => 
      client.models.Todo.create(todo)
    )
    
    const batchResults = await Promise.allSettled(batchPromises)
    results.push(...batchResults)
  }

  return results
}

// Implement caching
import { LRUCache } from 'lru-cache'

const cache = new LRUCache<string, any>({
  max: 500,
  maxAge: 1000 * 60 * 5, // 5 minutes
})

const getCachedTodos = async (userId: string) => {
  const cacheKey = `todos:${userId}`
  
  let todos = cache.get(cacheKey)
  if (!todos) {
    todos = await fetchTodosFromDB(userId)
    cache.set(cacheKey, todos)
  }
  
  return todos
}
```

### GraphQLクエリ最適化

```typescript
// Use specific field selection
const TODO_FIELDS = /* GraphQL */ `
  id
  content
  completed
  createdAt
  updatedAt
`

const LIST_TODOS = /* GraphQL */ `
  query ListTodos($limit: Int, $nextToken: String) {
    listTodos(limit: $limit, nextToken: $nextToken) {
      items {
        ${TODO_FIELDS}
      }
      nextToken
    }
  }
`

// Implement proper pagination
const fetchTodosPage = async (limit = 20, nextToken?: string) => {
  return await client.graphql({
    query: LIST_TODOS,
    variables: { limit, nextToken }
  })
}
```

## テストベストプラクティス

### 単体テスト

```typescript
// tests/components/TodoItem.test.ts
import { describe, it, expect, vi } from 'vitest'
import { mount } from '@vue/test-utils'
import TodoItem from '@/components/TodoItem.vue'

describe('TodoItem', () => {
  const mockTodo = {
    id: '1',
    content: 'Test todo',
    completed: false,
    createdAt: new Date().toISOString(),
  }

  it('renders todo content correctly', () => {
    const wrapper = mount(TodoItem, {
      props: { todo: mockTodo }
    })

    expect(wrapper.text()).toContain('Test todo')
    expect(wrapper.find('[data-testid="todo-content"]').text()).toBe('Test todo')
  })

  it('emits toggle event when checkbox is clicked', async () => {
    const wrapper = mount(TodoItem, {
      props: { todo: mockTodo }
    })

    const checkbox = wrapper.find('input[type="checkbox"]')
    await checkbox.trigger('change')

    expect(wrapper.emitted('toggle')).toBeTruthy()
    expect(wrapper.emitted('toggle')?.[0]).toEqual([mockTodo.id])
  })
})
```

### 結合テスト

```typescript
// tests/stores/todos.test.ts
import { describe, it, expect, beforeEach, vi } from 'vitest'
import { setActivePinia, createPinia } from 'pinia'
import { useTodosStore } from '@/stores/todos'

// Mock Amplify client
vi.mock('@/services/amplify', () => ({
  client: {
    models: {
      Todo: {
        list: vi.fn(),
        create: vi.fn(),
        update: vi.fn(),
        delete: vi.fn(),
      }
    }
  }
}))

describe('Todos Store', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })

  it('fetches todos successfully', async () => {
    const mockTodos = [
      { id: '1', content: 'Test 1', completed: false },
      { id: '2', content: 'Test 2', completed: true },
    ]

    vi.mocked(client.models.Todo.list).mockResolvedValueOnce({
      data: mockTodos
    })

    const store = useTodosStore()
    await store.fetchTodos()

    expect(store.todos).toEqual(mockTodos)
    expect(store.loading).toBe(false)
    expect(store.error).toBe(null)
  })
})
```

## デプロイとDevOps

### 環境管理

```yaml
# .github/workflows/deploy.yml
name: Deploy to Amplify

on:
  push:
    branches: [main, staging, development]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Build application
        run: npm run build
        
      - name: Deploy to Amplify
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Deploy backend
        run: |
          npm install -g @aws-amplify/cli
          amplify push --yes
```

### 監視とログ

```typescript
// utils/logger.ts
export const logger = {
  info: (message: string, meta?: Record<string, any>) => {
    console.log(JSON.stringify({
      level: 'info',
      message,
      timestamp: new Date().toISOString(),
      ...meta
    }))
  },
  
  error: (message: string, error?: Error, meta?: Record<string, any>) => {
    console.error(JSON.stringify({
      level: 'error',
      message,
      error: error?.message,
      stack: error?.stack,
      timestamp: new Date().toISOString(),
      ...meta
    }))
  }
}

// Error boundary for Vue components
export const errorHandler = (error: Error, context: string) => {
  logger.error(`Error in ${context}`, error)
  
  // Send to monitoring service
  if (import.meta.env.PROD) {
    // Send to AWS CloudWatch, Sentry, etc.
  }
}
```

!!! success "重要なポイント"
    - **コードを書く前にアーキテクチャを計画する**
    - **より良い開発者体験とバグの減少のためにTypeScriptを使用する**
    - **すべてのレイヤーで適切なエラーハンドリングを実装する**
    - **初日からセキュリティベストプラクティスに従う**
    - **パフォーマンスを最適化するが、可読性を犠牲にしない**
    - **重要な機能にテストを書く**
    - **本番環境ではすべてを監視しログを記録する**
