Heroku 容器部署 Action
这是一个功能强大且灵活的 GitHub Action，它可以自动化将任何 Docker 镜像部署到 Heroku 的完整流程。

此 Action 会从 Docker Hub（或其他容器注册中心）拉取一个 Docker 镜像，（可选地）在 Heroku 上创建一个新应用，设置环境变量，并完成部署。

功能特性
一键部署: 将 Docker 镜像拉取、重打标签、推送到 Heroku 并发布的全过程自动化。

创建新应用: 支持在部署时动态创建一个新的 Heroku 应用。

智能命名: 在创建新应用时，如果未指定名称，可根据 Docker 镜像名自动生成唯一的应用名。

环境变量配置: 支持在部署时动态设置多个 Heroku 环境变量。

支持私有镜像: 支持登录到 Docker Hub 以拉取私有镜像或避免速率限制。

使用方法
在您的工作流文件中，使用 misaya98/heroku-deploy-action@v1 (请替换为您自己的仓库和版本) 来调用此 Action。

name: Deploy to Heroku

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy App to Heroku
        uses: misaya98/heroku-deploy-action@v1
        with:
          # 必需：您的 Heroku API 密钥
          heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
          
          # 必需：您要部署的 Docker 镜像
          docker_image: 'lobehub/lobe-chat:latest'
          
          # 可选：指定 Heroku 应用名称
          heroku_app_name: 'my-awesome-app'
          
          # 可选：设置为 'true' 以创建新应用
          create_new_app: 'false'
          
          # 可选：设置环境变量
          heroku_config_vars: |
            NODE_ENV=production
            NEXTAUTH_URL=https://my-awesome-app.herokuapp.com
            NEXTAUTH_SECRET=${{ secrets.MY_NEXTAUTH_SECRET }}
            
          # 可选：用于私有镜像或避免速率限制
          dockerhub_username: ${{ secrets.DOCKERHUB_USERNAME }}
          dockerhub_token: ${{ secrets.DOCKERHUB_TOKEN }}

输入参数 (Inputs)
参数

描述

是否必需

默认值

heroku_api_key

您的 Heroku API 密钥。必须存放在 GitHub Secrets 中。

true

N/A

docker_image

完整的 Docker 镜像名称和标签 (例如 user/repo:latest)。

true

N/A

heroku_app_name

Heroku 应用的名称。如果留空，将根据 docker_image 自动生成。

false

''

create_new_app

设置为 'true' 以创建一个新的 Heroku 应用。

false

'false'

heroku_config_vars

您想设置的 Heroku 环境变量，每行一个，格式为 KEY=VALUE。

false

''

dockerhub_username

您的 Docker Hub 用户名。用于拉取私有镜像或避免速率限制。

false

''

dockerhub_token

您的 Docker Hub 访问令牌。必须存放在 GitHub Secrets 中。

false

''

必需的 Secrets
为了让此 Action 正常工作，您需要在您的仓库中设置以下 Secrets：
Settings -> Secrets and variables -> Actions -> New repository secret

HEROKU_API_KEY: 您的 Heroku 账户 API 密钥。您可以在 Heroku 的 账户设置 页面找到它。

示例场景
1. 部署到一个现有的应用
- uses: misaya98/heroku-deploy-action@v1
  with:
    heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
    docker_image: 'my-user/my-app:1.2.0'
    heroku_app_name: 'my-existing-heroku-app'

2. 创建一个指定名称的新应用并部署
- uses: misaya98/heroku-deploy-action@v1
  with:
    heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
    docker_image: 'my-user/my-app:latest'
    heroku_app_name: 'my-brand-new-app'
    create_new_app: 'true'

3. 创建一个自动命名的新应用并部署
- uses: misaya98/heroku-deploy-action@v1
  with:
    heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
    docker_image: 'my-user/my-app:latest'
    create_new_app: 'true' # heroku_app_name 留空

许可证
本项目根据 MIT License 授权。
