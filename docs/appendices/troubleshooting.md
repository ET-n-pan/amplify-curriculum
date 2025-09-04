# Troubleshooting Guide

This comprehensive troubleshooting guide covers common issues encountered during the AWS Amplify development training program.

## Environment Setup Issues

![Environment Issues Troubleshooting Placeholder](../images/screenshots/env-troubleshooting-placeholder.png)

### Node.js and npm Issues

!!! error "Node Version Incompatibility"
    **Symptoms**: Build fails, dependency conflicts, or CLI commands not working
    
    **Solutions**:
    ```bash
    # Check current Node version
    node --version
    
    # Install Node Version Manager (nvm)
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
    
    # Install and use Node 18+
    nvm install 18
    nvm use 18
    nvm alias default 18
    ```

!!! error "npm Permission Errors"
    **Symptoms**: `EACCES` errors when installing global packages
    
    **Solutions**:
    ```bash
    # Configure npm to use a different directory
    mkdir ~/.npm-global
    npm config set prefix '~/.npm-global'
    echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.profile
    source ~/.profile
    
    # Alternative: Fix permissions
    sudo chown -R $(whoami) $(npm config get prefix)/{lib/node_modules,bin,share}
    ```

### AWS CLI and Credentials

!!! error "AWS CLI Not Found"
    **Symptoms**: `aws: command not found`
    
    **Solutions**:
    ```bash
    # macOS with Homebrew
    brew install awscli
    
    # Windows
    # Download from: https://awscli.amazonaws.com/AWSCLIV2.msi
    
    # Linux
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install
    ```

!!! error "Invalid Credentials"
    **Symptoms**: `Unable to locate credentials` or permission denied errors
    
    **Solutions**:
    ```bash
    # Reconfigure AWS CLI
    aws configure
    
    # Check current configuration
    aws configure list
    aws sts get-caller-identity
    
    # Alternative: Use environment variables
    export AWS_ACCESS_KEY_ID=your_access_key
    export AWS_SECRET_ACCESS_KEY=your_secret_key
    export AWS_DEFAULT_REGION=us-east-1
    ```

## Amplify CLI Issues

![Amplify CLI Issues Placeholder](../images/screenshots/amplify-cli-issues-placeholder.png)

### Installation Problems

!!! error "Amplify CLI Installation Fails"
    **Symptoms**: npm install fails or CLI commands not found
    
    **Solutions**:
    ```bash
    # Clear npm cache
    npm cache clean --force
    
    # Uninstall and reinstall
    npm uninstall -g @aws-amplify/cli
    npm install -g @aws-amplify/cli@latest
    
    # Verify installation
    amplify --version
    which amplify
    ```

### Sandbox Issues

!!! error "Sandbox Deployment Fails"
    **Symptoms**: Resources fail to deploy, timeout errors
    
    **Solutions**:
    ```bash
    # Check AWS service limits
    aws service-quotas get-service-quota --service-code amplify --quota-code L-123456
    
    # Clear Amplify cache
    rm -rf ~/.amplify
    rm -rf .amplify
    
    # Try different region
    aws configure set region us-west-2
    
    # Restart sandbox with verbose logging
    amplify sandbox --debug
    ```

!!! error "Sandbox Stuck in Deployment"
    **Symptoms**: Deployment hangs indefinitely
    
    **Solutions**:
    ```bash
    # Force stop current deployment
    Ctrl+C
    
    # Check CloudFormation stacks
    aws cloudformation list-stacks --stack-status-filter CREATE_IN_PROGRESS UPDATE_IN_PROGRESS
    
    # Delete stuck resources
    amplify delete
    
    # Wait and retry
    sleep 60
    amplify sandbox
    ```

## Vue.js Integration Issues

![Vue Integration Issues Placeholder](../images/screenshots/vue-integration-issues-placeholder.png)

### TypeScript Compilation Errors

!!! error "TypeScript Build Failures"
    **Symptoms**: `tsc` errors, type checking failures
    
    **Solutions**:
    ```bash
    # Update TypeScript
    npm install -D typescript@latest
    
    # Clear TypeScript cache
    rm -rf node_modules/.cache
    npx tsc --build --clean
    
    # Check tsconfig.json configuration
    npx tsc --showConfig
    ```

!!! error "Module Resolution Issues"
    **Symptoms**: Cannot resolve imports, path mapping not working
    
    **Solutions**:
    ```json
    // Update tsconfig.json
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
    // Update vite.config.ts
    import path from 'path'
    
    export default defineConfig({
      resolve: {
        alias: {
          '@': path.resolve(__dirname, './src')
        }
      }
    })
    ```

### Amplify Configuration Issues

!!! error "amplify_outputs.json Not Found"
    **Symptoms**: Module not found error for amplify_outputs.json
    
    **Solutions**:
    ```bash
    # Ensure sandbox is running
    amplify sandbox
    
    # Check if file exists
    ls -la amplify_outputs.json
    
    # Generate configuration manually
    amplify generate config
    
    # Create type definition if needed
    echo 'declare module "*amplify_outputs.json"' > src/amplify_outputs.d.ts
    ```

!!! error "Amplify Configuration Errors"
    **Symptoms**: Runtime configuration errors, API calls fail
    
    **Solutions**:
    ```typescript
    // Debug configuration in main.ts
    import outputs from '../amplify_outputs.json'
    console.log('Amplify config:', outputs)
    
    // Validate configuration structure
    if (!outputs.auth || !outputs.data) {
      console.error('Invalid Amplify configuration')
    }
    
    // Re-configure Amplify
    import { Amplify } from 'aws-amplify'
    Amplify.configure(outputs)
    ```

## Authentication Issues

![Authentication Issues Placeholder](../images/screenshots/auth-issues-placeholder.png)

### Sign-up/Sign-in Problems

!!! error "User Already Exists"
    **Symptoms**: SignUp operation returns UserAlreadyExistsException
    
    **Solutions**:
    ```typescript
    // Check if user exists before sign-up
    import { signIn } from 'aws-amplify/auth'
    
    try {
      await signIn({ username, password })
      // User exists, redirect to app
    } catch (error) {
      if (error.name === 'UserNotFoundException') {
        // Proceed with sign-up
        await signUp({ username, password })
      }
    }
    ```

!!! error "Confirmation Code Issues"
    **Symptoms**: Invalid verification code, code expired
    
    **Solutions**:
    ```typescript
    // Resend confirmation code
    import { resendSignUpCode } from 'aws-amplify/auth'
    
    await resendSignUpCode({ username })
    
    // Check email/SMS for new code
    // Ensure code is entered correctly (no spaces)
    ```

### Session Management

!!! error "Token Expired Errors"
    **Symptoms**: API calls return unauthorized after some time
    
    **Solutions**:
    ```typescript
    // Implement automatic token refresh
    import { fetchAuthSession, signOut } from 'aws-amplify/auth'
    
    try {
      const session = await fetchAuthSession()
      if (!session.tokens) {
        throw new Error('No valid tokens')
      }
    } catch (error) {
      // Redirect to login
      await signOut()
      router.push('/login')
    }
    ```

## Data/API Issues

![API Issues Placeholder](../images/screenshots/api-issues-placeholder.png)

### GraphQL Schema Issues

!!! error "GraphQL Schema Sync Errors"
    **Symptoms**: Client queries don't match backend schema
    
    **Solutions**:
    ```bash
    # Regenerate GraphQL client code
    amplify generate graphql-client-code --format typescript
    
    # Update schema and restart sandbox
    # Edit amplify/data/resource.ts
    # Ctrl+C to stop sandbox
    amplify sandbox
    ```

!!! error "Authorization Errors"
    **Symptoms**: Unauthorized access to GraphQL operations
    
    **Solutions**:
    ```typescript
    // Check authorization rules in schema
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
    
    // Ensure user is authenticated before API calls
    import { getCurrentUser } from 'aws-amplify/auth'
    
    try {
      await getCurrentUser()
      // User is authenticated, proceed with API calls
    } catch (error) {
      // Redirect to login
    }
    ```

### Network and CORS Issues

!!! error "CORS Policy Errors"
    **Symptoms**: Blocked by CORS policy in browser console
    
    **Solutions**:
    ```typescript
    // CORS is handled automatically by Amplify
    // Ensure you're using the correct API endpoint
    
    // Check amplify_outputs.json for correct URLs
    import outputs from '../amplify_outputs.json'
    console.log('API URL:', outputs.data?.url)
    
    // Verify sandbox is running and accessible
    curl -X POST outputs.data.url -H "Content-Type: application/json"
    ```

## Build and Deployment Issues

![Build Issues Placeholder](../images/screenshots/build-issues-placeholder.png)

### Build Failures

!!! error "Vite Build Errors"
    **Symptoms**: Build command fails with various errors
    
    **Solutions**:
    ```bash
    # Clear build cache
    rm -rf dist
    rm -rf node_modules/.vite
    
    # Update dependencies
    npm update
    
    # Build with verbose output
    npm run build -- --debug
    
    # Check for syntax errors
    npx tsc --noEmit
    ```

!!! error "Memory Issues During Build"
    **Symptoms**: JavaScript heap out of memory errors
    
    **Solutions**:
    ```bash
    # Increase Node.js memory limit
    export NODE_OPTIONS="--max-old-space-size=4096"
    npm run build
    
    # Or modify package.json
    {
      "scripts": {
        "build": "NODE_OPTIONS=--max-old-space-size=4096 vite build"
      }
    }
    ```

## Performance Issues

![Performance Issues Placeholder](../images/screenshots/performance-issues-placeholder.png)

### Slow Development Server

!!! error "Slow Hot Module Replacement (HMR)"
    **Symptoms**: Changes take long to reflect in browser
    
    **Solutions**:
    ```typescript
    // Optimize Vite configuration
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

### Large Bundle Size

!!! error "Bundle Size Too Large"
    **Symptoms**: Slow loading times, large dist files
    
    **Solutions**:
    ```typescript
    // Implement code splitting
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
    
    // Use dynamic imports for large components
    const HeavyComponent = defineAsyncComponent(() => import('./HeavyComponent.vue'))
    ```

## Debug Tools and Commands

![Debug Tools Placeholder](../images/screenshots/debug-tools-placeholder.png)

### Useful Debug Commands

```bash
# Check all versions
node --version
npm --version
aws --version
amplify --version

# AWS debugging
aws sts get-caller-identity
aws configure list
aws logs describe-log-groups

# Amplify debugging
amplify status
amplify env list
amplify diagnose

# Network debugging
curl -I https://api-endpoint
nslookup api-endpoint
ping api-endpoint
```

### Browser Developer Tools

1. **Console Tab**: Check for JavaScript errors and warnings
2. **Network Tab**: Monitor API requests and responses  
3. **Application Tab**: Inspect local storage and session data
4. **Sources Tab**: Set breakpoints and debug code execution

### Log Analysis

```bash
# View Amplify logs
cat ~/.amplify/logs/amplify-cli.log

# View AWS CloudFormation events
aws cloudformation describe-stack-events --stack-name amplify-*

# View Lambda function logs
aws logs tail /aws/lambda/function-name --follow
```

## Getting Additional Help

### Community Resources

- **AWS Amplify Discord**: [discord.gg/amplify](https://discord.gg/amplify)
- **GitHub Issues**: [github.com/aws-amplify/amplify-cli/issues](https://github.com/aws-amplify/amplify-cli/issues)
- **Stack Overflow**: Tag questions with `aws-amplify`
- **AWS Forums**: [forums.aws.amazon.com](https://forums.aws.amazon.com)

### AWS Support Options

- **AWS Documentation**: [docs.amplify.aws](https://docs.amplify.aws)
- **AWS Support Center**: For paid support plans
- **AWS re:Post**: Community-driven Q&A platform

!!! tip "Before Asking for Help"
    1. Search existing issues and documentation
    2. Provide complete error messages and logs
    3. Include relevant configuration files
    4. Specify versions of all tools and dependencies
    5. Describe steps to reproduce the issue
