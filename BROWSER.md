# Browser — Tam Refer

gstack'in tarayıcı yüzeyi tek bir dokümanda. Headless Chromium daemon'ı, ~70+
komut, referans tabanlı öğe seçimi, kodlanabilir tarayıcı becerileri, gerçek
tarayıcı modu (Chrome yan paneli ile), kenar çubuğunda Claude PTY, ngrok
eşli-ajan akışı ve katmanlı komut enjeksiyonu savunması — hepsi düz metin
çıktısı veren derlenmiş bir CLI'nin arkasında. Çağrı başı ~100-200ms. Sıfır
bağlam-token ek yükü.

Son bir veya iki sürümde gstack kullandıysanız, verimlilik döngüsü yeni başlık:
`/scrape <niyet>` sayfayı bir kez tarar, `/skillify` bu akışı deterministik
bir Playwright betiğine dönüştürür ve aynı niyetle tekrar `/scrape` çağırmak
~30 saniyelik ajan yeniden keşfi yerine ~200ms'de çalışır.

---

## Hızlı başlangıç

```bash
# Bir kerelik: ikili dosyayı derle (browse/dist/browse, ~58MB)
bun install && bun run build

# $B değişkenini bir kez ayarla, unut
B=./browse/dist/browse           # veya ~/.claude/skills/gstack/browse/dist/browse

# Bir sayfayı tara
$B goto https://news.ycombinator.com
$B snapshot -i                   # Daha sonra tıklayabileceğiniz/doldurabileceğiniz @e referansları
$B click @e30                    # Anlık görüntüdeki 30 numaralı referansa tıkla
$B text                          # Temiz sayfa metnini al
$B screenshot /tmp/hn.png

# Yinelenen bir akışı kodla
/scrape latest hacker news stories
/skillify                        # ~/.gstack/browser-skills/hn-front/... dizinine yazar
/scrape hacker news front page   # İkinci çağrı: kodlanmış beceri üzerinden 200ms
```

```bash
# Claude'un gerçek zamanlı çalışmasını izle
$B connect                       # headed Chromium + Side Panel uzantısı
```

---

## İçindekiler

1. [Nedir](#nedir)
2. [Verimlilik döngüsü — `/scrape` + `/skillify`](#verimlilik-dongusu)
3. [Mimari](#mimari)
4. [Komut referansı](#komut-referansi)
5. [Anlık görüntü sistemi + referans tabanlı seçim](#anlik-goruntu-sistemi)
6. [Tarayıcı-becerileri çalışma zamanı](#tarayici-becerileri-calisma-zamani)
7. [Alan-becerileri (site bazlı ajan notları)](#alan-becerileri)
8. [Gerçek tarayıcı modu (`$B connect`)](#gercek-tarayici-modu) — [`--headed` + `--proxy` + `--navigate` (v1.28.0.0) dahil](#headed-modu--proxy--tarayici-yerel-indirmeler-v12800)
9. [Yan Panel + kenar çubuğu ajanı](#yan-panel--kenar-cubugu-ajani)
10. [Eşli-ajan — ngrok tüneli üzerinden uzak ajanlar](#esli-ajan)
11. [Kimlik doğrulama + belirteçler](#kimlik-dogrulama)
12. [Komut enjeksiyonu güvenlik yığını (L1–L6)](#guvenlik-yigini)
13. [Ekran görüntüleri, PDF'ler, görsel inceleme](#ekran-goruntuleri-pdf-gorsel)
14. [Yerel HTML — `goto file://` vs `load-html`](#yerel-html)
15. [Toplu iş uç noktası](#toplu-is-uc-noktasi)
16. [Konsol, ağ, iletişim kutusu yakalama](#yakalama)
17. [JS yürütme — `js` + `eval`](#js-yurutme)
18. [Sekmeler, çerçeveler, durum, izleme, gelen kutusu](#sekmeler-cerceveler-durum)
19. [CDP acil çıkış + CSS denetçisi](#cdp)
20. [Performans + ölçek](#performans)
21. [Çoklu-çalışma alanı izolasyonu](#coklu-calisma-alani)
22. [Ortam değişkenleri](#ortam-degiskenleri)
23. [Kaynak haritası](#kaynak-haritasi)
24. [Geliştirme + test](#gelistirme)
25. [Çapraz referanslar](#capraz-referanslar)
26. [Teşekkürler](#tesekkurler)

---

## Nedir

Derlenmiş bir CLI ikili dosyası, HTTP üzerinden kalıcı bir yerel Chromium
daemon'ı ile iletişim kurar. CLI ince bir istemcidir — bir durum dosyası okur,
bir komut gönderir, yanıtı stdout'a yazdırır. Daemon asıl işi
[Playwright](https://playwright.dev/) üzerinden yapar.

Erken dönemlerde bir Chrome MCP sunucusu olan her şey artık düz stdout üzerinden
gerçekleşiyor. JSON-schema çerçeveleme yok, protokol müzakeresi yok, kalıcı
WebSocket yok — Claude'un Bash aracı zaten var, o yüzden onu kullanıyoruz.

Üç artan mod:

- **Headless** (varsayılan). Daemon Chromium'u görünür bir pencere olmadan
  çalıştırır. En hızlı, en ucuz; `/qa`, `/design-review`, `/benchmark` gibi
  beceriler varsayılan olarak bunu kullanır.
- **`$B connect` ile Headed**. Aynı daemon, ancak Chromium görünür (GStack
  Browser olarak yeniden markalanmış) ve Side Panel uzantısı otomatik yüklenmiş.
  Her komutu gerçek zamanlı olarak işlerken izlersiniz.
- **Tünel üzerinden eşli-ajan**. Daemon, ngrok'un yönlendirdiği ikinci bir
  dinleyici bağlar. Uzak bir ajan (Codex, OpenClaw, Hermes, HTTP konuşabilen
  herhangi bir şey) kapsamlı, tek kullanımlık bir belirteç ile 26 komutluk bir
  izin listesi üzerinden yerel tarayıcınızı kontrol eder.

---

## Verimlilik döngüsü

v1.19.0.0'ın sunulan başlığı. İki gstack becerisi, tarayıcı-becerileri çalışma
zamanını sarmalar, böylece Claude'a ikinci kez bir sayfayı kazımasını
istediğinizde ~200ms'de çalışır.

### `/scrape <niyet>`

Sayfa verisi çekmek için tek giriş noktası. Kaput altında üç yol:

1. **Eşleşme yolu (~200ms)** — ajan `$B skill list` çalıştırır, niyeti her
   becerinin `triggers:` dizisi + `description` + `host` ile anlamsal olarak
   eşleştirir ve güvenilir bir eşleşme varsa `$B skill run <name>` çalıştırır.
2. **Prototip yolu (~30s)** — eşleşme yok, ajan sayfayı `$B goto`, `$B text`,
   `$B html`, `$B links` vb. ile tarar, JSON'u döndürür ve "`/skillify` de"
   tek satırlık bir öneri ekler.
3. **Değiştirici-niyet reddi** — *submit*, *click*, *fill* gibi fiiller
   `/automate`'e yönlendirilir (Aşama 2b, `TODOS.md`'de P0). `/scrape` sözleşme
   gereği salt okunurdur.

### `/skillify`

En başarılı `/scrape` prototipini kalıcı bir tarayıcı-becerisi olarak diske
kodla. On bir adım, üç kilitli sözleşme:

- **D1 — Kaynak koruması.** Açıkça sınırlanmış bir `/scrape` sonucu için ≤10
  ajan turuna kadar geriye doğru yürür. Soğuktan başlatılıyorsa belirli bir
  mesajla reddeder. Sohbet parçalarından sessiz sentez yok.
- **D2 — Sentez giriş dilimi.** YALNIZCA kullanıcının kabul ettiği JSON'u
  üreten son denemenin `$B` çağrılarını ve kullanıcının niyet dizesini çıkarır.
  Başarısız seçicileri atlar, sohbeti atlar, önceki oturum içeriğini atlar.
- **D3 — Atomik yazma.** Her şeyi
  `~/.gstack/.tmp/skillify-<spawnId>/` dizinine hazırlar, geçici dizine karşı
  `$B skill test` çalıştırır ve yalnızca test geçtiğinde + kullanıcı onayında
  son katman yoluna yeniden adlandırır. Test başarısız veya reddetme:
  geçici dizinin tamamını `rm -rf`. Yarı yazılmış hiçbir beceri hiçbir zaman
  `$B skill list`'te görünmez.

Değiştirici-akış kardeşi `/automate`, `TODOS.md`'de P0 olarak ayrılmıştır ve
bir sonraki dalda gönderilir — aynı skillify mekanizması, kodlanmamış
çalıştırılırken adım başına onay kapısı.

Tam tasarım + karar izi için
[`docs/designs/BROWSER_SKILLS_V1.md`](docs/designs/BROWSER_SKILLS_V1.md)
dosyasına bakın.

---

## Mimari

```
┌─────────────────────────────────────────────────────────────────┐
│  Claude Code                                                    │
│                                                                 │
│  $B goto https://staging.myapp.com                              │
│       │                                                         │
│       ▼                                                         │
│  ┌──────────┐    HTTP POST     ┌──────────────┐                 │
│  │ browse   │ ──────────────── │ Bun HTTP     │                 │
│  │ CLI      │  127.0.0.1:rand  │ daemon       │                 │
│  │          │  Bearer token    │              │                 │
│  │ compiled │ ◄──────────────  │  Playwright  │──── Chromium    │
│  │ binary   │  plain text      │  API calls   │    (headless    │
│  └──────────┘                  └──────────────┘     or headed)  │
│   ~1ms startup                  persistent daemon               │
│                                 auto-starts on first call       │
│                                 auto-stops after 30 min idle    │
└─────────────────────────────────────────────────────────────────┘
```

### Daemon yaşam döngüsü

1. **İlk çağrı.** CLI çalışan bir sunucu için `<project>/.gstack/browse.json`
   dosyasını kontrol eder. Bulamazsa — arka planda `bun run browse/src/server.ts`
   çalıştırır. Daemon Playwright üzerinden headless Chromium başlatır, rastgele
   bir port seçer (10000–60000), bir bearer token oluşturur, durum dosyasını
   yazar (chmod 600), istekleri kabul etmeye başlar. ~3 saniye.
2. **Sonraki çağrılar.** CLI durum dosyasını okur, bearer token ile bir HTTP POST
   gönderir, yanıtı yazdırır. ~100-200ms gidiş-dönüş.
3. **Boşta kapatma.** 30 dakika boyunca komut gelmezse daemon kapanır ve durum
   dosyasını temizler. Sonraki çağrı yeniden başlatır.
4. **Çökme kurtarma.** Chromium çökerse, daemon hemen çıkar — kendi kendini
   iyileştirme yok, hatayı gizleme. CLI bir sonraki çağrıda ölü daemon'u tespit
   eder ve yenisini başlatır.

### Çoklu-çalışma alanı izolasyonu

Her proje kökü (`git rev-parse --show-toplevel` ile tespit edilir) kendi
daemon'unu, portunu, durum dosyasını, çerezlerini ve günlüklerini alır.
Çalışma alanları arası çakışma yok. Durum `<project>/.gstack/browse.json`
konumunda.

| Çalışma alanı | Durum dosyası | Port |
|---------------|---------------|------|
| `/code/project-a` | `/code/project-a/.gstack/browse.json` | rastgele (10000–60000) |
| `/code/project-b` | `/code/project-b/.gstack/browse.json` | rastgele (10000–60000) |

---

## Komut referansı

Okuma, yazma ve meta olmak üzere ~70 komut. Seçiciler CSS, `snapshot`'tan `@e`
referansları veya `snapshot -C`'den `@c` referansları kabul eder. Tam tablo:

### Okuma

| Komut | Açıklama |
|-------|----------|
| `text [sel]` | Temiz sayfa metni (veya bir seçiciye kapsamlı) |
| `html [sel]` | innerHTML, veya seçici yoksa tam sayfa HTML'si |
| `links` | Tüm bağlantılar `metin → href` olarak |
| `forms` | Form alanları JSON olarak |
| `accessibility` | Tam ARIA ağacı |
| `media [--images\|--videos\|--audio] [sel]` | URL'ler, boyutlar, türler ile medya öğeleri |
| `data [--jsonld\|--og\|--meta\|--twitter]` | Yapılandırılmış veri: JSON-LD, OG, Twitter Cards, meta etiketleri |

### İnceleme

| Komut | Açıklama |
|-------|----------|
| `js <ifade>` | Sayfa bağlamında satır içi JavaScript ifadesi çalıştır, dize olarak döndür |
| `eval <dosya>` | Bir dosyadan JS çalıştır (/tmp veya cwd altındaki yol; `js` ile aynı sanal alan) |
| `css <sel> <özellik>` | Hesaplanmış CSS değeri |
| `attrs <sel\|@ref>` | Öğe öznitelikleri JSON olarak |
| `is <özellik> <sel\|@ref>` | Durum kontrolü: visible, hidden, enabled, disabled, checked, editable, focused |
| `console [--clear\|--errors]` | Yakalanan konsol mesajları |
| `network [--clear]` | Yakalanan ağ istekleri |
| `dialog [--clear]` | Yakalanan iletişim kutusu mesajları |
| `cookies` | Tüm çerezler JSON olarak |
| `storage` / `storage set <anahtar> <değer>` | localStorage + sessionStorage'ı oku; localStorage'a yaz |
| `perf` | Sayfa yükleme zamanlamaları |
| `inspect [sel] [--all] [--history]` | CDP üzerinden derin CSS — tam kural basamaklaması, kutu modeli, hesaplanmış stiller |
| `ux-audit` | Davranışsal analiz için sayfa yapısı: site kimliği, navigasyon, başlıklar, metin blokları, etkileşimli öğeler |
| `cdp <Domain.method> [json-params]` | Ham CDP metod gönderimi (reddet-varsayılan; `cdp-allowlist.ts`'te izin listesi) |

### Navigasyon

| Komut | Açıklama |
|-------|----------|
| `goto <url>` | URL'ye git (`http://`, `https://`, `file://`) |
| `load-html <dosya>` | Yerel HTML'yi bellekte yükle (`file://` URL yok; görüntü alanı ölçek değişikliklerinden etkilenmez) |
| `back`, `forward`, `reload` | Standart navigasyon |
| `url` | Geçerli sayfa URL'si |
| `wait <sel\|--networkidle\|--load>` | Öğe, ağ boşta veya sayfa yüklemesi bekle (15s zaman aşımı) |

### Etkileşim

| Komut | Açıklama |
|-------|----------|
| `click <sel\|@ref>` | Öğeye tıkla |
| `fill <sel> <değer>` | Girdiyi doldur |
| `select <sel> <değer>` | Açılır menü seçeneği seç (değer, etiket veya görünür metin) |
| `hover <sel>` | Öğenin üzerine gel |
| `type <metin>` | Odaklanmış öğeye yaz |
| `press <tuş>` | Playwright klavye tuşu (büyük/küçük harf duyarlı: Enter, Tab, ArrowUp, Shift+Enter, Control+A, ...) |
| `scroll [sel\|@ref]` | Öğeyi görünüme kaydır, veya seçici yoksa sayfa altına atla |
| `viewport [<GxY>] [--scale <n>]` | Görüntü alanı boyutu + isteğe bağlı `deviceScaleFactor` 1-3 (retina ekran görüntüleri) |
| `upload <sel> <dosya> [...]` | Dosya(lar) yükle |
| `dialog-accept [metin]` | Sonraki alert/confirm/prompt'u otomatik kabul et; prompt'lar için metin gönderilir |
| `dialog-dismiss` | Sonraki iletişim kutusunu otomatik reddet |

### Stil + temizlik

| Komut | Açıklama |
|-------|----------|
| `style <sel> <özellik> <değer>` | CSS özelliğini değiştir (geri alma desteği ile) |
| `style --undo [N]` | Son N stil değişikliğini geri al |
| `cleanup [--ads\|--cookies\|--sticky\|--social\|--all]` | Sayfa kalabalığını kaldır |

| Komut | Açıklama |
|-------|----------|
| `prettyscreenshot [--scroll-to <sel\|metin>] [--cleanup] [--hide <sel>...] [yol]` | İsteğe bağlı temizlik, kaydırma, gizleme ile temiz ekran görüntüsü |

### Görsel

| Komut | Açıklama |
|-------|----------|
| `screenshot [--selector <css>] [--viewport] [--clip x,y,w,h] [--base64] [sel\|@ref] [yol]` | Beş mod: tam sayfa, görüntü alanı, öğe kırpma, bölge kırpma, base64 |
| `pdf [yol] [--format letter\|a4\|legal] [...]` | Tam düzenli PDF: format, genişlik/yükseklik, kenar boşlukları, üst/alt şablonlar, sayfa numaraları, erişilebilirlik için --tagged, --toc Paged.js bekler |
| `responsive [önek]` | Üç ekran görüntüsü: mobil (375x812), tablet (768x1024), masaüstü (1280x720) |
| `diff <url1> <url2>` | İki URL arasındaki metin farkı |

### Çerezler + başlıklar

| Komut | Açıklama |
|-------|----------|
| `cookie <ad>=<değer>` | Geçerli sayfa alanında çerez ayarla |
| `cookie-import <json>` | JSON dosyasından çerezleri içe aktar |
| `cookie-import-browser [tarayıcı] [--domain d]` | Kurulu Chromium tarayıcılardan içe aktar (etkileşimli seçici veya doğrudan içe aktarma için `--domain`) |
| `header <ad>:<değer>` | Özel istek başlığı ayarla (hassas değerler otomatik sansürlenir) |
| `useragent <dize>` | Kullanıcı aracısı ayarla (bağlamı yeniden oluşturur, referansları geçersiz kılar) |

### Sekmeler + çerçeveler

| Komut | Açıklama |
|-------|----------|
| `tabs` | Açık sekmeleri listele |
| `tab <id>` | Sekmeye geç |
| `newtab [url] [--json]` | Yeni sekme aç; `--json` programlı kullanım için `{tabId, url}` döndürür |
| `closetab [id]` | Sekmeyi kapat |
| `tab-each <komut> [args...]` | Her açık sekmede bir komutu dağıt; JSON döndürür |
| `frame <sel\|@ref\|--name n\|--url pattern\|main>` | iframe bağlamına geç (veya ana çerçeveye dön); referansları temizler |

### Çıkarım

| Komut | Açıklama |
|-------|----------|
| `download <url\|@ref> [yol] [--base64]` | URL veya medya öğesini tarayıcı çerezlerini kullanarak indir |
| `scrape <images\|videos\|media> [--selector] [--dir] [--limit]` | Sayfadaki tüm medyayı toplu indir; `manifest.json` yazar |
| `archive [yol]` | Tam sayfayı CDP üzerinden MHTML olarak kaydet |

### Anlık görüntü

| Komut | Açıklama |
|-------|----------|
| `snapshot [-i] [-c] [-d N] [-s sel] [-D] [-a] [-o yol] [-C]` | `@e` referanslı erişilebilirlik ağacı; `-i` yalnızca etkileşimli, `-c` kompakt, `-d N` derinlik, `-s` kapsam, `-D` öncekine göre fark, `-a` açıklamalı ekran görüntüsü, `-C` imleç-etkileşimli `@c` referansları |

### Sunucu yaşam döngüsü

| Komut | Açıklama |
|-------|----------|
| `status` | Daemon durumu + mod (headless / headed / cdp) |
| `stop` | Daemon'u kapat |
| `restart` | Daemon'u yeniden başlat |
| `connect` | Side Panel uzantılı headed GStack Browser başlat |
| `disconnect` | headed Chrome'u kapat, headless moda dön |
| `focus [@ref]` | headed Chrome'u ön plana getir (macOS); `@ref` ayrıca görünüme kaydırır |
| `state save\|load <ad>` | Tarayıcı durumunu kaydet veya yükle (çerezler + URL'ler) |

### Devretme

| Komut | Açıklama |
|-------|----------|
| `handoff [neden]` | Kullanıcının devralması için geçerli sayfada görünür Chrome aç (CAPTCHA, MFA, karmaşık kimlik doğrulama) |
| `resume` | Kullanıcı devralmasından sonra yeniden anlık görüntü al, kontrolü AI'ya geri ver |

### Meta + zincirler

| Komut | Açıklama |
|-------|----------|
| `chain` (stdin üzerinden JSON) | Bir komut dizisi çalıştır. `[["cmd","arg1",...],...]`'ı `$B chain`'e borula. İlk hatada durur. |
| `inbox [--clear]` | Kenar çubuğu izci gelen kutusundaki mesajları listele |
| `watch [stop]` | Pasif gözlem — kullanıcı gezinirken periyodik anlık görüntüler; `stop` özet döndürür |

### Tarayıcı-becerileri çalışma zamanı

| Komut | Açıklama |
|-------|----------|
| `skill list` | Çözümlenmiş katmanlı tüm tarayıcı-becerilerini listele (proje > global > paketlenmiş) |
| `skill show <ad>` | SKILL.md'yi yazdır |
| `skill run <ad> [--arg k=v...] [--timeout=Ns]` | Beceri betiğini çağrı başına kapsamlı belirteçle çalıştır |
| `skill test <ad>` | Becerinin `script.test.ts`'ini paketlenmiş fixture'lara karşı çalıştır |
| `skill rm <ad> [--global]` | Kullanıcı katmanındaki bir beceriyi mezar taşıyla işaretle |

### Alan-becerileri

| Komut | Açıklama |
|-------|----------|
| `domain-skill save\|list\|show\|edit\|promote-to-global\|rollback\|rm <host?>` | Site bazlı ajan notları (host aktif sekmeden türetilir). Yaşam döngüsü: karantina → aktif (N=3 başarılı kullanım sonra sınıflandırıcı bayrağı olmadan) → global (açık tanıtım) |

Takma adlar: `setcontent`, `set-content`, `setContent` → `load-html` (kapsam
kontrollerinden önce standartlaştırılır, böylece salt okunur kapsamlı bir belirteç
takma adı kullanarak bir yazma komutu çalıştıramaz).

---

## Anlık görüntü sistemi

Tarayıcının temel yeniliği, Playwright'ın erişilebilirlik ağacı API'si üzerine
kurulu **referans tabanlı öğe seçimi**. DOM değişikliği yok. Enjekte edilmiş
betik yok. Sadece Playwright'ın yerel AX API'si.

### `@ref` nasıl çalışır

1. `page.locator(scope).ariaSnapshot()` YAML benzeri bir erişilebilirlik ağacı döndürür.
2. Anlık görüntü ayrıştırıcısı her öğeye referanslar atar (`@e1`, `@e2`, ...).
3. Her referans için bir Playwright `Locator` oluşturur (`getByRole` + nth-child kullanarak).
4. Referans→Locator eşlemesi `BrowserManager` üzerinde saklanır.
5. `click @e3` gibi sonraki komutlar Locator'ı arar ve `locator.click()` çağırır.

### Referans eskime tespiti

SPA'lar navigasyon olmadan DOM'u değiştirebilir (React router, sekme
geçişleri, modallar). Bu olduğunda, önceki bir `snapshot`'tan toplanan
referanslar artık var olmayan öğelere işaret edebilir. `resolveRef()` herhangi
bir referansı kullanmadan önce asenkron bir `count()` kontrolü çalıştırır —
öğe sayısı 0 ise, ajana `snapshot`'ı yeniden çalıştırmasını söyleyen bir
mesajla hemen hata fırlatır. Playwright'ın 30 saniyelik eylem zaman aşımını
beklemek yerine hızlıca başarısız olur (~5ms).

### Genişletilmiş anlık görüntü özellikleri

- **`--diff` (`-D`).** Her anlık görüntüyü temel olarak saklar. Sonraki `-D`
  çağrısında, nelerin değiştiğini gösteren bir birleşik fark döndürür. Bir
  eylemin (tıklama, doldurma vb.) gerçekten çalıştığını doğrulamak için bunu
  kullanın.
- **`--annotate` (`-a`).** Her referansın sınırlayıcı kutusunda geçici katman
  div'leri enjekte eder, referans etiketleri görünür şekilde bir ekran görüntüsü
  alır, ardından katmanları kaldırır. Çıktıyı kontrol etmek için `-o <yol>`
  kullanın.
- **`--cursor-interactive` (`-C`).** ERIŞİLEBİLİRLİK ağacının kaçtırdığı
  `cursor:pointer`, `onclick`, `tabindex>=0` gibi ARIA dışı etkileşimli
  öğeleri `page.evaluate` ile tarar. Deterministik `nth-child` CSS seçicileri
  ile `@c1`, `@c2`... referansları atar. Bunlar ARIA ağacının kaçtırdığı ancak
  kullanıcıların hala tıklayabileceği öğelerdir.

---

## Tarayıcı-becerileri çalışma zamanı

Yinelenen bir tarayıcı akışını deterministik bir Playwright betiğine kodlayan
görev başı dizinler. Bileşik katman.

### Bir tarayıcı-becerisininin anatomisi

```
browser-skills/<ad>/
├── SKILL.md                        # frontmatter + düzyazı sözleşmesi
├── script.ts                       # deterministik Playwright-via-browse-client mantığı
├── _lib/browse-client.ts           # SDK'nin satır içi kopyası (~3KB, kanonik ile bayt-özdeş)
├── fixtures/<host>-<tarih>.html    # fixture-tekrar testleri için yakalanan sayfa
└── script.test.ts                  # fixture'a karşı ayrıştırıcı testleri (daemon gerekmez)
```

Paketlenmiş referans `browser-skills/hackernews-frontpage/`: HN ana sayfasını
kazır, 30 hikayeyi JSON olarak döndürür. Deneyin:

```bash
$B skill list                            # hackernews-frontpage (bundled) gösterir
$B skill show hackernews-frontpage
$B skill run hackernews-frontpage        # 30 hikayenin JSON'unu ~200ms'de
$B skill test hackernews-frontpage       # fixture'a karşı script.test.ts çalıştırır
```

### Üç katmanlı depolama

`$B skill list` öncelik sırasına göre üç katmanı da yürür; ilk eşleşme kazanır.
Çözümlenmiş katman her beceri adının yanında yazdırılır:

| Katman | Yol | Ne zaman |
|--------|-----|----------|
| **Proje** | `<project>/.gstack/browser-skills/<ad>/` | Proje özgü beceriler (commitlenmiş veya gitignore edilmiş) |
| **Global** | `~/.gstack/browser-skills/<ad>/` | Kullanıcı başı beceriler, tüm projeler |
| **Paketlenmiş** | `<gstack-install>/browser-skills/<ad>/` | gstack ile gelir, salt okunur |

### Güven modeli

İki dikey eksen — daemon tarafı yetenek ve süreç tarafı ortam — bağımsız
yapılandırılmış.

| Eksen | Mekanizma | Varsayılan |
|-------|-----------|------------|
| **Daemon tarafı yetenek** | Çağrı başına kapsamlı belirteç, okuma+yazma kapsamına bağlı (tarayıcı-yürütme komutları eksi yönetici: `eval`, `js`, `cookies`, `storage`). Tek kullanımlık clientId, beceri adı + çağrı id'sini kodlar. Çağrı çıkışında iptal edilir. | Her zaman kapsamlı — asla daemon kök belirteci değil |
| **Süreç tarafı ortam** | `trusted: true` frontmatter `process.env`'i `GSTACK_TOKEN` hariç geçirir. `trusted: false` (varsayılan) minimal bir izin listesi dışında her şeyi atar (LANG, LC_ALL, TERM, TZ) ve kalıp şeritli gizli dizileri atar (TOKEN/KEY/SECRET/PASSWORD, AWS_*, ANTHROPIC_*, OPENAI_*, GITHUB_*, vb.) | Güvenilmeyen (katılım gerekli) |

`GSTACK_PORT` ve `GSTACK_SKILL_TOKEN` en son enjekte edilir, bu nedenle bir üst
süreç onları geçersiz kılamaz.

### Çıktı protokolü

stdout = JSON. stderr = akış günlükleri. Çıkış 0 / sıfır olmayan. Varsayılan
60s zaman aşımı, `--timeout=Ns` ile geçersiz kılınabilir. Maksimum stdout 1MB
(aşıldığında kırpılır + sıfır olmayan çıkış). `gh` / `kubectl` / `docker`
gelenekleriyle eşleşir.

### SDK dağıtımı nasıl çalışır

Her beceri kendi `browse-client.ts` kopyasını `_lib/browse-client.ts`
konumunda gönderir, kanonik `browse/src/browse-client.ts` ile bayt-özdeş.
`/skillify` kanonik SDK'yi her oluşturulan betiğin yanına kopyalar. Her beceri
tamamen kendi içinde: dizini herhangi bir yere kopyalayın, çalışır. Sürüm
kayması imkansız — SDK becerinin yazıldığı sürümde dondurulmuştur.

### Atomik yazma disiplini (`/skillify` D3)

`browse/src/browser-skill-write.ts` üç temel sağlar:

- `stageSkill(opts)` — dosyaları kısıtlayıcı izinlerle
  `~/.gstack/.tmp/skillify-<spawnId>/<ad>/` dizinine yazar.
- `commitSkill(opts)` — son katman yoluna atomik `fs.renameSync`. Sembolik
  bağlantılı hazırlama dizinlerini takip etmeyi reddeder (`lstat` kontrolü),
  mevcut becerilerin üzerine yazmayı reddeder, katman kökünde `realpath`
  disiplini çalıştırır.
- `discardStaged(stagedDir)` — hazırlama dizinini + çağrı başı sarmalayıcıyı
  `rm -rf` ile siler. Idempotent. Test başarısızlığı veya onay reddi üzerinde
  çağrılır.

"Neredeyse gönderildi" durumu yoktur. Testler geçer + kullanıcı onaylar = atomik
yeniden adlandırma. Testler başarısız veya kullanıcı reddeder = hazırlama
kaybolur.

Tam tasarım gerekçesi için
[`docs/designs/BROWSER_SKILLS_V1.md`](docs/designs/BROWSER_SKILLS_V1.md)
dosyasına bakın.

---

## Alan-becerileri

Tarayıcı-becerilerinden farklı zihinsel model: ajan tarafından yazılmış bir
site hakkındaki *notlar* (deterministik betikler değil). Her ana bilgisayar adı
için bir tane. Yaşam döngüsü:

1. `domain-skill save <host>` — ajan site hakkında bir not yazar (ör.,
   "GitHub: PR oluşturma personel olmayanlar için `--draft` bayrağı gerektirir",
   "X.com: timeline imleç sayfalaması kullanır, sayfa numaraları değil").
   Varsayılan durum: **karantina**.
2. **N=3** başarılı kullanımdan sonra L4 komut enjeksiyonu sınıflandırıcısı
   notu işaretlemeden otomatik olarak **aktif**e yükselir.
3. `domain-skill promote-to-global <host>` bunu global katmana kaldırır
   (makine geneli, tüm projeler).
4. `domain-skill rollback <host>` düşürür; `domain-skill rm <host>` mezar
   taşıyla işaretler.

Sınıflandırıcı bayrağı L4 komut enjeksiyonu taraması tarafından otomatik olarak
ayarlanır; ajanlar manuel olarak ayarlamaz.

Depolama:
- Proje başı: `<project>/.gstack/domain-skills/<host>.md`
- Global: `~/.gstack/domain-skills/<host>.md`

Kaynak: `browse/src/domain-skills.ts`, `domain-skill-commands.ts`.

---

## Gerçek tarayıcı modu

`$B connect` **GStack Browser**'ı başlatır — Playwright tarafından kontrol
edilen, Side Panel uzantısı otomatik yüklenmiş ve bot-karşıtı gizlilik
yamaları uygulanmış, yeniden markalanmış bir Chromium. Her komutu gerçek
zamanlı olarak görünür bir pencerede işlerken izlersiniz.

```bash
$B connect              # GStack Browser'ı başlatır, headed
$B goto https://app.com # görünür pencerede gezinir
$B snapshot -i          # gerçek sayfadan referanslar
$B click @e3            # gerçek pencerede tıklar
$B focus                # pencereyi ön plana getir (macOS)
$B status               # Mode: cdp gösterir
$B disconnect           # headless moda geri dön
```

Pencerede üstte ince altın bir parıltı çizgisi ve sağ altta yüzen bir "gstack"
tableti bulunur, böylece hangi Chrome penceresinin kontrol edildiğini her zaman
bilirsiniz.

### "GStack Browser" ne anlama gelir

Günlük Chrome'unuz değil — Dock ve menü çubuğunda özel markalama, bot-karşıtı
gizlilik (Google ve NYTimes gibi siteler captcha olmadan çalışır), özel bir
kullanıcı aracısı ve `launchPersistentContext` ile önceden yüklenmiş gstack
uzantısı olan Playwright yönetimli bir Chromium. Sekmeleriniz ve yer
imlerinizle olan normal Chrome'unuz dokunulmadan kalır.

### Headed mod ne zaman kullanılır

- Claude'un uygulamanızda tıklayışını izlemek istediğiniz **QA testi**
- Claude'un tam olarak ne gördüğünü görmeniz gereken **Tasarım incelemesi**
- Headless davranışının gerçek Chrome'dan farklı olduğu **Hata ayıklama**
- Ekranınızı paylaştığınız **Demolar**
- **Eşli-ajan** oturumları (uzak ajan yerel tarayıcınızı kontrol eder)

### CDP duyarlı beceriler

Gerçek tarayıcı modundayken, `/qa` ve `/design-review` otomatik olarak çerez
içe aktarma istemlerini ve headless geçici çözümlerini atlar — headed tarayıcıda
zaten oturum açtığınız herhangi bir oturum vardır.

### Headed modu + proxy + tarayıcı-yerel indirmeler (v1.28.0.0)

Headless tarayıcıları engelleyen, Playwright varsayılanlarını parmak izi
taranan veya kimlik doğrulamalı yukarı akış proxy'lerinin arkasındaki siteler
için üç koordineli bayrak:

```bash
# Görünür Chromium. DISPLAY olmadan Linux konteynerlerinde otomatik Xvfb başlatır.
$B --headed goto https://example.com

# SOCKS5 ile kimlik doğrulama — Chromium SOCKS5 kimlik bilgilerini isteyemediği için
# $B kimlik doğrulama el sıkışmasını işleyen yerel bir 127.0.0.1 köprüsü çalıştırır.
$B --proxy socks5://user:pass@residential.proxy.host:1080 goto https://example.com

# HTTP/HTTPS proxy doğrudan Chromium'a geçirilir.
$B --proxy http://corp-proxy:3128 goto https://example.com

# Content-Disposition, yönlendirme zincirleri, bot-karşıtı CDN'ler için tarayıcı-yerel
# indirme; page.request.fetch()'in başarısız olduğu yerlerde.
$B download "https://protected.example.com/file" /tmp/file.bin --navigate

# Birleşik.
$B --headed --proxy socks5://user:pass@host:1080 \
   download "https://protected.example.com/file" /tmp/file.bin --navigate
```

**Kimlik bilgisi politikası.** Kimlik bilgilerini URL (`socks5://user:pass@host`)
VEYA ortam değişkenleri `BROWSE_PROXY_USER` / `BROWSE_PROXY_PASS` ile iletin —
asla ikisi birden değil. `$B` her ikisi de ayarlandığında net bir ipucuyla reddeder;
sessiz geçersiz kılma "benim makinamda çalışıyor" hata ayıklama tuzakları
oluşturur.

**Daemon disiplini.** `--proxy` ve `--headed` daemon başlatma yapılandırmasıdır.
A yapılandırmasıyla çalışan bir daemon, B yapılandırmasıyla yeni bir çağrı ile
karşılaştığında sekme durumu, çerezler veya oturumları sessizce yeniden
başlatmak yerine `browse disconnect` ipucu ile çıkış kodu 1 döndürür.

**Gizlilik kapsamı.** `--headed` veya `--proxy` ayarlandığında, `$B` yalnızca
`navigator.webdriver`'ı maskeler — Chromium'un
`--disable-blink-features=AutomationControlled` ve küçük bir başlangıç betiği
aracılığıyla. `navigator.plugins`, `navigator.languages` veya `window.chrome`'u
sahteleştirmiyoruz — modern parmak izi yazılımları tutarlılık için bunları
kontrol eder ve sabit değerler sentezlemek DAHA FAZLA bot benzeri
işaretleyebilir, daha az değil. ChromeDriver'ın `cdc_` çalışma zamanı
yapıları ve Permissions API yaması hala temizlenir.

**Konteyner desteği.** `DISPLAY` olmadan Linux'ta `--headed`, `xdpyinfo` boş bir
yuva bildirene kadar görüntü aralığını (`:99`, `:100`, ...) tarar, sonra Xvfb
başlatır. Bağlantı kesme-temizliği, kaydedilen PID'nin `/proc/<pid>/cmdline`
değerinin `Xvfb` ile eşleştiğini VE başlangıç zamanının eşleştiğini herhangi bir
sinyal göndermeden önce doğrular — PID yeniden kullanım tuzakları yok.
`WAYLAND_DISPLAY` ayarlandığında başlatmayı tamamen atlar (Chromium doğal olarak
Wayland kullanır). Standart Debian/Ubuntu konteynerleri kutudan çalışır;
minimal görüntüler (alpine, distroless) headed Chromium'un render edilmesi için
font/dbus/gtk kitaplıklarına ihtiyaç duyabilir.

**Başarısızlık modları.** SOCKS5 yukarı akış reddedildi veya erişilemez — 3
yeniden deneme sonrası (5s bütçe) hızlı başarısızlık, kırmızı sansürlü hata.
Orta akış yukarı akış düşüşü — köprü yalnızca etkilenen istemci bağlantısını
öldürür; tarayıcı trafiğini bozabilecek taşıma yeniden denemeleri yok.

---

## Yan Panel + kenar çubuğu ajanı

GStack Browser'a entegre gelen Chrome uzantısı, bir Side Panel'de her tarama
komutunun canlı etkinlik akışını, sayfada `@ref` katmanlarını ve kenar
çubuğunun içinde etkileşimli bir Claude PTY gösterir.

### Terminal bölmesi (başlık)

Side Panel'in birincil yüzeyi **Terminal bölmesi** — kenar çubuğundan doğrudan
yazabileceğiniz canlı bir `claude -p` PTY. Etkinlik / Referanslar / Denetçi,
alt bilginin `debug` geçişi arkasındaki hata ayıklama katmanlarıdır. WebSocket
kimlik doğrulaması `Sec-WebSocket-Protocol` kullanır (tarayıcılar bir WebSocket
yükseltmesinde `Authorization` ayarlayamaz) ve PTY oturum belirteci,
`POST /pty-session` üzerinden oluşturulan 30 dakikalık bir HttpOnly çerezidir.

Araç çubuğunun Temizle düğmesi ve Denetçi'nin "Koda Gönder" eylemi, her ikisi
de `sidepanel-terminal.js` tarafından ortaya çıkarılan
`window.gstackInjectToTerminal(text)` aracılığıyla canlı Claude PTY'ye metin
borular. Ayrı bir `/sidebar-command` POST yok — canlı REPL tek yürütme
yüzeyidir.

### Etkinlik akışı

Her tarama komutunun kayan akışı — ad, argümanlar, süre, durum, hatalar.
Claude çalışırken gerçek zamanlı görünür. SSE (`/activity/stream`) tarafından
desteklenir ve Bearer belirtecini VEYA HttpOnly `gstack_sse` oturum çerezini
(`POST /sse-session` üzerinden oluşturulan 30 dakikalık akış kapsamlı çerez)
kabul eder.

### Referanslar sekmesi

`$B snapshot`'tan sonra, mevcut `@ref` listesini (rol + ad) gösterir, böylece
Claude'un ne hedeflediğini görebilirsiniz.

### CSS Denetçisi

`$B inspect` (CDP tabanlı) tarafından desteklenir. Tam CSS kural basamaklamasını,
hesaplanmış stilleri, kutu modeli ve değişiklik geçmişini görmek için sayfadaki
herhangi bir öğeye tıklayın. "Koda Gönder" düğmesi Claude PTY'ye bir açıklama
enjekte eder.

### Kenar çubuğu mimarisi

| Bileşen | Bulunduğu yer | Notlar |
|---------|---------------|--------|
| Side Panel UI | `extension/sidepanel.js`, `sidepanel-terminal.js` | Chrome uzantı yüzeyi |
| Background SW | `extension/background.js` | Sekme olaylarını, port yönetimini yönetir |
| Content script | `extension/content.js` | Sayfa katmanları, `gstack` tableti |
| Terminal ajanı | `browse/src/terminal-agent.ts` | PTY başlatma, yaşam döngüsü, kimlik doğrulama |
| Kenar çubuğu yardımcıları | `browse/src/sidebar-utils.ts` | URL sansürleme, yardımcılar |

Bunlardan herhangi birini değiştirmeden önce, `CLAUDE.md`'deki "Sidebar
architecture" altındaki yorum blokunu okuyun — sessiz hatalar genellikle
bileşenler arası akışı anlamamaktan kaynaklanır.

### Manuel kurulum (günlük Chrome'unuz için)

Uzantıyı günlük Chrome'unuzda istiyorsanız (Playwright kontrollü olmayan):

```bash
bin/gstack-extension    # chrome://extensions'ı açar, yolunu panoya kopyalar
```

Veya manuel olarak: `chrome://extensions` → Geliştirici modunu aç → Paketlenmemiş
yükle → `~/.claude/skills/gstack/extension` dizinine git → uzantıyı sabitle →
`$B status`'tan portu gir.

---

## Eşli-ajan

Uzak AI ajanları (Codex, OpenClaw, Hermes, HTTP konuşabilen herhangi bir şey)
ngrok tüneli üzerinden yerel tarayıcınızı kontrol edebilir. Tüm akış 26 komutluk
bir izin listesi, kapsamlı belirteçler ve bir reddetme günlüğü ile korunur.

### Nasıl çalışır

```bash
/pair-agent                     # bir kurulum anahtarı oluşturur, bağlantı talimatlarını yazdırır
# Talimatları uzak ajana kopyala
# Uzak ajan çalıştırır:
#   POST <tunnel-url>/connect kurulum anahtarı ile → kapsamlı bir belirteç alır (24s, tek istemci)
#   POST <tunnel-url>/command belirteç ile → izin verilen komutları çalıştırır
```

### Çift-dinleyici mimarisi (v1.6.0.0+)

`pair-agent` etkinleştirildiğinde, daemon **iki HTTP dinleyicisi** bağlar:

- **Yerel dinleyici** (`127.0.0.1:LOCAL_PORT`). Tam komut yüzeyi. Asla ngrok
  tarafından yönlendirilmez. Claude Code'unuz, Side Panel, makinenizdeki her
  şey tarafından kullanılır.
- **Tünel dinleyicisi** (`127.0.0.1:TUNNEL_PORT`). Kilitli izin listesi —
  `/connect`, `/command` (kapsamlı belirteçler + 26 komutluk tarayıcı-yürütme
  izin listesi), `/sidebar-chat`. ngrok yalnızca bu portu yönlendirir.

Tünel üzerinden gönderilen kök belirteçler 403 döndürür. SSE uç noktaları
30 dakikalık HttpOnly `gstack_sse` çerezi kullanır (`/command`'a karşı asla
geçerli değildir).

### 26 komutluk tünel izin listesi

`browse/src/server.ts`'te `TUNNEL_COMMANDS` olarak tanımlanmıştır. Saf geçiş
işlevi `canDispatchOverTunnel(command)` birim test için dışa aktarılmıştır. Küme:

```
goto, click, text, screenshot, html, links, forms, accessibility,
attrs, media, data, scroll, press, type, select, wait, eval,
newtab, tabs, back, forward, reload, snapshot, fill, url, closetab
```

Önemle eksik olanlar: `pair`, `unpair`, `cookies`, `setup`, `launch`, `restart`,
`stop`, `tunnel-start`, `token-mint`, `state`, `connect`, `disconnect`. Deneyen
uzak bir ajan 403 ve reddetme günlüğünde yeni bir giriş alır.

### Tünel reddetme günlüğü

`~/.gstack/security/attempts.jsonl` — yalnızca ekleme, kaynak + alan adının
tuzlanmış SHA-256'sı (ham IP yok, tam istek gövdesi yok), 10MB'da 5 nesille
döner. Cihaz başına tuz `~/.gstack/security/device-salt` (mod 0600).

Tam operatör kılavuzu için
[`docs/REMOTE_BROWSER_ACCESS.md`](docs/REMOTE_BROWSER_ACCESS.md) dosyasına bakın.

### Sekme sahipliği

Kapsamlı belirteçler varsayılan olarak `tabPolicy: 'own-only'`. Eşlenmiş bir
ajan kendi sekmesini oluşturmak için `newtab` yapabilir ve o sekmeyi serbestçe
kontrol edebilir, ancak başka bir çağırıcının sahip olduğu sekmelerde `goto`,
`fill` veya `click` yapamaz. `tabs` TÜM sekme meta verilerini listeler (kabul
edilmiş bir takas — ARCHITECTURE.md'ye bakın), ancak sahip olunmayan sekmelerin
`text`/`html`/`snapshot` içeriği sahiplik kontrolleri tarafından engellenir.

---

## Kimlik doğrulama

Üç belirteç türü, üç yaşam süresi, üç kapsam.

| Belirteç | Tarafından oluşturuldu | Yaşam süresi | Kapsam |
|----------|----------------------|--------------|--------|
| **Kök belirteç** | Daemon başlangıcı (rastgele UUID) | Daemon süreç yaşam süresi | Tam komut yüzeyi, yalnızca yerel dinleyici — tünel üzerinden 403 |
| **Kurulum anahtarı** | `POST /pair` | 5 dakika, tek kullanım | Tek kullanım: `/connect`'te sun, kapsamlı bir belirteç al |
| **Kapsamlı belirteç** | `POST /connect` (kurulum anahtarı ile) | 24 saat | İstemci başı, izin listesi bağlı, isteğe bağlı sekme kapsamlı |

Kök belirteç chmod 600 ile `<project>/.gstack/browse.json` dosyasına yazılır.
Tarayıcı durumunu değiştiren her komut `Authorization: Bearer <belirteç>`
içermelidir.

### SSE oturum çerezi (v1.6.0.0+)

SSE uç noktaları (`/activity/stream`, `/inspector/events`) Bearer belirtecini
VEYA `POST /sse-session` üzerinden oluşturulan 30 dakikalık HttpOnly
`gstack_sse` çerezini kabul eder. `?token=<ROOT>` sorgu-parametre kimlik
doğrulaması artık desteklenmemektedir. Bu, Chrome uzantısının kök belirteci
uzantı depolamasına koymadan etkinlik akışına abone olmasını sağlar.

### PTY oturum çerezi

Terminal bölmesi `POST /pty-session` üzerinden oluşturulan ayrı bir oturum
çerezi, `gstack_pty` kullanır. Farklı kapsam — canlı `claude` PTY'yi
başlatabilir/sürdürebilir, keyfi `/command` çağrıları yürütemez. `/health` uç
noktası bu belirteci yüzeye çıkarmamalıdır.

### Belirteç kayıt defteri

`browse/src/token-registry.ts` üç tür için oluşturma/doğrulama/iptal işlemlerini
artı belirteç başı hız sınırlamayı yönetir. Kurulum anahtarları tek kullanımlık;
kapsamlı belirteçlerin 24 saatlik kayan bir penceresi vardır; kök belirteç her
daemon başlangıcında döndürülür.

---

## Güvenlik yığını

Komut enjeksiyonuna karşı katmanlı savunma. Her katman, her kullanıcı mesajında
ve güvenilmeyen içerik taşıyabilecek her araç çıktısında (Read, Glob, Grep,
WebFetch, `$B`'den sayfa metni) senkron olarak çalışır.

| Katman | Modül | Yaşadığı yer |
|--------|-------|-------------|
| **L1** Veri işaretleme | `content-security.ts` | hem sunucu + kenar çubuğu ajanı |
| **L2** Gizli öğe şeritleri | `content-security.ts` | her ikisi |
| **L3** ARIA + URL engelleme listesi + zarf sarmalama | `content-security.ts` | her ikisi |
| **L4** TestSavantAI ML sınıflandırıcısı (22MB ONNX) | `security-classifier.ts` | yalnızca kenar çubuğu ajanı* |
| **L4b** Claude Haiku transkript kontrolü | `security-classifier.ts` | yalnızca kenar çubuğu ajanı |
| **L5** Kanarya belirteç (oturum-sızdırma tespiti) | `security.ts` | her ikisi — derlenmişte enjekte, ajanda kontrol |
| **L6** `combineVerdict` topluluğu | `security.ts` | her ikisi |

\* `security-classifier.ts` derlenmiş browse ikili dosyasından içe aktarılamaz —
`@huggingface/transformers` v4, Bun derlemesinin geçici çıkarma dizininden
`dlopen` yapamadığı `onnxruntime-node` gerektirir. Derlenmiş ikili dosya
yalnızca L1–L3, L5, L6 çalıştırır.

### Eşikler

- `BLOCK: 0.85` — çapraz onaylanırsa BLOCK'a neden olacak tek katman puanı
- `WARN: 0.75` — çapraz onay eşiği. L4 VE L4b her ikisi >= 0.75 olduğunda → BLOCK
- `LOG_ONLY: 0.40` — transkript sınıflandırıcısını geçitler (tüm katmanlar < 0.40 olduğunda Haiku'yu atlar)
- `SOLO_CONTENT_BLOCK: 0.92` — etiketsiz içerik sınıflandırıcıları için tek katman eşiği

### Topluluk kuralı

Yalnızca ML içerik sınıflandırıcısı VE transkript sınıflandırıcısı her ikisi
>= WARN bildirdiğinde BLOCK'la. Tek katman yüksek güvenilirlik WARN'a düşürülür —
bu Stack Overflow talimat-yazma FP azaltımıdır. **Kanıbanya sızıntısı her zaman
BLOCK'lar (deterministik).**

### Ortam düğmeleri

- `GSTACK_SECURITY_OFF=1` — acil öldürme anahtarı. Sınıflandırıcı ısınmış olsa
  bile kapalı kalır. Kanarya hala enjekte edilir; sadece ML taraması atlanır.
- `GSTACK_SECURITY_ENSEMBLE=deberta` — katılım DeBERTa-v3 topluluğu. L4c
  sınıflandırıcı olarak ProtectAI DeBERTa-v3-base-injection-onnx ekler. 721MB
  ilk çalıştırma indirmesi. Topluluk etkinleştirildiğinde, BLOCK için 3 ML
  sınıflandırıcısından 2'sinin >= WARN'da anlaşması gerekir.
- Sınıflandırıcı model önbelleği: `~/.gstack/models/testsavant-small/` (112MB,
  yalnızca ilk çalıştırma) artı `~/.gstack/models/deberta-v3-injection/`
  (721MB, yalnızca topluluk etkinleştirildiğinde).
- Saldırı günlüğü: `~/.gstack/security/attempts.jsonl` (tuzlanmış SHA-256 +
  yalnızca alan adı, 10MB'da döner, 5 nesil).
- Cihaz başı tuz: `~/.gstack/security/device-salt` (0600).
- Oturum durumu: `~/.gstack/security/session-state.json` (süreçler arası,
  atomik).

Kenar çubuğu başlığındaki kalkan simgesi canlı durumu gösterir. Tam tehdit
modeli için ARCHITECTURE.md § "Prompt injection defense" bölümüne bakın.

---

## Ekran görüntüleri, PDF'ler, görsel

### Ekran görüntüsü modları

| Mod | Sözdizimi | Playwright API'si |
|-----|-----------|-------------------|
| Tam sayfa (varsayılan) | `screenshot [yol]` | `page.screenshot({ fullPage: true })` |
| Yalnızca görüntü alanı | `screenshot --viewport [yol]` | `page.screenshot({ fullPage: false })` |
| Öğe kırpma (bayrak) | `screenshot --selector <css> [yol]` | `locator.screenshot()` |
| Öğe kırpma (konumsal) | `screenshot "#sel" [yol]` veya `screenshot @e3 [yol]` | `locator.screenshot()` |
| Bölge kırpma | `screenshot --clip x,y,w,h [yol]` | `page.screenshot({ clip })` |

Öğe kırpma CSS seçicileri (`.class`, `#id`, `[attr]`) veya `@e`/`@c`
referansları kabul eder. **`button` gibi etiket seçicileri konumsal buluş
tarafından yakalanmaz** — `--selector` bayrak formunu kullanın.

`--base64` diske yazmak yerine `data:image/png;base64,...` döndürür —
`--selector`, `--clip`, `--viewport` ile birleşir.

Karşılıklı dışlama: `--clip` + seçici, `--viewport` + `--clip` ve
`--selector` + konumsal seçicinin hepsi hata fırlatır.

### Retina ekran görüntüleri — `viewport --scale`

`viewport --scale <n>` Playwright'ın `deviceScaleFactor`'ünü ayarlar
(bağlam düzeyinde, 1–3 sınırı):

```bash
$B viewport 480x600 --scale 2
$B load-html /tmp/card.html
$B screenshot /tmp/card.png --selector .card
# 400x200 CSS pikseldeki .card → card.png 800x400 piksel
```

Yalnızca `--scale N` ( `WxH` yok) mevcut görüntü alanı boyutunu korur. Ölçek
değişiklikleri bağlam yeniden oluşturmayı tetikler, bu da `@e`/`@c`
referanslarını geçersiz kılar — sonrasında `snapshot`'ı yeniden çalıştırın.
`load-html` ile yüklenen HTML, bellek içi yeniden oynatma yoluyla yeniden
oluşturmadan sağ kurtulur. Headed modda reddedilir (gerçek tarayıcı ölçeği
kontrol eder).

### PDF oluşturma

`pdf` tam Playwright yüzeyini artı birkaç eklemeyi kabul eder:

- **Düzen:** `--format letter|a4|legal`, `--width <boyut>`, `--height <boyut>`,
  `--margins <boyut>`, `--margin-top/right/bottom/left <boyut>`
- **Yapı:** `--toc` (yüklenmişse Paged.js bekler), `--outline`,
  `--tagged` (PDF/A erişilebilirlik), `--print-background`,
  `--prefer-css-page-size`
- **Markalama:** `--header-template <html>`, `--footer-template <html>`,
  `--page-numbers`
- **Sekmeler:** `--tab-id <N>` belirli bir sekmeyi render etmek için
- **Büyük yükler:** `--from-file <payload.json>` (shell argv sınırlarından kaçınır)

### Duyarlı ekran görüntüleri

`responsive [önek]` — tek çağrıda üç ekran görüntüsü: mobil (375x812),
tablet (768x1024), masaüstü (1280x720). `{prefix}-mobile.png` vb. olarak kaydeder.

### `prettyscreenshot`

Tek çağrıda temizlik + kaydırma + öğe gizlemeyi birleştirir:

```bash
$B prettyscreenshot --cleanup --scroll-to "hero section" --hide ".cookie-banner" /tmp/clean.png
```

---

## Yerel HTML

Bir web sunucusunda olmayan HTML'yi render etmenin iki yolu:

| Yaklaşım | Ne zaman | Sonraki URL | Göreceli varlıklar |
|-----------|----------|-------------|---------------------|
| `goto file://<mutlak-yol>` | Dosya zaten diskte | `file:///...` | Dosyanın dizinine göre çözümlenir |
| `goto file://./<göreceli>`, `goto file://~/<göreceli>` | Mutlakağa akıllı ayrıştırılır | `file:///...` | Aynı |
| `load-html <dosya>` | Bellekte oluşturulan HTML, üst dizin bağlamı gerekmez | `about:blank` | Bozuk (yalnızca kendi içindeki HTML) |

Her ikisi de `eval` ile aynı güvenli dizin politikasıyla cwd veya `$TMPDIR`
altındaki dosyalarla kapsamlıdır. `file://` URL'leri sorgu dizelerini ve
parçaları korur (SPA yönelmeleri çalışır).

`load-html`'ın bir uzantı izin listesi (`.html`, `.htm`, `.xhtml`, `.svg`) ve
ikili dosyaları HTML olarak yanlış adlandırılmış olarak reddetmek için bir
majik-bayt koklama var. 50MB boyut sınırı (`GSTACK_BROWSE_MAX_HTML_BYTES` ile
geçersiz kılınabilir).

`load-html` içeriği daha sonraki `viewport --scale` çağrılarını bellek içi
yeniden oynatma yoluyla sağ kalır (TabSession yüklenen HTML'yi + waitUntil'i
izler). Yeniden oynatma tamamen bellek içidir — HTML asla `state save` ile
diske kalıcı olarak yazılmaz, bu da sırların veya müşteri verilerinin sızdırmasını
önler.

---

## Toplu iş uç noktası

`POST /batch` tek bir HTTP isteğinde birden fazla komut gönderir. Komut başına
gidiş-dönüş gecikmesini ortadan kaldırır — her HTTP çağrısının 2-5s maliyetli
olduğu ngrok üzerinden uzak ajanlar için kritik.

```json
POST /batch
Authorization: Bearer <belirteç>

{
  "commands": [
    {"command": "text", "tabId": 1},
    {"command": "text", "tabId": 2},
    {"command": "snapshot", "args": ["-i"], "tabId": 3},
    {"command": "click", "args": ["@e5"], "tabId": 4}
  ]
}
```

Her komut `handleCommandInternal` üzerinden yönlendirilir — komut başına tam
güvenlik hattı (kapsam kontrolleri, alan doğrulama, sekme sahipliği, içerik
sarmalama) uygulanır. Komut başına hata izolasyonu: bir başarısızlık toplu
işi iptal etmez. Toplu iş başına maksimum 50 komut. İç içe toplu işler reddedilir.
Hız sınırlama: 1 toplu iş = ajan başına sınır karşı 1 istek.

Desen: 20 sayfa tarayan bir ajan 20 sekme açar (tek tek `newtab` veya toplu),
sonra 20 `text` komutuyla `POST /batch` → seri ~40-100 saniye yerine toplam
~2-3 saniyede 20 sayfa içeriği.

---

## Yakalama

Konsol, ağ ve iletişim kutusu olayları O(1) dairesel tamponlara akar
(her biri 50.000 kapasite), `Bun.write()` ile asenkron olarak diske boşaltılır:

- Konsol: `.gstack/browse-console.log`
- Ağ: `.gstack/browse-network.log`
- İletişim kutusu: `.gstack/browse-dialog.log`

`console`, `network` ve `dialog` komutları bellek içi tamponlardan okur
(diske değil), böylece disk yavaş olsa bile yakalama gerçek zamanlıdır.

İletişim kutuları (alert, confirm, prompt) tarayıcı kilitlenmesini önlemek için
varsayılan olarak otomatik kabul edilir. `dialog-accept <metin>` prompt yanıt
metnini kontrol eder.

---

## JS yürütme

`js` satır içi bir ifade çalıştırır. `eval` bir JS dosyası çalıştırır. Her ikisi
de **aynı JS sanal alanında** çalışır — tek fark satır içi vs dosya. Her ikisi
de `await` destekler — `await` içeren ifadeler otomatik olarak asenkron bir
bağlamda sarılır:

```bash
$B js "await fetch('/api/data').then(r => r.json())"   # otomatik sarılmış
$B js "document.title"                                  # sarmaya gerek yok
$B eval my-script.js                                    # await ile dosya
```

`eval` dosyaları için, tek satırlık dosyalar ifade değerini doğrudan döndürür.
Çok satırlı dosyalar `await` kullanırken açık `return` gerektirir. "await"
kelimesini içeren yorumlar sarmayı tetiklemez.

Yol güvenliği: `eval` cwd veya `/tmp` dışındaki yolları reddeder. `js` hiç
dosya okumaz.

---

## Sekmeler, çerçeveler, durum

### Sekmeler

```bash
$B tabs                          # tüm açık sekmeleri listele
$B tab 3                         # 3. sekmeye geç
$B newtab https://example.com    # yeni sekme aç, geç
$B newtab --json                 # programlı: {"tabId":N,"url":...} döndürür
$B closetab                      # geçerli sekmeyi kapat
$B closetab 2                    # 2. sekmeyi kapat
$B tab-each "text"               # her sekmede "text" çalıştır, JSON döndür
```

`tab-each <komut>` her açık sekmede bir komutu dağıtır ve bir JSON dizisi
döndürür — "açtığım her sekmenin metnini ver" için kullanışlı.

### Çerçeveler

```bash
$B frame "#stripe-iframe"        # seçiciyle iframe'e geç
$B frame @e7                     # referansla
$B frame --name "checkout"       # name özniteliğiyle
$B frame --url "stripe.com"      # URL örüntü eşleşmesiyle
$B frame main                    # üst çerçeveye geri dön
```

Geçişte referanslar temizlenir (iframe'in kendi AX ağacı vardır).

### Durum kaydet/yükle

```bash
$B state save oturumum          # çerezler + URL'leri .gstack/browse-state-oturumum.json dosyasına kaydet
$B state load oturumum          # geri yükle
```

Bellek içi `load-html` içeriği kasıtlı olarak kalıcı yapılmaz (sızıntıyı önlemek
için).

### İzleme

```bash
$B watch                         # pasif gözlem: kullanıcı gezinirken 5s'de bir anlık görüntü
$B watch stop                    # nelerin değiştiğinin özetini döndür
```

Tarayıcıyı manuel olarak kullanırken ve sonunda Claude'un ne yaptığını görmesini
istemeyip `snapshot` çağrılarını spamlamak istemediğinizde kullanışlı.

### Gelen kutusu

```bash
$B inbox                         # kenar çubuğu izci gelen kutusundaki mesajları listele
$B inbox --clear                 # okuduktan sonra temizle
```

Kenar çubuğu izcisi (Chrome uzantısının başlatabileceği bir arka plan süreci),
kullanıcı fark etmesini istediği bir şey olduğunda Claude için notlar bırakır.
`.gstack/browser-scout.jsonl` içinde saklanır.

---

## CDP

### `$B cdp` — ham Chrome DevTools Protokolü gönderimi

Reddet-varsayılan. Yalnızca `browse/src/cdp-allowlist.ts`'te listelenen yöntemler
(`CDP_ALLOWLIST` sabiti) erişilebilir; diğer herhangi bir yöntem 403 döndürür.
Her izin listesi girdisi kapsam (sekme vs tarayıcı) ve çıktı (güvenilir vs
güvenilmeyen) bildirir. Güvenilmeyen yöntemler (veri-sızdırma şeklindeki, örn.
`Network.getResponseBody`) UNTRUSTED-zarf sarmalı çıktı alır.

```bash
$B cdp Page.getLayoutMetrics
$B cdp Network.enable
$B cdp Accessibility.getFullAXTree --json '{"max_depth":5}'
```

İzin verilen yöntemleri keşfetmek için: `browse/src/cdp-allowlist.ts` dosyasını okuyun.

### `$B inspect` — CDP tabanlı CSS denetçisi

```bash
$B inspect ".header"                # header için tam kural basamaklaması
$B inspect ".header" --all          # kullanıcı-agent kurallarını dahil et
$B inspect ".header" --history      # değişiklik geçmişini göster
```

Eşleşen kural basamaklamasını özgüllük, hesaplanmış stiller, kutu modeli ve
(`--history` ile) sayfa yüklendikten bu yana `$B style` ile yapılan her CSS
değişikliğini döndürür. `browse/src/cdp-inspector.ts`'teki sayfa başına kalıcı
CDP oturumu tarafından desteklenir.

### `$B ux-audit`

```bash
$B ux-audit
```

Site kimliği, navigasyon, başlıklar (上限 50), metin blokları, etkileşimli
öğeler (上限 200) ile JSON döndürür — tam HTML dökmeden davranışsal analiz için
sayfa yapısı. `/qa` ve `/design-review` tarafından ucuz kapsam haritaları için
kullanılır.

---

## Performans

| Araç | İlk çağrı | Sonraki çağrılar | Çağrı başı bağlam ek yükü |
|------|----------|------------------|---------------------------|
| Chrome MCP | ~5s | ~2-5s | ~2000 token (şema + protokol) |
| Playwright MCP | ~3s | ~1-3s | ~1500 token (şema + protokol) |
| **gstack browse** | **~3s** | **~100-200ms** | **0 token** (düz metin stdout) |
| **gstack browse + kodlanmış beceri** | **~3s** | **~200ms** | **0 token** (tek beceri çağrısı) |

20 komutluk bir tarayıcı oturumunda, MCP araçları yalnızca protokol çerçevelemesi
üzere 30.000–40.000 token yakar. gstack sıfır yakar. Kodlanmış-beceri yolu
20 komutluk bir oturumu tek bir `$B skill run` çağrısına indirir.

### Neden MCP yerine CLI

MCP uzak hizmetler için iyi çalışır. Yerel tarayıcı otomasyonu için saf ek yük
ekler:

- **Bağlam şişirmesi** — her MCP çağrısı tam JSON şemaları içerir. Basit bir
  "sayfa metnini al" olması gerekenden 10x daha fazla bağlam token'ı maliyetindedir.
- **Bağlantı kırılganlığı** — kalıcı WebSocket/stdio bağlantıları düşer ve
  yeniden bağlanamaz.
- **Gereksiz soyutlama** — Claude'un zaten bir Bash aracı var. Stdout'a yazdıran
  bir CLI mümkün olan en basit arayüzdür.

gstack bunların hepsini atlar. Derlenmiş ikili dosya. Düz metin girdi, düz metin
çıktı. Protokol yok. Şema yok. Bağlantı yönetimi yok.

---

## Çoklu-çalışma alanı

Her proje kökü (`git rev-parse --show-toplevel` ile tespit edilir) kendi
daemon'unu, portunu, durum dosyasını, çerezlerini ve günlüklerini alır. Çalışma
alanları arası çakışma yok.

| Çalışma alanı | Durum dosyası | Port |
|---------------|---------------|------|
| `/code/project-a` | `/code/project-a/.gstack/browse.json` | rastgele (10000–60000) |
| `/code/project-b` | `/code/project-b/.gstack/browse.json` | rastgele (10000–60000) |

Tarayıcı-becerileri üç katmanlı arama projesi → global → paketlenmiş şeklinde
yürür, bu nedenle `/code/project-a/.gstack/browser-skills/foo/` konumundaki bir
proje katmanlı beceri, global `~/.gstack/browser-skills/foo/`'u yalnızca
project-a içinde gölgeler.

---

## Ortam değişkenleri

| Değişken | Varsayılan | Açıklama |
|----------|-----------|----------|
| `BROWSE_PORT` | 0 (rastgele 10000–60000) | HTTP sunucusu için sabit port (hata ayıklama geçersiz kılması) |
| `BROWSE_IDLE_TIMEOUT` | 1800000 (30 dak) | Boşta kapatma zaman aşımı (ms) |
| `BROWSE_STATE_FILE` | `.gstack/browse.json` | Durum dosyası yolu |
| `BROWSE_SERVER_SCRIPT` | otomatik algılandı | `server.ts` yolu |
| `BROWSE_CDP_URL` | (yok) | Gerçek tarayıcı modu için `channel:chrome` olarak ayarlayın |
| `BROWSE_CDP_PORT` | 0 | CDP portu (dahili olarak kullanılır) |
| `BROWSE_HEADLESS_SKIP` | 0 | Chromium başlatmayı tamamen atla (yalnızca test donanımı) |
| `BROWSE_TUNNEL` | 0 | Çift-dinleyici tünel mimarisini etkinleştir (`NGROK_AUTHTOKEN` gerektirir) |
| `BROWSE_TUNNEL_LOCAL_ONLY` | 0 | Yalnızca test — ngrok olmadan her iki dinleyiciyi yerel olarak bağla |
| `GSTACK_BROWSE_MAX_HTML_BYTES` | 52428800 (50MB) | `load-html` boyut sınırı |
| `GSTACK_SECURITY_OFF` | ayarlanmamış | Acil öldürme anahtarı — ML sınıflandırıcısını devre dışı bırak |
| `GSTACK_SECURITY_ENSEMBLE` | ayarlanmamış | 3 sınıflandırıcı topluluğu için `deberta` olarak ayarlayın (721MB indirme) |

---

## Kaynak haritası

```
browse/
├── src/
│   ├── cli.ts                   # İnce istemci — durum okur, HTTP gönderir, yazdırır
│   ├── server.ts                # Bun HTTP daemon — komutları yönlendirir, çift-dinleyici
│   ├── browser-manager.ts       # Chromium yaşam döngüsü, sekmeler, referans eşlemesi, çökme tespiti
│   ├── socks-bridge.ts          # Chromium'un konuşamadığı kimlik doğrulama el sıkışmasını işleyen yerel 127.0.0.1 SOCKS5 köprüsü
│   ├── proxy-config.ts          # --proxy URL ayrıştırma + kimlik bilgisi çözümleme (URL vs ortam, her ikisinde hızlı başarısızlık)
│   ├── proxy-redact.ts          # Günlüklere/hatalara yansıtılan herhangi bir proxy URL'si için kimlik bilgisi sansürleme yardımcısı
│   ├── xvfb.ts                  # Xvfb otomatik başlatma + PID + başlangıç zamanı doğrulama ile yetim temizlik
│   ├── stealth.ts               # navigator.webdriver maskesi + cdc_ temizliği + Permissions API yaması
│   ├── browse-client.ts         # Kanonik SDK — becerilerin _lib/browse-client.ts olarak içe aktardığı
│   ├── snapshot.ts              # AX ağacı → @e/@c referansları → Locator eşlemesi; -D/-a/-C işleme
│   ├── read-commands.ts         # Değiştirmeyen: text, html, links, js, css, is, dialog, ...
│   ├── write-commands.ts        # Değiştiren: goto, click, fill, upload, dialog-accept, ...
│   ├── meta-commands.ts         # state, watch, inbox, frame, ux-audit, chain, diff, ...
│   ├── browser-skills.ts       # 3 katmanlı yürüyüş + frontmatter ayrıştırıcı + mezar taşları
│   ├── browser-skill-commands.ts # $B skill list/show/run/test/rm + spawnSkill
│   ├── browser-skill-write.ts   # /skillify için D3 atomik stage/commit/discard yardımcısı
│   ├── skill-token.ts           # mintSkillToken / revokeSkillToken (çağrı başı, kapsamlı)
│   ├── domain-skills.ts         # Site başı ajan notları (durum makinesi: quarantined→active→global)
│   ├── domain-skill-commands.ts # $B domain-skill save/list/show/edit/promote/rollback/rm
│   ├── cdp-allowlist.ts         # Reddet-varsayılan CDP yöntem izin listesi
│   ├── cdp-bridge.ts            # CDP oturum yaşam döngüsü köprüsü
│   ├── cdp-commands.ts          # $B cdp gönderici
│   ├── cdp-inspector.ts         # $B inspect — sayfa başına kalıcı CDP oturumu
│   ├── activity.ts              # ActivityEntry, CircularBuffer, SSE aboneleri, gizlilik filtreleme
│   ├── buffers.ts               # Konsol/ağ/iletişim kutusu dairesel tamponlar (O(1) halka)
│   ├── tab-session.ts           # Sekme başı oturum durumu (load-html yeniden oynatma, referans eşleme kapsamı)
│   ├── token-registry.ts        # Kök + kurulum anahtarları + kapsamlı belirteçler için oluştur/doğrula/iptal
│   ├── sse-session-cookie.ts    # /activity/stream + /inspector/events için 30 dakikalık HttpOnly çerez
│   ├── pty-session-cookie.ts    # Ayrı kapsam: canlı Claude PTY kimlik doğrulaması
│   ├── tunnel-denial-log.ts     # ~/.gstack/security/attempts.jsonl yazıcı (tuzlanmış)
│   ├── path-security.ts         # validateOutputPath / validateReadPath / validateTempPath
│   ├── url-validation.ts        # goto için URL güvenlik kontrolleri
│   ├── content-security.ts      # L1-L3: veri işaretleme, gizli şerit, ARIA, URL engelleme listesi, zarflar
│   ├── security.ts              # L5 kanarya + L6 karar birleştirici + eşikler
│   ├── security-classifier.ts   # L4 ML sınıflandırıcı (TestSavant + isteğe bağlı DeBERTa topluluğu)
│   ├── terminal-agent.ts        # Side Panel Claude PTY yöneticisi (kimlik doğrulama + yaşam döngüsü)
│   ├── sidebar-utils.ts         # Kenar çubuğu URL sansürleme + yardımcılar
│   ├── cookie-import-browser.ts # Gerçek Chromium tarayıcılarından çerez şifre çözme + içe aktarma
│   ├── cookie-picker-routes.ts  # /cookie-picker/* için HTTP yolları
│   ├── cookie-picker-ui.ts      # Çerez seçici için kendi içinde HTML/CSS/JS
│   ├── network-capture.ts       # $B network için ağ isteği yakalama
│   ├── media-extract.ts         # $B media için medya öğesi çıkarma
│   ├── project-slug.ts          # Durum yolları için proje kısa adı türetme
│   ├── error-handling.ts        # safeUnlink / safeKill / isProcessAlive
│   ├── platform.ts              # İşletim sistemi tespiti (macOS, Linux, Windows)
│   ├── telemetry.ts             # Anonim katılımlı kullanım telemetrisi
│   ├── find-browse.ts           # Çalışan daemon'u bul veya başlat
│   └── config.ts                # Yapılandırma çözümleme (ortam / dosyalar)
├── test/                        # Entegrasyon testleri + HTML fixture'ları
└── dist/
    └── browse                   # Derlenmiş ikili dosya (~58MB, Bun --compile)

browser-skills/
└── hackernews-frontpage/        # Paketlenmiş referans becerisi
    ├── SKILL.md
    ├── script.ts
    ├── _lib/browse-client.ts
    ├── fixtures/hn-2026-04-26.html
    └── script.test.ts

scrape/SKILL.md.tmpl             # /scrape gstack becerisi — eşleşme-veya-prototip giriş noktası
skillify/SKILL.md.tmpl           # /skillify gstack becerisi — son /scrape'ı kalıcı beceriye kodla
```

---

## Geliştirme

### Ön koşullar

- [Bun](https://bun.sh/) v1.0+
- Playwright'ın Chromium'u (`bun install` tarafından otomatik kurulur)

### Hızlı başlangıç

```bash
bun install                      # bağımlılıkları + Playwright Chromium'u kur
bun test                         # tüm entegrasyon testleri (~3s yalnızca browse)
bun run dev <komut>              # CLI'yi kaynaktan çalıştır (derleme yok)
bun run build                    # browse/dist/browse konumuna derle
```

### Geliştirme modu vs derlenmiş ikili dosya

Geliştirme sırasında derlenmiş ikili dosya yerine `bun run dev` kullanın. CLI'yi
Bun ile doğrudan `browse/src/cli.ts` olarak çalıştırır, böylece anında geri
bildirim alırsınız:

```bash
bun run dev goto https://example.com
bun run dev text
bun run dev snapshot -i
bun run dev click @e3
```

Derlenmiş ikili dosya (`bun run build`) yalnızca dağıtım için gereklidir. Bun'un
`--compile` bayrağını kullanarak `browse/dist/browse` konumunda tek bir ~58MB
çalıştırılabilir dosya üretir.

### Testleri çalıştırma

```bash
bun test                                    # tüm testler
bun test browse/test/commands               # komut entegrasyon testleri
bun test browse/test/snapshot               # anlık görüntü testleri
bun test browse/test/cookie-import-browser  # çerez içe aktarma birim testleri
bun test browse/test/browser-skill-write    # D3 atomik yazma yardımcı testleri
bun test browse/test/tunnel-gate-unit       # canDispatchOverTunnel saf testleri
```

Testler `browse/test/fixtures/` dizininden HTML fixture'ları sunan yerel bir HTTP
sunucusu (`browse/test/test-server.ts`) başlatır, ardından CLI'yi bu sayfalara
karşı çalıştırır.

### Yeni komut ekleme

1. İşleyiciyi `read-commands.ts` (değiştirmeyen) veya `write-commands.ts`
   (değiştiren) veya `meta-commands.ts` (sunucu / yaşam döngüsü) dosyasına ekleyin.
2. Rotayı `server.ts`'te kaydedin.
3. Girdiyi `browse/src/commands.ts`'teki `COMMAND_DESCRIPTIONS`'a ekleyin
   (net bir `description` ve `usage` ile — `gen-skill-docs` doğrulama paketi
   `description`'da `|` karakterlerine izin vermez).
4. Gerekirse bir HTML fixture'ı ile `browse/test/commands.test.ts`'te bir test
   durumu ekleyin.
5. Doğrulamak için `bun test` çalıştırın.
6. Derlemek için `bun run build` çalıştırın.
7. SKILL.md'yi yeniden oluşturmak için `bun run gen:skill-docs` çalıştırın
   (komut aşağı akış komut referansı tablosunda görünür).

### Yeni tarayıcı-becerisi ekleme

El ile yazılmış bir beceri için: `browser-skills/hackernews-frontpage/` kopyasını
kullanın, SKILL.md frontmatter'ı güncelleyin, `script.ts`'yi hedef sitenize
karşı yeniden yazın, fixture'ı yeniden yakalayın, ayrıştırıcı testini güncelleyin.
`bun test` SKILL.md sözleşmesini (kardeş SDK bayt-özdeşliği, frontmatter şeması)
doğrular.

Ajan yazılmış bir beceri için: sayfayı `/scrape <niyet>` ile bir kez tarayın,
`/skillify` deyin, onay geçidinde önerilen adı kabul edin. Beceri test geçtikten
sonra `~/.gstack/browser-skills/<ad>/` konumuna yerleşir.

### Aktif beceriye dağıtma

Aktif beceri `~/.claude/skills/gstack/` konumunda yaşar. Değişiklikler yaptıktan
sonra:

```bash
cd ~/.claude/skills/gstack
git fetch origin && git reset --hard origin/main
bun run build
```

Veya ikili dosyayı doğrudan kopyalayın:

```bash
cp browse/dist/browse ~/.claude/skills/gstack/browse/dist/browse
```

---

## Çapraz referanslar

- [`ARCHITECTURE.md`](ARCHITECTURE.md) — sistem düzeyinde mimari, çift-dinleyici tünel tasarımı, komut enjeksiyonu savunma tehdit modeli
- [`CLAUDE.md`](CLAUDE.md) — proje düzeyinde talimatlar, kenar çubuğu mimari notları, güvenlik yığını kısıtlamaları
- [`docs/REMOTE_BROWSER_ACCESS.md`](docs/REMOTE_BROWSER_ACCESS.md) — `/pair-agent` için operatör kılavuzu (kurulum anahtarları, kapsamlı belirteçler, reddetme günlüğü)
- [`docs/designs/BROWSER_SKILLS_V1.md`](docs/designs/BROWSER_SKILLS_V1.md) — tarayıcı-becerileri çalışma zamanı tasarım dokümanı (Aşama 1 + 2a + yol haritası)
- [`scrape/SKILL.md`](scrape/SKILL.md) — `/scrape` becerisi: eşleşme-veya-prototip veri çıkarma
- [`skillify/SKILL.md`](skillify/SKILL.md) — `/skillify` becerisi: son `/scrape`'ı kalıcı beceriye kodla
- [`TODOS.md`](TODOS.md) — `/automate` (Aşama 2b P0), Aşama 3 çözücü enjeksiyonu, Aşama 4 değerlendirme + sanal alan

---

## Teşekkürler

Tarayıcı otomasyon katmanı Microsoft tarafından
[Playwright](https://playwright.dev/) üzerine inşa edilmiştir. Playwright'ın
erişilebilirlik ağacı API'si, bulucu sistemi ve headless Chromium yönetimi,
referans tabanlı etkileşimi mümkün kılan şeydir. Anlık görüntü sistemi — AX
ağacı düğümlerine `@ref` etiketleri atama ve bunları Playwright Bulucularına
geri eşleme — tamamen Playwright'ın ilkeleri üzerine inşa edilmiştir. Playwright
ekibine böyle sağlam bir temel inşa ettikleri için teşekkürler.

Komut enjeksiyonu L4 katmanı
[TestSavantAI/distilbert-v1.1-32](https://huggingface.co/TestSavantAI/distilbert-v1.1-32)
(112MB ONNX) kullanır ve isteğe bağlı topluluk katmanı
[ProtectAI/deberta-v3-base-prompt-injection-v2](https://huggingface.co/protectai/deberta-v3-base-prompt-injection-v2)
(721MB ONNX) kullanır — her ikisi de `@huggingface/transformers` aracılığıyla
yerel olarak çalışır.

CDP acil çıkışı, doğrudan Codex'in v1.4 tasarım geçişi sırasındaki T2 dış-ses
incelemesinden esinlenen bir izin listesi tarafından korunur: reddetme listesi
ile izin-ver-varsayılan değil, izin listesi ile reddet-varsayılan.