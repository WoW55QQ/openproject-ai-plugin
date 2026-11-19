---
name: openproject-conventions
description: 在 OpenProject（op.aplai.net）開單、建立或更新工作套件（work package／工單／Bug／Task）、改狀態、留言、設定關聯、記錄工時、上傳附件與大檔案上傳、讀寫 Wiki、文件或協作文件時使用——描述範本、狀態用法、建立前先查重複、刪除與批次修改前須確認、工時、附件、Wiki 與協作文件的注意事項。
---

# OpenProject 工單慣例

## 建立前

1. 先用 `search_work_packages` 搜尋關鍵字，確認沒有重複的工單。有的話改為留言或更新既有工單。
2. 用 `search_projects` 確認專案；不確定是哪個專案時問使用者，不要猜。
3. 用 `list_types` 取得該專案可用的類型。

## 標題

手動撰寫，一句話說明問題或任務，不加類型前綴。例：「匯出 PDF 時 App 閃退」。

## Bug 描述範本

```markdown
## 環境
- **版本 / Build**：
- **平台**：
- **帳號 / 角色**：
- **發生時間**：

## 重現步驟
1.
2.
3.

## 預期結果

## 實際結果

## 發生頻率
- [ ] 每次都發生
- [ ] 偶爾發生
- [ ] 只發生過一次

## 嚴重程度
- [ ] 阻斷：無法繼續使用
- [ ] 嚴重：主要功能失效，有替代方案
- [ ] 一般：功能異常但不影響主流程
- [ ] 輕微：文字、排版、提示訊息

## 附加資訊

## 備註
```

## Task 描述範本

```markdown
## 目標

## 背景 / 原因

## 工作項目
- [ ]

## 完成條件（Definition of Done）
- [ ] 功能可在測試環境操作
- [ ] 已附上截圖或測試結果
- [ ] 文件 / 說明已更新
- [ ] 已通知相關人員

## 相關資源

## 備註
```

使用者沒提供的欄位留空，不要編造。

## 狀態

| 狀態 | 用途 |
|---|---|
| New | 剛建立，還沒開始 |
| In progress | 進行中 |
| Closed | 完成並確認 |
| On hold | 暫停，等待他人或資源 |
| Rejected | 不做：重複、無法重現、需求取消 |

可用狀態依類型與工作流程而定；改狀態前先用 `list_statuses` 確認目標狀態可用。

## 工時

- 用 `create_time_entry` 記錄。記之前確認**日期**與**時數**；使用者說「昨天」「上週五」時換算成實際日期並在回覆中寫出來。不確定就問，不要猜。
- 活動（開發、會議等）用 `list_time_entry_activities`（帶 `project_id`）查可用選項；使用者沒指定且專案沒有預設時，可以不填。
- 只替使用者本人記工時。要替他人記之前先確認。
- 查詢用 `search_time_entries`（可依工作套件、專案、使用者、日期區間篩選）；進行中的計時器不會出現在結果中。

## 附件

- 本機檔案（安裝檔、影片、壓縮檔、截圖、log 等）一律用 `create_upload_url`（帶 `container_type`、`container_id`）取得上傳通行證，再執行回傳的 `curl_command`，把其中的 `<local file path>` 換成實際的本機檔案路徑。成功會回 JSON：`id`、`filename`、`filesize`、`content_type`、`download_url`。
- Windows PowerShell 要用 `curl.exe`，不要用 `curl`（那是 `Invoke-WebRequest` 的別名）。本機路徑含逗號或分號時，寫成 `-F "file=@\"C:/path/a,b.zip\""`。
- 一張通行證只能上傳一個檔案，10 分鐘內有效且只能用一次（失敗也算用掉）；每個檔案各申請一次，失敗就重新申請，不要重用舊的通行證。
- 檔案大小受 OpenProject 附件上限限制；Azure 前端單一請求約 230 秒上限，網路太慢的超大檔案可能逾時，逾時時請使用者改到網頁上傳。上傳逾時或結果不明時，先用 `list_attachments` 確認附件是否已建立，再決定是否重傳，避免重複附件。
- `create_attachment`（base64）只用於對話中產生、沒有存成檔案的內容（例如貼上的文字或剛產生的小段內容）；有本機檔案就不要用它。
- 上傳前確認要掛在哪個工作套件、Wiki 頁面或文件上。
- 檔案類型與大小受系統管理設定限制，被拒絕時把錯誤訊息轉告使用者。
- **查看或下載附件（例如工單裡的截圖）**：MCP 沒有直接回傳檔案內容的工具，也不要用 `readMcpResource` 讀附件網址（拿不到內容）。做法：
  1. 用 `list_attachments` 取得附件清單，每次呼叫都會給新的 `_links.downloadLocation.href`（約 6 小時有效的下載網址，不需要登入）。
  2. 用 `curl -sSL -o <暫存資料夾>/<檔名> "<downloadLocation>"` 下載（Windows PowerShell 用 `curl.exe`），網址要整串加引號。
  3. 圖片用讀檔工具開啟查看；其他檔案依類型處理。
  - 不要重用之前拿到的舊網址；下載回 403 就重新呼叫 `list_attachments` 取得新網址。`staticDownloadLocation`（`/api/v3/attachments/<id>/content`）需要網頁登入，MCP 用不了。

## Wiki

- 讀取前用 `search_wiki_pages` 找頁面，再用 `get_wiki_page` 取完整內容。
- `update_wiki_page` 會**整份取代**內容：先 `get_wiki_page` 取得最新內容與 `lock_version`，在其上修改後連同 `lock_version` 一起送出；收到「changed by someone else」就重新讀取再改。
- 更新時用 `comment` 說明改了什麼，會留在版本紀錄。
- 專案沒有啟用 Wiki 時無法建立頁面，請使用者請專案管理員到專案設定啟用。

## 文件

文件有兩種，用 `search_documents` 結果中的 `kind` 欄位分辨：

- **協作文件**（`collaborative`，網頁上的即時共同編輯文件，內容是 BlockNote 區塊）。
- **一般文件**（`classic`，描述為 Markdown）。

### 建立

`create_document` 的 `kind` 預設是 `collaborative`（與網頁一致），建立後內容為空，用 `insert_document_blocks` 填內容。只有使用者明確要一般文件時才用 `kind: classic`，此時內容放 `description`（Markdown）。

### 協作文件

所有修改都會**即時出現在正在編輯的人的畫面上**，動手前先想清楚要改哪裡。沒有任何刪除工具；要刪除區塊或文件請使用者到網頁操作。

流程：

1. 先 `get_document_content` 取得區塊樹（每個區塊有 `id`、`type`、非預設的 `props`、`content`、`children`），**沿用既有結構的層級與可折疊（`isToggleable`）設定**，不要自創不同的排版。大文件的整份樹沒有分頁，會佔大量 context，只在需要時讀。
2. 用 `insert_document_blocks` 新增：`placement` 可為 `before`、`after`（預設）、`first_child`、`last_child`，搭配 `reference_block_id`。文件為空時省略 `reference_block_id`；非空文件必須給。新區塊不能帶 `id`。一次最多 500 個區塊（含子區塊）、32 層、1 MB。
   - 空文件省略 `reference_block_id`，**不要傳空字串**。新建的協作文件若有人用瀏覽器開過，可能已有一個空段落：此時用 `update_document_block` 寫進那段，或以它為參考插入。
3. 用 `update_document_block` 修改既有區塊：只改該區塊本身，子區塊保留。`content` 會**整段取代**該區塊的文字（空字串為清空），所以**更新前務必先重讀**（`get_document_content`），否則會覆蓋別人剛改的字。型別不變時 `props` 會合併。
   - 傳 `content` 時要傳 JSON 陣列本身（或純字串），不要把陣列序列化成字串，否則會被當成純文字。
   - `props` 必須是該區塊類型 schema 內的鍵且型別正確（例如 heading 的 `level` 傳數字 `2`，不是字串 `"2"`）。
   - 改了 `type` 時只套用這次給的 `props`；子區塊不能用 update 改。
4. 連結的 `href` 與檔案類區塊的 `url` 只允許 `http`、`https`、`mailto`、`tel` 或 `/` 開頭的站內路徑。圖片、檔案類區塊的 `props.url` 也適用同樣規則；附件用上傳結果的 `download_url`。
5. 要放檔案連結時，先依「附件」流程上傳，再用回傳的 `download_url` 插入 `link`。
6. 做完回報插入或修改了哪幾段（區塊位置與內容摘要）。

範例：可折疊標題底下放描述與檔案連結（`insert_document_blocks` 的 `blocks`）：

```json
[{"type":"heading","props":{"level":2,"isToggleable":true},"content":"0.3.10","children":[
  {"type":"heading","props":{"level":3,"isToggleable":true},"content":"描述","children":[{"type":"paragraph","content":"修正 webcam 低光源模糊"}]},
  {"type":"heading","props":{"level":3,"isToggleable":true},"content":"檔案","children":[{"type":"paragraph","content":[{"type":"link","href":"https://op.aplai.net/api/v3/attachments/123/content","content":"aiGait-0.3.10.zip"}]}]}
]}]
```

錯誤處理：

- 「找不到區塊」：區塊已被改動或不存在，重新 `get_document_content` 後再操作。
- 「busy」：伺服器忙碌，等幾秒再重試。
- 「did not confirm the change; it may or may not have been applied」：**先 `get_document_content` 讀取確認是否已套用，再決定要不要重試**；不可直接重試 `insert_document_blocks`，否則內容會重複。
- 「Nothing was changed」：確定沒有變更，可以直接重試。
- 「Invalid parameters」／「Invalid request」：參數有誤，修正後再呼叫。
- 「Too much content」：內容太多，拆成多次呼叫。

### 一般文件

- 用 `search_documents`、`update_document` 讀寫標題與描述（Markdown）。
- 文件沒有版本號，無法偵測別人同時修改：更新前先 `search_documents` 讀取最新描述，在其上修改。

## 其他

- 工時、附件、Wiki、文件（含協作文件的區塊）都**沒有刪除工具**；需要刪除時請使用者到網頁操作。
- 在描述或留言寫 `#123` 會自動連到工作套件 123。
- 刪除（含 `delete_work_package_relation`）或一次修改超過 1 張工單前，先列出將變更的項目，等使用者確認後再執行。
- 使用者語意不明確時，不要擅自把工單改成 Closed 或 Rejected，先詢問。
