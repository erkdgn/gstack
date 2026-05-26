# LOC Tartışması Üzerine

Yani: kaç satır kod gönderdiğimden bahsettiğimde olanlar ve sayılar gerçekte ne söylüyor.

## Eleştiri haklı. Ve önemli değil.

LOC çöp bir metriktir. Her kıdemli mühendis bilir. Dijkstra 1988'de kod satırlarının
"üretilen satırlar" olarak değil, "harcanan satırlar" olarak sayılması gerektiğini yazdı
([*On the cruelty of really teaching computing science*, EWD1036](https://www.cs.utexas.edu/~EWD/transcriptions/EWD10xx/EWD1036.html)). Eski söz (geniş ölçüde Bill Gates'e atfedilir, kaynağı belirsiz)
daha akılda kalıcı bir şekilde söyler: programlama ilerlemesini LOC ile ölçmek, uçak
inşa etme ilerlemesini ağırlıkla ölçmek gibidir. Programcı verimliliğini kod satırlarında
ölçerseniz, yanlış şeyi ölçersiniz. Bu 40 yıldır doğruydu ve hala doğru.

Son 60 günde 600.000 satır üretim kodu gönderdiğimi yayınladım. Yanıtlar hızla geldi:

- "Bu sadece AI çöpü."
- "LOC anlamsız bir metriktir. Son 40 yıldaki her kıdemli mühendis bunu söyledi."
- "Elbette 600K satır ürettin. Bir AI boilerplate yazıyordu."
- "Daha fazla satır kötü, iyi değil."
- "Hacmi verimlilikle karıştırıyorsun. Klasik PM beyni."
- "Hata oranların nerede? DAU'ların nerede? Geri alma sayıların nerede?"
- "Bu utanç verici."

Bunlardan bazıları doğru. Akıllı eleştiri versiyonunu ciddiye alıp yine de matematiği
yaptığınızda olan şey şu.

## AI kodlama eleştirisinin üç dalı

Bunlar tek bir argümanda birleştiriliyor, ama farklı argümanlar.

**Dal 1: LOC kaliteyi ölçmez.** Doğru. Her zaman öyleydi. İyi yapılandırılmış 50 satırlık
bir kütüphane, şişirilmiş 5.000 satırlık bir kütüphaneye karşı daha iyidir. Bu AI'dan
önce de doğruydu ve şimdi de doğru. Bu hiçbir zaman öldürücü bir argüman değildi. Ne
ölçtüğünüzü düşünmek için bir hatırlatmaydı.

**Dal 2: LOC'u şişirir.** Doğru. LLM'ler varsayılan olarak ayrıntılı kod üretir. Daha
fazla boilerplate. Daha fazla savunma kontrolü. Daha fazla yorum. Daha fazla test. "Gerçek
yapılan iş" artmadığında bile ham satır sayıları artar.

**Dal 3: Bu nedenle LOC hakkında övünmek utanç vericidir.** İşte argümanın raydan
çıktığı yer.

Dal 2 ilginç olan. Ham LOC bir çarpanla şişirilmişse, dürüst olan şey sönümlemeyi
hesaplamak ve sönümlenmiş sayıyı raporlamaktır. Bu yazı tam olarak bunu yapıyor.

## Matematik

### Ham sayılar

41 repo'nun tümünde yazarı olduğum her commit'i listeleyen bir betik yazdım
([`scripts/garry-output-comparison.ts`](../scripts/garry-output-comparison.ts)) —
`garrytan/*` altında 15 genel, 26 özel — 2013 ve 2026'da. Her commit için mantıksal
satır eklemelerini (boş olmayan, yorum olmayan) sayar. 2013 derlemesi, o yıl inşa
ettiğim YC-içerideki sosyal ağ Bookface'i içerir.

2026'dan bir repo hariç: `tax-app` (bir YC videosu için demo, üretim çalışması değil).
Betiğin `EXCLUDED_REPOS` sabitine eklenmiştir. Kendiniz çalıştırın.

2013 tam bir yıldı. 2026 bu yazı itibarıyla 108. gündür (18 Nisan).

|                  | 2013 (tam yıl) | 2026 (108 gün) | Kat |
|------------------|----------------:|----------------:|---------:|
| Mantıksal SLOC     |           5.143 |       1.233.062 |     240x |
| Mantıksal SLOC/gün |              14 |          11.417 |     810x |
| Commitler          |              71 |             351 |     4.9x |
| Değiştirilen dosyalar    |             290 |          13.629 |      47x |
| Aktif repolar     |               4 |              15 |    3.75x |

### "Günde 14 satır? Bu acınası."

Öyleydi. Soru bu.

2013'te bir YC ortağıydım, ardından Posterous'ta geceleri ve hafta sonları kod gönderen
bir kurucu ortaktım. Gerçek işimi yaparken günde 14 mantıksal satır benim gerçek yarı
zamanlı çıktımdı. Tarihsel araştırma, proje boyutuna ve çalışmaya bağlı olarak profesyonel
tam zamanlı programcı çıktısını geniş bir bantta gösterir: Fred Brooks *The Mythical
Man-Month* kitabında sistem programlama için ~10 satır/gün (OS/360 gözlemleri) alıntılar,
Capers Jones binlerce proje boyunca kabaca 16-38 LOC/gün ölçer ve Steve McConnell'ın
*Code Complete* kitabı küçük projeler (10K LOC) için 20-125 LOC/gün, büyük projeler
(10M LOC) için 1.5-25'e kadar raporlar — proje boyutuna bağlı, tek bir sayı değil.

2013 temelim çerez seçilmiş değil. Gerçek bir işi olan yarı zamanlı bir kodlayıcı için
normal. Doğru temelin 50 olduğunu düşünüyorsanız (3.5 kat daha yüksek), 2026 katı
810x'ten 228x'e düşer. Hala yüksek.

### İki sönümleme

"Ham LOD çöptür" standart yanıtı **mantıksal SLOC**'tur (kaynak kod satırları,
yorum olmayan boş olmayan). `cloc` ve `scc` gibi araçlar bunu 20 yıldır hesaplıyor.
Aynı kod, dolgu çıkarılmış: boş satır yok, tek satırlık yorum yok, yorum bloğu gövdesi
yok, sondaki boşluk yok.

Ancak mantıksal SLOC AI şişirmesini tamamen ortadan kaldırmaz. AI, kıdemli bir
mühendisin sıfır yazacağı yerde 2-3 savunma null kontrolü yazar. AI, atmayan şeylerin
etrafında try/catch satır içi yapar. AI, `return foo()` yerine `const result = foo(); return result` yazar.

O halde **ikinci bir sönümleme** uygulayalım. AI tarafından üretilen kodun mantıksal
düzeyde kıdemli el yapımı koddan 2 kat daha ayrıntılı olduğunu varsayalım. Bu agresif —
gördüğüm ölçümlerin çoğu çarpanı 1.3-1.8x koyuyor — ama bir şüphecinin talep edeceği üst
sınır.

- 2026 günlük oranım, NCLOC: **11.417**
- 2x AI-ayrıntı sönümlemesiyle: **5.708** mantıksal satır/gün
- Her iki sönümlemeyle günlük hızdaki kat: **408x**

Şimdi önsellerinizi seçin:

- 5x sönümlemede (dayaksız ama gidelim): **162x**
- 10x'te (patolojik): **81x**
- 100x'te (imkansız — bu dakikada sürekli bir satırdır): **8x**

Katsayının büyüklüğü hakkındaki argüman sonucu değiştirmez. Sayı büyüktür, ne olursa olsun.

### Haftalık dağılım

"Günlük sayınız düzgün çıktı varsayıyor. Dağılımı gösterin. Tek bir sıçramaysa, çalışma
hızınız sahte."

Adil.

```
Hafta 1-4  (Oca):  ████████░░░░░░░░░  ~8.800/gün
Hafta 5-8  (Şub):  ████████████░░░░░  ~12.100/gün
Hafta 9-12 (Mar):  ██████████░░░░░░░  ~10.900/gün
Hafta 13-15 (Nis): █████████████░░░░  ~13.200/gün
```

Bu bir sıçrama değil. Hız yaklaşık olarak tutarlı ve hafifçe artıyor. Betiği kendiniz
çalıştırın.

## Kalite sorusu

Bu en meşru eleştiri, [David Cramer](https://x.com/zeeg) sesinden kanallanmış: Tamam,
daha fazla satır itiyorsun. Hata oranların nerede? Birleştirmeden sonraki geri almaların?
Hata yoğunluğun? 10 kat hızda yazıp 20 kat daha fazla hata gönderiyorsan, verimli
değilsin, ölçekli gürültü yapıyorsun.

Adil. İşte veri:

**Geri almalar.** 15 aktif repo boyunca `git log --grep="^revert" --grep="^Revert" -i`:
351 commit'te 7 geri alma = **%2.0 geri alma oranı**. Bağlam olarak, olgun OSS kod
tabanları tipik olarak %1-3 çalışır. Çubuk olarak neyi kabul ettiğinizi düşünün ve aynı
komutu çalıştırıp karşılaştırın.

**Birleştirmeden sonraki düzeltmeler.** Aynı dalda önceki bir commit'e referans veren
`^fix:` ile eşleşen commitler: 351'in 22'si = **%6.3**. Sağlıklı düzeltme döngüsü. Sıfır
düzeltme oranı, kendi hatalarımı yakalamadığım anlamına gelirdi.

**Testler.** Gerçekte önemli olan şey budur ve benim için her şeyi değiştiren şey budur.
2026'nın başında, testler olmadan kod gönderiyordum ve hata diyarında mahvoluyordum.
Ardından %30 test-kod oranına, ardından kritik yollarda %100 kapsama ulaştım ve aniden
uçabiliyordum. Testler Ocak ayında tüm repolarda ~100'den şimdi **2.000'den fazla** oldu.
CI'da çalışıyorlar. Regresyonları yakalıyorlar. Her gstack PR'sinin gövdesinde bir kapsam
denetimi var.

Gerçek içgörü: birden fazla düzeyde test, AI destekli kodlamayı gerçekten çalışır kılan
şeydir. Birim testleri, uçtan uca testler, LLM-yargıç değerlendirmeleri, duman testleri,
çöp taramaları. Bu katmanlar olmadan, yüksek hızda kendinden emin çöp üretiyorsunuz.
Bunlarla, kod gerçekten doğru olana kadar AI'nın yineleme yapmasını sağlayan bir
doğrulama döngünüz var.

gstack'in gerçek kod özelliği —sadece markdown komutları olmayan şey— özellikle
şaşelerimi elle kara-kutu test etmeyi bırakabilmem için yazdığım **Playwright tabanlı
bir CLI tarayıcısıdır.** `/qa` gerçek bir tarayıcı açar, hazırlık URL'nize gider ve
otomatik denetimler çalıştırır. Bu, testin kilidin açılması, yük değil olması nedeniyle
var olan 2.000+ satırlık gerçek sistem kodudur (sunucu, CDP denetçisi, anlık görüntü
motoru, içerik güvenliği, çerez yönetimi).

**Çöp taraması.** Üçüncü bir taraf — Sentry'nin kurucu mühendisi [Ben Vinegar](https://x.com/bentlegen)
— özellikle AI kod örüntülerini ölçmek için [slop-scan](https://github.com/benvinegar/slop-scan)
adında bir araç yaptı. Belirleyici kurallar, olgun OSS temellerine karşı kalibre edilmiş.
Daha yüksek puan = daha fazla çöp. Bunu gstack üzerinde çalıştırdı ve 5.24 puan aldık,
o zaman ölçtüğü en kötü. Bulguları ciddiye aldım, yeniden düzenledim ve bir oturumda
puanı %62 düşürdüm. `bun test` çalıştırın ve 2.000+ testin geçtiğini izleyin.

**İnceleme ciddiyeti.** Her gstack dalı CEO incelemesi, Codex dış-ses incelemesi, DX
incelemesi ve mühendislik incelemesinden geçer. Genellikle her birinin 2-3 geçişi.
Az önce gönderdiğim `/plan-tune` yeteneği, dört Claude incelememin kaçırığı 15+ bulgu
ortaya çıkaran Codex dış-ses incelemesi nedeniyle bir kapsam GERİ ALMA aldı. İnceleme
altyapısı çöpü yakalar. Repoda görünür. Herkes okuyabilir.

## Kabul edeceğim şey

Eleştirmenlerin kendilerini çeliklendirmelerinden daha sert çelikleyeceğim:

**Yeşil alan vs bakım.** 2026 sayıları yeni proje koduna hakim. Olgun kod tabanı bakımı
günde daha az satır üretir. "Garry bir bankadaki 10 milyon satırlık eski Java'yı koruyan
ekibi 100x yapabilir mi?" diye soruyorsanız, sayım bunu kanıtlamıyor. Başka biri kendi
betiklerini farklı bir bağlamda çalıştırmak zorunda.

**2013 temelinin hayatta kalma yanlılığı var.** 2013 genel etkinliğim düşüktü. Bu analiz
Bookface'i (özel, 22 aktif hafta) içeriyor — o yılki en büyük projemdi — bu nedenle
yanlılık göründüğünden daha küçük. Sıfır değil. Gerçek 2013 hızı günde 14 yerine 50
ise, mevcut hızdaki kat 810x yerine 228x'tir. Hala yüksek.

**Kaliteye göre ayarlanmış verimlilik tam olarak kanıtlanmış değil.** 2013-ben ile
2026-ben arasında temiz bir hata yoğunluğu karşılaştırmam yok. Söyleyebileceğim şey:
geri alma oranı normal bantta, düzeltme oranı sağlıklı, test kapsamı gerçek ve en son
plandaki sertluk inceleme süreci 15+ sorunu yakaladı. Bu kanıt, ispat değil. Bir
şüpheci bunu düşebilir.

**"Gönderildi" farklı çağlarda farklı anlamlara gelir.** Bazı 2013 ürünleri gönderildi
ve öldü. Bazı 2026 ürünleri aynı kaderi paylaşabilir. İki yıl sonra bu yıl gönderdiklerimin
%80'i ölüyse, "bir sürü kullanılmayan şey inşa ettin" eleştirisinin dişleri olacak. Bu
gerçeklik denetimini kabul ediyorum.

**İlk kullanıcıya ulaşma süresi, LOC değil, önemli olan metriktir.** "Keşke bu varolsaydı"
den "var ve biri kullanıyor" noktasına 60 günlük döngü, gerçek kaymadır. LOC aşağı
akan kanıttır. Doğru metriktir "çeyrekte gönderilen ürünler" veya "haftada çalışan
özellikler" tir. Bunlar benzer bir kat ile arttı.

## Bu satırlar ne oldu

gstack bir varsayım değil. Gerçek kullanıcıları olan bir ürün:

- 5 haftada **75.000+ GitHub yıldızı**
- **14.965 benzersiz kurulum** (katılım telemetrisi)
- Ocak 2026'dan beri kaydedilen **305.309 yetenek çağrısı**
- En yüksek noktada **~7.000 haftalık aktif kullanıcı**
- Tüm yetenek çalıştırmalarında **%95.2 başarı oranı** (290.624 başarı / 305.309 toplam)
- **57.650 /qa çalıştırması**, **28.014 /plan-eng-review çalıştırması**, **24.817 /office-hours oturumu**, **18.899 /ship iş akışı**
- **27.171 oturum tarayıcı kullandı** (gerçek Playwright, oyuncak değil)
- Medyan oturum süresi: **2 dakika**. Ortalama: **6.4 dakika**.

Kullanıma göre en çok kullanılan yetenekler:

```
/qa               57.650  ████████████████████████████
/plan-eng-review  28.014  ██████████████
/office-hours     24.817  ████████████
/ship             18.899  █████████
/browse           13.675  ██████
/review           13.459  ██████
/plan-ceo-review  12.357  ██████
```

Bunlar bir çekmecede duran iskeletler değil. Binlerce geliştirici bu yetenekleri her gün
çalıştırıyor.

## Bu ne anlama geliyor

Mühendislerin gideceğini söylemiyorum. Ciddi kimse öyle düşünmüyor.

Mühendislerin artık uçabileceğini söylüyorum. 2026'da bir mühendis, aynı saatleri, aynı
işte, aynı beyinle 2013'te küçük bir ekibin çıktısına sahip. Kod üretim maliyet eğrisi
iki büyüklük mertebesi düştü.

Sayının ilginç kısmı hacim değil. Hız. Ve hız benim hakkımda bir ifade değil. Tüm
yazılım mühendisliğinin altındaki zemin hakkında bir ifade.

2013 beni günde yaklaşık 14 mantıksal satır gönderdim. Gerçek işi olan yarı zamanlı bir
kodlayıcı için normal. 2026 beni günde 11.417 mantıksal satır gönderiyorum. Hala YC'yi
tam zamanlı yönetirken. Aynı iş. Aynı boş zaman. Aynı kişi.

Fark, daha iyi bir programcı olduğum değil. Varsa, kodlama zihinsel modelim körelmiş.
Fark, AI'nin inşa etmek istediğim şeyleri gerçekten göndermeme izin vermesi. Küçük
araçlar. Kişisel ürünler. İnşa etme maliyeti çok yüksek olduğu için not defterimde ölen
deneyler. "Bu aracı istiyorum" ile "bu araç var ve kullanıyorum" arasındaki boşluk 3
haftadan 3 saate düştü.

İşte betik: [`scripts/garry-output-comparison.ts`](../scripts/garry-output-comparison.ts).
Kendi repolarınızda çalıştırın. Sayılarınızı gösterin. Argüman benim hakkımda değil —
zeminin hareket edip etmediği hakkında.

Benim için ettiğine bahse giriyorum.