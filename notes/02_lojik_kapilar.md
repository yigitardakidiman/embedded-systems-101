# 2. Konu: Lojik Kapılar

Bu doküman, mikrodenetleyicilerin işlemci (CPU) ve Aritmetik Mantık Biriminin (ALU) temel yapı taşları olan lojik kapıları, doğruluk tablolarını, elektriksel aktiflik seviyelerini ve soru-cevap kayıtlarını içerir.

---

## 2.1 Lojik Kapı Kavramı ve AND (VE) Kapısı

- Lojik kapılar, transistörlerin bir araya gelmesiyle mantıksal kararlar üreten dijital devre elemanlarıdır.
- **AND Kapısı:** Mantıksal çarpma işlemidir ($Y = A \cdot B$).
  - Çıkışın HIGH ($1$) olabilmesi için **tüm girişlerin HIGH ($1$) olması şarttır**.
- **C Dilindeki Sembolleri:**
  - **Bit Düzeyinde (Bitwise) AND (`&`):** İki sayının karşılıklı bitlerini AND kapısından geçirir (`Y = A & B;`). Register'larda bit sıfırlama (maskeleme) için kullanılır.
  - **Mantıksal (Koşul) AND (`&&`):** `if` / `while` karar yapılarında iki koşulun aynı anda doğru olmasını denetler (`if (kapak_kapali && buton_basili)`).

| Giriş A | Giriş B | Çıkış Y (`A & B`) |
|:---:|:---:|:---:|
| 0 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | **1** |

- **Donanım Örneği:** Seri bağlı iki mekanik anahtar veya CNC tezgâhında "Güvenlik Kapağı Kapalı" (1) VE "Başlat Butonuna Basıldı" (1) koşulu sağlandığında motoru çalıştıran mantık.

```text
  Lojik Sembol:               Seri Anahtar Devresi (Fiziksel Karşılık):
    A ──+---+
        | & |─── Y = A . B      VCC ──[ / A ]──[ / B ]──( Motor )── GND
    B ──+---+                         Kapak     Başlat
                                (İkisi birden kapanmazsa akım geçmez)
```

### Soru - Cevap
> **Soru:** 3 girişli ($A, B, C$) bir AND kapısının doğruluk tablosunda kaç durum olur? Çıkışın $1$ olduğu kaç durum vardır ve girişler ne olmalıdır?  
> **Öğrenci Cevabı:** $8$ farklı durum olur ve HIGH olan sadece $1$ durum olur, o da `1, 1, 1`.  
> **Değerlendirme:** Doğru ($2^3 = 8$).

---

## 2.2 OR (VEYA) Kapısı

- Mantıksal toplama işlemidir ($Y = A + B$).
- Girişlerden **en az biri HIGH ($1$)** olduğunda çıkış HIGH ($1$) olur.
- Çıkışın LOW ($0$) olabilmesi için tüm girişlerin aynı anda LOW ($0$) olması gerekir.
- **C Dilindeki Sembolleri:**
  - **Bit Düzeyinde (Bitwise) OR (`|`):** İki sayının karşılıklı bitlerini OR kapısından geçirir (`Y = A | B;`). Register'larda belirli bir biti $1$ yapmak (Set) için kullanılır.
  - **Mantıksal (Koşul) OR (`||`):** Karar yapılarında koşullardan en az birinin doğru olmasını denetler (`if (yangin_sensoru || manuel_buton)`).

| Giriş A | Giriş B | Çıkış Y (`A \| B`) |
|:---:|:---:|:---:|
| 0 | 0 | **0** |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 1 |

- **Donanım Örneği:** Paralel bağlı anahtarlar; Yangın Sensörü ($1$) VEYA Manuel Yangın Butonu ($1$) tetiklendiğinde alarm sirenini ($1$) devreye sokan güvenlik devresi.

```text
  Lojik Sembol:               Paralel Anahtar Devresi (Fiziksel Karşılık):
    A ──\---\                         +──[ / A (Yangın Sensörü) ]──+
         |>=1|── Y = A + B      VCC ──+                             +──( Siren )── GND
    B ──/---/                         +──[ / B (Manuel Buton)   ]──+
                                      (En az biri kapanırsa siren çalar)
```

### Soru - Cevap
> **Soru:** 3 girişli bir OR kapısı için: 1) Çıkışın $0$ olduğu giriş kombinasyonu nedir? 2) Çıkışın $1$ olduğu toplam kaç durum vardır?  
> **Öğrenci Cevabı:** 1) `000`, 2) 7 durumun hepsi 1.  
> **Değerlendirme:** Doğru.

---

## 2.3 NOT (DEĞİL / Evirici) Kapısı ve Active-LOW Mantığı

- Tek girişli ve tek çıkışlıdır ($Y = \overline{A}$).
- Girişi tersler: $0 \rightarrow 1$, $1 \rightarrow 0$.
- **C Dilindeki Sembolleri:**
  - **Bit Düzeyinde (Bitwise) NOT (`~`):** Bir verinin tüm bitlerini tek tek tersler (`Y = ~A;`). Örneğin `~0b00001111` $\rightarrow$ `0b11110000`.
  - **Mantıksal (Koşul) NOT (`!`):** Bir koşulun doğruluk değerini tersler (`if (!buton_basili)` $\rightarrow$ `0` ise `1`, sıfırdan farklıysa `0` yapar).

| Giriş A | Çıkış Y (`~A` / `!A`) |
|:---:|:---:|
| 0 | **1** |
| 1 | **0** |

- **Gömülü Sistemlerde "Active-LOW" Mantığı:**
  Bir pinin üzerinde çizgi varsa ($\overline{\text{RESET}}$, $\overline{\text{ENABLE}}$, $\overline{\text{CS}}$) veya `_N` eki taşıyorsa bu sinyal **LOW ($0\text{V}$)** seviyesinde aktiftir.

```text
  NOT (Evirici) Sembolü:          Active-HIGH vs. Active-LOW Sürücü Bağlantısı:
    A ───|>o─── Y = !A              MCU (1 / 3.3V) ───────────> [ ENABLE  ] (Motor Çalışır)
         (Baloncuk = Tersle)        MCU (0 / 0.0V) ──o[EN_N]──> [ /ENABLE ] (Motor Çalışır)
```

### Soru - Cevap
> **Soru:** Datasheet'inde $\overline{\text{ENABLE}}$ (Active-LOW) pini bulunan bir motor sürücüyü çalıştırmak için MCU'dan bu pine ne gönderilmelidir? Eğer normal `ENABLE` (Active-HIGH) olsaydı ne gönderilirdi?  
> **Öğrenci Cevabı:** 1) Active-LOW olduğu için LOW gönderildiğinde çalışır. 2) Active-HIGH olsaydı HIGH olduğunda çalışırdı.  
> **Değerlendirme:** Doğru.

---

## 2.4 XOR (Özel VEYA / Exclusive-OR) Kapısı

- Girişler birbirinden **farklıysa $1$**, **aynıysa $0$** üretir ($Y = A \oplus B$).
- **C Dilindeki Sembolleri:**
  - **Bit Düzeyinde (Bitwise) XOR (`^`):** Karşılıklı bitler farklıysa `1`, aynıysa `0` üretir (`Y = A ^ B;`). Gömülü sistemlerde pin durumunu terslemek (Toggle: `PORTB ^= (1 << 5);`) için kullanılır.
  - **Mantıksal (Koşul) Karşılığı (`!=`):** C dilinde `^^` şeklinde ayrı bir mantıksal XOR operatörü yoktur; iki boolean (0/1) koşulun birbirinden farklı olup olmadığı `!=` (eşit değildir) ile kontrol edilir.

| Giriş A | Giriş B | Çıkış Y (`A ^ B`) |
|:---:|:---:|:---:|
| 0 | 0 | 0 |
| 0 | 1 | **1** |
| 1 | 0 | **1** |
| 1 | 1 | 0 |

- **Donanımsal Uygulamaları:**
  1. **Toplayıcı Devreler (Half Adder):** $1 + 1 = 10_2$ işleminde toplam biti doğrudan XOR kapısıyla elde edilir.
  2. **Parite ve Hata Denetimi (CRC):** UART vb. haberleşme hatlarında bit kontrolü için kullanılır.
  3. **Durum Değiştirme (NOT / Buffer Dönüşümü):** Bir girişi sabit tutulduğunda sinyali kontrol eder.

```text
  XOR Sembolü:                Sabit Girişle Davranış Dönüşümü:
    A ──))--\                   A ──))--\
         | =1|── Y = A ^ B           | =1|── !A   (B=1 ise NOT / Evirici olur)
    B ──))--/                   1 ──))--/

  Yarım Toplayıcı (Half Adder):
    A ──+──[ XOR ]──> Toplam (Sum)    (1 + 1 -> Sum = 0, Carry = 1)
    B ──+──[ AND ]──> Elde (Carry)
```

### Soru - Cevap
> **Soru:** 2 girişli bir XOR kapısının $B$ girişine sürekli HIGH ($1$) bağlanırsa:  
> 1) $A=0$ iken çıkış ne olur?  
> 2) $A=1$ iken çıkış ne olur?  
> 3) Bu kapı hangi temel kapı gibi davranmış olur?  
> **Öğrenci Yanıtı:** 1) 1 olur, 2) 0 olur, 3) AND gibi davranmış olur.  
> **Rehberlik / Hata Analizi:** 1 ve 2 doğruydu ($0 \rightarrow 1$ ve $1 \rightarrow 0$). Ancak giriş değerinin tam tersine döndüğü hatırlatıldı. 0'ı 1, 1'i 0 yapan kapı soruldu.  
> **Öğrenci Düzeltmesi:** **NOT kapısı gibi davranır.**  
> **Değerlendirme:** Doğru. (Bir bacağı $1$'e bağlanırsa NOT / Evirici; $0$'a bağlanırsa Buffer / Aynısını geçiren kapı olur).

---

## 2.5 Lojik Kapıların C Dilindeki Sembolleri (Özet Tablo)

Gömülü C'de lojik kapılar iki farklı seviyede kullanılır: **Bit Düzeyinde (Bitwise)** operatörler doğrudan register bitlerini değiştirmek için, **Mantıksal (Logical)** operatörler ise `if` / `while` içinde karar vermek için kullanılır.

| Lojik Kapı | Matematiksel Gösterim | C Bit Düzeyi (Bitwise) | C Mantıksal (Koşul) | Gömülü C Kullanım Örneği |
|---|:---:|:---:|:---:|---|
| **AND (VE)** | $Y = A \cdot B$ | **`&`** | **`&&`** | `PORTA &= ~(1 << 3);` *(Bit sıfırlama)* / `if (a && b)` |
| **OR (VEYA)** | $Y = A + B$ | **`\|`** | **`\|\|`** | `PORTA \|= (1 << 3);` *(Bit 1 yapma)* / `if (a \|\| b)` |
| **NOT (DEĞİL)** | $Y = \overline{A}$ | **`~`** | **`!`** | `maske = ~0x01;` *(Bitleri tersle)* / `if (!hazir)` |
| **XOR (Özel VEYA)** | $Y = A \oplus B$ | **`^`** | **`!=`** *(0/1 için)* | `PORTA ^= (1 << 3);` *(Pin durumu değiştirme / Toggle)* |

```mermaid
flowchart TD
    Q["C'de Lojik İşlem Amacın Ne?"] --> B["Donanım Register / Tekil Bit Değiştirme"]
    Q --> L["if / while İçinde Karar Verme (Doğru-Yanlış)"]
    B --> B1["Bitwise Operatörler: & | ~ ^"]
    L --> L1["Mantıksal Operatörler: && || ! !="]
```
