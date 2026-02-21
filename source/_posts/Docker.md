---
title: Docker 容器基礎概念與安裝
date: 2026-02-20
categories:
  - DevOps
tags : 
  - DevOps
  - Docker
---

## 前言
<br>
Docker 幾乎已經成為現代開發流程中不可或缺的一環。與其只是停留在會使用指令的層次，不如真正理解它背後的設計思維與應用價值。這次打算從基礎概念開始重新梳理，並思考如何把容器化技術融入自己的工作與研究場景。
在實務上，不論是搭建漏洞實驗環境、重現特定攻擊場景，或是建立報告產出流程，都會遇到環境不一致的問題。團隊成員使用不同作業系統、不同套件版本，往往讓原本簡單的流程變得複雜。
透過 Docker 可以把所有依賴與設定封裝進 image 映像檔中，讓環境具備可重現性與一致性，減少因系統差異而產生的錯誤與時間成本。

### 虛擬機 vs 容器

| 特性       | 虛擬機             | 容器                   |
|------------|--------------------|------------------------|
| Kernel     | 獨立 Kernel        | 共用宿主機 Kernel      |
| 啟動時間   | 數分鐘             | 數秒                   |
| 資源消耗   | 高                 | 低                     |
| 隔離性     | 完全隔離           | 程序級隔離             |
| 可移植性   | 較差               | 極佳                   |
| 映像檔大小 | GB 級別            | MB 級別                |

### 如何安裝

官方文件：

```bash= 
https://docs.docker.com/desktop/setup/install/windows-install/
```

選擇 Docker Desktop for Windows - x86_64 版本。

![Download Docker](/img/download_docker.png)

啟動完成即可以看到這台電腦現在的容器 (Container) 列表、映像 (Image) 列表、控制台、網路等等圖形化介面。

![圖形化 Docker](/img/docker-1.png)

### 驗證安裝

想確認更完整，可以在終端機執行 :

```bash=
docker --version
docker info
```

![docker 資訊](/img/docker-info.png)

若指令能正常顯示資訊，代表 Docker Engine 已成功啟動，執行 docker run 命令 :

```bash=
docker run hello-world
```

若本機尚未存在 hello-world image，Docker 會自動從 Docker Hub 下載官方映像檔，建立 container，並執行其預設程式。成功執行後，畫面會顯示測試訊息，代表整個流程（下載 → 建立 → 執行）皆運作正常。

![驗證安裝](/img/docker-check.png)

圖形化可以看的出來已經成功從 Docker Hub 下載官方 image。

![下載 Docker Hub image](/img/docker-hub-image.png)

### Docker 基礎架構

![Docker 架構](/img/docker-build.png)

#### Client (發送命令的人)

使用者操作 Docker 的地方，包含：

```bash=
docker run
docker build
docker pull
```

這些指令不是直接操作容器，而是發送請求給中間那層的 Docker Daemon。

#### Docker Host (核心運作區)

Docker Daemon 是 Docker 架構中的核心服務（即 Docker Engine）。它在系統背景持續運行，負責接收並處理來自 Docker Client 的指令請求。

當使用者在終端機執行 `docker run`、`docker build` 或 `docker pull` 等指令時，這些指令並不會直接操作容器或映像檔，而是透過 API 傳送給 Docker Daemon。真正建立、管理與執行容器相關資源的，是由 Daemon 負責完成。

Docker Daemon 的主要職責包括：
- 管理 Image（下載、儲存、刪除）
- 建立與啟動 Container
- 管理網路與 Volume
- 分配系統資源（CPU、記憶體）
- 與 Registry 溝通（例如 Docker Hub）

因此，可以將 Docker Client 視為操作介面，而 Docker Daemon 則是實際執行與管理容器運作的核心引擎。

Images 是容器的基礎模板，用來定義應用程式執行所需的環境與依賴。
例如常見的 `nginx`、`redis`、`python`、`ubuntu`，或是你自行透過 Dockerfile 建立的映像檔，都屬於 Image。

Image 本身是唯讀的（read-only），它包含：
- 作業系統基礎層
- 套件與函式庫
- 應用程式程式碼
- 啟動指令（entrypoint / cmd）

Container 是由 Image 建立出來的執行實例（runtime instance），當你使用某個 Image 啟動時，Docker 會在該唯讀 Image 之上建立一層可寫入層（writable layer），並讓程式在其中執行。

同一個 Image 可以建立多個 Container：
- 一個 nginx image → 可以啟動多個 nginx container
- 一個 python image → 可以同時跑不同的應用

簡單來說：
Image 定義環境
Container 執行環境(執行程式的地方)

#### Registry (倉庫)

Registry 是 image 的來源。

例如：

Docker Hub

私有 registry

裡面存放各種官方或第三方 image：

NGINX

Ubuntu

PostgreSQL

當你執行：

docker pull nginx

其實就是：

Client → Daemon → Registry → 把 image 拉回本機

理解 Docker 的核心結構，需掌握三個概念：
- Image：容器的模板（唯讀）
- Container：Image 的執行實例
- Docker Engine：負責建立與管理容器的核心服務

基本流程如下：

```bash=
Image → Container → Process
```

docker run 的本質，就是從 Image 建立 Container，並在其中執行指定程式。