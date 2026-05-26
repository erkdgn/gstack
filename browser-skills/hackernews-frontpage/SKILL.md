---
name: hackernews-frontpage
description: Hacker News ana sayfasını kazır (başlıklar, puanlar, yorum sayıları).
host: news.ycombinator.com
trusted: true
source: human
version: 1.0.0
args: []
triggers:
  - scrape hacker news frontpage
  - scrape hn frontpage
  - get hn top stories
  - latest hacker news stories
---

# Hacker News ana sayfa kazıyıcısı

Hacker News (`news.ycombinator.com`) ana sayfasını kazır ve en
iyi 30 haberi JSON olarak döndürür. Her haberin sırası, başlığı, bağlantı URL'si,
puan sayısı ve yorum sayısı bulunur.

## Kullanım

```
$ $B skill run hackernews-frontpage
{
  "stories": [
    { "rank": 1, "title": "...", "url": "...", "points": 412, "comments": 87 },
    ...
  ],
  "count": 30
}
```

## Nasıl çalışır

1. Daemon üzerinden `https://news.ycombinator.com` adresine gider.
2. Sayfa HTML'ini okur.
3. Her haber satırını (HN'nin kararlı `tr.athing` yapısı) tiplendirilmiş
   `Story` kaydına dönüştürür.
4. Standart çıktıya tek bir JSON belgesi yazar.

## Neden bu referans yetenek

`hackernews-frontpage`, en küçük ilginç browser-skill'dir: kimlik doğrulama yok,
kararlı HTML, deterministik çıktı, dosya-fikstürü-uyumlu. Her Aşama 1
bileşeni (SDK, kapsamlı token'lar, üç katmanlı arama, spawn yaşam döngüsü)
`$B skill run hackernews-frontpage` ve paketlenen
`script.test.ts` tarafından kullanılır.

HN HTML'si değiştiğinde ve seçicilerimiz bozulduğunda, test kaydedilen
fikstüre karşı kullanıcılar fark etmeden önce başarısız olur. İşte amaç bu.