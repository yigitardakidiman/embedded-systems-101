# Sınav Hazırlığı ve Eksik Kapatma Tekrar Oturumları

Bu doküman; 1–12. konuların genel sınav öncesi tekrar oturumlarını, öğrencinin sorulara verdiği yanıtları, hata analizlerini ve kapatılan eksikleri kayıt altına alır.

---

## Oturum 1: Bit İşlemleri, Maskeleme, Integer Promotion ve Lojik Kapılar (E1 & E6)
**Tarih:** 2026-11-25  
**Hedeflenen Eksikler:**
- **E1:** Bit sıfırlamada `~` unutulması (`x &= ~(1 << n)` vs `x &= (1 << n)`).
- **E6:** C dilinde `~` operatörü öncesi *integer promotion* (`uint8_t` değerin 32 bit `int` türüne genişlemesi).

### Soru ve Cevap Kayıtları

| # | Soru Özeti | Öğrenci Cevabı | Değerlendirme & Sonuç |
|---|---|---|---|
| **1** | `PORTB = 0xB6` iken diğer bitleri bozmadan Bit 4'ü sıfırla (`0` yap). (İlk soruda Bit 3 istenmişti; öğrenci Bit 3'ün zaten `0` olduğunu doğru tespit etti.) | `PORTB &= ~(1 << 4)`, maske `1110 1111`, yeni değer `0xA6` | **Doğru.** Bit sıfırlama kalıbı ve hex hesabı doğru. |
| **2** | `PORTB = 0xB6` üzerine yanlışlıkla `PORTB &= (1 << 4)` (`~` olmadan) yazılırsa ne olur? | `0x10` (`0001 0000`); yalnızca 4. LED açık kalır, diğer tüm LED'ler söner. | **Doğru.** `~` unutulduğunda diğer bitlerin sıfırlandığı net olarak kavrandı. |
| **3** | `uint8_t x = 0x0F; if (~x == 0xF0)` kodunda `LED_ON()` mu `LED_OFF()` mu çalışır? | **İlk yanıt:** `LED_ON`, `~x = 1111 0000`.<br>**İpucu sonrası (Integer Promotion):** `x` 32 bite genişler (`0x0000000F`), `~x = 0xFFFFFFF0` olur, `0xF0`'a eşit değildir → `LED_OFF`. | **Rehberlikle Doğru.** İlk yanıtta 8 bitlik düşünüldü; integer promotion hatırlatmasıyla 32 bitlik karşılık doğru hesaplandı. |
| **4** | `~x == 0xF0` karşılaştırmasını `LED_ON()` çalışacak şekilde düzelten iki farklı C yazımı nedir? | İlk denemede `(uint8_t)~x == 0xF0` doğru yazıldı; maskeleme yöntemi için `0xFF` maskesi bulundu, açıklama sonrası `if ((~x & 0xFF) == 0xF0)` kalıbı oluşturuldu. | **Rehberlikle Doğru.** Tür dönüşümü (cast) ve `0xFF` bit maskeleme yöntemleri açıklandı. |
| **5** | Pekiştirme: `uint8_t durum = 0xA5;` için `~durum`'un 32 bitlik hex değeri nedir? `uint8_t sensor = 0xF0;` tersini kontrol eden iki farklı `if` koşulunu yaz. | 1) `0xFFFFFF5A`<br>2) `if ((uint8_t)~sensor == ...)` ve `if ((~... & 0xFF) == ...)` | **Doğru.** 32 bitlik `~` sonucu tek seferde hatasız bulundu; her iki düzeltme kalıbı da yazıldı. |
| **6** | Pres makinesi: Kapı kapalı ($A=1$) VE Acil Stop basılı değil ($B=0$) şartını kuran lojik kapı devresi nedir? `MAKINE_REG` yazmacının 6. bitini sıfırlayan C kodu nedir? | 1) `~B` (NOT kapısı) yapıp $A$ ile AND kapısına sokmalıyız.<br>2) `MAKINE_REG &= ~(1 << 6)` | **Doğru.** Hem donanımsal lojik kapı tasarımı hem de `&= ~(1 << n)` bit sıfırlama kalıbı ilk denemede hatasız. |

### Hata Analizi ve Kazanımlar
- **E1 (Bit Sıfırlamada `~` Kullanımı):** `x &= ~(1 << n)` ile `x &= (1 << n)` arasındaki fark hem teorik hem de LED senaryosu üzerinde tam olarak oturdu. Soru 6'da hiç ipucu almadan `MAKINE_REG &= ~(1 << 6)` doğru yazıldı. Ayrıca `notes/sinav_03_genel_degerlendirme.md` Soru 1.2 altına düzeltme notu eklendi. (**E1 Kapatıldı**)
- **E6 (`~` ve Integer Promotion):** C'de `uint8_t` gibi `int`'ten küçük türlerin bit işlemine (`~`) girmeden önce `int` (32 bit) türüne yükseltildiği (`0x0F` $\rightarrow$ `0x0000000F` $\rightarrow$ `~` $\rightarrow$ `0xFFFFFFF0`) kavrandı. Üst 24 biti temizlemek için kullanılan iki yöntem:
  1. **Tür Dönüşümü (Cast):** `if ((uint8_t)~x == 0xF0)`
  2. **Bit Maskeleme (`& 0xFF`):** `if ((~x & 0xFF) == 0xF0)`
  (**E6 Kapatıldı**)

### Hâlâ Zayıf Olan / Aralıklı Tekrarda Sorulacak Noktalar
- `~x` sonrası üst bitleri atarken `(uint8_t)~x` (cast) ile `(~x & 0xFF)` (maskeleme) yazımlarının el alışkanlığı kazanması için **Oturum 3 (Gömülü C Temelleri)** içinde aralıklı tekrar sorusu olarak tekrar sorulacak.
