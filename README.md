# OpenProject AI 插件（op.aplai.net）

讓 Claude Code 與 Codex CLI 直接操作公司的 OpenProject：搜尋、建立、更新工作套件，留言，設定關聯等。
登入使用公司 Microsoft 帳號，AI 代理的權限與你在網頁上的權限相同，所有操作記在你名下。

## 前置需求

- 已安裝 Claude Code 或 Codex CLI 其中之一：
  - Claude Code：<https://code.claude.com/docs/en/setup>
  - Codex CLI：<https://developers.openai.com/codex/cli>
- 公司 Microsoft 帳號，且已被加入 OpenProject（<https://op.aplai.net>）。
- 以下的 `claude`、`codex` 指令請在 PowerShell（Windows）或終端機（macOS）執行，不是在 Claude 對話裡輸入。若你已在 Claude Code 對話中，也可以改輸入 `/plugin marketplace add WoW55QQ/openproject-ai-plugin` 與 `/plugin install openproject@aplai-openproject`。

## Claude Code

```text
claude plugin marketplace add WoW55QQ/openproject-ai-plugin
claude plugin install openproject@aplai-openproject
```

重新啟動 Claude Code 後輸入 `/mcp`，選 `openproject` → Authenticate。瀏覽器會開啟 OpenProject，按「Microsoft 帳號」登入並按「授權」。

## Codex CLI

1. 開啟設定檔 `config.toml`：
   - Windows PowerShell：先執行 `New-Item -ItemType Directory -Force $env:USERPROFILE\.codex`（資料夾已存在也沒關係），再執行 `notepad $env:USERPROFILE\.codex\config.toml`，若提示檔案不存在，選「是」建立。
   - macOS：`mkdir -p ~/.codex && touch ~/.codex/config.toml && open -e ~/.codex/config.toml`
2. 把 [`codex/config.toml.example`](codex/config.toml.example) 的內容貼到檔案最後並存檔。
   - **注意**：檔案裡若已有 `[mcp_servers.openproject]`，不要重複貼上，否則 Codex 會讀取失敗；請改為替換掉舊的那一段。
3. 執行：

```text
codex mcp login openproject
```

瀏覽器開啟後同樣用 Microsoft 帳號登入並授權。

## 試試看

- 「列出我負責、還沒結案的工作套件」
- 「在某某專案開一張 Bug：匯出 PDF 時閃退，Android 14」
- 「把 #181 改成 Closed，留言：已在 v1.3 修正」

## 登出與撤銷

- 撤銷授權：OpenProject → 右上頭像 → 我的帳號 → 存取權杖 → 撤銷對應的應用程式。
- 離職或不再需要時，請通知管理員停用帳號。

## 疑難排解

| 狀況 | 處理 |
|---|---|
| 瀏覽器顯示 `invalid_redirect_uri` | 回呼埠被改過，或 Codex 設定的 `url` 與範例不同（Codex 回呼網址由 `url` 算出，多一個 `/` 也不行）。Claude Code 必須是 33418，Codex 必須是 33419 |
| Claude Code 的 `/mcp` 無法完成登入 | 改在 PowerShell 或終端機執行 `claude mcp login plugin:openproject:openproject` |
| 登入時說埠號被占用 | 關掉占用 33418 或 33419 的程式後重試 |
| 授權後卻顯示沒有權限，或看到別人的名字 | 瀏覽器登入了錯的 Microsoft 帳號。登出 Microsoft 後重新執行登入，選公司帳號 |
| 登入後工具回「找不到」 | 你的 OpenProject 帳號沒有該專案權限，請專案管理員把你加入成員 |
| 登入頁出現帳號密碼欄位 | 正常只會有 Microsoft 按鈕；若看到帳號密碼欄位，請勿輸入，並通知管理員 |
