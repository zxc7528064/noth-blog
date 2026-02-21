---
title: CRTP 學習
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

在過去的實戰經驗中，內網滲透往往是透過既有漏洞取得初始跳板，例如利用 CVE 進入主機後，直接開始橫向移動與權限提升，最終目標通常也是指向 Active Directory。然而這種方式偏向結果導向 (1day CVE → Webshell → RDP → 關防毒 → 打 MSF → 進 AD → 橫向移動)，雖然能成功達成目標，但對於 AD 架構本身的設計邏輯、權限模型與認證機制，並沒有完整且系統化地梳理過。思考後發現，最大的差異是軍中通常是以目標導向為主，與其只依賴工具與既有手法，不如回頭把 AD 的核心知識與攻擊原理重新整理一次。剛好兩位朋友對 CRTP 有興趣，邀請一起準備，於是決定花時間系統化補齊這一塊。