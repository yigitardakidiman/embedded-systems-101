# 1. Konu: Bit, Byte ve Sayı Sistemleri

Bu doküman, ikili mantık, sayı tabanları ve donanım yazmaçlarında (register) kullanılan bit düzeyindeki işlemleri, işlenen ders notlarını ve interaktif soru-cevap kayıtlarını içerir.

---

## 1.1 Bit ve Byte Kavramı (Fiziksel ve Mantıksal Karşılığı)

- **Bit (Binary Digit):** Dijital sistemlerde bilginin en küçük birimidir. Yalnızca **`0`** veya **`1`** mantıksal değerlerini alabilir.
  - **Fiziksel Karşılığı:** Mikrodenetleyici pinlerindeki gerilim seviyesidir ($0\text{V} = \text{LOW / 0}$, $3.3\text{V} / 5\text{V} = \text{HIGH / 1}$).
- **Byte:** 8 adet bitin yan yana gelmesiyle oluşan temel veri birimidir ($1 \text{ Byte} = 8 \text{ bit}$).
- **Kapasite Hesabı ($2^n$ Kuralı):** $n$ adet bit ile birbirinden farklı **$2^n$** durum temsil edilir.
  - 1 bit $\rightarrow 2^1 = 2$ durum (`0`, `1`)
  - 8 bit $\rightarrow 2^8 = 256$ durum ($0 - 255$)

```text
Fiziksel Voltaj  <--->  Mantıksal Bit       8 Bit = 1 Byte (Register Kutusu)
 3.3V / 5.0V     <--->   1 (HIGH)       +---+---+---+---+---+---+---+---+
 0.0V (GND)      <--->   0 (LOW)        | 1 | 0 | 1 | 1 | 0 | 0 | 1 | 0 |
                                        +---+---+---+---+---+---+---+---+
                                         b7  b6  b5  b4  b3  b2  b1  b0
```

### Soru - Cevap
> **Soru:** Bir mikrodenetleyicide sadece 3 bitlik bir register alanımız olsaydı, bu 3 bit ile en fazla kaç farklı durumu temsil edebilirdik? En küçük ve en büyük sayısal değerleri ne olurdu?  
> **Öğrenci Cevabı:** $2^3 = 8$ durum, en küçük $0$, en büyük $7$.  
> **Değerlendirme:** Doğru ($0, 1, 2, 3, 4, 5, 6, 7$).

---

## 1.2 İkili (Binary) Sayı Sistemi ve Basamak Ağırlıkları

- Binary taban $2$'dir. Her basamak $2$'nin bir kuvvetini temsil eder.
- 8 bitlik bir sayıda basamak değerleri sağdan sola:
  $$\text{Bit 7}(128) - \text{Bit 6}(64) - \text{Bit 5}(32) - \text{Bit 4}(16) - \text{Bit 3}(8) - \text{Bit 2}(4) - \text{Bit 1}(2) - \text{Bit 0}(1)$$
- **LSB (Least Significant Bit):** En düşük değerlikli sağdaki bit ($2^0 = 1$).
- **MSB (Most Significant Bit):** En yüksek değerlikli soldaki bit ($2^7 = 128$).
- **Dönüşüm Mantığı:** Sadece `1` olan basamakların ağırlıkları toplanır.

```text
  MSB (En Yüksek)                                   LSB (En Düşük)
  +-----+-----+-----+-----+-----+-----+-----+-----+
  | 2^7 | 2^6 | 2^5 | 2^4 | 2^3 | 2^2 | 2^1 | 2^0 |  <-- Basamak Kuvveti
  | 128 |  64 |  32 |  16 |  8  |  4  |  2  |  1  |  <-- Basamak Ağırlığı
  +-----+-----+-----+-----+-----+-----+-----+-----+
  |  0  |  0  |  0  |  1  |  0  |  1  |  0  |  1  |  <-- Örnek: 0001 0101
  +-----+-----+-----+-----+-----+-----+-----+-----+
                       |           |           |
                       v           v           v
                      16     +     4     +     1   =  21 (Decimal)
```

### Soru - Cevap
> **Soru:** `0001 0101` binary sayısının onluk (decimal) tabandaki karşılığı nedir?  
> **Öğrenci İlk Yanıtı:** 19  
> **Rehberlik / Hata Analizi:** Basamaklar tek tek açıldı: Bit 0 = 1, Bit 2 = 4, Bit 4 = 16. Bit 1 ($2$) ile Bit 2 ($4$) karıştırılmıştı.  
> **Öğrenci Düzeltmesi:** **21** ($16 + 4 + 1$).  
> **Değerlendirme:** Doğru.

---

## 1.3 Onluk (Decimal) Sayıyı İkili (Binary) Sayıya Çevirme

İki yöntem kullanılır:
1. **Ağırlık Çıkarma Yöntemi (Pratik Zihinsel Yöntem):** Sayıdan sığabilen en büyük $2^n$ ağırlığı çıkarılır ve o basamağa `1` yazılır.
2. **2'ye Sürekli Bölme Yöntemi (Algoritmik Yöntem):** Sayı 2'ye bölünür, kalanlar tersten (son kalandan ilk kalana) dizilir.

```text
Yöntem 1: Ağırlık Çıkarma (26 için)       Yöntem 2: 2'ye Sürekli Bölme (26 için)
  26 - 16 = 10  --> Bit 4 = 1               26 / 2 = 13  Kalan: 0  (LSB - En sağ)
  10 -  8 =  2  --> Bit 3 = 1               13 / 2 =  6  Kalan: 1   |
   2 -  2 =  0  --> Bit 1 = 1                6 / 2 =  3  Kalan: 0   |  Alttan
  Diğer bitler  --> 0                        3 / 2 =  1  Kalan: 1   |  Üste Oku
  Sonuç: 0001 1010                           1 / 2 =  0  Kalan: 1  (MSB - En sol)
                                            Sonuç (8-bit): 0001 1010
```

### Soru - Cevap
> **Soru:** Onluk tabandaki $26$ sayısını 8 bitlik binary forma çeviriniz.  
> **Öğrenci Cevabı:** `0001 1010` ($16 + 8 + 2 = 26$).  
> **Değerlendirme:** Doğru.

---

## 1.4 Onaltılık (Hexadecimal) Sayı Sistemi ve Nibble Kavramı

- Tabanı **16**'dır: `0 - 9` ve `A(10), B(11), C(12), D(13), E(14), F(15)`. C kodlarında `0x` ön eki ile gösterilir.
- **Nibble:** 4 bitlik gruptur. $2^4 = 16$ değer alabildiği için:
  $$\mathbf{1 \text{ Hex Basamağı}} = \mathbf{1 \text{ Nibble (4 bit)}}$$
  $$\mathbf{1 \text{ Byte (8 bit)}} = \mathbf{2 \text{ Hex Basamağı}} \quad (0\text{x}00 - 0\text{x}FF)$$

```text
  |<------- Üst Nibble (4 bit) ------>|<------- Alt Nibble (4 bit) ------>|
  +--------+--------+--------+--------+--------+--------+--------+--------+
  |  1 (8) |  1 (4) |  0 (2) |  0 (1) |  0 (8) |  1 (4) |  1 (2) |  1 (1) |
  +--------+--------+--------+--------+--------+--------+--------+--------+
  |            8 + 4 = 12             |            4 + 2 + 1 = 7          |
  |             Hex: 'C'              |             Hex: '7'              |
  +-----------------------------------+-----------------------------------+
                                   0xC7
```

### Soru - Cevap
> **Soru:** `1100 0111` binary sayısının Hexadecimal (`0x...`) karşılığı nedir?  
> **Öğrenci Cevabı:** `0xC7` (`1100 = 12 = C`, `0111 = 7 = 7`).  
> **Değerlendirme:** Doğru.

---

## 1.5 Bit Düzeyinde Mantıksal İşlemler (AND ve OR)

- **AND Operatörü (`&`):** Yalnızca iki bit de `1` ise sonuç `1`.
  - Kullanım Amacı: **Maskeleme (Bit Sıfırlama / Clear)**.
- **OR Operatörü (`|`):** En az bir bit `1` ise sonuç `1`.
  - Kullanım Amacı: **Biti 1 Yapma (Set Etme)**.

```text
  AND (&) ile Maskeleme (Bit Sıfırlama)     OR (|) ile Bit 1 Yapma (Set)
    A:     1 0 1 1  0 0 1 0                   A:     1 0 1 1  0 0 1 0
    B:  &  1 1 0 0  0 0 1 1 (Maske)           B:  |  1 1 0 0  0 0 1 1 (Maske)
    -----------------------                   -----------------------
    Sonuç: 1 0 0 0  0 0 1 0 (130)             Sonuç: 1 1 1 1  0 0 1 1 (243)
           ^-- Sadece ikisi 1 ise 1                  ^-- En az biri 1 ise 1
```

### Soru - Cevap
> **Soru:** $A = \text{`1011 0010`}$, $B = \text{`1100 0011`}$ için:  
> 1) $A \ \& \ B = ?$  
> 2) $A \ | \ B = ?$  
> **Öğrenci Yanıtı:** 1. Cevap: 130 (`1000 0010`), 2. Cevap: 113.  
> **Rehberlik / Hata Analizi:** 1. cevap doğru ($128 + 2 = 130$). 2. soruda ise öğrenci `1 | 1` durumlarını yanlışlıkla `0` alarak farkında olmadan XOR sonucunu bulmuştu. OR kuralı ($1 \ | \ 1 = 1$) tekrar hatırlatıldı.  
> **Öğrenci Düzeltmesi:** **243** (`1111 0011` $\rightarrow 128 + 64 + 32 + 16 + 2 + 1$).  
> **Değerlendirme:** Doğru.

---

## 1.6 Bit Düzeyinde Mantıksal İşlemler (XOR ve NOT)

- **XOR Operatörü (`^`):** Bitler birbirinden farklıysa `1`, aynıysa `0` üretir.
  - Kullanım Amacı: **Durum Değiştirme (Toggle)**. Bir pini her çevrimde terslemek için `PORT ^= (1 << PIN);` kullanılır.
- **NOT Operatörü (`~`):** Tekli operatördür, tüm bitleri tersler ($0 \rightarrow 1, 1 \rightarrow 0$).
  - Kullanım Amacı: **Maske Tersleme**. Bir pini sıfırlarken `&= ~maske` kalıbında kullanılır.

```text
  XOR (^) ile Durum Değiştirme (Toggle)     NOT (~) ile Bit Tersleme
    A:     1 0 1 0  1 1 0 0                   A:     1 0 1 0  1 1 0 0
    B:  ^  0 0 0 0  1 1 1 1 (Alt 4 bit)      ~A:     0 1 0 1  0 0 1 1
    -----------------------                   -----------------------
    Sonuç: 1 0 1 0  0 0 1 1                   (Her 1 -> 0, her 0 -> 1 olur)
           (Aynı=0, Farklı=1)
```

### Soru - Cevap
> **Soru:** $A = \text{`1010 1100`}$, $B = \text{`0000 1111`}$ için:  
> 1) $A \oplus B = ?$  
> 2) $\sim A = ?$  
> **Öğrenci Cevabı:** 1) `1010 0011`, 2) `0101 0011`.  
> **Değerlendirme:** Doğru.

---

## 1.7 Bit Kaydırma Operatörleri (`<<` ve `>>`)

- **Sola Kaydırma (`<<`):** Bitleri sola kaydırır, sağdan `0` girer. Matematiksel olarak $2$ ile çarpmaktır ($x \ll n = x \times 2^n$).
  - Gömülüdeki Amacı: **Bit Maskesi Oluşturma** (`1 << n`).
- **Sağa Kaydırma (`>>`):** Bitleri sağa kaydırır. Matematiksel olarak $2$'ye bölmektir ($x \gg n = x \div 2^n$).
  - Gömülüdeki Amacı: **Veri Ayrıştırma** (Örn: Bir baytın içindeki durum bitini en sağa çekip okumak).

```text
  Sola Kaydırma (1 << 5) — Maske Oluşturma:
    Başlangıç (1) : [0][0][0][0] [0][0][0][1]
                      <--- 5 adım sola kaydır (Sağdan 0 girer)
    Sonuç (0x20)  : [0][0][1][0] [0][0][0][0]   (Bit 5 = 1, Değer = 32)

  Sağa Kaydırma (0100 1000 >> 3) — Bölme / Ayrıştırma:
    Başlangıç (72): [0][1][0][0] [1][0][0][0]
                      3 adım sağa kaydır ---> (Soldan 0 girer, sağdaki 3 bit düşer)
    Sonuç (9)     : [0][0][0][0] [1][0][0][1]   (72 / 2^3 = 9)
```

### Soru - Cevap
> **Soru:**  
> 1) `(1 << 5)` ifadesinin 8 bitlik binary karşılığı nedir?  
> 2) $X = \text{`0b0100 1000`}$ ($72$) ise $X \gg 3$ binary ve onluk değeri nedir?  
> **Öğrenci Cevabı:** 1) `0010 0000`, 2) `0000 1001` (Onluk karşılığı: 9).  
> **Değerlendirme:** Doğru.

---

## 🎯 1. Bölüm Genel Değerlendirme (Donanım Senaryosu)

**Senaryo:** Mikrodenetleyicinin `PORTB` yazmacının mevcut değeri `0x5C` olarak okunmuştur.

1. **Soru:** `0x5C` binary karşılığı ve hangi pinlerin HIGH/LOW olduğu?
   - **Öğrenci Cevabı:** `0101 1100` $\rightarrow$ Pin 2, 3, 4, 6 HIGH; Pin 0, 1, 5, 7 LOW. *(Doğru)*
2. **Soru:** Diğer pinleri bozmadan **Pin 1'i HIGH** yapmak için hangi operatör, maske ve yeni Hex değer?
   - **Öğrenci Cevabı:** OR işlemi, `(1 << 1)` maskesi, yeni değer `0x5E`. *(Doğru - `PORTB |= (1 << 1);`)*
3. **Soru:** `0x5E` durumundayken diğer pinleri bozmadan **Pin 3'ü LOW** yapmak için terslenecek maske ve yeni Hex değer?
   - **Öğrenci Cevabı:** `~(1 << 3)` maskesi, yeni değer `0x56`. *(Doğru - `PORTB &= ~(1 << 3);`)*
