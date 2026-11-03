# OpenProject AI 插件（op.aplai.net）

讓 Claude Code 與 Codex CLI 直接操作公司的 OpenProject：搜尋、建立、更新工作套件，留言，設定關聯等。
登入使用公司 Microsoft 帳號，AI 代理的權限與你在網頁上的權限相同，所有操作記在你名下。

## Claude Code

```text
claude plugin marketplace add WoW55QQ/openproject-ai-plugin
claude plugin install openproject@aplai-openproject
```

重新啟動 Claude Code 後輸入 `/mcp`，選 `openproject` → Authenticate。瀏覽器會開啟 OpenProject，按「Microsoft 帳號」登入並按「授權」。

## Codex CLI

1. 把 [`codex/config.toml.example`](codex/config.toml.example) 的內容加到 `~/.codex/config.toml`（Windows：`%USERPROFILE%\.codex\config.toml`）。
2. 執行：

```text
codex mcp login openproject
```

瀏覽器開啟後同樣用 Microsoft 帳號登入並授權。

## 試試看

- 「列出我負責、還沒結案的工作套件」
- 「在 aiGait 專案開一張 Bug：匯出 PDF 時閃退，Android 14」
- 「把 #181 改成 Closed，留言：已在 v1.3 修正」

## 登出與撤銷

- 撤銷授權：OpenProject → 右上頭像 → 我的帳號 → 存取權杖 → 撤銷對應的應用程式。
- 離職或停用帳號：**必須在 OpenProject 停用使用者**。只在 Microsoft Entra 停用不會斷開已登入的 AI 代理。

## 疑難排解

| 狀況 | 處理 |
|---|---|
| 瀏覽器顯示 `invalid_redirect_uri` | 回呼埠被改過。Claude Code 必須是 33418，Codex 必須是 33419 |
| 登入時說埠號被占用 | 關掉占用 33418 或 33419 的程式後重試 |
| 登入後工具回「找不到」 | 你的 OpenProject 帳號沒有該專案權限，請專案管理員把你加入成員 |
| 登入頁出現帳號密碼欄位 | 不會發生；若有，請通知管理員 |
