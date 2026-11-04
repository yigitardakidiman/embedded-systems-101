# Gömülü Yazılım ve Donanım Temelleri (Embedded 101)

Bu depo; elektrik-elektronik ve programlama temellerinden başlayarak donanıma yakın, güvenli ve verimli gömülü yazılım geliştirme sürecini adım adım belgeleyen kapsamlı bir eğitim arşividir.

Eğitim ilkeleri ve pedagojik kurallar [agent.md](agent.md) dosyasında tanımlanmıştır.

---

## 📌 Müfredat ve Ders Notları

Her konu; detaylı teorik anlatımını, donanımsal mikrodenetleyici karşılıklarını ve derste gerçekleştirilen tüm **interaktif soru-cevap / hata analizi kayıtlarını** içeren müstakil bir dokümana sahiptir:

| # | Konu Başlığı | Durum | Ders Notu & Soru-Cevaplar |
|:---:|---|:---:|:---:|
| **1** | **Bit, Byte ve Sayı Sistemleri** | Tamamlandı | [01_bit_byte_sayi_sistemleri.md](notes/01_bit_byte_sayi_sistemleri.md) |
| **2** | **Lojik Kapılar** | Tamamlandı | [02_lojik_kapilar.md](notes/02_lojik_kapilar.md) |
| **3** | **Elektriksel Sinyaller** | Tamamlandı | [03_elektriksel_sinyaller.md](notes/03_elektriksel_sinyaller.md) |
| **4** | **Programlama Temelleri (Gömülü C)** | Tamamlandı | [04_programlama_temelleri.md](notes/04_programlama_temelleri.md) |
| **5** | **Algoritma Kurma** | Tamamlandı | [05_algoritma_kurma.md](notes/05_algoritma_kurma.md) |
| **6** | **Geliştirme Ortamları ve İşletim Sistemleri** | Tamamlandı | [06_gelistirme_ortamlari.md](notes/06_gelistirme_ortamlari.md) |
| **7** | **Mikrodenetleyici Mimarisi** | Tamamlandı | [07_mikrodenetleyici_mimarisi.md](notes/07_mikrodenetleyici_mimarisi.md) |
| **-** | **Genel Değerlendirme Sınavı (1-7)** | Tamamlandı | [sinav_01_genel_degerlendirme.md](notes/sinav_01_genel_degerlendirme.md) |
| **8** | **Haberleşme Protokolleri (UART, I2C, SPI)** | *Devam Ediyor* | `notes/08_haberlesme_protokolleri.md` |
| **9** | **Pointer ve Struct** | Beklemede | `notes/09_pointer_ve_struct.md` |
| **10** | **Interrupt (Kesme)** | Beklemede | `notes/10_interrupt_kesme.md` |
| **11** | **Timer ve Zamanlama** | Beklemede | `notes/11_timer_ve_zamanlama.md` |
| **12** | **Debug ve Datasheet Okuma** | Beklemede | `notes/12_debug_ve_datasheet.md` |

---

## 🗂️ Dizin Yapısı

```text
embedded101/
├── agent.md                          # Eğitmen kuralları ve eğitim metodolojisi
├── README.md                         # Ana gösterge paneli ve müfredat haritası
└── notes/                            # Konu bazlı modüler ders notları ve soru-cevaplar
    ├── 01_bit_byte_sayi_sistemleri.md
    ├── 02_lojik_kapilar.md
    ├── 03_elektriksel_sinyaller.md
    ├── 04_programlama_temelleri.md
    └── ... (ilerledikçe eklenecekler)
```
