---
name: timesheet
description: 把上一週（或指定期間）的工時拆成 4–6 個大項目並分類，產出可貼進工時表的 TSV 與可審的草稿。當使用者說「填工時表」「週報工時」「上週時數怎麼分」「timesheet」時使用。需要 ~/.config/timesheet/config.yaml；沒有就先引導建立。
---

# timesheet

週工時拆分。人負責拍板，skill 負責蒐證據、估時、記住上次的決定。

## 0. 讀設定

依序讀 `~/.config/timesheet/config.yaml`、`rules.yaml`、`ledger.yaml`。
缺 `config.yaml` → 複製 `examples/config.example.yaml` 過去，只問三件事：
工時來源、要開哪些證據來源、輸出要不要寫 Sheet。其他用預設。

期間：使用者沒說就是上一個完整工作週（週一到週五）。
週欄名照工時表慣例：跨月時月底那幾天自成一欄。

## 1. 工時總量（週 TTL）

照 `hours.source` 順序找：

- `records-json`：卡鐘紀錄 json，欄位名在 config 對應
- `fhr-txt`：tab 分隔的出勤原檔（9 欄）
- 都沒有 → `hours.fallback_weekly`，草稿標「未驗證」

有卡鐘時每日工時 = 下班 − 上班 − `lunch_hours`。
`hours.count_as_work` 列的假別當整天（預設 `remote_work`、`forgot_punch`）；
其他請假不進表。使用者說「這天算」就照他的。

## 2. 蒐證據

只開 `evidence.*.enabled: true` 的來源，每個來源只抓期間內的：

| 來源 | 抓什麼 | 用途 |
|---|---|---|
| tracker | 指派給使用者、期間內有活動的任務（id、標題、日期範圍、狀態） | 任務清單、剩餘時數歸屬 |
| calendar | 已接受的事件（拒絕與未回覆不算） | 會議時數、活動值守 |
| jira | 期間內更新且與使用者有關的票 | 對應工單 |
| git | `roots` 下每個 repo `git log --all --author=<me> --since --until` | commit 跨度 |
| slack | 使用者自己的訊息時間戳 | 無卡日、無 commit 日的佐證 |

git 只收 `git.roots` 底下的 repo；不在清單裡的不看。
Slack 只引用使用者本人的訊息，不抓別人的。

## 3. 分類

每個任務依序過三關，先命中先贏：

1. **帳本** `ledger.yaml`：工單 id 有記錄 → 沿用
2. **規則** `rules.yaml`：關鍵字命中 → 該分類；會議、課程、workshop 用 `meetings.default`，`meetings.exceptions` 另計
3. **問人**：都沒命中 → 草稿標 `?`，一次列出全部問

使用者拍板後，把新決定寫進 `ledger.yaml`（工單 id、分類、日期、一句理由）。
若同一個關鍵字連續兩週都被人改成同一個分類，提議加進 `rules.yaml`，列出改動，人點頭才寫。

## 4. 估時

三層，加總必須等於當週 TTL：

1. 會議：實際長度
2. commit：同日同 repo 首末時間當跨度，重疊時壓縮到不超過當日總量
3. 剩餘：歸當日 tracker 進行中的主任務

精度 0.25h。無卡日按當日證據比例分 8h。

每列標依據代碩：`會`、`加`（已送加班單）、`C`（commit）、`推`（只有任務日期範圍）、`無卡`。
一列可複合，寫最主要的；只有 `推`／`無卡` 的列算弱依據，草稿最上面統計強／混合／弱各多少小時。

## 5. 合併成 4–6 列

同一個工單、同一條產品線、同一類會議合併成一列。
語意不同才拆到第 7 列。合併後每列的「對應細項」欄列出原始工單編號。

## 6. 標題

草稿有兩個標題欄：

- **報表標題**：給主管、PM、會計看。消融式縮短：拿掉不改變意思的詞（系統、機制、建置、後端、主站），
  留「誰、做什麼、為了哪個活動」。不用工程術語，不用內部代號。
- **對應細項**：原本的技術描述與工單編號，留在草稿，不進 TSV。

## 7. 草稿 → 批准 → 輸出

草稿寫到 `output.dir/<週一日期>/draft.md`，格式：

```
| 報表標題 | 對應細項 | Hours | <分類 1> | <分類 2> | 依據 |
...
| TTL: |  | <總量> |  |  |  |
```

給使用者看，等他改。他說 approve 才：

1. 寫 `week.tsv`（只有報表標題、Hours、各分類欄）
2. 更新 `ledger.yaml`
3. `output.sheet.enabled` 為 true 才寫 Sheet：找 `output.sheet.block` 的區塊與該週欄；
   找不到區塊、找不到週欄、欄位已有值 → 報錯，不寫

## 不做的事

- 不自己決定 `?` 的分類
- 不把 40h 當事實；沒卡鐘就標未驗證
- 不抓別人的 Slack 訊息
- 不在草稿被批准前寫任何外部系統
