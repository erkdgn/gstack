# GStack Browser V0 — Yapay Zeka Doğal Geliştirme Tarayıcısı

**Tarih:** 2026-03-30
**Yazar:** Garry Tan + Claude Code
**Durum:** Aşama 1a gönderildi, Aşama 1b devam ediyor
**Dal:** garrytan/gstack-as-browser

## Tez

Başka her AI tarayıcısı (Atlas, Dia, Comet, Chrome Auto Browse) bir tüketici tarayıcısıyla başlayıp üzerine AI ekliyor. GStack Browser bunu tersine çeviriyor. Claude Code'u çalışma zamanı olarak alıyor ve ona bir tarayıcı görüntü alanı veriyor.

Aracı birinci sınıf vatandaştır. Tarayıcı tuvaldir. Skill'ler birinci sınıf yeteneklerdir. "AI yardımıyla bir tarayıcı kullanmıyorsunuz." Web'i görebilen ve etkileşime girebilen bir AI kullanıyorsunuz.

Bu, post-IDE çağı için IDE'dir. Kod terminalde yaşar. Ürün tarayıcıda yaşar. AI her ikisinde aynı anda çalışır. Cursor'un metin düzenleyicileri için yaptığını GStack Browser tarayıcı için yapıyor.

## Bugün Ne (Aşama 1a, gönderildi)

gstack kenar çubuğu eklentisinin içine gömüldüğü Playwright'ın Chromium'unu saran çift tıklanabilir bir macOS .app. Açtığınızda Claude Code ekranınızı görebilir, sayfalarda gezinebilir, formları doldurabilir, ekran görüntüleri alabilir, CSS inceleyebilir, katmanları temizleyebilir ve herhangi bir gstack skill'ini çalıştırabilir. Hepsi bir terminale dokunmadan.

```
GStack Browser.app (389MB, 189MB DMG)
├── Derlenmiş browse ikili dosyası (58MB) — CLI + HTTP sunucu
├── Chrome eklentisi (172KB) — kenar çubuğu, aktivite akışı, denetçi
├── Playwright'ın Chromium'u (330MB) — gerçek tarayıcı
└── Başlatıcı betiği — proje dizinini bağlar, çevre değişkenlerini ayarlar
```

Başlat → Chromium kenar çubuğu ile açılır → eklenti browse sunucusuna otomatik bağlanır → aracı ~5 saniyede hazır.

## Ne Olacak

### Aşama 1b: Geliştirici UX (sonraki)

**Komut Paleti (Cmd+K):** İmza etkileşimi. Bulanık filtrelenmiş bir skill seçici açar. QA testi başlatmak için "/qa", hata ayıklamak için "/investigate", PR oluşturmak için "/ship" yazın. Skill'ler browse sunucusundan getirilir, sabit kodlanmaz. Palet her şeye giriş noktasıdır.

**Hızlı Ekran Görüntüsü (Cmd+Shift+S):** Mevcut görüntü alanını yakalar ve "Ne görüyorsun?" bağlamıyla kenar çubuğu sohbetine aktarır. AI ekran görüntüsünü analiz eder ve size eyleme geçirilebilir geri bildirim verir. Bir tuş vuruşunda görsel hata raporları.

**Durum Çubuğu:** Her sayfanın altında kalıcı bir 30px çubuk. Aracı durumunu (boşta/düşünüyor), çalışma alanı adını, mevcut dalı ve otomatik algılanan geliştirme sunucularını gösterir. Bir geliştirme sunucusu hapına tıklayarak gezinebilirsiniz. AI'nın ne yaptığına dair her zaman görünen bağlam.

**Otomatik Algılanan Geliştirme Sunucuları:** Başlatmada yaygın bağlantı noktalarını tarar (3000, 3001, 4200, 5173, 5174, 8000, 8080). Tam olarak bir sunucu bulunursa otomatik olarak ona gider. Durum çubuğundaki geliştirme sunucusu hapları tek tıkla geçiş için.

### Aşama 2: BoomLooper Entegrasyonu

Kenar çubuğu yerel bir `claude -p` alt süreci yerine BoomLooper'ın Phoenix/Elixir API'lerine bağlanır. BoomLooper şunları sağlar:

- **Çoklu aracı orkestrasyonu.** Her biri kendi tarayıcı sekmesinde 5 aracı paralel olarak başlat. Biri QA çalıştırır, biri tasarım incelemesi yapar, biri regresyonları izler.
- **Docker altyapısı.** Her arıcı yalıtılmış bir kapsayıcı alır. Kapsayıcı içindeki tarayıcı geliştirme sunucusunu test eder. Bağlantı noktası çakışması yok, durum sızıntısı yok.
- **Oturum kalıcılığı.** Aracı konuşmaları tarayıcı yeniden başlatmalarından sağ çıkar. Kaldığınız yerden devam edin.
- **Takım görünürlüğü.** Takım arkadaşlarınız aracılarınızın ne yaptığını gerçek zamanlı olarak görebilir. Çift programlama gibi, ama çift 5 AI arıcısı ve siz şefsiniz.

### Aşama 3: Browse'un BoomLooper Aracı Olması

browse ikili dosyası BoomLooper'da bir MCP aracı olur. Docker kapsayıcılarındaki arıcılar, geliştirme sunucularını test etmek, ekran görüntüleri almak, formları doldurmak ve dağıtımları doğrulamak için browse komutlarını kullanır. Çapraz platform derlemesi (linux-arm64/x64) gerekli.

### Aşama 4: Chromium Çatalı (tetikleyici kontrollü)

Eklenti yan paneli sert API sınırlarına çarptığında, GStack Browser dış kullanıcılara gönderildiğinde, derleme altyapısı mevcut olduğunda ve işletme bakımı haklı çıkardığında: Chromium'u çatallayın. Brave'ın `chromium_src` override deseni, CC destekli 6 haftalık yeniden tabanlama (2-4 saat CC ile vs 1-2 hafta insan). ~20-30 dosya değiştirildi.

### Aşama 5: Yerel Kabuk

Yerel kenar çubuğu, yalıtılmış Chromium hizmeti ile SwiftUI/AppKit uygulama kabuğu. Tam platform entegrasyonu. Chromium çatalı yerel bir kenar çubuğu içeriyorsa Aşama 5 tarafından geçersiz kılınabilir.

## Vizyon: Bir AI Tarayıcısı Ne Yapabilir

### 1. Gördüğünüzü Görür

Tarayıcı AI'nın gözleridir. Ekran görüntüleri üzerinden değil (yapabilir ama), DOM erişimi, CSS incelemesi, ağ izleme ve erişilebilirlik ağacı ayrıştırma üzerinden. AI pikselleri değil, sayfa yapısını anlar.

**Bugün:** `snapshot` komutu herhangi bir sayfanın erişilebilirlik ağacı temsilini döndürür. AI her düğmeyi, bağlantıyı, form alanını ve metin öğesini "görebilir". Öğe referansları (`@e1`, `@e2`) AI'nın tıklamasını, doldurmasını ve etkileşime girmesini sağlar.

**Sonraki:** Gerçek zamanlı sayfa gözlemi. AI bir sayfanın ne zaman değiştiğini, konsolda bir hatanın ne zaman göründüğünü, bir ağ isteğinin ne zaman başarısız olduğunu fark eder. Sorulmadan proaktif hata ayıklama.

**Gelecek:** Görsel anlama. AI önce/sonra ekran görüntülerini karşılaştırarak görsel regresyonları yakalar. Piksel düzeyinde tasarım incelemesi. "Bu düğme 3px sola taşındı ve yazı tipi 14px'den 13px'e değişti."

### 2. Gördüğüne Göre Hareket Eder

Sayfaları okumak değil, onlara bir insan kullanıcının yapacağı gibi etkileşime girmek.

**Bugün:** Tıkla, doldur, seç, üzerine gel, yaz, kaydır, dosya yükle, diyalogları yönet, gezin, sekmeleri yönet. Hepsi browse sunucusu üzerinden basit komutlarla.

**Sonraki:** Çok adımlı kullanıcı akışları. "Giriş yap, ayarlara git, saat dilimini değiştir, onay mesajını doğrula." AI her adımda doğrulama ile komutları zincirler.

**Gelecek:** Otonom QA aracısı. "Bu sayfadaki her bağlantıyı test et. Her formu doldur. Bozmaya çalış." AI bir betim olmadan kapsamlı etkileşim testi çalıştırır. İnsanların düşünmediği kombinasyonları denediği için insan testçinin kaçıracağı hataları bulur.

### 3. Gezinirken Kod Yazar

Bu temel farklılaştırıcıdır. AI tarayıcıdaki hatayı görebilir VE kodu aynı anda düzeltebilir.

**Bugün:** Kenar çubuğu sohbeti Claude Code'a bağlanır. "Bu düğme hizasız" dersiniz ve AI CSS'yi okur, sorunu tanımlar ve bir düzeltme önerir. `/design-review` skill'i ekran görüntüleri alır, görsel sorunları tanımlar ve önce/sonra kanıtıyla düzeltmeleri işler.

**Sonraki:** Canlı yeniden yükleme döngüsü. AI CSS/HTML düzenler, tarayıcı otomatik olarak yeniden yükler, AI düzeltmeyi görsel olarak doğrular. Basit görsel düzeltmeler için insan döngüde değil. "Bu sayfadaki tüm boşluk sorunlarını düzelt" 30 saniyelik bir görev olur.

**Gelecek:** Tam yığın hata ayıklama. AI tarayıcıda bir 500 hatası görür, sunucu günlüklerini okur, başarısız satıra kadar izler, düzeltmeyi yazar ve tarayıcıda doğrular. Bir komut: "Bu sayfa bozuk. Düzelt."

### 4. Tüm Yığını Anlar

Tarayıcı yalnızca bir görüntü alanı değil. Uygulamanın sağlığına bir penceredir.

**Bugün:**
- Konsol günlüğü yakalama — her `console.log`, `console.error` ve uyarı
- Ağ isteği izleme — her XHR, fetch, websocket ve statik varlık
- Performans metrikleri — Core Web Vitals, kaynak zamanlama, boyama olayları
- Çerez ve depolama incelemesi — localStorage, sessionStorage okuma ve yazma
- CSS incelemesi — hesaplanan stiller, kutu modeli, kural basamaklaması

**Sonraki:**
- Ağ isteği yeniden oynatma — "bu başarısız isteği farklı parametrelerle yeniden oynat"
- Performans regresyon algılama — "bu sayfa dünden 200ms yavaş"
- Bağımlılık denetimi — "bu sayfa 47 üçüncü taraf betik yüküyor"
- Erişilebilirlik denetimi — "bu formun etiketi yok, bu renkler kontrastı geçemiyor"

**Gelecek:**
- Tam uygulama telemetrisi — gerçek zamanlı CPU, bellek, GPU kullanımı
- Çapraz tarayıcı testi — Chrome, Firefox, Safari'de aynı test pakiti
- Gerçek kullanıcı izleme korelasyonu — "bu hata üretim kullanıcılarının %12'sini etkiliyor"

### 5. Çalışma Alanı Modeli

Tarayıcı IS çalışma alanıdır. Bir çalışma alanında bir sekme değil. Çalışma alanının kendisi.

**Bugün:** Her tarayıcı oturumu bir proje dizinine bağlıdır. Kenar çubuğu mevcut dalı gösterir. Durum çubuğu algılanan geliştirme sunucularını gösterir.

**Sonraki:** Çoklu proje desteği. Tarayıcıyı kapatmadan projeler arasında geçiş yap. Her proje kendi sekme setini, kendi arıcısını, kendi bağlamını alır. VSCode çalışma alanları gibi, ama tarayıcı için.

**Gelecek:** Takım çalışma alanları. Birden fazla geliştirici bir tarayıcı çalışma alanını paylaşır. Birbirlerinin arıcılarının çalıştığını görün. Bir kişinin gezindiği ve diğerinin AI'nın gerçek zamanlı olarak şeyleri düzelttiğini izlediği işbirlikçi hata ayıklama.

### 6. Skill'ler Tarayıcı Yetenekleri Olarak

Her gstack skill'i bir tarayıcı yeteneği olur.

| Skill | Tarayıcı Yeteneği |
|-------|-------------------|
| `/qa` | Her sayfayı test et, hataları bul, düzelt, düzeltmeleri doğrula |
| `/design-review` | Ekran görüntüsü → analiz et → CSS'yi düzelt → tekrar ekran görüntüsü al |
| `/investigate` | Tarayıcıda hatayı gör → koda izle → düzelt → doğrula |
| `/benchmark` | Sayfa performansını ölç → regresyonları algıla → uyar |
| `/canary` | Dağıtılan siteyi izle → periyodik olarak ekran görüntüsü al → değişikliklerde uyar |
| `/ship` | Testleri çalıştır → diff'i incele → PR oluştur → tarayıcıda dağıtımı doğrula |
| `/cso` | Gerçek tarayıcıda XSS, açık yönlendirmeler, clickjacking için sayfayı denetle |
| `/office-hours` | Rakip sitelerde gezin → gözlemleri sentezle → tasarım dokümanı |

Komut paleti (Cmd+K) merkezdir. Skill'lerin var olduğunu bilmenize gerek yok. Ne istediğinizi yazarsınız, bulanık filtre doğru skill'i bulur ve AI tarayıcıyı bağlam olarak kullanarak çalıştırır.

### 7. Tasarım Döngüsü

AI destekli tasarım bir döngüdür, bir devir değildir.

```
Taslak oluştur (GPT Image API)
  → Tarayıcıda incele (canlı site ile yan yana)
  → Geri bildirimle yinele ("başlığı daha uzun yap")
  → Yönü onayla
  → Üretim HTML/CSS oluştur
  → Tarayıcıda önizle
  → /design-review ile ince ayar yap
  → Gönder
```

Tarayıcı "Figma'da nasıl göründüğü" ile "üretilimde nasıl göründüğü" arasındaki boşluğu kapatır. Çünkü AI her ikisini de aynı anda görebilir.

### 8. Güvenlik Döngüsü

Gerçek bir tarayıcıda CSO incelemesi, yalnızca statik analiz değil.

- Her giriş alanına XSS yükleri enjekte et, çalışıp çalışmadığını kontrol et
- Farklı bir kaynaktan istekleri yeniden oynatarak CSRF test et
- Oluşturulmuş URL'lere gezinerek açık yönlendirmeleri kontrol et
- CSP başlıklarının gerçekten uygulandığını (yalnızca mevcut değil) doğrula
- Çerezleri ve token'ları gerçek zamanlı olarak manipüle ederek kimlik doğrulama akışlarını test et
- Siteyi bir iframe içinde yükleyerek clickjacking'i kontrol et

Statik analiz desenleri yakalar. Tarayıcı testi gerçekliği yakalar.

### 9. İzleme Döngüsü

Dağıtım sonrası kanarya izleme, gerçek bir tarayıcıda.

```
Dağıt → Tarayıcı üretim URL'sini yükler
  → Ekran görüntüsü temel çizgisi
  → Her 5 dakikada bir: ekran görüntüsü, karşılaştır, konsolu kontrol et
  → Uyar: görsel regresyon, yeni konsol hataları, performans düşüşü
  → Kritik hata algılanırsa otomatik geri alma
```

AI yargısıyla sentetik izleme. Yalnızca "sayfa 200 döndürdü mü" değil, "sayfa doğru görünüyor mu ve düzgün çalışıyor mu."

## Mimari

```
+-------------------------------------------------------+
|                  GStack Browser                        |
|                                                        |
|  +------------------+  +---------------------------+  |
|  |   Chromium        |  |   Eklenti Yan Paneli       |  |
|  |   (Playwright)    |  |   ├── Sohbet (Claude Code)  |  |
|  |                   |  |   ├── Aktivite Akışı        |  |
|  |   ┌────────────┐  |  |   ├── Öğe Referansları     |  |
|  |   │ Durum Çubuğu  │  |  |   ├── CSS Denetçisi        |  |
|  |   └────────────┘  |  |   ├── Komut Paleti      |  |
|  +--------┬──────────+  |   └── Ayarlar             |  |
|           │              +-------------┬--------------+  |
+-----------┼────────────────────────────┼─────────────────+
            │                            │
            v                            v
  +---------┴-----------+    +-----------┴-----------+
  |  Browse Sunucusu      |    |  Kenar Çubuğu Aracısı        |
  |  (HTTP + SSE)       |    |  (claude -p sarmalayıcı)  |
  |  :34567             |    |  gstack skill'lerini çalıştırır   |
  |                     |    |  Sekme başına izolasyon     |
  |  Komutlar:          |    |                       |
  |  goto, click, fill  |    |  Gelecek: BoomLooper   |
  |  snapshot, screenshot|   |  GenServer aracıları     |
  |  css, inspect, eval |    |                       |
  +---------┬-----------+    +-----------┬-----------+
            │                            │
            v                            v
  +---------┴-----------+    +-----------┴-----------+
  |  Kullanıcının Uygulaması        |    |  Claude Code          |
  |  localhost:3000     |    |  (kodu okur/yazar)  |
  |  (veya herhangi bir URL)       |    |                       |
  +---------------------+    +-----------------------+
```

## Rekabetçi Manzara

| Tarayıcı | Yaklaşım | Farklılaştırıcı | Zayıflık |
|---------|----------|---------------|----------|
| **Atlas** | Chromium çatalı + AI katmanı | Aracılı tarayıcı, "OWL" yalıtılmış Chromium | Tüketici odaklı, kod entegrasyonu yok |
| **Dia** | AI doğal tarayıcı | Temiz UI, AI etkileşimi için tasarlanmış | Geliştirme aracı yok, kod düzenleme yok |
| **Comet** | AI tarayıcısı | Çoklu aracılı gezinme | Erken, belirsiz geliştirici iş akışı |
| **Chrome Auto Browse** | Eklenti | Google'ın kendi, derin Chrome entegrasyonu | Yalnızca eklenti, kod düzenleme yok |
| **Cursor** | VSCode çatalı + AI | Sınıfında en iyi kod düzenleme | Tarayıcı görüntü alanı yok |
| **GStack Browser** | CC çalışma zamanı + tarayıcı görüntü alanı | Tarayıcıda hatayı gör, kodda düzelt, doğrula | Şu anda yalnızca macOS, tüketici özellikleri yok |

GStack Browser tüketici tarayıcılarıyla rekabet etmez. Tarayıcı ile düzenleyici arasında geçiş yapma iş akımıyla rekabet eder. Amaç bu geçişi görünmez kılmaktır.

## Tasarım Sistemi

DESIGN.md'den:
- **Birincil vurgu:** Amber-500 (#F59E0B) — aracı aktif, odak durumları, nabız
- **Arka plan:** Zinc-950 (#09090B) ile Zinc-800 (#27272A) arasında — koyu, yoğun
- **Tipografi:** JetBrains Mono (kod/durum), DM Sans (UI/etiketler)
- **Kenar yarıçapı:** 8px (md), 12px (lg), tam (haplar)
- **Hareket:** Aracı aktifte nabız animasyonu, 200ms geçişler
- **Düzen:** Kenar çubuğu (sağ), durum çubuğu (alt), palet (ortalanmış katman)

## Uygulama Durumu

| Bileşen | Durum | Notlar |
|-----------|--------|-------|
| .app paketi | **GÖNDERİLDİ** | 389MB, ~5s'de başlatır |
| DMG paketleme | **GÖNDERİLDİ** | 189MB sıkıştırılmış |
| `GSTACK_CHROMIUM_PATH` | **GÖNDERİLDİ** | Özel Chromium ikili dosya desteği |
| `BROWSE_EXTENSIONS_DIR` | **GÖNDERİLDİ** | Eklenti yolunu override etme |
| `/health` üzerinden kimlik doğrulama | **GÖNDERİLDİ** | .auth.json dosya yaklaşımını değiştirir, sunucu yeniden başlatmasında otomatik yenilenir |
| Derleme betiği | **GÖNDERİLDİ** | `scripts/build-app.sh` |
| Model yönlendirme | **GÖNDERİLDİ** | Eylemler için Sonnet, analiz için Opus (`pickSidebarModel`) |
| Hata ayıklama günlüğü | **GÖNDERİLDİ** | 40+ sessiz yakalama → 4 dosya boyunca önekli konsol günlüğü |
| Boşta zaman aşımı yok (başlıklı) | **GÖNDERİLDİ** | Pencere açık olduğu sürece tarayıcı canlı kalır |
| Çerez içe aktarma düğmesi | **GÖNDERİLDİ** | Kenar çubuğu alt bilgisinde tek tıkla, `/cookie-picker` açar |
| Kenar çubuğu ok ipucu | **GÖNDERİLDİ** | Kenar çubuğunu işaret eder, yalnızca kenar çubuğu gerçekten açıldığında gizlenir |
| Mimari dokümanı | **GÖNDERİLDİ** | `docs/designs/SIDEBAR_MESSAGE_FLOW.md` |
| Komut paleti | Planlanmış | Aşama 1b |
| Hızlı ekran görüntüsü | Planlanmış | Aşama 1b |
| Durum çubuğu | Planlanmış | Aşama 1b |
| Geliştirme sunucusu algılama | Planlanmış | Aşama 1b |
| BoomLooper entegrasyonu | Gelecek | Aşama 2 |
| Çapraz platform | Gelecek | Aşama 3 |
| Chromium çatalı | Tetikleyici kontrollü | Aşama 4 |
| Yerel kabuk | Ertelendi | Aşama 5 |

## 12 Aylık Vizyon

```
BUGÜN (Aşama 1)               6 AY (Aşama 2-3)          12 AY (Aşama 4-5)
─────────────                 ──────────────────            ────────────────────
macOS .app sarmalayıcı            BoomLooper çoklu aracı         Chromium çatalı VEYA
Eklenti kenar çubuğu             Docker kapsayıcıları              Yerel SwiftUI kabuğu
Yerel claude -p aracısı         Takım çalışma alanları                Çapraz platform
Tek proje                Linux/x64 browse               Otomatik güncelleme
Manuel skill çağırma              Otonom QA döngüleri            Skill pazarı
                              Performans izleme          Eklenti API'si
                              Gerçek zamanlı işbirliği         Kurumsal özellikler
```

12 aylık ideal: GStack Browser'ı açarsınız, projenizi algılar, geliştirme sunucunuzu başlatır, test paketinizi çalıştırır ve neyin bozuk olduğunu raporlar. "Düzelt" dersiniz ve AI her hatayı düzeltir, her düzeltmeyi görsel olarak doğrular ve bir PR oluşturur. PR'yi aynı tarayıcıda incelersiniz, onaylarsınız ve AI dağıtır ve kanaryayı izler. Hepsi bir pencerede.

İşte AI çalışma alanı olarak tarayıcı. Üzerine AI eklenmiş bir tarayıcı değil. Üzerine tarayıcı eklenmiş bir AI.

## İnceleme Geçmişi

Bu plan 4 incelemeden geçti:

1. **CEO İncelemesi** (`/plan-ceo-review`, SEÇİCİ GENİŞLEME) — 9 kapsam teklifi, 3 kabul edildi (Cmd+K, Cmd+Shift+S, durum çubuğu), 5 ertelendi, 1 atlandı
2. **Tasarım İncelemesi** (`/plan-design-review`) — 5/10 → 8/10 puan aldı, 9 tasarım kararı eklendi, 2 onaylanmış mockup üretildi
3. **Mühendislik İncelemesi** (`/plan-eng-review`) — 4 sorun bulundu, 0 kritik boşluk, test planı üretildi
4. **Codex İncelemesi** (dış ses) — 9 bulgu, 3 kritik boşluk yakalandı (sunucu paketleme, kimlik doğrulama dosyası konumu, proje bağlama). Tümü çözüldü.

Codex incelemesi 3 önceki incelemeden kurtulan gerçek mimari boşluk yakaladı. Çapraz model inceleme çalışıyor.