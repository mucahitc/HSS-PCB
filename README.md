# HSS-PCB — 2026 Kontrolcü Kartı

HSS-PCB, STM32 tabanlı bir ana kontrolcü kartının Altium tasarım kaynaklarını içerir. Depoda PCB yerleşimi, yedi şematik sayfası, projeye özel şematik/PCB kütüphaneleri ve kart görselleri bulunur. Şematikler; kontrolcü, güç, haberleşme, enkoder arayüzleri, EEPROM, barometre ve harici bağlantı bloklarına ayrılmıştır.

## 1. Amaç ve kapsam

**Dosyalarda görülen tasarım amacı:** `HSS 2026 Blok Diyagram.SchDoc`, ana kontrolcü ile yardımcı kontrolcü, yer kontrol istasyonu, iki enkoder, motor sürücü arayüzleri ve çevre birimleri arasındaki sistem ilişkilerini gösterir. Ana kontrolcü STM32H735IGT6 olarak tanımlanmıştır. Yardımcı kart için blok diyagramında STM32F411 geliştirme kartı belirtilir; yardımcı kartın ayrı tasarım dosyaları bu envanterde bulunmaz. [S1][S2]

Blok diyagramında azimut/zenit hareketi, mesafe ölçer ve silah/lazer tetikleme başlıkları da yer alır. Bunlar çizimdeki sistem işlevi adlarıdır; bu depo tek başına tamamlanmış, çalışır veya güvenliği doğrulanmış bir sistem kanıtı sunmaz. Buradaki inceleme kontrol listesi, belge tutarlılığına ve tehlikeli yüklerden ayrılmış kartın temel elektriksel güvenliğine yöneliktir; hareket, hedefleme veya tetikleme işlevlerinin işletim prosedürünü içermez.

## 2. Reponun nasıl okunacağı

Bu taslak için proje metni, şematik dosyalarının okunabilir bileşen/ağ/başlık kayıtları ve üç PNG görseli incelenmiştir. Altium’da proje derlemesi, elektriksel bağlantı denetimi veya PCB DRC çalıştırılmamıştır. `HSS_2026_PCB.PcbDoc` ile kütüphanelerin varlığı ve boyutları doğrulanmış; iç bağlantıları, footprint doğruluğu ve üretilebilirliği doğrulanmamıştır. Bir ağ adının bulunması, uçtan uca bağlantının doğru olduğunu veya işlevin yazılımla desteklendiğini tek başına göstermez.

## 3. Ana bloklar

| Blok | Kaynakta doğrulanan içerik | Doğrulanması gereken sınır |
| --- | --- | --- |
| Ana kontrolcü | `STM32 Sematik.SchDoc` içinde U4: `STM32H735IGT6`; güç, saat, reset ve debug ile ilgili kayıtlar | Gerçek montaj parçası, pin eşleşmeleri, saat ayarları ve çalışan firmware |
| Güç | `Power Sematik.SchDoc`; giriş koruma bölümü, ayrı 5 V ve 3,3 V ağları, izole olarak adlandırılan güç bölümü | İzin verilen giriş aralığı, akım bütçesi, ısıl sınırlar ve izolasyon performansı |
| Dahili modüller 1 | RS422–TTL, yardımcı kontrolcü haberleşmesi, iki enkoder arayüzü ve EEPROM başlıkları | Protokoller, elektriksel seviyeler, terminasyon ve kablolama |
| Dahili modüller 2 | BMP180 barometre ve motor sürücü arayüzü başlıkları; optokuplör bileşen kayıtları | Harici sürücü uyumluluğu ve sensörün gerçek montaj/çalışma durumu |
| Dahili modüller 3 | Boş UART/I²C, GPIO/timer erişimleri ve montaj delikleri | Erişim noktalarının pin sırası, ayrılmış işlevleri ve mekanik ölçüler |
| Harici modüller | OLED, fan ve lazer/tetikleme olarak adlandırılmış bağlantı bölümleri | Harici cihaz modelleri, yük sınırları ve arayüz gereksinimleri |

Kaynaklar: [S2][S3][S4][S5][S6][S7]. “Motor sürücü” başlığı, kart üzerinde motor akımını sağlayan eksiksiz bir güç katının bulunduğunu kanıtlamaz; mevcut kayıtlar kontrol arayüzünü belgelendirir. Yani kart motor sürücülere çıktı verir.

## 4. STM32 ve kontrol mimarisi

U4, `STM32H735IGT6` olarak kayıtlıdır. Aynı U4 referansının birden fazla şematik parçasında görünmesi, birden fazla MCU bulunduğu anlamına gelmez. Şematikte kontrolcü sinyalleri ve besleme bölümü ayrı çizim parçalarıyla temsil edilir. [S2]

Doğrulanan mimari öğeler:

- UART/USART, SPI ve I²C işlevlerini taşıyan ağ adları bulunur. Şematik başlıkları bunları yardımcı kontrolcü, yer kontrol istasyonu, enkoderler ve çevre birimleriyle ilişkilendirir.
- Ekran, barometre ve EEPROM için ortak I²C işlev adları görülür. Ayrı bir boş I²C ve iki boş UART bağlantısı da belgelenmiştir.
- `NRST`, `BOOT0`, SWDIO, SWCLK ve SWO işlev adları mevcuttur.
- `PH0-OSC_IN`, `PH1-OSC_OUT`, `PC14-OSC32_IN` ve `PC15-OSC32_OUT` ağları ile CR1/CR2 kristal kayıtları vardır. CR1’in kütüphane adı `16_MHz_Crystal`, yorum alanı `RH100` şeklindedir.
- BT1 butonu ve üst görünüşte `RESET` yazısı bulunur; kontrolcü çevresinde çok sayıda bypass/dekuplaj kondansatörü kayıtlıdır.

## 5. Güç bölümleri

Güç şematiğinde `+24V`, `+5V`, `+3.3V`, `+5V_I`, `+3.3V_I`, `GND` ve `GND_I` adları bulunur. Bunlar tasarımın adlandırdığı nominal güç alanlarıdır. [S3]

| Bölüm | Şematikte görülen kayıtlar | Belirsizlik |
| --- | --- | --- |
| Giriş ve ters polarite koruma | CN1 `XT30`, F1 sigorta, Q1 `IRF9540N`, ZD1 ve C1; “Ters Polar Koruma” başlığı | Bağlantı yönü, koruma davranışı, sigortanın kesin tipi ve giriş sınırları doğrulanmalı |
| 5 V dış besleme | “5V Dış Besleme”; CARD1 yorumunda `MP1584EN` | Modülün gerçek modeli, ayarı, yük kapasitesi ve soğutması doğrulanmalı |
| İzole olarak adlandırılan 5 V besleme | U1 `TDN 5-2411WISM`; “İzole 5V Besleme” başlığı | Tam parça kodu, çevre devresi ve kart üzerindeki izolasyon sınırı doğrulanmalı |
| 3,3 V beslemeler | U2/U3 `LD1086DT33TR`; “İzole 3.3V Besleme” ve “3.3V Dış Besleme” başlıkları | Giriş/çıkış bağlantıları, yük bütçesi ve ısıl yeterlilik doğrulanmalı |
| Göstergeler ve ölçüm erişimi | “Güç LED’leri”, T1–T5 test noktaları | Her test noktasının ağı ve referans toprağı doğrulanmalı |

`_I` eki, ilgili sayfa başlıklarıyla birlikte izole alan niyetini gösterir.GND/GND_I ilişkisi PCB, kablolar ve ölçüm cihazları dâhil bütün bağlantı zincirinde değerlendirilmelidir.

Blok diyagramındaki harici besleme ve sistem yük değerleri, bu PCB’nin giriş/çıkış kapasitesi olarak kullanılmamalıdır. Kart için doğrulanmış toplam güç, sürekli akım yoktur.

## 6. Modüller, kütüphaneler ve konnektör yapısı

`Dahili Moduller 1.SchDoc` içinde üç `MAX490ESA+` kaydı, `HCPL-2411`, koruma/terminasyon elemanları ve U8 `M24C64-WMN6TP` EEPROM yer alır. `Dahili Moduller 2.SchDoc` içinde U15 `BMP180`, `IS2801-4` ve `PC814` kütüphane kayıtları vardır. Bunlar seçilmiş şematik bileşenleridir; satın alma veya monte edilmiş parça listesi yerine geçmez. [S4][S5]

Konnektörlerin aşağıdaki özeti, **sembol envanteridir; kablo bağlantı talimatı değildir**:

| Kaynak | Referanslar | Kayıtlı sembol türü |
| --- | --- | --- |
| Güç şematiği | CN1 | XT30 |
| STM32 şematiği | CN2; CN3 | 3 pin header; 6 pin JST adlı sembol |
| Dahili modüller 1 | CN4; CN5, CN6, CN7 | 4 pin; 6 pin JST adlı semboller |
| Dahili modüller 2 | CN14, CN16 | 5 pin JST adlı semboller |
| Dahili modüller 3 | CN11, CN24, CN25 | 4 pin JST adlı semboller |
| Harici modüller | CN9; CN8, CN10, CN12, CN13 | 4 pin; 2 pin JST adlı semboller |

Üst görünüşte YKK, RS422 TTL, Enkoder 1/2, OLED, boş UART/I²C ve motor sürücü bölge yazıları görülebilir. Alt görünüşte bir modül ve ek erişim pedleri vardır. Bu görseller yerleşim incelemesine yardımcı olur; üretim montaj çizimi veya pinout belgesi değildir. [S8][S9]

Özel kütüphaneler:

- `HSS 2026 Sematik Kutubhane.SchLib`: projeye ait şematik sembol kütüphanesi.
- `HSS 2026 PCB Kutubhane.PcbLib`: projeye ait PCB footprint kütüphanesi.

Sembol pinleri, footprint pad numaraları ve gerçek parçalar arasındaki eşleşme ayrıca doğrulanmalıdır. 3B model görünüşü tek başına bu eşleşmeyi doğrulamaz.

## 7. Layer stack-up ve kart görselleri

`Stack-up.png` dört bakır katman gösterir. Dosya adı büyük `S` ile başlar. Görseldeki sıra yukarıdan aşağıya şöyledir: [S10]

| Sıra | Ad / malzeme | Tür | Görseldeki bakır ağırlığı | Görseldeki kalınlık |
| --- | --- | --- | --- | --- |
| — | Top Solder | Lehim maskesi | — | 0.4 mil |
| L1 | SIG_1/POW / CF-004 | Signal | 1 oz | 1.378 mil |
| — | Dielectric 2 / PP-006 | Prepreg | — | 2.8 mil |
| L2 | GND / CF-004 | Plane | 1/2 oz | 0.689 mil |
| — | Dielectric 1 / FR-4 | Dielectric | — | 12.6 mil |
| L3 | PWR / CF-004 | Plane | 1/2 oz | 0.689 mil |
| — | Dielectric 3 / PP-006 | Prepreg | — | 2.8 mil |
| L4 | SIG_2/GND / CF-004 | Signal | 1 oz | 1.378 mil |
| — | Bottom Solder | Lehim maskesi | — | 0.4 mil |

Top/Bottom Overlay satırları da görselde vardır; bakır katman sayısına dâhil değildir. Katman adları kullanım niyetini gösterir. Özellikle GND ve PWR adları, düzlemlerin bölünmeden tek bir ağa bağlandığını kanıtlamaz.

Görseller: [üst görünüş][S8] · [alt görünüş][S9] · [katman yapısı][S10]. Bunlar CAD görselleridir; fiziksel üretim veya test fotoğrafı olarak değerlendirilmemelidir.

## 8. Dosya envanteri

İncelenen commit’te kök dizinde 16 dosya vardır. Aşağıdaki boyutlar GitHub depo ağacındaki bayt değerleridir. [S0]

| Dosya | Boyut (bayt) | İçerik / rol |
| --- | ---: | --- |
| `2026_HSS_Kontrocu_Karti.PrjPcb` | 41527 | Ana Altium PCB projesi; açılış noktası |
| `2026_HSS_Kontrocu_Karti.PrjPcbStructure` | 758 | Şematik yapı/sayfa ilişkisi kaydı |
| `HSS 2026 Blok Diyagram.SchDoc` | 55808 | Sistem blok diyagramı; sayfa 1 |
| `Power Sematik.SchDoc` | 133120 | Güç şematiği; sayfa 2 |
| `STM32 Sematik.SchDoc` | 400384 | Ana kontrolcü; sayfa 3 |
| `Dahili Moduller 1.SchDoc` | 260096 | Haberleşme, enkoder arayüzleri, EEPROM; sayfa 4 |
| `Harici Moduller.SchDoc` | 68608 | Harici bağlantılar; sayfa 5 |
| `Dahili Moduller 2.SchDoc` | 147456 | Barometre ve motor sürücü arayüzleri; sayfa 6 |
| `Dahili Moduller 3.SchDoc` | 86528 | Boş arayüzler/pedler ve montaj delikleri; sayfa 7 |
| `HSS_2026_PCB.PcbDoc` | 11169280 | PCB tasarım belgesi |
| `HSS 2026 Sematik Kutubhane.SchLib` | 137216 | Özel şematik kütüphanesi |
| `HSS 2026 PCB Kutubhane.PcbLib` | 6476288 | Özel PCB kütüphanesi |
| `Stack-up.png` | 27153 | Katman yapısı ekran görüntüsü |
| `Top-view.png` | 278019 | Üst CAD görünüşü |
| `Bottom-view.png` | 143641 | Alt CAD görünüşü |
| `README.md` | 85 | Taslaktan önce yalnızca kullanıcı eki bağlantısı ve boş satırlar |

## 9. Kaynaklar

Aşağıdaki bağlantılar aynı commit’e sabitlenmiştir. Daha yeni bir tasarım revizyonunda bu README yeniden doğrulanmalıdır.

- [İncelenen depo ağacı][S0]
- [Sistem blok diyagramı][S1]
- [STM32 şematiği][S2]
- [Güç şematiği][S3]
- [Dahili modüller 1][S4], [dahili modüller 2][S5], [dahili modüller 3][S6]
- [Harici modüller][S7]
- [Üst görünüş][S8], [alt görünüş][S9], [stack-up][S10]
- [Altium proje dosyası][S11] ve [proje yapı kaydı][S12]
- [Önceki README’de bulunan ek bağlantısı](https://github.com/user-attachments/assets/0f2163db-d62e-4ff3-9ee2-b9bd5b7222f3): içeriği bu teknik doğrulamanın dayanağı olarak kullanılmamıştır.

[S0]: https://github.com/mucahitc/HSS-PCB/tree/45e84431dabc1762213e53c8db8c0698ee49b6e4
[S1]: https://github.com/mucahitc/HSS-PCB/blob/45e84431dabc1762213e53c8db8c0698ee49b6e4/HSS%202026%20Blok%20Diyagram.SchDoc
[S2]: https://github.com/mucahitc/HSS-PCB/blob/45e84431dabc1762213e53c8db8c0698ee49b6e4/STM32%20Sematik.SchDoc
[S3]: https://github.com/mucahitc/HSS-PCB/blob/45e84431dabc1762213e53c8db8c0698ee49b6e4/Power%20Sematik.SchDoc
[S4]: https://github.com/mucahitc/HSS-PCB/blob/45e84431dabc1762213e53c8db8c0698ee49b6e4/Dahili%20Moduller%201.SchDoc
[S5]: https://github.com/mucahitc/HSS-PCB/blob/45e84431dabc1762213e53c8db8c0698ee49b6e4/Dahili%20Moduller%202.SchDoc
[S6]: https://github.com/mucahitc/HSS-PCB/blob/45e84431dabc1762213e53c8db8c0698ee49b6e4/Dahili%20Moduller%203.SchDoc
[S7]: https://github.com/mucahitc/HSS-PCB/blob/45e84431dabc1762213e53c8db8c0698ee49b6e4/Harici%20Moduller.SchDoc
[S8]: https://github.com/mucahitc/HSS-PCB/blob/45e84431dabc1762213e53c8db8c0698ee49b6e4/Top-view.png
[S9]: https://github.com/mucahitc/HSS-PCB/blob/45e84431dabc1762213e53c8db8c0698ee49b6e4/Bottom-view.png
[S10]: https://github.com/mucahitc/HSS-PCB/blob/45e84431dabc1762213e53c8db8c0698ee49b6e4/Stack-up.png
[S11]: https://github.com/mucahitc/HSS-PCB/blob/45e84431dabc1762213e53c8db8c0698ee49b6e4/2026_HSS_Kontrocu_Karti.PrjPcb
[S12]: https://github.com/mucahitc/HSS-PCB/blob/45e84431dabc1762213e53c8db8c0698ee49b6e4/2026_HSS_Kontrocu_Karti.PrjPcbStructure



https://github.com/user-attachments/assets/0f2163db-d62e-4ff3-9ee2-b9bd5b7222f3


