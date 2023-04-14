## The Android App Bundle format

Android App Bundle, Google Play'e yüklediğiniz bir dosyadır (.aab dosya uzantılı).

App bundle'lar, uygulamanızın kodunu ve kaynaklarını şekil 1'de gösterildiği gibi modüller halinde düzenleyen imzalı
binary'lerdir. Her modülün kodu ve kaynakları, bir APK'da bulacağınıza benzer şekilde düzenlenir ve bu mantıklıdır çünkü
bu modüllerin her biri ayrı APK'lar olarak oluşturulabilir. Google Play daha sonra base APK, feature APK'ları,
configuration APK'ları ve ( split APK'ları desteklemeyen cihazlar için) multi-APK'lar gibi kullanıcılara sunulan çeşitli
APK'ları oluşturmak için app bundle'ı kullanır. `Drawable/`, `values/` ve `lib/` dizinleri gibi mavi renkli dizinler,
Google Play'in her modül için konfigürasyon APK'ları oluşturmak için kullandığı kod ve kaynakları temsil eder.

![App bundle format](https://developer.android.com/static/images/app-bundle/aab_format-2x.png)
Şekil 1. Android App Bundle. Bir base modül, iki feature modülü ve iki asset paketi içeren bir Android App Bundle'ın
içeriği.

{: .note }
Not: Her benzersiz uygulama veya applicationID için bir app bundle oluşturursunuz. Yani, tek bir uygulama projesinden
uygulamanızın birden çok sürümünü oluşturmak için product flavors kullanıyorsanız ve bu sürümlerin her biri benzersiz
bir applicationID kullanıyorsa, uygulamanızın her sürümü için ayrı bir app bundle oluşturmanız gerekir.

Aşağıdaki listede app bundle'ın bazı dosya ve dizinleri daha ayrıntılı olarak açıklanmaktadır:

- ***base/, feature1/ ve feature2/:*** Bu top-level dizinlerin her biri uygulamanızın farklı bir modülünü temsil eder.
  Uygulamanızın base modülü her zaman app bundle'ın `base` dizininde bulunur. Ancak her bir feature modülünün dizinine,
  modülün manifestosundaki `split` attribute'i tarafından belirtilen ad verilir. Daha fazla bilgi edinmek
  için [feature module manifest](https://developer.android.com/guide/playcore/feature-delivery#feature-module-manifest)
  hakkında bilgi edinin.

- ***asset_pack_1/ ve asset_pack_2/:*** Büyük, grafik gerektiren uygulamalar veya oyunlar için assetleri asset paketleri
  halinde modülerleştirebilirsiniz. Asset paketleri, büyük boyut sınırları nedeniyle oyunlar için idealdir. Her bir
  asset paketinin bir cihaza nasıl ve ne zaman indirileceğini üç teslim moduna göre özelleştirebilirsiniz: yükleme
  zamanı(install-time), hızlı takip(fast-follow) ve talep üzerine(on-demand). Tüm asset paketleri Google Play'de
  barındırılır ve buradan sunulur. App bundle'ınıza asset paketlerini nasıl ekleyeceğiniz hakkında daha fazla bilgi
  edinmek için [Play Asset Delivery](https://developer.android.com/guide/app-bundle/asset-delivery)'ye genel bakış
  bölümüne bakın.

- ***BUNDLE-METADATA/:*** Bu dizin, araçlar veya uygulama mağazaları için yararlı bilgiler içeren meta data dosyalarını
  içerir. Bu tür meta data dosyaları ProGuard eşlemelerini ve uygulamanızın DEX dosyalarının tam listesini içerebilir.
  Bu dizindeki dosyalar uygulamanızın APK'larında paketlenmez.

- ***Module Protocol Buffer (*.pb) dosyaları:*** Bu dosyalar, Google Play gibi uygulama mağazalarına her bir uygulama
  modülünün içeriğini açıklamaya yardımcı olan meta datalar sağlar. Örneğin, `BundleConfig.pb`, app bundle'ı oluşturmak
  için derleme araçlarının hangi sürümünün kullanıldığı gibi bundle'ın kendisi hakkında bilgi sağlar ve `native.pb`
  ve `resources.pb`, Google Play APK'ları farklı cihaz yapılandırmaları için optimize ederken yararlı olan her modüldeki
  kodu ve kaynakları açıklar.

- ***manifest/:*** APK'ların aksine, app bundle'lar her modülün AndroidManifest.xml dosyasını bu ayrı dizinde saklar.

- ***dex/:*** APK'ların aksine, app bundle'lar her modül için DEX dosyalarını bu ayrı dizinde saklar.

- ***res/, lib/ ve assets/:*** Bu dizinler tipik bir APK'daki dizinlerle aynıdır. App bundle'ınızı yüklediğinizde,
  Google Play bu dizinleri inceler ve dosya yollarını koruyarak yalnızca hedef cihaz yapılandırmasını karşılayan
  dosyaları paketler.

- ***root/:*** Bu dizin, daha sonra bu dizinin bulunduğu modülü içeren herhangi bir APK'nın root'una taşınacak dosyaları
  depolar. Örneğin, bir app bundle'ın `base/root/` dizini,
  uygulamanızın [Class.getResource()](https://developer.android.com/reference/java/lang/Class#getResource(java.lang.String))
  kullanarak yüklediği Java tabanlı kaynakları içerebilir. Bu dosyalar daha sonra uygulamanızın base APK'sının ve Google
  Play'in oluşturduğu her çoklu APK'nın root dizinine taşınır. Bu dizin içindeki path'ler de korunur. Yani, dizinler (ve
  alt dizinleri) de APK'nın root dizinine taşınır.

{: .warning }
Dikkat: Bu dizindeki içerik APK'nın root dizinindeki diğer dosya ve dizinlerle çakışırsa, Play Console yükleme sırasında
tüm app bundle'ı reddeder. Örneğin, bir `root/lib/` dizini ekleyemezsiniz çünkü bu, her APK'nın zaten içerdiği `lib`
diziniyle çakışır.

## Overviewof split APKs

Optimize edilmiş uygulamalar sunmanın temel bileşenlerinden biri Android 5.0 (API seviyesi 21) ve üzeri sürümlerde
bulunan split APK mekanizmasıdır. Split APK'lar normal APK'lara çok benzer; derlenmiş DEX bytecode, kaynaklar ve bir
Android manifestosu içerirler. Ancak, Android platformu birden fazla yüklü split APK'yı tek bir uygulama olarak
değerlendirebilir. Yani, ortak kod ve kaynaklara erişimi olan birden fazla split APK yükleyebilir ve cihazda yüklü tek
bir uygulama olarak görünebilirsiniz.

Split APK'ların avantajı, monolitik bir APK'yı (yani uygulamanızın desteklediği tüm özellikler ve cihaz yapılandırmaları
için kod ve kaynaklar içeren bir APK), kullanıcının cihazına gerektiği gibi yüklenen daha küçük, ayrı paketlere ayırma
yeteneğidir.

Örneğin, bir split APK yalnızca birkaç kullanıcınızın ihtiyaç duyduğu ek bir özelliğin kodunu ve kaynaklarını içerirken,
başka bir split APK yalnızca belirli bir dil veya ekran yoğunluğu için kaynakları içerebilir. Bu split APK'ların her
biri, kullanıcı talep ettiğinde veya cihaz tarafından gerekli görüldüğünde indirilir ve yüklenir.

Aşağıda, tam uygulama deneyiminizi oluşturmak için bir cihaza birlikte yüklenebilecek farklı APK türleri
açıklanmaktadır. Uygulama projenizi bu APK'ları destekleyecek şekilde nasıl yapılandıracağınızı bu sayfanın ilerleyen
bölümlerinde öğreneceksiniz.

- ***Base APK:*** Bu APK, diğer tüm split APK'ların erişebileceği kod ve kaynakları içerir ve uygulamanız için temel
  işlevselliği sağlar. Bir kullanıcı uygulamanızı indirmek istediğinde, ilk olarak bu APK indirilir ve yüklenir. Bunun
  nedeni, yalnızca base APK'nın manifestosunun uygulamanızın servisleri, content providerlari, izinleri, platform sürümü
  gereksinimleri ve sistem özelliklerine olan bağımlılıklarının tam bir bildirimini içermesidir. Google Play,
  uygulamanız için base APK'yı projenizin app (veya base) modülünden oluşturur. Uygulamanızın ilk indirme boyutunu
  küçültmekle ilgileniyorsanız, bu modüle dahil edilen tüm kod ve kaynakların uygulamanızın base APK'sına dahil
  edildiğini aklınızda bulundurmanız önemlidir.

- ***Configuration APKs:*** Bu APK'ların her biri belirli bir ekran yoğunluğu, CPU mimarisi veya dil için yerel
  kütüphaneler ve kaynaklar içerir. Bir kullanıcı uygulamanızı indirdiğinde, cihazı yalnızca kendi cihazını hedefleyen
  configuration APK'ları indirir ve yükler. Her configuration APK, bir base APK veya feature module APK'nın
  bağımlılığıdır. Yani, kod ve kaynak sağladıkları APK ile birlikte indirilir ve yüklenirler. Base ve feature
  modüllerinin aksine, configuration APK'lar için ayrı bir modül oluşturmazsınız. Base ve feature modülleriniz için
  alternatif, [configuration'a özgü kaynakları düzenlemek](https://developer.android.com/guide/topics/resources/providing-resources#AlternativeResources)
  üzere standart pratikler kullanırsanız, Google Play sizin için configuration APK'ları otomatik olarak oluşturur.

- ***Feature module APKs:*** Bu APK'ların her biri, feature modülleri kullanarak modüler hale getirdiğiniz uygulamanızın
  bir özelliği için kod ve kaynaklar içerir. Daha sonra bu özelliğin bir cihaza nasıl ve ne zaman indirileceğini
  özelleştirebilirsiniz. Örneğin, [Play Core Library](https://developer.android.com/guide/app-bundle/playcore)
  kullanılarak, kullanıcıya ek işlevsellik sağlamak için base APK cihaza yüklendikten sonra özellikler isteğe bağlı
  olarak yüklenebilir. Fotoğraf çekme ve gönderme özelliğini yalnızca kullanıcı bu işlevi kullanmayı talep ettiğinde
  indirip yükleyen bir sohbet uygulaması düşünün. Feature modülleri yükleme sırasında mevcut olmayabileceğinden, tüm
  ortak kod ve kaynakları base APK'ya dahil etmelisiniz. Yani, feature modülünüz yükleme sırasında yalnızca base APK'nın
  kod ve kaynaklarının mevcut olduğunu varsaymalıdır. Google Play, projenizin feature modüllerinden uygulamanız için
  feature modül APK'ları oluşturur.

Üç feature modülüne sahip ve birden fazla cihaz yapılandırmasını destekleyen bir uygulama düşünün. Aşağıdaki Şekil 1,
uygulamanın çeşitli APK'ları için bağımlılık ağacının nasıl görünebileceğini göstermektedir. Base APK'nın ağacın başını
oluşturduğunu ve diğer tüm APK'ların base APK'ya bağlı olduğunu unutmayın. (Bu APK'lar için modüllerin bir Android App
Bundle'da nasıl temsil edildiğini merak ediyorsanız, Android App Bundle formatına bakın)

![Dependency Tree](https://developer.android.com/static/images/app-bundle/apk_splits_tree-2x.png)
Şekil 1. Split APK'lar kullanılarak sunulan bir uygulama için bağımlılık ağacı

Unutmayın, bu APK'ları kendiniz oluşturmanıza gerek yoktur; Google Play, Android Studio ile oluşturduğunuz tek bir
imzalı app bundle kullanarak bunu sizin için yapar. App Bundle formatı ve nasıl oluşturulacağı hakkında daha fazla bilgi
edinmek
için [Android App Bundle'ları oluşturma, dağıtma ve yükleme bölümü](https://developer.android.com/guide/app-bundle/build)
ne gidin.

### Devices running Android 4.4 (API level 19) and lower

Android 4.4 (API düzeyi 19) ve daha düşük sürümleri çalıştıran cihazlar split APK'ların indirilmesini ve yüklenmesini
desteklemediğinden, Google Play bunun yerine bu cihazlara multi-APK adı verilen ve cihazın yapılandırması için optimize
edilmiş tek bir APK sunar. Yani, multi-APK'ler tam uygulama deneyiminizi temsil eder ancak diğer ekran yoğunlukları ve
CPU mimarileri için olanlar gibi gereksiz kod ve kaynakları içermez.

Bununla birlikte, uygulamanızın desteklediği tüm diller için kaynaklar içerirler. Bu, örneğin kullanıcıların farklı bir
multi-APK indirmek zorunda kalmadan uygulamanızın tercih ettiği dil ayarını değiştirmesine olanak tanır.

Multi-APK'ler, feature modüllerini daha sonra isteğe bağlı olarak indirme özelliğine sahip değildir. Bu APK'ya bir
feature modülü eklemek
için, [feature modülünü oluştururken](https://developer.android.com/studio/projects/dynamic-delivery#create_dynamic_feature)
Talep Üzerine(On-demand) özelliğini devre dışı bırakmanız
veya Fusing özelliğini etkinleştirmeniz gerekir.

App Bundle'larda uygulamanızın desteklediği her cihaz yapılandırması için APK oluşturmanız, imzalamanız, yüklemeniz ve
yönetmeniz gerekmediğini unutmayın. Yine de tüm uygulamanız için tek bir app bundle oluşturup yüklüyorsunuz ve gerisini
Google Play sizin için hallediyor. Dolayısıyla, Android 4.4 veya daha düşük sürümleri çalıştıran cihazları desteklemeyi
planlasanız da planlamasanız da, Google Play hem siz hem de kullanıcılarınız için esnek bir hizmet mekanizması sağlar.

### User language changes

App bundle'lar ile cihazlar yalnızca uygulamanızı çalıştırmak için ihtiyaç duydukları kodu ve kaynakları indirir.
Dolayısıyla, dil kaynakları için, kullanıcının cihazı yalnızca cihaz ayarlarında seçili olan bir veya daha fazla dille
eşleşen uygulamanızın dil kaynaklarını indirir.

Bir kullanıcı cihaz ayarlarında dilini değiştirdiğinde, uygulamanın yeni dilde görüntülenebilmesi için Google Play'in
bazı ek split APK'ları indirmesi ve yüklemesi gerekebilir.

Google Play, geçişten hemen sonra ek dilleri indirmeye çalışır. Kullanıcı cihazı çevrimdışıysa, indirme başarısız olursa
veya kaynaklar çok büyükse Google Play, cihaz koşulları daha uygun olduğunda arka planda indirmeyi tekrar dener. Android
9.0 (API seviyesi 28) veya daha düşük bir cihazda çalışırken, yeni language split APK'ların yüklenmesi sırasında
uygulamanız ön plandaysa, uygulama öldürülür.

Uygulamanız tüm dillerin cihazda her zaman kullanılabilir olmasını gerektiriyorsa, [derleme yapılandırmanızda language
split'i devre dışı](https://developer.android.com/guide/app-bundle/configure-base#disable_config_apks) bırakabilirsiniz.

Uygulamanız cihaz ayarlarında seçilen kullanıcı dillerinden bağımsız olarak ek dillerin indirilmesini gerektiriyorsa (
örneğin uygulama içi bir dil seçici uygulamak için), [bunları isteğe bağlı olarak indirmek için](https://developer.android.com/guide/playcore/feature-delivery/on-demand#lang_resources) Play Core kütüphanesini
kullanabilirsiniz.