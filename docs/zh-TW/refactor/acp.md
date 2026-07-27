---
read_when:
    - 重構 ACP 工作階段生命週期或 ACPX 程序清理
    - 偵錯 ACPX 孤兒程序、PID 重複使用或多閘道清理安全性
    - 變更衍生的 ACP 或子代理工作階段的 sessions_list 可見性
    - 設計背景工作、ACP 工作階段或程序租約的擁有權中繼資料
sidebarTitle: ACP lifecycle refactor
summary: 明確化 ACP 工作階段與 ACPX 程序擁有權的遷移計畫
title: ACP 生命週期重構
x-i18n:
    generated_at: "2026-07-26T08:47:01Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: bda66f0acc93216c3d9386ca3ebf7f544efd306cd7f53386391f0c48e5dc8f06
    source_path: refactor/acp.md
    workflow: 16
---

ACP 生命週期目前可以運作，但其中太多內容是在事後推斷。
程序清理會從 PID、命令字串、包裝器
路徑和即時程序表重建擁有權。工作階段可見性則會從工作階段金鑰字串加上次要 `sessions.list({ spawnedBy })` 查詢重建擁有權。
這讓範圍狹窄的修正成為可能，但也很容易遺漏邊界情況：
PID 重複使用、加上引號的命令、介接器的孫程序、多閘道狀態根目錄、
`cancel` 與 `close`，以及 `tree` 與 `all` 的可見性，都變成必須各自
重新找出相同擁有權規則的不同位置。

這項重構讓擁有權成為一級概念。目標不是新的 ACP 產品
介面，而是為現有 ACP 與 ACPX 行為提供更安全的內部契約。

## 目標

- 除非目前的即時證據與 OpenClaw 擁有的租約相符，否則清理絕不向程序傳送訊號。
- `cancel`、`close` 和啟動時收割具有不同的生命週期意圖。
- `sessions_list`、`sessions_history`、`sessions_send` 和狀態檢查使用
  相同的請求者擁有工作階段模型。
- 多閘道安裝無法收割彼此的 ACPX 包裝器。
- 舊 ACPX 工作階段記錄在移轉期間仍可運作。
- 執行階段仍由外掛擁有；核心不會得知 ACPX 套件的詳細資訊。

## 非目標

- 取代 ACPX 或變更公開的 `/acp` 命令介面。
- 將供應商特定的 ACP 介接器行為移入核心。
- 要求使用者在升級前手動清理狀態。
- 讓 `cancel` 關閉可重複使用的 ACP 工作階段。

## 目標模型

### 閘道執行個體身分

每個閘道程序都應具有穩定的執行階段執行個體 ID：

```ts
type GatewayInstanceId = string;
```

它可以在閘道啟動時產生，並於該安裝的存續期間保存於狀態中。
它不是安全機密；而是用來避免將某個閘道的 ACP 程序
與另一個閘道的程序混淆的擁有權辨別資訊。

### ACP 工作階段擁有權

每個產生的 ACP 工作階段都應具有正規化的擁有權中繼資料：

```ts
type AcpSessionOwner = {
  sessionKey: string;
  spawnedBy?: string;
  parentSessionKey?: string;
  ownerSessionKey: string;
  agentId: string;
  backend: "acpx";
  gatewayInstanceId: GatewayInstanceId;
  createdAt: number;
};
```

閘道應在已知這些欄位的工作階段資料列中回傳它們。
可見性篩選應是對資料列中繼資料的純粹檢查：

```ts
canSeeSessionRow({
  row,
  requesterSessionKey,
  visibility,
  a2aPolicy,
});
```

這可移除可見性檢查中隱藏的次要 `sessions.list({ spawnedBy })` 呼叫。
產生的跨代理程式 ACP 子工作階段之所以由請求者擁有，是因為
資料列如此記載，而不是因為第二次查詢碰巧找到它。

### ACPX 程序租約

每次產生的包裝器啟動都應建立租約記錄：

```ts
type AcpxProcessLease = {
  leaseId: string;
  gatewayInstanceId: GatewayInstanceId;
  sessionKey: string;
  wrapperRoot: string;
  wrapperPath: string;
  rootPid: number;
  processGroupId?: number;
  commandHash: string;
  startedAt: number;
  state: "open" | "closing" | "closed" | "lost";
};
```

包裝器程序會以可攜式引數接收租約 ID 和閘道執行個體 ID：

```sh
--openclaw-acpx-lease-id ... --openclaw-gateway-instance-id ...
```

當平台允許時，驗證應優先採用不會因命令引號而混淆的
即時程序中繼資料：

- 根 PID 仍然存在
- 即時包裝器路徑位於 `wrapperRoot` 之下
- 程序群組在可用時與租約相符
- 引數包含預期的租約 ID
- 命令雜湊或可執行檔路徑與租約相符

如果無法驗證即時程序，清理會採取失敗即關閉。

## 生命週期控制器

引入單一 ACPX 生命週期控制器，以擁有程序租約與清理
政策：

```ts
interface AcpxLifecycleController {
  ensureSession(input: AcpRuntimeEnsureInput): Promise<AcpRuntimeHandle>;
  cancelTurn(handle: AcpRuntimeHandle): Promise<void>;
  closeSession(input: {
    handle: AcpRuntimeHandle;
    discardPersistentState?: boolean;
    reason?: string;
  }): Promise<void>;
  reapStartupOrphans(): Promise<void>;
  verifyOwnedTree(lease: AcpxProcessLease): Promise<OwnedProcessTree | null>;
}
```

`cancelTurn` 僅請求取消當前回合。它不得收割可重複使用的包裝器
或介接器程序。

`closeSession` 可以收割，但前提是先載入工作階段記錄、
載入租約，並驗證即時程序樹仍屬於該租約。

`reapStartupOrphans` 從狀態中的開放租約開始。它可以使用程序
表尋找子孫程序，但不應先掃描任意看似 ACP 的
命令，再判定它們可能屬於我們。

## 包裝器契約

產生的包裝器應保持精簡。它們應：

- 在支援的情況下，於程序群組中啟動介接器
- 將正常終止訊號轉送至程序群組
- 偵測父程序死亡
- 父程序死亡時傳送 SIGTERM，然後讓包裝器保持運作，直到 SIGKILL
  後備機制執行
- 在可用時，將根 PID 和程序群組 ID 回報給生命週期控制器

包裝器不應決定工作階段政策。它們只對自己的介接器群組
強制執行本機程序樹清理。

## 工作階段可見性契約

可見性應使用正規化的資料列擁有權：

```ts
type SessionVisibilityInput = {
  requesterSessionKey: string;
  row: {
    key: string;
    agentId: string;
    ownerSessionKey?: string;
    spawnedBy?: string;
    parentSessionKey?: string;
  };
  visibility: "self" | "tree" | "agent" | "all";
  a2aPolicy: AgentToAgentPolicy;
};
```

規則：

- `self`：僅限請求者工作階段。
- `tree`：請求者工作階段，加上由請求者擁有或產生的資料列。
- `all`：所有相同代理程式的資料列、a2a 允許的跨代理程式資料列，以及即使一般 a2a 已停用，仍由請求者擁有
  且產生的跨代理程式資料列。
- `agent`：僅限相同代理程式，除非明確的擁有者關係指出該資料列
  屬於請求者。

這讓 `tree` 和 `all` 保持單調性：`all` 不得隱藏
`tree` 會顯示的已擁有子工作階段。

## 移轉計畫

### 第 1 階段：新增身分與租約

- 將 `gatewayInstanceId` 新增至閘道狀態。
- 在 ACPX 狀態目錄下新增 ACPX 租約儲存區。
- 在產生生成的包裝器前寫入租約。
- 在新的 ACPX 工作階段記錄中儲存 `leaseId`。
- 為舊記錄保留現有 PID 和命令欄位。

### 第 2 階段：租約優先清理

- 將關閉清理改為先載入 `leaseId`。
- 傳送訊號前，依租約驗證即時程序擁有權。
- 僅為舊版記錄保留目前的根 PID 和包裝器根目錄後備機制。
- 完成已驗證的清理後，將租約標記為 `closed`。
- 當程序在清理前已消失時，將租約標記為 `lost`。

### 第 3 階段：租約優先的啟動時收割

- 啟動時收割會掃描開放租約。
- 針對每個租約驗證根程序並收集子孫程序。
- 以子程序優先順序收割已驗證的程序樹。
- 使用有界的保留期間，使舊 `closed` 和 `lost` 租約到期。
- 僅將命令標記掃描保留為暫時的舊版後備機制，並在可行時以
  包裝器根目錄和閘道執行個體加以防護。

### 第 4 階段：工作階段擁有權資料列

- 將擁有權中繼資料新增至閘道工作階段資料列。
- 讓 ACPX、子代理程式、背景工作和工作階段儲存區寫入器填入
  `ownerSessionKey` 或 `spawnedBy`。
- 將工作階段可見性檢查轉換為使用資料列中繼資料。
- 移除可見性判定時的次要 `sessions.list({ spawnedBy })` 查詢。

### 第 5 階段：移除舊版啟發式方法

經過一個版本週期後：

- 非舊版 ACPX 清理不再依賴儲存的根命令字串
- 移除啟動時的命令標記掃描
- 移除可見性後備清單查詢
- 對缺少或無法驗證的租約，保留防禦性的失敗即關閉行為

## 測試

新增兩套表格驅動的測試套件。

程序生命週期模擬器：

- PID 被不相關的程序重複使用
- PID 被另一個閘道的包裝器根程序重複使用
- 儲存的包裝器命令已加上 shell 引號，即時 `ps` 命令則沒有
- 介接器子程序結束，但孫程序仍留在程序群組中
- 父程序死亡時的 SIGTERM 後備機制會進一步執行 SIGKILL
- 程序清單無法使用
- 程序遺失的過時租約
- 具有包裝器、介接器子程序和孫程序的啟動孤兒程序

工作階段可見性矩陣：

- `self`、`tree`、`agent`、`all`
- a2a 啟用與停用
- 相同代理程式的資料列
- 跨代理程式資料列
- 由請求者擁有且產生的跨代理程式 ACP 資料列
- 沙箱化請求者受限於 `tree`
- 列出、歷程記錄、傳送和狀態動作

重要的不變條件：只要設定的可見性包含請求者工作階段樹，
由請求者擁有且產生的子工作階段就會可見，而且 `all` 的能力
不得低於 `tree`。

## 相容性注意事項

舊工作階段記錄可能沒有 `leaseId`。它們應使用舊版的
失敗即關閉清理路徑：

- 要求存在即時根程序
- 預期使用產生的包裝器時，要求包裝器根目錄擁有權
- 對非包裝器根程序要求命令一致
- 絕不僅根據過時的已儲存 PID 中繼資料傳送訊號

如果無法驗證舊版記錄，請不要處理它。啟動時租約清理和
下一個版本週期最終應淘汰此後備機制。

## 成功標準

- 關閉舊的或過時的 ACPX 工作階段時，不得終止另一個閘道的程序。
- 父程序死亡後，不會留下難以終止且仍在執行的介接器孫程序。
- `cancel` 會中止進行中的回合，但不關閉可重複使用的工作階段。
- `sessions_list` 可在 `tree` 和 `all` 下顯示
  由請求者擁有的跨代理程式 ACP 子工作階段。
- 啟動時清理由租約驅動，而非廣泛的命令字串掃描。
- 聚焦的程序與可見性矩陣測試涵蓋先前需要逐一審查修正的
  每個邊界情況。
