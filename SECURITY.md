# 安全性說明

## 回報安全漏洞

如果你發現安全性問題，請不要直接開公開的 Issue。請透過以下方式聯繫：

請在 GitHub 的私訊或 Coocolab 的社群管道告知。

## 已知安全考量

**Chrome DevTools Protocol（CDP）**
本工具透過 CDP 連接 TradingView 桌面版。CDP 是 Chromium 應用程式的標準除錯介面，必須由使用者手動啟用（加上 `--remote-debugging-port=9222` 參數）。

**本機網路只**
CDP 連線只在 `localhost:9222` 上，不對外暴露。但請確認你的防火牆設定，避免其他程序存取這個 port。

**不儲存憑證**
本工具不儲存任何 TradingView 登入憑證或 API 金鑰。

**資料不離開本機**
所有資料處理在本機進行。TradingView 的市場資料不會傳送到任何外部伺服器。

## 使用建議

- 只在個人電腦上使用，不要在共用電腦上開啟 CDP port
- 使用完畢後關閉 TradingView 的 debug 模式（直接關閉用啟動腳本開啟的視窗）
- 定期更新本工具以獲得最新的安全修復
