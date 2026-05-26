# Tasarım İnceleme Kontrol Listesi (Lite)

> **DESIGN_METHODOLOGY'nin alt kümesi** — buraya öğe eklerken, aynı zamanda `scripts/gen-skill-docs.ts` dosyasındaki `generateDesignMethodology()` fonksiyonunu da güncelleyin ve bunun tersi de geçerli.

## Talimatlar

Bu kontrol listesi **difteki kaynak koduna** uygulanır — oluşturulan çıktıya değil. Değiştirilen her ön uç dosyasını (yalnızca dif parçalarını değil, tam dosyayı) okuyun ve anti-desenleri işaretleyin.

**Tetikleyici:** Bu kontrol listesini yalnızca dif ön uç dosyalarına dokunuyorsa çalıştırın. Algılamak için `gstack-diff-scope` kullanın:

```bash
source <(~/.claude/skills/gstack/bin/gstack-diff-scope <base> 2>/dev/null)
```

`SCOPE_FRONTEND=false` ise, tüm tasarım incelemesini sessizce atlayın.

**DESIGN.md kalibrasyonu:** Depo kökünde `DESIGN.md` veya `design-system.md` varsa, önce onu okuyun. Tüm bulgular projenin belirtilen tasarım sistemine göre kalibre edilir. DESIGN.md'de açıkça kabul görmüş desenler işaretlenmez. DESIGN.md yoksa, evrensel tasarım ilkelerini kullanın.

---

## Güven Seviyeleri

Her öğe bir algılama güven seviyesi ile etiketlenir:

- **[YÜKSEK]** — Grep/desen eşleşmesi ile güvenilir şekilde algılanabilir. Kesin bulgular.
- **[ORTA]** — Desen toplama veya buluşsal yöntemle algılanabilir. Bulgular olarak işaretleyin ancak biraz gürültü bekleyin.
- **[DÜŞÜK]** — Görsel niyeti anlamayı gerektirir. Şöyle sunun: "Olası sorun — görsel olarak doğrulayın veya /design-review çalıştırın."

---

## Sınıflandırma

**OTOMATİK-DÜZELT** (yalnızca mekanik CSS düzeltmeleri — YÜKSEK güven, tasarım yargısı gerekmez):
- Değiştirmeden `outline: none` → `outline: revert` veya `&:focus-visible { outline: 2px solid currentColor; }` ekleyin
- Yeni CSS'de `!important` → kaldırın ve özgüllüğü düzeltin
- Gövde metninde `font-size` < 16px → 16px'ye yükseltin

**SOR** (diğer her şey — tasarım yargısı gerektirir):
- Tüm AI çöp bulguları, tipografi yapısı, boşluk seçimleri, etkileşim durumu boşlukları, DESIGN.md ihlalleri

**DÜŞÜK güven öğeleri** → şöyle sunun: "Olası: [açıklama]. Görsel olarak doğrulayın veya /design-review çalıştırın." Asla OTOMATİK-DÜZELT yapmayın.

---

## Çıktı Formatı

```
Tasarım İncelemesi: N sorun (X otomatik-düzeltilebilir, Y girdi-gerekli, Z olası)

**OTOMATİK-DÜZELTİLDİ:**
- [dosya:satır] Sorun → düzeltme uygulandı

**GİRDİ-GEREKLİ:**
- [dosya:satır] Sorun açıklaması
  Önerilen düzeltme: önerilen düzeltme

**OLASI (görsel olarak doğrulayın):**
- [dosya:satır] Olası sorun — /design-review ile doğrulayın
```

İsteğe bağlı: `test_stub` — projenin test çerçevesini kullanarak bu bulgu için iskelet test kodu.

Sorun bulunamazsa: `Tasarım İncelemesi: Sorun bulunamadı.`

Ön uç dosyası değiştirilmemişse: sessizce atlayın, çıktı yok.

---

## Kategoriler

### 1. AI Çöpü Algılama (6 öğe) — en yüksek öncelik

Bunlar, saygın bir stüdyodaki hiçbir tasarımcının göndermeyeceği AI tarafından oluşturulmuş UI'nin ayırt edici işaretleridir.

- **[ORTA]** Mor/mor/çivit gradyan arka planlar veya mordan mor renk şemaları. `#6366f1`–`#8b5cf6` aralığındaki değerlere sahip `linear-gradient` veya mor/mor'a çözünen CSS özel özelliklerini arayın.

- **[DÜŞÜK]** 3 sütunlu özellik ızgarası: renkli-dairedeki-simge + kalın başlık + 2 satırlık açıklama, simetrik olarak 3 kez tekrarlanan. Tam olarak 3 çocuk içeren ve her biri dairesel bir eleman + başlık + paragraf içeren bir grid/flex kabını arayın.

- **[DÜŞÜK]** Bölüm dekorasyonu olarak renkli dairelerdeki simgeler. `border-radius: 50%` + simgeler için dekoratif kap olarak kullanılan bir arka plan rengine sahip elemanları arayın.

- **[YÜKSEK]** Ortalanmış her şey: tüm başlıklarda, açıklamalarda ve kartlarda `text-align: center`. `text-align: center` yoğunluğu için grep yapın — metin kaplarının >%60'ı ortalamayı kullanıyorsa, işaretleyin.

- **[ORTA]** Her elemanda tekdüze kabarık kenar-yarıçapı: kartlara, düğmelere, girdilere, kaplara aynı büyük yarıçap (16px+) tekdüze olarak uygulanmış. `border-radius` değerlerini toplayın — >%80'i aynı ≥16px değerini kullanıyorsa, işaretleyin.

- **[ORTA]** Genel hero kopyası: "[X]'e Hoş Geldiniz", "...nın gücünü açın", "...için hepsi bir arada çözümünüz", "...nızı devrimleştirin", "...akışınızı kolaylaştırın". HTML/JSX içeriğinde bu desenleri grep'leyin.

### 2. Tipografi (4 öğe)

- **[YÜKSEK]** Gövde metninde `font-size` < 16px. `body`, `p`, `.text` veya temel stiller üzerinde `font-size` bildirimlerini grep'leyin. 16px'in altındaki (veya temel 16px olduğunda 1rem'in altındaki) değerler işaretlenir.

- **[YÜKSEK]** Difte 3'ten fazla yazı tipi ailesi tanıtıldı. Ayrı `font-family` bildirimlerini sayın. Değiştirilen dosyalarda >3 benzersiz aile görünüyorsa işaretleyin.

- **[YÜKSEK]** Başlık hiyerarşisi düzeyleri atlıyor: aynı dosya/bileşende `h2` olmadan `h1` ardından `h3`. Başlık etiketleri için HTML/JSX'i kontrol edin.

- **[YÜKSEK]** Kara listedeki yazı tipleri: Papyrus, Comic Sans, Lobster, Impact, Jokerman. Bu isimler için `font-family` grep'leyin.

### 3. Boşluk & Düzen (4 öğe)

- **[ORTA]** DESIGN.md bir boşluk ölçeği belirttiğinde, 4px veya 8px ölçeğinde olmayan keyfi boşluk değerleri. `margin`, `padding`, `gap` değerlerini belirtilen ölçeğe karşı kontrol edin. Yalnızca DESIGN.md bir ölçek tanımladığında işaretleyin.

- **[ORTA]** Duyarlı işlem olmaksızın sabit genişlikler: `max-width` veya `@media` kesme noktaları olmaksızın kaplar üzerinde `width: NNNpx`. Mobilde yatay kaydırma riski.

- **[ORTA]** Metin kaplarında eksik `max-width`: satırların >75 karakter olmasına izin veren `max-width` ayarı olmayan gövde metni veya paragraf kapları. Metin sarmalayıcılarında `max-width` olup olmadığını kontrol edin.

- **[YÜKSEK]** Yeni CSS kurallarında `!important`. Eklenen satırlarda `!important` grep'leyin. Neredeyse her zaman düzgün şekilde düzeltilmesi gereken bir özgüllük kaçış kapısıdır.

### 4. Etkileşim Durumları (3 öğe)

- **[ORTA]** Hover/focus durumları eksik olan etkileşimli elemanlar (düğmeler, bağlantılar, girdiler). Yeni etkileşimli eleman stilleri için `:hover` ve `:focus` veya `:focus-visible` pseudo-sınıflarının mevcut olup olmadığını kontrol edin.

- **[YÜKSEK]** Değiştirme odak göstergesi olmaksızın `outline: none` veya `outline: 0`. `outline:\s*none` veya `outline:\s*0` grep'leyin. Bu klavye erişilebilirliğini kaldırır.

- **[DÜŞÜK]** Etkileşimli elemanlarda < 44px dokunma hedefleri. Düğmeler ve bağlantılarda `min-height`/`min-width`/`padding` kontrol edin. Birden fazla özellikten etkili boyutu hesaplamayı gerektirir — yalnızca koddan düşük güven.

### 5. DESIGN.md İhlalleri (3 öğe, koşullu)

Yalnızca `DESIGN.md` veya `design-system.md` mevcutsa uygulayın:

- **[ORTA]** Belirtilen palettte olmayan renkler. Değiştirilen CSS'teki renk değerlerini DESIGN.md'de tanımlanan paletle karşılaştırın.

- **[ORTA]** Belirtilen tipografi bölümünde olmayan yazı tipleri. `font-family` değerlerini DESIGN.md'nin yazı tipi listesiyle karşılaştırın.

- **[ORTA]** Belirtilen ölçekte olmayan boşluk değerleri. `margin`/`padding`/`gap` değerlerini DESIGN.md'nin boşluk ölçeğiyle karşılaştırın.

---

## Baskılar

Bunları işaretlemeyin:
- DESIGN.md'de bilinçli seçimler olarak açıkça belgelenen desenler
- Üçüncü taraf/satıcı CSS dosyaları (node_modules, vendor dizinleri)
- CSS sıfırlama veya normalize stil sayfaları
- Test fikstür dosyaları
- Oluşturulan/minify edilmiş CSS