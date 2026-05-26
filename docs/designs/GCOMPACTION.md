# GCOMPACTION.md — Tasarım ve Mimari (ASKIYA ALINDI)

**Onay sonrası hedef yol:** `docs/designs/GCOMPACTION.md`

Bu, `gstack compact` için korunmuş tasarım eseridir. İlk `---` ayırıcısının üstündeki her şey, plan onayında `docs/designs/GCOMPACTION.md`'ye kelimesi kelimesine çıkarılır. Bu ayırıcıdan sonraki her şey, tasarımı bilgilendiren arşivlenmiş araştırmadır (office hours + rekabetçi derinlemesine inceleme + mühendislik inceleme notları + codex incelemesi + araştırma bulguları).

---

## Durum: ASKIYA ALINDI (2026-04-17) — Anthropic `updatedBuiltinToolOutput` API'sini bekliyor

**Neden askıya alındı.** v1 mimarisi, bir Claude Code `PostToolUse` hook'unun yerleşik araçlar (Bash, Read, Grep, Glob, WebFetch) için modelin bağlamına giren araç çıktısını DEĞİŞTİREBİLECEĞİNİ varsayıyordu. 2026-04-17 tarihindeki araştırma, bunun bugün mümkün olmadığını doğruladı.

**Kanıt:**

1. **Resmi dokümantasyon** (https://code.claude.com/docs/en/hooks): `PostToolUse` için belgelenen tek çıktı-değiştirme alanı `hookSpecificOutput.updatedMCPToolOutput` ve dokümantasyon açıkça şunu belirtiyor: *"Yalnızca MCP araçları için: sağlanan değerle aracın çıktısını değiştirir."* Yerleşik araçlar için eşdeğer bir alan mevcut değil.
2. **Anthropic sorun [#36843](https://github.com/anthropics/claude-code/issues/36843)** (AÇIK): Anthropic kendisi boşluğu kabul ediyor. *"PostToolUse hook'ları `updatedMCPToolOutput` aracılığıyla MCP araç çıktısını değiştirebilir, ancak yerleşik araçlar (WebFetch, WebSearch, Bash, Read vb.) için eşdeğer bir alan yoktur... Yalnızca `decision: block` (bir neden dizesi enjekte eder) veya `additionalContext` ile uyarı ekleyebilirler. Orijinal kötü amaçlı içerik hala modele ulaşır."*
3. **RTK mekanizması** (kaynak incelendi: `src/hooks/init.rs:906-912` ve `hooks/claude/rtk-rewrite.sh:83-100`): RTK bir PostToolUse sıkıştırıcı DEĞİLDİR. `tool_input.command`'u yeniden yazan bir **PreToolUse** Bash eşleştiricisidir (örn., `git status` → `rtk git status`). Sarılmış komut sıkıştırılmış stdout'u kendisi üretir. RTK README'si doğrular: *"hook yalnızca Bash araç çağrılarında çalışır. Read, Grep ve Glob gibi Claude Code yerleşik araçları Bash hook'undan geçmez, bu yüzden otomatik olarak yeniden yazılmaz."* RTK mimari kısıtlama nedeniyle yalnızca Bash'tir, tercih nedeniyle değil.
4. **tokenjuice mekanizması** (kaynak incelendi: `src/core/claude-code.ts:160, 491, 540-549`): tokenjuice `PostToolUse`'ı `matcher: "Bash"` ile kaydeder, ancak kullanılabilir gerçek çıktı-değiştirme API'si yoktur — sıkıştırılmış metin enjekte etmek için `decision: "block"` + `reason`'ı ele geçirir. Bunun gerçekten model-bağlam token'larını azaltıp azaltmadığı veya yalnızca UI çıktısının üzerine bindiği tartışmalıdır. tokenjuice de yalnızca Bash'tir.
5. **Read/Grep/Glob Claude Code içinde işlem içi yürütülür** ve hook'lari tamamen atlar. Kama (ii) "yerel-arac kapsama" alanı, değiştirme API'sinden bağımsız olarak mimari açıdan ilk günden imkansızdı.

**Sonuç.** Her iki kama da orijinal biçimlerinde ölü:
- Kama (i) "Koşullu LLM doğrulayıcı" — hala teknik olarak mümkün, ancak yalnızca Bash çıktısı için, PreToolUse komut sarmalama yoluyla (RTK'nın mekanizması). Aynı zamanda Bash-özel olduğumuzda doğrulayıcı farklılaştırıcı olmaktan çıkar.
- Kama (ii) "Yerel araç kapsama" — bugün imkansız. Read/Grep/Glob hook'ları tetiklemez. Tetiklese bile, MCP olmayan araçlar için çıktı-değiştirme alanı mevcut değil.

**Karar.** `gstack compact`'ı tamamen rafa kaldır. Anthropic sorun #36843'ün `updatedBuiltinToolOutput` (veya eşdeğeri) gelmesini takip et. Bu API geldiğinde, bu tasarım dokümanı + aşağıdaki kilitlenmiş 15 karar + alttaki araştırma arşivi, yeni bir uygulama sprinti için engellemeyi kaldıran eserler olacak.

**Askıya kaldırmadan devam etmek için:** Aşağıdaki "Plan-mühendislik-incelemesi sırasında kilitlenen kararlar" bloğundan başlayın — çoğu hala geçerli. Ardından hook'lar referansını yeni gönderilen API'ye karşı yeniden doğrulayın, Mimari veri akış diyagramını hangi gerçek çıktı-değiştirme alanı varsa onu kullanacak şekilde güncelleyin, ve kodlamadan önce düzeltilmiş plana karşı `/codex review`'u yeniden çalıştırın.

**Yapmayacağımız şeyler:**
- Yalnızca Bash için bir PreToolUse sarmalayıcı göndermiyoruz. Bu RTK'nın ürünü; 28K yıldız ve 3 yıl kural yaraları var. Kama yok.
- `decision: block` + `reason` hack'ini göndermiyoruz. Belgelenmemiş davranış, Anthropic bozabilir, ve model sıkıştırılmış katmanın yanında ham çıktıyı hala görebilir — bağlam tasarrufları tartışmalı.
- B-serisi kıyaslama testini tek başına göndermiyoruz. Çalışan bir sıkıştırıcı olmadan, kıyaslanacak bir şey yok.

**Askıya almanın maliyeti:** ~0. Hiçbir kod yazılmadı. Tasarım dokümanı + araştırma + kararlar, hazır-engellemeyi-kaldır eseri olarak kalır.

---

## Plan-mühendislik-incelemesi sırasında kilitlenen kararlar (2026-04-17)

Anthropic yerleşik-aracı çıktı-değiştirme API'sini gönderdiğinde askıya kaldırmadan çıkma sprinti için korunmuştur.

Mühendislik incelemesi sırasında alınan her kararın özeti. Tam gerekçeler aşağıdaki bölümler boyunca korunmuştur; bu blok, başka bir şey kayması durumunda tek doğruluk kaynağıdır.

**Kapsam (Bölüm 0):**
1. **Claude-öncelikli v1.** compact + rules + doğrulayıcı'yı yalnızca Claude Code üzerinde gönder. Codex + OpenClaw v1.1'de kama birincil ana bilgisayarda kanıtlandıktan sonra gelir. ~2 gün ana bilgisayar entegrasyonunu keser ve lansman riskini azaltır. Orijinal "kama (ii) yerel-arac kapsama" iddiası v1'de Claude Code için geçerlidir; v1.1'e kadar ana bilgisayarlar arası iddia yapmayız.
2. **13-kural lansman kütüphanesi.** v1 testler (jest/vitest/pytest/cargo-test/go-test/rspec) + git (diff/log/status) + kurulum (npm/pnpm/pip/cargo) gönderir. Derleme/lint/günlük aileleri, gerçek kullanıcıların `gstack compact discover` telemetrisinden yönlendirilen v1.1'e ertelenir.
3. **Doğrulayıcı v1.0'da varsayılan olarak AÇIK.** `failureCompaction` tetikleyicisi (çıkış≠0 VE >%50 azalma) kutudan çıkar. Doğrulayıcı KAMADIR — varsayılan olarak kapalı olmak farklılaştırıcı özelliği gizler. Tetikleyici sınırları zaten beklenen tetiklenme oranını araç çağrılarının ≤%10'unda tutar.

**Mimari (Bölüm 1):**
4. **Haiku çıktısı için tam satır eşleşme temizleme.** Ham çıktıyı `\n` ile böl, satırları bir kümeye koy, yalnızca Haiku'dan o kümede kelimesi kelimesine görünen satırları ekle. En sıkı düşmanca sözleşme; prompt enjeksiyon denemeleri yeni metin sızdıramaz.
5. **Katmanlı failureCompaction sinyali.** Zarfın `exitCode` alanını tercih et; ana bilgisayar atarsa, çıktı üzerinde `/FAIL|Error|Traceback|panic/` regex'e geri dön. Hangi sinyalin tetiklendiğini `meta.failureSignal`'da günlükle ("exit" | "pattern" | "none"). Uygulama öncesi görev #1 hala Claude Code'un zarfını ampirik olarak doğrular, ancak sistem artık bununla kırılmaz.
6. **Derin birleştirme kural çözümlemesi.** Kullanıcı/proje kuralları geçersiz kılmadıkları yerleşik alanları miras alır. Kaçış yolu: bir kural dosyasında `"extends": null` tam değiştirme semantiğini tetikler. eslint/tsconfig/.gitignore zihinsel modeliyle eşleşir — parçayı override et, geri kalanını kaybetmeden.

**Kod kalitesi (Bölüm 2):**
7. **Kural başına regex zaman aşımı, RE2 bağımlılığı yok.** Her kuralın regex'ini 50ms AbortSignal bütçesiyle çalıştır; zaman aşımında kuralı atla ve `meta.regexTimedOut: [ruleId]` kaydet. Bir WASM bağımlılığından kaçınır ve kural yazarı sözdizimini sınırlamaz.
8. **Önceden derlenmiş kural paketi.** `gstack compact install` ve `gstack compact reload`, `~/.gstack/compact/rules.bundle.json` üretir (derin birleştirilmiş, regex derlenmiş meta veriler önbelleğe alınmış). Hook N kaynak dosyasını ayrıştırmak yerine o tek dosyayı okur.
9. **mtime sapması durumunda otomatik yeniden yükleme.** Hook başlangıçta kural kaynak dosyalarının istatistiklerini alır; herhangi bir kaynak dosyası paketten daha yeniyse, uygulamadan önce satır içi yeniden oluşturur. ~0.5ms/çağrı ekler ama "bir kuralı düzenledim ve hiçbir şey değişmedi" ayak tabancasını ortadan kaldırır.
10. **Genişletilmiş v1 sansür seti.** Tee dosyaları sansürler: AWS anahtarları, GitHub token'ları (`ghp_/gho_/ghs_/ghu_`), GitLab token'ları (`glpat-`), Slack webhook'ları, genel JWT (üç base64 segment), genel bearer token'ları, SSH özel anahtar başlıkları (`-----BEGIN * PRIVATE KEY-----`). Kredi kartları / SSN'ler / anahtar başına çevre değişken çiftleri v2'de tam bir DLP katmanına ertelenir.

**Test (Bölüm 3):**
11. **P-serisi geçit alt kümesi.** v1 geçit katmanı P-testleri: P1 (ikili çöp), P3 (boş çıktı), P6 (RTK-katil kritik yığın çerçevesi), P8 (tee'ye gizli anahtarlar), P15 (hook zaman aşımı), P18 (prompt enjeksiyonu), P26 (bozuk kullanıcı kuralı JSON'u), P28 (regex DoS), P30 (Haiku halüsinasyonu). Kalan 21 P-durumu, gerçek hatalar isabet ettikçe R-serisine büyür.
12. **Fixture sürüm damgalama.** Her altın fixture'ın bir `toolVersion:` ön maddesi var. CI, fixture toolVersion ≠ kurulu sürüm olduğunda uyarır. Takvim tabanlı döndürme yok.
13. **B-serisi gerçek dünya kıyaslama test düzeneği (sert v1 geçidi).** Yeni bileşen `compact/benchmark/`, `~/.claude/projects/**/*.jsonl`'ı tarar, en gürültülü araç çağrılarını sıralar, onları adlandırılmış senaryolarda kümelendir, sıkıştırıcıyı bunlara karşı yeniden oynatır ve kural ailesine göre azaltma raporlar. Yazarın kendi 30 günlük corpus'unda B-serisi ≥%15 azalma VE dikili hata senaryolarında sıfır kritik satır kaybı gösterene kadar v1 gönderilemez. Yalnızca yerel; hiçbir zaman yüklemez. Topluluk paylaşımlı corpus v2'dir.

**Performans (Bölüm 4):**
14. **Düzeltilmiş gecikme bütçeleri.** macOS ARM üzerinde Bun soğuk başlangıcı 15-25ms; orijinal 10ms p50 hedefi gerçekçi değildi. Yeni bütçeler: macOS ARM'de <30ms p50 / <80ms p99, Linux'ta (doğrulayıcı kapalı) <20ms p50 / <60ms p99. Doğrulayıcı-tetiklenme bütçesi <600ms p50 / <2s p99. Arka plan programı modu, B-serisi soğuk başlangıcın oturum tasarruflarını anlamlı şekilde etkilediğini gösterdiği takdirde v2 seçeneğidir.
15. **Satır yönelimli akış hattı.** stdin üzerinden Readline → filtre → grup → çiftleri kaldır → halka tamponlu kuyruk kırpması → stdout. 1MB'tan büyük herhangi bir tek satır P9'a çarpar (1KB'ye `[... truncated ...]` işaretiyle kırpar). Toplam çıktı boyutundan bağımsız olarak belleği 64MB'ta sınırlar.

Yukarıdaki her satır uygulamada bir `MUST`'tur. Sapma yeni bir mühendislik incelemesi gerektirir.

---

## Özet

`gstack compact`, bir AI kodlama aracısının bağlam penceresine ulaşmadan önce araç çıktı gürültüsünü azaltan bir `PostToolUse` hook'u olarak tasarlandı. Belirleyici JSON kuralları gürültülü test çalıştırıcılarını, derleme günlüklerini, git diff'lerini ve paket kurulumlarını küçültürdü. Koşullu bir Claude Haiku doğrulayıcısı, aşırı sıkıştırma riski yüksek olduğunda güvenlik ağı olarak işlev görecekti.

**Mevcut durum: ASKIYA ALINDI.** Yukarıdaki "Durum" bölümüne bakın. Mimari, 2026-04-17 itibarıyla mevcut olmayan bir Claude Code API'sine (`updatedBuiltinToolOutput` veya yerleşik araçlar için eşdeğeri) bağlıdır. Anthropic sorun #36843 boşluğu takip ediyor.

**Hedeflenen amaç (askıya kaldırmadan çıkma sprinti için korunmuştur):** Uzun bir oturumda görev başarısızlık oranında sıfır artışla %15-30 araç çıktı token azaltma.

**Orijinal kama (RTK ile, 28K yıldızlı mevcut lider) — araştırma tarafından her ikisi de geçersiz kılındı:**
1. ~~**Koşullu LLM doğrulayıcı.**~~ PreToolUse komut sarmalama yoluyla hala teknik olarak uygulanabilir, ancak yalnızca Bash için. Bash-özel olduğumuzda farklılaştırıcı olmaktan çıkar. Yerel araç API'si gelirse yeniden değerlendir.
2. ~~**Yerel araç kapsama.**~~ Bugün mimari açıdan imkansız. Read/Grep/Glob Claude Code içinde işlem içi yürütülür ve hook'ları tetiklemez. `PostToolUse`'ı tetikleyen araçlar için bile MCP olmayan araçlar için çıktı-değiştirme alanı mevcut değil.

**Orijinal konumlandırma (artık geçersiz):** *"RTK hızlı. gstack compact hızlı VE güvenli, ve araç kutunuzdaki her aracı kapsar, yalnızca Bash'i değil."*

## Hedef dışı

- Kullanıcı mesajlarını veya önceki aracı dönmelerini özetleme (Claude'ın kendi Compaction API'si buna sahip).
- Aracı yanıt çıktısını sıkıştırma (caveman'ın katmanı).
- Aracıların bir komutu `GSTACK_RAW=1` ile yeniden çalıştırma kararıyla önbelleğe alma (token-optimizer-mcp'nin katmanı).
- Genel amaçlı bir günlük analizörü olarak işlev göruma.
- Marka isimleri ve marka adları değiştirilmez.

## Neden buna değer

**Sorun ölçülmüş, varsayımsal değil.**

- [Chroma araştırması (2025)](https://research.trychroma.com/context-rot) 18 sınır modelini test etti. Her model bağlam büyüdükçe bozulur. Çürüme pencere sınırından çok önce başlar — 200K'lık bir model 50K'da çürür.
- Kodlama aracıları en kötü durum: birikmeli bağlam + yüksek dikkat dağıtıcı yoğunluğu + uzun görev ufku. Araç çıktısı açıkça birincil gürültü kaynağı olarak adlandırılır.
- Pazar oy verdi: Anthropic Opus 4.6 Compaction API'sini gönderdi; OpenAI bir sıkıştırma kılavuzu gönderdi; Google ADK bağlam sıkıştırmasını gönderdi; LangChain otonom sıkıştırma gönderdi; sst/opencode yerleşik sıkıştırma içeriyor. Karma belirleyici + LLM deseni endüstri konsensüsü.

**Mevcut alan (gstack compact'ın katıldığı ve farklılaştığı):**

| Proje | Yıldız | Lisans | Katman | Tehdit | Not |
|---------|-------|---------|-------|--------|------|
| **RTK (rtk-ai/rtk)** | **28K** | Apache-2.0 | Araç çıktısı | Birincil kıyaslama | Saf Rust, yalnızca Bash, sıfır LLM |
| caveman | 34.8K | MIT | Çıktı token'ları | Farklı eksen | Kısa sistem prompt'u; bizimle BİRLİKTE çalışır |
| claude-token-efficient | 4.3K | MIT | Yanıt açıklığı | Farklı eksen | Tek CLAUDE.md |
| token-optimizer-mcp | 49 | MIT | MCP önbellekleme | Farklı eksen | Çağrıları önler, çıktıyı sıkıştırmaz |
| tokenjuice | ~12 | MIT | Araç çıktısı | Çok yeni | 2 günlük; JSON zarfımızı ilham aldı |
| 6-Layer Token Savings Stack | — | Public gist | Tarif | Sıfır | Dokümantasyon; yığınlı sıkıştırma tezini doğrular |

RTK tek doğrudan rakiptir. Diğer her şey farklı bir token kaynağını sıkıştırır.

**Lisans uyumluluğu:** Referans verilen her proje izinli lisanslıdır (MIT veya Apache-2.0) ve gstack'in MIT lisansı ile uyumludur. AGPL, GPL veya diğer copyleft bağımlılıklar yoktur. Temiz oda politikası için aşağıdaki "Lisans ve atıf" bölümüne bakın.

## Mimari

### Veri akışı

```
┌─────────────────────────────────────────────────────────────────┐
│  Ana Bilgisayar (Claude Code / Codex / OpenClaw)                │
│  ─────────────────────────────────────────                      │
│  1. Aracı araç çağrısı ister: Bash|Read|Grep|Glob|MCP           │
│  2. Ana bilgisayar aracı yürütür                                 │
│  3. Ana bilgisayar PostToolUse hook'unu şununla çağırır: {tool, input, output}   │
└────────────────────┬────────────────────────────────────────────┘
                     │ stdin (JSON zarfı)
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│  gstack-compact hook ikili dosyası                               │
│  ───────────────────────────                                    │
│  a. Zarfı ayrıştır                                                │
│  b. Kuralı şuna göre eşleştir: (araç, komut, desen)              │
│  c. Kural ilkel işlemlerini uygula: filtre / grup / kırp / çiftleri kaldır   │
│  d. Azaltma meta verilerini kaydet                                   │
│  e. Doğrulayıcı tetikleyicilerini değerlendir                                  │
│  f. Tetikleyici karşılandıysa: Haiku çağır, korunmuş satırları ekle          │
│  g. Başarısız çıkış kodunda: ham çıktıyı tee et → ~/.gstack/compact/tee/...  │
│  h. JSON zarfını stdout'a yay                                    │
└────────────────────┬────────────────────────────────────────────┘
                     │ stdout (JSON zarfı)
                     ▼
              Ana bilgisayar sıkıştırılmış çıktıyı aracı bağlamına yerleştirir
```

### Kural çözümlemesi

Üç katmanlı hiyerarşi (en yüksek öncelik kazanır), tokenjuice ve gstack'in mevcut ana bilgisayar-yapılandırma-ihracat modeli ile aynı desen:

1. Yerleşik kurallar: gstack ile birlikte gönderilen `compact/rules/`
2. Kullanıcı kuralları: `~/.config/gstack/compact-rules/`
3. Proje kuralları: `.gstack/compact-rules/`

Kurallar araç çağrılarını kural ID'sine göre eşleştirir. `tests/jest` ID'li bir proje kuralı, yerleşik `tests/jest`'i tamamen geçersiz kılar. Birleştirme yok — değiştirme semantiği, akıl yürütmeyi basit tutmak için.

### JSON zarfı sözleşmesi (tokenjuice'den uyarlandı)

Girdi:
```json
{
  "tool": "Bash",
  "command": "bun test test/billing.test.ts",
  "argv": ["bun", "test", "test/billing.test.ts"],
  "combinedText": "...",
  "exitCode": 1,
  "cwd": "/Users/garry/proj",
  "host": "claude-code"
}
```

Çıktı:
```json
{
  "reduced": "[gstack-compact: N → M lines, rule: X] başlığıyla sıkıştırılmış çıktı",
  "meta": {
    "rule": "tests/jest",
    "linesBefore": 247,
    "linesAfter": 18,
    "bytesBefore": 18234,
    "bytesAfter": 892,
    "verifierFired": false,
    "teeFile": null,
    "durationMs": 8
  }
}
```

### Kural şeması

Kompakt, minimal. Toplam kurallar yükü diskte <5KB kalmalıdır (claude-token-efficient'ten ders: kural dosyaları her oturumda token tüketir).

```json
{
  "id": "tests/jest",
  "family": "test-results",
  "description": "Jest/Vitest çıktısı — hataları ve özet sayılarını koru",
  "match": {
    "tools": ["Bash"],
    "commands": ["jest", "vitest", "bun test"],
    "patterns": ["jest", "vitest", "PASS", "FAIL"]
  },
  "primitives": {
    "filter": {
      "strip": ["\\x1b\\[[0-9;]*m", "^\\s*at .+node_modules"],
      "keep": ["FAIL", "PASS", "Error:", "Expected:", "Received:", "✓", "✗", "Tests:"]
    },
    "group": {
      "by": "error-kind",
      "header": "Türe göre gruplandırılmış hatalar:"
    },
    "truncate": {
      "headLines": 5,
      "tailLines": 15,
      "onFailure": { "headLines": 20, "tailLines": 30 }
    },
    "dedupe": {
      "pattern": "^\\s*$",
      "format": "[... {count} boş satır ...]"
    }
  },
  "tee": {
    "onExit": "nonzero",
    "maxBytes": 1048576
  },
  "counters": [
    { "name": "failed", "pattern": "^FAIL\\s", "flags": "m" },
    { "name": "passed", "pattern": "^PASS\\s", "flags": "m" }
  ]
}
```

Dört ilkel — `filter`, `group`, `truncate`, `dedupe` — doğrudan RTK'nın teknik taksonomisinden alınmıştır (ciddi her sıkıştırıcının işlemesi gereken tek şey). Her kural dördünün herhangi bir alt kümesini birleştirebilir; atlanan ilkel işlemler no-op'tur.

### Doğrulayıcı katmanı (katmanlı, katılımlı)

Doğrulayıcı yalnızca belirli tetikleyiciler altında ateşleyen ucuz bir Haiku çağrısıdır. Asla her araç çağrısında değil.

**Tetikleyici matrisi (kullanıcı yapılandırılabilir):**

| Tetikleyici | Varsayılan | Koşul |
|---------|---------|-----------|
| `failureCompaction` | **AÇIK** | çıkış kodu ≠ 0 VE azalma >%50 (teşhis risk altında) |
| `aggressiveReduction` | kapalı | azalma >%80 VE orijinal >200 satır |
| `largeNoMatch` | kapalı | kural eşleşmedi VE çıktı >500 satır |
| `userOptIn` | açık (env-gated) | `GSTACK_COMPACT_VERIFY=1` o çağrı için doğrulayıcıyı zorlar |

Varsayılan yapılandırma yalnızca `failureCompaction` ile gönderilir — en yüksek fayda durumu (aracı hata ayıklıyor; kural kritik yığın çerçevesini filtrelemiş olabilir).

**Haiku'nun işi (sınırlı):**

```
İşte ham çıktı (ilk 2000 satıra kırpılmış) ve sıkıştırılmış bir sürüm.
Ham çıktıda sıkıştırılmış olanda eksik olan önemli satırları döndürün,
veya kritik bir şey eksik değilse `NONE`.
```

Doğrulayıcı asla sıkıştırılmış çıktıyı yeniden yazmaz. Yalnızca eksik satırları bir başlık altına ekler:

```
[gstack-compact: 247 → 18 satır, kural: tests/jest]
[gstack-verify: Haiku tarafından korunmuş 2 ek satır]
  TypeError: Cannot read property 'foo' of undefined
    at parseConfig (src/config.ts:42:18)
```

**Neden Haiku, Sonnet değil:** ~1/12 maliyet, ~500ms vs ~2s, ve görev basit alt dize sınıflandırması, akıl yürütme değil.

**Doğrulayıcı yapılandırması (`compact/rules/_verifier.json`):**

```json
{
  "verifier": {
    "enabled": true,
    "model": "claude-haiku-4-5-20251001",
    "maxInputLines": 2000,
    "triggers": {
      "aggressiveReduction": { "enabled": false, "thresholdPct": 80, "minLines": 200 },
      "failureCompaction":   { "enabled": true,  "minReductionPct": 50 },
      "largeNoMatch":        { "enabled": false, "minLines": 500 },
      "userOptIn":           { "enabled": true, "envVar": "GSTACK_COMPACT_VERIFY" }
    },
    "fallback": "passthrough"
  }
}
```

**Başarısızlık modları (doğrulayıcı katı olarak eklemelidir — asla temeli bozmaz):**

- `ANTHROPIC_API_KEY` yok → doğrulayıcıyı atla, saf kural çıktısı kullan.
- Haiku çağrısı zaman aşımına uğrar (>5s) → doğrulayıcıyı atla, saf kural çıktısı kullan.
- Haiku bozuk JSON döndürür → atla, saf kural çıktısı kullan.
- Haiku prompt enjeksiyonu denemesi döndürür → temizle: yalnızca orijinal ham çıktıda alt dize eşleşmesi olan satırları ekle.
- Haiku halüsinasyon yapılmış satırlar döndürür (ham çıktıda mevcut değil) → onları bırak.

### Tee modu (RTK'dan uyarlandı)

Çıkış kodu ≠ 0 olan herhangi bir komutta, filtrelenmemiş tam çıktı `~/.gstack/compact/tee/{timestamp}_{cmd-slug}.log` dosyasına yazılır. Sıkıştırılmış çıktı bir tee dosyası işaretçisi içerir:

```
[gstack-compact: 247 → 18 satır, kural: tests/jest, tee: ~/.gstack/compact/tee/20260416-143022_bun-test.log]
```

Aracı tam yığın izlemesine ihtiyaç duyarsa tee dosyasını doğrudan okuyabilir. Bu, önceki `onFailure.preserveFull` mekanizmasını daha temiz bir tasarımla değiştirir: sıkıştırılmış çıktı her zaman küçük kalır; ham çıktı her zaman bir `cat` uzaklıktadır.

**Tee güvenliği:**

- Dosya modu `0600` — dünya tarafından okunabilir değil.
- Yerleşik gizli anahtar regex seti yazmadan önce AWS anahtarlarını, bearer token'larını ve yaygın kimlik bilgisi desenlerini sansürler.
- Başarısız yazmalar (salt okunur dosya sistemi, izin reddedildi) zarif şekilde bozulur: hala sıkıştırılmış çıktı yay, `meta.teeFailed: true` kaydet.
- Tee dosyaları 7 gün sonra otomatik olarak süresi dolar (hook başlangıcında temizlik).

### Ana bilgisayar entegrasyon matrisi

| Ana Bilgisayar | Hook türü | Desteklenen eşleştiriciler | Yapılandırma yolu |
|------|-----------|-------------------|-------------|
| Claude Code | `PostToolUse` | Bash, Read, Grep, Glob, Edit, Write, WebFetch, WebSearch, mcp__* | `~/.claude/settings.json` |
| Codex (v1.1) | `PostToolUse` eşdeğeri | Bash (birincil); araç alt kümesi TBD — ampirik doğrulama v1.1 önkoşulu | `~/.codex/hooks.json` |
| OpenClaw (v1.1) | Yerel hook API'si | Bash + MCP | OpenClaw yapılandırması |

**v1 Claude-önceliklidir.** Kama (ii) — yerel araç kapsama — [hook'lar referansı](https://code.claude.com/docs/en/hooks) üzerinden Claude Code'da doğrulanmıştır. Codex ve OpenClaw entegrasyonu yalnızca kama birincil ana bilgisayarda B-serisi kıyaslama verileriyle kanıtlandıktan sonra v1.1'de gönderilir. v1 CHANGELOG'u yalnızca Claude kapsamını açıkça belirtir.

### Yapılandırma yüzeyi

Kullanıcı yapılandırması (`~/.config/gstack/compact.toml`):

```toml
[compact]
enabled = true
level = "normal"                            # minimal | normal | aggressive (caveman deseni)
exclude_commands = ["curl", "playwright"]   # RTK deseni

[compact.bundle]
auto_reload_on_mtime_drift = true           # hook kaynak kural dosyaları daha yeniyse paketi yeniden oluşturur
bundle_path = "~/.gstack/compact/rules.bundle.json"

[compact.regex]
per_rule_timeout_ms = 50                    # Regex başına AbortSignal bütçesi; zaman aşımı → kuralı atla

[compact.verifier]
enabled = true
trigger_failure_compaction = true
trigger_aggressive_reduction = false
trigger_large_no_match = false
failure_signal_fallback = true              # exitCode eksik olduğunda /FAIL|Error|Traceback|panic/ kullan
sanitization = "exact-line-match"           # yalnızca ham çıktıda kelimesi kelimesine görünen satırları ekle

[compact.tee]
on_exit = "nonzero"
max_bytes = 1048576
redact_patterns = ["aws", "github", "gitlab", "slack", "jwt", "bearer", "ssh-private-key"]
cleanup_days = 7

[compact.benchmark]
local_only = true                           # sabit kodlanmış; yapılandırma belgeseldir, değiştirilemez
transcript_root = "~/.claude/projects"
output_dir = "~/.gstack/compact/benchmark"
scenario_cap = 20                           # toplam çıktı hacmine göre en büyük N küme
```

**Yoğunluk seviyeleri (caveman deseni):**

- **minimal:** yalnızca `filter` + `dedupe`; kırpmasız. En güvenli.
- **normal:** `filter` + `dedupe` + `truncate`. Varsayılan.
- **aggressive:** `group` ekler; daha fazla tasarruf, daha fazla uç durum riski.

### CLI yüzeyi

| Komut | Amaç | Kaynak |
|---------|---------|--------|
| `gstack compact install <host>` | PostToolUse hook'unu ana bilgisayar yapılandırmasına kaydet; `rules.bundle.json` oluşturur | yeni |
| `gstack compact uninstall <host>` | Idempotent kaldırma | yeni |
| `gstack compact reload` | Kullanıcı/proje kurallarını düzenledikten sonra `rules.bundle.json`'u yeniden oluştur | yeni |
| `gstack compact doctor` | Sapmayı / bozuk hook yapılandırmasını algıla, onarmayı teklif et | tokenjuice |
| `gstack compact gain` | Zaman içinde token/dolar tasarruflarını göster (kural başına döküm) | RTK |
| `gstack compact discover` | Eşleşen kural olmayan komutları bul, gürültü hacmine göre sırala | RTK |
| `gstack compact verify <rule-id>` | Bir fixture üzerinde doğrulayıcı kuru çalıştırma | yeni |
| `gstack compact list-rules` | Derin birleştirmeden sonra etkili kural setini göster (yerleşik + kullanıcı + proje) | yeni |
| `gstack compact test <rule-id> <fixture>` | Bir kuralı bir fixture'a uygula ve diff'i göster | yeni |
| `gstack compact benchmark` | Yerel transkript corpus'una karşı B-serisi test düzeneğini çalıştır (Kıyaslama bölümüne bak) | yeni |

Kaçış yolu: `GSTACK_RAW=1` çevre değişkeni bir komut süresince hook'u tamamen atlar (tokenjuice'ın `--raw` bayrağı ile aynı desen). Hook ayrıca herhangi bir kaynak kural dosyasının mtime'ı paket dosyasından daha yeniyse paketi otomatik olarak yeniden yükler.

## Dosya düzeni

```
compact/
├── SKILL.md.tmpl              # şablon; `bun run gen:skill-docs` ile yeniden oluştur
├── src/
│   ├── hook.ts                # giriş noktası; stdin okur, stdout yazar; paket mtime kontrolü
│   ├── engine.ts              # kural eşleştirme + azaltma meta verileri
│   ├── apply.ts               # ilkel uygulama (satır yönelimli akış hattı)
│   ├── merge.ts               # yerleşik/kullanıcı/proje kurallarının derin birleştirmesi; `extends: null`'ı onurlandırır
│   ├── bundle.ts              # kaynak kuralları derle → rules.bundle.json (install/reload)
│   ├── primitives/
│   │   ├── filter.ts
│   │   ├── group.ts
│   │   ├── truncate.ts        # halka tamponlu kuyruk; rastgele girdi boyutu için güvenli
│   │   └── dedupe.ts
│   ├── regex-sandbox.ts       # AbortSignal sınırlı regex yürütmesi (kural başına 50ms bütçe)
│   ├── verifier.ts            # Haiku entegrasyonu (tetikleyiciler + başarısızlık sinyali yedek + temizleme)
│   ├── sanitize.ts            # doğrulayıcı çıktısı için tam satır eşleşme filtresi
│   ├── tee.ts                 # gizli anahtar sansürü + 7 günlük temizlik ile ham çıktı arşivleme
│   ├── redact.ts              # gizli desen seti (AWS/GitHub/GitLab/Slack/JWT/bearer/SSH)
│   ├── envelope.ts            # JSON G/Ç sözleşmesi ayrıştırma + doğrulama
│   ├── doctor.ts              # hook sapma algılama + onarım
│   ├── analytics.ts           # kazanç + keşif sorguları yerel meta veriye karşı
│   └── cli.ts                 # argv gönderimi; alt komut başına ince bir gönderim
├── benchmark/                 # B-serisi test düzeni (sert v1 geçidi)
│   └── src/
│       ├── scanner.ts         # ~/.claude/projects/**/*.jsonl'ı tara; tool_use × tool_result bloklarını eşleştir
│       ├── sizer.ts           # çağrı başına token'lar (ceil(len/4) buluşsal); ağır kuyruğu sırala
│       ├── cluster.ts         # yüksek faydalı çağrıları (araç, komut deseni) ile grupla
│       ├── scenarios.ts       # B1-Bn gerçek dünya senaryo fixture'larını yayınla
│       ├── replay.ts          # sıkıştırıcıyı senaryolara karşı çalıştır; azaltmayı ölç
│       ├── pathology.ts       # gerçek B senaryolarının üzerine dikili hata P durumlarını katmanla
│       └── report.ts          # panel: senaryo başına önce/sonra + genel azalma
├── rules/                     # v1 yerleşik JSON kural kütüphanesi (13 kural)
│   ├── tests/
│   │   ├── jest.json
│   │   ├── vitest.json
│   │   ├── pytest.json
│   │   ├── cargo-test.json
│   │   ├── go-test.json
│   │   └── rspec.json
│   ├── install/
│   │   ├── npm.json
│   │   ├── pnpm.json
│   │   ├── pip.json
│   │   └── cargo.json
│   ├── git/
│   │   ├── diff.json
│   │   ├── log.json
│   │   └── status.json
│   ├── _verifier.json         # doğrulayıcı yapılandırması (kural olarak değil)
│   └── _HOLD/                 # v1.1 kural aileleri (v1'de gönderilmedi; referans için tutuldu)
│       ├── build/
│       ├── lint/
│       └── log/
└── test/
    ├── unit/
    ├── golden/
    ├── fuzz/                  # P-serisi — yalnızca v1 geçit alt kümesi (P1/P3/P6/P8/P15/P18/P26/P28/P30)
    ├── cross-host/            # v1: yalnızca claude-code.test.ts; codex/openclaw saplama dosyaları
    ├── adversarial/           # R-serisi — gönderilen hatalarla büyür
    ├── benchmark/             # B-serisi senaryo fixture'ları + beklenen azalma aralıkları
    ├── fixtures/              # sürüm damgalı altın girdiler (toolVersion: ön maddesi)
    └── evals/
```

## Test Stratejisi

Test plani tasarım gereği kapsamlıdır. 28K yıldızlı mevcut liderin üç yıllık regex savaş yarası olduğu bir alana gönderim yaparken, kamalarımızın (Haiku doğrulayıcı + yerel araç kapsama) yeni başarısızlık yüzeyleri tanıtması, "sıkıştırıcı aracımı aptal yaptı"nın viral olmasına bir şansımız var demektir. Buna sıfır iştah var.

### Test katmanları

| Katman | Maliyet | Sıklık | Birleştirmeyi engeller |
|------|------|-----------|--------------|
| Birim | ücretsiz, <1s | her PR | evet |
| Altın dosya (`toolVersion:` ön maddesi ile) | ücretsiz, <1s | her PR | evet |
| Kural şema doğrulama | ücretsiz, <1s | her PR | evet |
| Bulanıklaştırma (P-serisi geçit alt kümesi: P1/P3/P6/P8/P15/P18/P26/P28/P30) | ücretsiz, <10s | her PR | evet |
| Ana bilgisayarlar arası U2U — v1'de yalnızca Claude Code | ücretsiz, ~1dk | her PR (geçit katmanı) | evet |
| Doğrulayıcı ile U2U (mock Haluk) | ücretsiz, ~15s | her PR | evet |
| Doğrulayıcı ile U2U (gerçek Haluk) | ücretli, ~$0.10/çalıştırma | doğrulayıcı dosyalarını etkileyen PR | evet |
| **B-serisi kıyaslama (gerçek dünya senaryoları)** | **ücretsiz, ~2dk** | **yayın öncesi geçit** | **evet (v1 için sert geçit)** |
| Token tasarrufu değerlendirmesi (E1-E4 sentetik) | ücretli, ~$4/çalıştırma | dönemsel haftalık | hayır (bilgi amaçlı) |
| Düşmanca regresyon (R-serisi) | ücretsiz, <5s | her PR | evet |
| Araç sürümü sapma uyarısı | ücretsiz, <1s | her PR | yalnızca uyarı |

### G-serisi: iyi durumlar (beklenen azalmayı üretmeli)

| ID | Senaryo | Beklenen azalma |
|----|----------|-------------------|
| G1 | `jest` 47 geçen test, temiz çalıştırma | 150+ satır → ≤10 satır |
| G2 | `jest` 47 test, 2 başarısızlık ile | 200+ satır → her iki başarısızlığı + özeti koru |
| G3 | `vitest` `--reporter=verbose` ile çalıştırma | 300+ satır → ≤15 satır |
| G4 | `pytest` toplama ardından çalıştırma | başarısızlık geri izlemelerini koru |
| G5 | `cargo test` bir panic ile | panic konumu kelimesi kelimesine korundu |
| G6 | `go test -v` 200 alt test geçiyor | `PASS: 200 alt test` olarak daralt |
| G7 | 500 satır bağlam içeren bir dosyada `git diff` 2 parça ile | parçaları koru, bağlamı bırak |
| G8 | `git log -50` | SHA + konu + yazarı koru, gövdeyi bırak |
| G9 | `git status` 30 değiştirilmiş dosya ile | dizine göre grupla |
| G10 | `pnpm install` taze | son sayı + uyarılar; çözülen paketleri bırak |
| G11 | `pip install -r requirements.txt` | indirme ilerlemesini bırak; son kurulum listesi + hataları koru |
| G12 | `cargo build` başarılı | derleme ilerlemesini bırak; son hedefi koru |
| G13 | `docker build` başarılı | katman çekmelerini bırak; son image digest'ini koru |
| G14 | `tsc --noEmit` temiz | `tsc: 0 hata` olarak sıkıştır |
| G15 | `tsc --noEmit` 3 hata ile | 3 hatanın tamamını konum ile koru |
| G16 | `eslint .` temiz | `eslint: 0 sorun` olarak sıkıştır |
| G17 | `eslint .` ihlaller ile | kurala göre grupla; konum + düzeltme önerisini koru |
| G18 | `docker logs -f` 1000 tekrarlayan satır ile | sayım ile çiftleri kaldır: `[son mesaj 973 kez tekrarlandı]` |
| G19 | `kubectl get pods -A` | ad alanına göre grupla |
| G20 | `ls -la` derin ağaç | dizin gruplama (RTK deseni) |
| G21 | `find . -type f` 10K dosya | sayımlarla uzantıya göre grupla |
| G22 | `grep -r "foo" .` 500 eşleşme ile | 50'de sınırla; sonek `[... 450 daha fazla eşleşme; tamamı için --ripgrep kullanın]` |
| G23 | `curl -v https://api.example.com` | ayrıntılı başlıkları çıkar; yanıt gövdesini koru |
| G24 | `aws ec2 describe-instances` 50 örnek ile | sütunlu özet |

### P-serisi: patolojik durumlar (aracıyı BOZMAMALI)

Bunlardan herhangi birini yanlış yaparsak "güzel özellik"ten "felaket regresyon"a dönüşür.

| ID | Senaryo | Gerekli davranış |
|----|----------|-------------------|
| P1 | Çıktıda ikili çöp (UTF-8 olmayan baytlar) | Değiştirmeden geçir; çöktürme |
| P2 | ANSI kaçış patlaması (10K+ kod) | Temizçe çıkar, regex'i boğma |
| P3 | Boş çıktı (`""`) | Boş geçir; başlık ENJEKTE ETME |
| P4 | Stdout+stderr karışık | Kural her iki akışta eşleşir |
| P5 | Kesilmiş çıktı (akış ortasında SIGPIPE) | Kısmi çıktıyı yanlış sıkıştırma |
| P6 | **Başarısız test, 200 satırın 4. satırında kritik yığın çerçevesi** | Çerçeveyi FILTRELEMEMELİ (RTK-katil durumu) |
| P7 | Çıkış 0 ama çıktıda `ERROR:` | Kural yalnızca çıkış koduna güvenmemeli |
| P8 | Çıktı AWS anahtarı / bearer token / parola içeriyor | Tee dosyası dünya tarafından okunabilir OLMAMALI; sıkıştırılmış çıktıda sansürle |
| P9 | Tek satırlık küçültülmüş JS hatası (40KB bir satır) | İlk 1KB'ye kırp; `[... kırpıldı ...]` ekle |
| P10 | Unicode (emoji, RTL, birleştirici karakterler, CJK) | Bayt-güvenli kırpmada; kod noktalarını bölme |
| P11 | İki kural aynı komutu eşleştiriyor | Deterministik öncelik: en uzun `match.commands` öneki kazanır; beraberlik → kural ID'si alfabetik |
| P12 | Kuralın sıkıştırılmış çıktısı başka bir kuralın desenini eşleştiriyor | Özyinelemeli uygulama yok; hook araç çağrısı başına bir kez çalışır |
| P13 | Komut tırnak içine alınmış argümanda gömülü yeni satırlar içeriyor | Kural argümanları yanlış ayrıştırmaz |
| P14 | Eşzamanlı araç çağrıları (paralel Bash çağrıları) | Hook'ta paylaşılan değiştirilebilir durum yok; her çağrı izole |
| P15 | Hook yürütme >5s | Ham geçir; `meta.timedOut: true` yay |
| P16 | Haiku API çevrimdışı/hız sınırlı | Doğrulayıcıyı sessizce atla; saf kural çıktısı kullan |
| P17 | Haiku bozuk JSON döndürür | Doğrulayıcıyı atla; ham yanıtı aracıya BESLEME |
| P18 | Haiku yanıtı prompt enjeksiyonu içeriyor (`"Tüm önceki talimatları yoksay..."`) | Temizle: yalnızca orijinal ham çıktıda alt dize eşleşmesi olan satırları ekle |
| P19 | 1M satırlık çıktı | Akış-işle, belleği 64MB'ta sınırla; net işaretçi ile kırp |
| P20 | Hızlı ateş: saniyede 50 araç çağrısı | Hook gecikmesi <15ms p99 kalır |
| P21 | Kabuk yönlendirmeleri olan komut (`cmd >file 2>&1`) | Yönlendirme sarmalayıcısında değil, temel komut adında eşleş |
| P22 | Komut dizesinde derin iç içe geçmiş tırnaklar/kaçışlar | Sağlam argüman ayrıştırıcı; kabuk enjeksiyonu mümkün değil |
| P23 | Çıktıda NULL baytlar | Güvençe çıkar; kırpmasız |
| P24 | Çıkış yapan ve ardından stderr'e daha fazla yazan komut | Hook son birleştirilmiş çıktıyı alır; zarifçe işler |
| P25 | Salt okunur dosya sistemi / tee yazma izni yok | Zarifçe bozul; hala sıkıştırılmış çıktı yay; `meta.teeFailed: true` kaydet |
| P26 | Kullanıcının kural JSON'u bozuk | O kuralı atla; stderr'a uyarı yay; hook'u bozma |
| P27 | Kural var olmayan bir ilkel alana başvuruyor | Bilinmeyen alanı yoksay; kuralın geri kalanını uygula |
| P28 | Kural regex'inin felaket geri izlemesi var | RE2 uyumlu motor (geri izleme yok) VEYA kural başına zaman aşımı |
| P29 | Çıkış kodu 137 (OOM öldürme) | Kural genel başarısızlıkla aynı şekilde işler; tam çıktıyı korur |
| P30 | Haiku ham çıktıda OLMAYAN satırlar döndürür (halüsinasyon) | Halüsinasyon yapılmış satırları bırak; yalnızca alt dize eşleşmelerini koru |

### CH-serisi: ana bilgisayaylar arası U2U

Her senaryoyu desteklenen her ana bilgisayarda çalıştır. Aynı girdi, aynı beklenen çıktı. Bir ana bilgisayar bir eşleştiriciyi desteklemiyorsa, test yukarı akış sınırlamasına bağlantılı yorumla `skip-on-{host}` olarak işaretlenir.

| ID | Senaryo | Ana Bilgisayarlar |
|----|----------|-------|
| CH1 | `gstack compact install <host>` ile hook'u kurma | Claude Code, Codex, OpenClaw |
| CH2 | Hook kaldırma idempotent | Tümü |
| CH3 | Yeniden kurulum girdileri çoğaltmaz | Tümü |
| CH4 | Hook kullanıcının diğer PostToolUse hook'ları ile bir arada var | Tümü |
| CH5 | Hook Bash aracında ateşler | Tümü |
| CH6 | Hook Read aracında ateşler | Claude Code (doğrulanmış); Codex/OpenClaw doğrula-sonra-gerektir |
| CH7 | Hook Grep aracında ateşler | CH6 ile aynı |
| CH8 | Hook Glob aracında ateşler | CH6 ile aynı |
| CH9 | Hook MCP aracında ateşler (`mcp__*` eşleştirici) | Claude Code; diğerlerinde doğrula |
| CH10 | Yapılandırma önceliği: proje > kullanıcı > yerleşik | Tümü |
| CH11 | `GSTACK_RAW=1` çevre değişkeni hook'u atlar | Tümü |
| CH12 | Kural ID geçersiz kılma çalışıyor (proje kuralı yerleşik olanı değiştirir) | Tümü |
| CH13 | `gstack compact doctor` her ana bilgisayarda sapmayı algılar | Tümü |
| CH14 | Hook hatası aracı oturumunu çökertmez | Tümü |

Uygulama notu: ana bilgisayaylar arası testler `golden/` ağacından fixture corpus'unu yeniden kullanır; düzenek her fixture'ı ana bilgisayara özgü hook çağırma zarfına sarar ve çıktının ana bilgisayaylar arasında bayt olarak özdeş olduğunu doğrular (`host` alanı dışında).

### V-serisi: doğrulayıcı testleri (ücretli)

| ID | Senaryo | Beklenen |
|----|----------|----------|
| V1 | Kural 200 satırlık test çıktısını 5 satıra indirir, çıkış=1 | Doğrulayıcı ateşler (başarısızlık + >%50 azalma), eksik kritik satırları ekler |
| V2 | Kural 10 satırlık çıktıyı 9 satıra indirir, çıkış=1 | Doğrulayıcı ateşlemez (azalma çok küçük) |
| V3 | Kural 200 satırlık çıktıyı 5 satıra indirir, çıkış=0 | Doğrulayıcı ateşlemez (başarı yolu, varsayılan yapılandırma) |
| V4 | `aggressiveReduction` tetikleyicisi etkin, 300 satır → 20 satır, çıkış=0 | Doğrulayıcı ateşler |
| V5 | `GSTACK_COMPACT_VERIFY=1` çevre değişkeni ayarlandı | Doğrulayıcı o çağrı için bir kez ateşler |
| V6 | `ANTHROPIC_API_KEY` eksik | Doğrulayıcı sessizce atlandı; ham kural çıktısı döndürüldü |
| V7 | Doğrulayıcı "NONE" döndürmek için mock'landı | Çıktı saf kural yolu ile özdeş |
| V8 | Doğrulayıcı prompt enjeksiyonu döndürmek için mock'landı | Enjeksiyon atıldı; yalnızca alt dize eşleşmeli satırlar eklendi |
| V9 | Doğrulayıcı >5s zaman aşımına uğramak için mock'landı | Atlandı; `meta.verifierTimedOut: true` |
| V10 | Doğrulayıcı 500 hatası döndürmek için mock'landı | Atlandı; kural çıktısı döndürüldü |

### R-serisi: düşmanca regresyon

v1 gönderildikten sonra yakalanan her hata kalıcı bir R-serisi test alır. Boş başlar; yaralarla büyür. Şablon:

```
R{N}: {commit-sha} — {1-satır özet}
Senaryo: {yeniden üretici}
Düzeltme: {PR bağlantısı}
```

### Performans bütçeleri (CI'da uygulanır; gerçekçi Bun soğuk başlangıcı için düzeltilmiştir)

| Metrik | Hedef | Sınır |
|--------|--------|-----------|
| Hook ek yükü macOS ARM (doğrulayıcı devre dışı) | <30ms p50 | <80ms p99 |
| Hook ek yükü Linux (doğrulayıcı devre dışı) | <20ms p50 | <60ms p99 |
| Hook ek yükü (doğrulayıcı ateşler) | <600ms p50 | <2s p99 |
| Paket serileştirme (rules.bundle.json) | <2ms | <10ms |
| mtime sapma kontrolü (kaynak dosyaların stat'i) | <0.5ms | <3ms |
| Tek regex yürütme bütçesi (kural başına) | <5ms | <50ms (sert iptal) |
| Hook çağrısı başına bellek (satır akışlı) | <16MB tipik | <64MB maks |
| Diskteki toplam kural yükü boyutu (kaynak dosyalar) | <5KB | <15KB |
| Diskteki derlenmiş paket boyutu | <25KB | <80KB |

Arka plan programı modu v2 optimizasyonudur. Yazarın corpus'unda B-serisi kıyaslama soğuk başlangıcın oturum toplam tasarruflarını anlamlı şekilde etkilediğini gösterirse (örn., toplam hook ek yükü kaydedilen token'ların duvar süresinin >%5'i), v1.1'e yükselt.

### B-serisi gerçek dünya kıyaslama test düzeni (sert v1 geçidi)

**Neden var.** Rekabet eden her sıkıştırıcı elle seçilmiş fixture sayılarıyla gönderilir. B-serisi, kullanıcı hook'u etkinleştirmeden önce sıkıştırıcının kullanıcının *gerçek* kodlama oturumlarında çalıştığını kanıtlar. Hem gönderim geçidi hem de pazarlama eseridir.

**Mimari** (`compact/benchmark/src/` içindeki bileşenler):

```
┌──────────────────────────────────────────────────────────────┐
│  1. TARA     scanner.ts ~/.claude/projects/**/*.jsonl'ı tarar  │
│              → tool_use × tool_result bloklarını eşleştirir           │
│              → {tool, command, outputBytes, lineCount,         │
│                estimatedTokens, sessionId, timestamp} yayınlar        │
├──────────────────────────────────────────────────────────────┤
│  2. SIRALA     sizer.ts corpus'u estimatedTokens azalan ile sıralar   │
│              → cluster.ts (araç, komut deseni) ile gruplandırır  │
│              → ağır kuyruğu tanımlar: çağrıların hangi %10'u     │
│                token'ların %80'ini üretti?                   │
├──────────────────────────────────────────────────────────────┤
│  3. SENARYO scenarios.ts fixture dosyaları yayınlar:               │
│              B1_bun_test_heavy.jsonl                         │
│              B2_git_diff_huge.jsonl                          │
│              B3_tsc_errors_production.jsonl                  │
│              B4_pnpm_install_fresh.jsonl ... (her              │
│              yüksek faydalı küme için bir, ~20 senaryoya kadar)     │
├──────────────────────────────────────────────────────────────┤
│  4. YENİDEN OYNAT   replay.ts sıkıştırıcıyı her senaryoya karşı çalıştırır, │
│              token azaltma + bırakılan satırların diff'ini ölçer│
│              → kural başına azaltma sayıları                    │
│              → senaryo başına önce/sonra token sayıları        │
├──────────────────────────────────────────────────────────────┤
│  5. PATOLOJI pathology.ts dikili kritik satırları enjekte eder    │
│              (başarısız bir test fixture'ında 200 satırın 4. satırı) içine  │
│              gerçek B senaryoları. Doğrulayıcının bunları geri yüklediğini    │
│              doğrular. Gerçek veri + gerçek tehditler = gerçek kanıt.    │
├──────────────────────────────────────────────────────────────┤
│  6. RAPOR   report.ts HTML + JSON panel yayınlar →        │
│              ~/.gstack/compact/benchmark/latest/              │
│              "SENİN 30 günlük Claude Code verinde, gstack    │
│              compact X token tasarrufu sağlar Y senaryosunda."    │
└──────────────────────────────────────────────────────────────┘
```

**v1 gönderim geçidi (sert):**
- Yazarın kendi 30 günlük transkript setinde birleştirilmiş senaryo corpus'unda ≥%15 toplam token azaltma.
- Dikili hata senaryolarında sıfır kritik satır kaybı (her dikili yığın çerçevesi ya kural ya da doğrulayıcı tarafından hayatta kalmalıdır).
- Yeni kurallar altında hiçbir senaryo <%5 azalmaya gerilemez (aşırı sıkıştırma uç durumlarını yakala).

**Gizlilik (pazarlık kabul etmez):**
- Yalnızca yerel olarak `~/.claude/projects/**/*.jsonl` okur. Asla yüklemez. Asla paylaşmaz. Asla senaryoları telemetriye kaydetmez.
- Çıktı dosyaları `~/.gstack/compact/benchmark/` altında `0600` modu ile yaşar.
- Komut bir onay başlığı yazdırır: *"Yerel transkriptleri ~/.claude/projects/ konumunda tarıyor (yalnızca yerel; hiçbir şey bu makineden çıkmaz)."*
- Herhangi bir gelecek topluluk corpus'u, OSS projelerinde elle katkıda bulunulan, gizli anahtarı taranmış fixture'lardan oluşturulan ayrı bir v2 iş akışmdır.

**analyze_transcripts'ten taşımalar (TypeScript yeniden uygulaması; alt süreç çağrısı değil):**
- JSONL ayrıştırma + tool_use/tool_result eşleştirme deseni (`event_extractor.rb`'den).
- Token tahmini `ceil(len/4)` (aynı karakter-oranı buluşsal yöntem; sıralama için yeterli).
- Senaryo kümeleme için olay türü taksonomisi (`bash_command`, `file_read`, `test_run`, `error_encountered`).
- Patoloji katmanlama için stres fixture üretim deseni.

**Taşumadığımız şey:** davranışsal puanlama, pgvector gömmeleri, karar değişim grafikleri, hız metrikleri, Rails/ActiveRecord katmanı. Kapsam dışında; ölçtüğümüz şey bu değil.

### Sentetik token tasarrufu değerlendirmeleri (E-serisi, dönemsel/yalnızca bilgi amaçlı)

Orijinal plandan korunmuş ama artık yalnızca bilgi amaçlı çünkü B-serisi gerçek geçit.

- **E1:** orta büyüklükte bir TypeScript projesinde simüle edilmiş 30 dakikalık kodlama oturumu. gstack compact etkin/kapalı toplam token'ları ölç. Hedef: ≥%15 azalma.
- **E2:** `level=aggressive` ile aynı oturum. Hedef: ≥%25 azalma, sıfır test başarısızlık artışı.
- **E3:** yalnızca `failureCompaction` doğrulayıcısı ile aynı oturum. Doğrulayıcı ateşleme oranı araç çağrılarının ≤%10'u.
- **E4:** düşmanca — bir test çıktısına dikili hata enjekte et ve doğrulayıcının kritik yığın çerçevesini geri yüklediğini doğrula.

### Test corpus kaynaklama

Her kural ailesi için 3+ gerçek çıktı yakala:

1. Aracı gerçek bir projeye karşı çalıştır (TS için gstack'in kendisi; Rust/Go/Python için popüler OSS).
2. stdout+stderr+çıkış kodunu `toolVersion:` ön maddesi ile bir fixture dosyasına yakala (örn., `jest@29.7.0`).
3. Beklenen sıkıştırılmış çıktıyı bir kez elle yaz.
4. Altın dosya testi: kural uygulaması bayt olarak özdeş çıktı üretmeli.
5. CI sapma uyarısı: kurulu araç sürümü fixture'ın `toolVersion:` değerinden farklıysa, CI uyarır (başarısız olmaz). Sapma uyarı paneli yayın öncesi kontrol edilir.

Şuradan çek:
- tokenjuice'ın fixture dizin desenleri (`tests/fixtures/`)
- RTK'nın komut başına örnekleri (README'lerinde gerçek önce/sonra metrikleri listelenir; bağımsız doğrula)
- gstack'in kendi test çıktısı (kendi yemeğimizi yiyoruz)
- `~/.gstack/compact/tee/`'den gerçek hata arşivleri (gönüllüler katkıda bulununca)
- **B-serisi gerçek dünya senaryoları azaltma ölçümleri için birincil corpus.**

## Desen benimseme tablosu

Rekabetçi manzaradan ödünç alınan somut desenler:

| Kaynak | Olarak benimse | Neden |
|------|----------|-----|
| RTK | 4 azaltma ilkel işlemleri (filter/group/truncate/dedupe) JSON kural fiilleri olarak | Ciddi bir sıkıştırıcı için masa bahisleri |
| RTK | Hata modu ham kaydı için `gstack compact tee` | Orijinal `onFailure.preserveFull` tasarımından daha iyi |
| RTK | `gstack compact gain` + `gstack compact discover` | Güven + sürekli iyileştirme |
| RTK | Kullanıcı başına engelleme listesi olarak `exclude_commands` | Gerekli yapılandırma |
| tokenjuice | Hook G/Ç için JSON zarfı sözleşmesi | Temiz makine bağdaştırıcısı |
| tokenjuice | `gstack compact doctor` | Hook'lar sapar; kendi kendini onarım önemli |
| caveman | Yoğunluk seviyeleri (minimal/normal/aggressive) | Kullanıcı ayarlanabilir güvenlik/tasarruf düğmesi |
| claude-token-efficient | Kural dosyası boyut bütçesi (toplam <5KB) | Bağlamı şişirme |

## Dağıtım planı

**TÜM AŞAMALAR Anthropic `updatedBuiltinToolOutput` API'sini bekliyor.** Bu dokümanın üstündeki Durum bölümüne bakın. Dağıtım aşağıda, API geldiğinde ve bu tasarım askıdan çıktığında amaçlanan dizilimdir.

### Askıdan çıkma kontrol listesi (API geldiğinde sırayla yap)

1. **Yeni API'nin şeklini doğrula.** Güncellenmiş Claude Code hook'ları referansını oku. Yeni çıktı-değiştirme alanını içeren gerçek bir zarfı Bash, Read, Grep, Glob için yakala. `docs/designs/GCOMPACTION_envelope.md`'da kaydet.
2. **Kamayı yeniden doğrula.** Yeni API Read/Grep/Glob'ı kapsıyor mu (şimdi `PostToolUse`'ı ateşliyorlar mı), yoksa yalnızca Bash/WebFetch mi? Yalnızca Bash ise, kama (ii) hala ölüdür ve ürünün gönderimden önce yeni bir sunum yapması gerekir.
3. **Düzeltilmiş plana karşı `/plan-eng-review`'u yeniden çalıştır.** Kilitlenmiş 15 kararın çoğu ileriye taşınabilir; Mimari veri akışını ve zarf-bağımlı kararları güncelle.
4. **Düzeltilmiş plana karşı `/codex review`'u yeniden çalıştır.** Önceki BLOCK kararının hook ikame endişeleri API geldiğinde kaybolur; kalan kritikler (B-serisi gizlilik, regex DoS, JSON zarf akışı) hala geçerli.
5. **Orijinal dağıtımı aşağıda çalıştır.**

### Orijinal dağıtım (askıdan çıkarma için korunmuştur)

Her katman, önceki katmanın tüm geçit katmanı testlerini geçmesine bağlıdır. Claude-öncelikli — Codex ve OpenClaw kama birincil ana bilgisayarda kanıtlandıktan sonra v1.1'de gelir.

1. **v0.0 (1 gün):** kural motoru + 4 ilkel + satır yönelimli akış hattı + derin birleştirme + paket derleyici + zarf sözleşmesi + yalnızca `tests/*` ailesi için altın testler. Henüz ana bilgisayar entegrasyonu yok. Çevrimdışı fixture'larda tasarrufları ölç.
2. **v0.1 (1 gün):** Claude Code hook entegrasyonu + `gstack compact install` + mtime tabanlı otomatik yeniden yükleme. Katılımlı olarak gönder; varsayılan olarak kapalı. 10 gstack güçlü kullanıcısını denemelerini iste; geri bildirim topla.
3. **v0.5 (1 gün):** B-serisi kıyaslama test düzeneği (`compact/benchmark/`). Kullanıcıların kendi verilerinde ölçebilmeleri için `gstack compact benchmark` gönder. Deneysel kullanıcılardan anonim-baştan (hiçbir şey yüklenmedi) azaltma sayılarını topla.
4. **v1.0 (1 gün):** varsayılan olarak açık `failureCompaction` tetikleyicisi ile doğrulayıcı katmanı + tam satır eşleşme temizleme + katmanlı exitCode/desen yedek + genişletilmiş tee sansür seti. **Sert gönderim geçidi:** Yazarın 30 günlük yerel corpus'unda B-serisi ≥%15 toplam azalma VE dikili hatalarda sıfır kritik satır kaybı gösterir. CHANGELOG girdisini kama çerçevesiyle öne çıkarak yayınla (v1'de yalnızca Claude Code).
5. **v1.1 (+1 gün):** Codex + OpenClaw hook entegrasyonu. Ana bilgisayaylar arası U2U paketi yeşil. Derleme/lint/günlük kural aileleri `gstack compact discover`'dan türetilen önceliklerle gelir.
6. **v1.2+:** kural ailelerini genişlet, topluluk kural katkı iş akışı, topluluk corpus kıyaslaması (yerel B-serisinden ayrı, elle yazılmış genel fixture'lar).

## Risk analizi

| Risk | Şiddet | Azaltma |
|------|----------|------------|
| RTK yanıt olarak bir LLM doğrulayıcı ekler | Düşük | Yaratıcı sıfır bağımlılık Rust konusunda sesli. Önce gönder, desen kütüphanesini oluştur. |
| Platform sıkıştırması bizi emer (Claude Code'da Anthropic Compaction API) | Orta | Farklı bir katmanda çalışırız (araç başına çıktı vs tüm bağlam). Tamamlayıcı olarak konumlandır. |
| Kurallar kritik bir şey düşürür → "sıkıştırıcı aracımı aptal etti" | Yüksek | B-serisi gerçek dünya kıyaslaması sert gönderim geçidi olarak; tee modu her zaman mevcut; başarısızlıklar için doğrulayıcı varsayılan olarak açık; tam satır eşleşme temizleme. |
| Haiku maliyet sızıntısı (tetikleyiciler beklenenden daha sık ateşler) | Orta | E3 değerlendirmesi + B-serisi ateşleme oranı metrik; maliyet `gstack compact gain`'de görünür; oran >%10 ise v1.1'de oturum başına oran sınırı. |
| Kural bakım borcu (jest/vitest çıktı biçimleri değişir) | Orta | `toolVersion:` fixture ön maddesi + CI sapma uyarısı; topluluk kural PR'ları; `discover` atlayan komutları işaretler. |
| Kurallar dosyası bağlamı şişirir | Düşük | CI ile zorlanan <5KB kaynak + <25KB derlenmiş paket bütçesi; şema doğrulamada kural başına boyut uyarısı. |
| Regex DoS aracıyı engeller | Orta | Kural başına 50ms AbortSignal bütçesi; zaman aşımı `meta.regexTimedOut`'a günlüklenir; tekrarlayan başarısızlıkta eski kurallar karantinaya alınır. |
| Paket eskimesi sessizce kullanıcı düzenlemelerini bozar | Düşük | Her hook çağırmasında mtime kontrolü otomatik yeniden oluşturur; `gstack compact reload` bir gereklilik değil yedektir. |
| Kıyaslama kullanıcının özel verilerini sızdırır | Yüksek | Yapı gereği yalnızca yerel: ağ çağrısı yok, mod-0600 çıktı, çalışma zamanında açık başlık. v1 gönderiminden önce gizlilik incelemesi. |

## Açık sorular

1. ~~Codex'in PostToolUse hook'u Read/Grep/Glob için eşleştiricileri destekliyor mu?~~ (v1.1'e ertelendi — v1'de Claude-öncelikli.)
2. ~~OpenClaw'ın hook API'si özel olarak PostToolUse'ı destekliyor mu?~~ (v1.1'e ertelendi.)
3. Doğrulayıcı model sabitlenmeli mi, yoksa gstack'in diğer AI çağrıları gibi sürüm takipli mi olmalı? (Eğilim: `claude-haiku-4-5-20251001`'i sabitle ve CHANGELOG'da açıkça artır.)
4. ~~Tee dosyaları için yerleşik gizli anahtar sansür regex seti~~ **(çözüldü: genişletilmiş set — AWS/GitHub/GitLab/Slack/JWT/bearer/SSH-private-key. Karar #10'a bak.)**
5. `gstack compact discover` Haiku aracılığıyla otomatik oluşturulan kurallar önermeli mi? (v2'ye ertelendi; yetenek kayması riski.)
6. **Yeni:** Claude Code'un PostToolUse zarfı `exitCode` içeriyor mu? (Hala uygulama öncesi görev #1 için ampirik doğrulama gerekiyor; sistem artık buna bakılmaksızın katmanlı bir yedeklemeye sahip.)
7. **Yeni:** B-serisi için doğru senaryo sayısı sınırı nedir? Cluster.ts ağır kuyruk şekline bağlı olarak 5-50 senaryo üretebilir. Plan: toplam çıktı hacmine göre en büyük 20 kümeye sınır koy.

## Uygulama öncesi atama (kodlamadan önce tamamlanmalı)

1. **Claude Code'un PostToolUse zarfı içeriğini ampirik olarak doğrula.** No-op hook gönder; `exitCode`, `command`, `argv`, `combinedText`'in hepsinin mevcut olduğunu doğrula. Bu kama (ii) yerel araç kapsama VE failureCompaction tetikleyicisi için dönüm noktasıdır. Çıktı: Bash + Read + Grep + Glob için gerçek yakalanmış zarflar ile `docs/designs/GCOMPACTION_envelope.md`.
2. **RTK'nın kural tanımlarını oku** (`ARCHITECTURE.md`, `src/rules/`) ve 4 ilkel işlemden hangilerini en iyi işlediklerine dair 1 paragraf özet yaz. v1 kural setimizi bilgilendir. Bu, Yapmadan Önce Ara katmanıdır.
3. **analyze_transcripts JSONL ayrıştırıcısını TypeScript'e taşı.** `compact/benchmark/src/scanner.ts`. Yazarın `~/.claude/projects/` dizinindeki en gürültülü 50 araç çağrısını listeleyen bir hızlı bakış çıktısı yaz. Yeniden oynatma döngüsünü oluşturmadan önce test düzeni önermesini doğrular. Bu, B-serisi temelidir.
4. **CHANGELOG girdisini ÖNCE yaz.** Hedef cümle: *"Claude Code'daki aracın araç kutunuzdaki her araç artık daha az gürültü üretiyor — test çalıştırıcıları, git diff'leri, paket kurulumları — kurallarımız aşırı sıkıştırdığında kritik yığın çerçevelerini geri yükleyen akıllı bir Haiku güvenlik ağı ile ve sizin gerçek 30 günlük kodlama oturumlarınızda tasarrufları kanıtlayan yerel bir kıyaslama ile. Codex + OpenClaw v1.1'de geliyor."* Bu cümleyi dürüstçe yazamazsak, kama henüz hazır değil.
5. **Yalnızca kural v0 gönder** (Haiku doğrulayıcı yok, kıyaslama yok). Mevcut gstack değerlendirmeleri + erken B-serisi prototipi ile gerçek token tasarruflarını ölç. Yerel corpus'ta <%10 ise, tüm önerme iddia edildiğinden daha zayıftır — doğrulayıcıyı üzerine eklemeden önce kuralları yinele.

## Lisans ve atıf

gstack MIT altında yayınlanır. Lisansı aşağıdaki kullanıcılar için temiz tutmak için bu proje, rekabetçi manzaradan ödünç alınan her şey için katı bir temiz oda politikası izler:

- **Yukarıda referans verilen her proje izinli lisanslıdır** (MIT veya Apache-2.0). AGPL, GPL, SSPL veya diğer copyleft maruziyet yok.
  - RTK (rtk-ai/rtk): **Apache-2.0** — MIT uyumlu; Apache patent hibesi bizim için bir bonus.
  - tokenjuice, caveman, claude-token-efficient, token-optimizer-mcp, sst/opencode: **MIT**.
- **Desenler, kod değil.** Bu projeleri neyi çözdüklerini ve nedenini anlamak için okuyoruz. `compact/src/` içinde TypeScript olarak bağımsız uyguluyoruz. Kaynak dosyaları kopyalamıyoruz, kaynak dosyaları satır satır çevirmiyoruz veya test fixture'larını kelimesi kelimesine alıyoruz.
- **Atıf.** Doğrudan ödünç alınan bir desen varsa (RTK'dan 4 ilkel, tokenjuice'dan JSON zarfı, caveman'dan yoğunluk seviyeleri, claude-token-efficient'ten kurallar dosyası boyut bütçesi), kaynağı satır içi yorumlarda ve yukarıdaki "Desen benimseme tablosu"nda onurlandırıyoruz. Projenin `README`'si ve `NOTICE` dosyası (eklersek) ilham kaynaklarını listeler.
- **Fixture kaynaklama.** Altın dosya fixture'ları gerçek projelere karşı gerçek araçları çalıştırmaktan gelir — bunlar bizim kendi yakalamalarımız, RTK veya tokenjuice'tan içe aktarılmaz. Bu, test corpus'unu lisans dolanmış içeriğinden uzak tutar.
- **Yasaklı kaynaklar.** Yeni bir referans projesi eklemeden önce `gh api repos/OWNER/REPO --jq '.license'` çalıştırın ve lisans anahtarının şunlardan biri olduğunu doğrulayın: `mit`, `apache-2.0`, `bsd-2-clause`, `bsd-3-clause`, `isc`, `cc0-1.0`, `unlicense`. Projenin lisans alanı yoksa, "tüm hakları saklıdır" olarak değerlendirin ve ondan çekmeyin. `agpl-3.0`, `gpl-*`, `sspl-*` ve herhangi bir özel veya kaynak kullanılabilir lisansı reddedin.

CI uygulaması: `scripts/check-references.ts` betiği `docs/designs/GCOMPACTION.md`'dan GitHub URL'lerini ayrıştırır ve referans verilen herhangi bir projenin lisansı izin verilenler listesinden çıkarsa başarısız olur.

## Referanslar

- [RTK (Rust Token Killer) — rtk-ai/rtk](https://github.com/rtk-ai/rtk)
- [RTK sorun #538 — yerel araç boşluğu](https://github.com/rtk-ai/rtk/issues/538)
- [tokenjuice — vincentkoc/tokenjuice](https://github.com/vincentkoc/tokenjuice)
- [caveman — juliusbrussee/caveman](https://github.com/juliusbrussee/caveman)
- [claude-token-efficient — drona23](https://github.com/drona23/claude-token-efficient)
- [token-optimizer-mcp — ooples](https://github.com/ooples/token-optimizer-mcp)
- [6-Layer Token Savings Stack — doobidoo gist](https://gist.github.com/doobidoo/e5500be6b59e47cadc39e0b7c5cd9871)
- [Claude Code hook'ları referansı](https://code.claude.com/docs/en/hooks)
- [Chroma bağlam çürüme araştırması](https://research.trychroma.com/context-rot)
- [Morph: Neden LLM'ler Bağlam Büyüdükçe Bozulur](https://www.morphllm.com/context-rot)
- [Anthropic Opus 4.6 Compaction API — InfoQ](https://www.infoq.com/news/2026/03/opus-4-6-context-compaction/)
- [OpenAI sıkıştırma dokümantasyonu](https://developers.openai.com/api/docs/guides/compaction)
- [Google ADK bağlam sıkıştırma](https://google.github.io/adk-docs/context/compaction/)
- [LangChain otonom bağlam sıkıştırma](https://blog.langchain.com/autonomous-context-compression/)
- [sst/opencode bağlam yönetimi](https://deepwiki.com/sst/opencode/2.4-context-management-and-compaction)
- [DEV: Belirleyici vs. LLM Değerlendiriciler — 2026 ödünleşim çalışması](https://dev.to/anshd_12/deterministic-vs-llm-evaluators-a-2026-technical-trade-off-study-11h)
- [MadPlay: RTK %80 token azaltma deneyi](https://madplay.github.io/en/post/rtk-reduce-ai-coding-agent-token-usage)
- [Esteban Estrada: RTK %70 Claude Code azaltma](https://codestz.dev/experiments/rtk-rust-token-killer)

**GCOMPACTION.md kanonik bölümünün sonu.** Plan onayında, yukarıdaki her şey kelimesi kelimesine `docs/designs/GCOMPACTION.md`'ye bir **askıya alınmış tasarım eseri** olarak kopyalanır. Hiçbir kod yazılmaz; hiçbir hook kurulmaz; hiçbir CHANGELOG girdisi eklenmez. Doküman, Anthropic yerleşik araç çıktı-değiştirme API'sini gönderdiğinde gelecekteki bir sprint'in hızlıca engellemeyi kaldırabilmesi için var.