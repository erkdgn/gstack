# Chrome Yan Panel + Conductor: İhtiyacımız Olan Şey

## Ne inşa ediyoruz

Şu anda Claude bir Conductor çalışma alanında çalışırken — dosyaları düzenlerken,
testler çalıştırırken, uygulamanıza göz atarken — onu sadece Conductor'ın sohbet
penceresinden izleyebilirsiniz. Claude uygulamanızda QA yapıyorsa, araç çağrılarının
kaydırıldığını görürsünüz ama tarayıcıyı gerçekten *göremezsiniz*.

Bunu düzelten bir Chrome yan paneli inşa ettik. `$B connect` çalıştırdığınızda,
Chrome, Claude'un gerçek zamanlı olarak ne yaptığını gösteren bir yan panel ile açılır.
Yan panelde mesajlar yazabilirsiniz ve Claude onlara göre hareket eder — "kayıt ol
düğmesine tıkla", "ayarlar sayfasına git", "gördüğünü özetle."

Sorun: yan panel şu anda kendi ayrı Claude örneğini çalıştırıyor. Ana Conductor
oturumunun ne yaptığını göremiyor ve ana oturum yan panelin ne yaptığını göremiyor.
Bunlar birbiriyle konuşmayan iki ayrı ajan.

Düzeltme basit: yan paneli Conductor oturumuna bir *pencere* haline getirmek,
ayrı bir şey değil.

## Conductor'dan ihtiyacımız olan 3 şey

### 1. Ajanın ne yaptığını izlememize izin verin

Aktif oturumun etkinliklerine abone olmamız gereken bir yol gerekiyor. SSE akışı
veya WebSocket gibi, bize etkinlikleri olduğu gibi gönderen bir şey:

- "Claude `src/App.tsx`'i düzenliyor"
- "Claude `npm test` çalıştırıyor"
- "Claude diyor ki: CSS sorununu düzelteceğim..."

Yan panel zaten bu etkinlikleri nasıl render edeceğini biliyor — araç çağrıları
kompakt rozetler, metin sohbet balonları olarak gösterilir. Conductor'ın oturumundan
uzantumuza bir boruya ihtiyacımız var.

### 2. Oturuma mesaj göndermemize izin verin

Kullanıcı Chrome yan panelinde "diğer düğmeye tıkla" yazdığında, bu mesaj
kullanıcının çalışma alanı sohbetinde yazmış gibi Conductor oturumunda görünmelidir.
Ajan bunu bir sonraki turunda alır ve ona göre hareket eder.

Bu sihirli an: kullanıcı Chrome'u izliyor, bir şeylerin yanlış olduğunu görüyor,
yan panelde bir düzeltme yazıyor ve Claude yanıt veriyor — kullanıcı hiç pencere
değiştirmeden.

### 3. Bir dizinden çalışma alanı oluşturmamıza izin verin

`$B connect` başlattığında, dosya izolasyonu için bir git worktree oluşturur.
Bu worktree'i bir Conductor çalışma alanı olarak kaydetmek istiyoruz, böylece kullanıcı
yan panel ajanının dosya değişikliklerini Conductor'ın dosya ağacında görebilir.
Bu aynı zamanda birden fazla tarayıcı oturumu için, her birinin kendi çalışma alanı
olduğu temel sağlar.

## Neden önemli

Bugün `/qa` ve `/design-review` siyah bir kutu gibi hissediliyor. Claude "3 sorun buldum"
diyor ama neye baktığını göremezsiniz. Yan panel Conductor'a bağlandığında:

- **Claude'un uygulamanızı test ettiğini gerçek zamanlı izlersiniz** — her tıklama,
  her gezinme, her ekran görüntüsü Chrome'da izlerken görünür
- **Kesintiye uğratabilirsiniz** — "hayır, mobil görünümü test et" veya "o sayfayı atla" —
  pencereler arasında geçiş yapmadan
- **Bir ajan, iki görünüm** — kodunuzu düzenleyen aynı Claude tarayıcıyı da kontrol eder.
  Bağlam tekrarı yok, eski durum yok

## Zaten inşa edilenler (gstack tarafında)

Tarafımızdaki her şey tamam ve gönderiliyor:

- `$B connect` çalıştırdığınızda otomatik yüklenen Chrome uzantısı
- Otomatik açılan yan panel (kullanıcı için sıfır kurulum)
- Akışlı etkinlik render edici (araç çağrıları, metin, sonuçlar)
- Mesaj kuyruklama ile sohbet girişi
- Durum banner'ları ile yeniden bağlanma mantığı
- Kalıcı sohbet geçmişi ile oturum yönetimi
- Ajan yaşam döngüsü (başlat, durdur, öldür, zaman aşımı algılama)

Tarafımızdaki tek değişiklik: veri kaynağını "yerel `claude -p` alt sürecinden"
"Conductor oturum akışına" değiştirmek. Uzantı kodu aynı kalır.

**Tahmini çaba:** 2-3 gün Conductor mühendisliği, 1 gün gstack entegrasyonu.