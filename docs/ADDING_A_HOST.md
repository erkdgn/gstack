# gstack'e Yeni Host Ekleme

gstack bildirimsel bir host yapılandırma sistemi kullanır. Desteklenen her AI kodlama
aracısı (Claude, Codex, Factory, Kiro, OpenCode, Slate, Cursor, OpenClaw) tipli bir
TypeScript yapılandırma nesnesi olarak tanımlanır. Yeni bir host eklemek bir doska
oluşturmak ve yeniden dışa aktarmak anlamına gelir. Üreteç, kurulum veya araçlarda sıfır
kod değişikliği.

## Nasıl çalışır

```
hosts/
├── claude.ts        # Birincil host
├── codex.ts         # OpenAI Codex CLI
├── factory.ts       # Factory Droid
├── kiro.ts          # Amazon Kiro
├── opencode.ts      # OpenCode
├── slate.ts         # Slate (Random Labs)
├── cursor.ts        # Cursor
├── openclaw.ts      # OpenClaw (karma: yapılandırma + bağdaştırıcı)
└── index.ts         # Kayıt defteri: tümünü içe aktarır, Host türünü türetir
```

Her yapılandırma dosyası, üretece şunu söyleyen bir `HostConfig` nesnesi dışa aktarır:
- Üretilen yeteneklerin nereye konacağı (yollar)
- Frontmatter'ı nasıl dönüştüreceği (izin listesi/ret listesi alanları)
- Hangi Claude'a özgü referansların yeniden yazılacağı (yollar, araç adları)
- Otomatik kurulum için hangi ikili dosyanın algılanacağı
- Hangi çözücü bölümlerin bastırılacağı
- Kurulum zamanında hangi varlıkların sembolik bağlanacağı

Üreteç, kurulum betiği, platform-algılama, kaldırma, sağlık denetimleri, worktree
kopyası ve testlerin hepsi bu yapılandırmalardan okunur. Hiçbirinde host başına kod yoktur.

## Adım adım: yeni bir host ekle

### 1. Yapılandırma dosyasını oluştur

Başlangıç noktası olarak var olan bir yapılandırmayı kopyalayın. `hosts/opencode.ts` iyi
bir minimal örnektir. `hosts/factory.ts` araç yeniden yazımlarını ve koşullu alanları
gösterir. `hosts/openclaw.ts` farklı araç modellerine sahip hostlar için bağdaştırıcı
örüntüsünü gösterir.

`hosts/myhost.ts` oluştur:

```typescript
import type { HostConfig } from '../scripts/host-config';

const myhost: HostConfig = {
  name: 'myhost',
  displayName: 'MyHost',
  cliCommand: 'myhost',        // `command -v` algılaması için ikili dosya adı
  cliAliases: [],              // alternatif ikili dosya adları

  globalRoot: '.myhost/skills/gstack',
  localSkillRoot: '.myhost/skills/gstack',
  hostSubdir: '.myhost',
  usesEnvVars: true,           // yalnızca Claude için false (harfi harfine ~ yolları kullanır)

  frontmatter: {
    mode: 'allowlist',         // 'allowlist' yalnızca listelenen alanları tutar
    keepFields: ['name', 'description'],
    descriptionLimit: null,    // limitleri olan hostlar için 1024 olarak ayarlayın
  },

  generation: {
    generateMetadata: false,   // yalnızca Codex için true (openai.yaml)
    skipSkills: ['codex'],     // codex yeteneği yalnızca Claude'a özeldir
  },

  pathRewrites: [
    { from: '~/.claude/skills/gstack', to: '~/.myhost/skills/gstack' },
    { from: '.claude/skills/gstack', to: '.myhost/skills/gstack' },
    { from: '.claude/skills', to: '.myhost/skills' },
  ],

  runtimeRoot: {
    globalSymlinks: ['bin', 'browse/dist', 'browse/bin', 'gstack-upgrade', 'ETHOS.md'],
    globalFiles: { 'review': ['checklist.md', 'TODOS-format.md'] },
  },

  install: {
    prefixable: false,
    linkingStrategy: 'symlink-generated',
  },

  learningsMode: 'basic',
};

export default myhost;
```

### 2. İndekse kaydet

`hosts/index.ts` dosyasını düzenle:

```typescript
import myhost from './myhost';

// ALL_HOST_CONFIGS dizisine ekle:
export const ALL_HOST_CONFIGS: HostConfig[] = [
  claude, codex, factory, kiro, opencode, slate, cursor, openclaw, myhost
];

// Yeniden dışa aktarmalara ekle:
export { claude, codex, factory, kiro, opencode, slate, cursor, openclaw, myhost };
```

### 3. .gitignore'a ekle

`.myhost/` satırını `.gitignore` dosyasına ekle (üretilen yetenek belgeleri gitignore edilir).

### 4. Üret ve doğrula

```bash
# Yeni host için yetenek belgelerini üret
bun run gen:skill-docs --host myhost

# Çıktının var olduğunu ve .claude/skills sızıntısı olmadığını doğrula
ls .myhost/skills/gstack-*/SKILL.md
grep -r ".claude/skills" .myhost/skills/ | head -5
# (boş olmalı)

# Tüm hostlar için üret (yenisini de içerir)
bun run gen:skill-docs --host all

# Sağlık panosu yeni hostu gösterir
bun run skill:check
```

### 5. Testleri çalıştır

```bash
bun test test/gen-skill-docs.test.ts
bun test test/host-config.test.ts
```

Parametreli duman testleri yeni hostu otomatik olarak algılar. Yazılacak sıfır test
kodu. Şunları doğrularlar: çıktı var, yol sızıntısı yok, geçerli frontmatter,
tazelik denetimi geçer, codex yeteneği hariç tutulur.

### 6. README.md'yi güncelle

Yeni host için kurulum talimatlarını uygun bölüme ekle.

## Yapılandırma alanı referansı

Her alanda JSDoc yorumlarıyla birlikte tüm `HostConfig` arayüzü için
`scripts/host-config.ts` dosyasına bakın.

Temel alanlar:

| Alan | Amacı |
|-------|---------|
| `frontmatter.mode` | `allowlist` (yalnızca listelenenleri tut) veya `denylist` (listelenenleri çıkar) |
| `frontmatter.descriptionLimit` | Maksimum karakter, sınır yok için `null` |
| `frontmatter.descriptionLimitBehavior` | `error` (derleme başarısız), `truncate`, `warn` |
| `frontmatter.conditionalFields` | Şablon değerlerine göre alan ekle (örn., sensitive → disable-model-invocation) |
| `frontmatter.renameFields` | Şablon alanlarını yeniden adlandır (örn., voice-triggers → triggers) |
| `pathRewrites` | İçerikte harfi harfine replaceAll. Sıra önemlidir. |
| `toolRewrites` | Claude araç adlarını yeniden yaz (örn., "use the Bash tool" → "run this command") |
| `suppressedResolvers` | Bu host için boş dönen çözücü işlevleri |
| `coAuthorTrailer` | Commitler için git ortak yazar dizesi |
| `boundaryInstruction` | Çapraz model çağrıları için istem-enjeksiyonu karşıtı uyarı |
| `adapter` | Karmaşık dönüşümler için bağdaştırıcı modülü yolu |

## Bağdaştırıcı örüntüsü (farklı araç modellerine sahip hostlar için)

Dizi-değiştirme araç yeniden yazımları yeterli değilse (hostun temelden farklı araç
anlambilimi varsa), bağdaştırıcı örüntüsünü kullanın. `hosts/openclaw.ts` ve
`scripts/host-adapters/openclaw-adapter.ts` dosyalarına bakın.

Bağdaştırıcı, tüm genel yeniden yazımlardan sonra bir son işleme adımı olarak çalışır.
`transform(content: string, config: HostConfig): string` işlevini dışa aktarır.

## Doğrulama

`scripts/host-config.ts` içindeki `validateHostConfig()` işlevi şunları denetler:
- İsim: tireli küçük harf alfanümerik
- CLI komutu: tireli/altçizgili alfanümerik
- Yollar: yalnızca güvenli karakterler (alfanümerik, `.`, `/`, `$`, `{}`, `~`, `-`, `_`)
- Yapılandırmalar arasında yinelenen isim, hostSubdirs veya globalRoot yok

Tüm yapılandırmaları denetlemek için `bun run scripts/host-config-export.ts validate` çalıştırın.