# Day 2: Amplify Gen2 Basics

---

## Learning Objectives

By the end of this day, you will be able to:

- Understand AWS Amplify Gen2 architecture and benefits
- Initialize Amplify Gen2 in your Vue.js project
- Navigate the Amplify CLI and Sandbox environment
- Configure your first backend resources
- Deploy and test your local Amplify backend

## AWS Amplify Gen2 Overview

![Amplify Gen2 Architecture Placeholder](../images/diagrams/amplify-gen2-architecture-placeholder.png)

### What's New in Gen2

AWS Amplify Gen2 represents a significant evolution from Gen1, focusing on:

| Aspect | Gen1 | Gen2 |
|--------|------|------|
| **Configuration** | JSON/YAML files | TypeScript-first configuration |
| **Type Safety** | Runtime errors | Compile-time type checking |
| **Local Development** | Limited sandbox | Full-featured local sandbox |
| **Backend Definition** | CLI-driven | Code-driven with Git workflows |
| **Deployment** | CLI commands | Git-based deployments |

### Key Advantages

**TypeScript** **Type Safety** **Local Development** **Git Integration**

!!! tip "Why Gen2?"
    - **Developer Experience**: TypeScript-first approach with full IntelliSense
    - **Faster Iteration**: Local sandbox for rapid development and testing
    - **Better Collaboration**: Infrastructure as code with Git workflows
    - **Production Ready**: Automated deployments with branch-based environments

## Understanding Project Structure

![Amplify Project Structure Placeholder](../images/screenshots/amplify-project-structure-placeholder.png)

### Gen2 Project Layout

After initializing Amplify, your project will include:

```
amplify-training-project/
├── amplify/
│   ├── auth/                    # Authentication configuration
│   ├── data/                    # Data models and API schema
│   ├── functions/               # Lambda functions
│   ├── storage/                 # File storage configuration
│   ├── backend.ts               # Main backend configuration
│   └── package.json             # Backend dependencies
├── src/
│   └── (your Vue.js app)        # Frontend application
└── amplify_outputs.json         # Generated configuration file
```

### Key Configuration Files

| File | Purpose | Language |
|------|---------|----------|
| `amplify/backend.ts` | Main backend definition | TypeScript |
| `amplify/data/resource.ts` | Data models and GraphQL schema | TypeScript |
| `amplify/auth/resource.ts` | Authentication configuration | TypeScript |
| `amplify_outputs.json` | Generated client configuration | JSON |

## Amplify CLI Installation and Setup

![Amplify CLI Installation Process Placeholder](../images/screenshots/amplify-cli-installation-placeholder.png)

### Verify Amplify CLI Installation

First, let's ensure the Amplify CLI is properly installed:

1. **Installation Check**: Verify CLI version and availability
2. **Access Configuration**: Ensure AWS credentials are set
3. **Connectivity Test**: Validate connection to AWS services

```bash
# Verify Amplify CLI installation
amplify --version
# Should output @aws-amplify/cli version 12.x.x or later

# Check AWS configuration
aws configure list
# Verify your credentials are properly set

# Test AWS connectivity
aws sts get-caller-identity
# Should return your AWS account information
```

### Configure Amplify (if not done already)

If you haven't configured the Amplify CLI yet:

```bash
amplify configure
```

![Amplify Configuration Process Placeholder](../images/screenshots/amplify-configure-process-placeholder.png)

This will:
1. Open a browser to AWS Console
2. Guide you through IAM user creation
3. Set up local credentials
4. Configure region settings

## Initialize Amplify Gen2 Project

![Amplify Initialization Process Placeholder](../images/screenshots/amplify-init-process-placeholder.png)

### Step 1: Initialize Amplify in Vue Project

Navigate to your project directory from Day 1:

```bash
cd amplify-training-project

# Initialize Amplify Gen2
amplify init --quickstart

# Follow the prompts:
# ? Enter a name for the project: amplify-training-project
# ? Initialize the project with the above configuration? Yes
```

### Step 2: Examine Generated Files

After initialization, examine the new files:

```bash
# List Amplify-related files
ls -la amplify/
find . -name "amplify_outputs.json" -o -name ".amplifyrc"
```

![Generated Files Structure Placeholder](../images/screenshots/generated-files-structure-placeholder.png)

### Step 3: Understanding backend.ts

Examine the main backend configuration file:

```typescript title="amplify/backend.ts"
import { defineBackend } from '@aws-amplify/backend'
import { auth } from './auth/resource'
import { data } from './data/resource'

/**
 * @see https://docs.amplify.aws/react/build-a-backend/ to add storage, functions, and more
 */
const backend = defineBackend({
  auth,
  data,
})
```

This file is the entry point for your backend configuration, importing and combining different resources.

## Understanding the Sandbox Environment

![Sandbox Environment Overview Placeholder](../images/screenshots/sandbox-environment-placeholder.png)

### What is the Sandbox?

The Amplify Sandbox is a local development environment that:

- **Runs locally** but provisions real AWS resources
- **Isolates development** from production environments
- **Hot reloads** backend code changes in real-time
- **Auto-cleanup** resources when stopped

### Sandbox vs Deployed Environments

| Environment | Purpose | Lifecycle | Resources |
|-------------|---------|-----------|-----------|
| **Sandbox** | Local development | Temporary | Auto-cleanup |
| **Branch Environment** | Feature development | Git-based | Persistent |
| **Production** | Live application | Manual control | Persistent |

## Starting Your First Sandbox

![Sandbox Startup Process Placeholder](../images/screenshots/sandbox-startup-placeholder.png)

### Step 1: Start the Sandbox

```bash
# Start the local sandbox
amplify sandbox

# This will:
# 1. Install backend dependencies
# 2. Compile TypeScript backend code
# 3. Deploy resources to AWS
# 4. Generate amplify_outputs.json
# 5. Watch for changes
```

![Sandbox Ready Placeholder](../images/screenshots/sandbox-ready-placeholder.png)

### Step 2: Monitor the Sandbox

Watch the console output for:

- **Compilation**: TypeScript backend code compilation
- **Deployment**: AWS resource creation progress
- **Configuration**: Generated client configuration
- **Ready State**: Sandbox ready for development

### Step 3: Verify AWS Resources

Check the AWS Console to see created resources:

![AWS Console Resources Placeholder](../images/screenshots/aws-console-resources-placeholder.png)

```bash
# List CloudFormation stacks
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE

# Check Amplify app in console
aws amplify list-apps
```

## Default Backend Resources

![Default Resources Diagram Placeholder](../images/diagrams/default-resources-placeholder.png)

### Authentication (Cognito)

By default, Amplify Gen2 creates:

- **User Pool**: For user management and authentication
- **User Pool Client**: For application integration
- **Identity Pool**: For AWS service access

### Data (AppSync + DynamoDB)

Default data resources include:

- **GraphQL API**: Powered by AWS AppSync
- **DynamoDB Tables**: For data storage
- **Resolvers**: Connecting GraphQL to DynamoDB

### Generated Configuration

The `amplify_outputs.json` file contains:

```json title="amplify_outputs.json"
{
  "auth": {
    "user_pool_id": "us-east-1_xxxxxxxxx",
    "aws_region": "us-east-1",
    "user_pool_client_id": "xxxxxxxxxxxxxxxxxx"
  },
  "data": {
    "url": "https://xxxxxxxxxx.appsync-api.us-east-1.amazonaws.com/graphql",
    "aws_region": "us-east-1",
    "default_authorization_type": "AMAZON_COGNITO_USER_POOLS"
  }
}
```

## Working with Amplify CLI

![Amplify CLI Commands Placeholder](../images/screenshots/amplify-cli-commands-placeholder.png)

### Essential Commands

```bash
# View project status
amplify status

# Start sandbox environment
amplify sandbox

# Generate types for GraphQL
amplify generate graphql-api-code

# View backend details
amplify console

# Stop sandbox and cleanup resources
amplify sandbox --delete
```

### Sandbox Management

```bash
# Start sandbox in background
amplify sandbox &

# View sandbox logs
amplify sandbox --logs

# Force restart sandbox
amplify sandbox --delete && amplify sandbox
```

## Integrating with Vue Frontend

![Vue Amplify Integration Placeholder](../images/screenshots/vue-amplify-integration-placeholder.png)

### Step 1: Install Amplify Libraries

```bash
# Install Amplify Vue libraries
npm install @aws-amplify/ui-vue aws-amplify
```

### Step 2: Configure Amplify in Vue

Update your `src/main.ts` file:

```typescript title="src/main.ts"
import { createApp } from 'vue'
import { Amplify } from 'aws-amplify'
import outputs from '../amplify_outputs.json'
import App from './App.vue'
import router from './router'
import { createPinia } from 'pinia'

// Configure Amplify
Amplify.configure(outputs)

const app = createApp(App)

app.use(createPinia())
app.use(router)

app.mount('#app')
```

### Step 3: Test the Integration

![Testing Setup Placeholder](../images/screenshots/testing-setup-placeholder.png)

Create a simple test component to verify integration:

```vue title="src/components/AmplifyTest.vue"
<template>
  <div class="amplify-test">
    <h3>Amplify Integration Test</h3>
    <p>Backend Status: {{ backendStatus }}</p>
    <button @click="testConnection">Test Connection</button>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { generateClient } from 'aws-amplify/data'

const backendStatus = ref('Not tested')

const client = generateClient()

const testConnection = async () => {
  try {
    // Simple test query
    backendStatus.value = 'Testing...'
    
    // This will test the connection to your GraphQL API
    const response = await client.graphql({
      query: `query ListTodos { listTodos { items { id } } }`
    })
    
    backendStatus.value = 'Connected ✅'
  } catch (error) {
    console.error('Connection test failed:', error)
    backendStatus.value = 'Failed ❌'
  }
}
</script>
```

### Step 4: Add Test to Your App

Update `src/App.vue` to include the test component:

```vue title="src/App.vue"
<template>
  <div id="app">
    <nav>
      <router-link to="/">Home</router-link> |
      <router-link to="/about">About</router-link>
    </nav>
    
    <AmplifyTest />
    
    <router-view />
  </div>
</template>

<script setup lang="ts">
import AmplifyTest from './components/AmplifyTest.vue'
</script>
```

## Testing Your Setup

![Working Integration Placeholder](../images/screenshots/working-integration-placeholder.png)

### Step 1: Start Both Servers

Open two terminal windows:

```bash
# Terminal 1: Start Amplify Sandbox
amplify sandbox

# Terminal 2: Start Vue Dev Server
npm run dev
```

### Step 2: Verify Frontend

Visit http://localhost:3000 and confirm:

1. **Vue App Loads**: Your application starts correctly
2. **Amplify Integration**: No console errors
3. **Test Component**: Amplify Test component displays
4. **Connection Test**: Test button works (may show expected errors for now)

### Step 3: Check GraphQL API

![GraphQL Playground Placeholder](../images/screenshots/graphql-playground-placeholder.png)

Access the GraphQL API directly:

```bash
# Get the GraphQL endpoint from amplify_outputs.json
cat amplify_outputs.json | grep url

# Open the GraphQL playground in browser
# (endpoint URL + /console)
```

## Day 2 Completion Checklist

![Day 2 Completion Status Placeholder](../images/screenshots/day2-completion-status-placeholder.png)

Ensure you have completed all items:

### ✅ Amplify Setup
- [ ] Amplify CLI installed and configured
- [ ] Amplify Gen2 project initialized successfully
- [ ] Backend TypeScript configuration understood
- [ ] Project structure familiarized

### ✅ Sandbox Environment
- [ ] Sandbox started successfully
- [ ] AWS resources created and visible in console
- [ ] amplify_outputs.json generated
- [ ] Sandbox commands working

### ✅ Frontend Integration
- [ ] Amplify libraries installed
- [ ] Vue app configured with Amplify
- [ ] Test component created and working
- [ ] No console errors in browser

### ✅ Understanding
- [ ] Gen2 vs Gen1 differences understood
- [ ] Sandbox vs production environment concepts clear
- [ ] TypeScript-first configuration benefits grasped
- [ ] Basic Amplify CLI commands memorized

## Troubleshooting Common Issues

### Amplify CLI Version Issues

```bash
# Update to latest Amplify CLI
npm install -g @aws-amplify/cli@latest

# Clear global npm cache if needed
npm cache clean --force
```

### AWS Permission Errors

If you encounter permission issues:

1. Verify IAM user has necessary policies
2. Check AWS region consistency
3. Ensure credentials are up to date
4. Try `aws sts get-caller-identity` to verify access

### Sandbox Startup Problems

```bash
# Clean and restart
amplify sandbox --delete
rm -rf node_modules amplify_outputs.json
npm install
amplify sandbox
```

### TypeScript Compilation Errors

```bash
# Install backend dependencies
cd amplify
npm install
cd ..

# Restart sandbox
amplify sandbox
```

## Next Steps

You're ready for [Day 3: Authentication (Cognito)](day03-authentication.md) where we'll:

- Configure AWS Cognito authentication
- Add sign-up and sign-in functionality
- Implement user session management
- Secure your application with authentication guards

---

**🚀 Great progress!** You now have a working Amplify Gen2 backend connected to your Vue frontend.
