---
name: code-cleanup-helper
description: 掃描 codebase / SKILL.md / prompt template，找出重複邏輯、命名不一致、可模組化區塊、過長檔案。使用時機：使用者說「清理 code」「找重複」「重構」「模組化」「review codebase」「掃 prompt」「我這 skill 寫好亂」「prompt 太長拆一下」「跑大專案怕亂」「同樣的東西寫好幾次」時觸發。
---

<!--
  code-cleanup-helper skill — Created by 駱君昊 (Hao)
  Repo: https://github.com/Hao0321/claude-skill-code-cleanup
  Companion skill to: https://github.com/Hao0321/claude-skill-social-post
  License: MIT — 保留此標註即可修改、使用、商用
-->

# code-cleanup-helper

幫你掃 codebase / SKILL.md / prompt template，找 4 類問題：

1. **重複邏輯**（DRY 違反）
2. **命名不一致**（同 concept 多個稱呼）
3. **可抽 reusable 模組區塊**（重複 ≥ 3 次 + 邏輯獨立）
4. **過長檔案 / 函數**（> 800 行 / > 100 行）

**永遠先報告 + 等使用者確認，絕不自動修改檔案**。

## 三階段工作流

### Phase 1：掃描（scan）

讀取目標資料夾，建立索引：
- 每檔案行數
- 跨檔案 substring matching（≥ 30 字一字不差）
- 高頻 keyword 統計（≥ 4 字、出現 ≥ 5 次）
- 規則 / heading / function 命名 pattern

工具：`Bash` (wc, grep, sort, uniq) + `Glob` + `Grep`

### Phase 2：報告（report）

輸出結構化發現（4 個 dimension）：

```
=== Cleanup Report ===

Dimension 1: 重複內容
- 「<關鍵字>」散落 N 次（檔案 A, B, C）→ 建議抽成 single reference

Dimension 2: 命名不一致
- 「<concept>」有 X 種寫法：<list>
- 建議統一為：<recommended>

Dimension 3: 可模組化候選
- <段落> 重複 N 次（位置）→ 建議抽成新檔案 / shared section

Dimension 4: 過長檔案
- <file> N 行（超出 < 警告線 >）→ 建議拆檔策略
```

### Phase 3：建議（suggest）

不直接改 code，輸出 refactor proposal：
- 抽出哪段成新模組（建議檔名 + 內容大綱）
- 命名統一建議（哪個版本保留）
- 拆檔策略（拆成幾個 file，每個負責什麼）

使用者確認後才用 `Edit` / `Write` 工具改檔案。

## 觸發場景

- 寫到專案 1000+ 行覺得開始亂
- skill prompt 越寫越長覺得有重複
- 想 fork 別人的 skill 但先看 codebase 狀態
- 兩個禮拜沒 review codebase 想盤點
- v0.x → v0.x+1 升級前先 cleanup
- 多人協作前盤點命名 inconsistency

## 不要做

- 自動修改檔案（必須使用者確認）
- 刪除任何內容（只報告，不刪）
- 推到 GitHub（使用者自己 commit）
- 修改測試 / CI 設定 / package.json
- 改 license / readme 等 meta files
- 改 `.git/` 內任何東西

## 4 個 dimension 詳細

### Dimension 1: 重複內容（DRY 違反）

**掃描方式**：

```bash
# 跨檔案 ≥ 30 字一字不差重複
cat <files> | grep -o '.\{30,\}' | sort | uniq -c | sort -rn | awk '$1 >= 3'

# 高頻 keyword 散落
for kw in <key concepts>; do
  count=$(grep -r "$kw" <files> | wc -l)
  echo "「$kw」出現 $count 次"
done
```

**警告線**：
- 同一字串散落 **≥ 5 個檔案** = 嚴重 DRY 違反
- 同 keyword 出現 **≥ 50 次** = 過度重複，應抽變數
- 同一段邏輯重複 **≥ 3 次** = 應抽函數 / module

### Dimension 2: 命名不一致

**掃描方式**：

```bash
# 抓所有 R[N] vs 規則 [N] 形式
grep -rho "R[0-9]\+：" <files> | sort -u
grep -rho "規則 [0-9]\+" <files> | sort -u
```

**常見 inconsistency patterns**：
- 中英混用：「Skill」vs「技能」vs「skill」
- 編號格式：「R5」vs「規則 5」vs「Rule 5」
- 變數命名：camelCase vs snake_case vs kebab-case 混用
- 同義詞：user / account / profile / member
- emoji 編號：⭐⭐⭐ vs (5/5) vs ★★★

**處理建議**：
- 選一個 canonical 形式，其他全部 rename
- 在 SKILL.md 開頭定義 vocabulary 區塊

### Dimension 3: 可抽模組區塊

**掃描方式**：
- 找重複 ≥ 3 次的 logic block
- 邏輯獨立 + 跨檔案 reuse 機會
- 規則描述 pattern（例：所有 R[N] 都有相似結構 → 可抽 schema）

**模組化候選範例**：
- Header block（每個檔案開頭重複）
- Constant 定義（version / 日期 / 數字常數）
- 同樣的「檢查表」格式
- 共用 emoji 鍵 / glyph

### Dimension 4: 過長檔案 / 函數

**警告線**：

| 檔案類型 | 警告 | 嚴重 |
|---|---|---|
| SKILL.md | > 200 行 | > 400 行 |
| references/*.md | > 400 行 | > 800 行 |
| 一般 .py / .ts | > 500 行 | > 1000 行 |
| 單函數 | > 50 行 | > 100 行 |

**處理建議**：
- SKILL.md 過長 → 抽出 references/
- references/*.md 過長 → 拆 sub-references/
- 函數過長 → 拆 helper function

## 範例：跑 social-post skill（2,100 行）真實結果

跑 `Hao0321/claude-skill-social-post` 整個 repo 的 cleanup 報告：

```
=== Cleanup Report — social-post skill v0.7.1 ===

📊 Repo 概況：11 個檔案 / 2,100 行總計

Dimension 1: 重複內容散落（top 10）
- 「鐵粉」 65 次散落 ⚠️ 過度使用
- 「Day 1」 70 次散落 ⚠️ 建議抽成 facts.md 單一引用
- 「敘事意圖」 34 次
- 「mega-viral」 28 次
- 「Day 5」 26 次 / 「Day 6」 22 次 / 「Day 7」 19 次
- 「Mode B」 19 次
- 「4 段 4 句」 14 次
- 「F6b」 56 次（跨 4 個檔案：SKILL.md 13 / case_studies 31 / formulas 11 / evaluation 1）

Dimension 2: 命名不一致 🎯 嚴重
- 「規則」雙命名：
  - SKILL.md 用「R[N]：」（R1-R14，13 種）
  - case_studies.md 用「規則 [N]」（規則 1-10）
  - 兩套同源系統互不引用 → 讀者需要心智 mapping
  - **建議統一為「R[N]」**

Dimension 3: 可抽模組候選
- 「Day 1-7 戰績數字」散落 200+ 次 → 抽 references/battle_records.md
- 「F6b 變體 A/B/C/D」描述跨 3 個檔案重複 → 抽 references/f6b_variants.md
- 「演算法權重表」evaluation.md + SKILL.md 都有 → 統一 evaluation.md

Dimension 4: 過長檔案
- case_studies.md 604 行 ⚠️ 接近警告線（800）
  - 9 個 Case 各 50-80 行 → 建議拆 case_studies/case_N.md
- formulas.md 505 行 ⚠️ 接近警告線
  - F1-F18 共 18 個公式 → 建議拆 formulas/part_1.md / part_2.md / part_3.md
- SKILL.md 377 行 ⚠️ 超出 200 建議線
  - R1-R14 + 警告 + 補丁混雜 → 建議抽 references/rule_history.md

📊 整體健康度：⚠️ 中度（建議 cleanup）
- 重複 score: 6.5/10（多核心概念散落但有必要）
- 命名 score: 4/10（兩套規則命名是主要債）
- 模組化 score: 5/10（有 references 但內部還太擠）
- 長度 score: 5/10（兩檔接近警告線）

🔧 建議優先 cleanup（影響 v0.8 release）：
1. 統一「R[N]」vs「規則 N」命名（30 min）
2. 抽出 references/battle_records.md（45 min）
3. case_studies.md 拆檔（60 min）
```

## 設計理念

跟 anthropic-skills 的 progressive disclosure 對齊：
- Phase 1 掃描快（< 30 sec）
- Phase 2 報告濃縮（< 1 KB 給人讀）
- Phase 3 建議才展開 detail

跟 social-post skill 的「敘事意圖」對齊：
- 不只看 syntax 重複，看 **semantic** 重複（同 concept 不同字也算）
- 例：「viral 主因」「viral 必要條件」「能爆款的關鍵」 = 同 concept 三種寫法

## License

MIT — 保留此標註即可修改 / 使用 / 商用。

## Author

駱君昊 (Hao) · MetaFantasy Co-Founder

Repo: https://github.com/Hao0321/claude-skill-code-cleanup
Companion: https://github.com/Hao0321/claude-skill-social-post
