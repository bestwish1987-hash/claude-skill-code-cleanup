# Changelog

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
