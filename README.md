# Deploy React Vite App to GitHub Pages using GitHub Actions

This guide provides a comprehensive walkthrough of deploying a React Vite application to GitHub Pages using GitHub Actions for continuous deployment.

## Table of Contents
- [Overview](#overview)
- [Step-by-Step Deployment Guide](#step-by-step-deployment-guide)
  - [1. Setting up a Vite React Project](#1-setting-up-a-vite-react-project)
  - [2. Creating a GitHub Repository](#2-creating-a-github-repository)
  - [3. Creating the Deployment Workflow](#3-creating-the-deployment-workflow)
  - [4. Configuring GitHub Actions Permissions](#4-configuring-github-actions-permissions)
  - [5. Setting up GitHub Pages](#5-setting-up-github-pages)
  - [6. Fixing Asset Links](#6-fixing-asset-links)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)
- [Resources](#resources)


## Overview

This project is built with:
- React - A JavaScript library for building user interfaces
- Vite - A modern frontend build tool that significantly improves the development experience
- GitHub Actions - For continuous integration and deployment
- GitHub Pages - For hosting the built application

## Step-by-Step Deployment Guide

### 1. Setting up a Vite React Project

If you're starting from scratch, create a new Vite project:

```bash
# Create new Vite project using React template
npm create vite@latest my-project -- --template react

# Navigate to project directory
cd my-project

# Install dependencies
npm install

# Start development server
npm run dev
```

Initialize Git repository:

```bash
git init
git add .
git commit -m "Initial Vite project setup"
```

### 2. Creating a GitHub Repository

1. Go to https://github.com/new
2. Name your repository (e.g., react-deploy-demo)
3. Choose "Public" visibility (required for GitHub Pages with a free account)
4. Click "Create repository"

Connect your local repository:

```bash
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/react-deploy-demo.git
git push -u origin main
```

### 3. Creating the Deployment Workflow

Create a new file at `.github/workflows/deploy.yml` with the following content:

```yml
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        uses: bahmutov/npm-install@v1

      - name: Build project
        run: npm run build

      - name: Upload production-ready build files
        uses: actions/upload-artifact@v4
        with:
          name: production-files
          path: ./dist

  deploy:
    name: Deploy
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: production-files
          path: ./dist

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
```

This workflow performs two key jobs:
1. **Build**: Checks out the code, installs dependencies, builds the project, and uploads the build files as an artifact
2. **Deploy**: Downloads the build artifacts and deploys them to GitHub Pages

Push the workflow file:

```bash
git add .github/workflows/deploy.yml
git commit -m "Add deployment workflow"
git push
```

### 4. Configuring GitHub Actions Permissions

1. Go to your GitHub repository
2. Navigate to **Settings > Actions > General**
3. Scroll down to "Workflow permissions"
4. Select **Read and write permissions**
5. Click **Save**

This is an essential step as the deployment workflow needs to write to your repository to create the `gh-pages` branch.

If your initial deployment failed, go to the Actions tab, find the failed workflow, and click **Re-run failed jobs**.

### 5. Setting up GitHub Pages

After the workflow runs successfully:

1. Go to **Settings > Pages**
2. Set **Source** to "Deploy from a branch"
3. Select the `gh-pages` branch and the `/ (root)` folder
4. Click **Save**

GitHub will now build and deploy your site. You'll see a message with the URL where your site is published (typically `https://YOUR_USERNAME.github.io/react-deploy-demo/`).

### 6. Fixing Asset Links

If your deployed site shows a blank page with missing assets, you need to configure the base path in Vite:

1. Open `vite.config.js`
2. Add the `base` property with your repository name:

```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react()],
  base: '/react-deploy-demo/' // Replace with your repository name
})
```

3. Commit and push this change:

```bash
git add vite.config.js
git commit -m "Fix: Add base path for GitHub Pages deployment"
git push
```

4. Wait for the deployment workflow to complete

## Troubleshooting

- **Blank page after deployment**: Check browser console for errors. Typically, this is due to incorrect asset paths - ensure you've set the correct `base` in vite.config.js.
- **Failed Actions workflow**: Verify that you've enabled the correct permissions for GitHub Actions.
- **404 errors**: Make sure GitHub Pages is set to use the gh-pages branch, and double-check that the branch exists.

## FAQ

- **How to setup a custom domain?**
  Follow the instructions at [Vite Deploy Demo Custom Domain](https://github.com/sitek94/vite-deploy-demo-custom-domain)

- **What if I'm using React Router?**
  For client-side routing to work, you'll need to configure your router to use your repository name as the basename:
  ```jsx
  <BrowserRouter basename="/react-deploy-demo">
    {/* Your routes */}
  </BrowserRouter>
  ```

- **How do I update my deployed site?**
  Simply push changes to your main branch, and the GitHub Actions workflow will automatically build and deploy the updated version.

## Resources

- [Vite Documentation](https://vitejs.dev/)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [React Documentation](https://react.dev/)
- [Deploying Vite Apps to GitHub Pages (Vite Guide)](https://vitejs.dev/guide/static-deploy.html#github-pages)
