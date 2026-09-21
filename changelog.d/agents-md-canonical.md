### Changed
- Agent 慣例檔改以 `AGENTS.md` 為唯一真檔並移除 `CLAUDE.md`（Claude Code 在專案沒有 `CLAUDE.md` 時會讀 `AGENTS.md`）；`GEMINI.md`、`.github/copilot-instructions.md` 改指向 `AGENTS.md`。
- Policy Check 改用 `SanHsien/paulsha-conventions`（上游 v1.0.15 加 `agent_files.canonical`／`agent_files.required`）；`dependency-freshness` 每月比對上游引擎版本。
