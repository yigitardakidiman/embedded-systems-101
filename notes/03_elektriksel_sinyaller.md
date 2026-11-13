# 3. Konu: Elektriksel Sinyaller

Bu doküman; dijital ve analog sinyaller, lojik eşikler, boşta kalan (floating) pinler, kenar algılama, zamanlama, PWM ve step motor kontrol sinyallerini ve ilgili soru-cevap kayıtlarını içerir.

---

## 3.1 HIGH / LOW Mantığı, Gerilim Seviyeleri ve Belirsiz Bölge

- Bir mikrodenetleyici pinindeki $1$ ve $0$, fiziksel gerilim aralıklarıdır.
- **3.3V Sistemlerde Eşikler:**
  - $0\text{V} - 0.8\text{V} \rightarrow$ Güvenli **LOW (0)**
  - $2.0\text{V} - 3.3\text{V} \rightarrow$ Güvenli **HIGH (1)**
  - $0.8\text{V} - 2.0\text{V} \rightarrow$ **Belirsiz (Undefined) Bölge:** Giriş transistörleri yarı iletken kalır, mikrodenetleyici aşırı akım çekebilir ve çevresel parazitleri rastgele $1/0$ olarak okur.
- **Floating (Boşta Kalan) Pin:** Bir pine ne gerilim ne toprak bağlı değilse, pin anten gibi çalışıp parazit üretir. Çözüm: **Pull-up** (VCC'ye) veya **Pull-down** (GND'ye) direnç bağlamaktır.

```text
  3.3V Gerilim Eşik Bölgeleri        Buton Bağlantı Türleri
  3.3V +--------------------+
       |  Güvenli HIGH (1)  |        1) Floating (HATALI):   2) Pull-Up (Önerilen):
  2.0V +--------------------+           MCU Pin ──[Buton]      3.3V ──[10kΩ]──+── MCU Pin
       |  BELİRSİZ BÖLGE    |           (Havada = Anten!)                     |
  0.8V +--------------------+                                              [Buton] ── GND
       |  Güvenli LOW (0)   |        3) Pull-Down:
  0.0V +--------------------+           3.3V ──[Buton]──+── MCU Pin
                                                        |
                                                     [10kΩ] ── GND
```

### Soru - Cevap
> **Soru:** 3.3V mikrodenetleyici giriş pinine bağlı butona direnç takılmadı ve pin boşta kaldı. Butona basılmadığında pinden kesinlikle $0$ mı okunur, ne yaşanır?  
> **Öğrenci Cevabı:** Kesinlikle sıfır olmaz. $0-1$ arasında salınım yapacak şekilde dalgalanır, bu da kartın yanlış girdiler almasına sebep olur.  
> **Değerlendirme:** Doğru.

---

## 3.2 Sinyal Geçişleri (Yükselen Kenar ve Düşen Kenar)

- **Seviye (Level):** Sinyalin sabit kaldığı durumdur (HIGH veya LOW).
- **Yükselen Kenar (Rising Edge):** Sinyalin $0 \rightarrow 1$ (LOW'dan HIGH'a) geçtiği anlık andır.
- **Düşen Kenar (Falling Edge):** Sinyalin $1 \rightarrow 0$ (HIGH'dan LOW'a) geçtiği anlık andır.
- **Kesme (Interrupt) İlişkisi:** Düğmeye basılı tutulduğu sürece sürekli tetiklenmemek, yalnızca basılma veya bırakılma anını yakalamak için kenar tetikleme (edge triggering) kullanılır.

```text
  Voltaj
  3.3V (HIGH) ──────┐             ┌───────────────┐
                    │ Falling     │ Rising        │
                    ▼ Edge (1->0) ▲ Edge (0->1)   ▼
  0.0V (LOW)        └─────────────┘               └───────
              Boşta │  Basılı Tutuluyor │ Bırakıldı
             (Pull-up)   (Basılma Anı)    (Bırakılma Anı)
```

### Soru - Cevap
> **Soru:** Pull-up dirençli bir butonda basılmıyorken gerilim HIGH ($1$), basıldığında LOW ($0$)'dur. Butona parmakla ilk basılma anını yakalamak için hangi kenar (Rising mi, Falling mi) seçilmelidir?  
> **Öğrenci Cevabı:** Falling edge algılar, çünkü basılmadığında HIGH sinyal yukarıda devam eder, basıldığında aşağıya doğru bir düşüş yaşanır.  
> **Değerlendirme:** Doğru.

---

## 3.3 Periyodik Sinyaller (Kare Dalga, Sinüs, Periyot ve Frekans)

- **Periyot ($T$):** Sinyalin bir tam çevrimini tamamlama süresidir (saniye veya ms).
- **Frekans ($f$):** 1 saniyedeki çevrim sayısıdır (Hertz - Hz).
  $$f = \frac{1}{T} \quad \Longleftrightarrow \quad T = \frac{1}{f}$$
- **Kare Dalga:** Yalnızca HIGH ve LOW seviyeleri olan sinyaldir. Mikrodenetleyicinin saat frekansı (System Clock, örn: 16 MHz, 72 MHz) kare dalgadır.

```text
  Kare Dalga (Dijital Saat / MCU)          Sinüzoidal Dalga (Analog / AC Şebeke)
  3.3V |   +-------+       +-------+       +V |     .--.
       |   |       |       |       |          |   .'    '.
  0.0V +---+       +-------+       +---     0 +--+--------+--------+--> Zaman
           |<-- T (Periyot) -->|           -V |            '.    .'
           |<-T_H->|<-T_L->|                  |              '--'
```

### Soru - Cevap
> **Soru:** $1\text{ kHz}$ ($1000\text{ Hz}$) frekansındaki bir kare dalganın 1 periyodu kaç ms sürer? Sinyal süresinin yarısında HIGH, yarısında LOW kalacaksa kaçar ms bekletilmelidir?  
> **Öğrenci Cevabı:** 1) 0.001 (saniye), 2) 0.0005 (saniye).  
> **Rehberlik / Birim Çevrimi:** Saniye hesabı doğruydu; milisaniyeye çevrildi: $1\text{ ms}$ ve $0.5\text{ ms}$ ($500\text{ }\mu\text{s}$).  
> **Değerlendirme:** Doğru.

---

## 3.4 PWM (Pulse Width Modulation - Darbe Genişlik Modülasyonu)

- Dijital bir pinden ortalama analog gerilim üretmek için kullanılır (LED parlaklığı, DC motor hız kontrolü).
- **Duty Cycle (Görev Çevrimi):**
  $$\text{Duty Cycle (\%)} = \frac{T_{\text{HIGH}}}{T} \times 100$$
- **Ortalama Gerilim:**
  $$V_{\text{ort}} = V_{\text{max}} \times \frac{\text{Duty Cycle}}{100}$$

```text
  %25 Duty Cycle (V_ort = 0.825V @ 3.3V):
  3.3V | ███         ███         ███         (T_HIGH = 2.5 ms, T_LOW = 7.5 ms)
  0.0V +─┴─┴─────────┴─┴─────────┴─┴─────────

  %50 Duty Cycle (V_ort = 1.65V @ 3.3V):
  3.3V | ██████      ██████      ██████      (T_HIGH = 5.0 ms, T_LOW = 5.0 ms)
  0.0V +─┴────┴──────┴────┴──────┴────┴──────

  %75 Duty Cycle (V_ort = 2.475V @ 3.3V):
  3.3V | █████████   █████████   █████████   (T_HIGH = 7.5 ms, T_LOW = 2.5 ms)
  0.0V +─┴───────┴───┴───────┴───┴───────┴───
       |<--- T --->|
```

### Soru - Cevap
> **Soru:** $3.3\text{V}$ seviyeli bir MCU ile $\%25$ Duty Cycle ve $10\text{ ms}$ periyotlu PWM üretilirse: 1) Ortalama gerilim kaç Volt olur? 2) Kaç ms HIGH, kaç ms LOW kalır?  
> **Öğrenci Cevabı:** 1) $0.825\text{V}$, 2) $2.5\text{ ms}$ HIGH, $7.5\text{ ms}$ LOW.  
> **Değerlendirme:** Doğru ($3.3 \times 0.25 = 0.825\text{V}$).

---

## 3.5 STEP ve DIRECTION Sinyalleri (Step Motor Kontrolü)

- Step motor sürücülerine (A4988, TMC2209 vb.) iki temel sinyal gönderilir:
  - **DIR:** Yön sinyalidir (HIGH = Saat yönü, LOW = Ters yön).
  - **STEP:** Her bir darbe (yükselen kenar) motorun 1 adım ilerlemesini sağlar.
  - **Darbe Adedi $\rightarrow$ Konumu / Pozisyonu belirler.**
  - **Sinyal Frekansı $\rightarrow$ Dönüş Hızını belirler.**

```text
  +-------+   DIR (1=CW, 0=CCW)    +--------------+   Bobin A/B   +------------+
  |  MCU  |----------------------->| Step Sürücü  |==============>| Step Motor |
  |       |----------------------->| (A4988/TMC)  |               | (200 Adım) |
  +-------+  STEP (Darbe Treni)    +--------------+               +------------+

  STEP Sinyali:  _│¯│_│¯│_│¯│_│¯│_   (Her yükselen kenar = 1 Adım, Sayı = Konum, Hız = f)
```

### Soru - Cevap
> **Soru:** 1 turu 200 adım olan step motorun tam 3 tur dönmesi için toplam kaç darbe gönderilmelidir? Bu 3 turun tam 1 saniyede bitmesi için STEP sinyalinin frekansı kaç Hz olmalıdır?  
> **Öğrenci Yanıtı:** 1) 600 darbe. 2. soruda formül soruldu.  
> **Rehberlik:** Frekansın 1 saniyedeki darbe sayısı olduğu hatırlatıldı ($f = \text{Darbe} / \text{Süre}$).  
> **Öğrenci Düzeltmesi:** **$600\text{ Hz}$**.  
> **Değerlendirme:** Doğru.

---

## 🎯 3. Bölüm Genel Değerlendirme (Konveyör ve Fan Senaryosu)

**Senaryo:** $3.3\text{V}$ MCU ile çalışan konveyör bandı, soğutma fanı ve limit anahtarı sistemi.

1. **Soru:** Limit anahtarına basılmadığında pinde $1.2\text{V}$ kararsız voltaj okunması neden tehlikelidir? Kararlı HIGH için hangi direnç nereye bağlanır?
   - **Öğrenci Cevabı:** $1.2\text{V}$ ne HIGH ne LOW belirsiz bölgedir. Pull-up direnci VCC ucuna takılmalıdır. *(Doğru)*
2. **Soru:** Kutunun anahtara ilk çarptığı an ve kutunun anahtardan ayrıldığı an hangi kenarlarla yakalanır?
   - **Öğrenci Cevabı:** 1) Falling edge, 2) Rising edge. *(Doğru)*
3. **Soru:** $500\text{ Hz}$ frekans, $\%40$ Duty Cycle, $3.3\text{V}$ için: 1) Periyot? 2) $T_{\text{HIGH}}$? 3) $V_{\text{ort}}$?
   - **Öğrenci Cevabı:** 1) $2\text{ ms}$, 2) $0.8\text{ ms}$, 3) $1.32\text{V}$. *(Doğru)*
4. **Soru:** 1 turu 400 adım olan motorun $5$ turu $2$ saniyede dönmesi için DIR pini seviyesi, darbe sayısı ve sinyal frekansı ne olmalıdır?
   - **Öğrenci Cevabı:** 1) HIGH, 2) 2000 darbe, 3) $1000\text{ Hz}$. *(Doğru - $2000 / 2 = 1000\text{ Hz}$)*
