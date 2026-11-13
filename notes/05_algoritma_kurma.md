# 5. Konu: Algoritma Kurma

Bu doküman; gömülü sistemlerde doğrudan kodlamaya geçmeden önce problemin analiz edilmesi, Girdi-Süreç-Çıktı (IPO) modelinin kurulması, akış diyagramları ve durum değişkeni (state/flag) mantığıyla sözde kod oluşturma adımlarını içerir.

---

## 5.1 Problemi Analiz Etme ve Girdi - Süreç - Çıktı (IPO) Modeli

Gömülü sistemler fiziksel dünya ile etkileşime giren sistemlerdir. Kod yazmaya başlamadan önce sistem IPO modeliyle ayrıştırılır:

1. **Girdiler (Inputs):** Dış dünyadan mikrodenetleyiciye gelen elektriksel veriler (Sensörler, butonlar, anahtarlar, haberleşme paketleri).
2. **Süreç (Process):** Mikrodenetleyicinin bu veriyi anlamlandırması, matematiksel/mantıksal kararlar alması ve durum kontrolleri.
3. **Çıktılar (Outputs):** Sistemin fiziksel dünyaya verdiği tepkiler (Motor sürücüleri, röleler, LED'ler, ekranlar, buzzer).

- **Uç Durum (Edge Case / Güvenlik Analizi):** Normal çalışma akışı dışındaki risklerin (kablo kopması, sensör arızası, sıkışma vb.) baştan tespit edilip algoritmaya güvenlik önlemlerinin eklenmesidir.

```mermaid
flowchart LR
    subgraph I["GİRDİLER (Inputs)"]
        I1["Bilet Butonu"]
        I2["Araç Algılama Sensörü"]
        I3["Güvenlik Fotoseli (Edge Case)"]
    end

    subgraph P["SÜREÇ (MCU Karar)"]
        P1["Durum ve Güvenlik Kontrolü"]
    end

    subgraph O["ÇIKTILAR (Outputs)"]
        O1["Bariyer Motoru (Yukarı/Aşağı)"]
    end

    I1 & I2 & I3 --> P1 --> O1
```

### Soru - Cevap
> **Soru:** Otomatik otopark bariyer sistemi için: 1) Girdiler, 2) Çıktılar, 3) Düşünülmesi gereken en az bir Uç Durum (Edge Case) nedir?  
> **Öğrenci Cevabı:**  
> 1) Kamera, hareket sensörü (araç algılama), bilet butonu, geçiş fotoseli.  
> 2) Motorun dönüp bariyeri kaldırması (motor sürücü/röle çıkışı).  
> 3) Motorun çok hızlı kapanıp bariyeri geçmek üzere olan arabanın üstüne kapatması.  
> **Değerlendirme:** Doğru. Bariyer aşağı inerken araç geçiş fotoseli araya bir cisim girdiğini algılarsa iniş iptal edilip bariyer tekrar yukarı çekilmelidir.

---

## 5.2 Akış Diyagramları (Flowcharts) ve Temel Şekiller

Mantıksal akışı görselleştirmek için kullanılan uluslararası semboller:

- **Oval (Elips):** `BAŞLA` / `BİTİR` terminalleri.
- **Dikdörtgen (İşlem / Process):** Donanımsal bir eylem veya değişken ataması (`motor_calistir()`, `sayac = 0`).
- **Paralelkenar (Girdi/Çıktı - I/O):** Dış dünyadan veri alma veya veri gönderme (`buton_oku()`, `adc_oku()`).
- **Eşkenar Dörtgen / Baklava (Karar / Decision):** Mantıksal şart (`if`). İçinden iki çıkış oku çıkar: **EVET (True)** ve **HAYIR (False)**.
- **Geri Dönen Oklar:** Bir işlemin ardından akışın başa dönmesi gömülü sistemlerdeki `while(1)` sonsuz döngüsünü temsil eder.

| Sembol | Şekil | Gömülüdeki Anlamı |
|---|---|---|
| **Terminal** | `([ BAŞLA ])` | Programın başlangıcı / bitişi |
| **Girdi / Çıktı (I/O)** | `[/ Pin Oku /]` | Sensör, buton veya UART verisi okuma |
| **İşlem (Process)** | `[ İşlem ]` | Değişken güncelleme, hesaplama, çıkış verme |
| **Karar (Decision)** | `{ Şart? }` | `if` koşulu (EVET / HAYIR olmak üzere 2 çıkış) |

```mermaid
flowchart TD
    S(["BAŞLA"]) --> IO[/"Buton Durumunu Oku"/]
    IO --> Q{"Butona Basıldı mı?"}
    Q -- "EVET" --> A1["LED'i Yak (HIGH)"]
    Q -- "HAYIR" --> A2["LED'i Söndür (LOW)"]
    A1 --> IO
    A2 --> IO
```

### Soru - Cevap
> **Soru:** Bir mikrodenetleyicide butona basılınca LED yakan, basılmayınca söndüren ve sürekli çalışan sistemin akış şemasında:  
> 1) "Butona basıldı mı?" hangi şekille gösterilir, kaç ok çıkar?  
> 2) "LED'i Yak/Söndür" hangi şekille gösterilir?  
> 3) Sistemin durmaksızın çalışması görsel olarak nasıl gösterilir?  
> **Öğrenci Cevabı:** 1) Eşkenar dörtgen (2 ok çıkar), 2) Dikdörtgen, 3) Ok kendine (başa) geri döner.  
> **Değerlendirme:** Doğru.

---

## 5.3 Sözde Kod (Pseudocode) ve Durum (State / Flag) Mantığı

Mikrodenetleyiciler çok hızlıdır (saniyede milyonlarca döngü). Tek bir butona basıldığında bir basışta LED'i yakan, ikinci basışta söndüren (Push-button Toggle) bir sistemde doğrudan `if (buton == 1)` yazılırsa parmak butondayken LED saniyede binlerce kez yanıp söner.

Bunun önüne geçmek için:
1. **Durum Değişkeni (State/Flag):** `led_durumu` saklanır.
2. **Durum Tersleme:** `led_durumu = !led_durumu;` (veya XOR: `led_durumu ^= 1;`).
3. **Elin Çekilmesini Bekleme (Bloklama / Debounce):** Kullanıcı butonu bırakana kadar döngüde beklenir.

```text
  Parmağın Butona Basma Süresi (~100 ms):
  Buton:  ______/‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾\______
                |<-- MCU 100.000+ kez okur -->|

  1) Flag/Bekleme YOKSA:  LED -> 1,0,1,0,1,0,1,0... (Rastgele kalır!)
  2) Flag + while(press): LED -> ^1 (Yükselen anda SADECE 1 kez değişir, bırakılana kadar bekler)
```

### Soru - Cevap
> **Soru 1:** Başlangıçta `led_durumu = 0` iken butona basıldığında bu durumu tersine çevirmek için nasıl bir şart kurulur?  
> **Öğrenci Cevabı:**  
> ```text
> if (button_press) {
>     if (led_durumu == 0) {
>         led_durumu = 1;
>     } else {
>         led_durumu = 0;
>     }
> }
> ```  
> **Değerlendirme:** Mantık tamamen doğru.  
> 
> **Soru 2:** Kullanıcı parmağını butondan çekene kadar işlemcinin beklemesini sağlayan tek satırlık bloklayan kod nedir?  
> **Öğrenci Cevabı:**  
> ```c
> while (button_press) {
>     // Buton bırakılana kadar burada bekle
> }
> ```  
> **Değerlendirme:** Tamamen doğru.
