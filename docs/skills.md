# Yetenek Derinlemesine İncelemeler

Her gstack yeteneği için ayrıntılı kılavuzlar — felsefe, iş akışı ve örnekler.

| Yetenek | Uzmanınız | Ne yapar |
|-------|----------------|--------------|
| [`/office-hours`](#office-hours) | **YC Ofis Saatleri** | Buradan başlayın. Kod yazmadan önce ürününüzü yeniden çerçeveleyen altı zorlayıcı soru. Çerçevenize meydan okur, önkabulları sorgular, uygulama alternatifleri üretir. Tasarım belgesi her aşağı akış yeteneğine beslenir. |
| [`/plan-ceo-review`](#plan-ceo-review) | **CEO / Kurucu** | Sorunu yeniden düşünün. İsteğin içinde saklı 10 yıldızlı ürünü bulun. Dört mod: Genişletme, Seçici Genişletme, Kapsamı Tutma, Daraltma. |
| [`/plan-eng-review`](#plan-eng-review) | **Mühendislik Müdürü** | Mimariyi, veri akışını, diyagramları, uç durumları ve testleri kilitleyin. Gizli kabulleri açıkça çıkarır. |
| [`/plan-design-review`](#plan-design-review) | **Kıdemli Tasarımcı** | Etkileşimli plan modu tasarım incelemesi. Her boyutu 0-10 puanlar, 10'un nasıl göründüğünü açıklar, planı düzeltir. Plan modunda çalışır. |
| [`/design-consultation`](#design-consultation) | **Tasarım Ortağı** | Sıfırdan tam bir tasarım sistemi oluştur. Alanı bilir, yaratıcı riskler önerir, gerçekçi ürün modellemeleri üretir. Tüm diğer aşamaların kalbinde tasarım. |
| [`/review`](#review) | **Kıdemli Mühendis** | CI'yi geçen ama üretimde patlayan hataları bulun. Bariz olanları otomatik düzeltir. Eksiklik boşluklarını işaretler. |
| [`/investigate`](#investigate) | **Hata Ayıklayıcı** | Sistematik kök neden hata ayıklama. Demir Kural: araştırma olmadan düzeltme yok. Veri akışını izler, hipotezleri test eder, 3 başarısız düzeltmeden sonra durur. |
| [`/design-review`](#design-review) | **Kodlayan Tasarımcı** | Canlı site görsel denetimi + düzeltme döngüsü. 80 maddelik denetim, ardından bulduklarını düzeltir. Atomik commitler, öncesi/sonrası ekran görüntüleri. |
| [`/design-shotgun`](#design-shotgun) | **Tasarım Kaşifi** | Birden fazla AI tasarım varyantı üret, tarayıcında bir karşılaştırma panosu aç ve bir yön onaylayana kadar yinele. Zevk belleği tercihlerinize doğru önyargılıdır. |
| [`/design-html`](#design-html) | **Tasarım Mühendisi** | Üretim kalitesinde Pretext-yerel HTML üretir. Onaylanmış modellemeler, CEO planları, tasarım incelemeleri veya sıfırdan çalışır. Yeniden boyutlandırmada metin yeniden akışır, yükseklikler içeriğe uyum sağlar. Akıllı API yönlendirmesi tasarım türüne göre. React/Svelte/Vue için çerçeve algılama. |
| [`/qa`](#qa) | **QA Lideri** | Uygulamanızı test edin, hataları bulun, atomik commitlerle düzeltin, yeniden doğrulayın. Her düzeltme için otomatik regresyon testleri üretir. |
| [`/qa-only`](#qa) | **QA Raporlayıcı** | /qa ile aynı metodoloji ama yalnızca rapor. Kod değişikliği olmadan saf hata raporu istediğinizde kullanın. |
| [`/scrape`](#scrape) | **Tarayıcı Veri Çıkarıcı** | Bir web sayfasından veri çekin. İlk çağrı `$B` aracılığıyla prototip oluşturur; eşleşen bir niyette sonraki çağrılar ~200ms'de kodlanmış bir tarayıcı-yetenek çalıştırır. |
| [`/skillify`](#skillify) | **Yetenek Kodlayıcı** | Sohbetinizde geri yürür, son `/scrape` prototipini bulur, betik + test + sabit sentezler, testi çalıştırır, commit etmeden önce sorar. |
| [`/ship`](#ship) | **Yayınlama Mühendisi** | Ana dalı senkronize et, testleri çalıştır, kapsamı denetle, push et, PR aç. Test çerçeveniz yoksa bir tane kurar. Tek komut. |
| [`/land-and-deploy`](#land-and-deploy) | **Yayınlama Mühendisi** | PR'yi birleştir, CI ve dağıtımı bekle, üretim sağlığını doğrula. "Onaylandı"dan "üretimde doğrulandı"ya tek komut. |
| [`/canary`](#canary) | **SRE** | Dağıtım sonrası izleme döngüsü. Tarayıcı artalan sürecini kullanarak konsol hataları, performans gerilemeleri ve sayfa hataları için izler. |
| [`/benchmark`](#benchmark) | **Performans Mühendisi** | Sayfa yükleme sürelerini, Core Web Vitals'ı ve kaynak boyutlarını temel çizgiye alın. Her PR'den önce ve sonra karşılaştırın. Zaman içindeki eğilimleri izleyin. |
| [`/cso`](#cso) | **Baş Güvenlik Sorumlusu** | OWASP Top 10 + STRIDE tehdit modelleme güvenlik denetimi. Enjeksiyon, kimlik doğrulama, şifreleme ve erişim kontrolü sorunlarını tarar. |
| [`/document-release`](#document-release) | **Teknik Yazar** | Yayımladığınız şeyle eşleşmesi için tüm proje belgelerini güncelleyin. Eskimiş README'leri otomatik yakalar. |
| [`/document-generate`](#document-generate) | **Teknik Yazar** | Koddan bir özellik için Diataxis belgeleri (eğitim / nasıl yapılır / referans / açıklama) üretin. |
| [`/retro`](#retro) | **Mühendislik Müdürü** | Ekip farkındalı haftalık geriye bakış. Kişi başına dağılımlar, gönderi serileri, test sağlık eğilimleri, büyüme fırsatları. |
| [`/browse`](#browse) | **QA Mühendisi** | Aracıya gözler verin. Gerçek Chromium tarayıcı, gerçek tıklamalar, gerçek ekran görüntüleri. Komut başına ~100ms. |
| [`/setup-browser-cookies`](#setup-browser-cookies) | **Oturum Yöneticisi** | Gerçek tarayıcınızdan (Chrome, Arc, Brave, Edge) çerezleri başsız oturuma aktarın. Kimlik doğrulamalı sayfaları test edin. |
| [`/autoplan`](#autoplan) | **İnceleme Ardışık Düzeni** | Tek komut, tamamen incelenmiş plan. CEO → tasarım → mühendislik → DX incelemesini kodlanmış karar ilkelleriyle otomatik çalıştırır. Yalnızca zevk kararlarını onayınız için sunar. |
| [`/plan-devex-review`](#plan-devex-review) | **DX Gözden Geçiren** | Plan aşaması DX incelemesi. TTHW (merhaba-dünyaya-kadar-süre), büyülü anlar, sürtünme noktaları, kişi izleri. Üç mod: Genişletme, Cilalama, Önceliklendirme. |
| [`/devex-review`](#devex-review) | **DX Gözden Geçiren (canlı)** | Canlı geliştirici deneyimi denetimi. Gerçek katılım akışını yürür, TTHW'yi ölçer, belge yalanlarını yakalar. |
| [`/plan-tune`](#plan-tune) | **Soru Ayarlayıcı** | AskUserQuestion duyarlılığını soru başına ayarlayın. Soruları asla-sorulmayacak, her zaman-sorulacak veya yalnızca-tek-yön için olarak işaretleyin. |
| [`/learn`](#learn) | **Bellek** | gstack'in oturumlar arası öğrendiklerini yönetin. Projeye özgü örüntüleri ve tercihleri inceleyin, arayın, budayın ve dışa aktarın. |
| [`/context-save`](#context-save) | **Durum Kaydet** | Çalışma bağlamını (git durumu, kararlar, kalan iş) kaydedin, böylece herhangi bir gelecek oturumu devam edebilir. |
| [`/context-restore`](#context-restore) | **Durum Geri Yükle** | Kaydedilmiş bir bağlamdan devam edin, hatta Conductor çalışma alanı el sıkışmaları arasında bile. |
| [`/health`](#health) | **Kod Kalitesi Panosu** | Tür denetleyici, linter, testler, ölü kod algılama sarmallar. Ağırlıklı 0-10 puanı hesaplar; zaman içindeki eğilimleri izler. |
| [`/landing-report`](#landing-report) | **Yayınlama Sırası Panosu** | Çalışma alanı farkında yayınlama sırasının salt okunur anlık görüntüsü. Hangi sürüm yuvaları alınmış, hangi kardeş çalışma alanlarında devam eden iş var. |
| [`/benchmark-models`](#benchmark-models) | **Model Karşılaştırma** | Yetenekler için yan yana çapraz model karşılaştırma (Claude vs GPT vs Gemini). Gecikme, belirteçler, maliyet, isteğe bağlı LLM-hakemli kalite. |
| | | |
| **Çoklu AI** | | |
| [`/codex`](#codex) | **İkinci Görüş** | OpenAI Codex CLI'dan bağımsız inceleme. Üç mod: kod incelemesi (geç/kal kapısı), sertlik meydan okuması ve oturum sürekliliği ile açık danışma. Hem `/review` hem `/codex` çalıştırıldığında çapraz model analizi. |
| [`/pair-agent`](#pair-agent) | **Uzak Aracı Köprüsü** | Uzak bir AI aracısını (OpenClaw, Codex, Cursor, Hermes) tarayıcınızla eşleştirin. Kapsamlı tünel, kilitli izin verilenler listesi, oturum belirteci. |
| [`/setup-gbrain`](#setup-gbrain) | **Bellek Eşitleme** | Makineler arası oturum bellek eşitlemesi için gbrain kurun. Sıfırdan canlıya tek komut. |
| [`/sync-gbrain`](#sync-gbrain) | **Brain'i Güncel Tut** | gbrain'ı bu deponun koduna karşı yenileyin; aracıya `gbrain search`/`code-def` ne zaman kullanacağını öğretin. Etkisiz; yeniden çalıştırmak güvenli. |
| | | |
| **Güvenlik ve Yardımcılar** | | |
| [`/careful`](#safety--guardrails) | **Güvenlik Korkulukları** | Yıkıcı komutlardan önce uyarır (rm -rf, DROP TABLE, force-push, git reset --hard). Herhangi bir uyarıyı geçersiz kılın. Yaygın yapı temizlikleri beyaz listeye alınır. |
| [`/freeze`](#safety--guardrails) | **Düzenleme Kilidi** | Tüm dosya düzenlemelerini tek bir dizinle kısıtlayın. Dışarıdaki Edit ve Write'ı engeller. Hata ayıklama için kaza önleme. |
| [`/guard`](#safety--guardrails) | **Tam Güvenlik** | /careful + /freeze'yi tek komutta birleştirir. Üretim çalışması için maksimum güvenlik. |
| [`/unfreeze`](#safety--guardrails) | **Kilit Aç** | /freeze sınırını kaldırır, her yerde düzenlemelere tekrar izin verir. |
| [`/open-gstack-browser`](#open-gstack-browser) | **GStack Tarayıcı** | Kenar çubuğu, bot-karşıtı gizlilik, otomatik model yönlendirmesi, çerez içe aktarma ve Claude Code entegrasyonu ile GStack Tarayıcı başlatın. Her eylemi canlı izleyin. |
| [`/setup-deploy`](#setup-deploy) | **Dağıtım Yapılandırıcı** | `/land-and-deploy` için tek seferlik kurulum. Platformunuzu, üretim URL'nizi ve dağıtım komutlarınızı algılar. |
| [`/gstack-upgrade`](#gstack-upgrade) | **Kendi Güncelleyicisi** | gstack'i en son sürüme yükseltin. Genel vs satıcı kurulumunu algılar, her ikisini senkronize eder, neyin değiştiğini gösterir. |
| [`/make-pdf`](#make-pdf) | **PDF Üreteci** | Herhangi bir markdown dosyasını yayın kalitesinde PDF'ye dönüştürün. Düzgün kenar boşlukları, sayfa numaraları, kapak sayfaları, tıklanabilir içindekiler. |
| [`/ios-qa`](#ios-qa) | **iOS QA Lideri** | USB CoreDevice tüneli + gömülü StateServer üzerinden canlı cihaz iOS QA. Swift kaynağını okur, erişimcileri kod üretir, gerçek iPhone'u sürer. İsteğe bağlı olarak cihazı uzak aracılar için Tailscale üzerinden kullanıma sunar. |
| [`/ios-fix`](#ios-fix) | **iOS Otonom Düzeltici** | Gerçek bir iPhone'da bul→düzelt→doğrula döngüsünü kapatır. Üreten bir anlık görüntüyü yakalar, kaynağı düzeltir, yeniden derler, yeniden dağıtır, doğrular. |
| [`/ios-design-review`](#ios-design-review) | **iOS Tasarımcı Gözü** | Gerçek bir iPhone'da 10 boyutlu Apple HIG denetimi. Her ekranı puanlar, 10'un nasıl görüneceğini söyler. |
| [`/ios-clean`](#ios-clean) | **iOS Köprü Temizliği** | DebugBridge SPM + `#if DEBUG` bağlantılamasını çıkarmak için kolaylık sarmalayıcı. Yapısal Release-derleme koruması Package.swift + CI'dadır; bu yetenek yönlendirmeli el ile kaldırmalar içindir. |
| [`/ios-sync`](#ios-sync) | **iOS Köprü Yeniden Senkronizasyonu** | En son yukarı akış gstack karşı erişimcileri ve Swift şablonlarını yeniden üretir. Yeni `@Observable` sınıfları eklediğinizde veya gstack'i yükselttiğinizde çalıştırın. |

---

## `/office-hours`

Her proje buradan başlamalı.

Planlamadan önce, incelemeden önce, kod yazmadan önce — bir YC tarzı ortakla oturun ve
neyi inşa ettiğinizi gerçekten düşünün. Ne inşa ettiğinizi düşündüğünüzü değil. *Gerçekte*
neyi inşa ettiğinizi.

### Yeniden çerçeveleme

Gerçek bir projede olan şey. Kullanıcı şunu söyledi: "Takvimim için günlük brifing uygulaması
inşa etmek istiyorum." Makul bir istek. Sonra acıyı sordu — spesifik örnekler, varsayımsal
değil. Bir asistanın şeyleri kaçırdığını, eski bilgilerle birden fazla Google Hesabı takvim
öğelerini, AI çöpü olan hazırlık belgelerini, yanlış konumları olan ve izlenmesi sonsuz süre
alan etkinlikleri açıkladılar.

Şununla geri geldi: *"Çerçevelemeye meydan okuyacağım çünkü çerçevesini aşmış olduğunuzu
düşünüyorum. 'Çok Google Takvim yönetimi için günlük brifing uygulaması' dediniz. Ama
gerçekte açıkladığınız şey bir kişisel kabin şefi AI."*

Ardından kullanıcının açıkladığını fark etmediği beş yeteneği çıkardı:

1. **Takviminizi izler** tüm hesaplar arasında ve eski bilgileri, eksik konumları, izin boşluklarını algılar
2. **Gerçek hazırlık işi üretir** — lojistik özetleri değil, ama bir yönetim kurulu toplantısı, bir podcast,
   bir bağış toplama etkinliği için *entelektüel iş*
3. **CRM'inizi yönetir** — kiminle buluştuğunuz, ilişki ne, ne istiyorlar, geçmiş ne
4. **Zamanınızı önceliklendirir** — hazırlığın erken başlaması gerektiğini işaretler, proaktif olarak zaman bloklar,
   öneme göre etkinlikleri sıralar
5. **Parayı kaldıraç için kullanır** — devretmek veya otomatikleştirmek için aktif olarak yollar arar

O yeniden çerçeveleme tüm projeyi değiştirdi. Bir takvim uygulaması inşa etmek üzereydiler. Şimdi
on kat daha değerli bir şey inşa ediyorlar — çünkü yetenek özellik isteği yerine acılarını dinledi.

### Önkabul meydan okuması

Yeniden çerçevelendikten sonra, doğrulamanız için önkabulları sunar. "Bu kulağa hoş geliyor
mu?" değil — ürün hakkında yanlışlanabilir iddialar:

1. Takvim bağ veri kaynağıdır, ama değer üstteki zeka katmanındadır
2. Asistan değiştirilmez — güçlendirilir
3. En dar kama, gerçekten çalışan bir günlük brifingdir
4. CRM entegrasyonu iyi-olması-gereken, değil iyi-olursa-daha-iyi

Kabul edersiniz, reddedersiniz veya ayarlarsınız. Kabul ettiğiniz her önkabul, tasarım belgesinde
yük taşıyan olur.

### Uygulama alternatifleri

Ardından dürüst çaba tahminleriyle 2-3 somut uygulama yaklaşımı üretir:

- **Yaklaşım A: Önce Günlük Brifing** — en dar kama, yarın gönderilir, M çaba (insan: ~3 hafta / CC: ~2 gün)
- **Yaklaşım B: Önce CRM** — önce ilişki grafiğini inşa et, L çaba (insan: ~6 hafta / CC: ~4 gün)
- **Yaklaşım C: Tam Vizyon** — her şey bir anda, XL çaba (insan: ~3 ay / CC: ~1.5 hafta)

Gerçek kullanımdan öğrendiğiniz için A'yı önerir. CRM verisi doğal olarak ikinci haftada gelir.

### İki mod

**Başlangıç modu** — bir iş inşa eden kurucular ve girişimciler için. YC ortaklarının ürünleri
değerlendirdiği şekilde damıtılmış altı zorlayıcı soru alırsınız: talep gerçekliği, durum quo,
çaresiz spesifiklik, en dar kama, gözlem ve sürpriz, geleceğe uyum. Bu sorular kasıtlı olarak
rahatsız edicidir. Ürününüze ihtiyaç duyan spesifik bir insanı adlandılayamıyorsanız, herhangi
bir kod yazmadan önce öğrenmeniz gereken en önemli şey budur.

**İnşa edici modu** — hackathonlar, yan projeler, açık kaynak, öğrenme ve eğlenme için.
Fikrinizin en havalı versiyonunu bulmanıza yardımcı olan hevesli bir işbirlikçi alırsınız.
Birinin "vay" demesini sağlayacak ne olurdu? Paylaşabileceğiniz bir şeye en hızlı yol nedir?
Sorular sorgulayıcı değil, üreticidir.

### Tasarım belgesi

Her iki mod da `~/.gstack/projects/` dizinine yazılan bir tasarım belgesiyle biter — ve o belge
doğrudan `/plan-ceo-review` ve `/plan-eng-review` beslenir. Tam yaşam döngüsü şudur:
`office-hours → plan → implement → review → QA → ship → retro`.

Tasarım belgesi onaylandıktan sonra, `/office-hours` oturum sırasında söylediğiniz şeyler
hakkında ne fark ettiğini yansıtır — genel övgü değil, belirli geri çağırmalar. Gözlemler
tasarım belgesinde de görünür, bu nedenle daha sonra tekrar okuduğunuzda onlara tekrar
rastlarsınız.

---

## `/plan-ceo-review`

Bu benim **kurucu modum**.

Burada modelin zevk, hırslar, kullanıcı empatisi ve uzun zaman ufkuyla düşünmesini istiyorum.
İsteği harfi harfine almasını istemiyorum. Önce daha önemli bir soru sormasını istiyorum:

**Bu ürün gerçekten ne için?**

Bunu **Brian Chesky modu** olarak düşünüyorum.

Amaç bariz bileti uygulamak değildir. Amaç, problemi kullanıcının bakış açısından yeniden
düşünmek ve kaçınılmaz, büyüleyici ve belki biraz büyülü hissettiren versiyonu bulmaktır.

### Örnek

Craigslist tarzı bir ilan uygulaması inşa ettiğimi ve şunu söylediğimi varsayalım:

> "Satıcılara ürünleri için fotoğraf yüklemelerine izin verelim."

Zayıf bir asistan bir dosya seçici ekleyecek ve bir resim kaydedecek.

Gerçek ürün bu değil.

`/plan-ceo-review`'da, modelin "fotoğraf yükleme"nin bile özellik olup olmadığını sormasını
istiyorum. Belki gerçek özellik birinin gerçekten satan bir ilan oluşturmasına yardımcı olmaktır.

Gerçek iş buysa, tüm plan değişir.

Şimdi model şunu sormalı:

* Ürürü fotoğraftan tanıyabilir miyiz?
* SKU veya model numarasını çıkarabilir miyiz?
* Web'de arayıp başlığı ve açıklamayı otomatik olarak taslaklayabilir miyiz?
* Özellikleri, kategoriyi ve fiyat karşılaştırmalarını çekebilir miyiz?
* Hangi fotoğrafın kahraman görseli olarak en iyi dönüşümü sağlayacağını önerebilir miyiz?
* Yüklenen fotoğrafın çirkin, karanlık, dağınık veya düşük güvenilir olduğunu algılayabilir miyiz?
* Deneyimi 2007'den kalma ölü bir form yerine premium hissettirebilir miyiz?

`/plan-ceo-review` benim için bunu yapar.

Sadece "bu özelliği nasıl eklerim?" diye sormaz.
**"Bu isteğin içinde saklı 10 yıldızlı ürün nedir?"** diye sorar.

### Dört mod

- **KAPSAM GENİŞLETME** — büyük düşün. Aracı iddialı versiyonu önerir. Her genişletme,
  kabul etmeye karar verdiğiniz bireysel bir karar olarak sunulur. Coşkuyla önerir.
- **SEÇİCİ GENİŞLETME** — mevcut kapsamınızı temel olarak tutun, ama başka nelerin mümkün
  olduğunu görün. Aracı fırsatları nötr önerilerle birer birer ortaya çıkarır — değerli
  olanları seçersiniz.
- **KAPSAMI TUT** — mevcut plan üzerinde maksimum titizlik. Genişletme yok.
- **KAPSAM DARALTMA** — minimum uygulanabilir versiyonu bulun. Diğer her şeyi kesin.

Vizyonlar ve kararlar `~/.gstack/projects/` dizinine kalıcı hale getirilir, böylece konuşmanın
ötesinde hayatta kalır. Olağanüstü vizyonlar, ekip için deponuzdaki `docs/designs/` dizinine
yükseltilebilir.

---

## `/plan-eng-review`

Bu benim **mühendislik müdürü modum**.

Ürün yönü doğru olduğunda, tamamen farklı bir zeka türü istiyorum. Daha fazla yayılan
fikir üretimi istemiyorum. "Harika olmaz mıydı"yı istemiyorum. Modelin en iyi teknik
liderim olmasını istiyorum.

Bu mod şunları kapatmalı:

* mimari
* sistem sınırları
* veri akışı
* durum geçişleri
* başarısızlık modları
* uç durumlar
* güven sınırları
* test kapsamı

Ve benim için şaşırtıcı derecede büyük bir kilidin açılması: **diyagramlar**.

LLM'ler sistemi çizmeye zorladığınızda çok daha tam hale gelir. Sıra diyagramları, durum
diyagramları, bileşen diyagramları, veri akış diyagramları, hatta test matrisleri. Diyagramlar
gizli kabulleri açıkça çıkarır. El ile sallanan planlamayı çok daha zor hale getirir.

Yani `/plan-eng-review`, ürün vizyonunu taşıyabilecek teknik omurgayı inşa etmemi istediğim yerdir.

### Örnek

Aynı ilan uygulaması örneğini alın.

`/plan-ceo-review` işini zaten yaptı. Gerçek özelliğin sadece fotoğraf yükleme olmadığına
karar verdik. Akıllı bir ilan akışı:

* fotoğrafları yükler
* ürünü tanımlar
* web'den ilan zenginleştirir
* güçlü bir başlık ve açıklama taslarlar
* en iyi kahraman görselini önerir

Şimdi `/plan-eng-review` devralır.

Şimdi modelin şöyle sorular yanıtlamasını istiyorum:

* Yükleme, sınıflandırma, zenginleştirme ve taslak üretimi için mimari nedir?
* Hangi adımlar senkron, hangileri arka plan işlerine gider?
* Uygulama sunucusu, nesne depolama, görüntü modeli, arama/zenginleştirme API'leri ve ilan veritabanı arasındaki sınırlar nelerdir?
* Yükleme başarılı olur ama zenginleştirme başarısız olursa ne olur?
* Ürün tanımlaması düşük güvenilirlikteyse ne olur?
* Yeniden denemeler nasıl çalışır?
* Yinelenen işleri nasıl önleriz?
* Ne zaman ne kalıcı hale gelir ve ne güvenle yeniden hesaplanabilir?

Ve burada diyagramlar istiyorum — mimari diyagramlar, durum modelleri, veri akış diyagramları,
test matrisleri. Diyagramlar gizli kabulleri açıkça çıkarır. El ile sallanan planlamayı çok
daha zor hale getirir.

İşte `/plan-eng-review` budur.

"Fikri küçültmek" değil.
**Fikri inşa edilebilir hale getirmek.**

### İnceleme Hazırlığı Panosu

Her inceleme (CEO, Mühendislik, Tasarım) sonucunu günlüğe kaydeder. Her inceleme sonunda
bir pano görürsünüz:

```
+====================================================================+
|                    İNCELEME HAZIRLIĞI PANOSU                       |
+====================================================================+
| İnceleme          | Çalıştırma | Son Çalıştırma            | Durum    | Gerekli |
|-----------------|------|---------------------|-----------|----------|
| Mühendislik İncelemesi      |  1   | 2026-03-16 15:00    | TEMIZ     | EVET      |
| CEO İncelemesi      |  1   | 2026-03-16 14:30    | TEMIZ     | hayır      |
| Tasarım İncelemesi   |  0   | —                   | —         | hayır      |
+--------------------------------------------------------------------+
| KARAR: TEMIZLENDİ — Mühendislik İncelemesi geçti                                |
+====================================================================+
```

Mühendislik İncelemesi gerekli tek geçittir (`gstack-config set skip_eng_review true` ile
devre dışı bırakılabilir). CEO ve Tasarım bilgilendirme amaçlıdır — sırasıyla ürün ve UI
değişiklikleri için önerilir.

### Plandan QA'ya akış

`/plan-eng-review` test incelemesi bölümünü bitirdiğinde, `~/.gstack/projects/` dizinine bir
test planı yapıtı yazar. Daha sonra `/qa` çalıştırdığınızda, o test planını otomatik olarak
alır — mühendislik incelemeniz el ile kopyala-yapıştır olmadan doğrudan QA testine beslenir.

---

## `/plan-design-review`

Bu benim **kıdemli tasarımcı planınızı inceleyen modum** — tek bir satır kod yazmadan.

Çoğu plan, arka ucun ne yaptığını açıklar ama kullanıcının gerçekte ne gördüğünü asla
belirtmez. Boş durumlar? Hata durumları? Yüklenme durumları? Mobil düzen? AI çöpü riski?
Bu kararlar "uygulama sırasında çözülür" diye ertelenir — ve ardından bir mühendis boş durum
olarak "Öğe bulunamadı." gönderir çünkü kimse daha iyi bir şey belirtmemiştir.

`/plan-design-review` planlama sırasında, düzeltilmesi ucuzken, tüm bunları yakalar.

`/plan-ceo-review` ve `/plan-eng-review` gibi çalışır — etkileşimli, bir seferde bir sorun,
**DUR + AskUserQuestion** örüntüsü ile. Her tasarım boyutunu 0-10 puanlar, 10'un nasıl
göründüğünü açıklar, ardından planı oraya taşımak için düzenler. Puanlama işi yönlendirir:
düşük puan = çok sayıda düzeltme, yüksek puan = hızlı geçiş.

Plan üzerinde yedi geçiş: bilgi mimarisi, etkileşim durumu kapsamı, kullanıcı yolculuğu,
AI çöpü riski, tasarım sistemi hizalaması, duyarlılık/erişilebilirlik ve çözülmemiş tasarım
kararları. Her geçişte boşluklar bulur ve ya doğrudan düzeltir (bariz olanlar) ya da bir tasarım
seçimi yapmanızı ister (gerçek ödünleşimler).

### Örnek

```
Siz:   /plan-design-review

Claude: İlk Tasarım Puanı: 4/10

        "Bu plan bir kullanıcı panosu açıklıyor ama asla kullanıcının
         ilk ne gördüğünü belirtmiyor. 'Kart ve simgeler ile' diyor —
         bu her SaaS şablonu gibi görünüyor. Sıfır yüklenme durumu,
         sıfır boş durum ve mobil davranıştan bahsetmiyor."

        Geçiş 1 (Bilgi Mimarisi): 3/10
        "10, her ekran için birincil/ikincil/üçüncül içerik
         hiyerarşisini tanımlardı."
        → Planı bilgi hiyerarşisi bölümü eklendi

        Geçiş 2 (Etkileşim Durumları): 2/10
        "Plan 4 UI özelliğine sahip ama 20 etkileşim durumundan 0'ını
         belirtiyor (4 özellik × 5 durum each)."
        → Planı etkileşim durumu tablosu eklendi

        Geçiş 4 (AI Çöpü): 4/10
        "Plan 'kart ve simgeler ile temiz, modern UI' ve
         'gradyan ile kahraman bölümü' diyor. Bunlar en iyi 2
         AI tarafından üretilmiş görünümlü örüntüdür."
        → UI açıklamaları spesifik, kasıtlı alternatiflerle yeniden yazıldı

        Genel: 4/10 → 8/10 düzeltmelerden sonra
        "Plan tasarım açısından tamamlanmış. Uygulama sonrası görsel QA için
         /design-review çalıştırın."
```

Yeniden çalıştırdığınızda, zaten 8+ olan bölümler hızlı bir geçiş alır. 8'in altındaki
bölümler tam muamele alır. Uygulama sonrası canlı site görsel denetimleri için
`/design-review` kullanın.

---

## `/design-consultation`

Bu benim **tasarım ortağım modum**.

`/plan-design-review` zaten var olan bir siteyi denetler. `/design-consultation` henüz hiçbir
şeyiniz olmadığında içindir — tasarım sistemi yok, yazı tipi seçimleri yok, renk paleti yok.
Sıfırdan başlıyorsunuz ve kıdemli bir tasarımcının sizinle tüm görsel kimliği birlikte
inşa etmek için oturmasını istiyorsunuz.

Bu bir form değil, bir sohbet. Aracı ürününüz, kullanıcılarınız ve kitleniz hakkında sorar.
Ürününüzün iletişmesi gereken şeyi — güven, hız, zanaat, sıcaklık, neyse uyan şey — hakkında
düşünür ve bunu somut seçimlere geriye doğru çalışır. Ardından tam, tutarlı bir tasarım sistemi
önerir: estetik yön, tipografi (belirli rollerle 3+ yazı tipi), onaltılık değerlerle renk paleti,
aralık ölçeği, düzen yaklaşımı ve hareket stratejisi. Her önerinin bir gerekçesi vardır. Her
seçim her diğer seçimi güçlendirir.

Ama tutarlılık masa örtüsüdür. Her geliştirici araç panosu aynı görünür — temiz sans-serif,
soluk gri, mavi vurgu. Hepsi tutarlı. Hepsi unutulabilir. "Güzel" görünen bir ürün ile
insanların gerçekten tanıdığı bir ürün arasındaki fark, **kasıtlı yaratıcı risklerdir**:
başlıklar için beklenmedik serif, kategorinizde kimsenin kullanmadığı cesir vurgu, verilerinizin
havadar yerine otoriter hissettiren daha sıkı aralık.

`/design-consultation` gerçekten bundan ibarettir. Yalnızca güvenli bir sistem önermez. Güvenli
seçimler VE riskler önerir — ve hangisinin hangisi olduğunu size söyler. "Kategorinizde okuryazar
kalmayı sağlayan seçimler bunlar. Ve burada gelenekten kopmanız gerektiğini düşünüyorum, ve
nedenini." Hangi riskleri alacağınızı seçersiniz. Aracı tüm sistemin her iki şekilde de tutarlı
olduğunu denetler.

İsterseniz, aracı alanınızda zaten ne olduğunu araştırır — gerçek sitilerin ekran görüntülerini
alır, yazı tiplerini ve renklerini ve aralıklarını analiz eder — böylece seçimler yapmadan önce
manzarayı görebilirsiniz. Bu kopyalamakla ilgili değil. Bu, topun yakınında olmayı sağlamakla
ilgili, böylece geleneklerin ne olduğunu biliyorsunuz ve sonra hangilerinin kırmanın değerli
olduğuna karar veriyorsunuz.

Sistem üzerinde anlaştıktan sonra, etkileşimli bir HTML önizleme sayfası üretir — sadece renk
örnekleri ve yazı tipi örnekleri değil, gerçekçi ürün sayfaları. Bir panosu inşa ediyorsanız,
kenar çubuğu, veri tabloları ve istatistik kartları olan bir panosu görürsünüz. Bir pazarlama
sitesi inşa ediyorsanız, gerçek kopya ve bir CTA ile bir kahraman bölümü görürsünüz. Hepsi
tasarım sisteminizde, ürün adınızla, açık ve karanlık modda işlenir. Tek bir üretim kodu satırı
yazılmadan ürününüzün nasıl hissettirebileceğini görürsünüz.

Ardından depo kökünüzdeki `DESIGN.md` dosyasına yazar — projenizin tasarım gerçek kaynağı —
ve `CLAUDE.md` dosyasını günceller, böylece her gelecek Claude Code oturumu sistemi
kabul eder. Bu noktadan itibaren `/design-review` buna karşı denetim yapabilir ve ön ucnuzda
çalışan herhangi bir aracı kuralları bilir.

### Örnek

```
Siz:   /design-consultation

Claude: README'den bu bir mühendislik ekipleri için geliştirici analitik panosu
        gibi görünüyor. Doğru mu? İstediğiniz hissiyati söyleyin — ve önce
        manzarayı görmemi ister misiniz?

Siz:    Evet, mühendislik müdürleri için. Bana ne var neyi göster.

Claude: [Linear, Vercel, Datadog, Posthog'u tarar — ekran görüntüleri alır]

        İşte manzara. Hepsi temiz sans-serif ve soluk mavi-gri paletlerde
        birleşiyor. Linear karanlık-öncelikli ve sıkı aralıkla öne çıkıyor.
        Geri kalanı birbirinin yerine kullanılabilir.

        İşte önerim — ve nerede güvenli oynayıp risk alacağım:

        ESTETIK: Endüstriyel/Faydacı — veri-önce, her piksel yerini hak ediyor
        TIPOGRAFI: Geist (gövde + UI), Instrument Serif (görüntü), JetBrains Mono (veri)
        RENK: Tutumlu — kumlu nötrler, tek teal vurgu (#0D9488)
        ARALIK: 4px temel, kompakt yoğunluk
        DUZEN: Izgara-disiplinli, 12-sütun
        HAREKET: Minimal-işlevsel — yalnızca kavramayı yardımcı olan geçişler

        GUVENLI SECIMLER (kategori beklentilerini karşıla):
          Gövde için Geist — kullanıcılarınız bu yazı tipini zaten Vercel'den biliyor
          Kumlu nötrler — veri-ağır UI'lar için sıcak gri hissi doğru
          Izgara-disiplinli düzen — panolar için öngörülebilir hizalama gerekir

        RISKLER (öne çıkacağınız yerler):
          Görüntü başlıkları için Instrument Serif — geliştirici araçlarında
            kimse serif kullanmıyor. Anında hiyerarşi kontrastı yaratır ve
            sans-serif dolu bir alanda "zanaat önemsediğimizi" sinyali verir.
          Mavi yerine tek teal vurgu — bu alanda diğerleri mavi kullanıyor.
            Teal, güvenilir hissettirecek kadar yakın, tanınabilir olacak kadar farklı.
          4px kompakt yoğunluk — Linear'den daha sıkı, "ekranınızın
            gayrimenkulüne saygı duyuyoruz" diyor.

        Güvenli seçimler sizi okuryazar tutar. Riskler, ürününüzün
        kendi yüzünü aldığı yerler. Hangi riskler ilginizi çekiyor?

Siz:    Serif riskini seviyorum. Görelim.

Claude: Önizleme tarayıcınızda açıldı.
        [/tmp/design-consultation-preview-1710000000.html]

        Sayfa gerçekçi bir panosu gösteriyor: kenar çubuğu navigasyonu, avatarlı başlık,
        istatistik kartları, veri tablosu — hepsi tasarım sisteminizde.
        Kahraman ölçeğinde Instrument Serif, gövde metninde Geist, veri tablosunda
        JetBrains Mono. Düğmeler ve etkin durumlarda teal vurgu.
        Köşede açık ve karanlık mod geçişi.

        Nasıl hissettiriyor? DESIGN.md yazmadan önce bir şey ayarlamak ister misiniz?

Siz:    Gönder.

Claude: DESIGN.md yazıldı (tipografi, renk, aralık, düzen, hareket).
        CLAUDE.md tasarım sistemi yönergeleriyle güncellendi.
        Tüm gelecek oturumları bu seçimleri kabul edecek.
```

---

## `/design-review`

Bu benim **kodlayan tasarımcı modum**.

`/plan-design-review` uygulamadan önce planınızı inceler. `/design-review` uygulamadan
sonra canlı siteyi denetler ve düzeltir.

Canlı sitenizde 80 maddelik bir görsel denetim çalıştırır, ardından bir düzeltme döngüsüne
girer: her tasarım bulgusu için kaynak dosyayı bulur, minimum CSS/stillendirme değişikliğini
yapar, `style(design): FINDING-NNN` ile commit eder, doğrulamak için yeniden gezinir ve
öncesi/sonrası ekran görüntüleri alır. Düzeltme başına bir commit, tamamen ikiye ayrılabilir.

Kendi kendini düzenleme sezgisi tasarım çalışması için ayarlanmıştır — yalnızca CSS
değişiklikleri serbest geçiş alır (doğası gereği güvenli ve geri alınabilir), ancak bileşen
JSX/TSX dosyalarındaki değişiklikler risk bütçesine karşı sayılır. 30 düzeltmede sert üst sınır.
Risk puanı %20'yi aşarsa, durur ve sorar.

### Örnek

```
Siz:   /design-review https://myapp.com

Claude: [Canlı sitede tam 80 maddelik görsel denetim çalıştırır]
        Tasarım Puanı: C  |  AI Çöpü Puanı: D
        12 bulgu (4 yüksek, 5 orta, 3 cilalama)

        9 tasarım sorunu düzeltiliyor...

        style(design): FINDING-001 — 3 sütunlu simge ızgarasını asimetrik düzen ile değiştir
        style(design): FINDING-002 — başlık ölçeği 48/32/24/18/16 ekle
        style(design): FINDING-003 — gradyan kahramanı kaldır, cesir tipografi kullan
        style(design): FINDING-004 — başlıklar için ikinci yazı tipi ekle
        style(design): FINDING-005 — öğe rolüne göre border-radius çeşitle
        style(design): FINDING-006 — gövde metnini sola hizala, başlıklar için merkezi ayır
        style(design): FINDING-007 — tüm etkileşimli öğelere hover/odak durumları ekle
        style(design): FINDING-008 — prefers-reduced-motion medya sorgusu ekle
        style(design): FINDING-009 — gövde metni için maksimum içerik genişliğini 680px olarak ayarla

        Son denetim:
        Tasarım Puanı: C → B+  |  AI Çöpü Puanı: D → A
        9 düzeltme uygulandı (8 doğrulanmış, 1 en-iyi-çaba). 3 ertelendi.

        [Öncesi/sonrası ekran görüntüleri ile rapor .gstack/design-reports/ dizinine kaydedildi]
```

Dokuz commit, her biri bir endişeyi etkiliyor. AI Çöpü puanı D'den A'ya gitti çünkü en
tanınabilir üç örüntü (gradyan kahraman, 3 sütunlu ızgara, tekdüze yarıçap) gitti.

---

## `/design-shotgun`

Bu benim **tasarım keşif modum**.

O hissi biliyorsunuz. Bir özellik, bir sayfa, bir açılış ekranınız var... ve neye
benzemesi gerektiğinden emin değilsiniz. Claude'a açıklayabilir ve bir yanıt alabilirsiniz.
Ama bir yanıt bir perspektif demektir ve tasarım bir zevk işidir. Seçenekleri görmeniz gerekir.

`/design-shotgun`, GPT Image API kullanarak 3 görsel tasarım varyantı üretir, tarayıcınızda
bir karşılaştırma panosu açar ve geri bildiriminizi bekler. Bir yön seçersiniz, değişiklik
isteyebilir veya tamamen yeni varyantlar isteyebilirsiniz. Pano remiks, yeniden üret ve onaylama
eylemlerini destekler.

### Döngü

1. Ne istediğinizi açıklarsınız (veya var olan bir sayfayı işaret edersiniz)
2. Yetenek marka kısıtlamaları için `DESIGN.md` dosyanızı okur (varsa)
3. PNG olarak 3 farklı tasarım varyantı üretir
4. 3'ünü yan yana karşılaştırma panosu tarayıcınızda açar
5. Beğendiğiniz üzerinde "Onayla"yı tıklarsınız veya başka bir tur için geri bildirim verirsiniz
6. Onaylanan varyant, bir `approved.json` ile `~/.gstack/projects/$SLUG/designs/` dizinine kaydedilir

O `approved.json`, `/design-html`'i beslemenin bir yoludur. Tasarım ardışık düzeni zincirlenir:
shotgun yönü seçer, design-html onu çalışan kod olarak işler. Ama `/design-html` CEO planları,
tasarım incelemeleri veya sadece bir açıklama ile de çalışır.

### Zevk belleği

Yetenek tercihlerinizi oturumlar arası hatırlar. Sürekli olarak yoğun tasarımlar yerine
minimal tasarımları tercih ederseniz, gelecek üretimlere önyargılı davranır. Bu, yapılandırdığınız
bir ayar değildir... onaylarınızdan ortaya çıkar.

### Örnek

```
Siz:   /design-shotgun — geliştirici araçları açılış sayfası için kahraman bölümü

Claude: [3 varyant üretir]
        Varyant A: Cesir tipografi, karanlık arka plan, kod parçacığı kahramanı
        Varyant B: Bölünmüş düzen, sol tarafta ürün ekran görüntüsü, sağ tarafta kopya
        Varyant C: Minimal, ortalanmış başlık, gradyan vurgu

        [localhost:PORT konumunda karşılaştırma panosu açılır]

Siz:    [Tarayıcıda Varyant A üzerinde "Onayla"yı tıklar]

Claude: Varyant A onaylandı. ~/.gstack/projects/myapp/designs/ dizinine kaydedildi
        Sonraki: bu modellemeden üretim HTML'si üretmek için /design-html çalıştırın.
```

---

## `/design-html`

Bu benim **tasarımdan-koda modum**.

Her AI kod üretim aracı statik CSS üretir. Sabit yükseklikler. Yeniden boyutlandırmada
taşan metin. Kırılma yerine akan kırılma noktaları. Çıktı tam olarak bir görünüm alanı
boyutunda doğru görünür ve diğer her boyutta kırılır.

`/design-html` bunu düzeltir. Cheng Lou (eski React çekirdeği, Midjourney ön ucu) tarafından
[Pretext](https://github.com/chenglou/pretext) kullanarak HTML üretir. Pretext, DOM ölçümü
olmadan metin düzenini hesaplayan 15KB'lık bir kütüphanedir. Metin yeniden akar. Yükseklikler
içeriğe uyum sağlar. Kartlar kendilerini boyutlandırır. Sohbet balonları daraltıp sarar.
Hepsi milisaniyenin altında, hep dinamik.

Birden fazla giriş kaynağı ile çalışır: `/design-shotgun`'dan onaylanmış bir modelleme,
`/plan-ceo-review`'dan bir CEO planı, `/plan-design-review`'dan tasarım incelemesi bağlamı,
sağladığınız bir PNG veya sadece istediğiniz şeyin açıklaması. Ne bağlamın var olduğunu
algılar ve nasıl devam etmek istediğinizi sorar.

### Akıllı API yönlendirmesi

Her sayfanın tam Pretext motoruna ihtiyacı yoktur. Yetenek tasarımı okur ve doğru araçları seçer:

- **Basit düzenler** (açılış, pazarlama): yeniden boyutlandırmada yükseklik farkında `prepare()` + `layout()`
- **Kart ızgaraları** (panosu, listeleme): kendini boyutlandıran kartlar için `prepare()` + `layout()`
- **Sohbet UI'ları**: sıfır israf pikseli ile sıkı oturan balonlar için `walkLineRanges()`
- **Editoryal düzenler**: engeller etrafında akan metin için `layoutNextLine()`
- **Karmaşık editoryal**: el ile satır işleme için `layoutWithLines()` ile tam motor

### Arıtma döngüsü

1. Onaylanmış modellemeyi `approved.json` dosyasından okur
2. Uygulama özelliklerini çıkarmak için GPT-4o görüntü kullanır (renkler, tipografi, düzen)
3. Satır içi Pretext ile kendi kendine yeten HTML üretir (15KB, sıfır ağ bağımlılığı)
4. Değişiklikleri anında görmek için bir canlı-yeniden-yükleme sunucusu başlatır
5. 3 görünüm alanında (mobil, tablet, masaüstü) düzeni doğrulamak için ekran görüntüleri alır
6. AskUserQuestion: neyin değişmesi gerekiyor?
7. Edit aracı ile cerrahi düzenlemeler (tam yeniden üretim değil)
8. "Bitti" diyene kadar tekrarla

### Çerçeve algılama

Projeniz React, Svelte veya Vue kullanıyorsa (`package.json` dosyasından algılanır), yetenek
vanilla HTML yerine bir çerçeve bileşeni üretmeyi teklif eder. Çerçeve çıktısı, satır içi
satıcı yerine `npm install @chenglou/pretext` kullanır.

### Örnek

```
Siz:   /design-html

Claude: Onaylanmış modelleme bulundu: variant-A.png (kahraman bölümü)
        Tasarım türü: basit düzen → prepare() + layout() kullanılıyor

        [Pretext ile HTML üretir, canlı-yeniden-yükleme sunucusu açar]
        [375px, 768px, 1440px'te ekran görüntüleri — hepsi doğrulandı]

        HTML localhost:3456 konumunda canlı. Tarayıcınızı yeniden boyutlandırın.
        Metin dinamik olarak yeniden akar. Ne değişmesi gerekiyor?

Siz:    Başlığı daha büyük yap ve CTA'nın altına daha fazla boşluk ekle

Claude: [font-size ve margin düzenler, tarayıcı otomatik yeniden yüklenir]
        Güncellendi. İki değişiklik uygulandı.

Siz:    Bitti

Claude: ~/.gstack/projects/myapp/designs/hero-20260330/finalized.html dizinine kaydedildi
```

---

## `/review`

Bu benim **paranoyak kıdemli mühendis modum**.

Testleri geçmek, dalın güvenli olduğu anlamına gelmez.

`/review` var, çünkü CI'de hayatta kalıp yine de üretimde yüzünüze tokat atan bir hata
sınıfı var. Bu mod hayallerini daha büyük inşa etmekle ilgili değil. Planı daha güzel hale
getirmekle ilgili değil. Şunu sormakla ilgili:

**Hala ne bozulabilir?**

Bu yapısal bir denetimdir, stil ince bir geçiş değildir. Modelin şöyle şeyler bulmasını
istiyorum:

* N+1 sorguları
* eski okumalar
* yarış koşulları
* kötü güven sınırları
* eksik dizinler
* kaçış hataları
* bozulmuş değişmezler
* kötü yeniden deneme mantığı
* gerçek başarısızlık modunu kaçıran testler
* unutulmuş enum işleyiciler — yeni bir durum veya tür sabiti ekleyin ve `/review` onu
  yalnızca değiştirdiğiniz dosyalarda değil, kod tabanınızdaki her switch ifadesi ve
  izin verilenler listesinde izler

### Önce-Düzelt

Bulgular eylem alır, sadece listelenmez. Bariz mekanik düzeltmeler (ölü kod, eski yorumlar,
N+1 sorguları) otomatik olarak uygulanır — her biri için `[AUTO-FIXED] dosya:satır Sorun → ne yapıldı`
görürsünüz. Gerçekte belirsiz sorunlar (güvenlik, yarış koşulları, tasarım kararları)
kararınız için ortaya çıkar.

### Eksiklik boşlukları

`/review` artık tam versiyonun 30 dakikadan az CC zamanına mal olduğu kısa yol
uygulamalarını işaretler. %80 çözümünü seçtiniz ve %100 çözüm bir göl ise okyanus değil,
inceleme bunu çağırır.

### Örnek

Akıllı ilan akışının uygulandığını ve testlerin yeşil olduğunu varsayalım.

`/review` hala sormalı:

* İlan fotoğraflarını veya taslak önerilerini işlerken N+1 sorgusu ekledim mi?
* İstemci tarafından sağlanan dosya meta verilerine gerçek dosyayı doğrulamak yerine güveniyorum
  muyum?
* İki sekme kapak fotoğrafı seçimini veya öğe detaylarını geçersiz kılabilir mi?
* Başarısız yüklemeler sonsuza kadar depolamada yetim dosyalar bırakır mı?
* "Tam olarak bir kahraman görseli" kuralı eşzamanlılık altında bozulabilir mi?
* Zenginleştirme API'leri kısmen başarısız olursa, zararsız bir şekilde bozulur musunuz yoksa çöp
  kaydeder misiniz?
* Taslak üretimine web verisi çekerek kazara istem enjeksiyonu veya güven sınırı sorunu mu
  yarattım?

`/review`'un amacı budur.

Burada dalkavukluk istemiyorum.
Modelin üretim olayını olmadan önce hayal etmesini istiyorum.

---

## `/investigate`

Bir şey bozulduğunda ve nedenini bilmediğinizde, `/investigate` sistematik hata ayıklayıcınızdır.
Demir Kuralını takip eder: **önce kök neden araştırması, sonra düzeltme yok.**

Tahmin edip yamalamak yerine, veri akışını izler, bilinen hata örüntülerine karşı eşleştirir
ve hipotezleri birer birer test eder. Üç düzeltme denemesi başarısız olursa, çabalayıp durmak
yerine mimariyi sorgular ve durur. Saatlerce süren "bir şey daha deneyelim" sarmalını önler.

---

## `/qa`

Bu benim **QA lider modum**.

`/browse` aracıya gözler verir. `/qa` ona bir test metodolojisi verir.

En yaygın kullanım durumu: bir özellik dalındasınız, kodlamayı yeni bitirdiniz ve her şeyin
çalıştığını doğrulamak istiyorsunuz. Sadece `/qa` deyin — git diff'inizi okur, değişikliklerinizin
hangi sayfaları ve rotaları etkilediğini tanımlar, tarayıcıyı başlatır ve her birini test eder.
URL gerekmez. Manuel test planı gerekmez.

Dört mod:

- **Fark-farkında** (özellik dallarında otomatik) — `git diff main` okur, etkilenen sayfaları
  tanımlar, özel olarak test eder
- **Tam** — tüm uygulamanın sistematik keşfi. 5-15 dakika. 5-10 iyi kanıtlanmış sorunu belgeler.
- **Hızlı** (`--quick`) — 30 saniyelik duman testi. Ana sayfa + en iyi 5 navigasyon hedefi.
- **Regresyon** (`--regression baseline.json`) — tam mod çalıştır, ardından önceki bir temel çizgiye karşı diff al.

### Otomatik regresyon testleri

`/qa` bir hatayı düzeltip doğruladığında, bozulan tam senaryoyu yakalayan bir regresyon testi
otomatik olarak üretir. Testler QA raporuna kadar tam atıf izleme içerir.

### Örnek

```
Siz:   /qa https://staging.myapp.com

Claude: [12 sayfa keşfeder, 3 form doldurur, 2 akış test eder]

        QA Raporu: staging.myapp.com — Sağlık Puanı: 72/100

        İlk 3 Sorun:
        1. KRİTİK: Ödeme formusu boş zorunlu alanlarla gönderilir
        2. YÜKSEK: Mobil navigasyon menüsü bir öğe seçtikten sonra kapanmaz
        3. ORTA: Panosu grafiği 1024px altında kenar çubuğunun üstüne biner

        [Ekran görüntüleri ile tam rapor .gstack/qa-reports/ dizinine kaydedildi]
```

**Kimlik doğrulamalı sayfaları test etme:** Gerçek tarayıcı oturumlarınızı içe aktarmak için
önce `/setup-browser-cookies` kullanın, ardından `/qa` giriş arkasındaki sayfaları test edebilir.

---

## `/ship`

Bu benim **yayınlama makinesi modum**.

Ne inşa edeceğime karar verdiğimde, teknik planı çivilediğimde ve ciddi bir inceleme
yaptığımda, daha fazla konuşma istemiyorum. Yürütme istiyorum.

`/ship` son mil içindir. Hazır bir dal içindir, ne inşa edeceğine karar vermek için değil.

Burada model beyin fırtınası ortağı gibi davramayı bırakıp disiplinli bir yayınlama mühendisi
gibi davranmalı: ana ile senkronize et, doğru testleri çalıştır, dal durumunun sağlıklı
olduğundan emin ol, repo bekliyorsa değişiklik günlüğünü veya sürümlemeyi güncelle, push et
ve PR oluştur veya güncelle.

### Test önyükleme

Projenizin bir test çerçevesi yoksa, `/ship` bir tane kurar — çalışma zamanınızı algılar,
en iyi çerçeveyi araştırır, kurar, gerçek kodunuz için 3-5 gerçek test yazar, CI/CD
(GitHub Actions) kurar ve TESTING.md oluşturur. %100 test kapsamı hedeftir — testler vibe
kodlamayı yolo kodlama yerine güvenli hale getirir.

### Kapsam denetimi

Her `/ship` çalıştırması, diff'inizden bir kod yolu haritası oluşturur, karşılık gelen
testleri arar ve kalite yıldızlarıyla bir ASCII kapsam diyagramı üretir. Boşluklar otomatik
olarak test üretilir. PR gövdiniz kapsamı gösterir: `Testler: 42 → 47 (+5 yeni)`.

### İnceleme geçidi

`/ship`, PR oluşturmadan önce [İnceleme Hazırlığı Panosunu](#inceleme-hazırlığı-panosu) denetler.
Mühendislik İncelemesi eksikse, sorar — ama sizi engellemez. Kararlar dal başına kaydedilir,
böylece asla tekrar sorulmazsınız.

İlginç iş bittiğinde ve sadece sıkıcı yayınlama işi kaldığında birçok dal ölür. İnsanlar
o kısmı erteler. AI yapmamalı.

---

## `/land-and-deploy`

Bu benim **dağıtım ardışık düzen modum**.

`/ship` PR'yi oluşturur. `/land-and-deploy` işi bitirir: birleştir, dağıt, doğrula.

PR'yi birleştirir, CI'yi bekler, dağıtımın bitmesini bekler, ardından üretimde kanarya
denetimleri çalıştırır. "Onaylandı"dan "üretimde doğrulandı"ya tek komut. Dağıtım başarısız
olursa, neyin başarısız olduğunu ve geri alınıp alınamayacağını söyler.

Yeni bir projede ilk çalıştırma, geri alınamaz bir şey yapmadan önce ardışık düzeni
doğrulamanız için bir kuru çalıştırma yürümesini tetikler. Bundan sonra, yapılandırmaya güvenir
ve doğrudan çalışır.

### Kurulum

Önce `/setup-deploy` çalıştırın. Platformunuzu (Fly.io, Render, Vercel, Netlify, Heroku,
GitHub Actions veya özel), üretim URL'nizi ve sağlık denetimi uç noktalarınızı algılar
ve yapılandırmayı CLAUDE.md dosyasına yazar. Tek seferlik, 60 saniye.

### Örnek

```
Siz:   /land-and-deploy

Claude: PR #42 birleştiriliyor...
        CI: 3/3 denetim geçti
        Dağıtım: Fly.io — v2.1.0 dağıtılıyor...
        Sağlık denetimi: https://myapp.fly.dev/health → 200 OK
        Kanarya: 5 sayfa denetlendi, 0 konsol hatası, p95 < 800ms

        Üretim doğrulandı. v2.1.0 canlı.
```

---

## `/canary`

Bu benim **dağıtım sonrası izleme modum**.

Dağıtımdan sonra, `/canary` canlı siteyi sorunlar için izler. Tarayıcı artalan sürecini
kullanarak anahtar sayfalarınızda döngü yapar, konsol hataları, performans gerilemeleri,
sayfa hataları ve görsel anormallikleri denetler. Periyodik ekran görüntüleri alır ve
dağıtım öncesi temel çizgilere karşılaştırır.

Riskli bir dağıtımdan hemen sonra `/land-and-deploy` ardından kullanın veya periyodik olarak
çalışacak şekilde zamanlayın.

```
Siz:   /canary https://myapp.com

Claude: 8 sayfayı 2 dakikada bir izliyor...

        Döngü 1: ✓ Tüm sayfalar sağlıklı. p95: 340ms. 0 konsol hatası.
        Döngü 2: ✓ Tüm sayfalar sağlıklı. p95: 380ms. 0 konsol hatası.
        Döngü 3: ⚠ /dashboard — yeni konsol hatası: "TypeError: Cannot read
                   property 'map' of undefined" at dashboard.js:142
                 Ekran görüntüsü kaydedildi.

        Uyarı: 3 izleme döngüsünden sonra 1 yeni konsol hatası.
```

---

## `/benchmark`

Bu benim **performans mühendisi modum**.

`/benchmark` sayfalarınız için performans temel çizgileri oluşturur: yükleme süresi,
Core Web Vitals (LCP, CLS, INP), kaynak sayıları ve toplam aktarım boyutu. Gerilemeleri
yakalamak için bir PR'den önce ve sonra çalıştırın.

Gerçek Chromium ölçümleri için tarayıcı artalan sürecini kullanır, sentetik tahminler değil.
Birden fazla çalıştırma ortalaması. Sonuçlar kalıcı hale gelir, böylece PR'ler arasında
eğilimleri izleyebilirsiniz.

```
Siz:   /benchmark https://myapp.com

Claude: 5 sayfa karşılaştırmalı (her biri 3 çalıştırma)...

        /           yükleme: 1.2s  LCP: 0.9s  CLS: 0.01  kaynaklar: 24 (890KB)
        /dashboard  yükleme: 2.1s  LCP: 1.8s  CLS: 0.03  kaynaklar: 31 (1.4MB)
        /settings   yükleme: 0.8s  LCP: 0.6s  CLS: 0.00  kaynaklar: 18 (420KB)

        Temel çizgi kaydedildi. Değişikliklerden sonra karşılaştırmak için tekrar çalıştırın.
```

---

## `/cso`

Bu benim **Baş Güvenlik Sorumlum**.

Herhangi bir kod tabanında `/cso` çalıştırın ve OWASP Top 10 + STRIDE tehdit modelleme denetimi
gerçekleştirir. Enjeksiyon güvenlik açıkları, bozuk kimlik doğrulama, hassas veri maruziyeti,
XML harici varlıklar, bozuk erişim kontrolü, güvenlik yanlış yapılandırması, XSS, güvensiz
serileştirme, bilinen güvenlik açığı olan bileşenler ve yetersiz günlük kaydı için tarar.
Her bulgu önem derecesi, kanıt ve önerilen düzeltme içerir.

```
Siz:   /cso

Claude: OWASP Top 10 + STRIDE güvenlik denetimi çalıştırılıyor...

        KRİTİK: Kullanıcı aramasında SQL enjeksiyonu (app/models/user.rb:47)
        YÜKSEK: Oturum belirteçleri localStorage'da saklanıyor (app/frontend/auth.ts:12)
        ORTA: /api/login uç noktasında oran sınırlaması eksik
        DÜŞÜK: X-Frame-Options başlığı ayarlanmamış

        4 bulgu, 12 dosya tarandı. 1 kritik, 1 yüksek.
```

---

## `/document-release`

Bu benim **teknik yazar modum**.

`/ship` PR'yi oluşturduktan sonra ancak birleştirmeden önce, `/document-release` projedeki her
belgelendirme dosyasını okur ve diff ile karşılaştırır. Dosya yollarını, komut listelerini,
proje yapı ağaçlarını ve kaymış olan başka her şeyi günceller. Riskli veya öznel değişiklikler
soru olarak ortaya çıkar — diğer her şey otomatik olarak halledilir.

```
Siz:   /document-release

Claude: 3 commit boyunca 21 dosya değiştirildi analiz ediliyor. 8 belgelendirme dosyası bulundu.

        README.md: yetenek sayısı 9'dan 10'a güncellendi, yeni yetenek tabloya eklendi
        CLAUDE.md: proje yapısına yeni dizin eklendi
        CONTRIBUTING.md: güncel — değişiklik gerekmedi
        TODOS.md: 2 öğe tamamlandı olarak işaretlendi, 1 yeni öğe eklendi

        Tüm belgeler güncellendi ve commit edildi. PR gövdesi belge diff ile güncellendi.
```

Ayrıca CHANGELOG sesini cilalar (asla girdilerin üzerine yazmadan), tamamlanmış TODOS'ları
temizler, çapraz-belge tutarlılığını denetler ve VERSION artışlarını yalnızca uygun olduğunda
sorar.

---

## `/retro`

Bu benim **mühendislik müdürü modum**.

Hafta sonunda gerçekte ne olduğunu bilmek istiyorum. Hisler değil — veriler. `/retro` commit
geçmişini, çalışma örüntülerini ve gönderi hızını analiz eder ve dürüst bir geriye bakış yazar.

Ekip farkındalıdır. Komutu kimin çalıştırdığını tanımlar, kendi çalışmanızda en derin
muameleyi yapar, ardından her katkıda bulunana övgü ve büyüme fırsatları ile ayrıntılı
bir dağılım verir. Commitler, LOC, test oranı, PR boyutları ve düzeltme oranı gibi ölçümleri
hesaplar. Commit zaman damgalarından kodlama oturumlarını algılar, hotspot dosyalarını bulur,
gönderi sergilerini izler ve haftanın en büyük gönderisini tanımlar.

Ayrıca test sağlığını izler: toplam test dosyaları, bu dönemde eklenen testler, regresyon
testi commitleri ve eğilim deltaları. Test oranı %20'nin altına düşerse, büyüme alanı olarak
işaretler.

### Örnek

```
Siz:   /retro

Claude: 1 Mart haftası: 47 commit (3 katkıda bulunana), 3.2k LOC, %38 test, 12 PR, zirve: 22:00 | Seri: 47g

        ## Haftanız
        32 commit, +2.4k LOC, %41 test. Zirve saatleri: 21-23.
        En büyük gönderi: çerez içe aktarma sistemi (tarayıcı şifre çözme + seçici UI).
        İyi yaptığınız şey: şifreleme, UI ve 18 birim testi ile tam bir özellik
        tek odaklı bir itmeyle gönderildi...

        ## Ekip Dağılımı

        ### Alice
        12 commit app/services/ odaklı. Her PR 200 LOC altında — disiplinli.
        Fırsat: test oranı %12 — ödeme karmaşıklaşmadan önce yatırım yapmaya değer.

        ### Bob
        3 commit — panodaki N+1 sorguyu düzeltti. Küçük ama yüksek etki.
        Fırsat: bu hafta yalnızca 1 aktif gün — herhangi bir şeyde engellenip engellenmediğini denetleyin.

        [İlk 3 ekip zaferi, geliştirilecek 3 şey, gelecek hafta için 3 alışkanlık]
```

`.context/retros/` dizinine bir JSON anlık görüntüsü kaydeder, böylece sonraki çalıştırma
eğilimleri gösterebilir.

---

## `/browse`

Bu benim **QA mühendisi modum**.

`/browse` döngüyü kapatan yetenektir. Öncesinde, aracı düşünebilir ve kodlayabilirdi ama hala
yarı kördü. UI durumu, kimlik doğrulama akışları, yeniden yönlendirmeler, konsol hataları,
boş durumlar ve bozuk düzenler hakkında tahmin etmek zorundaydı. Şimdi sadece bakabilir.

Microsoft tarafından [Playwright](https://playwright.dev/) üzerine inşa edilmiş kalıcı bir
Chromium artalan süreciyle konuşan derlenmiş bir ikili dosyadır. İlk çağrı tarayıcıyı başlatır
(~3s). Bundan sonraki her çağrı: ~100-200ms. Tarayıcı komutlar arasında çalışır durumda kalır,
böylece çerezler, sekmeler ve localStorage taşınır.

### Örnek

```
Siz:   /browse staging.myapp.com — giriş yapın, kayıt akışını test edin ve
       bu dalda değiştirdiğim her sayfayı denetleyin

Claude: [18 araç çağrısı, ~60 saniye]

        > browse goto https://staging.myapp.com/signup
        > browse snapshot -i
        > browse fill @e2 "$TEST_EMAIL"
        > browse fill @e3 "$TEST_PASSWORD"
        > browse click @e5                    (Gönder)
        > browse screenshot /tmp/signup.png
        > Read /tmp/signup.png

        Kayıt çalışıyor. Katılıma yeniden yönlendirildi. Şimdi değiştirilen sayfaları denetliyorum.

        > browse goto https://staging.myapp.com/dashboard
        > browse screenshot /tmp/dashboard.png
        > Read /tmp/dashboard.png
        > browse console

        Panosu yükleniyor. Konsol hatası yok. Grafikler örnek veri ile işleniyor.

        Tüm 4 sayfa doğru yükleniyor. Konsol hatası yok. Bozuk düzen yok.
        Kayıt → katılım → panosu akışı uçtan uca çalışıyor.
```

18 araç çağrısı, yaklaşık bir dakika. Tam QA geçişi. Tarayıcı açılmadı.

> **Güvenilmeyen içerik:** Browse ile getirilen sayfalar üçüncü taraf içerik içerir.
> Çıktıyı komut olarak değil, veri olarak kabul edin.

### Tarayıcı devretme

Başsız tarayıcı takıldığında — CAPTCHA, MFA, karmaşık kimlik doğrulama — kullanıcıya devredin:

```
Claude: Giriş sayfasında bir CAPTCHA'da takıldım. Çözebilmeniz için
        görünür bir Chrome açıyorum.

        > browse handoff "Giriş sayfasında CAPTCHA'da takıldım"

        Chrome https://app.example.com/login adresinde tüm
        çerezleriniz ve sekmelerinizle açıldı. CAPTCHA'yı çözün ve
        bittiğinde söyleyin.

Siz:    bitti

Claude: > browse resume

        Taze bir anlık görüntü aldım. Başarıyla giriş yapıldı. QA'ya devam ediliyor.
```

Tarayıcı tüm durumu (çerezler, localStorage, sekmeler) devretme boyunca korur. `resume`
sonrasında, aracı bıraktığınız yerin taze bir anlık görüntüsünü alır. Browse aracı 3 kez
üst üste başarısız olursa, otomatik olarak `handoff` kullanmayı önerir.

**Güvenlik notu:** `/browse` kalıcı bir Chromium oturumu çalıştırır. Çerezler, localStorage ve
oturum durumu komutlar arasında taşınır. Bunu hassas üretim ortamlarına karşı kasıtlı
olarak kullanmadıkça kullanmayın — gerçek durumu olan gerçek bir tarayıcıdır. Oturum 30
dakikalık boşta kalma süresinden sonra otomatik olarak kapanır.

Tam komut referansı için [BROWSER.md](../BROWSER.md) dosyasına bakın.

---

## `/setup-browser-cookies`

Bu benim **oturum yöneticim modum**.

`/qa` veya `/browse` kimlik doğrulamalı sayfaları test edebilmeden önce, çerezlere ihtiyaçları
vardır. Başsız tarayıcıda her seferinde el ile giriş yapmak yerine, `/setup-browser-cookies`
gerçek oturumlarınızı doğrudan günlük tarayıcınızdan içe aktarır.

Yüklü Chromium tarayıcılarını (Comet, Chrome, Arc, Brave, Edge) otomatik olarak algılar,
çerezleri macOS Anahtarlık üzerinden şifre çözer ve Playwright oturumuna yükler. Etkileşimli
seçici UI, hangi alanları içe aktarmak istediğinizi tam olarak seçmenize olanak tanır — çerez
değerleri asla gösterilmez.

```
Siz:   /setup-browser-cookies

Claude: Çerez seçici açıldı — tarayıcınızda içe aktarmak istediğiniz
        alanları seçin, ardından bittiğinde söyleyin.

        [Tarayıcıda github.com, myapp.com alanlarını seçersiniz]

Siz:    bitti

Claude: 2 alan içe aktarıldı (47 çerez). Oturum hazır.
```

Ya da UI'yı tamamen atlayın:

```
Siz:   /setup-browser-cookies github.com

Claude: Comet'den github.com için 12 çerez içe aktarıldı.
```

---

## `/autoplan`

Bu benim **inceleme otomatik pilot modum**.

`/plan-ceo-review`, ardından `/plan-design-review`, ardından `/plan-eng-review`'u tek tek
çalıştırmak, 15-30 ara soru yanıtlamak anlamına gelir. Her soru değerlidir, ama bazen
geçit dizisinin her karar için durmadan tamamen çalışmasını istersiniz.

`/autoplan` üç inceleme yeteneğini diskten okur ve sıralı olarak çalıştırır: CEO → Tasarım →
Mühendislik. Altı kodlanmış ilke (bütünlüğü tercih et, mevcut örüntüleri eşleştir, geri
alınabilir seçenekleri seç, benzer geçmiş kararlar için kullanıcının seçtiği seçeneği tercih
et, belirsiz öğeleri ertele ve güvenliği yükselt) kullanarak kararları otomatik olarak verir.
Zevk kararları (yakın yaklaşımlar, sınırda kapsam genişletmeleri, çapraz-model anlaşmazlıklar)
kaydedilir ve son bir onay geçidinde sunulur.

Tek komut, tamamen incelenmiş plan çıktı.

```
Siz:   /autoplan

Claude: CEO incelemesi çalıştırılıyor... [4 kapsam kararı otomatik çözüldü]
        Tasarım incelemesi çalıştırılıyor... [3 tasarım boyutu otomatik puanlandı]
        Mühendislik incelemesi çalıştırılıyor... [2 mimari kararı otomatik çözüldü]

        ZEVK KARARLARI (girdi gerekiyor):
        1. Kapsam: Codex arama eklenmesini önerdi — sınırda genişletme. Eklensin mi?
        2. Tasarım: İki yaklaşım 1 puan içinde puanlandı. Hangisi daha doğru hissettiriyor?

        [Her iki seçeneği de bağlam ile gösterir]

Siz:    1) Evet, arama ekle. 2) Seçenek A.

Claude: Plan tamamlandı. 9 karar otomatik çözüldü, 2 zevk kararı onaylandı.
```

---

## `/learn`

Bu benim **kurumsal bellek modum**.

gstack her oturumdan öğrenir. Örüntüler, tuzaklar, tercihler, mimari kararlar... bunlar
`~/.gstack/projects/$SLUG/learnings.jsonl` dosyasında birikir. Her öğrenmenin bir güven puanı,
kaynak atıfı ve başvurduğu dosyalar vardır.

`/learn` gstack'in emdiklerini görmenizi, belirli örüntüleri aramanızı, eski girdileri
budamanızı (başvurulan dosyalar artık mevcut olmadığında) ve ekip paylaşımı için öğrenmeleri
dışa aktarmanızı sağlar. Gerçek sihir diğer yeteneklerde... diğer yetenekler önerilerde
bulunmadan önce öğrenmeleri otomatik olarak arar ve geçmiş bir içgörü ilgili olduğunda
"Önceki öğrenme uygulandı" gösterir.

```
Siz:   /learn

Claude: Bu proje için 23 öğrenme (14 yüksek güven, 6 orta, 3 düşük)

        İlk örüntüler:
        - [9/10] API yanıtları her zaman { data, error } zarfında sarılı
        - [8/10] Testler test/support/factories.ts içindeki fabrika yardımcılarını kullanır
        - [8/10] Tüm DB sorguları repository örüntüsünden geçer, asla doğrudan değil

        3 muhtemelen eski (başvurulan dosyalar silindi):
        - "auth middleware JWT kullanır" — auth/middleware.ts silindi
        [Bunları budamak ister misiniz? E/H]
```

---

## `/open-gstack-browser`

Bu benim **birlikte-varlık modum**.

`/browse` varsayılan olarak başsız çalışır. Aracının ne gördüğünü görmezsiniz. `/open-gstack-browser`
bunu değiştirir. Otomatik olarak yüklenen kenar çubuğu uzantısı ile GStack Browser (bot-karşıtı
gizlilik ile yeniden markal edilmiş Chromium) başlatır. Her eylemi gerçek zamanlı olarak izlersiniz.

Kenar çubuğu sohbeti, tarayıcıyı kontrol eden bir Claude örneğidir. Doğru modele otomatik
yönlendirme yapar: navigasyon ve eylemler için Sonnet (tıkla, git, doldur, ekran görüntüsü),
okuma ve analiz için Opus. Kenar çubuğu alt bilgisinden tek tıkla çerez içe aktarma. Tarayıcı
penceresi açık olduğu sürece canlı kalır... başlı modda boşta kalma zaman aşımı yok. Menü
çubuğu "Chrome for Testing" yerine "GStack Browser" diyor.

Kenar çubuğu aracısı katmanlı bir istem enjeksiyonu savunması gönderir: yerel bir 22MB ML
sınıflandırıcısı her sayfayı ve araç çıktısını tarar, bir Haiku transkript denetimi tüm
konuşma üzerinde oy verir, bir kanarya belirteç oturum-sızdırma girişimlerini yakalar ve bir
karar birleştirici engellemeden önce iki sınıflandırıcının anlaşmasını gerektirir. Başlıktaki
bir kalkan simgesi durumu gösterir (yeşil/amber/kırmızı). Ayrıntılar için
[ARCHITECTURE.md](../ARCHITECTURE.md#prompt-injection-defense-sidebar-agent) dosyasına bakın.

```
Siz:   /open-gstack-browser

Claude: Kenar çubuğu uzantısı ile GStack Browser başlatıldı.
        Bot-karşıtı gizlilik etkin. Tüm $B komutları başlı modda çalışır.
        Tarayıcı aracısını yönlendirmek için kenar çubuğuna yazın.
        Kenar çubuğu model yönlendirmesi: eylemler için sonnet, analiz için opus.
```

---

## `/setup-deploy`

Tek seferlik dağıtım yapılandırması. İlk `/land-and-deploy` komutunuzdan önce bunu çalıştırın.

Dağıtım platformunuzu (Fly.io, Render, Vercel, Netlify, Heroku, GitHub Actions veya özel)
otomatik olarak algılar, üretim URL'nizi, sağlık denetimi uç noktalarınızı ve dağıtım durum
komutlarınızı keşfeder. Her şeyi CLAUDE.md dosyasına yazar, böylece tüm gelecek dağıtımlar
otomatiktir.

```
Siz:   /setup-deploy

Claude: Algılandı: Fly.io (fly.toml bulundu)
        Üretim URL'si: https://myapp.fly.dev
        Sağlık denetimi: /health → 200 bekler
        Dağıtım komutu: fly deploy
        Durum komutu: fly status

        CLAUDE.md dosyasına yazıldı. Hazır olduğunuzda /land-and-deploy çalıştırın.
```

---

## `/codex`

Bu benim **ikinci görüş modum**.

`/review` hataları Claude'ın perspektifinden yakaladığında, `/codex` tamamen farklı bir AI —
OpenAI'nin Codex CLI'si — aynı diff'i incelemeye getirir. Farklı eğitim, farklı kör noktalar,
farklı güçler. Örtüşme kesinlikle gerçek olanı size söyler. Her birinden benzersiz bulgular,
hiçbirinin tek başına yakalayamayacağı hataları bulduğunuz yerdir.

### Üç mod

**İnceleme** — mevcut diff'e karşı `codex review` çalıştırır. Codex her değiştirilen dosyayı
okur, bulguları önem derecesine göre sınıflandırır (P1 kritik, P2 yüksek, P3 orta) ve bir
GEÇ/KAL kararı döndürür. Herhangi bir P1 bulgu = KAL. İnceleme tamamen bağımsızdır — Codex
Claude'un incelemesini görmez.

**Meydan okuma** — sertlik modu. Codex kodunuzu kırmaya aktif olarak çalışır. Uç durumlar,
yarış koşulları, güvenlik delikleri ve yük altında başarısız olacak varsayımlar arar.
Maksimum akıl yürütme çabası kullanır (`xhigh`). Kodunuz için bir sızma testi olarak düşünün.

**Danışma** — oturum sürekliliği ile açık konuşma. Codex'e kod tabanı hakkında herhangi bir
şey sorun. Takip soruları aynı oturumu yeniden kullanır, bu nedenle bağlam taşınır. "Bunu doğru
düşünüyor muyum?" anları için harika.

### Çapraz model analizi

Hem `/review` (Claude) hem `/codex` (OpenAI) aynı dalı incelediğinde, bir çapraz model
karşılaştırması alırsınız: hangi bulgular örtüşüyor (yüksek güven), hangileri Codex'e özgü
(farklı perspektif) ve hangileri Claude'a özgü. Bu, kod incelemesine "iki doktor, aynı hasta"
yaklaşımıdır.

```
Siz:   /codex review

Claude: Bağımsız Codex incelemesi çalıştırılıyor...

        CODEX INCELEMESI: GEÇ (3 bulgu)
        [P2] Ödeme işleyicisinde yarış koşulu — eşzamanlı ücretler
             danışma kilidi olmadan çift borçlandırabilir
        [P3] user.email üzerinde downcase'den önce eksik null denetimi
        [P3] Belirteç karşılaştırması sabit zamanlı karşılaştırma kullanmıyor

        Çapraz model analizi (/review karşılaştırması):
        ÖRTÜŞME: Ödeme işleyicisinde yarış koşulu (ikisi de yakaladı)
        CODEX'E ÖZGÜ: Belirteç karşılaştırma zamanlama saldırısı
        CLAUDE'A ÖZGÜ: İlan fotoğraflarında N+1 sorgu
```

---

## Güvenlik ve Korkuluklar

Herhangi bir Claude Code oturumuna güvenlik rayları ekleyen dört yetenek. Claude Code'un
PreToolUse kancaları aracılığıyla çalışırlar — şeffaf, oturum kapsamlı, yapılandırma dosyası yok.

### `/careful`

Üretimde çalışırken, yıkıcı komutlar çalıştırırken veya sadece bir güvenlik ağı istediğinizde
"dikkatli olun" veya `/careful` çalıştırın. Her Bash komutu bilinen tehlikeli örüntülere karşı
denetlenir:

- `rm -rf` / `rm -r` — özyinelemeli silme
- `DROP TABLE` / `DROP DATABASE` / `TRUNCATE` — veri kaybı
- `git push --force` / `git push -f` — geçmiş yeniden yazma
- `git reset --hard` — commitleri atma
- `git checkout .` / `git restore .` — işlenmemiş çalışmayı atma
- `kubectl delete` — üretim kaynak silme
- `docker rm -f` / `docker system prune` — konteyner/görüntü kaybı

Yaygın yapı yapıtı temizlikleri (`rm -rf node_modules`, `dist`, `.next`, `__pycache__`, `build`,
`coverage`) beyaz listeye alınır — rutin işlemlerde yanlış alarm yok.

Herhangi bir uyarıyı geçersiz kılabilirsiniz. Korkuluklar kaza önleme amaçlıdır, erişim kontrolü değil.

### `/freeze`

Tüm dosya düzenlemelerini tek bir dizinle kısıtlayın. Bir faturalandırma hatası ayıklarken,
Claude'un `src/auth/` içindeki ilgili olmayan kodu "düzeltmesini" istemezsiniz. `/freeze src/billing`
o yol dışındaki tüm Edit ve Write işlemlerini engeller.

`/investigate` bunu otomatik olarak etkinleştirir — ayıklanan modülü algılar ve düzenlemeleri o
dizinle dondurur.

```
Siz:   /freeze src/billing

Claude: Düzenlemeler src/billing/ ile kısıtlandı. Kaldırmak için /unfreeze çalıştırın.

        [Daha sonra, Claude src/auth/middleware.ts dosyasını düzenlemeye çalışır]

Claude: ENGELLENDİ — Dondurma sınırı dışında düzenleme (src/billing/).
        Bu değişiklik atlanıyor.
```

Not: bu yalnızca Edit ve Write araçlarını engeller. `sed` gibi Bash komutları hala sınır dışındaki
dosyaları değiştirebilir — bu bir güvenlik alanı değil, kaza önlemedir.

### `/guard`

Tam güvenlik modu — `/careful` + `/freeze` tek komutta birleştirir. Yıkıcı komut uyarıları artı
dizin kapsamlı düzenlemeler. Üretimde çalışırken veya canlı sistemlerde hata ayıklarken kullanın.

### `/unfreeze`

`/freeze` sınırını kaldırır, her yerde düzenlemelere tekrar izin verir. Kancalar oturum için
kayıtlı kalır — sadece her şeye izin verirler. Yeni bir sınır belirlemek için tekrar `/freeze`
çalıştırın.

---

## `/gstack-upgrade`

Tek komutla gstack'i güncel tutun. Kurulum türünüzü algılar (`~/.claude/skills/gstack`
konumunda genel vs projenizdeki `.claude/skills/gstack` konumunda satıcı), yükseltmeyi çalıştırır,
çift kurulumunuz varsa her iki kopyayı senkronize eder ve neyin değiştiğini gösterir.

```
Siz:   /gstack-upgrade

Claude: Mevcut sürüm: 0.7.4
        En son sürüm: 0.8.2

        Yenilikler:
        - CAPTCHA'lar ve kimlik doğrulama duvarları için tarayıcı devretme
        - /codex çoklu AI ikinci görüş
        - /qa artık her zaman tarayıcı kullanıyor
        - Güvenlik yetenekleri: /careful, /freeze, /guard
        - Proaktif yetenek önerileri

        0.8.2 sürümüne yükseltildi. Hem genel hem proje kurulumları senkronize edildi.
```

Yeni bir sürüm mevcut olduğunda her oturumun başında gstack sessizce yükseltmesi için
`~/.gstack/config.yaml` dosyasında `auto_upgrade: true` ayarlayın — komut istemini tamamen
atlar.

---

## Greptile entegrasyonu

[Greptile](https://greptile.com), PR'lerinizi otomatik olarak inceleyen bir YC şirketidir.
Gerçek hataları yakalar — yarış koşulları, güvenlik sorunları, CI'yi geçen ama üretimde
patlayan şeyler. Bu adamlar gerçekten hayatımı kurtardı. Seviyorum bunları.

### Kurulum

GitHub deponuza [greptile.com](https://greptile.com) adresinden Greptile'ı kurun — yaklaşık
30 saniye sürer. PR'lerinizi incelemeye başladığında, gstack yorumlarını otomatik olarak alır.
Ek yapılandırma gerekmez.

### Nasıl çalışır

Herhangi bir otomatik inceleyicide sorun triajdır. Greptile iyi, ama her yorum gerçek bir
sorun değil. Bazıları yanlış pozitif. Bazları üç commit önce zaten düzelttiğiniz şeyleri
işaret eder. Bir triaj katmanı olmadan, yorumlar birikir ve onları yok saymaya başlarsınız —
bu da amacı defeating eder.

gstack bunu çözer. `/review` ve `/ship` artık Greptile farkındadır. Greptile'ın yorumlarını
okur, her birini sınıflandırır ve eylem alır:

- **Geçerli sorunlar** kritik bulgulara eklenir ve göndermeden önce düzeltilir
- **Zaten düzeltilmiş sorunlar** yakalamayı kabul eden otomatik bir yanıt alır
- **Yanlış pozitifler** geri itilir — onaylarsınız ve neden yanlış olduğunu açıklayan bir yanıt çıkar

Sonuç iki katmanlı bir incelemedir: Greptile asenkron olarak PR'da şeyleri yakalar, ardından
`/review` ve `/ship` bu bulguları normal iş akışının bir parçası olarak triaj eder. Hiçbir
şey çatlaktan düşmez.

### Geçmişten öğrenme

Onayladığınız her yanlış pozitif `~/.gstack/greptile-history.md` dosyasına kaydedilir. Gelecek
çalıştırmalar kod tabanınız için bilinen FP örüntülerini otomatik olarak atlar. Ve `/retro`
zaman içinde Greptile'ın vuruş oranını izler — böylece sinyal-gürültü oranının iyileşip
iyileşmediğini görebilirsiniz.

### Örnek

```
Siz:   /ship

Claude: [ana dalı senkronize eder, testleri çalıştırır, birleştirme öncesi inceleme...]

        Greptile bu PR'da 3 yorum buldu:

        [GEÇERLI] app/services/payment_service.rb:47 — Yarış koşulu:
        eşzamanlı ücretler çift borçlandırabilir. Ücret bloğu etrafında
        DB düzeyinde danışma kilidi öneriyor.

        [ZATEN DÜZELTİLDİ] app/models/user.rb:12 — downcase'den önce
        e-postada eksik null denetimi.
        → abc1234 commit'inde zaten düzeltildi. Otomatik yanıt veriliyor.

        [YANLIŞ POZİTİF] lib/auth.rb:88 — "Belirteç karşılaştırması sabit
        zamanlı karşılaştırma kullanmalı." Burada zaten
        ActiveSupport::SecurityUtils.secure_compare kullanıyoruz.

        payment_service.rb'de yarış koşulu düzeltiliyor...

        Auth belirteç yorumu bir yanlış pozitif.
        Seçenekler: A) Greptile'ı açıklama  B) Yine de düzelt  C) Yoksay

Siz:    A

Claude: Greptile'a yanıt verildi. Tüm testler geçti.
        PR: github.com/you/app/pull/42
```

Üç Greptile yorumu. Bir gerçek düzeltme. Biri otomatik kabul. Biri yanlış pozitif geri
itildi ve bir yanıtla. Toplam ek süre: yaklaşık 30 saniye.

---

## `/ios-qa`

Canlı cihaz iOS QA'sı. Çatalın yük taşıyan içgörüsü şuydu: simüle etme, XCTest çalıştırma,
WebDriverAgent'a getirme. Test altındaki uygulamaya bir HTTP sunucusu gömün ve bir Mac
tarafındaki artalan sürecinden USB CoreDevice IPv6 tüneli üzerinden sürün.

Aracı Swift kaynak kodunuzu okur, `@Snapshotable` ile işaretlenmiş alanlara sahip
`@Observable` sınıflarını bulur, tipli erişimcileri kod üretir, bir hata ayıklama köprüsü
dağıtır, ardından kapalı bir bul→düzelt→doğrula döngüsü çalıştırır.

### Bir diyagramda mimari

```
       ┌──────────────────────┐   USB CoreDevice (IPv6)   ┌──────────────────┐
       │ gstack-ios-qa daemon │ ────────────────────────▶ │ iOS uygulaması          │
       │ (Mac, bun/TS)        │   bearer + X-Session-Id   │ StateServer      │
       │ - önyükleme belirtecini döndürür │                           │ (yalnızca geri döngü)  │
       │ - oturum belirteçleri basar │                           └──────────────────┘
       │ - yetenek katmanları   │
       │ - denetim + sansür │
       └──────────────────────┘
                ▲
                │ Tailscale (isteğe bağlı, --tailnet)
                │
       ┌──────────────────────┐
       │ Uzak aracı         │
       │ (OpenClaw, vb.)     │
       └──────────────────────┘
```

iOS uygulamasının `StateServer` yalnızca geri döngü bağlar (`::1` + `127.0.0.1`). Mac artalan
süreci tailnet kimlik doğrulaması, yetenek katmanları ve denetim izine sahiptir. Uzak aracılar
asla önyükleme belirtecini görmez — yalnızca Tailscale kimlik geçidi ile basılan kısa ömürlü
oturum belirteçleri (varsayılan 1 saat, en fazla 24 saat).

### Kilidin açılması: USB-tethered + Tailscale = herhangi bir aracıdan uzak iOS QA

Zaten sahip olduğunuz bir Mac artı zaten sahip olduğunuz bir iPhone artı Tailscale ücretsiz
katmanı, çoğu ekibin BrowserStack/Sauce Labs için ödediğinin yerini alır. Tailnet'inizdeki
herhangi bir HTTP yapabilen aracı, onlara bir oturum belirteci basarsanız iOS uygulamasını
 sürebilir. Tailscale ACL'leri hangi kimliklerin hangi yetenek katmanında Mac'e erişebileceğini
kapsama alır.

Çalıştırılabilir kurulum için `ios-qa/docs/tailscale-acl-example.md` dosyasına bakın.

### Yetenek katmanları

| Katman | Uç noktalar |
|------|-----------|
| gözlem | `/screenshot`, `/elements`, `GET /state/*`, `/state/snapshot`, `/healthz` |
| etkileşim | gözlem + `/tap`, `/swipe`, `/type`, `/session/*` |
| değişiklik | etkileşim + `POST /state/<anahtar>` |
| geri yükleme | değişiklik + `POST /state/restore` |

Varsayılan basılan belirteçler `etkileşim` alır. Daha yüksek katmanlar açık sahip basmasını gerektirir.

---

## `/ios-fix`

Demir Kural: üreten bir anlık görüntü olmadan düzeltme yok. Aracı `GET /state/snapshot`
aracılığıyla hata öncesi durumu yakalar, düzeltmeyi yazar, yeniden derler, yeniden dağıtır,
anlık görüntüyü geri yükler ve hatanın gittiğini doğrular. Anlık görüntü bir regresyon testi
sabiti olur, böylece hata sessizce tekrarlayamaz.

iOS için `/qa`'nın bul-hata → düzelt → yeniden doğrula döngüsünü yansıtır.

---

## `/ios-design-review`

Gerçek bir iPhone'da tasarımcının gözü QA'sı. Aynı `/ios-qa` artalan sürecine gözlem katmanı
modunda bağlanır ve her ekranın ekran görüntüsünü çeker. 10 boyutu 0-10 puanlar: tipografi
hiyerarşisi, aralık ritmi, renk hiyerarşisi, dokunma hedefleri, yükleme/boş/hata durumları,
erişilebilirlik, animasyon disiplini, iOS deyim hizalaması, bilgi yoğunluğu, AI-çöpü denetimi.

7'nin altındaki her puan için, önerilen düzeltme ile sorunu sunmak üzere AskUserQuestion kullanır.

---

## `/ios-clean`

Kolaylık sarmalayıcı. DebugBridge'ın gönderilmesine karşı yapısal Release-derleme koruması
`Package.swift`'tedir (`.when(configuration: .debug)`) artı bir CI değişmez testi. `/ios-clean`,
güdümlü el ile kaldırma akışı isteyen veya `/ios-qa` üzerinden geçmeden SPM bağımlılığını
el ile ekleyen geliştiriciler içindir.

---

## `/ios-sync`

gstack'i yükselttikten veya yeni `@Observable` sınıfları ekledikten sonra çalıştırın. Neyin
kurulu olduğunu algılar, en son yukarı akış şablonlarına karşı gen-erişimcileri çalıştırır,
değişen Swift dosyalarını yeniler, uygulamanın yeniden derlendiğini doğrular. Önbellek anahtarı
geçersizleştirme Swift sürüm değişikliklerini, oluşturucu git revizyon değişikliklerini ve
kaynak değişikliklerini işler.