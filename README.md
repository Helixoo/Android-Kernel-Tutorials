> [!CAUTION]
> **Bu kılavuzu kullanarak tüm riskleri kabul etmiş olursunuz -** cihazın brick olması, önyükleme hataları veya diğer sorunlar dahil. **Herhangi bir hasar için sorumluluk kabul etmiyoruz.**
> 
> Sorular **yalnızca** **tüm belgeleri okuduysanız** ve **önce kendi araştırmanızı yaptıysanız** dikkate alınacaktır.

## İlk Android Çekirdeğinizi Derlemek İçin Başlangıç Dostu Bir Kılavuz!

![Android](https://img.shields.io/badge/Android-3DDC84?logo=android&logoColor=white)
[![Linux](https://img.shields.io/badge/Linux-FCC624?logo=linux&logoColor=black)](#)
[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](/LICENSE)
[![Telegram](https://img.shields.io/badge/Telegram-2CA5E0?logo=telegram&logoColor=white)](https://t.me/SamsungTweaks)

**Neler Öğreneceksiniz:**  

- Çekirdek kökünü (kernel root) anlamak & derleme için doğru derleyicileri seçmek
- Çekirdeği özelleştirmek ve çekirdek yamalarını uygulamak.  
- Samsung'un anti-root korumalarını kaldırmak.  
- Derlenmiş çekirdekten imzalı bir önyükleme imajı (boot image) oluşturmak

**Gereksinimler:**
- Çalışan bir 🧠  
- Sabır  
- x86_64 (AMD64) Linux tabanlı bir PC/Sunucu (Debian tabanlı önerilir)  
- Temel Linux komutları ve Bash betikleme bilgisi  
- Temel sürüm kontrolü (Git) bilgisi  
  - Bu, bir çekirdek oluştururken iyi bir uygulamadır. Bazı dosyaları düzenlediğinizi ve kaynağı karıştırdığınızı fark ettiğinizi hayal edin - bu tek komut `git stash`, yaptığınız tüm kaydedilmemiş (uncommitted) değişiklikleri geri almanıza yardımcı olabilir. Ne kadar havalı değil mi :)
  
  - Çekirdek derlemeyi öğrenmeye başlamadan **önce** [buradan biraz Git öğrenin](./Git-for-beginners/)!
	
### 🛠 Çekirdekleri derlemek için gerekli bağımlılıkları yükleyin

> [!TIP]
> En güvenilir ve sorunsuz deneyim için, herhangi bir işletim sisteminde çalışan, çekirdek derleme için kararlı, test edilmiş bir ortam sağlayan önceden yapılandırılmış Docker kapsayıcımızı kullanmanızı **şiddetle öneririz**. [Sürümler sayfasından](https://github.com/ravindu644/Android-Kernel-Tutorials/releases) indirin ve ekli talimatları izleyin.

<details>
<summary><strong>Docker kapsayıcısının nasıl göründüğünü görmek için genişletin</strong></summary>

![Kernel Builder Docker Container](./screenshots/kernel-builder.png)

*Fedora üzerinde çalışan Ubuntu tabanlı Docker kapsayıcısının ekran görüntüsü (tam kalitede görüntülemek için tıklayın)*

</details>

Ancak, Docker kapsayıcısını kullanmak istemiyorsanız, Ubuntu/Fedora için bağımlılıkları yükleme komutları şunlardır:

<details>
<summary><strong>🟧 Ubuntu/Debian tabanlı dağıtımlar (Ubuntu, Linux Mint, Debian, vb.)</strong></summary>

```bash
sudo apt update && sudo apt install -y git device-tree-compiler lz4 xz-utils zlib1g-dev openjdk-17-jdk gcc g++ python3 python-is-python3 p7zip-full android-sdk-libsparse-utils erofs-utils \
default-jdk git gnupg flex bison gperf build-essential zip curl libc6-dev libncurses-dev libx11-dev libreadline-dev libgl1 libgl1-mesa-dev \
python3 make sudo gcc g++ bc grep tofrodos python3-markdown libxml2-utils xsltproc zlib1g-dev python-is-python3 libc6-dev libtinfo6 \
make repo cpio kmod openssl libelf-dev pahole libssl-dev libarchive-tools zstd rsync --fix-missing && wget http://security.ubuntu.com/ubuntu/pool/universe/n/ncurses/libtinfo5_6.3-2ubuntu0.1_amd64.deb && sudo dpkg -i libtinfo5_6.3-2ubuntu0.1_amd64.deb
```
</details>

<details>
<summary><strong>🟦 Fedora/Red Hat tabanlı dağıtımlar (Fedora, CentOS, RHEL, vb.)</strong></summary>

```bash
sudo dnf group install "c-development" "development-tools" && \
sudo dnf install -y git dtc lz4 xz zlib-devel java-17-openjdk-devel python3 \
p7zip p7zip-plugins android-tools erofs-utils java-latest-openjdk-devel \
ncurses-devel libX11-devel readline-devel mesa-libGL-devel python3-markdown \
libxml2 libxslt dos2unix kmod openssl elfutils-libelf-devel dwarves \
openssl-devel libarchive zstd rsync
```
</details>

<br>

### Hızlı Bağlantılar : 
01. 📁 [Cihazınız için çekirdek kaynak kodunu indirme](#downloading-kernel-source)
02. 🧠 [Çekirdek kökünü (Kernel root) anlamak](#understanding-kernel-root)
03. 🧠 [non-GKI ve GKI çekirdeklerini anlamak](#understanding-non-gki-gki-kernels)
04. 👀 [Derlemeye Hazırlık](#preparing-for-compilation)
05. ⚙️ [Çekirdeği Özelleştirme (Geçici Yöntem)](#customizing-kernel-temporary-method)
06. ⚙️ [Çekirdeği Özelleştirme (Kalıcı Yöntem)](#customizing-kernel-permanent-method)
07. [⁉️ Samsung'un anti-root korumaları nasıl kaldırılır?](#nuke-samsung-anti-root-protections)
08. 🟢 [Ek Yamalar](#additional-patches)
09. ✅ [Çekirdeği Derlemek](#compiling-the-kernel)
10. 🟥 [Bilinen derleme sorunlarını düzeltme](#fixing-known-compiling-issues)
11. 🟡 [Derlenmiş Çekirdekten İmzalı Bir Önyükleme İmajı (Boot Image) Oluşturma](#building-signed-boot-image)

---

> [!NOTE]
> Yeni başlayan biri değilseniz ve resmi Google kaynaklarından bir GKI 2.0 çekirdeği oluşturmak istiyorsanız, [gki-2.0](https://github.com/ravindu644/Android-Kernel-Tutorials/tree/gki-2.0) dalına (branch) geçin.
>
> Harika eğitim için [@TheWildJames](https://github.com/TheWildJames)'e teşekkürler!
>
> Yapılacaklar:
>
> - Özelleştirme destekli otomatik bir çekirdek oluşturmak için Samsung/Google'ın resmi GKI Derleme Sistemlerini (1.0 / 2.0+) kullanma hakkında ayrı bir kılavuz yazın. 
>
> - Çakışmalara veya cihaz çökmelerine neden olmadan 500'den fazla oluşturulmuş Yüklenebilir Çekirdek Modülünü (.ko sürücüleri) `vendor_boot` ve `vendor_dlkm` imajlarına bağlama ve enjekte etme konusunda bir kılavuz yazın.

---

<h2 id="downloading-kernel-source"> ✅ Cihazınız için çekirdek kaynak kodunu indirme</h2>

- **⚠️ Cihazınız Samsung ise,**

#### 01. Çekirdek kaynağını buradan indirin: [Samsung Opensource]( https://opensource.samsung.com/main)

<img src="./screenshots/1.png">

#### 02. Kaynak zip dosyasından ```Kernel.tar.gz``` dosyasını çıkarın, bu komutu kullanarak arşivden çıkarın ve lütfen bunu yapmak için herhangi bir uygulama kullanmayın:

```bash
tar -xvf Kernel.tar.gz && rm Kernel.tar.gz
```

<img src="./screenshots/2.png">

**Not:** Dosya ve klasörlerdeki salt okunur hatalarını kaldırmak için tüm çekirdek dizinine 755 izni vermek iyi bir fikirdir. Bu, dosyaları düzenlerken ve çekirdeği güncellerken (upstreaming) sorunları önler.

**Düzeltmek için bu komutu çalıştırın:**

```
chmod +755 -R /path/to/extracted/kernel/
```

**Önce:**
<img src="./screenshots/3.png">

**Sonra:**
<img src="./screenshots/4.png">

**Aşağıdaki video yukarıda belirtilen tüm adımları göstermektedir:** 

[🎥 Samsung'un Kernel.tar.gz dosyasını çıkarma & gerekli izinleri verme](https://www.youtube.com/watch?v=QLymPkTpC2Y)

<hr>

- **⚠️ Diğer cihazlar için,** Bunları OEM'inizin sitelerinde veya OEM'inizin **resmi** GitHub depolarında bulabilirsiniz:

  <img src="./screenshots/13.png">

## <span id="understanding-non-gki-gki-kernels">✅ `non-GKI` ve `GKI çekirdeklerini` anlamak</span>

### 01. GKI projesi tanıtımı

- **Genel Çekirdek İmajı (Generic Kernel Image)** veya **GKI**, **çekirdek çekirdeğini (kernel core) birleştirerek ve SoC ve Kart desteğini çekirdek çekirdeğinden yüklenebilir satıcı modüllerine taşıyarak** çekirdek parçalanmasını azaltmayı (ve ayrıca Android kararlılığını artırmayı) amaçlayan bir Android projesidir.

### 02. `pre-GKI`/`non-GKI` ve `GKI` linux sürüm tablosu
| Pre-GKI | GKI 1.0 | GKI 2.0 |
|---------|---------|---------|
| 3.10    | 5.4     | 5.10    |
| 3.18    |         | 5.15    |
| 4.4     |         | 6.1     |
| 4.9     |         | 6.6     |
| 4.14    |         |         |
| 4.19    |         |         |
#### Açıklama:

1. **pre-GKI veya non-GKI**:
   - En eski Android çekirdek dalı, muhtemelen Linux sürüm 2.x'ten başlar.
   - Bu çekirdekler **cihaza özgüdür** çünkü genellikle SoC'lerin ve OEM'lerin ihtiyaçlarını karşılamak için büyük ölçüde değiştirilmiştir.
   - ACK'de kullanımdan kaldırılmaya başlandı, çünkü `linux-4.19.y` dalı son Linux 4.19.325 ile zaten EoL (Ömür Sonu) durumuna ulaştı.

3. **GKI 1.0**:
   - Android'in Genel Çekirdek İmajı'nın ilk nesli, çekirdek sürümü **5.4** ile başlar.
   - Bu ilk GKI nesli yalnızca android11-5.4 ve android12-5.4 dallarına sahiptir ve Google, GKI 1.0'ın kullanımdan kaldırıldığını duyurdu.
   - GKI'nın ilk nesli, GKI proje hedeflerine ulaşamadığı için ikinci nesil GKI kadar henüz olgunlaşmamıştır.
   - Bu çekirdekler **cihaza özgü** olarak kabul edilir, ancak daha yaygındır, OEM'lerin ve SoC Üreticilerinin onlara nasıl davrandığına bağlıdır.
   - SoC Üreticileri genellikle SoC özelliklerini eklemek için GKI 1.0 çekirdeğini değiştirir. Bu değişikliklerden **Mediatek GKI (mGKI)** ve **Qualcomm GKI (qGKI)** terimleri ortaya çıkar.

4. **GKI 2.0**:
   - Android'in Genel Çekirdek İmajı'nın ikinci nesli, çekirdek sürümü **5.10** ile başlar.
   - Bu ikinci nesilde, GKI projesi düzgün bir şekilde olgunlaşmaya başlıyor.
   - Bu çekirdek "evrensel" olarak kabul edilir, çünkü Google'ın GKI çekirdek kaynağıyla oluşturulan bir GKI çekirdeğini, doğru ve eşleşirse **bazı** cihazlarda önyükleyebilirsiniz.

### Notlar:
- **LTS = Long-Term Support (Uzun Süreli Destek)**: Bu çekirdekler kararlıdır, iyi korunur ve uzun süreli güncellemeler alır.
- **GKI = Generic Kernel Image (Genel Çekirdek İmajı)**: Android cihazlarda çekirdeği standartlaştırmak için Google tarafından tanıtılan birleşik bir çekirdek çerçevesi.
- **SoC = System on Chip (Yonga Üzerinde Sistem)**
- **ACK = Android Common Kernel (Android Ortak Çekirdeği)**: Android ihtiyaçlarını karşılamak üzere değiştirilmiş bir Android linux LTS çekirdek dalı.
- Samsung gibi OEM'ler ihtiyaçlarını karşılamak için GKI 2.0 çekirdeklerini değiştirmeye devam edebilir ve bozuk SD Kart ve bozuk Ses gibi bazı sorunlara neden olabilir. 
  - **Bu yüzden mümkünse onların GKI çekirdek kaynağını kullanın.**

- 4.19 çekirdekleri için, bunlar ağırlıklı olarak non-GKI uygulamalarıdır, çünkü gerçek GKI, Android 11 ile çekirdek 5.4'e kadar resmi olarak tanıtılmamıştı. 

  - OEM'ler genellikle 4.19 için Android Ortak Çekirdeği'ne dayalı, yoğun şekilde özelleştirilmiş, cihaza özgü uygulamalar kullanır. İlgileniyorsanız Android Ortak Çekirdeği deposuna başvurabilirsiniz.
  - Bilginiz olsun, 4.19 ile deneysel GKI geliştirmesi vardı (android-4.19-gki-dev dalı), ancak bu yaygın olarak dağıtılmadı. Resmi GKI uygulaması çekirdek 5.4 ile başladı.
  - Örnekler:
     1. Çekirdek 4.19'a sahip çoğu Samsung cihazı, OEM'e özgü değişikliklerle non-GKI uygulamaları kullanır.
     2. Gerçek GKI benimsemesi, Android 11+ ile gelen çekirdek 5.4 veya daha yüksek sürümlü yeni cihazlarla standart hale geldi.

## <span id="understanding-kernel-root">✅ ```Çekirdek kökünü``` (Kernel root) anlamak</span>

<img src="./screenshots/6.png">

- Yukarıdaki ekran görüntüsünde görebileceğiniz gibi, bu Linux çekirdek kaynak kodudur.
- **Terminalde mavi ile vurgulanan** klasörlere sahip olmalıdır.
- **Geleneksel GKI çekirdeklerinde,** çekirdek kökü "common" adlı bir klasörde bulunur.

- **GKI Samsung Qualcomm çekirdek kaynaklarında**, derleme için `msm-kernel` yerine `common` çekirdeği kullanmalısınız.
- **Bazı GKI Samsung MediaTek çekirdek kaynaklarında**, çekirdek kökü `kernel-SÜRÜM.YAMA_DÜZEYİ` olarak adlandırılır.
  - örn., `kernel-5.15`

## <span id="preparing-for-compilation">✅ Derlemeye Hazırlık</span>

- Çekirdeği derlemenin 2 yolu vardır.  

1. Bir derleme betiği **olmadan**.  
2. Bir derleme betiği **ile**.  

Yeni başlıyorsanız, önce çekirdeği bir derleme betiği olmadan oluşturmayı denemenizi öneririm. Mantığı anladıktan sonra, hayatınızı kolaylaştırmak için bir derleme betiği kullanabilirsiniz :)

---

## 🟠 Yöntem 1: Bir derleme betiği olmadan.

### 01. Doğru derleyiciyi seçmek.

- Çekirdeği derlemeden önce, çekirdeğimizi oluşturmak için kullanılacak uyumlu derleyicileri belirlemeliyiz.

- Çekirdek sürümünüzü kontrol etmek için `Makefile` dosyanızı açabilirsiniz.  

  ![Makefile ekran görüntüsü](./screenshots/31.png)  
  *Çekirdek sürümü = `VERSION.PATCHLEVEL.SUBLEVEL`*

- Benim durumumda, çekirdek sürümü **4.14.113**.

- **Çekirdek sürümünüz için doğru derleyiciyi seçme** hakkında tam bilgiyi [burada](./toolchains/) (based on my experience, btw).

- Benim durumumda bunlar: [clang-r383902b](https://github.com/ravindu644/Android-Kernel-Tutorials/releases/download/toolchains/clang-r383902b.tar.gz), [arm-gnu-toolchain-14.2.rel1-x86_64-aarch64-none-linux-gnu](https://github.com/ravindu644/Android-Kernel-Tutorials/releases/download/toolchains/arm-gnu-toolchain-14.2.rel1-x86_64-aarch64-none-linux-gnu.tar.xz)

- Çekirdek sürümünüz için doğru derleyici(ler)i oradan indirin ve bunları yeni bir klasör(ler)e şu şekilde çıkarın:

  ![Makefile ekran görüntüsü](./screenshots/32.png)  
  *Çıkarılan clang*

  ![Makefile ekran görüntüsü](./screenshots/33.png)  
  *Çıkarılan çapraz derleyici (cross compiler)*

---
### 02. Derleyici konumlarını PATH'e aktarmak (export)

- Doğru derleyicileri indirmiş olsak bile, sistemimiz (Ana İşletim Sistemi) çekirdeğimizi oluşturmak için hangi derleyiciyi kullanacağını otomatik olarak bilmeyecektir.  

- Varsayılan olarak, sistemin derleyicilerini kullanacaktır, bu da eski çekirdeklerle uyumsuz olabilir.  
  → Böyle bir durumda, derleme anında başarısız olacaktır.  

- Bu yüzden görevimiz, indirilen derleyicileri sistemimizin `PATH`'ine bağlamaktır.  
  Sisteme şunu söylemeliyiz: "Kendi clang'ini değil, buradaki `clang` ikili dosyasını (binary) kullan!"  

---

#### 💡 `PATH` nedir?
`PATH`, Linux/Unix'te dizinlerin bir listesini saklayan bir ortam değişkenidir.  
Bir komut yazdığınızda (örneğin `clang` veya `gcc`), sistem ilk eşleşen çalıştırılabilir dosyayı bulmak için `PATH` içindeki dizinlere **soldan sağa** bakar.  

İndirdiğiniz derleyicinin klasörünü `PATH`'in **başına** ekleyerek, derleme sisteminin sistem varsayılanı yerine **sizin derleyicinizi** seçmesini sağlarsınız.

---

- `PATH` değişkeninizin neye benzediğini kontrol etmek için terminalde `echo $PATH` yazabilirsiniz:  

  ![PATH ekran görüntüsü](./screenshots/34.png)  
  - Amacımız derleyicilerimizin konumlarını `/usr/local/sbin`'in sol tarafına eklemektir :)

- Çıkarılan derleyici klasörlerinde, ikili dosyalar (çalıştırılabilir dosyalar) genellikle `bin` klasörünün içinde bulunur, bunun gibi:  

  ![Bin klasörü ekran görüntüsü](./screenshots/35.png)

- O `bin` klasörünün tam yolunu kopyalayın ve bu konumları `PATH`'e şu şekilde aktarın:  

  ```bash
  export PATH="/path/to/first/compiler/bin:/path/to/second/compiler/bin:$PATH"
  ```

- **Benim durumumda,** şöyle görünüyordu:

  ```bash
  export PATH="/home/kernel-builder/toolchains/clang-r383902b/bin:/home/kernel-builder/toolchains/gcc/arm-gnu-toolchain-14.2.rel1-x86_64-aarch64-none-linux-gnu/bin:$PATH"
  ```

**Gördüğünüz gibi, araç zincirlerini (toolchains) başarıyla `PATH`'imize aktardık:**

  ![Bin klasörü ekran görüntüsü](./screenshots/36.png)

**Doğrulama için,** terminale `clang -v` yazarak gerçekten bağlandığını doğrulayın!

  ![Bin klasörü ekran görüntüsü](./screenshots/37.png)
  *Başardık!*

---

### 03. Çekirdeği `make` ile derlemek

- Adım 02'de dışa aktardığımız `PATH` değişkeninin **yalnızca o anda açık olan terminalde geçerli olduğunu** unutmayın.  

  **Bu yüzden onu kapatmayın** - çekirdek kaynağında gezinmek ve daha fazla derleme komutu çalıştırmak için o terminal penceresini kullanın.

- **Şimdi,** o terminal penceresini kullanarak, **çekirdek kaynağınızın köküne** şu şekilde gidin: `cd /path/to/kernel-root`

  ![Bin klasörü ekran görüntüsü](./screenshots/38.png)

---

**💡 Bilmekte Fayda Var:** Bir **defconfig** (varsayılan yapılandırma), çekirdek için bir ön ayar dosyası gibidir.

- Derleme sistemine hangi özelliklerin etkinleştirileceğini veya devre dışı bırakılacağını söyler.
- Yaygın defconfig konumları `arch/arm64/configs` veya `arch/arm64/configs/vendor` şeklindedir.

---

- **Benim durumumda,** defconfig dosyam `arch/arm64/configs` konumunda ve adı `exynos9820-beyondxks_defconfig`.

  - **Ayrıca,** **özel amaçlarım** için yapılmış birden fazla defconfig dosyam var, isimleri: `common.config`, `ksu.config` ve `nethunter.config`.
  - Belirli değişiklikler için kendi özelleştirilmiş defconfig'lerinizi de oluşturabilirsiniz (buna daha sonra değineceğiz)!

- Şimdi, derleyicilerimize "çekirdeği oluşturmak için bu defconfig'leri kullan" dememiz gerekiyor!  
- Bunu yapmak için, sadece aşağıdaki komutu çalıştırın:

```bash
make \
  ARCH=arm64 \
  CC=clang \
  CROSS_COMPILE=aarch64-none-linux-gnu- \
  CLANG_TRIPLE=aarch64-none-linux-gnu- \
  senin_defconfigin senin_ikinci_defconfigin senin_ucuncu_defconfigin
```
---

**💡 Açıklama:**

1. **ARCH=arm64** → Oluşturduğumuz çekirdeğin mimarisini belirtir.

    - Bizim durumumuzda, bu 64-bit ARM'dir.

2. **CC=clang** → `make` komutuna `clang` derleyicisini kullanmasını söyler.

    - **Bu değeri değiştirmeyin.** Olduğu gibi kalsın!

3. **CROSS_COMPILE=aarch64-none-linux-gnu-** → Çapraz derleyici ikili dosyaları için önek (örn. `aarch64-none-linux-gnu-gcc`).

    - Bu değeri GCC'nizin `bin` klasörünü açarak alabilirsiniz. Tüm ikili dosyalar aynı öneke sahiptir!

    ![Bin klasörü ekran görüntüsü](./screenshots/39.png)  
    *Vurgulanan kısma bakın. `aarch64-none-linux-gnu-`, tüm ikili dosyalar için ortak önektir ve `CROSS_COMPILE` değişkeninin değeridir.*

4. **CLANG_TRIPLE=aarch64-linux-gnu-** → Clang'e tam olarak hangi hedef mimari, işletim sistemi ve ABI için derleme yapacağını söyler.

    - Çekirdek derleme sisteminin ARM64 Linux'a özgü özellikleri ve bayrakları etkinleştirebilmesini sağlar.
    - Bu, yolunuzda (path) tam olarak `aarch64-linux-gnu-` adında bir ikili dosya olmasını **gerektirmez** — Clang bunu dahili olarak bir hedef belirtimi olarak kullanır.
    - Ayrıca üçlü olarak `aarch64-none-linux-gnu-` da kullanabilirsiniz; satıcı alanı (`none`) genellikle Clang tarafından yoksayılır.

5. **senin_defconfigin ...** → Bunlar, derlemeye hangi çekirdek özelliklerinin, sürücülerin ve seçeneklerin dahil edileceğini tanımlayan yapılandırma dosyalarıdır (`defconfig`ler).

**Bu, Android çekirdeğini derlemek için `make` komutunun mutlak iskeletidir. Bu kodun herhangi bir parçasını çıkarmaya çalışmayın!**

---

- Şimdi, yukarıdaki komutu çalıştırdığınızda, derleme sistemi tüm `defconfig` dosyalarınızı okuyacak ve bunları `.config` adında tek bir dosyada birleştirecektir!

  ![Bin klasörü ekran görüntüsü](./screenshots/40.png)
  *Komutu çalıştırmadan **önceki** ekran görüntüsü*

  ![Bin klasörü ekran görüntüsü](./screenshots/41.png)
  *Komutu çalıştırdıktan **sonraki** ekran görüntüsü*

**Bu, nihai yapılandırmayı `.config` adında gizli bir dosyaya yazacaktır, bu dosya derleme sistemi tarafından çekirdeği derlemek için kullanılacaktır:**

  ![Bin klasörü ekran görüntüsü](./screenshots/42.png)

---

- Çekirdeği derlemeden önce, `.config` içeriğini GUI (Grafiksel Kullanıcı Arayüzü) yoluyla düzenlemek isterseniz, `menuconfig` aracını kullanabilirsiniz.  

- `menuconfig`'i başlatmak için, `.config` oluştururken kullandığınız komutun aynısını (yani `CC` ve `CROSS_COMPILE` kısımları) kullanın, ancak sonunda defconfig adları yerine şu şekilde `menuconfig` kullanın:

```bash
make \
  ARCH=arm64 \
  CC=clang \
  CROSS_COMPILE=aarch64-none-linux-gnu- \
  CLANG_TRIPLE=aarch64-none-linux-gnu- \
  menuconfig
```

  ![Bin klasörü ekran görüntüsü](./screenshots/43.png)  
  *Buna benzer bir şey açılacak. İhtiyaçlarınıza göre düzenlemekten çekinmeyin.*

**`menuconfig` içinde gezinmek için yön tuşlarını kullanın. Düzenlemeyi bitirdiğinizde, çekirdeği oluşturmaya devam etmek için `menuconfig`'den çıkın.**

**Not:** Özelleştirme kısmı burada tartışılmamaktadır; bu konu Yöntem 2'de ele alınmıştır. Bu sadece "Çekirdeği derleme"nin temelleridir.

---

- Şimdi, nihai yapılandırma dosyasını (`.config`) başarıyla oluşturduk ve gerekirse `menuconfig` kullanarak özelleştirdik.  

- Yapılacak tek şey çekirdeği derlemek!  

- Derlemek için, aynı başlangıca sahip ( `ARCH`, `CC` ve `CROSS_COMPILE` kısımları) komutu çalıştırın, ancak bu sefer **sonunda herhangi bir defconfig veya menuconfig belirtmeyin**. Şunun gibi:

```bash
make \
  ARCH=arm64 \
  CC=clang \
  CROSS_COMPILE=aarch64-none-linux-gnu- \
  CLANG_TRIPLE=aarch64-none-linux-gnu-
```

---

### 💡 Bu ne işe yarar:

Bu komut, derleme sistemine az önce oluşturduğunuz `.config` dosyasını kullanarak çekirdeği derlemeye hemen başlamasını söyler. `.config` dosyasındaki tüm ayarlar ve seçenekler şimdi derleme sürecine rehberlik edecektir.

---

**Yukarıdaki komutu çalıştırdığınızda, derleme sistemi aynı çekirdek kök dizininde çekirdeği derlemeye başlayacaktır:**

  ![Bin klasörü ekran görüntüsü](./screenshots/44.png)  

### Temel Eğitim yeterli! 

**Yapabileceğiniz en kolay ve en tembel yönteme atlayalım xD**
**`Yöntem 02`'de derlemeyi daha derinlemesine inceleyeceğiz!**

---

## 🟠 Yöntem 2: Bir derleme betiği ile.

### 01. Çekirdek Kaynağını indirdikten veya klonladıktan sonra, çekirdeğimizi derlemek için bir derleme betiğimiz (build script) olmalıdır.

- Bir derleme betiği oluşturmadan önce, çekirdeğimizi oluşturmak için kullanacağımız uyumlu derleyicileri belirlemeliyiz.

- Çekirdek sürümünüzü kontrol etmek için çekirdek kökü içinde ```make kernelversion``` komutunu çalıştırın.

<img src="./screenshots/5.png">

- Benim durumumda, çekirdek sürümü **5.4**, qualcomm yonga seti ile, bu da [qGKI](https://github.com/ravindu644/Android-Kernel-Tutorials#-understanding-non-gki--gki-kernels)'dır.

- **Çekirdek sürümünüz için doğru derleyiciyi seçme** hakkında tam bilgiyi [burada](./toolchains/) (based on my experience, btw).

- Derleme betiklerim her şeyi sizin için hallettiğinden **bu araç zincirlerinin hiçbirini manuel olarak indirmeniz gerekmediğini** unutmayın :)

- Sonra, [build_scripts](./build_scripts/) dizinine gidin, uygun betiği seçin, indirin ve çekirdeğinizin kök dizinine yerleştirin.

<img src="./screenshots/7.png">

<hr>

> [!CAUTION]
>
> Bu GKI derleme betikleri sadece kaynaktan çekirdek `Image` dosyasını derler. Şunları **İÇERMEYEBİLİR**:
> - OEM ağaç dışı sürücüler (örn. Samsung'un `sec_*`, EFUSE tetikleyicileri, TrustZone işleyicileri)
> - Yalnızca resmi OEM derleme sistemleri aracılığıyla oluşturulan satıcıya özgü modüller.
>
> Bu `Image` dosyasını önyükleyici kilidini açtıktan sonra **ilk özel ikili dosyanız** olarak flashlamak, cihazınızı **kalıcı olarak hard brick** yapabilir (kullanılamaz hale getirebilir) — özellikle **Samsung MediaTek GKI 2.0+** modellerinde.
>
> Neden? Çünkü eksik güvenlik sürücüleri düzgün EFUSE işlemesini engelleyebilir ve sistem flashlamanızı bir kurcalama ihlali olarak değerlendirip geri döndürülemez bir brick durumuna yol açabilir.
>
> Ben zaten bu şekilde bir telefonu brick yaptım — bu yüzden **bunu ciddiye alın.**
>
> Hala devam etmek ve özellikle Samsung MTK cihazları için *güvenli* ve *önyüklenebilir* bir GKI çekirdeği oluşturmayı öğrenmek istiyorsanız, **SM-A166P depoma** bakın:
>
> 👉 https://github.com/ravindu644/android_kernel_a166p
>
> **ÖZET:** **SATICI SÜRÜCÜLERİ OLMADAN GKI `Image` DOSYASINI TEK BAŞINA FLASHLAMAYIN — ÖZELLİKLE SAMSUNG MTK CİHAZLARINDA**

<hr>

### 02. Derleme betiğini düzenleyin:

**Derleme betiğini bir metin düzenleyicide açın ve şu değişiklikleri yapın:**

- `your_defconfig` kısmını `arch/arm64/configs` içinde bulunan mevcut defconfig dosyanızla değiştirin.

- GKI 2.0 çekirdeklerinde, bu normalde `gki_defconfig`'dir.

- Ancak her ihtimale karşı, `arch/arm64/configs` veya `arch/arm64/configs/vendor` dizinlerini kontrol ettiğinizden emin olun.

- Defconfig dosyanız `arch/arm64/configs` dizinindeyse, `your_defconfig` kısmını sadece defconfig dosyanızın adıyla değiştirin.

- Defconfig dosyanız `arch/arm64/configs/vendor` dizinindeyse, `your_defconfig` kısmını şu şekilde değiştirin:
  
  - `vendor/defconfig_dosyasinin_adi`
  - Örnek yama: [burada](./patches/005.edit-defconfig.patch)

  <img src="./screenshots/12.png">

**❗Cihazınız Samsung Exynos ise, çekirdeğin ayrılmış bir 'out' dizininde derlenmesini desteklemez. Bu yüzden, [derleme betiğinizi bu şekilde düzenleyin](./patches/001.nuke_out.patch)**  

---
#### ⚠️ [ÖNEMLİ] : *Cihazınız Samsung ise, genellikle "bazı" çekirdeklerde cihaza özgü bazı değişkenler kullanır.*

- **Örnek olarak,** Galaxy S23 FE çekirdek kaynak kodunda, `TARGET_SOC=s5e9925`, `PLATFORM_VERSION=12` ve `ANDROID_MAJOR_VERSION=s` adlı değişkenleri kullandıklarını görebiliriz.

- **Bu değişkenleri doğru şekilde dışa aktarmazsak (export),** benim durumumda çekirdek derlenemedi.

- Endişelenmeyin, bu gerekli değişkenleri genellikle `README_Kernel.txt` dosyalarında veya kendi `build_kernel.sh` dosyalarında belirtirler. 

  <img src="./screenshots/16.png">

**Bu tür değişkenleri derleme betiğimize düzgün bir şekilde entegre etmek için bu örnek yamaya bakın:** [burada](./patches/007.Define-OEM-Variables.patch)

**Not:** Çok fazla düşünmeyin, Platform ve Android sürümleri için 12 ve S gibi değerler kullansalar bile, daha yüksek bir Android sürümüne sahip olsanız bile aynısını kullanın.

---

🔴 **Cihazınız MediaTek yonga setine sahipse, genellikle RAW çekirdek `Image` önyüklemesini desteklemez. Bu nedenle, bunun yerine gzip ile sıkıştırılmış bir çekirdek `Image.gz` oluşturmalısınız.**

- [Bunun için gerekli yama burada](./patches/014.build_gzip_compressed_kernel.patch)

---

### 03. Makefile dosyasını düzenleyin.

- "Makefile" dosyanızda şu değişkenleri bulursanız: ```REAL_CC``` veya ```CFP_CC```, bunları "Makefile"dan kaldırın, ardından Makefile dosyanızda "wrapper"ı arayın. Bir Python dosyasıyla ilgili bir satır varsa, o satırın/fonksiyonun tamamını da kaldırın.

    - Wrapper'ı kaldırmanın örnek yaması: [buraya tıklayın](./patches/004.remove_gcc%20wrapper.patch)

<hr>

### 04. Şimdi, bu komutu kullanarak ```build_xxxx.sh``` dosyasına çalıştırma izni verin.
  ```
  chmod +x build_xxxx.sh
  ```
### 05. Son olarak, derleme betiğini şu komutu kullanarak çalıştırın:
  ```./build_xxxx.sh```

<img src="./screenshots/8.png">

- Betiği ilk kez çalıştırdığınızda, gerekli tüm bağımlılıkları yüklemeye başlayacak ve çekirdek sürümünüze bağlı olarak gerekli araç zincirlerini (toolchains) indirmeye başlayacaktır.

- İlk çalıştırmayı yarıda kesmediğinizden emin olun. Herhangi bir şekilde kesilirse, `toolchains` klasörünü "~/". dizininden silin ve tekrar deneyin: ```rm -rf ~/toolchains```

<img src="./screenshots/9.png">

### İlk çalıştırma tamamlandıktan sonra, çekirdek derlenmeye başlamalıdır, 

<img src="./screenshots/11.png">

### ve "menuconfig" görünmelidir.

<img src="./screenshots/10.png">

- **Ek notlar:**
    - `warning:` olarak görüntülenen her şeyi tamamen görmezden gelebilirsiniz
      - Örn: `warning: ignoring unsupported character '`
<hr>

## <span id="customizing-kernel-temporary-method">✅ Çekirdeği Özelleştirme (Geçici Yöntem)</span>
- *menuconfig* göründüğünde, içinde gezinebilir ve Çekirdeği gerektiği gibi grafiksel bir şekilde özelleştirebilirsiniz.  

- **Örnek olarak,** **Çekirdek adını özelleştirebilir, yeni sürücüleri etkinleştirebilir, yeni dosya sistemlerini etkinleştirebilir, güvenlik özelliklerini devre dışı bırakabilir** ve daha fazlasını yapabiliriz :)

#### *menuconfig* içinde klavyenizdeki ok tuşlarını (← → ↑ ↓) kullanarak gezinebilir ve etkinleştirmek için `y`, devre dışı bırakmak için `n` veya bir modül `<M>` olarak etkinleştirmek için `m` tuşuna basabilirsiniz.

### 1. Çekirdek adını değiştirme.

- Sanırım bunun için açıklamaya gerek yok:

    <img src="./screenshots/14.png" width="60%">

- Konum: `General setup  ---> Local version - append to kernel release`

<img src="./screenshots/gif/1.gif">

### 2. BTRFS desteğini etkinleştirme.

- Btrfs, güvenilirlik ve ölçeklenebilirlik için ideal olan, yazma sırasında kopyalama (copy-on-write), anlık görüntüler (snapshots) ve yerleşik RAID özelliklerine sahip modern bir Linux dosya sistemidir.

- Konum: `File systems  ---> < > Btrfs filesystem support`

<img src="./screenshots/gif/2.gif">

### 3. Daha fazla CPU Yöneticisi (Governor) Etkinleştirme

- **CPU yöneticileri, işlemcinin hızını nasıl ayarlayacağını kontrol eder.**
-  Performans odaklı yöneticiler (maksimum hız için "performance" gibi) veya pil tasarrufu sağlayanlar ("powersave" gibi) arasında seçim yapabilirsiniz.
-  Cihaz performans gerektiren görevleri yerine getirirken aşırı ısınırsa bunun SoC'nizin ömrünü etkileyebileceğini lütfen unutmayın.

**Daha fazla CPU Yöneticisi Etkinleştirme:**

- Konum: `CPU Power Management  ---> CPU Frequency scaling  ---> `

<img src="./screenshots/gif/3.gif">

**Varsayılan CPU Yöneticisini Değiştirme:**

- Konum: `CPU Power Management  ---> CPU Frequency scaling  ---> Default CPUFreq governor (performance)  --->`

<img src="./screenshots/gif/4.gif">

### 4. Daha fazla G/Ç (IO) Zamanlayıcısı Etkinleştirme

- **G/Ç zamanlayıcıları, sisteminizin depolamaya veri okuma ve yazma işlemlerini nasıl yöneteceğini kontrol eder.**
- Farklı zamanlayıcılar, ne yaptığını ne yaptığınıza bağlı olarak (oyun oynama, gezinme veya pil tasarrufu gibi) sisteminizi hızlandırabilir veya daha sorunsuz çalışmasına yardımcı olabilir.
- Konum: `IO Schedulers  --->`

<img src="./screenshots/15.png">

### menuconfig ile ilgili sorun, derleme betiğini her çalıştırdığınızda bunu yapmak zorunda olmanızdır.

- menuconfig kullanarak yaptığınız tüm değişiklikler `out` dizinindeki `.config` adlı geçici gizli bir dosyada kaydedilir.

  <img src="./screenshots/18.png">

- ve derleme betiğini her çalıştırdığınızda sıfırlanır.

  <img src="./screenshots/17.png">

- Yani, değişikliklerimizi kaydetmek için kalıcı bir yönteme ihtiyacımız var, değil mi?  

## <span id="customizing-kernel-permanent-method">✅ Çekirdeği Özelleştirme (Kalıcı Yöntem)</span>

- Bu yöntemde, **değişikliklerimizi saklamak için ayrı bir `custom.config` oluşturacağız** ve **bunu derleme betiğimize bağlayacağız.** 

- Bundan sonra, derleme betiğini çalıştırdığımızda, **önce `.config` dosyasını oluşturmak için OEM defconfig'inizi kullanacak, ardından `custom.config` dosyamızdaki değişiklikleri tekrar `.config` ile birleştirecektir.** 

**Temel bir fikir edinmek için bu örneklere bakın:** [yama](./patches/008.add-custom-defconfig-support.patch), [commit](https://github.com/ravindu644/android_kernel_m145f_common/commit/c427dbebed22c5bb314b4c94c711deffe671b14c)

---

### 🤓 `custom.config` dosyamıza nasıl değişiklik eklenir?

- İlk olarak, **etkinleştirmek** veya **devre dışı bırakmak** istediğiniz tam **çekirdek yapılandırma seçeneğini** bulmalıyız.

- Örnek **çekirdek yapılandırma seçeneği**: `CONFIG_XXXX=y`

  - `CONFIG_XXXX`: Çekirdek seçeneğinin veya özelliğinin adı **( `CONFIG_` ile başlamalıdır )**
  - `=y`: Bu "evet" anlamına gelir -> seçenek etkindir ve çekirdeğe dahil edilecektir.
  - `=n`: Bu "hayır" anlamına gelir -> seçenek devre dışıdır.

- **Çekirdek yapılandırma seçeneğinin** adını şu şekilde bulabilirsiniz:

  - Derleme betiğini çalıştırın ve `menuconfig` görünene kadar bekleyin.
  - Etkinleştirmek istediğiniz seçeneğe/özelliğe gidin.
  - Klavyenizde `shift + ?` tuşlarına basın, seçenek/özellik hakkında bir açıklama görünecektir.
  - menuconfig'in sol üst köşesinde **çekirdek yapılandırma seçeneğinin** adını göreceksiniz.

    <img src="./screenshots/19.png">

  - **O adı kopyalayın** ve etkinleştirmek veya devre dışı bırakmak için `custom.config` dosyanıza `=y` veya `=n` ile ekleyin.

    <img src="./screenshots/20.png">

## <span id="nuke-samsung-anti-root-protections">⁉️ Samsung'un anti-root korumaları nasıl kaldırılır?</span>

 - ### [Buraya taşındı](./samsung-rkp/)

## <span id="additional-patches">🟢 Ek Yamalar</span>

### 01. Wi-Fi, dokunmatik, ses vb. gibi bozuk sistem işlevlerini düzeltmek için.
> [!NOTE]
> Bunu atlatmak genellikle iyi bir uygulama değildir, çünkü bunun gibi bir şey **son çare** olarak kullanılır,
>
> açık kaynaklı linux sürücüsü bulunamadığında. (örn. Tescilli sürücüler)
>
> Ancak, yeni başlayanlar veya Yüklenebilir Çekirdek Modüllerini göndermek isteyen çekirdek geliştiricileri için **bu sorun değildir.**

---

  - Bazı cihazlarda, **özel bir çekirdek derlemek Wi-Fi, dokunmatik, ses gibi sistem düzeyindeki işlevleri bozabilir ve hatta sistemin açılmamasına neden olabilir.**

  - Bunun arkasındaki neden, cihazın linux'un kötü amaçlı çekirdek modüllerinin yüklenmesini önleyen önceden oluşturulmuş güvenlik özelliği `(symversioning, signature)` nedeniyle harici çekirdek modüllerini `(*.ko)` yükleyememesidir.

  - Bu sorunu düzeltmek için, çekirdeği bu modülleri yüklemeye zorlamak üzere [bu yamayı kullanın](./patches/010.Disable-CRC-Checks.patch).

  **Böyle bir sorununuz olmasa bile, bu yamayı kullanmak yine de iyi bir uygulamadır.**

  ---

### 02. Düzeltme: `There's an internal problem with your device.` (Cihazınızda dahili bir sorun var) hatası.

**Nedeni:**

  ```
Userspace reads /proc/config.gz and spits out an error message after boot
finishes when it doesn't like the kernel's configuration. In order to
preserve our freedom to customize the kernel however we'd like, show
userspace the stock defconfig so that it never complains about our
kernel configuration.
  ```

  *(Kullanıcı alanı /proc/config.gz dosyasını okur ve çekirdeğin yapılandırmasını beğenmediğinde önyükleme bittikten sonra bir hata mesajı verir. Çekirdeği istediğimiz gibi özelleştirme özgürlüğümüzü korumak için, kullanıcı alanına stok defconfig'i gösterin, böylece çekirdek yapılandırmamız hakkında asla şikayet etmez.)*

- Bu sorunu düzeltmek için, OEM'inizin Defconfig dosyasının bir kopyasını alın ve adını `stock_defconfig` olarak değiştirin.

  <img src="./screenshots/30.png">

- Ardından, Android'i defconfig'in değişmediğini düşünmesi için kandırmak üzere aşağıdaki yamayı kullanın:

  - [Yama](./patches/011.stock_defconfig.patch), [Commit](https://github.com/ravindu644/android_kernel_a047f_eur/commit/d306bd4c4c84a12be5235e31540f40fb9c1a1066)
    
## <span id="compiling-the-kernel">✅ Çekirdeği Derlemek</span>

- Çekirdeği istediğiniz gibi özelleştirdikten sonra, sadece **menuconfig'den çıkın**.  
- Çıktıktan sonra, çekirdek derlenmeye başlayacaktır!

<img src="./screenshots/gif/5.gif">

### 💡 Her şey böyle sorunsuz giderse, 

  <img src="./screenshots/21.png">

### derlenmiş çekirdek `Image` dosyasını çekirdek kökünüzdeki `build` klasörünün içinde bulacaksınız!

  <img src="./screenshots/22.png">

## <span id="fixing-known-compiling-issues">🟥 Bilinen derleme sorunlarını düzeltme</span>

- **Çekirdek derlemeniz sırasında herhangi bir hatayla karşılaşırsanız,** [düzeltmelere](./patches/) gidin ve sorununuzun orada belirtilip belirtilmediğine bakın.

**[Bilinen sorunlar ve düzeltmeleri hakkında bilgi edinmek için buraya tıklayın](./patches/README.md)**

## <span id="building-signed-boot-image">🟡 Derlenmiş Çekirdekten İmzalı Bir Önyükleme İmajı (Boot Image) Oluşturma</span>

- Android cihazlarda, **`kernel` imajı genellikle `boot` bölümünün içinde bulunur.**

  <img src="./screenshots/23.png">

- Yani, tek yapmamız gereken **stok ROM'dan boot imajını almak, paketini açmak, çekirdeğini bizim "oluşturduğumuz" ile değiştirmek, yeniden paketlemek, flashlamak** ve **keyfini çıkarmak :)**

**Paket açma ve yeniden paketleme işlemi için, Magisk'in yerleşik önyükleme imajı paket açıcısı ve yeniden paketleyicisi olan `magiskboot`'u kullanacağız!**

### 01. En son Magisk APK'sını indirme ve çıkarma

- En son Magisk APK'sını [GitHub sürümlerinden](https://github.com/topjohnwu/Magisk/releases/latest) indirin ve şu şekilde çıkarın:

  <img src="./screenshots/24.png">

### 02. Çıkarılan klasörden `magiskboot`'u alma & Sistem PATH'ine ekleme

- `magiskboot` ikili dosyası, `extracted_magisk_apk/lib/<arch>` klasörünün içinde `libmagiskboot.so` dosya adıyla bulunacaktır:

  <img src="./screenshots/26.png">

**Adını `magiskboot` olarak değiştirin ve şununla sistem PATH'inize yükleyin:**

  <img src="./screenshots/27.png">

Hızlı komutlar:

```bash
# libmagiskboot.so dosyasının adını magiskboot olarak değiştirme
mv libmagiskboot.so magiskboot 

# magiskboot'a çalıştırılabilir izinler verme
chmod +x magiskboot 

# magiskboot'u sistem PATH'ine yükleme
sudo cp magiskboot /usr/local/bin/
```

### 03. `boot.img` paketini açma

1. Stok ROM'unuzdan `boot` imajını çıkarın ve yeni bir klasörün içine yerleştirin

  <img src="./screenshots/28.png">

**✔️ Sadece Samsung notu:**

  - **Samsung cihazlarda,** bu imajlar genellikle `AP_XXXX.tar.md5` dosyasının içinde bulunur.

  - Tek yapmanız gereken `md5` uzantısını kaldırmak için `AP_XXXX.tar.md5` dosyasının adını `AP_XXXX.tar` olarak değiştirmek, `AP_XXXX.tar` dosyasını çıkarmak ve çıkarılan klasörden `boot.img.lz4` dosyasını almaktır.

  - Ardından, **aşağıdaki komutu kullanarak bu lz4 dosyasını açın** ve RAW `boot.img` dosyanızı elde edeceksiniz

    ```bash
lz4 boot.img.lz4
```  
    
    <img src="./screenshots/25.png">

2. Şimdi, `boot.img` paketini açmak için aşağıdaki komutu çalıştırın:

  ```bash
  magiskboot unpack boot.img
  ```

  <img src="./screenshots/45.png">

#### 🟠 Yukarıdaki ekran görüntüsünde görebileceğiniz gibi, paketi açılmış `boot.img`'nin orijinal `kernel`'i, boot.img'nin bulunduğu klasörle aynı klasörde bulunur.

**Not:** Yeniden paketleme işlemi için gerekli olduğundan orijinal boot.img dosyasını silmeyin.

### 03. `boot.img`'yi yeniden paketleme

- Şimdi, tek yapmamız gereken **orijinal `kernel`'i derlenmiş özel çekirdeğimizle değiştirmek.**

**Örnek:**

<img src="./screenshots/gif/6.gif">
<br>

**Ne yaptım?**

1. `out/arch/arm64/boot` veya `build` klasöründen derlenmiş `Image` dosyasını, `magiskboot` kullanarak `boot.img` dosyamızı açtığımız klasöre kopyaladım

2. Orijinal `kernel`'i sildim ve `Image` dosyasının adını `kernel` olarak değiştirdim 😎

3. Ardından aşağıdaki komutu kullanarak `boot.img`'yi yeniden paketledim:


```bash
magiskboot repack boot.img
```

  <img src="./screenshots/28.png">

### 🟨 Yeni boot imajımız, `new-boot.img` adıyla stok `boot.img` paketini açtığımız klasörün içinde yer alacaktır.

- `new-boot.img` dosyasını başka bir konuma kopyalayın ve adını `boot.img` olarak değiştirin

- Şimdi, tek yapmanız gereken **o `boot.img` dosyasını fastboot moduyla** veya **Download moduyla** (Samsung) flashlamaktır.

**✔️ Sadece Samsung notu:**  

- Aşağıdaki komutu kullanarak ODIN ile flashlanabilir bir `tar` dosyası oluşturabilirsiniz:  

  ```bash
  tar -cvf "Custom-Kernel.tar" boot.img
  ```

- Ardından, o `tar` dosyasını ODIN'in AP yuvasını kullanarak flashlayın :)

---

Yazan: [@ravindu644](https://t.me/ravindu) ve katkıda bulunanlar

**Telegram'a Katılın:** [@SamsungTweaks](https://t.me/SamsungTweaks)

---

```
