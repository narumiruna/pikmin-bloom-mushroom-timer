# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案簡介
Pikmin Bloom 蘑菇計時器 - 提供多組倒數計時的網頁工具，支援彈性時間輸入格式、重生/緩衝設定、和諧鈴聲與可選重複提醒功能。

## 架構與技術選擇
- **單檔案架構**：所有 HTML、CSS、JavaScript 集中於 `index.html`，純前端實作，無依賴套件
- **原生技術棧**：使用原生 JavaScript（ES6+）與 HTML5 API，不使用任何框架或建置工具
- **Web Audio API**：使用 `AudioContext`、`OscillatorNode` 與 `GainNode` 產生和諧鈴聲（C 大調和弦：C5-E5-G5-C6，搭配淡入淡出效果）
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
### Timer 物件結構（index.html:570-578）
```javascript
{
  target: Date.now() + targetSec * 1000,  // 目標結束時間戳
  original: targetSec,                    // 原始秒數
  finished: false,                         // 是否已結束
  element: null,                           // DOM 元素參考
  repeatBeep: repeatBeep,                  // 是否重複播放鈴聲
  beepInterval: null                       // 重複鈴聲的 interval ID
}
```

### 時間計算公式（index.html:683-702）
```javascript
totalSec = baseSec + (reviveMin × 60) - bufferSec
```
- `baseSec`：使用者輸入的基礎時間（解析自 hh:mm:ss、mm:ss 或 ss，支援空白分隔）
- `reviveMin`：重生時間（分鐘）
- `bufferSec`：緩衝時間（秒）
- `repeatBeep`：是否啟用重複鈴聲

### 主要函式
- `parseTime(str)` (index.html:495-507)：解析時間字串為秒數，支援三種格式與空白分隔
- `secToHHMMSS(sec)` (index.html:511-516)：將秒數轉為 `hh:mm:ss` 格式
- `playBeep()` (index.html:518-558)：播放和諧鈴聲（C 大調和弦，搭配淡入淡出）
- `addTimer(targetSec, repeatBeep)` (index.html:560-612)：新增計時器卡片，支援重複鈴聲選項
- `updateTimers()` (index.html:615-679)：每 250ms 更新所有計時器，處理重複鈴聲邏輯

## UI 狀態與樣式
### 計時器狀態（index.html:627-639）
- **正常**：黑色文字
- **警告**（剩餘 ≤ 15 秒）：橘色文字（`.warn`）搭配閃爍動畫
- **結束**（剩餘 = 0 秒）：紅色文字與背景閃爍（`.expired` + `.finished`）搭配呼吸動畫

### 顏色配置（index.html:561-568）
六種漸層卡片背景色：淡藍、淡黃、淡紫、淡綠、淡粉、淡青

## 擴充指引
### 新增計時器欄位（如標題、備註）
1. 修改表單（index.html:433-467）：新增輸入欄位
2. 更新 `timer` 物件（index.html:570-578）：加入新屬性
3. 調整 `addTimer()` 與 `updateTimers()` 邏輯以顯示新欄位

### 調整鈴聲
修改 `playBeep()` 中的 `beepSeq` 陣列（index.html:520-525），每個音符包含頻率（freq）與持續時間（dur）。可調整音階、持續時間、間隔（gap）或音量（gainNode.gain 參數）

### 調整重複鈴聲間隔
修改 `updateTimers()` 中的 `setInterval` 延遲參數（index.html:650）：將 `3000` 毫秒改為其他數值

### 修改警告閾值
調整 `updateTimers()` 中的判斷式（index.html:638）：將 `remain <= 15` 改為其他秒數

## 實作慣例
- 變數命名採駝峰式（camelCase），語意化命名
- DOM 操作集中於 `addTimer()` 與 `updateTimers()` 函式
- 即時驗證於 `input` 事件處理（index.html:469-491）
- 使用 `setInterval` 與 `clearInterval` 管理重複鈴聲
- 所有樣式皆內嵌於 `<style>` 標籤，便於單檔發布

## 重要功能說明
### 重複鈴聲機制
- 使用者可在表單勾選「重複鈴聲」選項
- 計時結束時，若啟用重複鈴聲，會每 3 秒播放一次
- 點擊 ✕ 按鈕移除計時器時，會自動清除重複播放的 interval
- 重複鈴聲的 interval ID 儲存於 `timer.beepInterval` 中

### 時間輸入彈性
- 支援冒號分隔：`1:50:30`、`5:10`、`30`
- 支援空白分隔：`1 50 30`、`5 10`、`30`
- 使用 `replace(/\s+/g, ':')` 將空白轉換為冒號
