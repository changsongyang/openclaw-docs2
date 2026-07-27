---
read_when:
    - clawdbot-d63.2 / clawdbot-04b を実装しています
    - SQLite のセッション保持、リセット、削除、またはエージェント削除時のアーカイブ処理を変更している場合
    - SQLite 時代のアーティファクト群とレガシー JSONL サイドカーを区別する必要があります
summary: セッションに属するすべての SQLite トランスクリプト成果物をアーカイブするためのパス 3 の計画
title: パス 3 SQLite セッションアーティファクトファミリー
x-i18n:
    generated_at: "2026-07-26T09:08:41Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 29f4d541b2df5a06468fd0cee620b4340ee33eea1064f7d3ee823580c7b5760e
    source_path: plan/path3-sqlite-session-artifact-family.md
    workflow: 16
---

# パス 3 SQLite セッションアーティファクトファミリー

このメモでは `clawdbot-d63.2` のスコープを定めます。一方、`clawdbot-d63.1` は
`src/config/sessions/session-accessor.sqlite.ts` 内の重複するリセット／削除アーカイブヘルパーを担当します。
この作業中、実装ファイルには未コミットの変更があったため、このアーティファクトでは
並行作業中のワーカーと競合しないよう、正確な契約とパッチ箇所を記録します。

## 正式なファミリー

SQLite への切り替え後、アクティブなセッショントランスクリプトは SQLite の行です。セッションの
アーカイブファミリーは次のとおりです。

- エントリの現在の `sessionId` に対応する `transcript_events`、
  `transcript_event_identities`、および `sessions` の行。
- `entry.compactionCheckpoints[*].preCompaction.sessionId` が参照するすべての `sessionId` に対応する、
  同じ SQLite トランスクリプト行セット。
- `entry.compactionCheckpoints[*].postCompaction.sessionId` が参照するすべての `sessionId` に対応する、
  同じ SQLite トランスクリプト行セット。
- `entry.usageFamilySessionIds` 内のすべての `sessionId` に対応する、
  同じ SQLite トランスクリプト行セット。

残っているどの `session_entries` 行からも、また残っているどのエントリの Compaction または
使用量ファミリーのメタデータからも参照されなくなった行のみをアーカイブします。これにより、
最後の有効な参照がなくなるまで、チェックポイントの分岐／復元および使用量ロールアップの状態が
保持されます。

## 切り替え後の非ファミリーアーティファクト

生成されたトピックトランスクリプトファイルのバリアントとトラジェクトリサイドカーは、アクティブな
SQLite ランタイム状態ではありません。これらはレガシーファイルアーティファクトです。

- `<sessionId>-topic-<thread>.jsonl` などのトピックバリアントは、ファイルベースの
  トランスクリプト形式にのみ存在します。SQLite では、トピックごとの JSONL ファイルの代わりに、
  正規のセッション ID と `session_routes`／エントリ配信メタデータを使用します。
- `.trajectory.jsonl` や `.trajectory-path.json` などのトラジェクトリサイドカーは、
  実際の JSONL `sessionFile` パスに基づいて命名されます。SQLite の `sessionFile` 値は
  `sqlite:<agentId>:<sessionId>:<storePath>` マーカーであり、サイドカーファイルを
  指定するものではありません。
- アーカイブ層のリーダーは、引き続きレガシーのアーカイブ済み JSONL ファイルを
  読み取る必要がありますが、ランタイム保持処理では、アクティブなセッションディレクトリを
  スキャンしたり、SQLite セッションの JSONL トランスクリプトファイルを再度開いたりしてはなりません。

Doctor インポートは、レガシーのプライマリ JSONL ファイルと、それらに隣接するトラジェクトリサイドカーの
移行を引き続き担当します。ランタイムの SQLite 保持処理に、2 つ目のインポーターや
ファイルフォールバックを追加してはなりません。

## パッチ箇所

並行するパスを追加するのではなく、`clawdbot-d63.1` によって導入された SQLite アーカイブヘルパーを
拡張します。

1. `deleteSqliteSessionStateIfUnreferenced` の近くにローカルコレクターを追加します。
   - `collectSqliteSessionArtifactFamily(entry: SessionEntry): Set<string>`
   - `entry.sessionId`、チェックポイント前後のセッション ID、および
     `usageFamilySessionIds` を含めます。
   - 空文字列を除外し、決定論的に重複を排除します。

2. 削除後のストア用の参照コレクターを追加します。
   - `readReferencedSqliteSessionArtifactFamilyIds(database): Set<string>`
   - 現在の `session_entries` を反復処理し、各 `entry_json` を解析して、
     残っているすべてのエントリから同じファミリー ID を収集します。

3. 現在、削除された 1 つの `sessionId` をアーカイブしている
   リセット／削除／メンテナンスの呼び出し元を変更し、削除されたエントリの完全なファミリーを渡します。

4. ファミリー ID ごとに、呼び出し元の理由（`reset` または
   `deleted`）を指定して SQLite トランスクリプト行をアーカイブし、そのファミリー ID が
   削除後の参照セットに存在しない場合にのみ `sessions` 行を削除します。

5. トランスクリプトイベントの削除は、既存の SQLite セッション行クリーンアップパスに
   集約したままにします。アクティブな JSONL の読み取りを追加してはなりません。

## 対象を絞ったテスト

`src/config/sessions/session-accessor.conformance.test.ts`、または `clawdbot-d63.1` のコミット後に
並行するライフサイクルテストへ、SQLite 専用テストを追加します。

- Compaction 前のトランスクリプトを持つエントリを削除すると、現在のセッションと
  Compaction 前のセッションの両方がアーカイブされ、その後、両方の SQLite 行セットが削除されます。
- Compaction 前のセッションを共有する 2 つのエントリのうち一方を削除しても、
  最後に参照しているエントリが削除されるまでは、共有されている事前セッションは
  一切アーカイブされません。
- `usageFamilySessionIds` を持つエントリを削除すると、他のエントリがその使用量ファミリーを
  参照していない場合、先行セッションの SQLite トランスクリプト行がアーカイブされます。
- SQLite マーカーを持つトピック形式のセッションキーによって、生成された
  トピック JSONL の読み取りやサイドカーの検索が行われることはありません。

対象を絞った検証では、次を使用します。

```bash
node scripts/run-vitest.mjs src/config/sessions/session-accessor.conformance.test.ts
```

この Codex ワークツリーでは、広範な `pnpm` ゲートは Crabbox/Testbox 上で実行します。
