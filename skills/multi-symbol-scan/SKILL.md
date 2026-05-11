# 技能：多標的掃描

## 說明
同時掃描多個標的，批次取得報價、截圖或指標資料。

## 使用時機
- 使用者說「掃描我的觀察清單」、「同時看這幾個標的」、「批次截圖」
- 晨間報告需要掃描多個標的

## 執行步驟

### 批次截圖
```
batch_run (
  symbols: ["BTCUSD", "ETHUSD", "SOLUSDT"],
  action: "screenshot"
)
```

### 批次取得 OHLCV
```
batch_run (
  symbols: ["BTCUSD", "ETHUSD", "SOLUSDT"],
  action: "get_ohlcv"
)
```

### 晨間掃描流程
1. `morning_brief` → 自動掃描 `rules.json` 的 watchlist
2. 套用偏向判斷條件
3. 輸出結構化報告
4. `session_save` → 儲存今日報告

## 輸出格式

```
掃描結果（2025-05-11 09:00）

BTCUSD  | $94,250 | RSI: 58 | 偏向：偏多  | 關鍵位：$95,500
ETHUSD  | $3,180  | RSI: 45 | 偏向：中立  | 關鍵位：$3,250
SOLUSDT | $178.50 | RSI: 62 | 偏向：偏多  | 關鍵位：$185.00

整體：偏多，主導標的為 BTC 和 SOL
```

## 注意事項
- 切換標的時等待圖表完全載入（約 1-2 秒）再讀取資料
- 批次操作盡量用 `summary: true` 減少輸出量
- 超過 10 個標的的掃描建議分批執行
