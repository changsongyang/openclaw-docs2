---
read_when:
    - OpenClaw çalışma zamanı verilerini, önbelleği, transkriptleri, görev durumunu veya geçici dosyaları SQLite'a taşıma
    - Eski JSON veya JSONL dosyalarından doctor geçişleri tasarlama
    - Yedekleme, geri yükleme, VFS veya worker depolama davranışını değiştirme
    - Oturum kilitlerini, budama, kesme veya JSON uyumluluk yollarını kaldırma
summary: Yapılandırmayı dosya tabanlı tutarken SQLite'ı birincil kalıcı durum ve önbellek katmanı hâline getirme geçiş planı
title: Veritabanı öncelikli durum yeniden düzenlemesi
x-i18n:
    generated_at: "2026-07-26T23:00:13Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: ae4d72f04c1228742cc7ea3cc87a96b06aa1e2b750ace23cca5b387844746186
    source_path: refactor/database-first.md
    workflow: 16
---

# Veritabanı Öncelikli Durum Yeniden Düzenlemesi

## Karar

İki düzeyli bir SQLite düzeni kullanın:

- Genel veritabanı: `~/.openclaw/state/openclaw.sqlite`
- Ajan veritabanı: ajana ait çalışma alanı,
  transkript, VFS, yapıt ve büyük ajan başına çalışma zamanı durumu için ajan başına bir SQLite veritabanı
- Yapılandırma dosya destekli kalır: `openclaw.json`
  veritabanının dışında kalır. Çalışma zamanı kimlik doğrulama profilleri SQLite'a taşınır; harici sağlayıcı veya CLI
  kimlik bilgisi dosyaları OpenClaw'ın veritabanı dışında sahipleri tarafından yönetilmeye devam eder.

Genel veritabanı, kontrol düzlemi veritabanıdır. Ajan keşfini,
paylaşılan Gateway durumunu, eşleştirmeyi, cihaz/Node durumunu, görev ve akış kayıtlarını, Plugin
durumunu, zamanlayıcı çalışma zamanı durumunu, yedekleme meta verilerini ve geçiş durumunu barındırır.

Ajan veritabanı, veri düzlemi veritabanıdır. Ajanın oturum
meta verilerini, transkript olay akışını, VFS çalışma alanını veya geçici ad alanını, araç
yapıtlarını, çalıştırma yapıtlarını ve aranabilir/dizinlenebilir ajana özgü önbellek verilerini barındırır.

Bu, büyük ajan çalışma alanlarını,
transkriptleri ve ikili geçici verileri paylaşılan Gateway yazma hattına zorlamadan tek bir kalıcı genel görünüm sağlar.

## Kesin Sözleşme

Bu geçişin tek bir kanonik çalışma zamanı biçimi vardır:

- Oturum satırları yalnızca oturum meta verilerini kalıcılaştırır.
  `transcriptLocator`, transkript dosya yollarını, ilişkili JSONL yollarını, kilit yollarını,
  budama meta verilerini veya dosya dönemi uyumluluk işaretçilerini kalıcılaştırmamalıdır.
- Transkript kimliği her zaman SQLite kimliğidir: `{agentId, sessionId}` ve
  protokolün gerektirdiği durumlarda isteğe bağlı konu meta verileri.
- `sqlite-transcript://...` bir çalışma zamanı veya protokol kimliği değildir. Yeni kod
  transkript konumlandırıcılarını türetmemeli, kalıcılaştırmamalı, aktarmamalı, ayrıştırmamalı veya taşımamalıdır. Çalışma zamanı ve
  testler hiçbir şekilde sözde konumlandırıcı içermemelidir; belgeler bu dizeden
  yalnızca yasaklamak amacıyla söz edebilir.
- Eski `sessions.json`, transkript JSONL'si, `.jsonl.lock`, budama, kırpma
  ve eski oturum yolu mantığı yalnızca doctor geçiş/içe aktarma yoluna aittir.
- Eski oturum yapılandırma takma adları yalnızca doctor geçişine aittir. Çalışma zamanı
  `session.idleMinutes`, `session.resetByType.dm` veya
  yapılandırılmış başka bir ajan için ajanlar arası `agent:main:*` ana oturum takma adlarını yorumlamaz.
- Oturum yönlendirme kimliği, türü belirlenmiş ilişkisel durumdur. Yoğun kullanılan çalışma zamanı ve kullanıcı arayüzü yolları
  `sessions.session_scope`, `sessions.account_id`,
  `sessions.primary_conversation_id`, `conversations` ve
  `session_conversations` değerlerini okumalıdır; eski çağrı noktaları silinirken geçici bir uyumluluk
  gölgesi olması dışında `session_key` değerini ayrıştırmamalı veya sağlayıcı kimliği için
  `session_entries.entry_json` içinde arama yapmamalıdır.
- `dm` ile `direct` gibi kanal düzeyindeki doğrudan mesaj işaretçileri, yönlendirme
  söz dağarcığıdır; transkript konumlandırıcısı veya dosya deposu uyumluluk tanıtıcısı değildir.
- Eski kanca işleyicisi yapılandırması yalnızca doctor uyarı/geçiş yüzeylerine aittir.
  Çalışma zamanı `hooks.internal.handlers` değerini yüklememelidir; kancalar yalnızca keşfedilen
  kanca dizinleri ve `HOOK.md` meta verileri üzerinden çalışır.
- Çalışma zamanı başlatma, yoğun kullanılan yanıt yolları, Compaction, sıfırlama, kurtarma, tanılama,
  TTS, bellek kancaları, alt ajanlar, Plugin komut yönlendirmesi, protokol sınırları ve
  kancalar `{agentId, sessionId}` değerini çalışma zamanı boyunca aktarmalıdır.
- Testler, SQLite transkript satırlarını
  `{agentId, sessionId}` üzerinden başlangıç verileriyle doldurmalı ve doğrulamalıdır. Yalnızca JSONL yolu iletimini,
  çağıran tarafından sağlanan konumlandırıcının korunmasını veya transkript dosyası uyumluluğunu kanıtlayan testler;
  doctor içe aktarmasını, oturum dışı destek/hata ayıklama
  somutlaştırmasını veya protokol biçimini kapsamıyorsa silinmelidir.
- `runEmbeddedPiAgent(...)`, hazırlanmış çalışan çalıştırmaları ve içteki gömülü
  deneme transkript konumlandırıcılarını kabul etmemelidir. SQLite transkript
  yöneticisini `{agentId, sessionId}` ile açar ve bu yöneticiyi içselleştirilmiş
  PI uyumlu ajan oturumuna aktarır; böylece eski çağıranlar çalıştırıcının
  JSON/JSONL transkriptleri yazmasına neden olamaz.
- Çalıştırıcı tanılamaları, çalışma zamanı/önbellek/yük izleme kayıtlarını SQLite'ta saklamalıdır.
  Çalışma zamanı tanılamaları JSONL dosyası geçersiz kılma ayarları veya genel
  transkript JSONL dışa aktarma yardımcıları sunmamalıdır; kullanıcıya yönelik dışa aktarmalar, dosya
  adlarını yeniden çalışma zamanına beslemeden veritabanı satırlarından açık
  yapıtlar oluşturabilir.
- Ham akış günlük kaydı, `OPENCLAW_RAW_STREAM=1` ve SQLite tanılama satırlarını kullanır.
  Eski pi-mono `PI_RAW_STREAM`, `PI_RAW_STREAM_PATH` ve
  `raw-openai-completions.jsonl` dosya günlükleyicisi sözleşmesi OpenClaw
  çalışma zamanının veya testlerinin parçası değildir.
- QMD bellek dizinleme, SQLite transkriptlerini markdown dosyalarına aktarmamalıdır.
  QMD yalnızca yapılandırılmış bellek dosyalarını dizinler; oturum transkripti araması
  SQLite destekli kalır.
- QMD SDK alt yolu, yeni kod için yalnızca QMD'ye özeldir. SQLite oturum transkripti
  dizinleme yardımcıları `memory-core-host-engine-session-transcripts` üzerinde bulunur; herhangi bir
  QMD yeniden dışa aktarımı yalnızca uyumluluk içindir ve çalışma zamanı kodu tarafından kullanılmamalıdır.
- Yerleşik bellek dizinleri, sahibi olan ajan veritabanında bulunur. Çalışma zamanı yapılandırması ve
  çözümlenmiş çalışma zamanı sözleşmeleri `memorySearch.store.path` değerini sunmamalıdır; doctor
  bu eski yapılandırma anahtarını siler ve mevcut kod ajan
  `databasePath` değerini dahili olarak aktarır.

Uygulama çalışması, bu ifadeler doctor/içe aktarma/dışa aktarma/hata ayıklama sınırları
dışında istisnasız doğru olana kadar kod silmeye devam etmelidir.

## Hedef durum ve ilerleme

### Kesin hedef

- Kontrol düzlemi durumunu tek bir genel SQLite veritabanı barındırır:
  `state/openclaw.sqlite`.
- Veri düzlemi durumunu ajan başına tek bir SQLite veritabanı barındırır:
  `agents/<agentId>/agent/openclaw-agent.sqlite`.
- Yapılandırma dosya destekli kalır. `openclaw.json` bu veritabanı
  yeniden düzenlemesinin parçası değildir.
- Eski dosyalar yalnızca doctor geçiş girdileridir.
- Çalışma zamanı, etkin durum olarak oturum veya transkript JSONL'sini hiçbir zaman yazmaz veya okumaz.

### Hedef durumlar

- `not-started`: dosya dönemi çalışma zamanı kodu hâlâ etkin durum yazar.
- `migrating`: doctor/içe aktarma kodu dosya verilerini SQLite'a taşıyabilir.
- `dual-read`: geçici köprü hem SQLite'ı hem de eski dosyaları okur. Yalnızca
  doctor'a özgü olduğu açıkça belgelenmedikçe bu durum, bu yeniden düzenleme için
  yasaktır.
- `sqlite-runtime`: çalışma zamanı yalnızca SQLite'ı okur ve yazar.
- `clean`: eski çalışma zamanı API'leri ve testleri kaldırılır ve koruma
  gerilemeleri önler.
- `done`: belgeler, testler, yedekleme, doctor geçişi ve değişiklik denetimleri
  temiz durumu kanıtlar.

### Mevcut durum

- Oturumlar: çalışma zamanı için `clean`. Oturum satırları ajan başına veritabanında bulunur,
  çalışma zamanı API'leri `{agentId, sessionId}` veya `{agentId, sessionKey}` kullanır ve
  `sessions.json` yalnızca doctor'a özgü eski girdidir.
- Transkriptler: çalışma zamanı için `clean`. Transkript olayları, kimlikleri, anlık görüntüleri
  ve yörünge çalışma zamanı olayları ajan başına veritabanında bulunur. Çalışma zamanı artık
  transkript konumlandırıcılarını veya JSONL transkript yollarını kabul etmez.
- PI gömülü çalıştırıcı: `clean`. Gömülü PI çalıştırmaları, hazırlanmış çalışanlar, Compaction
  ve yeniden deneme döngüleri SQLite oturum kapsamını kullanır ve eski transkript tanıtıcılarını reddeder.
- Cron: çalışma zamanı için `clean`. Çalışma zamanı `cron_jobs` ve Cron'a ait `task_runs` kullanır;
  çalışma zamanı testleri SQLite `storeKey` adlandırmasını kullanır ve dosya dönemi Cron yolları yalnızca
  doctor eski geçiş testlerinde kalır.
- Görev kayıt defteri: `clean`. Görev ve Görev Akışı çalışma zamanı satırları
  `state/openclaw.sqlite` içinde bulunur; yayımlanmamış yan dosya SQLite içe aktarıcıları silinmiştir.
- Plugin durumu: `clean`. Plugin durumu/blob satırları paylaşılan genel
  veritabanında bulunur; eski Plugin durumu yan dosyası SQLite yardımcılarına karşı koruma uygulanır.
- Bellek: yerleşik bellek ve oturum transkripti dizinleme için `sqlite-runtime`.
  Bellek dizini tabloları ajan başına veritabanında bulunur, Plugin bellek durumu
  paylaşılan Plugin durumu satırlarını kullanır ve eski bellek dosyaları doctor geçiş girdileri
  veya kullanıcı çalışma alanı içeriğidir.
- Yedekleme: `sqlite-runtime`. Yedekleme, sıkıştırılmış SQLite anlık görüntülerini hazırlar, canlı
  WAL/SHM yan dosyalarını hariç tutar, SQLite bütünlüğünü doğrular ve yedekleme çalıştırmalarını
  genel veritabanına kaydeder.
- Çalışma alanı kurulumu: `sqlite-runtime`. Kurulum tamamlanması, çalışma alanı tasdikleri
  ve oluşturulan önyükleme karmaları, türü belirlenmiş paylaşılan SQLite tablolarında bulunur. Çalışma zamanı
  kullanımdan kaldırılmış çalışma alanı JSON'unu ve `.attested` yan dosyalarını okumaz veya yazmaz;
  doğrulanmış içe aktarma ve doğrulanmış kaldırma işlemlerini Doctor yürütür.
- Doctor geçişi: kasıtlı olarak `migrating`. Doctor eski JSON,
  JSONL ve kullanımdan kaldırılmış yan dosya depolarını SQLite'a aktarır, geçiş çalıştırmalarını/kaynaklarını
  kaydeder ve başarıyla aktarılan kaynakları kaldırır.
- Çalıştırma onayları: `file-runtime`. TypeScript ve macOS hâlâ
  etkin durum dizininin `exec-approvals.json` değerini okur ve yazar; ayrılmış
  `exec_approvals_config` şemasının henüz bir çalışma zamanı sahibi yoktur. Gelecekteki bir geçiş
  aynı durum için doctor içe aktarmasını eklemeli ve her iki çalışma zamanını birlikte taşımalıdır.
- E2E betikleri: çalışma zamanı kapsamı için `clean`. Docker MCP başlangıç verisi oluşturma işlemi SQLite
  satırları yazar. Çalışma zamanı bağlamı Docker betiği, eski JSONL'yi yalnızca
  doctor geçiş başlangıç verisi içinde oluşturur ve eski oturum dizini yolunu açıkça adlandırır.

### Kalan çalışmalar

- [x] Cron çalışma zamanı testi deposu değişkenlerini, doctor eski girdileri olmadıkları sürece
      `storePath` adından uzaklaştırarak yeniden adlandırın.
      Dosyalar: `src/cron/service.test-harness.ts`,
      `src/cron/service.runs-one-shot-main-job-disables-it.test.ts`,
      `src/cron/service/timer.regression.test.ts`,
      `src/cron/service/ops.test.ts`, `src/cron/service/store.test.ts`,
      `src/cron/service.heartbeat-ok-summary-suppressed.test.ts`,
      `src/cron/service.main-job-passes-heartbeat-target-last.test.ts`,
      `src/cron/store.test.ts`.
      Kanıt: `pnpm check:database-first-legacy-stores`; `rg -n 'storePath' src/cron --glob '!**/commands/doctor/**'`.
- [x] Eski dosya dönemi dışa aktarma testi taklitlerini kaldırın veya yeniden adlandırın.
      Dosya: `src/auto-reply/reply/commands-export-test-mocks.ts`.
      Kanıt: `rg -n 'resolveSessionFilePath|sessionFile|storePath|transcriptLocator' src/auto-reply/reply`.
- [x] Docker çalışma zamanı bağlamı eski JSONL başlangıç verisinin açıkça yalnızca doctor'a özgü olmasını sağlayın.
      Dosya: `scripts/e2e/session-runtime-context-docker-client.ts`.
      Kanıt: `rg -n 'sessions\\.json|sessionFile|\\.jsonl' scripts/e2e/session-runtime-context-docker-client.ts` yalnızca
      `seedBrokenLegacySessionForDoctorMigration` gösterir.
- [x] Herhangi bir şema değişikliğinden sonra Kysely tarafından oluşturulan türleri uyumlu tutun.
      Dosyalar: `src/state/openclaw-state-schema.sql`,
      `src/state/openclaw-agent-schema.sql`,
      `src/state/*generated*`.
      Kanıt: bu geçişte şema değişikliği yoktur; `pnpm db:kysely:check`;
      `pnpm lint:kysely`.
- [x] Dokunulan depolar, komutlar ve betikler için odaklanmış testleri yeniden çalıştırın.
      Kanıt: `pnpm test src/cron/service/store.test.ts src/cron/store.test.ts src/cron/service.heartbeat-ok-summary-suppressed.test.ts src/cron/service.main-job-passes-heartbeat-target-last.test.ts src/cron/service.every-jobs-fire.test.ts src/cron/service.persists-delivered-status.test.ts src/cron/service.runs-one-shot-main-job-disables-it.test.ts src/cron/service/ops.test.ts src/cron/service/timer.regression.test.ts src/auto-reply/reply/commands-export-session.test.ts extensions/telegram/src/thread-bindings.test.ts extensions/slack/src/monitor/message-handler/prepare.test.ts src/acp/translator.session-lineage-meta.test.ts`; `git diff --check`.
- [x] `done` ilan etmeden önce değişiklik denetimini veya uzaktan geniş kapsamlı kanıtı çalıştırın.
      Kanıt: `pnpm check:changed --timed -- <changed extension paths>`, geçici Node 24/pnpm kurulumunun ve
      eşitlenmiş `.git` içermeyen çalışma alanı için açık yol yönlendirmesinin ardından
      Hetzner Crabbox çalıştırması `run_3f1cabf6b25c` üzerinde başarılı oldu.

### Gerilemeye izin vermeyin

- Transkript konumlandırıcısı yok.
- Etkin oturum dosyası yok.
- Doctor eski geçiş testleri dışında sahte JSONL test fikstürü yok.
- Kysely'nin beklendiği yerlerde ham SQLite erişimi yok.
- Yeni dosya dönemi veritabanı geçişi yok. Genel şema `1` sürümünde kalır.
  Yayımlanmış ajan başına `1` sürümü şemasında, kararlı bellek kaynağı kimlikleri için
  `2` sürümüne yönelik sınırlandırılmış tek bir çalışma zamanı geçişi vardır.

## Kod Okuma Varsayımları

Bu planı engelleyen takip ürünü kararı yoktur. Uygulama
şu varsayımlarla ilerlemelidir:

- Doğrudan `node:sqlite` kullanın ve bu depolama yolu için WAL sıfırlamasına karşı güvenli bir Node çalışma zamanı
  (22.22.3+, 24.15+ veya 25.9+) gerektirin.
- Tam olarak bir normal yapılandırma dosyası tutun. Bu yeniden düzenlemede yapılandırmayı, plugin
  manifestlerini veya Git çalışma alanlarını SQLite'a taşımayın.
- Çalışma zamanı uyumluluk dosyaları gerekli değildir. Eski JSON ve JSONL dosyaları yalnızca
  geçiş girdileridir. Dala özgü SQLite yardımcı dosyaları hiçbir zaman yayımlanmadığından
  içe aktarılmak yerine silinir.
- Eski dosyadan veritabanına geçişin sahibi `openclaw doctor --fix`'dir. Çalışma zamanı
  başlangıcı yalnızca yayımlanmış SQLite şema sürümleri arasındaki sınırlı yükseltmelerin sahibidir;
  dosya dönemine ait durumu içe aktarmamalıdır.
- Kimlik bilgisi uyumluluğu da aynı kurala uyar: çalışma zamanı kimlik bilgileri
  SQLite'ta tutulur. Eski `auth-profiles.json`, ajan başına `auth.json` ve paylaşılan
  `credentials/oauth.json` dosyaları doctor geçiş girdileridir ve içe aktarmadan
  sonra kaldırılır.
- Oluşturulan model kataloğu durumu veritabanı desteklidir. Çalışma zamanı kodu
  `agents/<agentId>/agent/models.json` yazmamalıdır; mevcut `models.json` dosyaları eski
  doctor girdileridir ve `agent_model_catalogs` içine aktarıldıktan sonra kaldırılır.
- Çalışma zamanı, transkript konumlandırıcılarını geçirmemeli, normalleştirmemeli veya aralarında köprü kurmamalıdır. Etkin
  transkript kimliği SQLite'taki `{agentId, sessionId}`'dir. Dosya yolları
  yalnızca eski doctor girdileridir ve `sqlite-transcript://...`, bir
  sınır tanıtıcısı olarak ele alınmak yerine çalışma zamanı, protokol, kanca ve plugin
  yüzeylerinden kaldırılmalıdır.
- Çalışma zamanındaki SQLite transkript okumaları, eski JSONL girdi biçimi geçişlerini çalıştırmaz veya
  uyumluluk için transkriptlerin tamamını yeniden yazmaz. Eski girdi normalleştirmesi,
  açık doctor/içe aktarma yardımcı programlarında kalır. Doctor, SQLite satırlarını eklemeden
  önce eski JSONL transkript dosyalarını normalleştirir; mevcut çalışma zamanı satırları
  zaten güncel transkript şemasında yazılır. Yörünge/oturum dışa aktarımı
  bu satırları olduğu gibi okur ve dışa aktarma sırasında eski geçişleri gerçekleştirmemelidir.
- Eski transkript JSONL ayrıştırma/geçiş yardımcıları yalnızca doctor içindir. Çalışma zamanı
  transkript biçimi kodu yalnızca güncel SQLite transkript bağlamını oluşturur; eski JSONL
  girdi yükseltmelerini satırları eklemeden önce doctor üstlenir.
- Çalışma zamanının sahip olduğu eski JSONL transkript akış yardımcısı silindi. Doctor
  içe aktarma kodu, eski dosyaların açıkça okunmasını üstlenir; çalışma zamanı oturum geçmişi
  SQLite satırlarını okur.
- Codex app-server bağlamaları, Codex plugin-state ad alanındaki kurallı
  anahtar olarak OpenClaw `sessionId` değerini kullanır. `sessionKey`,
  yönlendirme/görüntüleme için meta veridir ve kalıcı oturum kimliğinin yerini almamalı veya
  transkript dosyası kimliğini yeniden canlandırmamalıdır.
- Bağlam motorları güncel çalışma zamanı sözleşmesini doğrudan alır. Kayıt defteri,
  motorları `sessionKey`, `transcriptScope` veya `prompt` değerlerini silen
  yeniden deneme uyumluluk katmanlarıyla sarmalamamalıdır; güncel veritabanı öncelikli
  parametreleri kabul edemeyen motorlar, köprülenmek yerine belirgin biçimde başarısız olmalıdır.
- Yedekleme çıktısı tek bir arşiv dosyası olarak kalmalıdır. Veritabanı içerikleri
  bu arşive ham canlı WAL yardımcı dosyaları olarak değil, kompakt SQLite anlık görüntüleri olarak girmelidir.
- Transkript araması yararlıdır ancak veritabanı öncelikli ilk
  sürüm için gerekli değildir. Şemayı, FTS daha sonra eklenebilecek şekilde tasarlayın.
- Veritabanı sınırı oturana kadar çalışan yürütmesi, ayarların arkasında
  deneysel olarak kalmalıdır.

## Kod Okuma Bulguları

Mevcut dal, kavram kanıtlama aşamasını çoktan geçmiştir. Paylaşılan
veritabanı mevcuttur, Node `node:sqlite` küçük bir çalışma zamanı yardımcısı üzerinden bağlanmıştır ve
önceki depolar artık `state/openclaw.sqlite` veya sahibi olan
`openclaw-agent.sqlite` veritabanına yazmaktadır.

Kalan iş SQLite'ı seçmek değil; yeni sınırı temiz tutmak
ve hâlâ eski dosya dünyasına benzeyen uyumluluk biçimli tüm arayüzleri silmektir:

- Oturum `storePath` artık bir çalışma zamanı kimliği, test sabiti biçimi veya
  durum yükü alanı değildir. Çalışma zamanı ve köprü testleri artık
  `storePath` sözleşme adını içermez; bu eski terminolojinin sahibi doctor/geçiş kodudur.
- Oturum yazmaları artık eski işlem içi `store-writer.ts`
  kuyruğundan geçmez. SQLite yama yazmaları işlemin dışında hazırlanır, ardından açık
  çakışma algılaması içeren kısa ve eşzamanlı bir doğrulama/uygulama işlemi kullanır.
- Eski yol keşfinin hâlâ geçerli geçiş kullanımları vardır ancak çalışma zamanı kodu
  `sessions.json` ve transkript JSONL dosyalarını olası yazma
  hedefleri olarak ele almayı bırakmalıdır.
- Ajana ait tablolar, ajan başına SQLite veritabanlarında bulunur. Küresel veritabanı
  kayıt defteri/kontrol düzlemi satırlarını tutar; transkript kimliği, ajan başına transkript
  satırlarındaki `{agentId, sessionId}`'dir. Çalışma zamanı kodu transkript dosyası
  yollarını kalıcılaştırmamalı veya transkript konumlandırıcılarını geçirmemelidir.
- Doctor zaten birkaç eski dosyayı içe aktarmaktadır. Temizlik çalışması, bunu
  doctor'ın çağırdığı, kalıcı bir geçiş raporuna sahip tek ve açık bir geçiş
  uygulaması hâline getirmektir.

Uygulamayı engelleyen başka ürün sorusu yoktur.

## Mevcut Kod Yapısı

Dalın hâlihazırda gerçek bir paylaşılan SQLite temeli vardır:

- Çalışma zamanı alt sınırı artık WAL sıfırlamasında güvenli bir Node derlemesi gerektiriyor: 22.22.3+,
  24.15+ veya 25.9+. `package.json`, CLI çalışma zamanı koruması, yükleyici varsayılanları,
  macOS çalışma zamanı bulucusu, CI ve genel kurulum belgelerinin tümü birbiriyle uyumludur.
- `src/state/openclaw-state-db.ts`, `openclaw.sqlite` öğesini açar, WAL'ı ayarlar,
  `synchronous=NORMAL`, `busy_timeout=30000`, `foreign_keys=ON` öğelerini ayarlar ve
  `src/state/openclaw-state-schema.sql` öğesinden türetilen oluşturulmuş şema
  modülünü uygular.
- Kysely tablo türleri ve çalışma zamanı şema modülleri, kaydedilmiş `.sql` dosyalarından oluşturulan
  tek kullanımlık SQLite veritabanlarından üretilir; çalışma zamanı kodu artık
  genel, aracı başına veya proxy yakalama veritabanları için kopyalanıp yapıştırılmış şema dizelerini tutmaz.
- Çalışma zamanı depoları, SQLite satır şekillerini elle yansıtmak yerine seçilen ve eklenen
  satır türlerini oluşturulan Kysely `DB` arayüzlerinden türetir. Ham SQL,
  şema uygulaması, pragma'lar ve yalnızca geçişe yönelik DDL ile sınırlı kalır.
- Genel SQLite şeması `user_version = 1` düzeyinde kalır. Aracı başına şema
  `2` sürümündedir; açıcı, yayımlanmış `1` sürümündeki
  bellek kaynağı anahtarını kararlı bir tamsayı kimliğine atomik olarak geçirir. Dosyadan veritabanına içe aktarma
  doctor kodunda kalır.
- İlişkisel sahiplik, sahiplik sınırının kurallı olduğu yerlerde zorunlu kılınır:
  kaynak geçiş satırları `migration_runs` öğesinden, görev teslim durumu
  `task_runs` öğesinden ve transkript kimliği satırları
  transkript olaylarından basamaklı olarak silinir.
- Mevcut paylaşılan tablolar arasında `agent_databases`,
  `auth_profile_stores`, `auth_profile_state`,
  `plugin_state_entries`, `plugin_blob_entries`, `media_blobs`,
  `skill_uploads`, `capture_sessions`, `capture_events`, `capture_blobs`,
  `sandbox_registry_entries`, `cron_jobs`, `commitments`,
  `delivery_queue_entries`, `model_capability_cache`,
  `workspace_setup_state`, `workspace_path_aliases`, `workspace_attestations`,
  `workspace_generated_bootstrap_hashes`, `native_hook_relay_bridges`,
  `current_conversation_bindings`, `plugin_binding_approvals`,
  `tui_last_sessions`, `acp_sessions`, `acp_replay_sessions`,
  `acp_replay_events`, `task_runs`, `task_delivery_state`, `flow_runs`,
  `subagent_runs`, `migration_runs` ve `backup_runs` bulunur.
- Plugin'lere ait rastgele durumlar, ana makineye ait türlendirilmiş tablolar edinmez. Kurulu
  plugin'ler, sürümlendirilmiş JSON yükleri için `plugin_state_entries` ve
  baytlar için `plugin_blob_entries` kullanır; bunlar ad alanı/anahtar sahipliği, TTL temizliği,
  yedekleme ve plugin geçiş kayıtlarını içerir. Ana makinenin sorgu sözleşmesine sahip olduğu
  `plugin_binding_approvals` gibi durumlarda, ana makineye ait plugin orkestrasyon durumu
  yine türlendirilmiş tablolara sahip olabilir.
- Plugin geçişleri, ana makine şeması geçişleri değil, plugin'e ait ad alanları üzerindeki veri geçişleridir.
  Bir plugin, kendi sürümlendirilmiş durum/blob girdilerini bir geçiş sağlayıcısı
  aracılığıyla geçirebilir ve ana makine, kaynak/çalıştırma durumunu normal
  geçiş defterine kaydeder. Ana makinenin kendisi yeni bir
  plugin'ler arası sözleşmenin sahipliğini üstlenmediği sürece yeni plugin kurulumları
  `openclaw-state-schema.sql` öğesinin değiştirilmesini gerektirmez.
- `src/state/openclaw-agent-db.ts`,
  `agents/<agentId>/agent/openclaw-agent.sqlite` öğesini açar, veritabanını
  genel DB'ye kaydeder ve aracıya yerel oturum, transkript, VFS, yapıt, önbellek
  ve bellek dizini tablolarının sahipliğini üstlenir. Paylaşılan çalışma zamanı keşfi artık bu sorguyu
  her çağrı noktasında yeniden uygulamak yerine oluşturulmuş türlere sahip
  `agent_databases` kayıt defterini okur.
- Genel ve aracı başına veritabanları; veritabanı rolü,
  şema sürümü, zaman damgaları ve aracı veritabanları için aracı kimliği içeren bir `schema_meta` satırı kaydeder. Genel DB
  `user_version = 1` düzeyinde kalır; aracı başına DB'ler, sınırlı
  bellek kaynağı kimliği geçişinden sonra `2` sürümünü kullanır.
- Aracı başına oturum kimliği artık `session_id` ile anahtarlanan,
  `session_key`, `session_scope`, `account_id`,
  `primary_conversation_id`, zaman damgaları, görüntüleme alanları, model meta verileri,
  düzenek kimliği ve üst/oluşturma bağlantılarını sorgulanabilir sütunlar olarak içeren kurallı bir `sessions` kök tablosuna sahiptir. `session_routes`,
  `session_key` öğesinden geçerli
  `session_id` öğesine uzanan benzersiz etkin rota dizinidir; böylece bir rota anahtarı, sık erişilen okumaların yinelenen `sessions.session_key` satırları arasında seçim yapmasına neden olmadan
  yeni ve kalıcı bir oturuma taşınabilir. Eski
  `session_entries.entry_json` uyumluluk biçimli yükü, yabancı anahtar aracılığıyla
  kalıcı `session_id` köküne bağlıdır; artık bir oturumun
  şema düzeyindeki tek temsili değildir.
- Aracı başına harici konuşma kimliği de ilişkiseldir:
  `conversations` normalleştirilmiş sağlayıcı/hesap/konuşma kimliğini depolar ve
  `session_conversations`, bir OpenClaw oturumunu bir veya daha fazla harici
  konuşmaya bağlar. Bu, birden fazla eşin `session_key` içinde yanlış beyanda bulunmadan
  kasıtlı olarak tek bir oturumla eşlenebildiği paylaşılan ana DM oturumlarını kapsar. SQLite ayrıca
  doğal sağlayıcı kimliği için benzersizliği zorunlu kılar; böylece aynı
  kanal/hesap/tür/eş/iş parçacığı demeti konuşma kimlikleri arasında çatallanamaz.
  Paylaşılan ana doğrudan eşler bir `participant` rolüyle bağlanır; böylece tek bir
  OpenClaw oturumu, eski eşleri belirsiz ilişkili satırlara indirgemeden
  birden fazla harici DM eşini temsil edebilir. `sessions.primary_conversation_id` hâlâ
  geçerli türlendirilmiş teslim hedefine işaret eder. Kapalı yönlendirme/durum sütunları,
  yalnızca TypeScript birleşim türlerine güvenmek yerine SQLite `CHECK` kısıtlamalarıyla zorunlu kılınır.
  Çalışma zamanı oturum projeksiyonu, türlendirilmiş oturum/konuşma
  sütunlarını uygulamadan önce `session_entries.entry_json` içindeki uyumluluk yönlendirme gölgelerini temizler;
  böylece eski JSON yükleri teslim hedeflerini yeniden etkinleştiremez.
  Alt aracı duyuru yönlendirmesi de türlendirilmiş SQLite teslim bağlamını gerektirir;
  artık uyumluluk `SessionEntry` rota alanlarına geri dönmez.
  Gateway `chat.send` açık teslim kalıtımı, `origin`/`last*` uyumluluk alanları yerine türlendirilmiş SQLite
  teslim bağlamını okur.
  `tools.effective` da sağlayıcı/hesap/iş parçacığı bağlamını eski `last*` oturum girdisi gölgelerinden değil,
  türlendirilmiş SQLite teslim/yönlendirme satırlarından türetir.
  Sistem olayı istem bağlamı; kanal/hedef/hesap/iş parçacığı alanlarını
  `origin` gölgeleri yerine türlendirilmiş teslim alanlarından yeniden oluşturur.
  Paylaşılan `deliveryContextFromSession` yardımcısı ve oturumdan konuşmaya
  eşleyici artık `SessionEntry.origin` öğesini tamamen yok sayar; sık erişilen rota kimliğini
  yalnızca türlendirilmiş teslim alanları ve ilişkisel konuşma satırları oluşturabilir.
  Çalışma zamanı oturum girdisi normalleştirmesi, `entry_json` öğesini kalıcılaştırmadan veya
  projekte etmeden önce `origin` öğesini kaldırır ve gelen meta veri, yeni kaynak
  gölgeleri oluşturmak yerine türlendirilmiş kanal/sohbet alanlarını ve ilişkisel konuşma satırlarını yazar.
- Transkript olayları, transkript anlık görüntüleri ve yörünge çalışma zamanı olayları artık
  kurallı aracı başına `sessions` köküne başvurur ve oturum
  silindiğinde basamaklı olarak silinir. Transkript kimliği/idempotans satırları, tam
  transkript olayı satırından basamaklı olarak silinmeye devam eder.
- Bellek çekirdeği dizinleri artık açık aracı veritabanı tabloları olan
  `memory_index_meta`, `memory_index_sources`, `memory_index_chunks` ve
  `memory_embedding_cache` öğelerini kullanır; `memory_index_state` ise revizyon değişikliklerini izler.
  İsteğe bağlı FTS/vektör yan dizinleri, genel `meta`, `files`, `chunks`,
  `chunks_fts` veya `chunks_vec` tabloları yerine `memory_index_chunks_fts` ve
  `memory_index_chunks_vec` olarak adlandırılır. Kurallı adlar, mevcut
  yol/kaynak satırı şeklini ve serileştirilmiş gömme uyumluluğunu korur. Bu tablolar
  türetilmiş/arama önbelleğidir, kurallı transkript depolaması değildir; silinebilir ve
  bellek çalışma alanı dosyaları ile yapılandırılmış kaynaklardan yeniden oluşturulabilir.
  Yayımlanmış genel adlı bir bellek dizini açıldığında meta verileri, kaynakları,
  parçaları ve gömme önbelleği kurallı tablolara geçirilir; türetilmiş FTS/vektör
  tabloları kurallı adları altında yeniden oluşturulur.
- Alt aracı çalıştırma kurtarma durumu artık dizinlenmiş alt, istekte bulunan ve denetleyici
  oturum anahtarlarına sahip türlendirilmiş paylaşılan `subagent_runs` satırlarında bulunur. Eski
  `subagents/runs.json` dosyası yalnızca Doctor temizleme girdisidir. Çalıştırma girdileri
  geçici kurtarma durumu olduğundan Doctor, kullanımdan kaldırma makbuzunu kaydeder ve
  dosyayı içe aktarmadan atar. SQLite satırları budandıktan sonra bir dosya
  girdilerinin etkin mi yoksa eski mi olduğunu kanıtlayamayacağından, operatörler
  bu sınırı aşan yükseltmeden önce dosya dönemindeki etkin çalıştırmaların tamamlanmasını beklemelidir.
- Geçerli konuşma bağlamaları artık normalleştirilmiş konuşma kimliğiyle anahtarlanan
  türlendirilmiş paylaşılan `current_conversation_bindings` satırlarında bulunur; hedef aracı/oturum
  sütunları, konuşma türü, durum, sona erme ve meta veriler yinelenen opak bir bağlama kaydı
  yerine ilişkisel sütunlarda depolanır.
  Kalıcı bağlama anahtarı, doğrudan/grup/kanal başvurularının çakışmaması için
  normalleştirilmiş konuşma türünü içerir ve SQLite geçersiz bağlama
  türü/durum değerlerini reddeder. Eski
  `bindings/current-conversations.json` dosyası yalnızca doctor geçiş girdisidir.
- Teslim kuyruğu kurtarma artık kanal, hedef,
  hesap, oturum, yeniden deneme, hata, platform gönderimi ve kurtarma durumuna yönelik türlendirilmiş kuyruk sütunlarını
  yeniden oynatma JSON'unun üzerine bindirir. `entry_json` yeniden oynatma yüklerini, kancaları ve biçimlendirme
  yükünü korur; ancak sık erişilen kuyruk yönlendirmesi/durumu için türlendirilmiş sütunlar yetkilidir.
- TUI son oturum geri yükleme işaretçileri artık karma değeri alınmış TUI bağlantısı/oturum kapsamıyla anahtarlanan
  türlendirilmiş paylaşılan `tui_last_sessions` satırlarında bulunur.
  Çalışma zamanı yalnızca SQLite'ı okur ve yazar, her kapsamı atomik olarak ekler veya günceller ve
  Heartbeat oturumlarını hariç tutar. `openclaw doctor --fix` eski TUI JSON dosyasını sıkı biçimde doğrular,
  daha yeni SQLite satırlarını korur, kurallı sonucu doğrular
  ve değişmemiş eski dosyayı arşiv olarak bırakmak yerine kaldırır.
- Discord komut dağıtımı karmaları artık paylaşılan plugin durumu SQLite
  deposunda bulunur. Çalışma zamanı yalnızca uygulama kapsamındaki tam anahtarları okur ve yazar. Doctor,
  yeniden oluşturulabilir eski `discord/command-deploy-cache.json` dosyasını
  içe aktarmadan siler; böylece bir sonraki başlatma tek bir kurallı uzlaştırma gerçekleştirir.
- Varsayılan TTS tercihleri artık `speech-core` plugin'i altında anahtarlanan paylaşılan plugin durumu SQLite satırlarında bulunur.
  Eski `settings/tts.json` dosyası yalnızca doctor geçiş
  girdisidir; çalışma zamanı artık TTS tercihleri JSON dosyalarını okumaz veya yazmaz ve
  eski yol çözümleyicisi doctor geçiş modülünde bulunur.
- Gizli hedef meta verileri artık her kimlik bilgisi hedefini bir yapılandırma dosyası gibi göstermek yerine
  depolardan söz eder. `openclaw.json` yapılandırma deposu olarak kalır;
  kimlik doğrulama profili hedefleri, sağlayıcı biçimli kimlik bilgilerini JSON yükleri olarak tutan türlendirilmiş SQLite `auth_profile_stores` satırlarını kullanır.
- Gizli bilgi denetimi artık kullanımdan kaldırılmış aracı başına `auth.json` dosyalarını taramaz. Bu eski dosya hakkında
  uyarma, dosyayı içe aktarma ve kaldırma işlemlerinin sahipliği Doctor'a aittir.
- Eski kimlik doğrulama profili yol yardımcıları artık doctor eski kodunda bulunur. Çekirdek kimlik doğrulama
  profili yol yardımcıları, `auth-profiles.json` veya `auth-state.json` çalışma zamanı yollarını değil,
  SQLite kimlik doğrulama deposu kimliğini ve görüntüleme konumlarını sunar.
- Alt aracı çalıştırma kurtarma ve OpenRouter model yetenek önbelleği çalışma zamanı modülleri
  artık SQLite anlık görüntü okuyucularını/yazıcılarını yalnızca doctor'a ait eski JSON
  içe aktarma yardımcılarından ayrı tutar. OpenRouter yetenekleri, tek bir opak önbellek blob'u veya sağlayıcıya özgü bir ana makine tablosu yerine
  `provider_id = "openrouter"` altında türlendirilmiş genel
  `model_capability_cache` satırlarını kullanır. Alt aracı çalıştırma
  `taskName` değeri türlendirilmiş `subagent_runs.task_name` sütununda depolanır;
  `payload_json` kopyası yeniden oynatma/hata ayıklama verisidir, sık erişilen görüntüleme veya
  arama alanlarının kaynağı değildir.
- `src/agents/filesystem/virtual-agent-fs.sqlite.ts`, aracı veritabanındaki `vfs_entries` tablosu
  üzerinde bir SQLite VFS uygular. Dizin okumaları, özyinelemeli
  dışa aktarmalar, silmeler ve yeniden adlandırmalar; bütün bir ad alanını taramak veya `LIKE` yol eşleştirmesine güvenmek yerine
  dizinlenmiş `(namespace, path)` önek aralıklarını kullanır.
- `src/agents/runtime-worker.entry.ts`, çalışanlar için çalıştırma başına SQLite VFS, araç yapıtı,
  çalıştırma yapıtı ve kapsamlı önbellek depoları oluşturur.
- Çalışma alanı önyükleme tamamlanma durumu, doğrulama güncelliği ve oluşturulan önyükleme
  karmaları artık standart çalışma alanı kimliğiyle anahtarlanan, türü belirlenmiş ortak
  `workspace_setup_state`, `workspace_path_aliases`,
  `workspace_attestations` ve `workspace_generated_bootstrap_hashes`
  satırlarında bulunur. Kalıcı sözcüksel ve gerçek yol takma adları, yapılandırılmış bir sembolik
  bağlantı kaybolduktan sonra kaybolan çalışma alanı korumasını kararlı tutar; yeniden hedeflenen
  takma adlar güvenli biçimde başarısız olur. Çalışma zamanı artık
  `openclaw-workspace-state.json`, `.openclaw/workspace-state.json`, durum dizini
  `workspace-attestations/*.attested` veya eşdüzey `<workspace>.attested`
  yardımcı dosyalarını okumaz ya da bunlara yazmaz. `openclaw doctor --fix` eski kaynakları doğrular ve sahiplenir,
  bunları geçiş makbuzlarıyla SQLite'a aktarır, standart satırları doğrular ve
  ancak bundan sonra sahiplenilen dosyaları kaldırır.
- Ortak şema bir `exec_approvals_config` tekil satırı ayırır, ancak
  çalışma zamanı geçişi hâlâ beklemededir. TypeScript ve macOS yardımcı uygulaması hâlâ
  durum kapsamlı JSON dosyasını kullanır ve birlikte SQLite'a geçmelidir.
- TypeScript cihaz kimliği artık türü belirlenmiş `device_identities` satırlarını kullanır;
  yalnızca doctor tarafından yapılan eski JSON aktarımı çalışma zamanı sahibinin dışında tutulur. Cihaz kimlik doğrulaması,
  eşgüdümlü bir şema ve çalışma zamanları arası geçiş beklenirken hâlâ dosya tabanlıdır;
  `device_auth_tokens` bu takip çalışması için ayrılmış olarak kalır.
- GitHub Copilot belirteç değişimi önbelleği, `github-copilot/token-cache/default`
  altındaki ortak SQLite Plugin durum tablosunu kullanır. Bu, sağlayıcının sahip olduğu bir önbellek durumudur;
  bu nedenle kasıtlı olarak ana makine şemasına bir tablo eklemez.
- GitHub Copilot Compaction artık `openclaw-compaction-*.json`
  çalışma alanı yardımcı dosyalarına yazmaz. Test düzeneği, izlenen SDK oturumu için SDK geçmişi Compaction RPC'sini
  çağırır ve OpenClaw kalıcı oturum/transkript durumunu
  uyumluluk işaretçisi dosyaları yerine SQLite'ta tutar.
- Ortak Swift çalışma zamanı (`OpenClawKit`), cihaz
  kimliği için aynı `state/openclaw.sqlite#table/device_identities` biçimini ve satır anahtarlarını kullanır.
  TypeScript Doctor bu kapsayıcılara erişemediği için Apple kapsayıcısındaki eski dosyalar Swift geçiş
  sahibi tarafından içe aktarılır. Swift cihaz kimlik doğrulaması, eşgüdümlü kimlik doğrulama takip çalışması için
  dosya tabanlı kalır.
- Android cihaz kimliği ve önbelleğe alınmış cihaz kimlik doğrulaması, uygulamaya yerel depolar olarak kalır. Bunlar
  Android'in sahip olduğu ayrı bir geçiş gerektirir; ana makine SQLite talepleri mevcut
  Android davranışını açıklamaz.
- Android bildirimlerinin son paket geçmişi, türü belirlenmiş
  `android_notification_recent_packages` satırlarını kullanır. Çalışma zamanı artık eski
  SharedPreferences CSV anahtarlarını taşımaz veya okumaz.
- Eski `identity/device.json` mevcut olduğunda, SQLite kimlik satırı geçersiz olduğunda
  veya SQLite kimlik deposu açılamadığında cihaz kimliği oluşturma işlemi güvenli biçimde başarısız olur.
  Doctor önce bu dosyayı içe aktarıp kaldırır; böylece çalışma zamanı başlangıcı, geçişten önce eşleştirme
  kimliğini sessizce değiştiremez.
- Cihaz kimliği seçimi bir JSON dosyası konumlandırıcısı değil, SQLite satır anahtarıdır. Testler
  ve Gateway yardımcıları açık kimlik anahtarları geçirir; kullanımdan kaldırılmış
  `identity/device.json` dosya adını yalnızca doctor geçişi ve güvenli başlangıç engeli bilir.
- Oturum sıfırlama uyumluluğu artık doctor yapılandırma geçişinde bulunur:
  `session.idleMinutes`, `session.reset.idleMinutes` içine taşınır;
  `session.resetByType.dm`, `session.resetByType.direct` içine taşınır ve
  çalışma zamanı sıfırlama ilkesi yalnızca standart sıfırlama anahtarlarını okur.
- Eski yapılandırma uyumluluğu artık `src/commands/doctor/` altında bulunur. Normal
  `readConfigFileSnapshot()` doğrulaması, doctor eski sürüm algılayıcılarını içe aktarmaz
  veya eski sürüm sorunlarına açıklama eklemez; `runDoctorConfigPreflight()`, doctor
  onarımı/raporlaması için bu sorunları ekler. Doctor yapılandırma akışı
  `src/commands/doctor/legacy-config.ts` öğesini içe aktarır ve eski OAuth profil kimliği onarımı
  `src/commands/doctor/legacy/oauth-profile-ids.ts`
  altında bulunur.
- Doctor dışındaki komutlar, eski yapılandırma onarımını otomatik olarak çalıştırmaz. Örneğin,
  `openclaw update --channel` artık geçersiz eski yapılandırmada başarısız olur ve
  doctor geçiş kodunu sessizce içe aktarmak yerine kullanıcıdan doctor'ı çalıştırmasını ister.
- Web push, APNs, Voice Wake, güncelleme denetimleri ve yapılandırma sağlığı artık
  bütünüyle opak JSON blobları yerine abonelikler, VAPID anahtarları, Node kayıtları, tetikleyici satırları,
  yönlendirme satırları, güncelleme bildirimi durumu ve yapılandırma sağlığı girdileri için türü belirlenmiş
  ortak SQLite tablolarını kullanır. Web Push ve APNs yazma işlemleri yalnızca etkilenen
  birincil anahtar satırını ekler veya günceller; yapılandırma sağlığı, yapılandırma yoluna göre uzlaştırma yapar.
  Bunların çalışma zamanı modülleri, yalnızca Doctor tarafından kullanılan eski JSON içe aktarma yardımcılarından ayrı kalır.
- APNs çalışma zamanı yalnızca `apns_registrations` öğesini okur ve yazar. Açık
  `openclaw doctor --fix`, kullanımdan kaldırılmış
  `push/apns-registrations.json` öğesini katı biçimde içe aktarır, mevcut standart satırları korur,
  işlemi doğrular, bir makbuz kaydeder ve gizli bilgiler içeren JSON'u kaldırır.
  Makbuz destekli yeniden denemeler yalnızca temizlik yaparken
  `apns_registration_tombstones`, ilk onarımdan önceki geçersiz kılmaları kapsar; böylece
  eski röle izinleri veya cihaz belirteçleri yeniden etkinleşemez.
- Node ana makine yapılandırması artık ortak SQLite veritabanında türü belirlenmiş tekil bir satır kullanır.
  Eski `node.json` dosyası veya yarıda kesilmiş bir sahiplenme işlemi mevcutken çalışma zamanı
  güvenli biçimde başarısız olur; açık `openclaw doctor --fix`, normal çalışma zamanı kullanımından önce
  bunu katı biçimde içe aktarır ve kaldırır.
- Cihaz/Node eşleştirmesi, kanal eşleştirmesi, kanal izin listeleri ve önyükleme durumu
  artık bütünüyle opak JSON blobları yerine türü belirlenmiş SQLite satırlarını kullanır. Plugin bağlama
  onayları ve Cron işi durumu da aynı ayrımı izler: çalışma zamanı modülleri
  SQLite destekli işlemler ve tarafsız anlık görüntü yardımcıları sunar; eşleştirme/önyükleme
  ile Plugin bağlama onayı anlık görüntüsü yazma işlemleri tabloları boşaltmak yerine satırları
  birincil anahtara göre uzlaştırırken doctor, eski JSON dosyalarını
  `src/commands/doctor/legacy/*` modülleri aracılığıyla içe aktarır/kaldırır.
- Yüklü Plugin kayıtları artık SQLite yüklü Plugin dizininde bulunur.
  Çalışma zamanı yapılandırma okuma/yazma işlemleri artık eski
  `plugins.installs` tarafından oluşturulmuş yapılandırma verilerini taşımaz veya korumaz; doctor, normal
  çalışma zamanı kullanımından önce bu eski yapılandırma biçimini SQLite'a aktarır.
- QQBot kimlik bilgisi kurtarma anlık görüntüleri artık
  `qqbot/credential-backups` altındaki SQLite Plugin durumunda bulunur. Çalışma zamanı artık
  `qqbot/data/credential-backup*.json` öğesine yazmaz; QQBot doctor sözleşmesi, etkin durum dizinindeki
  bu eski yedekleme dosyalarını içe aktarır ve arşivler.
- Gateway yeniden yükleme planlaması, dahili bir `installedPluginIndex.installRecords.*`
  fark ad alanı altında SQLite yüklü Plugin dizini anlık görüntülerini karşılaştırır. Çalışma zamanı
  yeniden yükleme kararları artık bu satırları sahte `plugins.installs` yapılandırma
  nesneleri içine sarmalamaz.
- Matrix hesap kimlik bilgileri artık SQLite Plugin durumunda bulunur. Çalışma zamanı yalnızca
  bu standart depoyu okur; Doctor, hesapları çözümlenebildiğinde kullanımdan kaldırılmış
  `credentials/matrix/credentials*.json` dosyalarını içe aktarır, doğrular ve arşivler.
- Çekirdek eşleştirme ve Cron çalışma zamanı modülleri artık eski JSON yol oluşturucularını kullanmaz.
  Kullanımdan kaldırılmış eşleştirme yolu SDK yardımcısı yalnızca geçiş uyumluluğu için kalır;
  dosya okumaları ve içe aktarımları doctor durum geçişinin sorumluluğundadır. Doctor'a ait eski
  modüller `pending.json`, `paired.json`, `bootstrap.json` ve
  `cron/jobs.json` kaynak yollarını yalnızca içe aktarma testleri ve geçiş için oluşturur. Eski Cron
  işi biçimi normalleştirmesi ve JSONL geçmişi içe aktarımı
  `src/commands/doctor/cron/` altında bulunur; eski SQLite geçmişi sonlandırması,
  durum veritabanı açılırken çalışır.
- `src/commands/doctor/legacy/runtime-state.ts`, Node ana makine yapılandırması dâhil eski JSON durum
  dosyalarını doctor üzerinden SQLite'a aktarır. Yeni eski dosya
  içe aktarıcıları `src/commands/doctor/legacy/` altında kalır.
- `src/commands/doctor/state-migrations.ts`, eski `sessions.json` ve
  `*.jsonl` transkriptlerini doğrudan SQLite'a aktarır ve başarıyla aktarılan kaynakları kaldırır. Artık
  kök eski transkriptleri `agents/<agentId>/sessions/*.jsonl` üzerinden hazırlamaz
  veya içe aktarmadan önce standart bir JSONL hedefi oluşturmaz.
- Durum bütünlüğü doctor denetimleri artık eski oturum dizinlerini taramaz veya
  sahipsiz JSONL silme seçeneği sunmaz. Eski transkript dosyaları yalnızca geçiş girdileridir
  ve içe aktarma ile kaynak kaldırma işlemlerinin sahibi geçiş adımıdır.
- Eski korumalı alan kayıt defteri içe aktarımı
  `src/commands/doctor/legacy/sandbox-registry.ts` altında bulunur; etkin korumalı alan kayıt defteri
  okuma ve yazma işlemleri yalnızca SQLite kullanmaya devam eder.
- Eski oturum transkripti sağlık/içe aktarma onarımı
  `src/commands/doctor/legacy/session-transcript-health.ts` altında bulunur; çalışma zamanı komut
  modülleri artık JSONL transkript ayrıştırma veya etkin dal onarım kodu içermez.

Tamamlanan birleştirme/silme işlemlerinden öne çıkanlar:

- Plugin durumu artık paylaşılan `state/openclaw.sqlite` veritabanını kullanıyor. Eski
  dal yerelindeki `plugin-state/state.sqlite` yardımcı dosya içe aktarıcısı kaldırıldı çünkü
  bu SQLite düzeni hiçbir zaman yayımlanmadı. Yoklama/test yardımcıları, Plugin durumuna
  özgü bir SQLite yolu sunmak yerine paylaşılan `databasePath` değerini bildiriyor.
- Görev ve Görev Akışı çalışma zamanı tabloları artık `tasks/runs.sqlite` ve
  `tasks/flows/registry.sqlite` yerine paylaşılan `state/openclaw.sqlite` veritabanında
  bulunuyor; eski yardımcı dosya içe aktarıcıları, aynı yayımlanmamış düzen
  nedeniyle kaldırıldı.
- `src/config/sessions/store.ts` artık gelen meta veriler, rota güncellemeleri
  veya güncellenme zamanı okumaları için `storePath` gerektirmiyor.
  Komut kalıcılığı, CLI oturum temizliği, alt ajan derinliği, kimlik doğrulama
  geçersiz kılmaları ve transkript oturum kimliği ajan/oturum satırı API'lerini
  kullanıyor. Yazmalar, iyimser çakışma yeniden denemesiyle SQLite satır yamaları
  olarak uygulanıyor.
- Oturum hedefi çözümlemesi artık eski `sessions.json` yollarını değil,
  ajan başına veritabanı hedeflerini sunuyor. Paylaşılan Gateway, ACP meta verileri,
  doctor rota onarımı ve `openclaw sessions`, `agent_databases` ile yapılandırılmış
  ajanları numaralandırıyor.
- Gateway oturum yönlendirmesi artık `resolveGatewaySessionDatabaseTarget` kullanıyor;
  döndürülen hedef, eski bir oturum deposu dosya yolu yerine `databasePath`
  ve aday SQLite satır anahtarlarını taşıyor.
- Kanal oturumu çalışma zamanı türleri artık güncellenme zamanı okumaları,
  gelen meta veriler ve son rota güncellemeleri için `{agentId, sessionKey}` sunuyor.
  Eski `saveSessionStore(storePath, store)` uyumluluk türü kaldırıldı.
- Plugin çalışma zamanı, uzantı API'si ve Plugin SDK oturum yüzeyleri artık
  etkin oturumun tüm depo/dosya uyumluluk yardımcıları yerine SQLite destekli
  oturum satırı yardımcılarını sunuyor. Kök kitaplık uyumluluk dışa aktarımları,
  eski dahili ve geçiş çağırıcıları için yalnızca Plugin SDK dışında kullanılabilir
  durumda kalıyor. Eski `resolveLegacySessionStorePath` yardımcısı kaldırıldı; eski
  `sessions.json` yol oluşturma işlemi artık geçiş ve test fikstürleriyle sınırlı.
- `src/config/sessions/session-entries.sqlite.ts` artık kurallı oturum girdilerini ajan başına
  veritabanında depoluyor ve satır düzeyinde okuma/upsert/silme yaması desteğine
  sahip. Çalışma zamanı upsert/yama/silme işlemleri artık büyük-küçük harf
  varyantlarını taramıyor veya eski takma ad anahtarlarını budamıyor; kurallı
  hâle getirme doctor'a ait. Bağımsız JSON içe aktarma yardımcısı kaldırıldı ve
  geçiş, tüm oturum tablosunu değiştirmek yerine daha yeni satırları upsert ile
  birleştiriyor. Genel okuma/listeleme/yükleme yardımcıları, etkin oturum meta
  verilerini türü belirlenmiş `sessions` ve `conversations`
  satırlarından yansıtıyor; `entry_json` bir uyumluluk/hata ayıklama
  gölgesidir ve türü belirlenmiş oturum kimliği veya teslimat bağlamı kaybedilmeden
  eski ya da geçersiz olabilir.
- `src/config/sessions/delivery-info.ts` artık teslimat bağlamını türü belirlenmiş
  ajan başına `sessions` + `conversations` + `session_conversations`
  satırlarından çözümlüyor. Artık çalışma zamanı teslimat kimliğini
  `session_entries.entry_json` üzerinden yeniden oluşturmuyor; türü belirlenmiş bir konuşma
  satırının eksikliği, çalışma zamanı geri dönüşü değil, bir doctor geçiş/onarım
  sorunudur.
- Depolanan oturumu sıfırlama kararları artık türü belirlenmiş
  `sessions.session_scope`, `sessions.chat_type` ve `sessions.channel` meta verilerini
  tercih ediyor. `sessionKey` ayrıştırması yalnızca komut hedeflerindeki
  açık iş parçacığı/konu son ekleri için kalıyor; grup ile doğrudan sıfırlama
  sınıflandırması artık anahtar biçiminden gelmiyor.
- Oturum listesi/durum görüntüleme sınıflandırması artık türü belirlenmiş
  sohbet meta verilerini ve Gateway oturum türünü kullanıyor. Artık
  `session_key` içindeki `:group:` veya `:channel:`
  alt dizelerini kalıcı grup/doğrudan doğrusu olarak kabul etmiyor.
- Sessiz yanıt ilkesi seçimi artık yalnızca açık konuşma türünü veya yüzey
  meta verilerini kullanıyor. Artık `session_key` alt dizelerinden
  doğrudan/grup ilkesi tahmin etmiyor.
- Oturum görüntüleme modeli çözümlemesi artık ajan kimliğini
  `session_key` içinden ayırmak yerine SQLite oturum veritabanı hedefinden
  alıyor.
- Ajandan ajana duyuru hedefi doldurma işlemi artık yalnızca türü belirlenmiş
  `sessions.list` `deliveryContext` kullanıyor. Artık kanal/hesap/iş parçacığı
  yönlendirmesini eski `origin`, yansıtılmış `last*`
  alanları veya `session_key` biçiminden kurtarmıyor.
- `sessions_send` iş parçacığı hedefi reddi artık türü belirlenmiş
  SQLite yönlendirme meta verilerini okuyor. Artık hedef anahtarındaki iş parçacığı
  son eklerini ayrıştırarak hedefleri reddetmiyor veya kabul etmiyor.
- Grup kapsamlı araç ilkesi doğrulaması artık mevcut veya oluşturulan
  oturumun türü belirlenmiş SQLite konuşma yönlendirmesini okuyor. Artık
  `sessionKey` kodunu çözerek grup/kanal kimliğine güvenmiyor; çağıran
  tarafından sağlanan grup kimlikleri, bunları doğrulayan türü belirlenmiş bir
  oturum satırı olmadığında atılıyor.
- Kanal modeli geçersiz kılma eşleştirmesi artık açık grup ve üst konuşma
  meta verilerini kullanıyor. Artık `parentSessionKey` içinden üst konuşma
  kimliklerinin kodunu çözmüyor.
- Depolanan model geçersiz kılma devralımı artık türü belirlenmiş oturum
  bağlamından açık bir üst oturum anahtarı gerektiriyor. Artık üst geçersiz
  kılmaları `sessionKey` içindeki `:thread:` veya
  `:topic:` son eklerinden türetmiyor.
- Eski oturum iş parçacığı bilgisi sarmalayıcısı ve yüklenen Plugin iş
  parçacığı ayrıştırıcısı kaldırıldı; hiçbir çalışma zamanı kodu
  `config/sessions/thread-info` içe aktarmıyor.
- Kanal konuşması yardımcısı artık tam oturum anahtarı ayrıştırma köprülerini
  sunmuyor. Çekirdek, sağlayıcıya ait ham konuşma kimliklerini hâlâ
  `resolveSessionConversation(...)` üzerinden normalleştiriyor ancak rota olgularını
  `sessionKey` üzerinden yeniden oluşturmuyor.
- Tamamlama teslimatı, gönderme ilkesi ve görev bakımı artık sohbet türünü
  `session_key` biçiminden türetmiyor. Eski sohbet türü anahtar ayrıştırıcısı
  silindi; bu yollar türü belirlenmiş oturum meta verileri, türü belirlenmiş
  teslimat bağlamı veya açık teslimat hedefi söz dağarcığı gerektiriyor.
- Oturum listesi/durumu, tanılamalar, onay hesabı bağlama, TUI Heartbeat
  filtreleme ve kullanım özetleri artık sağlayıcı/hesap/iş parçacığı/görüntüleme
  yönlendirmesi için `SessionEntry.origin` içeriğini taramıyor. Kalan tek çalışma
  zamanı `origin` okumaları, oturum dışı kavramlar veya mevcut tur
  teslimat nesneleridir.
- Onay isteğinin yerel konuşma araması artık türü belirlenmiş ajan başına
  oturum yönlendirme satırlarını okuyor. Artık kanal/grup/iş parçacığı konuşma
  kimliğini `sessionKey` içinden ayrıştırmıyor; türü belirlenmiş meta
  verilerin eksikliği bir geçiş/onarım sorunudur.
- Gateway oturum değişikliği/sohbet/oturum olay yükleri artık
  `SessionEntry.origin` veya `last*` rota gölgelerini yinelemiyor;
  istemciler türü belirlenmiş `channel`, `chatType` ve
  `deliveryContext` değerlerini alıyor.
- Heartbeat teslimat çözümlemesi artık türü belirlenmiş SQLite
  `deliveryContext` değerini doğrudan alabiliyor ve Heartbeat çalışma zamanı,
  mevcut yönlendirme için uyumluluk `session_entries` gölgelerine güvenmek
  yerine ajan başına oturum teslimat satırını iletiyor.
- Cron yalıtılmış ajan teslimat hedefi çözümlemesi de uyumluluk girdisi
  yüküne geri dönmeden önce mevcut rotasını türü belirlenmiş ajan başına oturum
  teslimat satırından dolduruyor.
- Alt ajan duyurusu kaynak çözümlemesi artık türü belirlenmiş isteyen oturum
  teslimat bağlamını `loadRequesterSessionEntry` üzerinden iletiyor ve bu satırı
  uyumluluk `last*`/`deliveryContext` gölgelerine tercih ediyor.
- Gelen oturum meta verisi güncellemeleri artık önce türü belirlenmiş ajan
  başına teslimat satırıyla birleştiriliyor; eski `SessionEntry` teslimat
  alanları yalnızca türü belirlenmiş bir konuşma satırı bulunmadığında geri
  dönüş olarak kullanılıyor.
- Yeniden başlatma/güncelleme teslimat çıkarımı artık türü belirlenmiş
  SQLite teslimat `threadId` değerinin, `sessionKey` içinden
  ayrıştırılan konu/iş parçacığı parçalarına üstün gelmesini sağlıyor; ayrıştırma
  yalnızca eski iş parçacığı biçimli anahtarlar için geri dönüştür.
- Kanca ajan bağlamı kanal kimlikleri artık önce türü belirlenmiş SQLite
  konuşma kimliğini, ardından açık ileti meta verilerini tercih ediyor. Artık
  sağlayıcı/grup/kanal parçalarını `sessionKey` içinden ayrıştırmıyor.
- Gateway `chat.send` harici rota devralımı artık kanal/doğrudan/grup
  kapsamını `sessionKey` parçalarından çıkarsamak yerine türü belirlenmiş
  SQLite oturum yönlendirme meta verilerini okuyor. Kanal kapsamlı oturumlar
  yalnızca türü belirlenmiş oturum kanalı ve sohbet türü, depolanan teslimat
  bağlamıyla eşleştiğinde devralıyor; paylaşılan ana oturumlar daha katı
  CLI/istemci-meta-verisi-yok kuralını koruyor.
- Yeniden başlatma nöbetçisi uyandırma ve devam yönlendirmesi artık Heartbeat
  uyandırmalarını veya yönlendirilmiş ajan turu devamlarını kuyruğa almadan önce
  türü belirlenmiş SQLite teslimat/yönlendirme satırlarını okuyor. Artık teslimat
  bağlamını oturum girdisi JSON gölgesinden yeniden oluşturmuyor.
- Gateway `tools.effective` bağlam çözümlemesi artık sağlayıcı, hesap,
  hedef, iş parçacığı ve yanıt modu girdileri için türü belirlenmiş SQLite
  teslimat/yönlendirme satırlarını okuyor. Artık bu etkin yönlendirme alanlarını
  eski `session_entries.entry_json` kaynak gölgelerinden kurtarmıyor.
- Gerçek zamanlı sesli danışma yönlendirmesi artık üst/arama teslimatını
  türü belirlenmiş ajan başına SQLite oturum satırlarından çözümlüyor. Gömülü
  ajan ileti rotasını seçerken artık uyumluluk `SessionEntry.deliveryContext` gölgelerine
  geri dönmüyor.
- ACP oluşturma Heartbeat aktarımı ve üst akış yönlendirmesi artık üst
  teslimatı türü belirlenmiş SQLite oturum satırlarından okuyor. Artık üst
  teslimat bağlamını uyumluluk oturum girdisi gölgelerinden yeniden oluşturmuyor.
- Oturum teslimat rotasını koruma artık türü belirlenmiş sohbet meta
  verilerini ve kalıcı teslimat sütunlarını izliyor. Artık kanal ipuçlarını,
  doğrudan/ana işaretlerini veya iş parçacığı biçimini `sessionKey`
  içinden çıkarmıyor; dahili web sohbeti rotaları yalnızca SQLite oturum için
  türü belirlenmiş/kalıcı teslimat kimliğine zaten sahipse harici bir hedefi
  devralıyor.
- Genel oturum teslimatı çıkarımı artık yalnızca tam olarak eşleşen türü
  belirlenmiş SQLite oturum teslimat satırını okuyor. Artık iş parçacığı/konu
  son eklerini ayrıştırmıyor veya iş parçacığı biçimli bir anahtardan temel
  oturum anahtarına geri dönmüyor.
- Yanıt gönderimi, yeniden başlatma nöbetçisi kurtarması ve gerçek zamanlı
  sesli danışma yönlendirmesi artık iş parçacığı yönlendirmesi için tam olarak
  eşleşen türü belirlenmiş SQLite oturum/konuşma satırlarını kullanıyor. Artık
  iş parçacığı kimliklerini veya temel oturum teslimat bağlamını iş parçacığı
  biçimli oturum anahtarlarını ayrıştırarak kurtarmıyor.
- Gömülü PI geçmiş sınırlaması artık sağlayıcı, sohbet türü ve eş kimliği
  için türü belirlenmiş SQLite oturum yönlendirme izdüşümünü
  (`sessions` + birincil `conversations`) kullanıyor. Artık
  sağlayıcı, DM, grup veya iş parçacığı biçimini `sessionKey` içinden
  ayrıştırmıyor.
- Cron aracı teslimat çıkarımı artık yalnızca açık teslimatı veya mevcut türü
  belirlenmiş teslimat bağlamını kullanıyor. Artık kanal, eş, hesap veya iş
  parçacığı hedeflerinin kodunu `agentSessionKey` içinden çözmüyor.
- Çalışma zamanı oturum satırları artık eski `lastProvider` rota takma
  adını taşımıyor. Yardımcılar ve testler türü belirlenmiş `lastChannel`
  ve `deliveryContext` alanlarını kullanıyor; eski rota takma adlarını veya
  kalıcı `origin` gölgelerini yalnızca doctor geçişi dönüştürmelidir.
- Transkript olayları, VFS satırları ve araç yapıtı satırları artık ajan
  başına veritabanına yazılıyor. Yayımlanmamış küresel transkript dosyası
  eşleme tablosu kaldırıldı; doctor bunun yerine eski kaynak yollarını kalıcı
  geçiş satırlarına kaydediyor.
- Çalışma zamanı transkript araması artık JSONL bayt uzaklıklarını taramıyor
  veya eski transkript dosyalarını yoklamıyor. Gateway sohbet/medya/geçmiş
  yolları transkript satırlarını SQLite'tan okuyor; oturum JSONL'si artık bir
  çalışma zamanı durumu veya dışa aktarma biçimi değil, yalnızca eski bir doctor
  girdisidir.
- Transkript üst ve dal ilişkileri, yol benzeri `agent-db:...transcript_events...`
  konumlandırıcı dizeleri yerine SQLite transkript başlıklarındaki yapılandırılmış
  `parentTranscriptScope: {agentId, sessionId}` meta verilerini kullanıyor.
- Transkript yöneticisi sözleşmesi artık örtük kalıcı
  `create(cwd)` veya `continueRecent(cwd)` oluşturucularını sunmuyor. Kalıcı
  transkript yöneticileri açık bir `{agentId, sessionId}` kapsamıyla açılıyor; yalnızca
  bellek içi yöneticiler, testler ve saf transkript dönüşümleri için kapsamdan bağımsız kalır.
- Çalışma zamanı transkript deposu API'leri dosya sistemi yollarını değil, SQLite kapsamını çözümler. Eski
  `resolve...ForPath` yardımcısı ve kullanılmayan `transcriptPath` yazma seçenekleri
  çalışma zamanı çağıranlarından kaldırıldı.
- Çalışma zamanı oturum çözümlemesi artık `{agentId, sessionId}` kullanır ve harici
  sınırlar için `sqlite-transcript://<agent>/<session>` dizgeleri türetmemelidir.
  Eski mutlak JSONL yolları yalnızca doctor geçiş girdileridir.
- Yerel hook aktarıcısının doğrudan köprü kayıtları artık aktarım kimliğine göre anahtarlanan, türü belirlenmiş paylaşılan
  `native_hook_relay_bridges` satırlarında bulunur. Çalışma zamanı artık bu kısa ömürlü köprü
  kayıtları için bir `/tmp` JSON kayıt defteri veya opak genel kayıtlar yazmaz.
- `runEmbeddedPiAgent(...)` artık bir transkript konumlandırıcı parametresine sahip değildir.
  Hazırlanmış worker tanımlayıcıları da transkript konumlandırıcılarını içermez. Çalışma zamanı oturum
  durumu ve kuyruğa alınmış takip çalıştırmaları, türetilmiş transkript tanıtıcıları yerine
  `{agentId, sessionId}` taşır.
- Gömülü Compaction artık SQLite kapsamını `agentId` ve `sessionId` üzerinden alır.
  Compaction hook'ları, bağlam motoru çağrıları, CLI yetkilendirmesi ve protokol yanıtları
  türetilmiş `sqlite-transcript://...` tanıtıcılarını almamalıdır. Dışa aktarma/hata ayıklama kodu
  satırlardan açık kullanıcı yapıtları oluşturabilir, ancak genel bir oturum JSONL dışa aktarma
  yolu sağlamaz veya dosya adlarını yeniden çalışma zamanı kimliğine beslemez.
- `/export-session` transkript satırlarını SQLite'tan okur ve yalnızca istenen
  bağımsız HTML görünümünü yazar. Gömülü görüntüleyici artık bu satırlardan oturum JSONL'sini
  yeniden oluşturmaz veya indirmez.
- Bağlam motoru yetkilendirmesi artık agent kimliğini kurtarmak için bir transkript
  konumlandırıcısını ayrıştırmaz. Hazırlanmış çalışma zamanı bağlamı, çözümlenen `agentId`
  değerini yerleşik Compaction bağdaştırıcısına taşır.
- Transkript yeniden yazma ve canlı araç sonucu kısaltma artık transkript
  durumunu `{agentId, sessionId}` ile okur ve kalıcılaştırır; transkript güncelleme olayı yükleri için
  geçici konumlandırıcılar türetmez.
- Transkript durumu yardımcı yüzeyi artık konumlandırıcı tabanlı
  `readTranscriptState`, `replaceTranscriptStateEvents` veya
  `persistTranscriptStateMutation` çeşitlerine sahip değildir. Çalışma zamanı çağıranları
  `{agentId, sessionId}` API'lerini kullanmalıdır. Doctor içe aktarma işlemi eski dosyaları açık dosya
  yoluyla okur ve SQLite satırları yazar; konumlandırıcı dizgelerini taşımaz.
- Çalışma zamanı oturum yöneticisi sözleşmesi artık `open(locator)`,
  `forkFrom(locator)` veya `setTranscriptLocator(...)` sunmaz. Kalıcı oturum
  yöneticileri yalnızca `{agentId, sessionId}` ile açılır; listeleme/çatallama yardımcıları,
  transkript yöneticisi cephesi yerine satır odaklı oturum ve denetim noktası API'lerinde bulunur.
- Gateway transkript okuyucu API'leri kapsam önceliklidir. Bunlar
  `{agentId, sessionId}` alır ve yanlışlıkla çalışma zamanı kimliğine dönüşebilecek konumsal bir
  transkript konumlandırıcısını kabul etmez. Etkin transkript konumlandırıcısı ayrıştırması
  kaldırılmıştır; eski kaynak yolları yalnızca doctor içe aktarma kodu tarafından okunur.
- Transkript güncelleme olayları da kapsam önceliklidir. `emitSessionTranscriptUpdate`
  artık çıplak bir konumlandırıcı dizgesini kabul etmez ve dinleyiciler bir tanıtıcıyı
  ayrıştırmadan `{agentId, sessionId}` ile yönlendirme yapar.
- Gateway oturum iletisi yayını, oturum anahtarlarını bir transkript
  konumlandırıcısından değil, agent/oturum kapsamından çözümler. Eski transkript-konumlandırıcıdan-oturum
  anahtarına çözümleyici/önbellek kaldırılmıştır.
- Gateway oturum geçmişi SSE'si, canlı güncellemeleri agent/oturum kapsamına göre süzer. Bir
  akışın güncelleme alıp almaması gerektiğine karar vermek için artık transkript konumlandırıcı
  adaylarını, gerçek yolları veya dosya biçimli transkript kimliklerini standartlaştırmaz.
- Oturum yaşam döngüsü hook'ları artık `session_end` üzerinde transkript
  konumlandırıcıları türetmez veya sunmaz. Hook tüketicileri `sessionId`, `sessionKey`, sonraki oturum
  kimliklerini ve agent bağlamını alır; transkript dosyaları yaşam döngüsü
  sözleşmesinin parçası değildir.
- Sıfırlama hook'ları da artık transkript konumlandırıcıları türetmez veya sunmaz.
  `before_reset` yükü, kurtarılan SQLite iletilerini ve sıfırlama nedenini taşırken
  oturum kimliği hook bağlamında kalır.
- Agent test düzeneği sıfırlaması artık bir transkript konumlandırıcısını kabul etmez. Sıfırlama gönderimi,
  neden ile birlikte `sessionId`/`sessionKey` kapsamına alınır.
- Agent uzantısı oturum türleri artık `transcriptLocator` sunmaz; uzantılar
  dosya biçimli bir transkript kimliğine erişmek yerine oturum bağlamını ve çalışma zamanı
  API'lerini kullanmalıdır.
- Plugin Compaction hook'ları artık transkript konumlandırıcılarını sunmaz. Hook bağlamı
  zaten oturum kimliğini taşır ve transkript okumaları dosya biçimli tanıtıcılar yerine SQLite
  kapsamına duyarlı API'lerden geçmelidir.
- `before_agent_finalize` hook'ları, yerel hook aktarıcısı yükleri dâhil olmak üzere artık
  `transcriptPath` sunmaz. Sonlandırma hook'ları yalnızca oturum bağlamını kullanır.
- Gateway sıfırlama yanıtları artık döndürülen girdide bir transkript
  konumlandırıcısı sentezlemez. Sıfırlama, SQLite transkript satırlarını oluşturur, temiz
  oturum girdisini döndürür ve transkript erişimini kapsam duyarlı okuyuculara bırakır.
- Gömülü çalıştırma ve Compaction sonuçları artık oturum muhasebesi için transkript
  konumlandırıcılarını sunmaz. Otomatik Compaction yalnızca etkin `sessionId`,
  Compaction sayaçlarını ve token meta verilerini günceller.
- Gömülü deneme sonuçları artık `transcriptLocatorUsed` döndürmez ve
  bağlam motoru `compact()` sonuçları artık transkript konumlandırıcılarını döndürmez.
  Çalışma zamanı yeniden deneme döngüleri yalnızca ardıl bir `sessionId` kabul eder.
- Teslimat yansısı transkript ekleme sonuçları artık transkript
  konumlandırıcılarını döndürmez. Çağıranlar eklenen `messageId` değerini alır; transkript güncelleme sinyalleri
  SQLite kapsamını kullanır.
- Üst oturum çatallama yardımcıları yalnızca çatallanan `sessionId` değerini döndürür. Alt agent
  hazırlığı, alt agent/oturum kapsamını motorlara iletir.
- CLI çalıştırıcı parametreleri ve geçmişin yeniden tohumlanması artık transkript konumlandırıcılarını kabul etmez.
  CLI geçmiş okumaları, SQLite transkript kapsamını `{agentId,
sessionId}` ve oturum anahtarı bağlamından çözümler.
- CLI ve gömülü çalıştırıcı test fikstürleri artık etkin oturumları `*.jsonl` dosyalarıymış gibi göstermek veya
  çalışma zamanı parametrelerinden bir `sqlite-transcript://...` dizgesi geçirmek yerine SQLite transkript satırlarını
  oturum kimliğine göre tohumlar ve okur.
- Oturum araç sonucu koruması olayları, bellek içi bir yöneticinin türetilmiş bir
  konumlandırıcısı olmadığında bile bilinen oturum kapsamından yayınlanır. Testleri artık etkin
  `/tmp/*.jsonl` transkript dosyalarını taklit etmez.
- BTW ve Compaction denetim noktası yardımcıları artık transkript satırlarını
  SQLite kapsamına göre okur ve çatallar. Denetim noktası meta verileri artık yalnızca oturum kimliklerini
  ve yaprak/girdi kimliklerini saklar; türetilmiş konumlandırıcılar artık denetim noktası yüklerine yazılmaz.
- Gateway transkript anahtarı araması protokol sınırlarında SQLite transkript
  kapsamını kullanır ve artık transkript dosya adlarının gerçek yolunu çözümlemez veya istatistiklerini almaz.
- Otomatik Compaction transkript döndürmesi, ardıl transkript satırlarını
  doğrudan SQLite transkript deposu üzerinden yazar. Oturum satırları, kalıcı bir JSONL yolu veya
  kalıcılaştırılmış konumlandırıcıyı değil, yalnızca ardıl oturum kimliğini tutar.
- Gömülü bağlam motoru Compaction işlemi, SQLite adlandırmalı transkript döndürme
  yardımcılarını kullanır. Döndürme testleri artık JSONL ardıl yolları oluşturmaz veya
  etkin oturumları dosya olarak modellemez.
- Yönetilen giden görüntü saklama, transkript iletisi önbelleğini
  dosya sistemi stat çağrıları yerine SQLite transkript istatistiklerinden anahtarlar.
- Çalışma zamanı oturum kilitleri ve bağımsız eski `.jsonl.lock` doctor
  hattı kaldırılmıştır.
- Microsoft Teams çalışma zamanı barrel'ı ve genel Plugin SDK artık
  eski dosya kilidi yardımcısını yeniden dışa aktarmaz; kalıcı Plugin durum yolları SQLite desteklidir.
- Oturum yaşına/sayısına göre budama ve açık oturum temizliği kaldırılmıştır.
  Eski içe aktarmanın sahibi doctor'dır; eski oturumlar açıkça sıfırlanır veya silinir.
- Doctor bütünlük kontrolleri artık eski bir JSONL dosyasını bir SQLite oturum satırı için geçerli etkin
  transkript olarak saymaz. Etkin transkript sağlığı yalnızca SQLite'a dayanır;
  eski JSONL dosyaları geçiş/yetim temizleme girdileri olarak bildirilir.
- Doctor artık `agents/<agent>/sessions/` değerini gerekli çalışma zamanı
  durumu olarak değerlendirmez. Bu dizini yalnızca zaten mevcutsa, eski içe aktarma
  veya yetim temizleme girdisi olarak tarar.
- Gateway `sessions.resolve`, oturum yama/sıfırlama/Compaction yolları, alt agent
  oluşturma, hızlı iptal, ACP meta verileri, Heartbeat yalıtımlı oturumlar ve TUI
  yamalama artık normal çalışma zamanı işinin yan etkisi olarak eski oturum anahtarlarını
  taşımaz veya budamaz.
- CLI komutu oturum çözümlemesi artık bir `storePath` yerine sahibi olan
  `agentId` değerini döndürür ve normal `--to` veya `--session-id`
  çözümlemesi sırasında artık eski ana oturum satırlarını kopyalamaz. Eski ana satır standartlaştırması
  yalnızca doctor'a aittir.
- Çalışma zamanı alt agent derinlik çözümlemesi artık `sessions.json` veya JSON5
  oturum depolarını okumaz. SQLite `session_entries` değerini agent kimliğine göre okur ve eski
  derinlik/oturum meta verileri yalnızca doctor içe aktarma yolu üzerinden girebilir.
- Kimlik doğrulama profili oturum geçersiz kılmaları, dosya biçimli bir oturum deposu çalışma zamanını
  tembel yüklemek yerine doğrudan `{agentId, sessionKey}` satır upsert işlemleriyle kalıcılaştırılır.
- Otomatik yanıt ayrıntılılık geçidi ve oturum güncelleme yardımcıları artık SQLite
  oturum satırlarını oturum kimliğine göre okur/upsert eder ve kalıcı satır durumuna dokunmadan önce
  eski bir depo yolu gerektirmez.
- Komut çalıştırma oturum meta verisi yardımcıları artık girdi odaklı adları ve modül
  yollarını kullanır; eski `session-store` komut yardımcı yüzeyi kaldırılmıştır.
- Önyükleme başlığı tohumlama ve manuel Compaction sınırı sağlamlaştırma artık
  SQLite transkript satırlarını doğrudan değiştirir. Çalışma zamanı çağıranları yazılabilir
  `.jsonl` yolları değil, oturum kimliğini iletir.
- Sessiz oturum döndürme yeniden oynatımı, son kullanıcı/asistan konuşma sıralarını
  SQLite transkript satırlarından `{agentId, sessionId}` ile kopyalar. Artık kaynak veya hedef
  transkript konumlandırıcılarını kabul etmez.
- Yeni çalışma zamanı oturum satırları artık transkript konumlandırıcılarını saklamaz. Çağıranlar
  doğrudan `{agentId, sessionId}` kullanır; dışa aktarma/hata ayıklama komutları satırları oluştururken
  çıktı dosyası adlarını seçebilir.
- Yeni bir kalıcı transkript oturumu başlatmak artık SQLite satırlarını her zaman
  kapsama göre açar. Oturum yöneticisi artık önceki dosya dönemi transkript
  yolunu veya konumlandırıcısını yeni oturumun kimliği olarak yeniden kullanmaz.
- Kalıcı transkript oturumları açık
  `openTranscriptSessionManagerForSession({agentId, sessionId})` API'sini kullanır. Eski
  statik `SessionManager.create/openForSession/list/forkFromSession` cepheleri
  kaldırılmıştır; böylece testler ve çalışma zamanı kodu yanlışlıkla dosya dönemi oturum
  keşfini yeniden oluşturamaz.
- Plugin çalışma zamanı artık `api.runtime.agent.session.resolveTranscriptLocatorPath` sunmaz;
  Plugin kodu SQLite satır yardımcılarını ve kapsam değerlerini kullanır.
- Genel `session-store-runtime` SDK yüzeyi artık yalnızca oturum satırı
  ve transkript satırı yardımcılarını dışa aktarır. Odaklanmış SQLite şema/yol/işlem yardımcıları
  `sqlite-runtime` içinde bulunur; ham açma/kapatma/sıfırlama yardımcıları birinci taraf
  testleri için yalnızca yerel olarak kalır.
- Eski `.jsonl` yörünge/denetim noktası dosya adı sınıflandırıcıları artık
  doctor eski oturum dosyası modülünde bulunur. Çekirdek oturum doğrulaması artık normal
  SQLite oturum kimliklerine karar vermek için dosya yapıtı yardımcılarını içe aktarmaz.
- Active Memory engelleyen alt agent çalıştırmaları, Plugin durumu altında geçici veya kalıcı
  `session.jsonl` dosyaları oluşturmak yerine SQLite transkript satırlarını kullanır. Eski
  `transcriptDir` seçeneği kaldırılmıştır.
- Tek seferlik slug oluşturma ve sistem agent'ı planlayıcı çalıştırmaları, geçici
  `session.jsonl` dosyaları oluşturmak yerine SQLite transkript satırlarını kullanır.
- `llm-task` yardımcı çalıştırmaları ve gizli taahhüt çıkarımı da SQLite
  transkript satırlarını kullanır; dolayısıyla yalnızca modele yönelik bu yardımcı oturumlar artık
  geçici JSON/JSONL transkript dosyaları oluşturmaz.
- `TranscriptSessionManager` artık yalnızca açılmış bir SQLite transkript kapsamıdır.
  Çalışma zamanı kodu bunu `openTranscriptSessionManagerForSession({agentId,
sessionId})` ile açar; oluşturma, dallandırma, sürdürme, listeleme ve çatallama akışları,
  statik yönetici cepheleri yerine bunların sahibi olan SQLite satır yardımcılarında bulunur.
  Doctor/içe aktarma/hata ayıklama kodu, açık eski kaynak dosyalarını
  çalışma zamanı oturum yöneticisinin dışında işler.
- Eski `SessionManager.newSession()` ve
  `SessionManager.createBranchedSession()` cephe yöntemleri kaldırıldı. Yeni
  oturumlar ve transkript alt öğeleri, zaten açık olan bir yöneticiyi farklı bir
  kalıcı oturuma dönüştürmek yerine bunların sahibi olan SQLite
  iş akışı tarafından oluşturulur.
- Üst transkript çatallama kararları ve çatal oluşturma artık
  `storePath` veya `sessionsDir` kabul etmez; saklanan dosya sistemi yolu meta verileri yerine
  `{agentId, sessionId}` SQLite transkript kapsamını kullanır.
- Memory-host artık işlem yapmayan oturum dizini transkript
  sınıflandırma yardımcılarını dışa aktarmaz; transkript filtreleme artık girdi oluşturulurken SQLite satır
  meta verilerinden türetilir.
- Memory-host ve QMD oturum dışa aktarma testleri SQLite transkript kapsamlarını kullanır. Eski
  `agents/<agentId>/sessions/*.jsonl` yolları yalnızca bir testin
  doctor/içe aktarma/dışa aktarma uyumluluğunu bilinçli olarak kanıtladığı yerlerde kapsanmaya devam eder.
- QA-lab ham oturum incelemesi artık `agents/qa/sessions/sessions.json` okumak yerine gateway
  üzerinden `sessions.list` kullanır; MSteams geri bildirimi
  sahte bir JSONL yolu oluşturmadan doğrudan SQLite transkriptlerine eklenir.
- Paylaşılan gelen kanal dönüşleri artık eski bir
  `storePath` yerine `{agentId, sessionKey}` taşır. LINE, WhatsApp, Slack, Discord, Telegram, Matrix, Signal,
  iMessage, BlueBubbles, Feishu, Google Chat, IRC, Nextcloud Talk, Zalo,
  Zalo Personal, QA Channel, Microsoft Teams, Mattermost, Synology Chat, Tlon,
  Twitch ve QQBot kayıt yolları artık güncellenme zamanı meta verilerini okur ve
  gelen oturum satırlarını SQLite kimliği aracılığıyla kaydeder.
- Transkript konumlandırıcı kalıcılığı etkin oturum satırlarından kaldırıldı.
  `resolveSessionTranscriptTarget`; `agentId`, `sessionId` ve isteğe bağlı
  konu meta verilerini döndürür; eski transkript dosya
  adlarını içe aktaran tek kod doctor'dır.
- Çalışma zamanı transkript üstbilgileri SQLite sürümü `1` ile başlar. Eski JSONL V1/V2/V3
  şekil yükseltmeleri yalnızca doctor içe aktarımında bulunur ve içe aktarılan üstbilgileri,
  satırlar depolanmadan önce geçerli SQLite transkript sürümüne normalleştirir.
- Önce veritabanı koruması artık `SessionManager.listAll` ve
  `SessionManager.forkFromSession` kullanımını yasaklar; oturum listeleme ve çatallama/geri yükleme iş akışları
  satır/kapsamlı SQLite API'lerinde kalmalıdır.
- Koruma ayrıca doctor/içe aktarma kodu dışında eski transkript JSONL ayrıştırma/etkin dal onarımı yardımcı
  adlarını da yasaklar; böylece çalışma zamanı ikinci bir eski
  transkript geçiş yolu geliştiremez.
- Gömülü PI çalıştırmaları, gelen transkript tanıtıcılarını reddeder. Çalışan başlatılmadan önce ve
  deneme transkript durumuna dokunmadan önce yeniden SQLite
  `{agentId, sessionId}` kimliğini kullanırlar. Eski bir `/tmp/*.jsonl` girdisi,
  çalışma zamanı yazma hedefini seçemez.
- Önbellek izleme, Anthropic yükü, ham akış ve tanılama zaman çizelgesi kayıtları
  artık türü belirlenmiş SQLite `diagnostic_events` satırlarına yazılır. Gateway kararlılık paketleri
  artık türü belirlenmiş SQLite `diagnostic_stability_bundles` satırlarına yazılır. Eski
  `diagnostics.cacheTrace.filePath`, `OPENCLAW_CACHE_TRACE_FILE`,
  `OPENCLAW_ANTHROPIC_PAYLOAD_LOG_FILE` ve
  `OPENCLAW_DIAGNOSTICS_TIMELINE_PATH` JSONL geçersiz kılma yolları kaldırıldı ve
  normal kararlılık yakalama artık `logs/stability/*.json` dosyaları yazmaz.
- Cron kalıcılığı artık her kaydetmede tüm iş tablosunu
  silip yeniden eklemek yerine SQLite `cron_jobs` satırlarını uzlaştırır. Plugin hedefi
  geri yazmaları, eşleşen cron satırlarını doğrudan günceller ve çalışma zamanı cron durumunu
  aynı durum veritabanı işleminde tutar.
- Cron çalışma zamanı çağıranları artık kararlı bir SQLite cron deposu anahtarı kullanır. Eski
  `cron.store` yolları yalnızca doctor içe aktarma girdileridir; üretim gateway'i, görev
  bakımı, durum, çalıştırma geçmişi ve Telegram hedefi geri yazma yolları
  `resolveCronStoreKey` kullanır ve artık anahtar yolunu normalleştirmez. Cron durumu artık
  dosya biçimli eski `storePath` alanı yerine `storeKey` bildirir.
- Cron çalışma zamanı yükleme ve zamanlama artık `jobId`, `schedule.cron`, sayısal `atMs`, dize
  boolean'ları veya eksik `sessionTarget` gibi eski kalıcı iş şekillerini normalleştirmez.
  Satırlar SQLite'a eklenmeden önce bu onarımların sahibi doctor eski içe aktarımıdır.
- ACP oluşturma artık transkript JSONL dosya yollarını çözümlemez veya kalıcılaştırmaz. Oluşturma
  ve iş parçacığı bağlama kurulumu, SQLite oturum satırını doğrudan kalıcılaştırır ve
  oturum kimliğini saklanan transkript kimliği olarak tutar.
- ACP oturum meta veri API'leri artık SQLite satırlarını `agentId` temelinde okur/listeler/yukarı ekler ve
  artık `storePath` değerini ACP oturum girdisi sözleşmesinin bir parçası olarak sunmaz.
- Oturum kullanım muhasebesi ve gateway kullanım toplaması artık transkriptleri
  yalnızca `{agentId, sessionId}` ile çözümler. Maliyet/kullanım önbelleği ve keşfedilen oturum
  özetleri artık transkript konumlandırıcı dizeleri oluşturmaz veya döndürmez.
- Gateway sohbet ekleme, iptal-kısmi kalıcılığı, `/sessions.send` ve
  web sohbeti medya transkript yazmaları doğrudan SQLite transkript
  kapsamı üzerinden eklenir. Gateway transkript ekleme yardımcısı artık
  bir `transcriptLocator` parametresi kabul etmez.
- SQLite transkript keşfi artık yalnızca transkript kapsamlarını ve istatistiklerini listeler:
  `{agentId, sessionId, updatedAt, eventCount}`. Kullanılmayan
  `listSqliteSessionTranscriptLocators` uyumluluk yardımcısı ve satır başına
  `locator` alanı kaldırıldı.
- Transkript onarım çalışma zamanı artık yalnızca
  `repairTranscriptSessionStateIfNeeded({agentId, sessionId})` sunar. Konumlandırıcı tabanlı eski
  onarım yardımcısı silindi; doctor/hata ayıklama kodu açık
  kaynak dosya yollarını okur ve konumlandırıcı dizelerini hiçbir zaman taşımaz.
- ACP yeniden oynatma defteri çalışma zamanı artık oturum başına yeniden oynatma satırlarını
  `acp/event-ledger.json` yerine paylaşılan SQLite durum veritabanında depolar; doctor eski dosyayı
  içe aktarır ve kaldırır.
- Gateway transkript okuyucu yardımcıları artık eski
  `session-utils.fs` modül adı yerine
  `src/gateway/session-transcript-readers.ts` içinde bulunur. Yedek yeniden deneme geçmişi denetimi,
  eski dosya yardımcısı yüzeyi yerine SQLite transkript içeriğine göre adlandırılır.
- Gateway'e eklenen sohbet ve compaction yardımcıları artık değerleri transkript yolları veya
  kaynak dosyalar olarak adlandırmak yerine SQLite transkript kapsamını
  dahili yardımcı API'lerinden geçirir.
- Bootstrap sürdürme algılaması artık SQLite transkript satırlarını
  `hasCompletedBootstrapTranscriptTurn` aracılığıyla denetler; artık dosya biçimli
  bir yardımcı adı sunmaz.
- Gömülü çalıştırıcı testleri artık SQLite transkript kimliğini kullanır ve yeni bir
  transkript yöneticisinin açılması her zaman açık bir `sessionId` gerektirir.
- Bellek dizinleme yardımcıları artık baştan sona SQLite transkript terminolojisini kullanır:
  ana makine `listSessionTranscriptScopesForAgent` ve
  `sessionTranscriptKeyForScope` dışa aktarır, hedefli eşitleme `sessionTranscripts` kuyruğa alır,
  genel oturum arama isabetleri opak `transcript:<agent>:<session>` yollarını sunar
  ve dahili DB kaynak anahtarı sahte bir dosya yolu yerine
  `source_kind='sessions'` altında `session:<session>` olur.
- Genel Plugin SDK kalıcı yinelenenleri ayıklama yardımcısı artık dosya biçimli
  seçenekler sunmaz. Çağıranlar SQLite kapsam anahtarları sağlar ve kalıcı yinelenenleri ayıklama satırları
  paylaşılan Plugin durumunda bulunur.
- Microsoft Teams SSO belirteçleri kilitli JSON dosyalarından SQLite Plugin
  durumuna taşındı. Doctor, `msteams-sso-tokens.json` öğesini içe aktarır, yüklerden kurallı SSO belirteç
  anahtarlarını yeniden oluşturur ve kaynak dosyayı kaldırır. Yetki verilmiş OAuth belirteçleri
  mevcut özel kimlik bilgisi dosyası sınırlarında kalır.
- Matrix eşitleme önbelleği durumu `bot-storage.json` konumundan SQLite Plugin
  durumuna taşındı. Doctor, eski ham veya sarmalanmış eşitleme yüklerini içe aktarır ve
  kaynak dosyayı kaldırır. Etkin Matrix ve QA Lab Matrix bağdaştırıcısı istemcileri, sahte bir
  `sync-store.json` veya `bot-storage.json` yolu değil, SQLite eşitleme deposu kök dizini geçirir.
- Matrix eski kriptografi geçiş durumu
  `legacy-crypto-migration.json` konumundan SQLite Plugin durumuna taşındı. Doctor eski durum dosyasını içe aktarır;
  Matrix SDK IndexedDB anlık görüntüleri `crypto-idb-snapshot.json` konumundan SQLite Plugin blob'larına taşındı.
  Matrix kurtarma anahtarları ve kimlik bilgileri SQLite Plugin durumu satırlarıdır; bunların eski JSON dosyaları
  yalnızca doctor geçiş girdileridir.
- Memory Wiki etkinlik günlükleri artık `.openclaw-wiki/log.jsonl` yerine
  SQLite Plugin durumunu kullanır. Memory Wiki geçiş sağlayıcısı eski
  JSONL günlüklerini içe aktarır; wiki markdown ve kullanıcı kasası içeriği, çalışma alanı içeriği olarak
  dosya destekli kalır.
- Memory Wiki artık `.openclaw-wiki/state.json` veya kullanılmayan
  `.openclaw-wiki/locks` dizinini oluşturmaz. Geçiş sağlayıcısı, eski bir kasa hâlâ bunları içeriyorsa
  kullanımdan kaldırılan bu Plugin meta veri dosyalarını kaldırır.
- Sistem aracısı denetim girdileri artık `audit/crestodian.jsonl` yerine çekirdek SQLite Plugin durumunu kullanır.
  Doctor eski JSONL denetim günlüğünü içe aktarır ve
  başarılı içe aktarımdan sonra kaldırır.
- Yapılandırma yazma/gözlemleme denetim girdileri artık `logs/config-audit.jsonl` yerine
  çekirdek SQLite Plugin durumunu kullanır. Doctor eski JSONL denetim günlüğünü içe aktarır ve
  başarılı içe aktarımdan sonra kaldırır.
- macOS eşlikçi uygulaması artık `openclaw.json` öğesini düzenlerken uygulamaya yerel
  `logs/config-audit.jsonl` veya `logs/config-health.json` yan dosyalarını yazmaz. Yapılandırma
  dosyası dosya destekli kalır, kurtarma anlık görüntüleri yapılandırma dosyasının yanında kalır
  ve kalıcı yapılandırma denetim/sağlık durumu Gateway SQLite deposuna aittir.
- Sistem aracısı kurtarma bekleyen onayları artık `crestodian/rescue-pending/*.json` veya
  `openclaw/rescue-pending/*.json` yerine çekirdek SQLite Plugin durumunu kullanır.
  Bu kısa ömürlü güvenlik yetenekleri hiçbir zaman içe aktarılmaz; doctor,
  bir yükseltmenin eski bir yazmayı yeniden etkinleştirememesi için kullanımdan kaldırılan her iki dizini de siler.
- Phone Control geçici etkinleştirme durumu artık
  `plugins/phone-control/armed.json` yerine SQLite Plugin durumunu kullanır. Doctor eski etkinleştirilmiş durum
  dosyasını `phone-control/arm-state` ad alanına aktarır ve dosyayı kaldırır.
- Doctor artık JSONL transkriptlerini yerinde onarmaz veya yedek JSONL
  dosyaları oluşturmaz. Etkin dalı SQLite'a aktarır ve eski kaynağı kaldırır.
- Oturum belleği kancası transkript araması, yalnızca kapsamlı
  `{agentId, sessionId}` SQLite okumalarını kullanır. Yardımcısı artık transkript konumlandırıcılarını,
  eski dosya okumalarını veya dosya yeniden yazma seçeneklerini kabul etmez ya da türetmez.
- Codex uygulama sunucusu konuşma bağlamaları artık SQLite Plugin durumunu
  OpenClaw oturum anahtarı veya açık `{agentId, sessionId}` kapsamıyla anahtarlar. Transkript yolu
  yedek bağlamalarını korumamalıdırlar.
- Codex uygulama sunucusu yansıtılmış geçmiş okumaları yalnızca SQLite transkript kapsamını kullanır;
  kimliği transkript dosya yollarından kurtarmamalıdır.
- Rol sıralama ve compaction sıfırlama yolları artık eski transkript
  dosyalarının bağlantısını kaldırmaz; sıfırlama yalnızca SQLite oturum satırını ve transkript kimliğini döndürür.
- Gateway sıfırlama ve denetim noktası yanıtları, temiz oturum satırları ile oturum
  kimliklerini döndürür. Artık istemciler için SQLite transkript konumlandırıcıları oluşturmazlar.
- Memory-core dreaming artık eksik JSONL dosyalarını arayarak oturum satırlarını temizlemez.
  Alt aracı temizliği, dosya sistemi varlık denetimleri yerine oturum çalışma zamanı API'si üzerinden yapılır.
  Transkript alımı testleri, `agents/<id>/sessions` sabitleri veya konumlandırıcı
  yer tutucuları oluşturmak yerine SQLite satırlarını doğrudan başlangıç verileriyle doldurur.
- Bellek transkript dizinleme, alıntı/okuma yardımcıları için sanal bir
  arama isabeti yolu olarak `transcript:<agentId>:<sessionId>` sunabilir. Kalıcı dizin kaynağı
  ilişkiseldir (`source_kind='sessions'`, `source_key='session:<sessionId>'`,
  `session_id=<sessionId>`), dolayısıyla değer bir çalışma zamanı transkript konumlandırıcısı
  veya dosya sistemi yolu değildir ve oturum çalışma zamanı API'lerine asla geri aktarılmamalıdır.
- Gateway doctor bellek durumu, kısa süreli hatırlama ve aşama sinyali sayılarını
  `memory/.dreams/*.json` yerine SQLite Plugin durum satırlarından okur; CLI ve
  doctor çıktısı artık bu depolama alanını bir yol olarak değil, SQLite deposu olarak etiketler.
- Memory-core çalışma zamanı, CLI durumu, Gateway doctor yöntemleri ve Plugin SDK
  cepheleri artık eski `.dreams/session-corpus` dosyalarını denetlemez veya arşivlemez.
  Bu dosyalar yalnızca taşıma girdileridir; doctor bunları SQLite'a aktarır ve
  doğrulamadan sonra kaynağı siler. Etkin oturum alımı kanıt satırları
  artık sanal SQLite yolu `memory/session-ingestion/<day>.txt` kullanır; çalışma zamanı
  hiçbir zaman `.dreams/session-corpus` konumuna durum yazmaz veya buradan durum türetmez.
- Memory-core genel yapıtları, SQLite ana bilgisayar olaylarını sanal JSON
  yapıtı `memory/events/memory-host-events.json` olarak sunar; artık eski
  `.dreams/events.jsonl` kaynak yolunu yeniden kullanmaz.
- Korumalı alan kapsayıcı/tarayıcı kayıtları artık türü belirlenmiş oturum, imaj, zaman damgası,
  arka uç/yapılandırma ve tarayıcı bağlantı noktası sütunlarıyla paylaşılan
  `sandbox_registry_entries` SQLite tablosunu kullanır. Doctor, eski yekpare ve
  parçalanmış JSON kayıt dosyalarını içe aktarır ve başarıyla aktarılan kaynakları kaldırır. Çalışma zamanı okumaları
  doğruluk kaynağı olarak türü belirlenmiş satır sütunlarını kullanır; `entry_json` yalnızca bir yeniden oynatma/hata ayıklama
  kopyasıdır.
- Taahhütler artık tüm depoyu içeren bir JSON blobu yerine türü belirlenmiş, paylaşılan bir
  `commitments` tablosu kullanır. Çalışma zamanı; dizine alınmış kapsam, teslimat aralığı, kayan
  üst sınır, durum ve deneme sorgularının yanı sıra eşzamanlı SQLite işlemlerini kullanır;
  `record_json` yalnızca bir yeniden oynatma/hata ayıklama kopyasıdır. Açık doctor onarımı,
  eski `commitments.json` öğesinin tamamını doğrular, daha yeni SQLite satırlarını korur, sonucu
  doğrular ve ancak bundan sonra değişmemiş kaynağı kaldırır. Çalışma zamanı kullanımdan kaldırılan dosyayı
  hiçbir zaman okumaz veya dosyaya yazmaz.
- Web Push abonelikleri ve oluşturulan VAPID kimliği artık türü belirlenmiş, paylaşılan
  `web_push_subscriptions` ve `web_push_vapid_keys` satırlarını kullanır. Çalışma zamanı kaydı,
  süre sonu temizliği ve ilk kullanımda anahtar oluşturma işlemleri satır düzeyinde SQLite
  işlemlerini kullanır. Açık Doctor onarımı, kullanımdan kaldırılan her iki JSON deposunu da
  doğrular, SQLite yazımından önce bunlar üzerinde hak iddia eder, bunları atomik olarak içe aktarır, çakışan
  VAPID kimliklerini reddeder, sonucu doğrular ve ancak bundan sonra
  hak iddialarını kaldırır. Doctor, eski bir Gateway'in kullanımdan kaldırılan dosyaları yeniden oluşturamaması için
  içe aktarma işleminin tamamı boyunca durum dizini bakım kilidini tutar. Kayıt,
  teslimat, silme ve anahtar çözümleme; Doctor bekleyen eski kaynakları
  veya kesintiye uğramış hak iddialarını çözene kadar güvenli biçimde başarısız olur.
- Cron iş tanımları, zamanlama durumu ve çalıştırma geçmişinde artık çalışma zamanı
  JSON yazıcıları veya okuyucuları bulunmaz. Çalışma zamanı; türü belirlenmiş zamanlama,
  yük, teslimat, hata uyarısı, oturum, durum ve çalışma zamanı durumu sütunlarının yanı sıra
  tanılama, teslimat, oturum/çalıştırma, model ve
  token toplamları için Cron'a ait `task_runs` ayrıntısıyla birlikte `cron_jobs` satırlarını kullanır.
  `job_json` yalnızca bir yeniden oynatma/hata ayıklama kopyasıdır; `state_json`, henüz sık kullanılan sorgu alanları
  bulunmayan iç içe çalışma zamanı tanılamalarını saklarken çalışma zamanı,
  sık kullanılan durum alanlarını türü belirlenmiş sütunlardan yeniden oluşturur. Doctor,
  eski `jobs.json`, `jobs-state.json` ve `runs/*.jsonl` dosyalarını içe aktarır ve içe aktarılan
  kaynakları kaldırır. Plugin hedefi geri yazımları, tüm Cron deposunu yükleyip değiştirmek yerine eşleşen `cron_jobs`
  satırlarını günceller.
- Gateway başlangıcı, çalışma zamanı projeksiyonundaki eski `notify: true` işaretçilerini
  yok sayar. Doctor, kullanımdan kaldırılan ham `cron.webhook` öğesini yalnızca
  bu işaretçileri açık SQLite teslimatına dönüştürürken okur, ardından yapılandırma anahtarını kaldırır.
- Giden ve oturum teslimat kuyrukları artık kuyruk durumunu, girdi türünü,
  oturum anahtarını, kanalı, hedefi, hesap kimliğini, yeniden deneme sayısını, son denemeyi/hatayı,
  kurtarma durumunu ve platform gönderim işaretçilerini paylaşılan
  `delivery_queue_entries` tablosunda türü belirlenmiş sütunlar olarak depolar. Çalışma zamanı kurtarması bu sık kullanılan alanları
  türü belirlenmiş sütunlardan okur; yeniden deneme/kurtarma değişiklikleri de yeniden oynatma JSON'unu tekrar yazmadan
  bu sütunları doğrudan günceller. Tam JSON yükü yalnızca
  ileti gövdeleri ve diğer seyrek kullanılan yeniden oynatma verileri için yeniden oynatma/hata ayıklama blobu olarak kalır.
- Yönetilen giden görüntü kayıtları artık türü belirlenmiş, paylaşılan
  `managed_outgoing_image_records` satırlarını kullanır. Çalışma zamanı yalnızca türü belirlenmiş sütunları okur;
  JSON sütunu bir yeniden oynatma/hata ayıklama kopyasıdır. Özgün görüntü baytları,
  yönetilen medya dizininde adlandırılmış ek yapıtları olarak kalır.
- Discord model seçici tercihleri, komut dağıtım karmaları ve iş parçacığı bağlamaları
  artık paylaşılan SQLite Plugin durumunu kullanır. Bunların eski JSON içe aktarma planları,
  çekirdek taşıma kodunda değil, Discord Plugin kurulum/doctor taşıma yüzeyinde bulunur.
- Plugin eski içe aktarma algılayıcıları,
  `doctor-legacy-state.ts` veya `doctor-state-imports.ts` gibi doctor adlandırmalı modülleri kullanır; normal kanal çalışma zamanı
  modülleri eski JSON algılayıcılarını içe aktarmamalıdır.
- BlueBubbles geçmişi yakalama imleçleri ve gelen ileti tekilleştirme işaretçileri artık paylaşılan SQLite
  Plugin durumunu kullanır. Bunların eski JSON içe aktarma planları, çekirdek taşıma kodunda değil,
  BlueBubbles Plugin kurulum/doctor taşıma yüzeyinde bulunur.
- Telegram güncelleme ofsetleri, çıkartma önbelleği satırları, gönderilen ileti önbelleği satırları,
  konu adı önbelleği satırları ve iş parçacığı bağlamaları artık paylaşılan SQLite Plugin
  durumunu kullanır. Bunların eski JSON içe aktarma planları, çekirdek taşıma kodunda değil,
  Telegram Plugin kurulum/doctor taşıma yüzeyinde bulunur.
- iMessage geçmişi yakalama imleçleri, yanıt kısa kimlik eşlemeleri ve gönderilen yankı tekilleştirme satırları
  artık paylaşılan SQLite Plugin durumunu kullanır. Eski `imessage/catchup/*.json`,
  `imessage/reply-cache.jsonl` ve `imessage/sent-echoes.jsonl` dosyaları
  yalnızca doctor girdileridir.
- Feishu ileti tekilleştirme satırları artık `feishu/dedup/*.json` dosyaları veya
  kullanımdan kaldırılan elle yazılmış `dedup.*` deposu yerine çekirdeğin hak iddia edilebilir tekilleştirme mekanizmasını
  (paylaşılan SQLite Plugin durumundaki `feishu.dedup.*` ad alanları) kullanır;
  yeniden oynatma koruması önbelleği yükseltmeden sonra yeniden oluşturulduğundan eski veriler içe aktarılmaz.
- Microsoft Teams konuşmaları, anketleri, bekleyen yükleme arabellekleri ve geri bildirim
  öğrenimleri artık paylaşılan SQLite Plugin durum/blob tablolarını kullanır. Bekleyen yükleme
  yolu, medya arabelleklerinin base64 JSON yerine SQLite BLOB'ları olarak depolanması için `plugin_blob_entries` kullanır.
  Çalışma zamanı yardımcı adları artık `*-fs` dosya deposu adlandırması yerine SQLite/durum adlandırmasını
  kullanır ve eski `storePath` uyumluluk katmanı bu depolardan kaldırılmıştır. Eski JSON içe aktarma planı,
  Microsoft Teams Plugin kurulum/doctor taşıma yüzeyinde bulunur.
- Zalo tarafından barındırılan giden medya artık `openclaw-zalo-outbound-media` JSON/bin geçici yan dosyaları
  yerine paylaşılan SQLite `plugin_blob_entries` kullanır.
- Fark görüntüleyici HTML'si ve meta verileri artık `meta.json`/`viewer.html` geçici dosyaları
  yerine paylaşılan SQLite `plugin_blob_entries` kullanır. Görüntüleyici HTML'si bir gzip blobu olarak depolanır
  ve yalnızca URL token karması kalıcılaştırılır. İşlenmiş PNG/PDF çıktıları,
  kanal teslimatı hâlâ bir dosya yolu gerektirdiğinden geçici somutlaştırmalar olarak kalır;
  bunların süre sonu meta verilerinin sahibi JSON yan dosyaları olmadan SQLite'tır.
- Canvas yönetilen belgeleri artık varsayılan bir `state/canvas/documents` dizini yerine
  paylaşılan SQLite `plugin_blob_entries` kullanır. Canvas ana bilgisayarı bu blobları
  doğrudan sunar; yerel dosyalar yalnızca açık `host.root`
  operatör içeriği için veya aşağı akıştaki bir medya okuyucusu yol gerektirdiğinde geçici somutlaştırma amacıyla oluşturulur.
- File Transfer denetim kararları artık sınırsız `audit/file-transfer.jsonl` çalışma zamanı günlüğü
  yerine paylaşılan SQLite `plugin_state_entries` kullanır. Doctor,
  eski JSONL denetim dosyasını Plugin durumuna aktarır ve sorunsuz bir içe aktarmadan sonra kaynağı kaldırır.
- ACPX süreç kiralamaları ve Gateway örnek kimliği artık paylaşılan SQLite Plugin
  durumunu kullanır. Doctor, eski `gateway-instance-id` dosyasını Plugin durumuna aktarır
  ve kaynağı kaldırır.
- ACPX tarafından oluşturulan sarmalayıcı betikleri ve yalıtılmış Codex ana dizini,
  kalıcı OpenClaw durumu değil, OpenClaw geçici kökü altındaki geçici somutlaştırmalardır.
  Kalıcı ACPX çalışma zamanı kayıtları, SQLite kiralama ve Gateway örneği satırlarıdır;
  artık buraya çalışma zamanı durumu yazılmadığından eski ACPX `stateDir` yapılandırma yüzeyi kaldırılmıştır.
- Gateway medya ekleri artık kurallı bayt deposu olarak paylaşılan `media_blobs` SQLite tablosunu
  kullanır. Kanal ve korumalı alan uyumluluk yüzeylerine döndürülen yerel yollar,
  kalıcı medya deposu değil, veritabanı satırının geçici somutlaştırmalarıdır. Çalışma zamanı medya izin listeleri artık eski
  `$OPENCLAW_STATE_DIR/media` veya yapılandırma dizini `media` köklerini içermez; bu dizinler
  yalnızca doctor içe aktarma kaynaklarıdır.
- Kabuk tamamlama artık `$OPENCLAW_STATE_DIR/completions/*` önbellek
  dosyalarını yazmaz. Kurulum, doctor, güncelleme ve sürüm duman testi yolları; kalıcı tamamlama önbelleği
  dosyaları yerine oluşturulan tamamlama çıktısını veya profil kaynaklamayı kullanır.
- Gateway Skills yükleme hazırlığı artık paylaşılan `skill_uploads` ve
  `skill_upload_chunks` satırlarını kullanır. Parçalar yükleme sırasında ayrı ayrı işlemsel kalır;
  ardından tamamlama işlemi doğrulanmış tek bir arşiv BLOB'u oluşturur ve parça
  satırlarını kaldırır. Yükleyici yalnızca kurulum devam ederken geçici olarak somutlaştırılmış bir arşiv yolu alır.
  Doctor, geçici yüklemeleri içe aktarmak yerine kullanımdan kaldırılan bir saatlik dosya sistemi
  hazırlık ağacını atar.
- Alt ajan satır içi ekleri artık çalışma alanındaki
  `.openclaw/attachments/*` altında somutlaştırılmaz. Oluşturma yolu SQLite VFS başlangıç girdilerini hazırlar,
  satır içi çalıştırmalar bu girdileri ajan başına çalışma zamanı karalama ad alanına yerleştirir
  ve disk destekli araçlar ek yolları için bu SQLite karalama alanının üzerine katman ekler. Eski
  alt ajan çalıştırması ek dizini kayıt sütunları ve temizleme kancaları kaldırılmıştır.
- CLI görüntü hazırlama artık kararlı `openclaw-cli-images` önbellek
  dosyalarını tutmaz. Harici CLI arka uçları hâlâ dosya yolları alır ancak bu yollar,
  temizlenen çalıştırma başına geçici somutlaştırmalardır.
- Önbellek izi tanılamaları, Anthropic yük tanılamaları, ham model akışı
  tanılamaları, tanılama zaman çizelgesi olayları ve Gateway kararlılık paketleri artık
  `logs/*.jsonl` veya `logs/stability/*.json` dosyaları yerine SQLite satırlarına yazılır.
  Çalışma zamanı yolu geçersiz kılma bayrakları ve ortam değişkenleri kaldırılmıştır; dışa aktarma/hata ayıklama
  komutları dosyaları veritabanı satırlarından açıkça somutlaştırabilir.
- macOS yardımcı uygulamasında artık kayan bir `diagnostics.jsonl` yazıcısı yoktur. Uygulama
  günlükleri birleşik günlük kaydına gider ve kalıcı Gateway tanılamaları SQLite destekli kalır.
- macOS bağlantı noktası koruyucusu kayıt listesi artık bir Application Support JSON dosyası
  veya opak tekil blob yerine türü belirlenmiş, paylaşılan SQLite
  `macos_port_guardian_records` satırlarını kullanır. Makineye özgü bağlantı noktalarını koordine ettikleri için
  tüm macOS uygulama profilleri ana bilgisayar genelindeki aynı yerel veritabanını kullanır. Eski, JSON yazan
  bir uygulama kopyası çalışırken her defter işlemi engellenir. Taşıma, kaynağın anlık görüntüsünü almak
  ve daha sonra yeniden doğrulamak amacıyla eski defterin kararlı dosya kilidi protokolüne yalnızca katılır.
  Her eski satırı, bu kilidi tutmadan canlı komut ve süreç başlangıcı olgularından çözümler;
  ardından yetkili SQLite satırlarını yeniden okur, planı uygular, her makbuzu doğrular ve kaynağı kaldırır.
  Kaldırma yeniden denemeleri, kullanımdan kaldırılan eski makbuzların yeniden ortaya çıkamaması için
  eksik satırları yeniden planlar. SSH oluşturma işlemini yaptıktan sonra eski bir yazıcıyı mahsur bırakmaması için
  kilit kısa ömürlü kalır. Geçiş kasıtlı olarak tek yönlüdür: kararlı durum çalışma zamanı JSON'u asla okumaz,
  yansıtmaz veya yazmaz ve yalnızca JSON kullanan yapılara geri dönüldüğünde daha yeni SQLite makbuzları korunmaz.
- Gateway tekil kilitleri artık geçici dizin kilit dosyaları yerine
  `gateway_locks` kapsamı altında türü belirlenmiş, paylaşılan SQLite `state_leases` satırlarını kullanır.
  Fly ve OAuth sorun giderme belgeleri artık eski dosya kilidi temizliği yerine
  SQLite kiralama/kimlik doğrulama yenileme kilidine yönlendirir.
- Gateway yeniden başlatma sentinel durumu artık `restart-sentinel.json` yerine türü belirlenmiş paylaşılan SQLite
  `gateway_restart_sentinel` satırlarını kullanıyor; çalışma zamanı sentinel türünü, durumunu, yönlendirmesini, iletisini, devam bilgisini ve istatistiklerini
  türü belirlenmiş sütunlardan okuyor. Bu sütunlar yetkili kaynaktır; `payload_json` yalnızca bir
  yeniden oynatma/hata ayıklama gölgesidir. Çalışma zamanındaki okuma, yazma ve temizleme yolları yalnızca SQLite kullanır.
  Sınırları belirlenmiş tek bir durum taşıma modülü, normal yeniden başlatma kurtarmasından önce
  doğrulanmış eski bir güncelleme sonrası sentinel verisini içe aktarmak, türü belirlenmiş satırı doğrulamak
  ve kaynak dosyayı kaldırmak için başlangıçta ve Doctor sırasında çalışır. Kararlı durumdaki hiçbir çalışma zamanı modülü
  eski dosyayı okumaz, yazmaz veya temizlemez.
- Gateway yeniden başlatma amacı ve gözetmen devretme durumu artık
  `gateway-restart-intent.json` ve
  `gateway-supervisor-restart-handoff.json` yan dosyaları yerine türü belirlenmiş paylaşılan
  SQLite `gateway_restart_intent` ve `gateway_restart_handoff` satırlarını kullanıyor.
- Gateway tekil örnek koordinasyonu artık `gateway.<hash>.lock` dosyalarını yazmak yerine
  `gateway_locks` altında türü belirlenmiş `state_leases` satırlarını kullanıyor. Kira satırı
  kilit sahibini, sona erme zamanını, heartbeat bilgisini ve hata ayıklama yükünü barındırır; atomik
  edinme/serbest bırakma sınırını SQLite yönetir. Kullanımdan kaldırılan dosya kilidi dizini seçeneği
  kaldırıldı; testler doğrudan SQLite satır kimliğini kullanıyor.
- `cron/runs/*.jsonl` dosyalarını tarayan eski ve başvurulmayan Cron kullanım raporu yardımcısı
  silindi. Cron çalıştırma geçmişi raporları, Cron'un sahip olduğu `task_runs` satırlarını okuyor.
- Ana oturum yeniden başlatma kurtarması artık `agents/*/sessions`
  dizinlerini taramak yerine SQLite `agent_databases` kayıt defteri aracılığıyla aday ajanları keşfediyor.
- Gemini oturum bozulması kurtarması artık yalnızca SQLite oturum satırını siliyor;
  eski bir `storePath` geçidine ihtiyaç duymuyor veya türetilmiş bir
  transkript JSONL yolunun bağlantısını kaldırmayı denemiyor.
- Yol geçersiz kılma işlemi artık değişmez `undefined`/`null` ortam
  değerlerini ayarlanmamış olarak kabul ederek testler veya kabuk devretmeleri sırasında depo kökünde yanlışlıkla
  `undefined/state/*.sqlite` veritabanlarının oluşturulmasını önlüyor.
- Yapılandırma sağlığı parmak izleri artık `logs/config-health.json` yerine türü belirlenmiş paylaşılan SQLite
  `config_health_entries` satırlarını kullanarak normal yapılandırma dosyasını
  kimlik bilgisi olmayan tek yapılandırma belgesi olarak tutuyor. macOS yardımcı uygulaması yalnızca
  işlem yerelindeki sağlık durumunu tutuyor ve eski JSON yan dosyasını yeniden oluşturmuyor.
- Kimlik doğrulama profili çalışma zamanı artık kimlik bilgisi JSON dosyalarını içe aktarmıyor veya yazmıyor.
  Yetkili kimlik bilgisi deposu SQLite'tır; `auth-profiles.json`, ajan başına
  `auth.json` ve paylaşılan `credentials/oauth.json`, içe aktarıldıktan sonra
  kaldırılan Doctor taşıma girdileridir.
- Kimlik doğrulama profili kaydetme/durum testleri artık türü belirlenmiş SQLite kimlik doğrulama tablolarını doğrudan
  doğruluyor ve eski kimlik doğrulama profili dosya adlarını yalnızca Doctor taşıma girdileri olarak kullanıyor.
- `openclaw secrets apply` yalnızca yapılandırma dosyasını, ortam dosyasını ve SQLite
  kimlik doğrulama profili deposunu temizliyor. Artık kullanımdan kaldırılmış ajan başına
  `auth.json` dosyasını düzenleyen uyumluluk mantığını taşımıyor; bu dosyayı içe aktarma ve silme işlemlerinin sahibi Doctor'dır.
- Hermes gizli bilgi taşıma planları, içe aktarılan API anahtarı profillerini doğrudan
  SQLite kimlik doğrulama profili deposuna uygular. Artık ara hedef olarak
  `auth-profiles.json` dosyasını yazmıyor veya doğrulamıyor.
- Kullanıcıya yönelik kimlik doğrulama belgeleri artık kullanıcılara
  `auth-profiles.json` dosyasını incelemelerini veya kopyalamalarını söylemek yerine
  `state/openclaw.sqlite#table/auth_profile_stores/<agentDir>` öğesini açıklıyor; eski OAuth/kimlik doğrulama JSON
  adları yalnızca Doctor içe aktarma girdileri olarak belgelenmeye devam ediyor.
- MCP OAuth oturumları artık paylaşılan `state/openclaw.sqlite` içindeki sürümlü
  `mcp_oauth_stores` satırlarını kullanıyor. SDK'ya ait belirteç, istemci kaydı ve keşif
  nesneleri, bağımlılık uzantısı alanlarının korunması için doğrulanmış tek bir JSON yükü olarak kalırken
  her okuma/değiştirme/yazma işlemi kısa tek bir Kysely
  işlemi içinde kaydediliyor. Paylaşılan tek bir SQLite kirası yenileme, oturum açma ve oturum kapatma işlemlerini serileştiriyor;
  gömülü MCP aktarımları artık MCP SDK'nın bu kiranın dışında yenileme yapmasına
  izin vermiyor. Doctor, kullanımdan kaldırılmış `mcp-oauth/*.json`
  depolarını kaynak makbuzlarıyla yalnızca kendisi içe aktarıp kaldırıyor ve çalışma zamanında dosya geri dönüşü bulunmuyor.
- Çekirdek durum yolu yardımcıları artık kullanımdan kaldırılmış `credentials/oauth.json`
  dosyasını sunmuyor. Eski dosya adı yalnızca Doctor kimlik doğrulama içe aktarma yolunda yereldir.
- Kurulum, güvenlik, ilk kullanım, model kimlik doğrulaması ve SecretRef belgeleri artık
  ajan başına kimlik doğrulama profili JSON dosyaları yerine SQLite kimlik doğrulama profili satırlarını ve tüm durumun yedeklenmesini/taşınmasını açıklıyor.
- PI model keşfi artık kurallı kimlik bilgilerini bellek içi
  `pi-coding-agent` kimlik doğrulama deposuna geçiriyor. Keşif sırasında artık
  ajan başına `auth.json` oluşturmuyor, temizlemiyor veya yazmıyor.
- Voice Wake tetikleyici ve yönlendirme ayarları artık `settings/voicewake.json`, `settings/voicewake-routing.json` veya
  opak genel satırlar yerine türü belirlenmiş paylaşılan SQLite tablolarını
  kullanıyor; Doctor eski JSON dosyalarını içe aktarıyor ve başarılı bir
  taşıma sonrasında kaldırıyor.
- Güncelleme denetimi durumu artık `update-check.json` veya opak genel bir blob yerine
  türü belirlenmiş paylaşılan bir `update_check_state` satırını kullanıyor; Doctor
  eski JSON dosyasını içe aktarıyor ve başarılı bir taşıma sonrasında kaldırıyor.
- Yapılandırma sağlığı durumu artık `logs/config-health.json` veya opak genel bir blob yerine
  türü belirlenmiş paylaşılan `config_health_entries` satırlarını kullanıyor; Doctor
  eski JSON dosyasını içe aktarıyor ve başarılı bir taşıma sonrasında kaldırıyor.
- Plugin konuşma bağlama onayları artık opak paylaşılan SQLite durumu veya
  `plugin-binding-approvals.json` yerine türü belirlenmiş
  `plugin_binding_approvals` satırlarını kullanıyor; eski dosya bir Doctor taşıma girdisidir.
- Genel geçerli konuşma bağlamaları artık
  `bindings/current-conversations.json` dosyasını yeniden yazmak yerine türü belirlenmiş
  `current_conversation_bindings` satırlarını depoluyor; Doctor eski JSON dosyasını içe aktarıyor ve
  başarılı bir taşıma sonrasında kaldırıyor.
- Memory Wiki içe aktarılan kaynak eşitleme defterleri artık `.openclaw-wiki/source-sync.json` dosyasını yeniden yazmak yerine
  kasa/kaynak anahtarı başına bir SQLite Plugin durum satırı depoluyor;
  taşıma sağlayıcısı eski JSON defterini içe aktarıp kaldırıyor.
- Memory Wiki ChatGPT içe aktarma çalıştırması kayıtları artık `.openclaw-wiki/import-runs/*.json` dosyasını yazmak yerine
  kasa/çalıştırma kimliği başına bir SQLite Plugin durum satırı depoluyor.
  Geri alma anlık görüntüleri, içe aktarma çalıştırması anlık görüntüsü
  arşivlemesi blob depolamaya taşınana kadar açık kasa dosyaları olarak kalıyor.
- Memory Wiki derlenmiş özetleri artık `.openclaw-wiki/cache/agent-digest.json` ve
  `.openclaw-wiki/cache/claims.jsonl` dosyalarını yazmak yerine sıkıştırılmış SQLite Plugin blob satırlarını
  depoluyor. Önbellek yeniden oluşturulabildiğinden Doctor
  eski önbellek dosyalarını içe aktarmadan siliyor.
- ClawHub Skills kurulum takibi artık çalışma zamanında `.clawhub/lock.json` ve
  `.clawhub/origin.json` yan dosyalarını yazmak veya okumak yerine
  çalışma alanı/Skill başına bir SQLite Plugin durum satırı depoluyor. Çalışma zamanı kodu dosya biçimli
  kilit dosyası/köken soyutlamaları yerine izlenen kurulum durum nesnelerini kullanıyor. Doctor,
  yapılandırılmış ajan çalışma alanlarından eski yan dosyaları içe aktarıyor ve temiz bir içe aktarma
  sonrasında kaldırıyor.
- Kurulu Plugin dizini artık `plugins/installs.json` yerine türü belirlenmiş paylaşılan SQLite
  `installed_plugin_index` tekil örnek satırını okuyor ve yazıyor;
  eski JSON dosyası yalnızca bir Doctor taşıma girdisidir ve içe aktarmadan sonra kaldırılır.
- Eski `plugins/installs.json` yol yardımcısı artık Doctor eski kodunda
  bulunuyor. Çalışma zamanı Plugin dizini modülleri JSON dosya yolu değil, yalnızca SQLite destekli kalıcılık
  seçeneklerini sunuyor.
- Gateway yeniden başlatma sentinel'i, yeniden başlatma amacı ve gözetmen devretme durumu artık genel
  opak bloblar yerine türü belirlenmiş paylaşılan SQLite satırlarını (`gateway_restart_sentinel`,
  `gateway_restart_intent` ve `gateway_restart_handoff`) kullanıyor. Çalışma zamanı yeniden başlatma kodunda
  dosya biçimli sentinel/amaç/devretme sözleşmesi bulunmuyor.
- Matrix eşitleme önbelleği, depolama meta verileri, ileti dizisi bağlamaları, gelen ileti yinelenme önleme işaretçileri,
  başlangıç doğrulama bekleme süresi durumu, SDK IndexedDB kripto anlık görüntüleri,
  kimlik bilgileri ve kurtarma anahtarları artık paylaşılan SQLite Plugin durum/blob
  tablolarını kullanıyor. Çalışma zamanı yol yapıları artık bir `storage-meta.json` meta veri
  yolu sunmuyor; bu dosya adı yalnızca eski bir taşıma girdisidir. Bunların eski JSON içe aktarma
  planı Matrix Plugin kurulum/Doctor taşıma yüzeyinde bulunuyor. Gelen ileti
  yinelenme önleme işaretçileri çekirdeğin talep edilebilir yinelenme önleme mekanizmasını (paylaşılan durum veritabanındaki
  `matrix.inbound-dedupe.*` ad alanları) kullanıyor; Matrix Doctor durum taşıması
  kullanımdan kaldırılmış kök başına `inbound-dedupe` satırlarını ve `inbound-dedupe.json` öğesini bir kez içe aktarıyor,
  ardından çalışma zamanı yalnızca talep edilebilir yinelenme önleme deposunu okuyor.
- Matrix başlangıcı artık eski Matrix dosya durumunu taramıyor, raporlamıyor veya tamamlamıyor.
  Matrix dosya algılama, eski kripto anlık görüntüsü oluşturma, oda anahtarı
  geri yükleme taşıma durumu, içe aktarma ve kaynak kaldırma işlemlerinin tümü Doctor'a aittir.
- Matrix çalışma zamanı taşıma barrel'ları kaldırıldı. Eski durum/kripto algılama
  ve değiştirme yardımcıları, çalışma zamanı API yüzeyinin parçası olmak yerine
  doğrudan Matrix Doctor tarafından içe aktarılıyor.
- Matrix taşıma anlık görüntüsü yeniden kullanım işaretçileri artık `matrix/migration-snapshot.json` yerine SQLite Plugin durumunda
  bulunuyor; Doctor, bir yan durum dosyası yazmadan aynı
  doğrulanmış taşıma öncesi arşivi yeniden kullanabiliyor.
- Nostr veri yolu imleçleri ve profil yayımlama durumu artık paylaşılan SQLite Plugin
  durumunu kullanıyor. Bunların eski JSON içe aktarma planı Nostr Plugin kurulum/Doctor
  taşıma yüzeyinde bulunuyor.
- Active Memory oturum açma/kapama ayarları artık `session-toggles.json` yerine paylaşılan SQLite Plugin durumunu
  kullanıyor; belleği yeniden açmak bir JSON nesnesini yeniden yazmak yerine satırı siliyor.
- Skill Workshop önerileri ve inceleme sayaçları artık çalışma alanı başına `skill-workshop/<workspace>.json` depoları yerine
  paylaşılan SQLite Plugin durumunu kullanıyor. Her
  öneri `skill-workshop/proposals` altında ayrı bir satır, inceleme
  sayacı ise `skill-workshop/reviews` altında ayrı bir satırdır.
- Skill Workshop inceleyici alt ajan çalıştırmaları artık `skill-workshop/<sessionId>.json` yan oturum
  yolları oluşturmak yerine çalışma zamanı oturum transkripti çözümleyicisini kullanıyor.
- ACPX işlem kiraları artık tüm dosyayı kapsayan bir `process-leases.json` kayıt defteri yerine
  `acpx/process-leases` altında paylaşılan SQLite Plugin durumunu kullanıyor.
  Her kira kendi satırında depolanarak çalışma zamanında JSON yeniden yazma yolu olmadan
  başlangıçtaki eski işlem temizliğini koruyor.
- ACPX sarmalayıcı betikleri ve yalıtılmış Codex ana dizini
  OpenClaw geçici kökünde oluşturuluyor. Gerektiğinde yeniden oluşturulurlar ve yedekleme veya
  taşıma girdileri değildirler.
- Alt ajan çalıştırma kayıt defteri kalıcılığı türü belirlenmiş paylaşılan `subagent_runs` satırlarını kullanıyor. Eski
  `subagents/runs.json` yolu artık yalnızca bir Doctor temizleme girdisidir. Doctor
  bunu durum bakım kilidi altında talep eder, atma kararını SQLite'a kaydeder
  ve geçici çalıştırma durumunu içe aktarmadan kaldırır. Çalışma zamanında hiçbir JSON
  okuyucu, yazıcı, önbellek veya geri dönüş kalmamıştır; yalnızca dosyada bulunan
  devam eden çalıştırmaların sürümler arası kurtarılması, bu kullanımdan kaldırma sınırında kasıtlı olarak desteklenmez.
  Çalışma zamanı testleri artık kayıt defteri davranışını kanıtlamak için geçersiz veya boş `runs.json` fikstürleri
  oluşturmuyor; doğrudan SQLite satırlarını ekleyip okuyor.
- Yedekleme, arşivlemeden önce durum dizinini hazırlar, veritabanı dışındaki dosyaları kopyalar,
  veritabanlarını çevrimiçi yedekleme ve çevrimdışı `VACUUM` ile anlık görüntüler, canlı WAL/SHM yan dosyalarını dışarıda bırakır,
  anlık görüntü meta verilerini arşiv manifestine kaydeder ve
  tamamlanan yedekleme çalıştırmalarını arşiv manifestiyle birlikte SQLite'a kaydeder. `openclaw backup
create` yazılan arşivi varsayılan olarak doğrular; `--no-verify`
  açık hızlı yoldur.
- `openclaw backup restore` çıkarmadan önce arşivi doğrular, doğrulayıcının
  normalleştirilmiş manifestini yeniden kullanır ve doğrulanmış manifest varlıklarını kayıtlı
  kaynak yollarına geri yükler. Yazma işlemleri için `--yes` gerektirir ve bir geri yükleme planı için `--dry-run`
  desteği sunar.
- Eski yedekleme geçici yol filtresi silindi. SQLite
  anlık görüntüleri arşiv oluşturulmadan önce hazırlandığından yedekleme artık eski oturum veya Cron JSON/JSONL dosyaları için
  canlı tar atlama listesine ihtiyaç duymuyor.
- Temel kurulum ve ilk kullanım çalışma alanı hazırlığı artık
  `agents/<agentId>/sessions/` dizinlerini oluşturmuyor. Yalnızca yapılandırmayı/çalışma alanını oluşturuyor;
  SQLite oturum satırları ve transkript satırları, aracı başına veritabanında
  gerektiğinde oluşturuluyor.
- Güvenlik izinlerini onarma işlemi artık `sessions.json` ve transkript
  JSONL dosyaları yerine genel ve aracı başına SQLite veritabanları ile
  WAL/SHM yan dosyalarını hedefliyor.
- Korumalı alan kayıt defteri çalışma zamanı adları artık etkin depoda eski
  JSON kayıt defteri terminolojisini taşımak yerine SQLite kayıt defteri türlerini doğrudan açıklıyor.
- `openclaw reset --scope config+creds+sessions`, yalnızca eski
  `sessions/` dizinlerini değil, aracı başına
  `openclaw-agent.sqlite` veritabanlarını ve WAL/SHM yan dosyalarını kaldırıyor.
- Gateway toplu oturum yardımcıları artık girdi odaklı adlar kullanıyor:
  `loadCombinedSessionEntriesForGateway`, `{ databasePath, entries }` döndürüyor.
  Eski birleşik depo adlandırması çalışma zamanı çağıranlarından kaldırıldı.
- Docker MCP kanalının başlangıç verilerini oluşturma işlemi artık
  `sessions.json` ve bir JSONL transkripti oluşturmak yerine ana oturum satırını ve transkript
  olaylarını aracı başına SQLite veritabanına yazıyor.
- Paketle gelen oturum belleği kancası artık önceki oturum bağlamını
  SQLite'tan `{agentId, sessionId}` ile çözümlüyor. Artık transkript yollarını veya
  `workspace/sessions` dizinlerini taramıyor, depolamıyor ya da oluşturmuyor.
- Paketle gelen komut günlüğü kancası artık
  `logs/commands.log` dosyasına ekleme yapmak yerine komut denetim satırlarını paylaşılan
  SQLite `command_log_entries` tablosuna yazıyor.
- Kanal eşleştirme izin listeleri artık çalışma zamanında yalnızca SQLite destekli
  okuma/yazma yardımcılarını kullanıma sunuyor. Kullanımdan kaldırılmış Plugin SDK yol çözümleyicisi
  geçiş uyumluluğu için korunuyor; dosya okuyucuları yalnızca doctor durum geçişi kodunda bulunuyor.
- `migration_runs`, eski durum geçişi yürütmelerini durum,
  zaman damgaları ve JSON raporlarıyla kaydediyor.
- `migration_sources`, içe aktarılan her eski dosya kaynağını karma, boyut,
  kayıt sayısı, hedef tablo, çalıştırma kimliği, durum ve kaynak kaldırma durumuyla kaydediyor.
- `backup_runs`, yedek arşiv yollarını, durumu ve JSON bildirimlerini kaydediyor.
- Genel şema, kullanılmayan bir `agents` kayıt defteri tablosunu tutmuyor. Çalışma zamanı
  gerçek bir aracı kaydı sahibine sahip olana kadar aracı veritabanı keşfi, kurallı
  `agent_databases` kayıt defteridir.
- Oluşturulan model kataloğu yapılandırması, aracı dizinine göre anahtarlanan türü belirlenmiş genel SQLite
  `agent_model_catalogs` satırlarında saklanıyor. Çalışma zamanı çağıranları
  `ensureOpenClawModelCatalog` kullanıyor; çalışma zamanı kodunda `models.json` uyumluluk API'si
  bulunmuyor. Uygulama SQLite'a yazıyor ve gömülü PI kayıt defteri, bir
  `models.json` dosyası oluşturulmadan bu depolanmış yükten dolduruluyor.
- İsteğe bağlı `memory.qmd.sessions` dışa aktarımı, kurallı transkript satırlarını
  aracı başına veritabanından okuyor ve açık bir QMD girdi yapıtı olarak QMD ana dizini altında
  arındırılmış Markdown oluşturuyor. Bu nedenle QMD oturum koleksiyonları ve yapıt
  kimliği eşlemeleri, yapılandırılmış harici araç köprüsünün bir parçası olmaya devam ediyor;
  bunlar ikinci bir kurallı transkript deposu değildir.
- QMD'nin kendi `index.sqlite`, YAML koleksiyon yapılandırması ve model indirmeleri
  `~/.openclaw/agents/<agentId>/qmd` altında harici araç yapıtları olarak kalıyor;
  `plugin_blob_entries` içine yansıtılmıyor. OpenClaw'a ait QMD koordinasyonu
  önce veritabanı yaklaşımını kullanır: paylaşılan `state_leases` yerleştirmeleri genel olarak serileştirir ve aracı başına
  `state_leases` koleksiyon/güncelleme/yerleştirme yazıcılarını serileştirir. Çalışma zamanı hiçbir
  QMD kilit yan dosyası oluşturmaz.
- İsteğe bağlı `memory-lancedb` Plugin'i artık örtük olarak OpenClaw tarafından yönetilen bir depo şeklinde
  `~/.openclaw/memory/lancedb` oluşturmuyor. Bu, harici bir LanceDB arka ucudur ve operatör açık bir
  `dbPath` yapılandırana kadar devre dışı kalır.
- `check:database-first-legacy-stores`, eski depo adlarını
  yazma tarzı dosya sistemi API'leriyle eşleştiren yeni çalışma zamanı kaynaklarında başarısız olur. Ayrıca kullanımdan kaldırılmış transkript köprüsü işaretçileri
  `transcriptLocator` veya `sqlite-transcript://...` öğelerini yeniden ekleyen çalışma zamanı
  kaynaklarında da başarısız olur. Geçiş, doctor, içe aktarma
  ve açık oturum dışı dışa aktarma koduna izin verilmeye devam edilir. `sessionFile`, `storePath` gibi daha geniş eski sözleşme
  adları ve eski `SessionManager` dosya dönemi
  cephelerinin hâlâ mevcut sahipleri vardır ve zorunlu bir ön kontrol hâline gelmeden önce
  ayrı geçiş koruması çalışması gerektirir. Koruma artık çalışma zamanı
  `cache/*.json` depolarını, genel
  `thread-bindings.json` yan dosyalarını, cron durum/çalıştırma günlüğü JSON'unu, yapılandırma sağlığı JSON'unu,
  yeniden başlatma ve kilit yan dosyalarını, Voice Wake ayarlarını, Plugin bağlama onaylarını,
  yüklü Plugin dizini JSON'unu, File Transfer denetim JSONL'sini, Memory Wiki etkinlik
  günlüklerini, paketle gelen eski `command-logger` metin günlüğünü ve pi-mono ham akış JSONL
  tanılama ayarlarını da kapsıyor. Ayrıca eski kök düzeyindeki doctor eski modül adlarını yasaklayarak
  uyumluluk kodunun `src/commands/doctor/` altında kalmasını sağlıyor. Android hata ayıklama işleyicileri
  de `camera_debug.log` veya
  `debug_logs.txt` önbellek dosyalarını hazırlamak yerine logcat/bellek içi çıktı kullanıyor.

## Hedef Şema Yapısı

Şemaları açık tutun. Ana makinenin sahip olduğu çalışma zamanı durumu, türü belirlenmiş tablolar kullanır. Plugin'in sahip olduğu
opak durum, `plugin_state_entries` / `plugin_blob_entries` kullanır; genel bir
ana makine `kv` tablosu yoktur.

Genel veritabanı:

```text
state_leases(scope, lease_key, owner, expires_at, heartbeat_at, payload_json, created_at, updated_at)
exec_approvals_config(config_key, raw_json, socket_path, has_socket_token, default_security, default_ask, default_ask_fallback, auto_allow_skills, agent_count, allowlist_count, updated_at_ms)
schema_meta(meta_key, role, schema_version, agent_id, app_version, created_at, updated_at)
agent_databases(agent_id, path, schema_version, last_seen_at, size_bytes)
task_runs(...)
task_delivery_state(...)
flow_runs(...)
subagent_runs(run_id, child_session_key, requester_session_key, controller_session_key, created_at, ended_at, cleanup_handled, payload_json)
current_conversation_bindings(binding_key, binding_id, target_agent_id, target_session_id, target_session_key, channel, account_id, conversation_kind, parent_conversation_id, conversation_id, target_kind, status, bound_at, expires_at, metadata_json, updated_at)
plugin_binding_approvals(plugin_root, channel, account_id, plugin_id, plugin_name, approved_at)
tui_last_sessions(scope_key, session_key, updated_at)
plugin_state_entries(plugin_id, namespace, entry_key, value_json, created_at, expires_at)
plugin_blob_entries(plugin_id, namespace, entry_key, metadata_json, blob, created_at, expires_at)
media_blobs(subdir, id, content_type, size_bytes, blob, created_at, updated_at)
skill_uploads(upload_id, kind, slug, force, size_bytes, sha256, actual_sha256, received_bytes, archive_blob, created_at, expires_at, committed, committed_at, idempotency_key_hash)
skill_upload_chunks(upload_id, byte_offset, size_bytes, chunk_blob)
web_push_subscriptions(endpoint_hash, subscription_id, endpoint, p256dh, auth, created_at_ms, updated_at_ms)
web_push_vapid_keys(key_id, public_key, private_key, subject, updated_at_ms)
apns_registrations(node_id, transport, token, relay_handle, send_grant, installation_id, relay_origin, topic, environment, distribution, token_debug_suffix, updated_at_ms)
apns_registration_tombstones(node_id, deleted_at_ms)
node_host_config(config_key, version, node_id, token, display_name, gateway_host, gateway_port, gateway_tls, gateway_tls_fingerprint, gateway_context_path, updated_at_ms)
device_identities(identity_key, device_id, public_key_pem, private_key_pem, created_at_ms, updated_at_ms)
device_auth_tokens(device_id, role, token, scopes_json, updated_at_ms)
macos_port_guardian_records(pid, port, command, mode, timestamp)
workspace_setup_state(workspace_key, workspace_path, version, bootstrap_seeded_at, setup_completed_at, updated_at)
workspace_path_aliases(alias_key, alias_path, workspace_key, workspace_path, updated_at_ms)
workspace_attestations(workspace_key, attested_at_ms, updated_at_ms)
workspace_generated_bootstrap_hashes(workspace_key, filename, sha256)
native_hook_relay_bridges(relay_id, pid, hostname, port, token, expires_at_ms, updated_at_ms)
model_capability_cache(provider_id, model_id, name, input_text, input_image, reasoning, supports_tools, context_window, max_tokens, cost_input, cost_output, cost_cache_read, cost_cache_write, updated_at_ms)
agent_model_catalogs(catalog_key, agent_dir, raw_json, updated_at)
managed_outgoing_image_records(attachment_id, session_key, agent_id, message_id, created_at, updated_at, retention_class, alt, original_media_id, original_media_subdir, original_content_type, original_width, original_height, original_size_bytes, original_filename, record_json, cleanup_pending)
gateway_restart_sentinel(sentinel_key, version, kind, status, ts, session_key, thread_id, delivery_channel, delivery_to, delivery_account_id, message, continuation_json, doctor_hint, stats_json, payload_json, updated_at_ms)
channel_pairing_requests(channel_key, account_id, request_id, code, created_at, last_seen_at, meta_json)
channel_pairing_allow_entries(channel_key, account_id, entry, sort_order, updated_at)
voicewake_triggers(config_key, position, trigger, updated_at_ms)
voicewake_routing_config(config_key, version, default_target_mode, default_target_agent_id, default_target_session_key, updated_at_ms)
voicewake_routing_routes(config_key, position, trigger, target_mode, target_agent_id, target_session_key, updated_at_ms)
update_check_state(state_key, last_checked_at, last_notified_version, last_notified_tag, last_available_version, last_available_tag, auto_install_id, auto_first_seen_version, auto_first_seen_tag, auto_first_seen_at, auto_last_attempt_version, auto_last_attempt_at, auto_last_success_version, auto_last_success_at, updated_at_ms)
config_health_entries(config_path, last_known_good_json, last_promoted_good_json, last_observed_suspicious_signature, updated_at_ms)
sandbox_registry_entries(registry_kind, container_name, session_key, backend_id, runtime_label, image, created_at_ms, last_used_at_ms, config_label_kind, config_hash, cdp_port, no_vnc_port, entry_json, updated_at)
cron_jobs(store_key, job_id, name, description, enabled, delete_after_run, created_at_ms, agent_id, session_key, schedule_kind, schedule_expr, schedule_tz, every_ms, anchor_ms, at, stagger_ms, session_target, wake_mode, payload_kind, payload_message, payload_model, payload_fallbacks_json, payload_thinking, payload_timeout_seconds, payload_allow_unsafe_external_content, payload_external_content_source_json, payload_light_context, payload_tools_allow_json, delivery_mode, delivery_channel, delivery_to, delivery_thread_id, delivery_account_id, delivery_best_effort, failure_delivery_mode, failure_delivery_channel, failure_delivery_to, failure_delivery_account_id, failure_alert_disabled, failure_alert_after, failure_alert_channel, failure_alert_to, failure_alert_cooldown_ms, failure_alert_include_skipped, failure_alert_mode, failure_alert_account_id, next_run_at_ms, running_at_ms, last_run_at_ms, last_run_status, last_error, last_duration_ms, consecutive_errors, consecutive_skipped, schedule_error_count, last_delivery_status, last_delivery_error, last_delivered, last_failure_alert_at_ms, job_json, state_json, runtime_updated_at_ms, schedule_identity, sort_order, updated_at)
delivery_queue_entries(queue_name, id, status, entry_kind, session_key, channel, target, account_id, retry_count, last_attempt_at, last_error, recovery_state, platform_send_started_at, entry_json, enqueued_at, updated_at, failed_at)
commitments(id, agent_id, session_key, channel, account_id, recipient_id, thread_id, sender_id, kind, sensitivity, source, status, reason, suggested_text, dedupe_key, confidence, due_earliest_ms, due_latest_ms, due_timezone, source_message_id, source_run_id, created_at_ms, updated_at_ms, attempts, last_attempt_at_ms, sent_at_ms, dismissed_at_ms, snoozed_until_ms, expired_at_ms, record_json)
migration_runs(id, started_at, finished_at, status, report_json)
migration_sources(source_key, migration_kind, source_path, target_table, source_sha256, source_size_bytes, source_record_count, last_run_id, status, imported_at, removed_source, report_json)
backup_runs(id, created_at, archive_path, status, manifest_json)
```

Aracı veritabanı:

```text
schema_meta(meta_key, role, schema_version, agent_id, app_version, created_at, updated_at)
sessions(session_id, session_key, session_scope, created_at, updated_at, started_at, ended_at, status, chat_type, channel, account_id, primary_conversation_id, model_provider, model, agent_harness_id, parent_session_key, spawned_by, display_name)
conversations(conversation_id, channel, account_id, kind, peer_id, parent_conversation_id, thread_id, native_channel_id, native_direct_user_id, label, metadata_json, created_at, updated_at)
session_conversations(session_id, conversation_id, role, first_seen_at, last_seen_at)
session_routes(session_key, session_id, updated_at)
session_entries(session_id, session_key, entry_json, updated_at)
transcript_events(session_id, seq, event_json, created_at)
transcript_event_identities(session_id, event_id, seq, event_type, has_parent, parent_id, message_idempotency_key, created_at)
transcript_snapshots(session_id, snapshot_id, reason, event_count, created_at, metadata_json)
vfs_entries(namespace, path, kind, content_blob, metadata_json, updated_at)
tool_artifacts(run_id, artifact_id, kind, metadata_json, blob, created_at)
run_artifacts(run_id, path, kind, metadata_json, blob, created_at)
trajectory_runtime_events(session_id, run_id, seq, event_json, created_at)
memory_index_meta(key, value)
memory_index_sources(id, path, source, hash, mtime, size)
memory_index_chunks(id, path, source, start_line, end_line, hash, model, text, embedding, updated_at)
memory_embedding_cache(provider, model, provider_key, hash, embedding, dims, updated_at)
memory_index_state(id, revision)
cache_entries(scope, key, value_json, blob, expires_at, updated_at)
```

`memory_index_sources.id` kararlı tamsayı birincil anahtardır; `(path, source)` benzersiz kalır.

Gelecekteki arama işlevleri, standart olay tablolarını değiştirmeden FTS tabloları ekleyebilir:

```text
transcript_events_fts(session_id, seq, text)
vfs_entries_fts(namespace, path, text)
```

Büyük değerler, JSON dizesi kodlaması yerine `blob` sütunlarını kullanmalıdır. Düz
SQLite araçlarıyla incelenebilir kalması gereken küçük yapılandırılmış veriler için
`value_json` kullanmaya devam edin.

`agent_databases` bu dalın standart kayıt defteridir. Gerçek bir aracı kaydı sahibi oluşana kadar
`agents` tablosu eklemeyin; aracı yapılandırması
`openclaw.json` içinde kalır.

## Doctor Geçiş Yapısı

Doctor, raporlanabilen ve yeniden çalıştırılması güvenli olan tek bir açık geçiş adımını
çağırmalıdır:

```bash
openclaw doctor --fix
```

`openclaw doctor --fix`, olağan yapılandırma ön kontrollerinden sonra durum geçişi
uygulamasını çağırır ve içe aktarmadan önce doğrulanmış bir yedek oluşturur. Çalışma zamanı
başlangıcı ve `openclaw migrate`, eski OpenClaw durum dosyalarını içe aktarmamalıdır.

Geçiş özellikleri:

- Tek bir geçiş çalışması tüm eski dosya kaynaklarını keşfeder ve herhangi bir şeyi
  değiştirmeden önce bir plan oluşturur.
- Doctor, eski dosyaları içe aktarmadan önce doğrulanmış bir geçiş öncesi yedek
  arşivi oluşturur.
- İçe aktarmalar eş etkili olup kaynak yolu, mtime, boyut, karma ve hedef
  tabloyla anahtarlanır.
- Başarıyla işlenen kaynak dosyalar, hedef veritabanı kaydedildikten sonra
  kaldırılır veya arşivlenir.
- Başarısız içe aktarmalar kaynağı değiştirmeden bırakır ve
  `migration_runs` içinde bir uyarı kaydeder.
- Geçiş oluşturulduktan sonra çalışma zamanı kodu yalnızca SQLite'ı okur.
- Sürüm düşürme/çalışma zamanı dosyalarına dışa aktarma yolu gerekli değildir.

## Geçiş Envanteri

Bunları genel veritabanına taşıyın:

- Görev kayıt defteri çalışma zamanı yazmaları artık paylaşılan veritabanını kullanıyor; yayımlanmamış
  `tasks/runs.sqlite` yan dosya içe aktarıcısı silindi. Anlık görüntü kayıtları görev
  kimliğine göre ekler veya günceller ve yalnızca eksik görev/teslimat satırlarını siler.
- Task Flow çalışma zamanı yazmaları artık paylaşılan veritabanını kullanıyor; yayımlanmamış
  `tasks/flows/registry.sqlite` yan dosya içe aktarıcısı silindi. Anlık görüntü kayıtları
  akış kimliğine göre ekler veya günceller ve yalnızca eksik akış satırlarını siler.
- Plugin durumu çalışma zamanı yazmaları artık paylaşılan veritabanını kullanıyor; yayımlanmamış
  `plugin-state/state.sqlite` yan dosya içe aktarıcısı silindi.
- Yerleşik bellek araması artık varsayılan olarak `memory/<agentId>.sqlite` kullanmıyor; dizin
  tabloları bunların sahibi olan ajan veritabanında bulunuyor ve açık
  `memorySearch.store.path` yan dosya kabul seçeneği, doctor yapılandırma
  geçişine devredildi.
- Yerleşik bellek yeniden dizinleme, ajan veritabanında yalnızca belleğe ait tabloları sıfırlar.
  Aynı veritabanı oturumların, transkriptlerin, VFS satırlarının, eserlerin ve
  çalışma zamanı önbelleklerinin de sahibi olduğundan SQLite dosyasının tamamını değiştirmemelidir.
- Tek parça ve parçalanmış JSON'daki korumalı alan kapsayıcı/tarayıcı kayıt defterleri. Çalışma zamanı
  yazmaları artık paylaşılan veritabanını kullanıyor; eski JSON içe aktarımı korunuyor.
- Cron işi tanımları, zamanlama durumu ve çalıştırma geçmişi artık paylaşılan SQLite'ı kullanıyor;
  doctor eski `jobs.json`, `jobs-state.json` ve
  `cron/runs/*.jsonl` dosyalarını içe aktarır/kaldırır
- Cihaz kimliği/kimlik doğrulaması, anında iletim, güncelleme denetimi, taahhütler, OpenRouter model
  önbelleği, yüklü Plugin dizini ve uygulama sunucusu bağlamaları
- Cihaz/Node eşleştirme ve önyükleme kayıtları artık türü belirlenmiş SQLite tablolarını kullanıyor
- Cihaz eşleştirme bildirimi aboneleri ve teslim edilmiş istek işaretçileri artık
  `device-pair-notify.json` yerine paylaşılan SQLite Plugin durumu tablosunu kullanıyor.
- Sesli arama kayıtları artık `calls.jsonl` yerine
  `voice-call` / `calls` ad alanı altında paylaşılan SQLite Plugin durumu tablosunu kullanıyor; Plugin CLI,
  SQLite destekli arama geçmişini takip eder ve özetler.
- QQBot Gateway oturumları, bilinen kullanıcı kayıtları ve başvuru dizini alıntı önbelleği artık
  `session-*.json`, `known-users.json` ve
  `ref-index.jsonl` yerine `qqbot` ad alanları altında (`gateway-sessions`,
  `known-users`, `ref-index`) SQLite Plugin durumunu kullanıyor. Bu eski dosyalar önbellektir ve taşınmaz.
- Discord model seçici tercihleri, komut dağıtım karmaları ve ileti dizisi bağlamaları
  artık `model-picker-preferences.json`, `command-deploy-cache.json` ve
  `thread-bindings.json` yerine `discord` ad alanları
  (`model-picker-preferences`, `command-deploy-hashes`, `thread-bindings`)
  altında SQLite Plugin durumunu kullanıyor; Discord doctor/kurulum geçişi eski dosyaları
  içe aktarır ve kaldırır.
- BlueBubbles telafi imleçleri ve gelen tekilleştirme işaretçileri artık
  `bluebubbles/catchup/*.json` ve
  `bluebubbles/inbound-dedupe/*.json` yerine `bluebubbles` ad alanları (`catchup-cursors`, `inbound-dedupe`)
  altında SQLite Plugin durumunu kullanıyor; BlueBubbles doctor/kurulum geçişi
  eski dosyaları içe aktarır ve kaldırır.
- Telegram güncelleme uzaklıkları, çıkartma önbelleği girdileri, yanıt zinciri ileti önbelleği
  girdileri, gönderilmiş ileti önbelleği girdileri, konu adı önbelleği girdileri ve ileti dizisi
  bağlamaları artık `update-offset-*.json`,
  `sticker-cache.json`, `*.telegram-messages.json`,
  `*.telegram-sent-messages.json`, `*.telegram-topic-names.json` ve
  `thread-bindings-*.json` yerine `telegram` ad alanları
  (`update-offsets`, `sticker-cache`, `message-cache`, `sent-messages`,
  `topic-names`, `thread-bindings`) altında SQLite Plugin durumunu kullanıyor; Telegram doctor/kurulum geçişi eski dosyaları içe
  aktarır ve kaldırır.
- iMessage telafi imleçleri, kısa yanıt kimliği eşlemeleri ve gönderilmiş yankı tekilleştirme satırları
  artık `imessage/catchup/*.json`,
  `imessage/reply-cache.jsonl` ve `imessage/sent-echoes.jsonl` yerine `imessage` ad alanları (`catchup-cursors`,
  `reply-cache`, `sent-echoes`) altında SQLite Plugin durumunu kullanıyor; iMessage
  doctor/kurulum geçişi eski dosyaları içe aktarır ve kaldırır.
- Microsoft Teams konuşmaları, anketleri, SSO belirteçleri ve geri bildirim öğrenimleri artık
  `msteams-conversations.json`,
  `msteams-polls.json`, `msteams-sso-tokens.json` ve `*.learnings.json` yerine SQLite Plugin durumu ad alanlarını (`conversations`, `polls`, `sso-tokens`,
  `feedback-learnings`) kullanıyor; Microsoft Teams doctor/kurulum geçişi eski dosyaları
  içe aktarır ve arşivler. Bekleyen yüklemeler kısa ömürlü bir SQLite önbelleğidir ve eski JSON önbellek
  dosyaları taşınmaz.
- Matrix eşitleme önbelleği, depolama meta verileri, ileti dizisi bağlamaları, gelen tekilleştirme işaretçileri,
  başlangıç doğrulama bekleme süresi durumu, kimlik bilgileri, kurtarma anahtarları ve SDK
  IndexedDB kripto anlık görüntüleri artık `bot-storage.json`, `storage-meta.json`, `thread-bindings.json`,
  `inbound-dedupe.json`, `startup-verification.json`, `credentials.json`,
  `recovery-key.json` ve `crypto-idb-snapshot.json` yerine
  `matrix` altında (`sync-store`, `storage-meta`, `thread-bindings`,
  çekirdek talep edilebilir tekilleştirme aracılığıyla `matrix.inbound-dedupe.*`,
  `startup-verification`, `credentials`, `recovery-key`, `idb-snapshots`)
  SQLite Plugin durumu/blob ad alanlarını kullanıyor; Matrix doctor/kurulum
  geçişi bu eski dosyaları (ve kullanımdan kaldırılmış kök başına
  `inbound-dedupe` SQLite satırlarını) hesap kapsamlı Matrix depolama köklerinden içe aktarır ve kaldırır.
- Nostr veri yolu imleçleri ve profil yayımlama durumu artık
  `bus-state-*.json` ve `profile-state-*.json` yerine
  `nostr` ad alanları (`bus-state`, `profile-state`) altında SQLite Plugin durumunu kullanıyor; Nostr doctor/kurulum
  geçişi eski dosyaları içe aktarır ve kaldırır.
- Active Memory oturum açma/kapatma ayarları artık `session-toggles.json` yerine
  `active-memory/session-toggles` altında SQLite Plugin durumunu kullanıyor.
- Skill Workshop teklif kuyrukları ve inceleme sayaçları artık çalışma alanı başına
  `skill-workshop/<workspace>.json` dosyaları yerine `skill-workshop/proposals` ve `skill-workshop/reviews`
  altında SQLite Plugin durumunu kullanıyor.
- Giden teslimat ve oturum teslimat kuyrukları artık kalıcı
  `delivery-queue/*.json`, `delivery-queue/failed/*.json` ve
  `session-delivery-queue/*.json` dosyaları yerine ayrı kuyruk adları
  (`outbound-delivery`, `session-delivery`) altında genel SQLite
  `delivery_queue_entries` tablosunu paylaşıyor. Doctor eski durum adımı,
  bekleyen ve başarısız satırları içe aktarır, eskimiş teslim edildi işaretçilerini kaldırır ve içe aktarma sonrasında eski
  JSON dosyalarını siler. Sık kullanılan yönlendirme ve yeniden deneme alanları türü belirlenmiş sütunlardır;
  JSON yükü yalnızca yeniden oynatma/hata ayıklama için korunur.
- ACPX işlem kiraları artık `process-leases.json` yerine `acpx/process-leases`
  altında SQLite Plugin durumunu kullanıyor.
- Yedekleme ve geçiş çalıştırması meta verileri

Bunları ajan veritabanlarına taşıyın:

- Ajan oturumu kökleri ve uyumluluk biçimli oturum girdisi yükleri. Çalışma zamanı yazmaları için
  tamamlandı: sık kullanılan oturum meta verileri `sessions` içinde sorgulanabilirken,
  eski biçimli tam `SessionEntry` yükü `session_entries` içinde kalır.
- Ajan transkript olayları. Çalışma zamanı yazmaları için tamamlandı.
- Compaction denetim noktaları ve transkript anlık görüntüleri. Çalışma zamanı yazmaları için tamamlandı:
  denetim noktası transkript kopyaları SQLite transkript satırlarıdır ve denetim noktası
  meta verileri `transcript_snapshots` içinde kaydedilir. Gateway denetim noktası yardımcıları
  artık bu değerleri kaynak dosyalar yerine transkript anlık görüntüleri olarak adlandırır.
- Ajan VFS geçici/çalışma alanı ad alanları. Çalışma zamanı VFS yazmaları için tamamlandı.
- Alt ajan ek yükleri. Çalışma zamanı yazmaları için tamamlandı: bunlar SQLite VFS
  başlangıç girdileridir ve hiçbir zaman kalıcı çalışma alanı dosyaları değildir.
- Araç eserleri. Çalışma zamanı yazmaları için tamamlandı.
- Çalıştırma eserleri. Ajan başına
  `run_artifacts` tablosu üzerinden çalışan çalışma zamanı yazmaları için tamamlandı.
- Ajan yerel çalışma zamanı önbellekleri. Ajan başına `cache_entries`
  tablosu üzerinden çalışan çalışma zamanı kapsamlı önbellek yazmaları için tamamlandı. Gateway genelindeki model önbellekleri,
  ajana özgü hâle gelmedikçe genel veritabanında kalır.
- ACP üst akış günlükleri. Çalışma zamanı yazmaları için tamamlandı.
- ACP yeniden oynatma kayıt defteri oturumları. `acp_replay_sessions` ve
  `acp_replay_events` üzerinden çalışma zamanı yazmaları için tamamlandı; eski `acp/event-ledger.json`
  yalnızca doctor girdisi olarak kalır.
- ACP oturum meta verileri. `acp_sessions` üzerinden çalışma zamanı yazmaları için tamamlandı; `sessions.json` içindeki eski
  `entry.acp` blokları yalnızca doctor geçişi girdisidir.
- Açık dışa aktarma dosyaları olmadıklarında yörünge yan dosyaları. Çalışma zamanı
  yazmaları için tamamlandı: yörünge yakalama, ajan veritabanına `trajectory_runtime_events`
  satırlarını yazar ve çalıştırma kapsamlı eserleri SQLite'a yansıtır. Eski yan dosyalar yalnızca doctor
  içe aktarma girdileridir; dışa aktarma yeni JSONL destek paketi çıktıları oluşturabilir
  ancak çalışma zamanında eski yörünge/transkript yan dosyalarını okumaz veya taşımaz.
  Çalışma zamanı yörünge yakalama SQLite kapsamını kullanıma sunar; JSONL yol yardımcıları
  dışa aktarma/hata ayıklama desteğiyle sınırlandırılmıştır ve çalışma zamanı modülünden yeniden dışa aktarılmaz.
  Gömülü çalıştırıcı yörünge meta verileri, bir transkript konumlandırıcısını kalıcılaştırmak yerine `{agentId, sessionId, sessionKey}`
  kimliğini kaydeder.

Bunları şimdilik dosya destekli tutun:

- `openclaw.json`
- sağlayıcı veya CLI kimlik bilgisi dosyaları
- Plugin/paket manifestleri
- disk modu seçildiğinde kullanıcı çalışma alanları ve Git depoları
- belirli bir günlük yüzeyi taşınmadıkça operatörün takip etmesi amaçlanan günlükler

## Geçiş Planı

### Aşama 0: Sınırı Sabitleme

Daha fazla satırı taşımadan önce kalıcı durum sınırını açık hâle getirin:

- Genel veritabanına bir `migration_runs` tablosu ekleyin.
  Eski durum geçişi yürütme raporları için tamamlandı.
- Dosyadan veritabanına içe aktarma için doctor'a ait tek bir durum geçiş hizmeti ekleyin.
  Tamamlandı: `openclaw doctor --fix` eski durum geçişi uygulamasını kullanıyor.
- `plan` öğesini salt okunur yapın ve `apply` öğesinin bir yedek oluşturmasını, içe aktarmasını, doğrulamasını ve
  ardından eski dosyaları silmesini veya karantinaya almasını sağlayın.
  Tamamlandı: doctor, doğrulanmış bir geçiş öncesi yedek oluşturur, yedek yolunu
  `migration_runs` içine aktarır ve içe aktarıcı/kaldırma yollarını yeniden kullanır.
- Geçiş kodu ve testler hâlâ bunları hazırlayabilir/okuyabilirken yeni çalışma zamanı kodunun eski durum dosyaları yazamaması için
  statik yasaklar ekleyin.
  Şu anda taşınmış eski depolar için tamamlandı; koruma ayrıca yasaklanmış çalışma zamanı
  transkript konumlandırıcı sözleşmeleri için iç içe testleri tarar.

### Aşama 1: Genel Denetim Düzlemini Tamamlama

Paylaşılan koordinasyon durumunu `state/openclaw.sqlite` içinde tutun:

- Ajanlar ve ajan veritabanı kayıt defteri
- Görev ve Task Flow kayıt defterleri
- Plugin durumu
- Korumalı alan kapsayıcı/tarayıcı kayıt defteri
- Cron/zamanlayıcı çalıştırma geçmişi
- Eşleştirme, cihaz, anında iletim, güncelleme denetimi, TUI, OpenRouter/model önbellekleri ve diğer
  küçük Gateway kapsamlı çalışma zamanı durumu
- Yedekleme ve geçiş meta verileri
- Gateway medya eki baytları. Çalışma zamanı yazmaları için tamamlandı; doğrudan dosya yolları,
  kanal göndericileri ve korumalı alan hazırlama ile uyumluluk için geçici oluşturmalardır.
  Çalışma zamanı izin listeleri eski durum/yapılandırma medya köklerini değil, SQLite oluşturma yollarını kabul eder.
  Doctor eski medya dosyalarını `media_blobs` içine aktarır ve satırların başarıyla yazılmasının ardından
  kaynak dosyaları kaldırır.
- Hata ayıklama proxy'si yakalama oturumları, olayları ve yük blob'ları. Tamamlandı: yakalamalar
  paylaşılan durum veritabanında bulunur ve paylaşılan durum veritabanı önyüklemesi, şeması,
  WAL ve meşgul zaman aşımı ayarları üzerinden açılır. Yük baytları
  `capture_blobs.data` içinde gzip ile sıkıştırılır; yalnızca proxy yakalamaya özgü oluşturulmuş şema/kod üretme hedefi,
  hata ayıklama proxy'si çalışma zamanı yan dosya veritabanı geçersiz kılması veya
  blob dizini yoktur. Doctor/başlangıç geçişi, etkin eski veritabanı/blob ortam
  geçersiz kılmaları dâhil olmak üzere yayımlanmış `debug-proxy/capture.sqlite` satırlarını
  ve başvurulan yük blob'larını içe aktarır, ardından CA sertifikalarını olduğu gibi bırakarak bu kaynakları arşivler.

Bu aşama ayrıca bu alt sistemlerden yinelenen yardımcı depo açıcılarını, izin yardımcılarını, WAL
kurulumunu, dosya sistemi budamasını ve uyumluluk yazıcılarını siler.

### Aşama 2: Temsilci Başına Veritabanlarını Kullanıma Alma

Her temsilci için bir veritabanı oluşturun ve bunu küresel veritabanından kaydedin:

```text
~/.openclaw/state/openclaw.sqlite
~/.openclaw/agents/<agentId>/agent/openclaw-agent.sqlite
```

Küresel `agent_databases` satırı yolu, şema sürümünü, son görülme
zaman damgasını ve temel boyut/bütünlük meta verilerini saklar. Çalışma zamanı kodu,
dosya yollarını doğrudan türetmek yerine temsilci veritabanını kayıt defterinden ister.

Temsilci veritabanı şunların sahibidir:

- `sessions` kanonik oturum kökü olarak; `session_entries`, bu köke
  bağlı uyumluluk biçimli yük tablosu ve
  `session_routes`, benzersiz etkin `session_key` araması olarak
- `conversations` ve `session_conversations`, oturumlara bağlı normalleştirilmiş sağlayıcı
  yönlendirme kimliği olarak
- `transcript_events`
- transkript anlık görüntüleri ve Compaction kontrol noktaları. Çalışma zamanı yazmaları için tamamlandı.
- `vfs_entries`
- `tool_artifacts` ve çalıştırma eserleri
- temsilciye yerel çalışma zamanı/önbellek satırları. Çalışan kapsamlı önbellekler için tamamlandı.
- ACP üst akış olayları
- açık dışa aktarma eserleri olmadıklarında yörünge çalışma zamanı olayları

### Aşama 3: Oturum Deposu API'lerini Değiştirme

Çalışma zamanı için tamamlandı. Dosya biçimli oturum deposu yüzeyi etkin bir
çalışma zamanı sözleşmesi değildir:

- Çalışma zamanı artık `loadSessionStore(storePath)` çağırmaz veya `storePath` değerini
  oturum kimliği olarak ele almaz.
- Çalışma zamanı satır işlemleri `getSessionEntry`, `upsertSessionEntry`,
  `patchSessionEntry`, `deleteSessionEntry` ve `listSessionEntries` şeklindedir.
- Tüm depoyu yeniden yazma yardımcıları, dosya yazıcıları, kuyruk testleri, takma ad budama ve
  eski anahtar silme parametreleri çalışma zamanından kaldırılmıştır.
- Kullanımdan kaldırılmış kök paket uyumluluk dışa aktarımları, 2026-10-12 tarihine kadar yalnızca doctor'a özgü
  `sessions.json` içe aktarıcısına temsil edilir; Plugin SDK uyumluluk okumaları
  kanonik SQLite satırlarını yansıtmaya devam eder.
- `sessions.json` ayrıştırması yalnızca doctor geçiş/içe aktarma kodunda ve
  doctor testlerinde kalır.
- Çalışma zamanı yaşam döngüsü geri dönüş okumaları JSONL ilk
  satırlarını değil, SQLite transkript başlıklarını okur.

Dosya kilidi parametrelerini, dosya bakımı olarak budama/kısaltma
terminolojisini, depo yolu kimliğini veya tek iddiası JSON kalıcılığı olan testleri
yeniden kullanıma alan her şeyi silmeye devam edin.

### Aşama 4: Transkriptleri, ACP Akışlarını, Yörüngeleri ve VFS'yi Taşıma

Her temsilci veri akışını veritabanına özgü hâle getirin:

- Transkript ekleme yazmaları; oturum başlığını güvence altına alan,
  ileti yinelenmezliğini denetleyen, üst kuyruğu seçen, `transcript_events` içine ekleyen
  ve sorgulanabilir kimlik meta verilerini `transcript_event_identities` içine kaydeden tek bir
  SQLite işlemi üzerinden yürür. Doğrudan transkript iletisi eklemeleri ve
  normal kalıcı `TranscriptSessionManager` eklemeleri için tamamlandı; açık dal
  işlemleri açık üst seçimlerini korur ve herhangi bir dosya konumlandırıcısı
  türetmeden SQLite satırları yazmaya devam eder.
- ACP üst akış günlükleri `.acp-stream.jsonl` dosyaları değil, satırlar hâline gelir. Tamamlandı.
- ACP oluşturma kurulumu artık transkript JSONL yollarını kalıcılaştırmaz. Tamamlandı.
- Çalışma zamanı yörünge yakalama, olay satırlarını/eserlerini doğrudan yazar. Açık
  destek/dışa aktarma komutu, dışa aktarma biçimi olarak destek paketi JSONL eserleri
  üretmeye devam edebilir ancak oturum dışa aktarımı oturum JSONL'sini yeniden oluşturmaz. Tamamlandı.
- Disk çalışma alanları, disk modu olarak yapılandırıldığında diskte kalır.
- VFS karalama alanı ve yalnızca VFS kullanan deneysel çalışma alanı modu temsilci veritabanını kullanır.

Geçiş, eski JSONL dosyalarını bir kez içe aktarır, sayıları/karmaları
`migration_runs` içinde kaydeder ve bütünlük denetimlerinden sonra içe aktarılan dosyaları kaldırır.

### Aşama 5: Yedekleme, Geri Yükleme, Vacuum ve Doğrulama

Yedeklemeler tek bir arşiv dosyası olarak kalır:

- Her küresel ve temsilci veritabanı için kontrol noktası oluşturun.
- Her veritabanının anlık görüntüsünü SQLite çevrimiçi yedekleme ve ardından çevrimdışı `VACUUM` ile alın.
- Sıkıştırılmış veritabanı anlık görüntülerini, yapılandırmayı, harici kimlik bilgilerini ve istenen
  çalışma alanı dışa aktarımlarını arşivleyin.
- Ham canlı `*.sqlite-wal` ve `*.sqlite-shm` dosyalarını dışarıda bırakın.
- Her veritabanı anlık görüntüsünü açıp `PRAGMA integrity_check` çalıştırarak doğrulayın.
  `openclaw backup create` bu arşiv doğrulamasını varsayılan olarak gerçekleştirir;
  `--no-verify` yalnızca yazma sonrası arşiv geçişini atlar, anlık görüntü
  oluşturma bütünlük denetimini değil.
- Geri yükleme, anlık görüntüleri hedef yollarına geri kopyalar. Geri yüklenen küresel veritabanları
  `1` sürümünü; geri yüklenen temsilci başına veritabanları `2` sürümünü kullanır ve `1` sürümlü anlık görüntüler
  açıldığında atomik olarak yükseltilir.

### Aşama 6: Çalışan Çalışma Zamanı

Veritabanı ayrımı uygulamaya alınırken çalışan modunu deneysel tutun:

- Çalışanlar temsilci kimliğini, çalıştırma kimliğini, dosya sistemi modunu ve veritabanı kayıt defteri kimliğini alır.
- Her çalışan kendi SQLite bağlantısını açar.
- Üst süreç kanal teslimatı, onaylar, yapılandırma ve iptal yetkisini elinde tutar.
- Etkin çalıştırma başına bir çalışanla başlayın; havuzlamayı yalnızca yaşam döngüsü ve veritabanı
  bağlantısı sahipliği kararlı hâle geldikten sonra ekleyin.

### Aşama 7: Eski Dünyayı Silme

Çalışma zamanı oturum yönetimi için tamamlandı. Eski dünyaya yalnızca açık
doctor girdisi veya destek/dışa aktarma çıktısı olarak izin verilir:

- Çalışma zamanında `sessions.json`, transkript JSONL'si, korumalı alan kayıt defteri JSON'u, görev
  yardımcı depo SQLite'ı veya Plugin durumu yardımcı depo SQLite'ı yazılmaz.
- JSON/oturum dosyası budama, dosya transkripti kısaltma, oturum dosyası kilitleri
  veya kilit biçimli oturum testleri yoktur.
- Amacı eski oturum dosyalarını güncel tutmak olan çalışma zamanı uyumluluk dışa
  aktarımları yoktur.
- Açık destek dışa aktarımları, kullanıcı tarafından istenen arşiv/somutlaştırma
  biçimleri olarak kalır ve dosya adlarını çalışma zamanı kimliğine geri beslememelidir.

## Yedekleme ve Geri Yükleme

Yedeklemeler tek bir arşiv dosyası olmalıdır ancak veritabanı yakalama
SQLite'a özgü olmalıdır:

1. Çevrimiçi yedeklemenin ilerleyebilmesi için yazma işlemlerini sınırlı tutun.
2. Yakalamadan önce her canlı küresel ve temsilci veritabanını doğrulayın.
3. Her veritabanını SQLite çevrimiçi yedekleme ile geçici bir yedekleme
   dizinine yakalayın, ardından canlı bağlantıyı kapatın ve özel kopyaya `VACUUM` uygulayın.
   Sahibi tarafından tanımlanmış SQLite yetenekleri gerektiren Plugin şemaları,
   sahibi güvenli bir anlık görüntü sözleşmesi sağlayana kadar kapalı biçimde başarısız olur.
4. Veritabanı anlık görüntülerini, yapılandırma dosyasını, kimlik bilgileri dizinini, seçili
   çalışma alanlarını ve bir bildirimi arşivleyin.
5. Her SQLite anlık görüntüsünün dosya biçimini doğrulayın, ardından kanonik OpenClaw
   veritabanlarını açıp `PRAGMA integrity_check` ve rol doğrulamasını çalıştırın. Ayrılmış
   Plugin şemaları, sahipleri bir doğrulayıcı sağlamadığı sürece opak kalır.
   `openclaw backup create` bunu varsayılan olarak gerçekleştirir; `--no-verify` yalnızca
   yazma sonrası arşiv geçişini bilerek atlamak içindir.

Birincil yedekleme biçimi olarak ham canlı `*.sqlite`, `*.sqlite-wal` ve `*.sqlite-shm` kopyalarına
güvenmeyin. Arşiv bildirimi veritabanı rolünü, temsilci kimliğini, şema sürümünü,
kaynak yolunu, anlık görüntü yolunu, bayt boyutunu ve bütünlük durumunu kaydetmelidir.

Geri yükleme, küresel veritabanını ve temsilci veritabanı dosyalarını arşiv
anlık görüntülerinden yeniden oluşturmalıdır. Küresel şema `1` sürümünde kalır; temsilci başına `1`
sürümlü anlık görüntüler, `2` sürümüne sınırlı çalışma zamanı yükseltmesi alır. Doctor,
dosyadan veritabanına içe aktarmanın tek sahibi olarak kalır. Geri yükleme komutu önce
arşivi doğrular, ardından her bildirim varlığını doğrulanmış ve ayıklanmış
yükten değiştirir.

## Çalışma Zamanı Yeniden Düzenleme Planı

1. Veritabanı kayıt defteri API'lerini ekleyin.
   - Küresel veritabanı ve temsilci başına veritabanı yollarını çözümleyin.
   - Küresel şemayı `user_version = 1` sürümünde tutun. Temsilci başına veritabanları, yayımlanmış `1`
     sürümlü bellek kaynağı biçiminden tek bir atomik geçişle `2` sürümünü kullanır.
   - Testler, yedekleme ve doctor tarafından kullanılan kapatma/kontrol noktası/bütünlük yardımcılarını ekleyin.

2. Yardımcı depo SQLite depolarını birleştirin.
   - Plugin durum tablolarını küresel veritabanına taşıyın. Çalışma zamanı
     yazmaları için tamamlandı; yayımlanmamış eski yardımcı depo içe aktarıcısı silindi.
   - Görev kayıt defteri tablolarını küresel veritabanına taşıyın. Çalışma zamanı
     yazmaları için tamamlandı; yayımlanmamış eski yardımcı depo içe aktarıcısı silindi.
   - TaskFlow tablolarını küresel veritabanına taşıyın. Çalışma zamanı yazmaları için tamamlandı;
     yayımlanmamış eski yardımcı depo içe aktarıcısı silindi.
   - Yerleşik bellek arama tablolarını her temsilci veritabanına taşıyın. Tamamlandı; açık
     özel `memorySearch.store.path` artık doctor yapılandırma geçişi tarafından kaldırılır.
     Tam yeniden indeksleme yalnızca bellek tabloları üzerinde yerinde çalışır; eski tüm dosyayı
     değiştirme yolu ve yardımcı depo indeks değiştirme yardımcısı silindi.
   - Bu alt sistemlerden yinelenen veritabanı açıcılarını, WAL kurulumunu, izin yardımcılarını ve
     kapatma yollarını silin.

3. Temsilcinin sahip olduğu tabloları temsilci başına veritabanlarına taşıyın.
   - Küresel veritabanı kayıt defteri aracılığıyla gerektiğinde temsilci veritabanı oluşturun. Tamamlandı.
   - Çalışma zamanı oturum girdilerini, transkript olaylarını, VFS satırlarını ve araç
     eserlerini temsilci veritabanlarına taşıyın. Tamamlandı.
   - Dal yerel paylaşımlı veritabanı oturum girdilerini, transkript olaylarını,
     VFS satırlarını veya araç eserlerini taşımayın; bu düzen hiçbir zaman yayımlanmadı. Doctor'da yalnızca eski
     dosyadan veritabanına içe aktarmayı tutun.

4. Oturum deposu API'lerini değiştirin.
   - `storePath` değerini çalışma zamanı kimliği olmaktan çıkarın. Çalışma zamanı için tamamlandı ve
     `check:database-first-legacy-stores` tarafından korunuyor: oturum meta verileri, yönlendirme güncellemeleri,
     komut kalıcılığı, CLI oturum temizliği, Feishu akıl yürütme önizlemeleri,
     transkript durumu kalıcılığı, alt temsilci derinliği, kimlik doğrulama profili oturum
     geçersiz kılmaları, üst çatallama mantığı ve QA-lab incelemesi artık
     veritabanını kanonik temsilci/oturum anahtarlarından çözümler.
     Gateway/TUI/UI/macOS oturum listesi yanıtları artık eski `path` yerine `databasePath`
     sunar; macOS hata ayıklama yüzeyleri, `session.store` yapılandırmasını yazmak yerine temsilci başına veritabanını
     salt okunur durum olarak gösterir.
     `/status`, sohbet odaklı yörünge dışa aktarımı ve CLI bağımlılık vekilleri artık
     eski depo yollarını yaymaz; transkript kullanım geri dönüşü SQLite'ı
     temsilci/oturum kimliğine göre okur. Çalışma zamanı ve köprü testleri artık
     `storePath` sunmaz; bu eski alan adının sahibi doctor/geçiş girdileridir.
     Gateway birleşik oturum yüklemesinde artık şablonlanmamış
     `session.store` değerleri için özel bir çalışma zamanı dalı yoktur; temsilci başına SQLite satırlarını toplar.
     Eski oturum kilidi doctor hattı ve `.jsonl.lock` temizleme yardımcısı
     kaldırıldı; artık oturum eşzamanlılık sınırı SQLite'tır.
     Yoğun çalışma zamanı çağrı noktaları
     `resolveSessionRowEntry` gibi satır odaklı yardımcı adları kullanır; eski `resolveSessionStoreEntry` uyumluluk
     takma adı çalışma zamanı ve Plugin SDK dışa aktarımlarından kaldırılmıştır.

- `{ agentId, sessionKey }` satır işlemlerini kullanın.
  Tamamlandı: `getSessionEntry`, `upsertSessionEntry`, `deleteSessionEntry`,
  `patchSessionEntry` ve `listSessionEntries`, oturum deposu yolu
  gerektirmeyen, öncelikle SQLite kullanan API'lerdir. Durum özeti, yerel agent durumu, sistem
  sağlığı ve `openclaw sessions` listeleme komutu artık agent başına satırları doğrudan
  okuyor ve `sessions.json` yolları yerine agent başına SQLite veritabanı yollarını gösteriyor.
- Tüm depo üzerinde silme/ekleme işlemlerini `upsertSessionEntry`,
  `deleteSessionEntry`, `listSessionEntries` ve SQL temizleme sorgularıyla değiştirin.
  Çalışma zamanı için tamamlandı: sık kullanılan yollar artık satır API'lerini ve çakışma durumunda
  yeniden denenen satır yamalarını kullanıyor; kalan tüm depo içe aktarma/değiştirme yardımcıları,
  geçiş içe aktarma kodu ve SQLite arka uç testleriyle sınırlıdır.
  - `store-writer.ts` ve yazıcı kuyruğu testlerini silin. Tamamlandı.
  - Oturum satırı upsert/yama işlemlerinden çalışma zamanı eski anahtar budamasını ve
    takma ad silme parametrelerini kaldırın. Tamamlandı.

5. Çalışma zamanı JSON kayıt defteri davranışını silin.
   - Korumalı alan kayıt defteri okumalarını ve yazmalarını yalnızca SQLite kullanacak hâle getirin. Tamamlandı.
   - Monolitik ve parçalı JSON'u yalnızca geçiş adımından içe aktarın. Tamamlandı.
   - Parçalı kayıt defteri kilitlerini ve JSON yazmalarını kaldırın. Tamamlandı.

- Şekil, sık kullanılan yollardaki operasyonel durum olarak kalıyorsa kayıt defteri
  satırlarını genel opak JSON biçiminde depolamak yerine tek bir türü belirlenmiş kayıt defteri tablosu kullanın. Tamamlandı.

6. Dosya kilidi biçimindeki oturum mutasyonunu silin.
   - Çalışma zamanı kilit oluşturma ve çalışma zamanı kilit API'leri için tamamlandı.
   - Bağımsız eski `.jsonl.lock` doctor temizleme hattı kaldırıldı.
   - Durum bütünlüğünde artık yetim transkript dosyalarını budayan ayrı bir
     yol yoktur; doctor geçişi eski JSONL kaynaklarını tek bir yerde içe aktarır/kaldırır.
   - Gateway tekil örnek koordinasyonu, `gateway_locks` altında türü belirlenmiş SQLite
     `state_leases` satırlarını kullanır ve artık bir dosya kilidi dizini bağlantı noktası sunmaz.
   - Genel Plugin SDK tekilleştirme kalıcılığı artık dosya kilitleri veya JSON
     dosyaları kullanmaz; paylaşılan SQLite plugin durumu satırlarını yazar. Tamamlandı.
   - QMD koordinasyonu, gömmeler için paylaşılan bir SQLite kirası ve her
     koleksiyon/güncelleme/gömme yazıcısı için agent başına bir SQLite kirası kullanır. Çalışma zamanı artık
     `qmd/embed.lock.lock` veya `agents/<agentId>/qmd-write.lock.lock` oluşturmaz;
     Doctor yalnızca kesinlikle eskimiş, kullanımdan kaldırılmış yardımcı dosyaları kaldırır. Tamamlandı.

7. İşçileri veritabanından haberdar hâle getirin.
   - İşçiler kendi SQLite bağlantılarını açar.
   - Teslimatın, kanal geri çağrılarının ve yapılandırmanın sahibi üst süreçtir.
   - İşçi, canlı tanıtıcılar yerine agent kimliğini, çalıştırma kimliğini, dosya sistemi modunu
     ve veritabanı kayıt defteri kimliğini alır.
   - `vfs-only` deneysel kalır ve depolama kökü olarak agent veritabanını kullanır.
   - İlk aşamada etkin çalıştırma başına bir işçi kullanın. Havuzlama, veritabanı bağlantısı
     ömrü ve iptal davranışı sıradanlaşana kadar bekleyebilir.

8. Yedekleme entegrasyonu.
   - Yedeklemeye, çevrim içi yedeklemenin ardından çevrim dışı `VACUUM` ile
     genel, agent ve plugin veritabanlarının anlık görüntüsünü almayı öğretin. Durum varlığı altında keşfedilen `*.sqlite` dosyaları için tamamlandı;
     kullanılamayan sahip yetenekleri gerektiren plugin şemaları kapalı durumda başarısız olur.
   - Kanonik SQLite bütünlüğü ve şema kimliği için yedek doğrulamasının yanı sıra,
     ayrılmış plugin anlık görüntüleri için genel dosya biçimi doğrulaması ekleyin. Yedek oluşturma
     ve varsayılan arşiv doğrulaması için tamamlandı.
   - Yedekleme çalıştırma meta verilerini SQLite'a kaydedin. Arşiv yolu, durum ve bildirim JSON'u içeren
     paylaşılan `backup_runs` tablosu aracılığıyla tamamlandı.
   - Doğrulanmış arşiv anlık görüntülerinden geri yükleme ekleyin. Tamamlandı: `openclaw backup
restore` ayıklamadan önce doğrular, doğrulayıcının normalleştirilmiş
     bildirimini kullanır, `--dry-run` desteği sunar ve kaydedilmiş kaynak yollarını
     değiştirmeden önce `--yes` gerektirir.
   - VFS/çalışma alanı dışa aktarımını yalnızca istendiğinde dahil edin; oturum
     iç bileşenlerini JSON veya JSONL olarak dışa aktarmayın.

9. Kullanımdan kalkmış testleri ve kodu silin. Bilinen çalışma zamanı oturum yüzeyleri için tamamlandı.

- `sessions.json` veya transkript JSONL dosyalarının çalışma zamanında oluşturulduğunu
  doğrulayan testleri kaldırın. Çekirdek oturum deposu, sohbet, Gateway transkript olayları,
  önizleme, yaşam döngüsü, komut oturum girdisi güncellemeleri, otomatik yanıt sıfırlama/izleme,
  memory-core Dreaming fikstürleri, onay hedefi yönlendirmesi, oturum transkripti
  onarımı, güvenlik izni onarımı, yörünge dışa aktarımı ve oturum dışa aktarımı için tamamlandı.
  Active Memory transkript testleri artık SQLite kapsamlarını ve geçici veya
  kalıcı JSONL dosyası oluşturulmadığını doğruluyor.
  Çalışma zamanı artık JSONL transkriptlerini kısaltmadığı için
  eski Heartbeat transkript budama regresyonu kaldırıldı.
  Agent oturum listeleme aracı testleri artık eski `sessions.json` yollarını
  Gateway yanıt biçimi olarak modellemiyor; uygulama/UI/macOS testleri `databasePath` kullanıyor.
  `/status` transkript kullanım testleri artık JSONL dosyaları yazmak
  yerine doğrudan SQLite transkript satırlarını başlangıç verileriyle dolduruyor.
  Gateway oturum yaşam döngüsü testleri artık doğrudan SQLite transkript başlangıç verisi yardımcılarını
  kullanıyor; eski tek satırlı oturum dosyası fikstür biçimi sıfırlama
  ve silme kapsamından kaldırıldı.
  `sessions.delete` artık dosya döneminden kalma bir `archived: []` alanı döndürmüyor; silme
  yalnızca satır mutasyonu sonucunu bildiriyor. Eski `deleteTranscript` seçeneği de
  kaldırıldı: bir oturumun silinmesi kanonik `sessions` kökünü kaldırır ve
  SQLite'ın oturuma ait transkript, anlık görüntü ve yörünge satırlarını basamaklı
  olarak silmesine izin verir; böylece hiçbir çağıran transkript yetimleri bırakamaz
  veya bir temizleme dalını unutamaz.
  Bağlam motoru yörünge yakalama testleri artık
  `session.trajectory.jsonl` okumak yerine yalıtılmış bir agent veritabanındaki
  `trajectory_runtime_events` satırlarını okuyor.
  Docker MCP kanal başlangıç verisi betikleri artık doğrudan SQLite satırlarını dolduruyor. Doğrudan
  `sessions.json` yazmaları doctor fikstürleriyle sınırlıdır.
  Tool Search Gateway E2E, `agents/<agentId>/sessions/*.jsonl` dosyalarını taramak yerine araç çağrısı kanıtlarını
  SQLite transkript satırlarından okuyor.
  Memory-core ana bilgisayar olayları ve oturum külliyatı geçici satırları artık paylaşılan
  SQLite plugin durumunda bulunuyor; `events.jsonl` ve `session-corpus/*.txt` yalnızca eski
  doctor geçiş girdileridir. Etkin satırlar `.dreams/session-corpus` değil,
  `memory/session-ingestion/` sanal yollarını kullanır. Eski memory-core Dreaming
  onarım modülü ve CLI/Gateway testleri, çalışma zamanı artık bu külliyat için
  dosya arşivi onarımının sahibi olmadığından kaldırıldı. Memory-core
  köprü/genel artefakt testleri artık `.dreams/events.jsonl` sunmuyor;
  SQLite destekli sanal JSON artefakt adını kullanıyor.
  Genel SDK/Codex test belgeleri artık oturum dosyaları yerine SQLite oturum durumu
  diyor ve kanal turu örneği artık bir `storePath` bağımsız değişkeni sunmuyor.
  Matrix eşitleme durumu artık doğrudan SQLite plugin durumu deposunu kullanıyor. Etkin
  istemci/çalışma zamanı sözleşmeleri bir `bot-storage.json` yolu değil, hesap depolama kökü
  geçirir ve doctor, kaynağı silmeden önce eski `bot-storage.json` verilerini SQLite'a aktarır.
  QA Lab Matrix yeniden başlatma/yıkıcı senaryoları artık sahte `bot-storage.json` dosyaları
  oluşturmak veya silmek yerine SQLite eşitleme satırını doğrudan değiştiriyor ve
  E2EE alt katmanı sahte bir `sync-store.json` yolu yerine eşitleme deposu kökü geçiriyor.
  Matrix depolama kökü seçimi artık kökleri eski eşitleme/iş parçacığı JSON
  dosyalarına göre puanlamıyor; kalıcı kök meta verilerini ve gerçek kriptografik durumu kullanıyor.
  Çalışma zamanı SQLite oturum arka ucu test paketi artık bir
  `sessions.json` üretmiyor; eski kaynak fikstürleri artık bunları içe aktaran doctor
  testlerinde bulunuyor.
  Gateway oturum testleri artık bir `createSessionStoreDir` yardımcısı veya
  kullanılmayan geçici oturum deposu yolu kurulumu sunmuyor; fikstür dizinleri açıktır ve doğrudan
  satır kurulumu SQLite oturum satırı adlandırmasını kullanır.
  Yalnızca doctor tarafından kullanılan JSON5 oturum deposu ayrıştırıcı kapsamı altyapı testlerinden
  doctor geçiş testlerine taşındı; böylece çalışma zamanı test paketleri artık eski
  oturum dosyası ayrıştırmasının sahibi değil.
  Microsoft Teams çalışma zamanı SSO/bekleyen yükleme testleri artık JSON yardımcı dosyaları
  veya ayrıştırıcıları taşımıyor; eski SSO belirteci ayrıştırması yalnızca plugin
  geçiş modülünde bulunuyor. Telegram testleri artık sahte `/tmp/*.json` depo
  yollarını başlangıç verileriyle doldurmuyor; SQLite destekli mesaj önbelleğini doğrudan sıfırlıyor. Genel
  OpenClaw test durumu yardımcısı artık eski bir `auth-profiles.json`
  yazıcısı sunmuyor; doctor kimlik doğrulama geçiş testleri bu fikstürün yerel sahibidir.
  TUI son oturum işaretçileri, çalıştırma onayları, Active Memory
  geçişleri, Matrix tekilleştirme/başlatma doğrulaması, Memory Wiki kaynak eşitlemesi,
  mevcut konuşma bağlamaları, ilk katılım kimlik doğrulaması ve Hermes gizli bilgi içe aktarımlarına yönelik
  çalışma zamanı testleri artık eski yardımcı dosyalar üretmiyor veya eski dosya adlarının bulunmadığını
  doğrulamıyor. Davranışı SQLite satırları ve genel depo API'leri aracılığıyla
  kanıtlıyorlar; eski kaynak dosya adlarının ait olduğu tek yer doctor/geçiş testleridir.
  Cihaz/Node eşleştirme, kanal allowFrom, yeniden başlatma niyetleri,
  yeniden başlatma devri, oturum teslimat kuyruğu girdileri, yapılandırma sağlığı, iMessage
  önbellekleri, Cron işleri, PI transkript başlıkları, alt agent kayıt defterleri ve yönetilen
  görüntü ekleri için çalışma zamanı testleri de artık yalnızca yok sayıldıklarını
  veya bulunmadıklarını kanıtlamak amacıyla kullanımdan kaldırılmış JSON/JSONL dosyaları oluşturmuyor.
  PI taşma kurtarmasında artık bir SessionManager yeniden yazma/kısaltma
  geri dönüşü yoktur: araç sonucu kısaltma ve bağlam motoru transkript yeniden yazmaları
  SQLite transkript satırlarını değiştirir, ardından etkin istem durumunu veritabanından yeniler.
  Kalıcı SessionManager mesaj eklemeleri, üst öğe seçimi ve eşgüçlülük için atomik SQLite
  transkript ekleme yardımcısına devredilir. Normal meta veri/özel girdi eklemeleri de
  mevcut üst öğeyi SQLite içinde seçer; böylece eski yönetici örnekleri SQLite öncesi
  üst öğe zinciri yarışlarını yeniden ortaya çıkarmaz.
  Tur ortası ön kontroller ve `sessions_yield` için sentetik PI kuyruk temizliği artık
  SQLite transkript durumunu doğrudan kırpar; eski SessionManager kuyruk kaldırma
  köprüsü ve testleri silindi.
  Compaction denetim noktası yakalama da yalnızca SQLite'tan anlık görüntü alır; çağıranlar artık
  alternatif transkript kaynağı olarak canlı bir SessionManager geçirmez.
- Eski dosyaları başlangıç verileriyle dolduran testleri yalnızca geçiş için tutun.
- Etkin çalışma zamanı yüzeyleri için JSON dosyası kanıtının yerini SQL satırı kanıtı aldı.

- Eski oturum/önbellek JSON yollarına çalışma zamanı yazmaları için statik yasaklar ekleyin.
  Depo koruması için tamamlandı.

10. Geçiş raporunu denetlenebilir hâle getirin.
    - Geçiş çalıştırmalarını başlangıç/bitiş zaman damgaları, kaynak
      yolları, kaynak karmaları, sayımlar, uyarılar ve yedekleme yoluyla SQLite'a kaydedin.
      Tamamlandı: eski durum geçişi yürütmeleri artık kaynak yol/tablo envanteri,
      kaynak dosyası SHA-256 değeri, boyutlar, kayıt sayıları, uyarılar ve yedekleme yolu içeren
      bir `migration_runs` raporunu kalıcı hâle getiriyor.
      Tamamlandı: eski durum geçişi yürütmeleri ayrıca kaynak düzeyinde denetim
      ve gelecekteki atlama/geriye dönük doldurma kararları için `migration_sources` satırlarını kalıcı hâle getiriyor.
    - Uygulamayı eşgüçlü hâle getirin. Kısmi bir içe aktarmadan sonra yeniden çalıştırma,
      önceden içe aktarılmış bir kaynağı atlamalı veya kararlı anahtara göre birleştirmelidir.
      Tamamlandı: oturum dizinleri, transkriptler, teslimat kuyrukları, plugin durumu, görev
      defterleri ve agent'a ait genel SQLite satırları kararlı anahtarlar veya
      upsert/değiştirme semantiği aracılığıyla içe aktarılır; böylece yeniden çalıştırmalar kalıcı
      satırları çoğaltmadan birleştirir.
    - Başarısız içe aktarmalar özgün kaynak dosyasını yerinde tutmalıdır.
      Tamamlandı: başarısız transkript içe aktarmaları artık özgün JSONL kaynağını
      algılanan yolunda bırakır ve `migration_sources`, sonraki doctor çalıştırması için kaynağı
      `removed_source=0` ile `warning` olarak kaydeder.

## Performans Kuralları

- İş parçacığı/işlem başına bir bağlantı uygundur; tanıtıcıları çalışanlar
  arasında paylaşmayın.
- WAL, `foreign_keys=ON`, 5 saniyelik meşgul zaman aşımı ve kısa `BEGIN IMMEDIATE`
  yazma işlemleri kullanın. SQLite'ın tek meşgul beklemesinin üzerine eşzamanlı kilit
  yeniden denemeleri eklemeyin.
- Açık muteks/geri basınç semantiği sunan eşzamansız bir işlem API'si
  eklenmedikçe işlem yardımcılarını eşzamanlı tutun.
- Üst öğe teslimatı yazmalarını küçük ve işlemsel tutun.
- Tüm depoyu yeniden yazmaktan kaçının; satır düzeyinde upsert/delete kullanın.
- Sık kullanılan kodu taşımadan önce aracıya göre listeleme, oturuma göre listeleme,
  güncellenme zamanı, çalıştırma kimliği ve süre sonu yolları için dizinler ekleyin.
- Büyük yapıtları, medyayı ve vektörleri base64 veya sayısal dizi JSON olarak değil,
  BLOB'lar veya parçalara ayrılmış BLOB satırları olarak depolayın.
- Opak plugin durumu girdilerini küçük ve kapsamlı tutun.
- Dosya sistemi budaması yerine TTL/süre sonu için SQL temizliği ekleyin.
  Veritabanına ait çalışma zamanı depoları için tamamlandı: medya, plugin durumu,
  plugin blob'ları, kalıcı tekilleştirme ve aracı önbelleğinin tümü SQLite satırları
  üzerinden sona erer. Kalan dosya sistemi temizliği, geçici somutlaştırmalar veya
  açık kaldırma komutlarıyla sınırlıdır.

## Statik Yasaklar

Eski durum yollarına yeni çalışma zamanı yazmalarında başarısız olan bir depo denetimi ekleyin:

- `sessions.json`
- `*.trajectory.jsonl`, somutlaştırılmış destek paketi çıktıları hariç
- `.acp-stream.jsonl`
- `acp/event-ledger.json`
- `cache/*.json` çalışma zamanı önbellek dosyaları
- `agents/<agentId>/agent/auth.json`
- `agents/<agentId>/agent/models.json`
- `credentials/oauth.json`
- `github-copilot.token.json`
- `openrouter-models.json`
- `auth-profiles.json`
- `auth-state.json`
- `exec-approvals.json`
- `openclaw-workspace-state.json`
- `workspace-state.json`
- `workspace-attestations/*.attested`
- eşdüzey `<workspace>.attested`
- Matrix `credentials*.json` ve `recovery-key.json`
- `cron/runs/*.jsonl`
- `cron/jobs.json`
- `jobs-state.json`
- `device-pair-notify.json`
- `devices/pending.json` / `devices/paired.json` / `devices/bootstrap.json`
  (2026.7'de kullanımdan kaldırıldı: çalışma zamanı deposu, paylaşılan durum
  veritabanındaki `device_pairing_*` / `device_bootstrap_tokens` öğesidir; eşleştirilmiş
  kayıtlar Gateway başlatılırken içe aktarılır, geçici bekleyen/önyükleme satırları bırakılır)
- `nodes/pending.json` / `nodes/paired.json` (2026.7'de kullanımdan kaldırıldı: Gateway başlatılırken eşleştirilmiş cihaz kayıtlarına katlandı)
- `identity/device.json`
- `identity/device-auth.json` (kullanımdan kaldırıldı; yalnızca Doctor tarafından `device_auth_tokens` içine aktarılır)
- `push/web-push-subscriptions.json` (kullanımdan kaldırıldı; yalnızca Doctor tarafından `web_push_subscriptions` içine aktarılır)
- `push/vapid-keys.json` (kullanımdan kaldırıldı; yalnızca Doctor tarafından `web_push_vapid_keys` içine aktarılır)
- `push/apns-registrations.json` (kullanımdan kaldırıldı; yalnızca Doctor tarafından `apns_registrations` içine aktarılır)
- `process-leases.json`
- `gateway-instance-id`
- `session-toggles.json`
- Memory-core `.dreams/events.jsonl`
- Memory-core `.dreams/session-corpus/`
- Memory-core `.dreams/daily-ingestion.json`
- Memory-core `.dreams/session-ingestion.json`
- Memory-core `.dreams/short-term-recall.json`
- Memory-core `.dreams/phase-signals.json`
- Memory-core `.dreams/short-term-promotion.lock`
- Skill Workshop `skill-workshop/<workspace>.json`
- Skill Workshop `skill-workshop/skill-workshop-review-*.json`
- Nostr `bus-state-*.json`
- Nostr `profile-state-*.json`
- `calls.jsonl`
- `known-users.json`
- `ref-index.jsonl`
- QQBot `session-*.json`
- BlueBubbles `bluebubbles/catchup/*.json`
- BlueBubbles `bluebubbles/inbound-dedupe/*.json`
- Telegram `update-offset-*.json`
- Telegram `sticker-cache.json`
- Telegram `*.telegram-messages.json`
- Telegram `*.telegram-sent-messages.json`
- Telegram `*.telegram-topic-names.json`
- Telegram `thread-bindings-*.json`
- iMessage `catchup/*.json`
- iMessage `reply-cache.jsonl`
- iMessage `sent-echoes.jsonl`
- Microsoft Teams `msteams-conversations.json`
- Microsoft Teams `msteams-polls.json`
- Microsoft Teams `msteams-sso-tokens.json`
- Microsoft Teams `*.learnings.json`
- Matrix `bot-storage.json`
- Matrix `sync-store.json`
- Matrix `thread-bindings.json`
- Matrix `inbound-dedupe.json`
- Matrix `startup-verification.json`
- Matrix `storage-meta.json`
- Matrix `crypto-idb-snapshot.json`
- Discord `model-picker-preferences.json`
- Discord `command-deploy-cache.json`
- korumalı alan kayıt defteri parçası JSON dosyaları
- `plugin-state/state.sqlite`
- geçici `openclaw-state.sqlite` çalışma zamanı yardımcı dosyaları
- `tasks/runs.sqlite`
- `tasks/flows/registry.sqlite`
- `bindings/current-conversations.json`
- `restart-sentinel.json`
- `gateway-restart-intent.json`
- `gateway-supervisor-restart-handoff.json`
- `gateway.<hash>.lock`
- `qmd/embed.lock.lock`
- `agents/<agentId>/qmd-write.lock.lock`
- `commands.log`
- `config-health.json`
- `port-guard.json`
- `settings/voicewake.json`
- `settings/voicewake-routing.json`
- `plugin-binding-approvals.json`
- `plugins/installs.json`
- `audit/file-transfer.jsonl`
- `audit/crestodian.jsonl`
- `crestodian/rescue-pending/*.json`
- `openclaw/rescue-pending/*.json`
- `plugins/phone-control/armed.json`
- Memory Wiki `.openclaw-wiki/log.jsonl`
- Memory Wiki `.openclaw-wiki/state.json`
- Memory Wiki `.openclaw-wiki/locks/`
- Memory Wiki `.openclaw-wiki/source-sync.json`
- Memory Wiki `.openclaw-wiki/import-runs/*.json`
- Memory Wiki `.openclaw-wiki/cache/agent-digest.json`
- Memory Wiki `.openclaw-wiki/cache/claims.jsonl`
- ClawHub `.clawhub/lock.json`
- ClawHub `.clawhub/origin.json`
- Tarayıcı profili süslemesi `.openclaw-profile-decorated`
- `SessionManager.open(...)` dosya destekli oturum açıcıları
- `SessionManager.listAll(...)` ve `TranscriptSessionManager.listAll(...)`
  döküm listeleme cepheleri
- `SessionManager.forkFromSession(...)` ve
  `TranscriptSessionManager.forkFromSession(...)` döküm çatallama cepheleri
- `SessionManager.newSession(...)` ve `TranscriptSessionManager.newSession(...)`
  değiştirilebilir oturum değiştirme cepheleri
- `SessionManager.createBranchedSession(...)` ve
  `TranscriptSessionManager.createBranchedSession(...)` dal oturumu cepheleri

Yasak, testlerin eski düzenekler oluşturmasına ve geçiş kodunun eski dosya
kaynaklarını okumasına/içe aktarmasına/kaldırmasına izin vermelidir. Yayımlanmamış
SQLite yardımcı dosyaları yasak kalır ve Doctor içe aktarma izinleri almaz.

## Tamamlanma Ölçütleri

- Çalışma zamanı verileri ve önbellek yazmaları genel veya aracı SQLite veritabanına gider.
- Çalışma zamanı artık oturum dizinleri, döküm JSONL'si, korumalı alan kayıt defteri
  JSON'u, görev yardımcı SQLite'ı veya plugin durumu yardımcı SQLite'ı yazmaz.
  Yayımlanmamış görev ve plugin durumu yardımcı SQLite içe aktarıcıları silinir.
- Eski dosya içe aktarma yalnızca Doctor tarafından yapılır.
- Yedekleme, sıkıştırılmış SQLite anlık görüntüleri ve bütünlük kanıtı içeren tek bir arşiv üretir.
- Aracı çalışanları disk, VFS karalama alanı veya deneysel yalnızca VFS
  depolamasıyla çalışabilir.
- Yapılandırma ve açık kimlik bilgisi dosyaları, beklenen tek kalıcı
  veritabanı dışı denetim dosyaları olarak kalır.
- Depo denetimleri, eski çalışma zamanı dosya depolarının yeniden kullanılmasını önler.
