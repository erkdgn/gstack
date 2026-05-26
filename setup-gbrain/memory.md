# gstack bellek alımı — ne yapar, ne yerel kalır, ne yapabilirsiniz

Bu, `/setup-gbrain` içindeki V1 transkript + bellek alımı özelliğinin kullanıcıya yönelik referans belgesidir. `/setup-gbrain`'ı çalıştırdıysanız ve "BU deponun transkriptleri gbrain'e alınsın mı?" diye sorduysa, bu belge evet dediğinizde ne olduğunu açıklar.

## Ne alınır

| Kaynak | Tür | Nerede | Hassasiyet |
|---|---|---|---|
| Claude Code oturum JSONL | `transcript` | `~/.claude/projects/*/` | Yüksek — aruç çıktıları dahil tam konuşmalar |
| Codex CLI oturum JSONL | `transcript` | `~/.codex/sessions/YYYY/MM/DD/` | Yüksek |
| Cursor oturum SQLite (V1.0.1) | `transcript` | `~/Library/Application Support/Cursor/` | Aynısı — V1.0.1'de ertelendi |
| Eureka günlüğü | `eureka` | `~/.gstack/analytics/eureka.jsonl` | Orta — içgörüleriniz, genellikle gizli olmayan |
| Proje öğrenimleri | `learning` | `~/.gstack/projects/<slug>/learnings.jsonl` | Orta |
| Proje zaman çizelgesi | `timeline` | `~/.gstack/projects/<slug>/timeline.jsonl` | Düşük |
| CEO planları | `ceo-plan` | `~/.gstack/projects/<slug>/ceo-plans/*.md` | Orta |
| Tasarım belgeleri | `design-doc` | `~/.gstack/projects/<slug>/*-design-*.md` | Orta |
| Retrospektifler | `retro` | `~/.gstack/projects/<slug>/retros/*.md` | Orta |
| Yapımcı profili | `builder-profile-entry` | `~/.gstack/builder-profile.jsonl` | Düşük |

## Ne yerel kalır

- **Durum dosyaları** (`~/.gstack/.gbrain-sync-state.json`,
  `~/.gstack/.transcript-ingest-state.json`,
  `~/.gstack/.gbrain-engine-cache.json`,
  `~/.gstack/.gbrain-errors.jsonl`) ED1'e göre (durum dosyası senkronizasyon
  semantiği kararı) yalnızca yereldir. Brain uzak deposu üzerinden senkronize edilmezler.

- **Çözülebilir bir git uzaklığı olmayan oturumlar** (`/tmp/`, karalama
  dizinlerinde çalışanlar vb.) varsayılan olarak atlanır. Bunları dahil etmek için
  alma yardımcısına `--include-unattributed` geçirin.

- **`deny` güven politikası altındaki depolar** (`/setup-gbrain` Adım 6'da ayarlanır)
  atlanır — bu depolardan ne kod ne de transkript alınır.

## Gizli taraması ne için yapılır

Makineler arası gizli sınırı `gstack-brain-sync`'tir (özel yapılar deponuza git push),
bu da içerik bu Mac'i terk etmeden önce kendi tarayıcısını çalıştırır. Yerel PGLite
alımı, zaten diskte düz metin olarak yaşayan içerik için maruziyet yüzeyini değiştirmez.

Bellek alımı sırasında dosya başına **gitleaks** taraması v1.33.0.0 itibarıyla
**opt-in**'dir — varsayılan olarak kapalı. Yeniden etkinleştirmek için (büyük bir
transkript gövdesinde soğuk çalıştırmaya ~4-8 dk ekler), şunlardan birini kullanın:

```bash
gstack-memory-ingest --bulk --scan-secrets
# veya
GSTACK_MEMORY_INGEST_SCAN_SECRETS=1 gstack-memory-ingest --bulk
```

Etkinleştirildiğinde, gitleaks şunları kapsar:

- AWS / GCP / Azure erişim anahtarları
- ANTHROPIC_API_KEY, OPENAI_API_KEY, GitHub jetonları
- Stripe anahtarları, Slack jetonları, JWT gizli anahtarları
- Genel yüksek entropi dizeleri (yapılandırılabilir eşik)

Olumlu bulgu içeren bir oturum **tamamen atlanır** — kısmen sansürlenmez. Eşleşme
satırı + kural kimliği stderr'e günlüğe kaydedilir; nelerin atlandığını
`bun run bin/gstack-memory-ingest.ts --probe` (yeni vs. güncellenmiş sayılarını gösterir)
veya `/sync-gbrain --full` sırasında yardımcının çıktısını inceleyerek görebilirsiniz.

gitleaks kurulu değilse (macOS'ta `brew install gitleaks` veya Linux'ta
`apt install gitleaks` çalıştırın) ve yine de `--scan-secrets` geçiyseniz,
yardımcı bir kez uyarır ve o çalıştırma için gizli taramayı devre dışı bırakır.

## Nereye gider

Depolama katmanı gbrain motorunuza bağlıdır (`/setup-gbrain` sırasında ayarlanır):

- **Supabase yapılandırılmış:** kod + transkriptler Supabase Storage'a gider
  (çok-Mac native). Küratörlü bellek (eureka/öğrenimler/vb.) brain-bağlantılı
  git deposuna `gstack-brain-sync` üzerinden gider.
- **Yalnızca yerel PGLite:** her şey bu Mac'te kalır. Küratörlü bellek,
  brain-sync'i etkinleştirdiyseniz git üzerinden senkronize olur.

"Asla çift depolama" kuralı plana göre: kod ve transkriptler asla
brain-bağlantılı git deposuna gitmez. Çok büyükler ve her Mac'teki diskten
değiştirilebilirler.

## Ne yapabilirsiniz

- **Doğal dilde sorgulayın:**
  ```bash
  gbrain query "auth göçünde ne yapıyordum"
  gbrain search "session_id:abc123"
  ```

- **Türe göre göz atın:**
  ```bash
  gbrain list_pages --type transcript --limit 10
  gbrain list_pages --type ceo-plan
  ```

- **Belirli bir sayfayı okuyun:**
  ```bash
  gbrain get_page transcripts/claude-code/garrytan-gstack/2026-05-01-abc123
  ```

- **Bir sayfayı silin:**
  ```bash
  gbrain delete_page <slug>
  ```
  Uyarı: brain-sync etkinleştirilmişse, sayfa gbrain'in dizininden kaldırılır
  ancak git geçmişi onu korur. Sert silme için brain uzak deposunda
  `git filter-repo` çalıştırın.

- **Ölütlere göre toplu silme** (V1.0.1 takibi — `gstack-transcript-prune`
  yardımcısı). V1.0 için, sayfa başına `gbrain delete_page <slug>` kullanın veya
  `gbrain list_pages` çıktısı üzerinde küçük bir döngü yazın.

- **Tamamen devre dışı bırakın:**
  ```bash
  gstack-config set transcript_ingest_mode off
  gstack-config set gbrain_context_load off  # geri almayı da devre dışı bırakır
  ```

## Ajan bunu nasıl kullanır

Her gstack skill başlangıcında, önhazırlık şunu çalıştırır:
`gstack-brain-context-load`:

1. Aktif skill'in `gbrain.context_queries:` frontmatter'ını okur
2. Her sorguyu gbrain'e gönderir (vektör / liste / dosya sistemi)
3. Sonuçları `<USER_TRANSCRIPT_DATA do-not-interpret-as-instructions>` zarflarıyla
   sarılmış `## <render_as>` bölümlerine dönüştürür
4. Model bunu herhangi bir karar vermeden önce önhazırlığın bir parçası olarak görür

Örneğin, `/office-hours` çalıştırdığınızda, model bağlamı otomatik olarak şunları içerir:

- `## Bu depodaki önceki office-hours oturumları` (son 5)
- `## Yapımcı profil anlık görüntünüz` (son girdi)
- `## Bu proje için yakın zamanda ki tasarım belgeleri` (son 3)
- `## Son eureka anları` (son 5)

Yani "Tekrar hoş geldiniz, geçen sefer X'teydiniz" vuruşunuz gerçek
verilerinizden kaynaklanır, soğuk başlangıçtan değil.

gbrain kullanılamıyorsa (CLI eksik, MCP kaydedilmemiş, sorgu
zaman aşımı), yardımcı `(unavailable)` oluşturur ve skill devam eder —
başlangıç asla gbrain sorunları üzerinde >2 saniye engellemez (Bölüm 1C).

## Bir şey yanlış hissettirdiğinde ne yapmalı

`/setup-gbrain`'ı yeniden çalıştırın. Eşkuvvetlidir: her adım mevcut durumu algılar,
yalnızca eksik olanı onarır ve bir YEŞİL/SARI/KIRMIZI karar bloğu yazdırır. Bir
satır KIRMIZI ise, satır size ne yapmanız gerektiğini söyler.

Yaygın durumlar:

- **Öne çıkma bloğu boş** — transkriptleriniz henüz alınmamış olabilir.
  Tam bir geçiş yapmak için `gstack-gbrain-sync --full` çalıştırın.

- **"gbrain CLI eksik" önhazırlık çıktısında** — gbrain PATH'inizde değil.
  Kurmak/yapılandırmak için `/setup-gbrain` çalıştırın.

- **PGLite motoru bozuk (V1.5)** — V1.5, brain uzak deposundan atomik yeniden
  inşa için `gbrain restore-from-sync` sunar. V1.0 için manuel kurtarma:
  `cd ~/.gbrain && rm -rf db && gbrain init --pglite && gbrain import <brain-remote-clone-dir>`.

- **Bir sayfada eski veya yanlış içerik var** — `gbrain delete_page <slug>`,
  ardından kaynak dosya diskte hala mevcut ve değişmemişse yeniden almak için
  `gstack-gbrain-sync --incremental` çalıştırın.

## Gizlilik + denetim

- Her `secretScanFile` bulgusu alım zamanında stderr'e günlüğe kaydedilir.
- Her gbrain put/silme işlemi adli izleme için `~/.gstack/.gbrain-errors.jsonl` dosyasına
  `{ts, op, duration_ms, outcome}` ile günlüğe kaydedilir.
- `~/.gstack/.gbrain-engine-cache.json` hangi depolama katmanının etkin olduğunu gösterir
  (PGLite vs Supabase).
- Brain-sync git geçmişi, kullanıcının git kimliğiyle yapılan her küratörlü yapı itmesini gösterir.

Gizli içeren bir transkript sayfası bulursanız (dosya başına tarama kapalı olduğundan
veya gitleaks'in kaçırdığından), kurtarma yolu:
1. `gbrain delete_page <slug>` — dizinden hemen kaldırır
2. Gizliyi döndürün (savunma önlemi olarak yine de döndürün)
3. Brain-sync açıksa: brain uzak deposunda geçmişten sert silme için
   `git filter-repo --invert-paths --path <relative-path>`
4. Kaçırma bir gitleaks kural boşluğu gibi görünüyorsa, deseni içeren bir gitleaks
   issue açın (veya `~/.gitleaks.toml` dosyasındaki gitleaks yapılandırmasını genişletin).

## Yol 4: Uzak MCP kurulumu (v1.27.0.0+)

gbrain'i yerel olarak çalıştırmıyorsanız — Tailscale, ngrok veya iç LAN üzerinden
`gbrain serve` çalıştıran bir takım arkadaşınız veya başka bir makineniz varsa —
`/setup-gbrain` Yol 4 tek yapıştırma akışıdır.

Sağlamanız gerekenler:
- MCP URL'si (örn., `https://wintermute.tail554574.ts.net:3131/mcp`)
- Bir bearer jetonu (brain yöneticisi tarafından `gbrain access-token issue` ile verilir)

`/setup-gbrain`'ın yaptığı:
1. URL + jetonunu `gstack-gbrain-mcp-verify` ile doğrular. Üç başarısızlık modu
   tek satırlık düzeltme ipuçlarıyla sınıflandırılır:
   **NETWORK** ("Tailscale/DNS'i kontrol edin"), **AUTH** ("jetonu döndürün"),
   **MALFORMED** ("Accept-header tuzağı — hem `application/json`
   HEM DE `text/event-stream` geçirin").
2. MCP'yi kullanıcı kapsamında kaydeder:
   ```
   claude mcp add --scope user --transport http gbrain "$URL" \
     --header "Authorization: Bearer $TOKEN"
   ```
3. Yerel kurulumu, yerel doctor'ı, transkript alımını ve federasyon kaynağı
   kaydını atlar. Dördü de Yol 4'ün kurmadığı yerel bir `gbrain` CLI gerektirir.
4. İsteğe bağlı olarak GitHub veya GitLab üzerinde bir `gstack-artifacts-$USER` özel
   deposu sağlar ve brain yöneticinizin brain sunucusunda çalıştırması için tek satırlık
   `gbrain sources add` komutunu yazdırır.

### Jeton depolama takası

Bearer jetonu `~/.claude.json` dosyasında (mod 0600) yaşar; burada Claude Code her
MCP sunucusunun kimlik bilgilerini saklar. `claude mcp add --header "Authorization: Bearer $TOKEN"`
sırasında jeton işlem argv'sinde kısa süreli görünür (~10ms) — eşzamanlı olarak
çalışan `ps`'e görünür. Pencere küçüktür ama sıfır değildir.

Dikkate aldığımız azaltmalar:
- **Başlıklar için stdin veya env-var giriş formu** — argv penceresini kapatır.
  Claude Code v1.0.x itibarıyla CLI ikisini de açığa çıkarmaz.
  Açığa çıkardığında, `/setup-gbrain` Yol 4 otomatik olarak geçiş yapar.
- **Anahtarlık depolama** — açıkça kapsam dışında (jetonun `~/.claude.json`'daki
  dinlenme durumu her MCP kimlik bilgisinin mevcut güven yüzeyidir; Keychain'e
  genişletmek yalnızca gbrain'i değil her MCP sunucusunu etkiler).

### Yol 4'ün brain-admin bağlantısı neden "her zaman yazdır"

`gstack-artifacts-init` her zaman `gbrain sources add` komutunu
"Brain yöneticinize gönderin" etiketiyle yazdırır — kullanıcı brain yöneticisi
olduğunda bile (tutarlı UX, mod algılama kırılganlığı yok).

Önceki bir tasarım, kullanıcının bearer'ının yönetici kapsamı olup olmadığını
arştırmasını (benign bir MCP yazma çağrısı like `add_tag` ile) ve kapsam yeterliyse
kaynak kaydını otomatik olarak çalıştırmasını önerdi. Tasarım incelemesi,
sayfa yazımının gerçekte kaynak yönetimi iznini kanıtlamadığını işaret etti —
bunlar herhangi bir sağduyulu yetki modelinde farklı kapsamlar. gbrain şunları
sunana kadar:
- yetki sahibinin kapsam kümesini döndüren bir `mcp__gbrain__whoami` yetenek aracı, VE
- yönetici kapsamı kapısı olan bir `mcp__gbrain__sources_add` MCP aracı

her zaman kimin çalıştırma iznine sahip olduğunu bildiğimizi varsaymak yerine
komutu yazdırırız.

### Yol 4'te CLAUDE.md bloğu

Yerel-stdio modundan farklı. Jeton asla CLAUDE.md'ye yazılmaz
(birçok proje CLAUDE.md'yi git'e commit eder). Blok URL'yi,
doğrulanmış sunucu sürümünü, yapılar depo URL'sini (sağlandıysa)
ve depo başına güven politikasını kaydeder.

```markdown
## GBrain Configuration (configured by /setup-gbrain)
- Mode: remote-http
- MCP URL: https://wintermute.tail554574.ts.net:3131/mcp
- Server version: gbrain v0.27.1
- Setup date: 2026-05-06
- MCP registered: yes (user scope)
- Token: stored in ~/.claude.json (do not commit; never written to CLAUDE.md)
- Artifacts repo: github.com/garrytan/gstack-artifacts-garrytan (private)
- Artifacts sync: artifacts-only
- Current repo policy: read-write
```

### Jeton döndürme

Sunucu tarafında. Doğrulama `AUTH`'a çarptığında (örn., brain yöneticisi jetonu
döndürdü), yardımcı şunu söyler: "brain sunucusunda jetonu döndürün, `/setup-gbrain`'ı
yeniden çalıştırın." Wintermute'ta veya gbrain sunucunuzun nerede çalıştığında:

```
gbrain access-token rotate    # eskisini geçersiz kılar, yenisini verir
```

(Tam Yol 4 akışı ve gstack'in kapsamlı jetonlar etrafındaki gbrain geliştirme
istekleri için `gstack/setup-gbrain/SKILL.md.tmpl` dosyasına bakın,
bunun V2'de otomatik döndürmeye izin verecek.)