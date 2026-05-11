# 技能：回放練習

## 說明
進入 TradingView 回放模式，逐 K 線練習交易，並追蹤損益。

## 使用時機
- 使用者說「我要練習交易」、「用回放模式」、「從某日期開始回放」
- 想在歷史資料上測試交易策略

## 執行步驟

### 開始回放
1. `replay_start (date: "2025-03-01")` → 進入指定日期的回放模式
2. `replay_status` → 確認進入回放並查看初始狀態

### 逐步推進
```
replay_step          # 前進一根 K 線
replay_status        # 查看當前日期、持倉、損益
```

### 自動推進
```
replay_autoplay (speed: 500)   # 每 500ms 前進一根（可調整）
```
停止自動推進：呼叫 `replay_autoplay (speed: 0)` 或 `replay_step`

### 執行交易
```
replay_trade (action: "buy",   size: 1)   # 做多 1 單位
replay_trade (action: "sell",  size: 1)   # 做空 1 單位
replay_trade (action: "close")            # 平倉
```

### 結束回放
```
replay_stop    # 回到即時模式
```

## 損益追蹤

每次呼叫 `replay_status` 都會回傳：
- 當前日期和時間
- 持倉方向和大小
- 未實現損益
- 已實現損益

## 練習建議

練習前先設定好你的交易規則（`rules.json`），然後：
1. 選一段有代表性的歷史區間
2. 用晨間報告的邏輯判斷偏向
3. 按照規則進出場
4. 記錄每筆交易的進場理由

## 注意事項
- 回放模式下無法使用即時報價工具
- 進入回放前確認 TradingView 已用 debug port 啟動
- 速度設定（speed）單位是毫秒，數字越小越快
