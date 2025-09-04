# Prerequisites

Before starting the AWS Amplify Development Manual training program, ensure your development environment and knowledge meet the following requirements.

## Required Software

![Development Tools Setup Placeholder](images/screenshots/dev-tools-setup-placeholder.png)

### Core Development Tools

| Tool | Version | Purpose | Installation Link |
|------|---------|---------|-------------------|
| **Node.js** | 18.x or later | JavaScript runtime | [nodejs.org](https://nodejs.org/) |
| **npm/yarn** | Latest | Package manager | Included with Node.js |
| **Git** | 2.x or later | Version control | [git-scm.com](https://git-scm.com/) |
| **Visual Studio Code** | Latest | Code editor | [code.visualstudio.com](https://code.visualstudio.com/) |

### AWS Tools

1. **AWS CLI** - Version 2.x or later for AWS service management
2. **AWS Account** - Active account with appropriate permissions
3. **Amplify CLI** - Latest version for project management

!!! warning "Required AWS Permissions"
    Your AWS account needs permissions for:
    
    - Amazon Cognito (Authentication)
    - AWS AppSync (GraphQL API)
    - Amazon DynamoDB (Database)
    - AWS Lambda (Serverless functions)
    - Amazon S3 (Storage)
    - AWS IAM (Identity & Access Management)

## Required Knowledge

### Essential Skills

!!! success "Must Have"
    - **JavaScript/TypeScript Basics** (ES6+, async/await, modules)
    - **Vue.js Fundamentals** (components, reactivity, lifecycle)
    - **HTML/CSS** (semantic markup, responsive design)
    - **REST APIs** (HTTP methods, JSON, status codes)
    - **Command Line** (basic terminal/command prompt usage)

### Recommended Experience

!!! info "Nice to Have"
    - Experience with cloud platforms (AWS, Azure, GCP)
    - Understanding of serverless architecture concepts
    - Basic knowledge of GraphQL
    - Familiarity with SAP systems (helpful for integration modules)
    - Version control experience (Git workflows)

## Environment Setup

![Environment Setup Process Placeholder](images/screenshots/environment-setup-process-placeholder.png)

### Step 1: Verify Node.js Installation

```bash
# Check Node.js version
node --version
# Should output v18.x.x or later

# Check npm version
npm --version
# Should output 9.x.x or later
```

### Step 2: Install AWS CLI

```bash
# Using Homebrew (macOS)
brew install awscli

# Or download installer from AWS
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```

### Step 3: Configure AWS Credentials

![AWS Configuration Screenshot Placeholder](images/screenshots/aws-config-placeholder.png)

```bash
aws configure
# Enter your AWS Access Key ID
# Enter your AWS Secret Access Key
# Enter default region (e.g., us-east-1)
# Enter default output format (json)
```

### Step 4: Install Amplify CLI

```bash
npm install -g @aws-amplify/cli
amplify --version
```

### Step 5: Configure Amplify

```bash
amplify configure
# This opens a browser for AWS Console sign-in
# Follow prompts to create IAM user for Amplify
```

## VS Code Extensions

![VS Code Extensions Screenshot Placeholder](images/screenshots/vscode-extensions-placeholder.png)

For optimal development experience, install these recommended extensions:

### Essential Extensions

- **Vue Language Features (Volar)** - Vue 3 support
- **TypeScript Vue Plugin (Volar)** - TypeScript integration
- **AWS Amplify** - Amplify project management
- **AWS CloudFormation** - Infrastructure as code support

### Useful Extensions

- **GitLens** - Enhanced Git capabilities
- **Prettier** - Code formatting
- **ESLint** - Code linting
- **Auto Rename Tag** - HTML/XML tag management
- **Bracket Pair Colorizer** - Code readability improvement

## Verification Checklist

Complete this checklist before starting Day 1:

- [ ] Node.js 18+ installed and verified
- [ ] AWS CLI installed and configured
- [ ] Amplify CLI globally installed
- [ ] AWS account with proper permissions
- [ ] VS Code with recommended extensions
- [ ] Git configured with credentials
- [ ] Terminal/command prompt accessible

![Final Environment Check Placeholder](images/screenshots/final-env-check-placeholder.png)

### Quick Verification Script

Create and run this script to verify your setup:

```bash
#!/bin/bash
echo "=== Development Environment Check ==="
echo ""

echo "Node.js version:"
node --version || echo "❌ Node.js not found"

echo ""
echo "npm version:"
npm --version || echo "❌ npm not found"

echo ""
echo "AWS CLI version:"
aws --version || echo "❌ AWS CLI not found"

echo ""
echo "Amplify CLI version:"
amplify --version || echo "❌ Amplify CLI not found"

echo ""
echo "Git version:"
git --version || echo "❌ Git not found"

echo ""
echo "AWS configuration:"
aws configure list || echo "❌ AWS not configured"

echo ""
echo "=== Check complete ==="
```

## Common Issues Troubleshooting

### Node.js Issues

!!! warning "Node Version Managers"
    If you have multiple Node.js versions, consider using:
    - **nvm** (macOS/Linux): Node Version Manager
    - **nvm-windows** (Windows): Windows version of nvm

### AWS Permissions

If you encounter permission errors:

1. Verify IAM user has required policies
2. Check AWS region consistency
3. Ensure credentials are correctly configured
4. Test with AWS CLI commands

### Amplify CLI Issues

Common solutions:

```bash
# Clear Amplify cache
amplify delete
rm -rf .amplify

# Reinstall Amplify CLI
npm uninstall -g @aws-amplify/cli
npm install -g @aws-amplify/cli
```

---

**Ready to start? Proceed to [Day 1: Kickoff & Setup](foundation/day01-kickoff-setup.md)** 
