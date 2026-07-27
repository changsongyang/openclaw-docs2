---
read_when:
    - clawdbot-d63.2 / clawdbot-04b uygulamasını gerçekleştiriyorsunuz
    - SQLite oturum saklama, sıfırlama, silme veya ajan silme arşivlemesini değiştiriyorsunuz
    - SQLite dönemi yapıt ailelerini eski JSONL yan dosyalarından ayırt etmeniz gerekir
summary: Bir oturuma ait tüm SQLite transkript yapıtlarını arşivlemek için Yol 3 planı
title: Yol 3 SQLite oturum yapıtı ailesi
x-i18n:
    generated_at: "2026-07-26T22:51:21Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 29f4d541b2df5a06468fd0cee620b4340ee33eea1064f7d3ee823580c7b5760e
    source_path: plan/path3-sqlite-session-artifact-family.md
    workflow: 16
---

# Yol 3 SQLite Oturum Yapıtı Ailesi

Bu not, `clawdbot-d63.1` öğesi `src/config/sessions/session-accessor.sqlite.ts` içindeki çakışan
sıfırlama/silme arşiv yardımcısının sahibi olurken `clawdbot-d63.2` kapsamını belirler.
Bu geçiş sırasında uygulama dosyasında kaydedilmemiş değişiklikler vardı; bu nedenle bu yapıt,
kardeş çalışanla yarışa girmeden tam sözleşmeyi ve yama noktalarını kaydeder.

## Yetkili aile

SQLite geçişinden sonra etkin oturum dökümleri SQLite satırlarıdır. Bir oturumun
arşiv ailesi şunlardan oluşur:

- Girdinin geçerli `sessionId` değeri için `transcript_events`,
  `transcript_event_identities` ve `sessions` satırları.
- `entry.compactionCheckpoints[*].preCompaction.sessionId` tarafından başvurulan her `sessionId` için
  aynı SQLite döküm satırı kümesi.
- `entry.compactionCheckpoints[*].postCompaction.sessionId` tarafından başvurulan her `sessionId` için
  aynı SQLite döküm satırı kümesi.
- `entry.usageFamilySessionIds` içindeki her `sessionId` için
  aynı SQLite döküm satırı kümesi.

Yalnızca kalan herhangi bir `session_entries` satırı veya kalan herhangi bir girdinin
Compaction ya da kullanım ailesi meta verileri tarafından artık başvurulmayan satırları
arşivleyin. Bu, son canlı başvuru ortadan kalkana kadar denetim noktası dalını/geri yüklemeyi
ve kullanım toplama durumunu korur.

## Geçişten sonraki aile dışı yapıtlar

Oluşturulan konu döküm dosyası varyantları ve yörünge yardımcı dosyaları etkin
SQLite çalışma zamanı durumu değildir. Bunlar eski dosya yapıtlarıdır:

- `<sessionId>-topic-<thread>.jsonl` gibi konu varyantları yalnızca dosya
  destekli döküm biçimi için bulunur. SQLite, konu başına JSONL dosyaları yerine
  kanonik oturum kimliğini ve `session_routes`/girdi teslim meta verilerini kullanır.
- `.trajectory.jsonl` ve `.trajectory-path.json` gibi yörünge yardımcı dosyaları,
  gerçek JSONL `sessionFile` yollarından adlandırılır. SQLite `sessionFile`
  değerleri `sqlite:<agentId>:<sessionId>:<storePath>` işaretçileridir ve yardımcı dosyaları
  adlandırmaz.
- Arşiv katmanı okuyucuları eski arşivlenmiş JSONL dosyalarını okumaya
  devam etmelidir; ancak çalışma zamanı saklama işlemi etkin oturum dizinlerini taramamalı
  veya SQLite oturumları için JSONL döküm dosyalarını yeniden açmamalıdır.

Doctor içe aktarma işlemi, eski birincil JSONL dosyaları ve bunların bitişiğindeki
yörünge yardımcı dosyaları için geçiş sahibi olmaya devam eder. Çalışma zamanı SQLite
saklama işlemi ikinci bir içe aktarıcı veya dosya geri dönüşü eklememelidir.

## Yama noktaları

Paralel bir yol eklemek yerine `clawdbot-d63.1` tarafından sunulan SQLite arşiv
yardımcısını genişletin.

1. `deleteSqliteSessionStateIfUnreferenced` yakınına yerel bir toplayıcı ekleyin:
   - `collectSqliteSessionArtifactFamily(entry: SessionEntry): Set<string>`
   - `entry.sessionId`, denetim noktası öncesi/sonrası oturum kimlikleri ve
     `usageFamilySessionIds` öğesini dahil edin.
   - Boş dizeleri filtreleyin ve yinelenenleri belirlenimsel olarak kaldırın.

2. Kaldırma sonrası depo için bir başvuru toplayıcı ekleyin:
   - `readReferencedSqliteSessionArtifactFamilyIds(database): Set<string>`
   - Geçerli `session_entries` üzerinde yineleyin, her `entry_json`
     öğesini ayrıştırın ve ayakta kalan her girdiden aynı aile kimliklerini toplayın.

3. Şu anda kaldırılan tek bir `sessionId` öğesini arşivleyen
   sıfırlama/silme/bakım çağırıcılarını, kaldırılan girdinin tam ailesini geçirecek şekilde değiştirin.

4. Her aile kimliği için SQLite döküm satırlarını çağırıcının gerekçesiyle
   (`reset` veya `deleted`) arşivleyin; ardından `sessions`
   satırını yalnızca aile kimliği kaldırma sonrası başvuru kümesinde yoksa silin.

5. Döküm olayı silme işlemini mevcut SQLite oturum satırı temizleme yolu
   üzerinden merkezî tutun. Etkin JSONL okumaları eklemeyin.

## Odaklanmış testler

`clawdbot-d63.1` kaydedildikten sonra `src/config/sessions/session-accessor.conformance.test.ts`
veya kardeş yaşam döngüsü testine yalnızca SQLite'a yönelik testler ekleyin:

- Compaction öncesi dökümü bulunan bir girdinin silinmesi hem geçerli
  oturumu hem de Compaction öncesi oturumu arşivler, ardından her iki SQLite satır kümesini kaldırır.
- Bir Compaction öncesi oturumunu paylaşan iki girdiden birinin silinmesi,
  son başvuran girdi kaldırılana kadar paylaşılan ön oturum için hiçbir şeyi arşivlemez.
- `usageFamilySessionIds` içeren bir girdinin silinmesi, başka hiçbir girdi
  bu kullanım ailesine başvurmuyorsa öncül SQLite döküm satırlarını arşivler.
- SQLite işaretçisi içeren konu biçimli bir oturum anahtarı, oluşturulmuş
  herhangi bir konu JSONL okumasına veya yardımcı dosya aramasına neden olmaz.

Odaklanmış doğrulamada şunu kullanın:

```bash
node scripts/run-vitest.mjs src/config/sessions/session-accessor.conformance.test.ts
```

Bu Codex çalışma ağacı için geniş `pnpm` geçitleri Crabbox/Testbox üzerinde kalmalıdır.
