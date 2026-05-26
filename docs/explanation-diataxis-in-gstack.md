# gstack neden belgelendirme için Diataxis kullanıyor

gstack'teki iki belge yeteneği — `/document-release` ve `/document-generate` — her
ikisi de Diataxis dilini konuşur. Yeni varlıklar dört kadran üzerinden puanlanır.
Kapsam boşlukları, kadranlara göre etiketlenmiş PR gövdelerinde ortaya çıkar. Bu belge,
bu sözlüğün neden yük taşıyan olduğunu ve neden daha basit bir "sadece markdown yaz"
yaklaşımının gstack'in ölçeğinde başarısız olduğunu açıklar.

## Sorun

Belgelendirme çürümesi, göz ardı edilmesi en kolay çürüme türüdür. Kod derlemeyi
bıraktığında hemen fark edersiniz. Bir test başarısız olduğunda CI bağırır. Belgeler
sessizce eskir — README hala ayrıştırılır, kurulum komutu hala kopyala-yapıştır
edilebilir — ve tek sinyal haftlar sonra kafası karışmış bir kullanıcının bir sorun
dosyası açması veya sessizce uzaklaşmasıdır.

gstack'in 45'ten fazla yeteneği var. Her biri bir SKILL.md artı bir `.tmpl` şablonu
artı ideal olarak bir yerlerde başlangıç eğitimi ve neden bu şekilde çalıştığının bir
açıklaması. Bunu gstack kullanıcılarının kendi projelerinde benzer yüzey alanına sahip
olduğu sayıyla çarpın ve bakım yükü gerçektir.

Naive başarısızlık modu "her takım belgeleri kendi biçiminde yazar." Bir projenin
Wiki'si var. Başkasının iç içe geçmiş README dosyaları var. Üçüncüsünün yalnızca
referanslı API belgeleri ve hiçbir eğitimi yok. Dördüncüsünün artık derlenmeyen
eğitimleri var. Bunların tümünü denetleyen bir araç yazamazsınız çünkü iyi kapsamın
neye benzediğine dair paylaşılan bir sözlük yoktur.

İkinci başarısızlık modu daha incedir: bir takım disiplinli olsa bile, mevcut zihin
durumlarına uyan belge türünü yazma eğilimindedirler. İnşa modundaki mühendisler
referans yazar. Lansman modundaki mühendisler eğitim yazar. Bakım modundaki mühendisler
sorun giderme nasıl yapılırını yazar. Kimse uyanıp "bugün bu mimariyi neden seçtiğimiz
açıklama belgesini yazacağım" demez — bu nedenle açıklama çürümesi en hızlı birikenidir.

## Yaklaşım

Diataxis (Daniele Procida, başlangıçta Divio'da, şimdi CPython, Django, NumPy,
FastAPI, GitHub belgeleri ve diğer birçoğunda benimsenmiştir) belgelendirmeyi **okuyucu
niyetine** göre dört kadranına böler:

```
                    KURAMSAL                        UYGULAMALI
                    (anlama)                       (yapma)

  ÇALIŞMA          +-----------------------------+----------------------------+
  (öğrenme)        |                             |                            |
                   |   AÇIKLAMA                  |   EĞİTİM                   |
                   |   "X neden var?"             |   "Beni X'i ilk defa       |
                   |                              |    adım adım gezdir"      |
                   |   kodu tartışır             |   kod öğretir             |
                   |                              |                            |
                   +-----------------------------+----------------------------+

  KULLANMA         +-----------------------------+----------------------------+
  (kullanma)       |                             |                            |
                   |   REFERANS                  |   NASIL YAPILIR            |
                   |   "Y'nin tam imzası         |   "X kullanarak Y'yi      |
                   |    nedir?"                   |    nasıl yaparım?"         |
                   |                              |                            |
                   |   kodu tanımlar             |   kod kullanır            |
                   |                              |                            |
                   +-----------------------------+----------------------------+
```

Eğitim modundaki bir okuyucu yaparak öğrenir. Rehberli bir yol ve garantili başarı
isterler. Nasıl yapılır modundaki bir okuyucu temelleri zaten bilir ve belirli bir
görev için reçete ister. Referans modundaki bir okuyucu doğru, eksiksiz, olgu tablosu
API kapsamı ister. Açıklama modundaki bir okuyucu bir tasarım kararını anlamak ister.

Aynı kişi farklı zamanlarda bir projeyi bu modların her birinden okur. Aynı paragraf
dördüne de hizmet edemez — eğitimler bir referans okuyucusunu yavaşlatacak el
tutuşması gerektirir; referans bir eğitim okuyucusunu bunaltabilecek eksiksizlik
gerektirir.

## Bu neden kapsama merceği olarak önemli

Diataxis terimleriyle yazılmış bir kapsam haritası size "belgeler güncellendi mi?" 
sorusuna deterministik bir yanıt verir — "bir README var mı" değil, "bu yeni yetenek
için bir eğitim, yaygın görev için bir nasıl yapılır, API için bir referans ve bariz
olmayan tasarım kararı için bir açıklama var mı?"

`/document-release` 1. Adım, diff'i yürür, yeni genel yüzeyi (yetenekler, CLI
bayrakları, yapılandırma seçenekleri, API uç noktaları) çıkarır ve her varlığı dört
kadran üzerinden puanlar. Sıfır kapsama sahip öğeler **kritik boşluklar** olur.
Yalnızca referans kapsama sahip öğeler (gstack'in kendi geçmişindeki en yaygın başarısızlık
modu) **yaygın boşluklar** olur. Her ikisi de gözden geçirenlerin gördüğü PR gövdesine
düşer.

`/document-generate` belgeleri kasten dört kadrandan yazar. Onları karıştırmayı reddeder:
bir eğitim "Yapılandırma" bölümü almaz, bir referans belgesi "Ne yapacaksınız" paragrafı
almaz. Yeteneğin 9 adımı referans → açıklama → nasıl yapılır → eğitim sırasını takip
eder çünkü bu sıralama bağımlılığı karşılar: referans sözlüğü sabitler, açıklama tasarımı
haklı çıkarır, nasıl yapılır her ikisine de dayanır, eğitimler en son ve en zor olanıdır.

## Ödünleşimler

**Diataxis, okuyucuların öğrenmesi gereken bir sözlük ekler.** "Referans ve açıklama"
ayrımını hiç duymamış bir kullanıcı başlangıçta etiketleri yabancı bulabilir. Azaltma
şudur: Diataxis etiketleri bir kez gördüğünüzde kendinden açıklamalıdır ve etiketler
asla belgelerin kendilerinde görünmez — kapsam haritasında ve PR gövdesinde, gözden
geçirenlerin gördüğü yerlerde görünür, son kullanıcıda değil.

**Bir dosya yerine dört dosya.** Küçük bir yeteneğin dört modu da karıştıran tek bir
`docs/SKILL.md` dosyası olabilir. Diataxis bunu dörde böler. Azaltma: AI üretimi
dört dosyalı yapıyı ucuzlatır, kadranlar arası çapraz bağlantılar mekaniktir (her
referans belgesi kendi nasıl yapılırına bağlantı verir, her nasıl yapılır kendi
referansına bağlantı verir vb.) ve denetlenebilirlikteki kazanımlar önemlidir —
`/document-release` kapsamı otomatik olarak puanlayabilir.

**Diataxis tek iyi çerçeve değildir.** "Her sayfa birinci sayfadır" (Mark Baker), *Write
the Docs* topluluğundaki dört belge türü, Google geliştirici belgelendirme stil rehberi
— hepsinin farklı kesimleri var. gstack Diataxis'i seçti çünkü en güçlü dış benimsenmeye
sahip (CPython, Django, NumPy, FastAPI vb.), bu da aşağı akış kullanıcıların sözlüğü
daha önce görmüş olma olasılığının en yüksek olduğu anlamına gelir ve kadran etiketleri
kapsam haritası sinyallerine temiz bir şekilde çevrilir.

## Düşünülen alternatifler

**"Sadece README bölümleri yazın."** gstack'in geçmişinde örtük olarak denendi. Başarısızlık
modu: eğitimler README'de birikti, README'ler 800+ satır oldu ve kimse 50. satırdan
sonrasını okumadı. Diataxis bunları adanmış dosyalara böler, her biri README'nin içindekiler
tablosundan bulunabilir.

**Özel yersel taksonomi.** Uyarlanabileceği için cazip. Her takım kendi sözlüğünü
icat edeceği ve `/document-release` projeler arası denet gücünü kaybedeceği için reddedildi.
Diataxis lingua franca'dır.

**Yalnızca otomatik üretilmiş referans.** JSDoc / TypeDoc / Sphinx gibi araçlarla birçok
proje için denendi. Açıklama olmayan referans belgeleri yeni gelenler için geçilmez hale
gelir; eğitimler olmadan API'ye başlamak zordur. Referans gerekli ama yeterli değildir.

**Hiçbir belgelendirme çerçevesi yok, sadece iç denetim.** Çoğu proje için durum quo'su.
Sessizce başarısız olur — kullanıcılar sorun dosyası açmak yerine uzaklaşırlar, bu nedenle
geri besleme döngüsü kırıktır. Diataxis, kullanıcılar şikayet etmeden önce bile yapılandırılmış
bir sinyal verir.

## İlgili

- **Bunu uygulayan yetenek için referans:** [`document-generate/SKILL.md`](../document-generate/SKILL.md)
- **Bu taksonomiyi kullanan denetim için referans:** [`document-release/SKILL.md`](../document-release/SKILL.md)
- **`/document-generate` kullanımı için eğitim:** [`tutorial-document-generate.md`](./tutorial-document-generate.md)
- **Yayınlanan bir özelliği belgelendirme nasıl yapılır:** [`howto-document-a-shipped-feature.md`](./howto-document-a-shipped-feature.md)
- **Diataxis ana sayfası:** https://diataxis.fr/ — Çerçeve için Procida'nın kanonik referansı