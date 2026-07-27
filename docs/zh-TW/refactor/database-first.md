---
read_when:
    - 將 OpenClaw 執行階段資料、快取、逐字稿、任務狀態或暫存檔案移至 SQLite
    - 設計從舊版 JSON 或 JSONL 檔案遷移的 doctor 機制
    - 變更備份、還原、VFS 或工作節點儲存行為
    - 移除工作階段鎖定、修剪、截斷或 JSON 相容性路徑
summary: 讓 SQLite 成為主要持久狀態與快取層，同時保留以檔案為基礎的設定之遷移計畫
title: 資料庫優先的狀態重構
x-i18n:
    generated_at: "2026-07-26T07:55:34Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: ae4d72f04c1228742cc7ea3cc87a96b06aa1e2b750ace23cca5b387844746186
    source_path: refactor/database-first.md
    workflow: 16
---

# 資料庫優先的狀態重構

## 決策

使用兩層 SQLite 配置：

- 全域資料庫：`~/.openclaw/state/openclaw.sqlite`
- 代理程式資料庫：每個代理程式各有一個 SQLite 資料庫，用於代理程式所擁有的工作區、
  逐字記錄、VFS、成品，以及大型的每代理程式執行階段狀態
- 設定仍以檔案為後端：`openclaw.json` 保留在
  資料庫之外。執行階段驗證設定檔移至 SQLite；外部提供者或命令列介面的
  認證資訊檔案仍由擁有者管理，並位於 OpenClaw 資料庫之外。

全域資料庫是控制平面資料庫。它負責代理程式探索、
共用閘道狀態、配對、裝置／節點狀態、任務與流程帳本、外掛
狀態、排程器執行階段狀態、備份中繼資料，以及遷移狀態。

代理程式資料庫是資料平面資料庫。它負責代理程式的工作階段
中繼資料、逐字記錄事件串流、VFS 工作區或暫存命名空間、工具
成品、執行成品，以及可搜尋／可建立索引的代理程式本機快取資料。

這能提供單一持久的全域檢視，同時不必將大型代理程式工作區、
逐字記錄與二進位暫存資料強制納入共用閘道寫入通道。

## 強制契約

此遷移只有一種標準執行階段形態：

- 工作階段資料列僅保存工作階段中繼資料。不得保存
  `transcriptLocator`、逐字記錄檔案路徑、同層 JSONL 路徑、鎖定路徑、
  修剪中繼資料，或檔案時代的相容性指標。
- 逐字記錄識別一律使用 SQLite 識別：`{agentId, sessionId}`，以及
  通訊協定需要時的選用主題中繼資料。
- `sqlite-transcript://...` 不是執行階段或通訊協定識別。新程式碼不得
  衍生、保存、傳遞、剖析或遷移逐字記錄定位器。執行階段與
  測試完全不應包含偽定位器；文件只能為了禁止該字串而
  提及它。
- 舊版 `sessions.json`、逐字記錄 JSONL、`.jsonl.lock`、修剪、截斷，
  以及舊工作階段路徑邏輯，只能存在於 doctor 遷移／匯入路徑中。
- 舊版工作階段設定別名只能存在於 doctor 遷移中。執行階段
  不得解讀 `session.idleMinutes`、`session.resetByType.dm`，或
  另一個已設定代理程式的跨代理程式 `agent:main:*` 主工作階段別名。
- 工作階段路由識別是具型別的關聯狀態。高頻執行階段與 UI 路徑
  應讀取 `sessions.session_scope`、`sessions.account_id`、
  `sessions.primary_conversation_id`、`conversations` 和
  `session_conversations`；不得剖析 `session_key`，也不得從
  `session_entries.entry_json` 挖掘提供者識別，但在刪除舊呼叫端期間，
  可將其作為相容性影子。
- 通道層級的直接訊息標記，例如 `dm` 與 `direct`，屬於路由
  詞彙，而非逐字記錄定位器或檔案儲存區相容性控制代碼。
- 舊版鉤子處理常式設定只能存在於 doctor 警告／遷移介面。
  執行階段不得載入 `hooks.internal.handlers`；鉤子只能透過探索到的
  鉤子目錄與 `HOOK.md` 中繼資料執行。
- 執行階段啟動、高頻回覆路徑、壓縮、重設、復原、診斷、
  TTS、記憶鉤子、子代理程式、外掛命令路由、通訊協定邊界，以及
  鉤子，都必須在執行階段中傳遞 `{agentId, sessionId}`。
- 測試應透過
  `{agentId, sessionId}` 植入並斷言 SQLite 逐字記錄資料列。若測試只證明 JSONL 路徑轉送、
  保留呼叫端提供的定位器，或逐字記錄檔案相容性，則應予以刪除；
  除非其涵蓋 doctor 匯入、非工作階段的支援／除錯
  具體化，或通訊協定形態。
- `runEmbeddedPiAgent(...)`、已備妥的工作程式執行，以及內層嵌入式
  嘗試，都不得接受逐字記錄定位器。它們以 `{agentId, sessionId}` 開啟 SQLite 逐字記錄
  管理器，並將該管理器傳遞給內部化的
  PI 相容代理程式工作階段，讓過時的呼叫端無法使執行器寫入
  JSON／JSONL 逐字記錄。
- 執行器診斷必須將執行階段／快取／承載資料追蹤記錄儲存在 SQLite。
  執行階段診斷不得公開 JSONL 檔案覆寫控制項或通用的
  逐字記錄 JSONL 匯出輔助函式；面向使用者的匯出功能可以從資料庫資料列具體化明確的
  成品，而不將檔案名稱回饋至執行階段。
- 原始串流記錄使用 `OPENCLAW_RAW_STREAM=1` 加上 SQLite 診斷資料列。
  舊版 pi-mono `PI_RAW_STREAM`、`PI_RAW_STREAM_PATH` 和
  `raw-openai-completions.jsonl` 檔案記錄器契約不屬於 OpenClaw
  執行階段或測試。
- QMD 記憶索引不得將 SQLite 逐字記錄匯出為 Markdown 檔案。
  QMD 僅為已設定的記憶檔案建立索引；工作階段逐字記錄搜尋仍以
  SQLite 為後端。
- 對新程式碼而言，QMD SDK 子路徑僅供 QMD 使用。SQLite 工作階段逐字記錄
  索引輔助函式位於 `memory-core-host-engine-session-transcripts`；任何
  QMD 重新匯出僅供相容性使用，執行階段程式碼不得使用。
- 內建記憶索引位於所屬的代理程式資料庫中。執行階段設定與
  已解析的執行階段契約不得公開 `memorySearch.store.path`；doctor
  會刪除該舊版設定鍵，而目前的程式碼會在內部傳遞代理程式的
  `databasePath`。

實作工作應持續刪除程式碼，直到這些敘述在
doctor／匯入／匯出／除錯邊界之外皆無例外地成立。

## 目標狀態與進度

### 強制目標

- 一個全域 SQLite 資料庫負責控制平面狀態：
  `state/openclaw.sqlite`。
- 每個代理程式各有一個 SQLite 資料庫，負責資料平面狀態：
  `agents/<agentId>/agent/openclaw-agent.sqlite`。
- 設定仍以檔案為後端。`openclaw.json` 不屬於此資料庫
  重構的範圍。
- 舊版檔案僅作為 doctor 遷移輸入。
- 執行階段絕不將工作階段或逐字記錄 JSONL 作為作用中狀態讀寫。

### 目標狀態

- `not-started`：檔案時代的執行階段程式碼仍在寫入作用中狀態。
- `migrating`：doctor／匯入程式碼可將檔案資料移入 SQLite。
- `dual-read`：暫時性橋接會同時讀取 SQLite 與舊版檔案。此重構
  禁止這種狀態，除非明確記載為
  僅供 doctor 使用。
- `sqlite-runtime`：執行階段僅讀寫 SQLite。
- `clean`：移除舊版執行階段 API 與測試，且防護機制可防止
  回歸。
- `done`：文件、測試、備份、doctor 遷移與變更檢查證明
  乾淨狀態。

### 目前狀態

- 工作階段：執行階段為 `clean`。工作階段資料列位於每代理程式資料庫中，
  執行階段 API 使用 `{agentId, sessionId}` 或 `{agentId, sessionKey}`，而
  `sessions.json` 僅作為 doctor 舊版輸入。
- 逐字記錄：執行階段為 `clean`。逐字記錄事件、識別、快照，
  以及軌跡執行階段事件，都位於每代理程式資料庫中。執行階段不再
  接受逐字記錄定位器或 JSONL 逐字記錄路徑。
- PI 嵌入式執行器：`clean`。嵌入式 PI 執行、已備妥的工作程式、壓縮，
  以及重試迴圈，都使用 SQLite 工作階段範圍，並拒絕過時的逐字記錄控制代碼。
- 排程：執行階段為 `clean`。執行階段使用 `cron_jobs` 與排程所擁有的 `task_runs`；
  執行階段測試使用 SQLite `storeKey` 命名，而檔案時代的排程路徑僅保留於
  doctor 舊版遷移測試中。
- 任務登錄：`clean`。任務與 Task Flow 執行階段資料列位於
  `state/openclaw.sqlite`；未發布的側車 SQLite 匯入器已刪除。
- 外掛狀態：`clean`。外掛狀態／Blob 資料列位於共用全域
  資料庫中；防護機制會阻止使用舊版外掛狀態側車 SQLite 輔助函式。
- 記憶：內建記憶與工作階段逐字記錄索引為 `sqlite-runtime`。
  記憶索引資料表位於每代理程式資料庫中，外掛記憶狀態使用
  共用的外掛狀態資料列，而舊版記憶檔案僅作為 doctor 遷移輸入
  或使用者工作區內容。
- 備份：`sqlite-runtime`。備份程序會壓縮 SQLite 快照、省略即時
  WAL／SHM 側車檔案、驗證 SQLite 完整性，並在
  全域資料庫中記錄備份執行。
- 工作區設定：`sqlite-runtime`。設定完成狀態、工作區證明，
  以及產生的啟動雜湊，皆位於具型別的共用 SQLite 資料表中。執行階段
  不會讀寫已停用的工作區 JSON 與 `.attested` 側車檔案；
  Doctor 負責經驗證的匯入與經確認的移除。
- Doctor 遷移：刻意為 `migrating`。Doctor 會將舊版 JSON、
  JSONL 與已停用的側車儲存區匯入 SQLite、記錄遷移執行／來源，
  並移除已成功處理的來源。
- 執行核准：`file-runtime`。TypeScript 與 macOS 仍會讀寫
  作用中狀態目錄的 `exec-approvals.json`；保留的
  `exec_approvals_config` 結構描述目前尚無執行階段擁有者。未來切換時必須
  新增同狀態 doctor 匯入，並同步遷移兩個執行階段。
- 端對端指令碼：執行階段涵蓋為 `clean`。Docker MCP 植入會寫入 SQLite
  資料列。執行階段情境 Docker 指令碼只會在
  doctor 遷移植入中建立舊版 JSONL，並明確命名舊版工作階段索引路徑。

### 剩餘工作

- [x] 重新命名排程執行階段測試的儲存區變數，使其不再使用 `storePath`，除非
      它們是 doctor 舊版輸入。
      檔案：`src/cron/service.test-harness.ts`、
      `src/cron/service.runs-one-shot-main-job-disables-it.test.ts`、
      `src/cron/service/timer.regression.test.ts`、
      `src/cron/service/ops.test.ts`、`src/cron/service/store.test.ts`、
      `src/cron/service.heartbeat-ok-summary-suppressed.test.ts`、
      `src/cron/service.main-job-passes-heartbeat-target-last.test.ts`、
      `src/cron/store.test.ts`。
      證明：`pnpm check:database-first-legacy-stores`；`rg -n 'storePath' src/cron --glob '!**/commands/doctor/**'`。
- [x] 移除或重新命名過時的檔案時代匯出測試模擬物件。
      檔案：`src/auto-reply/reply/commands-export-test-mocks.ts`。
      證明：`rg -n 'resolveSessionFilePath|sessionFile|storePath|transcriptLocator' src/auto-reply/reply`。
- [x] 使 Docker 執行階段情境的舊版 JSONL 植入明顯僅供 doctor 使用。
      檔案：`scripts/e2e/session-runtime-context-docker-client.ts`。
      證明：`rg -n 'sessions\\.json|sessionFile|\\.jsonl' scripts/e2e/session-runtime-context-docker-client.ts` 顯示僅有
      `seedBrokenLegacySessionForDoctorMigration`。
- [x] 在任何結構描述變更後，維持 Kysely 產生的型別一致。
      檔案：`src/state/openclaw-state-schema.sql`、
      `src/state/openclaw-agent-schema.sql`、
      `src/state/*generated*`。
      證明：本次沒有結構描述變更；`pnpm db:kysely:check`；
      `pnpm lint:kysely`。
- [x] 重新執行針對受影響儲存區、命令與指令碼的聚焦測試。
      證明：`pnpm test src/cron/service/store.test.ts src/cron/store.test.ts src/cron/service.heartbeat-ok-summary-suppressed.test.ts src/cron/service.main-job-passes-heartbeat-target-last.test.ts src/cron/service.every-jobs-fire.test.ts src/cron/service.persists-delivered-status.test.ts src/cron/service.runs-one-shot-main-job-disables-it.test.ts src/cron/service/ops.test.ts src/cron/service/timer.regression.test.ts src/auto-reply/reply/commands-export-session.test.ts extensions/telegram/src/thread-bindings.test.ts extensions/slack/src/monitor/message-handler/prepare.test.ts src/acp/translator.session-lineage-meta.test.ts`；`git diff --check`。
- [x] 宣告 `done` 前，執行變更閘門或遠端廣泛驗證。
      證明：`pnpm check:changed --timed -- <changed extension paths>` 已在
      Hetzner Crabbox 執行 `run_3f1cabf6b25c` 中通過；該次執行先暫時設定 Node 24／pnpm，
      並針對同步後、不含 `.git` 的工作區明確設定路徑路由。

### 不得回歸

- 不得使用逐字記錄定位器。
- 不得使用作用中的工作階段檔案。
- 不得使用偽造的 JSONL 測試固定資料，doctor 舊版遷移測試除外。
- 預期使用 Kysely 之處不得直接存取 SQLite。
- 不得新增檔案時代的資料庫遷移。全域結構描述維持版本 `1`。
  已發布的每代理程式版本 `1` 結構描述有一項範圍明確的執行階段遷移，可升級至
  版本 `2`，以提供穩定的記憶來源識別。

## 程式碼閱讀假設

沒有後續產品決策會阻礙此計畫。實作應依照
以下假設進行：

- 直接使用 `node:sqlite`，並要求此儲存路徑使用可安全重設 WAL 的 Node 執行階段
  （22.22.3+、24.15+ 或 25.9+）。
- 僅保留一個一般設定檔。此重構不得將設定、外掛
  資訊清單或 Git 工作區移入 SQLite。
- 不需要執行階段相容性檔案。舊版 JSON 與 JSONL 檔案僅作為
  遷移輸入。分支本機的 SQLite 輔助檔案從未正式發布，因此直接刪除，
  而不匯入。
- `openclaw doctor --fix` 負責將舊版檔案遷移至資料庫。執行階段
  啟動僅負責已發布 SQLite 結構描述版本之間的有限升級；
  不得匯入檔案時代的狀態。
- 認證資訊相容性遵循相同規則：執行階段認證資訊存放於
  SQLite。舊版 `auth-profiles.json`、各代理程式的 `auth.json`，以及共用的
  `credentials/oauth.json` 檔案都是 doctor 的遷移輸入，匯入後即予以
  移除。
- 產生的模型目錄狀態由資料庫支援。執行階段程式碼不得寫入
  `agents/<agentId>/agent/models.json`；現有的 `models.json` 檔案是舊版
  doctor 輸入，匯入 `agent_model_catalogs` 後即予以移除。
- 執行階段不得遷移、正規化或橋接逐字稿定位器。目前使用中的
  逐字稿識別資訊是 SQLite 中的 `{agentId, sessionId}`。檔案路徑僅供
  舊版 doctor 輸入使用，而 `sqlite-transcript://...` 必須從
  執行階段、通訊協定、掛鉤及外掛介面中消失，不得將其視為
  邊界控制代碼。
- 執行階段讀取 SQLite 逐字稿時，不會執行舊版 JSONL 項目格式遷移，
  也不會為了相容性重寫整份逐字稿。舊版項目正規化僅保留於
  明確的 doctor／匯入公用程式中。doctor 會先正規化舊版 JSONL 逐字稿
  檔案，再插入 SQLite 資料列；目前的執行階段資料列
  已使用目前的逐字稿結構描述寫入。軌跡／工作階段匯出
  會依原樣讀取這些資料列，且不得在匯出時執行舊版遷移。
- 舊版逐字稿 JSONL 剖析／遷移輔助程式僅供 doctor 使用。執行階段
  逐字稿格式程式碼僅建構目前的 SQLite 逐字稿內容；doctor
  負責在插入資料列前升級舊版 JSONL 項目。
- 舊有由執行階段管理的 JSONL 逐字稿串流輔助程式已刪除。doctor
  匯入程式碼負責明確讀取舊版檔案；執行階段工作階段歷程則讀取
  SQLite 資料列。
- Codex 應用程式伺服器繫結使用 OpenClaw `sessionId`，作為 Codex
  外掛狀態命名空間中的標準索引鍵。`sessionKey` 是用於
  路由／顯示的中繼資料，不得取代持久工作階段 ID，也不得恢復
  以逐字稿檔案作為識別資訊。
- 內容引擎會直接接收目前的執行階段合約。登錄機制
  不得以重試相容層包裝引擎並刪除 `sessionKey`、
  `transcriptScope` 或 `prompt`；無法接受目前
  資料庫優先參數的引擎應明確失敗，而非透過橋接運作。
- 備份輸出應維持為單一封存檔。資料庫內容應以
  精簡的 SQLite 快照形式納入該封存檔，而非原始的即時 WAL 輔助檔案。
- 逐字稿搜尋很實用，但首個資料庫優先版本不強制提供。
  結構描述的設計應允許日後加入 FTS。
- 在資料庫邊界穩定之前，工作者執行應透過設定維持在實驗性功能階段。

## 程式碼閱讀結果

目前的分支已超越概念驗證階段。共用
資料庫已存在，Node `node:sqlite` 已透過小型執行階段輔助程式完成接線，而
先前的儲存區現在會寫入 `state/openclaw.sqlite` 或其所屬的
`openclaw-agent.sqlite` 資料庫。

剩餘工作不是選擇 SQLite，而是維持新邊界的整潔，
並刪除任何仍看似舊有檔案世界的相容性介面：

- 工作階段 `storePath` 不再是執行階段識別資訊、測試固定資料格式或
  狀態承載內容欄位。執行階段與橋接測試不再包含
  `storePath` 合約名稱；doctor／遷移程式碼負責該舊版詞彙。
- 工作階段寫入不再經過舊有的程序內 `store-writer.ts`
  佇列。SQLite 修補寫入會在交易外完成準備，接著使用短暫的
  同步驗證／套用交易，並明確偵測衝突。
- 舊版路徑探索仍有有效的遷移用途，但執行階段程式碼應
  停止將 `sessions.json` 與逐字稿 JSONL 檔案視為可能的寫入
  目標。
- 代理程式所屬的資料表存放於各代理程式的 SQLite 資料庫中。全域資料庫保留
  登錄／控制平面資料列；逐字稿識別資訊是各代理程式逐字稿資料列中的
  `{agentId, sessionId}`。執行階段程式碼不得持久保存逐字稿檔案
  路徑，也不得遷移逐字稿定位器。
- doctor 已匯入數個舊版檔案。清理工作的目標是將其整理為
  單一且明確的遷移實作，由 doctor 呼叫，並產生持久的
  遷移報告。

沒有其他產品問題會阻礙實作。

## 目前的程式碼結構

此分支已有實際的共用 SQLite 基礎：

- 執行環境最低版本現在要求具備 WAL 重設安全性的 Node 組建版本：22.22.3+、
  24.15+ 或 25.9+。`package.json`、命令列介面執行環境防護、安裝程式預設值、
  macOS 執行環境定位器、CI 與公開安裝文件均已一致。
- `src/state/openclaw-state-db.ts` 會開啟 `openclaw.sqlite`、設定 WAL、
  `synchronous=NORMAL`、`busy_timeout=30000`、`foreign_keys=ON`，並套用
  衍生自 `src/state/openclaw-state-schema.sql`
  的已產生結構描述模組。
- Kysely 資料表型別與執行環境結構描述模組，是從使用已提交的 `.sql` 檔案建立的
  可拋棄式 SQLite 資料庫產生；執行環境程式碼不再為全域、每代理程式或代理
  擷取資料庫保留複製貼上的結構描述字串。
- 執行環境儲存區會從這些已產生的 Kysely `DB` 介面衍生選取與插入的資料列型別，
  而不再手動建立 SQLite 資料列形狀的影子定義。原始 SQL 仍僅限於結構描述套用、
  pragma 與僅供遷移使用的 DDL。
- 全域 SQLite 結構描述維持在 `user_version = 1`。每代理程式結構描述
  版本為 `2`；其開啟器會以不可分割方式，將已發布版本 `1`
  的記憶來源鍵遷移為穩定的整數識別碼。從檔案匯入資料庫的作業
  仍保留在 doctor 程式碼中。
- 關聯式所有權會在所有權邊界具有規範性的地方強制執行：
  來源遷移資料列會從 `migration_runs` 級聯、任務遞送狀態
  會從 `task_runs` 級聯，而逐字稿識別資料列會從
  逐字稿事件級聯。
- 目前的共用資料表包括 `agent_databases`、
  `auth_profile_stores`、`auth_profile_state`、
  `plugin_state_entries`、`plugin_blob_entries`、`media_blobs`、
  `skill_uploads`、`capture_sessions`、`capture_events`、`capture_blobs`、
  `sandbox_registry_entries`、`cron_jobs`、`commitments`、
  `delivery_queue_entries`、`model_capability_cache`、
  `workspace_setup_state`、`workspace_path_aliases`、`workspace_attestations`、
  `workspace_generated_bootstrap_hashes`、`native_hook_relay_bridges`、
  `current_conversation_bindings`、`plugin_binding_approvals`、
  `tui_last_sessions`、`acp_sessions`、`acp_replay_sessions`、
  `acp_replay_events`、`task_runs`、`task_delivery_state`、`flow_runs`、
  `subagent_runs`、`migration_runs` 及 `backup_runs`。
- 任意由外掛擁有的狀態不會取得由主機擁有的具型別資料表。已安裝的
  外掛會使用 `plugin_state_entries` 儲存具版本的 JSON 承載資料，並使用
  `plugin_blob_entries` 儲存位元組，同時具備命名空間／鍵所有權、TTL 清理、
  備份及外掛遷移記錄。當主機擁有查詢合約時，由主機擁有的外掛協調狀態
  仍可使用具型別資料表，例如
  `plugin_binding_approvals`。
- 外掛遷移是針對外掛所擁有命名空間的資料遷移，而非主機
  結構描述遷移。外掛可透過遷移提供者遷移自己的具版本狀態／Blob 項目，
  而主機會在一般遷移帳本中記錄來源／執行狀態。新安裝外掛
  不需要變更 `openclaw-state-schema.sql`，除非主機本身要接管
  新的跨外掛合約。
- `src/state/openclaw-agent-db.ts` 會開啟
  `agents/<agentId>/agent/openclaw-agent.sqlite`、在全域資料庫中註冊該資料庫，並擁有代理程式本機的工作階段、
  逐字稿、VFS、成品、快取及記憶索引資料表。共用執行環境探索現在會讀取具產生型別的
  `agent_databases` 登錄，而不再於每個呼叫位置重新實作該查詢。
- 全域及每代理程式資料庫會記錄一筆 `schema_meta` 資料列，其中包含資料庫角色、
  結構描述版本、時間戳記，以及代理程式資料庫的代理程式 ID。全域資料庫
  維持在 `user_version = 1`；每代理程式資料庫在有限範圍的
  記憶來源識別碼遷移後使用版本 `2`。
- 每代理程式工作階段識別現在具有規範性的 `sessions` 根資料表，以
  `session_id` 為鍵，並將 `session_key`、`session_scope`、`account_id`、
  `primary_conversation_id`、時間戳記、顯示欄位、模型中繼資料、
  測試框架 ID，以及父項／衍生連結設為可查詢欄位。`session_routes`
  是從 `session_key` 到目前
  `session_id` 的唯一作用中路由索引，因此路由鍵可移至新的持久工作階段，而不會
  讓熱路徑讀取必須在重複的 `sessions.session_key` 資料列之間選擇。舊的
  `session_entries.entry_json` 相容形狀承載資料會透過外部索引鍵附加於
  持久的 `session_id` 根；它不再是工作階段唯一的
  結構描述層級表示法。
- 每代理程式的外部對話識別也採用關聯式結構：
  `conversations` 儲存正規化的提供者／帳號／對話識別，而
  `session_conversations` 將一個 OpenClaw 工作階段連結至一或多個外部
  對話。這涵蓋共用主要 DM 工作階段，其中多個對等端可刻意對應至同一工作階段，
  而不必在 `session_key` 中提供不實資訊。SQLite 也會
  強制確保自然提供者識別的唯一性，因此相同的
  頻道／帳號／種類／對等端／討論串元組不會分岔至不同的對話 ID。
  共用主要直接對等端會以 `participant` 角色連結，因此一個
  OpenClaw 工作階段可代表多個外部 DM 對等端，而不必將
  較舊的對等端降級為含糊的相關資料列。`sessions.primary_conversation_id` 仍會
  指向目前的具型別遞送目標。已關閉的路由／狀態欄位
  會透過 SQLite `CHECK` 條件約束強制執行，而不僅依賴
  TypeScript 聯集。
  執行環境工作階段投影會先清除 `session_entries.entry_json` 中的相容性路由影子，
  再套用具型別的工作階段／對話
  欄位，因此過期的 JSON 承載資料無法復原遞送目標。
  子代理程式公告路由同樣要求具型別的 SQLite 遞送內容；
  它不再退回使用相容性 `SessionEntry` 路由欄位。
  閘道 `chat.send` 的明確遞送繼承會讀取具型別的 SQLite
  遞送內容，而非 `origin`/`last*` 相容性欄位。
  `tools.effective` 同樣會從具型別的
  SQLite 遞送／路由資料列衍生提供者／帳號／討論串內容，而非過期的 `last*` 工作階段項目影子。
  系統事件提示內容會從具型別遞送欄位重建
  頻道／收件者／帳號／討論串欄位，而非使用 `origin` 影子。
  共用的 `deliveryContextFromSession` 輔助程式及工作階段至對話
  對應器現在會完全忽略 `SessionEntry.origin`；只有具型別遞送欄位
  與關聯式對話資料列可建立熱路徑路由識別。
  執行環境工作階段項目正規化會先移除 `origin`，再持久化或
  投影 `entry_json`；傳入中繼資料則會寫入具型別的頻道／聊天
  欄位及關聯式對話資料列，而非建立新的來源影子。
- 逐字稿事件、逐字稿快照及軌跡執行環境事件現在
  會參照規範性的每代理程式 `sessions` 根，並在刪除工作階段時級聯。
  逐字稿識別／等冪性資料列會繼續從精確的
  逐字稿事件資料列級聯。
- 記憶核心索引現在使用明確的代理程式資料庫資料表
  `memory_index_meta`、`memory_index_sources`、`memory_index_chunks` 及
  `memory_embedding_cache`，並由 `memory_index_state` 追蹤修訂變更。
  選用的 FTS／向量側索引會命名為 `memory_index_chunks_fts` 及
  `memory_index_chunks_vec`，而非通用的 `meta`、`files`、`chunks`、
  `chunks_fts` 或 `chunks_vec` 資料表。規範名稱會保留目前的
  路徑／來源資料列形狀及序列化嵌入相容性。這些資料表
  是衍生／搜尋快取，而非規範性的逐字稿儲存區；它們可以
  刪除，並從記憶工作區檔案及已設定來源重新建立。
  開啟使用已發布通用名稱的記憶索引時，會將其中繼資料、來源、
  區塊及嵌入快取遷移至規範資料表；衍生的 FTS／向量
  資料表則會使用其規範名稱重新建立。
- 子代理程式執行復原狀態現在位於具型別的共用 `subagent_runs` 資料列中，
  並為子項、要求者及控制器工作階段鍵建立索引。舊的
  `subagents/runs.json` 檔案僅作為 Doctor 清理輸入。其執行項目屬於
  暫時性復原狀態，因此 Doctor 會記錄淘汰收據，
  並捨棄檔案而不匯入。由於在 SQLite 資料列已遭修剪後，
  檔案無法證明其中項目是作用中還是過期，
  操作者在跨越此邊界升級前，必須先讓檔案時代的作用中執行完成。
- 目前的對話繫結現在位於具型別的共用
  `current_conversation_bindings` 資料列中，以正規化對話 ID 為鍵，並將
  目標代理程式／工作階段欄位、對話種類、狀態、到期時間及中繼資料
  儲存為關聯式欄位，而非重複的不透明繫結記錄。
  持久繫結鍵包含正規化對話種類，因此
  直接／群組／頻道參照不會發生衝突，SQLite 也會拒絕無效的繫結
  種類／狀態值。舊的
  `bindings/current-conversations.json` 檔案僅作為 doctor 遷移輸入。
- 遞送佇列復原現在會將頻道、目標、
  帳號、工作階段、重試、錯誤、平台傳送及復原狀態的具型別佇列欄位疊加至
  重播 JSON。`entry_json` 會保留重播承載資料、掛鉤及格式化
  承載資料，但具型別欄位才是熱路徑佇列路由／狀態的權威來源。
- 終端介面上次工作階段還原指標現在位於具型別的共用
  `tui_last_sessions` 資料列中，以雜湊後的終端介面連線／工作階段範圍為鍵。
  執行環境僅讀寫 SQLite、以不可分割方式更新插入每個範圍，並
  排除心跳偵測工作階段。`openclaw doctor --fix` 會嚴格驗證
  舊的終端介面 JSON 檔案、保留較新的 SQLite 資料列、驗證規範結果，
  並移除未變更的舊版檔案，而非留下封存檔。
- Discord 命令部署雜湊現在位於共用外掛狀態 SQLite
  儲存區中。執行環境僅讀寫精確的應用程式範圍鍵。Doctor
  會直接刪除可重新建立的舊版 `discord/command-deploy-cache.json` 檔案
  而不匯入，因此下次啟動時會執行一次規範性協調。
- 預設 TTS 偏好設定現在位於共用外掛狀態 SQLite 資料列中，其鍵歸屬於
  `speech-core` 外掛。舊的 `settings/tts.json` 檔案僅作為 doctor 遷移
  輸入；執行環境不再讀寫 TTS 偏好設定 JSON 檔案，而
  舊版路徑解析器則位於 doctor 遷移模組中。
- 祕密目標中繼資料現在描述的是儲存區，不再假裝每個
  認證資訊目標都是設定檔。`openclaw.json` 仍是設定儲存區；
  驗證設定檔目標使用具型別的 SQLite `auth_profile_stores` 資料列，並將
  依提供者形狀組織的認證資訊保留為 JSON 承載資料。
- 祕密稽核不再掃描已淘汰的每代理程式 `auth.json` 檔案。Doctor 負責
  警告、匯入及移除該舊版檔案。
- 舊版驗證設定檔路徑輔助程式現在位於 doctor 舊版程式碼中。核心驗證
  設定檔路徑輔助程式公開的是 SQLite 驗證儲存區識別及顯示位置，
  而非 `auth-profiles.json` 或 `auth-state.json` 執行環境路徑。
- 子代理程式執行復原及 OpenRouter 模型能力快取執行環境模組
  現在會將 SQLite 快照讀取器／寫入器與僅供 doctor 使用的舊版 JSON
  匯入輔助程式分開。OpenRouter 能力使用 `provider_id = "openrouter"` 下具型別的通用
  `model_capability_cache` 資料列，而非
  單一不透明快取 Blob 或提供者專用主機資料表。子代理程式執行
  `taskName` 會儲存在具型別的 `subagent_runs.task_name` 欄位中；
  `payload_json` 副本是重播／偵錯資料，而非熱路徑顯示或
  查詢欄位的來源。
- `src/agents/filesystem/virtual-agent-fs.sqlite.ts` 會在代理程式資料庫
  `vfs_entries` 資料表上實作 SQLite VFS。目錄讀取、遞迴
  匯出、刪除及重新命名會使用具索引的 `(namespace, path)` 前綴範圍，
  而非掃描整個命名空間或依賴 `LIKE` 路徑比對。
- `src/agents/runtime-worker.entry.ts` 會為工作程序建立每次執行專用的 SQLite VFS、工具成品、
  執行成品及限定範圍的快取儲存區。
- 工作區啟動程序完成狀態、證明的新近程度，以及產生的啟動程序
  雜湊，現在會儲存在具型別的共用 `workspace_setup_state`、
  `workspace_path_aliases`、`workspace_attestations` 及
  `workspace_generated_bootstrap_hashes` 資料列中，並以標準工作區
  身分作為索引鍵。持久保存的詞法與實際路徑別名，可在設定的符號連結消失後
  維持已消失工作區的保護穩定；重新指向的別名會以封閉方式失敗。執行階段不再讀取或寫入
  `openclaw-workspace-state.json`、`.openclaw/workspace-state.json`、狀態目錄中的
  `workspace-attestations/*.attested`，或同層的 `<workspace>.attested`
  輔助檔案。`openclaw doctor --fix` 會驗證並認領舊版來源、
  將其匯入 SQLite 並附上遷移收據、驗證標準資料列，
  然後才移除已認領的檔案。
- 共用結構描述保留了一個 `exec_approvals_config` 單例資料列，但執行階段
  的切換仍待完成。TypeScript 與 macOS 輔助程式仍使用
  狀態範圍的 JSON 檔案，兩者必須一起移至 SQLite。
- TypeScript 裝置身分現在使用具型別的 `device_identities` 資料列，
  並將僅供 doctor 使用的舊版 JSON 匯入保留在執行階段擁有者之外。裝置驗證仍以
  檔案為後端，等待協調結構描述及跨執行階段遷移；
  `device_auth_tokens` 仍保留供該後續工作使用。
- GitHub Copilot 權杖交換快取會使用 `github-copilot/token-cache/default`
  下的共用 SQLite 外掛狀態資料表。這是由供應商擁有的快取狀態，
  因此刻意不新增主機結構描述資料表。
- GitHub Copilot 壓縮不再寫入 `openclaw-compaction-*.json`
  工作區輔助檔案。測試框架會針對追蹤中的 SDK 工作階段呼叫 SDK 歷史記錄壓縮 RPC，
  而 OpenClaw 會將持久的工作階段／逐字稿狀態保存在 SQLite 中，
  而非相容性標記檔案。
- 共用 Swift 執行階段（`OpenClawKit`）會針對裝置
  身分使用相同的 `state/openclaw.sqlite#table/device_identities` 形狀與資料列索引鍵。
  Apple 容器中的舊版檔案由 Swift 遷移擁有者匯入，因為 TypeScript Doctor
  無法存取這些容器。Swift 裝置驗證仍以檔案為後端，以待協調後續驗證工作。
- Android 裝置身分與快取的裝置驗證仍使用應用程式本機儲存區。
  它們需要由 Android 單獨負責的遷移；主機 SQLite 宣告並未描述
  目前的 Android 行為。
- Android 通知的近期套件歷史記錄使用具型別的
  `android_notification_recent_packages` 資料列。執行階段不再遷移或
  讀取舊的 SharedPreferences CSV 索引鍵。
- 當舊版 `identity/device.json` 存在、SQLite 身分資料列無效，
  或無法開啟 SQLite 身分儲存區時，裝置身分建立會以封閉方式失敗。Doctor 會先匯入並
  移除該檔案，因此執行階段啟動時無法在遷移前悄然輪替配對身分。
- 裝置身分選取使用 SQLite 資料列索引鍵，而非 JSON 檔案定位器。測試
  與閘道輔助程式會傳遞明確的身分索引鍵；只有 doctor 遷移與
  以封閉方式失敗的啟動閘門知道已淘汰的 `identity/device.json` 檔名。
- 工作階段重設相容性現在位於 doctor 設定遷移中：
  `session.idleMinutes` 會移至 `session.reset.idleMinutes`，
  `session.resetByType.dm` 會移至 `session.resetByType.direct`，而
  執行階段重設原則只會讀取標準重設索引鍵。
- 舊版設定相容性現在位於 `src/commands/doctor/` 下。一般的
  `readConfigFileSnapshot()` 驗證不會匯入 doctor 舊版偵測器，
  也不會標註舊版問題；`runDoctorConfigPreflight()` 會為
  doctor 修復／報告加入這些問題。doctor 設定流程會匯入
  `src/commands/doctor/legacy-config.ts`，而舊的 OAuth 設定檔 ID 修復則位於
  `src/commands/doctor/legacy/oauth-profile-ids.ts` 下。
- 非 doctor 命令不會自動執行舊版設定修復。例如，
  `openclaw update --channel` 現在遇到無效的舊版設定時會失敗，並要求
  使用者執行 doctor，而非悄然匯入 doctor 遷移程式碼。
- Web Push、APNs、語音喚醒、更新檢查及設定健康狀態現在會使用具型別的共用 SQLite
  資料表，分別儲存訂閱、VAPID 金鑰、節點註冊、觸發程序資料列、
  路由資料列、更新通知狀態及設定健康狀態項目，而非
  完整的不透明 JSON Blob。Web Push 與 APNs 寫入只會更新插入受影響的
  主鍵資料列；設定健康狀態則依設定路徑進行協調。其執行階段
  模組仍與僅供 Doctor 使用的舊版 JSON 匯入輔助程式分開。
- APNs 執行階段只會讀寫 `apns_registrations`。明確的
  `openclaw doctor --fix` 會嚴格匯入已淘汰的
  `push/apns-registrations.json`、保留現有的標準資料列、驗證
  交易、記錄收據，並移除包含機密資訊的 JSON。
  由收據支援的重試只會執行清理，而
  `apns_registration_tombstones` 會涵蓋首次修復前的失效處理，因此
  過期的轉送授權或裝置權杖無法復原。
- 節點主機設定現在使用共用 SQLite 資料庫中的具型別單例資料列。
  當舊的 `node.json` 檔案或中斷的認領作業仍存在時，執行階段會以封閉方式
  失敗；明確的 `openclaw doctor --fix` 會在正常執行階段使用前
  嚴格匯入並移除該檔案。
- 裝置／節點配對、頻道配對、頻道允許清單及啟動程序狀態，
  現在會使用具型別的 SQLite 資料列，而非完整的不透明 JSON Blob。外掛繫結
  核准與排程工作狀態也採用相同的拆分方式：執行階段模組會公開
  SQLite 後端作業與中立的快照輔助程式，而配對／啟動程序
  及外掛繫結核准快照寫入會依主鍵協調資料列，
  而非截斷資料表；doctor 則透過 `src/commands/doctor/legacy/*`
  模組匯入／移除舊的 JSON 檔案。
- 已安裝外掛記錄現在儲存在 SQLite 已安裝外掛索引中。
  執行階段設定讀寫不再遷移或保留舊的
  `plugins.installs` 編寫設定資料；doctor 會在正常執行階段使用前，
  將該舊版設定形狀匯入 SQLite。
- QQ Bot 認證資訊復原快照現在儲存在
  `qqbot/credential-backups` 下的 SQLite 外掛狀態中。執行階段不再寫入
  `qqbot/data/credential-backup*.json`；QQ Bot doctor 合約會從作用中的狀態目錄
  匯入並封存這些舊版備份檔案。
- 閘道重新載入規劃會在內部 `installedPluginIndex.installRecords.*`
  差異命名空間下比較 SQLite 已安裝外掛索引快照。執行階段
  重新載入決策不再將這些資料列包裝成假的 `plugins.installs` 設定
  物件。
- Matrix 帳戶認證資訊現在儲存在 SQLite 外掛狀態中。執行階段只會讀取
  該標準儲存區；當可解析其帳戶時，Doctor 會匯入、驗證並封存已淘汰的
  `credentials/matrix/credentials*.json` 檔案。
- 核心配對與排程執行階段模組不再使用舊版 JSON 路徑建構器。
  已淘汰的配對路徑 SDK 輔助程式仍保留僅供遷移使用的相容性；
  doctor 狀態遷移擁有其檔案讀取與匯入作業。由 Doctor 擁有的舊版
  模組只會為匯入測試與遷移建構 `pending.json`、`paired.json`、`bootstrap.json`
  及 `cron/jobs.json` 來源路徑。舊版排程
  工作形狀正規化與 JSONL 歷史記錄匯入位於
  `src/commands/doctor/cron/` 下；舊版 SQLite 歷史記錄的最終處理會在
  開啟狀態資料庫時執行。
- `src/commands/doctor/legacy/runtime-state.ts` 會由 doctor 將舊版 JSON 狀態
  檔案（包括節點主機設定）匯入 SQLite。新的舊版檔案
  匯入器會保留在 `src/commands/doctor/legacy/` 下。
- `src/commands/doctor/state-migrations.ts` 會將舊版 `sessions.json` 與
  `*.jsonl` 逐字稿直接匯入 SQLite，並移除已成功匯入的來源。它
  不再透過 `agents/<agentId>/sessions/*.jsonl` 暫存根目錄舊版逐字稿，
  也不會在匯入前建立標準 JSONL 目標。
- 狀態完整性 doctor 檢查不再掃描舊版工作階段目錄，
  也不再提供刪除孤立 JSONL 的選項。舊版逐字稿檔案僅作為遷移輸入，
  遷移步驟同時負責匯入與移除來源。
- 舊版沙箱登錄匯入位於
  `src/commands/doctor/legacy/sandbox-registry.ts` 下；作用中的沙箱登錄
  讀寫仍僅使用 SQLite。
- 舊版工作階段逐字稿健康狀態／匯入修復位於
  `src/commands/doctor/legacy/session-transcript-health.ts` 下；執行階段命令
  模組不再包含 JSONL 逐字稿剖析或作用中分支修復程式碼。

已完成的整併／刪除重點：

- 外掛狀態現在使用共用的 `state/openclaw.sqlite` 資料庫。舊的
  分支本機 `plugin-state/state.sqlite` 側車匯入器已移除，因為
  該 SQLite 配置從未發布。探測／測試輔助程式會回報共用的
  `databasePath`，而不是公開外掛狀態專用的 SQLite 路徑。
- Task 和 Task Flow 執行階段資料表現在位於共用的
  `state/openclaw.sqlite` 資料庫中，而非 `tasks/runs.sqlite` 和
  `tasks/flows/registry.sqlite`；舊的側車匯入器也因相同的未發布配置原因而移除。
- `src/config/sessions/store.ts` 不再需要以 `storePath` 處理傳入
  中繼資料、路由更新或更新時間讀取。命令持久化、命令列介面
  工作階段清理、子代理程式深度、驗證覆寫及逐字稿工作階段
  身分識別均使用代理程式／工作階段資料列 API。寫入會以 SQLite 資料列修補套用，
  並在樂觀式衝突時重試。
- 工作階段目標解析現在公開每個代理程式的資料庫目標，而非舊版
  `sessions.json` 路徑。共用閘道、ACP 中繼資料、doctor 路由修復及
  `openclaw sessions` 會列舉 `agent_databases` 和已設定的代理程式。
- 閘道工作階段路由現在使用 `resolveGatewaySessionDatabaseTarget`；回傳的
  目標會攜帶 `databasePath` 和候選 SQLite 資料列鍵，而非
  舊版工作階段儲存檔案路徑。
- 頻道工作階段執行階段型別現在公開 `{agentId, sessionKey}`，用於
  更新時間讀取、傳入中繼資料和最後路由更新。舊的
  `saveSessionStore(storePath, store)` 相容性型別已移除。
- 外掛執行階段、擴充功能 API 和外掛 SDK 工作階段介面現在公開
  由 SQLite 支援的工作階段資料列輔助程式，而非作用中工作階段的整體儲存區／檔案
  相容性輔助程式。根程式庫相容性匯出僅在外掛 SDK 之外保留，
  供舊版內部及遷移呼叫端使用。舊的 `resolveLegacySessionStorePath` 輔助程式已移除；
  舊版 `sessions.json` 路徑建構現在僅存在於遷移及測試固定資料中。
- `src/config/sessions/session-entries.sqlite.ts` 現在會將標準工作階段
  項目儲存在每個代理程式的資料庫中，並支援資料列層級的讀取／更新插入／刪除修補。
  執行階段更新插入／修補／刪除不再掃描大小寫變體或
  清除舊版別名鍵；標準化由 doctor 負責。獨立的
  JSON 匯入輔助程式已移除，且遷移合併會更新插入較新的資料列，
  而非取代整個工作階段資料表。公用讀取／列出／載入輔助程式
  會從具型別的 `sessions` 和 `conversations` 資料列投影常用工作階段中繼資料；
  `entry_json` 是相容性／偵錯影子資料，即使過時或無效，
  也不會遺失具型別的工作階段身分或傳遞內容。
- `src/config/sessions/delivery-info.ts` 現在會從具型別的每代理程式
  `sessions` + `conversations` + `session_conversations` 資料列解析傳遞內容。
  它不再從 `session_entries.entry_json` 重建執行階段傳遞身分；
  缺少具型別的對話資料列屬於 doctor 遷移／修復問題，
  而非執行階段備援。
- 已儲存工作階段的重設決策現在優先採用具型別的 `sessions.session_scope`、
  `sessions.chat_type` 和 `sessions.channel` 中繼資料。`sessionKey` 剖析
  僅保留用於命令目標上的明確討論串／主題後綴；群組與
  直接重設的分類不再取決於鍵的形狀。
- 工作階段清單／狀態顯示分類現在使用具型別的聊天中繼資料和
  閘道工作階段種類。它不再將 `session_key` 內的 `:group:` 或 `:channel:`
  子字串視為持久可靠的群組／直接對話判定依據。
- 靜默回覆政策的選取現在只使用明確的對話型別或介面
  中繼資料。它不再根據 `session_key` 子字串猜測
  直接／群組政策。
- 工作階段顯示模型解析現在會從 SQLite 工作階段
  資料庫目標取得代理程式 ID，而非從 `session_key` 中拆分。
- 代理程式間宣告目標的資料補全現在僅使用具型別的 `sessions.list`
  `deliveryContext`。它不再從舊版 `origin`、鏡像的 `last*` 欄位
  或 `session_key` 形狀復原頻道／帳號／討論串路由。
- `sessions_send` 的討論串目標拒絕現在會讀取具型別的 SQLite 路由
  中繼資料。它不再透過從目標鍵剖析討論串後綴來拒絕或接受目標。
- 群組範圍工具政策驗證現在會讀取目前或新建立工作階段的
  具型別 SQLite 對話路由。它不再透過解碼 `sessionKey` 來信任群組／頻道
  身分；若沒有具型別的工作階段資料列為呼叫端提供的群組 ID 背書，
  這些 ID 會遭到捨棄。
- 頻道模型覆寫比對現在使用明確的群組和父層
  對話中繼資料。它不再從 `parentSessionKey` 解碼父層對話 ID。
- 已儲存模型覆寫的繼承現在要求具型別工作階段內容提供明確的父層工作階段鍵。
  它不再從 `sessionKey` 中的 `:thread:` 或 `:topic:`
  後綴衍生父層覆寫。
- 舊的工作階段討論串資訊包裝函式和已載入外掛討論串剖析器已移除；
  執行階段程式碼不再匯入 `config/sessions/thread-info`。
- 頻道對話輔助程式不再公開完整工作階段鍵的剖析
  橋接。核心仍會透過 `resolveSessionConversation(...)` 正規化供應商擁有的原始對話 ID，
  但不會從 `sessionKey` 重建路由事實。
- 完成傳遞、傳送政策和任務維護不再從 `session_key`
  的形狀衍生聊天型別。舊的聊天型別鍵剖析器已刪除；
  這些路徑需要具型別的工作階段中繼資料、具型別的傳遞內容，
  或明確的傳遞目標詞彙。
- 工作階段清單／狀態、診斷、核准帳號繫結、終端介面心跳偵測
  篩選及用量摘要不再從 `SessionEntry.origin` 挖掘
  供應商／帳號／討論串／顯示路由。僅存的執行階段
  `origin` 讀取都是非工作階段概念或目前回合的傳遞物件。
- 核准要求的原生對話查詢現在會讀取具型別的每代理程式工作階段
  路由資料列。它不再從 `sessionKey` 剖析頻道／群組／討論串對話身分；
  缺少具型別的中繼資料屬於遷移／修復問題。
- 閘道工作階段變更／聊天／工作階段事件承載資料不再回傳
  `SessionEntry.origin` 或 `last*` 路由影子資料；用戶端會收到具型別的
  `channel`、`chatType` 和 `deliveryContext`。
- 心跳偵測傳遞解析現在可以直接接收具型別的 SQLite
  `deliveryContext`，而且心跳偵測執行階段會傳遞每個代理程式的
  工作階段傳遞資料列，不再依賴相容性 `session_entries`
  影子資料取得目前路由。
- 排程隔離代理程式的傳遞目標解析也會先從具型別的
  每代理程式工作階段傳遞資料列補全目前路由，再退回使用
  相容性項目承載資料。
- 子代理程式宣告來源解析現在會透過 `loadRequesterSessionEntry` 傳遞具型別的要求端工作階段
  傳遞內容，並優先使用該資料列，而非相容性
  `last*`/`deliveryContext` 影子資料。
- 傳入工作階段中繼資料更新現在會先與具型別的每代理程式
  傳遞資料列合併；只有在不存在具型別對話資料列時，舊的 `SessionEntry`
  傳遞欄位才會作為備援。
- 重新啟動／更新傳遞擷取現在會讓具型別的 SQLite 傳遞
  `threadId` 優先於從 `sessionKey` 剖析出的主題／討論串片段；
  剖析僅作為舊版討論串形狀鍵的備援。
- 掛鉤代理程式內容的頻道 ID 現在優先採用具型別的 SQLite 對話身分，
  其次才是明確的訊息中繼資料。它們不再從 `sessionKey`
  剖析供應商／群組／頻道片段。
- 閘道 `chat.send` 外部路由繼承現在會讀取具型別的 SQLite 工作階段
  路由中繼資料，而非從 `sessionKey` 片段推斷頻道／直接／群組範圍。
  只有在具型別的工作階段頻道和聊天型別與已儲存的傳遞內容相符時，
  頻道範圍工作階段才會繼承；共用主工作階段則保留較嚴格的
  命令列介面／無用戶端中繼資料規則。
- 重新啟動哨兵喚醒和接續路由現在會先讀取具型別的 SQLite
  傳遞／路由資料列，再將心跳偵測喚醒或已路由的代理程式回合
  接續作業排入佇列。它不再從工作階段項目的 JSON 影子資料
  重建傳遞內容。
- 閘道 `tools.effective` 內容解析現在會讀取具型別的 SQLite
  傳遞／路由資料列，以取得供應商、帳號、目標、討論串和回覆模式
  輸入。它不再從過時的 `session_entries.entry_json` 來源影子資料
  復原這些常用路由欄位。
- 即時語音諮詢路由現在會從具型別的每代理程式 SQLite
  工作階段資料列解析父層／通話傳遞。選擇內嵌代理程式
  訊息路由時，它不再退回使用相容性 `SessionEntry.deliveryContext` 影子資料。
- ACP 建立項目的心跳偵測轉送和父層串流路由現在會從具型別的 SQLite
  工作階段資料列讀取父層傳遞。它們不再從相容性工作階段項目
  影子資料重建父層傳遞內容。
- 工作階段傳遞路由保留現在依循具型別的聊天中繼資料和
  持久化傳遞欄位。它不再從 `sessionKey` 擷取頻道提示、直接／主
  標記或討論串形狀；只有當 SQLite 已具有該工作階段具型別／持久化的
  傳遞身分時，內部網頁聊天路由才會繼承外部目標。
- 通用工作階段傳遞擷取現在只會讀取完全相符的具型別 SQLite
  工作階段傳遞資料列。它不再剖析討論串／主題後綴，也不會
  從討論串形狀鍵退回至基礎工作階段鍵。
- 回覆分派、重新啟動哨兵復原和即時語音諮詢路由
  現在會使用完全相符的具型別 SQLite 工作階段／對話資料列處理討論串路由。
  它們不再透過剖析討論串形狀的工作階段鍵來復原討論串 ID
  或基礎工作階段傳遞內容。
- 內嵌 PI 歷程限制現在會使用具型別的 SQLite 工作階段路由
  投影（`sessions` + 主要 `conversations`）取得供應商、聊天型別
  和對等端身分。它不再從 `sessionKey` 剖析供應商、私訊、群組
  或討論串形狀。
- 排程工具傳遞推斷現在只使用明確的傳遞資訊或目前具型別的
  傳遞內容。它不再從 `agentSessionKey` 解碼頻道、對等端、帳號
  或討論串目標。
- 執行階段工作階段資料列不再攜帶舊的 `lastProvider` 路由別名。
  輔助程式和測試會使用具型別的 `lastChannel` 和 `deliveryContext` 欄位；
  doctor 遷移是唯一應轉換較舊路由別名或持久化
  `origin` 影子資料的位置。
- 逐字稿事件、VFS 資料列和工具成品資料列現在會寫入每個代理程式的
  資料庫。未發布的全域逐字稿檔案對應資料表已移除；doctor
  改為在持久遷移資料列中記錄舊版來源路徑。
- 執行階段逐字稿查詢不再掃描 JSONL 位元組偏移或探測舊版
  逐字稿檔案。閘道聊天／媒體／歷程路徑會從 SQLite 讀取逐字稿資料列；
  工作階段 JSONL 現在僅是舊版 doctor 輸入，而非執行階段狀態
  或匯出格式。
- 逐字稿的父層和分支關係會使用 SQLite 逐字稿
  標頭中的結構化 `parentTranscriptScope: {agentId, sessionId}` 中繼資料，而非類似路徑的
  `agent-db:...transcript_events...` 定位器字串。
- 逐字稿管理器合約不再公開隱含持久化的
  `create(cwd)` 或 `continueRecent(cwd)` 建構函式。持久化逐字稿
  管理器會使用明確的 `{agentId, sessionId}` 範圍開啟；僅限
  記憶體內管理器在測試和純逐字稿轉換中仍不受範圍限制。
- 執行階段逐字稿儲存區 API 解析 SQLite 範圍，而非檔案系統路徑。舊的 `resolve...ForPath` 輔助函式和未使用的 `transcriptPath` 寫入選項已從執行階段呼叫端移除。
- 執行階段工作階段解析現在使用 `{agentId, sessionId}`，且不得為外部邊界衍生 `sqlite-transcript://<agent>/<session>` 字串。舊版絕對 JSONL 路徑僅能作為 doctor 遷移輸入。
- 原生掛鉤轉送的直接橋接記錄現在位於具型別的共用 `native_hook_relay_bridges` 資料列中，並以轉送 ID 為鍵。執行階段不再為這些短期橋接記錄寫入 `/tmp` JSON 登錄檔或不透明的通用記錄。
- `runEmbeddedPiAgent(...)` 不再具有逐字稿定位器參數。預備的工作器描述元也省略逐字稿定位器。執行階段工作階段狀態和排入佇列的後續執行會攜帶 `{agentId, sessionId}`，而非衍生的逐字稿控制代碼。
- 內嵌壓縮現在從 `agentId` 和 `sessionId` 取得 SQLite 範圍。壓縮掛鉤、內容引擎呼叫、命令列介面委派和通訊協定回覆不得接收衍生的 `sqlite-transcript://...` 控制代碼。匯出／偵錯程式碼可以從資料列具體化明確的使用者成品，但不提供通用的工作階段 JSONL 匯出路徑，也不會將檔案名稱送回執行階段身分。
- `/export-session` 從 SQLite 讀取逐字稿資料列，且僅寫入要求的獨立 HTML 檢視。內嵌檢視器不再從這些資料列重建或下載工作階段 JSONL。
- 內容引擎委派不再解析逐字稿定位器來復原代理程式身分。預備的執行階段內容會將已解析的 `agentId` 攜帶至內建壓縮配接器。
- 逐字稿重寫和即時工具結果截斷現在依 `{agentId, sessionId}` 讀取並保存逐字稿狀態，且不會為逐字稿更新事件承載資料衍生暫時定位器。
- 逐字稿狀態輔助介面不再具有以定位器為基礎的 `readTranscriptState`、`replaceTranscriptStateEvents` 或 `persistTranscriptStateMutation` 變體。執行階段呼叫端必須使用 `{agentId, sessionId}` API。Doctor 匯入會依明確的檔案路徑讀取舊版檔案並寫入 SQLite 資料列；它不會遷移定位器字串。
- 執行階段工作階段管理器合約不再公開 `open(locator)`、`forkFrom(locator)` 或 `setTranscriptLocator(...)`。持久化工作階段管理器僅依 `{agentId, sessionId}` 開啟；列出／分支輔助函式位於以資料列為導向的工作階段和檢查點 API，而非逐字稿管理器外觀。
- 閘道逐字稿讀取器 API 以範圍優先。它們接受 `{agentId, sessionId}`，且不接受可能意外成為執行階段身分的位置式逐字稿定位器。作用中逐字稿定位器解析已移除；舊版來源路徑僅由 doctor 匯入程式碼讀取。
- 逐字稿更新事件也以範圍優先。`emitSessionTranscriptUpdate` 不再接受單獨的定位器字串，而接聽器會依 `{agentId, sessionId}` 路由，無須解析控制代碼。
- 閘道工作階段訊息廣播會從代理程式／工作階段範圍解析工作階段金鑰，而非從逐字稿定位器解析。舊的逐字稿定位器轉工作階段金鑰解析器／快取已移除。
- 閘道工作階段歷程 SSE 會依代理程式／工作階段範圍篩選即時更新。它不再將逐字稿定位器候選項、實體路徑或檔案形狀的逐字稿身分正規化，以判斷串流是否應接收更新。
- 工作階段生命週期掛鉤不再於 `session_end` 上衍生或公開逐字稿定位器。掛鉤取用端會取得 `sessionId`、`sessionKey`、下一個工作階段 ID 和代理程式內容；逐字稿檔案不屬於生命週期合約。
- 重設掛鉤也不再衍生或公開逐字稿定位器。`before_reset` 承載資料會攜帶復原的 SQLite 訊息和重設原因，而工作階段身分則保留在掛鉤內容中。
- 代理程式測試框架重設不再接受逐字稿定位器。重設分派的範圍由 `sessionId`/`sessionKey` 加上原因決定。
- 代理程式擴充功能工作階段型別不再公開 `transcriptLocator`；擴充功能應使用工作階段內容和執行階段 API，而非存取檔案形狀的逐字稿身分。
- 外掛壓縮掛鉤不再公開逐字稿定位器。掛鉤內容已攜帶工作階段身分，而逐字稿讀取必須透過支援 SQLite 範圍的 API 進行，而非使用檔案形狀的控制代碼。
- `before_agent_finalize` 掛鉤不再公開 `transcriptPath`，包括原生掛鉤轉送承載資料。完成掛鉤僅使用工作階段內容。
- 閘道重設回應不再於傳回的項目上合成逐字稿定位器。重設會建立 SQLite 逐字稿資料列、傳回乾淨的工作階段項目，並將逐字稿存取交由支援範圍的讀取器處理。
- 內嵌執行和壓縮結果不再為工作階段計量公開逐字稿定位器。自動壓縮僅更新作用中的 `sessionId`、壓縮計數器和權杖中繼資料。
- 內嵌嘗試結果不再傳回 `transcriptLocatorUsed`，而內容引擎 `compact()` 結果也不再傳回逐字稿定位器。執行階段重試迴圈僅接受後繼 `sessionId`。
- 傳遞鏡像逐字稿附加結果不再傳回逐字稿定位器。呼叫端會取得已附加的 `messageId`；逐字稿更新訊號使用 SQLite 範圍。
- 父工作階段分支輔助函式僅傳回分支後的 `sessionId`。子代理程式準備會將子代理程式／工作階段範圍傳遞給引擎。
- 命令列介面執行器參數和歷程重新植入不再接受逐字稿定位器。命令列介面歷程讀取會從 `{agentId,
sessionId}` 和工作階段金鑰內容解析 SQLite 逐字稿範圍。
- 命令列介面和內嵌執行器測試固定資料現在會依工作階段 ID 植入及讀取 SQLite 逐字稿資料列，而非假裝作用中工作階段是 `*.jsonl` 檔案，或透過執行階段參數傳遞 `sqlite-transcript://...` 字串。
- 即使記憶體內管理器沒有衍生的定位器，工作階段工具結果防護事件也會從已知工作階段範圍發出。其測試不再偽造作用中的 `/tmp/*.jsonl` 逐字稿檔案。
- BTW 和壓縮檢查點輔助函式現在會依 SQLite 範圍讀取並分支逐字稿資料列。檢查點中繼資料現在僅儲存工作階段 ID 和分葉／項目 ID；衍生的定位器不再寫入檢查點承載資料。
- 閘道逐字稿金鑰查詢在通訊協定邊界使用 SQLite 逐字稿範圍，且不再對逐字稿檔案名稱執行 realpath 或 stat。
- 自動壓縮逐字稿輪替會直接透過 SQLite 逐字稿儲存區寫入後繼逐字稿資料列。工作階段資料列僅保留後繼工作階段身分，而非永久 JSONL 路徑或持久化定位器。
- 內嵌內容引擎壓縮使用以 SQLite 命名的逐字稿輪替輔助函式。輪替測試不再建構 JSONL 後繼路徑，也不再將作用中工作階段模型化為檔案。
- 受管理的外送圖片保留會根據 SQLite 逐字稿統計資料設定其逐字稿訊息快取金鑰，而非呼叫檔案系統 stat。
- 執行階段工作階段鎖定和獨立的舊版 `.jsonl.lock` doctor 路徑已移除。
- Microsoft Teams 執行階段彙總模組和公用外掛 SDK 不再重新匯出舊的檔案鎖定輔助函式；持久外掛狀態路徑以 SQLite 為後端。
- 工作階段依存續時間／數量修剪和明確的工作階段清理已移除。Doctor 負責舊版匯入；過時工作階段會明確重設或刪除。
- Doctor 完整性檢查不再將舊版 JSONL 檔案視為 SQLite 工作階段資料列的有效作用中逐字稿。作用中逐字稿健康狀態僅以 SQLite 為準；舊版 JSONL 檔案會回報為遷移／孤立項目清理輸入。
- Doctor 不再將 `agents/<agent>/sessions/` 視為必要的執行階段狀態。它僅會在該目錄已存在時掃描，並將其作為舊版匯入或孤立項目清理輸入。
- 閘道 `sessions.resolve`、工作階段修補／重設／壓縮路徑、子代理程式產生、快速中止、ACP 中繼資料、心跳偵測隔離工作階段和終端介面修補，不再將遷移或修剪舊版工作階段金鑰作為一般執行階段工作的副作用。
- 命令列介面命令工作階段解析現在會傳回擁有該工作階段的 `agentId`，而非 `storePath`，且不再於一般 `--to` 或 `--session-id` 解析期間複製舊版主工作階段資料列。舊版主資料列正規化僅由 doctor 負責。
- 執行階段子代理程式深度解析不再讀取 `sessions.json` 或 JSON5 工作階段儲存區。它會依代理程式 ID 讀取 SQLite `session_entries`，而舊版深度／工作階段中繼資料只能透過 doctor 匯入路徑進入。
- 驗證設定檔工作階段覆寫會透過直接的 `{agentId, sessionKey}` 資料列 upsert 持久化，而非延遲載入檔案形狀的工作階段儲存區執行階段。
- 自動回覆詳細模式閘控和工作階段更新輔助函式現在會依工作階段身分讀取／upsert SQLite 工作階段資料列，且不再要求先有舊版儲存區路徑才能處理持久化資料列狀態。
- 命令執行工作階段中繼資料輔助函式現在使用以項目為導向的名稱和模組路徑；舊的 `session-store` 命令輔助介面已移除。
- 啟動程序標頭植入和手動壓縮邊界強化現在會直接變更 SQLite 逐字稿資料列。執行階段呼叫端傳遞工作階段身分，而非可寫入的 `.jsonl` 路徑。
- 靜默工作階段輪替重播會依 `{agentId, sessionId}` 從 SQLite 逐字稿資料列複製最近的使用者／助理對話輪次。它不再接受來源或目標逐字稿定位器。
- 全新的執行階段工作階段資料列不再儲存逐字稿定位器。呼叫端會直接使用 `{agentId, sessionId}`；匯出／偵錯命令在具體化資料列時可選擇輸出檔案名稱。
- 啟動新的持久化逐字稿工作階段時，現在一律會依範圍開啟 SQLite 資料列。工作階段管理器不再重複使用先前檔案時代的逐字稿路徑或定位器作為新工作階段的身分。
- 持久化逐字稿工作階段使用明確的 `openTranscriptSessionManagerForSession({agentId, sessionId})` API。舊的靜態 `SessionManager.create/openForSession/list/forkFromSession` 外觀已移除，因此測試和執行階段程式碼不會意外重新建立檔案時代的工作階段探索機制。
- 外掛執行階段不再公開 `api.runtime.agent.session.resolveTranscriptLocatorPath`；外掛程式碼會使用 SQLite 資料列輔助函式和範圍值。
- 公用 `session-store-runtime` SDK 介面現在僅匯出工作階段資料列和逐字稿資料列輔助函式。專用的 SQLite 結構描述／路徑／交易輔助函式位於 `sqlite-runtime`；原始開啟／關閉／重設輔助函式仍僅供第一方測試在本機使用。
- 舊版 `.jsonl` 軌跡／檢查點檔案名稱分類器現在位於 doctor 舊版工作階段檔案模組中。核心工作階段驗證不再匯入檔案成品輔助函式來判定一般 SQLite 工作階段 ID。
- 主動記憶阻塞式子代理程式執行會使用 SQLite 逐字稿資料列，而非在外掛狀態下建立暫時或持久化的 `session.jsonl` 檔案。舊的 `transcriptDir` 選項已移除。
- 一次性 slug 產生和系統代理程式規劃器執行會使用 SQLite 逐字稿資料列，而非建立暫時的 `session.jsonl` 檔案。
- `llm-task` 輔助程式執行及隱藏承諾擷取也使用 SQLite
  逐字稿資料列，因此這些僅供模型使用的輔助工作階段不再建立
  暫存 JSON/JSONL 逐字稿檔案。
- `TranscriptSessionManager` 現在只是一個已開啟的 SQLite 逐字稿範圍。
  執行階段程式碼使用 `openTranscriptSessionManagerForSession({agentId,
sessionId})` 開啟它；建立、分支、繼續、列出及分岔流程位於其
  所屬的 SQLite 資料列輔助程式中，而非靜態管理器外觀。
  Doctor／匯入／偵錯程式碼會在執行階段工作階段管理器之外處理明確的舊版來源檔案。
- 已移除過時的 `SessionManager.newSession()` 和
  `SessionManager.createBranchedSession()` 外觀方法。新的
  工作階段及逐字稿後代由其所屬的 SQLite
  工作流程建立，而非將已開啟的管理器變更為不同的
  持久化工作階段。
- 父逐字稿分岔決策及分岔建立不再接受
  `storePath` 或 `sessionsDir`；它們使用 `{agentId, sessionId}` SQLite
  逐字稿範圍，而非保留的檔案系統路徑中繼資料。
- 記憶體主機不再匯出無作用的工作階段目錄逐字稿
  分類輔助程式；逐字稿篩選現在會在建構項目時從 SQLite 資料列
  中繼資料衍生。
- 記憶體主機和 QMD 工作階段匯出測試使用 SQLite 逐字稿範圍。舊的
  `agents/<agentId>/sessions/*.jsonl` 路徑僅在測試刻意驗證
  Doctor／匯入／匯出相容性時繼續納入涵蓋。
- QA-lab 原始工作階段檢查現在透過閘道使用 `sessions.list`，
  而非讀取 `agents/qa/sessions/sessions.json`；MSteams 意見回饋會直接附加至
  SQLite 逐字稿，而不會虛構 JSONL 路徑。
- 共用的傳入頻道回合現在攜帶 `{agentId, sessionKey}`，而非
  舊版 `storePath`。LINE、WhatsApp、Slack、Discord、Telegram、Matrix、Signal、
  iMessage、BlueBubbles、Feishu、Google Chat、IRC、Nextcloud Talk、Zalo、
  Zalo Personal、QA Channel、Microsoft Teams、Mattermost、Synology Chat、Tlon、
  Twitch 及 QQ Bot 的記錄路徑現在會讀取更新時間中繼資料，並透過
  SQLite 身分記錄傳入工作階段資料列。
- 已從作用中工作階段資料列移除逐字稿定位器持久化。
  `resolveSessionTranscriptTarget` 會傳回 `agentId`、`sessionId` 及選用的
  主題中繼資料；只有 Doctor 程式碼會匯入舊版逐字稿檔案
  名稱。
- 執行階段逐字稿標頭從 SQLite 版本 `1` 開始。舊版 JSONL V1/V2/V3
  格式升級僅存在於 Doctor 匯入中，並會在儲存資料列前將匯入的標頭正規化為
  目前的 SQLite 逐字稿版本。
- 資料庫優先防護現在禁止 `SessionManager.listAll` 和
  `SessionManager.forkFromSession`；工作階段列出及分岔／還原工作流程
  必須維持使用資料列／範圍式 SQLite API。
- 此防護也會禁止 Doctor／匯入程式碼以外的舊版逐字稿 JSONL 剖析／作用中分支修復輔助程式
  名稱，使執行階段無法發展出第二條舊版
  逐字稿遷移路徑。
- 內嵌 PI 執行會拒絕傳入的逐字稿控制代碼。它們會在工作執行緒啟動前，
  以及嘗試接觸逐字稿狀態前，再次使用 SQLite
  `{agentId, sessionId}` 身分。過時的 `/tmp/*.jsonl` 輸入無法選取
  執行階段寫入目標。
- 快取追蹤、Anthropic 承載資料、原始串流及診斷時間軸記錄
  現在會寫入具型別的 SQLite `diagnostic_events` 資料列。閘道穩定性套件
  現在會寫入具型別的 SQLite `diagnostic_stability_bundles` 資料列。舊的
  `diagnostics.cacheTrace.filePath`、`OPENCLAW_CACHE_TRACE_FILE`、
  `OPENCLAW_ANTHROPIC_PAYLOAD_LOG_FILE` 和
  `OPENCLAW_DIAGNOSTICS_TIMELINE_PATH` JSONL 覆寫路徑已移除，且
  一般穩定性擷取不再寫入 `logs/stability/*.json` 檔案。
- 排程持久化現在會協調 SQLite `cron_jobs` 資料列，而非
  每次儲存時刪除並重新插入整個工作資料表。外掛目標
  寫回會直接更新相符的排程資料列，並使執行階段排程狀態維持於
  同一個狀態資料庫交易中。
- 排程執行階段呼叫端現在使用穩定的 SQLite 排程儲存區索引鍵。舊版
  `cron.store` 路徑僅供 Doctor 匯入使用；正式環境閘道、工作
  維護、狀態、執行歷程及 Telegram 目標寫回路徑使用
  `resolveCronStoreKey`，且不再對索引鍵進行路徑正規化。排程狀態現在
  回報 `storeKey`，而非舊有檔案形態的 `storePath` 欄位。
- 排程執行階段載入及排程不再正規化舊版持久化工作
  格式，例如 `jobId`、`schedule.cron`、數值型 `atMs`、字串布林值或
  缺少的 `sessionTarget`。Doctor 舊版匯入會在將資料列
  插入 SQLite 前負責這些修復。
- ACP 產生不再解析或持久化逐字稿 JSONL 檔案路徑。產生
  及執行緒繫結設定會直接持久化 SQLite 工作階段資料列，並保留
  工作階段 ID 作為逐字稿身分。
- ACP 工作階段中繼資料 API 現在依 `agentId` 讀取／列出／向上插入 SQLite 資料列，
  且不再將 `storePath` 公開為 ACP 工作階段項目合約的一部分。
- 工作階段用量計算及閘道用量彙總現在僅依
  `{agentId, sessionId}` 解析逐字稿。成本／用量快取及探索到的工作階段
  摘要不再合成或傳回逐字稿定位器字串。
- 閘道聊天附加、中止部分持久化、`/sessions.send` 及
  網頁聊天媒體逐字稿寫入會直接透過 SQLite 逐字稿
  範圍附加。閘道逐字稿注入輔助程式不再接受
  `transcriptLocator` 參數。
- SQLite 逐字稿探索現在僅列出逐字稿範圍及統計資料：
  `{agentId, sessionId, updatedAt, eventCount}`。已無作用的
  `listSqliteSessionTranscriptLocators` 相容性輔助程式及每資料列
  `locator` 欄位均已移除。
- 逐字稿修復執行階段現在僅公開
  `repairTranscriptSessionStateIfNeeded({agentId, sessionId})`。舊的
  定位器式修復輔助程式已刪除；Doctor／偵錯程式碼會讀取明確的
  來源檔案路徑，且絕不遷移定位器字串。
- ACP 重播分類帳執行階段現在會將各工作階段的重播資料列儲存於共用
  SQLite 狀態資料庫，而非 `acp/event-ledger.json`；Doctor 會匯入並
  移除舊版檔案。
- 閘道逐字稿讀取器輔助程式現在位於
  `src/gateway/session-transcript-readers.ts`，而非舊的
  `session-utils.fs` 模組名稱。備援重試歷程檢查現在依
  SQLite 逐字稿內容命名，而非舊的檔案輔助程式介面。
- 閘道注入聊天及壓縮輔助程式現在會透過內部輔助程式 API 傳遞 SQLite 逐字稿範圍，
  而不再將值命名為逐字稿路徑或
  來源檔案。
- 啟動程序接續偵測現在透過
  `hasCompletedBootstrapTranscriptTurn` 檢查 SQLite 逐字稿資料列；它不再公開檔案形態的
  輔助程式名稱。
- 內嵌執行器測試現在使用 SQLite 逐字稿身分，且開啟新的
  逐字稿管理器一律需要明確的 `sessionId`。
- 記憶體索引輔助程式現在從頭到尾使用 SQLite 逐字稿術語：
  主機匯出 `listSessionTranscriptScopesForAgent` 和
  `sessionTranscriptKeyForScope`、定向同步會將 `sessionTranscripts` 排入佇列、
  公開工作階段搜尋命中項目會公開不透明的 `transcript:<agent>:<session>` 路徑，
  且內部資料庫來源索引鍵為 `source_kind='sessions'` 下的
  `session:<session>`，而非虛假檔案路徑。
- 通用外掛 SDK 持久去重輔助程式不再公開檔案形態的
  選項。呼叫端提供 SQLite 範圍索引鍵，而耐久的去重資料列則位於
  共用外掛狀態中。
- Microsoft Teams SSO 權杖已從鎖定的 JSON 檔案移至 SQLite 外掛
  狀態。Doctor 會匯入 `msteams-sso-tokens.json`、從承載資料重建標準 SSO 權杖
  索引鍵，並移除來源檔案。委派的 OAuth 權杖仍維持在
  現有的私有認證資訊檔案邊界。
- Matrix 同步快取狀態已從 `bot-storage.json` 移至 SQLite 外掛
  狀態。Doctor 會匯入舊版原始或包裝過的同步承載資料，並移除
  來源檔案。作用中的 Matrix 及 QA Lab Matrix 配接器用戶端會傳入 SQLite 同步儲存區根
  目錄，而非虛假的 `sync-store.json` 或 `bot-storage.json` 路徑。
- Matrix 舊版加密遷移狀態已從
  `legacy-crypto-migration.json` 移至 SQLite 外掛狀態。Doctor 會匯入
  舊的狀態檔案；Matrix SDK IndexedDB 快照已從
  `crypto-idb-snapshot.json` 移至 SQLite 外掛二進位大型物件。Matrix 復原金鑰及
  認證資訊是 SQLite 外掛狀態資料列；其舊 JSON 檔案僅供 Doctor
  遷移輸入使用。
- Memory Wiki 活動記錄現在使用 SQLite 外掛狀態，而非
  `.openclaw-wiki/log.jsonl`。Memory Wiki 遷移提供者會匯入舊的
  JSONL 記錄；Wiki Markdown 及使用者保存庫內容仍以檔案形式
  儲存為工作區內容。
- Memory Wiki 不再建立 `.openclaw-wiki/state.json` 或未使用的
  `.openclaw-wiki/locks` 目錄。如果較舊的保存庫仍有這些已淘汰的
  外掛中繼資料檔案，遷移提供者會將其移除。
- 系統代理程式稽核項目現在使用核心 SQLite 外掛狀態，而非
  `audit/crestodian.jsonl`。Doctor 會匯入舊版 JSONL 稽核記錄，並在
  成功匯入後移除該記錄。
- 設定寫入／觀察稽核項目現在使用核心 SQLite 外掛狀態，而非
  `logs/config-audit.jsonl`。Doctor 會匯入舊版 JSONL 稽核記錄，並在
  成功匯入後移除該記錄。
- macOS 輔助應用程式在編輯 `openclaw.json` 時，不再寫入應用程式本機的
  `logs/config-audit.jsonl` 或 `logs/config-health.json` 附屬檔案。設定
  檔案仍以檔案形式儲存，復原快照仍位於設定檔案旁，
  而耐久的設定稽核／健康狀態則屬於閘道 SQLite 儲存區。
- 系統代理程式救援待核准項目現在使用核心 SQLite 外掛狀態，而非
  `crestodian/rescue-pending/*.json` 或 `openclaw/rescue-pending/*.json`。
  這些短期安全能力絕不會被匯入；Doctor 會捨棄
  這兩個已淘汰的目錄，使升級無法重新啟用過時的寫入。
- Phone Control 暫時啟用狀態現在使用 SQLite 外掛狀態，而非
  `plugins/phone-control/armed.json`。Doctor 會將舊版已啟用狀態
  檔案匯入 `phone-control/arm-state` 命名空間，並移除該檔案。
- Doctor 不再就地修復 JSONL 逐字稿或建立備份 JSONL
  檔案。它會將作用中分支匯入 SQLite，並移除舊版來源。
- 工作階段記憶體掛鉤逐字稿查詢使用 `{agentId, sessionId}` 純範圍
  SQLite 讀取。其輔助程式不再接受或衍生逐字稿定位器、
  舊版檔案讀取或檔案重寫選項。
- Codex 應用程式伺服器對話繫結現在會依
  OpenClaw 工作階段索引鍵或明確的 `{agentId, sessionId}` 範圍來設定 SQLite 外掛狀態索引鍵。它們不得
  保留逐字稿路徑備援繫結。
- Codex 應用程式伺服器鏡像歷程讀取僅使用 SQLite 逐字稿範圍；
  它們不得從逐字稿檔案路徑復原身分。
- 角色排序及壓縮重設路徑不再取消連結舊的逐字稿
  檔案；重設只會輪替 SQLite 工作階段資料列及逐字稿身分。
- 閘道重設及檢查點回應會傳回乾淨的工作階段資料列及工作階段
  ID。它們不再為用戶端合成 SQLite 逐字稿定位器。
- 記憶體核心夢境整理不再透過探查缺少的
  JSONL 檔案來修剪工作階段資料列。子代理程式清理會透過工作階段執行階段 API，而非
  檔案系統存在性檢查進行。其逐字稿擷取測試會直接植入 SQLite 資料列，
  而非建立 `agents/<id>/sessions` 固定資料或定位器
  預留位置。
- 記憶體逐字稿索引可將 `transcript:<agentId>:<sessionId>` 公開為
  供引用／讀取輔助程式使用的虛擬搜尋命中路徑。耐久的索引來源為
  關聯式（`source_kind='sessions'`、`source_key='session:<sessionId>'`、
  `session_id=<sessionId>`)，因此該值不是執行階段逐字稿定位器，
  也不是檔案系統路徑，且絕不可傳回工作階段執行階段 API。
- 閘道 doctor 記憶體狀態會從 SQLite 外掛狀態資料列讀取短期回憶與階段訊號計數，
  而不是從 `memory/.dreams/*.json` 讀取；命令列介面與
  doctor 輸出現在將該儲存空間標示為 SQLite 儲存區，而非路徑。
- Memory-core 執行階段、命令列介面狀態、閘道 doctor 方法及外掛 SDK
  門面不再稽核或封存舊版 `.dreams/session-corpus` 檔案。
  這些檔案僅作為遷移輸入；doctor 會將其匯入 SQLite，並在驗證後
  刪除來源。作用中工作階段擷取證據資料列
  現在使用虛擬 SQLite 路徑 `memory/session-ingestion/<day>.txt`；執行階段
  絕不會寫入 `.dreams/session-corpus`，也不會從中衍生狀態。
- Memory-core 公開成品將 SQLite 主機事件公開為虛擬 JSON
  成品 `memory/events/memory-host-events.json`；不再重複使用
  舊版 `.dreams/events.jsonl` 來源路徑。
- 沙箱容器／瀏覽器登錄現在使用共用的
  `sandbox_registry_entries` SQLite 資料表，並具有具型別的工作階段、映像、時間戳記、
  後端／設定及瀏覽器連接埠欄位。Doctor 會匯入舊版單體與
  分片 JSON 登錄檔案，並移除成功匯入的來源。執行階段讀取會
  將具型別資料列欄位視為事實來源；`entry_json` 僅為重播／偵錯
  副本。
- 承諾現在使用具型別的共用 `commitments` 資料表，而不是
  整個儲存區的 JSON Blob。執行階段使用具索引的範圍、傳送時間窗、滾動式
  上限、狀態及嘗試查詢，以及同步 SQLite 交易；
  `record_json` 僅為重播／偵錯副本。明確的 doctor 修復會驗證
  完整的舊版 `commitments.json`、保留較新的 SQLite 資料列、驗證
  結果，然後才移除未變更的來源。執行階段絕不讀取或
  寫入已淘汰的檔案。
- Web Push 訂閱與產生的 VAPID 身分現在使用具型別的共用
  `web_push_subscriptions` 與 `web_push_vapid_keys` 資料列。執行階段註冊、
  到期清理及首次使用金鑰產生作業使用資料列層級的 SQLite
  交易。明確的 Doctor 修復會驗證兩個已淘汰的 JSON 儲存區、
  在寫入 SQLite 前宣告其所有權、以不可分割方式匯入、拒絕
  衝突的 VAPID 身分、驗證結果，然後才移除
  宣告。Doctor 在完整匯入期間會持有狀態目錄維護鎖，
  使較舊的閘道無法重新建立已淘汰的檔案。在 Doctor 解決
  待處理的舊版來源或中斷的宣告前，註冊、
  傳送、刪除及金鑰解析都會採取失敗即關閉。
- 排程工作定義、排程狀態及執行歷程不再具有執行階段
  JSON 寫入器或讀取器。執行階段使用 `cron_jobs` 資料列，其中包含具型別的排程、
  承載資料、傳送、失敗警示、工作階段、狀態及執行階段狀態欄位，另加上
  排程擁有的 `task_runs` 詳細資訊，用於診斷、傳送、工作階段／執行、模型
  及 Token 總計。`job_json` 僅為重播／偵錯副本；`state_json` 保留尚無
  熱門查詢欄位的巢狀執行階段診斷，而執行階段會
  從具型別欄位重新補水熱門狀態欄位。Doctor 會匯入
  舊版 `jobs.json`、`jobs-state.json` 及 `runs/*.jsonl` 檔案，並移除
  已匯入的來源。外掛目標回寫會更新相符的 `cron_jobs`
  資料列，而不是載入並取代整個排程儲存區。
- 閘道啟動會忽略執行階段投影中的舊版 `notify: true`
  標記。Doctor 僅在將這些標記轉換為明確的 SQLite 傳送期間，
  讀取已淘汰的原始 `cron.webhook`，之後便移除該設定鍵。
- 輸出與工作階段傳送佇列現在會將佇列狀態、項目種類、
  工作階段鍵、頻道、目標、帳號 ID、重試次數、上次嘗試／錯誤、
  復原狀態及平台傳送標記，儲存為共用
  `delivery_queue_entries` 資料表中的具型別欄位。執行階段復原會從
  具型別欄位讀取這些熱門欄位，而重試／復原變更會直接更新這些欄位，
  無須重寫重播 JSON。完整 JSON 承載資料僅保留為
  訊息本文及其他冷重播資料的重播／偵錯 Blob。
- 受管理的輸出映像記錄現在使用具型別的共用
  `managed_outgoing_image_records` 資料列。執行階段僅讀取具型別欄位；
  JSON 欄位是重播／偵錯副本。原始映像位元組仍以具名
  附件成品形式保留在受管理的媒體目錄中。
- Discord 模型選擇器偏好設定、命令部署雜湊及討論串繫結
  現在使用共用 SQLite 外掛狀態。其舊版 JSON 匯入計畫位於
  Discord 外掛設定／doctor 遷移介面，而非核心遷移程式碼。
- 外掛舊版匯入偵測器使用 doctor 命名的模組，例如
  `doctor-legacy-state.ts` 或 `doctor-state-imports.ts`；一般頻道執行階段
  模組不得匯入舊版 JSON 偵測器。
- BlueBubbles 追趕游標與輸入去重標記現在使用共用 SQLite
  外掛狀態。其舊版 JSON 匯入計畫位於 BlueBubbles 外掛
  設定／doctor 遷移介面，而非核心遷移程式碼。
- Telegram 更新偏移量、貼圖快取資料列、已傳送訊息快取資料列、
  主題名稱快取資料列及討論串繫結現在使用共用 SQLite 外掛
  狀態。其舊版 JSON 匯入計畫位於 Telegram 外掛
  設定／doctor 遷移介面，而非核心遷移程式碼。
- iMessage 追趕游標、回覆短 ID 對應及已傳送回聲去重資料列
  現在使用共用 SQLite 外掛狀態。舊的 `imessage/catchup/*.json`、
  `imessage/reply-cache.jsonl` 及 `imessage/sent-echoes.jsonl` 檔案
  僅作為 doctor 輸入。
- Feishu 訊息去重資料列現在使用核心可宣告去重
  （共用 SQLite 外掛狀態中的 `feishu.dedup.*` 命名空間），而不是
  `feishu/dedup/*.json` 檔案或已淘汰的手工打造 `dedup.*` 儲存區；
  由於重播保護快取會在升級後重建，因此不會匯入舊版資料。
- Microsoft Teams 對話、投票、待處理上傳緩衝區及意見回饋
  學習資料現在使用共用 SQLite 外掛狀態／Blob 資料表。待處理上傳
  路徑使用 `plugin_blob_entries`，讓媒體緩衝區儲存為 SQLite BLOB，
  而不是 base64 JSON。執行階段輔助程式名稱現在使用 SQLite／狀態命名，
  而不是 `*-fs` 檔案儲存區命名，且舊的 `storePath` 相容層已從
  這些儲存區移除。其舊版 JSON 匯入計畫位於 Microsoft Teams
  外掛設定／doctor 遷移介面。
- Zalo 託管的輸出媒體現在使用共用 SQLite `plugin_blob_entries`，
  而不是 `openclaw-zalo-outbound-media` JSON／bin 暫存側車檔案。
- 差異檢視器 HTML 與中繼資料現在使用共用 SQLite `plugin_blob_entries`，
  而不是 `meta.json`/`viewer.html` 暫存檔案。檢視器 HTML 以
  gzip Blob 儲存，且僅持久保存 URL Token 雜湊。算繪出的 PNG／PDF 輸出
  仍維持暫存實體化，因為頻道傳送仍需要檔案路徑；
  其到期中繼資料由 SQLite 管理，不使用 JSON 側車檔案。
- Canvas 受管理文件現在使用共用 SQLite `plugin_blob_entries`，
  而不是預設的 `state/canvas/documents` 目錄。Canvas 主機會直接提供這些
  Blob；只有明確的 `host.root` 操作者內容，或下游媒體讀取器
  需要路徑而進行暫時實體化時，才會建立本機檔案。
- 檔案傳輸稽核決策現在使用共用 SQLite `plugin_state_entries`，
  而不是無界限的 `audit/file-transfer.jsonl` 執行階段記錄。Doctor
  會將舊版 JSONL 稽核檔案匯入外掛狀態，並在完整匯入後移除來源。
- ACPX 程序租約與閘道執行個體身分現在使用共用 SQLite 外掛
  狀態。Doctor 會將舊版 `gateway-instance-id` 檔案匯入外掛狀態，
  並移除來源。
- ACPX 產生的包裝函式指令碼與隔離的 Codex 主目錄，是 OpenClaw
  暫存根目錄下的暫時實體化，而非持久的 OpenClaw 狀態。
  持久的 ACPX 執行階段記錄是 SQLite 租約與閘道執行個體資料列；
  舊的 ACPX `stateDir` 設定介面已移除，因為執行階段狀態
  不再寫入該處。
- 閘道媒體附件現在使用共用 `media_blobs` SQLite 資料表作為
  標準位元組儲存區。傳回頻道與沙箱相容性介面的本機路徑，
  是資料庫資料列的暫存實體化，而非
  持久媒體儲存區。執行階段媒體允許清單不再包含舊版
  `$OPENCLAW_STATE_DIR/media` 或設定目錄 `media` 根目錄；這些目錄
  僅作為 doctor 匯入來源。
- Shell 自動完成不再寫入 `$OPENCLAW_STATE_DIR/completions/*` 快取
  檔案。安裝、doctor、更新及發行煙霧測試路徑使用產生的
  自動完成輸出或設定檔載入，而非持久的自動完成快取
  檔案。
- 閘道 Skills 上傳暫存現在使用共用 `skill_uploads` 與
  `skill_upload_chunks` 資料列。各區塊在上傳期間維持個別交易，
  接著提交會組裝一個已驗證的封存檔 BLOB，並移除區塊
  資料列。安裝程式只會在安裝執行期間收到暫時實體化的封存檔路徑。
  Doctor 會捨棄已淘汰、保留一小時的檔案系統
  暫存樹，而不是匯入暫時性上傳。
- 子代理程式內嵌附件不再實體化於工作區
  `.openclaw/attachments/*` 下。衍生路徑會準備 SQLite VFS 種子項目，
  內嵌執行會將這些項目植入每個代理程式的執行階段暫存命名空間，
  而磁碟型工具會在該 SQLite 暫存空間上疊加附件路徑。
  舊的子代理程式執行附件目錄登錄欄位及清理掛鉤已移除。
- 命令列介面映像補水不再維護穩定的 `openclaw-cli-images` 快取
  檔案。外部命令列介面後端仍會收到檔案路徑，但這些路徑是
  每次執行的暫存實體化，並附帶清理作業。
- 快取追蹤診斷、Anthropic 承載資料診斷、原始模型串流
  診斷、診斷時間軸事件及閘道穩定性套件現在
  寫入 SQLite 資料列，而不是 `logs/*.jsonl` 或
  `logs/stability/*.json` 檔案。
  執行階段路徑覆寫旗標及環境變數已移除；匯出／偵錯
  命令可明確地從資料庫資料列實體化檔案。
- macOS 配套應用程式不再具有滾動式 `diagnostics.jsonl` 寫入器。應用程式
  記錄會送至統一記錄系統，而持久的閘道診斷則由 SQLite 支援。
- macOS 連接埠守護程式記錄清單現在使用具型別的共用 SQLite
  `macos_port_guardian_records` 資料列，而不是 Application Support JSON 檔案
  或不透明的單一 Blob。所有 macOS 應用程式設定檔都使用同一個主機全域原生
  資料庫，因為它們會協調機器本機連接埠。當較舊且會寫入 JSON 的應用程式副本
  正在執行時，每個分類帳作業都會阻塞。遷移僅會加入舊
  分類帳的穩定檔案鎖定通訊協定，以建立快照並稍後重新驗證
  來源。遷移程序會根據即時命令與程序啟動事實解析每個舊版資料列，
  過程中不持有該鎖，接著重新讀取具權威性的 SQLite 資料列、套用
  計畫、驗證每張收據，並移除來源。移除重試會重新規劃
  缺少的資料列，使已淘汰的過期收據無法復活。該鎖維持
  短效，因此在 SSH 已衍生程序後，不會讓較舊的寫入器陷入停滯。切換
  刻意設計為單向：穩態執行階段絕不讀取、投影或寫入 JSON，
  而回復至僅支援 JSON 的組建不會保留較新的 SQLite 收據。
- 閘道單一執行個體鎖現在使用 `gateway_locks` 範圍下具型別的共用 SQLite
  `state_leases` 資料列，而不是暫存目錄鎖定檔案。Fly 與 OAuth
  疑難排解文件現在指向 SQLite 租約／驗證重新整理鎖，
  而不是過時的檔案鎖清理。
- 閘道重新啟動哨兵狀態現在使用具型別的共用 SQLite
  `gateway_restart_sentinel` 資料列，而非 `restart-sentinel.json`；執行階段會從
  具型別的欄位讀取哨兵種類、狀態、路由、訊息、接續資訊與統計資料。
  這些欄位才是權威資料來源；`payload_json` 僅是供重播／偵錯使用的
  影子資料。執行階段的讀取、寫入及清除路徑僅使用 SQLite。
  一個有界的狀態遷移模組會在啟動期間及 Doctor 執行時運作，在正常的重新啟動復原前
  匯入經驗證的舊版更新後哨兵、驗證具型別的資料列，並移除來源檔案。
  穩態執行階段模組皆不會讀取、寫入或清理舊版檔案。
- 閘道重新啟動意圖與監督程式交接狀態現在使用具型別的共用
  SQLite `gateway_restart_intent` 與 `gateway_restart_handoff` 資料列，而非
  `gateway-restart-intent.json` 與
  `gateway-supervisor-restart-handoff.json` 側車檔案。
- 閘道單例協調現在使用 `gateway_locks` 下具型別的
  `state_leases` 資料列，而非寫入 `gateway.<hash>.lock` 檔案。租約資料列
  保存鎖定擁有者、到期時間、心跳偵測與偵錯承載資料；SQLite 負責
  原子性的取得／釋放邊界。已移除停用的檔案鎖定目錄選項；
  測試會直接使用 SQLite 資料列識別資訊。
- 已刪除掃描 `cron/runs/*.jsonl`
  檔案、但未被參照的舊版排程用量報告輔助函式。排程執行歷程報告會讀取排程所擁有的 `task_runs` 資料列。
- 主要工作階段重新啟動復原現在透過 SQLite
  `agent_databases` 登錄檔探索候選代理程式，而非掃描 `agents/*/sessions`
  目錄。
- Gemini 工作階段損毀復原現在只會刪除 SQLite 工作階段資料列；
  不再需要舊版 `storePath` 閘門，也不會嘗試取消連結衍生的
  逐字稿 JSONL 路徑。
- 路徑覆寫處理現在會將字面值為 `undefined`/`null` 的環境
  值視為未設定，避免在測試或 shell 交接期間意外於存放庫根目錄建立 `undefined/state/*.sqlite`
  資料庫。
- 設定健康狀態指紋現在使用具型別的共用 SQLite `config_health_entries`
  資料列，而非 `logs/config-health.json`，使一般設定檔維持為
  唯一不含認證資訊的設定文件。macOS 伴隨應用程式僅保留
  程序區域的健康狀態，不會重新建立舊版 JSON 側車檔案。
- 驗證設定檔執行階段不再匯入或寫入認證資訊 JSON 檔案。
  正式認證資訊儲存區為 SQLite；`auth-profiles.json`、各代理程式的
  `auth.json` 及共用的 `credentials/oauth.json` 是 Doctor 遷移輸入，
  匯入後會予以移除。
- 驗證設定檔儲存／狀態測試現在會直接對具型別的 SQLite 驗證資料表進行判定，
  且只會將舊版驗證設定檔名稱用於 Doctor 遷移輸入。
- `openclaw secrets apply` 僅會清理設定檔、環境變數檔案與 SQLite
  驗證設定檔儲存區。不再包含編輯已停用的各代理程式
  `auth.json` 的相容性邏輯；該檔案的匯入與刪除由 Doctor 負責。
- Hermes 密鑰遷移會規劃並將匯入的 API 金鑰設定檔直接套用至
  SQLite 驗證設定檔儲存區。不再將
  `auth-profiles.json` 寫入或驗證為中繼目標。
- 面向使用者的驗證文件現在說明
  `state/openclaw.sqlite#table/auth_profile_stores/<agentDir>`，不再
  要求使用者檢查或複製 `auth-profiles.json`；舊版 OAuth／驗證 JSON
  名稱只會記載為 Doctor 匯入輸入。
- MCP OAuth 工作階段現在使用共用
  `state/openclaw.sqlite` 中具有版本的 `mcp_oauth_stores` 資料列。SDK 所擁有的權杖、用戶端註冊及探索
  物件仍會保留在一個經驗證的 JSON 承載資料中，讓相依套件的擴充欄位得以保留；
  每次讀取／修改／寫入則會在一個短暫的 Kysely
  交易中提交。一個共用 SQLite 租約會將重新整理、登入與登出序列化；
  內嵌式 MCP 傳輸不再允許 MCP SDK 於該
  租約外重新整理。Doctor 會以來源收據獨佔地匯入並移除已停用的 `mcp-oauth/*.json`
  儲存區，且執行階段沒有檔案後援機制。
- 核心狀態路徑輔助函式不再公開已停用的 `credentials/oauth.json`
  檔案。舊版檔名僅存在於 Doctor 驗證匯入路徑中。
- 安裝、安全性、初始設定、模型驗證與 SecretRef 文件現在說明
  SQLite 驗證設定檔資料列及完整狀態備份／遷移，而非
  各代理程式的驗證設定檔 JSON 檔案。
- PI 模型探索現在會將正式認證資訊傳入記憶體內的
  `pi-coding-agent` 驗證儲存區。探索期間不再建立、清理或寫入
  各代理程式的 `auth.json`。
- 語音喚醒觸發與路由設定現在使用具型別的共用 SQLite 資料表，
  而非 `settings/voicewake.json`、`settings/voicewake-routing.json` 或
  不透明的通用資料列；Doctor 會匯入舊版 JSON 檔案，並在遷移成功後移除這些檔案。
- 更新檢查狀態現在使用具型別的共用 `update_check_state` 資料列，而非
  `update-check.json` 或不透明的通用 Blob；Doctor 會匯入
  舊版 JSON 檔案，並在遷移成功後移除該檔案。
- 設定健康狀態現在使用具型別的共用 `config_health_entries` 資料列，而非
  `logs/config-health.json` 或不透明的通用 Blob；Doctor
  會匯入舊版 JSON 檔案，並在遷移成功後移除該檔案。
- 外掛對話繫結核准現在使用具型別的
  `plugin_binding_approvals` 資料列，而非不透明的共用 SQLite 狀態或
  `plugin-binding-approvals.json`；舊版檔案是 Doctor 遷移輸入。
- 通用目前對話繫結現在會儲存具型別的
  `current_conversation_bindings` 資料列，而非重寫
  `bindings/current-conversations.json`；Doctor 會匯入舊版 JSON 檔案，並
  在遷移成功後移除該檔案。
- Memory Wiki 匯入來源同步帳本現在會針對每個保存庫／來源金鑰儲存一筆 SQLite 外掛狀態資料列，
  而非重寫 `.openclaw-wiki/source-sync.json`；
  遷移提供者會匯入並移除舊版 JSON 帳本。
- Memory Wiki ChatGPT 匯入執行記錄現在會針對每個保存庫／執行 ID 儲存一筆 SQLite 外掛狀態資料列，
  而非寫入 `.openclaw-wiki/import-runs/*.json`。
  復原快照會繼續保留為明確的保存庫檔案，直到匯入執行快照的
  封存移至 Blob 儲存區為止。
- Memory Wiki 編譯摘要現在會儲存壓縮的 SQLite 外掛 Blob 資料列，
  而非寫入 `.openclaw-wiki/cache/agent-digest.json` 與
  `.openclaw-wiki/cache/claims.jsonl`。快取可重新建置，因此 Doctor
  會刪除舊快取檔案而不匯入。
- ClawHub Skill 安裝追蹤現在會針對每個工作區／Skill 儲存一筆 SQLite 外掛狀態資料列，
  而非在執行階段寫入或讀取 `.clawhub/lock.json` 與
  `.clawhub/origin.json` 側車檔案。執行階段程式碼會使用受追蹤的安裝
  狀態物件，而非檔案形式的鎖定檔／來源抽象。Doctor
  會從已設定的代理程式工作區匯入舊版側車檔案，並在
  完整匯入後移除這些檔案。
- 已安裝的外掛索引現在會讀寫具型別的共用 SQLite
  `installed_plugin_index` 單例資料列，而非 `plugins/installs.json`；
  舊版 JSON 檔案僅作為 Doctor 遷移輸入，並會在匯入後移除。
- 舊版 `plugins/installs.json` 路徑輔助函式現在位於 Doctor 舊版
  程式碼中。執行階段外掛索引模組僅公開由 SQLite 支援的持久化
  選項，而非 JSON 檔案路徑。
- 閘道重新啟動哨兵、重新啟動意圖與監督程式交接狀態現在使用
  具型別的共用 SQLite 資料列（`gateway_restart_sentinel`、
  `gateway_restart_intent` 與 `gateway_restart_handoff`），而非通用的
  不透明 Blob。執行階段重新啟動程式碼不再具有檔案形式的哨兵／意圖／交接
  合約。
- Matrix 同步快取、儲存中繼資料、討論串繫結、輸入去重標記、
  啟動驗證冷卻狀態、SDK IndexedDB 加密快照、
  認證資訊與復原金鑰現在使用共用 SQLite 外掛狀態／Blob
  資料表。執行階段路徑結構不再公開 `storage-meta.json` 中繼資料
  路徑；該檔名僅作為舊版遷移輸入。其舊版 JSON 匯入
  計畫位於 Matrix 外掛設定／Doctor 遷移介面中。輸入
  去重標記會透過核心可宣告去重機制（共用狀態資料庫中的 `matrix.inbound-dedupe.*`
  命名空間）運作；Matrix Doctor 狀態遷移會一次性匯入
  已停用的各根目錄 `inbound-dedupe` 資料列與 `inbound-dedupe.json`，
  之後執行階段僅讀取可宣告去重儲存區。
- Matrix 啟動程序不再掃描、報告或完成舊版 Matrix 檔案
  狀態。Matrix 檔案偵測、舊版加密快照建立、房間金鑰
  還原遷移狀態、匯入及來源移除全都由 Doctor 負責。
- Matrix 執行階段遷移彙整模組已移除。舊版狀態／加密偵測
  及異動輔助函式會由 Matrix Doctor 直接匯入，不再
  屬於執行階段 API 介面的一部分。
- Matrix 遷移快照重用標記現在位於 SQLite 外掛狀態中，
  而非 `matrix/migration-snapshot.json`；Doctor 仍可重用相同的
  已驗證遷移前封存，而不必寫入側車狀態檔案。
- Nostr 匯流排游標與設定檔發布狀態現在使用共用 SQLite 外掛
  狀態。其舊版 JSON 匯入計畫位於 Nostr 外掛設定／Doctor
  遷移介面中。
- 主動記憶工作階段切換現在使用共用 SQLite 外掛狀態，而非
  `session-toggles.json`；重新開啟記憶功能時會刪除該資料列，而非
  重寫 JSON 物件。
- Skill Workshop 提案與審查計數器現在使用共用 SQLite 外掛
  狀態，而非各工作區的 `skill-workshop/<workspace>.json` 儲存區。每項
  提案都是 `skill-workshop/proposals` 下的獨立資料列，而審查
  計數器則是 `skill-workshop/reviews` 下的獨立資料列。
- Skill Workshop 審查者子代理程式執行現在使用執行階段工作階段逐字稿
  解析器，而非建立 `skill-workshop/<sessionId>.json` 側車工作階段
  路徑。
- ACPX 程序租約現在使用 `acpx/process-leases` 下的共用 SQLite 外掛狀態，
  而非整檔式 `process-leases.json` 登錄檔。
  每個租約都以獨立資料列儲存，在無需執行階段 JSON 重寫路徑的情況下，
  保留啟動時清除過期程序的能力。
- ACPX 包裝函式指令碼與隔離的 Codex 主目錄會在
  OpenClaw 暫存根目錄中產生。系統會視需要重新建立它們，且它們不屬於備份或
  遷移輸入。
- 子代理程式執行登錄檔持久化會使用具型別的共用 `subagent_runs` 資料列。
  舊版 `subagents/runs.json` 路徑現在僅作為 Doctor 清理輸入。Doctor
  會在狀態維護鎖下宣告該路徑、將捨棄決策記錄於
  SQLite，並在不匯入暫時性執行狀態的情況下移除該路徑。不再保留任何執行階段 JSON
  讀取器、寫入器、快取或後援機制；在此停用邊界上，刻意不支援
  僅存在於檔案中的進行中執行跨版本復原。
  執行階段測試不再建立無效或空白的 `runs.json` 固定資料來驗證
  登錄檔行為；而是直接植入／讀取 SQLite 資料列。
- 備份會先暫存狀態目錄再建立封存、複製非資料庫檔案、
  使用線上備份加上離線 `VACUUM` 建立資料庫快照、省略即時 WAL／SHM 側車檔案、在封存資訊清單中記錄
  快照中繼資料，並將
  已完成的備份執行連同封存資訊清單記錄於 SQLite。`openclaw backup
create` 預設會驗證已寫入的封存；`--no-verify` 是
  明確的快速路徑。
- `openclaw backup restore` 會在解壓縮前驗證封存、重用
  驗證器的正規化資訊清單，並將經驗證的資訊清單資產還原至其
  記錄的來源路徑。寫入時必須使用 `--yes`，並支援使用 `--dry-run`
  產生還原計畫。
- 舊版備份揮發性路徑篩選器已刪除。由於 SQLite
  快照會在建立封存前完成暫存，備份不再需要針對舊版工作階段或排程 JSON／JSONL 檔案使用
  即時 tar 略過清單。
- 一般設定與新手引導的工作區準備不再建立
  `agents/<agentId>/sessions/` 目錄。它們只建立設定／工作區；
  SQLite 工作階段資料列與逐字稿資料列會依需求建立於
  每個代理程式的資料庫中。
- 安全性權限修復現在以全域及每個代理程式的 SQLite
  資料庫與 WAL/SHM 附屬檔案為目標，而非 `sessions.json` 與逐字稿
  JSONL 檔案。
- 沙箱登錄的執行階段名稱現在直接描述 SQLite 登錄種類，
  不再將舊版 JSON 登錄術語沿用至作用中的儲存區。
- `openclaw reset --scope config+creds+sessions` 會移除每個代理程式的
  `openclaw-agent.sqlite` 資料庫與 WAL/SHM 附屬檔案，而不僅是舊版
  `sessions/` 目錄。
- 閘道彙總工作階段輔助函式現在使用以項目為導向的名稱：
  `loadCombinedSessionEntriesForGateway` 會傳回 `{ databasePath, entries }`。
  舊有的合併儲存區命名已從執行階段呼叫端移除。
- Docker MCP 頻道的種子資料建立現在會將主要工作階段資料列與逐字稿
  事件寫入每個代理程式的 SQLite 資料庫，而非建立
  `sessions.json` 與 JSONL 逐字稿。
- 內建的工作階段記憶掛鉤現在會依 `{agentId, sessionId}` 從
  SQLite 解析上一個工作階段的情境。它不再掃描、儲存或合成
  逐字稿路徑或 `workspace/sessions` 目錄。
- 內建的命令記錄器掛鉤現在會將命令稽核資料列寫入共用的
  SQLite `command_log_entries` 資料表，而非附加至
  `logs/commands.log`。
- 頻道配對允許清單現在於執行階段只公開由 SQLite 支援的讀取／寫入輔助函式。
  已棄用的外掛 SDK 路徑解析器仍保留以供遷移相容；
  檔案讀取器僅存在於 doctor 狀態遷移程式碼中。
- `migration_runs` 會記錄舊版狀態遷移的執行狀態、
  時間戳記與 JSON 報告。
- `migration_sources` 會記錄每個已匯入舊版檔案來源的雜湊、大小、
  記錄數、目標資料表、執行 ID、狀態與來源移除狀態。
- `backup_runs` 會記錄備份封存檔路徑、狀態與 JSON 資訊清單。
- 全域綱要不會保留未使用的 `agents` 登錄資料表。在執行階段
  擁有實際的代理程式記錄擁有者之前，代理程式資料庫探索是標準的
  `agent_databases` 登錄。
- 產生的模型目錄設定會儲存在具型別的全域 SQLite
  `agent_model_catalogs` 資料列中，並以代理程式目錄作為索引鍵。執行階段呼叫端使用
  `ensureOpenClawModelCatalog`；執行階段程式碼中不存在 `models.json` 相容性 API。
  實作會寫入 SQLite，並從該儲存的承載資料載入內嵌 PI 登錄，
  而不建立 `models.json` 檔案。
- 選用的 `memory.qmd.sessions` 匯出會從
  每個代理程式的資料庫讀取標準逐字稿資料列，並在 QMD 主目錄下具體化經過清理的 Markdown，
  作為明確的 QMD 輸入成品。因此，QMD 工作階段集合與成品
  身分對應仍屬於已設定的外部工具橋接；
  它們不是第二個標準逐字稿儲存區。
- QMD 自有的 `index.sqlite`、YAML 集合設定與模型下載仍是
  `~/.openclaw/agents/<agentId>/qmd` 下的外部工具成品；它們不會
  鏡像至 `plugin_blob_entries`。由 OpenClaw 擁有的 QMD 協調採資料庫優先：
  共用的 `state_leases` 會全域序列化嵌入作業，而每個代理程式的
  `state_leases` 會序列化集合／更新／嵌入寫入器。執行階段不會建立
  QMD 鎖定附屬檔案。
- 選用的 `memory-lancedb` 外掛不再建立
  `~/.openclaw/memory/lancedb` 作為由 OpenClaw 隱式管理的儲存區。它是
  外部 LanceDB 後端，並會維持停用，直到操作員設定明確的
  `dbPath`。
- `check:database-first-legacy-stores` 會讓將
  舊版儲存區名稱與寫入型檔案系統 API 配對的新執行階段來源失敗。它也會讓重新引入已淘汰逐字稿橋接標記
  `transcriptLocator` 或 `sqlite-transcript://...` 的執行階段
  來源失敗。遷移、doctor、匯入及明確的非工作階段匯出程式碼仍獲允許。更廣泛的舊版契約
  名稱，例如 `sessionFile`、`storePath` 與舊有的 `SessionManager` 檔案時代
  門面仍有目前的擁有者，且需要另行進行遷移防護工作，
  才能成為必要的預檢。此防護現在也涵蓋
  執行階段 `cache/*.json` 儲存區、通用
  `thread-bindings.json` 附屬檔案、排程狀態／執行記錄 JSON、設定健康狀態 JSON、
  重新啟動與鎖定附屬檔案、語音喚醒設定、外掛繫結核准、
  已安裝外掛索引 JSON、檔案傳輸稽核 JSONL、記憶 Wiki 活動
  記錄、舊有內建 `command-logger` 文字記錄，以及 pi-mono 原始串流 JSONL
  診斷選項。它也會禁止舊有根層級 doctor 舊版模組名稱，
  使相容性程式碼維持在 `src/commands/doctor/` 下。Android 偵錯處理常式
  也會使用 logcat／記憶體內輸出，而非暫存 `camera_debug.log` 或
  `debug_logs.txt` 快取檔案。

## 目標結構描述形態

保持結構描述明確。主機擁有的執行階段狀態使用具型別的資料表。外掛擁有的
不透明狀態使用 `plugin_state_entries` / `plugin_blob_entries`；不存在
通用的主機 `kv` 資料表。

全域資料庫：

```text
state_leases(scope, lease_key, owner, expires_at, heartbeat_at, payload_json, created_at, updated_at)
exec_approvals_config(config_key, raw_json, socket_path, has_socket_token, default_security, default_ask, default_ask_fallback, auto_allow_skills, agent_count, allowlist_count, updated_at_ms)
schema_meta(meta_key, role, schema_version, agent_id, app_version, created_at, updated_at)
agent_databases(agent_id, path, schema_version, last_seen_at, size_bytes)
task_runs(...)
task_delivery_state(...)
flow_runs(...)
subagent_runs(run_id, child_session_key, requester_session_key, controller_session_key, created_at, ended_at, cleanup_handled, payload_json)
current_conversation_bindings(binding_key, binding_id, target_agent_id, target_session_id, target_session_key, channel, account_id, conversation_kind, parent_conversation_id, conversation_id, target_kind, status, bound_at, expires_at, metadata_json, updated_at)
plugin_binding_approvals(plugin_root, channel, account_id, plugin_id, plugin_name, approved_at)
tui_last_sessions(scope_key, session_key, updated_at)
plugin_state_entries(plugin_id, namespace, entry_key, value_json, created_at, expires_at)
plugin_blob_entries(plugin_id, namespace, entry_key, metadata_json, blob, created_at, expires_at)
media_blobs(subdir, id, content_type, size_bytes, blob, created_at, updated_at)
skill_uploads(upload_id, kind, slug, force, size_bytes, sha256, actual_sha256, received_bytes, archive_blob, created_at, expires_at, committed, committed_at, idempotency_key_hash)
skill_upload_chunks(upload_id, byte_offset, size_bytes, chunk_blob)
web_push_subscriptions(endpoint_hash, subscription_id, endpoint, p256dh, auth, created_at_ms, updated_at_ms)
web_push_vapid_keys(key_id, public_key, private_key, subject, updated_at_ms)
apns_registrations(node_id, transport, token, relay_handle, send_grant, installation_id, relay_origin, topic, environment, distribution, token_debug_suffix, updated_at_ms)
apns_registration_tombstones(node_id, deleted_at_ms)
node_host_config(config_key, version, node_id, token, display_name, gateway_host, gateway_port, gateway_tls, gateway_tls_fingerprint, gateway_context_path, updated_at_ms)
device_identities(identity_key, device_id, public_key_pem, private_key_pem, created_at_ms, updated_at_ms)
device_auth_tokens(device_id, role, token, scopes_json, updated_at_ms)
macos_port_guardian_records(pid, port, command, mode, timestamp)
workspace_setup_state(workspace_key, workspace_path, version, bootstrap_seeded_at, setup_completed_at, updated_at)
workspace_path_aliases(alias_key, alias_path, workspace_key, workspace_path, updated_at_ms)
workspace_attestations(workspace_key, attested_at_ms, updated_at_ms)
workspace_generated_bootstrap_hashes(workspace_key, filename, sha256)
native_hook_relay_bridges(relay_id, pid, hostname, port, token, expires_at_ms, updated_at_ms)
model_capability_cache(provider_id, model_id, name, input_text, input_image, reasoning, supports_tools, context_window, max_tokens, cost_input, cost_output, cost_cache_read, cost_cache_write, updated_at_ms)
agent_model_catalogs(catalog_key, agent_dir, raw_json, updated_at)
managed_outgoing_image_records(attachment_id, session_key, agent_id, message_id, created_at, updated_at, retention_class, alt, original_media_id, original_media_subdir, original_content_type, original_width, original_height, original_size_bytes, original_filename, record_json, cleanup_pending)
gateway_restart_sentinel(sentinel_key, version, kind, status, ts, session_key, thread_id, delivery_channel, delivery_to, delivery_account_id, message, continuation_json, doctor_hint, stats_json, payload_json, updated_at_ms)
channel_pairing_requests(channel_key, account_id, request_id, code, created_at, last_seen_at, meta_json)
channel_pairing_allow_entries(channel_key, account_id, entry, sort_order, updated_at)
voicewake_triggers(config_key, position, trigger, updated_at_ms)
voicewake_routing_config(config_key, version, default_target_mode, default_target_agent_id, default_target_session_key, updated_at_ms)
voicewake_routing_routes(config_key, position, trigger, target_mode, target_agent_id, target_session_key, updated_at_ms)
update_check_state(state_key, last_checked_at, last_notified_version, last_notified_tag, last_available_version, last_available_tag, auto_install_id, auto_first_seen_version, auto_first_seen_tag, auto_first_seen_at, auto_last_attempt_version, auto_last_attempt_at, auto_last_success_version, auto_last_success_at, updated_at_ms)
config_health_entries(config_path, last_known_good_json, last_promoted_good_json, last_observed_suspicious_signature, updated_at_ms)
sandbox_registry_entries(registry_kind, container_name, session_key, backend_id, runtime_label, image, created_at_ms, last_used_at_ms, config_label_kind, config_hash, cdp_port, no_vnc_port, entry_json, updated_at)
cron_jobs(store_key, job_id, name, description, enabled, delete_after_run, created_at_ms, agent_id, session_key, schedule_kind, schedule_expr, schedule_tz, every_ms, anchor_ms, at, stagger_ms, session_target, wake_mode, payload_kind, payload_message, payload_model, payload_fallbacks_json, payload_thinking, payload_timeout_seconds, payload_allow_unsafe_external_content, payload_external_content_source_json, payload_light_context, payload_tools_allow_json, delivery_mode, delivery_channel, delivery_to, delivery_thread_id, delivery_account_id, delivery_best_effort, failure_delivery_mode, failure_delivery_channel, failure_delivery_to, failure_delivery_account_id, failure_alert_disabled, failure_alert_after, failure_alert_channel, failure_alert_to, failure_alert_cooldown_ms, failure_alert_include_skipped, failure_alert_mode, failure_alert_account_id, next_run_at_ms, running_at_ms, last_run_at_ms, last_run_status, last_error, last_duration_ms, consecutive_errors, consecutive_skipped, schedule_error_count, last_delivery_status, last_delivery_error, last_delivered, last_failure_alert_at_ms, job_json, state_json, runtime_updated_at_ms, schedule_identity, sort_order, updated_at)
delivery_queue_entries(queue_name, id, status, entry_kind, session_key, channel, target, account_id, retry_count, last_attempt_at, last_error, recovery_state, platform_send_started_at, entry_json, enqueued_at, updated_at, failed_at)
commitments(id, agent_id, session_key, channel, account_id, recipient_id, thread_id, sender_id, kind, sensitivity, source, status, reason, suggested_text, dedupe_key, confidence, due_earliest_ms, due_latest_ms, due_timezone, source_message_id, source_run_id, created_at_ms, updated_at_ms, attempts, last_attempt_at_ms, sent_at_ms, dismissed_at_ms, snoozed_until_ms, expired_at_ms, record_json)
migration_runs(id, started_at, finished_at, status, report_json)
migration_sources(source_key, migration_kind, source_path, target_table, source_sha256, source_size_bytes, source_record_count, last_run_id, status, imported_at, removed_source, report_json)
backup_runs(id, created_at, archive_path, status, manifest_json)
```

代理程式資料庫：

```text
schema_meta(meta_key, role, schema_version, agent_id, app_version, created_at, updated_at)
sessions(session_id, session_key, session_scope, created_at, updated_at, started_at, ended_at, status, chat_type, channel, account_id, primary_conversation_id, model_provider, model, agent_harness_id, parent_session_key, spawned_by, display_name)
conversations(conversation_id, channel, account_id, kind, peer_id, parent_conversation_id, thread_id, native_channel_id, native_direct_user_id, label, metadata_json, created_at, updated_at)
session_conversations(session_id, conversation_id, role, first_seen_at, last_seen_at)
session_routes(session_key, session_id, updated_at)
session_entries(session_id, session_key, entry_json, updated_at)
transcript_events(session_id, seq, event_json, created_at)
transcript_event_identities(session_id, event_id, seq, event_type, has_parent, parent_id, message_idempotency_key, created_at)
transcript_snapshots(session_id, snapshot_id, reason, event_count, created_at, metadata_json)
vfs_entries(namespace, path, kind, content_blob, metadata_json, updated_at)
tool_artifacts(run_id, artifact_id, kind, metadata_json, blob, created_at)
run_artifacts(run_id, path, kind, metadata_json, blob, created_at)
trajectory_runtime_events(session_id, run_id, seq, event_json, created_at)
memory_index_meta(key, value)
memory_index_sources(id, path, source, hash, mtime, size)
memory_index_chunks(id, path, source, start_line, end_line, hash, model, text, embedding, updated_at)
memory_embedding_cache(provider, model, provider_key, hash, embedding, dims, updated_at)
memory_index_state(id, revision)
cache_entries(scope, key, value_json, blob, expires_at, updated_at)
```

`memory_index_sources.id` 是穩定的整數主鍵；`(path, source)` 仍具唯一性。

未來的搜尋功能可新增 FTS 資料表，而無須變更標準事件資料表：

```text
transcript_events_fts(session_id, seq, text)
vfs_entries_fts(namespace, path, text)
```

大型值應使用 `blob` 欄位，而非 JSON 字串編碼。保留
`value_json` 給必須能以一般 SQLite 工具檢查的小型結構化資料。

`agent_databases` 是此分支的標準登錄檔。在真正的代理程式記錄擁有者出現前，不要新增
`agents` 資料表；代理程式設定仍保留在
`openclaw.json` 中。

## Doctor 遷移形態

Doctor 應呼叫一個可產生報告且能安全重複執行的明確遷移步驟：

```bash
openclaw doctor --fix
```

`openclaw doctor --fix` 會在一般設定預先檢查後叫用狀態遷移實作，並在匯入前建立經驗證的備份。執行階段啟動和 `openclaw migrate` 不得匯入舊版 OpenClaw 狀態檔案。

遷移屬性：

- 單次遷移程序會探索所有舊版檔案來源，並在變更任何內容前產生計畫。
- Doctor 會在匯入舊版檔案前建立經驗證的遷移前備份封存檔。
- 匯入具等冪性，並以來源路徑、mtime、大小、雜湊和目標資料表為鍵。
- 目標資料庫提交後，成功處理的來源檔案會被移除或封存。
- 失敗的匯入會保留來源不變，並在 `migration_runs` 中記錄警告。
- 遷移存在後，執行階段程式碼只會讀取 SQLite。
- 不需要降級／匯出至執行階段檔案的路徑。

## 遷移清單

將以下項目移入全域資料庫：

- 任務登錄檔執行階段寫入現在使用共享資料庫；未發布的
  `tasks/runs.sqlite` 側載匯入器已刪除。快照儲存會依任務
  ID 執行 upsert，且僅刪除缺少的任務／傳遞資料列。
- Task Flow 執行階段寫入現在使用共享資料庫；未發布的
  `tasks/flows/registry.sqlite` 側載匯入器已刪除。快照儲存會
  依流程 ID 執行 upsert，且僅刪除缺少的流程資料列。
- 外掛狀態執行階段寫入現在使用共享資料庫；未發布的
  `plugin-state/state.sqlite` 側載匯入器已刪除。
- 內建記憶搜尋不再預設使用 `memory/<agentId>.sqlite`；其
  索引資料表位於所屬代理程式的資料庫中，而明確選用
  `memorySearch.store.path` 側載的設定已退役，改由 doctor 設定
  遷移處理。
- 內建記憶重新建立索引時，只會重設代理程式資料庫中由記憶功能擁有的資料表。
  它不得取代整個 SQLite 檔案，因為同一個資料庫也包含
  工作階段、逐字稿、VFS 資料列、成品和執行階段快取。
- 從單體和分片 JSON 遷移沙箱容器／瀏覽器登錄檔。執行階段
  寫入現在使用共享資料庫；仍保留舊版 JSON 匯入。
- 排程工作定義、排程狀態及執行歷程現在使用共享 SQLite；
  doctor 會匯入／移除舊版 `jobs.json`、`jobs-state.json` 和
  `cron/runs/*.jsonl` 檔案
- 裝置身分／驗證、推播、更新檢查、承諾、OpenRouter 模型
  快取、已安裝外掛索引及應用程式伺服器繫結
- 裝置／節點配對及啟動程序記錄現在使用具型別的 SQLite 資料表
- 裝置配對通知訂閱者和已傳遞要求標記現在使用
  共享 SQLite 外掛狀態資料表，而非 `device-pair-notify.json`。
- 語音通話記錄現在使用共享 SQLite 外掛狀態資料表中的
  `voice-call`／`calls` 命名空間，而非 `calls.jsonl`；外掛命令列介面
  會追蹤並彙整由 SQLite 支援的通話歷程。
- QQ Bot 閘道工作階段、已知使用者記錄及參照索引引述快取現在使用
  `qqbot` 命名空間（`gateway-sessions`、
  `known-users`、`ref-index`）下的 SQLite 外掛狀態，而非 `session-*.json`、`known-users.json`
  和 `ref-index.jsonl`。這些舊版檔案是快取，不會遷移。
- Discord 模型選擇器偏好設定、指令部署雜湊及討論串繫結
  現在使用 `discord` 命名空間
  （`model-picker-preferences`、`command-deploy-hashes`、`thread-bindings`）
  下的 SQLite 外掛狀態，而非 `model-picker-preferences.json`、`command-deploy-cache.json` 和
  `thread-bindings.json`；Discord doctor／設定遷移會匯入並
  移除舊版檔案。
- BlueBubbles 追趕游標及入站去重標記現在使用
  `bluebubbles` 命名空間（`catchup-cursors`、`inbound-dedupe`）
  下的 SQLite 外掛狀態，而非 `bluebubbles/catchup/*.json` 和
  `bluebubbles/inbound-dedupe/*.json`；BlueBubbles doctor／設定遷移
  會匯入並移除舊版檔案。
- Telegram 更新偏移量、貼圖快取項目、回覆鏈訊息快取
  項目、已傳送訊息快取項目、主題名稱快取項目及討論串
  繫結現在使用 `telegram` 命名空間
  （`update-offsets`、`sticker-cache`、`message-cache`、`sent-messages`、
  `topic-names`、`thread-bindings`）下的 SQLite 外掛狀態，而非 `update-offset-*.json`、
  `sticker-cache.json`、`*.telegram-messages.json`、
  `*.telegram-sent-messages.json`、`*.telegram-topic-names.json` 和
  `thread-bindings-*.json`；Telegram doctor／設定遷移會匯入並
  移除舊版檔案。
- iMessage 追趕游標、回覆短 ID 對應及已傳送回音去重資料列
  現在使用 `imessage` 命名空間（`catchup-cursors`、
  `reply-cache`、`sent-echoes`）下的 SQLite 外掛狀態，而非 `imessage/catchup/*.json`、
  `imessage/reply-cache.jsonl` 和 `imessage/sent-echoes.jsonl`；iMessage
  doctor／設定遷移會匯入並移除舊版檔案。
- Microsoft Teams 對話、投票、SSO 權杖及意見回饋學習成果現在
  使用 SQLite 外掛狀態命名空間（`conversations`、`polls`、`sso-tokens`、
  `feedback-learnings`），而非 `msteams-conversations.json`、
  `msteams-polls.json`、`msteams-sso-tokens.json` 和 `*.learnings.json`；Microsoft Teams
  doctor／設定遷移會匯入並封存舊版檔案。
  待處理上傳是短期 SQLite 快取，舊有 JSON 快取檔案
  不會遷移。
- Matrix 同步快取、儲存中繼資料、討論串繫結、入站去重標記、
  啟動驗證冷卻狀態、認證資訊、復原金鑰及 SDK
  IndexedDB 加密快照現在使用
  `matrix` 下的 SQLite 外掛狀態／Blob 命名空間（`sync-store`、`storage-meta`、`thread-bindings`、
  透過核心可宣告去重機制的 `matrix.inbound-dedupe.*`、
  `startup-verification`、`credentials`、`recovery-key`、`idb-snapshots`），
  而非 `bot-storage.json`、`storage-meta.json`、`thread-bindings.json`、
  `inbound-dedupe.json`、`startup-verification.json`、`credentials.json`、
  `recovery-key.json` 和 `crypto-idb-snapshot.json`；Matrix doctor／設定
  遷移會從帳號範圍的 Matrix 儲存根目錄匯入並移除這些舊版檔案
  （以及已退役的各根目錄 `inbound-dedupe` SQLite 資料列）。
- Nostr 匯流排游標及設定檔發布狀態現在使用
  `nostr` 命名空間（`bus-state`、`profile-state`）下的 SQLite 外掛狀態，而非
  `bus-state-*.json` 和 `profile-state-*.json`；Nostr doctor／設定
  遷移會匯入並移除舊版檔案。
- 主動記憶工作階段切換項現在使用
  `active-memory/session-toggles` 下的 SQLite 外掛狀態，而非 `session-toggles.json`。
- Skill Workshop 提案佇列及審查計數器現在使用
  `skill-workshop/proposals` 和 `skill-workshop/reviews` 下的 SQLite 外掛狀態，而非
  各工作區的 `skill-workshop/<workspace>.json` 檔案。
- 出站傳遞與工作階段傳遞佇列現在共用全域 SQLite
  `delivery_queue_entries` 資料表中的不同佇列名稱
  （`outbound-delivery`、`session-delivery`），而非持久化的
  `delivery-queue/*.json`、`delivery-queue/failed/*.json` 和
  `session-delivery-queue/*.json` 檔案。doctor 舊版狀態步驟會匯入
  待處理及失敗資料列、移除過時的已傳遞標記，並在匯入後刪除舊有
  JSON 檔案。高頻路由和重試欄位是具型別的資料欄；僅為重播／偵錯
  保留 JSON 承載資料。
- ACPX 程序租約現在使用 `acpx/process-leases` 下的 SQLite 外掛狀態，
  而非 `process-leases.json`。
- 備份及遷移執行中繼資料

將以下項目移至代理程式資料庫：

- 代理程式工作階段根目錄及相容形狀的工作階段項目承載資料。執行階段寫入已完成：
  高頻工作階段中繼資料可在 `sessions` 中查詢，而
  舊版形狀的完整 `SessionEntry` 承載資料仍位於 `session_entries`。
- 代理程式逐字稿事件。執行階段寫入已完成。
- 壓縮檢查點及逐字稿快照。執行階段寫入已完成：
  檢查點逐字稿副本是 SQLite 逐字稿資料列，檢查點
  中繼資料則記錄在 `transcript_snapshots` 中。閘道檢查點輔助程式
  現在將這些值稱為逐字稿快照，而非來源檔案。
- 代理程式 VFS 暫存／工作區命名空間。執行階段 VFS 寫入已完成。
- 子代理程式附件承載資料。執行階段寫入已完成：它們是 SQLite VFS
  種子項目，絕不會成為持久化工作區檔案。
- 工具成品。執行階段寫入已完成。
- 執行成品。透過各代理程式的
  `run_artifacts` 資料表進行的工作程式執行階段寫入已完成。
- 代理程式本機執行階段快取。透過各代理程式
  `cache_entries` 資料表進行的工作程式執行階段範圍快取寫入已完成。閘道層級的模型快取仍位於
  全域資料庫中，除非它們轉為代理程式專用。
- ACP 父串流記錄。執行階段寫入已完成。
- ACP 重播分類帳工作階段。透過
  `acp_replay_sessions` 和 `acp_replay_events` 進行的執行階段寫入已完成；舊版 `acp/event-ledger.json`
  僅保留作為 doctor 輸入。
- ACP 工作階段中繼資料。透過 `acp_sessions` 進行的執行階段寫入已完成；`sessions.json` 中的舊版
  `entry.acp` 區塊僅作為 doctor 遷移輸入。
- 非明確匯出檔案的軌跡側載檔。執行階段寫入已完成：
  軌跡擷取會寫入代理程式資料庫 `trajectory_runtime_events`
  資料列，並將執行範圍的成品鏡像至 SQLite。舊版側載檔僅作為 doctor
  匯入輸入；匯出可以具體化新的 JSONL 支援套件輸出，
  但不會在執行階段讀取或遷移舊有軌跡／逐字稿側載檔。
  執行階段軌跡擷取會公開 SQLite 範圍；JSONL 路徑輔助程式
  僅限匯出／偵錯支援使用，不會從執行階段模組重新匯出。
  內嵌執行器軌跡中繼資料會記錄 `{agentId, sessionId, sessionKey}`
  身分，而非持久保存逐字稿定位器。

以下項目目前仍以檔案為後端：

- `openclaw.json`
- 提供者或命令列介面認證資訊檔案
- 外掛／套件資訊清單
- 選取磁碟模式時的使用者工作區及 Git 儲存庫
- 供操作人員追蹤的記錄，除非特定記錄介面已遷移

## 遷移計畫

### 階段 0：凍結邊界

在移動更多資料列之前，先明確定義持久狀態邊界：

- 在全域資料庫中新增 `migration_runs` 資料表。
  舊版狀態遷移執行報告已完成。
- 新增單一由 doctor 擁有的狀態遷移服務，用於從檔案匯入資料庫。
  已完成：`openclaw doctor --fix` 使用舊版狀態遷移實作。
- 將 `plan` 設為唯讀，並讓 `apply` 建立備份、匯入、驗證，然後
  刪除或隔離舊檔案。
  已完成：doctor 會建立經驗證的遷移前備份，將備份路徑
  傳入 `migration_runs`，並重複使用匯入器／移除路徑。
- 新增靜態禁止規則，防止新的執行階段程式碼寫入舊版狀態檔案，同時
  仍允許遷移程式碼及測試植入／讀取它們。
  目前已遷移的舊版儲存區已完成；此防護也會掃描巢狀
  測試，尋找被禁止的執行階段逐字稿定位器合約。

### 階段 1：完成全域控制平面

將共享協調狀態保留在 `state/openclaw.sqlite`：

- 代理程式及代理程式資料庫登錄檔
- 任務與 Task Flow 分類帳
- 外掛狀態
- 沙箱容器／瀏覽器登錄檔
- 排程／排程器執行歷程
- 配對、裝置、推播、更新檢查、終端介面、OpenRouter／模型快取及其他
  小型閘道範圍執行階段狀態
- 備份及遷移中繼資料
- 閘道媒體附件位元組。執行階段寫入已完成；直接檔案路徑
  是為了與頻道傳送端及沙箱暫存相容而產生的暫時具體化檔案。
  執行階段允許清單接受 SQLite 具體化路徑，而非舊版
  狀態／設定媒體根目錄。doctor 會將舊版媒體檔案匯入
  `media_blobs`，並在成功寫入資料列後移除來源檔案。
- 偵錯 Proxy 擷取工作階段、事件及承載資料 Blob。已完成：擷取資料位於
  共享狀態資料庫，並透過共享狀態資料庫的啟動程序、結構描述、
  WAL 及忙碌逾時設定開啟。承載資料位元組在
  `capture_blobs.data` 中使用 gzip 壓縮；沒有偵錯 Proxy 執行階段側載資料庫覆寫、
  Blob 目錄，或僅供 Proxy 擷取使用的產生式結構描述／程式碼產生目標。
  doctor／啟動遷移會匯入已發布的 `debug-proxy/capture.sqlite` 資料列
  及其參照的承載資料 Blob，包括啟用中的舊版資料庫／Blob 環境
  覆寫，然後封存這些來源，同時保留 CA 憑證不變。

此階段也會從這些子系統中刪除重複的附屬開啟器、權限輔助函式、WAL
設定、檔案系統修剪，以及相容性寫入器。

### 階段 2：引入每個代理程式專屬的資料庫

為每個代理程式建立一個資料庫，並從全域 DB 註冊：

```text
~/.openclaw/state/openclaw.sqlite
~/.openclaw/agents/<agentId>/agent/openclaw-agent.sqlite
```

全域 `agent_databases` 資料列會儲存路徑、結構描述版本、上次出現的
時間戳記，以及基本的大小／完整性中繼資料。執行階段程式碼會向登錄取得
代理程式 DB，而不是直接推導檔案路徑。

代理程式 DB 負責：

- `sessions` 作為標準工作階段根，其中 `session_entries` 是附加至該根的
  相容性形狀承載資料表，而
  `session_routes` 則是唯一的作用中 `session_key` 查詢
- `conversations` 和 `session_conversations` 作為附加至工作階段的正規化提供者
  路由身分
- `transcript_events`
- 逐字記錄快照與壓縮檢查點。執行階段寫入已完成。
- `vfs_entries`
- `tool_artifacts` 和執行成品
- 代理程式本機執行階段／快取資料列。工作程式範圍快取已完成。
- ACP 父串流事件
- 軌跡執行階段事件，但明確的匯出成品除外

### 階段 3：取代工作階段儲存區 API

執行階段已完成。檔案形狀的工作階段儲存區介面不再是作用中的
執行階段合約：

- 執行階段不再呼叫 `loadSessionStore(storePath)`，也不再將 `storePath` 視為
  工作階段身分。
- 執行階段資料列作業為 `getSessionEntry`、`upsertSessionEntry`、
  `patchSessionEntry`、`deleteSessionEntry` 和 `listSessionEntries`。
- 全儲存區重寫輔助函式、檔案寫入器、佇列測試、別名修剪，以及
  舊版索引鍵刪除參數，均已從執行階段移除。
- 已棄用的根套件相容性匯出會委派給僅限 doctor 使用的
  `sessions.json` 匯入器，直到 2026-10-12；外掛 SDK 相容性讀取則
  繼續投影標準 SQLite 資料列。
- `sessions.json` 剖析僅保留在 doctor 遷移／匯入程式碼和
  doctor 測試中。
- 執行階段生命週期備援會讀取 SQLite 逐字記錄標頭，而不是 JSONL 的第一
  行。

繼續刪除任何重新引入檔案鎖定參數、
以修剪／截斷作為檔案維護的用語、儲存區路徑身分，或唯一斷言為 JSON 持久性的測試
之內容。

### 階段 4：移動逐字記錄、ACP 串流、軌跡及 VFS

讓每個代理程式資料串流都原生採用資料庫：

- 逐字記錄附加寫入會透過單一 SQLite 交易進行，該交易會確保
  工作階段標頭存在、檢查訊息等冪性、選取父尾端、插入
  `transcript_events`，並在 `transcript_event_identities` 中記錄可查詢的身分中繼資料。
  直接附加逐字記錄訊息及一般持久化的 `TranscriptSessionManager` 附加已完成；
  明確的分支作業會保留其明確的父項選擇，且仍會寫入 SQLite 資料列，
  而不推導任何檔案定位器。
- ACP 父串流日誌會成為資料列，而非 `.acp-stream.jsonl` 檔案。已完成。
- ACP 衍生設定不再持久保存逐字記錄 JSONL 路徑。已完成。
- 執行階段軌跡擷取會直接寫入事件資料列／成品。明確的
  支援／匯出命令仍可產生支援套件 JSONL 成品作為
  匯出格式，但工作階段匯出不會重新建立工作階段 JSONL。已完成。
- 設定為磁碟模式時，磁碟工作區會保留在磁碟上。
- VFS 暫存空間及實驗性僅限 VFS 的工作區模式使用代理程式 DB。

遷移會匯入舊 JSONL 檔案一次、在
`migration_runs` 中記錄計數／雜湊，並在完整性檢查後移除已匯入的檔案。

### 階段 5：備份、還原、Vacuum 及驗證

備份會維持為單一封存檔：

- 為每個全域及代理程式資料庫建立檢查點。
- 先使用 SQLite 線上備份建立每個 DB 的快照，接著離線執行 `VACUUM`。
- 封存精簡的 DB 快照、設定、外部認證資訊，以及要求的
  工作區匯出。
- 省略原始即時 `*.sqlite-wal` 和 `*.sqlite-shm` 檔案。
- 透過開啟每個 DB 快照並執行 `PRAGMA integrity_check` 來驗證。
  `openclaw backup create` 預設會執行此封存驗證；
  `--no-verify` 只會略過寫入後的封存檢查，而不會略過快照
  建立時的完整性檢查。
- 還原會將快照複製回其目標路徑。還原的全域 DB 使用
  版本 `1`；還原的每個代理程式 DB 使用版本 `2`，版本 `1` 快照則會在開啟時
  以不可分割方式升級。

### 階段 6：工作程式執行階段

在資料庫拆分落地期間，讓工作程式模式維持實驗性：

- 工作程式會接收代理程式 ID、執行 ID、檔案系統模式，以及 DB 登錄身分。
- 每個工作程式會開啟自己的 SQLite 連線。
- 父項保有頻道傳遞、核准、設定及取消權限。
- 一開始，每個作用中執行使用一個工作程式；僅在生命週期和 DB
  連線所有權穩定後才加入集區。

### 階段 7：刪除舊世界

執行階段工作階段管理已完成。舊世界僅允許作為明確的
doctor 輸入或支援／匯出輸出：

- 執行階段不會寫入 `sessions.json`、逐字記錄 JSONL、沙箱登錄 JSON、任務
  附屬 SQLite，或外掛狀態附屬 SQLite。
- 沒有 JSON／工作階段檔案修剪、檔案逐字記錄截斷、工作階段檔案鎖定，
  或鎖定形狀的工作階段測試。
- 沒有以維持舊工作階段檔案為最新狀態為目的的執行階段相容性
  匯出。
- 明確的支援匯出仍是使用者要求的封存／具現化
  格式，且不得將檔案名稱回饋至執行階段身分中。

## 備份與還原

備份應為單一封存檔，但資料庫擷取應
原生採用 SQLite：

1. 讓寫入交易保持有界，使線上備份能持續推進。
2. 擷取前驗證每個即時全域及代理程式資料庫。
3. 使用 SQLite 線上備份，將每個資料庫擷取至暫存備份
   目錄，接著關閉即時連線並對私有副本執行 `VACUUM`。
   需要擁有者定義之 SQLite 功能的外掛結構描述會採取失敗關閉，
   直到擁有者提供安全的快照合約。
4. 封存資料庫快照、設定檔、認證資訊目錄、選定的
   工作區，以及資訊清單。
5. 驗證每個 SQLite 快照的檔案形狀，接著開啟標準 OpenClaw
   資料庫，並執行 `PRAGMA integrity_check` 與角色驗證。專用
   外掛結構描述維持不透明，除非其擁有者提供驗證器。
   `openclaw backup create` 預設會執行此操作；`--no-verify` 僅用於
   刻意略過寫入後的封存檢查。

不要依賴原始即時 `*.sqlite`、`*.sqlite-wal` 和 `*.sqlite-shm` 副本作為
主要備份格式。封存資訊清單應記錄資料庫角色、
代理程式 ID、結構描述版本、來源路徑、快照路徑、位元組大小及完整性
狀態。

還原應從封存快照重建全域資料庫和代理程式資料庫檔案。
全域結構描述維持版本 `1`；每個代理程式版本 `1`
快照會進行有界的執行階段升級至版本 `2`。doctor 仍是
檔案至資料庫匯入的唯一擁有者。還原命令會先驗證
封存檔，接著從已驗證的解壓縮承載資料取代每項資訊清單資產。

## 執行階段重構計畫

1. 新增資料庫登錄 API。
   - 解析全域 DB 及每個代理程式 DB 路徑。
   - 將全域結構描述維持在 `user_version = 1`。每個代理程式 DB 使用版本 `2`，
     並從已發布的版本 `1` 記憶體來源形狀進行一次不可分割的遷移。
   - 新增供測試、備份及 doctor 使用的關閉／檢查點／完整性輔助函式。

2. 整併附屬 SQLite 儲存區。
   - 將外掛狀態資料表移入全域資料庫。執行階段
     寫入已完成；未發布的舊版附屬匯入器已刪除。
   - 將任務登錄資料表移入全域資料庫。執行階段
     寫入已完成；未發布的舊版附屬匯入器已刪除。
   - 將 Task Flow 資料表移入全域資料庫。執行階段寫入已完成；
     未發布的舊版附屬匯入器已刪除。
   - 將內建記憶體搜尋資料表移入每個代理程式資料庫。已完成；明確的
     自訂 `memorySearch.store.path` 現在會由 doctor 設定遷移移除。
     完整重新建立索引僅會就地針對記憶體資料表執行；舊的整檔
     交換路徑及附屬索引交換輔助函式已刪除。
   - 從這些子系統中刪除重複的資料庫開啟器、WAL 設定、權限輔助函式及
     關閉路徑。

3. 將代理程式擁有的資料表移入每個代理程式資料庫。
   - 透過全域資料庫登錄依需求建立代理程式 DB。已完成。
   - 將執行階段工作階段項目、逐字記錄事件、VFS 資料列及工具
     成品移至代理程式 DB。已完成。
   - 不要遷移分支本機的共用 DB 工作階段項目、逐字記錄事件、
     VFS 資料列或工具成品；該配置從未發布。doctor 中僅保留舊版
     檔案至資料庫匯入。

4. 取代工作階段儲存區 API。
   - 移除 `storePath` 作為執行階段身分。執行階段已完成，並由
     `check:database-first-legacy-stores` 守護：工作階段中繼資料、路由更新、
     命令持久化、命令列介面工作階段清理、Feishu 推理預覽、
     逐字記錄狀態持久化、子代理程式深度、認證設定檔工作階段
     覆寫、父項分叉邏輯及 QA-lab 檢查，現在會從標準代理程式／工作階段索引鍵
     解析資料庫。
     閘道／終端介面／UI／macOS 工作階段清單回應現在會公開 `databasePath`，
     而非舊版 `path`；macOS 偵錯介面會將每個代理程式資料庫顯示為
     唯讀狀態，而非寫入 `session.store` 設定。
     `/status`、由聊天驅動的軌跡匯出，以及命令列介面相依性代理，
     不再傳播舊版儲存區路徑；逐字記錄用量備援會依代理程式／工作階段身分
     讀取 SQLite。執行階段及橋接測試不再公開
     `storePath`；doctor／遷移輸入負責該舊版欄位名稱。
     閘道合併工作階段載入不再針對非範本化的
     `session.store` 值設有特殊執行階段分支；它會彙總每個代理程式的 SQLite 資料列。
     舊版工作階段鎖定 doctor 路徑及其 `.jsonl.lock` 清理輔助函式
     已移除；SQLite 現在是工作階段並行處理邊界。
     高頻執行階段呼叫位置使用資料列導向的輔助函式名稱，例如
     `resolveSessionRowEntry`；舊的 `resolveSessionStoreEntry` 相容性
     別名已從執行階段及外掛 SDK 匯出中移除。

- 使用 `{ agentId, sessionKey }` 資料列操作。
  已完成：`getSessionEntry`、`upsertSessionEntry`、`deleteSessionEntry`、
  `patchSessionEntry` 和 `listSessionEntries` 都是以 SQLite 優先的 API，
  不需要工作階段儲存區路徑。狀態摘要、本機代理程式狀態、健康情況，
  以及 `openclaw sessions` 清單命令現在會直接讀取各代理程式的資料列，
  並顯示各代理程式的 SQLite 資料庫路徑，而非 `sessions.json` 路徑。
- 將整個儲存區的刪除／插入，替換為 `upsertSessionEntry`、
  `deleteSessionEntry`、`listSessionEntries` 和 SQL 清理查詢。
  執行階段已完成：熱路徑現在使用資料列 API 和衝突重試資料列修補；
  剩餘的整個儲存區匯入／替換輔助函式僅限於遷移匯入程式碼和
  SQLite 後端測試。
  - 刪除 `store-writer.ts` 和寫入器佇列測試。已完成。
  - 從工作階段資料列的 upsert／修補中，刪除執行階段舊版鍵清除和別名刪除參數。
    已完成。

5. 刪除執行階段 JSON 登錄行為。
   - 讓沙箱登錄的讀寫僅使用 SQLite。已完成。
   - 僅從遷移步驟匯入單體式和分片式 JSON。已完成。
   - 移除分片式登錄鎖定和 JSON 寫入。已完成。

- 如果登錄資料列的結構仍是熱路徑操作狀態，請保留一個具型別的登錄資料表，
  而非將其儲存為一般的不透明 JSON。已完成。

6. 刪除檔案鎖定形式的工作階段變更。
   - 執行階段鎖定建立和執行階段鎖定 API 已完成。
   - 已移除獨立的舊版 `.jsonl.lock` doctor 清理流程。
   - 狀態完整性不再具有獨立的孤立逐字稿檔案清除路徑；
     doctor 遷移會在同一處匯入／移除舊版 JSONL 來源。
   - 閘道單例協調會使用 `gateway_locks` 下具型別的 SQLite
     `state_leases` 資料列，且不再公開檔案鎖定目錄接合面。
   - 一般外掛 SDK 的去重持久化不再使用檔案鎖定或 JSON
     檔案；它會寫入共用的 SQLite 外掛狀態資料列。已完成。
   - QMD 協調會針對嵌入使用共用 SQLite 租約，並針對每個集合／更新／嵌入寫入器
     使用各代理程式的 SQLite 租約。執行階段不再建立
     `qmd/embed.lock.lock` 或 `agents/<agentId>/qmd-write.lock.lock`；
     Doctor 僅移除確定已過時的退役附屬檔案。已完成。

7. 讓工作程式支援資料庫。
   - 工作程式會開啟自己的 SQLite 連線。
   - 父程序擁有傳遞、頻道回呼和設定。
   - 工作程式接收代理程式 ID、執行 ID、檔案系統模式和資料庫登錄識別資訊，
     而非即時控制代碼。
   - `vfs-only` 維持實驗性質，並使用代理程式資料庫作為其儲存根目錄。
   - 一開始每個使用中的執行保留一個工作程式。集區化可等到資料庫連線
     生命週期和取消行為都穩定無虞之後再進行。

8. 備份整合。
   - 讓備份功能先透過線上備份建立全域、代理程式和外掛資料庫的快照，
     接著離線執行 `VACUUM`。已針對狀態資產下探索到的
     `*.sqlite` 檔案完成；需要無法取得之擁有者能力的外掛結構描述會採取封閉失敗。
   - 新增標準 SQLite 完整性和結構描述識別資訊的備份驗證，
     並為專用外掛快照新增一般檔案結構驗證。備份建立和預設封存驗證已完成。
   - 在 SQLite 中記錄備份執行中繼資料。已透過共用的 `backup_runs`
     資料表完成，其中包含封存路徑、狀態和資訊清單 JSON。
   - 新增從已驗證封存快照還原的功能。已完成：`openclaw backup
restore`
     會在解壓縮前驗證、使用驗證器正規化後的資訊清單、支援
     `--dry-run`，並要求在替換已記錄的來源路徑前提供
     `--yes`。
   - 僅在要求時納入 VFS／工作區匯出；不要將工作階段內部資料匯出為
     JSON 或 JSONL。

9. 刪除過時的測試和程式碼。已針對已知的執行階段工作階段介面完成。

- 移除會斷言執行階段建立 `sessions.json` 或逐字稿
  JSONL 檔案的測試。核心工作階段儲存區、聊天、閘道逐字稿事件、
  預覽、生命週期、命令工作階段項目更新、自動回覆重設／追蹤，以及
  memory-core 夢境整理固定資料、核准目標路由、工作階段逐字稿修復、
  安全性權限修復、軌跡匯出和工作階段匯出均已完成。
  主動記憶逐字稿測試現在會斷言 SQLite 範圍，且不會建立暫存或
  持久化的 JSONL 檔案。
  舊有的心跳偵測逐字稿清除迴歸測試已移除，因為
  執行階段不再截斷 JSONL 逐字稿。
  代理程式工作階段清單工具測試不再將舊版 `sessions.json` 路徑
  模擬為閘道回應結構；應用程式／UI／macOS 測試改用 `databasePath`。
  `/status` 逐字稿使用量測試現在會直接植入 SQLite 逐字稿資料列，
  而非寫入 JSONL 檔案。
  閘道工作階段生命週期測試現在會直接使用 SQLite 逐字稿植入輔助函式；
  舊有的單行工作階段檔案固定資料結構已從重設和刪除涵蓋範圍中移除。
  `sessions.delete` 不再傳回檔案時代的 `archived: []` 欄位；刪除
  僅回報資料列變更結果。舊有的 `deleteTranscript` 選項也已移除：
  刪除工作階段會移除標準的 `sessions` 根目錄，並讓
  SQLite 串聯刪除工作階段擁有的逐字稿、快照和軌跡資料列，因此沒有
  任何呼叫端能留下孤立的逐字稿，或遺漏清理分支。
  上下文引擎軌跡擷取測試現在會從隔離的代理程式資料庫讀取
  `trajectory_runtime_events` 資料列，而非讀取
  `session.trajectory.jsonl`。
  Docker MCP 頻道植入指令碼現在會直接植入 SQLite 資料列。直接寫入
  `sessions.json` 僅限於 doctor 固定資料。
  Tool Search Gateway E2E 會從 SQLite 逐字稿資料列讀取工具呼叫證據，
  而非掃描 `agents/<agentId>/sessions/*.jsonl` 檔案。
  Memory-core 主機事件和工作階段語料庫暫存資料列現在位於共用的
  SQLite 外掛狀態中；`events.jsonl` 和 `session-corpus/*.txt` 僅是舊版
  doctor 遷移輸入。使用中資料列使用 `memory/session-ingestion/`
  虛擬路徑，而非 `.dreams/session-corpus`。舊有的 memory-core 夢境整理
  修復模組及其命令列介面／閘道測試已移除，因為執行階段不再
  負責該語料庫的檔案封存修復。Memory-core
  橋接／公開成品測試不再公開 `.dreams/events.jsonl`；它們
  使用由 SQLite 支援的虛擬 JSON 成品名稱。
  公開 SDK／Codex 測試文件現在使用 SQLite 工作階段狀態，而非工作階段
  檔案，且頻道回合範例不再公開 `storePath` 引數。
  Matrix 同步狀態現在會直接使用 SQLite 外掛狀態儲存區。使用中的
  用戶端／執行階段合約會傳遞帳戶儲存根目錄，而非 `bot-storage.json`
  路徑，doctor 則會先將舊版 `bot-storage.json` 匯入 SQLite，再刪除
  來源。QA Lab Matrix 重新啟動／破壞性情境現在會直接變更 SQLite 同步
  資料列，而非建立或刪除假的 `bot-storage.json` 檔案，且
  E2EE 基礎層會傳遞同步儲存區根目錄，而非假的
  `sync-store.json` 路徑。
  Matrix 儲存根目錄選擇不再依舊版同步／執行緒 JSON
  檔案為根目錄評分；它改用持久根目錄中繼資料和真實的加密狀態。
  執行階段 SQLite 工作階段後端測試套件不再建立假的
  `sessions.json`；舊版來源固定資料現在位於匯入它們的 doctor
  測試中。
  閘道工作階段測試不再公開 `createSessionStoreDir` 輔助函式或
  未使用的暫存工作階段儲存區路徑設定；固定資料目錄會明確指定，直接
  設定資料列則使用 SQLite 工作階段資料列命名。
  僅供 doctor 使用的 JSON5 工作階段儲存區剖析器涵蓋範圍已從基礎設施測試
  移至 doctor 遷移測試，因此執行階段測試套件不再負責舊版
  工作階段檔案剖析。
  Microsoft Teams 執行階段 SSO／待上傳測試不再攜帶 JSON 附屬檔案
  固定資料或剖析器；舊版 SSO 權杖剖析僅存在於外掛
  遷移模組中。Telegram 測試不再植入假的 `/tmp/*.json` 儲存區
  路徑；它們會直接重設由 SQLite 支援的訊息快取。一般
  OpenClaw 測試狀態輔助函式不再公開舊版 `auth-profiles.json`
  寫入器；doctor 驗證遷移測試會在本機擁有該固定資料。
  終端介面上次工作階段指標、exec 核准、主動記憶
  切換、Matrix 去重／啟動驗證、Memory Wiki 來源同步、
  目前對話繫結、上線驗證和 Hermes 機密匯入的執行階段測試，
  不再建立舊有的附屬檔案，或斷言舊檔名不存在。它們
  透過 SQLite 資料列和公開儲存區 API 證明行為；doctor／遷移
  測試是唯一應出現舊版來源檔名之處。
  裝置／節點配對、頻道 allowFrom、重新啟動意圖、
  重新啟動移交、工作階段傳遞佇列項目、設定健康情況、iMessage
  快取、排程工作、PI 逐字稿標頭、子代理程式登錄和受管理
  圖片附件的執行階段測試，也不再僅為證明已忽略或不存在
  退役的 JSON／JSONL 檔案而建立它們。
  PI 溢位復原不再具有 SessionManager 重寫／截斷
  後援：工具結果截斷和上下文引擎逐字稿重寫會變更
  SQLite 逐字稿資料列，接著從資料庫重新整理使用中的提示詞狀態。
  持久化的 SessionManager 訊息附加會委派給原子 SQLite
  逐字稿附加輔助函式，以處理父項選擇和冪等性。一般
  中繼資料／自訂項目附加也會在 SQLite 內選擇目前父項，因此
  過時的管理器執行個體不會重新引發 SQLite 之前的父項鏈競爭。
  針對回合中途預先檢查和 `sessions_yield` 的合成 PI 尾端清理，
  現在會直接修剪 SQLite 逐字稿狀態；舊有的 SessionManager 尾端移除
  橋接及其測試已刪除。
  壓縮檢查點擷取也僅從 SQLite 建立快照；呼叫端不再
  將即時 SessionManager 作為替代逐字稿來源傳入。
- 僅保留為遷移植入舊版檔案的測試。
- 使用中執行階段介面的 JSON 檔案證明已替換為 SQL 資料列證明。

- 新增禁止執行階段寫入舊版工作階段／快取 JSON 路徑的靜態規則。
  儲存庫防護已完成。

10. 讓遷移報告可供稽核。
    - 在 SQLite 中記錄遷移執行，包含開始／完成時間戳記、來源
      路徑、來源雜湊、計數、警告和備份路徑。
      已完成：舊版狀態遷移執行現在會持久化 `migration_runs`
      報告，其中包含來源路徑／資料表清冊、來源檔案 SHA-256、大小、
      記錄數、警告和備份路徑。
      已完成：舊版狀態遷移執行也會持久化 `migration_sources`
      資料列，以供來源層級稽核和未來的略過／回填決策使用。
    - 讓套用操作具備冪等性。在部分匯入後重新執行時，應略過
      已匯入的來源，或依穩定鍵合併。
      已完成：工作階段索引、逐字稿、傳遞佇列、外掛狀態、任務
      分類帳，以及代理程式擁有的全域 SQLite 資料列，會透過穩定鍵或
      upsert／替換語意匯入，因此重新執行時會合併而不重複建立持久
      資料列。
    - 匯入失敗時，必須將原始來源檔案保留在原位。
      已完成：逐字稿匯入失敗時，現在會將原始 JSONL 來源保留在
      偵測到的路徑，而 `migration_sources` 會將來源記錄為
      `warning`，並附上 `removed_source=0`，供下次 doctor 執行使用。

## 效能規則

- 每個執行緒／程序各用一個連線即可；不要在工作程序之間共用控制代碼。
- 使用 WAL、`foreign_keys=ON`、5 秒忙碌逾時，以及簡短的 `BEGIN IMMEDIATE`
  寫入交易。不要在 SQLite 的單次忙碌等待之上疊加同步鎖定重試。
- 除非／直到非同步交易 API 加入明確的互斥鎖／背壓語意，否則讓寫入交易輔助函式保持同步。
- 讓父層遞送寫入保持小型且具交易性。
- 避免重寫整個儲存區；使用資料列層級的 upsert／delete。
- 在移動熱路徑程式碼之前，先為依代理程式列出、依工作階段列出、更新時間、執行 ID 和到期路徑新增索引。
- 將大型成品、媒體和向量儲存為 BLOB 或分塊 BLOB 資料列，而非 base64 或數值陣列 JSON。
- 讓不透明的外掛狀態項目保持小型且限定範圍。
- 使用 SQL 清理 TTL／到期項目，而非修剪檔案系統。
  資料庫所擁有的執行階段儲存區已完成此作業：媒體、外掛狀態、外掛 Blob、
  持久性去重和代理程式快取都透過 SQLite 資料列到期。剩餘的
  檔案系統清理僅限於暫時具現化項目或明確的
  移除命令。

## 靜態禁令

新增一項存放庫檢查，讓新增至舊版狀態路徑的執行階段寫入失敗：

- `sessions.json`
- `*.trajectory.jsonl`，但具現化的支援套件輸出除外
- `.acp-stream.jsonl`
- `acp/event-ledger.json`
- `cache/*.json` 執行階段快取檔案
- `agents/<agentId>/agent/auth.json`
- `agents/<agentId>/agent/models.json`
- `credentials/oauth.json`
- `github-copilot.token.json`
- `openrouter-models.json`
- `auth-profiles.json`
- `auth-state.json`
- `exec-approvals.json`
- `openclaw-workspace-state.json`
- `workspace-state.json`
- `workspace-attestations/*.attested`
- 同層的 `<workspace>.attested`
- Matrix `credentials*.json` 和 `recovery-key.json`
- `cron/runs/*.jsonl`
- `cron/jobs.json`
- `jobs-state.json`
- `device-pair-notify.json`
- `devices/pending.json`／`devices/paired.json`／`devices/bootstrap.json`
  （已於 2026.7 淘汰：執行階段儲存區為共用狀態資料庫中的 `device_pairing_*`／
  `device_bootstrap_tokens`；配對記錄會在
  閘道啟動時匯入，暫時的待處理／啟動程序資料列則會被捨棄）
- `nodes/pending.json`／`nodes/paired.json`（已於 2026.7 淘汰：在閘道啟動時併入配對裝置記錄）
- `identity/device.json`
- `identity/device-auth.json`（已淘汰；僅由 Doctor 匯入至 `device_auth_tokens`）
- `push/web-push-subscriptions.json`（已淘汰；僅由 Doctor 匯入至 `web_push_subscriptions`）
- `push/vapid-keys.json`（已淘汰；僅由 Doctor 匯入至 `web_push_vapid_keys`）
- `push/apns-registrations.json`（已淘汰；僅由 Doctor 匯入至 `apns_registrations`）
- `process-leases.json`
- `gateway-instance-id`
- `session-toggles.json`
- Memory-core `.dreams/events.jsonl`
- Memory-core `.dreams/session-corpus/`
- Memory-core `.dreams/daily-ingestion.json`
- Memory-core `.dreams/session-ingestion.json`
- Memory-core `.dreams/short-term-recall.json`
- Memory-core `.dreams/phase-signals.json`
- Memory-core `.dreams/short-term-promotion.lock`
- Skill Workshop `skill-workshop/<workspace>.json`
- Skill Workshop `skill-workshop/skill-workshop-review-*.json`
- Nostr `bus-state-*.json`
- Nostr `profile-state-*.json`
- `calls.jsonl`
- `known-users.json`
- `ref-index.jsonl`
- QQ Bot `session-*.json`
- BlueBubbles `bluebubbles/catchup/*.json`
- BlueBubbles `bluebubbles/inbound-dedupe/*.json`
- Telegram `update-offset-*.json`
- Telegram `sticker-cache.json`
- Telegram `*.telegram-messages.json`
- Telegram `*.telegram-sent-messages.json`
- Telegram `*.telegram-topic-names.json`
- Telegram `thread-bindings-*.json`
- iMessage `catchup/*.json`
- iMessage `reply-cache.jsonl`
- iMessage `sent-echoes.jsonl`
- Microsoft Teams `msteams-conversations.json`
- Microsoft Teams `msteams-polls.json`
- Microsoft Teams `msteams-sso-tokens.json`
- Microsoft Teams `*.learnings.json`
- Matrix `bot-storage.json`
- Matrix `sync-store.json`
- Matrix `thread-bindings.json`
- Matrix `inbound-dedupe.json`
- Matrix `startup-verification.json`
- Matrix `storage-meta.json`
- Matrix `crypto-idb-snapshot.json`
- Discord `model-picker-preferences.json`
- Discord `command-deploy-cache.json`
- 沙箱登錄分片 JSON 檔案
- `plugin-state/state.sqlite`
- 臨時的 `openclaw-state.sqlite` 執行階段附屬檔案
- `tasks/runs.sqlite`
- `tasks/flows/registry.sqlite`
- `bindings/current-conversations.json`
- `restart-sentinel.json`
- `gateway-restart-intent.json`
- `gateway-supervisor-restart-handoff.json`
- `gateway.<hash>.lock`
- `qmd/embed.lock.lock`
- `agents/<agentId>/qmd-write.lock.lock`
- `commands.log`
- `config-health.json`
- `port-guard.json`
- `settings/voicewake.json`
- `settings/voicewake-routing.json`
- `plugin-binding-approvals.json`
- `plugins/installs.json`
- `audit/file-transfer.jsonl`
- `audit/crestodian.jsonl`
- `crestodian/rescue-pending/*.json`
- `openclaw/rescue-pending/*.json`
- `plugins/phone-control/armed.json`
- Memory Wiki `.openclaw-wiki/log.jsonl`
- Memory Wiki `.openclaw-wiki/state.json`
- Memory Wiki `.openclaw-wiki/locks/`
- Memory Wiki `.openclaw-wiki/source-sync.json`
- Memory Wiki `.openclaw-wiki/import-runs/*.json`
- Memory Wiki `.openclaw-wiki/cache/agent-digest.json`
- Memory Wiki `.openclaw-wiki/cache/claims.jsonl`
- ClawHub `.clawhub/lock.json`
- ClawHub `.clawhub/origin.json`
- 瀏覽器設定檔裝飾 `.openclaw-profile-decorated`
- `SessionManager.open(...)` 以檔案為後端的工作階段開啟器
- `SessionManager.listAll(...)` 和 `TranscriptSessionManager.listAll(...)`
  對話記錄列出介面
- `SessionManager.forkFromSession(...)` 和
  `TranscriptSessionManager.forkFromSession(...)` 對話記錄分叉介面
- `SessionManager.newSession(...)` 和 `TranscriptSessionManager.newSession(...)`
  可變工作階段取代介面
- `SessionManager.createBranchedSession(...)` 和
  `TranscriptSessionManager.createBranchedSession(...)` 分支工作階段介面

此禁令應允許測試建立舊版固定資料，並允許遷移程式碼
讀取／匯入／移除舊版檔案來源。尚未發布的 SQLite 附屬檔案仍受禁止，
且不會獲得 Doctor 匯入豁免。

## 完成條件

- 執行階段資料和快取寫入全域或代理程式 SQLite 資料庫。
- 執行階段不再寫入工作階段索引、對話記錄 JSONL、沙箱登錄
  JSON、任務附屬 SQLite 或外掛狀態附屬 SQLite。刪除尚未發布的任務
  和外掛狀態附屬 SQLite 匯入器。
- 舊版檔案匯入僅限 Doctor。
- 備份會產生單一封存檔，其中包含壓縮的 SQLite 快照和完整性證明。
- 代理程式工作程序可以使用磁碟、VFS 暫存空間或實驗性的純 VFS
  儲存空間執行。
- 設定檔和明確認證資訊檔案仍是唯一預期持久存在的
  非資料庫控制檔案。
- 存放庫檢查可防止重新引入舊版執行階段檔案儲存區。
