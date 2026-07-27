---
read_when:
    - Canlı bir Gateway üzerinde Yol 3 SQLite depolama geçişini doğruluyorsunuz
    - Beklenen eski JSONL sapmasını çalışma zamanı hatalarından ayırt etmeniz gerekir
    - Agent tarafından yönlendirilen canlı SQLite E2E test altyapısını oluşturuyor veya inceliyorsunuz
summary: Path 3 SQLite oturum/transkript geçişinin canlı Gateway kanıtı için tasarım
title: Yol 3 canlı SQLite E2E test düzeneği
x-i18n:
    generated_at: "2026-07-27T00:16:16Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 2749bf47cb4967bc80a5ed37a12f2a553f3b388ed8cd90cfb3217e1b5e8afae9
    source_path: reference/path3-live-sqlite-e2e-harness.md
    workflow: 16
---

Path 3 canlı SQLite E2E düzeneği, eski JSONL dosyaları geçiş girdisi veya arşiv malzemesi olarak kalırken Gateway'in kanonik oturum ve transkript deposu olarak SQLite kullandığını kanıtlar. Bu, normal bir kullanıcı tanılama aracı değil, bakımcılar için bir kanıt düzeneğidir.

Bir Gateway geçiş sonrası trafiği işledikten sonra, eski JSONL eşliği artık geçerli bir çalışma zamanı sağlık sinyali değildir. Sağlıklı biçimde geçirilmiş bir Gateway'deki SQLite transkript satırları eski JSONL sayılarından farklı olabilir; çünkü yeni turlar yalnızca SQLite'ı ilerletmelidir. Bu nedenle canlı düzenek her adımda Gateway davranışını, SQLite satır hareketini, eski dosyaların hareketsizliğini ve günlük sağlığını ölçmelidir.

## Komut biçimi

Amaçlanan canlı komut şudur:

```bash
node scripts/path3-live-sqlite-e2e.mjs \
  --url http://127.0.0.1:18789 \
  --agent main \
  --session-key agent:main:path3-live-e2e:<timestamp> \
  --json
```

Komut, zaten çalışmakta olan bir Gateway'e bağlanır. Daha sonra açık bir geçiş modu eklenmedikçe geçişi başlatmaz, durdurmaz, içe aktarmaz veya yeniden çalıştırmaz. Bir CI veya yalıtılmış yerel varyant
`test/helpers/openclaw-test-instance.ts` kullanabilir, ancak canlı kanıt yolu gerçek operatör Gateway'ini ve onun gerçek, aracı başına SQLite veritabanını incelemelidir.

## Yalıtılmış derlenmiş CLI kanıtı

Derlenmiş CLI kanıt çalıştırıcısı yalıtılmış bir eski oturum deposunu başlangıç verileriyle doldurur, yeniden derlenmiş Gateway'i başlatır ve başlangıcın, çalışma zamanı okumaları başlamadan önce etkin eski oturumları SQLite'a aktardığını kanıtlar. İlk Gateway başlangıcından önce `openclaw doctor --fix` çalıştırılmamalıdır; çünkü bu, kullanıcıların geçişten sonraki ilk açılışta karşılaştığı yükseltme yolu yerine manuel geçiş yolunu kanıtlar.

Başlangıç içe aktarımından sonra yalıtılmış kanıt, tanılama kanıtı olarak
`openclaw doctor --session-sqlite inspect` ve
`openclaw doctor --session-sqlite validate` çalıştırabilir. Bu doctor komutları, başlangıç yükseltme kanıtının geçiş yürütücüsü değildir. Ayrı doctor içe aktarma senaryoları, eski transkript dosyalarını yörünge yan dosyalarıyla birlikte başlangıç verileriyle doldurmalı ve SQLite kanonik kalırken doctor'ın bu yapıtları arşivlediğini doğrulamalıdır.

## Ön kontrol

Ön kontrol bir temel durum toplar ve Gateway kullanılamıyorsa bir kanıt turu göndermeden önce başarısız olur:

- `GET /health` ve Gateway ayrıntılı durumu, çalışan ve erişilebilir bir
  Gateway bildirmelidir.
- CLI ve Gateway sürümleri, test edilen dalla eşleşmelidir.
- Düzenek, etkin Gateway dosya günlüğü için bir günlük imleci kaydeder.
- Düzenek; `sessions`,
  `session_entries`, `transcript_events`, `transcript_event_identities` ve
  `session_routes` için aracı başına SQLite tablo sayılarını kaydeder.
- Düzenek; `mtime`, `size` ve eski
  `sessions.json`, başvurulan JSONL dosyaları ve olası kanıt oturumu JSONL
  yollarının varlığını kaydeder.
- `lsof -p <gateway-pid>`, SQLite DB/WAL/SHM tanıtıcılarını göstermeli ve etkin
  `.jsonl` veya `sessions.json` tanıtıcıları göstermemelidir.

`openclaw doctor --session-sqlite validate`, canlı modda yalnızca bilgi amaçlıdır.
Geçiş sonrası trafikten sonra eski dosyalara kıyasla beklenen sapmayı bildirebilir. Düzenek, doctor çıktısını çalışma zamanının başarılı/başarısız karar mercii olarak değil, sınıflandırma ve geçiş envanteri için kullanmalıdır.

## Aracı tarafından yürütülen senaryo

Canlı senaryo özel bir kanıt oturumu anahtarı kullanır ve mümkün olan her yerde Gateway'i genel RPC yolları üzerinden yönlendirir. Sıradan kalıcılığı çalıştırmak için tek bir aracı turu yeterli olmalıdır; ancak tam kanıt, daha önce ayrı ayrı canlı kontroller gerektiren 3.1b bağlantı noktalarını kapsamalıdır:

- Sıradan sohbet turu: kanıt oturumunu oluşturun veya yeniden kullanın, gerçek bir aracı
  istemi gönderin, son asistan sonucunu bekleyin ve `chat.history` ya da
  eşdeğer Gateway projeksiyonunu doğrulayın.
- Transkript kimliği: aynı işaretçinin Gateway geçmişinde ve varsa kararlı olay
  kimliği satırları dâhil SQLite transkript satırlarında göründüğünü doğrulayın.
- Oturum meta verisi erişimcileri: kanıt oturumunu ve seçilen mevcut canlı
  oturumları Gateway/oturum erişimcileri üzerinden okuyun ve SQLite satırlarıyla karşılaştırın.
- Oturum yaması projeksiyonu: kanıt oturumunda geri alınabilir bir model/oturum meta verisi
  değişikliği uygulayın, ardından projekte edilen satır ile Gateway yanıtının eşleştiğini doğrulayın.
- Compaction kontrol noktası yaşam döngüsü: yalnızca kanıt oturumunda veya düzenek
  tarafından oluşturulan sentetik bir sabit veri oturumunda bir kontrol noktasını listeleyin, dallandırın ve geri yükleyin.
- Yeniden başlatma kurtarması: güvenli kurtarma işaretçisi yolunu denetimli bir kanıt
  oturumunda veya yalıtılmış bir test örneğinde çalıştırın; canlı mod bu adımı yalnızca
  hedef oturum kümesi açıkça belirtilmiş ve geri alınabilir olduğunda çalıştırabilir.
- Temizleme yaşam döngüsü: kanıt oturumunu silin veya sıfırlayın, ardından SQLite
  yaşam döngüsü satırlarını ve arşivlenmiş transkript durumunu doğrulayın.

WhatsApp veya sesli arama girişi gibi canlı operatör Gateway'inde güvenle çalıştırılamayan taşıma sistemine özgü bağlantı noktaları, sahte harici taşıma sistemi yerine aynı SQLite sözleşmesine karşı sahip düzeyinde çalışma zamanı yoklamaları kullanmalıdır.

## Adım başına doğrulamalar

Her adım, önceki ve sonraki durumun anlık görüntüsünü alır ve yapılandırılmış bir doğrulama kaydı yazar:

- SQLite satır sayıları yalnızca beklendiği yerlerde ilerler.
- Yörünge çalışma zamanı satırları, çalışma zamanı olaylarını kaydeden işaretçi destekli kanıt oturumları için ilerler.
- Kanıt oturumu satırı beklenen `session_id`, duruma, zaman damgalarına,
  meta verilere ve rota satırlarına sahiptir.
- Gateway geçmişi/oturum projeksiyonu, SQLite transkript kuyruğuyla eşleşir.
- Hiçbir kanıt oturumu JSONL dosyası oluşturulmaz veya değiştirilmez.
- Hiçbir kanıt oturumu `.trajectory.jsonl`, `.trajectory-path.json` veya
  işaretçiden türetilen `trajectory/<session>.jsonl` yan dosyası oluşturulmaz.
- Mevcut eski JSONL dosyaları ve `sessions.json`, adım açıkça çevrimdışı
  bir geçiş veya arşiv işlemi olmadıkça değiştirilmeden kalır.
- Gateway işlemi `.jsonl` veya `sessions.json` tanıtıcılarını açmaz.
- Önceki imleçten sonraki günlüklerde; senaryo açıkça izin listesine almadıkça
  `ERROR`, `FATAL`, `SQLITE_`,
  `no such column`, oturum deposu kullanılamıyor, yeniden başlatma kurtarma hatası veya
  transkript uzlaştırma uyarısı bulunmaz.

Günlük taraması, başarılı/başarısız sözleşmesinin bir parçasıdır. Sağlık kontrollerine yanıt veren ancak SQLite şema hataları veya tekrarlanan transkript uzlaştırma hataları yayan bir Gateway, Path 3 için başarılı değildir.

## Kanıt yapıtı

Düzenek, kanıtı `.artifacts/path3-live-e2e/<timestamp>/` altında yazmalı ve git dışında tutmalıdır:

- `summary.json`: komut bağımsız değişkenleri, Gateway sürümü, sonuç, başarısız doğrulama ve
  yapıt yolları.
- `sqlite-before.json` ve `sqlite-after.json`: satır sayıları ve seçilen kanıt
  satırları.
- `legacy-files.json`: eski dosya varlığı, `mtime`, boyut ve her
  dosyanın değişip değişmediği.
- `gateway-log-scan.json`: imleç aralığı, eşleşen günlük satırları ve izin listesi
  kararları.
- `events.jsonl`: PR kanıt yorumlarına uygun, sıralı adım başına gözlemler.

PR kanıtı, tam transkriptleri veya özel mesaj içeriğini yapıştırmak yerine bu yapıtları özetlemelidir.

## Güvenlik kuralları

- Canlı mod, Gateway çalışırken eski JSONL'yi asla yeniden içe aktarmamalıdır.
- Canlı mod, açıkça seçilmiş ve geri alınabilir onarım yoklamaları dışında kanıt dışı oturumları değiştirmemelidir.
- Yıkıcı veya geniş kapsamlı tüm geçiş adımları, etkilenen SQLite DB'nin ve eski oturum dizininin yeni bir yedeğini gerektirir.
- Yedekler, dokunulan aracı DB'si/oturum diziniyle sınırlandırılmalı ve sınırsız disk büyümesini önlemek için tek bir kanıt çalıştırması boyunca yeniden kullanılmalıdır.
- Temizleme adımı, çağıran `--keep-artifacts` iletmediği sürece geride hiçbir kanıt oturumu, kanıt JSONL'si veya değiştirilmiş eski dosya bırakmamalıdır.

## Başarılı sonuç

Başarılı bir canlı çalıştırma; Gateway'in aracı tarafından yürütülen gerçek bir oturum akışını kabul ettiği, gözlemlenen tüm kanonik durumun SQLite'ta bulunduğu, eski çalışma zamanı dosyalarının hareketsiz kaldığı ve ölçülen zaman aralığında günlük sağlığının temiz kaldığı anlamına gelir. Bu, canlı trafikten sonra eski JSONL eşliğinin temiz kalacağı anlamına gelmez; SQLite kanonik depo olduğunda canlı sapma beklenir.
