# timesheet

把一週的工作時數拆到「幾個大項目」並分類的 agent skill。
給要填週報工時表、又不想每週從頭回想的人。

流程只有一條：

```
工時總量（卡鐘或預設） → 蒐證據（追蹤系統、行事曆、Jira、git、Slack）
→ 分類（帳本 → 關鍵字 → 問人） → 估時 → 草稿給人改 → 輸出 TSV（可選寫 Sheet）
```

不是全自動。時數是 skill 估、人改；分類第一次問人，之後靠帳本沿用。

## 安裝

Claude Code：

```bash
claude plugin marketplace add jimc1682000/timesheet
claude plugin install timesheet@timesheet
```

或直接把 `skills/timesheet/` 複製到你的 skills 目錄。

## 設定

三個檔，都在 `~/.config/timesheet/`，各人自己管：

| 檔 | 內容 | 誰改 |
|---|---|---|
| `config.yaml` | 工時來源、證據來源開關、輸出方式、Sheet 位置 | 人 |
| `rules.yaml` | 分類名稱、關鍵字、會議預設分類、例外 | skill 從帳本歸納，改動列給人看 |
| `ledger.yaml` | 工單 → 分類，一筆一行，跨週沿用 | skill 在人批准後寫入 |

範例在 `examples/`，格式在 `schema/`。公司或團隊可以另外維護一份 template，
但每個人的 `~/.config/timesheet/` 是自己的。

## 週 TTL 怎麼算

| 來源 | 做法 |
|---|---|
| 卡鐘 | 下班 − 上班 − 午休；`count_as_work` 列的假別（如遠端辦公、忘刷卡）算整天；其他請假不進表 |
| 沒有卡鐘 | `fallback_weekly`（預設 40） |

## 估時方法

1. 行事曆已接受的會議：算實際長度
2. 同一天同一個 repo 的 commit：首末時間當跨度
3. 剩餘時數：歸當天追蹤系統裡進行中的主任務
4. 精度 0.25h，每週各列加總等於 TTL

草稿每列帶「依據」欄：`會`議、`加`班單、`C`ommit、`推`估、`無卡`日。
被問「這個數字怎麼來的」時照欄回答。

## 輸出

- `draft.md`：完整草稿，含對應細項與依據，給人改
- `week.tsv`：Sheet 那四欄（項目、時數、分類 1、分類 2……），貼上就好
- Sheet 寫入：`output.sheet.enabled: true` 才做，找不到指定區塊就報錯、不寫

設計取捨與被否決的方案見 `docs/design.md`。

## License

MIT
