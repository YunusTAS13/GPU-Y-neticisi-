# GPU Yöneticisi (GPU Manager)

Linux işletim sistemlerinde hibrit grafik kartı modları arasındaki geçişi otomatikleştiren, harici ve dahili ekran kartlarının yönetimini arka planda sessizce yürüten ve sistem altyapısını dinamik olarak tespit eden hafif, bağımsız bir grafik arayüz (GUI) uygulamasıdır. 

Bu proje, özellikle dizüstü bilgisayar kullanıcılarının Linux dağıtımlarında yaşadığı GPU mod değişimi esnasındaki terminal karmaşasını ve bağımlılık sorunlarını çözmek amacıyla Rust programlama dili ve egui kütüphanesi kullanılarak geliştirilmiştir.

---

## Projenin Temel Amaçları ve Çözdüğü Sorunlar

Linux dünyasında donanım yönetimi genellikle uçbirim (terminal) komutları üzerinden yürütülmektedir. Grafik modu değiştirmek isteyen bir kullanıcının her seferinde ilgili aracın parametrelerini hatırlaması, sudo yetkileriyle komut çalıştırması ve arka planda açık kalan işlemler nedeniyle modlar arasında asılı kalması sık karşılaşılan problemlerdir. 

GPU Yöneticisi bu süreci şu yaklaşımlarla optimize eder:

1. **Terminal Penceresi İzolasyonu:** Grafik modu geçiş komutlarını işletim sisteminin alt katmanında (Process CLI) yürütür. Kullanıcının karşısına herhangi bir siyah uçbirim ekranı fırlatmaz, masaüstü deneyimini bozmaz.
2. **Evrensel Dağıtım ve Altyapı Uyumluluğu:** Program, üzerinde çalıştığı Linux dağıtımının ne olduğundan ziyade, sistemde aktif olan GPU yönetim mekanizmasını tespit eder. Dağıtımdan bağımsız olarak evrensel bir çalışma modeli sunar.
3. **Sıfır Bağımlılık ve Taşınabilirlik (AppImage):** Rust diliyle statik olarak derlenen proje, AppImage paketleme formatı sayesinde sisteme hiçbir harici kütüphane veya paket kurmanızı gerektirmez. Tek bir dosya halinde her dağıtımda çalışabilir.

---

## Teknik Mimari ve Akıllı Altyapı Tespiti

Uygulama başlatıldığı anda ilk olarak arka planda çalışan Linux sistem araçlarını tarar. `detect_system_tool` fonksiyonu aracılığıyla sistem hiyerarşisinde tanımlı olan belirli yolları denetler.

Programın desteklediği ve otomatik olarak algıladığı üç ana mekanizma şunlardır:

* **/usr/bin/system76-power:** Özellikle Pop!_OS ve Ubuntu tabanlı sistemlerde kullanılan, System76 tarafından geliştirilen gelişmiş güç ve grafik yöneticisi.
* **/usr/bin/envycontrol:** Fedora, Arch Linux, CachyOS ve Void Linux gibi modern dağıtımlarda yoğun olarak tercih edilen, CLI tabanlı evrensel ekran kartı anahtarlayıcısı.
* **/usr/bin/supergfxctl:** ASUS ROG ve TUF serisi dizüstü bilgisayarlar için optimize edilmiş, özel sürücü katmanlarıyla konuşabilen grafik yönetim servisi.

Arayüz üzerindeki butonlar ve yenileme mekanizmaları, yukarıdaki hiyerarşiye göre hangi araç bulunduysa ona uygun komut setlerini (`integrated`, `hybrid`, `compute`, `nvidia` vb.) tetikleyecek şekilde dinamik olarak yeniden yapılandırılır.

---

## Grafik Modları Arasındaki Farklar ve Çalışma Mantığı

Uygulama üzerinden yönetilebilen grafik modlarının donanımsal davranışları şu şekildedir:

### Entegre Grafik (Integrated Mode)
Bu mod seçildiğinde harici ekran kartı (NVIDIA) donanımsal düzeyde tamamen kapatılır ve derin uyku (Deep Sleep) moduna alınır. Sistemdeki tüm pencereler, masaüstü efektleri ve ekran çıkışları işlemcinin içerisindeki dahili grafik birimi (Intel HD Graphics / AMD Radeon Vega) tarafından işlenir. Kart elektrik tüketmediği için maksimum pil tasarrufu elde edilir.

### Hibrit Grafik (Hybrid Mode)
Hem dahili hem de harici ekran kartı aynı anda aktif durumdadır. Masaüstü arayüzü ve hafif uygulamalar dahili grafik kartı üzerinden çalışarak enerji tasarrufu sağlamaya devam eder. Ancak bir oyun, 3D modelleme programı veya ağır bir render motoru başlatıldığında, sistem yükü otomatik olarak harici NVIDIA karta devreder.

### Hesaplama Modu (Compute Mode)
Masaüstü pencerelerini ve ekran çıktısını yine dahili grafik kartı yönetir; harici ekran kartı ekrana görüntü çizmekle uğraşmaz. Fakat harici kart tamamen kapatılmaz, arka planda milisaniyeler içinde uyanacak şekilde tetikte bekletilir. Bilgisayarda Ollama, yerel yapay zeka modelleri (Qwen, GLM vb.), CUDA betikleri veya makine öğrenimi algoritmaları çalıştırıldığı anda harici GPU devreye girerek matematiksel hesaplamaları sırtlar.

---

## Lisans Bilgisi

Bu yazılım, açık kaynak topluluğunun özgürlüğünü korumak adına **GNU GPL v3.0 (General Public License)** ile lisanslanmıştır. 

GPL v3.0 lisans kuralları gereği:
* Bu programın kodlarını özgürce inceleyebilir, kopyalayabilir ve dağıtabilirsiniz.
* Kodlar üzerinde değişiklik yapabilir ve kendi projelerinize entegre edebilirsiniz.
* **Kritik Kural:** Bu projenin kodlarını kullanarak veya değiştirerek geliştirdiğiniz yeni yazılımları da yine aynı şekilde açık kaynaklı ve GPL v3.0 lisanslı olarak topluluğa sunmak zorundasınız. Kodların kapatılarak ticari bir sır haline getirilmesi yasal olarak engellenmiştir.

---

## Kurulum ve Kullanım Kılavuzu

Program taşınabilir yapıda olduğu için geleneksel bir kurulum sürecine ihtiyaç duymaz. Çalıştırmak için aşağıdaki adımları izlemeniz yeterlidir:

1.  GitHub deposunun ana sayfasından veya Sürümler (Releases) kısmından `GPU_Yoneitici_GPL_Linux.zip` dosyasını indirin.
2.  İndirdiğiniz arşiv dosyasını sağ tıklayarak bir klasöre çıkartın.
3.  Klasör içerisindeki `GPU_Yoneitici-x86_64.AppImage` dosyasına sağ tıklayıp **Özellikler** penceresini açın.
4.  **İzinler (Permissions)** sekmesine geçerek **"Dosyayı bir program gibi çalıştırmaya izin ver" (Allow executing file as program)** seçeneğini işaretleyin.
5.  Ayarları kapatıp dosyaya çift tıklayarak uygulamayı başlatın.

*Alternatif olarak terminal üzerinden çalıştırmak isterseniz şu komutları uygulayabilirsiniz:*
```bash
chmod +x GPU_Yoneitici-x86_64.AppImage
./GPU_Yoneitici-x86_64.AppImage
