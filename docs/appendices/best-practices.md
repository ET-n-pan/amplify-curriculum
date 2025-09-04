# Best Practices

This guide outlines industry best practices for AWS Amplify development, focusing on code quality, security, performance, and maintainability.

## Project Structure and Organization

![Project Organization Best Practices Placeholder](../images/diagrams/project-organization-placeholder.png)

### Frontend Structure

```
src/
├── components/           # Reusable UI components
│   ├── common/          # Generic components (Button, Modal, etc.)
│   ├── forms/           # Form-specific components
│   └── layout/          # Layout components (Header, Sidebar)
├── views/               # Page-level components
├── stores/              # Pinia state management
├── composables/         # Vue 3 composition functions
├── services/            # API and business logic
├── types/               # TypeScript type definitions
├── utils/               # Utility functions
├── assets/              # Static assets
└── router/              # Vue Router configuration
```

### Backend Structure

```
amplify/
├── auth/                # Authentication resources
├── data/                # GraphQL schema and resolvers
├── functions/           # Lambda functions
├── storage/             # File storage configuration
├── custom/              # Custom CloudFormation resources
└── backend.ts           # Main backend configuration
```

!!! tip "Organization Principles"
    - **Feature-based grouping**: Group related functionality together
    - **Clear separation of concerns**: Keep UI, business logic, and data separate
    - **Consistent naming conventions**: Use kebab-case for files, PascalCase for components
    - **Logical hierarchy**: Organize from general to specific

## TypeScript Best Practices

![TypeScript Best Practices Placeholder](../images/screenshots/typescript-best-practices-placeholder.png)

### Type Definitions

```typescript
// Define comprehensive interfaces
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

// Use utility types for flexibility
type CreateUserInput = Omit<User, 'id' | 'createdAt'>
type UpdateUserInput = Partial<Pick<User, 'name' | 'preferences'>>
```

### API Response Types

```typescript
// Generate types from GraphQL schema
import type { Schema } from '../amplify/data/resource'

type TodoType = Schema['Todo']['type']
type CreateTodoInput = Schema['Todo']['createType']
type UpdateTodoInput = Schema['Todo']['updateType']

// Create wrapper types for API responses
interface ApiResponse<T> {
  data: T
  errors?: Array<{
    message: string
    code: string
  }>
}

// Use discriminated unions for state management
type LoadingState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string }
```

### Component Props

```vue
<script setup lang="ts">
// Use generic interfaces for reusable components
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

// Provide defaults and constraints
const props = withDefaults(defineProps<Props>(), {
  loading: false
})

// Use computed properties for derived state
const sortedData = computed(() => {
  // Sorting logic here
  return props.data
})
</script>
```

## Vue.js 3 Best Practices

![Vue 3 Best Practices Placeholder](../images/screenshots/vue3-best-practices-placeholder.png)

### Composition API Patterns

```vue
<script setup lang="ts">
// Use composables for reusable logic
import { useAuthenticatedUser } from '@/composables/useAuth'
import { usePagination } from '@/composables/usePagination'
import { useAsyncData } from '@/composables/useAsyncData'

// Destructure only what you need
const { user, isAuthenticated } = useAuthenticatedUser()
const { currentPage, pageSize, totalPages } = usePagination()

// Use reactive references appropriately
const searchTerm = ref('')
const isLoading = ref(false)
const todos = ref<TodoType[]>([])

// Prefer computed over watchers when possible
const filteredTodos = computed(() =>
  todos.value.filter(todo =>
    todo.content.toLowerCase().includes(searchTerm.value.toLowerCase())
  )
)

// Use watchers for side effects
watch(searchTerm, async (newTerm) => {
  if (newTerm.length > 2) {
    await fetchTodos(newTerm)
  }
}, { debounce: 300 })

// Cleanup in onUnmounted
onUnmounted(() => {
  // Cancel subscriptions, clear timers, etc.
})
</script>
```

### State Management with Pinia

```typescript
// stores/todos.ts
export const useTodosStore = defineStore('todos', () => {
  // State
  const todos = ref<TodoType[]>([])
  const loading = ref(false)
  const error = ref<string | null>(null)

  // Getters (computed)
  const completedTodos = computed(() =>
    todos.value.filter(todo => todo.completed)
  )

  const todoCount = computed(() => todos.value.length)

  // Actions
  const fetchTodos = async (): Promise<void> => {
    loading.value = true
    error.value = null

    try {
      const response = await client.models.Todo.list()
      todos.value = response.data
    } catch (err) {
      error.value = err instanceof Error ? err.message : 'Unknown error'
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
      error.value = err instanceof Error ? err.message : 'Failed to create todo'
      throw err
    }
  }

  return {
    // State
    todos: readonly(todos),
    loading: readonly(loading),
    error: readonly(error),
    
    // Getters
    completedTodos,
    todoCount,
    
    // Actions
    fetchTodos,
    addTodo
  }
})
```

## Amplify Backend Best Practices

![Backend Best Practices Placeholder](../images/diagrams/backend-best-practices-placeholder.png)

### GraphQL Schema Design

```typescript
// amplify/data/resource.ts
import { type ClientSchema, a, defineData } from '@aws-amplify/backend'

const schema = a.schema({
  // Use descriptive model names
  BlogPost: a.model({
    title: a.string().required(),
    content: a.string().required(),
    status: a.enum(['DRAFT', 'PUBLISHED', 'ARCHIVED']),
    publishedAt: a.datetime(),
    authorId: a.id().required(),
    categoryId: a.id(),
    tags: a.string().array(),
    viewCount: a.integer().default(0),
    
    // Relationships
    author: a.belongsTo('Author', 'authorId'),
    category: a.belongsTo('Category', 'categoryId'),
    comments: a.hasMany('Comment', 'postId'),
  })
  .authorization((allow) => [
    // Fine-grained authorization
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
    
    // Relationships
    posts: a.hasMany('BlogPost', 'authorId'),
  })
  .authorization((allow) => [
    allow.owner(),
    allow.authenticated().to(['read']),
  ]),

  Category: a.model({
    name: a.string().required(),
    description: a.string(),
    
    // Relationships
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
    
    // Relationships
    post: a.belongsTo('BlogPost', 'postId'),
    author: a.belongsTo('Author', 'authorId'),
  })
  .authorization((allow) => [
    allow.owner().to(['create', 'update', 'delete']),
    allow.authenticated().to(['read']),
  ]),
})
```

### Authentication Configuration

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

## Security Best Practices

![Security Best Practices Placeholder](../images/diagrams/security-best-practices-placeholder.png)

### Frontend Security

```typescript
// Environment variable validation
const config = {
  apiUrl: import.meta.env.VITE_API_URL,
  region: import.meta.env.VITE_AWS_REGION,
  // Never expose sensitive keys in frontend
}

// Validate required environment variables
Object.entries(config).forEach(([key, value]) => {
  if (!value) {
    throw new Error(`Missing required environment variable: ${key}`)
  }
})

// Input validation and sanitization
import { z } from 'zod'

const CreateTodoSchema = z.object({
  content: z.string()
    .min(1, 'Content is required')
    .max(500, 'Content too long')
    .trim(),
  priority: z.enum(['low', 'medium', 'high']).default('medium'),
  dueDate: z.date().optional(),
})

const validateTodoInput = (input: unknown) => {
  try {
    return CreateTodoSchema.parse(input)
  } catch (error) {
    throw new Error(`Invalid input: ${error.message}`)
  }
}
```

### Backend Security

```typescript
// Lambda function with proper error handling
export const handler: AppSyncResolverHandler<any, any> = async (event) => {
  console.log('Event: ', JSON.stringify(event, null, 2))

  try {
    // Validate user is authenticated
    if (!event.identity?.sub) {
      throw new Error('Unauthorized')
    }

    // Input validation
    const { content } = event.arguments.input
    if (!content || content.trim().length === 0) {
      throw new Error('Content is required')
    }

    // Business logic here
    const result = await processRequest(event.arguments.input, event.identity.sub)

    return {
      statusCode: 200,
      body: JSON.stringify(result),
    }
  } catch (error) {
    console.error('Error:', error)
    
    // Don't expose internal error details
    const message = error instanceof Error 
      ? error.message 
      : 'Internal server error'
    
    throw new Error(message)
  }
}
```

### Secrets Management

```typescript
// amplify/backend.ts
import { defineBackend } from '@aws-amplify/backend'
import { auth } from './auth/resource'
import { data } from './data/resource'

const backend = defineBackend({
  auth,
  data,
})

// Add secrets for external API integration
backend.addOutput({
  custom: {
    sapApiEndpoint: process.env.SAP_API_ENDPOINT,
    // Never commit actual secrets - use AWS Secrets Manager
  }
})
```

## Performance Best Practices

![Performance Best Practices Placeholder](../images/diagrams/performance-best-practices-placeholder.png)

### Frontend Performance

```vue
<script setup lang="ts">
// Use lazy loading for heavy components
const HeavyChart = defineAsyncComponent({
  loader: () => import('@/components/HeavyChart.vue'),
  loadingComponent: LoadingSpinner,
  errorComponent: ErrorDisplay,
  delay: 200,
  timeout: 3000,
})

// Implement virtual scrolling for large lists
import { RecycleScroller } from 'vue-virtual-scroller'

// Use pagination instead of loading all data
const { data: todos, loading, error } = usePagination({
  queryFn: (page, limit) => client.models.Todo.list({
    limit,
    nextToken: page > 1 ? getNextToken(page) : undefined
  }),
  pageSize: 20
})

// Debounce search inputs
import { debounce } from 'lodash-es'

const debouncedSearch = debounce(async (term: string) => {
  await searchTodos(term)
}, 300)

// Optimize bundle size
import { defineAsyncComponent } from 'vue'
// Instead of: import SomeHeavyComponent from './SomeHeavyComponent.vue'
const SomeHeavyComponent = defineAsyncComponent(() => 
  import('./SomeHeavyComponent.vue')
)
</script>
```

### Backend Performance

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

### GraphQL Query Optimization

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

## Testing Best Practices

![Testing Best Practices Placeholder](../images/screenshots/testing-best-practices-placeholder.png)

### Unit Testing

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

### Integration Testing

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

## Deployment and DevOps

![Deployment Best Practices Placeholder](../images/diagrams/deployment-best-practices-placeholder.png)

### Environment Management

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

### Monitoring and Logging

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

!!! success "Key Takeaways"
    - **Plan your architecture** before writing code
    - **Use TypeScript** for better developer experience and fewer bugs
    - **Implement proper error handling** at every layer
    - **Follow security best practices** from day one
    - **Optimize for performance** but don't sacrifice readability
    - **Write tests** for critical functionality
    - **Monitor and log** everything in production
