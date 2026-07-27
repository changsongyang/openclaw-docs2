---
summary: /plugins/sdk-channel-outbound へリダイレクト
title: チャンネルメッセージ API
x-i18n:
    generated_at: "2026-07-26T09:38:37Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: bf0d607bd3287233cbb1fe47c15958bf57a81267ae1e37e45a1881f56e1370cb
    source_path: plugins/sdk-channel-message.md
    workflow: 16
---

このページは[チャネル送信 API](/ja-JP/plugins/sdk-channel-outbound)に移動しました。

`openclaw/plugin-sdk/channel-message` は、古い plugins 向けの非推奨の互換性
サブパスとして残されています。新しいチャネル plugins では、非推奨の
サブパスに新しいヘルパーを追加するのではなく、メッセージのライフサイクル、受信確認、
永続的な送信、ライブプレビューのヘルパーに `openclaw/plugin-sdk/channel-outbound` を使用してください。

削除計画: 外部 plugin の移行期間中はこれらのエイリアスを維持し、
呼び出し元が `channel-outbound` に移行した後、次回の SDK のメジャークリーンアップで
削除します。
