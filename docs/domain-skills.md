# Alan Yetenekleri

Aracının kendi yazdığı site bazlı notlar. Oturumlar boyunca birikir: bir aracı bir
web sitesi hakkında bariz olmayan bir şey keşfettiğinde, bir yetenek kaydeder ve
gelecek oturumlarda o host adı için not komut istemine bağlam olarak enjekte edilir.

Bu, gstack'in [browser-use/browser-harness](https://github.com/browser-use/browser-harness)
projesinden ödünç aldığı kısımdır. gstack site bazındaki notlar örüntüsünü kopyalar,
kendi kendini değiştiren çalışma zamanı örüntüsünü KOPYALAMAZ. Yetenekler komut
istemlerine yüklenen markdown metnidir; çalıştırılabilir kod değildir.

## Aracılar nasıl kullanır

```bash
# Aracı başarılı bir görevden sonra bir site hakkında öğrendiklerini yazdı.
# Host adı etkin sekmeden otomatik olarak alınır (aracı argümanı yoktur).
echo "# LinkedIn Başvuru Düğmesi

/jobs/view sayfalarındaki Başvuru düğmesi, 'jobs-apply-button-iframe'
sınıfıyla eşleşen bir iframe içindedir. Önce \$B frame --url 'apply' kullanın,
ardından anlık görüntü alın." | $B domain-skill save

# Nelerin kaydedildiğini gör
$B domain-skill list

# Belirli bir hostun yeteneğinin gövdesini oku
$B domain-skill show linkedin.com

# $EDITOR içinde etkileşimli olarak düzenle
$B domain-skill edit linkedin.com

# Etkin bir proje bazlı yeteneği global olana yükselt (projeler arası)
$B domain-skill promote-to-global linkedin.com

# Son bir düzenlemeyi geri al
$B domain-skill rollback linkedin.com

# Sil (mezar taşı — rollback ile kurtarılabilir)
$B domain-skill rm linkedin.com
```

## Durum makinesi

```
  ┌──────────────┐  3 başarılı kullanım        ┌────────┐  promote-to-global   ┌────────┐
  │ karantinada  │ ─────────────────────▶  │ etkin  │ ──────────────────▶  │ global │
  │ (proje bazlı)│  (sınıflandırıcı bayrağı yok)   │(proje)│  (el ile komut)    │        │
  └──────────────┘                          └────────┘                      └────────┘
         ▲                                       │
         │  kullanım sırasında sınıflandırıcı bayrağı           │  rollback (sürüm günlüğü)
         └───────────────────────────────────────┘
```

Yeni bir kayıt **karantinaya** düşer ve komut istemlerinde otomatik olarak çalışmaz. Bu
host adında L4 ML sınıflandırıcısı yetenek içeriğini işaretlemeden 3 başarılı
kullanımdan sonra, yetenek otomatik olarak projede **etkin** duruma yükseltilir. Etkin
yetenekler o host adı için her yeni kenar çubuğu-aracı oturumunda çalışır.

Bir yeteneği projeler arası çalıştırmak için (örneğin, "LinkedIn yeteneğimin üzerinde
çalıştığım her gstack projesinde olmasını istiyorum"), açıkça
`$B domain-skill promote-to-global <host>` çalıştırın. Bu tasarım gereği katılım
tabanlıdır (Codex T4 dış-ses incelemesi): genel projeler arası birikim, ilgisiz işlerde
bağlam sızıntısına neden olur.

## Depolama

Yetenekler iki yerde bulunur:

- **Proje bazlı**: `~/.gstack/projects/<slug>/learnings.jsonl` — `/learn` yeteneğinin
  kullandığı aynı JSONL dosyası. Alan yetenekleri `type:"domain"` satırlarıdır.
- **Global**: `~/.gstack/global-domain-skills.jsonl` — yalnızca `state:"global"`
  satırları.

Her iki dosya da yalnızca ekleme yapılabilen JSONL dosyalarıdır. Silmeler için mezar
taşları; boşta bir sıkıştırıcı dosyaları periyodik olarak yeniden yazar. Hoşgörülü
ayrıştırıcı, okuma sırasında kısmi sondaki satırları atar, böylece yazma ortasında
bir çökme sonraki okumaları bozmaz.

## Güvenlik modeli

Yetenekler, gelecekteki komut istemi bağlamına yüklenen aracı tarafından yazılmış
içeriktir. Bu onları klasik bir aracıdan-aracıya istem enjeksiyonu vektörü yapar. Plan,
birden fazla katmanla bunu açıkça ele alır:

| Katman | Ne | Nerede |
|-------|------|-------|
| L1-L3 | Veri işaretleme, gizli öğe çıkarma, ARIA düzenli ifadesi, URL engelleme listesi | `content-security.ts` (derlenmiş ikili dosya) |
| L4 | TestSavantAI ONNX sınıflandırıcısı | `security-classifier.ts` (kenar çubuğu-aracısı, derlenmemiş) |
| L4b | Claude Haiku transkript sınıflandırıcısı | `security-classifier.ts` (kenar çubuğu-aracısı) |
| L5 | Kanarya belirteç sızıntı algılama | `security.ts` |

L1-L3 denetimleri **kaydetme zamanında** (artalan sürecinde) çalışır. L4 ML
sınıflandırıcısı **yükleme zamanında** (kenar çubuğu-aracısında) çalışır, böylece bir
yetenek komut istemine yükleyen her oturum da içeriği yeniden doğrular. Bu, yalnızca
bir sınıflandırıcı modeli güncellemesinden sonra ortaya çıkan sorunları yakalar.

Kaydetme komutu, host adını **etkin sekmenin üst düzey kaynağından** türetir, aracı
argümanlarından değil. Bu, Codex'in işaretlediği kafa karışıklığı-vekil hatasını
kapatır: kötü niyetli bir sayfa yeniden yönlendirme zinciri aksi takdirde aracıyı
farklı bir alanı zehirlemeye kandırabilirdi.

## Hata referansı

| Hata | Neden | Eylem |
|-------|-------|--------|
| `Save blocked: classifier flagged content as potential injection` | Kaydetme sırasında L4 puanı ≥ 0.85 | Yetenekten talimat benzeri düzyazıyı çıkararak yeniden yazın; tekrar deneyin. |
| `Save blocked: <L1-L3 message>` | Kaydetme sırasında URL engelleme listesi eşleşmesi veya ARIA enjeksiyonu | Yetenek gövdesini şüpheli kalıplar için inceleyin. |
| `Save failed: empty body` | stdin veya `--from-file` üzerinden içerik yok | Markdown'ı `$B domain-skill save` komutuna yöneltin veya `--from-file <yol>` geçirin. |
| `Cannot save domain-skill: no top-level URL on active tab` | Sekme `about:blank` veya `chrome://...` | Önce `$B goto <hedef-site>`, ardından kaydedin. |
| `Cannot promote: skill is in state "quarantined"` | Yetenek henüz otomatik olarak yükseltilmedi | Sınıflandırıcı bayrakları olmadan 3 başarılı çalışmaya kadar bu projede kullanın. |
| `Cannot rollback: <host> has fewer than 2 versions` | Yalnızca bir sürüm var | Bunun yerine silmek için `$B domain-skill rm` kullanın. |

## Telemetri

Telemetri etkinleştirildiğinde (kapatılmadıkça varsayılan `community` modu), şu
olaylar `~/.gstack/analytics/browse-telemetry.jsonl` dosyasına yazılır:

- `domain_skill_saved {host, scope, state, bytes}`
- `domain_skill_save_blocked {host, reason}`
- `domain_skill_fired {host, source, version}`
- `domain_skill_state_changed {host, from_state, to_state}` (planlanıyor)

Yalnızca host adı — gövde içeriği yok, aracı metni yok. Tamamen devre dışı bırakmak için
`gstack-config set telemetry off` veya `GSTACK_TELEMETRY_OFF=1` kullanın.