---
read_when:
    - 開発用 Gateway テンプレートの使用
    - デフォルトの開発エージェント ID の更新
summary: 開発エージェントツールのメモ（C-3PO）
title: TOOLS.dev テンプレート
x-i18n:
    generated_at: "2026-07-26T09:45:10Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 3259107a9252ff3d01b98608e6005387cb54a75da5db64f833c945056abd4173
    source_path: reference/templates/TOOLS.dev.md
    workflow: 16
---

# TOOLS.md - ユーザーツールに関するメモ（編集可能）

このファイルは、外部ツールや規則についての_あなた自身の_メモ用です。どのツールが存在するかを定義するものではありません。OpenClaw は内部で組み込みツールを提供し、残りは Skills によって追加されます。

## 例

### imsg

- iMessage/SMS を送信する場合：送信相手と内容を記述し、送信前に確認します。
- 短いメッセージを推奨します。シークレットの送信は避けてください。

### sag

- テキスト読み上げ：音声、出力先のスピーカーまたは部屋、ストリーミングの有無を指定します。

ローカルツールチェーンについてアシスタントに知らせたいことがあれば、自由に追加してください。

## 関連項目

- [TOOLS.md テンプレート](/ja-JP/reference/templates/TOOLS)
