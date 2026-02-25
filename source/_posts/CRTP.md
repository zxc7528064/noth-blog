---
title: CRTP 學習準備與策略規劃
date: 2026-02-21
categories:
  - 紅隊
tags : 
  - CRTP
---

## 前言
<br>
CRTP（Certified Red Team Professional）是由 Altered Security 推出的紅隊證照，核心聚焦在 Active Directory 內網攻擊技術。

如果用一句話概括：

```bash=
CRTP 是一張專門針對 AD 攻擊路徑與權限濫用技術的證照。
```

在過去的實戰經驗中，內網滲透往往是透過既有漏洞取得初始跳板，例如利用 CVE 進入主機後，直接開始橫向移動與權限提升，最終目標通常也是指向 Active Directory。
整體流程大致為：

```bash=
1day CVE → Webshell → RDP → 關閉防護機制 → 工具利用 → 橫向移動 → 進入 AD
```
這種打法屬於高度結果導向的實戰流程。任務能完成、目標能達成，但回頭檢視會發現，對於 AD 架構本身的設計邏輯、權限模型以及認證機制，理解多半停留在操作層，而非系統層。思考後發現，最大的差異是軍中通常是以目標導向為主，與其只依賴工具與既有手法，不如回頭把 AD 的核心知識與攻擊原理重新整理一次。最近剛好有兩位朋友對 CRTP 感興趣，提議一起準備這張證照，於是決定花時間系統化補齊這一塊。

### CRTP 學習資源與流程

官方網址 : 

```bash= 
https://www.enterprisesecurity.io/
```

成功登入後，可在 Dashboard 中看到 `Attacking and Defending Active Directory Lab - CRTP` 訂閱 Lab。

主要功能區包含：
- `Lab Details`
- `Lab Manual`
- `Certification Exam`
- `Flag Verification`
- `Lab Material`
- `How to use Discord`
- `FAQs`

其中最核心的學習資源集中在 `Lab Manual`，內容涵蓋課程影片、PDF 教材與相關工具包，是整套 CRTP 的知識主體，其他區塊則偏向環境說明、考試資訊與實驗室操作輔助。

![CRTP-Lab Material](/img/Material.png)

基本學習思路 : 

```bash=
Lab Connection Guide
（解決 Lab 連線問題）
        ↓
Course Videos
（快速理解整體架構）
        ↓
Access Lab Material
（Lab 驗證）
        ↓
Walkthrough Videos
（僅在卡關時使用）
```
整體原則為：

```bash=
先建立全局架構，再透過 Lab 驗證理解，Walkthrough 僅作為輔助，而非主要學習來源。
```

### 學習效率優化

#### 雙語字幕輔助流程

CRTP 課程皆為全英文解說、無中文字幕，單純硬聽其實效率不高。與其反覆倒帶，不如把問題工程化處理。因此結合 Whisper 與 ChatGPT，先將影片轉為字幕，再進行翻譯與結構化整理，建立一套雙語輔助學習流程。保留技術術語原文，同時提升理解效率。後續也會將整個流程封裝成工具，放上 GitHub，並考慮導入 CI/CD 或容器化，方便長期維護與團隊使用。

因此建立以下流程：

```bash=
Whisper → 產生 SRT → ChatGPT 翻譯 → 結構化整理 → 建立雙語學習素材
```

##### 環境準備

安裝 Whisper :

```bash=
pip install -U openai-whisper
pip install torch
```

執行 Whisper : 

```bash=
Whisper
```

![Whisper](/img/Whisper.png)

下載 full_build.7z 並將 `ffmpeg.exe` 加入環境變數：

```bash=
https://www.gyan.dev/ffmpeg/builds/
```

![ffmpeg](/img/ffmpeg.png)

批量將所有 `.mp4` 檔案轉成 `.srt` 檔案，並存放在 `Course_output_en` 資料夾中 :

```bash= 
for %f in (*.mp4) do whisper "%f" --model small --language English --output_dir Course_output_en --output_format srt
```

![批量轉 srt 檔案](/img/srt.png)

轉換時間較長，後續可研究更高效處理方式。

![srt_en](/img/srt_en.png)

接著透過自製工具批量翻譯字幕：

```bash=
https://github.com/zxc7528064/SRT-Translator
```

翻譯後生成 srt_zh，並確認時間軸未發生錯位。

![srt_cn](/img/srt_cn.png)

實際匯入後，字幕與時間同步正常，即可正式進入學習階段。

![sucess_srt_zn](/img/sucess_srt_zn.png)

### Active Directory 攻擊體系框架

#### Module 1 
- Introduction to Active Directory and Attack Methodology
- Offensive PowerShell and .NET tradecraft
- Offensive PowerShell and .NET tradecraft

本文先記錄學習前的策略規劃，後續將依照實際進度拆解各模組內容。