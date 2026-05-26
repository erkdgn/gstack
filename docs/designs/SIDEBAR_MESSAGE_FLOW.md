# Kenar Çubuğu Akışı

GStack Browser kenar çubuğunun gerçekte nasıl çalıştığı. `sidepanel.js`, `background.js`, `content.js`, `terminal-agent.ts` veya kenar çubuğu ile ilgili sunucu endpoint'lerine dokunmadan önce bunu okuyun.

Kenar çubuğunun bir birincil yüzeyi var — **Terminal** bölmesi, etkileşimli bir `claude` PTY. Activity / Refs / Inspector, alt bilgi çubuğundaki `debug` geçişi arkasında hata ayıklama katmanları olarak kalır. Sohbet kuyruğu yolu (tek seferlik `claude -p`, sidebar-agent.ts), PTY kanıtlandığında kaldırıldı — Terminal bölmesi kesinlikle daha yeteneklidir.

## Bileşenler

```
┌─────────────────┐     ┌──────────────┐     ┌──────────────────┐
│  sidepanel.js + │────▶│  server.ts   │────▶│terminal-agent.ts │
│  -terminal.js   │     │  (derlenmiş)  │     │  (derlenmemiş)  │
│  (xterm.js)     │     │              │     │  PTY dinleyicisi    │
└─────────────────┘     └──────────────┘     └──────────────────┘
        ▲                       │                      │
        │  ws://127.0.0.1:<termPort>/ws (Sec-WebSocket-Protocol auth)
        └───────────────────────┼──────────────────────▶│ Bun.spawn(claude)
                                │                      │  terminal: {data}
                                │                      ▼
                                │              ┌──────────────────┐
                                │              │  claude PTY      │
                                │              └──────────────────┘
            POST /pty-session   │
            (Bearer AUTH_TOKEN) │
                                ▼
                       ┌──────────────────┐
                       │ pty-session-     │
                       │ cookie.ts        │
                       │ (bellek içi token │
                       │  kayıt defteri)       │
                       └──────────────────┘
                                │
                                │ POST /internal/grant (geri döngü)
                                ▼
                       ┌──────────────────┐
                       │  validTokens Set │
                       │  ajans belleğinde │
                       └──────────────────┘
```

Derlenmiş browse sunucusu harici çalıştırılabilir dosyaları `posix_spawn` yapamaz — `terminal-agent.ts` ayrı bir derlenmemiş `bun run` süreci olarak çalışır ve `claude` alt sürecine sahiptir.

## Başlangıç + ilk tuş vuruşu zaman çizelgesi

```
T+0ms     CLI `$B connect` çalıştırır
            ├── Sunucu başlar (derlenmiş)
            └── terminal-agent.ts'yi `bun run` ile başlatır

T+500ms   terminal-agent.ts başlar
            ├── 127.0.0.1:0 üzerinde Bun.serve (rastgele port)
            ├── <stateDir>/terminal-port yazar (sunucu /health için okur)
            ├── <stateDir>/terminal-internal-token yazar (geri döngü el sıkışması)
            └── claude'u yoklar → claude-available.json yazar

T+1-3s    Uzantı yüklenir, kenar çubuğu açılır
            ├── sidepanel-terminal.js: setState(IDLE), "Starting Claude Code..." gösterir
            └── tryAutoConnect(), window.gstackServerPort + token ayarlanana kadar yoklar

T+ready   tryAutoConnect, connect() çağırır
            ├── POST /pty-session (Authorization: Bearer AUTH_TOKEN)
            │   └── sunucu oturum tokeni oluşturur, ajansa /internal/grant gönderir
            │   └── {terminalPort, ptySessionToken} ile yanıt verir
            ├── GET /claude-available (ön kontrol)
            ├── new WebSocket(`ws://127.0.0.1:<terminalPort>/ws`,
            │                 [`gstack-pty.<token>`])
            │   └── Browser Sec-WebSocket-Protocol + Origin gönderir
            │   └── Arşı Origin VE tokeni yükseltmeden ÖNCE doğrular
            │   └── Arşı protokolü geri yansıtır (GEREKLI — browser olmadan
            │       bağlantıyı kapatır)
            ├── Açıkta: {type:"resize"} gönder, ardından tek bir \n byte gönder
            └── Arşı mesaj işleyicisi byte'i görür → spawnClaude()
```

## Kimlik Doğrulama: WebSocket Authorization başlıkları gönderemez

Browser WebSocket istemcileri `Authorization` ayarlayamaz. `new WebSocket(url, protocols)`'ün ikinci argümanı ile `Sec-WebSocket-Protocol` ayarlayabilirler. Bunu kullanırız:

1. `POST /pty-session` (auth: Bearer AUTH_TOKEN) → sunucu kısa ömürlü bir oturum tokeni oluşturur, geri döngü üzerinden ajansa iletir, JSON gövdesinde döndürür.
2. Uzantı `new WebSocket(url, ['gstack-pty.<token>'])` çağırır.
3. Arşı `Sec-WebSocket-Protocol` okur, `gstack-pty.`'yi ayırır, `validTokens` ile doğrular, protokolü geri yansıtır. Yansıtma zorunludur — olmadan Chromium yükseltme yanıtını alır almaz bağlantıyı kapatır.

Browser dışı çağıranlar (curl, entegrasyon testleri) için bir `Set-Cookie: gstack_pty=...` başlığı da döndürülür. Çerez yolu orijinal v1 tasarımıydı ama `SameSite=Strict` çerezleri, chrome-extension kaynağından server.ts:34567 → agent:<random> arasındaki çapraz port geçişini yaşamıyor. Protokol-token yolu browser'ın gerçekten kullandığı şeydir.

### Çift-token modeli

| Token | Yaşadığı yer | Kullanım amacı | Ömür |
|-------|----------|----------|----------|
| `AUTH_TOKEN` | `<stateDir>/browse.json`; server.ts içinde bellekte | `/pty-session` POST (çerez + token oluştur) | sunucu ömrü |
| `gstack-pty.<...>` (Sec-WebSocket-Protocol) | Sadece browser belleği; ajans `validTokens` Set'i | `/ws` yükseltme auth | 30 dk, WS kapandığında otomatik iptal |
| `INTERNAL_TOKEN` | `<stateDir>/terminal-internal-token`; ajans belleğinde | sunucu → ajans geri döngü `/internal/grant` | ajans ömrü |

`AUTH_TOKEN` doğrudan `/ws` için **asla** geçerli değildir. Oturum tokeni `/pty-session` veya `/command` için **asla** geçerli değildir. Katı ayrım, SSE veya sayfa içeriği token sızıntısının shell erişimine yükselmesini önler.

## Tehdit modeli

Terminal bölmesi **prompt enjeksiyon güvenlik yığınını kasıtlı olarak atlar** — kullanıcı doğrudan claude'a yazıyor, döngüde güvenilmeyen sayfa içeriği yok. Güven kaynağı klavyedir, herhangi bir yerel terminal ile aynı.

Bu güven varsayımı üç taşıyıcı aktarım garantisine dayanır:

1. **Sadece yerel dinleyici.** terminal-agent.ts sadece `127.0.0.1` bağlar. Çift dinleyici tünel yüzeyi (server.ts `TUNNEL_PATHS`) `/pty-session` veya `/terminal/*` içermez, bu nedenle tünel varsayılan reddetme ile 404 döndürür.
2. **Origin geçidi.** `/ws` yükseltmeleri `Origin: chrome-extension://<id>` gerektirir. Yerel bir web sayfası, Origin'i normal bir `http(s)://...` olduğu için shell'e karşı siteler arası WebSocket kaçırma saldırısı başlatamaz.
3. **Oturum tokeni kimlik doğrulaması.** Sadece kimliği doğrulanmış bir `/pty-session` POST tarafından oluşturulur, tek bir WS'ye kapsamlı, kapandığında otomatik iptal.

Bu üçünden birini bırakın ve tüm sekmeyi güvensiz hale getirin.

## Yaşam Döngüsü

- **İstekli otomatik bağlanma.** Kenar çubuğu açılır → tryAutoConnect, bootstrap global değişkenleri için yoklar ve ayarlanır ayarlanmaz bağlanır. Tuş vuruşu gerekmez.
- **WS başına bir PTY.** WebSocket kapatıldığında önce claude'a SIGINT gönderir, ardından 3s sonra SIGKILL. Oturum tokeni iptal edilir, bu nedenle çalınmış bir token yeniden oynatılamaz.
- **Kapandığında otomatik yeniden bağlanma yok.** Kullanıcı "Oturum sona erdi, yeni oturum başlatmak için tıklayın" mesajını görür. Otomatik yeniden bağlanma her yeniden yüklemede yeni bir claude oturumu yakar. v1.1, sekme/oturum kimliğine dayalı oturum sürdürme ekleyebilir (bkz. TODOS).
- **Her zaman manuel yeniden başlatma.** Terminal bölmesinin her zaman görünür araç çubuğunda bir `↻ Restart` düğmesi bulunur — oturum sırasında da çalışır, sadece ENDED durumundan değil.

## Hızlı eylem araç çubuğu

Terminal bölmesinin üst kısmındaki Restart düğmesinin yanında üç browser eylem düğmesi bulunur:

| Düğme | Davranış |
|--------|----------|
| 🧹 Temizle | `window.gstackInjectToTerminal(prompt)` — canlı PTY'ye "reklamları/bannerları kaldır" talimatı gönderir. terminaldeki claude bunu görür ve eyleme geçirir. |
| 📸 Ekran Görüntüsü | `POST /command screenshot` — doğrudan browse sunucusu çağrısı, PTY katılımı yok. |
| 🍪 Çerezler | `/cookie-picker` sayfasına yönlendirir. |

Denetçinin "Send to Code" düğmesi, CSS denetçi verilerini claude'a iletmek için aynı `gstackInjectToTerminal` yolunu kullanır.

## Hata ayıklama yüzeyleri (Activity / Refs / Inspector)

Alt bilgi çubuğundaki `debug` geçişi arkasında. SSE güdümlü, Terminal bölmesinden bağımsız:

- **Activity** — `/activity/stream` SSE üzerinden her browse komutunu akıtır.
- **Refs** — REST: `GET /refs` — geçerli sayfanın `@ref` öğe etiketleri.
- **Inspector** — CDP tabanlı öğe seçici; `/inspector/events` üzerinde SSE.

Hata ayıklama şeridi kapandığında, Terminal bölmesi yeniden görünür hale gelir.
xterm.js, kabı `display:none`'dan `display:flex`'e geçtiğinde otomatik olarak yeniden çizmez, bu nedenle sidepanel-terminal.js `#tab-terminal`'in sınıf niteliği üzerinde bir `MutationObserver` çalıştırır ve `.active` döndüğünde fit + yenileme zorlar.

## Dosyalar

| Bileşen | Dosya | Çalıştığı yer |
|-----------|------|---------|
| Kenar çubuğu UI kabı | `extension/sidepanel.html` + `sidepanel.js` + `sidepanel.css` | Chrome yan panel |
| Terminal UI | `extension/sidepanel-terminal.js` + `extension/lib/xterm.js` | Chrome yan panel |
| Servis çalışanı | `extension/background.js` | Chrome arka plan |
| İçerik betiği | `extension/content.js` | Sayfa bağlamı |
| HTTP sunucusu | `browse/src/server.ts` | Bun (derlenmiş ikili) |
| PTY aracı | `browse/src/terminal-agent.ts` | Bun (derlenmemiş) |
| PTY token deposu | `browse/src/pty-session-cookie.ts` | Bun (derlenmiş, server.ts içinde) |
| CLI giriş | `browse/src/cli.ts` | Bun (derlenmiş ikili) |
| Durum dosyası | `<stateDir>/browse.json` | Dosya sistemi |
| Terminal portu | `<stateDir>/terminal-port` | Dosya sistemi |
| İç token | `<stateDir>/terminal-internal-token` | Dosya sistemi |
| Claude yoklaması | `<stateDir>/claude-available.json` | Dosya sistemi |
| Aktif sekme | `<stateDir>/active-tab.json` | Dosya sistemi (claude okur) |