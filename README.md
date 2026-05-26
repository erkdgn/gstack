# gstack

> "Aralık'tan beri muhtemelen tek bir satır bile kod yazmadım, diyebilirim ki bu son derece büyük bir değişiklik." — [Andrej Karpathy](https://fortune.com/2026/03/21/andrej-karpathy-openai-cofounder-ai-agents-coding-state-of-psychosis-openclaw/), No Priors podcast, Mart 2026

Karpathy bunu söylediğinde, bunu nasıl yaptığını öğrenmek istedim. Bir kişi yirmi kişilik bir takım gibi nasıl ürün çıkarır? Peter Steinberger, temelde yalnız başına AI ajanlarıyla [OpenClaw](https://github.com/openclaw/openclaw) — 247K GitHub yıldızı — ürününü oluşturdu. Devrim burada. Doğru araçlarla donanmış tek bir geliştirici, geleneksel bir takımdan daha hızlı hareket edebilir.

Ben [Garry Tan](https://x.com/garrytan), [Y Combinator](https://www.ycombinator.com/)'ın Başkanı ve CEO'su. Binlerce startup ile çalıştım — Coinbase, Instacart, Rippling — bir garajda bir veya iki kişi olduklarında. YC'den önce, Palantir'deki ilk mühendis/PM/tasarımcılardan biriydim, Posterous'u kurdum (Twitter'a satıldı) ve YC'nin iç sosyal ağı Bookface'i geliştirdim.

**gstack benim cevabım.** Yirmi yıldır ürün geliştiriyorum ve şu an hayatımda hiç olmadığı kadar çok ürün çıkarıyorum. Son 60 günde: 3 üretim servisi, 40+ gönderilmiş özellik, yarı zamanlı, YC'yi tam zamanlı yönetirken. Mantıksal kod değişikliği bazında — AI'in şişirdiği ham LOC değil — 2026 koşu oranım **2013 tempomun ~810 katı** (11.417 vs günde 14 mantıksal satır). Yılbaşından bugüne (18 Nisan'a kadar), 2026 zaten **2013 yılının tamamının 240 katını** üretti. Bir demo repoyu hariç tutarak, 40 kamusal + özel `garrytan/*` reposu üzerinden ölçüldü. Çoğunu AI yazdı. Önemli olan kimin yazdığı değil, neyin gönderildiği.

> LOC eleştirmenleri, ham satır sayılarının AI ile şiştiğinde haksız değiller. Enflasyona göre düzeltildiğinde daha az üretken olduğum konusunda haksızlar. Çok daha üretkeğim. Tam metodoloji, uyarılar ve üretim betiği: **[LOC Tartışması Üzerine](docs/ON_THE_LOC_CONTROVERSY.md)**.

**2026 — 1.237 katkı ve sayıyor:**

![GitHub katkıları 2026 — 1.237 katkı, Ocak-Mart'ta büyük ivme](docs/images/github-2026.png)

**2013 — YC'de Bookface'i geliştirdiğimde (772 katkı):**

![GitHub katkıları 2013 — YC'de Bookface geliştirirken 772 katkı](docs/images/github-2013.png)

Aynı kişi. Farklı çağ. Fark, araçlar.

**gstack bunu nasıl yaptığımdır.** Claude Code'u sanal bir mühendislik takımına dönüştürür — ürünü yeniden düşünen bir CEO, mimariyi kilitleyen bir mühendislik yöneticisi, AI salatasını yakalayan bir tasarımcı, üretim hatalarını bulan bir inceleyici, gerçek bir tarayıcı açan bir QA lideri, OWASP + STRIDE denetimleri çalıştıran bir güvenlik sorumlusu ve PR'ı gönderen bir sürüm mühendisi. Yirmi üç uzman ve sekiz güç aracı, hepsi eğik çizgi komutları, hepsi Markdown, hepsi ücretsiz, MIT lisansı.

Bu benim açık kaynaklı yazılım fabrikam. Her gün kullanıyorum. Paylaşıyorum çünkü bu araçlar herkesin erişiminde olmalı.

Forklayın. Geliştirin. Kendinize uyarlayın. Ve ücretsiz açık kaynaklı yazılıma saldırmak istiyorsanız — yapabilirsiniz, ama önce denemenizi tercih ederim.

**Kimin için:**
- **Kurucular ve CEO'lar** — özellikle hâlâ ürün çıkarmak isteyen teknik olanlar
- **İlk kez Claude Code kullananlar** — boş bir prompt yerine yapılandırılmış roller
- **Teknik liderler ve kadrolu mühendisler** — her PR'da titiz inceleme, QA ve sürüm otomasyonu

## Hızlı başlangıç

1. gstack'i kurun (30 saniye — aşağıya bakın)
2. `/office-hours` çalıştırın — ne geliştirdiğinizi açıklayın
3. Herhangi bir özellik fikri üzerinde `/plan-ceo-review` çalıştırın
4. Değişiklik olan herhangi bir dalda `/review` çalıştırın
5. Sahgeleme URL'nizde `/qa` çalıştırın
6. Orada durun. Bunun size uygun olup olmadığını anlarsınız.

## Kurulum — 30 saniye

**Gereksinimler:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code), [Git](https://git-scm.com/), [Bun](https://bun.sh/) v1.0+, [Node.js](https://nodejs.org/) (yalnızca Windows)

### Adım 1: Makinenize kurun

Claude Code'u açın ve bunu yapıştırın. Claude gerisini halleder.

> gstack kurulumu: **`git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/.claude/skills/gstack && cd ~/.claude/skills/gstack && ./setup`** komutunu çalıştırın, ardından CLAUDE.md dosyasına tüm web taraması için gstack'ten /browse yeteneğini kullanmayı, asla mcp\_\_claude-in-chrome\_\_\* araçlarını kullanmamayı söyleyen bir "gstack" bölümü ekleyin ve kullanılabilir yetenekleri listeleyin: /office-hours, /plan-ceo-review, /plan-eng-review, /plan-design-review, /design-consultation, /design-shotgun, /design-html, /review, /ship, /land-and-deploy, /canary, /benchmark, /browse, /connect-chrome, /qa, /qa-only, /design-review, /setup-browser-cookies, /setup-deploy, /setup-gbrain, /retro, /investigate, /document-release, /document-generate, /codex, /cso, /autoplan, /plan-devex-review, /devex-review, /careful, /freeze, /guard, /unfreeze, /gstack-upgrade, /learn. Ardından kullanıcıya gstack'i mevcut projeye de eklemek isteyip istemediğini sorun, böylece takım arkadaşları da erişebilsin.

### Adım 2: Takım modu — paylaşılan repolar için otomatik güncelleme (önerilen)

Reponuzun içinden bunu yapıştırın. Sizi takım moduna geçirir, repoyu öyle başlatır ki takım arkadaşları gstack'i otomatik olarak alır ve değişikliği gönderir:

```bash
(cd ~/.claude/skills/gstack && ./setup --team) && ~/.claude/skills/gstack/bin/gstack-team-init required && git add .claude/ CLAUDE.md && git commit -m "require gstack for AI-assisted work"
```

Reponuzda satıcı dosyası yok, sürüm sapması yok, manuel yükseltme yok. Her Claude Code oturumu hızlı bir otomatik güncelleme kontrolüyle başlar (saatte bir kez ile sınırlı, ağ arızasına dayanıklı, tamamen sessiz).

Takım arkadaşlarını engellemek yerine yönlendirmeyi tercih ediyorsanız `required` yerine `optional` kullanın.

### OpenClaw

OpenClaw, Claude Code oturumlarını ACP üzerinden başlatır, bu nedenle Claude Code'da gstack kurulu olduğunda her gstack yeteneği çalışır. Bunu OpenClaw ajanınıza yapıştırın:

> gstack kurulumu: Claude Code için gstack kurmak için `git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/.claude/skills/gstack && cd ~/.claude/skills/gstack && ./setup` komutunu çalıştırın. Ardından AGENTS.md dosyasına bir "Kodlama Görevleri" bölümü ekleyin: kodlama çalışması için Claude Code oturumları başlatırken, oturuma gstack yeteneklerini kullanmasını söyleyin. Bu örnekleri ekleyin — güvenlik denetimi: "gstack'i yükle. /cso çalıştır", kod incelemesi: "gstack'i yükle. /review çalıştır", URL'de QA testi: "gstack'i yükle. /qa https:// çalıştır...", baştan sona özellik geliştirme: "gstack'i yükle. /autoplan çalıştır, planı uygula, ardından /ship çalıştır", geliştirmeden önce planla: "gstack'i yükle. /office-hours ardından /autoplan çalıştır. Planı kaydet, uygulama."

**Kurulumdan sonra, OpenClaw ajanınızla doğal olarak konuşun:**

| Siz şunu söylersiniz | Ne olur |
|----------------------|---------|
| "README'deki yazım hatasını düzelt" | Basit — Claude Code oturumu, gstack gerektirmez |
| "Bu repoda güvenlik denetimi çalıştır" | `Run /cso` ile Claude Code başlatır |
| "Bana bir bildirim özelliği geliştir" | /autoplan → uygulama → /ship ile Claude Code başlatır |
| "v2 API yeniden tasarımını planlamama yardım et" | /office-hours → /autoplan ile Claude Code başlatır, planı kaydeder |

Gelişmiş gönderim yönlendirmesi ve gstack-lite/gstack-full prompt şablonları için [docs/OPENCLAW.md](docs/OPENCLAW.md) dosyasına bakın.

### Yerel OpenClaw Yetenekleri (ClawHub üzerinden)

OpenClaw ajanınızda doğrudan çalışan, Claude Code oturumu gerektirmeyen dört metodoloji yeteneği. ClawHub'tan kurun:

```
clawhub install gstack-openclaw-office-hours gstack-openclaw-ceo-review gstack-openclaw-investigate gstack-openclaw-retro
```

| Yetenek | Ne yapar |
|---------|----------|
| `gstack-openclaw-office-hours` | 6 zorlayıcı soruyla ürün sorgulama |
| `gstack-openclaw-ceo-review` | 4 kapsam moduyla stratejik meydan okuma |
| `gstack-openclaw-investigate` | Kök neden hata ayıklama metodolojisi |
| `gstack-openclaw-retro` | Haftalık mühendislik retrospektifi |

Bunlar konuşma yetenekleridir. OpenClaw ajanınız bunları doğrudan sohbet üzerinden çalıştırır.

### Diğer AI Ajanları

gstack yalnızca Claude'da değil, 10 AI kodlama ajanında çalışır. Kurulum, hangi ajanların kurulu olduğunu otomatik olarak algılar:

```bash
git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/gstack
cd ~/gstack && ./setup
```

Veya `./setup --host <ad>` ile belirli bir ajanı hedefleyin:

| Ajan | Bayrak | Yetenekler şuraya kurulur |
|------|--------|---------------------------|
| OpenAI Codex CLI | `--host codex` | `~/.codex/skills/gstack-*/` |
| OpenCode | `--host opencode` | `~/.config/opencode/skills/gstack-*/` |
| Cursor | `--host cursor` | `~/.cursor/skills/gstack-*/` |
| Factory Droid | `--host factory` | `~/.factory/skills/gstack-*/` |
| Slate | `--host slate` | `~/.slate/skills/gstack-*/` |
| Kiro | `--host kiro` | `~/.kiro/skills/gstack-*/` |
| Hermes | `--host hermes` | `~/.hermes/skills/gstack-*/` |
| GBrain (mod) | `--host gbrain` | `~/.gbrain/skills/gstack-*/` |

**Başka bir ajan için destek mi eklemek istiyorsunuz?** [docs/ADDING_A_HOST.md](docs/ADDING_A_HOST.md) dosyasına bakın.
Tek bir TypeScript yapılandırma dosyası, sıfır kod değişikliği.

## Nasıl çalıştığını görün

```
Siz:    Takvimim için günlük bülten uygulaması geliştirmek istiyorum.
Siz:    /office-hours
Claude: [acıyı sorar — spesifik örnekler, varsayımsal olanlar değil]

Siz:    Birden fazla Google takvimi, güncel olmayan bilgiler içeren etkinlikler,
        yanlış konumlar. Hazırlık sonsuza kadar sürüyor ve sonuçlar yeterince iyi değil...

Claude: Çerçevelendirmeye itiraz edeceğim. "Günlük bülten uygulaması"
        dediniz. Ama aslında anlattığınız şey kişisel bir kurmay
        yardımcısı AI.
        [Farkında olmadan tanımladığınız 5 yeteneği çıkarır]
        [4 önkabulü sorgular — katılırsınız, katılmazsınız veya düzeltirsiniz]
        [Çaba tahminleriyle 3 uygulama yaklaşımı üretir]
        ÖNERİ: En dar kamaı yarın gönderin, gerçek
        kullanımdan öğrenin. Tam vizyon 3 aylık bir proje — gerçekten
        çalışan günlük bültenle başlayın.
        [tasarım belgesi yazar → otomatik olarak aşağı akış yeteneklerine besler]

Siz:    /plan-ceo-review
        [tasarım belgesini okur, kapsamı sorgular, 10 bölümlük inceleme çalıştırır]

Siz:    /plan-eng-review
        [veri akışı, durum makineleri, hata yolları için ASCII diyagramlar]
        [test matrisi, başarısızlık modları, güvenlik endişeleri]

Siz:    Planı onayla. Plan modundan çık.
        [11 dosya üzerinden 2.400 satır yazar. ~8 dakika.]

Siz:    /review
        [OTOMATİK DÜZELTİLDİ] 2 sorun. [SOR] Yarış durumu → düzeltmeyi onaylarsınız.

Siz:    /qa https://staging.myapp.com
        [gerçek tarayıcı açar, akışları tıklar, bir hata bulur ve düzeltir]

Siz:    /ship
        Testler: 42 → 51 (+9 yeni). PR: github.com/you/app/pull/42
```

"Günlük bülten uygulaması" dediniz. Ajan "bir kurmay yardımcısı AI'si geliştiriyorsunuz" dedi — çünkü özellik talebinize değil acınıza kulak verdi. Sekiz komut, baştan sona. Bu bir yardımcı pilot değil. Bu bir takım.

## Sprint

gstack bir araç koleksiyonu değil, bir süreçtir. Yetenekler bir sprintin çalıştığı sırayla çalışır:

**Düşün → Planla → Geliştir → İncele → Test Et → Gönder → Yansı**

Her yetenek bir sonrakine beslenir. `/office-hours`, `/plan-ceo-review`'ın okuduğu bir tasarım belgesi yazar. `/plan-eng-review`, `/qa`'nın alacağı bir test planı yazar. `/review`, `/ship`'in düzeltildiğini doğruladığı hataları yakalar. Önceki adımın ne olduğunu her adım bildiği için hiçbir şey aradan kaymaz.

| Yetenek | Uzmanınız | Ne yaparlar |
|---------|-----------|-------------|
| `/office-hours` | **YC Ofis Saatleri** | Buradan başlayın. Kod yazmadan önce ürününüzü yeniden çerçeveleyen altı zorlayıcı soru. Çerçevendirmenize itiraz eder, önkabulleri sorgular, uygulama alternatifleri üretir. Tasarım belgesi her aşağı akış yeteneğine beslenir. |
| `/plan-ceo-review` | **CEO / Kurucu** | Sorunu yeniden düşünün. Talebin içindeki 10 yıldızlı ürünü bulun. Dört mod: Genişletme, Seçici Genişletme, Kapsamı Koruma, Küçültme. |
| `/plan-eng-review` | **Mühendislik Yöneticisi** | Mimari, veri akışı, diyagramlar, uç durumlar ve testleri kilitle. Gizli varsayımları gün yüzüne çıkarır. |
| `/plan-design-review` | **Kıdemli Tasarımcı** | Her tasarım boyutunu 0-10 arasında puanlar, 10'un nasıl göründüğünü açıklar, ardından planı oraya ulaştırmak için düzenler. AI Salata tespiti. Etkileşimli — her tasarım kararı için bir AskUserQuestion. |
| `/plan-devex-review` | **Geliştirici Deneyimi Lideri** | Etkileşimli DX incelemesi: geliştirici kişiliklerini keşfeder, rakiplerin TTHW'sine göre kıyaslar, büyülü anınızı tasarlar, sürtünme noktalarını adım adım izler. Üç mod: DX GENİŞLETME, DX PARLATMA, DX ÖNCELİKLENDİRME. 20-45 zorlayıcı soru. |
| `/design-consultation` | **Tasarım Ortağı** | Sıfırdan eksiksiz bir tasarım sistemi oluşturun. Manzarayı araştırır, yaratıcı riskler önerir, gerçekçi ürün maketleri üretir. |
| `/review` | **Kadrolu Mühendis** | CI'dan geçip üretimde patlayan hataları bulun. Açık olanları otomatik düzeltir. Bütünlük boşluklarını işaretler. |
| `/investigate` | **Hata Ayıklayıcı** | Sistematik kök neden hata ayıklama. Demir Kural: soruşturma olmadan düzeltme yok. Veri akışını izler, hipotezleri test eder, 3 başarısız düzeltmeden sonra durur. |
| `/design-review` | **Kod Yazabilen Tasarımcı** | /plan-design-review ile aynı denetim, ardından bulduklarını düzeltir. Atomik gönderimler, önce/sonra ekran görüntüleri. |
| `/devex-review` | **DX Test Uzmanı** | Canlı geliştirici deneyimi denetimi. Katılım sürecinizi gerçekten test eder: belgeleri gezin, başlangıç akışını deneyin, TTHW'yi zamanlayın, hataların ekran görüntüsünü alın. `/plan-devex-review` puanlarıyla karşılaştırır — planınızın gerçeğe uyup uymadığını gösteren bumerang. |
| `/design-shotgun` | **Tasarım Kaşifi** | "Bana seçenekleri göster." 4-6 AI maket varyantı üretir, tarayıcınızda bir karşılaştırma tablosu açar, geri bildiriminizi toplar ve yineler. Tat belleği neyi sevdiğinizi öğrenir. Bir şeyi sevdiğinizde tekrarlayın, ardından `/design-html`'e devredin. |
| `/design-html` | **Tasarım Mühendisi** | Bir maketi gerçekten çalışan üretim HTML'ine dönüştürün. Pretext hesaplanmış düzen: metin yeniden akar, yükseklikler ayarlanır, düzenler dinamiktir. 30KB, sıfır bağımlılık. React/Svelte/Vue algılar. Tasarım türüne göre akıllı API yönlendirmesi (açılış sayfası vs gösterge paneli vs form). Çıktı gönderilebilir, bir demo değil. |
| `/qa` | **QA Lideri** | Uygulamanızı test edin, hataları bulun, atomik gönderimlerle düzeltin, yeniden doğrulayın. Her düzeltme için otomatik regresyon testleri üretir. |
| `/qa-only` | **QA Raporlayıcısı** | /qa ile aynı metodoloji ama yalnızca rapor. Kod değişikliği olmadan saf hata raporu. |
| `/pair-agent` | **Çoklu Ajan Koordinatörü** | Tarayıcınızı herhangi bir AI ajanıyla paylaşın. Bir komut, bir yapıştırma, bağlandı. OpenClaw, Hermes, Codex, Cursor veya curl yapabilen herhangi bir şeyle çalışır. Her ajan kendi sekmesini alır. Her şeyi izlemeniz için otomatik başlatılan headed mod. Uzak ajanlar için otomatik başlatılan ngrok tüneli. Kapsamlı belirteçler, sekme yalıtımı, hız sınırlaması, etkinlik atıfı. |
| `/cso` | **Baş Güvenlik Sorumlusu** | OWASP İlk 10 + STRIDE tehdit modeli. Sıfır gürültü: 17 yanlış pozitif dışlama, 8/10+ güven eşiği, bağımsız bulgu doğrulama. Her bulgu somut bir istismar senaryosu içerir. |
| `/ship` | **Sürüm Mühendisi** | Ana dalı senkronize et, testleri çalıştır, kapsamı denetle, gönder, PR aç. Test çerçeveniz yoksa sıfırdan başlatır. |
| `/land-and-deploy` | **Sürüm Mühendisi** | PR'ı birleştir, CI'ı ve dağıtımı bekle, üretim sağlığını doğrula. "Onaylandı"dan "üretimde doğrulandı"ya bir komut. |
| `/canary` | **SRE** | Dağıtım sonrası izleme döngüsü. Konsol hataları, performans gerilemeleri ve sayfa hataları için izler. |
| `/benchmark` | **Performans Mühendisi** | Sayfa yükleme sürelerini, Core Web Vitals'ı ve kaynak boyutlarını temel alın. Her PR'da önce/sonra karşılaştırması. |
| `/document-release` | **Teknik Yazar** | Az gönderdiğinizle eşleşmesi için tüm proje belgelerini güncelleyin. Eskimiş README'leri otomatik olarak yakalar. Bir Diataxis kapsam haritası (başvuru / nasıl yapılır / eğitim / açıklama) oluşturur, böylece boşluklar PR gövdesinde görünür. |
| `/document-generate` | **Belge Yazarı** | Diataxis çerçevesini kullanarak eksik belgeleri sıfırdan üretir. Önce kod tabanını araştırır, ardından kodla gerçekten eşleşen başvuru / nasıl yapılır / eğitim / açıklama belgeleri yazar. Bağımsız olarak veya kapsam haritası boşluklar bulduğunda `/document-release`'ten zincirlenebilir. Daha fazla: [eğitim](docs/tutorial-document-generate.md) • [nasıl yapılır](docs/howto-document-a-shipped-feature.md) • [neden Diataxis](docs/explanation-diataxis-in-gstack.md). |
| `/retro` | **Mühendislik Yöneticisi** | Takım odaklı haftalık retrospektif. Kişi başına dökümler, gönderim serileri, test sağlık eğilimleri, büyüme fırsatları. `/retro global` tüm projelerinizde ve AI araçlarınızda (Claude Code, Codex, Gemini) çalışır. |
| `/browse` | **QA Mühendisi** | Ajanınıza gözler verin. Gerçek Chromium tarayıcı, gerçek tıklamalar, gerçek ekran görüntüleri. Komut başına ~100ms. `/open-gstack-browser` kenar çubuğu, bot-karşıtı gizlilik ve otomatik model yönlendirmesi ile GStack Tarayıcı başlatır. |
| `/setup-browser-cookies` | **Oturum Yöneticisi** | Gerçek tarayıcınızdan (Chrome, Arc, Brave, Edge) headless oturuma çerezleri aktarın. Kimlik doğrulamalı sayfaları test edin. |
| `/autoplan` | **İnceleme Hattı** | Bir komut, tamamen incelenmiş plan. CEO → tasarım → mühendis incelemesini kodlanmış karar ilkeleriyle otomatik olarak çalıştırır. Yalnızca beğeni kararlarını onayınıza sunar. |
| `/learn` | **Bellek** | gstack'in oturumlar arasında öğrendiklerini yönetin. Proje özelinde kalıpları, tuzakları ve tercihleri gözden geçirin, arayın, budayın ve dışa aktarın. Öğrenmeler oturumlar arasında birikir, böylece gstack kod tabanınızda zamanla daha akıllı hale gelir. |

### Hangi incelemeyi kullanmalıyım?

| Şunun için geliştiriyorsanız | Plan aşaması (koddan önce) | Canlı denetim (gönderimden sonra) |
|------------------------------|----------------------------|-----------------------------------|
| **Son kullanıcılar** (UI, web uygulaması, mobil) | `/plan-design-review` | `/design-review` |
| **Geliştiriciler** (API, CLI, SDK, belgeler) | `/plan-devex-review` | `/devex-review` |
| **Mimari** (veri akışı, performans, testler) | `/plan-eng-review` | `/review` |
| **Yukarıdakilerin hepsi** | `/autoplan` (CEO → tasarım → müh. → DX çalıştırır, hangilerinin geçerli olduğunu otomatik algılar) | — |

### Güç araçları

| Yetenek | Ne yapar |
|---------|----------|
| `/codex` | **İkinci Görüş** — OpenAI Codex CLI'dan bağımsız kod incelemesi. Üç mod: inceleme (geç/kal kapısı), çekişmeli meydan okuma ve açık danışma. Hem `/review` hem de `/codex` çalıştırıldığında çapraz model analizi. |
| `/careful` | **Güvenlik Korkulukları** — yıkıcı komutlardan önce uyarır (rm -rf, DROP TABLE, force-push). Etkinleştirmek için "dikkatli ol" deyin. Herhangi bir uyarıyı geçersiz kılabilirsiniz. |
| `/freeze` | **Düzenleme Kilidi** — dosya düzenlemelerini bir dizinle sınırlar. Hata ayıklarken kapsam dışında yanlışlıkla değişiklikleri önler. |
| `/guard` | **Tam Güvenlik** — `/careful` + `/freeze` tek komutta. Üretim çalışması için maksimum güvenlik. |
| `/unfreeze` | **Kilidi Aç** — `/freeze` sınırını kaldır. |
| `/open-gstack-browser` | **GStack Tarayıcı** — kenar çubuğu, bot-karşıtı gizlilik, otomatik model yönlendirmesi (eylemler için Sonnet, analiz için Opus), tek tıkla çerez içe aktarma ve Claude Code entegrasyonu ile GStack Tarayıcı başlatın. Sayfaları temizleyin, akıllı ekran görüntüleri alın, CSS düzenleyin ve bilgileri terminalinize geri geçirin. |
| `/setup-deploy` | **Dağıtım Yapılandırıcısı** — `/land-and-deploy` için tek seferlik kurulum. Platformunuzu, üretim URL'nizi ve dağıtım komutlarınızı algılar. |
| `/setup-gbrain` | **GBrain Katılımı** — sıfırdan gbrain çalıştırmaya 5 dakikadan kısa sürede. PGLite yerel, Supabase mevcut URL veya Supabase Yönetim API'si üzerinden yeni bir Supabase projesi otomatik sağlama. Claude Code için MCP kaydı + repoya özel güven triadı (okuma-yazma/salt-okunur/reddet). [Tam kılavuz](USING_GBRAIN_WITH_GSTACK.md). |
| `/sync-gbrain` | **Beyni Güncel Tut** — bu reponun kodunu `gbrain sources add` + `gbrain sync --strategy code` aracılıığıyla gbrain'e yeniden dizine ekler, CLAUDE.md'deki `## GBrain Arama Kılavuzu` bloğunu yeniler ve yetenek kontrolü başarısız olduğunda kılavuzu otomatik kaldırır. `--incremental` (varsayılan), `--full`, `--dry-run`. Idempotent; yeniden çalıştırmak güvenlidir. |
| `/gstack-upgrade` | **Kendi Kendini Güncelleyici** — gstack'i en son sürüme yükseltir. Küresel vs satıcı kurulumu algılar, her ikisini senkronize eder, neyin değiştiğini gösterir. |
| `/ios-qa` | **iOS Canlı Cihaz QA'sı (v1.43.0.0+)** — uygulama içinde gömülü bir `StateServer` üzerinden USB CoreDevice ile gerçek bir iPhone'u yönetin. Swift kaynak kodunu okuyun, yazılan `@Observable` erişimcileri kodlayın, ajan döngüsünü çalıştırın. İsteğe bağlı `--tailnet` bayrağı cihazı OpenClaw veya Tailscale tailnet'inizdeki herhangi bir HTTP yetenekli ajana açar, böylece uzak ajanlar donanıma hiç dokunmadan iOS QA çalıştırabilir. Yetenek katmanı izin listesi (gözlemle/etkileşim/mutasyon/geri yükle), cihaz başına oturum kilidi, denetim günlüğü. |
| `/ios-fix`, `/ios-design-review`, `/ios-clean`, `/ios-sync` | iOS hata düzeltme döngüsü, tasarımcı gözüyle HIG denetimi, hata ayıklama-köprüsü temizliği ve erişimci yeniden senkronizasyonu. `docs/skills.md` dosyasına bakın. Uçtan uca gözden geçirme: [docs/howto-ios-testing-with-gstack.md](docs/howto-ios-testing-with-gstack.md). |

### Yeni ikili dosyalar (v0.19)

Eğik çizgi komutu yeteneklerinin ötesinde, gstack bir oturum içinde ait olmayan iş akışları için bağımsız CLI'lar sunar:

| Komut | Ne yapar |
|-------|----------|
| `gstack-model-benchmark` | **Çapraz model karşılaştırması** — aynı prompt'u Claude, GPT (Codex CLI üzerinden) ve Gemini üzerinden çalıştırın; gecikme, belirteçler, maliyet ve (isteğe bağlı olarak) LLM-yargı kalite puanını karşılaştırın. Sağlayıcı başına kimlik doğrulama algılanır, kullanılamayan sağlayıcılar temiz bir şekilde atlanır. Tablo, JSON veya markdown olarak çıktı. `--dry-run` API çağrısı harcamadan bayrakları + kimlik doğrulamayı doğrular. |
| `gstack-taste-update` | **Tasarım zevki öğrenimi** — `/design-shotgun`'dan onayları ve reddetmeleri kalıcı bir proje başına zevk profiline yazar. Haftada %5 azalır. Gelecekteki varyant üretimine geri beslenir, böylece sistem gerçekten neyi seçtiğinizi öğrenir. |
| `gstack-ios-qa-daemon` | **iOS QA arka plan programı** — bir ajan ile USB CoreDevice üzerinden bağlı bir iPhone arasında Mac tarafı aracı. Varsayılan olarak geri döngü; `--tailnet` kimlik doğrulamalı yetenek katmanlarıyla Tailscale'e bakan bir dinleyici açar. `~/.gstack/ios-qa-daemon.pid` üzerinde flock ile tek örnek. [docs/howto-ios-testing-with-gstack.md](docs/howto-ios-testing-with-gstack.md) dosyasına bakın. |
| `gstack-ios-qa-mint` | **iOS izin listesi yöneticisi** — tailnet izin listesi için sahibin yetki verme CLI'ı. `~/.gstack/ios-qa-allowlist.json` (mod 0600) üzerinde `grant`/`revoke`/`list`. Uzak ajanlar asla otomatik izin listesine eklenmez; bu açık niyet yoludur. |

### Sürekli kontrol noktası modu (isteğe bağlı, varsayılan olarak yerel)

`gstack-config set checkpoint_mode continuous` ayarlayın ve yetenekler çalışmanızı ilerledikçe `WIP:` öneki ve yapılandırılmış bir `[gstack-context]` gövdesi (kararlar, kalan iş, başarısız yaklaşımlar) ile otomatik olarak gönderir. Çökmelerden ve bağlam değişimlerinden kurtulur. `/context-restore` oturum durumunu yeniden oluşturmak için bu gönderimleri okur. `/ship`, bisect temiz kalsın diye PR'dan önce WIP gönderimlerini filtre-sıkıştırır (WIP olmayan gönderimleri koruyarak). Gönderim, `checkpoint_push=true` üzerinden isteğe bağlıdır — varsayılan, her WIP gönderiminde CI'ı tetiklememek için yalnızca yereldir.

### Alan yetenekleri + ham CDP kaçış kapısı

İki yeni tarayıcı ilkesi, gstack ajanını zamanla güçlendirir:

- **`$B domain-skill save`** — ajan, bir site başına not kaydeder (örn., "LinkedIn'in Başvur düğmesi bir iframe içinde yer alır") ve bir sonraki sefere o ana bilgisayar adını ziyaret ettiğinde otomatik olarak tetiklenir. Karantina → 3 başarılı kullanımdan sonra etkin → `$B domain-skill promote-to-global` ile isteğe bağlı çapraz proje tanıtımı. Depolama, `/learn`'in proje başına öğrenmeler dosyasının yanında yer alır. Tam başvuru: **[docs/domain-skills.md](docs/domain-skills.md)**.
- **`$B cdp <Domain.method>`** — seçilmiş komutların kaçırdığı nadir durumlar için ham Chrome DevTools Protokolü kaçış kapısı. Varsayılan-engelle: yöntemlerin `browse/src/cdp-allowlist.ts` dosyasına tek satırlık bir gerekçeyle açıkça eklenmesi gerekir. İki katmanlı mutex, tarayıcı kapsamlı CDP çağrılarını sekme başına çalışmayla serileştirir. Veri sızdırma yöntemleri için çıktı, UNTRUSTED zarfına sarılır.

> Korkuluklar, izin listesi veya arka plan programı olmayan ham CDP mi istiyorsunuz — sadece ajandan Chrome'a ince aktarım? [browser-use/browser-harness-js](https://github.com/browser-use/browser-harness-js) farklı bir felsefedir (ajan tarafından yazılan yardımcılar vs gstack'in seçilmiş komutları) ve gstack'in güvenlik yığınını istemiyorsanız iyi bir uyumdur. İkisi bir arada var olabilir: gstack'in `$B cdp`'si ve harness, Playwright'ın `newCDPSession`'ı aracılığıyla aynı Chrome'a bağlanabilir.

**[Her yetenek için derinlemesine incelemeler, örnekler ve felsefe →](docs/skills.md)**

### Karpathy'nin dört başarısızlık modu? Zaten kapsanmış.

Andrej Karpathy'nin [AI kodlama kuralları](https://github.com/forrestchang/andrej-karpathy-skills) (17K yıldız) dört başarısızlık modunu doğru belirler: yanlış varsayımlar, aşırı karmaşıklık, ortogonal düzenlemeler, zorunlu üzerinden bildirimsel. gstack'in iş akışı yetenekleri dördünü de uygular. `/office-hours` kod yazılmadan önce varsayımları gün yüzüne çıkarır. Karışıklık Protokolü, Claude'un mimari kararlar konusunda tahmin etmesini durdurur. `/review` gereksiz karmaşıklığı ve geçiş düzenlemelerini yakalar. `/ship`, görevleri test-öncelikli yürütmeyle doğrulanabilir hedeflere dönüştürür. Zaten Karpathy tarzı CLAUDE.md kuralları kullanıyorsanız, gstack, bunların yalnızca tekil promptlarda değil, tüm sprintlerde kalmasını sağlayan iş akışı uygulama katmanıdır.

## Paralel sprintler

gstack bir sprintle iyi çalışır. Aynı anda on sprintle koştuğunuzda ilginçleşir.

**Tasarım kalbin merkezindedir.** `/design-consultation` tasarım sisteminizi sıfırdan oluşturur, mevcut olanları araştırır, yaratıcı riskler önerir ve `DESIGN.md` yazar. Ama asıl sihir shotgun'dan HTML'e boru hattıdır.

**`/design-shotgun` keşfetme şeklinizdir.** Ne istediğinizi açıklarsınız. GPT Image kullanarak 4-6 AI maket varyantı üretir. Ardından tarayıcınızda tüm varyantları yan yana gösteren bir karşılaştırma tablosu açar. Favorilerinizi seçer, geri bildirim bırakırsınız ("daha fazla beyaz alan", "daha cesur başlık", "gradyanı kaldır") ve yeni bir tur üretir. Bir şeyi sevdiğinizde tekrarlayın. Tat belleği birkaç tur sonra gerçekten neyi sevdiğinizi yanlılık göstermeye başlar. Vizyonunuzu kelimelerle anlatıp AI'in anlamasını ummak yok. Seçenekleri görür, iyi olanları seçer ve görsel olarak yinelersiniz.

**`/design-html` gerçeğe dönüştürür.** Onaylanmış o maketi (`/design-shotgun`'dan, bir CEO planından, bir tasarım incelemesinden veya sadece bir açıklamadan) alın ve üretim kalitesinde HTML/CSS'e dönüştürün. Bir görünüm genişliğinde iyi görünen ama her yerde bozulan AI HTML türünden değil. Bu, hesaplanmış metin düzeni için Pretext kullanır: metin yeniden boyutlandırmada gerçekten akar, yükseklikler içeriğe ayarlanır, düzenler dinamiktir. 30KB ek yük, sıfır bağımlılık. Çerçevenizi (React, Svelte, Vue) algılar ve doğru formatı çıkarır. Akıllı API yönlendirmesi, açılış sayfası, gösterge paneli, form veya kart düzeni olmasına göre farklı Pretext kalıpları seçer. Çıktı gerçekten gönderilebilir bir şey, bir demo değil.

**`/qa` büyük bir atılımdı.** 6'dan 12 paralel çalışana geçmemi sağladı. Claude Code'un *"SORUNU GÖRÜYORUM"* deyip ardından bunu gerçekten düzeltmesi, bir regresyon testi üretmesi ve düzeltmeyi doğrulaması — çalışma şeklimi değişti. Ajanın artık gözleri var.

**Akıllı inceleme yönlendirmesi.** İyi yönetilen bir startup'taki gibi: CEO'nun altyapı hata düzeltmelerine bakması gerekmez, arka uç değişiklikleri için tasarım incelemesi gerekmez. gstack hangi incelemelerin çalıştırıldığını izler, hangisinin uygun olduğunu belirler ve akıllı şeyi yapar. Gönderim İnceleme Hazırlığı Panosu, göndermeden önce nerede olduğunuzu söyler.

**Her şeyi test edin.** `/ship`, projenizin test çerçevesi yoksa sıfırdan başlatır. Her `/ship` çalıştırması bir kapsam denetimi üretir. Her `/qa` hata düzeltmesi bir regresyon testi üretir. %100 test kapsamı hedeftir — testler hisseli kodlamayı yolo kodlama yerine güvenli hale getirir.

**`/document-release` hiç sahip olmadığınız mühendistir.** Projenizdeki her belge dosyasını okur, diff ile çapraz başvuru yapar ve kaymış olan her şeyi günceller. README, ARCHITECTURE, CONTRIBUTING, CLAUDE.md, TODOS — hepsi otomatik olarak güncel tutulur. Ve şimdi `/ship` onu otomatik olarak çağırıyor — ek bir komut olmadan belgeler güncel kalır.

**Gerçek tarayıcı modu.** `/open-gstack-browser`, bot-karşıtı gizlilik, özel markalaştırma ve kenar çubuğu uzantısı entegre edilmiş AI kontrollü bir Chromium olan GStack Tarayıcı başlatır. Google ve NYTimes gibi siteler captcha olmadan çalışır. Menü çubuğu "Chrome for Testing" yerine "GStack Browser" der. Normal Chrome'unuz dokunulmamış kalır. Mevcut tüm tarama komutları değişmeden çalışır. `$B disconnect` headless moda döner. Tarayıcı, pencere açık olduğu sürece canlı kalır... siz çalışırken boşta kalma zaman aşımı onu öldürmez.

**Kenar çubuğu ajanı — AI tarayıcı asistanınız.** Chrome yan panelinde doğal dil yazın ve bir alt Claude örneği çalıştırır. "Ayarlar sayfasına git ve ekran görüntüsünü al." "Bu formu test verileriyle doldur." "Bu listedeki her öğeyi gezin ve fiyatları çıkar." Kenar çubuğu doğru modele otomatik yönlendirir: hızlı eylemler (tıklama, gezinme, ekran görüntüsü) için Sonnet ve okuma/analiz için Opus. Her görev en fazla 5 dakika alır. Kenar çubuğu ajanı yalıtılmış bir oturumda çalışır, böylece ana Claude Code pencerenizi etkilemez. Kenar çubuğu alt bilgisinden tek tıkla çerez içe aktarma.

**Kişisel otomasyon.** Kenar çubuğu ajanı yalnızca geliştirici iş akışları için değil. Örnek: "Çocuğumun okulunun ebeveyn portalına git ve diğer tüm ebeveynlerin adlarını, telefon numaralarını ve fotoğraflarını Google Kişilerime ekle." Kimlik doğrulaması almanın iki yolu: (1) headed tarayıcıda bir kez oturum açın, oturumunuz kalır, veya (2) kenar çubuğu alt bilgisindeki "çerezler" düğmesine tıklayarak gerçek Chrome'unuzdan çerezleri içe aktarın. Kimlik doğrulandıktan sonra, Claude dizini gezin, verileri çıkarır ve kişileri oluşturur.

**Prompt enjeksiyonu savunması.** Düşmanca web sayfaları kenar çubuğu ajanınızı ele geçirmeye çalışır. gstack katmanlı bir savunma sunar: tarayıcıyla birlikte gelen 22MB ML sınıflandırıcı her sayfayı ve araç çıktısını yerel olarak tarar, bir Claude Haiku transkript kontrolü tam konuşma şekli üzerinde oylama yapar, sistem promptundaki rastgele bir kanarya belirteci metin, araç argümanları, URL'ler ve dosya yazımları arasında oturum sızdırma girişimlerini yakalar ve bir karar birleştirici engellemeden önce iki sınıflandırıcının anlaşmasını gerektirir (Stack Overflow tarzı talimat sayfalarında tek model yanlış pozitiflerini önler). Kenar çubuğu başlığındaki bir kalkan simgesi durumu gösterir (yeşil/amber/kırmızı). 2/3 anlaşma için `GSTACK_SECURITY_ENSEMBLE=deberta` aracılığıyla 721MB DeBERTa-v3 topluluğuna katılın. Acil durdurma anahtarı: `GSTACK_SECURITY_OFF=1`. Tam yığın için [ARCHITECTURE.md](ARCHITECTURE.md#prompt-injection-defense-sidebar-agent) dosyasına bakın.

**AI takıldığında tarayıcı devri.** Bir CAPTCHA, kimlik doğrulama duvarı veya MFA istemi mi var? `$B handoff` tüm çerezleriniz ve sekmeleriniz korunduğundan tam olarak aynı sayfada görünür bir Chrome açar. Sorunu çözün, Claude'a bittiğinizi söyleyin, `$B resume` tam kaldığı yerden devam eder. Ajan, 3 ardışık başarısızlıktan sonra otomatik olarak önerir.

**`/pair-agent` çapraz-ajan koordinasyonudur.** Claude Code'dasınız. OpenClaw da çalışıyor. Veya Hermes. Veya Codex. İkisinin de aynı web sitesine bakmasını istiyorsunuz. `/pair-agent` yazın, ajanınızı seçin ve bir GStack Tarayıcı penceresi açılır, böylece izleyebilirsiniz. Yetenek, bir talimat bloğu yazdırır. O bloğu diğer ajanın sohbetine yapıştırın. Oturum belirteci için tek seferlik bir kurulum anahtarı alışverişi yapar, kendi sekmesini oluşturur ve taramaya başlar. Her iki ajanın aynı tarayıcıda çalıştığını, her birinin kendi sekmesinde, birbirine müdahale edemediğini görürsünüz. ngrok kuruluysa, tünel otomatik olarak başlar, böylece diğer ajan tamamen farklı bir makinede olabilir. Aynı makinedeki ajanlar, kimlik bilgilerini doğrudan yazan sıfır sürtünmeli bir kısayol alır. Bu, farklı satıcılardan AI ajanlarının gerçek güvenlikle paylaşımlı bir tarayıcı üzerinden koordinasyon kurabildiği ilk kez: kapsamlı belirteçler, sekme yaliaıtım, hız sınırlaması, etki alanı kısıtlamaları ve etkinlik atfı.

**Çoklu AI ikinci görüş.** `/codex`, OpenAI'nin Codex CLI'ından bağımsız bir inceleme alır — aynı diff'e bakan tamamen farklı bir AI. Üç mod: geç/kal kapısı ile kod incelemesi, kodunuzu aktif olarak bozmaya çalışan çekişmeli meydan okuma ve oturum sürekliliği ile açık danışma. Hem `/review` (Claude) hem de `/codex` (OpenAI) aynı dalı incelediğinde, hangi bulguların örtüştüğünü ve hangilerinin her birine özel olduğunu gösteren bir çapraz model analizi alırsınız.

**Talep üzerine güvenlik korkulukları.** "Dikkatli ol" deyin ve `/careful` herhangi bir yıkıcı komuttan önce uyarır — rm -rf, DROP TABLE, force-push, git reset --hard. `/freeze`, hata ayıklarken düzenlemeleri bir dizine kilitler, böylece Claude yanlışlıkla ilgili olmayan kodu "düzeltmez". `/guard` her ikisini etkinleştirir. `/investigate`, soruşturulan modüle otomatik olarak dondurur.

**Proaktif yetenek önerileri.** gstack hangi aşamada olduğunuzu fark eder — beyin fırtınası, inceleme, hata ayıklama, test — ve doğru yeteneği önerir. Sevmediniz mi? "Önermeyi bırak" deyin ve oturumlar arasında hatırlar.

## 10-15 paralel sprint

gstack bir sprintle güçlüdür. Aynı anda on çalıştırıldığında dönüştürücüdür.

[Conductor](https://conductor.build), birden fazla Claude Code oturumunu paralel olarak çalıştırır — her biri kendi yalıtılmış çalışma alanında. Bir oturum yeni bir fikir üzerinde `/office-hours` çalıştırıyor, diğeri bir PR üzerinde `/review`, üçüncüsü bir özellik uyguluyor, dördüncüsü staging üzerinde `/qa`, ve altı tanesi daha başka dallarda. Hepsi aynı anda. Düzenli olarak 10-15 paralel sprint çalıştırıyorum — şu an pratik maksimum bu.

Sprint yapısı paralelliği çalışır hale getirir. Süreç olmadan, on ajan on kaos kaynağıdır. Süreçle — düşün, planla, geliştir, incele, test et, gönder — her ajan ne yapacağını ve ne zaman duracağını tam olarak bilir. Onları bir CEO'nun bir takımı yönettiği şekilde yönetirsiniz: önemli olan kararlara göz atın, gerisinin çalışmasına izin verin.

### Sesli giriş (AquaVoice, Whisper vb.)

gstack yeteneklerinin sesle kullanımı kolay tetikleyici ifadeleri vardır. Ne istediğinizi doğal olarak söyleyin —
"bir güvenlik kontrolü çalıştır", "web sitesini test et", "bir mühendislik incelemesi yap" — ve doğru yetenek etkinleşir. Eğik çizgi komut adlarını veya kısaltmaları hatırlamanız gerekmez.

## Kaldırma

### Seçenek 1: Kaldırma betiğini çalıştırın

gstack makinenize kuruluysa:

```bash
~/.claude/skills/gstack/bin/gstack-uninstall
```

Bu, yetenekler, sembolik bağlantılar, küresel durum (`~/.gstack/`), proje yerel durumu, tarama arka plan programları ve geçici dosyalar ile ilgilenir. Yapılandırma ve analitiği korumak için `--keep-state` kullanın. Onayı atlamak için `--force` kullanın.

### Seçenek 2: Manuel kaldırma (yerel repo yok)

Repon klonlanmış değilse (örn. bir Claude Code yapıştırması ile kurup klonu sildiyseniz):

```bash
# 1. Tarama arka plan programlarını durdur
pkill -f "gstack.*browse" 2>/dev/null || true

# 2. SKILL.md'si gstack/ içine yönelen yetenek dizinlerini kaldır
find ~/.claude/skills -mindepth 1 -maxdepth 1 -type d ! -name gstack 2>/dev/null |
while IFS= read -r dir; do
  link="$dir/SKILL.md"
  [ -L "$link" ] || continue
  target=$(readlink "$link" 2>/dev/null) || continue
  case "$target" in
    gstack/*|*/gstack/*)
      rm -f "$link"
      rmdir "$dir" 2>/dev/null || true
      ;;
  esac
done

# 3. gstack'i kaldır
rm -rf ~/.claude/skills/gstack

# 4. Küresel durumu kaldır
rm -rf ~/.gstack

# 5. Entegrasyonları kaldır (asla kurmadıklarınızı atlayın)
rm -rf ~/.codex/skills/gstack* 2>/dev/null
rm -rf ~/.factory/skills/gstack* 2>/dev/null
rm -rf ~/.kiro/skills/gstack* 2>/dev/null
rm -rf ~/.openclaw/skills/gstack* 2>/dev/null

# 6. Geçici dosyaları kaldır
rm -f /tmp/gstack-* 2>/dev/null

# 7. Proje başına temizlik (her proje kökünden çalıştırın)
rm -rf .gstack .gstack-worktrees .claude/skills/gstack 2>/dev/null
rm -rf .agents/skills/gstack* .factory/skills/gstack* 2>/dev/null
```

### CLAUDE.md'yi temizleyin

Kaldırma betiği CLAUDE.md'yi düzenlemez. gstack'in eklendiği her projede `## gstack` ve `## Yetenek yönlendirmesi` bölümlerini kaldırın.

### Playwright

`~/Library/Caches/ms-playwright/` (macOS) yerinde bırakılır çünkü diğer araçlar onu paylaşabilir. Başka hiçbir şeye gerek yoksa kaldırın.

---

Ücretsiz, MIT lisanslı, açık kaynaklı. Premium katman yok, bekleme listesi yok.

Nasıl yazılım geliştirdiğimi açık kaynaklı hale getirdim. Forklayabilir ve kendinize uyarlayabilirsiniz.

> **İşe alım yapıyoruz.** AI kodlama hızında gerçek ürünler çıkarmak ve gstack'i sağlamlaştırmak mı istiyorsunuz?
> YC'de çalışın — [ycombinator.com/software](https://ycombinator.com/software)
> Son derece rekabetçi maaş ve hisse. San Francisco, Dogpatch Bölgesi.

## GBrain — kodlama ajanınız için kalıcı bilgi

[GBrain](https://github.com/garrytan/gbrain), AI ajanları için kalıcı bir bilgi tabanıdır — ajanınızın oturumlar arasında gerçekten tuttuğu bellek olarak düşünün. GStack, sıfırdan "çalışıyor, ajanım çağırabiliyor"a tek komutla bir yol sunar.

```bash
/setup-gbrain
```

Dört yol, birini seçin:

- **Supabase, mevcut URL** — bulut ajanınız zaten bir beyin sağlamış; Session Pooler URL'sini yapıştırın, artık bu dizüstü bilgisayar aynı veriyi kullanıyor.
- **Supabase, otomatik sağlama** — bir Supabase Kişisel Erişim Belirteci yapıştırın; yetenek yeni bir proje oluşturur, sağlıklı olana kadar yoklar, pooler URL'sini alır, `gbrain init`'e iletir. Uçtan uca ~90 saniye.
- **PGLite yerel** — sıfır hesap, sıfır ağ, ~30 saniye. Yalnızca bu Mac'te yalıtılmış beyin. Önce denemek için harika; daha sonra `/setup-gbrain --switch` ile Supabase'e geçin.
- **Uzak gbrain MCP** — beyniniz başka bir makinede (Tailscale, ngrok, iç LAN) veya bir takım arkadaşının sunucusunda çalışıyor; bir MCP URL'si ve taşıyıcı belirteç yapıştırın. İsteğe bağlı olarak, bölünmüş-motor modunda simge-farkında kod araması için yerel bir PGLite ile eşleştirin. Yerel bir veritabanı kurmadan çapraz makine belleği için en iyisi.

Başlatmadan sonra, yetenek gbrain'i Claude Code için bir MCP sunucusu olarak kaydetmeyi sunar (`claude mcp add gbrain -- gbrain serve`), böylece `gbrain search`, `gbrain put` vb. birinci sınıf yazılı araçlar olarak görünür — bash kabuk çağrıları değil.

**Beyni güncel tutma.** Kodunu gbrain'e yeniden dizine eklemek için herhangi bir repodan `/sync-gbrain` çalıştırın (varsayılan olarak artımlı, tam yeniden dizine eklemek için `--full`, önizleme için `--dry-run`). Yetenek, çalışma dizinini `gbrain sources add` aracılığıyla federe kaynak olarak kaydeder, `gbrain sync --strategy code` çalıştırır ve ajanın `gbrain search`/`code-def`/`code-refs`'i Grep üzerinde tercih etmesi için projenizin CLAUDE.md'sine bir `## GBrain Arama Kılavuzu` bloğu yazar. Yetenek kontrolü başarısız olduğunda blok otomatik olarak kaldırılır — kurulu olmayan araçları gösteren eski kılavuz yok.

**Uzak başına güven politikası.** Makinenizdeki her repo üç katmandan birini alır:

- `read-write` — ajan beyinde arama yapabilir VE bu repodan yeni sayfalar yazabilir
- `read-only` — ajan arayabilir ama asla yazamaz (çok müşterili danışmanlar için en iyisi: paylaşılan beyne arayın, B İstemcisinin reposundayken A İstemcisinin çalışmasıyla kirletmeyin)
- `deny` — hiç gbrain etkileşimi yok

Yetenek repo başına bir kez sorar. Karar, aynı uzak deposunun iş ağaçları ve dallarında yapışkandır.

**GStack bellek senkronizasyonu (farklı özellik, aynı özel-repo altyapısı).** İsteğe bağlı olarak gstack durumunuzu (öğrenmeler, CEO planları, tasarım belgeleri, retros, geliştirici profili) bir özel git reposuna gönderir, böylece belleğiniz makineler arasında sizi takip eder, tek seferlik bir gizlilik istemi (her şey izin listesinde / yalnızca yapılar / kapalı) ve makinenizden ayrılmadan önce AWS anahtarlarını, belirteçleri, PEM bloklarını ve JWT'leri engelleyen derinlemesine savunma gizli tarayıcısı ile.

```bash
gstack-brain-init
```

**gstack'i Conductor'da mı çalıştırıyorsunuz?** Conductor, her çalışma alanının süreç ortamından `ANTHROPIC_API_KEY` ve `OPENAI_API_KEY`'i açıkça çıkarır, bu nedenle ücretli değerlendirmeler ve gbrain gömüleri kutudan çalışmaz. Bunun yerine Conductor'un çalışma alanı ortam yapılandırmasında `GSTACK_ANTHROPIC_API_KEY` ve `GSTACK_OPENAI_API_KEY` ayarlayın — gstack'in TS giriş noktaları bunları çalışma zamanında kurallı adlara yükseltir. Tam ayrıntılar ve yeni giriş noktalarına içe aktarmayı ekleme için katkıda bulunan kontrol listesi: [Conductor + GSTACK_* ortam değişkenleri](USING_GBRAIN_WITH_GSTACK.md#conductor--gstack_-env-vars).

**Tam shebang — her senaryo, her bayrak, her ikili yardımcı, her sorun giderme adımı:** [USING_GBRAIN_WITH_GSTACK.md](USING_GBRAIN_WITH_GSTACK.md)

Diğer başvurular: [docs/gbrain-sync.md](docs/gbrain-sync.md) (senkronizasyon özel kılavuzu) • [docs/gbrain-sync-errors.md](docs/gbrain-sync-errors.md) (hata dizini)

## Belgeler

| Belge | Kapsadığı konular |
|-------|-------------------|
| [Yetenek Derinlemesine İncelemeleri](docs/skills.md) | Her yetenek için felsefe, örnekler ve iş akışı (Greptile entegrasyonu dahil) |
| [Geliştirici Felsefesi](ETHOS.md) | Geliştirici felsefesi: Gölü Kaynat, Geliştirmeden Önce Ara, bilgiyunun üç katmanı |
| [GStack ile GBrain Kullanımı](USING_GBRAIN_WITH_GSTACK.md) | `/setup-gbrain` için her yol, bayrak, ikili yardımcı ve sorun giderme adımı |
| [GBrain Senkronizasyonu](docs/gbrain-sync.md) | Çapraz makine belleği kurulumu, gizlilik modları, sorun giderme |
| [Mimari](ARCHITECTURE.md) | Tasarım kararları ve sistem iç yapıları |
| [Tarayıcı Başvurusu](BROWSER.md) | `/browse` için tam komut başvurusu |
| [Katkıda Bulunma](CONTRIBUTING.md) | Geliştirici kurulumu, test, katkıda bulunan modu ve geliştirici modu |
| [Değişiklik Günlüğü](CHANGELOG.md) | Her sürümdeki yenilikler |

## Gizlilik ve Telemetri

gstack, projeyi geliştirmeye yardımcı olmak için **isteğe bağlı** kullanım telemetrisi içerir. İşte tam olarak ne olur:

- **Varsayılan kapalıdır.** Açıkça evet demedikçe hiçbir yere hiçbir şey gönderilmez.
- **İlk çalıştırmada,** gstack anonim kullanım verisi paylaşmak isteyip istemediğinizi sorar. Hayır diyebilirsiniz.
- **Gönderilen (kabul ederseniz):** yetenek adı, süre, başarı/başarısızlık, gstack sürümü, işletim sistemi. Hepsi bu.
- **Asla gönderilmeyen:** kod, dosya yolları, repo adları, dal adları, promptlar veya kullanıcı tarafından oluşturulan herhangi bir içerik.
- **Her zaman değiştirilebilir:** `gstack-config set telemetry off` her şeyi anında devre dışı bırakır.

Veriler [Supabase](https://supabase.com)'de depolanır (açık kaynak Firebase alternatifi). Şema [`supabase/migrations/`](supabase/migrations/) dizinindedir — tam olarak neyin toplandığını doğrulayabilirsiniz. Repodaki Supabase yayınlanabilir anahtarı bir genel anahtardır (Firebase API anahtarı gibi) — satır düzeyinde güvenlik ilkeleri tüm doğrudan erişimi reddeder. Telemetri, şema kontrollerini, etkinlik türü izin listelerini ve alan uzunluğu sınırlarını uygulayan doğrulanmış uç işlevlerinden akar.

**Yerel analitikler her zaman kullanılabilir.** Uzak veriye gerek kalmadan kişisel kullanım panonuzu yerel JSONL dosyasından görmek için `gstack-analytics` çalıştırın.

## Sorun giderme

**Yetenek görünmüyor mu?** `cd ~/.claude/skills/gstack && ./setup`

**`/browse` başarısız mı oluyor?** `cd ~/.claude/skills/gstack && bun install && bun run build`

**Eski kurulum mu?** `/gstack-upgrade` çalıştırın — veya `~/.gstack/config.yaml` dosyasında `auto_upgrade: true` ayarlayın

**Daha kısa komutlar mı istiyorsunuz?** `cd ~/.claude/skills/gstack && ./setup --no-prefix` — `/gstack-qa` yerine `/qa`'ya geçer. Tercihiniz gelecekteki yükseltmeler için hatırlanır.

**İsim uzaylı komutlar mı istiyorsunuz?** `cd ~/.claude/skills/gstack && ./setup --prefix` — `/qa` yerine `/gstack-qa`'ya geçer. gstack yanında başka yetenek paketleri çalıştırıyorsanız kullanışlıdır.

**Codex "SKILL.md geçersiz olduğu için yetenek(ler) yüklenmesi atlandı" mı diyor?** Codex yetenek açıklamalarınız eski. Düzelt: `cd ~/.codex/skills/gstack && git pull && ./setup --host codex` — veya repo yerel kurulumlar için: `cd "$(readlink -f .agents/skills/gstack)" && git pull && ./setup --host codex`

**Windows kullanıcıları:** gstack, Git Bash veya WSL üzerinden Windows 11'de çalışır. Bun'a ek olarak Node.js gereklidir — Bun'un Windows'ta Playwright'ın boru taşımacılığı ile bilinen bir hatası var ([bun#4253](https://github.com/oven-sh/bun/issues/4253)). Tarama sunucusu otomatik olarak Node.js'e geri döner. `bun` ve `node`'un ikisinin de PATH'inizde olduğundan emin olun.

Geliştirici Modu olmadan Windows'ta (MSYS2 / Git Bash), `setup`, `ln -snf` donmuş kopyalar ürettiği ve `git pull` üzerinde yenilenmediği için sembolik bağlantılar yerine dosya kopyalarına geri döner. **Her `git pull` sonrası yetenek dosyalarınızın repo ile eşleşmesi için `cd ~/.claude/skills/gstack && ./setup` yeniden çalıştırın.** `setup` sizi uyaran tek satırlık bir not yazdırır. Unix ve WSL sembolik bağlantıları korur ve yeniden çalıştırma gerektirmez.

**Claude yetenekleri göremediğini mi söylüyor?** Projenizin `CLAUDE.md` dosyasında bir gstack bölümü olduğundan emin olun. Şunu ekleyin:

```
## gstack
Tüm web taraması için gstack'ten /browse kullan. Asla mcp__claude-in-chrome__* araçlarını kullanma.
Kullanılabilir yetenekler: /office-hours, /plan-ceo-review, /plan-eng-review, /plan-design-review,
/design-consultation, /design-shotgun, /design-html, /review, /ship, /land-and-deploy,
/canary, /benchmark, /browse, /open-gstack-browser, /qa, /qa-only, /design-review,
/setup-browser-cookies, /setup-deploy, /setup-gbrain, /sync-gbrain, /retro, /investigate,
/document-release, /document-generate, /codex, /cso, /autoplan, /pair-agent, /careful, /freeze,
/guard, /unfreeze, /gstack-upgrade, /learn.
```

## Lisans

MIT. Sonsuza kadar ücretsiz. Bir şey geliştirin.