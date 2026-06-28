# Saha Ses Kayıtları - Proje Notları

Bu dosya, eniscakar.com (Hugo + PaperMod tabanlı field recording blogu) için
yapılan tüm önemli özelleştirmeleri ve çözümleri özetler. Yeni bir sohbette
yapay zekaya bu dosyayı yükleyerek hızlıca bağlam sağlayabilirsin.

---

## Genel Yapı

- **Sistem:** Hugo (statik site üreteci) + PaperMod teması
- **İşletim sistemi:** CachyOS (Arch tabanlı), `pacman` ile Hugo kurulu
- **Hugo sürümü:** 0.163.3 extended
- **Hosting:** Netlify (ücretsiz plan), GitHub'dan otomatik deploy
- **Domain:** eniscakar.com (WordPress'ten alınmış, Netlify DNS kullanıyor)
- **Proje klasörü:** ~/mysite
- **Tema, git submodule olarak eklendi:** themes/PaperMod

---

## Önemli Dosyalar ve Ne İşe Yaradıkları

### hugo.toml (ana yapılandırma)
- `baseURL = 'https://eniscakar.com/'`
- `locale = 'tr-TR'` (NOT: languageCode DEĞİL, bu Hugo sürümünde locale kullanılıyor)
- `theme = "PaperMod"`
- `[outputs] home = ["HTML", "RSS", "JSON"]` — JSON, arama özelliği için gerekli
- `[markup.goldmark.renderer] unsafe = true` — markdown içinde HTML kullanmaya izin verir (iframe, br, div vb.)
- `[params]`: defaultTheme = "light", disableThemeToggle = true (dark mode kapalı),
  ShowReadingTime = true, fuseOpts.* (arama ayarları)
- `[taxonomies]`: tag = "tags", category = "categories", species = "species"
  - DİKKAT: Taxonomy URL'si sağ taraftaki değerden gelir. "species" front matter
    alanı /species/ URL'sine dönüşür. Türkçe "turler" denenip çalışmadı, /species/ kullanıldı.
- Menüler: Hakkımda (/hakkimda/), Harita (/harita/), Türler (/species/), Ara (/search/)

### archetypes/posts.md (yeni post şablonu)
Yeni saha kaydı oluştururken otomatik gelen front matter şablonu.

### layouts/single.html (post detay sayfası - ÖZELLEŞTİRİLDİ)
- Cover SOLDA, içerik (ön metin + SoundCloud embed) SAĞDA yan yana düzen
- İçeriği iframe'e göre bölüyor: iframe ÖNCESİ = ön metin (sağ üstte),
  iframe + sonrasındaki SoundCloud attribution div'i = embed (sağ altta),
  geri kalan (galeri, habitat bilgisi vb.) = ana içerik (altta tam genişlik)
- ÖNEMLİ: SoundCloud kodu <iframe>...</iframe><div>...attribution...</div> şeklinde gelir.
  Bu div'i de embed'e dahil etmek için regex: <iframe.*?</iframe>(\s*<div.*?</div>)?

### layouts/list.html (ana sayfa ve liste sayfaları - ÖZELLEŞTİRİLDİ)
- Ana sayfa kartları da cover solda / içerik sağda düzeninde
- $hasCover kontrolü: cover'ı OLAN postlar yan-yana, OLMAYAN postlar standart tek-sütun
  (gelecekte saha kaydı dışı post eklendiğinde bozulmaması için)
- SoundCloud embed, içerikten regex ile ayıklanıp ayrı gösteriliyor
- Sıralama: record_date'e göre (sort $pages "Params.record_date" "desc")
  ÇÜNKÜ: date alanı hepsinde aynı (aynı gün oluşturuldu), gerçek kayıt tarihi record_date'te

### layouts/harita.html (interaktif harita - layout: harita)
- Leaflet.js (CDN) tabanlı, OpenStreetMap
- Her postun coordinates alanından otomatik pin üretir
- Pin'e tıklayınca: cover fotoğrafı + başlık + "Postu Gör" linki (popup)
- fitBounds ile pinleri otomatik kapsar (post eklendikçe genişler), maxZoom: 8
- Başlangıç: Anadolu merkezli [39.0, 35.0], zoom 6
- ÖNEMLİ: coordinates string'i jsonify ile değil, "{{ }}" + htmlEscape ile gömülmeli
  (jsonify çift tırnak sorunu yaratıyordu, linkler bozuluyordu)

### layouts/partials/extend_head.html
- GLightbox CSS + Leaflet CSS linkleri

### layouts/partials/extend_footer.html
- GLightbox JS — galeri fotoğraflarına lightbox (.post-content img seçicisiyle)

### assets/css/extended/custom.css (tüm özel stiller)
- --main-width: 1200px (sayfa genişliği)
- .post-cover-flex / .post-entry-flex: cover-içerik yan yana düzen (%40 / %55)
  - justify-content: center ile dikey ortalama
- .tag-entry .entry-cover { display: block !important } — tür sayfalarında cover'ı göster
  (PaperMod varsayılan olarak tag sayfalarında cover'ı gizliyor)
- .entry-embed, iframe margin-top: 16px (boşluklar)

### content/species/_index.md
- title = "Türler" — taxonomy sayfasının başlığını "species" yerine "Türler" yapar

### static/css/glightbox.min.css, static/js/glightbox.min.js
- Lightbox kütüphanesi (CDN'den indirilip yerele kondu, internet bağımsız çalışır)

### netlify.toml (Netlify build ayarları)
- HUGO_VERSION = "0.163.3"
- command = "hugo --gc --minify"
- publish = "public"

---

## Yeni Post Ekleme Akışı

```bash
cd ~/mysite
hugo new content posts/yeni-post-adi/index.md   # page bundle olarak (index.md ŞART)
```

Front matter'da doldurulacaklar:
- draft: false  (YAYINLAMA İÇİN ŞART — draft: true canlıda 404 verir!)
- cover.image: "cover.jpeg"
- habitat, species (liste: [Ardıç, karaçam]), location
- record_date: 2025-07-31  (TIRNAKSIZ, YYYY-MM-DD formatı — sıralama için kritik)
- coordinates: "37.784634, 35.058182"  (harita için)

Görseller (cover.jpeg, photo1.jpeg vb.) aynı klasöre konulur.
SoundCloud iframe kodu ön metin ile ana metin arasına yapıştırılır.

Yayınlama:
```bash
git add .
git commit -m "Yeni post"
git push   # Netlify otomatik deploy eder
```

---

## Çözülen Önemli Sorunlar (Tekrar Yaşanırsa)

1. **SoundCloud embed çalışmıyor:** hugo.toml'da unsafe = true gerekli.

2. **Cover görünmüyor:** Front matter'da PaperMod formatı kullanılmalı:
   cover: { image, alt, caption, relative: true }. Görsel index.md ile aynı klasörde olmalı.
   Page bundle için dosya adı MUTLAKA index.md (duyuru-test.md gibi değil).

3. **Galeri lightbox açmıyor:** GLightbox eklendi (extend_head + extend_footer).

4. **Taxonomy 404 / boş:** [taxonomies] tanımında URL sağ taraftaki değerden gelir.
   Türkçe isim çalışmadı, /species/ kullanıldı. _index.md ile başlık "Türler" yapıldı.

5. **Sıralama tesadüfi:** date alanı aynı olunca rastgele sıralıyordu. record_date'e
   göre sort eklendi. record_date TIRNAKSIZ ve YYYY-MM-DD olmalı.

6. **Harita pini linkleri çalışmıyor:** jsonify çift tırnak sorunu. "{{ .Permalink }}"
   ve htmlEscape kullanıldı.

7. **Canlıda 404 ama yerelde çalışıyor:** draft = true unutulmuş. -D sadece yerelde
   draft gösterir, canlı build draft'ları dahil etmez.

8. **Netlify "hugo: command not found":** netlify.toml ile HUGO_VERSION belirtildi.

---

## Olası Gelecek Geliştirmeler (Tartışıldı)

- RSS/podcast beslemesi
- Mevsim/tarih bazlı arşiv görünümü
- Kayıt ekipmanı sayfası
- Türlerin Latince isimleri
- Footer özelleştirme (şimdilik "Powered by" bırakıldı, açık kaynağa saygı)

---

## Genel Çalışma Notları

- Yerel test: `hugo server -D` → http://localhost:1313 (baseURL'den etkilenmez, görseller hep çalışır)
- Cache sorunu olursa: `rm -rf public resources` sonra tekrar build
- Tema güncelleme (opsiyonel): `git submodule update --remote --merge`
- Markdown içinde HTML kullanılabilir (unsafe = true sayesinde) — örn. <br> ile boşluk
