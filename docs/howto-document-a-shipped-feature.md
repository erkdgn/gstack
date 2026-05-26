# Yayımlanan bir özelliği nasıl belgelendirirsiniz

Bu, yayınlama sonrası iş akışıdır: bir PR'yi birleştirdiniz, belgeler eskidi ve bir
geçişte hem kapsam haritası hem de doldurulmuş boşluklar istiyorsunuz. Denetim için
`/document-release`, ardından bulduğu boşlukları doldurmak için `/document-generate`
çalıştıracaksınız.

## Ön koşullar

- gstack kurulu (`./setup` tamamlanmış; `which gstack` ile veya Claude Code'da `/`
  yazıp yetenekleri listeleyerek doğrulayın)
- Yayımlanan özelliğinizin bulunduğu dal checkout edilmiş
- GitHub veya GitLab'de bir PR var (önerilen — iş akışı PR gövdesini bir kapsam haritası
  ile günceller)

Henüz bir PR yoksa, önce `/ship` çalıştırarak bir tane oluşturun; `/document-release`
buna karşı çalışacak şekilde tasarlanmıştır.

## Adımlar

### 1. Mevcut kapsamı denetle

Çalıştırın:

```
/document-release
```

Yetenek, diffinizi temel dala karşı yürür, yeni genel yüzeyi (yetenekler, CLI
bayrakları, yapılandırma seçenekleri, API uç noktaları, yeni modüller) çıkarır ve her
varlığı dört Diataxis kadranı üzerinden puanlar. Şöyle bir kapsam haritası görürsünüz:

```
Coverage map:
  [varlık]         [referans?] [nasıl yapılır?] [eğitim?] [açıklama?]
  /new-skill       ✅ AGENTS.md  ❌        ❌          ❌
  --new-flag       ✅ README     ✅ README  ❌          ❌
  FooProcessor     ❌            ❌        ❌          ❌
```

Sıfır kapsama sahip öğeler **kritik boşluklar**. Yalnızca referans kapsama sahip öğeler
**yaygın boşluklar**. Her ikisi de gözden geçirenlerin görmesi için PR gövdesinde
`### Documentation Debt` alt bölümü olarak yer alır.

`/document-release` her şeyin kapsandığını bildiriyorsa, işiniz bitti. Bu nasıl yapılırın
geri kalanını atlayın.

### 2. PR gövdesindeki belgelendirme borç bölümünü okuyun

PR'nizi açın (yetenek URL'yi yazdırır). `## Documentation` → `### Documentation Debt`
bölümüne gidin. Her öğe, dolduracak Diataxis kadranıyla etiketlenmiştir:

```
### Documentation Debt

- ⚠️ /new-skill — AGENTS.md dosyasında referans var ama README'de nasıl yapılır örneği yok. Diataxis kadranı: nasıl yapılır.
- ⚠️ FooProcessor — sıfır kapsam. Diataxis kadranları: referans, açıklama.
```

Bu, bir sonraki adımın girdisidir. Her satır size neyin eksik olduğunu ve hangi
kadranın dolduracağını söyler.

### 3. Boşlukları /document-generate ile doldurun

Çalıştırın:

```
/document-generate
```

Yetenek kapsam sorusunu sorduğunda, borç bölümünde işaretlenen belirli varlıkları söyleyin.
Yetenek kod tabanını okur (1. Adım arkeoloji aşaması zorunludur), Diataxis kadranına
göre böler ve eksik belgeleri yazar.

Yeteneğin otomatik olarak keşfetmesine de izin verebilirsiniz: `/document-release`
boşlukları açıkça size geçtiyse (zincirlendiğinde bunu yapar), `/document-generate`
zaten ne yazacağını bilir.

### 4. Boşlukların kapandığını doğrulayın

`/document-release` komutunu yeniden çalıştırın:

```
/document-release
```

Kapsam haritası artık daha önce işaretlenen varlıkları, daha önce boş olan kadranlarda
yeşil onay işaretleriyle göstermelidir. PR gövdesinin Documentation Debt bölümü boş
olmalı veya yalnızca bilerek ertelenen öğeleri içermelidir.

## Doğrulama

PR'nizi açın ve şunları doğrulayın:

1. PR gövdesinde bir belge fark önizlemesi ile bir `## Documentation` bölümü var.
2. `### Documentation Debt` alt bölümü sıfır kritik boşluk listeliyor (veya yalnızca
   bilerek ertelenen öğeler).
3. `docs/` dizinindeki her üretilen belge dosyası düzgün açılıyor ve kardeşlerine
   çapraz bağlantı veriyor (referans → nasıl yapılır → eğitim → açıklama).
4. `grep -rE '\]\([^)]*\.md\)' docs/` çalıştırın ve hiçbir bağlantının eksik bir
   dosyaya işaret etmediğini doğrulayın.

Dördü de geçiyorsa, PR'niz eksiksiz belgelendirme ile birleştirilmeye hazır.

## Sorun giderme

**`/document-release` "No public surface changes detected." raporluyor.**
Diff yalnızca dahili (yeniden düzenlemeler, testler, altyapı). Belgeye gerek yok.
Birleştirmeye geçin.

**Boşluktaki Diataxis kadranı etiketi beklediğinizle eşleşmiyor.**
Yetenek, hangi kadranların önemli olduğuna karar vermek için bir varlık taksonomisi
kullanır (CLI bayrakları referans + nasıl yapılır ister; dahili modüller referans +
açıklama ister; kullanıcıya dönük özellikler dördünü de ister). Katılmıyorsanız,
üretimden sonra belgeleri el ile düzenleyerek geçersiz kılabilirsiniz. Denetim bir
kılavuzdur, bir kısıtlama değil.

**`/document-generate` çalışan bir sonuca ulaşmak için 8 adım süren bir eğitim yazıyor.**
Eğitimler 3 adım veya daha azında çalışan bir sonuç vermelidir. Yeteneği yeniden
çalıştırıp sıkıştırmasını isteyin veya el ile düzenleyin. 8. Adım Kalite Özdenetimi
bunlardan bazılarını yakalar ama hepsini değil.

**Bir özelliği belgelendirmek istiyorsunuz ama henüz bir PR yok.**
Önce PR'yi oluşturmak için `/ship` çalıştırın, ardından bu iş akışını uygulayın. PR
olmadan `/document-release` yine de denetim yapabilir ama PR gövdesi güncellemesini atlar.

**Üretilen bir referans belgesi, API imzalarını halüsinasyonla uydurmuş.**
Bir hata dosyası açın. Yeteneğin 1. Adım arkeolojisi, tam olarak bunu önlemek için
imzaları değil, uygulama dosyalarını baştan sona okumak zorundadır. Üretilen metni ve
gerçek kodu ekleyin, böylece arkeolojinin bunu neden kaçırdığını izleyebiliriz.

## İlgili

- **İlk kez `/document-generate` kullanımı için eğitim:** [tutorial-document-generate.md](./tutorial-document-generate.md)
- **gstack neden Diataxis çerçevesini kullanıyor:** [explanation-diataxis-in-gstack.md](./explanation-diataxis-in-gstack.md)
- **Denetim yeteneği için referans:** [`document-release/SKILL.md`](../document-release/SKILL.md)
- **Üretim yeteneği için referans:** [`document-generate/SKILL.md`](../document-generate/SKILL.md)