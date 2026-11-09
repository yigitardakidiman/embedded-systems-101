# 11. Konu: Timer (Zamanlayıcı) ve Zamanlama Mimarisi

Bu doküman; Bloklayan (`delay`) ile Bloklamayan (`millis` / `HAL_GetTick`) kod mimarilerinin farkını, Donanımsal Timer birimlerinin çalışma yapısını (`PSC`, `CNT`, `ARR`), Timer kesmelerini ve `CCR` yazmacı ile donanımsal PWM üretimini içerir.

---

## 11.1 Bloklayan (`delay`) vs. Bloklamayan (`millis` / `HAL_GetTick`) Kod Mantığı

- **Bloklayan Kod (`delay_ms` / `HAL_Delay`):** İşlemciyi belirtilen süre boyunca olduğu satırda hapseder. Bu süre zarfında `while(1)` içindeki diğer hiçbir görev (sensör okuma, buton kontrolü, haberleşme) çalışamaz.
- **Bloklamayan Kod (`millis()` / `HAL_GetTick()`):** Mikrodenetleyiciye enerji verildiğinden itibaren her $1\text{ ms}$'de bir artan sistem zamanlayıcısını referans alır:
  ```c
  if (su_anki_zaman - son_zaman >= 1000) {
      son_zaman = su_anki_zaman; // Yeni periyot için başlangıç zamanını sıfırla/güncelle
      gorevi_calistir();
  }
  ```
  İşlemci hiç bekleme yapmaz; süre dolmadıysa `if` bloğunu transit geçerek diğer görevleri yürütmeye devam eder.

### Soru - Cevap
> **Soru:**  
> 1) Hasta başı monitöründe ekranı güncellemek için `delay_ms(2000)` kullanılırsa kalp atış sensörü ve tuşlara ne olur?  
> 2) `son_zaman = su_anki_zaman;` satırı unutulursa ne olur?  
> **Öğrenci Cevabı:**  
> 1) `delay` koyarsak $2000\text{ ms}$ boyunca başka hiçbir işlemi yapamayız, program sadece orada bekler.  
> 2) Eşitliği koymazsak süre sıfırlama işlemi yapılmaz; belirtilen süreden sonraki her zaman değişiminde `if` içerisine tekrar girer.  
> **Değerlendirme:** Kusursuz.

---

## 11.2 Donanımsal Timer Mimarisi: `PSC`, `CNT` ve `ARR`

Donanımsal Timer, CPU'dan bağımsız olarak arka planda kendi kendine sayan donanımsal kronometredir:
1. **PSC (Prescaler - Ön Bölücü):** Yüksek hızlı ana saat sinyalini (örn: $72\text{ MHz}$) bölerek sayacın sayma hızını yavaşlatır:
   $$\text{Sayaç Frekansı} = \frac{f_{\text{clock}}}{\text{PSC}}$$
2. **CNT (Counter - Sayaç):** Bölünmüş saat darbeleriyle $0, 1, 2...$ şeklinde yukarı sayan yazmaçtır.
3. **ARR (Auto-Reload Register - Tavan / Periyot):** Sayacın kaça kadar sayıp sıfırlanacağını (Overflow / Taşma) ve ne sıklıkla Timer Kesmesi (Interrupt) üreteceğini belirler.
   *(Not: Donanım $0$'dan saymaya başladığı için yazmaçlara değerler `PSC - 1` ve `ARR - 1` olarak yazılır).*

### Soru - Cevap
> **Soru:** $72\text{ MHz}$ ($72\,000\,000\text{ Hz}$) ana saati `PSC` ile $72\,000$'e bölersek `CNT` kaç $\text{Hz}$ (kaç $\text{ms}$) ile sayar ve her $500\text{ ms}$'de bir kesme üretmek için `ARR` kaç yapılmalıdır?  
> **Öğrenci Cevabı:**  
> 1) $1000\text{ Hz}$ olur, bu da $1\text{ ms}$'ye denk gelir.  
> 2) `ARR = 500` yapmalıyız.  
> **Değerlendirme:** Tamamen doğru.

---

## 11.3 Timer ile Donanımsal PWM Üretimi: `CCR` Yazmacı

Timer donanımı, **`ARR` (Periyot Tavanı)** ve **`CCR` (Capture/Compare Register - Karşılaştırma Yazmacı)** kullanarak CPU'yu hiç yormadan PWM üretir:
- **`CNT < CCR` iken:** Pin çıkışı **HIGH ($3.3\text{V}$)** tutulur.
- **`CNT >= CCR` iken:** Pin çıkışı **LOW ($0\text{V}$)** çekilir.
- **Görev Oranı (Duty Cycle) Formülü:**
  $$\text{Duty Cycle (\%)} = \frac{\text{CCR}}{\text{ARR}} \times 100 \quad \Longleftrightarrow \quad \text{CCR} = \text{ARR} \times \frac{\text{Yüzde}}{100}$$

### Soru - Cevap ve Hata Analizi
> **Soru:** `ARR = 1000` iken $\%50$ ve $\%75$ Duty Cycle için hangi yazmaç kaça ayarlanmalıdır?  
> **Öğrenci İlk Yanıtı:** 1) `CCR = 50`, 2) `CCR = 25`.  
> **Hata Analizi ve Rehberlik:**  
> - Değiştirilmesi gereken yazmacın `ARR` değil **`CCR`** olduğu tespiti doğrudur.  
> - Ancak tavanımız `100` değil **`1000`**'dir ve `CCR` doğrudan HIGH kalma süresini belirler.  
> **Öğrenci Düzeltmesi:** 1) `500` yazmamız lazım. 2) `750` olmalıdır. *(Tamamen doğru)*

---

## 11.4 Konu Sonu Değerlendirme Testi (Bebek Küvözü Kontrol Kartı Senaryosu)

1. **Soru ($5000\text{ ms}$ Ekran Güncelleme ve Sürekli Solunum/Kalp Takibi):**
   - **Öğrenci Yanıtı:** 1) `delay` kullanırsak `delay` sırasında başka hiçbir işlem devam etmez. 2) `son_zaman = su_anki_zaman;` işlemin sıfırlanması (yeni periyot başlangıcı) için gereklidir. *(Tam puan)*
2. **Soru ($48\text{ MHz}$ Saat, $48\,000$ Bölücü ve $250\text{ ms}$ Kesme Hesabı):**
   - **Öğrenci Yanıtı:** 1) `CNT` $1000\text{ Hz}$ olur, o da $1\text{ ms}$ olur. 2) `ARR = 250` ayarlanmalıdır. *(Tam puan)*
3. **Soru (`ARR = 2000` İken $\%25$ ve $\%80$ Isıtıcı PWM Hesabı):**
   - **Yanıt ve Düzeltme:** $\%80$ için `CCR = 1600` ($2000 \times 0.80$) doğrudan doğru hesaplandı; $\%25$ için `1000` üzerinden düşünülen `250` değeri, `ARR = 2000` üzerinden **`CCR = 500`** olarak düzeltildi. *(Tamamlandı)*
