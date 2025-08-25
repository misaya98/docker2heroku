# Docker to Heroku GitHub Action

A GitHub Action that pulls a Docker image from a registry and deploys it to Heroku using Heroku's Container Registry. This action can optionally create new Heroku apps and set configuration variables.

## Features

- 🐳 **Docker Integration**: Pull images from Docker Hub or other registries
- 🚀 **Heroku Deployment**: Deploy directly to Heroku using Container Registry
- ⚡ **App Creation**: Optionally create new Heroku apps on-the-fly
- 🔧 **Config Management**: Set Heroku config variables during deployment
- 🔐 **Private Registry Support**: Works with private Docker images
- 📝 **Flexible Naming**: Auto-generate app names or use custom ones

## Quick Start

```yaml
name: Deploy to Heroku
on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: misaya98/docker2heroku@main
        with:
          heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
          docker_image: your-username/your-app:latest
          heroku_app_name: my-heroku-app
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `heroku_api_key` | Your Heroku API Key | ✅ Yes | - |
| `docker_image` | The full Docker image name and tag (e.g., `user/repo:latest`) | ✅ Yes | - |
| `heroku_app_name` | The name of the Heroku app | ❌ No | Auto-generated from image name |
| `create_new_app` | Set to `"true"` to create a new Heroku app | ❌ No | `false` |
| `heroku_config_vars` | Config variables (one per line, e.g., `KEY=VALUE`) | ❌ No | - |
| `dockerhub_username` | Docker Hub username (for private images) | ❌ No | - |
| `dockerhub_token` | Docker Hub token (for private images) | ❌ No | - |

## Prerequisites

1. **Heroku API Key**: Get your API key from [Heroku Account Settings](https://dashboard.heroku.com/account)
2. **Docker Image**: Your application must be containerized and available in a registry
3. **GitHub Secrets**: Store your Heroku API key as `HEROKU_API_KEY` in repository secrets

### Setting up Heroku API Key

1. Go to [Heroku Dashboard → Account Settings](https://dashboard.heroku.com/account)
2. Scroll down to "API Key" section
3. Click "Reveal" to show your API key
4. Copy the key and add it to your GitHub repository secrets as `HEROKU_API_KEY`

## Example Workflows

### Deploy to Existing App

```yaml
name: Deploy to Existing Heroku App
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: misaya98/docker2heroku@main
        with:
          heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
          docker_image: myusername/myapp:latest
          heroku_app_name: my-existing-app
```

### Create New App and Deploy

```yaml
name: Create New App and Deploy
on:
  workflow_dispatch:
    inputs:
      app_name:
        description: 'Heroku app name (optional)'
        required: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: misaya98/docker2heroku@main
        with:
          heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
          docker_image: myusername/myapp:latest
          heroku_app_name: ${{ github.event.inputs.app_name }}
          create_new_app: "true"
```

### Deploy with Config Variables

```yaml
name: Deploy with Environment Variables
on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: misaya98/docker2heroku@main
        with:
          heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
          docker_image: myusername/myapp:latest
          heroku_app_name: my-app
          heroku_config_vars: |
            NODE_ENV=production
            DATABASE_URL=${{ secrets.DATABASE_URL }}
            API_KEY=${{ secrets.API_KEY }}
```

### Deploy Private Docker Image

```yaml
name: Deploy Private Image
on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: misaya98/docker2heroku@main
        with:
          heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
          docker_image: myusername/private-app:latest
          heroku_app_name: my-private-app
          dockerhub_username: ${{ secrets.DOCKERHUB_USERNAME }}
          dockerhub_token: ${{ secrets.DOCKERHUB_TOKEN }}
```

### Manual Deployment Workflow

```yaml
name: Manual Deploy to Heroku
on:
  workflow_dispatch:
    inputs:
      docker_image:
        description: 'Docker image (e.g., user/repo:tag)'
        required: true
        type: string
      heroku_app_name:
        description: 'Heroku app name (optional)'
        required: false
        type: string
      create_new_app:
        description: 'Create new Heroku app'
        required: false
        type: boolean
        default: false
      config_vars:
        description: 'Config variables (KEY=VALUE, one per line)'
        required: false
        type: string

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: misaya98/docker2heroku@main
        with:
          heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
          docker_image: ${{ github.event.inputs.docker_image }}
          heroku_app_name: ${{ github.event.inputs.heroku_app_name }}
          create_new_app: ${{ github.event.inputs.create_new_app }}
          heroku_config_vars: ${{ github.event.inputs.config_vars }}
```

## How It Works

1. **Authentication**: Logs into Heroku Container Registry using your API key
2. **Docker Hub Login**: Optionally logs into Docker Hub for private images
3. **App Management**: Creates a new Heroku app if requested, or uses existing one
4. **Image Processing**: Pulls the specified Docker image and retags it for Heroku
5. **Deployment**: Pushes the image to Heroku Container Registry
6. **Configuration**: Sets any specified config variables
7. **Release**: Releases the new image to your Heroku app

## App Naming

When `create_new_app` is `true` and no `heroku_app_name` is provided:
- The action extracts the image name (e.g., `myapp` from `user/myapp:latest`)
- Adds a random 6-character suffix to ensure uniqueness
- Example: `myapp-a1b2c3`

## Troubleshooting

### Common Issues

**Error: "Invalid input. You must provide an 'Heroku app name' if you are not creating a new app."**
- Solution: Either set `create_new_app: "true"` or provide a `heroku_app_name`

**Error: "docker: unauthorized"**
- Solution: Check your Docker Hub credentials if using a private image

**Error: "App name is already taken"**
- Solution: Choose a different `heroku_app_name` or let the action auto-generate one

**Error: "Invalid credentials"**
- Solution: Verify your `HEROKU_API_KEY` secret is correct

### Docker Image Requirements

Your Docker image must:
- Expose a port via `EXPOSE` directive in Dockerfile
- Listen on the port specified by Heroku's `$PORT` environment variable
- Be publicly accessible or provide Docker Hub credentials for private images

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/amazing-feature`
3. Commit your changes: `git commit -m 'Add amazing feature'`
4. Push to the branch: `git push origin feature/amazing-feature`
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

If you encounter any issues or have questions:
1. Check the [troubleshooting section](#troubleshooting)
2. Review [Heroku's container deployment documentation](https://devcenter.heroku.com/articles/container-registry-and-runtime)
3. Open an issue in this repository

---

**Made with ❤️ for the developer community**