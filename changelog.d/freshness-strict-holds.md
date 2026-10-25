### Fixed
- `check_dependency_freshness.py --strict` 不再把已有 `# freshness-hold:` 或未過期延後的過期下限當成失敗，與報告狀態和 `needs_attention` 一致。
