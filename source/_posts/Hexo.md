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

自己並非本科出身，早期更多是從實戰與攻防經驗中累積能力。但隨著紅隊工作做得越久，越能理解：

```bash=
單純會打並不夠，只有理解開發流程、部署架構與自動化思維，才能真正看清整個系統的全貌。
```

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

一個基於 Node.js 的靜態網站生成框架，透過 Markdown + 模板引擎，在建置階段產生靜態網站。

核心流程

```bash= 
Markdown (.md)
        ↓
hexo generate
        ↓
產生 public/ 靜態 HTML
```

重點理解：
.md 是原始內容
public/ 是 build 產物
Hexo 不提供動態後端
最終輸出的是純靜態網站

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

Hexo 在 build 時使用 butterfly 當作模板

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

看到畫面即代表成功。

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
public/ = build 產物
Actions = 自動化機器人

### Git 基本設定與推送流程

```bash=
Working Directory
(實際檔案)
   │
   │ git add
   ▼
Staging Area
(暫存區)
   │
   │ git commit
   ▼
Local Repository
(main 分支)
   │
   │ git push
   ▼
Remote Repository
(GitHub)
```

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

GitHub Pages 不會幫你執行 hexo generate，因為 GitHub Pages 只是靜態檔案伺服器，不負責 build 過程。
👉 GitHub 只會部署「已存在的靜態檔」
👉 main 分支如果只有原始碼，不會自動生成
解決方式：
需要一個「自動建置機器人」，新增 .github/workflows/deploy.yml 檔案

```bash=
# Workflow 名稱（顯示在 GitHub Actions 頁面）
name: Deploy Hexo to GitHub Pages

# 觸發條件
on:
  push:
    branches:
      - main   # 當 push 到 main 分支時觸發

# 設定 workflow 權限
permissions:
  contents: write   # 允許 workflow push 到 gh-pages 分支

jobs:
  deploy:   # 工作名稱（可以自訂）
    runs-on: ubuntu-latest   # 使用 GitHub 提供的 Ubuntu 環境執行

    steps:
      # 1️⃣ 把你的 repo 原始碼抓下來
      - uses: actions/checkout@v4

      # 2️⃣ 安裝 Node.js 環境
      - uses: actions/setup-node@v4
        with:
          node-version: 18   # 指定 Node.js 版本

      # 3️⃣ 安裝專案依賴（package.json 裡的套件）
      - run: npm install

      # 4️⃣ 產生靜態網站（生成 public/ 資料夾）
      - run: npx hexo generate

      # 5️⃣ 把 public/ 裡的靜態檔 push 到 gh-pages 分支
      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}   # 使用 GitHub 內建 token
          publish_dir: ./public   # 指定要部署的資料夾
```

CI/CD 的本質是：每次程式碼變動，自動觸發建置流程，產生一致且可重現的產物。

### GitHub Pages 設定步驟

讀取一個 branch 裡已經存在的 HTML 檔案。

```bash=
Repository → Settings → Pages
Source: Deploy from branch
Branch: gh-pages
Folder: / (root)
```

### Cloudflare 設定

整體流程如下：

```bash=
購買網域（Spaceship）
        ↓
加入 Cloudflare
        ↓
取得 Nameserver
        ↓
回 Spaceship 修改 NS（Nameserver）
        ↓
Cloudflare 接管成功（Active）
```

#### 購買網域

首先到 **Spaceship** 購買網域，選擇名稱簡單、價格合理的即可。

官方網站：
```bash=
https://www.spaceship.com/ 
```

完成購買後，即可在後台看到你的網域（例如：noth.tech）。

![SpaceShip後台](/img/SpaceShip-1.png)

#### 將網域加入 Cloudflare

將 **noth.tech** 域名加進 Cloudflare。

![Cloudflare](/img/Cloudflare.png)

選擇 free 方案

![Cloudflare-2](/img/Cloudflare-2.png)

在設定流程中，Cloudflare 會分配兩組 Nameserver，例如：

```bash=
amy.ns.cloudflare.com
nile.ns.cloudflare.com
```

![Name-Server](/img/NameServer-2.png)

#### 修改 Nameserver（Spaceship）

- 進入域名管理器
- 選擇你的網域（例如：noth.tech）
- 找到 Nameserver 設定
- 選擇「自訂 Nameserver」
- 將原本的 NS 改為 Cloudflare 提供的

![Change-NameServer](/img/NameServer.png)

修改完成後，需等待 DNS 更新（約 5～30 分鐘）；當 Cloudflare 顯示網域狀態為 **Active** 代表已成功接管該網域。

![NS-Active](/img/NS-Active.png)

#### 設定 DNS

到 Cloudflare → DNS → Add record

新增一筆：

```bash=
Type: CNAME
Name: blog
Target: zxc7528064.github.io
Proxy: DNS only（關橘雲）
```

![CNAME](/img/CNAME.png)

重點：一定要灰雲（DNS only）
如果開啟橘雲（Proxied），可能會導致：
- GitHub Pages 無法驗證 Custom Domain
- DNS check failed
- HTTPS 無法啟用
- 網站可能出現 404

GitHub Pages 設定 Custom Domain

進入：

```bash=
repo → Settings → Pages
```

Custom domain 填入 **blog.noth.tech** 

![Custom Domain](/img/Custom-Domain.png)

Hexo 設定，修改 **_config.yml**

```bash=
url: https://blog.noth.tech
root: /
```

![fix-Hexopath](/img/fix-Hexopath.png)

如果設定錯誤（例如 /noth-blog），會導致：
- CSS / JS 路徑錯誤
- 頁面樣式異常
- 資源 404

CI/CD（GitHub Actions）

確保 workflow 一定要有 **clean** 步驟

```bash=
- run: |
    npx hexo clean
    npx hexo generate
```

若缺少 hexo clean，可能出現：
- 舊 CSS / JS 殘留
- Mixed Content（HTTP 資源）
- 修改設定後網站未更新

這類問題大多不是單一設定錯誤，而是 DNS、CI/CD 與靜態資源路徑之間的連動問題。

### 部署踩坑紀錄

把 Hexo Blog 部署到 GitHub Pages，並搭配 Cloudflare 綁定自訂網域時，實際踩了幾個關鍵坑，這裡整理最重要的幾點。

#### DNS 設定（Cloudflare）

重點：關橘雲（Proxy）

如果開啟 Proxy（橘雲），會出現：
- GitHub Pages 無法驗證 Custom Domain
- DNS check failed
- HTTPS 無法啟用
- 甚至出現 404

原因： Cloudflare Proxy 會攔截流量，導致 GitHub 無法直接驗證網域

#### CI/CD（GitHub Actions）

部署流程：

```bash=
- run: |
    npx hexo clean
    npx hexo generate

- uses: peaceiris/actions-gh-pages@v3
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: ./public
```

重點：一定要加 **hexo clean**

如果缺少 **hexo clean**，會出現：
- 舊 CSS / JS 殘留
- 修改設定後網站沒有變化
- 錯誤的路徑持續存在

原因：public/ 會殘留舊 build

#### CNAME 被覆蓋

每次 git push 後 Custom Domain 消失，GitHub Pages 設定被清掉

Custom Domain 並不是 GitHub 記住的設定，而是透過 **gh-pages** 分支中的 **CNAME** 檔來維持

當使用 GitHub Actions 部署時：
→ 每次都會覆蓋整個 gh-pages 分支  
→ 如果沒有包含 CNAME，設定就會被清除

在專案中新增：

```bash=
source/CNAME
```

內容：

```bash=
blog.noth.tech
```

## 結論

這次的部署過程，其實並不是單一設定錯誤，而是多個系統之間的交互影響：

- Hexo 負責靜態資源生成（build）
- GitHub Actions 負責自動化部署（deploy）
- GitHub Pages 負責網站託管（hosting）
- Cloudflare 負責 DNS 與流量轉發（DNS/CDN）

在這樣的架構下，任何一層設定錯誤，都可能導致整體行為異常。

實際踩過的問題，本質上可以歸納為三種類型：

1. **資源路徑問題（Hexo root 設定）**
2. **部署覆蓋問題（gh-pages 與 CNAME）**
3. **請求路徑改變（Cloudflare Proxy 導致驗證失敗）**

在這次實作後，更清楚理解：

> 部署不只是把網站丟上去，而是理解 build、deploy、hosting 與 DNS 如何協同運作。

這也是從「會使用工具」走向「理解系統」的一個過程。