---
x-i18n:
    generated_at: "2026-07-26T22:34:00Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: a8712b1aeb2e605055c22cf308049e5e74fdf33061870026be20bd55cb0c3d1d
    source_path: AGENTS.md
    workflow: 16
---

# Dokümantasyon Kılavuzu

Bu dizin, dokümantasyon yazımını, Mintlify bağlantı kurallarını ve dokümantasyon i18n politikasını yönetir.

## Mintlify Kuralları

- Dokümantasyon Mintlify üzerinde barındırılır (`https://docs.openclaw.ai`).
- `docs/**/*.md` içindeki dahili dokümantasyon bağlantıları, `.md` veya `.mdx` son eki olmadan köke göreli kalmalıdır (örnek: `[Config](/gateway/configuration)`).
- Bölümler arası başvurularda, köke göreli yollardaki bağlantı çapaları kullanılmalıdır (örnek: `[Hooks](/gateway/configuration-reference#hooks)`).
- Mintlify bağlantı çapası oluşturma işlemi bunlarda kırılgan olduğundan, dokümantasyon başlıklarında uzun tirelerden ve kesme işaretlerinden kaçınılmalıdır.
- README ve GitHub tarafından işlenen diğer dokümanlarda, bağlantıların Mintlify dışında da çalışması için mutlak dokümantasyon URL'leri korunmalıdır.
- Dokümantasyon içeriği genel kalmalıdır: kişisel cihaz adları, ana makine adları veya yerel yollar kullanılmamalı; `user@gateway-host` gibi yer tutucular kullanılmalıdır.

## Dokümantasyon İçeriği Kuralları

- Dokümantasyonda, kullanıcı arayüzü metinlerinde ve seçici listelerinde, bölüm açıkça çalışma zamanı sırasını veya otomatik algılama sırasını açıklamıyorsa hizmetleri/sağlayıcıları alfabetik olarak sıralayın.
- Birlikte sunulan plugin adlandırmasını, kök `AGENTS.md` içindeki depo genelindeki plugin terminolojisi kurallarıyla tutarlı tutun.
- Oluşturulan dokümanları asla elle düzenlemeyin: `docs/plugins/reference/**`, `docs/plugins/reference.md` ve `docs/plugins/plugin-inventory.md`, `pnpm plugins:inventory:gen` kaynağından; `docs/docs_map.md`, `pnpm docs:map:gen` kaynağından; `docs/maturity/**` ise `pnpm maturity:render` kaynağından gelir.

## Dahili Dokümantasyon

- Uzun ömürlü özel operatör dokümantasyonu `~/Projects/manager/docs/` içinde yer almalıdır.
- Depoya özel dahili taslak/yansıtma dokümanları, yok sayılan `docs/internal/` altında bulunabilir.
- `docs/internal/**` sayfalarını asla `docs/docs.json` gezinmesine eklemeyin veya genel dokümantasyondan bu sayfalara bağlantı vermeyin.
- Bir sayfa daha sonra zorla eklenirse `scripts/docs-sync-publish.mjs`, `docs/internal/**` içeriğini genel `openclaw/docs` yayımlama deposundan hariç tutar ve temizler.
- Dahili dokümanlarda depo yollarından, özel uygulama adlarından, 1Password öğe adlarından ve çalışma talimatlarından söz edilebilir ancak asla gizli değerler bulunmamalıdır.

## Olgunluk Puan Kartını Düzenleme

`taxonomy.yaml` ve `qa/maturity-scores.yaml` kaynak girdilerdir; `docs/maturity/` altındaki oluşturulan olgunluk dokümanları izdüşümlerdir ve puan, LTS, sınıflandırma, QA profili veya kanıt tabloları için elle düzenlenmemelidir.
`scripts/qa/render-maturity-docs.ts` oluşturma işlemini yönetir; işlenen dokümanları yenilemek için `pnpm maturity:render`, doğrulamak için ise `pnpm maturity:check` kullanın.
`.github/workflows/maturity-scorecard.yml` yapıt önizlemelerini işler ve oluşturulan dokümanlar için PR'lar açabilir; `.github/workflows/openclaw-release-checks.yml` ise sürüm QA'sı için bunu tetikler.
Bir bakım sorumlusu açıkça temizlenmiş ve depoya işlenmiş bir izdüşüm istemediği sürece, belirlenimci `qa-evidence.json.scorecard` verilerini GitHub Actions yapıtlarında tutun.
İnsan müdahaleleri, kaynak durumunu bir PR içinde değiştirmeli ve gerekçeyi, ayrıca genel veya hassas kısımları çıkarılmış kanıtları açıklamalıdır.

## Dokümantasyon i18n

- Yabancı dildeki dokümantasyon bu depoda sürdürülmez. Oluşturulan yayımlama çıktısı ayrı `openclaw/docs` deposunda bulunur (genellikle yerel olarak `../openclaw-docs` adıyla klonlanır).
- Burada `docs/<locale>/**` altında yerelleştirilmiş doküman eklemeyin veya düzenlemeyin.
- Bu depodaki İngilizce dokümanları ve sözlük dosyalarını doğruluk kaynağı olarak kabul edin.
- İşlem hattı: İngilizce dokümanları burada güncelleyin, gerektiğinde `docs/.i18n/glossary.<locale>.json` dosyasını güncelleyin, ardından yayımlama deposu eşitlemesinin ve `scripts/docs-i18n` işleminin `openclaw/docs` içinde çalışmasına izin verin.
- `scripts/docs-i18n` işlemini yeniden çalıştırmadan önce, İngilizce kalması veya sabit bir çeviri kullanması gereken tüm yeni teknik terimler, sayfa başlıkları veya kısa gezinme etiketleri için sözlük girdileri ekleyin.
- `pnpm docs:check-i18n-glossary`, değiştirilen İngilizce dokümantasyon başlıkları ve kısa dahili dokümantasyon etiketleri için koruyucudur.
- Çeviri belleği, yayımlama deposundaki oluşturulan `docs/.i18n/*.tm.jsonl` dosyalarında bulunur.
- `docs/.i18n/README.md` bölümüne bakın.
