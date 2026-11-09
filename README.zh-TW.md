# OpenClaw 2026.10.5 子代理交接修補

[English](README.md) · [驗證紀錄](docs/VALIDATION.md) · [精確版本資料](VERSION.json)

公開分享已實作的子代理／父代理接續修補，包含原始碼 patch、回歸測試與重現條件，採 MIT 授權。

**這是社群修補，不是 OpenClaw 官方發行版，也未宣稱已合併到官方。** 同版本不代表每個人一定遇到，仍取決於子代理批次、稽核設定與執行時序。

## 適用版本

- 已核對基準：**OpenClaw 2026.10.5**，官方標籤 `v2026.10.5`。
- 官方基準 commit：`1b58cb9a1b4f4353e1add63c8506db4d890962e8`。
- 本修補包版本：**2026.10.5-fix.1**，不是官方 npm 版本號。
- 原修補原碼 commit：`a02d409a4c751ecd83239ed71aae948d1c868bcd`。
- 原部署 build ID：`2026.10.5-a02d409a4c75-2026-11-03T21-37-55.869Z`。
- 其他版本、後續官方修正與 Linux／Windows 實機：本次未驗證。

原修補 commit 是來源識別，不是本倉庫內的 commit；將 patch 套入精確官方基準後，可重建相同的完整 Git 原碼樹，雜湊列於 `VERSION.json`。

## 修正什麼

1. **子代理做完，父代理卻無法接回結果。**
   啟用 execution-identity 稽核時，完成批次的識別字串可能超過 256 字元，出現 `execution identity admission token violates its bounded contract`。修補對新完成批次使用完整識別的 SHA-256，先保存版本再派送；已嘗試過的舊批次仍保留原識別，避免升級後重做任務。測試中 5 個 UUID 可重現，但不是所有情境都固定第 5 個才出錯。
2. **接續訊息與尚未結束的回覆搶到同一個會話。**
   在原回覆已持有會話、但執行標記尚未落盤的空檔，讓一般 peer follow-up 等待原回覆釋放，保留既有內部接續與 restart 例外。這不代表所有 `SESSION_WORK_START_CHANGED` 都是同一原因。
3. **指令根本沒啟動，卻被誤報成子程序收尾身分遺失。**
   POSIX 的啟動准入拒絕仍照常拒絕，但保留正確的取消／收尾交接，回傳原始錯誤，不再用 `service child cleanup identity lost` 遮住原因。Windows 與已啟動後的身分遺失路徑維持原規則。

另包含更清楚的完整性錯誤分類，以及原修補使用的測試 fixture 校正。**不加 admin、不改 RLS、不關閉完整性檢查，也不提高識別長度上限。**

## 如何使用

1. 依 [英文 README 的指令](README.md#get-and-apply-the-source-patch)，建立全新的官方 `v2026.10.5` 原碼 checkout。
2. 核對基準 commit 與 `SHA256SUMS`，先做 `git apply --check`，成功才套用 patch。
3. 依上游 Node／pnpm 版本要求安裝依賴，執行 [回歸測試](docs/VALIDATION.md)。
4. 隔離環境驗證後，才自行選擇是否按 OpenClaw 官方 operator 流程部署；本倉庫不提供自動覆蓋正式環境的安裝器。

不要直接修改已安裝的 `node_modules` 或執行中的 Gateway。正式切換前需保全相符的程式與狀態並驗證還原；新版寫入狀態後，不能只換回舊套件就當作完成回滾。

## 驗證與限制

- 公開 patch 已核對套用後的完整 Git tree 與原修補一致，也驗證原碼反向套用可還原基準。
- 定向測試、歷史 build 與現場基本驗收分開列示，詳見 [驗證紀錄](docs/VALIDATION.md)。**沒有宣稱完整 CI 全綠。**
- 已嘗試過的舊長批次、沒有版本標記的長 pause，不會被自動改識別或重播。
- 原現場基本驗收確認 native child 完成及回到父代理，不等於所有 `sessions_spawn`、全部平台與所有歷史錯誤都已驗完。
- 沒有包含內部對話、正式資料庫、部署設定、憑證或客戶資料。

## 授權

[MIT License](LICENSE)，保留 OpenClaw 上游著作權。由 [ladylikegroup-oss](https://github.com/ladylikegroup-oss) 分享維護。
