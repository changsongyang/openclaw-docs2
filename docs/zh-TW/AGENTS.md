---
x-i18n:
    generated_at: "2026-07-26T07:07:35Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: a8712b1aeb2e605055c22cf308049e5e74fdf33061870026be20bd55cb0c3d1d
    source_path: AGENTS.md
    workflow: 16
---

# 文件指南

此目錄負責文件撰寫、Mintlify 連結規則與文件國際化政策。

## Mintlify 規則

- 文件託管於 Mintlify（`https://docs.openclaw.ai`）。
- `docs/**/*.md` 中的內部文件連結必須維持根目錄相對路徑，且不得包含 `.md` 或 `.mdx` 後綴（範例：`[Config](/gateway/configuration)`）。
- 章節交叉參照應在根目錄相對路徑上使用錨點（範例：`[Hooks](/gateway/configuration-reference#hooks)`）。
- 文件標題應避免使用長破折號和撇號，因為 Mintlify 的錨點產生機制對此較不穩定。
- README 和其他由 GitHub 轉譯的文件應保留絕對文件 URL，讓連結在 Mintlify 之外也能正常運作。
- 文件內容必須維持通用性：不得包含個人裝置名稱、主機名稱或本機路徑；請使用 `user@gateway-host` 之類的預留位置。

## 文件內容規則

- 對於文件、UI 文案和選擇器清單，除非該章節明確描述執行階段順序或自動偵測順序，否則服務／供應商應依字母順序排列。
- 隨附外掛的命名應與根目錄 `AGENTS.md` 中全儲存庫通用的外掛術語規則保持一致。
- 產生的文件絕不可手動編輯：`docs/plugins/reference/**`、`docs/plugins/reference.md` 和 `docs/plugins/plugin-inventory.md` 來自 `pnpm plugins:inventory:gen`；`docs/docs_map.md` 來自 `pnpm docs:map:gen`；`docs/maturity/**` 來自 `pnpm maturity:render`。

## 內部文件

- 長期使用的私人操作人員文件應放在 `~/Projects/manager/docs/`。
- 儲存庫本機的內部暫存／鏡像文件可以放在已忽略的 `docs/internal/` 下。
- 絕不可將 `docs/internal/**` 頁面加入 `docs/docs.json` 導覽，也不可從公開文件連結至這些頁面。
- 如果之後強制加入頁面，`scripts/docs-sync-publish.mjs` 會從公開的 `openclaw/docs` 發布儲存庫排除並移除 `docs/internal/**`。
- 內部文件可以提及儲存庫路徑、私人應用程式名稱、1Password 項目名稱和操作手冊，但絕不可包含祕密值。

## 成熟度計分卡編輯

`taxonomy.yaml` 和 `qa/maturity-scores.yaml` 是來源輸入；`docs/maturity/` 下產生的成熟度文件是投影，不應為了分數、LTS、分類法、QA 設定檔或證據表格而手動編輯。
`scripts/qa/render-maturity-docs.ts` 負責產生作業；使用 `pnpm maturity:render` 重新整理已提交的文件，並使用 `pnpm maturity:check` 驗證這些文件。
`.github/workflows/maturity-scorecard.yml` 會轉譯成品預覽，並可開啟產生文件的 PR；`.github/workflows/openclaw-release-checks.yml` 會為發布 QA 分派該工作。
除非維護者明確要求提交經過清理的投影，否則請將具決定性的 `qa-evidence.json.scorecard` 資料保留在 GitHub Actions 成品中。
人工覆寫必須在 PR 中變更來源狀態，並說明原因及提供公開或經遮蔽處理的證據。

## 文件國際化

- 本儲存庫不維護外語文件。產生的發布輸出位於獨立的 `openclaw/docs` 儲存庫中（本機通常會將其複製為 `../openclaw-docs`）。
- 請勿在此新增或編輯 `docs/<locale>/**` 下的本地化文件。
- 請將本儲存庫中的英文文件及詞彙表檔案視為單一真實來源。
- 流水線：在此更新英文文件、視需要更新 `docs/.i18n/glossary.<locale>.json`，接著讓發布儲存庫同步，並在 `openclaw/docs` 中執行 `scripts/docs-i18n`。
- 重新執行 `scripts/docs-i18n` 前，請為任何必須保留英文或使用固定翻譯的新技術術語、頁面標題或簡短導覽標籤新增詞彙表項目。
- `pnpm docs:check-i18n-glossary` 是已變更英文文件標題與簡短內部文件標籤的防護機制。
- 翻譯記憶位於發布儲存庫中產生的 `docs/.i18n/*.tm.jsonl` 檔案。
- 請參閱 `docs/.i18n/README.md`。
