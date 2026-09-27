### Changed
- Tests 的 pytest 改為無條件 gate：拿掉 template 留下的「偵測有無測試才跑」條件（policy R-19 標為可被靜默跳過的高風險樣式）；`tests/` 若消失或收不到測試，job 直接失敗而不是回報 success。Persona Scope 的安裝步驟移除用不到的 `pytest`，R-19 不再把它誤認為測試 gate（上游下一版會轉 FAIL）。
- 文件引用清掉 R-22 的 55 筆懸空引用：repo 內有的檔案改寫成完整路徑；執行時才產生或在 repo 外的檔名列進新的 `.doc-drift-allow`；ADR 屬當時的決策紀錄（大量「檔名:行號」），整份豁免。
