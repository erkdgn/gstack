# Mimari

Bu belge gstack'in **neden** bu şekilde oluşturulduğunu açıklar. Kurulum ve komutlar için CLAUDE.md'ye, katkıda bulunmak için CONTRIBUTING.md'ye bakın.

## Temel fikir

gstack, Claude Code'a kalıcı bir tarayıcı ve bir deye görüşlü iş akışı yeteneği verir. Zor olan kısım tarayıcıdır — geriye kalan her şey Markdown'dur.

Temel içgörü: Bir tarayıcıyla etkileşime giren bir yapay zeka temsilcisinin **saniyenin altında gecikme** ve **kalıcı durum** (state) gereksinimi vardır. Her komut tarayıcıyı soğuk başlatıyorsa, her araç çağrısı başına 3-5 saniye beklersiniz. Tarayıcı komutlar arasında ölürse, çerezleri, sekmeleri ve oturumları kaybedersiniz. Bu nedenle gstack, CLI'nin localhost HTTP üzerinden konuştuğu uzun ömürlü bir Chromium arka plan programı çalıştırır.

```
Claude Code                     gstack
─────────                      ──────
                               ┌──────────────────────┐
  Tool call: $B snapshot -i    │  CLI (derlenmiş binary)│
  ─────────────────────────→   │  • durum dosyasını okur │
                               │  • POST /command       │
                               │    localhost:PORT'a     │
                               └──────────┬───────────┘
                                          │ HTTP
                               ┌──────────▼───────────┐
                               │  Sunucu (Bun.serve)   │
                               │  • komutu yönlendirir │
                               │  • Chromium ile konuşur│
                               │  • düz metin döndürür │
                               └──────────┬───────────┘
                                          │ CDP
                               ┌──────────▼───────────┐
                               │  Chromium (headless)   │
                               │  • kalıcı sekmeler     │
                               │  • çerezler taşınır    │
                               │  • 30dk boşta zaman aşımı│
                               └───────────────────────┘
```

İlk çağrı her şeyi başlatır (~3s). Sonraki her çağrı: ~100-200ms.

## Neden Bun

Node.js de çalışırdı. Burada Bun üç nedene daha iyidir:

1. **Derlenmiş binary'ler.** `bun build --compile` tek bir ~58MB çalıştırılabilir dosya üretir. Çalışma zamanında `node_modules` yok, `npx` yok, PATH yapılandırması yok. Binary doğrudan çalışır. Bu önemlidir çünkü gstack, kullanıcıların bir Node.js projesi yönetmeyi beklemediği `~/.claude/skills/` konumuna kurulur.

2. **Yerel SQLite.** Çerez şifre çözme, Chromium'un SQLite çerez veritabanını doğrudan okur. Bun'da yerleşik `new Database()` vardır — `better-sqlite3` yok, yerel eklenti derlemesi yok, gyp yok. Farklı makinelerde bozulan bir şey daha az.

3. **Yerel TypeScript.** Sunucu geliştirme sırasında `bun run server.ts` olarak çalışır. Derleme adımı yok, `ts-node` yok, hata ayıklamak için source map yok. Derlenmiş binary dağıtım içindir; kaynak dosyalar geliştirme içindir.

4. **Yerleşik HTTP sunucusu.** `Bun.serve()` hızlıdır, sadedir ve Express veya Fastify gerektirmez. Sunucu toplam ~10 rotayı işler. Bir framework fazla yük getirirdi.

Darboğaz her zaman Chromium'dur, CLI veya sunucu değil. Bun'un başlatma hızı (~1ms derlenmiş binary için, ~100ms Node için) güzel ama seçmemizin nedeni bu değil. Derlenmiş binary ve yerel SQLite sebebidir.

## Arka plan programı modeli

### Neden komut başına tarayıcı başlatmıyoruz?

Playwright Chromium'u ~2-3 saniyede başlatabilir. Tek bir ekran görüntüsü için bu sorun değil. 20+ komutlu bir QA oturumu içinse 40+ saniye tarayıcı başlatma yükü oluşur. Daha kötüsü: komutlar arasındaki tüm durumu kaybedersiniz. Çerezler, localStorage, oturumlar, açık sekmeler — hepsi gider.

Arka plan programı modeli şu anlama gelir:

- **Kalıcı durum.** Bir kez oturum açın, oturum açık kalsın. Bir sekme açın, açık kalsın. localStorage komutlar arasında kalıcıdır.
- **Saniyenin altında komutlar.** İlk çağrıdan sonra, her komut yalnızca bir HTTP POST'dur. Chromium'un çalışması dahil ~100-200ms gidiş-dönüş.
- **Otomatik yaşam döngüsü.** Sunucu ilk kullanımda otomatik başlar, 30 dakika boşta kaldıktan sonra otomatik kapanır. Süreç yönetimine gerek yoktur.

### Durum dosyası

Sunucu `.gstack/browse.json` dosyasını yazar (tmp + rename ile atomik yazma, mod 0o600):

```json
{ "pid": 12345, "port": 34567, "token": "uuid-v4", "startedAt": "...", "binaryVersion": "abc123" }
```

CLI sunucuyu bulmak için bu dosyayı okur. Dosya eksikse veya sunucu bir HTTP sağlık kontrolünde başarısız olursa, CLI yeni bir sunucu başlatır. Windows'ta PID tabanlı süreç algılama Bun binary'lerinde güvenilmezdir, bu nedenle sağlık kontrolü (GET /health) tüm platformlarda birincil canlılık sinyalidir.

### Port seçimi

10000-60000 arasında rastgele port (çakışmada en fazla 5 yeniden deneme). Bu, 10 Conductor çalışma alanının sıfır yapılandırma ve sıfır port çakışmasıyla kendi browse arka plan programını çalıştırabileceği anlamına gelir. Eski yaklaşım (9400-9409 taraması) çoklu çalışma alanı kurulumlarında sürekli bozuluyordu.

### Sürüm otomatik yeniden başlatma

Derleme, `git rev-parse HEAD` çıktısını `browse/dist/.version` dosyasına yazar. Her CLI çağrısında, binary'nin sürümü çalışan sunucunun `binaryVersion` değeriyle eşleşmezse, CLI eski sunucuyu öldürür ve yenisini başlatır. Bu, "eski binary" hata sınıfını tamamen önler — binary'yi yeniden derleyin, sonraki komut otomatik olarak alır.

## Güvenlik modeli

### Yalnızca localhost

HTTP sunucusu `127.0.0.1`'e bağlanır, `0.0.0.0`'a değil. Ağdan erişilemez.

### Çift dinleyici tünel mimarisi (v1.6.0.0)

Bir kullanıcı `pair-agent --client` çalıştırdığında, arka plan programı uzaktaki eşleştirilmiş bir temsilcinin tarayıcıyı kullanabilmesi için bir ngrok tüneli başlatır. Tam arka plan programı yüzeyini internete açmak (rastgele bir ngrok alt alanının arkasında bile), `/health`'in herhangi bir Origin sahteciliğinde kök token'ı sızdırması ve `/cookie-picker`'ın token'ı her çağırıcının alabileceği HTML'e gömmesi anlamına geliyordu.

Çözüm **iki HTTP dinleyicidir**, bir değil:

- **Yerel dinleyici** (`127.0.0.1:LOCAL_PORT`) — her zaman bağlıdır. Önyükleme (`/health` ile token teslimi), `/cookie-picker`, `/inspector/*`, `/welcome`, `/refs`, sidebar-agent API'si ve tam komut yüzeyini sunar. Hiçbir zaman yönlendirilmez.
- **Tünel dinleyici** (`127.0.0.1:TUNNEL_PORT`) — `/tunnel/start` üzerinde tembel olarak bağlanır, `/tunnel/stop` üzerinde kaldırılır. Kilitli bir izin listesi sunar: `/connect` (eşleştirme seremonisi, yetkisiz + hız sınırlı), `/command` (yalnızca kapsamlı token'lar, tarayıcı-kullanma komut izin listesiyle daha fazla kısıtlı) ve `/sidebar-chat`. Diğer her şey 404 döner.

ngrok yalnızca tünel portunu yönlendirir. Güvenlik özelliği **fiziksel port ayrımından** gelir: bir tünel çağırıcısı `/health` veya `/cookie-picker`'a erişemez çünkü bu yollar o TCP soketinde mevcut değildir. Başlık çıkarımı (`x-forwarded-for` kontrolü, origin kontrolü) güvenilmezdir (ngrok başlık davranışı değişebilir; yerel proxy'ler bu başlıkları ekleyebilir); soket ayrımı güvenilirdir.

| Uç nokta | Yerel dinleyici | Tünel dinleyici | Notlar |
|---|---|---|---|
| `GET /health` | herkese açık (başlı/eklenti olmadıkça token yok) | 404 | Eklenti için token önyüklemesi yalnızca yerel olarak gerçekleşir |
| `GET /connect` | herkese açık (`{alive:true}`) | herkese açık (`{alive:true}`) | Tünel canlılığı için yoklama yolu |
| `POST /connect` | herkese açık (hız sınırı 300/dk) | herkese açık (hız sınırı) | pair-agent için kurulum anahtarı değişimi |
| `POST /command` | yetkili (Bearer root VEYA kapsamlı) | yetkili (yalnızca kapsamlı, izin listeli komutlar) | Tünel üzerinde kök token = 403 |
| `POST /sidebar-chat` | yetkili | yetkili | Uzak temsilcinin yerel sidebar'a mesaj göndermesine izin verir |
| `POST /pair` | yalnızca kök | 404 | Eşleştirme basımı — yerel operatör eylemi |
| `POST /tunnel/{start,stop}` | yalnızca kök | 404 | Arka plan programı yapılandırması |
| `POST /token`, `DELETE /token/:id` | yalnızca kök | 404 | Kapsamlı token basımı/iptal |
| `GET /cookie-picker`, `GET /cookie-picker/*` | herkese açık UI, yetkili API | 404 | Yalnızca yerel — yerel tarayıcı DB'lerini okur |
| `GET /inspector`, `/inspector/events`, vb. | yetkili | 404 | Eklenti geri çağırması, yalnızca yerel |
| `GET /welcome` | herkese açık | 404 | GStack Browser açılış sayfası, yalnızca yerel |
| `GET /refs` | yetkili | 404 | Ref haritası — iç durum |
| `GET /activity/stream` | Bearer VEYA HttpOnly `gstack_sse` çerezi | 404 | SSE. ?token= sorgu parametresi artık kabul edilmiyor |
| `GET /inspector/events` | Bearer VEYA HttpOnly `gstack_sse` çerezi | 404 | SSE. /activity/stream ile aynı çerez |
| `POST /sse-session` | yetkili (Bearer) | 404 | 30 dakikalık salt-görüntüleme SSE oturum çerezini basar |

**Tünel yüzey reddi günlükleri.** Tünel dinleyicisindeki her reddetme (`path_not_on_tunnel`, `root_token_on_tunnel`, `missing_scoped_token`, `disallowed_command:*`) zaman damgası, kaynak IP (`x-forwarded-for`'dan), yol ve yöntem ile asenkron olarak `~/.gstack/security/attempts.jsonl` dosyasına kaydedilir. Günlük taşması DoS'unu önlemek için global olarak 60 yazma/dk hız sınırına tabidir. Deneme günlüğünü prompt-enjeksiyon tarayıcısıyla paylaşır.

**SSE oturum çerezleri.** EventSource Authorization başlıkları gönderemez, bu nedenle eklenti önyükleme sırasında kök Bearer ile bir kez `/sse-session` uç noktasına POST yapar ve 30 dakikalık salt-görüntüleme çerezi (`gstack_sse`, HttpOnly, SameSite=Strict) alır. Çerez YALNIZCA `/activity/stream` ve `/inspector/events` için geçerlidir — kapsamlı bir token DEĞİLDİR ve `/command` üzerinde kullanılamaz. Kapsam izolasyonu modül sınırı tarafından uygulanır: `sse-session-cookie.ts`'nin `token-registry.ts`'den içe aktarması yoktur.

**Bu dalga ilişkin hedef dışı** (#1136 olarak izleniyor): çerez-içe-aktarma-tarayıcı yolu Chrome'u `--remote-debugging-port=<rastgele>` ile başlatır. App-Bound Encryption v20 ile Windows'ta, aynı kullanıcı yerel süreci bu porta bağlanabilir ve şifresi çözülmüş v20 çerezlerini sızdırabilir — bu, SQLite DB'sini doğrudan okumaya (DPAPI bağlamı olmadan v20'nin şifresini çözemez) göre bir yetki yükseltme yoludur. Düzeltme yönü TCP yerine `--remote-debugging-pipe`'dır; CDP istemcisinin yeniden yapılandırılmasını gerektirir.

### Bearer token yetkilendirmesi

Her sunucu oturumu rastgele bir UUID token üretir, mod 0o600 (yalnızca sahibinin okuyabileceği) ile durum dosyasına yazar. Tarayıcı durumunu değiştiren her HTTP isteği `Authorization: Bearer <token>` içermelidir. Token eşleşmezse, sunucu 401 döner.

Bu, aynı makinedeki diğer süreçlerin browse sunucunuzla konuşmasını engeller. Çerez seçici UI'ı (`/cookie-picker`) ve sağlık kontrolü (`/health`) yerel dinleyicide muaftır — bunlar 127.0.0.1'e bağlıdır ve komut çalıştırmaz. Tünel dinleyicisinde `/connect` dışında hiçbir şey muaftır.

### Çerez güvenliği

Çerezler gstack'in işlediği en hassas verilerdir. Tasarım:

1. **Anahtarlık erişimi kullanıcı onayı gerektirir.** Tarayıcı başına ilk çerez içe aktarma macOS Anahtarlık iletişim kutusunu tetikler. Kullanıcının "İzin Ver" veya "Her Zaman İzin Ver" seçeneğine tıklaması gerekir. gstack asla kimlik bilgilerine sessizce erişmez.

2. **Şifre çözme süreç içinde gerçekleşir.** Çerez değerleri bellekte (PBKDF2 + AES-128-CBC) çözülür, Playwright bağlamına yüklenir ve asla düz metin olarak diske yazılmaz. Çerez seçici UI'ı çerez değerlerini asla göstermez — yalnızca alan adları ve sayılar.

3. **Veritabanı salt-okunurdur.** gstack, Chromium çerez DB'sini (çalışan tarayıcı ile SQLite kilit çakışmalarını önlemek için) geçici bir dosyaya kopyalar ve salt-okunur açar. Gerçek tarayıcınızın çerez veritabanını asla değiştirmez.

4. **Anahtar önbellekleme oturum başınadır.** Anahtarlık parolası + türetilmiş AES anahtarı sunucunun yaşam süresi boyunca bellekte önbelleğe alınır. Sunucu kapandığında (boşta zaman aşımı veya açık durdurma), önbellek temizlenir.

5. **Günlüklerde çerez değeri yok.** Konsol, ağ ve iletişim kutusu günlükleri hiçbir zaman çerez değerleri içermez. `cookies` komutu çerez meta verilerini (alan adı, ad, son kullanma) çıkarır, ancak değerler kesilir.

### Kabuk enjeksiyonu önleme

Tarayıcı kayıt defteri (Comet, Chrome, Arc, Brave, Edge) sabit kodlanmıştır. Veritabanı yolları bilinen sabitlerden oluşturulur, asla kullanıcı girişinden değil. Anahtarlık erişimi, kabuk dize interpolasyonu değil, açık argüman dizileriyle `Bun.spawn()` kullanır.

### Sunucu çıkışında Unicode arıtma (v1.38.0.0)

CDP tarafından toplanan sayfa içeriği yalnız UTF-16 vekil yarımçiftleri (sayfadaki bozuk JavaScript dize işlemeninden kaynaklanan yetim yüksek veya düşük vekiller) içerebilir. Bunlar `JSON.stringify`'e ulaştığında, Bun onları `\uD800` tarzı kaçış dizileri olarak yayar ve bu dizileri aşağı akış tüketicisinin `JSON.parse`'ı kabul eder, ancak Anthropic API 400 hatasıyla reddeder — tek bir tuhaf sayfayı oturum öldüren bir hataya dönüştürür. Savunma tek noktalıdır, sayfadan türetilmiş dizeleri gönderen her sunucu çıkışına uygulanır.

| Çıkış yolu | Modül | Arıtma noktası |
|---|---|---|
| `POST /command` (HTTP) | `browse/src/server.ts` | `handleCommandInternal` sarmalayıcısı (`handleCommandInternalImpl` sonucunu arıtır) |
| `POST /command/batch` | `browse/src/server.ts` | Aynı sarmalayıcı — toplu tüketiciler onu devralır |
| `GET /activity/stream` (SSE) | `browse/src/server.ts` | `JSON.stringify`'e geçirilen `sanitizeReplacer` |
| `GET /inspector/events` (SSE) | `browse/src/server.ts` | `JSON.stringify`'e geçirilen `sanitizeReplacer` |

`sanitizeReplacer`, kodlama sırasında her dize değerini temizleyen bir `JSON.stringify` değiştirici fonksiyondur. Kodlama sonrası regex burada çalışmaz — `JSON.stringify`, regex eşleşmeden önce `\uD800`'i `"\\ud800"` literal kaçış dizisine zaten dönüştürmüştür, bu nedenle değiştiricinin kodlama ardışık düzeni içinde çalışması gerekir. Saf dize yardımcısı `sanitizeLoneSurrogates`, `text/plain` yanıtları için doğrudan kullanılır.

**Mimari değişmez.** Sayfa içeriğinden türetilmiş dizeleri gönderen her yeni SSE/WebSocket yazıcısı veya HTTP yanıtı iki yoldan birini ZORUNLU olarak izlemelidir: nesne yükleri için `JSON.stringify(payload, sanitizeReplacer)`, veya metin gövdeleri için `sanitizeLoneSurrogates(body)`. İkisini de atlayan yeni yüzeyler sistemi eşzamansız hale getirir. `server.ts`'deki her iki SSE üreticisindeki satır içi yorumlar bunu belirtir; `browse/test/server-sanitize-surrogates.test.ts`, hata yeniden üretimi + değişmez testleriyle bağlantıyı sabitler (`handleCommandInternalImpl` yeniden adlandırma, merkezi arıtma satırı, değiştirici varlığı, SSE üreticilerinin değiştirici ile stringify'ı).

### Prompt enjeksiyonu savunması (sidebar agent)

Chrome sidebar agent'ı araçlara (Bash, Read, Glob, Grep, WebFetch) sahiptir ve düşmanca web sayfalarını okur, bu nedenle gstack'in prompt enjeksiyonuna en açık olan parçasıdır. Savunma katmanlıdır, tek noktalı değil.

1. **L1-L3 içerik güvenliği (`browse/src/content-security.ts`).** Her sayfa-içeriği komutunda ve her araç çıktısında çalışır: veri işaretleme, gizli öğe çıkarma, ARIA regex, URL engelleme listesi ve güven sınırı zarf sarmalayıcısı. Hem sunucuda hem de agent'ta uygulanır.

2. **L4 ML sınıflandırıcısı — TestSavantAI (`browse/src/security-classifier.ts`).** Agent ile paketlenmiş 22MB BERT-small ONNX modeli (int8 nicemlenmiş). Yerel olarak çalışır, ağ yok. Claude görmeden önce her kullanıcı mesajını ve her Read/Glob/Grep/WebFetch araç çıktısını tarar. `GSTACK_SECURITY_ENSEMBLE=deberta` ile isteğe bağlı 721MB DeBERTa-v3 topluluğu.

3. **L4b transkript sınıflandırıcısı.** Yalnızca metne değil, tam konuşma şekline (kullanıcı mesajı, araç çağrıları, araç çıktısı) bakan bir Claude Haiku geçişi. `LOG_ONLY: 0.40` ile sınırlandırılmıştır, bu nedenle çoğu temiz trafik ücretli çağrıyı atlar.

4. **L5 kanarya token (`browse/src/security.ts`).** Oturum başlangıcında sistem prompt'una enjekte edilen rastgele bir token. `text_delta` ve `input_json_delta` akışlarındaki yuvarlanan-arabellek algılama, token Claude'ın çıktısında, araç argümanlarında, URL'lerde veya dosya yazmalarında herhangi bir yerde belirirse onu yakalar. Belirleyici ENGEL — token sızarsa, saldırgan Claude'ı sistem prompt'unu açığa çıkarmaya ikna etmiştir ve oturum sonlanır.

5. **L6 topluluk birleştirici (`combineVerdict`).** ENGEL, iki ML sınıflandırıcının >= `WARN` (0.75) düzeyinde anlaşmasını gerektirir, tek bir güvenilir eşleşme yeterli değildir. Bu, Stack Overflow talimat-yazma yanlış pozitif azaltımıdır. Araç çıktısı taramalarında, tek katmanlı yüksek güvenli ENGEL'ler doğrudan çalışır — içerik kullanıcı tarafından yazılmamıştır, bu nedenle yanlış pozitif endişesi geçerli değildir.

**Kritik kısıtlama:** `security-classifier.ts` yalnızca sidebar-agent sürecinde çalışır, asla derlenmiş browse binary'sinde değil. `@huggingface/transformers` v4, `onnxruntime-node` gerektirir ve bu, Bun compile'ın geçici çıkarma dizininden `dlopen` işleminde başarısız olur. Yalnızca saf dize parçaları (kanarya enjekte/kontrol, karar birleştirici, saldırı günlüğü, durum) `security.ts`'dedir ve bu `server.ts`'den içe aktarmak için güvenlidir.

**Ortam değişkeni düğmeleri:** `GSTACK_SECURITY_OFF=1` gerçek bir kapatma anahtarıdır (ML taramasını atlar, kanarya yine de enjekte edilir). Model önbelleği: `~/.gstack/models/testsavant-small/` (112MB, ilk çalıştırma) ve `~/.gstack/models/deberta-v3-injection/` (721MB, yalnızca isteğe bağlı). Saldırı günlüğü: `~/.gstack/security/attempts.jsonl` (tuzlanmış sha256 + alan adı, 10MB'de döner, 5 nesil). Cihaz başına tuz: `~/.gstack/security/device-salt` (0600), FS-yazılamaz ortamlarda hayatta kalmak için süreç içinde önbelleğe alınır.

**Görünürlük.** Sidebar başlığı `/sidebar-chat` üzerinden yoklanan bir kalkan simgesi (yeşil/amber/kırmızı) gösterir. Kanarya sızıntısı veya ENGEL kararı olduğunda kesin katman puanlarıyla ortalanmış bir banner görünür. `bin/gstack-security-dashboard` yerel girişimleri toplar; `supabase/functions/community-pulse` kullanıcılar arası isteğe bağlı topluluk telemetrisini toplar.

## Ref sistemi

Ref'ler (`@e1`, `@e2`, `@c1`), agent'in CSS seçicileri veya XPath yazmadan sayfa öğelerine nasıl hitap ettiğidir.

### Nasıl çalışır

```
1. Agent çalıştırır: $B snapshot -i
2. Sunucu Playwright'ın page.accessibility.snapshot() çağrısını yapar
3. Ayrıştırıcı ARIA ağacını gezer, sıralı ref'ler atar: @e1, @e2, @e3...
4. Her ref için bir Playwright Locator oluşturur: getByRole(role, { name }).nth(index)
5. BrowserManager örneğinde Map<string, RefEntry> saklar (role + name + Locator)
6. Açıklamalı ağacı düz metin olarak döndürür

Daha sonra:
7. Agent çalıştırır: $B click @e3
8. Sunucu @e3 → Locator → locator.click() olarak çözer
```

### Neden Locators, DOM mutasyonu değil

Bariz yaklaşım DOM'a `data-ref="@e1"` nitelikleri enjekte etmektir. Bu şu durumlarda bozulur:

- **CSP (İçerik Güvenliği Politikası).** Birçok üretim sitesi betiklerden DOM değişikliğini engeller.
- **React/Vue/Svelte hidrasyon..** Framework uzlaştırması enjekte edilen nitelikleri kaldırabilir.
- **Shadow DOM.** Dışarıdan shadow köklerin içine erişilemez.

Playwright Locators DOM'a dışarıdan bakar. Erişilebilirlik ağacını (Chromium'un dahili olarak tuttuğu) ve `getByRole()` sorgularını kullanır. DOM mutasyonu yok, CSP sorunu yok, framework çakışması yok.

### Ref yaşam döngüsü

Ref'ler gezinme sırasında temizlenir (ana çerçevede `framenavigated` olayı). Bu doğrudur — gezinmeden sonra tüm locator'lar eskimiş olur. Agent yeni ref'ler almak için `snapshot` komutunu tekrar çalıştırmalıdır. Bu tasarımdandır: eski ref'ler sessizce yanlış öğeye tıklamak yerine yüksek sesle başarısız olmalıdır.

### Ref eskime algılama

SPA'ler DOM'u `framenavigated`'i tetiklemeden mutasyona uğratabilir (örn. React router geçişleri, sekme değişimleri, modal açılışları). Bu, sayfa URL'si değişmemiş olsa bile ref'leri eskimiş hale getirir. Bunu yakalamak için `resolveRef()`, herhangi bir ref'i kullanmadan önce asenkron bir `count()` kontrolü gerçekleştirir:

```
resolveRef(@e3) → entry = refMap.get("e3")
                → count = await entry.locator.count()
                → eğer count === 0: throw "Ref @e3 is stale — element no longer exists. Run 'snapshot' to get fresh refs."
                → eğer count > 0: return { locator }
```

Bu, Playwright'ın 30 saniyelik eylem zaman aşımının eksik bir öğe üzerinde sona ermesini beklemek yerine hızlı başarısız olur (~5ms ek yük). `RefEntry`, Locator'ın yanında `role` ve `name` meta verilerini saklar, böylece hata mesajı agent'a öğenin ne olduğunu söyleyebilir.

### İmleç-etkileşimli ref'ler (@c)

`-C` bayrağı, tıklanabilir ancak ARIA ağacında olmayan öğeleri bulur — `cursor: pointer` ile stillendirilmişler, `onclick` niteliklerine sahipler veya özel `tabindex`'leri var. Bunlar ayrı bir ad alanında `@c1`, `@c2` ref'leri alır. Bu, framework'lerin aslında düğme olan `<div>` olarak render ettiği özel bileşenleri yakalar.

## Günlük mimarisi

Üç halka tampon (her biri 50.000 giriş, O(1) ekleme):

```
Tarayıcı olayları → CircularBuffer (bellek içi) → .gstack/*.log dosyasına asenk flush
```

Konsol mesajları, ağ istekleri ve iletişim kutusu olaylarının her birinin kendi tamponu vardır. Flush her 1 saniyede bir gerçekleşir — sunucu yalnızca son flushtan bu yana yeni girişleri ekler. Bu şu anlama gelir:

- HTTP istek işleme asla disk I/O tarafından engellenmez
- Günlükler sunucu çökmelerinden kurtulur (en fazla 1 saniyelik veri kaybı)
- Bellek sınırlıdır (50K giriş × 3 tampon)
- Disk dosyaları ekleme-yalnızdır, dış araçlar tarafından okunabilir

`console`, `network` ve `dialog` komutları diskten değil, bellek içi tamponlardan okur. Disk dosyaları otopsi hata ayıklama içindir.

## SKILL.md şablon sistemi

### Sorun

SKILL.md dosyaları Claude'a browse komutlarını nasıl kullanacağını söyler. Belgeler var olmayan bir bayrak listeliyorsa veya eklenen bir komutu kaçırıyorsa, agent hatalarla karşılaşır. El ile bakılan belgeler her zaman koddan ayrışır.

### Çözüm

```
SKILL.md.tmpl          (insan yazısı düzyazı + yer tutucular)
       ↓
gen-skill-docs.ts      (kaynak kodu meta verilerini okur)
       ↓
SKILL.md               (işlenmiş, otomatik oluşturulan bölümler)
```

Şablonlar insan kararlığı gerektiren iş akışlarını, ipuçlarını ve örnekleri içerir. Yer tutucular derleme zamanında kaynak koddan doldurulur:

| Yer tutucu | Kaynak | Ne üretir |
|-------------|--------|-------------------|
| `{{COMMAND_REFERENCE}}` | `commands.ts` | Kategorize edilmiş komut tablosu |
| `{{SNAPSHOT_FLAGS}}` | `snapshot.ts` | Örneklerle bayrak referansı |
| `{{PREAMBLE}}` | `gen-skill-docs.ts` | Başlatma bloğu: güncelleme kontrolü, oturum izleme, katılımcı modu, AskUserQuestion formatı |
| `{{BROWSE_SETUP}}` | `gen-skill-docs.ts` | Binary keşfi + kurulum talimatları |
| `{{BASE_BRANCH_DETECT}}` | `gen-skill-docs.ts` | PR hedefleme yetenekleri (ship, review, qa, plan-ceo-review) için dinamik temel dal algılama |
| `{{QA_METHODOLOGY}}` | `gen-skill-docs.ts` | /qa ve /qa-only için paylaşılan QA metodoloji bloğu |
| `{{DESIGN_METHODOLOGY}}` | `gen-skill-docs.ts` | /plan-design-review ve /design-review için paylaşılan tasarım denetim metodolojisi |
| `{{REVIEW_DASHBOARD}}` | `gen-skill-docs.ts` | /ship uçuş öncesi için İnceleme Hazırlık Paneli |
| `{{TEST_BOOTSTRAP}}` | `gen-skill-docs.ts` | /qa, /ship, /design-review için test framework algılama, önyükleme, CI/CD kurulumu |
| `{{CODEX_PLAN_REVIEW}}` | `gen-skill-docs.ts` | /plan-ceo-review ve /plan-eng-review için isteğe bağlı çapraz-model plan incelemesi (Codex veya Claude alt agent geri dönüşü) |
| `{{DESIGN_SETUP}}` | `resolvers/design.ts` | `$D` design binary'si için keşif modeli, `{{BROWSE_SETUP}}`'ı yansıtır |
| `{{DESIGN_SHOTGUN_LOOP}}` | `resolvers/design.ts` | /design-shotgun, /plan-design-review, /design-consultation için paylaşışılan karşılaştırma panosu geri bildirim döngüsü |
| `{{UX_PRINCIPLES}}` | `resolvers/design.ts` | /design-html, /design-shotgun, /design-review, /plan-design-review için kullanıcı davranışsal temeller (tarama, tatmin edicilik, iyi niyet rezervi, trunk testi) |
| `{{GBRAIN_CONTEXT_LOAD}}` | `resolvers/gbrain.ts` | Anahtar kelime çıkarma, sağlık farkındalığı ve veri-araştırma yönlendirmesi ile beyin öncelikli bağlam arama. 10 beyin-duyarlı yeteneğe enjekte edilir. Beyin olmayan ana bilgisayarlarda bastırılır. |
| `{{GBRAIN_SAVE_RESULTS}}` | `resolvers/gbrain.ts` | Varlık zenginleştirme, gazlama işleme ve yetenek başına kaydetme talimatları ile yetenek sonrası beyen kalıcılığı. 8 yeteneğe özel kaydetme formatı. |

Bu yapısal olarak sağlamdır — bir komut kodda mevcutsa, belgelerde görünür. Mevcut değilse, görünemez.

### Preamble

Her yetenek, kendi mantığından önce çalışan bir `{{PREAMBLE}}` bloğu ile başlar. Tek bir bash komutunda beş şeyi yönetir:

1. **Güncelleme kontrolü** — `gstack-update-check` çağırır, yükseltme olup olmadığını bildirir.
2. **Oturum izleme** — `~/.gstack/sessions/$PPID` dosyasına dokunur ve aktif oturumları sayar (son 2 saatte değiştirilen dosyalar). 3+ oturum çalışırken, tüm yetenekler "ELI16 moduna" girer — her soru kullanıcıyı bağlam üzerinde yeniden zemine oturtur çünkü pencereleri yönetmektedir.
3. **Operasyonel öz-gelişim** — her yetenek oturumunun sonunda, agent başarısızlıklar (CLI hataları, yanlış yaklaşımlar, proje tuhaflıkları) üzerine yansıtır ve gelecek oturumlar için operasyonel öğrenmeleri projenin JSONL dosyasına kaydeder.
4. **AskUserQuestion formatı** — evrensel format: bağlam, soru, `RECOMMENDATION: Choose X because ___`, harfli seçenekler. Tüm yeteneklerde tutarlı.
5. **Önce Ara, Sonra Oluştur** — altyapı veya alışılmadık desenler oluşturmadan önce, önce arayın. Bilginin üç katmanı: denenmiş-ve-doğru (Katman 1), yeni-ve-popüler (Katman 2), birinci-ilkeler (Katman 3). Birinci-ilkeler muhakemesi geleneksel bilgelik yanlış olduğunda, agent "öureka anını" adlandırır ve kaydeder. Tam oluşturcu felsefe için `ETHOS.md`'ye bakın.

### Neden çalışma zamanında değil, işlenmiş olarak kaydedilir?

Üç neden:

1. **Claude SKILL.md'yi yetenek yükleme zamanında okur.** Bir kullanıcı `/browse` çağırdığında derleme adımı yoktur. Dosya zaten mevcut ve doğru olmalıdır.
2. **CI tazelik doğrulayabilir.** `gen:skill-docs --dry-run` + `git diff --exit-code` birleştirme öncesi eski belgeleri yakalar.
3. **Git blame çalışır.** Bir komutun ne zaman eklendiğini ve hangi commit'te olduğunu görebilirsiniz.

### Şablon test katmanları

| Katman | Ne | Maliyet | Hız |
|------|------|------|-------|
| 1 — Statik doğrulama | SKILL.md'deki her `$B` komutunu ayrıştır, kayıt defterine karşı doğrula | Ücretsiz | <2s |
| 2 — `claude -p` ile E2E | Gerçek Claude oturumu başlat, her yeteneği çalıştır, hataları tara | ~$3.85 | ~20dk |
| 3 — LLM-as-judge | Sonnet belgeleri netlik/tamlık/eyleme-dönüştürülebilirlik puanlar | ~$0.15 | ~30s |

Katman 1 her `bun test` çalıştırmasında çalışır. Katman 2+3 `EVALS=1` ile sınırlandırılır. Fikir şu: sorunların %95'ini ücretsiz yakala, LLM'leri yalnızca karar çağrıları için kullan.

## Komut yönlendirme

Komutlar yan etkilere göre kategorize edilir:

- **READ** (text, html, links, console, cookies, ...): Mutasyon yok. Yeniden denemek güvenlidir. Sayfa durumunu döndürür.
- **WRITE** (goto, click, fill, press, ...): Sayfa durumunu mutasyona uğratır. Idempotent değildir.
- **META** (snapshot, screenshot, tabs, chain, ...): Okuma/yazmaya neatly uymayan sunucu düzeyinde işlemler.

Bu yalnızca organizasyonel değil. Sunucu bunu yönlendirme için kullanır:

```typescript
if (READ_COMMANDS.has(cmd))  → handleReadCommand(cmd, args, bm)
if (WRITE_COMMANDS.has(cmd)) → handleWriteCommand(cmd, args, bm)
if (META_COMMANDS.has(cmd))  → handleMetaCommand(cmd, args, bm, shutdown)
```

`help` komutu üç kümenin tamamını döndürür, böylece agent'lar kullanılabilir komutları kendileri keşfedebilir.

## Hata felsefesi

Hatalar insanlar için değil, yapay zeka agent'ları içindir. Her hata mesajı eyleme geçirilebilir olmalıdır:

- "Element not found" → "Element not found or not interactable. Run `snapshot -i` to see available elements."
- "Selector matched multiple elements" → "Selector matched multiple elements. Use @refs from `snapshot` instead."
- Timeout → "Navigation timed out after 30s. The page may be slow or the URL may be wrong."

Playwright'ın yerel hataları, iç yığın izlerini çıkarmak ve rehberlik eklemek için `wrapError()` üzerinden yeniden yazılır. Agent, insan müdahalesi olmadan hatayı okuyup ne yapması gerektiğini bilmelidir.

### Çökme kurtarma

Sunucu kendini iyileştirmeye çalışmaz. Chromium çökerse (`browser.on('disconnected')`), sunucu hemen çıkar. CLI sonraki komutta ölü sunucuyu algılar ve otomatik yeniden başlatır. Bu, yarı-ölü bir tarayıcı sürecine yeniden bağlanmaya çalışmaktan daha basit ve güvenilirdir.

## E2E test altyapısı

### Oturum çalıştırıcısı (`test/helpers/session-runner.ts`)

E2E testleri `claude -p`'yi tamamen bağımsız bir alt süreç olarak başlatır — Agent SDK ile değil, çünkü bu Claude Code oturumları içinde iç içe geçemez. Çalıştırıcı:

1. Prompt'u geçici bir dosyaya yazar (kabuk kaçış sorunlarından kaçınır)
2. `sh -c 'cat prompt | claude -p --output-format stream-json --verbose'` başlatır
3. Gerçek zamanlı ilerleme için stdout'tan NDJSON akıştırır
4. Yapılandırılabilir bir zaman aşımına karşı yarıştır
5. Tam NDJSON transkriptini yapılandırılmış sonuçlara ayrıştırır

`parseNDJSON()` fonksiyonu saf bir fonksiyondur — girdi/çıkış yok, yan etki yok — bu da onu bağımsız olarak test edilebilir kılar.

### Gözlemlenebilirlik veri akışı

```
  skill-e2e-*.test.ts
        │
        │ runId oluşturur, her çağrıya testName + runId geçirir
        │
  ┌─────┼──────────────────────────────┐
  │     │                              │
  │  runSkillTest()              evalCollector
  │  (session-runner.ts)         (eval-store.ts)
  │     │                              │
  │  araç çağrısı başına:        addTest() başına:
  │  ┌──┼──────────┐              savePartial()
  │  │  │          │                   │
  │  ▼  ▼          ▼                   ▼
  │ [HB] [PL]    [NJ]          _partial-e2e.json
  │  │    │        │             (atomik üzerine yazma)
  │  │    │        │
  │  ▼    ▼        ▼
  │ e2e-  prog-  {name}
  │ live  ress   .ndjson
  │ .json .log
  │
  │  başarısızlık durumunda:
  │  {name}-failure.json
  │
  │  TÜM dosyalar ~/.gstack-dev/ içinde
  │  Çalıştırma dizini: e2e-runs/{runId}/
  │
  │         eval-watch.ts
  │              │
  │        ┌─────┴─────┐
  │     HB oku     partial oku
  │        └─────┬─────┘
  │              ▼
  │        dashboard render
  │        (10dk+ eski? uyarı)
```

**Bölünmüş sahiplik:** session-runner kalp atışına (mevcut test durumu), eval-store kısmi sonuçlara (tamamlanmış test durumu) sahiptir. İzleyici her ikisini okur. Bileşenlerden hiçbiri diğerini bilmez — veriyi yalnızca dosya sistemi üzerinden paylaşır.

**Her şey ölümcül değil:** Tüm gözlemlenebilirlik girdi/çıkışı try/catch ile sarılır. Bir yazma hatası asla bir testin başarısız olmasına neden olmaz. Testlerin kendisi gerçeklik kaynağıdır; gözlemlenebilirlik en-iyi-çabadır.

**Makine tarafından okunabilir tanılama:** Her test sonucu `exit_reason` (success, timeout, error_max_turns, error_api, exit_code_N), `timeout_at_turn` ve `last_tool_call` içerir. Bu, şu gibi `jq` sorgularını mümkün kılar:
```bash
jq '.tests[] | select(.exit_reason == "timeout") | .last_tool_call' ~/.gstack-dev/evals/_partial-e2e.json
```

### Değerlendirme kalıcılığı (`test/helpers/eval-store.ts`)

`EvalCollector` test sonuçlarını biriktirir ve iki şekilde yazar:

1. **Artımlı:** `savePartial()` her testten sonra `_partial-e2e.json` yazar (atomik: `.tmp` yaz, `fs.renameSync`). Öldürmelerden kurtulur.
2. **Final:** `finalize()` zaman damgalı bir değerlendirme dosyası yazar (örn. `e2e-20260314-143022.json`). Kısmi dosya asla temizlenmez — gözlemlenebilirlik için final dosyasının yanında kalır.

`eval:compare` iki değerlendirme çalışmasını karşılaştırır. `eval:summary` `~/.gstack-dev/evals/` içindeki tüm çalışmalardaki istatistikleri toplar.

### Test katmanları

| Katman | Ne | Maliyet | Hız |
|------|------|------|-------|
| 1 — Statik doğrulama | `$B` komutlarını ayrıştır, kayıt defterine karşı doğrula, gözlemlenebilirlik birim testleri | Ücretsiz | <5s |
| 2 — `claude -p` ile E2E | Gerçek Claude oturumu başlat, her yeteneği çalıştır, hataları tara | ~$3.85 | ~20dk |
| 3 — LLM-as-judge | Sonnet belgeleri netlik/tamlık/eyleme-dönüştürülebilirlik puanlar | ~$0.15 | ~30s |

Katman 1 her `bun test` çalıştırmasında çalışır. Katman 2+3 `EVALS=1` ile sınırlandırılır. Fikir: sorunların %95'ini ücretsiz yakala, LLM'leri yalnızca karar çağrıları ve entegrasyon testi için kullan.

## Kasıtlı olarak burada olmayanlar

- **WebSocket akışı yok.** HTTP istek/yanıt daha basit, curl ile hata ayıklanabilir ve yeterince hızlıdır. Akış, marjinal bir fayda için karmaşıklık ekler.
- **MCP protokolü yok.** MCP istek başına JSON şema yükü ekler ve kalıcı bir bağlantı gerektirir. Düz HTTP + düz metin çıktısı token'lar üzerinde daha hafif ve hata ayıklamak daha kolaydır.
- **Çok kullanıcılı destek yok.** Çalışma alanı başına bir sunucu, bir kullanıcı. Token yetkilendirmesi derinlemesine savunmadır, çoklu-kiracılık değil.
- **Windows/Linux çerez şifre çözme yok.** macOS Anahtarlık desteklenen tek kimlik bilgisi deposudur. Linux (GNOME Keyring/kwallet) ve Windows (DPAPI) mimari olarak mümkündür ancak uygulanmamıştır.
- **Iframe otomatik keşfi yok.** `$B frame` çerçeveler arası etkileşimi destekler (CSS seçici, @ref, `--name`, `--url` eşleştirmesi), ancak ref sistemi `snapshot` sırasında iframe'leri otomatik taramaz. Önce açıkça bir çerçeve bağlamına girmelisiniz.