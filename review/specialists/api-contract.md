# API Sözleşmesi Uzman İnceleme Kontrol Listesi

Kapsam: SCOPE_API=true olduğunda
Çıktı: JSON nesneleri, satır başına bir bulgu. Şema:
{"severity":"CRITICAL|INFORMATIONAL","confidence":N,"path":"dosya","line":N,"category":"api-contract","summary":"...","fix":"...","fingerprint":"path:line:api-contract","specialist":"api-contract"}
İsteğe bağlı: line, fix, fingerprint, evidence, test_stub.
Bulgu yoksa: `NO FINDINGS` çıktısı ve başka hiçbir şey.

---

## Kategoriler

### Bozucu Değişiklikler
- Yanıt gövdelerinden kaldırılan alanlar (istemciler bunlara bağlı olabilir)
- Değiştirilen alan türleri (string → number, object → array)
- Mevcut uç noktalara eklenen yeni zorunlu parametreler
- Değiştirilen HTTP yöntemleri (GET → POST) veya durum kodları (200 → 201)
- Eski yolu yönlendirme/takma ad olarak korumadan yeniden adlandırılan uç noktalar
- Değiştirilen kimlik doğrulama gereksinimleri (public → authenticated)

### Sürümleme Stratejisi
- Sürüm artışı olmaksızın yapılan bozucu değişiklikler (v1 → v2)
- Aynı API'de karıştırılan birden fazla sürümleme stratejisi (URL vs header vs sorgu parametresi)
- Son kullanım zaman çizelgesi veya geçiş kılavuzu olmaksızın kullanım dışı bırakılan uç noktalar
- Merkezi hale getirilmek yerine denetleyiciler arasında dağınık olan sürüme özgü mantık

### Hata Yanıtı Tutarlılığı
- Mevcut olanlardan farklı hata biçimleri döndüren yeni uç noktalar
- Standart alanları eksik olan hata yanıtları (hata kodu, mesaj, ayrıntılar)
- Hata türüyle eşleşmeyen HTTP durum kodları (hatalar için 200, doğrulama için 500)
- İç uygulama ayrıntılarını sızdıran hata mesajları (yığın izleri, SQL)

### Hız Sınırlama & Sayfalama
- Benzer uç noktalarda hız sınırlama varken yeni uç noktalarda eksik olan hız sınırlama
- Geriye dönük uyumluluk olmaksızın sayfalama değişiklikleri (offset → cursor)
- Belgelendirme olmaksızın değiştirilen sayfa boyutları veya varsayılan limitler
- Sayfalandırılmış yanıtlarda eksik toplam sayı veya sonraki-sayfa göstergeleri

### Belgelendirme Sapması
- Yeni uç noktaları veya değiştirilen parametreleri yansıtacak şekilde güncellenmemiş OpenAPI/Swagger belirtimi
- Değişikliklerden sonra eski davranışı açıklayan README veya API belgeleri
- Artık çalışmayan örnek istekler/yanıtlar
- Yeni uç noktalar veya değiştirilen parametreler için eksik belgelendirme

### Geriye Dönük Uyumluluk
- Eski sürümlerdeki istemciler: bozulacak mı?
- Zorla güncelleme yapamayan mobil uygulamalar: API onlar için hala çalışıyor mu?
- Abonelere bildirimde bulunulmaksızın değiştirilen webhook yükleri
- Yeni özellikleri kullanmak için SDK veya istemci kitaplığı değişiklikleri gerekli