---
title: "初始化"
date: 2023-02-28T16:47:49+08:00
draft: false
---

### 部署一个自用的技术 blog
1. 使用hugo
2. 使用github pages
3. 使用自定义域名20140620.xyz
4. 域名相关配置, 并且要支持 https
5. 添加两个memu, 一个是 about, 一个是 index
6. 需要配置 github actions
7. 有新的内容时自动部署

### 大致步骤如下：

* 安装 Hugo
首先，需要在本地安装 Hugo。在 macOS 上，可以使用 Homebrew 安装 Hugo

`brew install hugo`

* 创建 Hugo 网站
使用 Hugo 创建网站的步骤如下

`hugo new site 20140620`

* 使用主题
Hugo 支持使用主题来美化网站

`git clone https://github.com/adityatelange/hugo-PaperMod themes/PaperMod --depth=1`

* 20140620/config.toml
```toml
baseURL = 'https://20140620.xyz/'
languageCode = 'zh-cn'
title = '懒宅++'
theme =  "PaperMod"

[Params]
  mainSections = ["posts"]
  email = "gao.jin.gan.gmail"
  intro = true

[taxonomies]
  tag = "tags"
  series = "series"
  category = "categories"

[menu]
  [[menu.main]]
    name = "主页"
    identifier = "index"
    url = "/"
    weight = 1

  [[menu.main]]
    title =  "关于"
    name = "关于"
    identifier = "about"
    url = "/about/"
    weight = 2
```

* 新建一个文章, 填充内容
`hugo new posts/init/init.md`

* 配置 `GitHub Pages`
在 GitHub 上创建一个新的仓库，并将仓库的名称设置为 `20140620.github.io`

* 在 20140620 目录下执行以下命令
`hugo`

* 配置自定义域名
在 `DNS` 服务商处添加一条 `a` 记录，指向 `185.199.108.153`, 并且申请 `ssl` 证书

* 配置 HTTPS
在 `GitHub Pages` 上启用 `HTTPS`，具体步骤可以参考官方文档：https://docs.github.com/en/free-pro-team@latest/github/working-with-github-pages/securing-your-github-pages-site-with-https

* 添加菜单
在主题的配置文件中，可以设置网站的菜单。参照 `config.toml ` 中的内容

* 配置 GitHub Actions

`GitHub Actions` 可以用于自动化构建和部署网站。可以在 20140620 目录下创建一个名为 `.github/workflows/hugo.yml` 的文件，内容如下：

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["master"]

  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.108.0
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass Embedded
        run: sudo snap install dart-sass-embedded
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: recursive
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v3
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          # For maximum backward compatibility with Hugo modules
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          git clone https://github.com/adityatelange/hugo-PaperMod themes/PaperMod --depth=1
          hugo \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v1
```

* 自动部署

每当有新的内容提交到 GitHub 仓库时，`GitHub Actions` 将自动构建和部署网站。可以在 GitHub 仓库的 `Settings` -> `Pages` 中查看网站的地址和状态。

目录结构如下：
```bash
.
├── LICENSE
├── archetypes
│   └── default.md
├── config.toml
├── content
│   ├── about
│   │   └── about.md
│   └── posts
│       ├── gpt
│       │   └── index.md
│       └── init
│           └── init.md
├── static
│   └── img
│       └── 01.jpg
└── themes
    └── PaperMod
        └── ......

51 directories, 131 files

```
