# Conductor Oturum Akışı API Önerisi

## Sorun

Claude gerçek tarayıcınızı CDP üzerinden kontrol ettiğinde (gstack `$B connect`),
iki pencereye bakarsınız: **Conductor** (Claude'un düşünmesini görmek için) ve
**Chrome** (Claude'un eylemlerini görmek için).

gstack'ın Chrome uzantısı Yan Panel tarama etkinliğini gösterir — her komutu, sonucu
ve hatayı. Ama *tam* oturum yansıtma (Claude'un düşünmesi, araç çağrıları, kod düzenlemeleri)
için Yan Panel'in Conductor'ın konuşma akışını açığa çıkarmasına ihtiyacı var.

## Bu ne sağlar

gstack Chrome uzantisi Yan Panel'inde, şunları gösteren bir "Oturum" sekmesi:

- Claude'un düşünme/içerik (performans için kesilmiş)
- Araç çağrısı isimleri + simgeler (Düzenle, Bash, Oku, vb.)
- Maliyet tahminleri ile dönüş sınırları
- Konuşma ilerledikçe gerçek zamanlı güncellemeler

Kullanıcı her şeyi tek bir yerde görür — tarayıcısında Claude'un eylemleri + Yan Panel'de
Claude'un düşünmesi — pencereler arasında geçiş yapmadan.

## Önerilen API

### `GET http://127.0.0.1:{PORT}/workspace/{ID}/session/stream`

Claude Code'un konuşmasını NDJSON etkinlikleri olarak yeniden yayayan Sunucu-Gönderilen
Etkinlikler uç noktası.

**Etkinlik türleri** (Claude Code'un `--output-format stream-json` formatını yeniden kullanır):

```
event: assistant
data: {"type":"assistant","content":"Şu sayfayı kontrol edeyim...","truncated":true}

event: tool_use
data: {"type":"tool_use","name":"Bash","input":"$B snapshot","truncated_input":true}

event: tool_result
data: {"type":"tool_result","name":"Bash","output":"[snapshot çıktısı...]","truncated_output":true}

event: turn_complete
data: {"type":"turn_complete","input_tokens":1234,"output_tokens":567,"cost_usd":0.02}
```

**İçerik kesme:** Araç girdileri/çıktıları akışta 500 karakterle sınırlıdır. Tam
veri Conductor'ın kullanıcı arayüzünde kalır. Yan Panel bir özet görünümüdür, bir
yedek değil.

### `GET http://127.0.0.1:{PORT}/api/workspaces`

Aktif çalışma alanlarını listeleyen keşif uç noktası.

```json
{
  "workspaces": [
    {
      "id": "abc123",
      "name": "gstack",
      "branch": "garrytan/chrome-extension-ctrl",
      "directory": "/Users/garry/gstack",
      "pid": 12345,
      "active": true
    }
  ]
}
```

Chrome uzantısı, tarama sunucusunun git deposunu ((`/health` yanıtından) bir çalışma
alanının dizini veya adıyla eşleştirerek çalışma alanını otomatik olarak seçer.

## Güvenlik

- **Sadece yerel ana bilgisayar.** Claude Code'un kendi hata ayıklama çıktısıyla aynı
  güven modeli.
- **Kimlik doğrulama gerekmez.** Conductor kimlik doğrulama isterse, uzantının SSE
  isteklerinde ilettiği bir Bearer belirteci çalışma alanı listelemesine ekleyin.
- **İçerik kesme** bir gizlilik özelliğidir — uzun kod çıktıları, dosya içerikleri
  ve hassas araç sonuçları asla Conductor'ın tam kullanıcı arayüzünden ayrılmaz.

## gstack ne inşa eder (uzantı tarafı)

Yan Panel "Oturum" sekmesinde zaten iskelelendirilmiş (şu anda yer tutucu gösteriyor).

Conductor'ın API'si mevcut olduğunda:
1. Yan Panel, port araştırması veya manuel giriş ile Conductor'ı keşfeder
2. `/api/workspaces`'ı getirir, tarama sunucusunun deposuyla eşleştirir
3. `/workspace/{id}/session/stream`'e `EventSource` açar
4. Render eder: asistan mesajları, araç isimleri + simgeler, dönüş sınırları, maliyet
5. Zarifçe geri döner: "Tam oturum görünümü için Conductor'ı bağlayın"

Tahmini çaba: `sidepanel.js`'de ~200 SATIR.

## Conductor ne inşa eder (sunucu tarafı)

1. Çalışma alanı başına Claude Code'un stream-json'unu yeniden yayayan SSE uç noktası
2. Aktif çalışma alanı listesi ile `/api/workspaces` keşif uç noktası
3. İçerik kesme (araç girdileri/çıktılarında 500 karakter sınırı)

Tahmini çaba: Conductor zaten Claude Code akışını kendi kullanıcı arayüzü
render etme için dahili olarak yakalıyorsa ~100-200 SATIR.

## Tasarım kararları

| Karar | Seçim | Gerekçe |
|----------|--------|-----------|
| Taşıyıcı | SSE (WebSocket değil) | Tek yönlü, otomatik yeniden bağlanma, daha basit |
| Format | Claude'ın stream-json'u | Conductor bunu zaten ayrıştırıyor; yeni şema yok |
| Keşif | HTTP uç noktası (dosya değil) | Chrome uzantıları dosya sistemini okuyamaz |
| Kimlik doğrulama | Yok (yerel ana bilgisayar) | Tarama sunucusu, CDP portu, Claude Code ile aynı |
| Kesme | 500 karakter | Yan Panel ~300px genişliğinde; uzun içerik işe yaramaz |