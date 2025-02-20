---
title: "技术博客架构升级：解锁高效写作新体验"
date: 2025-02-07 19:18:42
draft: false
categories: ["日志"]
tags: ["Git", "Hugo"]
cover: false
---

最近我对自己的技术博客架构做了一次重要升级，实现了文章内容与静态网站生成器的完全解耦。这个方案让写作回归纯粹，同时保持了自动化部署的优势。以下是具体的实现方案：

### 🛠️ 方案架构

1. **主仓库**：`flowstone/flowstone.github.io`​  
    主仓库仅保留静态网站生成器的相关配置，果断移除了所有文章内容。如此一来，主仓库更加简洁，专注于网站生成的核心配置工作，为后续的自动化部署奠定坚实基础。
2. **文章仓库**：`flowstone/blog-articles`​  
    文章仓库则全心全意地承担起存放 Markdown 格式博文的重任。它就像一个专属的知识宝库，让每一篇文章都能在自己的 “小天地” 里有序存放，便于管理和维护。
3. **自动化桥梁**：GitHub Actions 工作流  
    每日定时同步文章变更，实现自动构建部署

### ⚙️ 核心工作流配置

添加了同步文章工作流`sync-articles.yml`​

```yml
name: Content Sync Automation

on:
  push:
    branches: [main]
  schedule:
    - cron: '0 0 * * *' # 每天UTC时间0点同步
  workflow_dispatch:

jobs:
  sync-content:
    runs-on: ubuntu-latest
  
    steps:
      - name: Checkout Main Repo
        uses: actions/checkout@v3
        with:
          persist-credentials: false

      - name: Sync Articles Repo
        uses: actions/checkout@v3
        with:
          repository: flowstone/blog-articles
          path: content/posts
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Configure Git
        run: |
          git config --global user.name "Automation Bot"
          git config --global user.email "bot@flowstone.dev"

      # Check if content/post already exists and is a submodule
      - name: Check if submodule exists
        id: check_submodule
        run: |
          if [ -d "content/post/.git" ]; then
            echo "Submodule already exists"
            echo "exists=true" >> $GITHUB_ENV
          else
            echo "Submodule does not exist"
            echo "exists=false" >> $GITHUB_ENV
          fi

      # If content/post is not a submodule, add it as a submodule
      - name: Add submodule if not exists
        if: env.exists == 'false'
        run: |
          git submodule add https://github.com/flowstone/blog-articles content/post
          git submodule update --init --recursive
          git commit -m "Add blog articles repository as a submodule"
          git push origin main

      # Sync articles if there are changes
      - name: Sync articles
        if: env.exists == 'true'
        run: |
          # 拉取远程子模块更新
          git submodule update --remote content/post
          
          # 检查是否有更改，如果有更改则提交
          git diff --exit-code content/post || (
            git add content/post
            git commit -m "Sync articles from new repository"
            git push origin main
          )
```

上述代码展示了核心工作流的详细配置，通过精确的定时任务和细致的变更检测，确保文章同步的准确性和高效性。

如下图，文章的目录已经变成子模块![](https://ueyao.github.io/image-hosting/blog/2025/02/08f834ac884a41b0a99021279c9ec947.png)​

### 🌟 方案优势

##### 关注点分离

在写作过程中，我们只需将全部精力聚焦于 `blog-articles`​ 仓库，无需再为其他无关事务分心。而主仓库则始终保持整洁有序，仅包含静态网站生成器的必要配置，大大提高了管理效率。

##### 智能同步机制

该方案采用每日定时增量同步（UTC 0 点）的方式，就像一个精准的时钟，按时完成文章的同步工作。同时，通过精准的 diff 检测，能够敏锐地捕捉到文章的细微变化，避免了无效构建，节省了大量的时间和资源。此外，还支持手动触发即时同步，满足了特殊情况下的紧急需求。

### 🔧 实现注意事项

为了确保方案的顺利实施，我们需要遵循特定的目录结构规范。

```
blog-articles/
├── 2023
│   ├── tech-article.md
│   └── life-article.md
└── 2024
    └── new-feature.md
```

按照年份对文章进行分类存放，不仅方便查找和管理，还能使整个仓库的结构更加清晰明了。

### 🚀 升级体验

自从采用了这一创新方案，我的写作效率得到了显著提升。现在，我可以通过 VS Code 的 Markdown 插件直接向文章仓库提交修改，再也不用担心会误触静态网站生成器的配置。自动同步机制更是贴心至极，为我节省了手动操作时间，让我真正实现了 “写后即忘” 的理想写作状态，全身心地投入到创作中去。

‍

‍
