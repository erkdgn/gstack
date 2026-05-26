# Plan Tuning v1 — Tasarım Dokümanı

**Durum:** Uygulama için onaylandı (2026-04-18)
**Dal:** garrytan/plan-tune-skill
**Yazarlar:** Garry Tan (kullanıcı), Claude Opus 4.7 + OpenAI Codex gpt-5.4'den AI destekli incelemelerle
**Yerine geçen kapsam:** [PLAN_TUNING_V0.md](./PLAN_TUNING_V0.md) (gözlemsel alt taban) üzerine yazım stili + LOC-makbuzları katmanı ekler. V0 yerinde değişmeden kalır.
**İlgili:** [PACING_UPDATES_V0.md](./PACING_UPDATES_V0.md) — çıkarılan adımlama yeniden tasarımı, V1.1 planı.

## Bu doküman nedir

/plan-tune v1'in ne olduğunun, ne OLMADIĞININ, neyi düşündüğümüzün ve her kararı neden aldığımızın kanonik kaydı. Depoya işlenir, böylece gelecekteki katkı sağlayıcılar (ve gelecekteki Garry) arkeoloji yapmadan akıl yürütmeyi izleyebilir. Kullanıcı başına yerel plan eserlerinin yerine geçer.

## Teşekkür

Bu plan **[Louise de Sadeleer](https://x.com/LouiseDSadeleer/status/2045139351227478199)** sayesinde var, gstack'i teknik olmayan bir kullanıcı olarak baştan sona yaşadı ve bize deneyimin nasıl hissettirdiği hakkında gerçeği söyledi. Onun spesifik geri bildirimleri:

1. "Bir süre sonra biraz yoruldum ve biraz katı hissettirdi." — *adımlama/yorgunluk*
2. "Evet evet evet diyeceğim" (mimari inceleme sırasında). — *kopma*
3. "Ürettiği kod satır sayısına vurgu yapması komik. AI elbette onun için üretti." — *LOC çerçevelemesi*
4. "Mühendis olmayan biri olarak bu biraz karmaşık." — *jargon yoğunluğu + sonuç çerçevelemesi*

V1 doğrudan #3 ve #4'ü ele alır: ilk kullanımda jargon açıklama + gerçek bir kişi tarafından okuyucu için yazılmış gibi sonuç çerçeveli yazım stili, artıca savunulabilir bir LOC yeniden çerçevelemesi. Louise'in #1 ve #2'si (adımlama/yorgunluk) ayrı bir tasarım turu gerektirir — [PACING_UPDATES_V0.md](./PACING_UPDATES_V0.md)'ye V1.1 planı olarak çıkarıldı.

## Özellik, bir paragrafta

gstack skill çıktısı üründür. Düz yazı teknik olmayan bir kurucu için iyi okunmuyorsa, incelemeden kopar ve "evet evet evet"e tıklarlar. V1, tier ≥ 2 her skill'e uygulanan bir yazım stili standardı ekler: ilk kullanımda jargon açıklama (küratörlü ~50 terim listesinden), sonuç terimleriyle çerçevelenmiş sorular ("kullanıcılarınız için ne bozulursa..."), kısa cümleler, somut isimler. V0 düz yazısını isteyen güçlü kullanıcılar `gstack-config set explain_level terse` ayarlayabilir. İkili anahtar, kısmi modlar yok. Artı: README'nin "600,000+ satır üretim kodu" çerçevelemesi — Louise tarafından haklı olarak LOC vanity olarak adlandırılan — `scc` destekli bir betikten gerçek bir hesaplanmış 2013-vs-2026 pro-rata katıyla, genel-vs-özel repo görünürlüğü hakkında dürüst uyarılarla değiştirilir.

## Daha küçük sürümü neden oluşturuyoruz

V1, birden fazla inceleme geçişinde dört önemli kapsam revizyonundan geçti. Son kapsam, herhangi bir ara sürümden daha küçüktür çünkü her inceleme geçişi gerçek sorunları yakaladı.

**Revizyon 1 — Dört seviyeli deneyim ekseni (reddedildi).** Orijinal teklif: kullanıcılara ilk çalıştırmada deneyimli bir geliştirici, solo deneyimsiz bir mühendis, takımda çalışan teknik olmayan veya tamamen teknik olmayan olup olmadıklarını sormak. Skill'ler seviyeye göre uyarlanır. CEO incelemesinin önerme meydan okuması adımında reddedildi çünkü (a) katılım sorması V1'in azaltmaya çalıştığı tam anda sürtünme ekler, (b) "hangi seviyeyim?" en çok yardıma ihtiyaç duyan kullanıcılar için kendisi kafa karıştırıcı bir sorudur, (c) teknik uzmanlık tek boyutlu değil (tasarımcı CSS'de A seviyesi, dağıtımda D seviyesi), (d) mühendisler de teknik olmayan kullanıcıların faydalandığı aynı yazım standartlarından faydalanır.

**Revizyon 2 — Varsayılan olarak ELI10, terse geri dönüş (kabul edildi).** Her skill'in çıktısı varsayılan olarak yazım standardına uyar. V0 düz yazısını isteyen güçlü kullanıcılar `explain_level: terse` ayarlar. Codex Geçiş 1 kritik boşlukları yakaladı (statik markdown geçidi, ana bilgisayar duyarlı yollar, README güncelleme mekanizması) — üçü de entegre edildi.

**Revizyon 3 — ELI10 + inceleme adımlama yeniden tasarımı (önerildi, kapsam geri alındı).** Bir adımlama iş kolu eklendi: bulguları sırala, iki yönlü kapıları otomatik kabul et, aşama başına en fazla 3 AskUserQuestion promptu, döndürme komutu ile Sessiz Kararlar bloğu. Louise'in #1 ve #2'sini doğrudan ele almak amaçlandı. Mühendislik incelemesi Geçiş 2 puanlama formülü ve yol tutarlılık hatalarını yakaladı. Mühendislik incelemesi Geçiş 3 + Codex Geçiş 2, adımlama iş kolunda düz metin düzenlemeyle düzeltilemeyen 10+ yapısal boşluk ortaya çıkardı.

**Revizyon 4 — Yalnızca ELI10 + LOC (nihai).** Kullanıcı kapsam küçültmeyi seçti: yazım stili + LOC makbuzları ile V1 gönder, adımlamayı [PACING_UPDATES_V0.md](./PACING_UPDATES_V0.md) aracılığıyla V1.1'e ertele. Bu onaylı V1 kapsamıdır.

Ortak çizgi: her inceleme geçiği doğru şekilde hırssı daralttı, kalan kapsamda yapısal boşluk kalmayana kadar. CEO inceleme skill'inin SCOPE REDUCTION moduyla eşleşir, mühendislik incelemesi yoluyla geç ulaşıldı, stratejik seçim yoluyla erken değil.

## v1 Kapsamı (şimdi oluşturduğumuz)

1. **Önyazıda Yazım Stili bölümü** (`scripts/resolvers/preamble.ts`). Altı kural: skill çağırma başına ilk kullanımda jargon açıklama, sonuç çerçevelemesi, kısa cümleler / somut isimler / etken fiiller, kararlar kullanıcı etkisiyle kapanır, koşulsuz-ilk kullanımda açıklama (kullanıcı terimi yapıştırmış olsa bile), kullanıcı dönüşü geçersiz kılma (kullanıcı "kısa ol" derse → o yanıt için atla).
2. **Repo aitli liste ile jargon sınırı** (`scripts/jargon-list.json`). ~50 küratörlü yüksek frekanslı teknik terim. Listede olmayan terimler yeterince düz İngilizce varsayılır. Terimler `gen-skill-docs` zamanında üretilen SKILL.md düz yazısına satır içi edilir (sıfır çalışma zamanı maliyeti).
3. **Terse geri dönüş** (`gstack-config set explain_level terse`). İkili: `default` vs `terse`. Terse, Yazım Stili bloğunu tamamen atlar ve V0 düz yazı stilini kullanır.
4. **Ana bilgisayar duyarlı önyazı yankısı.** `_EXPLAIN_LEVEL=$(${binDir}/gstack-config get explain_level 2>/dev/null || echo "default")`. Mevcut V0 `ctx.paths.binDir` deseni ile ana bilgisayar taşınabilir.
5. **gstack-config doğrulama.** `explain_level: default|terse`'i başlıkta belgele. Değerleri beyaz listele. Bilinmeyen değerlerde özel mesaj + `default`'a varsayılan ile uyar.
6. **README'de LOC yeniden çerçevelemesi.** "600,000+ satır üretim kodu" hero çerçevelemesini kaldır. `<!-- GSTACK-THROUGHPUT-PLACEHOLDER -->` bağlayıcısı ekle. Derleme zamanı betiği bağlayıcıyı hesaplanmış kat + uyarı ile değiştirir.
7. **`scc` destekli verimlilik betiği** (`scripts/garry-output-comparison.ts`). 2013 + 2026'nın her biri için, Garry tarafından yazılmış genel commit'leri listele, `git diff`'ten eklenen satırları çıkar, `scc --stdin` ile sınıflandır (veya regex yedeklemesi). Dil başına döküm + uyarılarla `docs/throughput-2013-vs-2026.json` çıktısı ver.
8. **`scc` bağımsız kurulum betiği olarak** (`scripts/setup-scc.sh`). Bir `package.json` bağımlılığı değil (gerçekten isteğe bağlı — kullanıcıların %95'i verimlilik asla çalıştırmaz). OS algılar ve `brew install scc` / `apt install scc` çalıştırır / GitHub sürümleri bağlantısını yazdırır.
9. **README güncelleme boru hattı** (`scripts/update-readme-throughput.ts`). Mevcutsa `docs/throughput-2013-vs-2026.json`'u okur, bağlayıcıyı hesaplanmış sayıyla değiştirir. Eksikse, CI'nın reddettiği `GSTACK-THROUGHPUT-PENDING` işaretçisini yazar — katkı sağlayıcıyı commit etmeden önce betiği çalıştırmaya zorlar.
10. **/retro, ham SLOC + ağırlıklı commit'leri mantıksal SLOC'un üstünde ekler.** Ham SLOC bağlam için kalır ama görsel olarak düşürülür.
11. **Yükseltme geçişi** (`gstack-upgrade/migrations/v<VERSION>.sh`). Yükseltme sonrası tek seferlik interaktif prompt, V0 düz yazısını `explain_level: terse` ile geri yüklemeyi tercih eden kullanıcılar için. Bayrak dosyası geçitli.
12. **Dokümantasyon.** CLAUDE.md bir Yazım Stili bölümü kazanır (proje kuralı). CHANGELOG.md V1 girdisi alır (kullanıcıya yönelik anlatı, kapsam küçültme + V1.1 adımlamayı bahseder). README.md bir Yazım Stili açıklayıcı bölümü alır (~80 kelime). CONTRIBUTING.md jargon listesi bakımı üzerine bir not alır (terim eklemek/kaldırmak için PR'ler).
13. **Testler.** 6 yeni test dosyası + mevcut `gen-skill-docs.test.ts` genişletmesi. LLM-yargı U2U dışında tümü geçit katmanı (dönemsel).
14. **V0 uyku negatif testleri.** 5D boyut adlarının ve 8 arketip adlarının varsayılan mod skill çıktısında görünmediğini iddia eder. V0 psikografik makinerinin V1'e sızmasını önler.
15. **V1 ve V1.1 tasarım dokümanları.** PLAN_TUNING_V1.md (bu dosya). PACING_UPDATES_V0.md (V1 uygulaması sırasında çıkarılan ekten oluşturulan V1.1 planı). TODOS.md P0 girdisi.

## Ertelendi

**V1.1'e (adanmış tasarım dokümanı ile açık):**
- İnceleme adımlama yeniden tasarımı (sıralama, otomatik kabul, aşama başına en fazla 3, Sessiz Kararlar bloğu, döndürme mekanizması). Gerekçe: bkz. [PACING_UPDATES_V0.md](./PACING_UPDATES_V0.md) §"Neden çıkarıldı." Düz metin değişiklikleriyle düzeltilemeyen 10+ yapısal boşluk var.
- Önyazı ilk çalıştırma meta-prompt denetimi (lake tanıtımı, telemetri, proaktif, yönlendirme). Louise hepsini ilk çalıştırmada gördü; yorgunluğa karşı sayılır. V1.1 N. oturuma kadar bastırmayı düşünür.

**V2'ye (veya sonrasına):**
- Soru günlüğünden kavışık sinyal algılama, anında çeviri tekliflerini yönlendirme.
- 5D psikografik odaklı skill uyarlaması (V0 E1 öğesi).
- /plan-tune anlatım + /plan-tune vibe (V0 E3 öğesi).
- Skill başına veya konu başına açıklama seviyeleri.
- Takım profilleri.
- AST tabanlı "teslim edilen özellikler" metriği.

## Tamamen reddedildi (düşünüldü, yapılmıyor)

- **Dört seviyeli beyan edilen deneyim ekseni (A/B/C/D).** CEO incelemesi önerme meydan okuması sırasında reddedildi. Yukarıdaki "Daha küçük sürümü neden oluşturuyoruz"a bak.
- **ELI10 yeni bir çözücü dosyası olarak (`scripts/resolvers/eli10-writing.ts`).** Codex Geçiş 1, önyazının AskUserQuestion Format bölümündeki mevcut "akıllı 16 yaşındaki" çerçevelleme ile çelişkiyi yakaladı. Bunun yerine mevcut önyazıya katlayın.
- **Yazım Stili bloğunun çalışma zamanı bastırılması.** Codex Geçiş 1, `gen-skill-docs`'ün statik Markdown ürettiğini yakaladı — çalışma zamanı `EXPLAIN_LEVEL=terse` zaten pişmiş içeriği gizleyemez. Çözüm: koşullu düzyazı geçidi (düz yazı kuralı, V0'ın `QUESTION_TUNING` geçidi ile aynı kategori).
- **Varsayılan ve terse arasında orta yazım modu.** Revizyon 3, "terse = açıklama yok ama sonuç çerçevelemesi koru" önerdi. Codex Geçiş 2, geçiş mesajlaşması ile çelişkiyi yakaladı. İkili kazan: terse = V0 düz yazısı, nokta.
- **Çalışma zamanında kullanıcı tarafından düzenlenebilir jargon listesi.** Revizyon 3, `~/.gstack/jargon-list.json`'u kullanıcı geçersiz kılması olarak önerdi. Codex Geçiş 2, üretim zamanı satır içi ile çelişkiyi yakaladı. Çözüm: yalnızca repo aitli, PR'ler ile ekle/kaldır, etkili olmak için yeniden oluştur.
- **package.json'da `devDependencies.optional` alanı.** Gerçek bir npm/bun alanı değil. Mühendislik incelemesi Geçiş 2 yakaladı. Bunun yerine bağımsız kurulum betiği.
- **README'de aynı dizeyi hem değiştirme bağlayıcısı hem CI-reddetme işaretçisi olarak kullanma.** Mühendislik incelemesi Geçiş 2 / Codex Geçiş 2, boru hattının kendi güncelleme yolunu yok ettiğini yakaladı. İki dize çözümü: `GSTACK-THROUGHPUT-PLACEHOLDER` (bağlayıcı, çalıştırmalar arasında kalır) vs `GSTACK-THROUGHPUT-PENDING` (açık "derleme çalışmadı" işaretçisi, CI reddeder).
- **"Her teknik terim bir açıklama alır" kabul kriteri olarak.** Codex Geçiş 2, küratörlü liste kuralı ile çelişkiyi yakaladı. Kabul kriteri kurala uyacak şekilde yeniden yazıldı: "scripts/jargon-list.json'da görünen her terim bir açıklama alır."
- **"/autoplan başına ≤ 12 AskUserQuestion promptu" kabul kriteri.** V1'den kaldırıldı — bu hedef şu anda V1.1'deki adımlama yeniden tasarımını gerektirir.

## Mimari

```
~/.gstack/
  developer-profile.json           # V0'dan değişmedi
  config.yaml                       # + explain_level anahtarı (default | terse)

scripts/
  jargon-list.json                  # YENİ: ~50 repo aitli terim (üretim zamanı satır içi)
  garry-output-comparison.ts        # YENİ: scc + git yıl başına, yazar kapsamlı
  update-readme-throughput.ts       # YENİ: README bağlayıcı değiştirme
  setup-scc.sh                      # YENİ: OS algılayan scc kurucu
  resolvers/preamble.ts             # DEĞİŞTİRİLDİ: Yazım Stili bölümü + EXPLAIN_LEVEL yankısı

docs/
  designs/PLAN_TUNING_V1.md         # YENİ: bu dosya
  designs/PACING_UPDATES_V0.md      # YENİ: V1.1 planı (çıkarıldı)
  throughput-2013-vs-2026.json      # YENİ: hesaplanmış, işlenmiş

~/.claude/skills/gstack/bin/
  gstack-config                     # DEĞİŞTİRİLDİ: explain_level başlığı + doğrulama

gstack-upgrade/migrations/
  v<VERSION>.sh                     # YENİ: V0 → V1 interaktif prompt
```

### Veri akışı

```
Kullanıcı tier-≥2 skill çalıştırır
       │
       ▼
Önyazı bash (çağırma başına):
  _EXPLAIN_LEVEL=$(${binDir}/gstack-config get explain_level 2>/dev/null || "default")
  echo "EXPLAIN_LEVEL: $_EXPLAIN_LEVEL"
       │
       ▼
Üretilmiş SKILL.md gövdesi (statik Markdown, gen-skill-docs'ta pişirilmiş):
  - AskUserQuestion Format bölümü (mevcut V0)
  - Yazım Stili bölümü (YENİ, koşullu düzyazı geçidi)
       │
       ├── "EXPLAIN_LEVEL: terse VEYA kullanıcı bu dönüşte 'kısa ol' derse atla"
       ├── 6 yazım kuralı (jargon, sonuç, kısa, etki, ilk kullanım, geçersiz kılma)
       └── scripts/jargon-list.json'dan satır içi jargon listesi
       │
       ▼
Aracı çalışma zamanı EXPLAIN_LEVEL + kullanıcı dönüş sinyaline göre uygular veya atlar
       │
       ▼
V0 QUESTION_TUNING + soru günlüğü + tercihler değişmedi
       │
       ▼
Kullanıcıya çıktı (ilk kullanımda açıklama, sonuç çerçeveli, kısa cümleler; veya terse ise V0 düz yazısı)
```

### Veri akışı: verimlilik betiği (derleme zamanı)

```
bun run build
   │
   ├── gen:skill-docs (jargon listesi satır içi edilmiş SKILL.md dosyalarını yeniden oluşturur)
   ├── update-readme-throughput (JSON mevcutsa okur; bağlayıcıyı değiştirir VEYA PENDING işaretçisi yazar)
   └── diğer adımlar (ikili derleme, vb.)

Ayrı olarak, talep üzerine:
bun run scripts/garry-output-comparison.ts
   │
   ├── scc ön kontrolü (eksikse → setup-scc.sh ipucu ile çık)
   ├── 2013 + 2026 için: garrytan/* genel repolarında Garry tarafından yazılmış commit'leri listele
   ├── Her commit için: git diff, EKLENEN satırları çıkar, scc --stdin ile sınıflandır
   └── docs/throughput-2013-vs-2026.json yaz (dil başına + uyarılar)
```

## Güvenlik + gizlilik

- **Yeni kullanıcı verisi yok.** V1 önyazı düz yazısı + yapılandırma anahtarını genişletir. Yeni kişisel veri toplanmaz.
- **Hassas verilerin çalışma zamanı dosya okuması yok.** Jargon listesi repo işlenmiş küratörlü bir listedir.
- **Geçiş betiği tek seferlik.** Bayrak dosyası yeniden ateşlemeyi önler.
- **scc yalnızca genel repolarda çalışır.** Özel çalışmaya erişim yok.

## Karar günlüğü (artıları/eksileri)

### Karar A: Dört seviyeli deneyim ekseni vs. varsayılan olarak ELI10 — YANIT: VARSAYILAN OLARAK ELI10

**Dört seviyeli eksen (reddedildi):** Kullanıcılara ilk çalıştırmada A/B/C/D olarak kendi tanımlamalarını sorun. Skill'ler seviyeye göre uyarlanır.
- Artıları: Açık kullanıcı egemenliği. Güçlü kullanıcılar V0 davranışını alır.
- Eksileri: Katılım sürtünmesi ekler. Kullanıcıları kendilerini etiketlemeye zorlar. Teknik uzmanlık tek boyutlu değil. Mühendisler de teknik olmayan kullanıcıların faydalandığı aynı yazım standartlarından faydalanır.

**Varsayılan olarak ELI10, terse geri dönüş ile (seçilen):** Her skill'in çıktısı varsayılan olarak yazım standardına uyar. Güçlü kullanıcılar `explain_level: terse` ayarlar.
- Artıları: Katılım sorusu yok. İyi yazım herkese fayda sağlar. Güçlü kullanıcıların hala bir kaçış yolu var.
- Eksileri: Yükseltme üzerine V0 davranışını sessizce değiştirir → geçiş promptu gerektirir.

### Karar B: Yeni çözücü dosyası vs. mevcut önyazıyı genişletme — YANIT: MEVCUT OLANI GENİŞLET

**Yeni çözücü (reddedildi):** `scripts/resolvers/eli10-writing.ts` ayrı bir üreteç olarak.
- Artıları: Modüler.
- Eksileri (Codex #7): Önyazının AskUserQuestion Format bölümündeki mevcut "akıllı 16 yaşındaki" çerçeveleme ile çelişir. İki gerçeklik kaynağı.

**Önyazıyı genişlet (seçilen):** Yazım Stili bölümü doğrudan AskUserQuestion Format altına `scripts/resolvers/preamble.ts`'ye eklenir.
- Artıları: Bir gerçeklik kaynağı. Mevcut kurallarla birleşir.
- Eksileri: `preamble.ts` büyür.

### Karar C: Çalışma zamanı bastırma vs. koşullu düzyazı geçidi — YANIT: KOŞULU DÜZYAZI GEÇİDİ

**Çalışma zamanı bastırma (reddedildi):** `explain_level` önyazı okuması bastırma mantığını tetikler.
- Artıları: Daha basit zihinsel model.
- Eksileri (Codex #1): `gen-skill-docs` statik Markdown üretir. Pişirildikten sonra içerik geriye dönük olarak gizlenemez. Çalışma zamanı bastırma kurgusaldır.

**Koşullu düzyazı geçidi (seçilen):** "EXPLAIN_LEVEL: terse VEYA kullanıcı bu dönüşte 'kısa ol' derse bu bloğu atla." Düzyazı kuralı; aracı çalışma zamanında uyar veya uymaz.
- Artıları: Test edilebilir. V0'ın `QUESTION_TUNING` deseniyle eşleşir. Mekanizma hakkında dürüst.
- Eksileri: Aracı düzyazı uyumuna bağlıdır (sabit çalışma zamanı geçidi yok).

### Karar D: Jargon listesi konumu — çalışma zamanı kullanıcı düzenlenebilir vs. repo aitli üretim zamanı — YANIT: REPO AİTLİ ÜRETİM ZAMANI

**Çalışma zamanında kullanıcı tarafından düzenlenebilir (reddedildi):** `~/.gstack/jargon-list.json` `scripts/jargon-list.json`'u geçersiz kılar.
- Artıları: Kullanıcı alanlarına özgü terimler ekleyebilir.
- Eksileri (Codex #4, Geçiş 2): Üretim zamanı satır içi, kullanıcı düzenlemelerinin yeniden oluşturma gerektirdiği anlamına gelir. Çelişki.

**Repo aitli, üretim zamanı satır içi (seçilen):** Yalnızca `scripts/jargon-list.json`. Eklemek/kaldırmak için PR'ler. `bun run gen:skill-docs` terimleri önyazı düz yazısına satır içi eder.
- Artıları: Bir gerçeklik kaynağı. Sıfır çalışma zamanı maliyeti. Mevcut derleme ile birleştirilebilir.
- Eksileri: Kullanıcılar yerel olarak terim ekleyemez. Azaltma: CONTRIBUTING.md'de belgelenmiş; PR'ler kabul edilir.

### Karar E: V1'de adımlama yeniden tasarımı vs. V1.1 — YANIT: V1.1 (çıkarıldı)

**V1'de adımlama (reddedildi):** Sıralama + otomatik kabul + Sessiz Kararlar + aşama başına en fazla 3上限 + döndürme mekanizması paketi.
- Artıları: Louise'in yorgunluğunu doğrudan ele alır.
- Eksileri (Mühendislik incelemesi Geçiş 3 + Codex Geçiş 2): Düz metin düzenlemeyle düzeltilemeyen 10+ yapısal boşluk. Oturum durum modeli tanımsız. Soru günlüğünden `phase` alanı eksik. Kayıt defteri dinamik inceleme bulgularını kapsamaz. Döndürme mekanizmasının uygulaması yok. Geçiş promptunun kendisi bir kesinti. İlk çalıştırma önyazı promptları da sayılır. Adımlama düzyazı olarak mevcut aşama başına sor执行 sırasını tersine çeviremez.

**V1.1'e çıkar (seçilen):** V1'de ELI10 + LOC gönder. Adımlama kendi tasarım turunu tam inceleme döngüsü ile alır.
- Artıları: V1'i dürüstçe gönderir. V1.1'e V1 kullanımından gerçek temel veri verir (Louise'in V1 transkripti). CEO incelemesinden SCOPE REDUCTION moduyla eşleşir.
- Eksileri: Louise'in yorgunluk şikayeti V1.1'e kadar tam olarak ele alınmaz. Azaltma: V1 yine yazım kalitesi aracılığıyla deneyimini iyileştirir; V1.1 adımlama ile takip eder.

### Karar F: README güncelleme mekanizması — tek dize vs. iki dize — YANIT: İKİ DİZE

**Tek dize (reddedildi):** `<!-- GSTACK-THROUGHPUT-MULTIPLE: N× -->` hem değiştirme bağlayıcısı hem CI-reddetme işaretçisi olarak.
- Artıları: Basit.
- Eksileri (Codex Geçiş 2): Boru hattı kendini yok eder — CI işaretçiyi içeren commit'leri reddeder, ama işaretçi bağlayıcının kendisidir.

**İki dize (seçilen):** `GSTACK-THROUGHPUT-PLACEHOLDER` (bağlayıcı, kararlı) + `GSTACK-THROUGHPUT-PENDING` (açık eksik derleme işaretçisi, CI reddeder).
- Artıları: Bağlayıcı kalıcı; CI gerçek başarısızlık durumunu yakalar.
- Eksileri: Hatırlanacak iki sembol.

## İnceleme kaydı

| İnceleme | Çalıştırma | Durum | Entegre edilen temel bulgular |
|---|---|---|---|
| CEO İncelemesi | 1 | TEMIZ (KAPSAMI TUT) | Önerme dönüşümü: dört seviyeli eksenden varsayılan olarak ELI10. Çapraz model gerilimleri açık kullanıcı seçimi ile çözüldü. |
| Codex İncelemesi | 2 | SORUNLAR_BULUNDU + kapsam küçültmeye yöneltti | Geçiş 1: 25 bulgu, 3 kritik engelleyici (statik markdown, ana bilgisayar yolları, README mekanizması). Geçiş 2: düzeltilmiş plan üzerinde 20 bulgu, V1.1 çıkarmaya yöneltti. |
| Mühendislik İncelemesi | 3 | TEMIZ (KAPSAM_KÜÇÜLTÜLDÜ) | Geçiş 1: kritik boşluklar + 3 karar (tümü A). Geçiş 2: puanlama formülü hatası, yol çelişkisi, sahte `devDependencies.optional` alanı. Geçiş 3: adımlama yapısal boşluklarını tanımladı, çıkarmaya yöneltti. |
| DX İncelemesi | 1 | TEMIZ (ÖNCELİKLENDİRME) | 3 kritik (docs planı, yükseltme geçişi, hero anı). 9 Sessiz DX Kararları olarak otomatik kabul edildi. |

İnceleme raporu `~/.gstack/` içinde `gstack-review-log` aracılığıyla kalıcı hale getirildi. Plan dosyası tam geçmiş ile `~/.claude/plans/system-instruction-you-are-working-transient-sunbeam.md` konumunda tutuldu.