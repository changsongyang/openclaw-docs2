---
x-i18n:
    generated_at: "2026-07-26T09:07:26Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 90c6c85a837448f4e5ceccdccf73489db801ad502cbbb2f3eb04d6aff7e902f0
    source_path: plan/swarms.md
    workflow: 16
---

# Swarms — コードモードでのエージェントのファンアウトとオーケストレーション

ステータス: リリース済み — `docs/tools/swarm.md` により置き換えられました。このドキュメントは
実装設計の記録として残されています。

## 1. 概要と目的

**Swarm** は、コードモードのスクリプトから決定論的にオーケストレーションされる多数のサブエージェントです。
N 個の読み取り担当をファンアウトし、敵対的に調査結果を検証し、ステートフルな
優先順位付け機構を通じて統合し、判断ゲートでループします。制御フロー（`Promise.all`、
`while`、`if`）_そのもの_がオーケストレーションです。意図的に**グラフ DSL、
新しいモード、新しいトップレベルのツールサーフェスはありません**。

OpenClaw コードモード（QuickJS-WASI、スナップショット/再開、ブリッジリクエスト）が
基盤です。待機中のブリッジ呼び出しは VM スナップショットと Gateway の再起動をまたいで存続し、
停止した位置から正確に再開します。これはジャーナル再生設計より堅牢であり、
スクリプトに決定性の制約を課しません。

命名: 製品/ドキュメント上の名称は **Swarm** です。コード識別子はそのまま維持します:
`agents.*` ゲスト API、`tools.swarm` 設定、`swarm` グループ列。

## 2. 決定事項（メンテナー、2026-07-17）

- コスト: 設定上限を強制します。Swarm ごとのトークン予算は任意です。必須の予算はありません。
- 承認: 子は**フェイルクローズド/非対話型**で実行されます。承認が必要な
  アクションは拒否され、その拒否は子の結果で報告され、スクリプトが
  判断します。ファンアウトによってオペレーターへのプロンプトが大量発生することはありません。
- v1 はモデルが記述するアドホックなスクリプトのみに対応します。保存済み/名前付きワークフロー、CLI/Cron
  エントリは後で対応します（Cron 向けのヘッドレスコードモードはすでに存在します）。
- 子の ID: デフォルトでは `tools.swarm.defaultAgentId`
  設定による専用ワーカーエージェントを使用します（既存のサブエージェント対象許可リストに照らして検証）。
  スポーンごとに `agentId` で上書きできます。コアにはバンドル済みのエージェント ID は含まれません。
  ドキュメントでは軽量な `worker` エージェント設定を推奨します。
- Codex のソース変更はありません。Codex ハーネスは spawn/wait イディオムを使用します（§8）。

## 3. アーキテクチャの概要

```
コードモードスクリプト（QuickJS VM、Gateway）      Codex V8 スクリプト（codex プロセス）
  agents.run(...) ── 待機中のブリッジ呼び出し       tools.sessions_spawn / tools.agents_wait
        │                                                │ item/tool/call RPC（各 ≤600s）
        ▼                                                ▼
             コア（ハーネス非依存、このリポジトリ）
  sessions_spawn {collect:true, outputSchema, fastMode, groupId}
  agents_wait {ids, timeoutSeconds}
        │
  サブエージェントレジストリ（SQLite）: コレクター完了レコード、Swarm グループ ID
        │
  子 = 通常のサブエージェントセッション（レーン上限あり、承認はフェイルクローズド）
        │
  sessions.changed SSE ──► Control UI のドット / サイドバー / チャンネルのステータスメッセージ
```

スポーン/完了/確定のセマンティクスには、単一の正規オーナー（コアツール + レジストリ）があります。
待機トランスポートは 2 つです。QuickJS はブリッジ呼び出しを無期限に待機させ（スナップショット）、
Codex は有界 RPC で `agents_wait` をポーリングします。

## 4. 設定ゲート（v1）

新しい `tools.swarm`（グローバル + エージェントごとの上書き、`tools.codeMode` と
同じマージパターン）:

```jsonc
"tools": {
  "swarm": {
    "enabled": false,            // マスターゲート、デフォルトは OFF
    "maxConcurrent": 8,          // 同時に実行する子の数（Swarm レーン上限）
    "maxChildrenPerGroup": 50,   // Swarm グループごとの実行中の子の数
    "maxTotalPerGroup": 200,     // グループごとの生涯スポーン数（暴走防止の最終手段）
    "waitTimeoutSecondsMax": 600,
    "defaultAgentId": ""         // 任意。スポーンで agentId を省略した場合の子エージェント ID
  }
}
```

- Zod: `CodeModeSchema` のようなユニオン `boolean | strict object`
  （`src/config/zod-schema.agent-runtime.ts`）。`swarm: true` → `{enabled: true}`。
- `src/config/types.tools.ts` の型（エージェントごと、およびトップレベルの `tools` の両方）、
  `schema.labels.ts` のラベル、`schema.help.runtime.ts` のヘルプ。
- `resolveCodeModeConfig`（`src/agents/code-mode.ts:215`）を踏襲する
  解決ヘルパー `resolveSwarmConfig(cfg, agentId)`。すべての数値をクランプします。
- 無効時のゲート効果: `agents_wait` ツールはカタログに存在しません。
  `sessions_spawn` の `collect`/`outputSchema`/`fastMode`/`groupId` パラメーターは、
  設定キーを明記した明確なエラーで拒否されます。それ以外の動作変更はありません。
- `defaultAgentId` は `resolveSubagentAllowedTargetIds`
  （`src/agents/subagent-target-policy.ts`）を通じて検証されます。不明な ID → フォールバックではなくスポーンエラー。

## 5. コア: コレクターモードのスポーン + `agents_wait`（v1）

### 5.1 `sessions_spawn` の追加（すべて Swarm 有効時のみ）

- `collect: boolean` — true の場合、子の実行は
  アナウンス/ステアリング配信の代わりに、`expectsCompletionMessage: false` と**コレクター完了レコード**を使用して
  登録されます。ツールは `{ runId, sessionKey }` を即座に返します。
  チャンネル/スレッドにはバインドされません。
- `outputSchema: object` — JSON Schema。子のツールサーフェスに合成
  `structured_output` ツールが追加されます。システムプロンプトの追記により、
  最終結果を指定してこのツールを正確に 1 回呼び出すよう指示します。検証に失敗した場合、
  子には再試行を促す通知が 1 回送られます。その後、完了レコードには
  `structured: undefined`、生テキスト、および `schemaError` が格納されます。
- `fastMode: true | "auto" | false` — 既存の `FastMode` 軸
  （`src/shared/fast-mode.ts`）を使用し、`resolveSubagentModelAndThinkingPlan`
  （`src/agents/subagent-spawn-plan.ts`）を介して、model/thinking とともに子セッションのパッチへ渡されます。
  省略時は継承します。
- `groupId: string` — Swarm グループのスタンプ。デフォルトは
  `swarm:<requesterSessionKey>:<runId-of-requesting-run>` です。レジストリレコードおよび子セッション行に
  永続化されます。上限、一覧表示、一括アーカイブ、ドットに使用されます。
- `label: string` はすでに存在し、ドットと `subagents list` に表示されます。
- 子エージェント ID: `params.agentId` → なければ `tools.swarm.defaultAgentId` → なければ
  リクエスターのエージェント（既存の動作）。

### 5.2 承認はフェイルクローズド

コレクターの子は非対話型の承認コンテキストで実行されます。オペレーターの承認が
必要となるツール呼び出しはすべて、子から確認できる構造化された拒否
（`approval_required`）として解決され、子はその阻害要因を結果で報告することが
想定されています。実装: コレクターモードの子の実行に強制的な `deny` リゾルバーを指定し、
既存の exec/tool 承認ポリシーの配管を再利用します。
コレクターの子からオペレーター向けサーフェスへ承認イベントは送出されません。

### 5.3 `agents_wait` ツール（新規、ゲート対象）

```
agents_wait({ ids: string[], timeoutSeconds?: number })
→ {
    completed: [{ runId, status: "done"|"failed"|"killed"|"timeout",
                  result: string, structured?: unknown, schemaError?: string,
                  sessionKey, label?, usage?: {inputTokens, outputTokens} }],
    pending: string[]
  }
```

- 少なくとも 1 つの ID が完了するとすぐに返ります（最初の完了/競合
  セマンティクスによりパイプラインが可能）。またはタイムアウト時に `completed: []` とともに返ります。
- `timeoutSeconds` のデフォルトは 30 で、`waitTimeoutSecondsMax` にクランプされます。
- 冪等: すでに完了した ID は、そのレコードを再度返します（レコードは
  グループがアーカイブされるまで保持されます）。不明な ID → throw ではなく、ID ごとのエラーエントリ。
- 所有権: 実行をスポーンしたセッション（またはその親チェーン）のみが、その実行を待機できます。
  コードモードの `wait` と同じ所有権ルールです（`code-mode.ts:1684`）。
- レジストリ: 完了レコードは既存のサブエージェントレジストリ SQLite
  ストア（`subagent-registry.store.sqlite.ts`）に格納されます。新しいフィールドのみで、
  新しいストアやスキーマバージョンの引き上げはありません（追加列のみ。§9 の制約を参照）。

### 5.4 上限の適用

- `maxConcurrent`: コレクターの子は既存のサブエージェントレーンで実行されますが、
  Swarm グループごとにカウントされます。上限を超えたスポーンは FIFO でキューに入ります
  （ホスト側のスポーンパス内。runId は即座に返され、スロットが空くと実行が開始されます）。
- `maxChildrenPerGroup` / `maxTotalPerGroup`: 上限を超えるとスポーンは型付きエラーで
  拒否されます。エラーテキストには設定キーが明記されます。
- 深度: コレクターの子は `DEFAULT_SUBAGENT_MAX_SPAWN_DEPTH` セマンティクスを維持します
  （ネストが明示的に設定されていない限り、子はリーフになります）。

## 6. テスト契約（v1、レーン A）

- 単体: 設定の解決/クランプ。無効時のゲート拒否。groupId
  のデフォルト設定。上限の適用（キュー + 拒否）。wait の競合セマンティクス。wait
  の冪等性。所有権による拒否。構造化出力の検証 + 再試行の促し +
  schemaError パス。セッションパッチへの fastMode の受け渡し。defaultAgentId
  の検証。
- 統合（vitest、モックモデルランタイム）: コレクターの子を 3 つスポーンし、ループ内で
  待機して、最初の完了順序と最終的な全件処理をアサートします。Gateway 再起動の
  シミュレーション: レジストリ再読み込み → 永続化された完了情報から wait が解決します。
- すべてのテストは `*.test.ts` に併置します。ライブモデル呼び出しはありません。

## 7. QuickJS ゲストサーフェス（レーン B、コアの後）

- ゲストグローバルは `CONTROLLER_SOURCE`
  （`src/agents/code-mode.worker.ts:190-374`）にインストールされ、予約名は
  `code-mode-namespaces.ts` に追加されます:
  - `agents.run(prompt, opts) → Promise<result|structured>` — シュガー:
    コレクタースポーン + 専用ブリッジメソッド（`agentWait`）での待機中の await。
    ホストは完了時にこれを確定します（ポーリングなし、スナップショットセーフ）。
  - `agents.session(system, opts) → Promise<handle>`;
    `handle.send(input, opts) → Promise<...>`; `handle.close()`。（v1.1 —
    run() の後にリリースされ、`mode:"session"` + ターンごとのコレクターレコードを使用します）。
  - `phase(title)`、`log(message)` — 応答を待たないブリッジ通知 →
    Swarm 進行状況イベント。
- `CodeModeBridgeMethod`（`code-mode.ts:91`）に追加されるブリッジメソッド:
  `agentSpawn`、`agentWait`、`swarmNote`。`agentSpawn`/`agentWait` は
  **構造上**リプレイセーフです。冪等性キー `(codeModeRunId, bridgeId)` は
  レジストリレコードに保存されます。再起動時には永続化された完了情報から再確定され、
  二重にスポーンされることはありません。
- 保留中の `agentWait` ブリッジ呼び出しは、実行のスナップショット TTL を延長します
  （保留中のエージェントセットがシグナルであり、フラグはありません）。
- `API.read("agents.d.ts")` 仮想ファイルには、型付きサーフェスと
  ファンアウト/ゲート/サイクルのイディオム（`createCodeModeApiVirtualFiles`、
  `code-mode-namespaces.ts:876`）が記載されます。

## 8. Codex ハーネスへの投影（後続レーン）

- `sessions_spawn`（新しいパラメーターを含む）と `agents_wait` は、
  既存の動的ツールブリッジを通過します。Codex のコードモードスクリプト内では、
  自動的に `tools.*` として表示されます（検証済み: `codex-rs/code-mode/src/runtime/globals.rs:14-65`、
  `codex-rs/core/src/tools/spec_plan.rs:448-507`）。
- `agents_wait` には長時間の動的ツールタイムアウトクラス（上限 600s、
  `extensions/codex/src/app-server/dynamic-tool-execution.ts:37-39`）が適用され、
  タイムアウト/リプレイセーフとしてマークされます。
- Codex の親のグループキー: `swarm:<parentSessionKey>:<turnId>`。
- Codex ネイティブの `spawn_agent` サブエージェントは共存し、そのタスクミラー行は
  同じ進行状況サーフェスに反映されます。

## 9. 永続化と保持

- 新しいストアはありません。レジストリレコードは既存のサブエージェントレジストリ
  SQLite テーブルを拡張し、子は通常の `sessions` 行です。追加列のみです。
  **SQLite スキーマバージョンの引き上げを必要とする変更には、事前にメンテナーの明示的な
  承認が必要です**（リポジトリポリシー）。
- レジストリレコード + 子セッションメタデータ上の Swarm グループ ID。
- 保持: 完了したコレクターレコードは**グループアーカイブ**まで存続します。
  親の実行が完了すると（または TTL が期限切れになると）、グループの子は一括で
  アーカイブされます（既存の `DEFAULT_SUBAGENT_ARCHIVE_AFTER_MINUTES`
  スイープをグループ単位で動作するよう拡張します）。

## 10. 進行状況サーフェス（「ドット」）— 後続レーン

- 暗黙的で、ハーネスによって駆動されます。既存の `sessions.changed` SSE +
  レジストリから導出され、`phase`/`log` の注記でセマンティクスを追加します。
  エージェントによるレンダリングはありません。
- Control UI: ワークスペースウィジェットファミリー
  （`ui/src/lib/workspace/widgets/`）の `swarm` レンダラー。フェーズ別にグループ化された
  ドットグリッド、ナレーター行、ドットごとのステータス/ラベル/モデル。サイドバーの子ツリーは変更しません。
- チャンネル: グループごとに、スロットリングされた編集済みステータスメッセージを 1 件使用します
  （`docs/concepts/streaming.md` に従い、子ごとのメッセージは送信しません）。

## 11. Labs ページ（Control UI、独立レーン）

Settings → **Labs**：実験的機能の切り替え。最初の項目は **Code Mode**
と **Swarm**。各行には、名前、1 行の説明、ドキュメントへのリンク、既存の
`config.patch` RPC（RFC 7396 merge-patch — 
`tools.codeMode.enabled` / `tools.swarm.enabled` を設定）経由で接続されたトグルに加え、該当する場合は
「再起動が必要」というヒントを表示する。見つけやすくしつつ、文言によって実験的な状態であることを
明確にする。i18n：すべての文字列を通常の `en.ts` + 同期パイプライン経由にする。

## 12. 配置（後日）

- `placement` の生成時オプション：`"local"`（デフォルト）| 既存の
  ワーカー環境ディスパッチ（`sessions.dispatch`）経由の `"cloud:<profile>"`。共有ボックスの SSH サンドボックス子プロセスでは
  不十分であることが判明した場合、プール配置を後日導入する。
- オーケストレーター VM は常に Gateway 上に残り、settle/dots/budget は
  配置を意識しない。

## 13. 対象外

- グラフ DSL は導入しない — 制御フロー自体がグラフである（意図的であり、文書化する）。
- Codex のソースは変更せず、Codex Code Mode の内部実装も再利用しない。
- v1 では保存済み／名前付きワークフローを導入せず、CLI エントリーポイントも設けない。
- 子プロセスごとのオペレーター承認を上位へ伝播させない。
- ファンアウト規模での 1:1 クラウドプロビジョニングは行わない。
- 定常実行時の互換シムは導入しない。swarm は新しいサーフェスであり、ゲートで制御する。

## 14. ビルドフェーズ／PR の分割

1. **レーン A（コア）**：§4 の設定 + §5 の生成／待機／上限／承認 + §6 のテスト。
2. **レーン C（Labs ページ）**：§11 — 独立しており、先にマージ可能。
3. **レーン B（QuickJS サーフェス）**：§7 — A の契約がマージされた後。
4. dots レンダラー（§10）、Codex プロジェクション（§8）、`agents.session`（§7 v1.1）、
   配置（§12）、ユーザードキュメントの書き直し — この順序で後続 PR とする。

各 PR：CI がグリーン、`$autoreview` がクリーン、デフォルトではゲート無効、main はリリース可能。
