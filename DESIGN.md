# Tasarım Sistemi — gstack

## Ürün Bağlamı
- **Bunun ne olduğu:** gstack için topluluk web sitesi — Claude Code'u sanal bir mühendislik takımına dönüştüren bir CLI aracı
- **Kime yönelik:** gstack'i keşfeden geliştiriciler, mevcut topluluk üyeleri
- **Alan/endüstri:** Geliştirici araçları (akranlar: Linear, Raycast, Warp, Zed)
- **Proje türü:** Topluluk dashboard'u + pazarlama sitesi

## Estetik Yön
- **Yön:** Endüstriyel/Faydacı — önce işlev, veri-yoğun, kişilik fontu olarak monospace
- **Süsleme seviyesi:** Kasıtlı — yüzeylerde maddelik için ince gürültü/doku dokusu
- **Ruh hali:** Zanaatına özen gösteren biri tarafından inşa edilmiş ciddi araç. Sıcak, soğuk değil. CLI mirası markanın ta kendisi.
- **Referans siteler:** formulae.brew.sh (rakip, ama bizimki canlı ve interaktif), Linear (karanlık + ölçülü), Warp (sıcak aksanlar)

## Tipografi
- **Display/Hero:** Satoshi (Black 900 / Bold 700) — geometrik ama sıcak, ayırt edici harf formları (küçük 'a' ve 'g'). Inter değil, Geist değil. Fontshare CDN'den yüklenir.
- **Gövde:** DM Sans (Regular 400 / Medium 500 / Semibold 600) — temiz, okunabilir, geometrik display'den biraz daha samimi. Google Fonts'tan yüklenir.
- **UI/Etiketler:** DM Sans (gövde ile aynı)
- **Veri/Tablolar:** JetBrains Mono (Regular 400 / Medium 500) — kişilik fontu. Tabular-nums destekler. Monospace belirgin olmalı, kod bloklarına gizlenmemeli. Google Fonts'tan yüklenir.
- **Kod:** JetBrains Mono
- **Yükleme:** DM Sans + JetBrains Mono için Google Fonts, Satoshi için Fontshare. `display=swap` kullanın.
- **Ölçek:**
  - Hero: 72px / clamp(40px, 6vw, 72px)
  - H1: 48px
  - H2: 32px
  - H3: 24px
  - H4: 18px
  - Gövde: 16px
  - Küçük: 14px
  - Başlık: 13px
  - Mikro: 12px
  - Nano: 11px (JetBrains Mono etiketler)

## Renk
- **Yaklaşım:** Ölçülü — amber aksan nadir ve anlamlı. Dashboard verisi renk alır; krom nötr kalır.
- **Birincil (karanlık mod):** amber-500 #F59E0B — sıcak, enerjik, "terminal imleci" olarak okunur
- **Birincil (aydınlık mod):** amber-600 #D97706 — beyaz arka planlara karşı kontrast için daha koyu
- **Birincil metin aksanı (karanlık mod):** amber-400 #FBBF24
- **Birincil metin aksanı (aydınlık mod):** amber-700 #B45309
- **Nötrler:** Soğuk çinko griler
  - zinc-50: #FAFAFA (en açık)
  - zinc-400: #A1A1AA
  - zinc-600: #52525B
  - zinc-800: #27272A
  - Yüzey (karanlık): #141414
  - Taban (karanlık): #0C0C0C
  - Yüzey (aydınlık): #FFFFFF
  - Taban (aydınlık): #FAFAF9
- **Anlamsal:** başarı #22C55E, uyarı #F59E0B, hata #EF4444, bilgi #3B82F6
- **Karanlık mod:** Varsayılan. Siyaha yakın taban (#0C0C0C), #141414'de yüzey kartları, #262626'da kenarlıklar.
- **Aydınlık mod:** Sıcak taş taban (#FAFAF9), beyaz yüzey kartları, taş kenarlıklar (#E7E5E4). Amber aksan kontrast için amber-600'ya kayar.

## Aralık
- **Temel birim:** 4px
- **Yoğunluk:** Rahat — sıkışık değil (Bloomberg Terminal değil), geniş değil (pazarlama sitesi değil)
- **Ölçek:** 2xs(2px) xs(4px) sm(8px) md(16px) lg(24px) xl(32px) 2xl(48px) 3xl(64px)

## Düzen
- **Yaklaşım:** Dashboard için ızgara-disiplinli, açılış sayfası için editöryal hero
- **Izgara:** lg+'da 12 sütun, mobilde 1 sütun
- **Maksimum içerik genişliği:** 1200px (6xl)
- **Kenar yuvarlaklığı:** sm:4px, md:8px, lg:12px, full:9999px
  - Kartlar/paneller: lg (12px)
  - Düğmeler/girişler: md (8px)
  - Rozetler/pill'ler: full (9999px)
  - Skill çubukları: sm (4px)

## Hareket
- **Yaklaşım:** Minimal-işlevsel — sadece anlayışı kolaylaştıran geçişler. Dashboard'un canlı akışı hareketin ta kendisidir.
- **Hafifleme:** giriş(ease-out / cubic-bezier(0.16,1,0.3,1)) çıkış(ease-in) hareket(ease-in-out)
- **Süre:** mikro(50-100ms) kısa(150ms) orta(250ms) uzun(400ms)
- **Animasyonlu öğeler:** canlı akış noktası nabzı (2s sonsuz), skill çubuğu dolması (600ms ease-out), hover durumları (150ms)

## Gürültü Dokusu
Maddelik için tüm sayfaya ince bir gürültü katmanı uygulayın:
- Karanlık mod: opaklık 0.03
- Aydınlık mod: opaklık 0.02
- SVG feTurbulence filtresini body::after üzerinde CSS background-image olarak kullanın
- pointer-events: none, position: fixed, z-index: 9999

## Kararlar Günlüğü
| Tarih | Karar | Gerekçe |
|-------|-------|---------|
| 2026-03-21 | İlk tasarım sistemi | /design-consultation tarafından oluşturuldu. Endüstriyel estetik, sıcak amber aksan, Satoshi + DM Sans + JetBrains Mono. |
| 2026-03-21 | Aydınlık mod amber-600 | amber-500 beyaz arka plana karşı çok parlak/soluk; amber-700 çok kahverengi/umbersı. amber-600 tatlı nokta. |
| 2026-03-21 | Gürültü dokusu | Düz karanlık yüzeylere maddelik katar. "Genel SaaS şablonu" tektipliliğini önler. |