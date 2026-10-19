# fridge-recipe-notes — 冰箱便利貼食譜

單檔靜態網頁（`index.html`＋`manual.html`＋PWA）。輸入幾樣食材 → 比對內建家常食譜 → 以便利貼牆呈現 → 點開看做法、列印成 PDF、五星評分累積點數。

- 本機預覽：port **8830**（`.claude/launch.json` 的 `fridge-recipe-notes`）。
- **目標客群：女性上班族**（2026-10-19 使用者指定）。視覺走乾燥玫瑰粉／奶油底、粉彩便利貼、紙膠帶（條紋／直紋／點點三種輪替）、手寫體 `LXGW WenKai TC`；文案用「下班」語氣；有「下班快手菜」料理時間篩選。改版面時維持這個調性，不要改回軟木板咖啡色。
- **部署**（2026-10-19 使用者說「要」）：公開 repo <https://github.com/M255525/fridge-recipe-notes>，GitHub Pages（Actions workflow，push 到 master 自動部署）<https://m255525.github.io/fridge-recipe-notes/>。已補 `github-pages` environment 的 master 分支規則。無序號授權。

## 架構重點
- 預設值：`DEFAULT_INGREDIENTS`（雞蛋、番茄、洋蔥、豆腐、青蔥）、`PRESETS`（4 組快速範例）、`AI_DEFAULT_MODEL`。
- `RECIPES`：34 道內建食譜，`main`＝主要食材（決定匹配度與是否顯示）、`extra`＝配料（只加權排序）、`season`＝調味料（不比對）。
- `SYN` 同義詞表把使用者輸入轉成標準名稱（蛋→雞蛋、絞肉→豬絞肉…）；新增食譜時 `main` 要用標準名稱。`haveIt` 另外做 ≥2 字的包含比對。
- 排序 `sortResults`：命中數＋配料命中×0.3＋匹配比例；另有「我的評分高」「料理時間短」。
- 評分存 `localStorage.fridgeRecipeNotesRatings`（`{recipeId: 1~5}`），點數＝星數總和；再點同一星數＝取消。AI 食譜 id 為 `ai-<菜名hash>`，存 `fridgeRecipeNotesAiRecipes`（最多 40 筆，之後找食譜也會比對）。
- PDF：`downloadPdf` 把 `buildPrintHtml` 填進 `#printArea`，`@media print` 只顯示它，暫改 `document.title` 為「菜名_食譜」後 `window.print()`（使用者自選「另存為 PDF」），不用 jsPDF 以避開中文字型問題。
- 選用 BYOK AI：`callLLM`（Gemini/OpenRouter/OpenAI/Claude，格式同 line-sticker-studio），金鑰依服務商存在 `fridgeRecipeNotesApi.keys`；失敗自動退回規則式。執行中顯示 `#aiLoading` 狀態卡（2026-10-19 使用者要求）：`startAiLoading`／`stopAiLoading` 管輪播訊息 `AI_MSGS`、假進度條（指數趨近 92%，完成才補滿）、已等待秒數、取消（AbortController 傳進 `callLLM` 的 signal）；卡片用 sticky 固定在便利貼牆頂端，不要整張卡做晃動動畫（按鈕會變成移動目標）。
- 頂部跑馬燈共用工作區公告 GAS（滑鼠移入暫停）；頁尾警語＋創作者＋visitor-badge（`m255525.fridgerecipenotes`，手冊 `.manual`）；manual.html 有 `#creator`。
- 測試掛鉤：`window._recipe`（state、RECIPES、generate、openDetail、rate、buildPrintHtml、downloadPdf）。驗證 PDF 用 Playwright `emulateMedia('print')`＋`page.pdf()`，不要真的按列印鈕。

## 驗證紀錄（2026-10-19）
- 預設食材出 12 張便利貼；評分 4＋5 星重新整理後點數 9、已評 2 道；依評分排序、15 分鐘篩選正確；「蕃茄」被判定為已存在的番茄不重複加入。
- page.pdf 只印單一食譜、中文正常；375px 寬無水平捲動。AI 創意食譜尚未用真實金鑰測試。
