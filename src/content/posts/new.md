---
title: 博客搭建
pubDate: 2025-05-29
modDate: 2025-06-02
categories: ["记录"]
description: "搭建过程"
draft: false
pin: true
---
第一篇文章

```frontmatter
---
title: 标题
pubDate: 发布时间
upDate: 更新时间
categories: ["分类"]
description: "简述"
draft: 草稿
pin: 置顶
slug: 链接
---
```

## 搭建过程
### fork仓库

[活版印字](https://github.com/moeyua/astro-theme-typography)

### 安装pnpm

```bash
npm install -g pnpm
```

### 安装依赖

```bash
pnpm install
```

### 启动开发服务器

```bash
pnpm dev
```

### Actions自动构建
在仓库中新建文件`.github\workflows\deploy.yml`，内容如下：

```yaml
name: Deploy Astro to Cloud Server

on:
  push:
    branches:
      - main  # 监控 main 分支的推送事件

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # Step 1: 检出代码
      - name: Checkout code
        uses: actions/checkout@v3

      # Step 2: 设置 Node.js 环境
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: ${{ steps.detect-package-manager.outputs.manager }}
          cache-dependency-path: ${{ env.BUILD_PATH }}/${{ steps.detect-package-manager.outputs.lockfile }}

      # Step 3: 安装 pnpm
      - name: Install pnpm
        run: npm install -g pnpm

      # Step 4: 安装依赖并构建
      - name: Install dependencies and build
        run: |
          pnpm install       # 安装依赖
          pnpm run build     # 构建 Astro 项目

      # Step 5: 推送到 GitHub Pages
      - name: Deploy to GitHub Pages
        uses: JamesIves/github-pages-deploy-action@v4.6.4
        with:
          token: ${{ secrets.TOKEN }}
          repository-name: njuvtk/njuvtk.github.io
          branch: main
          folder: dist/
          commit-message: "${{ github.event.head_commit.message }} Updated By Github Actions"

      # Step 6: 将静态文件ftp上传
      - name: Upload static files to FTP server
        uses: SamKirkland/FTP-Deploy-Action@4.3.1
        with:
          server: ${{ secrets.IP }}  # FTP 服务器地址
          username: ${{ secrets.UN }}  # FTP 用户名
          password: ${{ secrets.PW }}  # FTP 密码
          local-dir: dist/  # 本地静态文件目录
          server-dir: /  # 远程服务器目录
```

### 配置Actions secrets

1. 进入仓库的Settings->Secrets->Actions，点击New repository secret。
2. 输入名称和值，点击Add secret。
3. 重复以上步骤，添加以下secrets：
   - `TOKEN`：GitHub token，用于推送代码到GitHub Pages。
   在GitHub的settings->developer settings->personal access tokens->tokens(classic) -> generate new token (classic) -> 勾选repo，点击generate token。
   - `IP`：FTP服务器地址。
   - `UN`：FTP用户名。
   - `PW`：FTP密码。

### 部署到GitHub Pages
通过action推送到GitHub pages仓库
发现一个问题：样式丢失？
一开始以为是推送的问题，没想到啊！
通过询问Copilot问题得到解决。
```Copilot的回答
非常棒！你的排查结果完全正确：  
如果没有 .nojekyll 文件，GitHub Pages 会用 Jekyll 处理，而 Jekyll 会默认忽略所有下划线开头的目录（如 _astro），导致样式、脚本等静态资源无法被访问，从而页面“完全无样式”。

**解决办法：**  
只需要在仓库根目录添加一个空的 .nojekyll 文件即可。

建议你在每次部署 Astro、Next.js、Vite 等生成 _astro 或 _next 目录的静态站点到 GitHub Pages 时都包含这个文件。

这样 GitHub Pages 就会正常托管所有静态资源，页面样式和交互也会全部恢复正常！

如需自动化部署脚本或有其他 Pages 相关问题，欢迎随时提问。
```
所以只需要在public文件夹下新建一个空文件.nojekyll就解决啦！`/public/.nojekyll`