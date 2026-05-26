# QA Sorun Taksonomisi

## Ciddiyet Düzeyleri

| Ciddiyet | Tanım | Örnekler |
|----------|-------|----------|
| **kritik** | Temel bir iş akışını engeller, veri kaybına neden olur veya uygulamayı çökertir | Form gönderimi hata sayfasına yönlendirir, ödeme akışı bozuk, onay olmadan veri silinir |
| **yüksek** | Önemli bir özellik bozuk veya kullanılamaz, geçici çözüm yok | Arama yanlış sonuçlar döndürür, dosya yükleme sessizce başarısız olur, kimlik doğrulama yönlendirme döngüsü |
| **orta** | Özellik çalışıyor ancak belirgin sorunlar var, geçici çözüm mevcut | Yavaş sayfa yüklemesi (>5sn), form doğrulama eksik ancak gönderim hala çalışıyor, düzen yalnızca mobilde bozuk |
| **düşük** | Küçük kozmetik veya cilalama sorunu | Altbilgide yazım hatası, 1px hizalama sorunu, üzerine gelme durumu tutarsız |

## Kategoriler

### 1. Görsel/Arayüz
- Düzen bozulmaları (çakışan öğeler, kırpılmış metin, yatay kaydırma çubuğu)
- Bozuk veya eksik görseller
- Yanlış z-index (öğelerin arkasında görünmesi)
- Yazı tipi/renk tutarsızlıkları
- Animasyon sorunları (kasma, tamamlanmamış geçişler)
- Hizalama sorunları (ızgara dışı, düzensiz aralık)
- Karanlık mod / tema sorunları

### 2. İşlevsel
- Bozuk bağlantılar (404, yanlış hedef)
- Ölü düğmeler (tıklama hiçbir şey yapmaz)
- Form doğrulama (eksik, yanlış, atlanabilir)
- Yanlış yönlendirmeler
- Durumun korunmaması (yenileme veya geri düğmesinde veri kaybı)
- Yarış koşulları (çift gönderim, eski veri)
- Arama yanlış veya hiç sonuç döndürmüyor

### 3. Kullanıcı Deneyimi
- Kafa karıştırıcı gezinme (breadcrumb yok, çıkmaz sokaklar)
- Eksik yükleme göstergeleri (kullanıcı bir şey olduğunu bilmiyor)
- Yavaş etkileşimler (>500ms geri bildirim olmadan)
- Belirsiz hata mesajları ("Bir şeyler ters gitti" detay yok)
- Yıkıcı eylemlerden önce onay olmaması
- Sayfalar arasında tutarsız etkileşim kalıpları
- Çıkmaz sokaklar (geri dönme yolu yok, sonraki eylem yok)

### 4. İçerik
- Yazım ve dilbilgisi hataları
- Güncel olmayan veya yanlış metin
- Yer tutucu / lorem ipsum metni kalmış
- Kırpılmış metin (üç nokta veya "daha fazla" olmadan kesilmiş)
- Düğmelerde veya form alanlarında yanlış etiketler
- Eksik veya yararsız boş durumlar

### 5. Performans
- Yavaş sayfa yüklemeleri (>3 saniye)
- Kasmalı kaydırma (düşen kareler)
- Düzen kaymaları (yüklemeden sonra içeriğin atlaması)
- Aşırı ağ istekleri (tek sayfada >50)
- Büyük optimize edilmemiş görseller
- Engelleyen JavaScript (yükleme sırasında sayfa yanıt vermemez)

### 6. Konsol/Hatalar
- JavaScript istisnaları (yakalanmamış hatalar)
- Başarısız ağ istekleri (4xx, 5xx)
- Kullanımdan kaldırma uyarıları (yaklaşan kırılmalar)
- CORS hataları
- Karışık içerik uyarıları (HTTPS üzerinde HTTP kaynakları)
- CSP ihlalleri

### 7. Erişilebilirlik
- Görsellerde eksik alt metin
- Etiketlenmemiş form girdileri
- Klavye gezintisi bozuk (öğelere sekme ile ulaşılamıyor)
- Odak tuzakları (modal veya dropdown'dan çıkılamıyor)
- Eksik veya yanlış ARIA nitelikleri
- Yetersiz renk kontrastı
- Ekran okuyucu ile ulaşılamayan içerik

## Sayfa Başına Keşif Kontrol Listesi

Bir QA oturumu sırasında ziyaret edilen her sayfa için:

1. **Görsel tarama** — Açıklamalı ekran görüntüsü al (`snapshot -i -a -o`). Düzen sorunları, bozuk görseller, hizalama için kontrol et.
2. **Etkileşimli öğeler** — Her düğmeye, bağlantıya ve kontrole tıkla. Her biri söylediği şeyi yapıyor mu?
3. **Formlar** — Doldur ve gönder. Boş gönderim, geçersiz veri, uç durumlar (uzun metin, özel karakterler) test et.
4. **Gezinme** — İçeri/dışarı tüm yolları kontrol et. Breadcrumb'lar, geri düğmesi, derin bağlantılar, mobil menü.
5. **Durumlar** — Boş durum, yükleme durumu, hata durumu, dolu/taşkın durum kontrol et.
6. **Konsol** — Etkileşimlerden sonra `console --errors` çalıştır. Yeni JS hataları veya başarısız istekler var mı?
7. **Duyarlılık** — İlgiliyse, mobil ve tablet görünümlerini kontrol et.
8. **Kimlik doğrulama sınırları** — Oturum kapatıldığında ne olur? Farklı kullanıcı rolleri?