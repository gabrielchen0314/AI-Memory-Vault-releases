# AI Memory Vault — Releases

此 repo 為 [AI-Memory-Vault](https://github.com/gabrielchen0314/AI-Memory-Vault) 的公開發布頁面。

主 repo（原始碼）維持 Private；安裝檔與版本資訊統一發布於此。

---

## 安裝

前往 [Releases](https://github.com/gabrielchen0314/AI-Memory-Vault-releases/releases/latest) 下載最新版安裝檔：

```
AI-Memory-Vault-Setup-vX.Y.Z.exe
```

執行安裝程式，依精靈指示完成安裝即可。

---

## 自動更新（v3.13.3+）

安裝程式會自動設定更新來源，**無需額外操作**。

有新版本時，MCP 啟動後會彈出 Windows 通知：

```
AI Memory Vault 發現新版本 vX.Y.Z
目前版本：vA.B.C
[立即更新]  [稍後]
```

點「**立即更新**」即自動下載並靜默安裝。

### 手動設定（舊版安裝或其他機器）

```powershell
setx VAULT_UPDATE_SOURCE "https://raw.githubusercontent.com/gabrielchen0314/AI-Memory-Vault-releases/main"
```

---

## latest.json

`latest.json` 記錄最新版本資訊，供自動更新機制讀取：

```json
{
  "version": "3.13.3",
  "installer": "AI-Memory-Vault-Setup-v3.13.3.exe",
  "sha256": "...",
  "download_url": "https://github.com/.../releases/download/v3.13.3/AI-Memory-Vault-Setup-v3.13.3.exe",
  "published": "2026-06-14T00:00:00Z"
}
```