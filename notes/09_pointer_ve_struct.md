# 9. Konu: Pointer (İşaretçi) ve Struct Yapıları

Bu doküman; RAM bellek adresleme mantığını, Pointer (`&` ve `*`) operatörlerini, Call by Value vs. Call by Reference farkını, `struct` veri paketleme yapılarını ve donanım yazmaçlarına (`GPIOA->ODR`, `USART1->DR`) erişimde kullanılan Struct Pointer (`->`) mekanizmasını içerir.

---

## 9.1 Bellek Adresleri ve Pointer (İşaretçi) Mantığı

- **RAM Yapısı:** Mikrodenetleyicinin RAM belleği, her birinin benzersiz bir **fiziksel adresi (kapı numarası)** olan $1\text{ baytlık}$ kutulardan oluşur.
- **Pointer (İşaretçi):** Kendi içinde sıradan bir sayı tutmak yerine, başka bir değişkenin **RAM'deki fiziksel adresini** tutan özel değişkendir.
- **Temel Operatörler:**
  1. **`&` (Address-of / Adres Operatörü):** Değişkenin RAM'deki kapı numarasını (adresini) verir (`&sicaklik`).
  2. **`*` (Dereference / İçeriğe Erişim Operatörü):**
     - Tanımlarken (`uint8_t *ptr;`): Değişkenin bir pointer olduğunu belirtir.
     - İşlem yaparken (`*ptr = 40;`): *"Tuttuğun adrese git ve o adresteki kutunun içindeki değere müdahale et"* anlamına gelir.
- **Kritik Ayrım:**
  - `ptr` (yıldızsız) $\rightarrow$ **Adresin kendisini** (`0x20000010` gibi kapı numarasını) verir.
  - `*ptr` (yıldızlı) $\rightarrow$ O adresteki kutunun **içindeki değeri** (`150` gibi veriyi) verir.

### Soru - Cevap ve Hata Analizi
> **Soru:**  
> ```c
> uint8_t hiz = 100; // Adresi: 0x20000010
> uint8_t *p_hiz = &hiz;
> *p_hiz = 150;
> ```
> Bu kod çalıştıktan sonra `p_hiz` (yıldızsız) neyi tutar ve `hiz` değişkeninin değeri kaç olur?  
> **Öğrenci İlk Yanıtı:** `p_hiz` `150` tutuyor ve `hiz` değişkeni de `150` oldu çünkü `*p_hiz = 150` kısmında `hiz` değişkeninin adresine gidip değeri `150` ile değiştirdi.  
> **Hata Analizi ve Rehberlik:**  
> - `hiz` değişkeninin `150` olduğu ve açıklaması **tamamen doğrudur**.  
> - Ancak `p_hiz` (yıldızsız) cebimizdeki adres kağıdıdır; evin içindeki eşya (`150`) değişse de kağıtta yazan kapı numarası (**`0x20000010`**) değişmez.  
> **Kontrol Sorusu:** `ptr` (yıldızsız) ve `*ptr` (yıldızlı) neyi tutar?  
> **Öğrenci Cevabı:** `ptr` adresi, `*ptr` adres içindeki değeri tutuyor. *(Tamamen doğru)*

---

## 9.2 Gömülü Sistemlerde Pointer Neden Kullanılır?

1. **Fonksiyonların Orijinal Değişkeni Güncellemesi ve RAM Tasarrufu:**
   - **Call by Value (Değerle Çağırma):** Fonksiyona normal değişken gönderilirse (`fonk(x)`), derleyici değişkenin **kopyasını** çıkarır; orijinal değişken değişmez.
   - **Call by Reference (Adresle Çağırma):** Fonksiyona değişkenin adresi gönderilirse (`fonk(&x)`), fonksiyon doğrudan adrese gidip **orijinal değişkeni** günceller ve büyük verileri kopyalamadığı için RAM tasarrufu sağlar.
2. **Doğrudan Donanım Yazmaçlarına (Registers) Erişim:**
   - Mikrodenetleyicinin çevre birimleri (GPIO, UART, Timer) sabit bellek adreslerindedir (Örn: `0x40020014`). C dilinde bu fiziksel adreslere doğrudan veri yazmanın tek yolu Pointer kullanmaktır.

### Soru - Cevap
> **Soru:**  
> 1) `void sensor_oku(uint8_t *sicaklik_ptr, uint8_t *nem_ptr)` fonksiyonuna değişkenleri göndermek için hangi operatör kullanılır?  
> 2) Pointer kullanılmasaydı `main` içindeki `ortam_sicakligi = 0` değişkeni kaç kalırdı?  
> **Öğrenci Cevabı:**  
> 1) `&` işaretini kullanmalıyız ki değerlerin adresine erişebilelim.  
> 2) Değerler `0` kalırdı çünkü değerin kendisini değil kopyasını değiştirmiş olur (Call by Value).  
> **Değerlendirme:** Kusursuz.

---

## 9.3 Struct (Yapı) Nedir ve Neden Kullanılır?

- **Amacı:** Diziler (Array) yalnızca aynı veri tipindeki elemanları tutabilirken, **`struct`** birbiriyle ilişkili **farklı veri tiplerini (`uint8_t`, `uint16_t`, `float`) tek bir paket altında** toplamayı sağlar.
- **OOP (`class`) İlişkisi:** C dilindeki `struct`, C++ ve diğer nesne yönelimli dillerdeki `class` yapısının atasıdır. Saf C'deki `struct` içinde fonksiyon (metot) bulunmaz, yalnızca ilişkili verileri paketler.
- **Nokta (`.`) Operatörü:** Bir `struct` değişkeninin doğrudan kendisi elimizdeyse alt elemanlarına `.` ile erişilir:
  ```c
  typedef struct {
      uint16_t voltaj_mV;
      uint8_t  sicaklik_C;
      uint8_t  sarj_yuzdesi;
  } Batarya_t;

  Batarya_t ana_batarya;
  ana_batarya.sarj_yuzdesi = 85;
  ```

### Soru - Cevap
> **Soru:**  
> 1) `Batarya_t` kalıbından `ana_batarya` oluşturup `sarj_yuzdesi` değerini `85` yapan kod nedir?  
> 2) Neden normal dizi yerine `struct` kullanıldı?  
> **Öğrenci Cevabı:**  
> 1) `Batarya_t ana_batarya; ana_batarya.sarj_yuzdesi = 85;`  
> 2) Çünkü dizilerde sadece aynı/benzer türdeki verileri tutabiliyoruz.  
> **Değerlendirme:** Tamamen doğru.

---

## 9.4 Struct Pointer ve Ok (`->`) Operatörü

- **Neden Struct Pointer?** Büyük bir `struct` paketini fonksiyona kopyalayarak göndermek RAM ve işlemci zamanı israfıdır. Bu nedenle fonksiyonlara daima `struct`'ın **adresi (Pointer)** gönderilir.
- **`.` vs. `->` Kuralı:**
  - Değişkenin **kendisi** varsa $\rightarrow$ **`.` (Nokta)** kullanılır (`ana_batarya.sarj_yuzdesi = 85;`).
  - Değişkenin **Pointer'ı (Adresi)** varsa $\rightarrow$ **`->` (Ok)** kullanılır (`p_bat->sarj_yuzdesi = 85;`).
- **Donanım Karşılığı (`GPIOA->ODR` Sırrı):**
  - STM32 gibi mikrodenetleyicilerde her donanım biriminin yazmaçları (`MODER`, `IDR`, `ODR`) bir `struct` olarak modellenmiştir.
  - `GPIOA` veya `USART1` ifadeleri, o donanımın fiziksel başlangıç adresini tutan birer **Struct Pointer**'dır. Bu yüzden donanım yazmaçlarına erişirken daima `->` operatörü (`GPIOA->ODR |= (1 << 5);`) kullanılır.

### Soru - Cevap
> **Soru:**  
> 1) `void motor_durdur(Motor_t *m_ptr)` içinde `hiz_rpm` değerini `0` yapmak için ne yazılır?  
> 2) `USART1->DR = 'A';` satırında `USART1`'in Struct Pointer olduğu nereden anlaşılır?  
> **Öğrenci Cevabı:**  
> 1) `m_ptr->hiz_rpm = 0;`  
> 2) `USART1` struct pointer'dır çünkü `->` işareti kullanılmış; `.` kullanılsaydı değerin (değişkenin) kendisi olacaktı.  
> **Değerlendirme:** Tamamen doğru.
