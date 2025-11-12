# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案簡介
Pikmin Bloom 蘑菇計時器 - 提供多組倒數計時的網頁工具，支援 hh:mm:ss 欄位輸入、重生/緩衝設定與結束提示音效。

## 架構與技術選擇
- **單檔案架構**：所有 HTML、CSS、JavaScript 集中於 `index.html`，純前端實作，無依賴套件
- **原生技術棧**：使用原生 JavaScript（ES6+）與 HTML5 API，不使用任何框架或建置工具
- **Web Audio API**：使用 `AudioContext` 與 `OscillatorNode` 產生鈴聲（三連音：880Hz、1170Hz、659Hz）
- **通知系統**：整合 Web Notifications API，於計時結束時推送瀏覽器通知

## 開發方式
```bash
# 預覽應用程式（任選一種方式）
# 方法 1: 直接在瀏覽器開啟檔案
open index.html  # macOS
xdg-open index.html  # Linux

# 方法 2: 使用 Python 簡易伺服器
python3 -m http.server 8000
# 然後瀏覽至 http://localhost:8000/index.html
```

無需 build、lint 或測試指令，純靜態網頁。

## 核心資料結構與邏輯
### Timer 物件結構（index.html:409-416）
```javascript
{
  target: Date.now() + targetSec * 1000,  // 目標結束時間戳
  original: targetSec,                    // 原始秒數
  finished: false,                         // 是否已結束
  element: null,                           // DOM 元素參考
  timeBox: null,                           // 時間顯示區塊
  removed: true                            // 是否已移除（可選）
}
```

### 時間計算公式（index.html:508）
```javascript
totalSec = baseSec + (reviveMin × 60) - bufferSec
```
- `baseSec`：使用者輸入的基礎時間（解析自 hh:mm:ss、mm:ss 或 ss）
- `reviveMin`：重生時間（分鐘）
- `bufferSec`：緩衝時間（秒）

### 主要函式
- `parseTime(str)` (index.html:355-367)：解析時間字串為秒數，支援三種格式
- `secToHHMMSS(sec)` (index.html:370-375)：將秒數轉為 `hh:mm:ss` 格式
- `playBeep()` (index.html:377-396)：播放三連音提示音
- `addTimer(targetSec)` (index.html:399-437)：新增計時器卡片
- `updateTimers()` (index.html:439-491)：每 250ms 更新所有計時器（250ms 間隔於 index.html:495 設定）

## UI 狀態與樣式
### 計時器狀態（index.html:446-450）
- **正常**：黑色文字
- **警告**（剩餘 ≤ 15 秒）：橘色文字（`.warn`）搭配閃爍動畫
- **結束**（剩餘 = 0 秒）：紅色文字與背景閃爍（`.expired` + `.finished`）搭配呼吸動畫

### 顏色配置（index.html:401-408）
六種漸層卡片背景色：淡藍、淡黃、淡紫、淡綠、淡粉、淡青

## 擴充指引
### 新增計時器欄位（如標題、備註）
1. 修改表單（index.html:306-323）：新增輸入欄位
2. 更新 `timer` 物件（index.html:409-416）：加入新屬性
3. 調整 `addTimer()` 與 `updateTimers()` 邏輯以顯示新欄位

### 調整鈴聲
修改 `playBeep()` 中的 `beepSeq` 陣列（index.html:378）與持續時間參數（index.html:379）

### 修改警告閾值
調整 `updateTimers()` 中的判斷式（index.html:448）：將 `remain <= 15` 改為其他秒數

## 實作慣例
- 變數命名採駝峰式（camelCase），語意化命名
- DOM 操作集中於 `addTimer()` 與 `updateTimers()` 函式
- 即時驗證於 `input` 事件處理（index.html:329-351）
- 所有樣式皆內嵌於 `<style>` 標籤，便於單檔發布
