# 安裝指南（給 Claude Code 用）

這份文件是給 Claude Code（或任何 LLM agent）安裝和設定 TradingView MCP server 的步驟說明。如果使用者請你安裝這個專案，請照以下步驟執行。

## 第一步：Clone 並安裝

```bash
git clone https://github.com/coocolab/Coocolab-Youtube.git ~/Coocolab-Youtube/Coocolab-Tradingview-MCP
cd ~/Coocolab-Youtube/Coocolab-Tradingview-MCP
npm install
```

如果使用者指定了不同的安裝路徑，用那個路徑替換 `~/Coocolab-Youtube/Coocolab-Tradingview-MCP`。

## 第二步：設定交易規則

複製範例規則檔並讓使用者填入內容：

```bash
cp ~/Coocolab-Youtube/Coocolab-Tradingview-MCP/rules.example.json ~/Coocolab-Youtube/Coocolab-Tradingview-MCP/rules.json
```

告訴使用者：「請打開 `rules.json`，填入你的觀察清單（你交易的標的）、偏向判斷條件（什麼情況算偏多/偏空）以及交易規則（每個交易日開始前要確認的事項）。這些是晨間報告每天會自動套用的設定。」

## 第三步：加入 MCP 設定

把 server 加到使用者的 Claude Code MCP 設定檔。設定檔位置是 `~/.claude/.mcp.json`（全域）或 `.mcp.json`（專案層級）。

```json
{
  "mcpServers": {
    "tradingview": {
      "command": "node",
      "args": ["/Users//Coocolab-Youtube/Coocolab-Tradingview-MCP/src/server.js"]
    }
  }
}
```

把「」換成實際的系統使用者名稱。Mac/Linux 用 `echo $USER` 查詢，Windows 用 `echo %USERNAME%`。

如果設定檔已存在且有其他 server，把 `tradingview` 這筆合併進現有的 `mcpServers` 物件，不要覆蓋其他 server。

## 第四步：啟動 TradingView 桌面版

TradingView 桌面版必須在啟用 Chrome DevTools Protocol 的情況下運行。

**自動偵測並啟動（建議）：**
MCP server 連線後，使用 `tv_launch` 工具——它會自動偵測 Mac、Windows、Linux 上的 TradingView。

**手動啟動（依平台）：**

Mac：
```bash
/Applications/TradingView.app/Contents/MacOS/TradingView --remote-debugging-port=9222
```

Windows：
```bash
%LOCALAPPDATA%\TradingView\TradingView.exe --remote-debugging-port=9222
```

Linux：
```bash
/opt/TradingView/tradingview --remote-debugging-port=9222
```

## 第五步：重啟 Claude Code

MCP server 只在 Claude Code 啟動時載入。加完設定後：

1. 結束 Claude Code（Ctrl+C）
2. 重新啟動 Claude Code
3. tradingview MCP server 應該會自動連線

## 第六步：確認連線

使用 `tv_health_check` 工具。預期回應：

```json
{
  "success": true,
  "cdp_connected": true,
  "chart_symbol": "...",
  "api_available": true
}
```

如果 `cdp_connected: false`，代表 TradingView 沒有用 `--remote-debugging-port=9222` 啟動。

## 第七步：跑第一次晨間報告

問 Claude：「跑 morning_brief 給我今日偏向」

Claude 會掃描你的觀察清單，讀取你的指標，套用 `rules.json` 的條件，輸出每個標的的偏向。

要儲存：「用 session_save 存今天的報告」

明天要取回：「用 session_get 拿昨天的報告」

## 第八步：安裝 CLI（可選）

要在任何地方使用 `tv` CLI 指令：

```bash
cd ~/Coocolab-Youtube/Coocolab-Tradingview-MCP
npm link
```

之後就可以在任何地方用 `tv status`、`tv quote`、`tv pine compile` 等指令。

## 故障排除

| 問題 | 解決方法 |
|------|----------|
| `cdp_connected: false` | 用 `--remote-debugging-port=9222` 啟動 TradingView |
| `ECONNREFUSED` | TradingView 沒在運行，或 port 9222 被擋住 |
| Claude Code 看不到 MCP server | 檢查 `~/.claude/.mcp.json` 語法，重啟 Claude Code |
| `tv` 指令找不到 | 在專案目錄執行 `npm link` |
| 工具回傳舊資料 | TradingView 可能還在載入中——等幾秒再試 |
| Pine Editor 工具失敗 | 先開啟 Pine Editor 面板（`ui_open_panel pine-editor open`） |

## 接下來讀什麼

- `rules.json` — 你的個人交易規則（使用 morning_brief 前先填好）
- `CLAUDE.md` — 工具使用決策樹（Claude Code 自動載入）
- `README.md` — 完整工具說明，含晨間報告工作流
