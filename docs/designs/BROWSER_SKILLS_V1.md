# Tarayıcı-Becerileri v1 — tekrarlanan tarayıcı akışlarının kodlanması

**Durum:** 1. Aşama `garrytan/browserharness` üzerinde yayınlandı. 2-4. Aşamalar aşağıda listelenmiştir.
**Son güncelleme:** 2026-04-26
**Yazarlar:** garrytan (/plan-eng-review ve /codex dış-ses incelemesi ile)

## Bu nedir?

Tarayıcı-becerileri, tekrarlanan bir tarayıcı akışını deterministik bir Playwright
betiğine kodlayan görev-bazlı dizinlerdir. Her becerinin yapısı:

```
browser-skills/<isim>/
├── SKILL.md                        # ön-madde + düzyazı sözleşme
├── script.ts                       # deterministik mantık
├── _lib/browse-client.ts           # SDK'nın vendored kopyası
├── fixtures/<host>-<tarih>.html     # testler için yakalanan sayfa
└── script.test.ts                  # fixture'e karşı parser testleri
```

Bir kullanıcı (veya 2. Aşamada, akışı yeni başarıyla gerçekleştirmiş bir ajan) beceriyi
bir kez oluşturur. Sonraki çağrılar betiği çalıştırır ve ajnın `$B` temelleriyle
yeniden keşfederek harcayacağı 30 saniye yerine 200ms'de JSON döndürür.

Yayınlanan referans `hackernews-frontpage`'dir: HN ana sayfasını kazır, 30 haberi
JSON olarak döndürür. `$B skill list` ve `$B skill run hackernews-frontpage` deneyin.

## Bu, domain-skills'ten (v1.8.0.0) neden farklı?

- **Domain-skills** = "ajan bir site hakkında bilgiler hatırlar." Ana bilgisayar adına göre
  anahtarlanan, oturum başlangıcında istemlere enjekte edilen JSONL notları. Karantina →
  aktif → genel tanıtımı işleyen durum makinesi.
- **Browser-skills** = "ajan prosedürleri deterministik betilere kodlar." Görev-bazlı dizinler,
  `$B skill run` ile yürütülür, daemon'da spawn-bazlı yetenek izolasyonu için
  kapsamlı belirteçler.

İkisi de aynı zihinsel modeli kullanır (ana bilgisayar-bazlı, üç katmanlı kapsama).
Prosedür katmanı, daha büyük verimlilik kazancının yaşandığı yerdir çünkü kazıma ve form
otomasyonunu gizil uzaydan yeniden üretilebilir koda iter.

## Bu neden mevcut P1 ("kendi-yazan `$B` komutları") değil?

Orijinal P1, Codex'in T1 itirazı nedeniyle engellendi: ajan-yazılmış TypeScript
daemon *içinde* güvenli çalıştırılamaz (ortam genel değişkenleri, kurucu araçları,
onay ve yürütme arasında üst-düzey-await TOCTOU). Doğru tasarım, "süreç-dışı işçi
izolasyonu ile yetenek-geçişli IPC" idi. Bu, hiç yayınlanmayabilecek zor bir projeydi.

Tarayıcı-becerileri, betikleri daemon *dışında* bağımsız Bun süreçleri olarak
çalıştırarak tüm sorunu aşar. Daemon asla beceri kodunu içe aktarmaz veya değerlendirmez.
Beceriler, daemon ile geri döngü HTTP üzerinden haberleşir — herhangi bir harici
istemcinin kullanacağı aynı tel formatı.

Onaylanan plan, mevcut P1'in yerine geçer.

---

## Aşamalar

| Aşama | Dal | Kapsam |
|-------|-----|--------|
| **1** | `garrytan/browserharness` | SDK, depolama, `$B skill list/run/show/test/rm` alt komutları, kapsamlı-belirteç modeli, paketlenmiş `hackernews-frontpage` referansı. **Yayınlandı (v1.19.0.0, 2a. Aşama ile birleştirildi).** |
| **2a** | `garrytan/browserharness` (devam) | `/scrape <intent>` (salt-okunur, eşleşme/prototip yolları ile tek giriş noktası) + `/skillify` (prototipi kalıcı beceriye kodlar). `browse/src/browser-skill-write.ts` D3 atomik-yazma yardımcısını ekler. **v1.19.0.0 yayını.** |
| **2b** | yeni (`browser-skills-automate`) | `/automate` beceri şablonu (mutasyona uğrayan-akış kardeşi `/scrape`). `/skillify` ve D3 yardımcısını yeniden kullanır. Kodlanmamış çalıştırıldığında mutasyona uğrayan-adım-bazlı onay kapısı. TODOS içinde P0. |
| **3** | yeni (`browser-skills-resolver`) | Oturum başlangıcında resolver enjeksiyonu (ana bilgisayar-bazlı tarayıcı-beceri keşfi). Domain-skill enjeksiyonunu yansıtır. `gstack-config browser_skillify_prompts` düğmesi. |
| **4** | yeni | Değerlendirme test altyapısı (LLM-hakemi), fixture-eskilik algılama, canlı sayfalara karşı periyodik yeniden doğrulama, güvenilmeyen spawn'lar için işletim sistemi düzeyinde FS korumalı alanı. |

---

## 1. Aşama mimarisi

### Kilitlenmiş kararlar (13)

1. **1. Aşama = tam depolama + SDK + alt komutlar + paketlenmiş referans.** Henüz ajan
   yazımı yok. 2. Aşama `/scrape` ve `/automate`'i yayınlar.
2. **2. Aşamada iki fiil: `/scrape` (salt-okunur) ve `/automate` (mutasyona uğrayan).**
   Becerileştirme onay-kapısı makinesini paylaşırlar ama ayrı beceri şablonları olarak
   yaşarlar.
3. **Mevcut kendi-yazan-`$B` P1'i TODOS.md'dekinin yerine geçer.** Aynı
   kullanıcı-görünür hedef, daemon-içi izolasyon sorunu yok.
4. **SDK dağıtımı: her beceri içinde kardeş dosya (Seçenek E).** Kanonik SDK
   `browse/src/browse-client.ts` konumundadır (~250 SATIR). Her beceri
   `<skill>/_lib/browse-client.ts` konumunda bir kopya gönderir. 2. Aşamanın
   üreteci, oluşturulan her betiğin yanına güncel SDK'yı kopyalar. Her beceri
   tamamen kendi-yeterlidir: dizini herhangi bir yere kopyalayın, çalışır. Sürüm
   kayması imkansızdır (SDK, becerinin yazıldığı sürümde donmuştur). Disk maliyeti:
   beceri başına ~3KB.
5. **Üç katmanlı arama: paketlenmiş → genel → proje.** Paketlenmiş beceriler gstack
   kurulumuyla salt-okunur olarak gelir (`<gstack-install>/browser-skills/<isim>/`).
   Genel: `~/.gstack/browser-skills/<isim>/`. Proje-bazlı:
   `<project>/.gstack/browser-skills/<isim>/`. Arama, katmanları proje → genel →
   paketlenmiş öncelik sırasına göre yürür; ilk eşleşme kazanır. **`$B skill list`
   her beceri adının yanında çözümlenen katmanı yazdırır**, böylece "neden o çalıştı?"
   hiçbir zaman bir hata ayıklama gizemi değildir.
6. **Güven modeli: spawn zamanında kapsamlı belirteçler, ortam-temizleme-korumalı-alan DEĞİL.**
   Aşağıdaki "Güven modeli"ne bakın. (Codex bunu güvenlik tiyatrosu olarak işaretledikten
   sonra orijinal ortam-temizleme planından revize edildi.)
7. **Tek doğruluk kaynağı: yalnızca SKILL.md ön-maddesi.** `meta.json` yok.
   Ön-madde ana bilgisayar, tetikleyiciler, argümanlar, sürüm, kaynak, güvenilir bilgilerini
   tutar. SHA256/eskilik, ayrı bir `.checksum` yan dosyası olarak 4. Aşamaya ertelenir
   (eğer hiç gelirse).
8. **INDEX.json yok. Dizini yürüyün.** `$B skill list` üç katmanı numaralandırır
   ve her SKILL.md ön-maddesini ayrıştırır. 50 beceri için ~5-10ms.
   "Dizin diskten ayrıldı" hata sınıfının tamamını ortadan kaldırır.
9. **`$B skill run` çıktı protokolü.** stdout = JSON. stderr = akışlı
   günlükler. Çıkış 0 / sıfır olmayan. Varsayılan 60sn zaman aşımı, `--timeout=Ns` ile
   geçersiz kılınır. Maks stdout 1MB (aşıldığında kesme + sıfır olmayan çıkış). `gh` /
   `kubectl` / `docker` kurallarıyla eşleşir.
10. **Fixture yeniden oynatma: iki test türü için iki desen.** SDK birim testi
    test-içi bir sahte HTTP sunucusu kurar. Uçtan-uca beceri testleri, paketlenmiş
    HTML fixture'larını betiğin dışa aktarılan parser işlevi aracılığıyla ayrıştırır
    (daemon gerekmez). 1. Aşama fixture-only, `hackernews-frontpage` için yeterlidir;
    2. Aşama `/automate` daha zengin fixture'lara ihtiyaç duyar.
11. **Referans beceri: `hackernews-frontpage`.** HN ana sayfasını kazır
    (başlıklar, puanlar, yorumlar). Kimlik doğrulama yok, kararlı HTML, ideal fixture-test
    hedefi.
12. **Belirteç/bağlantı noktası keşfi: spawn edilen beceriler için kapsamlı-belirteç salt-ortam;
    bağımsız hata ayıklama çalıştırmaları için durum-dosyası geri dönüşü.** `$B skill run`
    aracılığıyla spawn edildiğinde, SDK ortamdan `GSTACK_PORT` + `GSTACK_SKILL_TOKEN`
    okur. Bağımsız `bun run script.ts` için SDK, `<project>/.gstack/browse.json`
    konumuna geri döner (`config.ts:50`'deki gerçek durum-dosyası yolu).
13. **CHANGELOG dürüstlüğü.** 1. Aşama lideri: insanlar gstack'in çalıştırdığı
    deterministik tarayıcı betiklerini el ile yazabilir. 1. Aşama, ajan yazımının
    bir sonraki sürümde yayınlandığını açıkça belirtir. Uydurulmuş performans
    numaraları yok — 1. Aşamada öncesi/sonrası yok.

### Güven modeli (detayda karar #6)

İki dikey eksen:

| Eksen | Mekanizma | Varsayılan |
|-------|-----------|------------|
| **Daemon-tarafı yetenek** | Spawn-bazlı kapsamlı belirteç, `read+write` kapsamına bağlı (17 komutluk tarayıcı-yönetme yüzeyi, `eval`/`js`/`cookies`/`storage` gibi yönetici komutları hariç). Tek-kullanımlık clientId, beceri adı + spawn kimliğini kodlar. Spawn çıktığında iptal edilir. | Her zaman kapsamlı (asla daemon kök belirteci değil). |
| **Süreç-tarafı ortam erişimi** | SKILL.md ön-maddesi `trusted: true` olduğu zaman `GSTACK_TOKEN` hariç `process.env`'i geçirir. `trusted: false` (varsayılan) her şeyi en küçük izin verilenler listesi dışında bırakır (LANG, LC_ALL, TERM, TZ, kilitli PATH) ve açıkça gizli-desen anahtarlarını soyar (TOKEN/KEY/SECRET/PASSWORD, AWS_*, AZURE_*, GCP_*, ANTHROPIC_*, OPENAI_*, GITHUB_*, vb.). | Güvenilmeyen (katılım gerekli). |

`GSTACK_PORT` ve `GSTACK_SKILL_TOKEN` her zaman en son enjekte edilir, böylece
bir üst süreç bunları ortamda ayarlayarak geçersiz kılamaz.

**Bu neyi doğru getiriyor:** daemon-tarafı kapsamlı belirteç, daemon tarafından
zorunlu kılınabilir. `eval` (yönetici kapsamı) çağırmaya çalışan bir beceri, SDK
ortaya çıkarsa bile 403 alır. Yetenek sınırı doğru yerdedir.

**Bu neyi KAPAMAZ:** Bun'un yerleşik FS korumalı alanı yok. Güvenilmeyen bir
beceri hala `import 'fs'` yapabilir ve işletim sistemi kullanıcısının okuyabileceği
her şeyi okuyabilir (örn. `~/.ssh/id_rsa`). Ortam temizliği hijyen, korumalı alan değil.
İşletim sistemi düzeyinde izolasyon (`sandbox-exec`, ad alanları) 4. Aşama çalışmasıdır
ve mevcut güvenilir/güvenilmeyen sözleşmesinin arkasına temiz bir şekilde düşer.

Orijinal plan, ortam temizliğine korumalı alan diyordu. Codex bunu haklı olarak
tiyatro olarak işaretledi. Revize edilmiş plan, olduğu gibi adlandırır: en iyi çaba
hijyeni artı savunma-derinliğinde, gerçek sınır daemon-tarafı kapsamlı belirteçtedir.

### Dosya düzeni

```
browse/src/
├── browse-client.ts                # kanonik SDK (~250 SATIR)
├── browser-skills.ts               # 3-katmanlı yürüyüş + ön-madde parser + mezar taşları
├── browser-skill-commands.ts       # $B skill list/show/run/test/rm + spawnSkill
└── skill-token.ts                  # mintSkillToken / revokeSkillToken sarmalayıcıları

browser-skills/
└── hackernews-frontpage/           # paketlenmiş referans beceri
    ├── SKILL.md
    ├── script.ts
    ├── _lib/browse-client.ts        # kanonik kopyanın bayt-özdeş kopyası
    ├── fixtures/hn-2026-04-26.html
    └── script.test.ts

browse/test/
├── skill-token.test.ts              # mint/revoke yaşam döngüsü, kapsam doğrulamaları
├── browse-client.test.ts            # sahte HTTP sunucusu, tel formatı, kimlik doğrulama
├── browser-skills-storage.test.ts   # 3-katmanlı yürüyüş, ön-madde, mezar taşları
└── browser-skill-commands.test.ts   # parseRunArgs, dispatch, ortam temizliği, spawn

test/skill-validation.test.ts       # genişletilmiş: paketlenmiş-beceri sözleşme kontrolleri
```

### Ne değişmez

- Domain-skills depolama, durum makinesi veya enjeksiyon. Dokunulmamış.
- Tünel-yüzey izin verilenler listesi (`server.ts:118-123`). Aynı 17 komut.
- L1-L6 güvenlik yığını. Tarayıcı-becerileri 1. Aşamada istemlere metin enjekte etmez;
  3. Aşamanın resolver enjeksiyonu mevcut UNTRUSTED zarfına biner.
- `cli.ts`'deki `sendCommand()` HTTP istemcisi. SDK, farklı bir kaygıya sahip
  ayrı bir modüldür (kitaplık vs CLI süreci).

---

## Codex dış-ses bulguları (inceleme-sonrası yanıtlar)

/codex incelemesi 8 bulgu işaretledi. Plan bunları şu şekilde ele alır:

| # | Bulgu | 1. Aşama yanıtı |
|---|---------|------------------|
| 1 | Güven modeli FS korumalı alanı olmadan sahte | Yukarıdaki karar #6 (kapsamlı belirteçler) ile **Kapatıldı**. |
| 2 | 1. Aşama bir paketlenmiş beceri için aşırı tasarlanmış (arama katmanları, mezar taşları, vb.) | **Kabul edildi ama tutuldu.** Kullanıcı, 2. Aşama ajan yazımını gelmeden önce mimariyi kilitlemek için tam 1. Aşamayı seçti. Her alt sistem, veriler daha sonra kullanılmaz derse temiz kaldırmaya yetecek kadar küçük. |
| 3 | `cli.ts:398`'deki mevcut istemci deseni kardeş SDK'yı gereksiz yapabilir | **Yanlış olduğu doğrulandı.** 398. satır `extractTabId()`'in sonu (bir bayrak-parser). Gerçek HTTP istemcisi `sendCommand()`, cli.ts:401-467 konumundadır, ancak CLI'ye bağlıdır (`process.stdout.write`, `process.exit`, sunucu-yeniden başlatma kurtarması). Kitaplık olarak yeniden kullanılabilir değil. Yeni `browse-client.ts` tel formatını yansıtır ama kitaplık şeklindedir. |
| 4 | "İlk eşleşme kazanır" araması opak | `$B skill list` ve `$B skill show` içinde çözümlenen katmanın satır içi listelenmesiyle **Azaltıldı**. Gelecek: katman geçersiz kılma kafa karıştırıcıysa isteğe bağlı `--source bundled\|global\|project` bayrağı. |
| 5 | Atomik beceri paketleme indeks sorusundan daha önemli; sembolik bağ savunmaları | **1. Aşama için kapatıldı**: paketlenmiş beceriler gstack kurulumunun bir parçası olarak gelir (canlı yazma yok; kurulum dizinindeki salt-okunur dosyalar olmaları nedeniyle atomik). 2. Aşamanın `writeBrowserSkill` önce geçici bir dizine yazar, ardından yeniden adlandırır ve mevcut `browse/src/path-security.ts`'ten `realpath`/`lstat` disiplinini kullanır. |
| 6 | 2. Aşama etkinlik akışından sentez zayıf (kayıplı dairesel tampon) | **2. Aşama tasarımı için açık konu.** Etkinlik akışı telemetridir, yeniden oynatma IR'si değil. 2. Aşama, yapılandırılmış bir kaydedici VEYA ajanın kendi bağlamından betiği sıfırdan yazması için yeniden istemlenmesi gerektirecek. 2. Aşamanın tasarım geçişinde karar verin. |
| 7 | Bun çalışma zamanı gerilemesi: bağımsız Bun süreçleri olarak beceri betikleri Bir çalışma zamanı gereksinimini yeniden tanıtıyor | **2. Aşama dağıtımı için açık konu.** 1. Aşama bunu aşar çünkü paketlenmiş referans beceri gstack kurulumu içinde gelir (zaten Bun ile oluşturuluyor). 2. Aşamanın (a) her üretilen beceri ile Bir ikili gönderme, (b) becerileri kendi-yeterli çalıştırılabilir dosyalara derleme veya (c) `cli.ts`'nin HTTP deseni ile Node.js kullanma arasında karar vermesi gerekir. |
| 8 | `file://` fixture'ları zamanlama/kimlik doğrulama/gezinti/tembel hidrasyon kanıtlamaz | **Belgelenmiş sınır.** `hackernews-frontpage` için yeterli. 2. Aşama `/automate` daha zengin fixture'lara ihtiyaç duyacak (zamanlama ile sahte daemon, kaydedilmiş HAR yeniden oynatma, vb.). |

---

## 2a. Aşama — `/scrape` + `/skillify` (v1.19.0.0 yayını)

İki beceri şablonu artı bir yardımcı modül. `/scrape <intent>` sayfa verisi çekmek
için tek giriş noktasıdır; yeni bir niyet üzerindeki ilk çağrı `$B` temelleri
aracılığıyla prototip oluşturur ve JSON döndürür, eşleşen bir niyet üzerindeki
sonraki çağrılar kodlanmış bir tarayıcı-beceriye ~200ms'de yönlendirilir.
`/skillify` en son başarılı prototipi diskte kalıcı bir tarayıcı-beceriye kodlar.
Mutasyona uğrayan-akış kardeşi `/automate` 2b. Aşamaya ertelendi (TODOS içinde P0).

### v1.19.0.0 plan incelemesi sırasında kilitlenmiş kararlar (`/plan-eng-review`)

| ID | Karar | Kilitlenmiş davranış |
|----|----------|---------------------|
| **D1** | `/skillify` menşe koruması | ≤10 ajan turu geriye yürüyüp iyi-sınırlandırılmış bir `/scrape` çağrısını arar (prototipin niyet satırı + onu takip eden JSON çıktısı). Bulunamazsa, şu mesajla reddeder: *"Bu konuşmada yakın bir /scrape sonucu bulunamadı. Önce /scrape <intent> çalıştırın, ardından /skillify deyin."* Sessiz geri dönüş yok. |
| **D2** | Sentez giriş dilimi | Şablon, ajana SADECE kullanıcının kabul ettii JSON'u üreten son-deneme `$B` çağrılarını ve kullanıcının belirtilen niyet dizesini çıkarmasını söyler. Başarısız seçici denemelerini bırakır, ilgisiz sohbeti bırakır, daha erken oturum içeriğini bırakır. Codex bulgusu #6'yı seçenek (b) (ajanın kendi bağlamından yeniden istem, yapılandırılmış bir kaydedici değil) ile kapatır. |
| **D3** | Atomik yazma disiplini | `/skillify` `~/.gstack/.tmp/skillify-<spawnId>/` konumuna yazar, geçici dizine karşı `$B skill test` çalıştırır ve başarı + kullanıcı onayı üzerine sadece o zaman nihai katman yoluna yeniden adlandırır. Test başarısızlığı veya onay reddinde: geçici dizini tamamen `rm -rf` (asla-onaylanmamış beceriler için mezar taşı yok). Codex bulgusu #5'e göre `realpath`/`lstat` disiplini ile yeni modül `browse/src/browser-skill-write.ts` (`stageSkill` / `commitSkill` / `discardStaged`). |
| **D4** | Test kapsamı | 5 kapı-katman E2E (kazıma eşleşmesi, kazıma prototipi, becerileştirme mutlu yolu, becerileştirme menşe reddi, onay-kapısı reddi) + 1 birim testi (atomik-yazma yardımcı başarısızlık temizleme) + 1 el ile doğrulanmış duman (mutasyon-niyet reddi). `test/helpers/touchfiles.ts`'te kaydedildi. |

### Devredilenler

- **Varsayılan katman: genel.** Prosedürler için yalın genel, `/skillify` zamanında proje-bazlı geçersiz kılma (domain-skill kapsamını yansıtır). 1. Aşama depolama yardımcıları her iki arama yolunu destekler.
- **Bun çalışma zamanı dağıtımı.** Codex bulgusu #7 açık kalır. 2a. Aşama, Bun'un PATH üzerinde olduğunu varsayar (gstack zaten `setup:6-15` aracılığıyla gerektirir). `/skillify` SKILL.md "Sınırlamalar" bölümünde belgelenmiştir. Gerçek düzeltme 4. Aşamada gelir.

## 2b. Aşama — `/automate` taslağı

`/scrape`'in mutasyona uğrayan-akış kardeşi. Aynı becerileştirme deseni (`/skillify`
ve D3 yardımcısını olduğu gibi yeniden kullanır). Fark: kodlanmamış çalıştırıldığında
mutasyona uğrayan-adım-bazlı UNTRUSTED-sarmalanmış özet + `AskUserQuestion` onay kapısı.
Kodlamadan sonra, beceri katılımsız çalışır (kodlanmış betik hangi `$B click`/`fill`/`type`
çağrılarının çalışacağını numaralandırır). `TODOS.md`'deki P0 girdisine bakın.

## 3. Aşama taslağı

Oturum başlangıcında resolver enjeksiyonu. `server.ts:722-743`'teki domain-skill
enjeksiyonunu yansıtır:

```ts
const browserSkillsBlock = await renderBrowserSkillsForHost(hostname, projectSlug);
if (browserSkillsBlock) {
  systemPrompt += `\n\n${browserSkillsBlock}`;
}
```

`renderBrowserSkillsForHost()` 3 katmanı okur, `host` alanı eşleşen becerileri filtreler
ve UNTRUSTED-sarmalanmış bir blok yayar.

`gstack-config browser_skillify_prompts` (varsayılan kapalı): açıkken, `/qa`,
`/design-review`, vb.'de görev-sonu hatırlatmaları, etkinlik akışı tek bir ana bilgisayarda
≥N komut gösterdiğinde VE o ana bilgisayar+niyet için henüz beceri mevcut değilken tetiklenir.

## 4. Aşama taslağı

- LLM-hakem değerlendirmesi ("ajan beceriye mi ulaşıyor, yeniden mi keşfediyor?").
- Fixture-eskilik algılama — paketlenmiş fixture'ı canlı sayfa ile karşılaştırma.
- Güvenilmeyen spawn'lar için işletim sistemi düzeyinde FS korumalı alanı (macOS'ta
  `sandbox-exec`, Linux'ta ad alanları / seccomp).
- `$B skill upgrade <isim>` — kanonik SDK değiştiğinde kardeş SDK kopyasını yeniden oluşturma.

---

## Doğrulama (1. Aşama)

`bun test` yeni test dosyalarını geçirir:
- `browse/test/skill-token.test.ts` — 15 doğrulama
- `browse/test/browse-client.test.ts` — 26 doğrulama
- `browse/test/browser-skills-storage.test.ts` — 31 doğrulama
- `browse/test/browser-skill-commands.test.ts` — 29 doğrulama
- `browser-skills/hackernews-frontpage/script.test.ts` — 13 doğrulama
- `test/skill-validation.test.ts` — 7 yeni paketlenmiş-beceri doğrulaması

Daemon çalışırken uçtan-uca:

```bash
$B skill list                            # hackernews-frontpage gösterir (paketlenmiş)
$B skill show hackernews-frontpage       # SKILL.md yazdırır
$B skill run hackernews-frontpage        # 30 haberin JSON'unu döndürür
$B skill test hackernews-frontpage       # script.test.ts çalıştırır
```