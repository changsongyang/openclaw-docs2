---
read_when:
    - ACP oturumu yaşam döngüsünü veya ACPX işlem temizliğini yeniden düzenleme
    - ACPX sahipsiz süreçlerinde, PID yeniden kullanımında veya çoklu Gateway temizleme güvenliğinde hata ayıklama
    - Oluşturulan ACP veya alt ajan oturumları için sessions_list görünürlüğünü değiştirme
    - Arka plan görevleri, ACP oturumları veya işlem kiralamaları için sahiplik meta verilerini tasarlama
sidebarTitle: ACP lifecycle refactor
summary: ACP oturumu ve ACPX süreç sahipliğini açık hâle getirmeye yönelik geçiş planı
title: ACP yaşam döngüsü yeniden düzenlemesi
x-i18n:
    generated_at: "2026-07-27T00:16:15Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: bda66f0acc93216c3d9386ca3ebf7f544efd306cd7f53386391f0c48e5dc8f06
    source_path: refactor/acp.md
    workflow: 16
---

ACP yaşam döngüsü şu anda çalışıyor, ancak bunun çok büyük bir bölümü sonradan çıkarıma dayanıyor.
Süreç temizleme, sahipliği PID'lerden, komut dizelerinden, sarmalayıcı
yollarından ve canlı süreç tablosundan yeniden oluşturuyor. Oturum görünürlüğü ise sahipliği
oturum anahtarı dizelerinden ve ikincil `sessions.list({ spawnedBy })` aramalarından yeniden oluşturuyor.
Bu, dar kapsamlı düzeltmeleri mümkün kılıyor ancak uç durumların gözden kaçmasını da kolaylaştırıyor:
PID'nin yeniden kullanılması, tırnak içine alınmış komutlar, bağdaştırıcının alt süreçlerinin alt süreçleri, birden çok Gateway durum kökü,
`cancel` ile `close` ve `tree` ile `all` görünürlüğü, aynı sahiplik kurallarının
yeniden keşfedildiği ayrı yerler hâline geliyor.

Bu yeniden düzenleme, sahipliği birinci sınıf hâle getiriyor. Amaç yeni bir ACP ürün
yüzeyi değil; mevcut ACP ve ACPX davranışı için daha güvenli bir iç sözleşmedir.

## Hedefler

- Temizleme, mevcut canlı kanıt OpenClaw'a ait bir kiralamayla eşleşmedikçe hiçbir sürece sinyal göndermez.
- `cancel`, `close` ve başlangıçta artık süreçleri temizleme farklı yaşam döngüsü amaçlarına sahiptir.
- `sessions_list`, `sessions_history`, `sessions_send` ve durum kontrolleri
  istek sahibine ait aynı oturum modelini kullanır.
- Birden çok Gateway içeren kurulumlar birbirlerinin ACPX sarmalayıcılarını temizleyemez.
- Eski ACPX oturum kayıtları geçiş sırasında çalışmaya devam eder.
- Çalışma zamanı Plugin'e ait kalır; çekirdek ACPX paket ayrıntılarını öğrenmez.

## Hedef Dışı Konular

- ACPX'i değiştirmek veya genel `/acp` komut yüzeyini değiştirmek.
- Satıcıya özgü ACP bağdaştırıcı davranışını çekirdeğe taşımak.
- Kullanıcıların yükseltmeden önce durumu elle temizlemesini zorunlu kılmak.
- `cancel` işleminin yeniden kullanılabilir ACP oturumlarını kapatmasını sağlamak.

## Hedef Model

### Gateway Örneği Kimliği

Her Gateway süreci kararlı bir çalışma zamanı örnek kimliğine sahip olmalıdır:

```ts
type GatewayInstanceId = string;
```

Bu kimlik Gateway başlatılırken oluşturulabilir ve ilgili kurulumun ömrü boyunca
durumda kalıcı olarak saklanabilir. Bir güvenlik sırrı değildir; bir Gateway'in ACP süreçlerinin
başka bir Gateway'in süreçleriyle karıştırılmasını önlemek için kullanılan bir sahiplik ayırıcısıdır.

### ACP Oturum Sahipliği

Başlatılan her ACP oturumu normalleştirilmiş sahiplik meta verilerine sahip olmalıdır:

```ts
type AcpSessionOwner = {
  sessionKey: string;
  spawnedBy?: string;
  parentSessionKey?: string;
  ownerSessionKey: string;
  agentId: string;
  backend: "acpx";
  gatewayInstanceId: GatewayInstanceId;
  createdAt: number;
};
```

Gateway, bilindikleri oturum satırlarında bu alanları döndürmelidir.
Görünürlük filtrelemesi, satır meta verileri üzerinde yapılan saf bir kontrol olmalıdır:

```ts
canSeeSessionRow({
  row,
  requesterSessionKey,
  visibility,
  a2aPolicy,
});
```

Bu, görünürlük kontrollerindeki gizli ikincil `sessions.list({ spawnedBy })` çağrılarını
ortadan kaldırır. Başlatılmış, aracılar arası bir ACP alt oturumu istek sahibine aittir; çünkü
ikinci bir sorgu tesadüfen onu bulduğu için değil, satır böyle belirttiği için.

### ACPX Süreç Kiralamaları

Oluşturulan her sarmalayıcı başlatması bir kiralama kaydı oluşturmalıdır:

```ts
type AcpxProcessLease = {
  leaseId: string;
  gatewayInstanceId: GatewayInstanceId;
  sessionKey: string;
  wrapperRoot: string;
  wrapperPath: string;
  rootPid: number;
  processGroupId?: number;
  commandHash: string;
  startedAt: number;
  state: "open" | "closing" | "closed" | "lost";
};
```

Sarmalayıcı süreç, kiralama kimliğini ve Gateway örnek kimliğini taşınabilir
bağımsız değişkenler olarak alır:

```sh
--openclaw-acpx-lease-id ... --openclaw-gateway-instance-id ...
```

Platform izin verdiğinde doğrulama, komutların tırnak içine alınmasıyla
karıştırılamayacak canlı süreç meta verilerini tercih etmelidir:

- kök PID hâlâ mevcut
- canlı sarmalayıcı yolu `wrapperRoot` altında
- mevcut olduğunda süreç grubu kiralamayla eşleşiyor
- bağımsız değişkenler beklenen kiralama kimliğini içeriyor
- komut karması veya yürütülebilir dosya yolu kiralamayla eşleşiyor

Canlı süreç doğrulanamazsa temizleme güvenli biçimde başarısız olur.

## Yaşam Döngüsü Denetleyicisi

Süreç kiralamalarının ve temizleme politikasının sahibi olan tek bir ACPX yaşam döngüsü
denetleyicisi kullanıma sunun:

```ts
interface AcpxLifecycleController {
  ensureSession(input: AcpRuntimeEnsureInput): Promise<AcpRuntimeHandle>;
  cancelTurn(handle: AcpRuntimeHandle): Promise<void>;
  closeSession(input: {
    handle: AcpRuntimeHandle;
    discardPersistentState?: boolean;
    reason?: string;
  }): Promise<void>;
  reapStartupOrphans(): Promise<void>;
  verifyOwnedTree(lease: AcpxProcessLease): Promise<OwnedProcessTree | null>;
}
```

`cancelTurn` yalnızca tur iptali ister. Yeniden kullanılabilir sarmalayıcı
veya bağdaştırıcı süreçlerini temizlememelidir.

`closeSession` temizleme yapabilir, ancak yalnızca oturum kaydını yükledikten,
kiralamayı yükledikten ve canlı süreç ağacının hâlâ bu kiralamaya ait olduğunu doğruladıktan sonra.

`reapStartupOrphans`, durumdaki açık kiralamalardan başlar. Alt süreçleri bulmak için süreç
tablosunu kullanabilir ancak önce ACP'ye benzer rastgele komutları tarayıp ardından
bunların muhtemelen bize ait olduğuna karar vermemelidir.

## Sarmalayıcı Sözleşmesi

Oluşturulan sarmalayıcılar küçük kalmalıdır. Şunları yapmalıdır:

- desteklendiğinde bağdaştırıcıyı bir süreç grubunda başlatmak
- normal sonlandırma sinyallerini süreç grubuna iletmek
- üst sürecin sonlanmasını algılamak
- üst süreç sonlandığında SIGTERM göndermek, ardından SIGKILL
  geri dönüşü çalışana kadar sarmalayıcıyı canlı tutmak
- mümkün olduğunda kök PID'yi ve süreç grubu kimliğini yaşam döngüsü denetleyicisine
  bildirmek

Sarmalayıcılar oturum politikasına karar vermemelidir. Yalnızca kendi bağdaştırıcı
grupları için yerel süreç ağacı temizliğini uygularlar.

## Oturum Görünürlüğü Sözleşmesi

Görünürlük, normalleştirilmiş satır sahipliğini kullanmalıdır:

```ts
type SessionVisibilityInput = {
  requesterSessionKey: string;
  row: {
    key: string;
    agentId: string;
    ownerSessionKey?: string;
    spawnedBy?: string;
    parentSessionKey?: string;
  };
  visibility: "self" | "tree" | "agent" | "all";
  a2aPolicy: AgentToAgentPolicy;
};
```

Kurallar:

- `self`: yalnızca istek sahibi oturum.
- `tree`: istek sahibi oturum ile istek sahibine ait veya ondan başlatılmış satırlar.
- `all`: aynı aracıya ait tüm satırlar, a2a tarafından izin verilen aracılar arası satırlar ve genel a2a
  devre dışı olsa bile istek sahibine ait başlatılmış aracılar arası satırlar.
- `agent`: açık bir sahip ilişkisi satırın istek sahibine ait olduğunu belirtmedikçe
  yalnızca aynı aracı.

Bu, `tree` ile `all` davranışını monoton hâle getirir: `all`, `tree` tarafından gösterilecek
sahip olunan bir alt oturumu gizlememelidir.

## Geçiş Planı

### 1. Aşama: Kimlik ve Kiralamalar Ekleme

- Gateway durumuna `gatewayInstanceId` ekleyin.
- ACPX durum dizini altında bir ACPX kiralama deposu ekleyin.
- Oluşturulan bir sarmalayıcıyı başlatmadan önce kiralama yazın.
- Yeni ACPX oturum kayıtlarında `leaseId` değerini saklayın.
- Eski kayıtlar için mevcut PID ve komut alanlarını koruyun.

### 2. Aşama: Önce Kiralamaya Dayalı Temizleme

- Kapatma temizliğini önce `leaseId` yükleyecek şekilde değiştirin.
- Sinyal göndermeden önce canlı süreç sahipliğini kiralamaya göre doğrulayın.
- Mevcut kök PID ve sarmalayıcı kökü geri dönüşünü yalnızca eski kayıtlar için koruyun.
- Doğrulanmış temizlemeden sonra kiralamaları `closed` olarak işaretleyin.
- Süreç temizlemeden önce sona ermişse kiralamaları `lost` olarak işaretleyin.

### 3. Aşama: Önce Kiralamaya Dayalı Başlangıç Temizliği

- Başlangıç temizliği açık kiralamaları tarar.
- Her kiralama için kök süreci doğrulayın ve alt süreçleri toplayın.
- Doğrulanmış ağaçları alt süreçlerden başlayarak temizleyin.
- Eski `closed` ve `lost` kiralamalarını sınırlı bir saklama aralığıyla sona erdirin.
- Komut işareti taramasını yalnızca geçici bir eski sistem geri dönüşü olarak koruyun; mümkün olduğunda
  sarmalayıcı kökü ve Gateway örneğiyle sınırlandırın.

### 4. Aşama: Oturum Sahipliği Satırları

- Gateway oturum satırlarına sahiplik meta verileri ekleyin.
- ACPX, alt aracı, arka plan görevi ve oturum deposu yazıcılarına
  `ownerSessionKey` veya `spawnedBy` alanlarını doldurmayı öğretin.
- Oturum görünürlüğü kontrollerini satır meta verilerini kullanacak şekilde dönüştürün.
- Görünürlük sırasında yapılan ikincil `sessions.list({ spawnedBy })` aramalarını kaldırın.

### 5. Aşama: Eski Sezgisel Yöntemleri Kaldırma

Bir sürüm aralığından sonra:

- eski olmayan ACPX temizliğinde saklanan kök komut dizelerine güvenmeyi bırakın
- başlangıçtaki komut işareti taramalarını kaldırın
- görünürlük geri dönüşü liste aramalarını kaldırın
- eksik veya doğrulanamayan kiralamalar için güvenli biçimde başarısız olan koruyucu davranışı sürdürün

## Testler

İki tablo güdümlü test paketi ekleyin.

Süreç yaşam döngüsü simülatörü:

- PID'nin ilgisiz bir süreç tarafından yeniden kullanılması
- PID'nin başka bir Gateway'in sarmalayıcı kökü tarafından yeniden kullanılması
- saklanan sarmalayıcı komutu kabuk tarafından tırnak içine alınmışken canlı `ps` komutunun alınmamış olması
- bağdaştırıcı alt sürecinin sona ermesi ancak onun alt sürecinin süreç grubunda kalması
- üst süreç sonlandığında SIGTERM geri dönüşünün SIGKILL aşamasına ulaşması
- süreç listesinin kullanılamaması
- süreci eksik, bayat kiralama
- sarmalayıcı, bağdaştırıcı alt süreci ve onun alt sürecini içeren başlangıç artığı

Oturum görünürlüğü matrisi:

- `self`, `tree`, `agent`, `all`
- a2a etkin ve devre dışı
- aynı aracıya ait satır
- aracılar arası satır
- istek sahibine ait, başlatılmış aracılar arası ACP satırı
- `tree` ile sınırlandırılmış korumalı alan istek sahibi
- listeleme, geçmiş, gönderme ve durum eylemleri

Önemli değişmez: istek sahibine ait başlatılmış bir alt oturum, yapılandırılmış
görünürlüğün istek sahibi oturum ağacını kapsadığı her yerde görünürdür ve `all`,
`tree` değerinden daha kısıtlı değildir.

## Uyumluluk Notları

Eski oturum kayıtlarında `leaseId` bulunmayabilir. Bunlar eski,
güvenli biçimde başarısız olan temizleme yolunu kullanmalıdır:

- canlı bir kök süreç gerektir
- oluşturulmuş bir sarmalayıcı beklendiğinde sarmalayıcı kökü sahipliğini gerektir
- sarmalayıcı olmayan kökler için komut uyuşmasını gerektir
- yalnızca bayat saklanmış PID meta verilerine dayanarak asla sinyal gönderme

Eski bir kayıt doğrulanamıyorsa ona dokunmayın. Başlangıç kiralama temizliği ve
sonraki sürüm aralığı, geri dönüşü zaman içinde kullanımdan kaldırmalıdır.

## Başarı Ölçütleri

- Eski veya bayat bir ACPX oturumunu kapatmak başka bir Gateway'in sürecini sonlandıramaz.
- Üst sürecin sonlanması, inatçı bağdaştırıcı alt süreçlerinin alt süreçlerini çalışır durumda bırakmaz.
- `cancel`, yeniden kullanılabilir oturumları kapatmadan etkin turu iptal eder.
- `sessions_list`, istek sahibine ait aracılar arası ACP alt oturumlarını hem
  `tree` hem de `all` altında gösterebilir.
- Başlangıç temizliği geniş kapsamlı komut dizesi taramalarıyla değil, kiralamalarla yürütülür.
- Odaklanmış süreç ve görünürlük matrisi testleri, daha önce tek seferlik inceleme
  düzeltmeleri gerektiren tüm uç durumları kapsar.
