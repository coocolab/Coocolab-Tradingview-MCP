# TradingView MCP — Coocolab 版

用 Claude Code 直接控制你的 TradingView 桌面版。讀圖、改圖、寫 Pine Script、跑晨間掃描——全部用對話完成。

改寫自 [@tradesdontlie](https://github.com/tradesdontlie) 的原版 [tradingview-mcp](https://github.com/tradesdontlie/tradingview-mcp)，感謝他們打造了這個工具的地基。本版本由 [Coocolab 酷可實驗室](https://www.youtube.com/@coocolab) 改寫為繁體中文，並加入晨間報告工作流、交易規則設定檔、以及 TradingView Desktop v2.14+ 的啟動修復。

> [!WARNING]
> **本工具與 TradingView Inc. 或 Anthropic 無任何官方關係。** 本工具透過 Chrome DevTools Protocol 連接你本機運行的 TradingView 桌面版。使用前請閱讀[免責聲明](#免責聲明)。

> [!IMPORTANT]
> **需要有效的 TradingView 訂閱。** 本工具不會繞過任何 TradingView 付費牆，只讀取並控制已在你電腦上執行的 TradingView 桌面版。

> [!NOTE]
> **所有資料處理皆在本機進行。** 不會傳送任何資料到外部伺服器。你的 TradingView 資料不會離開你的電腦。

---

## 30 秒看懂

```
你說：「幫我跑晨間掃描」
Claude 掃描你的觀察清單，讀取所有指標，套用你的交易規則，輸出每個標的的偏多/偏空/中立判斷。
```

```
你說：「把圖切換到 BTCUSD 日線」
Claude 幫你切換，不用手動操作。
```

```
你說：「幫我寫一個追蹤 EMA 交叉的 Pine Script」
Claude 寫好、注入編輯器、編譯、回報錯誤，一條龍。
```

---

## 這個版本新增了什麼

| 功能 | 說明 |
|------|------|
| `morning_brief` | 一鍵掃描觀察清單，讀取所有指標，根據你的規則輸出今日偏向 |
| `session_save` / `session_get` | 把每日報告存到 `~/.tradingview-mcp/sessions/`，可比對今日 vs 昨日 |
| `rules.json` | 把你的交易規則寫一次，晨間掃描每天自動套用 |
| 啟動修復 | 修復 TradingView Desktop v2.14+ 的 `tv_launch` 相容性問題 |
| `tv brief` CLI | 終端機一個指令跑完晨間報告 |

---

## 一鍵安裝

把這段貼進 Claude Code，它會幫你處理好所有事情：

```
幫我安裝 TradingView MCP Coocolab 版。
1. 把 https://github.com/coocolab/Coocolab-Tradingview-MCP.git clone 到 ~/Coocolab-Tradingview-MCP，執行 cd ~/Coocolab-Tradingview-MCP 然後 npm install。
2. 找出我的系統使用者名稱（echo $USER），然後把以下設定加到 ~/.claude/.mcp.json（跟現有 server 合併，不要覆蓋）：
   { "mcpServers": { "tradingview": { "command": "node", "args": ["/Users/YOUR_USERNAME/Coocolab-Tradingview-MCP/src/server.js"] } } }
3. 把 rules.example.json 複製成 rules.json 並打開讓我填入交易規則。
4. 重啟 Claude Code 並用 tv_health_check 確認連線。
```

或者照下面的手動步驟操作。

---

## 需求

- **TradingView 桌面版**（需要付費訂閱才能取得即時資料）
- **Node.js 18+**
- **Claude Code**（使用 MCP 工具）或任何終端機（使用 CLI）
- **macOS、Windows 或 Linux**

---

## 快速開始

### 1. Clone 並安裝

```bash
git clone https://github.com/coocolab/Coocolab-Tradingview-MCP.git ~/Coocolab-Tradingview-MCP
cd ~/Coocolab-Tradingview-MCP
npm install
```

### 2. 設定你的交易規則

```bash
cp rules.example.json rules.json
```

打開 `rules.json` 填入：
- **watchlist**（每天要掃描的標的）
- **bias_criteria**（什麼條件算偏多/偏空/中立）
- **risk_rules**（每個交易日開始前要確認的規則）

### 3. 啟動 TradingView（附帶 CDP）

TradingView 必須在開啟 debug port 的情況下運行。

**Mac：**
```bash
./scripts/launch_tv_debug_mac.sh
```

**Windows：**
```bash
scripts\launch_tv_debug.bat
```

**Linux：**
```bash
./scripts/launch_tv_debug_linux.sh
```

### 4. 加入 Claude Code

在 `~/.claude/.mcp.json` 加入以下設定（跟現有 server 合併）：

```json
{
  "mcpServers": {
    "tradingview": {
      "command": "node",
      "args": ["/Users/YOUR_USERNAME/Coocolab-Tradingview-MCP/src/server.js"]
    }
  }
}
```

Mac 查詢使用者名稱：`echo $USER`

### 5. 確認連線

重啟 Claude Code，然後問：「用 tv_health_check 確認 TradingView 是否連線」

### 6. 跑第一次晨間報告

問 Claude：「跑 morning_brief 給我今日偏向」

或從終端機：
```bash
npm link   # 安裝 tv 指令（只需要做一次）
tv brief
```

---

## 晨間報告工作流

**每次開盤前：**

1. TradingView 已開啟（附帶 debug port）
2. 終端機執行 `tv brief`（或問 Claude：「跑 morning_brief」）
3. Claude 掃描觀察清單裡的每個標的，讀取指標數值，套用你的 `rules.json` 規則，輸出：

```
BTCUSD  | 偏向：偏空  | 關鍵價位：94,200  | 注意：4H RSI 穿越 50
ETHUSD  | 偏向：中立  | 關鍵價位：3,180   | 注意：日線 Ribbon 方向
SOLUSDT | 偏向：偏多  | 關鍵價位：178.50  | 注意：守住 20 EMA 之上

整體判斷：謹慎操作。BTC 領跌，SOL 是例外——留意背離。
```

4. 儲存：「幫我存今天的報告」（使用 `session_save`）
5. 明天比對：「拿出昨天的報告」（使用 `session_get`）

---

## 工具完整列表（81 個 MCP 工具）

### 晨間報告（本版新增）

| 工具 | 說明 |
|------|------|
| `morning_brief` | 掃描觀察清單，讀取指標，回傳結構化資料供 Claude 產出今日偏向。自動讀取 `rules.json`。 |
| `session_save` | 把產出的報告存到 `~/.tradingview-mcp/sessions/YYYY-MM-DD.json` |
| `session_get` | 取得今日報告（若今日未存則取昨日） |

### 讀取圖表

| 工具 | 使用時機 | 輸出大小 |
|------|----------|----------|
| `chart_get_state` | 第一步——取得標的、時間框架、所有指標名稱與 ID | ~500B |
| `data_get_study_values` | 讀取所有指標當前數值（RSI、MACD、BB、EMA 等） | ~500B |
| `quote_get` | 取得最新價格、OHLC、成交量 | ~200B |
| `data_get_ohlcv` | 取得 K 線資料。**用 `summary: true`** 取得精簡統計 | 500B（精簡）/ 8KB（100根） |

### 自訂指標資料（Pine 畫圖）

| 工具 | 使用時機 |
|------|----------|
| `data_get_pine_lines` | 水平價格線（支撐/壓力、交易時段價位） |
| `data_get_pine_labels` | 文字標記加價格（例如「前日高 24550」、「偏多」） |
| `data_get_pine_tables` | 資料表格（交易時段統計、分析儀表板） |
| `data_get_pine_boxes` | 價格區間，以 {high, low} 格式輸出 |

**務必使用 `study_filter`** 指定目標指標：`study_filter: "我的指標名稱"`

### 圖表控制

| 工具 | 說明 |
|------|------|
| `chart_set_symbol` | 切換標的（BTCUSD、AAPL、ES1!、NYMEX:CL1!） |
| `chart_set_timeframe` | 切換時間框架（1、5、15、60、D、W、M） |
| `chart_set_type` | 切換圖表類型（K 線、平均 K 線、折線、面積、磚型） |
| `chart_manage_indicator` | 新增/移除指標。**使用完整名稱**："Relative Strength Index" 而非 "RSI" |
| `chart_scroll_to_date` | 跳到指定日期（ISO 格式："2025-01-15"） |

### Pine Script 開發

| 工具 | 步驟 |
|------|------|
| `pine_set_source` | 1. 注入程式碼到編輯器 |
| `pine_smart_compile` | 2. 編譯並自動偵測錯誤 |
| `pine_get_errors` | 3. 讀取編譯錯誤 |
| `pine_get_console` | 4. 讀取 log.info() 輸出 |
| `pine_save` | 5. 儲存到 TradingView 雲端 |

### 回放模式

| 工具 | 步驟 |
|------|------|
| `replay_start` | 從指定日期進入回放模式 |
| `replay_step` | 前進一根 K 線 |
| `replay_autoplay` | 自動前進（用 ms 設定速度） |
| `replay_trade` | 做多/做空/平倉 |
| `replay_status` | 查詢持倉、損益、當前日期 |
| `replay_stop` | 回到即時模式 |

### 多窗格、警報、畫圖、UI

| 工具 | 說明 |
|------|------|
| `pane_set_layout` | 切換格局：`s`、`2h`、`2v`、`2x2`、`4`、`6`、`8` |
| `pane_set_symbol` | 設定任一窗格的標的 |
| `draw_shape` | 畫水平線、趨勢線、矩形、文字 |
| `alert_create` / `alert_list` / `alert_delete` | 管理價格警報 |
| `batch_run` | 對多個標的/時間框架批次執行動作 |
| `watchlist_get` / `watchlist_add` | 讀取/修改觀察清單 |
| `capture_screenshot` | 截圖（區域：full、chart、strategy_tester） |
| `tv_launch` / `tv_health_check` | 啟動 TradingView 並確認連線 |

---

## CLI 指令

```bash
tv brief                           # 跑晨間報告
tv session get                     # 取得今日已存報告
tv session save --brief "..."      # 儲存報告
tv status                          # 確認連線狀態
tv quote                           # 取得當前價格
tv symbol BTCUSD                   # 切換標的
tv ohlcv --summary                 # 價格摘要
tv screenshot -r chart             # 截取圖表
tv pine compile                    # 編譯 Pine Script
tv pane layout 2x2                 # 四格圖表佈局
tv stream quote | jq '.close'      # 監控即時收盤價
```

完整指令列表：`tv --help`

---

## 故障排除

| 問題 | 解決方法 |
|------|----------|
| `cdp_connected: false` | TradingView 沒有用 `--remote-debugging-port=9222` 啟動，請使用啟動腳本 |
| `ECONNREFUSED` | TradingView 沒有在運行，或 port 9222 被擋住 |
| Claude Code 看不到 MCP server | 檢查 `~/.claude/.mcp.json` 語法，重啟 Claude Code |
| `tv` 指令找不到 | 在專案目錄執行 `npm link` |
| `morning_brief` 找不到 rules.json | 執行 `cp rules.example.json rules.json` 並填入內容 |
| `morning_brief` 觀察清單是空的 | 在 `rules.json` 的 `watchlist` 陣列加入標的 |
| 工具回傳舊資料 | TradingView 還在載入中，等幾秒再試 |
| Pine Editor 工具失敗 | 先開啟 Pine Editor 面板：`ui_open_panel pine-editor open` |

---

## 架構

```
Claude Code  ←→  MCP Server (stdio)  ←→  CDP (port 9222)  ←→  TradingView Desktop (Electron)
```

- **78 個原版工具** + **3 個晨間報告工具** = 共 81 個 MCP 工具
- **傳輸方式**：MCP over stdio + CLI（`tv` 指令）
- **連線方式**：Chrome DevTools Protocol，localhost:9222
- **無外部網路請求**——全部在本機運行

---

## 致謝

本專案改寫自 [@tradesdontlie](https://github.com/tradesdontlie) 的 [tradingview-mcp](https://github.com/tradesdontlie/tradingview-mcp)。原版工具是一切的基礎，請去幫他們的 repo 點星星。

繁體中文改寫、晨間報告工作流優化：[Coocolab 酷可實驗室](https://www.youtube.com/@coocolab)

---

## 免責聲明

本專案僅供**個人、教育及研究用途**。本工具使用 Chrome DevTools Protocol（CDP），這是所有 Chromium 應用程式內建的標準除錯介面。不對任何 TradingView 專有協定進行逆向工程，不連接 TradingView 的伺服器，也不繞過任何存取控制。

使用本軟體即表示你同意：自行負責確保符合 [TradingView 使用條款](https://www.tradingview.com/policies/)；本工具存取的是未公開記錄的內部 API，可能隨時改變；不得用於商業利用 TradingView 市場資料；作者不承擔任何帳號封禁或停權的責任。**使用風險自負。**

## 授權

MIT — 詳見 [LICENSE](LICENSE)。適用於原始碼，不適用於 TradingView 的軟體、資料或商標。
