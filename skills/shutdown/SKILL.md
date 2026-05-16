---
name: 收工
description: 收工同步技能。當使用者說「收工」、「結束了」、「準備換電腦」、「該同步的同步」、「先到這裡」等結束工作時，執行三方同步：更新 Obsidian 工作日誌、git commit + push、報告狀態。若專案涉及 Firebase（有 firebase.json），一併同步 Firebase 即時資料庫與部署狀態，方便製作過程視覺化即時展示、進度追蹤。
---

# 收工同步技能

對話結束前，把今天的工作完整保存到四個家：
- **GDrive**：自動同步（不用管）
- **Obsidian 工作日誌**：更新「上次做到哪」+「最近更動紀錄」
- **GitHub**：commit + push 本地變動
- **Firebase**（若有 firebase.json）：一併部署，方便製作過程視覺化即時展示

## 收工 SOP

### 步驟 1：盤點今天做了什麼

從對話歷史中分析：
- 完成了哪些檔案？
- 做了哪些決策？
- 踩到哪些新坑？

若對話中沒有實質變動（只是問問題），則回應：「今天沒有實質進度，略過同步。」結束技能。

### 步驟 2：確認工作目錄

從對話脈絡推斷當前專案路徑，預設：
- 專案總資料夾：`~/我的雲端硬碟/Jarvis專案/`
- Obsidian 工作日誌：`~/我的雲端硬碟/2ndBrain/工作日誌/`

若無法確認，詢問使用者：「請告訴我專案名稱。」

### 步驟 3：更新 Obsidian 工作日誌

在工作日誌資料夾中找對應的 `專案名.md`，或建立新檔案。

更新內容：
- `上次做到哪` 段落：最後動作、完成檔案、對話脈絡摘要
- `最近更動紀錄` 表格：新增一行（今天日期 + 變更摘要 + ✅✅✅）
- `踩坑筆記`：若有新坑，依分類追加

**若工作日誌不存在**：建立新檔案，模板如下：

```markdown
# {專案名} 工作日誌

> 📌 進度日誌（變動快）。請每次收工時更新。

## 上次做到哪

**最後動作**：（描述）
**完成檔案**：（列出）
**對話脈絡**：（摘要）

## 最近更動紀錄

| 日期 | 變更摘要 | GDrive | Obsidian | GitHub |
|------|----------|--------|----------|--------|
| {今天} | {描述} | ✅ | ✅ | ✅ |

## 踩坑筆記

（待記錄）
```

### 步驟 4：Git commit + push

```bash
cd "{專案路徑}"
git add -u
git add -A  # 包含新檔案
git commit -m "{標題行}

- {具體變動1}
- {具體變動2}
- {踩坑記錄（若有）}"
git push origin main
```

**不 commit 的檔案**：
- `.claude/`（Claude Code 設定，含敏感資訊）
- `node_modules/`
- `*.log`
- `.env`（除非要 commit）

若專案尚非 git repo，先初始化：
```bash
cd "{專案路徑}"
git init
gh repo create {專案名} --public --source=. --push
```

### 步驟 5：Firebase 同步（專案有 firebase.json 時才執行）

```bash
# 檢查是否為 Firebase 專案
if [ -f "{專案路徑}/firebase.json" ]; then
  # 部署 Firebase 專案（視覺化展示、進度追蹤用）
  cd "{專案路徑}"
  firebase deploy --only hosting,database --project {project_id}
fi
```

**用途**：方便製作過程視覺化即時展示、進度追蹤（Firebase Realtime Database + Hosting）。

### 步驟 6：報告同步狀態

提供四勾表格：

| 平台 | 變動內容 | 狀態 |
|------|----------|------|
| GDrive | 自動同步 | ✅ |
| Obsidian | 工作日誌更新 | ✅ |
| GitHub | commit + push | ✅ |
| Firebase | Firebase 部署（若有 firebase.json） | ✅ |

## 不該做的事

- ❌ 對「沒實質進度」的對話執行同步（例：純問問題、查資料）
- ❌ commit `.claude/`、`node_modules/`、`.env`
- ❌ commit message 寫「更新」、「修改」等無意義標題
- ❌ 刪除既有工作日誌內容，應以「追加」為主