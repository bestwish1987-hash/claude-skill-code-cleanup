# Changelog

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
