# Bakım Uzman İnceleme Kontrol Listesi

Kapsam: Her zaman açık (her inceleme)
Çıktı: JSON nesneleri, satır başına bir bulgu. Şema:
{"severity":"INFORMATIONAL","confidence":N,"path":"dosya","line":N,"category":"maintainability","summary":"...","fix":"...","fingerprint":"path:line:maintainability","specialist":"maintainability"}
İsteğe bağlı: line, fix, fingerprint, evidence, test_stub.
Bulgu yoksa: `NO FINDINGS` çıktısı ve başka hiçbir şey.

---

## Kategoriler

### Ölü Kod ve Kullanılmayan İthalatlar
- Değiştirilen dosyalarda atanmış ancak hiç okunmamış değişkenler
- Tanımlanmış ancak hiç çağrılmamış fonksiyonlar/yöntemler (repo genelinde Grep ile kontrol edin)
- Değişiklikten sonra artık referans verilmeyen import/require ifadeleri
- Yorum satırına alınmış kod blokları (ya kaldırın ya da neden var olduklarını açıklayın)

### Sihirli Sayılar ve Dize Bağlantısı
- Mantıkta kullanılan çıplak sayısal sabitler (eşikler, limitler, yeniden deneme sayıları) — adlandırılmış sabitler olmalı
- Başka yerlerde sorgu filtresi veya koşul olarak kullanılan hata mesajı dizgeleri
- Yapılandırma olması gereken sabit kodlanmış URL'ler, bağlantı noktaları veya ana bilgisayar adları
- Birden fazla dosyada yinelenen sabit değerler

### Eskimiş Yorumlar ve Belge Dizgeleri
- Bu dif'te kod değiştirildikten sonra eski davranışı açıklayan yorumlar
- Tamamlanmış işe referans veren TODO/FIXME yorumları
- Mevcut fonksiyon imzasıyla eşleşmeyen parametre listelerine sahip belge dizgeleri
- Artık kod akışıyla eşleşmeyen ASCII diyagramları içeren yorumlar

### DRY İhlalleri
- Dif içinde birden fazla kez görünen benzer kod blokları (3+ satır)
- Paylaşılan bir yardımcının daha temiz olacağı kopyala-yapıştır desenleri
- Test dosyaları arasında yinelenen yapılandırma veya kurulum mantığı
- Bir arama tablosu veya eşlem olabilecek yinelenen koşul zincirleri

### Koşullu Yan Etkiler
- Bir koşula dallanan ancak bir dalı yan etkisi unuturan kod yolları
- Bir eylemin gerçekleştiğini iddia eden ancak eylemin koşullu olarak atlandığı günlük mesajları
- Bir dalın ilgili kayıtları güncellediği ancak diğerinin güncellemediği durum geçişleri
- Yalnızca mutlu yolda çalışan, hata/kenar yolları eksik olay yayımları

### Modül Sınırı İhlalleri
- Başka bir modülün iç uygulamasına erişim (kurala göre özel yöntemlere erişim)
- Bir hizmet/model üzerinden gitmesi gereken denetleyiciler/görünümlerdeki doğrudan veritabanı sorguları
- Arayüzler üzerinden iletişim kurması gereken bileşenler arasındaki sıkı bağ