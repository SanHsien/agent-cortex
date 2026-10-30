# 路線圖

> 本 repo 後續工作的單一真相源。做完一項，就在同一個 commit 刪掉那一列；新工作只加在這裡。
> 現況段的數字是 2026-10-29 依 `main` `4b1046de0331` 查的快照，開工前先重查 GitHub。

## 現況
`hamanpaul/paulsha-cortex` 的 Windows-first 維護型分支，負責多 Agent 交付流程（Candidate → Verify → Independent Review → Delivery → CompletionRecord），見 `README.md:216-224`。2026-10-29 查：open issue／PR 0、Dependabot 與 code scanning open 0，`main` 上 Tests、CodeQL、Dependency freshness、Upstream check 最近一次皆 success（2026-10-29 `gh run list -R SanHsien/agent-cortex -b main`）；上游最新 release 仍是 v0.1.14（2026-10-09），已審到 v0.1.14（`tools/upstream_baseline.json` `reviewed_release`）。卡點是 coordinator／monitor／porcelain 整批落後上游兩輪 release，要不要追是產品方向決定（`docs/UPSTREAM.md:40`）。`gh api repos/SanHsien/agent-cortex` 回 `fork=false`，文件已對齊（2026-10-30）。

## 近期：可以直接動手
| 項目 | 為什麼 | 驗收方式 | 建議執行者 | 證據 |
| --- | --- | --- | --- | --- |
| 依政策停用 `agy` executor：fork 生效的 model-identities roster 不再列 `executor: agy`（或以設定關閉該身分），並在 `docs/DECISIONS.md` 寫明理由與觸發條件 | 維護者 2026-10-27 政策：第三方 agent 不得經 `agy --print` 等方式使用 Antigravity 個人訂閱；內建 roster 把 agy 排第一順位做 planning／review | `grep -n "executor: agy" paulsha_cortex/coordinator/data/model-identities.yaml` 無輸出（或設定層證明該身分停用）；`pwsh -NoProfile -File tools\dev_check.ps1` 全綠；planning 身分改由其他 independence_domain 承接的測試存在且通過 | Claude Opus | `paulsha_cortex/coordinator/data/model-identities.yaml:4-11`；`paulsha_cortex/coordinator/launcher.py:790`（`agy --print`） |

## 中期：等條件成立再做
| 項目 | 觸發條件 | 驗收方式 | 建議執行者 | 證據 |
| --- | --- | --- | --- | --- |
| 審查上游下一個 release，逐區判定後推進 baseline | `Upstream check` 轉 failure，或 `gh api repos/hamanpaul/paulsha-cortex/releases/latest` 出現比 `reviewed_release` 新的 tag | `docs/UPSTREAM.md` 新增同日條目（採用／待採用／不適用逐區附理由）；`tools/upstream_baseline.json` 三個水位推進；`python tools/check_upstream_updates.py` exit 0 | Codex | `docs/UPSTREAM.md:1-8`；`tools/upstream_baseline.json` |
| 完整 release sync：coordinator／monitor／porcelain／CLI／control 以 v0.1.14（或更新 tag）為整批單位併入 | 維護者決定要追上游 daemon／多 UID 部署模型 | 在獨立分支完成（依 `docs/UPSTREAM.md:40` 的例外），`tools\dev_check.ps1` 全綠才併入；`docs/UPSTREAM.md` 把對應「待採用」改成採用並附 commit；`launcher.build_claude_argv`／`build_codex_argv` 簽章若變，agent-relay 指向新版 cortex 跑 `python -m unittest discover -s tests` 仍綠 | Claude Opus | `docs/UPSTREAM.md:40`；`docs/DECISIONS.md:126-136` |
| 移植上游 #1319 的「單檔 frontmatter 錯誤隔離」到 monitor refresh | 上游修正落地，或 fork 實際遇到 refresh 因單檔 frontmatter 錯誤卡住 | 新測試：一份格式錯誤的 planning artifact 不讓整次 refresh 失敗、錯誤寫進 diagnostics；dev_check 全綠 | Claude Sonnet | `docs/UPSTREAM.md:41`；`paulsha_cortex/monitor/correlation.py:170-194` |
| 修 `_yaml.py` subset parser 的單引號純量、非 ASCII 數字、行內註解缺陷 | 上游修正落地（照搬），或維護者授權 fork 自行偏離上游，或 fork 實際讀到受影響的 frontmatter | 每個缺陷各一條修前失敗、修後通過的測試（如 `x: 'C:\new\temp'` 保留反斜線、`x: ²` 丟 `YAMLError`）；dev_check 全綠 | Claude Sonnet | `docs/DECISIONS.md:138-145` |

## 刻意不做
| 不做什麼 | 理由 | 什麼情況下重新考慮 |
| --- | --- | --- |
| 對外發佈衍生 wheel／sdist，或新增看似涵蓋上游程式碼的授權檔 | 上游仍無 LICENSE（2026-10-29 `gh api repos/hamanpaul/paulsha-cortex` 回 `license: null`），見 `docs/DECISIONS.md:27-31` | 上游補上授權，或取得原作者明確同意 |
| 移植 `trust_root/`、`qualification/`、上游 release／canary workflow | 屬 Linux 多 UID 系統部署與上游自家發版流程，fork 是 Windows 原生、版本號獨立（`docs/UPSTREAM.md` 2026-10-25 區域判定） | 維護者決定改走多 UID Linux 部署 |
| 恢復經 `agy` CLI 使用 Antigravity 個人訂閱的派工管道 | 維護者 2026-10-27 政策 | 改用 `GEMINI_API_KEY` 或 Vertex 的正式 API 計費路徑，且 launcher 不再經個人訂閱 CLI |
| 在本 repo 設計 relay 專用介面或信件協定 | 協定與派工契約歸 agent-relay（其 `SPEC.md` 第 7 節）；本 repo 只提供 launcher 被 import | agent-relay 的 SPEC 明文要求本 repo 提供新介面 |

## 維護方式
上游以 release tag 為審查單位（不追 `main`，理由見 `docs/UPSTREAM.md:8-11`），`Upstream check` 每週一跑；新 tag 出現才審，審完寫 `docs/UPSTREAM.md` 再推進 baseline。依賴由 `Dependency freshness` 每週檢查，policy 引擎重釘到 `SanHsien/paulsha-conventions` 的新 tag 後跑 dev_check。fork 自有版本號以 `VERSION` 為準，不跟上游 tag。
