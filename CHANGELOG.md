# Changelog

## v0.3.1 — 2026-06-10

### Added — Dimension 9e: 個人化設定當預設（源自 M90）
- 掃「作者個資/品牌/個人路徑/個人 keyword map」被當 **default** 烤進 src 程式碼（非 example）
- 掃「函數 default 吃作者個人 map」→ 陌生採用者 match 不到 = 輸出對不上
- 掃「純 CJK keyword 無語言無關 fallback」→ 非中文採用者全 miss
- 真實起源：開源 video-autopilot-kit 的 b-roll matcher 預設吃作者的中文主題表，
  採用者回報「剪出來畫面對不上字幕」。判斷句：**預設行為要對「沒有我資料的陌生人」成立**。

## v0.3 — 2026-06-02

加入 **Dimension 9: 開源/交接文件健檢**（Mode B 第 5 個 dimension）。把專案開源/交接給陌生人前，抓「我熟到忘了講」的隱性洞。

### Added — Mode B Dimension 9: 開源/交接文件健檢

- **9a 主力工具定位顛倒** — 主力工具被講成「選用/optional」、次要工具被列「必需」→ 採用者拿錯工具當主力
- **9b 隱性外部依賴沒標需求** — 核心功能背後的依賴（Computer Use / API key / 特定 app / 系統權限）沒寫進需求 → 採用者跑不起來
- **9c onboarding 無 minimum-viable** — 問卷要全填才給價值，缺「必答 vs 選填」分層或「丟給 AI 訪談你」低門檻路徑
- **9d broken folder ref** — 文件引用不存在的資料夾（承 Dimension 7 + 全域 grep 收尾紀律）

判斷句：「拿掉這個依賴，核心功能還能跑嗎？」不能 → 必標需求。「零基礎陌生人能 5 分鐘跑起來、知道先用哪條路嗎？」不能 → onboarding 沒過。

### Real origin — 自己開源 video-autopilot-kit 時踩到的

開源一套 CapCut 影片自動化 kit 後，採用者回報 3 個洞：
- 需求清單把次要的 ffmpeg 列「必需」、主力 CapCut 列「選用」→ 主次顛倒
- 全篇沒提核心依賴 **Computer Use**（CapCut 沒 API，自動化全靠它操作 GUI）→ 採用者跑不動
- 6 區問卷要全填才能開始 → 採用者嫌久

→ 這 3 個洞固化成 Dimension 9 的自動掃描。**self-demo loop** 再現。

## v0.2 — 2026-05-13

加入 **Mode B Repo Audit**（4 個新 dimension），跑 release ship 前最後 sanity check。

### Added — Mode B: Repo Audit

**Dimension 5: 私公版 sync GAP** — 適用 dual-repo skill（私人版 + 開源版）
- 自動跑 `diff -q` 比對哪些檔案 desync
- 區分 by-design diff（author signature） vs 真實 sync gap

**Dimension 6: Release 一致性** — git tag / gh release / CHANGELOG / README 對齊
- 抓「有 tag 但沒 release」「有 release 但沒 tag」
- 抓 CHANGELOG 缺最新版 entry
- 抓 README 缺最新版提及

**Dimension 7: Cross-link 完整性**
- 內部 .md ref 是否 broken
- Cross-repo URL 是否還活著

**Dimension 8: 版本標記漂移**
- 多檔案 version mention 不一致
- 例：README 提 v0.7.2 但 latest tag 是 v0.7.3 → 推 release 沒更 README

### Changed — Mode A 重新分組

原 v0.1 的 4 個 dimension 維持不變，但歸進「Mode A: Codebase Cleanup」，方便跟 Mode B 區隔。

### Real demo — 自己 audit 姐妹 repo 抓到的

跑 social-post v0.7.3 audit：
- Dimension 5: ✅ 私公版同步 OK（除了 author signature by-design）
- Dimension 6: ⚠️ CHANGELOG 缺 v0.7.3 entry（剛 release 但 doc 沒更）
- Dimension 7: ✅ 13 個外部 link 全 live
- Dimension 8: ⚠️ README + SKILL.md 都缺 v0.7.3 提及

→ cleanup-helper v0.2 立刻抓到自己姐妹 repo 的 doc drift。**self-demo loop**。

### Added — 觸發詞庫擴充

新增 audit 相關觸發詞：
- 「audit 我的 repo」
- 「check release 一致性」
- 「私公版 diff」
- 「版本對齊」
- 「release ship 前盤點」

## v0.1 — 2026-05-13

初版發布。

### Added

- **三階段工作流**：掃描（Phase 1） → 報告（Phase 2） → 建議（Phase 3）
- **4 個檢查 dimension**：
  1. 重複內容（DRY 違反）
  2. 命名不一致
  3. 可模組化區塊
  4. 過長檔案 / 函數
- **真實 case study**：用本 skill 掃描姐妹 repo `social-post skill v0.7.1`，產出 2,100 行 / 11 檔的完整 cleanup report
- **安全閘**：永遠先報告 + 等使用者確認，絕不自動修改檔案
- **觸發詞庫**：「清理 code」「找重複」「重構」「模組化」「review codebase」「掃 prompt」「我這 skill 寫好亂」「prompt 太長拆一下」

### Designed for

- prompt / SKILL.md / markdown 型 codebase（主要設計目標）
- 一般程式碼（py / ts / js）的 Phase 1-2 也可用

### 起源故事

開發 `social-post skill` 兩週累積到 2,100 行後發現：
- 「R[N]」（SKILL.md）vs「規則 [N]」（case_studies.md）= **同源系統兩套命名**
- 「鐵粉」65 次 / 「Day 1」70 次 / 「F6b」56 次散落 = 慢性 DRY 債
- case_studies.md 604 行 / formulas.md 505 行 = 接近 800 警告線

這些是 ESLint 看不到的 **semantic 技術債**。寫個 skill 自動掃才不會變一坨亂。
