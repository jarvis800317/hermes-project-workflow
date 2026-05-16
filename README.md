# Hermes Agent 專案工作流程設定（Mac mini + Hermes Agent 環境）

> 版本：v1.1
> 更新日期：2026-05-16
> 適用環境：macOS + Hermes Agent + Google Drive

---

## 概述

本流程讓 Hermes Agent 支援「開新專案 → 實作 → 收工 → 繼續」的延續性工作節奏，適合需要跨 session 接續的長期專案。

**核心價值**：
- 節省 context tokens（不用每次重新交代背景）
- 結構化記錄進度，防止遺漏
- **四**方自動同步（GDrive + GitHub + Obsidian + **Firebase**）

---

## 工作架構

### 四個家

| 平台 | 路徑 | 用途 |
|------|------|------|
| 📋 GDrive | `~/我的雲端硬碟/Jarvis專案/` | 工作桌，自動跨設備同步 |
| 🐙 GitHub | `jarvis800317/Jarvis專案` | 程式碼的家，備份 + 公開 |
| 📘 Obsidian | `~/我的雲端硬碟/2ndBrain/工作日誌/` | 工作室，進度日誌 |
| 🔥 Firebase | `專案內 firebase.json` | 視覺化即時展示（進度追蹤），僅專案有 Firebase 元件時觸發 |

### 資料夾結構

```
Jarvis專案/
├── projects/                    # 所有專案放在這
│   └── {專案名}/
│       ├── CLAUDE.md            # 專案藍圖
│       ├── .gitignore
│       └── ...                  # 專案檔案
├── README.md
└── .gitignore

~/我的雲端硬碟/2ndBrain/工作日誌/
└── {專案名}.md                  # 各專案的工作日誌
```

---

## 三個核心技能

### 1. 開新專案（init-project）

**觸發**：說「開新專案 XXX」或「新建專案」

**做的事**：
1. 在 `projects/` 建立專案資料夾
2. 初始化 Git + 建立 GitHub 公開 repo
3. 建立 `CLAUDE.md` 專案藍圖
4. 建立 Obsidian 工作日誌
5. 建立 `.gitignore`

**產出**：
- `projects/{專案名}/`（本地資料夾）
- GitHub repo：`https://github.com/jarvis800317/{專案名}`
- Obsidian 工作日誌：`工作日誌/{專案名}.md`

---

### 2. 收工（shutdown）

**觸發**：說「收工」、「結束了」、「先到這裡」

**做的事**：
1. 盤點今天做了什麼（從對話歷史分析）
2. 更新 Obsidian 工作日誌：
   - 「上次做到哪」段落
   - 「最近更動紀錄」表格（✅✅✅）
   - 「踩坑筆記」（若有新坑）
3. Git commit + push 到 GitHub
4. 若專案含有 `firebase.json` → 部署 Firebase Hosting + Realtime Database

**不會做的事**：
- 對「純問問題」執行同步
- commit `.claude/`、`node_modules/`、`.env`
- 寫「更新」、「修改」等無意義 commit message
- Firebase 同步：僅在 `firebase.json` 存在時執行（專案無 Firebase 元件則跳過）

**輸出**：
```
| 平台 | 變動內容 | 狀態 |
|------|----------|------|
| GDrive | 自動同步 | ✅ |
| Obsidian | 工作日誌更新 | ✅ |
| GitHub | commit + push | ✅ |
| Firebase | 部署（僅有 firebase.json） | ✅ |
```

---

### 3. 繼續（resume）

**觸發**：說「繼續」、「接續工作」、「上次做到哪」

**做的事**：
1. 讀取 Obsidian 工作日誌
2. 摘要進度：「上次做到哪」、「最近記錄」、「踩坑筆記」
3. 建議下一步

**輸出範例**：
```
📋 專案：coordinate-hunter
上次做到：完成座標獵人遊戲，已發布至 GitHub Pages
目前進度：100%

🕳️ 待注意：座標系統用 (-5,5)，與地圖資料不同，需做轉換

📍 建議下一步：著手處理座標轉換模組
```

---

## 每日自動備份（安全網）

**用途**：萬一忘記說「收工」，每天 22:00 自動檢查並備份。

**機制**：Cronjob 執行 `~/.hermes/scripts/jarvis-backup.sh`

**邏輯**：
- 檢查 Jarvis專案 是否有未提交的變動
- 有變動 → commit + push
- 無變動 → 安靜跳過

**注意**：自動備份只做 Git 同步，不更新工作日誌（工作日誌必須手動在「收工」時更新）。

---

## 使用範例

### 情境一：開第一個專案

```
你：開新專案 coordinate-hunter
我：
✅ 專案「coordinate-hunter」已初始化完成！
📁 位置：~/我的雲端硬碟/Jarvis專案/projects/coordinate-hunter/
🐙 GitHub：https://github.com/jarvis800317/coordinate-hunter
📝 工作日誌：~/我的雲端硬碟/2ndBrain/工作日誌/coordinate-hunter.md

下一步：請告訴我你想做什麼功能？
```

### 情境二：實作後收工

```
你：收工
我：
盤點今天的工作：
- 完成了座標輸入介面
- 實作了隨機座標產生器
- 設定 60 秒倒計時

同步狀態：
| 平台 | 變動內容 | 狀態 |
|------|----------|------|
| GDrive | 自動同步 | ✅ |
| Obsidian | 工作日誌已更新 | ✅ |
| GitHub | 已 commit + push | ✅ |
| Firebase | 已部署（專案含 firebase.json） | ✅ |
```

### 情境三：隔天繼續

```
你：繼續
我：
📋 專案：coordinate-hunter
上次做到：完成計時器功能，正在整合評分系統
目前進度：75%

🕳️ 踩坑記錄：計時器與.Random座標有 JS 衝突，需調查

📍 建議下一步：修復計時器衝突 → 完成評分系統 → 測試
```

---

## 專案命名規則

- 英文小寫 + 連字號（例：`coordinate-hunter`、`taolin-tool`）
- 不要有空格或特殊字元
- 每個專案獨立資料夾在 `projects/` 內

---

## 安全規則

- `.claude/` 永不 commit
- `.env`、`*.key`、`credentials.*` 永不 commit
- API Key 只存環境變數，不寫進程式碼
- 不把敏感資訊寫進 Obsidian 筆記

---

## 安裝流程（一次性）

本 workflow 已預設安裝完畢，包含：

- [x] `skill:收工`（~/.hermes/skills/workflow/shutdown/）
- [x] `skill:繼續`（~/.hermes/skills/workflow/resume/）
- [x] `skill:init-project`（~/.hermes/skills/workflow/init-project/）
- [x] `projects/` 資料夾結構
- [x] Obsidian 工作日誌目錄
- [x] 每日自動備份 Cronjob（22:00 執行）
- [x] AGENTS.md 已更新專案工作區設定

**若要重新設定**：參考本文件的「三個核心技能」章節，手動建立對應的 SKILL.md。

---

## 更新紀錄

| 日期 | 版本 | 更新內容 |
|------|------|----------|
| 2026-05-16 | v1.1 | 加入 Firebase 為第四個同步目標（條件觸發：僅在專案含 firebase.json 時執行） |
| 2026-05-15 | v1.0 | 初版建立（對應 Hermes Agent Mac mini 環境）|

---

*本文件由 Hermes Agent 協助建立*