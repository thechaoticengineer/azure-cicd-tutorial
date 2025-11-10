# azure-cicd-tutorial

## Overview

This repository demonstrates a simple workflow for deploying applications to Azure using GitHub Actions. While it can be improved and extended, the main goal is to show how easy it is to set up a basic CI/CD pipeline to Azure. Once you have this foundation in place, you can enhance it further based on your needs.

The project itself is not the focus - it's just a Blazor app generated from a template, with mock tests included. These are only here to provide something concrete to build and deploy through the GitHub Actions workflow.

## Prerequisites

- Azure account with an active subscription
- GitHub account
- Basic understanding of Git and GitHub
- Familiarity with Azure Portal

## Setup Steps

## #1 create app service

1. Log in to Azure Portal (https://portal.azure.com)
2. Click "Create a resource" or search for "App Service"
3. Select "Web App" from the marketplace
4. Fill in the required information:
   - **Subscription**: Select your Azure subscription
   - **Resource Group**: Create new or select existing
   - **Name**: Enter a unique name for your app (this will be your URL: `<name>.azurewebsites.net`)
   - **Publish**: Choose "Code" (or Docker Container if needed)
   - **Runtime stack**: Select your application runtime (e.g., .NET, Node.js, Python, Java)
   - **Operating System**: Windows or Linux
   - **Region**: Choose the closest region to your users
5. Select your pricing tier (App Service Plan):
   - **Free (F1)**: For development/testing
   - **Basic**: For small production workloads
   - **Standard/Premium**: For production apps with scaling needs
6. Click "Review + Create" and then "Create"
7. Wait for the deployment to complete (usually takes 1-2 minutes)
8. Once created, navigate to your App Service resource

## #2 configure app service to allow deployment from github actions

1. Navigate to your App Service in Azure Portal
2. In the left menu, go to **Settings** → **Configuration**
3. Navigate to **Settings** → **Basic Auth Publishing Credentials**
4. Enable **SCM Basic Auth** (set to "On")
   - This allows GitHub Actions to authenticate and deploy to your App Service
   - Without this, deployment will fail with authentication errors
5. Click **Save** at the top
6. Go to **Deployment** → **Deployment Center** (optional, to verify deployment settings)
7. Your App Service is now ready to accept deployments from GitHub Actions

## #3 add repository secrets to github with publish profile

1. In Azure Portal, navigate to your App Service
2. Click **Get publish profile** button at the top toolbar
3. A `.PublishSettings` XML file will be downloaded to your computer
4. Open the file in a text editor and copy its entire contents
5. Go to your GitHub repository on GitHub.com
6. Navigate to **Settings** → **Secrets and variables** → **Actions**
7. Click **New repository secret**
8. Create a new secret:
   - **Name**: `AZURE_WEBAPP_PUBLISH_PROFILE` (must match the name used in the workflow file)
   - **Value**: Paste the entire contents of the publish profile XML file
9. Click **Add secret**
10. The secret is now securely stored and ready to use in GitHub Actions

## #4 create github action workflow

This repository already contains a complete CI/CD workflow at `.github/workflows/ci-cd.yml`

### Key components of the workflow:

**Environment Variables** (defined at the top):
- `DOTNET_VERSION`: Specifies .NET version (9.0.x)
- `PROJECT_PATH`: Path to the project file to publish
- `AZURE_WEBAPP_NAME`: Name of your Azure App Service
- `PUBLISH_DIRECTORY`: Where the compiled app will be placed

**Two-stage deployment process:**

1. **Build and Test Job** (`build-and-test`):
   - Checks out the code from the repository
   - Sets up .NET SDK
   - Restores NuGet dependencies
   - Builds the project in Release configuration
   - Runs all tests to ensure code quality
   - Publishes the app (compiles and packages for deployment)
   - Uploads the published app as an artifact for the next job

2. **Deploy Job** (`deploy`):
   - Waits for build-and-test to complete successfully (`needs: build-and-test`)
   - Downloads the published artifact
   - Deploys to Azure App Service using the publish profile secret

**Trigger**: The workflow runs manually via `workflow_dispatch` (can be triggered from the Actions tab)