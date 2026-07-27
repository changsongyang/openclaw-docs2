---
read_when:
    - 你正在實作 clawdbot-d63.2 / clawdbot-04b
    - 你正在處理 SQLite 工作階段保留、重設、刪除或代理程式刪除封存作業
    - 你需要區分 SQLite 時代的成品系列與舊版 JSONL 附屬檔案
summary: 封存屬於某個工作階段的所有 SQLite 對話記錄成品之路徑 3 計畫
title: 路徑 3 SQLite 工作階段成品系列
x-i18n:
    generated_at: "2026-07-26T07:24:09Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 29f4d541b2df5a06468fd0cee620b4340ee33eea1064f7d3ee823580c7b5760e
    source_path: plan/path3-sqlite-session-artifact-family.md
    workflow: 16
---

# 路徑 3 SQLite 工作階段成品家族

本說明界定 `clawdbot-d63.2` 的範圍，而 `clawdbot-d63.1` 負責 `src/config/sessions/session-accessor.sqlite.ts` 中範圍重疊的
重設／刪除封存輔助函式。
本次處理期間實作檔案處於未清理狀態，因此本成品記錄
確切的合約與修補位置，以免與同層工作者產生競爭。

## 權威家族

切換至 SQLite 後，作用中的工作階段逐字稿會成為 SQLite 資料列。工作階段的
封存家族如下：

- 項目目前 `sessionId` 的 `transcript_events`、`transcript_event_identities` 與 `sessions` 資料列。
- `entry.compactionCheckpoints[*].preCompaction.sessionId` 所參照之每個 `sessionId` 的相同 SQLite 逐字稿資料列集合。
- `entry.compactionCheckpoints[*].postCompaction.sessionId` 所參照之每個 `sessionId` 的相同 SQLite 逐字稿資料列集合。
- `entry.usageFamilySessionIds` 中每個 `sessionId` 的相同 SQLite 逐字稿資料列集合。

僅封存已不再由任何其餘
`session_entries` 資料列，或任何其餘項目的壓縮或用量家族
中繼資料所參照的資料列。這會保留檢查點分支／還原與用量彙總狀態，直到
最後一個作用中參照消失為止。

## 切換後的非家族成品

產生的主題逐字稿檔案變體與軌跡附屬檔案並非作用中的
SQLite 執行階段狀態。它們是舊版檔案成品：

- 像 `<sessionId>-topic-<thread>.jsonl` 這類主題變體只存在於
  以檔案為後端的逐字稿格式。SQLite 使用標準工作階段 ID 加上
  `session_routes`／項目傳遞中繼資料，而非每個主題各自的 JSONL 檔案。
- 像 `.trajectory.jsonl` 與 `.trajectory-path.json` 這類軌跡附屬檔案
  是依照實際 JSONL `sessionFile` 路徑命名。SQLite `sessionFile` 值是
  `sqlite:<agentId>:<sessionId>:<storePath>` 標記，並非附屬檔案的
  名稱。
- 封存層讀取器必須繼續讀取舊版封存 JSONL 檔案，但
  執行階段保留機制不得掃描作用中工作階段目錄，亦不得為 SQLite 工作階段重新開啟 JSONL
  逐字稿檔案。

Doctor 匯入仍是舊版主要 JSONL 檔案及其
相鄰軌跡附屬檔案的遷移負責者。SQLite 執行階段保留機制不應新增第二個
匯入器或檔案後援機制。

## 修補位置

擴充由 `clawdbot-d63.1` 引入的 SQLite 封存輔助函式，而非
新增平行路徑。

1. 在 `deleteSqliteSessionStateIfUnreferenced` 附近新增本機收集器：
   - `collectSqliteSessionArtifactFamily(entry: SessionEntry): Set<string>`
   - 納入 `entry.sessionId`、檢查點前／後工作階段 ID，以及
     `usageFamilySessionIds`。
   - 篩除空字串，並以確定性方式去除重複項目。

2. 為移除後的儲存區新增參照收集器：
   - `readReferencedSqliteSessionArtifactFamilyIds(database): Set<string>`
   - 逐一處理目前的 `session_entries`、剖析每個 `entry_json`，並從
     每個保留的項目收集相同的家族 ID。

3. 變更目前只封存單一已移除
   `sessionId` 的重設／刪除／維護呼叫端，改為傳入已移除項目的完整家族。

4. 針對每個家族 ID，以呼叫端提供的
   原因（`reset` 或 `deleted`）封存 SQLite 逐字稿資料列，且僅在
   移除後的參照集合中不存在該家族 ID 時，才刪除 `sessions` 資料列。

5. 透過現有 SQLite
   工作階段資料列清理路徑，集中處理逐字稿事件刪除。不得新增作用中 JSONL 讀取。

## 聚焦測試

將僅限 SQLite 的測試新增至 `src/config/sessions/session-accessor.conformance.test.ts`，
或在 `clawdbot-d63.1` 提交後新增至同層生命週期測試：

- 刪除具有壓縮前逐字稿的項目時，會封存目前
  工作階段與壓縮前工作階段，接著移除兩者的 SQLite 資料列集合。
- 刪除共用壓縮前工作階段的兩個項目之一時，
  在最後一個參照項目移除之前，不會封存共用壓縮前工作階段的任何內容。
- 刪除具有 `usageFamilySessionIds` 的項目時，若沒有其他項目參照該用量家族，
  則會封存前置工作階段的 SQLite 逐字稿資料列。
- 具有 SQLite 標記且形似主題的工作階段鍵，不會觸發任何產生的
  主題 JSONL 讀取或附屬檔案查詢。

聚焦驗證應使用：

```bash
node scripts/run-vitest.mjs src/config/sessions/session-accessor.conformance.test.ts
```

對此 Codex 工作樹而言，廣泛的 `pnpm` 關卡應繼續在 Crabbox／Testbox 上執行。
