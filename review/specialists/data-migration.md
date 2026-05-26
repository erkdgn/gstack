# Veri Göç Uzman İnceleme Kontrol Listesi

Kapsam: SCOPE_MIGRATIONS=true olduğunda
Çıktı: JSON nesneleri, satır başına bir bulgu. Şema:
{"severity":"CRITICAL|INFORMATIONAL","confidence":N,"path":"dosya","line":N,"category":"data-migration","summary":"...","fix":"...","fingerprint":"path:line:data-migration","specialist":"data-migration"}
İsteğe bağlı: line, fix, fingerprint, evidence, test_stub.
Bulgu yoksa: `NO FINDINGS` çıktısı ve başka hiçbir şey.

---

## Kategoriler

### Geri Döndürülebilirlik
- Bu göç veri kaybı olmaksızın geri alınabilir mi?
- Karşılık gelen bir down/geri alma göçü var mı?
- Geri alma gerçekten değişikliği mi geri alıyor yoksa sadece no-op mu?
- Geri alma mevcut uygulama kodunu bozar mı?

### Veri Kaybı Riski
- Hala veri içeren sütunları düşürmek (önce kullanım dışı bırakma dönemi ekleyin)
- Verileri kesen sütun türü değişiklikleri (varchar(255) → varchar(50))
- Kodun bunlara referans vermediğini doğrulamadan tabloları kaldırmak
- Tüm referansları güncellemeden sütunları yeniden adlandırmak (ORM, ham SQL, görünümler)
- Mevcut NULL değerleri olan sütunlara eklenen NOT NULL kısıtlamaları (önce geri doldurma gerekir)

### Kilit Süresi
- Büyük tablolarda CONCURRENTLY olmaksızın ALTER TABLE (PostgreSQL)
- >100K satırlı tablolarda CONCURRENTLY olmaksızın indeks ekleme
- Tek bir kilit ediniminde birleştirilebilecek birden fazla ALTER TABLE ifadesi
- Yoğun trafik saatlerinde özel kilitler edinen şema değişiklikleri

### Geri Doldurma Stratejisi
- DEFAULT değeri olmaksızın yeni NOT NULL sütunlar (kısıtlamadan önce geri doldurma gerekir)
- Toplu olarak doldurulması gereken hesaplanmış varsayılanlara sahip yeni sütunlar
- Mevcut kayıtlar için eksik geri doldurma betiği veya rake görevi
- Toplu işlemek yerine tüm satırları tek seferde güncelleyen geri doldurma (tabloyu kilitler)

### İndeks Oluşturma
- Üretim tablolarında CONCURRENTLY olmaksızın CREATE INDEX
- Yinelenen indeksler (yeni indeks mevcut olanla aynı sütunları kapsıyor)
- Yeni yabancı anahtar sütunlarında eksik indeksler
- Tam indeksin daha yararlı olacağı (veya tam tersi) kısmi indeksler

### Çok Aşamalı Güvenlik
- Uygulama koduyla belirli bir sırayla dağıtılması gereken göçler
- Mevcut çalışan kodu bozan şema değişiklikleri (önce kodu dağıtın, sonra göç geçirin)
- Dağıtım sınırı varsayan göçler (eski kod + yeni şema = çökme)
- Yuvarlanan dağıtım sırasında karışık eski/yeni kodu ele almak için eksik özellik bayrağı