# AI Memory Vault — 使用指南

> 這份是給**下載安裝來用**的人。AI Memory Vault 是以 Markdown 為底、RAG + MCP 的個人知識/記憶庫；
> 裝好後你的 AI 工具（Claude Code / VS Code Copilot / Codex / Antigravity）連上它，
> 就能在對話中自動取得你的專案脈絡、規則、直覺，並讀寫這個知識庫。

---

## 1. 安裝

1. 到 [Releases](https://github.com/gabrielchen0314/AI-Memory-Vault-releases/releases/latest) 下載 `AI-Memory-Vault-Setup-vX.Y.Z.exe`
2. 執行安裝精靈（建議保留「登入時自動啟動 MCP SSE Server」勾選）

安裝後有三支程式：`vault-mcp.exe`（MCP Server）、`vault-scheduler.exe`（排程）、`vault-cli.exe`（互動 CLI）。

## 2. 首次設定

開始選單開「環境設定」，或執行 `vault-cli.exe --setup`：
- **Vault 路徑**：知識庫（.md 檔）要放哪
- 使用者資訊、LLM provider（選填）

## 3. 連接你的 AI 工具（MCP）

架構：**一個常駐 SSE Server + 各 editor 用輕量 stdio bridge 連它**。SSE Server 開機自啟；
各工具 MCP 設定**一律用 stdio 指向 `vault-mcp.exe`**（它自動橋接到共用 SSE，不重載模型）。

> ⚠️ 不要用「直連 SSE url」——直連的 client 不會自動重連，Server 重啟後要手動重連。

| 工具 | 設定檔 | ai-memory-vault 寫法（stdio） |
|------|--------|------------------------------|
| Claude Code | `~/.claude.json` | `"command": "C:/.../vault-mcp.exe"` |
| VS Code Copilot | `%APPDATA%\Code\User\mcp.json` | `"type":"stdio","command":"C:/.../vault-mcp.exe"` |
| Codex App | `~/.codex/config.toml` | `[mcp_servers.ai-memory-vault]` `command="C:/.../vault-mcp.exe"` |
| Antigravity | `~/.gemini/antigravity/mcp_config.json` | `"mcpServers":{"ai-memory-vault":{"command":"C:/.../vault-mcp.exe"}}` |

連線成功時右下角會跳通知 `✓ vX.Y.Z 已連線，所有工具就緒`。

## 4. 日常使用

裝好連上後，多數功能在**對話中自動發生**：
- **自動脈絡**：對話開始自動注入導航、當前交接、專案脈絡。
- **查知識 / 記東西**：叫 AI「搜尋 vault 關於 X」或「把這個決定記進 vault」。
- **收工**：跟 AI 說「收工」→ 更新進度、精煉直覺、產每日回顧。
- **Agent 排程任務**（見下）：把重複性工作寫成排程，由 headless CLI 定時自動跑、產出寫回 Vault。

## 5. 排程任務（Agent Task）

讓排程在指定時間用 headless CLI（claude / codex）自動跑一件事、產出寫進 Vault。

- **建任務**：跟 AI 說要排什麼，它用 `create_agent_task` 幫你建（會驗證欄位）；或手寫
  `<Vault>/automation/tasks/{id}.md`（frontmatter 定 cron/CLI/權限/產出 + body 寫任務指示）。
- **前提**：SSE Server（開機自啟）要在跑；新增/改排程後**重啟 SSE**（重新登入或跑 `start-vault-sse.ps1`）才生效。
- **必記兩點**：cron 星期用名稱 `mon-fri`（勿用數字）；需查 Redmine 等 MCP 的 codex 任務要在 frontmatter 加 `codex_full_access: true`。
- **完整格式與細節**：裝好後在對話中 `search_vault("agent task 排程")`，或看 Vault 內 `knowledge/agent-task-scheduling.md`。

內建防護：cron 最小間隔 5 分鐘、以「產出檔存在且非空」判成敗、冪等（已產出則跳過）、同時最多 1 個 agent。

## 6. 自動更新

- 開機偵測到新版時會**彈窗問是否更新**（15 秒未選＝否）。
- 或在對話中叫 AI 執行 `apply_update()`；或手動下載新安裝檔覆蓋安裝。

## 7. 疑難排解

| 症狀 | 處理 |
|------|------|
| 開 editor 沒自動連 / 連不上 | MCP 設定要用 **stdio 指向 vault-mcp.exe**（非直連 SSE）；重開 editor |
| SSE 沒在跑 | 跑安裝目錄 `start-vault-sse.ps1`，或重新登入（開機自啟） |
| 剛開機連不上 | SSE 首次啟動載模型約 30 秒，稍等再開 editor |
| 殘留多個 vault-mcp 進程 | 新版已修 bridge 自動退出；仍有的話關閉所有 vault-mcp 再重啟 |
| 升級後工具一用就崩 | 新版會自動偵測索引不相容並重建；舊版需 `vault-mcp.exe --reindex` |

---

原始碼 / 開發 / Docker / 完整架構：<https://github.com/gabrielchen0314/AI-Memory-Vault>
