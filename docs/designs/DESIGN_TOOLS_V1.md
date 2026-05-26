# Tasarım: gstack Görsel Tasarım Oluşturma (`design` ikili dosyası)

/office-hours tarafından 2026-03-26 tarihinde oluşturuldu
Dal: garrytan/agent-design-tools
Depo: gstack
Durum: TASLAK
Mod: Girişimcilik

## Bağlam

gstack'ın tasarım becerileri (/office-hours, /design-consultation, /plan-design-review, /design-review) hepsi tasarımın **metin açıklamalarını** üretir — hex kodları, piksel özellikleri düzyazıda olan plan belgeleri, ASCII art wireframe'leri olan DESIGN.md dosyaları. Yaratıcı, OmniGraffle'da HelloSign'i el ile tasarlamış bir tasarımcı ve bunu utanç verici buluyor.

Değer birimi yanlış. Kullanıcıların daha zengin tasarım diline ihtiyacı yok — yürütebilir bir görsel esere ihtiyaçları var ki konuşma "bu özellik hoşuna gitti mi?"den "bu ekranda mı?"e dönüşsün.

## Problem Bildirimi

Tasarım becerileri tasarımı metin olarak açıklıyor, göstermiyor. Argus UX genel bakış planı örnek: 487 satır detaylı duygusal yay özellikleri, tipografi seçimleri, animasyon zamanlaması — sıfır görsel eser. "Tasarım" yapan bir AI kodlama ajanı, bakabileceğiniz ve içsel olarak tepki verebileceğiniz bir şey üretmelidir.

## Talep Kanıtı

Yaratıcı/birincil kullanıcı mevcut çıktıyı utanç verici buluyor. Her tasarım beceri oturumu, bir mockup olması gereken düzyazı ile bitiyor. GPT Image API artık doğru metin oluşturma ile piksel-mükemmel UI mockup'ları üretiyor — salt-metin çıktıyı haklı çıkaran yetenek açığı artık mevcut değil.

## En Dar Kama

OpenAI Images/Responses API'sini saran, derlenmiş bir TypeScript ikili dosyası (`design/dist/design`), beceri şablonlarından `$D` ile çağrılabilir (mevcut `$B` tarama ikili dosyası desenini yansıtır). Öncelikli entegrasyon sırası: /office-hours → /plan-design-review → /design-consultation → /design-review.

## Kabul Edilen Öncüller

1. GPT Image API (OpenAI Responses API üzerinden) doğru motor. Google Stitch SDK yedek.
2. **Görsel mockup'lar tasarım becerileri için varsayılan açık**, kolay atlama yolu ile — katılım değil. (Codex meydan okumasına göre revize edildi.)
3. Entegrasyon paylaşılan bir yardımcı programdır (beceri başına yeniden uygulama değil) — her becerinin çağırabileceği bir `design` ikili dosyası.
4. Öncelik: /office-hours önce, sonra /plan-design-review, /design-consultation, /design-review.

## Çapraz-Model Perspektif (Codex)

Codex bağımsız olarak çekirdek tezi doğruladı: "Başarısızlık markdown içindeki çıktı kalitesi değil; mevcut değer biriminin yanlış olmasıdır." Temel katkılar:
- Öncül #2'ye meydan okudu (katılım → varsayılan açık) — kabul edildi
- Görüş tabanlı kalite kapısı önerdi: okunamaz metin, eksik bölümler, bozuk düzen için GPT-4o görüş kullan, başarısız olursa bir kez otomatik yeniden dene
- 48 saatlik prototip kapsamlandı: paylaşılan `visual_mockup.ts` yardımcı programı, sadece /office-hours + /plan-design-review, kahraman mockup + 2 varyant

## Önerilen Yaklaşım: `design` İkili Dosyası (Yaklaşım B)

### Mimari

**Tarama ikili dosyasının derleme ve dağıtım desenini paylaşır** (bun build --compile, kurulum betiği, beceri şablonlarında $VARIABLE çözümlemesi) ama mimari olarak daha basittir — kalıcı daemon sunucusu yok, Chromium yok, sağlık kontrolleri yok, belirteç kimlik doğrulaması yok. Tasarım ikili dosyası OpenAI API çağrıları yapan ve diskte PNG'ler yazan durumsuz bir CLI'dir. Oturum durumu (çok turlu yineleme için) bir JSON dosyasıdır.

**Yeni bağımlılık:** `openai` npm paketi (`devDependencies`'e ekleyin, çalışma zamanı bağımlılıkları DEĞİL). Tasarım ikili dosyası taramadan ayrı derlenir, böylece openai tarama ikili dosyasını şişirmez.

```
design/
├── src/
│   ├── cli.ts            # Giriş noktası, komut gönderimi
│   ├── commands.ts        # Komut kayıt defteri (belgelendirme + doğrulama için tek kaynak)
│   ├── generate.ts        # Yapılandırılmış kısa betiden mockup'lar oluştur
│   ├── iterate.ts         # Mevcut mockup'lar üzerinde çok turlu yineleme
│   ├── variants.ts        # Kısa betiden N tasarım varyantı oluştur
│   ├── check.ts           # Görüş tabanlı kalite kapısı (GPT-4o)
│   ├── brief.ts           # Yapılandırılmış kısa beti tipi + birleştirme yardımcıları
│   └── session.ts         # Oturum durumu (çok turlu için yanıt kimlikleri)
├── dist/
│   ├── design             # Derlenmiş ikili dosya
│   └── .version           # Git karma özeti
└── test/
    └── design.test.ts     # Entegrasyon testleri
```

### Komutlar

```bash
# Yapılandırılmış kısa betiden kahraman mockup oluştur
$D generate --brief "Kodlama değerlendirme aracı için Kontrol Paneli. Koyu tema, krem vurgular. Gösteren: oluşturucu adı, puan rozeti, anlatı mektubu, puan kartları. Hedef: teknik kullanıcılar." --output /tmp/mockup-hero.png

# 3 tasarım varyantı oluştur
$D variants --brief "..." --count 3 --output-dir /tmp/mockups/

# Geri besleme ile mevcut bir mockup üzerinde yineleme yap
$D iterate --session /tmp/design-session.json --feedback "Puan kartlarını daha büyük yap, anlatıyı puanların üstüne taşı" --output /tmp/mockup-v2.png

# Görüş tabanlı kalite kontrolü (GEÇTİ/KALDI + sorunlar döndürür)
$D check --image /tmp/mockup-hero.png --brief "Oluşturucu adı, puan rozeti, anlatı içeren Kontrol Paneli"

# Kalite kapısı + otomatik yeniden deneme ile tek seferde
$D generate --brief "..." --output /tmp/mockup.png --check --retry 1

# Yapılandırılmış kısa betiyi JSON dosyası ile geçir
$D generate --brief-file /tmp/brief.json --output /tmp/mockup.png

# Kullanıcı incelemesi için karşılaştırma panosu HTML oluştur
$D compare --images /tmp/mockups/variant-*.png --output /tmp/design-board.html

# Rehberli API anahtarı kurulumu + duman testi
$D setup
```

**Kısa beti giriş modları:**
- `--brief "düz metin"` — serbest biçimli metin istemi (basit mod)
- `--brief-file path.json` — `DesignBrief` arayüzüyle eşleşen yapılandırılmış JSON (zengin mod)
- Beceriler bir JSON kısa beti dosyası oluşturur, /tmp'ye yazar ve `--brief-file` geçirir

**Tüm komutlar `commands.ts`'te kayıtlıdır**, `--check` ve `--retry` dahil `generate` üzerinde bayraklar olarak.

### Tasarım Keşfi İş Akışı (mühendislik incelemesinden)

İş akışı sıralıdır, paralel değil. PNG'ler görsel keşif içindir (insan-yönelimli), HTML wireframe'ler uygulama içindir (ajan-yönelimli):

```
1. $D variants --brief "..." --count 3 --output-dir /tmp/mockups/
   → 2-5 PNG mockup varyasyonu oluşturur

2. $D compare --images /tmp/mockups/*.png --output /tmp/design-board.html
   → HTML karşılaştırma panosu oluşturur (aşağıda özellik)

3. $B goto file:///tmp/design-board.html
   → Kullanıcı görsel Chrome'da tüm varyantları inceler

4. Kullanıcı favorisini seçer, derecelendirir, yorumlar, [Gönder]'e tıklar
   Ajan yoklar: $B eval document.getElementById('status').textContent
   Ajan okur: $B eval document.getElementById('feedback-result').textContent
   → Pano yok, yapıştırma yok. Ajan geri beslemeyi doğrudan sayfadan okur.

5. Claude onaylanan yönle eşleştiren DESIGN_SKETCH aracılığıyla HTML wireframe oluşturur
   → Ajan şeffaf PNG'den değil, incelenebilir HTML'den uygular
```

### Karşılaştırma Panosu Tasarım Özelliği (/plan-design-review'den)

**Sınıflandırıcı:** UYGULAMA UI'si (görev-odaklı, yardımcı program sayfası). Ürün markası yok.

**Düzen: Tek sütun, tam genişlikte mockup'lar.** Her varyant maksimum görüntü sadakati için tam görüntü alanı genişliği alır. Kullanıcılar dikey olarak varyantları kaydırır.

```
┌─────────────────────────────────────────────────────────────┐
│  BAŞLIK ÇUBUĞU                                              │
│  "Tasarım Keşfi" . proje adı . "3 varyant"                  │
│  Mod göstergesi: [Geniş keşif] | [DESIGN.md ile eşleşen]  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              VARYANT A (tam genişlik)                    │  │
│  │         [ mockup PNG, max-width: 1200px ]              │  │
│  ├───────────────────────────────────────────────────────┤  │
│  │ (●) Seç   ★★★★☆   [Ne sevmediniz/ne sevmediniz?____]   │  │
│  │            [Buna benzer daha fazla]                            │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              VARYANT B (tam genişlik)                    │  │
│  │         [ mockup PNG, max-width: 1200px ]              │  │
│  ├───────────────────────────────────────────────────────┤  │
│  │ ( ) Seç   ★★★☆☆   [Ne sevmediniz/ne sevmediniz?____]   │  │
│  │            [Buna benzer daha fazla]                            │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
│  ... (daha fazla varyant için kaydır)                             │
│                                                             │
│  ─── ayırıcı ─────────────────────────────────────────    │
│  Genel yön (isteğe bağlı, varsayılan olarak katlanmış)         │
│  [metin alanı, 3 satır, odakta genişleyen]                       │
│                                                             │
│  ─── YENİDEN OLUŞTUR ÇUBUĞU (#f7f7f7 arka plan) ──────────────────    │
│  "Daha fazlasını keşfetmek ister misiniz?"                                    │
│  [Tamamen farklı]  [Tasarımım ile eşleş]  [Özel: ______]   │
│                                          [Yeniden oluştur ->]    │
│  ─────────────────────────────────────────────────────────  │
│                                        [ ✓ Gönder ]         │
└─────────────────────────────────────────────────────────────┘
```

**Görsel özellik:**
- Arka plan: #fff. Gölgeler yok, kart kenarlıkları yok. Varyant ayrımı: 1px #e5e5e5 çizgi.
- Tipografi: sistem yazı tipi yığını. Başlık: 16px yarı kalın. Etiketler: 14px yarı kalın. Geri bildirim yer tutucusu: 13px normal #999.
- Yıldız derecelendirme: 5 tıklanabilir yıldız, dolu=#000, boş=#ddd. Renkli değil, animasyonlu değil.
- Radyo düğmesi "Seç": açık favori seçimi. Varyant başına bir, birbirini dışlayan.
- "Buna benzer daha fazla" düğmesi: varyant başına, o varyantın stilini tohum olarak kullanarak yeniden oluşturmayı tetikler.
- Gönder düğmesi: #000 arka plan, beyaz metin, sağa hizalanmış. Tek CTA.
- Yeniden oluştur çubuğu: #f7f7f7 arka plan, geri bildirim alanından görsel olarak farklı.
- Maks genişlik: mockup görüntüleri için 1200px ortalanmış. Kenar boşlukları: 24px yanlar.

**Etkileşim durumları:**
- Yükleniyor (görüntüler hazır olmadan sayfa açılır): varyant başına "Varyant A oluşturuluyor..." iskelet nabzı. Yıldızlar/metin alanı/seçim devre dışı.
- Kısmi başarısızlık (3'ten 2'si başarılı): iyi olanları göster, başarısız olan için varyant başına [Yeniden Dene] ile hata kartı.
- Gönderim-sonrası: "Geri bildirim gönderildi! Kodlama ajanınıza dönün." Sayfa açık kalır.
- Yeniden oluşturma: yumuşak geçiş, eski varyantları soluklaştır, iskelet nabızları, yenilerini soluklaştır. Kaydırma üste sıfırlar. Önceki geri bildirim temizlenir.

**Geri besleme JSON yapısı** (gizli #feedback-result elemanına yazılır):
```json
{
  "preferred": "A",
  "ratings": { "A": 4, "B": 3, "C": 2 },
  "comments": {
    "A": "Aralığı seviyorum, başlık doğru hissettiriyor",
    "B": "Çok yoğun ama iyi renk paleti",
    "C": "Tamamen yanlış ruh"
  },
  "overall": "A ile devam et, CTA'yı daha büyük yap",
  "regenerated": false
}
```

**Erişilebilirlik:** Yıldız derecelendirmeleri klavye ile gezilebilir (ok tuşları). Metin alanları etiketli ("Varyant A için Geri Bildirim"). Gönder/Yeniden Oluştur klavye ile erişilebilir görünür odak halkası ile. Tüm metin beyaz üzerinde #333+.

**Duyarlı:** >1200px: rahat kenar boşlukları. 768-1200px: daha sıkı kenar boşlukları. <768px: tam genişlik, yatay kaydırma yok.

**Ekran görüntüsü onayı (sadece $D evolve için ilk sefer):** "Bu, tasarım evrimi için canlı sitenizin ekran görüntüsünü OpenAI'e gönderir. [Devam et] [Bir daha sorma]" ~/.gstack/config.yaml içinde design_screenshot_consent olarak saklanır.

Neden sıralı: Codex çapraz inceleme, raster PNG'lerin ajanlar için opak olduğunu (DOM, durumlar, fark yapılabilir yapı yok) tespit etti. HTML wireframe'ler koda geri köprü korur. PNG insanın "evet, bu doğru" demesi içindir. HTML ajanın "bunu nasıl oluşturacağımı biliyorum" demesi içindir.

### Temel Tasarım Kararları

**1. Durumsuz CLI, daemon değil**
Tarama kalıcı bir Chromium örneğine ihtiyaç duyar. Tasarım sadece API çağrılarıdır — sunucu için hiçbir neden yok. Çok turlu yineleme için oturum durumu, `previous_response_id` içeren `/tmp/design-session-{id}.json`'a yazılan bir JSON dosyasıdır.
- **Oturum kimliği:** `${PID}-${timestamp}`'dan oluşturulur, `--session` bayrağı ile geçirilir
- **Keşif:** `generate` komutu oturum dosyasını oluşturur ve yolunu yazdırır; `iterate` `--session` ile okur
- **Temizleme:** /tmp'deki oturum dosyaları geçicidir (işletim sistemi temizler); açık temizleme gerekmez

**2. Yapılandırılmış kısa beti girişi**
Kısa beti, beceri düzyazısı ile görüntü oluşturma arasındaki arayüzdür. Beceriler onu tasarım bağlamından oluşturur:
```typescript
interface DesignBrief {
  goal: string;           // "Kodlama değerlendirme aracı için Kontrol Paneli"
  audience: string;       // "Teknik kullanıcılar, YC ortakları"
  style: string;          // "Koyu tema, krem vurgular, minimal"
  elements: string[];     // ["oluşturucu adı", "puan rozeti", "anlatı mektubu"]
  constraints?: string;   // "Maks genişlik 1024px, mobil-ilk"
  reference?: string;     // Mevcut ekran görüntüsüne veya DESIGN.md alıntısına yol
  screenType: string;     // "desktop-dashboard" | "mobile-app" | "landing-page" | vb.
}
```

**3. Tasarım becerilerinde varsayılan açık**
Beceriler varsayılan olarak mockup oluşturur. Şablon atlama dilini içerir:
```
Görsel mockup oluşturuluyor... (görsellere ihtiyacınız yoksa "atla" deyin)
```

**4. Görüş kalite kapısı**
Oluşturduktan sonra, isteğe bağlı olarak görüntüyü GPT-4o görüş ile kontrol eder:
- Metin okunabilirliği (etiketler/başlıklar okunabilir mi?)
- Düzen tamlığı (tüm istenen öğeler mevcut mu?)
- Görsel tutarlılık (gerçek bir UI gibi mi, kolaj değil mi?)
Başarısızlıkta bir kez otomatik yeniden dene. Hala başarısız olursa, yine de uyarı ile sunar.

**5. Çıktı konumu: keşifler /tmp'de, onaylı finaller `docs/designs/`'te**
- Keşif varyantları `/tmp/gstack-mockups-{session}/` konumuna gider (geçici, işlenmez)
- Sadece **kullanıcının onayladığı nihai** mockup `docs/designs/`'e kaydedilir (işlenir)
- Varsayılan çıktı dizini CLAUDE.md `design_output_dir` ayarı ile yapılandırılabilir
- Dosya adı deseni: `{skill}-{description}-{timestamp}.png`
- Mevcut değilse `docs/designs/` oluştur (mkdir -p)
- Tasarım belgesi işlenen görüntü yoluna başvurur
- Her zaman Read aracı ile kullanıcıya gösterin (Claude Code'da görüntüleri satır içi render eder)
- Bu, repo şişirmesini önler: sadece onaylı tasarımlar işlenir, her keşif varyantı değil
- Geri dönüş: git deposunda değilse, `/tmp/gstack-mockup-{timestamp}.png` konumuna kaydet

**6. Güven sınırı kabulü**
Varsayılan açık oluşturma, tasarım kısa beti metnini OpenAI'e gönderir. Bu, tamamen yerel olan mevcut HTML wireframe yolu ile karşılaştırıldığında yeni bir harici veri akışıdır. Kısa beti sadece soyut tasarım açıklamaları (hedef, stil, öğeler) içerir, asla kaynak kodu veya kullanıcı verisi. $B'den ekran görüntüleri OpenAI'e GÖNDERİLMEZ (DesignBrief'teki referans alanı ajan tarafından kullanılan yerel bir dosya yoludur, API'ye yüklenmez). Bunu CLAUDE.md'de belgeleyin.

**7. Hız sınırı azaltma**
Varyant oluşturma kademeli paralel kullanır: her API çağrısını `Promise.allSettled()` ile 1 saniye arayla başlatır. Bu, görüntü oluşturmada 5-7 RPM hız sınırını önlerken hala tamamen seri olandan daha hızlıdır. Herhangi bir çağrı 429 verirse, üstel geri dönüş ile yeniden dener (2sn, 4sn, 8sn).

### Şablon Entegrasyonu

**Mevcut resolver'a ekleyin:** `scripts/resolvers/design.ts` (YENİ dosya DEĞİL)
- `{{DESIGN_SETUP}}` yer tutucusu için `generateDesignSetup()` ekleyin (`generateBrowseSetup()`'i yansıtır)
- `{{DESIGN_MOCKUP}}` yer tutucusu için `generateDesignMockup()` ekleyin (tam keşif iş akışı)
- Tüm tasarım resolver'larını bir dosyada tutar (mevcut kod tabanı kuralıyla tutarlı)

**Yeni HostPaths girdisi:** `types.ts`
```typescript
// claude barındırıcısı:
designDir: '~/.claude/skills/gstack/design/dist'
// codex barındırıcısı:
designDir: '$GSTACK_DESIGN'
```
Not: Codex çalışma zamanı kurulumu (`setup` betiği) de `GSTACK_DESIGN` ortam değişkenini dışa aktarmalıdır, `GSTACK_BROWSE`'ın nasıl ayarlandığına benzer şekilde.

**`$D` çözümleme bash bloğu** (`{{DESIGN_SETUP}}` tarafından oluşturulur):
```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
D=""
[ -n "$_ROOT" ] && [ -x "$_ROOT/.claude/skills/gstack/design/dist/design" ] && D="$_ROOT/.claude/skills/gstack/design/dist/design"
[ -z "$D" ] && D=~/.claude/skills/gstack/design/dist/design
if [ -x "$D" ]; then
  echo "DESIGN_READY: $D"
else
  echo "DESIGN_NOT_AVAILABLE"
fi
```
`DESIGN_NOT_AVAILABLE` ise: beceriler mevcut `DESIGN_SKETCH` desenine geri döner (HTML wireframe oluşturma). Tasarım mockup'ı aşamalı bir iyileştirmedir, zorlu bir gereksinim değil.

**Mevcut resolver'da yeni işlevler:** `scripts/resolvers/design.ts`
- `{{DESIGN_SETUP}}` için `generateDesignSetup()` ekleyin — `generateBrowseSetup()` desenini yansıtır
- `{{DESIGN_MOCKUP}}` için `generateDesignMockup()` ekleyin — tam oluştur+kontrol+sun iş akışı
- Tüm tasarım resolver'larını bir dosyada tutar (mevcut kod tabanı kuralıyla tutarlı)

### Beceri Entegrasyonu (Öncelik Sırası)

**1. /office-hours** — Görsel Taslak bölümünü değiştir
- Yaklaşım seçiminden sonra (4. Aşama), kahraman mockup + 2 varyant oluştur
- Tüm üçünü Read aracı ile sun, kullanıcıya seçmesini söyle
- İstenirse yineleme yap
- Seçilen mockup'ı tasarım belgesinin yanına kaydet

**2. /plan-design-review** — "Daha iyisinin nasıl göründüğü"
- Bir tasarım boyutu <7/10 puan aldığında, 10/10'un nasıl görüneceğini gösteren bir mockup oluştur
- Yan yana: mevcut ($B ile ekran görüntüsü) vs. önerilen ($D ile mockup)

**3. /design-consultation** — Tasarım sistemi önizlemesi
- Önerilen tasarım sisteminin görsel önizlemesini oluştur (tipografi, renkler, bileşenler)
- /tmp HTML önizleme sayfasını uygun bir mockup ile değiştir

**4. /design-review** — Tasarım niyeti karşılaştırması
- Plan/DESIGN.md özelliklerinden "tasarım niyeti" mockup'ı oluştur
- Görsel fark için canlı site ekran görüntüsü ile karşılaştır

### Oluşturulacak Dosyalar

| Dosya | Amaç |
|------|---------|
| `design/src/cli.ts` | Giriş noktası, komut gönderimi |
| `design/src/commands.ts` | Komut kayıt defteri |
| `design/src/generate.ts` | Responses API üzerinden GPT Image oluşturma |
| `design/src/iterate.ts` | Oturum durumu ile çok turlu yineleme |
| `design/src/variants.ts` | N tasarım varyantı oluştur |
| `design/src/check.ts` | Görüş tabanlı kalite kapısı |
| `design/src/brief.ts` | Yapılandırılmış kısa beti tipleri + yardımcıları |
| `design/src/session.ts` | Oturum durumu yönetimi |
| `design/src/compare.ts` | HTML karşılaştırma panosu oluşturucu |
| `design/test/design.test.ts` | Entegrasyon testleri (sahte OpenAI API) |
| (yok — mevcut `scripts/resolvers/design.ts`'e ekleyin) | `{{DESIGN_SETUP}}` + `{{DESIGN_MOCKUP}}` resolver'ları |

### Değiştirilecek Dosyalar

| Dosya | Değişiklik |
|------|--------|
| `scripts/resolvers/types.ts` | `designDir`'i HostPaths'e ekle |
| `scripts/resolvers/index.ts` | DESIGN_SETUP + DESIGN_MOCKUP resolver'larını kaydet |
| `package.json` | `design` derleme komutu ekle |
| `setup` | Tarama yanında tasarım ikili dosyasını derle |
| `scripts/resolvers/preamble.ts` | Codex barındırıcısı için `GSTACK_DESIGN` ortam değişkeni dışa aktarımı ekle |
| `test/gen-skill-docs.test.ts` | Yeni resolver'lar için DESIGN_SKETCH test paketini güncelle |
| `setup` | Tasarım ikili dosyası derleme + Codex/Kiro varlık bağlama ekle |
| `office-hours/SKILL.md.tmpl` | Görsel Taslak bölümünü `{{DESIGN_MOCKUP}}` ile değiştir |
| `plan-design-review/SKILL.md.tmpl` | Düşük puanlayan boyutlar için `{{DESIGN_SETUP}}` + mockup oluşturma ekle |

### Yeniden Kullanılacak Mevcut Kod

| Kod | Konum | Kullanım Amacı |
|------|----------|----------|
| Tarama CLI deseni | `browse/src/cli.ts` | Komut gönderim mimarisi |
| `commands.ts` kayıt defteri | `browse/src/commands.ts` | Tek kaynak doğruluk deseni |
| `generateBrowseSetup()` | `scripts/resolvers/browse.ts` | `generateDesignSetup()` için şablon |
| `DESIGN_SKETCH` resolver | `scripts/resolvers/design.ts` | `DESIGN_MOCKUP` resolver için şablon |
| HostPaths sistemi | `scripts/resolvers/types.ts` | Çoklu-barındırıcı yol çözümlemesi |
| Derleme boru hattı | `package.json` derleme betiği | `bun build --compile` deseni |

### API Detayları

**Oluştur:** OpenAI Responses API ile `image_generation` aracı
```typescript
const response = await openai.responses.create({
  model: "gpt-4o",
  input: briefToPrompt(brief),
  tools: [{ type: "image_generation", size: "1536x1024", quality: "high" }],
});
// Yanıt çıktı öğelerinden görüntüyü çıkar
const imageItem = response.output.find(item => item.type === "image_generation_call");
const base64Data = imageItem.result; // base64 kodlu PNG
fs.writeFileSync(outputPath, Buffer.from(base64Data, "base64"));
```

**Yinele:** `previous_response_id` ile aynı API
```typescript
const response = await openai.responses.create({
  model: "gpt-4o",
  input: feedback,
  previous_response_id: session.lastResponseId,
  tools: [{ type: "image_generation" }],
});
```
**NOT:** `previous_response_id` üzerinden çok turlu görüntü yineleme, prototip doğrulaması gerektiren bir varsayımdır. Responses API konuşma dizisini destekler, ancak oluşturulan görüntülerin görsel bağlamını düzenleme tarzı yineleme için koruyup korumadığı belgelerde doğrulanmamıştır. **Geri dönüş:** çok turlu çalışmazsa, `iterate` orijinal kısa beti + birleştirilmiş geri bildirimi tek bir istemde yeniden oluşturmaya geri döner.

**Kontrol:** GPT-4o görüş
```typescript
const check = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [{
    role: "user",
    content: [
      { type: "image_url", image_url: { url: `data:image/png;base64,${imageData}` } },
      { type: "text", text: `Bu UI mockup'unu kontrol et. Kısa beti: ${brief}. Metin okunabilir mi? Tüm öğeler mevcut mu? Gerçek bir UI gibi mi görünüyor? Geçti veya KALDI sorunlarıyla birlikte döndür.` }
    ]
  }]
});
```

**Maliyet:** Tasarım oturumu başına ~$0.10-$0.40 (1 kahraman + 2 varyant + 1 kalite kontrolü + 1 yineleme). Her beceri çağrısında zaten olan LLM maliyetlerinin yanında ihmal edilebilir.

### Kimlik Dorulama (duman testi ile doğrulanmış)

**Codex OAuth belirteçleri görüntü oluşturma için ÇALIŞMAZ.** 2026-03-26'da test edildi: hem Images API hem de Responses API, `~/.codex/auth.json` access_token'ı "Missing scopes: api.model.images.request" hatasıyla reddeder. Codex CLI'nin de yerel imagegen yeteneği yoktur.

**Kimlik doğrulama çözümleme sırası:**
1. `~/.gstack/openai.json` dosyasını oku → `{ "api_key": "sk-..." }` (dosya izinleri 0600)
2. `OPENAI_API_KEY` ortam değişkenine geri dön
3. İkisi de yoksa → rehberli kurulum akışı:
   - Kullanıcıya söyle: "Tasarım mockup'ları görüntü oluşturma izinlerine sahip bir OpenAI API anahtarına ihtiyaç duyuyor. platform.openai.com/api-keys adresinden bir tane alın"
   - Kullanıcıdan anahtarı yapıştırmasını iste
   - 0600 izinleriyle `~/.gstack/openai.json`'a yaz
   - Anahtarın çalıştığını doğrulamak için bir duman testi çalıştır (1024x1024 test görüntüsü oluştur)
   - Duman testi geçerse, devam et. Başarısız olursa, hatayı göster ve DESIGN_SKETCH'e geri dön.
4. Kimlik doğrulama varsa ama API çağrısı başarısız olursa → DESIGN_SKETCH'e geri dön (mevcut HTML wireframe yaklaşımı). Tasarım mockup'ları aşamalı bir iyileştirmedir, asla zorlu bir gereksinim değildir.

**Yeni komut:** `$D setup` — rehberli API anahtarı kurulumu + duman testi. Anahtarı güncellemek için her zaman çalıştırılabilir.

## Prototipte Doğrulanacak Varsayımlar

1. **Görüntü kalitesi:** "Piksel-mükemmel UI mockup'ları" irdeleyici. GPT Image oluşturma güvenilir şekilde doğru metin oluşturma, hizalama ve aralık gerçek UI sadakatinde üretmeyebilir. Görüş kalite kapısı yardımcı olur, ancak "uygulamak için yeterince iyi" başarı kriterinin tam prototip doğrulaması gerektirir.
2. **Çok turlu yineleme:** `previous_response_id`'nin görsel bağlamı koruyup korumadığı kanıtlanmamıştır (API Detayları bölümüne bakın).
3. **Maliyet modeli:** Tahmini $0.10-$0.40/oturum gerçek dünyada doğrulama gerektirir.

**Prototip doğrulama planı:** Commit 1'i (çekirdek oluştur + kontrol) oluştur, farklı ekran türlerinde 10 tasarım kısa betisi çalıştır, beceri entegrasyonuna geçmeden önce çıktı kalitesini değerlendir.

## CEO Genişletme Kapsamı (/plan-ceo-review SCOPE EXPANSION ile kabul edildi)

### 1. Tasarım Hafızası + Keşif Genişliği Kontrolü
- Onaylanan mockup'lardan görsel dili DESIGN.md'ye otomatik çıkar
- DESIGN.md mevcutsa, gelecek mockup'ları kurulan tasarım diline kısıtla
- DESIGN-md yoksa (önyükleme), çeşitli yönlerde GENİŞ keşfet
- Kademeli kısıtlama: daha kurulan tasarım = daha dar keşif bandı
- Karşılaştırma panosu keşif kontrolleri ile YENİDEN OLUŞTUR bölümü alır:
  - "Tamamen farklı bir şey" (geniş keşif)
  - "___ seçeneği gibi daha fazla" (bir favori etrafında dar)
  - "Mevcut tasarımım ile eşleş" (DESIGN.md ile kısıtla)
  - Belirli yön değişiklikleri için serbest metin girişi
  - Yeniden oluştur sayfayı yeniler, ajan yeni sunum için yoklar

### 2. Mockup Fark Alma
- `$D diff --before eski.png --after yeni.png` görsel fark oluşturur
- Değişen bölgelerin vurgulandığı yan yana
- Farkları tanımlamak için GPT-4o görüş kullanır
- Kullanım: /design-review, yineleme geri bildirimi, PR incelemesi

### 3. Ekran Görüntüsünden Mockup'a Evrim
- `$D evolve --screenshot mevcut.png --brief "daha sakin yap"`
- Canlı site ekran görüntüsünü alır, nasıl görünmesi gerektiğini gösteren mockup oluşturur
- Gerçeklikten başlar, boş tuvalden değil
- /design-review eleştirisi ile görsel düzeltme önerisi arasındaki köprü

### 4. Tasarım Niyeti Doğrulama
- /design-review sırasında, onaylı mockup'ı (docs/designs/) canlı ekran görüntüsünün üzerine bindir
- Sapmayı vurgula: "X tasarladın, Y inşa ettin, işte boşluk"
- Tam döngüyü kapatır: tasarım -> uygula -> görsel olarak doğrula
- $B ekran görüntüsü + $D diff + görüş analizi birleştirir

### 5. Duyarlı Varyantlar
- `$D variants --brief "..." --viewports masaüstü,tablet,mobil`
- Birden fazla görüntü alanı boyutunda otomatik mockup oluşturma
- Karşılaştırma panosu eşzamanlı onay için duyarlı ızgara gösterir
- Duyarlı tasarımı mockup aşamasından birinci sınıf bir kaygı yapar

### 6. Tasarımdan-Koda İstem
- Karşılaştırma panosu onayından sonra, yapılandırılmış uygulama istemi otomatik oluştur
- Onaylı PNG'den görüş analizi ile renkleri, tipografiyi, düzeni çıkarır
- DESIGN.md ve HTML wireframe ile yapılandırılmış özellik olarak birleştirir
- "Onaylı tasarım" ile "ajan kodlamaya başlar" arasındaki boşluğu sıfır yorumlama açığıyla köprüler

### Gelecek Motorları (BU planın kapsamında DEĞİL)
- Magic Patterns entegrasyonu (mevcut tasarımlardan desen çıkar)
- Variant API (ne zaman gönderirlerse, çok-variasyonlu React kodu + önizleme)
- Figma MCP (çift yönlü tasarım dosyası erişimi)
- Google Stitch SDK (ücretsiz TypeScript alternatifi)

## Açık Sorular

1. Variant bir API gönderdiğinde, entegrasyon yolu nedir? (Tasarım ikili dosyasında ayrı bir motor, veya bağımsız bir Variant ikili dosyası?)
2. Magic Patterns nasıl entegre olmalı? ($D'de başka bir motor, veya ayrı bir araç?)
3. Tasarım ikili dosyası ne zaman çoklu oluşturma arka uçlarını desteklemek için bir eklenti/motor mimarisine ihtiyaç duyar?

## Başarı Kriterleri

- Bir UI fikri üzerinde `/office-hours` çalıştırmak tasarım belgesinin yanında gerçek PNG mockup'lar üretir
- `/plan-design-review` çalıştırmak düzyazı değil, mockup olarak "daha iyisinin nasıl göründüğünü" gösterir
- Mockup'lar bir geliştiricinin onlardan uygulama yapabileceği kadar iyi
- Kalite kapısı açıkça bozuk mockup'ları yakalar ve yeniden dener
- Tasarım oturumu başına maliyet $0.50'in altında kalır

## Dağıtım Planı

Tasarım ikili dosyası tarama ikili dosyası ile birlikte derlenir ve dağıtılır:
- `bun build --compile design/src/cli.ts --outfile design/dist/design`
- `./setup` ve `bun run build` sırasında derlenir
- Mevcut `~/.claude/skills/gstack/` kurulum yolu üzerinden sembolik bağlanır

## Sonraki Adımlar (Uygulama Sırası)

### Commit 0: Prototip doğrulama (altyapı inşa etmeden ÖNCE GEÇMELİ)
- 3 farklı tasarım kısa betisini GPT Image API'ne gönderen tek dosyalık prototip betiği (~50 satır)
- Doğrular: metin oluşturma kalitesi, düzen doğruluğu, görsel tutarlılık
- Çıktı "utanç verici kötü AI sanatı" ise, DUR. Yaklaşımı yeniden değerlendir.
- Bu, 8 dosya altyapısı inşa etmeden önce çekirdek varsayımı doğrulamanın en ucuz yoludur.

### Commit 1: Tasarım ikili çekirdeği (oluştur + kontrol + karşılaştır)
- `design/src/` ile cli.ts, commands.ts, generate.ts, check.ts, brief.ts, session.ts, compare.ts
- Kimlik doğrulama modülü (~/.gstack/openai.json oku, ortam değişkenine geri dön, rehberli kurulum akışı)
- `compare` komutu varyant başına geri bildirim metin alanları ile HTML karşılaştırma panosu oluşturur
- `package.json` derleme komutu (taramadan ayrı `bun build --compile`)
- `setup` betiği entegrasyonu (Codex + Kiro varlık bağlama dahil)
- Sahte OpenAI API sunucusu ile birim testleri

### Commit 2: Varyantlar + yineleme
- `design/src/variants.ts`, `design/src/iterate.ts`
- Kademeli paralel oluşturma (başlangıçlar arasında 1sn gecikme, 429'da üstel geri dönüş)
- Çok turlu oturum durumu yönetimi
- Yineleme akışı + hız sınırı işleme için testler

### Commit 3: Şablon entegrasyonu
- Mevcut `scripts/resolvers/design.ts`'e `generateDesignSetup()` + `generateDesignMockup()` ekle
- `scripts/resolvers/types.ts`'te HostPaths'e `designDir` ekle
- `scripts/resolvers/index.ts`'te DESIGN_SETUP + DESIGN_MOCKUP resolver'larını kaydet
- Codex barındırıcısı için `scripts/resolvers/preamble.ts`'e GSTACK_DESIGN ortam değişkeni dışa aktarımını ekle
- `test/gen-skill-docs.test.ts`'i güncelle (DESIGN_SKETCH test paketi)
- SKILL.md dosyalarını yeniden oluştur

### Commit 4: /office-hours entegrasyonu
- Görsel Taslak bölümünü `{{DESIGN_MOCKUP}}` ile değiştir
- Sıralı iş akışı: varyantlar oluştur → $D compare → kullanıcı geri bildirimi → DESIGN_SKETCH HTML wireframe
- Onaylı mockup'ı docs/designs/'e kaydet (sadece onaylanan, keşifler değil)

### Commit 5: /plan-design-review entegrasyonu
- `{{DESIGN_SETUP}}` ve düşük puanlayan boyutlar için mockup oluşturma ekle
- "10/10'un nasıl göründüğü" mockup karşılaştırması

### Commit 6: Tasarım Hafızası + Keşif Genişliği Kontrolü (CEO genişletme)
- Mockup onayından sonra, GPT-4o görüş ile görsel dili çıkar
- Çıkarılan renkler, tipografi, aralık, düzen desenleri ile DESIGN.md yaz/güncelle
- DESIGN.md mevcutsa, tüm gelecek mockup istemlerine kısıtlama bağlamı olarak besle
- Karşılaştırma panosu HTML'sine YENİDEN OLUŞTUR bölümü ekle (çini taşları + serbest metin + yenileme döngüsü)
- Kısa beti oluşturmada kademeli kısıtlama mantığı

### Commit 7: Mockup Fark Alma + Tasarım Niyeti Doğrulama (CEO genişletme)
- `$D diff` komutu: iki PNG alır, farkları tanımlamak için GPT-4o görüş kullanır, katman oluşturur
- `$D verify` komutu: canlı siteyi $B ile ekran görüntüsü alır, docs/designs/'ten onaylı mockup ile fark alır
- /design-review şablonuna entegrasyon: onaylı mockup mevcut olduğunda otomatik doğrula

### Commit 8: Ekran Görüntüsünden Mockup'a Evrim (CEO genişletme)
- `$D evolve` komutu: ekran görüntüsü + kısa beti alır, "nasıl görünmesi gerektiği" mockup'ı oluşturur
- Ekran görüntüsünü referans görüntüsü olarak GPT Image API'sine gönderir
- /design-review entegrasyonu: "İşte düzeltmenin nasıl görünmesi gerektiği" görsel öneriler

### Commit 9: Duyarlı Varyantlar + Tasarımdan-Koda İstem (CEO genişletme)
- Çoklu boyut oluşturma için `$D variants` üzerinde `--viewports` bayrağı
- Karşılaştırma panosu duyarlı ızgara düzeni
- Onaydan sonra yapılandırılmış uygulama istemi otomatik oluştur
- Onaylı PNG'den görüş analizi ile renkleri, tipografiyi, düzeni istem için çıkar

## Görev

Variant'a bir API inşa etmesini söyle. Yatırımcıları olarak: "AI ajanlarının görsel tasarımları programlı olarak oluşturduğu bir iş akışı inşa ediyorum. GPT Image API bugün çalışıyor — ama Variant'ı tercih ederim çünkü çok-variasyon yaklaşımı tasarım keşfi için daha iyi. Bir API uç noktası gönder: istem içeri, React kodu + önizleme görüntüsü dışarı. İlk entegrasyon partneriniz olacağım."

## Doğrulama

1. `bun run build` `design/dist/design` ikili dosyasını derler
2. `$D generate --brief "Geliştirici aracı için açılış sayfası" --output /tmp/test.png` gerçek bir PNG üretir
3. `$D check --image /tmp/test.png --brief "Açılış sayfası"` GEÇTİ/KALDI döndürür
4. `$D variants --brief "..." --count 3 --output-dir /tmp/variants/` 3 PNG üretir
5. Bir UI fikri üzerinde `/office-hours` çalıştırmak satır içi mockup'lar üretir
6. `bun test` geçer (beceri doğrulama, gen-skill-docs)
7. `bun run test:evals` geçer (E2E testleri)

## Düşünme tarzınız hakkında fark ettiklerim

- Düzine metin açıklamaları ve ASCII sanat hakkında "bu tasarım değil" dediniz. Bu bir tasarımcının içgüdüsü — bir şeyi açıklamakla göstermek arasındaki farkı biliyorsunuz. AI araçları inşa eden çoğu insan bu boşluğu fark etmiyor çünkü hiçbir zaman tasarımcı değildi.
- /office-hours'ı önce önceliklendirdiniz — yukarı akış kaldıraç noktası. Beyin fırtınası gerçek mockup'lar üretirse, her aşağı akış becerisi (/plan-design-review, /design-review) düzyazıyı yeniden yorumlamak yerine görsel bir esere başvurabilir.
- Variant'ı finanse ettiniz ve hemen "bir API'leri olmalı" düşündünüz. Bu yatırımcı-kullanıcı düşüncesi — şirketi değerlendirmiyorsunuz, ürünlerinin iş akışınıza nasıl uyduğunu tasarlıyorsunuz.
- Codex katılım öncülüne meydan okuduğunda, hemen kabul ettiniz. Ego savunması yok. Doğru cevaba giden en hızlı yol bu.

## Özellik İnceleme Sonuçları

Belge 1 tur çapraz incelemeden geçti. 11 sorun yakalandı ve düzeltildi.
Kalite puanı: 7/10 → düzeltmelerden sonra tahmini 8.5/10.

Düzeltilen sorunlar:
1. OpenAI SDK bağımlılığı bildirildi
2. Görüntü verisi çıkarma yolu belirtilti (response.output öğe şekli)
3. --check ve --retry bayrakları resmi olarak komut kayıt defterine kaydedildi
4. Kısa beti giriş modları belirlendi (düz metin vs JSON dosyası)
5. Resolver dosya çelişkisi düzeltildi (mevcut design.ts'e ekle)
6. HostPaths Codex ortam değişkeni kurulumu not edildi
7. "Taramayı yansıtır" "derleme/dağıtım desenini paylaşır" olarak yeniden çerçevelendi
8. Oturum durumu belirlendi (kimlik oluşturma, keşif, temizleme)
9. "Piksel-mükemmel" prototip doğrulaması gerektiren bir varsayım olarak işaretlendi
10. Çok turlu yineleme kanıtlanmamış olarak işaretlendi, geri dönüş planı ile
11. $D keşif bash bloğu DESIGN_SKETCH'e geri dönüş ile tamamen belirlendi

## Mühendislik İnceleme Tamamlama Özeti

- Adım 0: Kapsam Meydan Okuması — kapsam olduğu gibi kabul edildi (tam ikili dosya, kullanıcı azaltma önerisini geçersiz kıldı)
- Mimari İnceleme: 5 sorun bulundu (openai bağımlılık ayrımı, zarif bozulma, çıktı dizini yapılandırması, kimlik doğrulama modeli, güven sınırı)
- Kod Kalitesi İnceleme: 1 sorun bulundu (8 dosya vs 5, 8 tutuldu)
- Test İnceleme: diyagram üretildi, 42 boşluk tanımlandı, test planı yazıldı
- Performans İnceleme: 1 sorun bulundu (kademeli başlangıç ile paralel varyantlar)
- KAPSAMDA DEĞİL: Google Stitch SDK entegrasyonu, Figma MCP, Variant API (ertelendi)
- Zaten var olan: tarama CLI deseni, DESIGN_SKETCH resolver, HostPaths sistemi, gen-skill-docs boru hattı
- Dış ses: 4 geçiş (Claude yapılandırılmış 12 sorun, Codex yapılandırılmış 8 sorun, Claude çapraz 1 ölümcül kusur, Codex çapraz 1 ölümcül kusur). Temel içgörü: sıralı PNG→HTML iş akışı "opak raster" ölümcül kusurunu çözdü.
- Başarısızlık modları: 0 kritik boşluk (tüm tanımlanan başarısızlık modlarının hata işleme + planlanmış testleri var)
- Göl Puanı: 7/7 öneri tam seçeneği seçti

## GSTACK İNCELEME RAPORU

| İnceleme | Tetikleyici | Neden | Çalışmalar | Durum | Bulgular |
|--------|---------|-----|------|--------|----------|
| Office Hours | `/office-hours` | Tasarım beyin fırtınası | 1 | BİTTİ | 4 öncül, 1 revize edildi (Codex: katılım→varsayılan açık) |
| CEO İncelemesi | `/plan-ceo-review` | Kapsam & strateji | 1 | TEMİZ | GENİŞLETME: 6 önerildi, 6 kabul edildi, 0 ertelendi |
| Müh. İnceleme | `/plan-eng-review` | Mimari & testler (gerekli) | 1 | TEMİZ | 7 sorun, 0 kritik boşluk, 4 dış ses |
| Tasarım İncelemesi | `/plan-design-review` | UI/UX boşlukları | 1 | TEMİZ | puan: 2/10 -> 8/10, 5 karar alındı |
| Dış Ses | yapılandırılmış + çapraz | Bağımsız meydan okuma | 4 | BİTTİ | Sıralı PNG->HTML iş akışı, güven sınırı not edildi |

**CEO GENİŞLETMELERİ:** Tasarım Hafızası + Keşif Genişliği, Mockup Fark Alma, Ekran Görüntüsü Evrimi, Tasarım Niyeti Doğrulama, Duyarlı Varyantlar, Tasarımdan-Koda İstem.
**TASARIM KARARLARI:** Tek sütun tam genişlik düzeni, varyant başına "Buna benzer daha fazla", açık radyo Seçim, yumuşak solma yeniden oluşturma, iskelet yükleme durumları.
**ÇÖZÜLmemiş:** 0
**KARAR:** CEO + MÜH + TASARIM TEMİZLENDİ. Uygulamaya hazır. Commit 0 ile başlayın (prototip doğrulama).