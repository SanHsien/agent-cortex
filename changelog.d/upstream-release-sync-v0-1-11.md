### Changed
- 上游 v0.1.9／v0.1.10／v0.1.11（1189 個 commit）以 release 邊界審查並記入 `docs/UPSTREAM.md`：coordinator／monitor／porcelain 契約整批列為待採用（觸發：獨立分支完整 release sync），release／trust_root／qualification 不適用；`tools/upstream_baseline.json` 推進到 v0.1.11（PR 水位 1250、issue 水位 1237）。

### Fixed
- `_yaml.py` subset parser：inline list 的引號值與單引號跳脫、`key:` 後同縮排 dash 的 indentless sequence 不再解析失敗（取自上游 v0.1.11，含回歸測試）。
