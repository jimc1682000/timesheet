# timesheet

## 用途

把一週工時拆成幾個大項目並分類的 agent skill，以 Claude Code plugin 發布。skill 本體在 `skills/timesheet/`，設定範例在 `examples/`，格式在 `schema/`，設計取捨在 `docs/design.md`。

## 驗證

目前沒有自動檢查。改動 `schema/` 或 `examples/` 時，確認每個範例仍符合同名 schema；改動流程、估時或輸出時，同步更新 README 對應段落。

## 邊界

- 這是 public repo，也是 plugin 來源（`.claude-plugin/marketplace.json` 的 `source` 是 `./`），根目錄檔案會隨 plugin 安裝；只放範例，不放個人設定或真實工時資料。
- 使用者設定放在各人的 `~/.config/timesheet/`，repo 只提供範例與 schema。
