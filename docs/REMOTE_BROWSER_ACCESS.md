# Uzak Tarayıcı Erişimi — Bir GStack Tarayıcısına Nasıl Eşleştirilir

Bir GStack Tarayıcı sunucusu, HTTP isteği yapabilen herhangi bir AI aracısıyla paylaşılabilir.
Aracı, gerçek bir Chromium tarayıcısına kapsamlı erişim elde eder: sayfaları gezin, içerik
oku, öğelere tıkla, formları doldur, ekran görüntüsü al. Her aracı kendi sekmesini alır.

Bu belge uzak aracılar için referanstır. Hızlı başlangıç talimatları, gerçek kimlik
bilgileriyle birlikte `$B pair-agent` tarafından üretilir.

## Mimari

```
Sizin Makineniz                          Uzak Aracı
─────────────                         ────────────
GStack Tarayıcı Sunucusu                 Herhangi bir AI aracısı
  ├── Chromium (Playwright)           (OpenClaw, Hermes, Codex, vb.)
  ├── Yerel dinleyici  127.0.0.1:LOCAL         │
  │    (önyükleme, CLI, kenar çubuğu, çerezler)      │
  ├── Tünel dinleyicisi 127.0.0.1:TUNNEL ◄───────┤
  │    (yalnızca pair-agent: /connect, /command,   │
  │     /sidebar-chat — kilitli izin verilenler listesi)      │
  ├── ngrok tüneli (yalnızca tünel portunu iletir) │
  │     https://xxx.ngrok.dev ─────────────────┘
  └── Belirteç Kayıt Defteri
        ├── Kök belirteç (yalnızca yerel dinleyici)
        ├── Kurulum anahtarları (5 dk, tek kullanım)
        ├── Oturum belirteçleri (24 saat, kapsamlı)
        └── SSE oturum çerezleri (30 dk, akış-kapsamı)
```

### Çift dinleyici mimarisi (v1.6.0.0)

Artalan süreç iki HTTP soketi bağlar. **Yerel dinleyici** tam komut yüzeyini yalnızca
127.0.0.1'e sunar ve asla iletilemez. **Tünel dinleyicisi** `/tunnel/start` üzerinde
tembelce bağlanır (ve `/tunnel/stop` üzerinde kaldırılır) ve kilitli bir yol izin verilenler
listesi ile bağlanır. ngrok yalnızca tünel portunu iletir.

ngrok URL'nize rastlayan bir çağrıcı, `/health`, `/cookie-picker`, `/inspector/*` veya
`/welcome` uç noktalarına ulaşamaz — bu yollar o TCP soketinde mevcut değildir. Tünel
üzerinden gönderilen kök belirteçler 403 alır. Tünel dinleyicisi yalnızca `/connect`,
`/command` (kapsamlı belirteç + 26 komutluk tarayıcı sürme izin verilenler listesi ile) ve
`/sidebar-chat` uç noktalarını kabul eder.

Tam uç nokta tablosu için [ARCHITECTURE.md](../ARCHITECTURE.md#dual-listener-tunnel-architecture-v1600)
dosyasına bakın.

## Bağlantı Akışı

1. **Kullanıcı çalıştırır** `$B pair-agent` (veya Claude Code'da `/pair-agent`)
2. **Sunucu oluşturur** tek kullanımlık bir kurulum anahtarı (5 dakikada sona erer)
3. **Kullanıcı kopyalar** talimat bloğunu diğer aracının sohbetine
4. **Uzak aracı çalıştırır** kurulum anahtarı ile `POST /connect`
5. **Sunucu döndürür** kapsamlı bir oturum belirteci (varsayılan 24 saat)
6. **Uzak aracı oluşturur** `POST /command` ile `newtab` kullanarak kendi sekmesini
7. **Uzak aracı gezinir** oturum belirteci + tabId ile `POST /command` kullanarak

## API Referansı

### Kimlik Doğrulama

Tüm komut uç noktaları bir Bearer belirteci gerektirir:

```
Authorization: Bearer gsk_sess_...
```

`/connect` kimlik doğrulamasızdır (hız sınırlı) — uzak bir aracının kurulum anahtarını
kapsamlı bir oturum belirteci ile değiştirmesinin yoludur. `/health` yerel dinleyicide
kimlik doğrulamasızdır (önyükleme) ancak tünel dinleyicide MEVCUT DEĞİLDİR (404).

SSE uç noktaları (`/activity/stream`, `/inspector/events`) bir Bearer belirteci veya
HttpOnly `gstack_sse` çerezi kabul eder (`POST /sse-session` ile basılır, 30 dakikalık
TTL, yalnızca akış-kapsamı — `/command` karşı kullanılamaz). v1.6.0.0 itibarıyla
`?token=<ROOT>` sorgu dizesi kimlik doğrulaması artık kabul edilmez.

### Uç Noktalar

#### POST /connect
Kurulum anahtarını oturum belirteci ile değiştir. Kimlik doğrulama gerekmez. Dakikada
300 ile hız sınırlı (tahta saldırı savunması — kurulum anahtarları 24 rastgele bayttır,
kırılamaz).

```json
İstek:  {"setup_key": "gsk_setup_..."}
Yanıt: {"token": "gsk_sess_...", "expires": "ISO8601", "scopes": ["read","write"], "agent": "agent-name"}
```

#### POST /command
Bir tarayıcı komutu gönder. Bearer kimlik doğrulaması gerektirir.

```json
İstek:  {"command": "goto", "args": ["https://example.com"], "tabId": 1}
Yanıt: (komutun düz metin sonucu)
```

#### GET /health
Sunucu durumu. Kimlik doğrulama gerekmez. Durum, sekmeler, mod, çalışma süresi döndürür.

### Komutlar

#### Gezinme
| Komut | Argümanlar | Açıklama |
|---------|------|-------------|
| `goto` | `["URL"]` | Bir URL'ye git |
| `back` | `[]` | Geri git |
| `forward` | `[]` | İleri git |
| `reload` | `[]` | Sayfayı yeniden yükle |

#### İçerik Okuma
| Komut | Argümanlar | Açıklama |
|---------|------|-------------|
| `snapshot` | `["-i"]` | @ref etiketleriyle etkileşimli anlık görüntü (en kullanışlı) |
| `text` | `[]` | Tam sayfa metni |
| `html` | `["seçici?"]` | Öğenin veya tam sayfanın HTML'i |
| `links` | `[]` | Sayfadaki tüm bağlantılar |
| `screenshot` | `["/tmp/s.png"]` | Ekran görüntüsü al |
| `url` | `[]` | Geçerli URL |

#### Etkileşim
| Komut | Argümanlar | Açıklama |
|---------|------|-------------|
| `click` | `["@e3"]` | Bir öğeye tıkla (anlık görüntüden @ref kullan) |
| `fill` | `["@e5", "metin"]` | Bir form alanını doldur |
| `select` | `["@e7", "seçenek"]` | Açılır değer seç |
| `type` | `["metin"]` | Metin yaz (klavye) |
| `press` | `["Enter"]` | Bir tuşa bas |
| `scroll` | `["down"]` | Sayfayı kaydır |

#### Sekmeler
| Komut | Argümanlar | Açıklama |
|---------|------|-------------|
| `newtab` | `["URL?"]` | Yeni bir sekme oluştur (yazmadan önce gerekli) |
| `tabs` | `[]` | Tüm sekmeleri listele |
| `closetab` | `["id?"]` | Bir sekmeyi kapat |

## Anlık Görüntü → @ref Örüntüsü

Bu, en güçlü tarama örüntüsüdür. CSS seçiciler yazmak yerine:

1. Etiketli öğelerle etkileşimli bir anlık görüntü almak için `snapshot -i` çalıştırın
2. Anlık görüntü şöyle metin döndürür:
   ```
   [Sayfa Başlığı]
   @e1 [bağlantı] "Ana Sayfa"
   @e2 [düğme] "Oturum Aç"
   @e3 [giriş] "Ara..."
   ```
3. `@e` referanslarını doğrudan komutlarda kullanın: `click @e2`, `fill @e3 "arama sorgusu"`

Anlık görüntü sistemi böyle çalışır ve CSS seçicilerini tahmin etmekten çok daha güvenilirdir.
Her zaman önce `snapshot -i`, ardından referansları kullanın.

## Kapsamlar

| Kapsam | Neye izin verir |
|-------|---------------|
| `read` | snapshot, text, html, links, screenshot, url, tabs, console, vb. |
| `write` | goto, click, fill, scroll, newtab, closetab, vb. |
| `admin` | eval, js, cookies, storage, cookie-import, useragent, vb. |
| `meta` | tab, diff, frame, responsive, watch |

Varsayılan belirteçler `read` + `write` alır. Admin, eşleştirme sırasında `--admin` bayrağı gerektirir.

## Sekme Yalıtımı

Her aracı oluşturduğu sekmelere sahiptir. Kurallar:
- **Okuma:** Herhangi bir aracı herhangi bir sekmeyi okuyabilir (snapshot, text, screenshot)
- **Yazma:** Yalnızca sekme sahibi yazabilir (click, fill, goto, vb.)
- **Sahipsiz sekmeler:** Önceden var olan sekmeler yazma için yalnızca kök
- **İlk adım:** Etkileşim kurmadan önce her zaman `newtab`

## Hata Kodları

| Kod | Anlam | Ne yapmalı |
|------|---------|------------|
| 401 | Belirteç geçersiz, süresi dolmuş veya iptal edilmiş | Kullanıcıdan /pair-agent komutunu tekrar çalıştırmasını isteyin |
| 403 | Komut kapsamda değil veya sekme sizin değil | newtab kullanın veya --admin isteyin |
| 429 | Hız sınırı aşıldı (>10 istek/sn) | Retry-After başlığını bekleyin |

## Güvenlik Modeli

- **Fiziksel port ayrımı.** Yerel dinleyici ve tünel dinleyici ayrı TCP soketleridir. ngrok
  yalnızca tünel portunu iletir. Tünel çağrıcıları önyükleme uç noktalarına hiçbir şekilde
  ulaşamaz (404, yanlış port).
- **Tünel komut izin verilenler listesi.** Tünel üzerinden `/command` yalnızca 26 tarayıcı
  sürme komutunu kabul eder (goto, click, fill, snapshot, text, newtab, tabs, back, forward,
  reload, closetab, vb.). Sunucu yönetim komutları (tunnel, pair, token, useragent, js)
  tünelde reddedilir.
- **Kök belirteç tünel tarafından engellenir.** Tünel dinleyicisi üzerinden kök belirteç
  taşıyan bir istek, bir eşleştirme ipucu ile 403 döndürür. Tüneldede yalnızca kapsamlı
  oturum belirteçleri çalışır.
- **Kurulum anahtarları** 5 dakikada sona erer ve yalnızca bir kez kullanılabilir.
- **Oturum belirteçleri** 24 saatte sona erer (yapılandırılabilir).
- Kök belirteç asla talimat bloklarında veya bağlantı dizelerinde görünmez.
- **Admin kapsamı** (JS çalıştırma, çerez erişimi) varsayılan olarak reddedilir.
- Belirteçler anında iptal edilebilir: `$B tunnel revoke agent-name`
- **SSE kimlik doğrulaması** 30 dakikalık HttpOnly SameSite=Strict çerezi kullanır, yalnızca
  akış-kapsamı (`/command` karşı hiçbir zaman geçerli değil).
- **`/welcome` üzerinde yol geçişi korumalı** — `GSTACK_SLUG`, `^[a-z0-9_-]+$` ile eşleşmeli
  veya yerleşik şablona geri döner.
- **SSRF korumaları** `goto`, `download` ve kazıma yollarında — URL hedefini bir
  localhost/özel-aralık engelleme listesine karşı doğrular.
- **Tünel yüzey reddetme günlüğü.** Tünel dinleyicisindeki her reddetme
  (`path_not_on_tunnel`, `root_token_on_tunnel`, `missing_scoped_token`,
  `disallowed_command:*`), zaman damgası, kaynak IP, yol, yöntem ile
  `~/.gstack/security/attempts.jsonl` dosyasına eklenir. Dakikada 60 yazma ile hız sınırlı.
- Tüm aracı etkinliği, atıf ile günlüğe kaydedilir (clientId).

**Bilinen hedef dışı (#1136 olarak izleniyor):** Windows'ta, çerez-içe aktarma-tarayıcı yolu,
Chrome'u `--remote-debugging-port=<rastgele>` ile başlatır. App-Bound Encryption v20 ile,
aynı kullanıcı yerel süreci, şifresi çözülmüş v20 çerezlerini doğrudan SQLite DB'den
okumaya kıyasla bir yükseltme yolu olan bu porta bağlanabilir ve çerezleri sızdırabilir.
Düzeltme yönü, TCP yerine `--remote-debugging-pipe` kullanmaktır.

## Aynı Makine Kısayolu

Her iki aracı da aynı makinedeyse, kopyala-yapıştırı atlayın:

```bash
$B pair-agent --local openclaw    # ~/.openclaw/skills/gstack/browse-remote.json dosyasına yazar
$B pair-agent --local codex       # ~/.codex/skills/gstack/browse-remote.json dosyasına yazar
$B pair-agent --local cursor      # ~/.cursor/skills/gstack/browse-remote.json dosyasına yazar
```

Tünele gerek yok. Doğrudan localhost kullanır.

## ngrok Tüneli Kurulumu

Farklı makinelerdeki uzak aracılar için:

1. [ngrok.com](https://ngrok.com) adresinde kayıt olun (ücretsiz katman çalışır)
2. Kontrol panelinden kimlik doğrulama belirtecinizi kopyalayın
3. Kaydedin: `echo 'NGROK_AUTHTOKEN=sizin_belirteciniz' > ~/.gstack/ngrok.env`
4. İsteğe bağlı olarak kararlı bir etki alanı talep edin: `echo 'NGROK_DOMAIN=sizin-isminiz.ngrok-free.dev' >> ~/.gstack/ngrok.env`
5. Tünel ile başlatın: `BROWSE_TUNNEL=1 $B restart`
6. `$B pair-agent` çalıştırın — tünel URL'sini otomatik olarak kullanacak