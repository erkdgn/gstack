# DX Onur Listesi Referansı

Yalnızca mevcut inceleme geçişi için olan bölümü okuyun. Dosyanın tamamını yüklemeyin.

## Geçiş 1: Başlangıç

**Altın standartlar:**
- **Stripe**: Kartla ödeme almak için 7 satır kod. Belgeler, giriş yapınca SİZİN test API anahtarlarınızı önceden doldurur. Stripe Shell, belgeler sayfası içinde CLI çalıştırır. Yerel kurulum gerekmez.
- **Vercel**: `git push` = HTTPS ile küresel CDN'de canlı site. Her PR önizleme URL'si alır. Tek CLI komutu: `vercel`.
- **Clerk**: `<SignIn />`, `<SignUp />`, `<UserButton />`. 3 JSX bileşeni, kutudan çıkan e-posta, sosyal, MFA ile çalışan kimlik doğrulama.
- **Supabase**: Bir Postgres tablosu oluşturun, anında REST API + Realtime + kendi-belgeli-belge oluşturur.
- **Firebase**: `onSnapshot()`. Çevrimdışı kalıcılıkla tüm istemcilerde gerçek zamanlı senkronizasyon için 3 satır.
- **Twilio**: Konsolda Sanal Telefon. Numara satın almadan, kredi kartı olmadan SMS gönder/al. Sonuç: Aktivasyonda %62 iyileşme.

**Anti-kalıplar:**
- Herhangi bir değer sunmadan önce e-posta doğrulama (akışı keser)
- Sandbox ortamı öncesinde kredi kartı zorunluluğu
- Birden fazla yol içeren "kendi maceranı seç" (karar yorgunluğu; tek altın yol kazanır)
- Ayarlarda gizli API anahtarları (Stripe bunları kod örneklerine önceden doldurur)
- Dil değiştirme özelliği olmayan statik kod örnekleri
- Panodan ayrı belgeler sitesi (bağlam değiştirme)

## Geçiş 2: API/CLI/SDK Tasarımı

**Altın standartlar:**
- **Stripe önekli ID'ler**: Ücretler için `ch_`, müşteriler için `cus_`. Kendi-belgeli. Yanış ID türünü geçirmek imkansız.
- **Stripe genişletilebilir nesneler**: Varsayılan olarak ID dizgileri döndürür. `expand[]` ile satır içi tam nesneleri alır. 4 seviyeye kadar iç içe genişletme.
- **Stripe idempotency anahtarları**: Mutasyonlarda `Idempotency-Key` başlığını geçirin. Güvenli yeniden denemeler. "Çift mi ücretlendirdim?" endişesi yok.
- **Stripe API sürümleme**: İlk çağrı hesabı o günün sürümüne sabitler. `Stripe-Version` başlığı ile istek başına yeni sürümleri test edin.
- **GitHub CLI**: Terminal vs boru'yu otomatik algılar. Terminal'de insan-okunabilir, borulandığında sekme-ayrılmış. `gh pr <tab>` tüm PR eylemlerini gösterir.
- **SwiftUI aşamalı açılım**: `Button("Kaydet") { kaydet() }`'dan tam özelleştirmeye, her seviyede aynı API.
- **htmx**: JS yerine HTML nitelikleri. Toplam 14KB. `hx-get="/search" hx-trigger="keyup changed delay:300ms"`. Sıfır derleme adımı.
- **shadcn/ui**: Kaynak kodunu projenize kopyalayın. Her satırın sahibi sizsiniz. Bağımlık yok, sürüm çakışması yok.

**Anti-kalıplar:**
- Chattiness: Bir kullanıcı-görünür eylem için 5 çağrı gerektiren API
- Tutarsız adlandırma: `/users` (çoğul) vs `/user/123` (tekil) vs `/create-order` (URL'de fiil)
- Örtük hata: Yanıt gövdesine gömülü hata ile 200 OK
- Tanrı endpoint: Farklı davranış gösteren 47 parametre kombinasyonu
- Belgelendirme-gerekli API: İlk çağrıdan önce 3 sayfa belge = fazla seremoni

## Geçiş 3: Hata Mesajları ve Hata Ayıklama

**Hata kalitesinin üç seviyesi:**

**Seviye 1, Elm (Sohbet Derleyicisi):**
```
-- TÜR UYUMSUZLUĞU ---- src/Main.elm
Şu String değerlerle toplama yapamam:
42|   "merhaba" + 1
     ^^^^^^^
İpucu: Dizgeleri birleştirmek için (++) operatörünü kullanın.
```
Birinci şahıs, tam cümleler, kesin konum, önerilen düzeltme, daha fazla okuma.

**Seviye 2, Rust (Açıklamalı Kaynak):**
```
error[E0308]: tür uyumsuzluğu
 --> src/main.rs:4:20
yardım: burada ödünç almayı düşünün
  |
4 |     let isim: &str = &ismi_al();
  |                       +
```
Hata kodu öğreticiye bağlantı verir. Birincil + ikincil etiketler. Yardım bölümü kesin düzenlemeyi gösterir.

**Seviye 3, Stripe API (doc_url ile Yapılandırılmış):**
```json
{"error":{"type":"invalid_request_error","code":"resource_missing","message":"Böyle bir müşteri yok: 'cus_hicbiri'","param":"customer","doc_url":"https://stripe.com/docs/error-codes/resource-missing"}}
```
Beş alan, sıfır belirsizlik.

**Formül:** Ne oldu + Neden + Nasıl düzeltilir + Nereden daha fazla öğrenilir + Buna neden olan gerçek değerler.

**Anti-kalıp:** TypeScript "Bunu mu demek istediniz?" önerisini UZUN hata zincirlerinin EN ALTINA gömer. En eyleme geçirilebilir bilgi İLK sırada yer almalıdır.

## Geçiş 4: Belgeler ve Öğrenme

**Altın standartlar:**
- **Stripe belgeleri**: Üç sütunlu düzen (navigasyon / içerik / canlı kod). Giriş yapınca API anahtarları enjekte edilir. Dil değiştirici TÜM sayfalarda kalıcı. Hover ile vurgulama. Tarayıcı içi API çağrıları için Stripe Shell. Oluşturulan ve açık kaynaklı Markdoc. Özellikler, belgeler tamamlanana kadar gönderilmez. Belge katkıları performans incelemelerini etkiler.
- Geliştiricilerin %52'si belgelendirme eksikliği nedeniyle engelleniyor (Postman 2023)
- Dünyaca üstün belgelere sahip şirketlerde benimsenmede 2.5 kat artış görülüyor
- "Belgeler ürün olarak": Özellik ile birlikte gönderilir veya özellik gönderilmez

## Geçiş 5: Yükseltme ve Geçiş Yolu

**Altın standartlar:**
- **Next.js**: `npx @next/codemod upgrade major`. Tek komut Next.js, React, React DOM'u yükseltir ve tüm ilgili codemod'ları çalıştırır.
- **AG Grid**: v31+'den itibaren her sürüm bir codemod içerir.
- **Stripe API sürümleme**: Dahili olarak tek kod tabanı. Hesap başına sürüm sabitleme. Bozucu değişiklikler sizi asla şaşırtmaz.
- **Martin Fowler'ın boru hattı deseni**: Monolitik bir codemod yerine küçük, test edilebilir dönüşümlerden oluşan boru hattı.
- Maven Central'daki bozucu değişikliklerin %21.9'u belgelenmemişti (Ochoa ve ark., 2021)

## Geçiş 6: Geliştirici Ortamı ve Araçlar

**Altın standartlar:**
- **Bun**: npm install'dan 100 kat daha hızlı, Node.js çalışma zamanından 4 kat daha hızlı. Hız = DX'dir.
- Günde ortalama 87 kesinti; her birinden kurtulmak 25 dakika. Geliştiriciler günde sadece 2-4 saat kod yazıyor.
- DXI'da her 1 puanlık iyileşme = geliştirici başına haftada 13 dakika tasarruf.
- **GitHub Copilot**: %55.8 daha hızlı görev tamamlama. PR süresi 9.6 günden 2.4 güne.

## Geçiş 7: Topluluk ve Ekosistem

- Geliştirici araçları satın almadan önce ~14 maruziyet gerektirir (Matt Biilmann, Netlify). Üç aylık OKR döngüleriyle uyumsuz.
- Güçlü geliştirici deneyimine sahip ekiplerde 4-5 kat performans çarpanı (DevEx çerçevesi).

## Geçiş 8: DX Ölçümü

**Üç akademik çerçeve:**
1. **SPACE** (Microsoft Research, 2021): Memnuniyet, Performans, Aktivite, İletişim, Verimlilik. En az 3 boyutu ölçün.
2. **DevEx** (ACM Queue, 2023): Geri bildirim döngüleri, Bilişsel yük, Akış durumu. Algısal + iş akışı verilerini birleştirin.
3. **Fagerholm & Munch** (IEEE, 2012): Biliş, Duygu, İrade. Psikolojik "zihin üçlemesi".

## Claude Code Beceri DX Kontrol Listesi

Claude Code becerileri, MCP sunucuları veya AI ajan araçları için planları incelerken kullanın.

- [ ] **AskUserQuestion tasarımı**: Her çağrıda bir sorun. Bağlamı yeniden yerleştir (proje, dal, görev). Görsel geri bildirim için tarayıcı devri.
- [ ] **Durum depolama**: Global (~/.tool/) vs proje-bazlı ($SLUG/) vs oturum-bazlı. Denetim izleri için salt-ekleme JSONL.
- [ ] **Aşamalı onay**: İşaret dosyaları ile tek seferlik istemler. Asla tekrar sorma. Geri alınabilir.
- [ ] **Otomatik yükseltme**: Önbellek + erteleme geri çekilmesi ile sürüm kontrolü. Geçiş betikleri. Satır içi teklif.
- [ ] **Beceri birleştirme**: Fayda sağlayan zincirler. İnceleme zincirleme. Bölüm atlama ile satır içi çağırma.
- [ ] **Hata kurtarma**: Arızadan devam. Kısmi sonuçlar korunur. Kontrol noktası güvenli.
- [ ] **Oturum sürekliliği**: Zaman çizelgesi olayları. Sıkıştırma kurtarma. Oturumlar arası öğrenmeler.
- [ ] **Sınırlı özerklik**: Açık operasyonel sınırlar. Yıkıcı eylemler için zorunlu yükseltme. Denetim izleri.

Referans uygulamalar: gstack'in tasarım-silah döngüsü, otomatik yükseltme akışı, aşamalı onay, hiyerarşik depolama.