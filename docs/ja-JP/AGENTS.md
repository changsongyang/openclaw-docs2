---
x-i18n:
    generated_at: "2026-07-26T08:51:41Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: a8712b1aeb2e605055c22cf308049e5e74fdf33061870026be20bd55cb0c3d1d
    source_path: AGENTS.md
    workflow: 16
---

# ドキュメントガイド

このディレクトリは、ドキュメントの執筆、Mintlify のリンク規則、ドキュメントの i18n ポリシーを管理します。

## Mintlify の規則

- ドキュメントは Mintlify (`https://docs.openclaw.ai`) でホストされます。
- `docs/**/*.md` 内の内部ドキュメントリンクは、`.md` または `.mdx` のサフィックスを付けず、ルート相対のままにする必要があります（例: `[Config](/gateway/configuration)`）。
- セクション間の相互参照には、ルート相対パス上のアンカーを使用してください（例: `[Hooks](/gateway/configuration-reference#hooks)`）。
- Mintlify のアンカー生成では処理が不安定になるため、ドキュメントの見出しでは em ダッシュとアポストロフィを避けてください。
- README および GitHub でレンダリングされるその他のドキュメントでは、Mintlify の外部でもリンクが機能するよう、ドキュメントの絶対 URL を維持してください。
- ドキュメントの内容は汎用的なものにする必要があります。個人のデバイス名、ホスト名、ローカルパスは使用せず、`user@gateway-host` のようなプレースホルダーを使用してください。

## ドキュメント内容の規則

- ドキュメント、UI 文言、選択リストでは、セクションがランタイム順序または自動検出順序を明示的に説明している場合を除き、サービスやプロバイダーをアルファベット順に並べてください。
- バンドルされた Plugin の命名は、ルートの `AGENTS.md` にあるリポジトリ全体の Plugin 用語規則と一貫させてください。
- 生成されたドキュメントは手動で編集しないでください。`docs/plugins/reference/**`、`docs/plugins/reference.md`、`docs/plugins/plugin-inventory.md` は `pnpm plugins:inventory:gen` から、`docs/docs_map.md` は `pnpm docs:map:gen` から、`docs/maturity/**` は `pnpm maturity:render` から生成されます。

## 内部ドキュメント

- 長期的に使用する非公開の運用者向けドキュメントは、`~/Projects/manager/docs/` に配置してください。
- リポジトリ内のみで使用する内部のスクラッチ／ミラードキュメントは、無視対象の `docs/internal/` 以下に配置できます。
- `docs/internal/**` ページを `docs/docs.json` のナビゲーションに追加したり、公開ドキュメントからリンクしたりしないでください。
- 後からページが強制追加された場合でも、`scripts/docs-sync-publish.mjs` は公開 `openclaw/docs` 公開リポジトリから `docs/internal/**` を除外して削除します。
- 内部ドキュメントにはリポジトリパス、非公開アプリ名、1Password の項目名、ランブックを記載できますが、シークレット値は決して含めないでください。

## 成熟度スコアカードの編集

`taxonomy.yaml` と `qa/maturity-scores.yaml` がソース入力です。`docs/maturity/` 以下の生成された成熟度ドキュメントは投影であり、スコア、LTS、分類体系、QA プロファイル、またはエビデンステーブルを手動で編集しないでください。
`scripts/qa/render-maturity-docs.ts` が生成を管理します。コミット済みドキュメントを更新するには `pnpm maturity:render`、検証するには `pnpm maturity:check` を使用してください。
`.github/workflows/maturity-scorecard.yml` はアーティファクトのプレビューをレンダリングし、生成ドキュメントの PR を作成できます。`.github/workflows/openclaw-release-checks.yml` はリリース QA 用にこれをディスパッチします。
保守担当者がサニタイズ済みのコミット対象投影を明示的に求めない限り、決定論的な `qa-evidence.json.scorecard` データは GitHub Actions のアーティファクトに保持してください。
人手による上書きでは、PR でソース状態を変更し、その理由と公開済みまたは編集済みのエビデンスを説明する必要があります。

## ドキュメントの i18n

- 外国語のドキュメントはこのリポジトリでは管理されません。生成された公開出力は、別の `openclaw/docs` リポジトリに配置されます（ローカルでは `../openclaw-docs` としてクローンされることがよくあります）。
- ここでは `docs/<locale>/**` 以下にローカライズ済みドキュメントを追加または編集しないでください。
- このリポジトリの英語ドキュメントと用語集ファイルを正としてください。
- パイプライン: ここで英語ドキュメントを更新し、必要に応じて `docs/.i18n/glossary.<locale>.json` を更新した後、公開リポジトリの同期と `scripts/docs-i18n` を `openclaw/docs` で実行します。
- `scripts/docs-i18n` を再実行する前に、英語のまま維持するか固定訳を使用する必要がある新しい技術用語、ページタイトル、短いナビゲーションラベルを用語集に追加してください。
- `pnpm docs:check-i18n-glossary` は、変更された英語ドキュメントのタイトルと短い内部ドキュメントラベルを保護するガードです。
- 翻訳メモリは、公開リポジトリ内で生成される `docs/.i18n/*.tm.jsonl` ファイルに保存されます。
- `docs/.i18n/README.md` を参照してください。
