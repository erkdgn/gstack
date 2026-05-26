# Test Uzman İnceleme Kontrol Listesi

Kapsam: Her zaman açık (her inceleme)
Çıktı: JSON nesneleri, satır başına bir bulgu. Şema:
{"severity":"CRITICAL|INFORMATIONAL","confidence":N,"path":"dosya","line":N,"category":"testing","summary":"...","fix":"...","fingerprint":"path:line:testing","specialist":"testing"}
İsteğe bağlı: line, fix, fingerprint, evidence, test_stub.
Bulgu yoksa: `NO FINDINGS` çıktısı ve başka hiçbir şey.

---

## Kategoriler

### Eksik Negatif-Yol Testleri
- Hataları, reddetmeleri veya geçersiz girdiyi işleyen ve karşılık gelen TESTİ OLMAYAN yeni kod yolları
- Test edilmemiş koruma ifadeleri ve erken dönüşler
- Başarısızlık-yolu testi olmaksızın try/catch, rescue veya hata sınırlarındaki hata dalları
- Kodda iddia edilen ancak "reddedildi" durumu için hiçbir zaman test edilmemiş izin/kimlik doğrulama kontrolleri

### Eksik Sınır-Durumu Kapsamı
- Sınır değerleri: sıfır, negatif, maks-tamsayı, boş dizge, boş dizi, nil/null/undefined
- Tek elemanlı koleksiyonlar (döngülerde bir-fazla hatası)
- Kullanıcıya bakan girdilerde Unicode ve özel karakterler
- Yarış durumu testi olmaksızın eşzamanlı erişim desenleri

### Test İzolasyonu İhlalleri
- Değiştirilebilir durumu paylaşan testler (sınıf değişkenleri, global singleton'lar, temizlenmemiş DB kayıtları)
- Sıraya bağımlı testler (sıralı geçen, rastgele çalıştırıldığında başarısız olan)
- Sistem saatine, saat dilimine veya yerel aya bağımlı testler
- Stub/mock kullanmak yerine gerçek ağ çağrıları yapan testler

### Geçici (Flaky) Test Desenleri
- Zamanlama bağımlı iddialar (sleep, setTimeout, sıkı zaman aşımları ile waitFor)
- Sıralanmamış sonuçların sıralaması üzerinde iddialar (hash anahtarları, Set yinelemesi, zaman uyumsuz çözümleme sırası)
- Geri dönüş olmaksızın harici hizmetlere (API'ler, veritabanları) bağımlı testler
- Tohum kontrolü olmaksızın rastgeleleştirilmiş test verileri

### Güvenlik Uygulama Testleri Eksik
- "Yetkisiz" durumu için testi olmaksızın denetleyicilerdeki kimlik doğrulama/yetkilendirme kontrolleri
- Gerçekte engellediğini kanıtlayan testi olmaksızın hız sınırlandırma mantığı
- Kötü niyetli girdi için testi olmaksızın girdi temizleme
- Entegrasyon testi olmaksızın CSRF/CORS yapılandırması

### Kapsam Boşlukları
- Sıfır test kapsamı olan yeni genel yöntemler/fonksiyonlar
- Mevcut testlerin yalnızca eski davranışı kapsadığı, yeni dalı kapsamadığı değiştirilmiş yöntemler
- Birden fazla yerden çağrılan ancak yalnızca dolaylı olarak test edilen yardımcı fonksiyonlar