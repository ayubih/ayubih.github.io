---
title: "Setting Up GitHub Actions from Terminal: A Complete Guide"
date: 2025-08-26T19:30:00-05:00
draft: false
tags: ["github-actions", "devops", "terminal", "automation", "hugo"]
categories: ["tutorials"]
author: "Ayush"
showToc: true
description: "Learn how to create and configure GitHub Actions workflows from the terminal, with detailed explanations of each step for automated Hugo blog deployment."
cover:
  image: ""
  alt: "GitHub Actions workflow automation"
  caption: "Automate your deployments with GitHub Actions"
---

## Introduction

GitHub Actions is a powerful CI/CD platform that allows you to automate your software development workflows directly from your GitHub repository. In this guide, we'll walk through creating a GitHub Actions workflow from the terminal, specifically for deploying a Hugo blog to GitHub Pages.

## What are GitHub Actions?

GitHub Actions are automated workflows that run in response to events in your GitHub repository. They can:

- **Build and test** your code automatically
- **Deploy** your applications 
- **Run scripts** on schedule or events
- **Automate repetitive tasks**

## Prerequisites

Before we start, make sure you have:

- A GitHub repository
- Git installed and configured
- Terminal/command line access
- Basic knowledge of YAML syntax

## Step 1: Create the Workflow Directory

First, we need to create the proper directory structure for GitHub Actions:

```bash
# Navigate to your project root
cd your-project-directory

# Create the workflows directory
mkdir -p .github/workflows
```

**What this does:**
- `.github/` is a special directory that GitHub recognizes
- `workflows/` contains all your GitHub Actions workflow files
- `-p` flag creates parent directories if they don't exist

## Step 2: Create the Workflow File

```bash
# Create a new workflow file
touch .github/workflows/deploy.yml

# Or create with content using your preferred editor
nano .github/workflows/deploy.yml
```

**What this does:**
- Creates a YAML file that defines your workflow
- The filename can be anything descriptive (e.g., `deploy.yml`, `ci.yml`)
- YAML format is required for GitHub Actions

## Step 3: Understanding Workflow Structure

Let's break down a complete Hugo deployment workflow:

```yaml
# .github/workflows/hugo.yml
name: Deploy Hugo site to Pages

# Trigger conditions
on:
  push:
    branches:
      - main          # Runs when pushing to main branch
  workflow_dispatch:  # Allows manual trigger from GitHub UI

# Required permissions
permissions:
  contents: read      # Read repository files
  pages: write        # Write to GitHub Pages
  id-token: write     # Security token for deployment

# Prevent concurrent deployments
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default shell for all steps
defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest    # Virtual machine to use
    env:
      HUGO_VERSION: 0.148.2   # Hugo version to install
    
    steps:
      # Step 1: Install Hugo
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb \
            https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb
          sudo dpkg -i ${{ runner.temp }}/hugo.deb
      
      # Step 2: Install Sass (for CSS processing)
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      
      # Step 3: Download repository code
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive  # Include git submodules (themes)
          fetch-depth: 0         # Full git history
      
      # Step 4: Configure GitHub Pages
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      
      # Step 5: Install Node.js dependencies (if any)
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      
      # Step 6: Build the Hugo site
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
          TZ: America/Los_Angeles
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      
      # Step 7: Upload built site
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  # Deployment job (runs after build)
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build              # Wait for build job to complete
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## Step 4: Understanding Each Component

### **Triggers (`on`)**
```yaml
on:
  push:
    branches: [main]    # Run on push to main
  workflow_dispatch:    # Manual trigger button
  schedule:            # Run on schedule
    - cron: '0 2 * * 1'  # Every Monday at 2 AM
```

### **Jobs and Steps**
- **Jobs** run in parallel by default
- **Steps** run sequentially within a job
- Each step can run commands or use pre-built actions

### **Environment Variables**
```bash
# Set environment variables
env:
  HUGO_VERSION: 0.148.2
  NODE_VERSION: 18

# Use in steps
run: echo "Hugo version: $HUGO_VERSION"
```

### **GitHub Contexts**
- `${{ github.repository }}` - Repository name
- `${{ github.ref }}` - Branch/tag reference
- `${{ runner.temp }}` - Temporary directory
- `${{ steps.stepid.outputs.variable }}` - Step outputs

## Step 5: Commit and Push the Workflow

```bash
# Add the workflow file
git add .github/workflows/

# Commit with descriptive message
git commit -m "Add GitHub Actions workflow for Hugo deployment"

# Push to trigger the workflow
git push origin main
```

**What happens next:**
1. GitHub detects the new workflow file
2. The workflow runs automatically (if triggered by push)
3. You can see progress in the **Actions** tab of your repository

## Step 6: Monitor Workflow Execution

```bash
# Check workflow status (if you have GitHub CLI)
gh run list

# View specific run
gh run view [run-id]

# Watch logs in real-time
gh run watch
```

**Via GitHub Web Interface:**
1. Go to your repository
2. Click **Actions** tab
3. Select your workflow
4. View logs and status

## Common Workflow Patterns

### **Matrix Builds (Multiple Versions)**
```yaml
strategy:
  matrix:
    hugo-version: [0.147.0, 0.148.2]
    os: [ubuntu-latest, windows-latest]
```

### **Conditional Steps**
```yaml
- name: Deploy to production
  if: github.ref == 'refs/heads/main'
  run: echo "Deploying to production"
```

### **Secrets and Variables**
```yaml
- name: Deploy
  env:
    API_KEY: ${{ secrets.API_KEY }}
    ENVIRONMENT: ${{ vars.ENVIRONMENT }}
  run: deploy-script.sh
```

## Troubleshooting Common Issues

### **Workflow Not Triggering**
```bash
# Check file location
ls -la .github/workflows/

# Validate YAML syntax
yamllint .github/workflows/deploy.yml
```

### **Permission Errors**
```yaml
# Add required permissions
permissions:
  contents: read
  pages: write
  id-token: write
```

### **Build Failures**
- Check the **Actions** tab for error logs
- Verify all required secrets are set
- Ensure paths and commands are correct

## Advanced Features

### **Workflow Templates**
```bash
# Create reusable workflow templates
mkdir .github/workflow-templates
```

### **Composite Actions**
```bash
# Create custom actions
mkdir -p .github/actions/setup-hugo
```

### **Environment Protection**
- Set up deployment environments
- Add approval requirements
- Configure environment secrets

## Best Practices

1. **Use Specific Action Versions**
   ```yaml
   uses: actions/checkout@v4  # Pin to specific version
   ```

2. **Cache Dependencies**
   ```yaml
   - uses: actions/cache@v3
     with:
       path: ~/.npm
       key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
   ```

3. **Fail Fast**
   ```yaml
   strategy:
     fail-fast: true
   ```

4. **Use Descriptive Names**
   ```yaml
   name: "Deploy Hugo Blog to GitHub Pages"
   ```

## Security Considerations

- **Never commit secrets** to your repository
- **Use environment variables** for sensitive data
- **Limit permissions** to minimum required
- **Review third-party actions** before using

## Conclusion

GitHub Actions provides a powerful way to automate your development workflow directly from your repository. By understanding the YAML structure and following best practices, you can create robust CI/CD pipelines that save time and reduce manual errors.

### **Key Takeaways:**
- ✅ Workflows are defined in `.github/workflows/` directory
- ✅ YAML syntax defines triggers, jobs, and steps
- ✅ Each component serves a specific purpose
- ✅ Monitor execution through GitHub's Actions tab
- ✅ Use environment variables and secrets for configuration

Start with simple workflows and gradually add complexity as needed. The terminal provides full control over your GitHub Actions setup, making it easy to version control and collaborate on your automation pipelines.

---

**Next Steps:**
- Explore the GitHub Actions Marketplace for pre-built actions
- Set up notifications for workflow failures
- Create custom actions for your specific needs
- Implement advanced deployment strategies

Happy automating! 🚀