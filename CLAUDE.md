# 9/D Beden Eğitimi İstatistik Sitesi

## Proje Özeti
9-D sınıfı beden eğitimi derslerindeki futbol maçlarını, oyuncu istatistiklerini ve kadro dengeleme sistemini yöneten tek sayfalık (SPA) bir web uygulaması.

**Canlı Site:** https://mehmet-cinart.github.io/9-D-mac_istatistikleri/9dbedenistatistikler.html
**GitHub:** https://github.com/Mehmet-CinarT/9-D-mac_istatistikleri
**Renk Şeması:** Beyaz (#FFFFFF) + Mavi (#1E90FF)

---

## Dosya Yapısı

```
9-D Beden İstatistikler/
├── 9dbedenistatistikler.html    # Ana site dosyası (tek dosya: HTML + CSS + JS)
├── saha.png                      # Futbol sahası görseli (maç detay modalında kullanılır)
├── todo.md                       # Yapılacaklar listesi
├── CLAUDE.md                     # Bu dosya
├── Veriler/
│   ├── Maçlar ve Maç İstatikleri/
│   │   ├── Maç Tahtası.png       # Saha görseli (orijinal)
│   │   └── Maç Sonuçları.xlsx    # Excel şablonu
│   ├── Oyuncular ve İstatistikleri/
│   │   └── Oyuncular ve İstatistikleri.xlsx  # Oyuncu bilgileri
│   └── Gol ve Asist Krallığı Listesi/       # (boş dizin)
└── .claude/
    └── settings.json
```

---

## Teknolojiler

- **Frontend:** Vanilla HTML/CSS/JS (tek dosya, build sistemi yok)
- **Veritabanı:** Firebase Realtime Database (anlık senkronizasyon)
- **Hosting:** GitHub Pages (ücretsiz)
- **Firebase SDK:** CDN üzerinden yükleniyor (v10.12.0 compat)

---

## Firebase Yapılandırması

- **Proje:** `d-beden-istatistik`
- **Database URL:** `https://d-beden-istatistik-default-rtdb.europe-west1.firebasedatabase.app`
- **Veri yapısı:** `9d_matches` anahtarı altında maç dizisi tutuluyor
- **Oyuncu verileri:** Firebase'de tutulmuyor, `defaultPlayers` dizisinden geliyor ve maçlardan hesaplanıyor
- **Dinleme:** `db.ref('9d_matches').on('value', ...)` ile anlık güncelleme

> **ÖNEMLİ:** Firebase test modu 30 gün sonra sona erer. Rules kısmında `".read": true, ".write": true` olarak güncellenmeli.

---

## Giriş Sistemi (İki Mod)

### Gözlemci Paneli
- Şifre gerektirmez
- Maçları, oyuncu istatistiklerini, krallık listesini görüntüleyebilir
- Maç detaylarına tıklayabilir, saha dizilişini görebilir
- **Yapamaz:** Maç ekleme/silme/sıralama, kadro kurucu, Excel'e aktarma

### Kontrol Paneli
- **Şifre:** `kontrolpaneligiris1970`
- Tüm özellikler açık
- CSS sınıfı: `body.control-mode` eklenince `.control-only` elemanlar görünür
- JS düzeyinde de koruma var: `isControlMode()` fonksiyonu kontrol eder

---

## Sekmeler ve Özellikler

### 1. Ana Sayfa (`tab-anasayfa`)
- Özet istatistik kartları: Toplam Oyuncu, Toplam Maç, Toplam Gol, Toplam Asist
- Son 2 maç kartı
- Oyuncu kadro tablosu (tıklanabilir → oyuncu profili modalı)

### 2. Oyuncular (`tab-oyuncular`)
- Tüm oyuncular tablosu (G+A sıralamalı)
- Her oyuncuya tıklayınca **profil modalı** açılır:
  - Mevkiler, kullanılan ayak
  - Gol, asist, maç, G+A, galibiyet, beraberlik, mağlubiyet, kazanma oranı
  - Son 5 maç (G/B/M ikonu, skor bold, maç başı gol/asist)
- Excel'e Aktar butonu (kontrol paneli)

### 3. Maçlar (`tab-maclar`)
- **Maç ekleme formu** (kontrol paneli):
  - Skor girişi, tarih seçimi
  - Takım 1 oyuncuları seçildiğinde geri kalanlar otomatik Takım 2'ye atanır
  - Gol/asist olayları: takım skoruna göre limit kontrolü (anlık engelleme)
  - Aynı oyuncunun birden fazla olay satırı birleştirilerek kaydedilir
- **Maç kartları** (hafta sırasıyla):
  - Skor, kazanan bilgisi, gol/asist etiketleri
  - Tıklanınca **maç detay modalı** açılır
  - Yukarı/aşağı sıralama butonları + sil butonu (kontrol paneli)
- **Maç detay modalı:**
  - Skor ve sonuç
  - Saha dizilişi (saha.png üzerinde oyuncular mevkilerine göre konumlandırılır)
  - Takım 1: sol yarı saha, Takım 2: sağ yarı saha
  - Oyuncu isimleri sadece ilk isim gösterilir
  - Kadro listesi ve maç olayları (gol+asist yan yana birleştirilmiş)
- Excel'e Aktar butonu (kontrol paneli)

### 4. Gol & Asist Krallığı (`tab-krallik`)
- Gol krallığı sıralaması (altın/gümüş/bronz renkleri)
- Asist krallığı sıralaması

### 5. Kadro Kurucu (`tab-kadro`, sadece kontrol paneli)
- Takım 1 ve Takım 2 kişi sayısı girişi (min 4, varsayılan 6)
- Gelmeyen oyuncuları seçme sistemi (toplam 12'den az oyuncu gerektiğinde)
- Dengeli kadro oluşturma algoritması

---

## Kadro Kurucu Algoritması

### Oyuncu Güç Puanları (gizli, sitede gösterilmez)
Puanlar kod içinde `playerRatings` objesinde tutulur. Sitede hiçbir yerde gösterilmez.

### Sabit Mevki Atamaları (`fixedPositions`)
- Mehmet Çınar Turşak (id:1) → her zaman **ST**
- Fatih Doğanay (id:2) → her zaman **ST**
- Kağan Durak (id:3) → normalde **MOO**, ama Fatih ve Mehmet Çınar aynı takımdaysa **ST**
- Egemen Işık (id:6) → her zaman **MOO**

### Dağılım Kuralları
1. **Yusuf (id:7) ve Ömer (id:8)** her zaman farklı takımlarda
2. **Mehmet Çınar (id:1)** %50 ihtimalle Yusuf'la aynı, %50 karşı takımda
3. **Stoperler 3-2 bölüşülür:** Yusuf'un takımında 3 STP (dahil), Ömer'in takımında 2 STP (dahil)
4. **Kaleciler ve forvetler** rastgele dağılır (bazen aynı takımda olabilir)
5. **Her takımda her mevki kategorisinden** (KL, STP, MO, ST) en az 1 oyuncu
6. Kalan oyuncular kişi başı ortalama güce göre dengeli dağıtılır

### Mevki Kategorileri
- `KL` → Kaleci
- `STP` → Savunma (STP, MDO)
- `MO` → Orta Saha (MO, MOO)
- `ST` → Forvet (ST, SLK, SĞK)

### Kadro Listesi Sıralaması
Kaleci → Defans → Orta Saha → Forvet (hat başlıkları ile ayrılmış)

---

## Oyuncu Kadrosu (12 kişi)

| # | Ad Soyad | ID | Mevkiler | Ayak |
|---|----------|-----|----------|------|
| 1 | Mehmet Çınar Turşak | 1 | ST, MOO, SLK, SĞK | Sağ |
| 2 | Fatih Doğanay | 2 | ST, MOO, SLK, SĞK | Sağ |
| 3 | Kağan Durak | 3 | ST, MOO, SLK, SĞK | Sağ |
| 4 | Mehmet Ali Çınar | 4 | MOO, MO | Sağ |
| 5 | Mehmet Efe Güvenmez | 5 | MOO, MO | Sağ |
| 6 | Egemen Işık | 6 | MO, ST | Sağ |
| 7 | Yusuf Karadağ | 7 | STP, MO, MDO | Sağ |
| 8 | Ömer Ege Durmuşoğlu | 8 | STP, MO, MDO | Sağ |
| 9 | Ertuğrul Gazi Kuzucu | 9 | STP | Sağ |
| 10 | Eren Kaya | 10 | STP | Sağ |
| 11 | Boran Mir Şenoğlu | 11 | KL, STP | Sağ |
| 12 | Ege Demir Şen | 12 | KL | Sağ |

---

## Mevki Kısaltmaları

| Kısaltma | Mevki |
|----------|-------|
| KL | Kaleci |
| STP | Stoper |
| MDO | Merkez Defansif Orta Saha |
| MO | Merkez Orta Saha |
| MOO | Merkez Ofansif Orta Saha |
| SLK | Sol Kanat |
| SĞK | Sağ Kanat |
| ST | Santrfor |

---

## Saha Dizilişi Sistemi

Maç detay modalında `saha.png` üzerinde oyuncular konumlandırılır:
- **Takım 1:** Sol yarı saha (%2-%48 arası left değerleri)
- **Takım 2:** Sağ yarı saha (%52-%98 arası left değerleri)
- Her hat (KL, STP+MDO, MO+MOO, ST+SLK+SĞK) aynı yatay çizgide
- Aynı hattaki oyuncular dikeyde (%15-%85 arası) eşit aralıklarla dağıtılır
- Oyuncu isimleri sadece **ilk isim** olarak gösterilir
- Takım 1: mavi (#1E90FF), Takım 2: kırmızı (#e74c3c) noktalar

---

## Veri Akışı

1. **Maç kaydedildiğinde:** `saveData('9d_matches', matches)` → Firebase'e yazılır
2. **Firebase listener:** `db.ref('9d_matches').on('value', ...)` → `matches` güncellenir → `recalcPlayerStats()` → `renderAll()`
3. **İstatistik hesaplama:** `recalcPlayerStats()` tüm oyuncuların gol/asist/maç sayılarını maçlardan yeniden hesaplar
4. **Anlık senkronizasyon:** Bir cihazda yapılan değişiklik tüm cihazlarda anında görünür

---

## CSS Yapısı

Tüm stiller `<style>` etiketi içinde, harici dosya yok. Ana bölümler:
- Header (sticky, gradient mavi)
- Nav tabs
- Stat cards (grid)
- Table wrapper
- Modal overlay + modal
- Match cards (border-left mavi)
- Pitch container (absolute positioning)
- Team builder
- King cards (gol: yeşil gradient, asist: turuncu gradient)
- Login screen (overlay)
- Responsive breakpoint: 768px

---

## Geliştirme Notları

- Tüm proje tek HTML dosyasında (inline CSS + JS)
- Build sistemi, framework veya paket yöneticisi yok
- Oyuncu listesi değiştiğinde `defaultPlayers` dizisi güncellenmeli
- Excel dosyalarını okumak için `openpyxl` Python paketi kullanılabilir
- GitHub push için `git push origin main` yeterli
- Firebase API key'i client-side'da açık, bu normal (güvenlik Rules ile sağlanır)

---

## Otomatik Commit & Push Kuralı

**Her değişiklikten sonra otomatik olarak commit atılıp push edilmeli.**

Değişiklik yapıldığında şu adımlar izlenir:
1. `git add` ile değişen dosyalar eklenir
2. Türkçe ve açıklayıcı bir commit mesajı yazılır
3. `git push origin main` ile GitHub'a gönderilir

Remote URL: `https://github.com/Mehmet-CinarT/9-D-mac_istatistikleri.git`
Branch: `main`
