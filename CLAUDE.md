# TradingView MCP — Claude 操作說明

81 個工具，透過 CDP（port 9222）讀取並控制本機運行的 TradingView 桌面版圖表。

## 決策樹——什麼情況用什麼工具

### 「我的圖表上現在有什麼？」
1. `chart_get_state` → 標的、時間框架、圖表類型、所有指標名稱與 entity ID
2. `data_get_study_values` → 所有可見指標的當前數值（RSI、MACD、BB、EMA 等）
3. `quote_get` → 當前標的的即時價格、OHLC、成交量

### 「圖上有哪些價位線/標記？」
自訂 Pine 指標使用 `line.new()`、`label.new()`、`table.new()`、`box.new()` 繪圖。這些內容一般的資料工具看不到，要用：

1. `data_get_pine_lines` → 指標畫出的水平價格線（支撐/壓力、交易時段價位，去重後排序高到低）
2. `data_get_pine_labels` → 帶價格的文字標記（例如「前日高 24550」、「偏多 ✓」）
3. `data_get_pine_tables` → 表格資料（例如交易時段統計、分析儀表板）
4. `data_get_pine_boxes` → 價格區間，以 {high, low} 格式輸出

使用 `study_filter` 參數鎖定特定指標（例如 `study_filter: "Profiler"`）。

### 「給我價格資料」
- `data_get_ohlcv` 加 `summary: true` → 精簡統計（最高、最低、區間、漲跌幅%、平均量、最近 5 根）
- `data_get_ohlcv` 不加 summary → 所有 K 線（用 `count` 限制數量，預設 100）
- `quote_get` → 單筆最新價格快照

### 「幫我分析圖表」（完整報告工作流）
1. `quote_get` → 當前價格
2. `data_get_study_values` → 所有指標讀數
3. `data_get_pine_lines` → 自訂指標的關鍵價位
4. `data_get_pine_labels` → 帶說明的標記價位（例如「結算價」、「ASN O/U」）
5. `data_get_pine_tables` → 交易時段統計、分析表格
6. `data_get_ohlcv` 加 `summary: true` → 價格行為摘要
7. `capture_screenshot` → 視覺確認

### 「切換圖表」
- `chart_set_symbol` → 切換標的（例如「AAPL」、「ES1!」、「NYMEX:CL1!」）
- `chart_set_timeframe` → 切換時間框架（例如「1」、「5」、「15」、「60」、「D」、「W」）
- `chart_set_type` → 切換圖表類型（K 線、平均 K 線、折線、面積、磚型等）
- `chart_manage_indicator` → 新增或移除指標（使用完整名稱："Relative Strength Index"，不是 "RSI"）
- `chart_scroll_to_date` → 跳到指定日期（ISO 格式："2025-01-15"）
- `chart_set_visible_range` → 縮放到精確日期範圍（unix 時間戳）

### 「撰寫 Pine Script」
1. `pine_set_source` → 注入程式碼到編輯器
2. `pine_smart_compile` → 自動偵測並編譯，含錯誤檢查
3. `pine_get_errors` → 讀取編譯錯誤
4. `pine_get_console` → 讀取 log.info() 輸出
5. `pine_get_source` → 讀取當前程式碼（警告：複雜腳本可能非常大）
6. `pine_save` → 儲存到 TradingView 雲端
7. `pine_new` → 建立空白指標/策略/函式庫
8. `pine_open` → 依名稱載入已存腳本

### 「用回放練習交易」
1. `replay_start` 加 `date: "2025-03-01"` → 進入回放模式
2. `replay_step` → 前進一根 K 線
3. `replay_autoplay` → 自動前進（用 `speed` 參數設定毫秒間隔）
4. `replay_trade` 加 `action: "buy"/"sell"/"close"` → 執行交易
5. `replay_status` → 查詢持倉、損益、當前日期
6. `replay_stop` → 回到即時模式

### 「掃描多個標的」
- `batch_run` 加 `symbols: ["ES1!", "NQ1!", "YM1!"]` 和 `action: "screenshot"` 或 `"get_ohlcv"`

### 「在圖上畫圖」
- `draw_shape` → 水平線、趨勢線、矩形、文字（傳入 point，可選 point2）
- `draw_list` → 查看已畫的圖形
- `draw_remove_one` → 依 ID 移除
- `draw_clear` → 全部清除

### 「管理警報」
- `alert_create` → 設定價格警報（條件："crossing"、"greater_than"、"less_than"）
- `alert_list` → 查看有效警報
- `alert_delete` → 刪除警報

### 「操作 UI」
- `ui_open_panel` → 開啟/關閉 pine-editor、strategy-tester、watchlist、alerts、trading
- `ui_click` → 依 aria-label、文字或 data-name 點擊按鈕
- `layout_switch` → 依名稱載入已存佈局
- `ui_fullscreen` → 切換全螢幕
- `capture_screenshot` → 截圖（區域："full"、"chart"、"strategy_tester"）

### 「TradingView 沒在運行」
- `tv_launch` → 自動偵測並在 Mac/Win/Linux 啟動 TradingView，附帶 CDP
- `tv_health_check` → 確認連線正常

## 情境管理規則

這些工具可能回傳大量資料。遵守以下規則避免 context 爆量：

1. **`data_get_ohlcv` 一律加 `summary: true`**，除非特別需要個別 K 線資料
2. **Pine 工具一律加 `study_filter`**（知道要讀哪個指標時）——不要掃描所有指標
3. **Pine 工具不要用 `verbose: true`**，除非使用者特別要求含 ID/顏色的原始資料
4. **避免呼叫 `pine_get_source`**——複雜腳本可能超過 200KB，只有需要編輯程式碼時才讀
5. **避免對加密/受保護指標呼叫 `data_get_indicator`**——它們的 inputs 是加密的，改用 `data_get_study_values` 讀當前數值
6. **用 `capture_screenshot` 取得視覺情境**，而不是拉大量資料——截圖約 300KB 但能看到完整畫面
7. **`chart_get_state` 在開始時呼叫一次**，取得 entity ID 後就直接參照，不要重複呼叫
8. **限制 OHLCV 請求量**——快速分析用 `count: 20`，深入分析用 `count: 100`，特別需要時才用 `count: 500`

### 輸出大小估計（精簡模式）
| 工具 | 典型輸出 |
|------|----------|
| `quote_get` | ~200 bytes |
| `data_get_study_values` | ~500 bytes（所有指標） |
| `data_get_pine_lines` | ~1-3 KB（去重後的價位） |
| `data_get_pine_labels` | ~2-5 KB（最多 50 個） |
| `data_get_pine_tables` | ~1-4 KB（格式化後的行） |
| `data_get_pine_boxes` | ~1-2 KB（去重後的區間） |
| `data_get_ohlcv`（精簡） | ~500 bytes |
| `data_get_ohlcv`（100根） | ~8 KB |
| `capture_screenshot` | ~300 bytes（回傳檔案路徑，不是圖片資料） |

## 工具使用慣例

- 所有工具回傳 `{ success: true/false, ... }`
- Entity ID（來自 `chart_get_state`）是 session 專屬的——不要跨 session 快取
- Pine 指標必須**在圖表上可見**，Pine 繪圖工具才能讀取其資料
- `chart_manage_indicator` 需要**完整指標名稱**："Relative Strength Index" 不是 "RSI"，"Moving Average Exponential" 不是 "EMA"，"Bollinger Bands" 不是 "BB"
- 截圖存到 `screenshots/` 目錄，檔名含時間戳
- OHLCV 上限 500 根 K 線，trades 每次請求上限 20 筆
- Pine labels 預設每個指標最多 50 個（用 `max_labels` 覆蓋）

## 架構

```
Claude Code ←→ MCP Server (stdio) ←→ CDP (localhost:9222) ←→ TradingView Desktop (Electron)
```

Pine 繪圖路徑：`study._graphics._primitivesCollection.dwglines.get('lines').get(false)._primitivesDataById`
