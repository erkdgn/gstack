# Eğitim: 90 saniyede bir özellik için belgeler üretin

Zaten sahip olduğunuz bir projede `/document-generate` çalıştıracaksınız, doğru yerlerde
eğitim / nasıl yapılır / referans / açıklama belgeleri yazmasını izleyeceksiniz ve bir
PR'ye bırakabileceğiniz bir kapsam haritası ile biteceksiniz. Sonunda, dört hamleyi
bileceksiniz: kapsam, arkeoloji, bölümlendirme, yazma.

## İhtiyacınız olanlar

- gstack kurulu (`git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/.claude/skills/gstack && cd ~/.claude/skills/gstack && ./setup`)
- En az bir genel yüzeyi (bir CLI komutu, dışa aktarılan bir işlev, bir yapılandırma
  seçeneği, bir yetenek, bir API uç noktası) olan herhangi bir projede çalışan Claude Code
- Yaklaşık 90 saniye

Önceden bir `docs/` dizinine ihtiyacınız yok — yetenek eksikse bir tane oluşturur. Diataxis
terminolojisini bilmenize gerek yok — yetenek çıktıyı sizin için etiketler.

## 1. Adım: Yeteneği herhangi bir projede çağırın

Belgelendirmek istediğiniz projede Claude Code'u açın. Yazın:

```
/document-generate
```

Yetenek çıktı hedefi hakkında bir soru sorar:

```
A) Mevcut dosyalarda satır içi belgelendirme yaz (README, ARCHITECTURE, vb.)
B) Bağımsız belgelendirme dosyaları oluştur (örn., docs/ dizini)
C) İkisi de — mevcut dosyalarda satır içi özetler + bağımsız dosyalarda derin belgeler

ÖNERİ: Hem bulunabilirliği hem de derinliği en üst düzeye çıkardığı için C'yi seçin.
```

C'yi seçin. Bir README işaretçisi artı tam bir bağımsız belge seti alacaksınız.

## 2. Adım: Arkeoloji çalışmasını izleyin

Yetenek ~30 saniye boyunca sessiz kalırken kod tabanını okur. Bu kasıtlıdır — 1. Adım
"Kod Tabanı Arkeolojisi" aşaması iş akışındaki en önemli adımdır. Yetenek şunları okuyor:

- Tam depo yapısı
- README, ARCHITECTURE, CONTRIBUTING, CLAUDE.md (giriş noktaları)
- Belgelendirdiğiniz her şeyin uygulama dosyaları (tam dosya, yalnızca imzalar değil)
- Testler (uç durumları ve amaçlanan davranışı ortaya çıkarır)
- `// NOTE:`, `// DESIGN:`, `// WHY:` ile etiketlenmiş satır içi yorumlar

Bitirdiğinde, şöyle bir satır görürsünüz:

```
47 dosya araştırıldı, 12 genel yüzey öğesi, 8 kavram ve 4 tasarım kararı tanımlandı.
```

Bu sayı, yeteneğin dosya adlarından tahmin etmek yerine gerçekten kodu okuduğunu size söyler.

## 3. Adım: Diataxis bölümleme planını görün

Yetenek, hangi varlık için hangi kadranlarda yazacağını gösteren bir bölümleme planı
yazdırır:

```
Belgelendirme planı:
  [varlık]              [eğitim] [nasıl yapılır] [referans] [açıklama]
  WidgetService         ✅ yeni     ✅ yeni   ✅ yeni      ✅ yeni
  --verbose bayrağı        ❌        ✅ yeni   ✅ satır içi   ❌
  Bayesian zamanlayıcı    ❌        ❌       ✅ yeni      ✅ yeni
```

Her varlığın dört kadranın hepsine ihtiyacı yoktur. CLI bayrakları referans + nasıl yapılır
alır. Dahili modüller referans + açıklama alır. Kullanıcıya dönük özellikler dördünü de alır.
Yetenek varlık türüne göre seçer.

Planda 5'ten fazla belge varsa, yetenek devam etmeden önce onaylamanızı ister. Aksi
takdirde devam eder.

## 4. Adım: İnen ilk belgeyi okuyun

Referans belgeleri önce iner çünkü sözlüğü sabitlerler. Şöyle satırlar görürsünüz:

```
GENERATED: docs/reference-widget-service.md
```

O dosyayı açın. Katı bir yapıya sahiptir: bir paragraflık giriş, türler ve varsayılanlarla
tam API listesi, 2-3 çalıştırılabilir örnek ve sıradaki nasıl yapılır ve eğitime bağlantı
veren bir İlgili bölüm.

Diataxis'te referans belgeleri böyledir: gerçekçi, kapsamlı, anlatı yok. Bir seçeneğin
neden var olduğunu açıklamak istiyorsanız, bu yeteneğin bir sonraki yazacağı açıklama
belgesine aittir.

## 5. Adım: Açıklama, nasıl yapılır ve eğitimin belirmesini görün

Hızlı bir sırayla (her biri ~5-10 saniye), yetenek kalan kadranları yazar:

```
GENERATED: docs/explanation-widget-architecture.md
GENERATED: docs/howto-create-a-custom-widget.md
GENERATED: docs/tutorial-build-your-first-widget.md
```

Her birini açın. Birbirlerini tekrar etmediklerine dikkat edin:

- **Açıklama** sorunla başlar, ardından yaklaşım, ardından ödünleşimler ve düşünülen
  alternatifler
- **Nasıl yapılır** ön koşullar, tam komutlarla numaralandırılmış adımlar, bir doğrulama
  bölümü ve bir sorun giderme bölümü içerir
- **Eğitim** 3 adım veya daha azında çalışan bir sonuca ulaştırır, "Ne inşa ettiniz" ile
  biter

Yetenek bu yapıları zorlar. Bir nasıl yapılırda doğrulama bölümü eksikse, 8. Adım Kalite
Özdenetimi commit öncesinde yakalar.

## 6. Adım: Çapraz bağlantıları denetleyin

Her belge diğerlerine bağlantı verir. Referans belgesi İlgili bölümü: nasıl yapılır ve
eğitime bağlantı verir. Nasıl yapılır İlgili bölümü: referansa bağlantı verir. Eğitim
"Ne inşa ettiniz" bölümü: daha derin keşif için referansa bağlantı verir.

Bozuk bağlantı olmadığını doğrulamak için bir grep çalıştırın:

```bash
grep -rE '\]\([^)]*\.md\)' docs/ | head -10
```

Bağlantı verilen her dosya mevcut olmalıdır. Yeteneğin 7. Adım "Çapraz Belge Bağlantısı
ve Keşfedilebilirlik" adımı commit öncesinde bunu denetler.

## 7. Adım: PR gövdesindeki kapsam özetini görün

Açık bir PR'ye sahip bir özellik dalındaysanız, yetenek PR gövdesini bir
`## Documentation Generated` tablosu ile günceller:

```
## Documentation Generated

| Dosya | Kadran | Açıklama |
|------|----------|-------------|
| docs/tutorial-build-your-first-widget.md | Eğitim | Kurulumdan ilk çalışan widget'a yol gösterimi |
| docs/reference-widget-service.md | Referans | Türler, varsayılanlar, örneklerle tam widget API'si |
| docs/explanation-widget-architecture.md | Açıklama | Widget'ların neden yalıtılmış servisler olduğu |
| docs/howto-create-a-custom-widget.md | Nasıl yapılır | Özel widget'lar oluşturma ve kaydetme |
```

Bir gözden geçiren PR'yi açtığında tabloyu görür ve ne tür bir kapsamın gönderildiğini
hemen bilir.

## Ne inşa ettiniz

Artık dört farklı okuyucuya hizmet eden dört belgeye sahipsiniz:

- Projenize yeni gelen biri `tutorial-*.md` dosyasını okuyup çalışan bir şey elde edebilir
- Deneyimli bir kullanıcı belirli bir görevi başarmak için `howto-*.md` dosyasını okuyabilir
- Bir API çağırıcısı tam imzalar için `reference-*.md` dosyasını okuyabilir
- Bir kod gözden geçiren tasarımı anlamak için `explanation-*.md` dosyasını okuyabilir

Her biri bakımı yeterince kısa. Her birinin tek bir işi var. PR gövdesi hangi kadranların
kapsandığını gösterir. Daha sonra `/document-release` çalıştırırsanız, Diataxis kapsam
haritası bu varlığı tamamen kapsanmış (4/4 kadran) olarak raporlar.

## Sırada ne yapmalı

- **Boşluklarınız varsa** /document-release işaretledi ama doldurmadı: kapsamı özellikle
  o varlıklara sınırlandırarak `/document-generate` komutunu tekrar çalıştırın.
- **Dört kadranın neden var olduğunu anlamak istiyorsanız:** [explanation-diataxis-in-gstack.md](./explanation-diataxis-in-gstack.md) dosyasını okuyun.
- **Belirli bir yayımlanan özelliği belgelendirmek istiyorsanız** (tüm proje değil):
  [howto-document-a-shipped-feature.md](./howto-document-a-shipped-feature.md) dosyasını okuyun.
- **Yeteneğin kendisi için referans:** [`document-generate/SKILL.md`](../document-generate/SKILL.md).