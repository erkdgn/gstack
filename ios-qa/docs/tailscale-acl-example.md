# iOS QA daemon'u için Tailscale ACL örneği

Mac tarafındaki daemon, Tailscale arayüzünü yalnızca `--tailnet` geçtiğinizde
bağlar. Varsayılan olarak daemon yalnızca yerel-USB üzerinden çalışır. Bu belge,
iPhone'unuzu uzak ajanlara güvenli bir şekilde sunmak için tailnet üzerinden
iOS QA çalıştırabilecekleri adımları açıklar.

## Tehdit modeli özeti

- **iOS uygulaması StateServer:** her zaman yalnızca geri döngü (loopback). Mac
  üzerinden CoreDevice IPv6 tüneli ile erişilebilir. Asla doğrudan tailnet'e
  bağlanmaz.
- **Mac daemon'u:** tailnet arayüzüne sahip. İki dinleyici bağlar — geri döngü
  (tam yüzey, asla yönlendirilmez) ve tailnet (yetki katmanlı kilitli izin listesi).
- **Kimlik doğrulama:** Yerel `tailscaled` soketi üzerinden Tailscale kimlik
  doğrulaması (`/var/run/tailscale.sock` LocalAPI WhoIs). `~/.gstack/ios-qa-allowlist.json`
  konumundaki izin listesi dosyası, kimin ne yapabileceğinin tek kaynakıdır.

## Adım 1: Tailscape kurun ve çalıştırın

```bash
brew install --cask tailscale
# Giriş yapın + tailscaled'i başlatın, ardından doğrulayın:
tailscale status
```

Daemon'un LocalAPI soketini okuyabildiğini doğrulayın:

```bash
test -S /var/run/tailscale.sock && echo "socket present" || echo "MISSING"
```

Eğer eksikse, daemon tailnet dinleyicisini açmayı reddeder (kapalı-hata).

## Adım 2: Daemon'un ACL'ini kurun

Daemon'un hangi Tailscale kimliklerine hangi cihazları hangi yetki katmanında
kontrol etme izni olduğunu bilmesi gerekir. İzin listesi dosyası JSON'dur:

```json
{
  "version": 1,
  "entries": [
    {
      "identity": "you@example.com",
      "capabilities": ["restore"],
      "expires_at": null,
      "note": "Sahip — tam erişim"
    },
    {
      "identity": "ci@example.com",
      "capabilities": ["mutate"],
      "expires_at": "2026-12-31T00:00:00Z",
      "note": "CI çalıştırıcısı — durum yazabilir ama tam geri yükleme yapamaz"
    },
    {
      "identity": "tag:claude-readonly",
      "capabilities": ["observe"],
      "expires_at": null,
      "note": "Yalnızca okuma yapması gereken ajanlar"
    }
  ]
}
```

Kimlikler WhoIs aracılığıyla kurallaştırılır:

- **Kullanıcı OAuth:** `user@example.com` (`acct:` yok, alan adı yeniden yazımı yok).
- **Etiketli düğümler:** `tag:<tagname>` (küçük harfe dönüştürülmüş).
- **Düğüm anahtarları:** `node:<nodekey-hex>` (nadir; bunun yerine etiketler kullanın).

Yetki katmanları sıralıdır: `observe` < `interact` < `mutate` < `restore`.
`restore` vermek, tüm alt katmanları da kapsar.

## Adım 3: Uzak ajan için oturum token'ı oluşturun

Ajanların kendi kendilerine token oluşturmasına izin verebilirsiniz (kimlikleri izin
listesindeyse) veya onlar için sunucu tarafında oluşturabilirsiniz:

```bash
# Sunucu tarafı oluşturma (yalnızca sahip, cihazlı Mac'te yerel olarak çalışır):
gstack-ios-qa-mint --remote ci@example.com --capability mutate --ttl 1h

# Kendi kendine oluşturma (tailnet üzerinden ajan):
curl -X POST http://<mac-tailnet-ip>:9999/auth/mint \
  -H "Content-Type: application/json" \
  -d '{"capability": "interact"}'
# → {"session_token": "...", "expires_at": "...", "capability": "interact"}
```

## Adım 4: Tailscale ACL'yi sıkılaştırın (derinlemesine savunma)

Daemon'un izin listesi birincil erişim kontrolüdür. Ek önlem olarak:
tailnet ACL'sini, daemon portuna kimin erişebileceğini sınırlandırmak için
kısıtlayın.

```jsonc
// Tailscale yönetici konsolunuzda:
{
  "acls": [
    // CI çalıştırıcısının iOS QA Mac'ine yalnızca 9999 portunda erişmesine izin ver.
    {
      "action": "accept",
      "src": ["ci@example.com"],
      "dst": ["ios-qa-mac:9999"]
    },
    // Etiketli Claude ajanları — yalnızca gözlem katmanı (daemon tarafından uygulanır, ACL değil).
    {
      "action": "accept",
      "src": ["tag:claude-readonly"],
      "dst": ["ios-qa-mac:9999"]
    },
    // Varsayılan reddet.
    {
      "action": "drop",
      "src": ["*"],
      "dst": ["ios-qa-mac:9999"]
    }
  ]
}
```

## Adım 5: Denetim izi

Tailnet dinleyicisi üzerinden kimliği doğrulanmış her değiştirici istek,
`~/.gstack/security/ios-qa-audit.jsonl` dosyasına bir satır yazar:

```jsonl
{"ts":"2026-05-18T14:23:00Z","identity":"ci@example.com","device_udid":"00008101-XXXX","endpoint":"/tap","session_id":"abc...","capability":"interact","request_id":"req_001","status":200}
```

Reddedilen istekler (token yok, süresi dolmuş token, yetersiz yetki, izin
listesinde olmayan kimlik, hız sınırına ulaşılan) `~/.gstack/security/attempts.jsonl`
dosyasına yazılır.

## Hız limitleri

- `/auth/mint`: kimlik başına 60 saniyede 10 oluşturma. 11.'si 429 döndürür.
- Tailnet isteği başına gövde: 1MB sabit sınır (üstünde 413).
- Ekran görüntüsü yanıtı: 10MB sabit sınır (üstünde 500, temizlenmiş hata ile).

## Token ömrü

- Daemon tarafından oluşturulan oturum token'ları: varsayılan 1s TTL, `--tailnet-session-ttl` ile en fazla 24s.
- `POST /session/heartbeat` ile yenilenebilir (orijinal maksimum ile sınırlandırılmış `ttl_seconds` kadar uzatır).
- Önyükleme token'ı (iOS uygulaması başlatma ile daemon rotasyonu arasında): ~5s ömrü — daemon ilk kazımada hemen döndürür.

## Hata durumları

| Belirti | Neden | Eylem |
|---|---|---|
| Daemon tailnet dinleyicisini açmayı reddediyor | `/var/run/tailscale.sock` eksik veya izin hatası | Tailscale'i kurun; daemon'u çalıştıran kullanıcıyla `tailscale status` çalıştığını doğrulayın |
| `403 identity_not_allowed` | kimlik izin listesinde yok | Sahip oluşturma: `gstack-ios-qa-mint --remote <identity>` |
| `403 capability_insufficient` | token katmanı uç nokta gereksiniminin altında | Daha yüksek `--capability` katmanı ile sahip oluşturma |
| `429 rate_limited` | bir kimlikten >10 oluşturma/dakika | 60 saniye bekleyin; ajanın neden bu kadar sık yeniden oluşturduğunu araştırın |
| `/state/restore` üzerinde `409 schema_mismatch` | eski uygulama derlemesinden snapshot | Snapshot'ı atın; mevcut uygulama derlemesinden yeniden yakalayın |