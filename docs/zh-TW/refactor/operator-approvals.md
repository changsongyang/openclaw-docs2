---
read_when:
    - 變更 exec 或外掛的核准生命週期、儲存、通訊協定或授權
    - 在頻道中新增核准連結或原生核准控制項
    - 將子工作階段的核准狀態投影至父工作階段或協調器檢視中
summary: 為控制介面、原生應用程式、頻道與父工作階段設計持久且可深層連結的核准機制
title: 多介面操作者核准
x-i18n:
    generated_at: "2026-07-26T08:11:34Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 9defdaada1911df1184f64429e1787c4881e735c433d6dbc30a5946e11cc7cce
    source_path: refactor/operator-approvals.md
    workflow: 16
---

# 多介面操作者核准

此設計追蹤 [#103505](https://github.com/openclaw/openclaw/issues/103505)。它以一套由閘道擁有、以 SQLite 為後端的生命週期，取代程序區域的核准權限。每個由閘道擁有的執行或外掛／工具核准，都會取得一個穩定 ID、一條經過驗證的 Control UI 路由、具原子性的先回答者勝出解析機制，以及僅供操作者查看、投射至其來源與祖先工作階段串流的資訊。

行內動作與深層連結並存。不設核准模式切換。

## 目標

- 為執行與外掛／工具閘門提供單一持久核准物件。
- 穩定的 `${controlUiBasePath}/approve/{approvalId}` 路由。
- 可從任何已授權的 Control UI、原生應用程式或頻道介面進行解析。
- 在並行介面間維持具原子性的先回答者勝出行為。
- 相同的重試具冪等性；衝突的延遲回答無法覆寫勝出結果。
- 逾時、格式錯誤的受信任裁決、缺少路由、取消與重新啟動，一律採取封閉式失敗。
- 已要求與終止事件會送達來源工作階段，以及所有相關的父層／協調器擁有者。
- 頻道會接收具型別的核准與導覽動作；傳輸回呼資料仍由頻道私有。
- 既有的執行／外掛閘道方法維持相容，同時其實作逐步收斂至單一服務。

## 非目標

- 在閘道重新啟動後，持久保存或恢復遭封鎖的工具執行本身。
- 將核准 ID 或 URL 當作持有者認證資訊。
- 將核准提示附加至模型可見的逐字稿，或喚醒父層代理程式。
- 將核准政策、產品命令或審查者授權移入頻道外掛。
- 依頻道、裝置或祖先複製核准狀態。
- 重新設計執行允許清單、外掛政策組合或 `allow-always` 持久化，除非為了使終止結果不含歧義而有必要。
- 在第一階段使無閘道的嵌入式終端介面可供遠端存取。它仍僅限本機使用，且沒有審查者時必須採取封閉式失敗。

## 推出前基準與證據圖

此表記錄 #103505 建立時的實作狀態。下方的推出章節會追蹤建構於該基準之上的持久登錄、具型別動作、深層連結頁面與原生用戶端增量。

| 介面              | 基準進入點與擁有者                                                                                                                                                | 基準行為與缺口                                                                                                                                                                               |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 代理程式執行      | `src/agents/bash-tools.exec-approval-request.ts`、`src/agents/bash-tools.exec-host-shared.ts`                                                                   | 兩階段 `exec.approval.*` 登錄可避免早期 `/approve` 競爭，但逾時仍可能透過 `askFallback` 轉為允許。                                                        |
| 外掛工具閘門      | `src/agents/agent-tools.before-tool-call.ts`                                                                                                                    | 要求 `plugin.approval.*`；`timeoutBehavior: "allow"` 可核准已逾時的閘門。嵌入模式在 `src/infra/embedded-plugin-approval-broker.ts` 中具有獨立的程序區域權限。 |
| 外掛節點閘門      | `src/gateway/node-invoke-plugin-policy.ts`                                                                                                                      | 直接透過外掛管理器建立並廣播，重複了部分伺服器方法生命週期。                                                                                                                                |
| 閘道權限          | `src/gateway/server-aux-handlers.ts`、`src/gateway/exec-approval-manager.ts`、`src/gateway/server-methods/approval-shared.ts`                                   | 個別的執行與外掛管理器使用程序區域映射。終止項目保留 15 秒。先回答者勝出僅在單一程序內成立。                                                                                                 |
| 閘道通訊協定      | `packages/gateway-protocol/src/schema/exec-approvals.ts`、`packages/gateway-protocol/src/schema/plugin-approvals.ts`、`src/gateway/methods/core-descriptors.ts` | 執行僅有待處理的 `get`；外掛沒有 `get`；不存在供深層連結使用、不限種類的終止查詢。                                                                                   |
| 傳遞              | `src/infra/exec-approval-channel-runtime.ts`、`src/infra/approval-native-runtime.ts`、`src/infra/approval-handler-runtime.ts`                                   | 支援來源路由、核准者私訊、待處理重播、原生處理常式與程序內終止清理。另行後續處理會加入持久終止狀態調和。                                                                                     |
| 可攜式動作        | `src/interactive/payload.ts`、`src/plugin-sdk/interactive-runtime.ts`、`src/plugin-sdk/approval-reply-runtime.ts`                                               | 核准按鈕是包含 `/approve ...` 的命令動作；URL 與 Web App 目標是無型別的按鈕欄位。                                                                           |
| Telegram          | `extensions/telegram/src/approval-handler.runtime.ts`、`extensions/telegram/src/button-types.ts`                                                                | 轉譯器會剖析命令文字以辨識核准語意，之後才產生私有回呼資料。                                                                                                                               |
| Control UI        | `ui/src/app/exec-approval.ts`、`ui/src/app/overlays.ts`、`ui/src/components/exec-approval.ts`                                                                   | 核准 UI 是全域對話方塊。`ui/src/app-route-paths.ts` 與 `ui/src/app-routes.ts` 使用精確路由，並將未知路徑改寫至 Chat。                                                    |
| 工作階段擁有權    | `src/agents/subagent-registry.types.ts`、`src/agents/subagent-registry-read.ts`、`src/config/sessions/types.ts`                                                 | 控制器、要求者、明確父層與舊版衍生擁有權皆已存在，但核准事件不會投射至這些工作階段串流。                                                                                                     |
| 共用狀態          | `src/state/openclaw-state-schema.sql`、`src/state/openclaw-state-db.ts`                                                                                         | 既有的即時交易與 Kysely 條件式更新，可支援 `state/openclaw.sqlite` 中的持久比較後設定。                                                                   |

目前具代表性的測試包括 `src/gateway/exec-approval-manager.test.ts`、`src/gateway/server-methods/approval-shared.test.ts`、`src/agents/bash-tools.exec-gateway-approval.e2e.test.ts`、`extensions/telegram/src/approval-handler.runtime.test.ts` 與 `ui/src/e2e/approval-flow.e2e.test.ts`。

外掛 SDK 仍是唯一的頻道／外掛邊界。核准執行階段與呈現方式的變更，必須透過既有的 `src/plugin-sdk/approval-*.ts` 與 `src/plugin-sdk/interactive-runtime.ts` 子路徑匯出；外掛正式環境程式碼不得匯入閘道內部元件。

## 既有實作參考

Omnigent 提供了實用的使用者體驗與失敗語意：

- [`approval.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/runtime/policies/approval.py) 會暫停 ASK、套用各政策的逾時設定，且僅將完全相符的接受視為核准。
- [`sessions.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/server/routes/sessions.py) 包含伺服器端原生測試工具閘門，以及祖先要求／解析投射。
- [`ApprovePage.tsx`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/web/src/pages/ApprovePage.tsx) 提供獨立的行動版核准頁面。

請勿未經審慎評估就照搬其儲存宣稱。目前作用中的待處理狀態在 [`_elicitation_registry.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/server/_elicitation_registry.py) 中是程序區域狀態，而未使用的待處理資料表已由 [`e3b1f2a4c9d7_drop_pending_tool_calls_table.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/db/migrations/versions/e3b1f2a4c9d7_drop_pending_tool_calls_table.py) 移除。OpenClaw 刻意更進一步：SQLite 是權威來源，且每個終止轉換都是資料庫的比較後設定。

## 架構與擁有權

閘道擁有此生命週期：

1. 代理程式、外掛鉤子或節點政策提供特定種類的要求，以及程序區域的執行繫結。
2. 閘道會驗證該要求，並建立經過清理的審查者投射。
3. 核准服務會計算來源／擁有者受眾、插入標準資料列，然後登錄程序內等待器。
4. 完成持久插入後，閘道會發布既有核准事件、工作階段投射、頻道通知與原生推播。
5. 每個介面都透過相同服務進行解析。
6. 服務會提交一次終止轉換、喚醒執行階段等待器，並發布終止投射。
7. 事件傳遞失敗絕不會回復已提交的決定；用戶端會透過 `approval.get` 或清單重播進行復原。

擁有權邊界：

- `src/gateway/`：核准服務、授權、RPC 配接器、URL 建構、等待器生命週期與事件發布。
- `src/state/`：共用結構描述與產生的 Kysely 型別。
- `src/infra/`：經過清理的核准檢視模型與可攜式呈現建構。
- `src/agents/`：要求、等待並套用傳回的裁決；不進行持久化。
- `src/channels/` 與 `extensions/*`：轉譯具型別的動作、授權頻道使用者、編碼私有回呼，並更新已傳遞的控制項。
- `src/plugin-sdk/`：僅包含公開的核准與呈現合約。
- `ui/`：獨立頁面與既有佇列／對話方塊用戶端。

程序內等待器是一種通知機制，而非權威來源。登錄會先插入資料列並同步安裝等待器，之後才發布要求，因此解析者無法介入這些步驟之間。後續每個解析者都會先透過 SQLite 提交，再完成該等待器。

## 持久記錄

在共用狀態資料庫中新增一個 `operator_approvals` 資料表。

| 欄位                                             | 用途                                                                                                                                       |
| -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `approval_id`                                      | 全域唯一的標準 ID。為了通訊協定相容性，保留現有的 exec ID 與 `plugin:` ID，但絕不可從前綴推斷種類。      |
| `resolution_ref`                                   | 唯一且完整的 SHA-256 base64url 定位值，用於無法攜帶標準 ID 的傳輸回呼。它不是授權資訊，也不是公開 URL ID。 |
| `kind`                                             | 封閉式 `exec \| plugin` 鑑別欄位。                                                                                                        |
| `status`                                           | 封閉式 `pending \| allowed \| denied \| expired \| cancelled` 狀態。                                                                          |
| `presentation_json`                                | 已驗證且標有種類的審查者投影。原始執行階段請求、命令繫結與回呼承載資料仍僅存在於處理程序內。               |
| `source_agent_id`, `source_session_key`            | 來源身分與工作階段投影錨點。工作階段金鑰是持久的；輪替的工作階段 UUID 則不是。                                          |
| `audience_session_keys_json`                       | 由有界廣度優先擁有權走訪產生的已排序、去重 JSON 陣列。請求事件與終止事件使用相同的快照。 |
| `requested_by_device_id`, `requested_by_client_id` | 持久的請求者／稽核中繼資料。連線 ID 保留在記憶體中，不是跨介面的主體。                                         |
| `reviewer_device_ids_json`                         | 選用的明確指定審查者裝置，僅由受信任的核准執行階段提供。                                                  |
| `runtime_epoch`                                    | 擁有暫停執行作業的處理程序 epoch；用於在重新啟動後取消孤立資料列。                                                     |
| `created_at_ms`, `expires_at_ms`, `updated_at_ms`  | 權威時間資訊。                                                                                                                         |
| `decision`                                         | 存在明確使用者決定時記錄該決定。                                                                                                       |
| `terminal_reason`                                  | 封閉式原因，例如 `user`、`timeout`、`malformed-verdict`、`no-route`、`run-aborted` 或 `gateway-restart`。                                |
| `resolved_at_ms`, `resolver_kind`, `resolver_id`   | 保留於伺服器端的勝出者與稽核身分。審查者投影不包含原始解析者識別碼。                                           |
| `consumed_at_ms`, `consumed_by`                    | `allow-once` 的獨立重播防護；使用後不得清除已記錄的決定。                                                       |

必要索引：

| 索引                                      | 用途                                                                     |
| ------------------------------------------ | --------------------------------------------------------------------------- |
| 唯一 `(resolution_ref)`                  | 在插入期間拒絕跨欄位的 `approval_id`/`resolution_ref` 歧義。 |
| `(status, expires_at_ms)`                  | 尋找待處理核准並協調權威期限。               |
| `(source_session_key, created_at_ms DESC)` | 重播單一來源工作階段的近期核准。                             |
| `(resolved_at_ms)`                         | 依固定保留政策清除保留的終止核准。  |

對象陣列很小且有界。依工作階段篩選的重播會先透過 Kysely 選取可見的待處理資料列，再於應用程式碼中解碼並篩選有界的對象陣列；它不使用字串比對或原始 SQL JSON 查詢。

終止資料列保留 30 天，與 `src/audit/audit-event-store.ts` 中的中繼資料稽核保留期一致。清除是固定的維護政策，不是新的設定介面。資料庫是私有的本機控制平面狀態，但審查者 API 絕不可公開完整的已儲存請求或執行階段繫結。

## 狀態機與比較後設定

僅允許下列轉換：

- `pending -> allowed`：明確的 `allow-once` 或 `allow-always`。
- `pending -> denied`：明確拒絕、受信任的格式錯誤終止裁決，或沒有傳遞路徑。
- `pending -> expired`：已到達權威期限。
- `pending -> cancelled`：執行中止、正常關閉，或重新啟動後的孤立作業復原。

每個非允許的終止狀態，其有效裁決皆為拒絕。

解析會使用一個即時 SQLite 交易，以及等同下列內容的 Kysely 條件式更新：

```sql
UPDATE operator_approvals
SET status = ?, decision = ?, terminal_reason = ?, resolved_at_ms = ?
WHERE approval_id = ?
  AND status = 'pending'
  AND expires_at_ms > ?;
```

若更新未影響任何資料列，同一交易會讀取該記錄：

- 不存在或未獲授權：傳回找不到；不可洩露其是否存在。
- 仍在待處理中但已到達期限：以比較後設定將其改為 `expired`，然後傳回該終止資料列。
- 與已記錄決定相同：傳回具等冪性的成功結果與已記錄的勝出者。
- 不同決定：統一 API 傳回含已記錄勝出者的 `applied: false`；舊版轉接器則在其已發布合約要求時保留 `APPROVAL_ALREADY_RESOLVED`。
- 任何終止狀態：絕不可變更。

`now == expires_at_ms` 已過期。閘道時間具有權威性。

`allow-once` 執行會對 `consumed_at_ms IS NULL` 使用第二次 CAS，並繫結至現有的確切命令／系統執行內容。核准資料列在使用後仍保留為稽核記錄。

無法驗證身分或識別核准項目的格式錯誤 HTTP/RPC 輸入會遭拒絕且不進行任何變更，並且絕不可能核准。針對已知核准項目，若從受信任的測試工具／等待器收到格式錯誤的終止裁決，則會轉換為 `denied`。

## 閘道 API

新增不限定種類的審查者方法：

| 方法                                    | 合約                                                                                                                                                                                                            |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `approval.get { id }`                     | 傳回可見的待處理投影或保留的終止投影。                                                                                                                                                          |
| `approval.resolve { id, kind, decision }` | 接受標準 ID 或固定大小的傳輸參照，接著執行授權、種類與允許決定驗證、期限協調及終止 CAS。回應一律包含標準 ID。 |

CAS 成功後，立即傳回已提交的投影。舊版事件、頻道轉送器與推送終止器皆為盡力而為的後續作業；緩慢或失敗的介面絕不可延遲或回復勝出回應。

種類特定的請求驗證仍保留在 `exec.approval.request` 與 `plugin.approval.request` 中。現有的 `exec.approval.get/list/waitDecision/resolve` 與 `plugin.approval.list/waitDecision/resolve` 會成為連至標準服務的通訊協定邊界轉接器，因為它們是已發布的閘道 API。內部呼叫端會在同一項變更中遷移至該服務。

審查者投影是帶標籤的聯集：

```ts
type OperatorApproval = {
  id: string;
  status: OperatorApprovalStatus;
  presentation:
    | { kind: "exec"; commandText: string /* 安全的 exec 預覽 */ }
    | { kind: "plugin"; title: string; description: string /* 安全的外掛預覽 */ };
  // 共用生命週期欄位
};
```

穩定路徑是衍生而來，不會持久儲存。`approval.get` 傳回 `urlPath`；知道已核准公開來源的介面也可以收到絕對 `url`。審查者快照不包含來源與對象工作階段金鑰。閘道會將這些路由金鑰保留在伺服器端，供獨立的 `session.approval` 投影使用。

## 事件與可攜式動作

PR 1 保留已發布的事件名稱、承載資料及現有的記錄層級接收者篩選器：

- `exec.approval.requested`
- `exec.approval.resolved`
- `plugin.approval.requested`
- `plugin.approval.resolved`

這些舊版事件可能包含完整的執行階段請求，因此不得將其扇出至所有核准範圍的用戶端。PR 5 會透過已清理的生命週期投影新增帶標籤的生命週期欄位（`status`、`sourceSessionKey`、`urlPath`、終止中繼資料，以及呈現層級的 `kind`），而不擴大舊版事件的傳遞範圍。

新增核准範圍的 `session.approval` 投影事件。使用持久儲存的對象金鑰發布一次標準事件；完全相符的工作階段訂閱者會針對每個相符金鑰收到相同事件：

- `sessionKey`：接收投影的串流。
- `sourceSessionKey`：引發關卡的子項／來源。
- `phase`：`pending \| terminal`，依核准狀態加以鑑別。
- 一個安全的 `OperatorApproval` 投影。

用戶端使用 `sessions.messages.subscribe { key, agentId?, includeApprovals: true }` 選擇加入。成功回應會新增一個 `approvalReplay`，其中最多包含 1,000 個該確切串流金鑰目前的待處理核准，且訂閱用戶端也須獲得記錄層級的審查授權。`truncated: false` 使篩選後的重播具有權威性，重新連線的用戶端會以其取代本機待處理集合；`truncated: true` 是過載訊號，用戶端必須保留尚未看見的本機項目，直到標準查詢或後續生命週期事件確定其狀態。重播期間發現的後續持久逾時，會先僅向已訂閱且獲得記錄層級授權的對象發出終止墓碑，再傳回新快照。`operator.admin` 可以直接選擇加入；範圍更窄的用戶端則同時需要配對的裝置身分與 `operator.approvals`。僅訂閱工作階段絕不會授予核准可見性。

在 `src/gateway/server-broadcast.ts` 中將事件註冊於 `operator.approvals` 下。此投影僅供觀察：它絕不附加逐字稿資料列、發出 `sessions.changed` 或喚醒代理程式。

擴充 `src/interactive/payload.ts` 中的 `MessagePresentationAction`：

```ts
type MessagePresentationAction =
  | { type: "command"; command: string }
  | { type: "callback"; value: string }
  | {
      type: "approval";
      approvalId: string;
      approvalKind: "exec" | "plugin";
      decision: ExecApprovalDecision;
    }
  | { type: "url"; url: string }
  | { type: "web-app"; url: string };
```

核心會建立具型別的決策動作，並在有已核准的 Control UI 絕對來源時提供獨立的審查連結。頻道會將核准動作編碼為各自的回呼格式，並將處理結果傳送至標準服務。若標準 ID 長度允許，回呼會使用其完整且精確的值；否則會使用該資料列唯一的完整摘要 `resolution_ref`。此參照僅是精簡的查詢鍵：一般的閘道驗證、記錄授權、明確種類、允許決策驗證、期限協調，以及首次回覆 CAS 仍然適用。頻道不得截斷 ID、解析雜湊前綴、剖析 `/approve` 文字，或從 ID 前綴推斷種類。

保留 `button.url`、`button.webApp`，以及以命令為基礎的核准控制項，作為已棄用的外掛 SDK 相容性輸入。在 SDK 邊界將其正規化；並在同一個 PR 中遷移所有隨附的內部呼叫端。`/approve {id} {decision}` 仍是文字備援與命令列介面／聊天命令，而不是按鈕的語意合約。

## Control UI

路由為 `${basePath}/approve/{approvalId}`。ID 是唯一的路徑參數；來源工作階段身分來自記錄。

由於目前的路由器具有完全比對的靜態路由，並會將未知路徑改寫至聊天，因此請在一般路由正規化之前，於 `ui/src/app/bootstrap.ts` 中偵測此深層連結。重複使用一般的閘道／驗證設定，但在側邊欄殼層與全域強制回應視窗之外呈現獨立的核准頁面。

文件由提供其 URL 的閘道擁有。其初始連線會忽略完整應用程式中持久保存的遠端閘道選擇，而不變更或複製該選擇的設定；只有驗證維持在提供服務的閘道工作階段範圍內。受信任的原生驗證或另外確認的 `gatewayUrl` 覆寫可將其重新定向。核心會在外掛 HTTP 路由與靜態擴充功能偵測之前，保留單區段的 `/approve` 命名空間，包括以 `.json` 或 `.js` 結尾的 ID；停用 Control UI 服務時，保留路由會以 `404` 安全關閉。請將頁面保留在主要 Control UI 套件組合中，以免延遲載入區塊失敗，導致安全性決策卡在載入指示器上。

頁面狀態：

- 載入中
- 需要驗證
- 待處理
- 處理中
- 已在此核准或拒絕
- 已在其他位置處理
- 已過期
- 已取消
- 禁止存取／找不到
- 連線錯誤，可重試

頁面會呼叫閘道 RPC，而不是第二個未經驗證的 REST API。瀏覽器重新整理時會重新讀取持久狀態。頁面絕不會將閘道認證資訊放入 URL、查詢或片段中。

## 授權與隱私權

URL 是定位器，而不是授權依據。處理需要：

1. 已驗證的閘道連線；
2. `operator.approvals` 或 `operator.admin`；
3. 記錄層級的審查者授權。

記錄層級規則：

- `operator.admin` 可以審查。
- 若有 `reviewer_device_ids`，則以其為準。只有列出的已配對
  `operator.approvals` 裝置可以審查；要求端裝置沒有隱含的
  存取權，除非它也在清單中。
- 若沒有明確的審查者清單，提出要求的已配對
  `operator.approvals` 裝置可以審查自己的記錄。
- 真正的舊版記錄若沒有要求者或審查者繫結，仍保留廣泛的
  已配對裝置可見性，以免升級使已在等待中的工作無法繼續。
- 沒有裝置的內部執行階段可以透過具範圍限制的
  核准執行階段連線進行處理，但不能讀取。該權限僅來自
  伺服器驗證的執行階段權杖；公開的 `approval.resolve` 欄位無法
  建立該權限。
- 即時要求者連線的擁有權對舊版轉接器仍然有效；絕不會
  根據相符的用戶端名稱推斷。
- 受眾成員資格只會改變呈現方式，絕不會擴大授權。

`approval.get` 僅公開經過清理的審查者投影，並省略內部來源／受眾路由鍵。PR 5 的 `session.approval` 事件會攜帶其單一目的地 `sessionKey`，以及閘道在伺服器端套用持久保存的受眾快照後產生的 `sourceSessionKey`。現有的 exec／外掛事件會保留其歷史承載內容與受限制的接收者，直到消費端完成遷移。可執行要求、命令繫結與接續內容只保留在程序本機的等待器中。持久資料列包含安全的呈現內容，以及生命週期、路由和稽核中繼資料；絕不會儲存原始環境值、認證資訊、驗證標頭或頻道回呼資料。

## 受眾投影

在插入之前計算一次受眾，並持久保存排序後的快照。擁有權是一張圖，不一定是單一父系鏈：子項目可能同時有目前控制者與原始要求者，而這些擁有者可能通往不同的根節點。

使用確定性的廣度優先走訪：

1. 以來源工作階段鍵作為佇列初始項目。
2. 對每個移出佇列的鍵，讀取最新的子代理程式登錄資料列，並依固定順序將兩條不同的擁有權邊加入佇列：先 `controllerSessionKey`，再 `requesterSessionKey`。
3. 若存在可用的登錄資料列，請勿同時追蹤可能在引導後過時的工作階段項目譜系。否則，將單一目前備援邊 `parentSessionKey ?? spawnedBy` 加入佇列。
4. 加入佇列時進行正規化與去除重複項目，使第一條最短路徑優先。
5. 到達 64 個唯一鍵時停止；此受眾大小上限也會限制走訪深度。

登錄來源為 `src/agents/subagent-registry-read.ts`；擁有權欄位定義於 `src/agents/subagent-registry.types.ts`。工作階段備援欄位定義於 `src/config/sessions/types.ts`。

即使核准等待期間焦點／控制者擁有權發生變更，要求與終止投影也會使用同一份持久保存的受眾。這可確保每個收到要求投影的受眾工作階段串流都能完成終止清理。處理永遠以來源核准 ID 為目標；受眾工作階段絕不會收到複製的核准狀態。轉送頻道訊息的清理仍是下方獨立的傳遞定位器後續工作。

不得僅因核准而寫入逐字稿訊息、插入系統提示、啟動擁有者回合，或發出 `sessions.changed`。

## 已傳遞介面收斂

原生核准處理常式已會保留其已傳遞訊息項目足夠長的時間，以便取代或停用作用中的控制項。一般轉送的核准訊息目前會捨棄 `MessageReceipt`，因此在其他介面上做出決策後，其舊控制項可能仍顯示為待處理。獨立的後續工作會在共用狀態資料庫中使用 `operator_approval_deliveries` 子資料表來補足此缺口。

每個資料列會儲存核准 ID、唯一傳遞 ID、頻道／帳戶／精確路由、經過有界 JSON 驗證的頻道私有訊息定位器、傳遞時間戳記，以及終止處理狀態。它絕不會儲存回呼資料、決策權杖或原始核准要求。頻道擁有定位器編碼與訊息變更；核心擁有標準狀態、目標選擇、重試原則與備援終止文字。

傳遞登錄與終止處理可安全地競爭：

1. 待處理傳送傳回收據後，在同一個交易中插入傳遞定位器並讀取上層核准狀態。
2. 若上層已是終止狀態，則排程立即終止處理，而不是讓延遲傳遞維持待處理。
3. 每次提交終止轉換時，都會另外排程所有尚未完成的傳遞資料列；可捨棄的廣播不是觸發條件。
4. 頻道終止處理器會回報 `replaced`、`retired` 或 `unsupported`。已取代會抑制重複的終止訊息；已停用會傳送現有的終止後續訊息；不支援或失敗時則會使用備援，而不回復核准 CAS。
5. 啟動時會重試具有未完成傳遞的終止核准，使清理能承受閘道重新啟動。

此傳輸生命週期是選用的傳遞轉接器掛鉤，不是呈現器，也不是面向模型的訊息動作。QQ C2C／群組訊息目前沒有編輯、刪除或清除鍵盤的 API；該轉接器仍不受支援，在傳輸層取得變更 API 之前，只能在之後點擊時顯示標準事實。

## 重新啟動、逾時與路由語意

SQLite 持久化並不代表執行可以恢復。命令／工具繫結仍保留在記憶體中，因為其中可能包含安全性敏感的執行階段事實，而且它們不是可恢復的工作合約。

閘道啟動時：

- 產生新的執行階段 epoch；
- 以不可分割方式將舊 epoch 的待處理資料列轉換為 `cancelled`，原因為 `gateway-restart`；
- 保留資料列，讓其 URL 說明發生的情況；
- 絕不在缺少執行階段繫結的情況下執行後續核准。

計時器是喚醒最佳化機制。期限依據儲存在 `expires_at_ms`；讀取、等待與處理都會執行到期協調。

最終嚴格行為：

- 逾時 -> `expired`，拒絕；
- 無路由 -> `denied`，拒絕；
- 執行中止 -> `cancelled`，拒絕；
- 格式錯誤的受信任判定 -> `denied`，拒絕；
- 只有允許的明確准許決策 -> `allowed`。

目前已發布的 exec 行為仍與此合約衝突：

- `src/agents/bash-tools.exec-host-shared.ts` 可能套用 `askFallback`。
- `docs/tools/exec-approvals.md` 與 `docs/cli/approvals.md` 記載了該介面。

外掛核准現在會在逾時與判定格式錯誤時安全關閉；舊版
`timeoutBehavior` 欄位仍可接受，但會被忽略。exec 嚴格語意的
後續工作必須一併更新程式碼、型別、文件、測試與變更日誌，並經過
明確的擁有者／安全性審查。`askFallback` 在遷移期間可以繼續描述
關卡前的原則選擇，但不得將已建立之待處理記錄的逾時轉為核准。

## 相容性計畫

- 新增式閘道通訊協定；不提高通訊協定版本。
- 在外部邊界保留現有的 exec／外掛方法與事件。
- 保留現有 ID，包括 `plugin:` 前綴，但停止使用前綴作為型別資訊。
- 保留 `/approve` 文字命令行為。
- 保留舊版按鈕 URL／Web App 欄位與命令動作，作為外掛 SDK 相容性輸入；新的核心輸出具有型別。
- 在同一個具型別動作變更中，遷移所有隨附頻道與內部呼叫端。
- 為新的 URL／頁面及後續的逾時行為變更新增變更日誌項目。
- 不要新增引導模式設定。

## 推出計畫

### PR 1：持久生命週期

- 本設計說明。
- 共用 SQLite 結構描述、Kysely 產生作業、儲存區，以及 30 天清除機制。
- 閘道核准服務、執行階段等待器橋接，以及重新啟動時的孤立項目處理。
- 統一的 `approval.get/resolve`。
- Exec／外掛方法轉接器。
- 首次回覆優先、冪等性、到期、授權與消耗測試。
- 目前尚無 UI 或頻道行為變更。

### PR 2：具型別動作與頻道回呼

- 具型別的核准、URL 與 Web App 動作。
- 核心呈現建構器與外掛 SDK 匯出項目。
- 具明確擁有者種類的傳輸層私有回呼編碼。
- 針對超出傳輸限制之標準 ID 的持久固定大小回呼參照。
- 內建通道遷移，不再推斷命令文字與核准 ID。
- 以點擊介面上的標準首次回覆為真實狀態，並盡力更新作用中原生介面的終止狀態；持久化通道訊息終止處理仍留待後續完成。
- SDK 與內建通道測試。

### PR 3：Control UI 深層連結

- 獨立的已驗證核准頁面，以及能感知基底路徑的啟動路由。
- 繫結至提供服務的閘道，而不變更操作者已儲存的遠端選擇。
- 由核心擁有的核准 HTTP 命名空間，包括類似資產的 ID。
- 由閘道編寫的 URL 承載資料，以及在生命週期事件推出前持續輪詢待處理狀態。
- 行動裝置寬度、重新連線、競爭回覆、重新載入及掛載路徑的驗證。

### PR 4：原生用戶端

- iOS 與 Android 審查介面使用可感知種類的 `approval.get/resolve`；watchOS 透過已配對的 iPhone 轉送對審查者安全的提示與決定。
- Watch 提供其精簡轉送合約所支援的 exec 決定：允許一次與拒絕。
- 以標準首次回覆的終止真實狀態取代本機嘗試決定狀態。
- 遺失或語意不明的解析確認會凍結控制項，直到讀回標準狀態。
- 先前已發布的閘道 v4 執行個體透過狹窄的舊版方法備援保留 exec 審查功能；若要保留跨介面的終止狀態，則需要統一方法。
- 審查者警告與擁有者情境資訊在 iPhone、Watch 與 Android 上皆保持可見。
- 原生單元、建置及平台驗證。

### PR 5：祖先生命週期傳播

- 從 PR 1 中持久保存的受眾快照傳遞 `session.approval` 待處理／終止狀態。
- 精確工作階段訂閱、重新連線重播及終止墓碑，不變更逐字稿，也不喚醒代理程式。
- 生命週期回呼在持久化插入／CAS 後執行，且絕不成為核准權威。
- 巢狀子代理程式與重新連線驗證。

### PR 6：失敗時關閉行為

- 將 `node-invoke-plugin-policy.ts` 與嵌入式外掛代理程式遷移，不再使用重複權威。
- 嚴格的逾時、格式錯誤、無路由、繫結及允許一次取用語意。
- 淘汰已發布的寬鬆逾時設定，且在詢問進入待處理狀態後不再遵循這些設定。
- 多介面競爭與故障注入驗證。

### 後續工作：持久化遠端訊息清理

- 持久保存轉送傳遞定位資訊，並在重新啟動後終止處理每則已傳遞的通道訊息。
- 讓此傳輸生命週期與標準核准權威及具型別呈現動作彼此分離。

## 測試

必要的聚焦涵蓋範圍：

- 重新開啟 SQLite 後仍保留待處理與終止投影。
- 兩個並行解析器只會產生一個 CAS 勝出者。
- 相同決定的重試會以等冪方式成功；衝突的重試則傳回已記錄的勝出者。
- 在截止時間當下或之後解析時無法核准。
- `allow-once` 只能取用一次，且不會清除終止稽核狀態。
- 啟動時會取消較舊的執行階段週期。
- 未經授權的查詢與解析不會揭露記錄是否存在。
- 明確的審查者允許清單，以及一般的已配對 `operator.approvals` 行為。
- Exec 與外掛舊版方法共用相同儲存區。
- 閘道 request/list/get/resolve 結構描述與附加式事件承載資料。
- 具型別動作正規化、備援呈現、SDK 匯出項目及內建通道切換。
- Telegram 回呼編碼包含傳輸層私有資料，且不推斷命令字串。
- 直接子項、分支控制器／請求者擁有者、巢狀擁有者、重新指派、工作階段欄位備援、循環及受眾大小上限。
- 請求與終止受眾陣列完全相同。
- 擁有者投影不會變更逐字稿或喚醒代理程式。
- Control UI 路由可在 `/` 與已設定的基底路徑運作；重新整理後會顯示待處理或終止真實狀態。
- Control UI 與 Telegram 同時回覆時會顯示一名勝出者，落敗方則顯示「已在其他位置解析」。
- 原生核准識別碼與閘道擁有者識別碼會在路由與協調過程中保留完全相同的 UTF-8 位元組。
- 原生 RPC 系列協商會為每個已准入的閘道路由固定使用一個標準或舊版系列，且使用後絕不會無聲降級。
- 遺失原生解析確認時會凍結動作，直到讀回標準狀態；讀回失敗時不得捏造勝出者，亦不得確認 Watch 重新整理。
- 只有針對完全相同的已配對閘道擁有者，且已完成標準 iPhone 讀回時，才接受 Watch 快照請求關聯。
- 透過 Testbox／Crabbox 驗證使用者路徑，包括行動裝置寬度的核准頁面、Telegram 動作清理，以及橫跨 Android、iPhone 與 Watch 的一次待處理／解析／逾時落敗者往返流程。

## 可觀測性

發出結構化且不含內容的轉換記錄，其中包含核准 ID、種類、來源工作階段金鑰、狀態、原因及延遲時間。絕不記錄預覽內容或原始繫結。

追蹤：

- 依種類統計的請求數量；
- 依種類／狀態／原因統計的終止數量；
- 待處理量表；
- 從請求至終止的延遲時間；
- 解析競爭結果：勝出者、等冪重試、衝突、已過期；
- 傳遞路由數量與無路由拒絕數量；
- 啟動時孤立項目取消數量；
- 受眾大小。

即使後續事件傳遞失敗，已提交的轉換仍視為成功。生命週期訂閱者會透過 PR 5 重播與標準查詢復原。持久化通道訊息終止處理仍是上述獨立的後續工作。

## 待決事項

1. **可從外部存取的 Control UI 來源。**每個快照都會攜帶穩定的相對 `urlPath`。只有在閘道公開成功後，才能從快取的 Tailscale Serve／Funnel 位置公告絕對 URL；`allowedOrigins`、請求 Host 標頭、`gateway.remote.url`，以及僅供顯示的回送／LAN 候選項目均非標準來源。Telegram 可使用其已驗證的 Mini App 包裝程式，透過啟動程序保留核准路徑。在另行審查的明確公開 URL 合約建立前，任意反向 Proxy 仍只能使用相對路徑。絕不允許通道猜測來源。
2. **Exec 嚴格逾時相容性切換。**外掛核准逾時現在會失敗時關閉，且 `timeoutBehavior` 已淘汰。剩餘已發布的 `askFallback` 合約在待處理詢問逾時後停止授權執行前，仍需明確的擁有者／安全性審查、變更記錄、文件，以及遷移／淘汰決策。
3. **無閘道嵌入模式。**建議：初期僅限本機使用，之後在閘道存在時使其成為標準服務的用戶端。請勿公告任何沒有伺服器可解析的深層連結。
