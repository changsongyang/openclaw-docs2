---
read_when: You want agent sessions to run on ephemeral cloud machines instead of the Gateway host, or you are configuring cloudWorkers profiles.
sidebarTitle: Cloud Workers
status: active
summary: 'Oturumları tek kullanımlık bulut makinelerine yönlendirme: kaynak sağlama, worker çalışma zamanı, proxy üzerinden çıkarım ve akış hâlinde sonuçlar'
title: Bulut İşçileri
x-i18n:
    generated_at: "2026-07-26T23:40:07Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 5620be5957a20019d4687b3ec935ec1116fdef6ea05e42ab766508d2b54322a2
    source_path: gateway/cloud-workers.md
    workflow: 16
---

Bulut çalışanları, oturumla ilgili her şey her zamanki yerinde kalırken bir oturumun ajan döngüsünü tek kullanımlık bir bulut makinesinde çalıştırmasına olanak tanır: kenar çubuğunda görünür, canlı olarak yayınlanır ve transkript Gateway tarafından yönetilir. Gateway bir makine kiralar, makineye OpenClaw'ın sabitlenmiş bir kopyasını yükler, oturumun çalışma alanını makineyle eşitler ve tur döngüsünü kısıtlı bir `openclaw worker` sürecine devreder. Model çağrıları Gateway üzerinden geri proxy'lenir; böylece sağlayıcı kimlik bilgileri makinenizden asla ayrılmaz ve sağlayıcı kesintisiz tek bir akış gördüğü için istem önbelleğe alma çalışmaya devam eder.

İş tamamlandığında (veya makine çöktüğünde) makine atılır. Kalıcı durum — transkript, çalışma alanı commit'leri, yerleşim kayıtları — Gateway ile birlikte tutulur.

<Note>
Bulut çalışanları isteğe bağlıdır ve bir profil yapılandırılana kadar görünmez. Yapılandırılmamış kurulumlarda yeni RPC'ler, yapılandırma veya kullanıcı arayüzü görünmez.
</Note>

## Nerede ne çalışır?

| Konu                                                    | Konum                                                                            |
| ------------------------------------------------------- | -------------------------------------------------------------------------------- |
| Ajan döngüsü + araçlar (`exec`, `read`, `write`, `edit`, …) | Bulut çalışanı makinesi                                                          |
| Model çıkarımı ve sağlayıcı kimlik bilgileri            | Gateway (`{provider, model}` referansı tarafından proxy'lenir)                   |
| Transkript (kalıcı, oturum deposu)                      | Gateway                                                                          |
| Kenar çubuğuna canlı yayın                              | Çalışanın yeniden oynatılabilir olay akışıyla beslenen Gateway dağıtımı          |
| Çalışma alanının git geçmişi                            | Kimlik bilgileri olmadan makinede oluşturulur; Gateway commit'leri devralır ve push/PR işlemlerini yönetir |

Makine, `sshd` dışında gelen bağlantılar için herhangi bir porta ihtiyaç duymaz: Gateway sabitlenmiş SSH üzerinden dışarıya bağlanır ve ters tünel, çalışanın WebSocket bağlantısını geri taşır. Birlikte gelen Crabbox sağlayıcısı genel SSH yolunu zorunlu kılar ve yönetilen Tailscale kaydını devre dışı bırakır. Giden internet erişimi sağlayıcı politikasına bağlıdır; varsayılan AWS profili, ağını veya güvenlik grubunu kısıtlamadığınız sürece internete erişebilir.

## Gereksinimler

- Bir çalışan sağlayıcısı Plugin'i. Birlikte gelen `crabbox` Plugin'i, bulut arka uçları (AWS, Hetzner ve diğerleri) arasında kiralamalara aracılık eden [Crabbox](https://github.com/openclaw/crabbox) CLI'ını çalıştırır. `crabbox` ikili dosyası `PATH` üzerinde bulunmalı (veya `settings.binary` ayarlanmalı) ve sağlayıcı kimlik bilgileri önceden yapılandırılmış olmalıdır. AWS kabulü için Crabbox 0.38.1 veya üzeri gerekir.
- Crabbox AWS çalışanlarında geçerli `aws.instanceProfile` boş olmalıdır. Sağlayıcı, ayırmadan önce `crabbox config show --json` öğesini denetler ve ardından `crabbox inspect --json` öğesinin EC2 `DescribeInstances` üzerinden `providerMetadata.instanceProfileAttached: false` bildirmesini gerektirir. Örnek rolüne sahip veya yetkili meta verileri bulunmayan kiralamalar durdurulur ve reddedilir.
- Kiralanan makinede Node.js. Temel bulut imajlarında genellikle bulunmaz; profilin `setup` komutunda yükleyin.
- Oturuma ait yönetilen bir worktree içeren oturum (`worktree: true` ile oluşturun). Gönderim, bu worktree'nin içeriğini taşır; düz dizinler bildirim aynası olarak eşitlenir.

## Yapılandırma

`openclaw.json` içinde `cloudWorkers.profiles` altına bir profil ekleyin:

```json
{
  "cloudWorkers": {
    "profiles": {
      "aws": {
        "provider": "crabbox",
        "install": "bundle",
        "settings": {
          "provider": "aws",
          "class": "standard",
          "ttl": "8h",
          "idleTimeout": "45m",
          "setup": "test -x /usr/bin/node || (curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash - && sudo apt-get install -y nodejs)"
        }
      }
    }
  }
}
```

Profil alanları:

| Anahtar    | Anlamı                                                                                                                                                                                                                                         |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `provider` | Bir Plugin tarafından kaydedilen çalışan sağlayıcısı kimliği (birlikte gelen Plugin için `crabbox`).                                                                                                                                           |
| `install`  | `bundle` (varsayılan), çalışan Gateway'in derlemesini gönderir; `npm`, sabitlenmiş bütünlük bilgisiyle tam olarak yayımlanmış Gateway sürümünü yükler. `npm`, Gateway'in paketlenmiş bir sürümden çalışmasını gerektirir. |
| `settings` | Sağlayıcıya ait JSON. Crabbox için: `provider` (arka uç), `class` (makine sınıfı), `ttl`, `idleTimeout` (Go süreleri), isteğe bağlı `setup` ve mutlak `binary` yolu. OpenClaw bu kiralamalarda genel SSH'yi zorunlu kılar ve yönetilen Tailscale'i devre dışı bırakır. |
| `lifetime` | İsteğe bağlı depolanmış politika (`idleTimeoutMinutes`, `maxLifetimeMinutes`).                                                                                                                                                                  |

### Kurulum komutu

`settings.setup`, kiralanan makine SSH için hazır olduktan sonra ve OpenClaw yüklenmeden önce çalışır. **Her** sağlama girişiminde (kesintiye uğramış bir gönderimden sonraki yeniden oynatmalar dâhil) çalıştığından yinelenebilir olmalıdır; örnekteki gibi yüklemeleri bir `command -v`/`test -x` denetimiyle koruyun. Kurulum başarısız olursa sağlayıcı kiralamayı durdurur ve gönderim güvenli biçimde başarısız olur; yarı yapılandırılmış hiçbir makine çalışır durumda bırakılmaz.

### Yükleme kanalları

- **`bundle`**, çalışan Gateway'in `dist` öğesini, budanmış bir `package.json` öğesini ve derlemenin başvurduğu tüm çalışma alanı paketlerini paketler; bunların tümü bir içerik karmasıyla kapsanır. Makine, değiştirilmemiş paketi bu karmaya göre doğrular ve ardından üretim npm bağımlılıklarını yükler (betikler devre dışıdır). Bir geliştirme derlemesini çalışanda bu şekilde çalıştırabilirsiniz.
- **`npm`**, sürümün genel kayıt defterinde bulunduğunu kanıtlar, SHA-512 bütünlüğünü sabitler ve Gateway ile tam olarak eşleşen `openclaw@<version>` öğesini yükler.

## Oturum gönderme

Control UI'da **New Session** öğesini açın, yapılandırılmış çalışma zamanı OpenClaw olan bir ajan seçin, **Where** menüsünden yapılandırılmış bir **Cloud · profile** hedefi seçin ve görevi başlatın. Bulut seçimi gerekli yönetilen worktree'yi otomatik olarak etkinleştirir; Gateway oturumu oluşturur, gönderimi tamamlar ve ancak bundan sonra ilk turu gönderir. Oturum kenar çubuğundaki sunucu rozeti kalıcı yerleşim durumunu gösterir. Harici CLI oturum kataloglarında bulut hedefleri sunulmaz.

Eşdeğer RPC akışı şöyledir:

Yönetilen bir worktree ile oturum oluşturun, ardından gönderin (RPC, `operator.admin` gerektirir ve yalnızca profiller yapılandırıldığında bulunur):

Bulut çalışanları OpenClaw ajan çalışma zamanını çalıştırır. Bu çalışma zamanına çözümlenen bir `openai/*` veya başka bir model seçin; `claude-cli` gibi harici bir CLI çalışma zamanı için yapılandırılan oturumlar gönderilemez.

```bash
openclaw gateway call sessions.create \
  --params '{"key":"agent:main:big-refactor","worktree":true,"cwd":"/path/to/repo","worktreeName":"big-refactor"}'

openclaw gateway call sessions.dispatch \
  --timeout 1500000 \
  --params '{"key":"agent:main:big-refactor","profileId":"aws"}'
```

`sessions.dispatch`, yerel tur kabulünü kapatır, etkin işi boşaltır, kiralamayı sağlar, kurulumu çalıştırır, OpenClaw'ı önyükler, çalışma alanını eşitler ve yerleşim `active` çalışan sahipliğine ulaştığında döner. İlk gönderim için birkaç dakika ayırın; sağlayıcının desteklediği yerlerde kiralamalar ve yüklemeler önbelleğe alınır. Bundan sonra oturumla her zamanki gibi iletişim kurun; turlar otomatik olarak çalışana yönlendirilir.

Tamamlanan çalışan turları, uygun ve boyutu sınırlı çalışma alanı dosyalarını tur talebi serbest bırakılmadan önce oturumun yönetilen worktree'sine geri uzlaştırır. Sonlandırıcı çalışan olayı, onaylanmadan önce kalıcı bir bekleyen-sonuç engeli oluşturur. Ardından Gateway, tam bulut sonucunu uygulamadan önce `refs/openclaw/worker-results/` altında bir Git referansı olarak aşamalar; böylece uygulama sırasında Gateway dursa bile bulut sürümü kurtarılabilir durumda kalır. Çalışma alanı sonuçları Git dosya semantiğini kullanır: normal dosyalar, yürütülebilir bitleri, sembolik bağlantılar, eklemeler, değişiklikler ve silmeler korunurken boş dizinler ve diğer dizin kipleri korunmaz. Ortaya çıkan dosya değişiklikleri normal inceleme ve commit işlemi için yönetilen worktree'de kalır.

Uygulama, gönderim zamanı bildirimini birleştirme tabanı olarak kullanır. Yalnızca bulutta yapılan değişiklikler uygulanır, yalnızca yerelde yapılan değişiklikler yerinde kalır ve her iki tarafta değişen yollar için üç yönlü yereli koruma politikası kullanılır. Çakışmalı bir tur yine de tamamlanır: transkript sınırlı yol özetini ve aşamalanmış sonuç referansını bildirir, yerleşim aynı çakışmayı Control UI'a sunar ve çakışmayan bulut değişiklikleri uygulanmış durumda kalır. Bildirim, mevcut bir bulut dosyasını incelemek için `git show <ref>:<path>` ve herhangi bir çalışma alanı dizininden almak için üst düzey, değişmez yol belirtimli bir `git checkout <ref> -- <path>` komutu içerir. Komutları Bash veya zsh'de (Windows'ta Git Bash) çalıştırın. İnceleme yolun bulunmadığını bildirirse bulut sonucu bu yolu silmiştir; tutulan yerel yolu doğrulayıp elle kaldırın. Checkout bir dosya/dizin engeli bildirirse engelleyen yerel yolu taşıyın veya kaldırın ve yeniden deneyin. Aşamalanmış referansın kendisi yoksa bildirimi eski kabul edin ve yerel yolu değiştirmeyin. Çakışmalı aşamalanmış referanslar, normal tur engeli kaldırıldıktan sonra da kullanılabilir durumda kalır; sonraki temiz bir sonuç bildirimi temizler ve eski referansı kullanımdan kaldırır, açık engel kaldırma ise son temizleme sınırıdır.

Engellenmiş bir sonuç hâlâ uzlaştırılırken yeni bir tur, önceki talebin serbest bırakılması için en fazla 15 saniye bekler. İşlem hâlâ meşgulse tur, işlem yapılabilir bir “önceki bulut turunun çalışma alanı sonucu hâlâ uzlaştırılıyor” mesajıyla başarısız olur ve kısa süre sonra yeniden denenebilir. Yeniden başlatıldığında kurtarma, eski talepleri temizlemeden önce bekleyen ve aşamalanmış sonuçları bulur, bunların yerel uygulamasını tamamlar veya yeniden dener ve ölü ortamları ancak sonucu koruduktan sonra geri alır. Sınırlı SQLite geri alma günlüğü, kesintiye uğramış bir dosya sistemi uygulamasını önceden kabul edilmiş değişiklikleri yeniden oynatmadan kurtarılabilir hâle getirir.

İş tamamlandığında ve hiçbir tur çalışmadığında oturum menüsünü açıp **Stop cloud worker…** öğesini seçin. Gateway ortamı yok etmeden önce son bir çalışma alanı uzlaştırması gerçekleştirir. Zaten `draining` veya `reconciling` durumunda olan bir yerleşim, kapatma işlemini tamamlıyordur; oturumu silmeden önce rozetinin `reclaimed` olmasını bekleyin.

Bozuk veya kontrolden çıkmış bağlı bir çalışan için operatör, son çare olarak `environments.destroy` öğesini `{ "force": true }` ile çağırabilir. Zorunlu kapatma, ortamı yok etmeden önce yerleşimi kalıcı olarak başarısız olarak işaretler ve uzlaştırılmamış tüm uzak sonuçları terk eder.

Eşdeğer yönetim RPC'si şöyledir:

```bash
openclaw gateway call sessions.reclaim \
  --timeout 600000 \
  --params '{"key":"agent:main:big-refactor"}'
```

Yerleştirme, kalıcı bir durum makinesi (`local → requested → provisioning → syncing → starting → active`) üzerinden ilerler; böylece dağıtımın ortasında Gateway yeniden başlatılırsa makinelerin sızmasına yol açmak yerine uzlaştırma yapılır. Başarısız bir model sırası, etkin yerleştirmeyi yeniden deneme için kullanılabilir durumda tutar. Çalışma alanı yolu çakışmalarında yerel sürüm korunur, bulut sonucunun geri kalanı uygulanır ve inceleme için hazırlanmış bulut ref'i muhafaza edilir; diğer uzlaştırma veya yaşam döngüsü hataları, kurtarma güvenli biçimde yeniden deneyebilene ya da ortamı geri alabilene kadar kalıcı kurtarma çitini ve tanılama sonunu korur.

## Güvenlik modeli

- **Kapalı çalışan girişi.** Çalışanlar, tünellenmiş soket üzerinde kapalı bir yöntem izin listesine sahip özel bir protokol kullanır — bir çalışan, operatör RPC'lerini çağıramaz.
- **Gateway'in sahip olduğu araç yetkisi.** Gateway, her sıradan önce mevcut profil, sağlayıcı, ajan, grup, gönderen, sandbox, yetkilendirme, devralınan politika ve çalışma zamanı sınırı politikasını çalışanın sabit kodlama aracı kataloğuna yansıtır. Başlatma zarfı yalnızca bu nihai, kapalı söz dağarcığı alt kümesini taşır. Açıkça sınırlandırılmış zamanlanmış sıralar, bu kimliği kutuya göndermeden veya yeni bir gönderen katmanını yeniden uygulamadan güvenilir sahip grubu bağlamını yeniden kullanır. Çalışan kataloğunun dışındaki araçlar kullanılamaz durumda kalır; boş bir sonuç hiçbir araç olmadan çalışır.
- **Oluşturulan kimlik bilgileri, beklemede karma hâlinde.** Her dağıtım bir çalışan kimlik bilgisi oluşturur; Gateway yalnızca bunun karmasını saklar. Kimlik bilgisi döndürme ve sahip dönemi çitlemesi, oturum başına en fazla bir etkin sahip bulunmasını garanti eder — yeniden bağlanan eski bir çalışan çitlenir, hiçbir zaman birleştirilmez.
- **Ana makine anahtarı sabitleme.** Sağlayıcı, kutunun SSH ana makine anahtarını sağlama sırasında sunmalıdır; önyükleme katı sabitlemeyle bağlanır ve anahtar olmadan güvenli biçimde başarısız olur.
- **Kutuda kalıcı model, forge veya bulut kimlik bilgisi bulunmaz.** Model kimlik doğrulaması Gateway'de kalır (çıkarım, `{provider, model}` referansıyla aktarılır), çalışma alanı git commit'leri forge kimlik bilgileri olmadan oluşturulur ve Crabbox AWS kiralama meta verileri, kurulumdan önce bir örnek rolü açısından yetkili biçimde denetlenir. Kurulum komutlarını da kimlik bilgilerinden arındırılmış tutun.
- **Sağlayıcının sahip olduğu çıkış trafiği.** Ters tünel, OpenClaw'un doğrudan model erişimi gereksinimini ortadan kaldırır; ancak OpenClaw sağlayıcı güvenlik duvarlarını yeniden yazmaz. Görev gerektirdiğinde çalışan sağlayıcıdaki giden trafiği kısıtlayın.
- **Kalıcı, tam olarak bir kez kaydedilen transkriptler.** Çalışan, transkript gruplarını oturumun yaprağına karşı karşılaştır ve değiştir protokolüyle commit eder; eski bir taban, ücretli çıktıyı çoğaltmak veya yeniden temellendirmek yerine çalışmayı güvenli biçimde durdurur.

## Sorun giderme

- **`sessions.dispatch` bilinmeyen bir yöntem** — hiçbir `cloudWorkers.profiles` yapılandırılmamış veya çağıranın `operator.admin` yetkisi yok.
- **"Bulut çalışanı sıraları OpenClaw çalışma zamanını gerektirir"** — yapılandırılmış çalışma zamanı OpenClaw olan bir model seçin. `claude-cli` gibi harici CLI çalışma zamanları çalışan çıkarımını desteklemez.
- **"Çalışan önyüklemesi, kiralanan ana makinede Node.js gerektirir"** — `settings.setup` öğesine bir Node kurulumu ekleyin (yukarıya bakın).
- **AWS örnek rolü tasdiki başarısız oluyor** — `aws.instanceProfile` değerini (ayarlanmışsa `CRABBOX_AWS_INSTANCE_PROFILE` değerini de) temizleyin. Crabbox 0.38.1 veya daha yeni bir sürümünü yükleyin; eski ikili dosyalar AWS kabulü için gereken yetkili `providerMetadata.instanceProfileAttached` sözleşmesini sunmaz.
- **Dağıtım bir sağlayıcı hatasıyla başarısız oluyor** — yerleştirme kaydı ve `environments.list`, kurulum/önyükleme stderr sonu dâhil son hatayı saklar. Kutular hata durumunda yok edilir; bu nedenle bu son bölüm birincil adli inceleme kaynağıdır.
- **Dağıtım sırasında istemci zaman aşımı** — `openclaw gateway call` varsayılan olarak 10s zaman aşımı kullanır; `--timeout` değerini yeterince geniş verin (dağıtım her iki durumda da sunucu tarafında çalışmaya devam eder ve sağlama sırasında yapılan yeniden deneme `session cannot dispatch from placement provisioning` ile reddedilir).
- **2026.7.2 beta sürümünden yükseltme sonrasında çalışan geri alındı** — bu beta sürümleri eski çalışan başlatma sözleşmesini kullanıyordu. OpenClaw yeniden başlatıldığında boşta olan uyumsuz çalışanı yok eder, oturumu ve çalışma alanını korur, yerleştirmeyi geri alınmış olarak işaretler ve sonraki dağıtımda veya sırada güncel bir çalışan sağlar. Hâlâ başlatılırken kesintiye uğrayan bir beta çalışanı, temizlikten sonra başarısız olarak işaretlenir; güncel sözleşmeyle sağlamak için dağıtımı yeniden deneyin.
- **Bulut çalışma alanı çakışma bildirimi** — sıra tamamlandı ve listelenen her yolun yerel sürümünü korudu. Bulut sürümünü incelemek veya almak için bildirimdeki hazırlanmış ref komutlarını kullanın; zaten uygulanmış olan çakışmasız değişiklikler için yeniden deneme gerekmez.
- **“Önceki bulut sırasının çalışma alanı sonucu hâlâ uzlaştırılıyor”** — Gateway, önceki sonucun kalıcı çiti için kısa süre bekledi ve oturum talebini alamadı. Uzlaştırmanın tamamlanmasını bekleyin, ardından sırayı yeniden deneyin; Gateway'i yeniden başlatmak güvenlidir çünkü kurtarma, çalışmayan bir çalışanı geri almadan önce hazırlanmış sonuçları korur.
- **Kiralama bakımı** — `crabbox list --provider <backend>` etkin kiralamaları gösterir; `crabbox stop --provider <backend> --id <lease>` birini manuel olarak serbest bırakır. Boşta olan kiralamaların süresi, profilin `idleTimeout` değerine göre dolar.

## İlgili

- [Sandbox kullanımı](/tr/gateway/sandboxing) — yerel araç yürütmenin etki alanını azaltma
- [Oturumlar CLI'si](/tr/cli/sessions) — saklanan oturumları inceleme
- [Yapılandırma referansı](/tr/gateway/configuration-reference)
