# Deploy Docker Image to Heroku

[![GitHub](https://img.shields.io/github/license/misaya98/docker2heroku)](LICENSE)
[![GitHub release](https://img.shields.io/github/release/misaya98/docker2heroku.svg)](https://github.com/misaya98/docker2heroku/releases/)
[![GitHub marketplace](https://img.shields.io/badge/marketplace-docker2heroku-blue?logo=github)](https://github.com/marketplace/actions/deploy-docker-image-to-heroku)

A powerful and flexible GitHub Action that automates the complete process of deploying any Docker image to Heroku.

This action pulls a Docker image from Docker Hub (or other container registries), optionally creates a new Heroku app, configures environment variables, and completes the deployment.

## ✨ Features

- **🚀 One-Click Deployment**: Automates the entire process of pulling, retagging, pushing to Heroku, and releasing Docker images
- **📱 Create New Apps**: Supports dynamic creation of new Heroku applications during deployment
- **🎯 Smart Naming**: Automatically generates unique app names based on Docker image names when no name is specified
- **⚙️ Environment Configuration**: Supports dynamic configuration of multiple Heroku environment variables
- **🔐 Private Image Support**: Supports Docker Hub login for pulling private images or avoiding rate limits

## 🚀 Quick Start

Add this action to your workflow file (`.github/workflows/deploy.yml`):

```yaml
name: Deploy to Heroku

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy App to Heroku
        uses: misaya98/docker2heroku@v1
        with:
          # Required: Your Heroku API Key
          heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
          
          # Required: Docker image to deploy
          docker_image: 'lobehub/lobe-chat:latest'
          
          # Optional: Specify Heroku app name
          heroku_app_name: 'my-awesome-app'
          
          # Optional: Set to 'true' to create new app
          create_new_app: 'false'
          
          # Optional: Set environment variables
          heroku_config_vars: |
            NODE_ENV=production
            NEXTAUTH_URL=https://my-awesome-app.herokuapp.com
            NEXTAUTH_SECRET=${{ secrets.MY_NEXTAUTH_SECRET }}
            
          # Optional: For private images or rate limit avoidance
          dockerhub_username: ${{ secrets.DOCKERHUB_USERNAME }}
          dockerhub_token: ${{ secrets.DOCKERHUB_TOKEN }}
```

## 📋 Input Parameters

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `heroku_api_key` | Your Heroku API Key (must be stored in GitHub Secrets) | ✅ | N/A |
| `docker_image` | Full Docker image name and tag (e.g., `user/repo:latest`) | ✅ | N/A |
| `heroku_app_name` | Name of the Heroku app (auto-generated if empty) | ❌ | `''` |
| `create_new_app` | Set to `'true'` to create a new Heroku app | ❌ | `'false'` |
| `heroku_config_vars` | Heroku environment variables (one per line, format: `KEY=VALUE`) | ❌ | `''` |
| `dockerhub_username` | Docker Hub username (for private images or rate limits) | ❌ | `''` |
| `dockerhub_token` | Docker Hub access token (must be stored in GitHub Secrets) | ❌ | `''` |

## 🔑 Required Secrets

To use this action, you need to configure the following secrets in your repository:

**Settings** → **Secrets and variables** → **Actions** → **New repository secret**

- **`HEROKU_API_KEY`**: Your Heroku account API key. You can find it in your [Heroku Account Settings](https://dashboard.heroku.com/account).

### Optional Secrets (for private Docker images)

- **`DOCKERHUB_USERNAME`**: Your Docker Hub username
- **`DOCKERHUB_TOKEN`**: Your Docker Hub access token

## 📖 Usage Examples

### 1. Deploy to Existing App

```yaml
- uses: misaya98/docker2heroku@v1
  with:
    heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
    docker_image: 'my-user/my-app:1.2.0'
    heroku_app_name: 'my-existing-heroku-app'
```

### 2. Create New App with Specific Name

```yaml
- uses: misaya98/docker2heroku@v1
  with:
    heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
    docker_image: 'my-user/my-app:latest'
    heroku_app_name: 'my-brand-new-app'
    create_new_app: 'true'
```

### 3. Create New App with Auto-Generated Name

```yaml
- uses: misaya98/docker2heroku@v1
  with:
    heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
    docker_image: 'my-user/my-app:latest'
    create_new_app: 'true'
    # heroku_app_name is left empty for auto-generation
```

### 4. Deploy with Environment Variables

```yaml
- uses: misaya98/docker2heroku@v1
  with:
    heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
    docker_image: 'my-user/my-app:latest'
    heroku_app_name: 'my-app'
    heroku_config_vars: |
      NODE_ENV=production
      DATABASE_URL=${{ secrets.DATABASE_URL }}
      API_KEY=${{ secrets.API_KEY }}
```

### 5. Deploy Private Docker Image

```yaml
- uses: misaya98/docker2heroku@v1
  with:
    heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
    docker_image: 'my-private-registry/my-app:latest'
    heroku_app_name: 'my-app'
    dockerhub_username: ${{ secrets.DOCKERHUB_USERNAME }}
    dockerhub_token: ${{ secrets.DOCKERHUB_TOKEN }}
```

## 🔄 How It Works

1. **Determine App Name**: Uses provided name or generates one from the Docker image name
2. **Login to Heroku**: Authenticates with Heroku Container Registry
3. **Login to Docker Hub** (optional): Authenticates for private images or rate limit avoidance
4. **Install Heroku CLI**: Downloads and installs the latest Heroku CLI
5. **Create App** (conditional): Creates a new Heroku app if requested
6. **Pull Image**: Downloads the specified Docker image
7. **Retag Image**: Tags the image for Heroku's container registry format
8. **Push to Heroku**: Uploads the image to Heroku Container Registry
9. **Set Config Variables**: Configures environment variables in Heroku
10. **Release**: Deploys the new image to your Heroku app

## 🐛 Troubleshooting

### Common Issues

**Authentication Error**
- Ensure `HEROKU_API_KEY` is correctly set in your repository secrets
- Verify your Heroku API key is valid and not expired

**App Already Exists**
- If `create_new_app` is `true` but app name already exists, the action will fail
- Use a different app name or set `create_new_app` to `false`

**Docker Image Not Found**
- Verify the Docker image name and tag are correct
- For private images, ensure Docker Hub credentials are properly configured

**Rate Limiting**
- Docker Hub has rate limits for anonymous pulls
- Configure `dockerhub_username` and `dockerhub_token` to avoid limits

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request. For major changes, please open an issue first to discuss what you would like to change.

## 📝 License

This project is licensed under the [MIT License](LICENSE).
