# 技能：Pine Script 開發

## 說明
協助撰寫、注入、編譯、除錯 TradingView 的 Pine Script 指標和策略。

## 使用時機
- 使用者說「幫我寫一個 Pine Script」、「修改這個指標」、「這個腳本有錯誤」
- 需要自動化 Pine Script 的開發流程

## 執行步驟

### 撰寫新腳本
1. 根據需求撰寫 Pine Script 程式碼
2. `pine_set_source` → 注入程式碼到編輯器
3. `pine_smart_compile` → 編譯並自動偵測錯誤
4. `pine_get_errors` → 如有錯誤，讀取錯誤訊息並修正
5. `pine_get_console` → 讀取 log.info() 輸出確認邏輯
6. `pine_save` → 確認後儲存到 TradingView 雲端

### 讀取並修改現有腳本
1. `pine_get_source` → 讀取當前程式碼（警告：複雜腳本可能很大）
2. 修改程式碼
3. 執行上方的步驟 2-6

### 離線分析（不需要圖表）
- `pine_analyze` → 靜態分析程式碼結構
- `pine_check` → 伺服器端編譯檢查

## Pine Script 最佳實務

```pine
//@version=5
indicator("指標名稱", overlay=true)

// 參數設定
length = input.int(14, "週期")
src = input.source(close, "來源")

// 計算
value = ta.rsi(src, length)

// 繪圖
plot(value, "RSI", color.blue)
```

## 常見錯誤處理

| 錯誤 | 解決方法 |
|------|----------|
| `Undeclared identifier` | 檢查變數名稱拼寫 |
| `Cannot call 'series' with arguments` | 確認函數參數型別 |
| `Loop is too long` | 減少迴圈迭代次數或優化邏輯 |
| `Script has too many local variables` | 把部分計算移到全域 |

## 注意事項
- 避免不必要地呼叫 `pine_get_source`，複雜腳本可能超過 200KB
- Pine Editor 面板必須開啟才能使用 Pine 工具：`ui_open_panel pine-editor open`
