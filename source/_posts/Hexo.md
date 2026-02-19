---
title: Hexo 部署流程
date: 2026-02-19
categories:
  - DevOps
tags : 
  - DevOps
  - CI/CD
  - Hexo
---

## 前言
<br>
之前對於自架 Blog 並沒有太多實際經驗。趁著過年期間有空，決定從零開始搭建一個屬於自己的網站，也順便把整個部署流程完整走一遍。

在實作過程中，除了前端主題設定與結構調整外，也接觸到 CI/CD 自動化部署，包括 GitHub Actions、靜態檔生成與版本控制等流程。過去做紅隊時，思考重心多半放在攻擊面與弱點利用上，較少從「系統是如何被建構與部署」的角度去理解。這次實際操作後，才真正體會到工程化流程對穩定性與可維護性的價值。

自己並非本科出身，早期更多是從實戰與攻防經驗中累積能力。但隨著紅隊工作做得越久，越能理解：單純會打並不夠，只有理解開發流程、部署架構與自動化思維，才能真正看清整個系統的全貌。

## 如何安裝

### 安裝 Node.js

前往官網下載 Windows (.msi) 安裝包：

```bash=
https://nodejs.org/zh-tw/download
```

安裝完成後確認版本：

```bash=
node -v
npm -v
```

![check version](/img/image.png)

### 安裝 Hexo CLI（Hexo 的指令工具，用來建立與管理網站）

一個基於 Node.js 的靜態網站生成框架，
透過 Markdown + 模板引擎，在建置階段產生靜態網站。

```bash=
npm install -g hexo-cli
hexo -v
```

![hexo cli](/img/image-1.png)

建立新專案

```bash=
cd %USERPROFILE%\Desktop
hexo init noth-blog
cd noth-blog
npm install
```

![alt text](/img/image-2.png)

### 安裝 Butterfly 主題

```bash=
npm install hexo-theme-butterfly --save
```

打開 _config.yml，修改：
_config.yml 是 Hexo 的全域設定檔，用來控制網站標題、主題、URL 等基本配置。

```bash=
theme: butterfly
```

### 本機測試

```bash=
hexo clean   # 清除快取與舊生成檔案
hexo g       # 產生靜態網站
hexo s       # 啟動本機伺服器
```

打開：

```bash=
http://localhost:4000
```

看到畫面即代表成功

### Hexo → GitHub → Actions → Pages 架構

```bash=
你的電腦 (Hexo 原始碼)
        │
        ▼ git push main
GitHub main 分支 (存原始碼)
        │
        ▼
GitHub Actions 自動執行
 1. npm install
 2. hexo generate
 3. 產生 public/
 4. push 到 gh-pages
        │
        ▼
gh-pages 分支 (靜態網站檔)
        │
        ▼
GitHub Pages 對外顯示網站
```

重點理解：
main = 原始碼
gh-pages = 靜態網站
Actions = 自動化機器人

### Git 基本設定與推送流程

Git 基本設定與推送流程

```bash=
git config --global user.name "yourname"
git config --global user.email "yourname@gmail.com"
```

初次推送

```bash=
git add .
git commit -m "initial hexo setup"
git branch -M main
git push --set-upstream origin main
```

日常發文流程

```bash=
git add .
git commit -m "add new post"
git push
```

### CI/CD Workflow 設定

GitHub Pages 不會幫你執行 hexo generate
👉 GitHub 只會部署「已存在的靜態檔」
👉 main 分支如果只有原始碼，不會自動生成
解決方式：
新增 .github/workflows/deploy.yml

```bash=
name: Deploy Hexo to GitHub Pages

on:
  push:
    branches:
      - main

permissions:
  contents: write

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 18

      - run: npm install
      - run: npx hexo generate

      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

### GitHub Pages 設定步驟

```bash=
Repository → Settings → Pages
Source: Deploy from branch
Branch: gh-pages
Folder: / (root)
```

### 常見錯誤與排錯