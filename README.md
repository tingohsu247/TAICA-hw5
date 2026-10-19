# TAICA-hw5

以下是第二個對話版本（加入**升級系統、價格動態遞增與 CPS 每秒自動加分機制**）的網頁設計規格書純文字內容：

---

# 網頁設計規格書：點擊計分與升級系統網頁 (Clicker Game Specification)

## 1. 專案概述 (Overview)

* **專案名稱**：點擊計分與升級系統網頁 (Interactive Clicker Game)
* **專案目標**：打造一個具備視覺動態反饋、手動點擊計分、以及放置類升級（Idle / Incremental）機制的網頁遊戲體驗。
* **目標使用者**：一般網頁瀏覽者，尋求簡單、直覺且具備成就感互動體驗的用戶。

---

## 2. 功能規格與邏輯 (Functional Specifications)

### 2.1 核心計分與點擊系統 (Core Clicker)

* **按鈕位置**：位於頁面垂直與水平正中央。
* **計分顯示**：按鈕上方/下方清晰顯示當前累積總分數（Current Score）。
* **點擊邏輯**：
* 使用者每點擊一次中央按鈕，總分數 $+1$。
* 按鈕觸發 `:active` 或點擊事件時，必須有明顯的縮放動畫與回饋效果。



### 2.2 升級與自動加分系統 (Upgrade & CPS System)

* **升級項目 (Upgrades)**：
* 提供至少一種升級選項（例如：「自動點擊器 Auto-Clicker」）。
* **顯示資訊**：升級名稱、目前擁有數量 (Owned Amount)、當前購買價格 (Current Cost)、每秒產出加成 (CPS - Cost Per Second)。


* **動態價格遞增邏輯**：
* **基礎價格 (Base Cost)**：預設為 10 分。
* **價格成長公式**：每購買一次，該升級的下次購買價格呈指數成長：

$$\text{Price} = \lfloor \text{Base Price} \times 1.15^N \rfloor$$



*(其中 $N$ 為目前已擁有的該升級數量)*


* **每秒自動加分 (CPS Logic)**：
* **每秒產出計算**：每一單位升級每秒提供 $+1$ 分（或指定數值）。
* 當擁有一套數量 $N$ 的升級時，系統每秒自動增加總分 $+N$ 分。
* 使用前端定時器 (例如 `setInterval` 每 1000ms 或更高頻率) 實作分數自動增長與UI同步刷新。


* **購買按鈕狀態 (Button States)**：
* 當「當前總分數 < 升級價格」時：購買按鈕呈現停用狀態 (Disabled)，且無法點擊。
* 當「當前總分數 $\ge$ 升級價格」時：購買按鈕呈現啟用狀態 (Enabled)，點擊後扣除對應分數並增加升級數量與 CPS。



---

## 3. UI 與視覺設計規格 (UI & Visual Design)

### 3.1 背景設計 (Background)

* **樣式**：全螢幕動態移動漸層背景 (Animated Gradient Background)。
* **漸層色彩組合範例**：深紫色 `#1a0b2e` $\to$ 深藍色 `#0f172a` $\to$ 暗粉色 `#2d124d`。
* **動畫實作**：使用 CSS `background-size: 400% 400%` 搭配 `@keyframes` 讓背景漸層角度/位置在 10-15 秒內持續平滑循環移動。

### 3.2 佈局結構 (Layout)

1. **頂部/中央區域**：
* 總分顯示 (Score Display)：大字體，居中顯示。
* CPS 顯示 (Scores per second)：顯示當前每秒自動產出分數量（例：`每秒 +5 分`）。
* 中央點擊按鈕 (Main Click Button)：亮色系圓形或圓角矩形按鈕，位於畫面正中央。


2. **升級控制區域 (Upgrade Panel)**：
* 位於中央按鈕下方或側邊。
* 包含升級名稱、數量標籤、購買按鈕（含動態價格標示）。



### 3.3 動畫效果 (Animations & Interactions)

* **點擊按鈕動畫**：
* Hover：微幅放大 (Scale 1.05) 與陰影加深 (Box-shadow glow)。
* Active (按下時)：縮小回彈 (Scale 0.95)，動態流暢時間約 0.1s。


* **數值變更動畫**：數字變化時具備平滑過渡或淡入淡出效果。

---

## 4. 技術實作架構 (Technical Architecture)

### 4.1 技術棧 (Tech Stack)

* **HTML5**：結構化網頁元件。
* **CSS3**：Flexbox / Grid 佈局、漸層背景動畫、Transitions & Keyframes。
* **JavaScript (ES6+)**：DOM 操作、狀態管理 (State Management)、`setInterval` 遊戲循環 (Game Loop)。

### 4.2 狀態資料結構 (State Object)

```javascript
const gameState = {
  score: 0,
  upgrades: {
    autoClicker: {
      count: 0,
      baseCost: 10,
      currentCost: 10,
      cpsValue: 1
    }
  }
};

```

### 4.3 核心計算邏輯範例 (Core Logic)

```javascript
// 1. 手動點擊
function handleClick() {
  gameState.score += 1;
  updateUI();
}

// 2. 購買升級
function buyUpgrade() {
  const upgrade = gameState.upgrades.autoClicker;
  if (gameState.score >= upgrade.currentCost) {
    gameState.score -= upgrade.currentCost;
    upgrade.count += 1;
    // 計算下次價格: Base * (1.15 ^ Count)
    upgrade.currentCost = Math.floor(upgrade.baseCost * Math.pow(1.15, upgrade.count));
    updateUI();
  }
}

// 3. 自動加分計時器 (每秒執行)
setInterval(() => {
  const totalCPS = gameState.upgrades.autoClicker.count * gameState.upgrades.autoClicker.cpsValue;
  if (totalCPS > 0) {
    gameState.score += totalCPS;
    updateUI();
  }
}, 1000);

```

---

## 5. 測試與驗收標準 (Acceptance Criteria)

1. **基本點擊測試**：點擊中央按鈕時，分數正確 $+1$，按鈕表現縮放視覺動畫。
2. **升級購買測試**：
* 分數不足時無法購買升級。
* 分數足夠時點擊升級，總分正確扣除當前價格。
* 購買後升級數量 $+1$，下次購買價格正確乘以 $1.15$ 倍並無條件捨去小數點。


3. **CPS 自動加分測試**：擁有 $N$ 個升級時，不進行任何操作下，畫面分數每秒自動增加 $N$ 分。
4. **效能與相容性**：
* 背景動畫維持 60 FPS 流暢度，無卡頓。
* 支援 Desktop 及 Mobile 觸控點擊。
