# AI Memory Vault — 使用指南（安裝版）

> 這份是給**下載安裝包來用**的人。
> 想從原始碼架設 / 開發 → 見主 repo 的 `USAGE.md`。
>
> 本檔由 `AI_Engine/packaging/publish-update.ps1` 於每次發布時自動推送至
> releases repo，**請勿在 releases repo 直接編輯**——那裡的版本會在下次發布時被覆蓋。
> 要修改請改主 repo 的 `AI_Engine/packaging/release-usage.md`。

---

## 1. 安裝

從 [Releases](../../releases/latest) 下載最新的 `AI-Memory-Vault-Setup-<版本>.exe`，
執行安裝精靈。預設安裝到 `C:\Program Files\AI Memory Vault`。

升級時直接執行新版安裝檔即可，它會沿用你上次選的安裝目錄。

## 2. 首次設定

桌面點擊 **AI Memory Vault CLI**，會自動進入設定精靈，依序問：

1. **Vault 路徑**——知識庫要放哪（建議放在有備份的磁碟）
2. **使用者名稱與組織**
3. **LLM provider**——ollama（本機）／ gemini（需 API 金鑰）／ copilot
4. **回應語言**——預設 `zh-TW`
5. **Starter pack**——見下一節，**預設全不勾**

隨時可以重跑：`vault-cli --setup`，或只改其中一段：`vault-cli --setup-section packs`。

設定檔在 `%APPDATA%\AI-Memory-Vault\config.json`；API 金鑰放同目錄的 `.env`。

## 3. Starter pack（預設全不裝）

v4.0.0 起，引擎只出**機制**與中性預設，**政策**（風格規範、SOP、agent 角色這類
因人而異的東西）要你明確選才會進來。所以剛裝好的 Vault 是零政策的——
沒有 coding style 規範、沒有 agent 角色、`--end-of-day` 會明確拒絕執行，
而不是偷偷跑一套你沒同意的流程。

| Pack | 裝了之後 |
|------|---------|
| `coding-style` | coding-style guard 生效，各語言 style skill 可用 |
| `agent-roles` | `dispatch_agent` 有 14 個角色可派（依賴 `coding-style`） |
| `end-of-day` | `--end-of-day` 與收工排程可用 |

```powershell
vault-cli --setup-section packs     # 互動式勾選安裝／移除
```

三個 pack 裝的都是作者本人的工作方式，作為可運作的範例——
可以直接用、可以改，也可以完全不裝自己寫一套。
**不裝也能用全部 29 個 MCP 工具**，pack 影響的是行為政策，不是功能。

## 4. 接上編輯器（MCP）

### VS Code / Cursor

`mcp.json`：

```json
{
  "servers": {
    "ai-memory-vault": {
      "type": "stdio",
      "command": "C:/Program Files/AI Memory Vault/vault-mcp.exe",
      "args": [],
      "cwd": "C:/Program Files/AI Memory Vault"
    }
  }
}
```

### Claude Desktop / Claude Code

同樣指向 `vault-mcp.exe`，設定檔位置依各工具說明。

> ⚠️ 安裝時若改過目錄，請把路徑換成實際安裝位置。
> 多個編輯器要同時用 → 見「5. 多編輯器共用」。

## 5. 多編輯器共用（SSE 常駐）

同時開 VS Code + Claude Desktop + Codex 時，讓它們共用一個常駐 server，
避免各自啟一份、彼此搶 SQLite 寫入鎖：

```powershell
"C:\Program Files\AI Memory Vault\vault-mcp.exe" --mode api    # 常駐於 :8765，含內嵌排程
```

VS Code 可以直接連 SSE：

```json
{
  "servers": {
    "ai-memory-vault": { "type": "sse", "url": "http://127.0.0.1:8765/sse" }
  }
}
```

Claude Desktop / Codex 這類只講 stdio 的客戶端**不用改設定**——
`vault-mcp.exe` 會自己判斷：

| 狀況 | 行為 |
|------|------|
| SSE server 正在跑 | 自動橋接，成為它的 stdio proxy |
| SSE 沒跑，也沒有其他 stdio 實例 | 直接起 stdio server |
| SSE 沒跑，但已有 stdio 實例 | 靜默退出，不搶 SQLite |

## 6. 產出的執行檔

| 執行檔 | 用途 |
|--------|------|
| `vault-cli.exe` | 互動式 CLI（雙擊啟動） |
| `vault-mcp.exe` | MCP server；編輯器指向這個。加 `--mode api` 則轉為 SSE 常駐 |
| `vault-scheduler.exe` | 獨立排程守護（一般不需要，SSE 模式已內嵌） |

## 7. 自動更新

啟動時會檢查新版。有更新時會提示，按下後自動下載並安裝，完成／失敗都會跳通知。

不想更新：`vault-cli --dismiss-update`。

## 8. 常見問題

**Q：更新後我改過的設定不見了？**
不會。引擎擁有的檔案是唯讀投影、會被重生；你自己的筆記與設定受
provenance 閘門保護——沒有「這是引擎放的」基線，一律保留你的版本。

**Q：搜尋不到剛寫的筆記？**
用編輯器以外的方式（Obsidian、檔案總管）改過檔案後，呼叫一次 `sync_vault`。

**Q：想知道某個檔案歸誰管？**
`vault_doctor(action="ownership")`；懷疑被改壞用 `vault_doctor(action="reconcile")`。

**Q：怎麼知道這版改了什麼？**
看該版 Release 頁面的說明，內容取自 CHANGELOG。

---

<sub>問題回報請到主 repo 的 Issues。</sub>
