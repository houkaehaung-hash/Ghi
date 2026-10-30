# 交接給電腦版 Claude Code（Windows）

> 給 Claude：這份是雲端的 Claude session 留下的交接說明。使用者在 Windows 電腦上，想完成下面的設定。
> 請**照順序**做，每一步做完跟使用者回報結果。要安裝軟體、改設定，或需要使用者登入的步驟，先問過使用者再做。
> 請用繁體中文和使用者溝通。

## 背景

- 使用者想用 **myoffice**（https://github.com/Charonyuu/whatsmyagentdoing）把電腦上所有 AI agent 的工作狀態，即時顯示成一個辦公室畫面。每個 Claude Code / Codex 等 session 和 subagent 都會變成一個小人。
- 使用者也想要「一個主管 Claude 指揮多個員工 agent」的分工方式。
- 雲端 session 已經把 myoffice 放進 GitHub repo `houkaehaung-hash/ghi`（分支 `claude/install-whatsmyagentdoing-skill-tuliw7`，路徑 `.claude/skills/myoffice/`），但那是專案層級的副本。這台電腦要另外做**使用者層級**的安裝，所有專案才都能用。
- myoffice 只在 Linux 雲端環境測試過（Node v22，`--json`、`--scenes` 和網頁都正常），**還沒在 Windows 上測試過**。遇到問題請實際排查，不要假設它能跑。

## 任務

### 1. 檢查環境
- 執行 `git --version` 和 `node --version`。
- 需要 Git，以及 Node.js **18 以上**。缺少的話，問使用者是否同意用 winget 安裝：
  - `winget install --id Git.Git -e`
  - `winget install --id OpenJS.NodeJS.LTS -e`
  - 裝完後使用者可能要重開終端機，PATH 才會更新。

### 2. 安裝 myoffice skill
- 如果 `%USERPROFILE%\.claude\skills\myoffice` 已經存在，先看裡面是什麼，不要直接覆蓋。
- 安裝：
  ```
  git clone --depth 1 https://github.com/Charonyuu/whatsmyagentdoing "%USERPROFILE%\.claude\skills\myoffice"
  ```
- 驗證：`node "%USERPROFILE%\.claude\skills\myoffice\bin\cli.js" --json --hours 2`，應該印出 JSON，裡面至少有目前這個 session。
- 告訴使用者：skill 要在**新的 session** 才會出現；之後輸入 `/myoffice` 就會打開辦公室畫面。

### 3. 建立「員工」subagent
在 `%USERPROFILE%\.claude\agents\` 建立下面 4 個檔案（UTF-8，不要 BOM）。如果檔案已經存在，先問使用者再覆蓋。

`researcher.md`
```markdown
---
name: researcher
description: 調查程式碼或主題並回報發現。在規劃或寫程式前使用。
tools: Read, Grep, Glob, WebSearch, WebFetch
---
你是研究員。針對任務找出相關檔案、事實和限制，清楚簡短地回報。不要修改任何檔案。用繁體中文回覆。
```

`coder.md`
```markdown
---
name: coder
description: 實作程式碼修改。任務需要寫或改程式時使用。
tools: Read, Edit, Write, Bash, Grep, Glob
---
你是軟體工程師。完全照要求實作，配合既有的程式風格，並回報改了哪些檔案。用繁體中文回覆。
```

`tester.md`
```markdown
---
name: tester
description: 撰寫並執行測試來驗證修改。程式改完後使用。
tools: Read, Edit, Write, Bash, Grep, Glob
---
你是測試工程師。為這次修改新增或更新測試，執行後回報哪些通過、哪些失敗，失敗的附上錯誤輸出。用繁體中文回覆。
```

`reviewer.md`
```markdown
---
name: reviewer
description: 審查程式碼修改，找出 bug 和問題。完成前做最後檢查時使用。
tools: Read, Grep, Glob, Bash
---
你是程式碼審查者。檢查這次修改有沒有 bug、遺漏的邊界情況和難懂的程式碼。不要修改任何檔案。問題由嚴重到輕微列出。用繁體中文回覆。
```

### 4. 設定 GitHub（問過使用者再做）
- 檢查 `git config --global user.name` 和 `user.email` 有沒有設定，沒有的話請使用者提供。
- 問使用者要不要安裝 GitHub CLI：`winget install --id GitHub.cli -e`，然後請**使用者自己**執行 `gh auth login`（選 GitHub.com → HTTPS → Login with a web browser），因為要在瀏覽器登入。
- 問使用者要不要把 repo 抓下來：
  ```
  git clone https://github.com/houkaehaung-hash/ghi
  ```
  （目前只有一個分支：`claude/install-whatsmyagentdoing-skill-tuliw7`）

### 5. 收尾
- 請使用者開一個新的 Claude Code session，輸入 `/myoffice` 打開辦公室。
- 教使用者怎麼讓主管派工作，例如：
  > 你是主管。幫我把這個專案加上 XX 功能：先派 researcher 調查，再派 coder 實作、tester 測試，最後讓 reviewer 審查。能平行的就同時派。
- 提醒：同時開越多 agent，用量消耗越快。
- 最後總結哪些步驟完成了、哪些跳過或失敗（附上錯誤訊息）。
