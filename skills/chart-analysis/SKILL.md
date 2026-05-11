# 技能：圖表分析

## 說明
對當前 TradingView 圖表進行完整的技術分析，整合所有可見的指標資料、Pine 繪圖和價格資訊。

## 使用時機
- 使用者說「幫我分析圖表」、「現在這個圖怎麼看」、「給我完整分析」
- 需要在截圖前先整合所有資料

## 執行步驟

### 第一步：取得基本狀態
```
chart_get_state
```
→ 取得當前標的、時間框架、所有指標名稱與 entity ID

### 第二步：取得數值資料
```
quote_get                          # 當前價格、OHLC、成交量
data_get_study_values              # 所有指標讀數（RSI、MACD 等）
data_get_ohlcv (summary: true)    # 近期 K 線統計摘要
```

### 第三步：取得自訂指標資料（如有）
```
data_get_pine_lines    # 水平價位線
data_get_pine_labels   # 文字標記
data_get_pine_tables   # 資料表格
data_get_pine_boxes    # 價格區間
```

### 第四步：視覺確認
```
capture_screenshot (region: "chart")
```

## 輸出格式

用以下結構組織分析結果：

```
【標的】BTCUSD | 時間框架：4H | 圖表類型：K 線

【價格】
- 當前：$94,250
- 今日高/低：$95,100 / $93,800
- 成交量：較均量 +23%

【指標讀數】
- RSI(14)：58.3（中性偏多）
- MACD：金叉，柱狀圖擴大
- EMA 20/50：價格在兩者之上

【關鍵價位】（來自自訂指標）
- 壓力：$95,500（前日高）
- 支撐：$93,200（前日低）

【偏向】偏多，但接近壓力區需謹慎
```

## 注意事項
- 先呼叫 `chart_get_state` 一次取得 entity ID，後續呼叫直接參照，不要重複呼叫
- `data_get_ohlcv` 加 `summary: true`，除非使用者特別要求個別 K 線
- Pine 工具加 `study_filter` 鎖定特定指標，避免掃描所有指標
