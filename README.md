# Portaly Vibe 支付 Skill

這是一個專門協助串接 Portaly Vibe 的 Skill。

## 功能
- 支援 Portaly Vibe 的支付功能。
- 提供所有金流相關 API 的詳細文件與範例程式碼。
- 讓 Coding Agent 能夠輕鬆整合 Portaly Vibe 的支付功能到各種應用中。

## 前置條件
- 需要有 Portaly Vibe 的 API 金鑰才能使用此 Skill。

### API 金鑰申請步驟
1. 前往 Portaly 管理後台：`https://portaly.cc/admin`
2. 登入後，找到上方的 `變現工具` -> `訂閱服務` 頁面。
3. 點擊 `金鑰管理` 分頁，然後點擊 `建立新 API 金鑰` 按鈕。

** 請注意，API 金鑰與 Secret 是非常敏感的資訊，請妥善保管，並且 Portaly 僅會顯示一次完整的 API 金鑰。

## 使用說明
可以使用以下指令安裝 Portaly Vibe 支付 Skill。

目前這個 Skill 支援：
- Claude Code
- Cursor
- Codex
- OpenClaw

### Claude Code
```bash
git clone https://github.com/real-engine-tw/portaly-payment-skill.git ~/.claude/skills/portaly-payment-skill
```

### Codex
```bash
git clone https://github.com/real-engine-tw/portaly-payment-skill.git ~/.codex/skills/portaly-payment-skill
```

### OpenClaw
```bash
git clone https://github.com/real-engine-tw/portaly-payment-skill.git ~/.openclaw/skills/portaly-payment-skill
```

### Cursor

在你的專案根目錄下執行以下指令，將 Skill 安裝為 Cursor 的 Project Rule：

```bash
git clone https://github.com/real-engine-tw/portaly-payment-skill.git .cursor/rules/portaly-payment-skill
```

安裝完成後，Cursor Agent 會自動載入 `.cursor/rules/portaly-payment-skill/` 中的 Skill 定義，並在相關的對話中觸發使用。

> **注意：** 此指令會將 Skill 安裝到當前專案的 `.cursor/rules/` 目錄中。如果你想在多個專案中使用，需要在每個專案中分別執行此指令。

## Skill 觸發條件
你可以使用以下的 Prompt 來觸發 Portaly Vibe 支付 Skill：

- 幫我在 Portaly Vibe 上新增一個訂閱商品

你也可以在 Prompt 中說：
-  幫我串接 Portaly Vibe 的支付功能
-  我要使用 Portaly Vibe 的支付 API
-  請協助我整合 Portaly Vibe 的支付功能

## Windows 環境的注意事項
如果你在 Windows 環境使用此 Skill，請注意 PowerShell 可能會有中文編碼的問題，導致從本機傳輸至 Portaly 的 API 請求出現亂碼。建議在 PowerShell 中執行以下指令來解決此問題：

```powershell
$OutputEncoding = [System.Text.Encoding]::UTF8
```

