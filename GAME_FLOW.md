# 宇宙守護者 GUARDIANS — 啟動流程（詳細時間軸）

> 対象檔案：`C:\Users\kth\Documents\concept\Universe Protector\index.html`
> 本文只覆蓋「由瀏覽器載入檔案 → 第一關開始、玩家首次可以操作」這一段。
> 所有行號均以該檔案實際行號為準（全檔 3103 行）。狀態變數叫 **`G.mode`**，不是 `G.state`；檔案內沒有 `G.state` 這個名字。

---

## 0. 檔案骨架（先看這張，之後所有行號都由此定位）

| 行號 | 內容 |
|---|---|
| 1 | `<!DOCTYPE html>` |
| 3–97 | `<head>`：`<meta charset="utf-8">`（4）、`<title>宇宙守護者 GUARDIANS</title>`、全部 CSS（`#wrap`、`#stage`、`.screen`、`#hud`、`#banner`、`#toast`、`#scrQuiz`、`#musBtn` 之外的全部樣式） |
| 98–203 | `<body>`：`#wrap > #stage`，內含 canvas 與 8 個畫面層 |
| **204–2782** | **`<script>` #1：主程式**（全部遊戲邏輯、資料表、狀態機、主迴圈） |
| 2783–3069 | `<script>` #2：題庫模組（xlsx/csv 匯入、派發連結、成績碼），IIFE，不影響啟動畫面 |
| 3070 | `<script>` #3：`window.__BGM__='data:audio/mpeg;base64,…'`（單行 **2,634,966 字元**，全檔 95% 體積在此） |
| 3071–3081 | `<script>` #4：BGM wrapper，覆寫 `window.bgmStart` / `window.bgmStop` |
| 3082–3087 | `<style>`：`#musBtn` |
| 3088 | `<button id="musBtn">♪</button>` |
| 3089–3101 | `<script>` #5：`#musBtn` 綁定 |
| 3102 | `</body>` |

### 8 個畫面層（`.screen`，同一時間只有一個有 `.on`）

| 行號 | id | 內容 |
|---|---|---|
| 111 | `#scrTitle` | **HTML 內已寫死 `class="screen on"`** → 預設可見，不需要 JS 開啟 |
| 128 | `#scrBank` | 題庫 |
| 144 | `#scrSet` | 設定（147 `#segDiff` 難度、148 `#segQN` 每關題數、149 `#swPen` 答錯扣命、150 `#swAd` 自動調節難度、151 `#swMus` 背景音樂、154 `#btnCsv`、155 `#btnReset`、157 `#btnSetBack`） |
| 160 | `#scrLog` | 學習記錄（162 `#logBox`、164 `#btnCsv2`、165 `#btnLogBack`） |
| 169 | `#scrSelect` | 戰機選擇（171 `#shipRow` 空容器、172 `#btnGo`） |
| 175 | `#scrOver` | 任務失敗（177 `#overInfo`、179 `#btnRetry`、180 `#btnHome`） |
| 184 | `#scrWin` | 任務完成（186 `#winInfo`、188 `#btnAgain`） |
| 192 | `#scrPause` | 暫停（194 `#btnResume`、195 `#btnQuit`） |
| 198 | `#scrQuiz` | 答題（199 `#qNo`、200 `#qText`、201 `#qOpts`、202 `#qFb`） |

常駐（不屬於任何 `.screen`，任何畫面都見得到）：`100 #cv`、`101 #hud`（分數/最高分/命）、`106 #banner`、`107 #toast`、`108 #mute`、`109 #fs`、`3088 #musBtn`。

**沒有載入畫面（loading screen）**：全檔沒有 `scrLoad` 之類節點，載入期間純粹是 `#wrap` 的 radial-gradient 背景 + 空白黑色 `#stage`。

---

## 1. 啟動時間軸

> 每一步固定七欄：觸發 / 函數與行號 / state / 畫面 / 玩家可做 / 下一步。
> 「自動」= 瀏覽器解析 HTML 時同步執行，無需玩家動作。

---

### T0｜瀏覽器取得檔案

- **觸發**：使用者開啟 `index.html`（雙擊本機檔案或經 HTTP）。
- **函數／行號**：無（瀏覽器行為）。
- **state**：尚不存在。
- **畫面**：`html,body{background:#05070f}`（11）＋ `#wrap` 的 `radial-gradient(circle at 50% 40%,#0d1730,#05070f)`（14–15）→ 深藍黑底，`:root` 內容為空。
- **玩家可做**：無。
- **下一步**：T1 解析 `<head>`。

---

### T1｜`<head>` 解析：CSS 與 viewport 建立

- **觸發**：自動。
- **函數／行號**：無 JS。關鍵在 4 行 `meta charset="utf-8"`、5 行 `viewport`（`user-scalable=no, viewport-fit=cover`）、12–96 行 CSS。
- **state**：無。
- **畫面**：仍為空底色。`#stage` 未定尺寸（CSS 只給 `position:relative;background:#000`），實際寬高要等 T11 的 `resize()` 才會有值。
- **玩家可做**：無。
- **下一步**：T2。

---

### T2｜`<body>` DOM 建立（98–203）

- **觸發**：自動。
- **函數／行號**：HTML 本身；關鍵節點 `100 #cv`、`111 #scrTitle`（已帶 `on`）。
- **state**：DOM 尚未被 JS 讀取；`G` 還不存在。
- **畫面**：`#scrTitle` 因為 HTML 已寫死 `class="screen on"`，**在 JavaScript 執行之前就已經可見**：
  - `h1`「宇宙守護者 / GUARDIANS」（112）
  - 說明文字「黑暗艦隊突襲…」（113）
  - 三個 `btn mini jmap`「數學星／科學星／人文星」與小字「直接跳關（教師示範用）」（116–120）
  - 「開始任務」（114）、「設定」「學習記錄」「題庫」（122–124）
  - `.screen` 背景為 `rgba(3,6,14,.86)` + `backdrop-filter:blur(2px)`（26–28），所以此刻畫面已經是「標題卡片」的樣子，只是背後還沒有星空與戰機。
  - `#hud`（101）與 `#mute`/`#fs` 都在 `.screen` 之前，被半透明遮罩蓋住，`#hud` 內容為靜態初始值 `SCORE 0／HI 0／♥♥♥`（102–105）。
- **玩家可做**：按鈕**還沒有 onclick**（要到 T10 才綁），點了沒反應。
- **下一步**：T3 主程式開始執行。

---

### T3｜`<script>` #1 開始執行（204）

- **觸發**：自動（inline script，parser-blocking）。
- **函數／行號**：205 `"use strict";`、206–209 註解區塊。
- **state**：全域變數即將建立；此刻 `G` 仍未定義。
- **畫面**：不變（標題卡片仍在，但按鈕仍無反應）。
- **玩家可做**：無（同步執行，通常幾十毫秒內完成 T4–T11）。
- **下一步**：T4。

---

### T4｜Canvas 與邏輯座標系建立（211–218）

- **觸發**：自動。
- **函數／行號**：
  - 211 `const stage = document.getElementById('stage');`
  - 212 `const cv = document.getElementById('cv');`
  - 213 `const ctx = cv.getContext('2d');`、214 `ctx.imageSmoothingEnabled = false;`
  - 216 `const W = 320;`（**邏輯寬度固定 320**，靠 CSS 放大）
  - 217 `let H = 480;`（稍後由 `resize()` 覆寫）
  - 218 `let DPR = 1;`
- **state**：無 `G`；canvas 仍是預設 300×150，尚未設定像素尺寸。
- **畫面**：無變化。
- **玩家可做**：無。
- **下一步**：T5。

---

### T5｜音效系統「建構但不啟動」（240–316）

- **觸發**：自動。
- **函數／行號**：
  - 240 `let AC = null, muted = false, bgmTimer = null, bgmStep = 0, bgmOn = false;`
  - 241–247 `ac()`：**只有在被呼叫時**才 `new AudioContext()`，並在 `suspended` 時 `resume()`。
  - 248–259 `tone(f0,f1,dur,type,vol)`、260–272 `noise(dur,vol,cut)`、273–293 `sfx(n)`（17 種音效 key：`shoot`、`shoot2`、`hit`、`boom`、`big`、`ebullet`、`charge`、`pickup`、`bomb`、`warn`、`quiz`、`right`、`wrong`、`click`、`power`、`life`）
  - 295–299 `BGM`（16 步 chiptune 譜）、300–301 `SCALE`、302–315 `bgmStart()`、316 `bgmStop()`
- **state**：`AC === null`（AudioContext 未建立）、`muted === false`、`bgmOn === false`。**啟動階段不呼叫 `ac()`、不呼叫 `bgmStart()`**，所以完全無聲（瀏覽器 autoplay 政策也不會報錯）。
- **畫面**：無變化。
- **玩家可做**：無。
- **下一步**：T6。

---

### T6｜像素 sprite 引擎（319–392）

- **觸發**：自動。
- **函數／行號**：
  - 319 `IDX = { o:0, B:1, D:2, c:3, w:4, y:5, k:6, r:7, g:8, p:9 }`（10 個色位代號）
  - 320 `PAL`（預設調色板 10 色）
  - 322–360 `makeSprite(o)`：`o.halfW` 陣列 → 逐行寬度；回傳 `{w,h,grid,cx}`；提供 `rect/hline/vline/line/ell` 五個繪圖 primitive。
  - 362–378 `raster(spr, pal)`：把 grid 燒成離屏 `<canvas>`。
  - 381–388 調色板：`PAL_MATH`、`PAL_SCI`、`PAL_HUM`、`PAL_VOID`、`PAL_JET1/2/3`（三架戰機各自配色）。
  - 390 `WCOL`（每種副武器一種子彈色，拖尾看色就知道用哪把）。
  - 392 `hwOf(n,h)`。
- **state**：無 `G`。
- **畫面**：無變化。
- **玩家可做**：無。
- **下一步**：T7。

---

### T7｜22 個 sprite 的像素繪製（395–825）

- **觸發**：自動（`makeSprite` 立即執行 paint）。
- **函數／行號**：
  - 玩家：395 `SPR_FALCON`、417 `SPR_INTERCEPT`、434 `SPR_VULCAN`
  - 數學星：456 `SPR_PLUS`、473 `SPR_DIV`、489 `SPR_FRAC`
  - 科學星：509 `SPR_ATOM`、524 `SPR_FLASK`、540 `SPR_CELL`、555 `SPR_SINE`、569 `SPR_CUBE`、585 `SPR_DNA`、600 `SPR_MAGNET`、617 `SPR_COMPASS`、632 `SPR_COLUMN`
  - 人文星：650 `SPR_SCROLL`、665 `SPR_LANTERN`、681 `SPR_GATE`
  - 最終關：702 `SPR_VOID`
  - Boss：725 `SPR_BOSS_GEO`、750 `SPR_BOSS_REACTOR`、773 `SPR_BOSS_COLOSSUS`、798 `SPR_BOSS_VOID`
- **state**：無 `G`。
- **畫面**：無變化（全部畫在離屏 canvas，未貼上主畫面）。
- **玩家可做**：無。
- **下一步**：T8。

---

### T8｜`GFX` raster 快取建立（828–852）

- **觸發**：自動。
- **函數／行號**：828 `const GFX = { … }`，共 22 個 key：`falcon/intercept/vulcan/plus/div/frac/atom/flask/cell/scroll/lantern/gate/sine/cube/dna/magnet/compass/column/void/bossGeo/bossReactor/bossColossus/bossVoid`。每個值都是 `raster(SPR_x, PAL_x)` 產生的 `HTMLCanvasElement`。
- **state**：無 `G`。
- **畫面**：無變化。此步是啟動階段最耗時的一段（22 張逐像素 canvas），但仍是同步、幾十毫秒級。
- **玩家可做**：無。
- **下一步**：T9。

---

### T9｜資料表與遊戲狀態物件建立（855–1230）

- **觸發**：自動。
- **函數／行號**：
  - 855–859 `SHIPS`（三架戰機）
  - 862–866 `ECOL`（敵彈顏色表）
  - 867–884 `FOE`（16 種敵人）
  - 887–894 `mulberry32`、895–901 `tiers`、902–913 `nebula`
  - 918–1151 `MAPS`（4 張地圖，含 `paint()` 背景繪製函數）
  - 1156–1187 `QUIZ`（`math`/`sci`/`hum`/`final` 四組）
  - **1192–1215 `const G = { … }`** ← 整個遊戲唯一的狀態容器
  - 1218–1225 `DEF_SET` / `SET`（從 `localStorage['gw_set']` 讀回）／`saveSet()`
  - 1222–1226 `DIFFS` 與 1227 `DF()`
  - 1230–1236 `loadBoard()` / `saveScore()`
- **state**：`G` 建立，`G.mode === 'title'`（1193 的預設值，**開機就是 title，不是靠任何函數設定**）、`G.mapIdx === 0`、`G.ship === 0`、`G.lives === 3`、`G.bombs === 2`、`G.power === 1`、`G.quizUsed === {}`、`G.hi` 由 `localStorage['gw_hi']` 讀入（1196）。`SET.diff` 預設 `'normal'`（1218），`SET.qcount` 預設 `1`，`SET.penalty` 預設 `false`，`SET.adaptive` 預設 `true`。
- **畫面**：仍為 T2 的靜態標題卡片（按鈕未綁）。
- **玩家可做**：無。
- **下一步**：T10。

---

### T10｜UI 事件綁定（1239–1340、2406–2450、2607–2717）

- **觸發**：自動（依原始碼順序）。
- **函數／行號**：
  - 1239 `renderSet()`、1267 `renderLog()`、1292 `renderBoard()`、1307 `exportCsv()`（**只定義，未執行**）
  - **1321** `#btnSet` → `renderSet(); show('scrSet')`
  - **1322** `#btnSetBack` → `show('scrTitle')`
  - **1323** `#btnLog` → `renderLog(); show('scrLog')`
  - **1324** `#btnLogBack` → `show('scrTitle')`
  - 1325/1326 `#btnCsv`/`#btnCsv2` → `exportCsv`
  - 1327–1329 `#btnReset` → `confirm()` 後清空 `QLOG`
  - **1330–1340** `#fs` → 全螢幕切換（含 `orientation.lock('portrait')`）
  - 2406–2408 取得 `#hScore`/`#hHi`/`#hLife`；2409 `updateHUD()`（定義）
  - 2420 `banner()`、2429 `toast()`
  - 2440–2450 `#scrQuiz` 元素參考、`QLOG` 從 `localStorage['gw_log']` 讀入
  - **2607–2617** `keydown`、**2618–2625** `keyup`
  - **2654–2660** 觸控／滑鼠監聽
  - **2670–2674** `.jmap` × 3（跳關）
  - **2676–2681** `#btnStart`
  - **2682–2688** `#btnGo`
  - 2689–2695 `#btnRetry`、2696–2700 `#btnHome`、2701–2705 `#btnAgain`、2706 `#btnResume`、2707–2712 `#btnQuit`
  - **2713–2717** `#mute`（切換 `muted` 並改按鈕字元 ♪ / ✕）
- **state**：`G.mode` 仍 `'title'`；`keys` 物件建立（`left/right/up/down` 全 `false`）。
- **畫面**：不變。此刻標題畫面**所有主要按鈕都已可按**（但 `#btnBank` 要到 T17 的 `boot()` 才綁，所以載入完成前「題庫」是死鍵）。
- **玩家可做**：可點「開始任務」（會進入 T19 流程，但此時 T11 尚未執行，canvas 尺寸與 `player` 都還沒準備好 → 若真的在這瞬間按下，`startMap` 會拋錯）。正常情況下這段與 T11 在同一 tick 內完成，玩家不可能搶先點到。
- **下一步**：T11。

---

### T11｜主程式尾段初始化（2760–2766）★ 真正的「開機」

- **觸發**：自動（同一 tick，讀到 2766 行為止）。
- **函數／行號**：
  - **2761 `resize();`** → 進入 220–235：`#stage` 依視窗算寬高（最闊比例 0.72，224），`H = Math.round(W*h/w)`、`DPR = min(devicePixelRatio,2)`、設定 `cv.width/height`、`ctx.setTransform(DPR,0,0,DPR,0,0)`、`imageSmoothingEnabled=false`，最後 **234 `buildStars()`**（1353–1359，星點數 `W*H/5200`）。236–237 同時註冊 `resize` 與 `orientationchange` 監聽。
  - **2762** `G.hi = +(localStorage.getItem('gw_hi') || 0);`（重讀一次，覆蓋 1196 的值）
  - **2763 `updateHUD();`** → 2409–2418：寫入 `SCORE 0`、`HI n`、`♥♥♥`（`G.lives=3`）
  - **2764 `makeBG();`** → 1360–1372：以 `G.mapIdx=0` 讀 `MAPS[0]`（數學星，`seed:11`），用 `mulberry32(11)` / `mulberry32(788)` 產生 `bg0`（`paint` layer 0）、`bg1`（layer 1）兩張 320×H 離屏 canvas，各加 `rgba(4,5,10,0.45)` 暗化；`bgKey='math_'+H` 避免重複生成。
  - **2765 `newPlayer();`** → 1374–1382：建立 `player`（`G.ship=0` → FALCON）：`x:W/2, y:H-70, r:6, cd:0, invT:2, speed:190, tx/ty 同初始位置, touch:false, shield:0, sub:'none', subT:0, sub2:'none', subT2:0`。
  - **2766 `rafId = requestAnimationFrame(loop);`** ← 主迴圈正式啟動
- **state**：`G.mode === 'title'`（未變）、`G.mapIdx === 0`、`G.ship === 0`、`bgKey === 'math_' + H`、`player !== null`、`stars` 已填滿。
- **畫面**：下一幀開始有東西可看（見 T12）。此步之前 canvas 是純黑。
- **玩家可做**：仍是標題畫面。
- **下一步**：T12。

---

### T12｜第一幀 `loop()` → `draw()`（2749–2758、2173–2402）

- **觸發**：自動（`requestAnimationFrame`，約 16.7ms 後）。
- **函數／行號**：
  - 2749 `function loop(ts)`；2750 先自我續約 `rafId = requestAnimationFrame(loop)`；2751–2754 計 `dt = (ts-last)/1000`，上限 `0.05`；2755–2756 **`if(G.hitstop > 0){…} else if(G.mode === 'play' || G.mode === 'over' || G.mode === 'win') update(dt);`** → **`title` 不在名單內，所以 `update()` 完全不執行**；2757 `draw();` 無條件執行。
  - `draw()`（2173–2402）依序：2180–2185 天空漸層（`m.sky`）→ 2187–2188 `drawLayer(bg0, scroll0)` / `drawLayer(bg1, scroll1)`（2154，兩層無縫捲動）→ 2190–2196 星點 → 2198 掉落物 → 2227 敵人 → 2274 Boss（`G.boss` 為 `null`，略過）→ **2299 `if(player && G.mode !== 'over' && G.mode !== 'win')`** 玩家 → 2325 玩家子彈 → 2348 敵彈 → 2362 粒子 → 2370 爆炸環 → 2392 閃光 → 2401 `drawImage(vigCv,0,0)` 暗角。
- **state**：`G.t === 0`、`G.mt === 0`、`scroll0 === 0`、`scroll1 === 0`（因為 `update()` 沒跑，`scroll0/1` 永遠停在 0）。`G.mode === 'title'`。
- **畫面**：**靜止的數學星背景**（紫藍星雲＋青色網格＋金屬殘骸＋星點＋暗角），上面疊著 `.screen.on` 的半透明標題卡片。因為沒有 `update()`，星空不會動。
  - 注意：玩家戰機**不會出現**。`draw()` 2299 有判斷，但 2301 行 `const blink = player.invT > 0 && Math.floor(player.invT * 12) % 2 === 0;`，`player.invT` 停在 2（`update()` 沒跑所以永不遞減），`2*12=24`、`24%2===0` → `blink === true` → 2302 `if(!blink && g)` 不成立 → 戰機隱形。
- **玩家可做**：可按 T10 已綁定的按鈕；鍵盤 `P`/`Esc` 呼叫 `togglePause()`（2662），但 `G.mode==='title'` 兩個分支都不符合，無效果；`Space` 呼叫 `useBomb()`（2589），2590 `if(G.mode !== 'play' || G.bombs <= 0) return;` 直接返回。
- **下一步**：T13。

---

### T13｜標題畫面待機（`G.mode === 'title'`）

- **觸發**：玩家停留。
- **函數／行號**：`loop` 2749 持續畫；`show('scrTitle')` 由 1322 / 1324 / 2584 / 2696 / 2701 等處觸發，2584 `if(id === 'scrTitle') renderBoard();` → 1292–1306 在 `#scrTitle` 內動態插入／更新 `#boardBox`（前 5 名最高紀錄）。
- **state**：`G.mode === 'title'`。
- **畫面**：標題卡片，若有歷史紀錄，卡片下方多一段「最高紀錄 1. 數學星 …」。
- **玩家可做**（全部入口一覽，此即標題畫面完整按鈕表）：

| 按鈕 | HTML 行 | 綁定行 | 動作 |
|---|---|---|---|
| 開始任務 `#btnStart` | 114 | **2676–2681** | `ac(); sfx('click'); buildShipCards(); show('scrSelect'); G.mode='select'` |
| 數學星 `.jmap[data-m=0]` | 116 | **2670–2674** | 跳關：`ac()`→`hideAll()`→`resetRun()`（會先建第 0 關）→`G.ship=0`→`startMap(0)`→`G.mode='play'` |
| 科學星 `.jmap[data-m=1]` | 117 | 同上 | 同上但 `startMap(1)`（注意：`bgmStart()` **沒有**被呼叫 → 跳關模式無音樂） |
| 人文星 `.jmap[data-m=2]` | 118 | 同上 | 同上但 `startMap(2)` |
| 設定 `#btnSet` | 122 | **1321** | `renderSet(); show('scrSet')`（**不改 `G.mode`**，仍是 `'title'`） |
| 學習記錄 `#btnLog` | 123 | **1323** | `renderLog(); show('scrLog')`（同樣不改 `G.mode`） |
| 題庫 `#btnBank` | 124 | **3021–3025**（`boot()` 內） | `ac(); 填 #inCls; show('scrBank')` → **見 §5 已知缺陷 (1)：這個畫面實際打不開** |
| 靜音 `#mute` | 108 | **2713–2717** | 切換 `muted`，圖示 ♪↔✕（`sfx`/`tone`/`noise` 全部 early-return） |
| 全螢幕 `#fs` | 109 | **1330–1340** | `requestFullscreen()` + 鎖直向 |
| 背景音樂 `#musBtn` | 3088 | **3098**（`apply()` 在 3099 首次執行） | `window.__MUSIC__.mute(!on)` |
| 說明／教學 | — | — | **沒有這個按鈕**；教學文字只有 HTML 113 的一句 tagline |

- **下一步**：T14（背景自動發生）或玩家按「開始任務」→ T19。

---

### T14｜`<script>` #3：2.6MB base64 BGM 解析（3070）

- **觸發**：自動（T3 的主程式 script 結束後，parser 繼續讀到 3070）。
- **函數／行號**：3070 單行，指派 `window.__BGM__`。
- **state**：`window.__BGM__` 由 `undefined` 變成長 base64 字串。
- **畫面**：可能有一瞬間卡頓（2.6MB 字串解析）；標題畫面已經在跑，不影響。
- **玩家可做**：同 T13。
- **下一步**：T15。

---

### T15｜`<script>` #4：BGM wrapper 覆寫（3071–3081）

- **觸發**：自動。
- **函數／行號**：
  - 3076 `var origStop = window.bgmStop;`（= 原 316 行函數）
  - **3077** `window.bgmStart = function(){ try{ origStop(); }catch(e){} if(muted) return; if(window.ac) ac(); A.play().catch(){}; }`
  - **3078** `window.bgmStop = function(){ try{ origStop(); }catch(e){} A.pause(); }`
  - 3079 `window.__MUSIC__ = { el: A, mute(v){…} }`
- **state**：`window.bgmStart` / `window.bgmStop` 被替換。`A = new Audio(__BGM__)`，`loop=true`、`volume=0.30`（3074），但**未播放**（要等 T21 的 `bgmStart()`）。
- **畫面**：無變化。
- **玩家可做**：此刻起 `#musBtn` 才有作用（3089 script 在下一行）。
- **下一步**：T16。
- **備註（重要）**：3077 內呼叫的是 `origStop()`（即原 `bgmStop`），**不是**原 `bgmStart`，所以 302–315 的 chiptune 序列器永遠不會啟動；實際 BGM 只有這條 MP3。詳見 §5 (2)。

---

### T16｜`<script>` #5：`#musBtn` 綁定（3089–3101）

- **觸發**：自動。
- **函數／行號**：3091 取按鈕、3093–3097 `apply()`、**3098 `b.onclick`**、3099 首次 `apply()`（預設 `on = true`，所以立即 `__MUSIC__.mute(false)`；此時 `A.play()` 會因 autoplay 政策被瀏覽器擋下，`.catch()` 吞掉，無聲但也不報錯）。
- **state**：module 內部 `on === true`；與主程式的 `muted`、`SET` 無任何同步。
- **畫面**：右上角出現固定位置的 ♪ 圓角按鈕（CSS 3083–3086）。
- **玩家可做**：點 ♪ 切換 BGM。
- **下一步**：T17。

---

### T17｜`DOMContentLoaded` → `boot()`（3052–3054、2941–3050）

- **觸發**：自動（HTML 解析完成後）。
- **函數／行號**：
  - **3054** `if(document.readyState === 'loading') document.addEventListener('DOMContentLoaded', boot); else boot();`
  - **2941–2950**：`#swMus`（設定頁的背景音樂開關）綁定，內部閉包 `on = true`，`__MUSIC__.mute(!on)`
  - 2954–2963 `#fBank`（xlsx/csv 匯入）、2965–2973 `#inSheet`（Google Sheet ID）、2975 `#inCls`、2977–2994 `#segSubj`
  - 2996–3004 `#btnLink`（派發連結）、3006–3017 `#btnExp`（匯出派發 HTML）、3019 `#btnBankBack`、**3021–3025 `#btnBank`**
  - 3027–3049 為 `#scrOver`/`#scrWin` 掛 `MutationObserver`，畫面一開啟就插入「成績碼」區塊
- **state**：`G.mode` 仍 `'title'`（`boot()` 不碰 `G`）。
- **畫面**：無變化（只是多了很多「隱藏畫面」的事件）。
- **玩家可做**：此時「題庫」鍵才活起來（但仍打不開，見 §5）。
- **下一步**：T18。

---

### T18｜延遲 60ms：讀 `#b=` hash 或注入題庫（3055–3067）

- **觸發**：計時（`setTimeout(..., 60)`）。
- **函數／行號**：3056 取 `window.__INJECT__`；3058–3060 若 URL 有 `#b=` 則 `dec(h.slice(3))`（2933–2939 base64url→JSON）；3061–3066 `applyBank(cur)`（2903–2907：**直接覆寫 `QUIZ.math`/`QUIZ.sci`/`QUIZ.hum`**，並 `G.quizUsed = {}`），再 `msg()` 顯示題數。
- **state**：若帶題庫 → `QUIZ` 三科被整批替換（`QUIZ.final` 不受影響）；`G.quizUsed === {}`。
- **畫面**：標題畫面不變；只有 `#scrBank` 內的 `#bankMsg` 更新（玩家此刻看不到）。
- **玩家可做**：同 T13。
- **下一步**：等玩家按「開始任務」→ T19。

---

### T19｜玩家按「開始任務」→ 戰機選擇（`G.mode = 'select'`）

- **觸發**：**點擊 `#btnStart`（HTML 114）**。
- **函數／行號**：**2676–2681**
  ```js
  ac(); sfx('click'); buildShipCards(); show('scrSelect'); G.mode = 'select';
  ```
  - `ac()`（241）→ 這一下**第一次建立 AudioContext**（使用者手勢內，解鎖音訊）
  - `buildShipCards()`（**2720–2745**）：清空 `#shipRow`，對 `SHIPS`（855–859）三架各建一張 `.shipcard`：
    - 2728–2734 用 `GFX[s.spr]` 畫 2× 放大的預覽 canvas（CSS 再放大 3×，`imageSmoothingEnabled=false` 保像素感）
    - 2735–2737 名稱（`s.name`）與簡介（`s.desc`）
    - 2738–2742 `onclick` → `G.ship = i; sfx('click');` 重設 `.sel` class
  - `show('scrSelect')`（**2579–2585**）：把 `scrTitle/scrSelect/scrOver/scrWin/scrPause/scrQuiz/scrSet/scrLog` 八個逐一 `classList.toggle('on', k===id)` → **標題卡片關掉、選機卡片打開**
- **state**：**`G.mode === 'select'`**、`G.ship === 0`（預設選中 FALCON，2723 的 `i === G.ship` 決定 `.sel`）。
- **畫面**：`#scrSelect`（169–173）：「選擇戰機」、三張卡片橫排（FALCON 均衡／INTERCEPT 高速／VULCAN 重火力，各附像素預覽圖）、下方黃框「出擊」按鈕。背景仍是 T12 那張靜止的數學星。
- **玩家可做**：點卡片換機（2713?→ 2739 的 `G.ship = i`）；點「出擊」→ T21。
- **下一步**：T21（或直接按跳關 → T20b）。
- **注意**：`#scrSelect` **沒有返回鍵**（169–173 只有 `#btnGo`），選機畫面只能前進，不能回標題。

---

### T20｜（替代路徑）按「數學星／科學星／人文星」跳關

- **觸發**：**點擊 `.jmap`（HTML 116–118）**。
- **函數／行號**：**2670–2674**
  ```js
  ac(); sfx('click'); hideAll(); resetRun(); G.ship = 0; startMap(+el.dataset.m); G.mode = 'play';
  ```
  - `hideAll()`（2586）→ `show('')`，八個畫面全部關閉
  - `resetRun()`（**1383–1391**）→ 見 T21 說明（內部會 `startMap(0)`）
  - `startMap(i)`（**1392–1406**）再切到目標關
- **state**：`G.mode === 'play'`、`G.mapIdx === data-m`、`G.ship === 0`（**強制 FALCON**，即使之前選過別台）、`G.lives === DF().lives`（`normal` → 4）、`G.revives === 2`。
- **畫面**：直接進第 N 關（跳過選機畫面）。`banner()` 顯示關名。
- **玩家可做**：同 T22。
- **下一步**：T22。
- **差異**：**不呼叫 `bgmStart()`**，所以教師示範模式下沒有背景音樂；`#musBtn` 的 ♪ 也不會恢復音樂（它只 `A.play()`）。

---

### T21｜玩家按「出擊」→ 真正開始遊戲（`G.mode = 'play'`）★

- **觸發**：**點擊 `#btnGo`（HTML 172）**。
- **函數／行號**：**2682–2688**
  ```js
  sfx('click'); hideAll(); resetRun(); G.mode = 'play'; bgmStart();
  ```
  1. `hideAll()`（2586）
  2. **`resetRun()`（1383–1391）**：
     - 1385 `G.score=0; G.lives=DF().lives; G.bombs=2; G.power=1; G.revives=2;`（`DIFFS.normal.lives = 4`）
     - 1386 `G.hyper=0; G.hyperT=0; G.hyperReady=0;`
     - 1387 `G.t=0; G.mapIdx=0; G.quizUsed={}; G.kills=0;`
     - 1388 `G.combo=0; G.comboT=0; G.maxCombo=0; G.runLog=[]; G.adapt=0;`
     - 1389 `pbul=[]; ebul=[]; foes=[]; bits=[]; drops=[];`
     - **1390 `startMap(0);`**
  3. **`startMap(0)`（1392–1406）**：
     - 1393–1396 `G.mapIdx=0; G.mt=0; G.boss=null; G.bossT = MAPS[0].dur (78 秒); G.midDone=false;`
     - 1397 `spawnT=1.2; spawnRate=1.6;`
     - 1398 `ebul=[]; foes=[]; pending=[];`
     - 1399–1400 `bgKey=''; makeBG(); scroll0=0; scroll1=0;`
     - 1401 `newPlayer();`（重建戰機，套用 `G.ship` 的 `speed`；`invT=2` 開場 2 秒無敵）
     - 1403 `banner('數學星','MATHEMATICA')` → 2420–2427，顯示 2.2 秒
     - 1404 `setTimeout(() => toast(MAPS[0].intro), 1500)` → 1.5 秒後顯示「數學星：軌道被算式鎖死…」1.6 秒
     - 1405 `updateHUD();`
  4. 2686 `G.mode = 'play';`
  5. **2687 `bgmStart();`** → 呼叫 T15 覆寫版 → `origStop()`（清掉 chiptune timer，本來就沒跑）→ `ac()` → `A.play()`，MP3 BGM 開始
- **state**：**`G.mode === 'play'`**、`G.mapIdx === 0`、`G.mt === 0`、`G.bossT === 78`、`G.lives === 4`（normal）、`G.revives === 2`、`G.power === 1`、`spawnT === 1.2`、`spawnRate === 1.6`、`player.invT === 2`。
- **畫面**（進入 play 後的頭幾幀）：
  - 背景開始捲動（`update()` 1692–1693 遞增 `scroll0`（×0.35）、`scroll1`（×1.8）），星點下墜
  - `#banner` 大字「數學星」＋小字「MATHEMATICA」淡入（`transition:opacity .4s`，42–47 行 CSS）
  - 1.5 秒後 `#toast` 出現關卡介紹
  - `#hud` 顯示 `SCORE 0`／`HI`／`♥♥♥♥  ✦2`
  - 玩家戰機在畫面下方中央 (`y = H-70`)，因為 `invT=2` 而閃爍（**這次 `update()` 會跑，`invT` 會遞減，所以會正常閃爍後現身**）
  - 1.2 秒後 `spawnT` 到期 → `spawnWave()`（1432）第一批敵人從上方進入
- **玩家可做**：**這就是「首次可操作」的時刻** —— `G.mode` 變成 `'play'` 的下一幀（`loop` 2756 通過判斷）開始：
  - 鍵盤：`←→↑↓` 或 `WASD` 移動（`keydown` 2608–2617 設 `keys.*`，`update` 1713–1728 套用 `SHIPS[G.ship].speed`，位置夾在 `x∈[12,W-12]`、`y∈[30,H-16]`）
  - 觸控／滑鼠：`touchstart`/`mousedown` → `onDown`（**2635–2645**）把 `player.touch=true`、`tx/ty` 設在手指上方 46px；`touchmove`/`mousemove` → `onMove`（2646–2652）；放開 → `onUp`（2653）
  - `Space` → `useBomb()`（2589）
  - `P` / `Esc` → `togglePause()`（2662），`G.mode` → `'pause'`
  - 射擊**全自動**（`update` 1729–1748，依 `G.power` 1/2/3 發 1/2/3 發）
- **下一步**：T22（第一關進行中；Boss 於 `G.mt >= 78` 出現，Boss 死後 `onBossDead`（2519）出題 → 下一關）。

---

### T22｜第一關開場後的頭兩秒（時間軸收束）

| 相對時間 | 事件 | 行號 |
|---|---|---|
| 0s | `G.mode='play'`，`player.invT=2`（無敵＋閃爍），背景開始捲動 | 2686、1712、1692 |
| 0–0.4s | `#banner` 淡入（「數學星 / MATHEMATICA」） | 2424、CSS 43–47 |
| 1.2s | `spawnT` 到期 → `spawnWave()` 第一批（`FORMS` 隨機：`line`/`v`/`column`/`spiral`/`single`） | 1924–1928、1431–1455 |
| 1.5s | `#toast` 顯示 `MAPS[0].intro` | 1404、2429 |
| 2.0s | `player.invT` 歸零，開始會受傷 | 1712、1662 |
| 2.2s | banner 淡出 | 2426 |
| 3.1s | toast 淡出 | 2434 |
| 39s | 中樞艦（`G.mt > bossT*0.5`，且 `< bossT-7`）→ `spawnFoe(pool[0], W/2, true)` + toast「中樞艦接近」 | 1930–1935 |
| 74s | `spawnT` 停止生兵（`G.mt < G.bossT - 4` 不再成立） | 1925 |
| 78s | `G.mt >= G.bossT` → `spawnBoss()`，「警告：旗艦接近」，Boss HP 80 | 1937–1941、1571–1580 |

---

## 2. `G.mode` 完整狀態清單

檔案內只有 **7 個值**，全部寫在 1193 行的註解裡：`// title | select | play | pause | quiz | over | win`。
**不存在** `dead`、`clear`、`loading`、`boss`、`gameover` 這些值（「Boss 死後」是 `G.bossDeadT` 計時，不是 state）。

| state | 對應畫面（DOM） | 進入條件 | 離開條件 | 主要函數 | 行號 |
|---|---|---|---|---|---|
| `title` | `#scrTitle`（HTML 已 `on`） | ① 開機預設值 ② `#btnHome` ③ `#btnAgain` ④ `#btnQuit` | 按 `#btnStart` → `select`；按 `.jmap` → `play` | 無專屬 update；`draw()` 照畫、`update()` 不執行 | 1193、2696–2699、2701–2704、2707–2711 |
| `select` | `#scrSelect` | `#btnStart`（2676–2681） | 按 `#btnGo` → `play`（無返回路徑） | `buildShipCards()` | 2680、2720–2745 |
| `play` | 無（`hideAll()`，只剩 canvas + HUD） | ① `#btnGo` ② `#btnRetry` ③ `.jmap` ④ `togglePause()` 第二段 ⑤ `onBossDead` 的 quiz callback ⑥ `gameOver` 復活 callback ⑦ `__PW__.start()` | ① `togglePause()` → `pause` ② `showQuiz()` → `quiz` ③ `gameOver()` → `over`（或先進 `quiz` 復活） ④ `missionClear()` → `win` | `update(dt)` 1685–2038、`draw()` | 2686、2693、2673、2664、2525、2546、2773 |
| `pause` | `#scrPause` | `togglePause()`（2663） | `#btnResume`（2706）或再按 `P`/`Esc`（2664）→ `play`；`#btnQuit`（2707）→ `title` | 無（`update()` 不跑，畫面凍結） | 2663–2665 |
| `quiz` | `#scrQuiz` | `showQuiz()`（**2467**） | `answer()` 內 `setTimeout 2400ms` 後：`chain>0` → 再出題；否則執行 callback（`onBossDead` 的下一關／`gameOver` 的復活）；若 `G.lives<=0` → `gameOver()` | `showQuiz()` 2452–2481、`answer()` 2482–2514 | 2467、2507–2513 |
| `over` | `#scrOver` | `gameOver()` 且 `G.revives === 0`（**2558**） | `#btnRetry`（2689）→ `play`；`#btnHome`（2696）→ `title` | `gameOver()` 2537–2567 | 2558 |
| `win` | `#scrWin` | `missionClear()`（**2569**，由 `onBossDead` 在 `G.mapIdx >= MAPS.length-1` 時呼叫，2522–2523） | `#btnAgain`（2701）→ `title` | `missionClear()` 2568–2578 | 2569 |

### 誰讀 `G.mode`（全部）

| 讀取點 | 行號 | 判斷式 |
|---|---|---|
| `loop` | 2756 | `G.mode === 'play' \|\| 'over' \|\| 'win'` 才 `update(dt)` |
| `update` | **1700** | `if(G.mode !== 'play') return;`（背景捲動／星點在 1692–1698 已先跑，所以 `over`/`win` 時背景仍在動，實體全凍結） |
| `draw` | 2299 | `G.mode !== 'over' && G.mode !== 'win'` 才畫玩家 |
| `useBomb` | 2590 | `G.mode !== 'play'` 直接 return |
| `onDown` | 2636 | `G.mode !== 'play'` 直接 return |
| `onMove` | 2647 | `G.mode !== 'play' \|\| !player.touch` return |
| `togglePause` | 2663–2664 | `play`↔`pause` |

### 「非 state」但控制流程的旗標

| 變數 | 行號 | 作用 |
|---|---|---|
| `G.bossT` | 1207，設於 1395（= `MAPS[i].dur`）、1578（Boss 生成後設 9999）、2776（除錯 `bossNow()` 設 0） | 該關「Boss 倒數秒數」 |
| `G.bossDeadT` | 1208，設 0.9 於 1918 | Boss 死後 0.9 秒延遲，1703–1706 才 `onBossDead()` |
| `G.midDone` | 1213，1396 重置 | 中樞艦只出一次 |
| `G.hitstop` | 1198 | >0 時 `loop` 跳過 `update()`（2755） |
| `G.quizUsed` | 1210 | 每科已出過的題目索引，防重複 |
| `G.wrongQ` | 2504（動態建立） | 答錯的題目會被優先再出一次（2455–2456） |
| `G.revives` | 1385（=2） | >0 時死亡改為出題復活（2539） |

---

## 3. 所有輸入處理位置

| 輸入 | 綁定行號 | 處理函數／行號 | 說明 |
|---|---|---|---|
| `keydown` | **2608–2617** | 直接寫 inline | `←/a`、`→/d`、`↑/w`、`↓/s` 設 `keys.*`；`Space`→`useBomb()`（2612）；`p`/`Escape`→`togglePause()`（2613）；方向鍵與空白 `preventDefault()` |
| `keyup` | **2618–2625** | inline | 對應放開 |
| `touchstart` on `#cv` | **2654** | `onDown` 2635–2645 | `preventDefault`、`ac()`、`player.touch=true`、手指上方 46px |
| `touchmove` on `#cv` | **2655** | `onMove` 2646–2652 | 只在 `player.touch` 為真時跟隨 |
| `touchend` / `touchcancel` | **2656** / **2657** | `onUp` 2653 | `player.touch=false` |
| `mousedown` on `#cv` | **2658** | `onDown` | 桌機拖曳 |
| `mousemove` on `window` | **2659** | `onMove` | — |
| `mouseup` on `window` | **2660** | `onUp` | — |
| `#btnStart` click | 2676 | 2676–2681 | → `select` |
| `.jmap` click ×3 | 2670 | 2670–2674 | → `play`（跳關） |
| `#btnGo` click | 2682 | 2682–2688 | → `play`（正式開始） |
| `#btnRetry` | 2689 | 2689–2695 | → `play` |
| `#btnHome` | 2696 | 2696–2700 | → `title` |
| `#btnAgain` | 2701 | 2701–2705 | → `title` |
| `#btnResume` | 2706 | `togglePause()` | `pause`→`play` |
| `#btnQuit` | 2707 | 2707–2712 | `bgmStop()` + → `title` |
| `#mute` | 2713 | 2713–2717 | 切 `muted` |
| `#fs` | 1330 | 1330–1340 | 全螢幕 |
| `#btnSet` / `#btnSetBack` | 1321 / 1322 | — | 設定頁（不改 `G.mode`） |
| `#btnLog` / `#btnLogBack` | 1323 / 1324 | — | 記錄頁（不改 `G.mode`） |
| `#btnCsv` / `#btnCsv2` | 1325 / 1326 | `exportCsv()` 1307–1320 | — |
| `#btnReset` | 1327 | 1327–1329 | 清 `QLOG` |
| `#segDiff` / `#segQN` / `#swPen` / `#swAd` | 1240–1266（`renderSet()` 內動態建立） | — | 改 `SET.diff` / `SET.qcount` / `SET.penalty` / `SET.adaptive`，即時 `saveSet()` |
| `#swMus` | 2948 | 2945–2953（`boot()`） | 只呼叫 `__MUSIC__.mute()`，**不寫入 `SET`** |
| `#musBtn` | 3098 | 3090–3100 | 同 `#swMus` |
| `#fBank` / `#inSheet` / `#inCls` / `#segSubj` / `#btnLink` / `#btnExp` / `#btnBankBack` / `#btnBank` | 2954–3025 | `boot()` | 題庫模組（不影響啟動畫面） |
| `resize` / `orientationchange` | 236 / 237 | `resize()` 220–235 | 重算 `H`/`DPR`、重建星點（**不會重畫 `bg0/bg1`，但 `draw()` 每幀用 `bgKey === m.id+'_'+H` 的邏輯，下次 `makeBG()` 才會重做**） |

---

## 4. 跳過／自動前進的邏輯（重點：**幾乎沒有**）

1. **沒有任何倒數自動開始**：全檔沒有 `setTimeout` 把 `G.mode` 由 `title` 推去 `select`。唯一「自動前進」的位置是 **`answer()` 的 2400ms 延時**（2507–2513）：答完題 2.4 秒後自動執行 callback（下一關／復活），以及 **`G.bossDeadT = 0.9`**（1918 → 1703–1706）Boss 死後 0.9 秒自動跳結算。
2. **跳關捷徑**：`.jmap`（2670）讓教師直接進第 1/2/3 關，**強制 `G.ship = 0`**（FALCON），且**略過 `bgmStart()`**。
3. **開場兩段延時**：`banner()` 固定 2200ms 淡出（2426）、`toast()` 固定 1600ms 淡出（2434）、`startMap` 內 `intro` toast 延後 1500ms（1404）。
4. **題庫注入延時**：`boot()` 後 `setTimeout(..., 60)`（3055）才讀 `#b=` / `__INJECT__`。
5. **`#scrQuiz` 題數**：`onBossDead` 傳 `chain = Math.max(0, SET.qcount - 1)`（2528），所以「每關校驗題數 1/2/3」= 一次結算連續出 1/2/3 題（`answer` 2510 遞減 `chain`）。
6. **沒有「載入進度條」或 `window.onload` 等待**：主程式 204–2782 是同步 inline script，`requestAnimationFrame` 在 2766 就啟動。

---

## 5. 啟動流程中實際觀察到的缺陷（照住改之前先知）

1. **「題庫」按鈕打不開畫面**：`#btnBank` → `show('scrBank')`（3024），但 `show()`（2579–2585）的 id 清單只有
   `['scrTitle','scrSelect','scrOver','scrWin','scrPause','scrQuiz','scrSet','scrLog']`（2580），**不含 `scrBank`**。
   結果：按「題庫」會把其他畫面全關掉，`#scrBank` 卻不會被加上 `.on` → 玩家看到的是「標題卡片消失、只剩遊戲背景」。
   → 修法：把 `'scrBank'` 加進 2580 的陣列。
2. **chiptune BGM 永遠不會響**：3077 覆寫 `window.bgmStart` 時呼叫的是 `origStop()`（= 原 `bgmStop`），而非原 `bgmStart`（302–315）。所以 302 的 16 步序列器是死碼，`bgmStep`/`bgmTimer` 永不啟動；BGM 只有 3070 的 MP3。
   → 若想恢复 chiptune，把 3076 改成 `var origStart = window.bgmStart;`，3077 內改叫 `origStart()`。
3. **選機畫面無返回鍵**：`#scrSelect`（169–173）只有 `#btnGo`，進去了就只能打。
4. **標題畫面看不到戰機**：`player.invT` 停在 2（`update()` 在 `title` 不跑），`draw()` 2301 的閃爍判定剛好判成「隱藏」，所以 2299 的玩家繪製被跳過。看到的是「沒有戰機的靜止星空 + 標題卡片」。
5. **三套音樂開關互不同步**：`#mute`（`muted`，240／2713）、`#swMus`（`boot()` 閉包 `on`，2945）、`#musBtn`（3090 閉包 `on`）各自為政；`SET` 裡也**沒有**音樂欄位（`DEF_SET` 1218 只有 `diff/penalty/qcount/adaptive`）。設定頁的「背景音樂」開關重開頁面後會回到「開」。

---

## 6. 未完全確定的地方

| 項目 | 狀況 |
|---|---|
| `#scrBank` 是否**故意**用別的方式顯示（例如 CSS `:target` 或外部注入） | 已全文搜尋 `scrBank`，只有 128（HTML）與 3024（`show` 呼叫）兩處，沒有任何 `.on` 賦值 → 判定為缺陷，不是隱藏機制。若不放心，可在 3024 之後臨時 `console.log(document.getElementById('scrBank').className)` 確認。 |
| 載入期間的實際耗時 | 依裝置而定：22 張 sprite raster（T8）＋ 2 張 320×H 背景（T11 `makeBG`）＋ 2.6MB base64 解析（T14）。程式碼內沒有任何計時或 loading UI，無法由碼判斷，需要實機 profiling。 |
| `resize()` 在遊戲中途觸發時背景是否會失真 | `makeBG()` 有 `bgKey === m.id + '_' + H` 的防重複（1362），但 `resize()`（220–235）本身**不呼叫** `makeBG()`；`bg0/bg1` 會保持舊 `H` 尺寸，直到下次 `startMap()`。`drawLayer()` 只做垂直循環貼圖，理論上會出現接縫，未實機驗證。 |
| `player.tilt` / `player.px` | 在 `draw()` 2321–2324 被計算但 `ctx.rotate(0)`（2326），傾斜效果實際沒套用（死碼）。不影響啟動流程。 |

---

## 附：啟動流程一頁速查（給改 code 的人）

```
HTML/CSS(1-97) → DOM(98-203, #scrTitle 已 .on)
  → <script>#1(204-2782)
      canvas/常數(211-218)
      sprite 引擎(319-392) → 22 個 SPR(395-825) → GFX raster(828-852)
      資料表(855-1230)：SHIPS FOE MAPS QUIZ G(mod:title) SET DIFFS
      UI 綁定(1239-1340, 2608-2717)
      初始化(2761 resize→buildStars | 2764 makeBG | 2765 newPlayer | 2766 rAF(loop))
  → loop(2749)：title 時只 draw()，不 update()
      draw(2173)：bg0/bg1 + 星 + 暗角（靜止）
  → <script>#3(3070 __BGM__) → #4(3071-3081 覆寫 bgmStart/Stop) → #5(3089 musBtn)
  → DOMContentLoaded → boot(2941) → +60ms 讀 #b= 題庫(3055-3067)
  → [玩家] #btnStart(2676) → G.mode='select' + buildShipCards(2720)
  → [玩家] 卡片(2739 G.ship=i) → #btnGo(2682)
      hideAll → resetRun(1383) → startMap(0)(1392) → G.mode='play' → bgmStart()
  → 下一幀起 loop 2756 通過 → update(1685) → 玩家可操作
```
