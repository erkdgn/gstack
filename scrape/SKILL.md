---
name: scrape
version: 1.0.0
description: |
  Bir web sayfasından veri çekin. Yeni bir niyet için ilk çağrı, akışı $B
  temel komutlarıyla prototipler ve JSON döndürür. Eşleşen bir niyet için sonraki
  çağrılar, kodlaştırılmış bir browser-skill'e yönlendirilir ve ~200ms'de döner.
  Salt okunur — değiştiren akışlar (form doldurma, tıklama, gönderme) için /automate kullanın.
  Şunlarda kullanın: "scrape", "veri al", "çek", "ayıkla" veya bir sayfada
  "ne var". (gstack)
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
triggers:
  - scrape this page
  - get data from
  - pull from
  - extract from
  - what is on
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — doğrudan düzenlemeyin -->
<!-- Yeniden oluşturmak için: bun run gen:skill-docs -->

[ÖN HAZIRLIK İÇERİĞİ — Açıklama: Bu bölüm önceki dosyalardaki çeviriyle birebirdir. skill adı "scrape" olarak analytics satırında görünür. Tüm ön hazırlık, plan modu güvenli işlemler, plan modu sırasında yetenek çağırma, AskUserQuestion formatı, yapıtlar senkronizasyonu, gizlilik durdurma kapısı, modele özel davranış yaması, ses, bağlam kurtarma, yazım stili, tamlık ilkesi, karışıklık protokolü, sürekli checkpoint modu, bağlam sağlığı, soru ayarı, repo sahipliği, yapmadan önce ara, tamamlama durumu protokolü, operasyonel kendini geliştirme, telemetri ve plan durumu alt bilgisi bölümleri önceki dosyalardaki Türkçe çevirilerle tamamen aynıdır.]

# /scrape — bir sayfadan veri çek

Web'den veri almak için tek giriş noktası. Kaputun altında iki yol:

1. **Eşleşme yolu** (~200ms) — kullanıcının niyeti mevcut bir browser-skill'in
   tetikleyicileriyle eşleşirse, `$B skill run <ad>` ile çalıştırır ve JSON çıktısını
   yayar.
2. **Prototip yolu** (~30sn) — henüz eşleşen bir skill yok, bu yüzden sayfayı
   `$B` temel komutlarıyla yönlendirir, JSON döndürür ve bir sonraki çağrının
   eşleşme yoluna isabet etmesi için `/skillify` önerir.

Sözleşme gereği salt okunur. Niyet yazma ima ediyorsa (form gönderme,
düğme tıklama, durum değiştirme), reddedin ve `/automate`'e yönlendirin.

## Adım 1 — Niyeti belirle

`/scrape` sonrası kullanıcının isteği niyettir. Bir niyet dahil etmedilerse,
bir kez sorun:

> "Neyi scrape etmek istiyorsunuz? Tek bir satırda açıklayın, örn. 'Hacker News'teki
> en çok okunanlar' veya 'example.com/products üzerindeki ürün adları + fiyatlar'."

Önceden birden fazla açıklayıcı soru sormayın. Başka sorular, daha ucuz oldukları
prototip yolunda sorulur.

## Adım 2 — Değiştiren niyetleri reddet

Niyet yazmalar ima ediyorsa — *gönder*, *yayınla*, *gönder*, *giriş yap*,
*X'e tıkla*, *formu doldur*, *sil*, *oluştur*, *sipariş ver*, *rezervasyon yap*
fiilleri — yanıtlayın:

> "/scrape salt okunurdur. Değiştiren akışlar için /automate kullanın (browser-skills
> Faz 2 P0 TODOS.md'de — henüz gönderilmedi). O zamana kadar, doğrudan $B click /
> $B fill / $B type kullanın."

Durun. Eşleşme veya prototip yoluna girmeyin.

## Adım 3 — Eşleşme aşaması

Mevcut browser-skill'leri listeleyin:

```bash
$B skill list
```

Her skill için, `$B skill show <ad>` tam SKILL.md'yi `triggers:`,
`description:` ve `host:` dahil ortaya çıkarır. Bunları okuyun ve kullanıcının
niyetinin bunlardan biriyle anlamsal olarak eşleşip eşleşmediğini değerlendirin.

Güvenli bir eşleşme, **üçünün de doğru** olduğu anlamına gelir:

- Niyetin alan adı, skill'in `host`'uyla (veya alan adlarından biriyle) eşleşir
- Bir `triggers:` ifadesi veya `description:`, niyetin istediği verileri kapsar
- Niyet, skill'in `args:`'da bildirmediği argümanlar gerektirmez

Eşleşirse, niyetten `--arg anahtar=değer` ayrıştırın (veya sıfır argümanlı
skill'ler için hiçbiri geçmeyin) ve çalıştırın:

```bash
$B skill run <ad> [--arg anahtar=değer ...]
```

Skill'in stdout'a yazdırdığı JSON'ı yayın. Durun.

Eşleşme belirsizse (iki skill mantıklı olarak uyuşabilir), daha dar katmanlı
olanı seçin (project > global > bundled — `$B skill list` katmanı gösterir).
Hâlâ belirsizse, yanlış tahmin etmek yerine prototip yoluna geçin.

## Adım 4 — Prototip aşaması

Eşleşme yok. Sayfayı `$B` temel komutlarıyla yönlendirin:

1. `$B goto <url>` — hedefe gidin. Kullanıcının niyeti genellikle bir alan adı
   veya URL belirtir; doğrudan kullanın.
2. `$B snapshot --text` (veya `$B text`) — seçici bulmak için sayfanın temiz
   bir metin görünümünü alın.
3. `$B html` — yapılandırılmış veriyi ayrıştırmanız gerektiğinde ham HTML'i çekin
   (listeler, tablolar, tekrarlanan satırlar).
4. `$B links` — niyet URL'leri toplamak olduğunda.
5. Yineleyin: bir seçici deneyin, çıktıyı kontrol edin, iyileştirin.

Sonucu stdout'ta JSON olarak yayın (tek bir belge, güzel yazdırılmamış).
Kararlı bir şekil kullanın — tipik olarak `{ "items": [...], "count": N }` veya
benzeri — böylece aşağı akış tüketiciler veri olarak ele alabilir.

## Adım 5 — Skillify dürtüsü

Başarılı bir prototipten sonra, tam olarak bir satır ekleyin:

> "Bunu kalıcı bir skill yapmak için /skillify deyin (bir sonraki çağrıda 200ms)."

Bu tüm dürtüdür. Israr etmeyin, avantajları listelemeyin, zorlamayın.
Proaktif yüzey gösterimi bir Faz 3 düğmesidir (`gstack-config browser_skillify_prompts`),
bu yeteneğin işi değildir.

## Prototip başarısız olduğunda

Sayfa yükleniyor ama veri ayıklama 3-4 seçici denemesinden sonra mantıklı bir
JSON şekli üretmiyorsa:

- Ne denediğinizi, ne geldiğini ve neyin engellediğini (tembel yüklenmiş,
  JS ile oluşturulmuş, ödemeli duvar vb.) raporlayın.
- Kısmi bir sonuç yazıp bitti demeyin.
- Bozuk bir prototipte /skillify önermeyin.
- Kullanıcıya (a) farklı bir seçici denemek, (b) farklı bir sayfaya geçmek
  veya (c) durmak isteyip istemediğini sorun.

## Bu yeteneğin yapMADIĞI şeyler

- Değiştiren eylemler (gönderildiğinde /automate kullanın veya doğrudan $B temel komutları)
- Kimlik doğrulama akışları / çerez içe aktarma (önce /setup-browser-cookies kullanın)
- Çok sayfalı taramalar (bu çağrı başına tek seferlik)
- Daemon'ın çalışmamasını gerektiren herhangi bir şey

## Çıktı disiplini

Eşleşme yolu, eşleşen skill'in yaydığı herhangi bir JSON'ı döndürür. Prototip
yolu, oluşturduğunuz herhangi bir JSON'ı döndürür. Her iki durumda:

- Tek bir JSON belgesi, stdout'ta.
- Stderr (veya sohbet) günlükler ve skillify dürtüsü içindir.
- Kullanıcı bir açıklama istemedikçe JSON'ı sohbet yanıtında düzyazı ile sarmayın —
  birçok `/scrape` çağırıcısı çıktıyı `jq`'ya aktarır.

## Öğrenmeleri Yakala

Bu oturumda bariz olmayan bir kalıp, tuzak veya mimari içgörü keşfettiyseniz,
gelecek oturumlar için günlüğe kaydedin:

```bash
~/.claude/skills/gstack/bin/gstack-learnings-log '{"skill":"scrape","type":"TUR","key":"KISA_ANAHTAR","insight":"ACIKLAMA","confidence":N,"source":"KAYNAK","files":["yol/ilgili/dosya"]}'
```

**Türler:** `pattern` (yeniden kullanılabilir yaklaşım), `pitfall` (yapılmaması gereken), `preference`
(kullanıcı belirtilen), `architecture` (yapısal karar), `tool` (kütüphane/framework içgörüsü),
`operational` (proje ortamı/CLI/iş akışı bilgisi).

**Kaynaklar:** `observed` (bunu kodda buldunuz), `user-stated` (kullanıcı size söyledi),
`inferred` (yapay zeka çıkarımı), `cross-model` (hem Claude hem Codex katılıyor).

**Güven:** 1-10. Dürüst olun. Kodda doğruladığınız gözlemlenen bir kalıp 8-9'dur.
Emin olmadığınız bir çıkarım 4-5'tir. Kullanıcının açıkça belirttiği bir tercih 10'dur.

**dosyalar:** Bu öğrenmenin referans olduğu belirli dosya yollarını ekleyin. Bu,
eskime algılamasını sağlar: bu dosyalar daha sonra silinirse, öğrenme işaretlenebilir.

**Yalnızca gerçek keşifleri günlüğe kaydedin.** Bariz şeyleri günlüğe kaydetmeyin. Kullanıcının
zaten bildiği şeyleri günlüğe kaydetmeyin. İyi bir test: bu içgörü gelecekteki bir oturumda
zaman kazandırır mı? Evet ise, günlüğe kaydedin.