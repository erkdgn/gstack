# Performans Uzman İnceleme Kontrol Listesi

Kapsam: SCOPE_BACKEND=true VEYA SCOPE_FRONTEND=true olduğunda
Çıktı: JSON nesneleri, satır başına bir bulgu. Şema:
{"severity":"CRITICAL|INFORMATIONAL","confidence":N,"path":"dosya","line":N,"category":"performance","summary":"...","fix":"...","fingerprint":"path:line:performance","specialist":"performance"}
İsteğe bağlı: line, fix, fingerprint, evidence, test_stub.
Bulgu yoksa: `NO FINDINGS` çıktısı ve başka hiçbir şey.

---

## Kategoriler

### N+1 Sorgular
- İstekli yükleme olmaksızın döngülerde gezilen ActiveRecord/ORM ilişkileri (.includes, joinedload, include)
- Toplu işlenebilecek yineleme blokları içindeki veritabanı sorguları (each, map, forEach)
- Tembel yüklenmiş ilişkileri tetikleyen iç içe serileştiriciler
- DataLoader kullanımını kontrol ederek alan başına sorgulama yerine toplu iş yapan GraphQL çözücüler

### Eksik Veritabanı İndeksleri
- İndeksleri olmayan sütunlarda yeni WHERE yan cümleleri (göç dosyalarını veya şemayı kontrol edin)
- İndekslenmemiş sütunlarda yeni ORDER BY
- Bileşik indeksleri olmayan bileşik sorgular (WHERE a AND b)
- İndeksleri olmaksızın eklenen yabancı anahtar sütunları

### Algoritmik Karmaşıklık
- O(n^2) veya daha kötü desenler: koleksiyonlar üzerinde iç içe döngüler, Array.map içinde Array.find
- Bir hash/map/set araması kullanabilecek yinelenen doğrusal aramalar
- Döngülerde dize birleştirme (join veya StringBuilder kullanın)
- Bir kez yeterli olabilecekken birden fazla kez sıralama veya filtreleme yapılan büyük koleksiyonlar

### Paket Boyutu Etkisi (Ön Uç)
- Ağır olduğu bilinen yeni üretim bağımlılıkları (moment.js, lodash full, jquery)
- Derin içe aktarmalar (library/specific'ten import) yerine varil içe aktarmaları (library'den import)
- Optimizasyon olmaksızın işlenen büyük statik varlıklar (görseller, yazı tipleri)
- Rota düzeyinde parçalar için eksik kod bölme

### Render Performansı (Ön Uç)
- Getirme şelaleleri: paralel olabilecek sıralı API çağrıları (Promise.all)
- Kararsız referanslardan gelen gereksiz yeniden render'lar (render'da yeni nesneler/diziler)
- Pahalı hesaplamalarda eksik React.memo, useMemo veya useCallback
- Döngülerde DOM özelliklerini okuyup ardından yazmaktan gelen düzen sarsıntısı
- Kat altı görsellerde eksik loading="lazy"

### Eksik Sayfalama
- Sınırsız sonuç döndüren liste uç noktaları (LIMIT yok, sayfalama parametreleri yok)
- Veri hacmiyle büyüen LIMIT olmaksızın veritabanı sorguları
- Genişletma ile ID'ler yerine tam iç içe nesneler gömen API yanıtları

### Zaman Uyumsuz Bağlamlarda Engelleme
- Zaman uyumsuz fonksiyonlar içinde zaman uyumlu G/Ç (dosya okumaları, alt süreç, HTTP istekleri)
- Olay döngüsü tabanlı işleyiciler içinde time.sleep() / Thread.sleep()
- Çalışanı boşaltmaksızın ana iş parçacığını engelleyen CPU yoğun hesaplama