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

```bash= 
https://docs.docker.com/desktop/setup/install/windows-install/
```

選擇 Docker Desktop for Windows - x86_64 版本

![Download Docker](/img/download_docker.png)



