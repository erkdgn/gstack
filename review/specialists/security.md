# Güvenlik Uzman İnceleme Kontrol Listesi

Kapsam: SCOPE_AUTH=true VEYA (SCOPE_BACKEND=true VE dif > 100 satır) olduğunda
Çıktı: JSON nesneleri, satır başına bir bulgu. Şema:
{"severity":"CRITICAL|INFORMATIONAL","confidence":N,"path":"dosya","line":N,"category":"security","summary":"...","fix":"...","fingerprint":"path:line:security","specialist":"security"}
İsteğe bağlı: line, fix, fingerprint, evidence, test_stub.
Bulgu yoksa: `NO FINDINGS` çıktısı ve başka hiçbir şey.

---

Bu kontrol listesi ana KRİTİK geçişinden daha derine gider. Ana ajan zaten SQL enjeksiyonu, yarış durumları, LLM güveni ve enum tamlığını kontrol ediyor. Bu uzman kimlik doğrulama/yetkilendirme desenleri, kriptografik yanlış kullanım ve saldırı yüzeyi genişlemesine odaklanır.

## Kategoriler

### Güven Sınırlarında Girdi Doğrulama
- Denetleyici/işleyici düzeyinde doğrulama olmaksızın kabul edilen kullanıcı girdisi
- Veritabanı sorgularında veya dosya yollarında doğrudan kullanılan sorgu parametreleri
- Tür kontrolü veya şema doğrulaması olmaksızın kabul edilen istek gövdesi alanları
- Tür/boyut/içerik doğrulaması olmaksızın dosya yüklemeleri
- İmza doğrulaması olmaksızın işlenen webhook yükleri

### Kimlik Doğrulama ve Yetkilendirme Atlama
- Kimlik doğrulama ara katmanı eksik olan uç noktalar (rota tanımlarını kontrol edin)
- Varsayılan olarak "izin ver" olan yetkilendirme kontrolleri ("reddet" yerine)
- Rol yükseltme yolları (kullanıcı kendi rolünü/izinlerini değiştirebilir)
- Doğrudan nesne referansı güvenlik açıkları (A kullanıcısı, B kullanıcısının verilerine bir ID değiştirerek erişebilir)
- Oturum sabitleme veya oturum ele geçirme fırsatları
- Son kullanma süresini kontrol etmeyen belirteç/API anahtarı doğrulaması

### Enjeksiyon Vektörleri (SQL ötesi)
- Kullanıcı kontrollü argümanlarla alt süreç çağrıları ile komut enjeksiyonu
- Kullanıcı girdisi ile şablon enjeksiyonu (Jinja2, ERB, Handlebars)
- Dizin sorgularında LDAP enjeksiyonu
- Kullanıcı kontrollü URL'ler üzerinden SSRF (fetch, redirect, webhook hedefleri)
- Kullanıcı kontrollü dosya yolları ile yol geçişi (../../etc/passwd)
- HTTP başlıklarında kullanıcı kontrollü değerler ile başlık enjeksiyonu

### Kriptografik Yanlış Kullanım
- Güvenlik duyarlı işlemler için zayıf hash algoritmaları (MD5, SHA1)
- Belirteçler veya sırlar için öngörülebilir rastgelelik (Math.random, rand())
- Sırlar, belirteçler veya özetler üzerinde zaman sabit olmayan karşılaştırmalar (==)
- Sabit kodlanmış şifreleme anahtarları veya IV'ler
- Parola hash'inde eksik tuz

### Sırların Açığa Çıkması
- Kaynak kodundaki API anahtarları, belirteçler veya parolalar (yorumlarda bile)
- Uygulama günlüklerinde veya hata mesajlarında günlüğe kaydedilen sırlar
- URL'lerdeki kimlik bilgileri (sorgu parametreleri veya URL'de temel kimlik doğrulaması)
- Kullanıcılara döndürülen hata yanıtlarında hassas veriler
- Şifrelemenin beklendiği durumda düz metin olarak saklanan PII

### Kaçış Kapakları ile XSS
- Rails: kullanıcı kontrollü veriler üzerinde .html_safe, raw()
- React: kullanıcı içeriği ile dangerouslySetInnerHTML
- Vue: kullanıcı içeriği ile v-html
- Django: kullanıcı girdisi üzerinde |safe, mark_safe()
- Genel: temizlenmemiş verilerle innerHTML ataması

### Serileştirmeden Kurtarma
- Güvenilmeyen verilerin serileştirmeden kurtarılması (pickle, Marshal, YAML.load, çalıştırılabilir türlerin JSON.parse'ı)
- Şema doğrulaması olmaksızın kullanıcı girdisinden veya harici API'lerden kabul edilen serileştirilmiş nesneler