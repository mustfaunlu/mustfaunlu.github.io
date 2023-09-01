---
layout: default
title: Common patterns
grand_parent: App architecture
nav_order: 2
parent: Modularization
---
# Common modularization patterns

Tüm projelere uyan tek bir modülerleştirme stratejisi yoktur. Gradle'ın esnek yapısı nedeniyle bir projeyi nasıl organize edebileceğiniz konusunda çok az kısıtlama vardır. Bu sayfa, çok modüllü Android uygulamaları geliştirirken kullanabileceğiniz bazı genel kurallara ve yaygın eğilimlere genel bir bakış sunar.

{: .note }
Not: Bu sayfadaki öneriler ve en iyi pratikler, ölçeklenmelerini sağlamak, kaliteyi ve sağlamlığı artırmak ve test edilmelerini kolaylaştırmak için geniş bir uygulama yelpazesine uygulanabilir. Ancak, bunları kılavuz olarak ele almalı ve gerektiğinde gereksinimlerinize göre uyarlamalısınız.

[By layer or feature? Why not both?! Guide to Android app modularization](https://youtu.be/16SwTvzDO0A)

## High cohesion and low coupling principle
Modüler bir kod tabanını karakterize etmenin bir yolu da coupling ve cohesion özelliklerini kullanmaktır. Coupling, modüllerin birbirlerine ne derece bağlı olduğunu ölçer. Bu bağlamda cohesion, tek bir modülün öğelerinin işlevsel olarak nasıl ilişkili olduğunu ölçer. Genel bir kural olarak, düşük coupling ve yüksek cohesion için çaba göstermelisiniz:

- Düşük coupling, modüllerin birbirinden mümkün olduğunca bağımsız olması gerektiği anlamına gelir, böylece bir modülde yapılan değişikliklerin diğer modüller üzerinde sıfır veya minimum etkisi olur. Modüller diğer modüllerin iç işleyişi hakkında bilgi sahibi olmamalıdır.


- Yüksek cohesion, modüllerin bir sistem olarak hareket eden bir kod koleksiyonundan oluşması gerektiği anlamına gelir. Açıkça tanımlanmış sorumluluklara sahip olmalı ve belirli alan bilgisi sınırları içinde kalmalıdırlar. Örnek bir e-kitap uygulaması düşünün. Kitap ve ödeme ile ilgili kodları aynı modülde bir araya getirmek uygun olmayabilir çünkü bunlar iki farklı işlevsel alandır.


{: .note }
İpucu: İki modül büyük ölçüde birbirlerinin bilgisine dayanıyorsa, bu aslında tek bir sistem olarak hareket etmeleri gerektiğine dair iyi bir işaret olabilir. Tersine, bir modülün iki parçası birbiriyle sık sık etkileşime girmiyorsa, muhtemelen ayrı modüller olmalıdırlar.

## Types of modules
Modüllerinizi düzenleme şekliniz esas olarak uygulama mimarinize bağlıdır. Aşağıda, önerilen uygulama mimarimizi takip ederken uygulamanıza ekleyebileceğiniz bazı yaygın modül türleri yer almaktadır.

{: .note }
Not: Bu bölümde, [uygulama mimarisi kılavuzu](https://developer.android.com/topic/architecture)muzda özetlenen kavramlara aşina olduğunuz varsayılmaktadır.

### Data modules
Bir veri modülü genellikle bir veri repositorysi, veri kaynakları ve model sınıfları içerir. Bir veri modülünün üç temel sorumluluğu şunlardır:

1. Belirli bir domainin tüm verilerini ve iş mantığını kapsüllemek: Her bir veri modülü, belirli bir domaini temsil eden verilerin işlenmesinden sorumlu olmalıdır. Birbirleriyle ilişkili oldukları sürece birçok veri türünü işleyebilir.


2. Repository'yi harici bir API olarak göstermek: Bir veri modülünün genel API'si, verilerin uygulamanın geri kalanına gösterilmesinden sorumlu oldukları için bir repository olmalıdır.


3. Tüm uygulama implementasyonlarini ve veri kaynaklarını dışarıdan gizlemek: Veri kaynaklarına yalnızca aynı modüldeki veri repositoryleri tarafından erişilebilmelidir. Dışarıya karşı gizli kalmalıdırlar. Kotlin'in `private` veya `internal` görünürlük anahtar kelimesini kullanarak bunu zorunlu kılabilirsiniz.

![](https://developer.android.com/static/topic/modularization/images/2_data_modules.png)
Şekil 1. Örnek veri modülleri ve içerikleri.


### Feature modules
Özellik(Feature), bir uygulamanın işlevselliğinin genellikle bir ekrana veya kayıt ya da ödeme akışı gibi birbiriyle yakından ilişkili bir dizi ekrana karşılık gelen izole bir parçasıdır. Uygulamanızın bir bottom navigation bar'a ihtiyacı varsa, her bir hedefin bir feature olması muhtemeldir.

{: . note }
Anahtar Terim: "Feature modülü", Play Feature Delivery'de de kullanılan ve koşullu olarak sunulabilen veya isteğe bağlı olarak indirilebilen bir modülü tanımlayan bir terimdir. Ancak bu kılavuz bağlamında feature modülü, uygulamanızın işlevselliğinin farklı bir bölümünü kapsayan bir modüldür.

![](https://developer.android.com/static/topic/modularization/images/2_bottom_bar.png)

Şekil 2. Bu uygulamanın her sekmesi bir feature olarak tanımlanabilir

Featurelar, uygulamanızdaki ekranlarla veya hedeflerle ilişkilidir. Bu nedenle, mantıklarını ve state'lerini işlemek için ilişkili bir UI ve ViewModel'e sahip olmaları muhtemeldir. Tek bir Feature'ın tek bir view veya navigasyon hedefiyle sınırlı olması gerekmez. **Feature modülleri veri modüllerine bağlıdır**.

![](https://developer.android.com/static/topic/modularization/images/2_feature_modules.png)

Şekil 3. Örnek feature modülleri ve içerikleri.


### App modules
App modülleri uygulamaya bir giriş noktasıdır. Feature modüllerine bağlıdırlar ve genellikle root navigasyonu sağlarlar. Tek bir app modülü, [derleme varyantları](https://developer.android.com/studio/build/build-variants) sayesinde bir dizi farklı binary'ye derlenebilir.

![](https://developer.android.com/static/topic/modularization/images/2_demo_full_dep_graph.png)

Şekil 4. *Demo* ve *Full* ürün flavor modülleri bağımlılık grafiği.

Uygulamanız otomobil, giyim veya TV gibi birden fazla cihaz türünü hedefliyorsa, her biri için bir app modülü tanımlayın. Bu, platforma özgü bağımlılıkları ayırmaya yardımcı olur.

![](https://developer.android.com/static/topic/modularization/images/2_wear_dep_graph.png)

Şekil 5. Wear uygulaması bağımlılığı grafiği.

### Common modules
Common modüller olarak da bilinen ortak modüller, diğer modüllerin sıklıkla kullandığı kodları içerir. Fazlalığı azaltırlar ve bir uygulamanın mimarisinde belirli bir katmanı temsil etmezler. Aşağıda ortak modüllere örnekler verilmiştir:

- **UI modülü:** Uygulamanızda özel UI öğeleri veya ayrıntılı markalama kullanıyorsanız, tüm işlevlerin yeniden kullanılabilmesi için widget koleksiyonunuzu bir modülde toplamayı düşünmelisiniz. Bu, kullanıcı arayüzünüzün farklı özellikler arasında tutarlı olmasına yardımcı olabilir. Örneğin, temalarınız merkezileştirilirse, yeniden markalaşma gerçekleştiğinde acı verici bir refactor'dan kaçınabilirsiniz.


- **Analytics modülü:** Tracking genellikle yazılım mimarisi çok az dikkate alınarak iş gereksinimleri tarafından belirlenir. Analitik izleyiciler genellikle birbiriyle ilgisi olmayan birçok bileşende kullanılır. Sizin için de durum böyleyse, özel bir analitik modülüne sahip olmak iyi bir fikir olabilir.


- **Network modülü:** Birçok modül bir ağ bağlantısı gerektirdiğinde, bir http istemcisi sağlamaya adanmış bir modüle sahip olmayı düşünebilirsiniz. Özellikle istemciniz özel yapılandırma gerektirdiğinde kullanışlıdır.


- **Utility modül:** Yardımcı olarak da bilinen utility'ler, genellikle uygulama genelinde yeniden kullanılan küçük kod parçalarıdır. Yardımcı programlara örnek olarak test yardımcıları, para birimi biçimlendirme işlevi, e-posta doğrulayıcı veya özel bir operatör verilebilir.

### Test modules

Test modülleri yalnızca test amacıyla kullanılan Android modülleridir. Modüller, yalnızca testleri çalıştırmak için gerekli olan ve uygulamanın çalışma zamanı sırasında ihtiyaç duyulmayan test kodu, test kaynakları ve test bağımlılıkları içerir. Test modülleri, teste özel kodu ana uygulamadan ayırmak için oluşturulur ve böylece modül kodunun yönetimi ve bakımı daha kolay hale gelir.

#### Use cases for test modules
Aşağıdaki örnekler, test modüllerinin uygulanmasının özellikle faydalı olabileceği durumları göstermektedir:

- Paylaşılan test kodu: Projenizde birden fazla modül varsa ve test kodunun bir kısmı birden fazla modül için geçerliyse, kodu paylaşmak için bir test modülü oluşturabilirsiniz. Bu, tekrarları azaltmaya ve test kodunuzun bakımını kolaylaştırmaya yardımcı olabilir. Paylaşılan test kodu, özel doğrulamalar veya eşleştiriciler gibi yardımcı sınıflar veya işlevlerin yanı sıra simüle edilmiş JSON yanıtları gibi test verilerini de içerebilir.


- Daha Temiz Derleme Konfigürasyonları: Test modülleri, kendi build.gradle dosyalarına sahip olabildikleri için daha temiz derleme konfigürasyonları elde etmenizi sağlar. App modülünüzün build.gradle dosyasını yalnızca testlerle ilgili konfigürasyonlar ile doldurmak zorunda kalmazsınız.


- Entegrasyon Testleri: Test modülleri, kullanıcı arayüzü, iş mantığı, ağ istekleri ve veritabanı sorguları dahil olmak üzere uygulamanızın farklı bölümleri arasındaki etkileşimleri test etmek için kullanılan entegrasyon testlerini saklamak için kullanılabilir.


- Büyük ölçekli uygulamalar: Test modülleri özellikle karmaşık kod tabanlarına ve birden fazla modüle sahip büyük ölçekli uygulamalar için kullanışlıdır. Bu gibi durumlarda, test modülleri kod organizasyonunu ve sürdürülebilirliğini geliştirmeye yardımcı olabilir.

![](https://developer.android.com/static/topic/modularization/images/2_test_modules.png)

Şekil 6. Test modülleri, normalde birbirine bağımlı olacak modülleri izole etmek için kullanılabilir.

## Module to module communication
Modüller nadiren tamamen ayrı olarak bulunur ve genellikle diğer modüllere dayanır ve onlarla iletişim kurar. Modüller birlikte çalıştıklarında ve sık sık bilgi alışverişinde bulunduklarında bile bağlantıyı düşük tutmak önemlidir. Bazen iki modül arasında doğrudan iletişim, mimari kısıtlamalarda olduğu gibi arzu edilmez. Döngüsel bağımlılıklarda olduğu gibi bu imkansız da olabilir.

![](https://developer.android.com/static/topic/modularization/images/2_mediator.png)

Şekil 7. Döngüsel bağımlılıklar nedeniyle modüller arasında doğrudan, iki yönlü bir iletişim imkansızdır. Diğer iki bağımsız modül arasındaki veri akışını koordine etmek için bir [aracı](https://en.wikipedia.org/wiki/Mediator_pattern) modül gereklidir.

Bu sorunun üstesinden gelmek için diğer iki modül arasında aracılık yapan üçüncü bir modülünüz olabilir. Mediatör(Aracı) modül her iki modülden gelen mesajları dinleyebilir ve gerektiğinde iletebilir. Örnek uygulamamızda, olay farklı bir feature parçası olan ayrı bir ekrandan kaynaklansa bile ödeme ekranının hangi kitabın satın alınacağını bilmesi gerekir. Bu durumda mediatör, navigasyon grafiğinin sahibi olan modüldür (genellikle bir app modülü). Örnekte, Navigasyon bileşenini kullanarak ana sayfa featuredan ödeme feature'ına veri aktarmak için navigasyonu kullanıyoruz.

```kotlin
navController.navigate("checkout/$bookId")
```

Ödeme hedefi, kitap hakkında bilgi almak için kullandığı bir argüman olarak bir kitap id'si alır. Bir hedef feature'ın ViewModel'i içindeki navigasyon argümanlarını almak için saved state handle'ı kullanabilirsiniz.

```kotlin
class CheckoutViewModel(savedStateHandle: SavedStateHandle, …) : ViewModel() {

   val uiState: StateFlow<CheckoutUiState> =
      savedStateHandle.getStateFlow<String>("bookId", "").map { bookId ->
          // produce UI state calling bookRepository.getBook(bookId)
      }
}
```
Gezinme argümanları olarak nesneleri geçirmemelisiniz. Bunun yerine, featurelarin veri katmanından istenen kaynaklara erişmek ve bunları yüklemek için kullanabileceği basit id'ler kullanın. Bu şekilde, coupling'i düşük tutarsınız ve tek doğruluk kaynağı ilkesini ihlal etmezsiniz.

Aşağıdaki örnekte, her iki feature modülü de aynı veri modülüne bağlıdır. Bu, mediator modülünün iletmesi gereken veri miktarını en aza indirmeyi mümkün kılar ve modüller arasındaki coupling'i düşük tutar. Nesneleri aktarmak yerine, modüller primitif ID'leri değiş tokuş etmeli ve kaynakları paylaşılan bir veri modülünden yüklemelidir.

![](https://developer.android.com/static/topic/modularization/images/2_shared_data.png)

Şekil 8. Paylaşılan bir veri modülüne dayanan iki feature modülü.


## Dependency inversion
Bağımlılık tersine çevirme, kodunuzu soyutlamayı, somut bir uygulamadan ayrı olacak şekilde düzenlemenizdir.

- Soyutlama: Uygulamanızdaki bileşenlerin veya modüllerin birbirleriyle nasıl etkileşime gireceğini tanımlayan bir sözleşme. Soyutlama modülleri sisteminizin API'sini tanımlar ve arayüzler ve modeller içerir.


- Somut uygulama: Soyutlama modülüne bağlı olan ve bir soyutlamanın davranışını uygulayan modüller.

Soyutlama modülünde tanımlanan davranışa bağlı olan modüller, belirli uygulamalar yerine yalnızca soyutlamanın kendisine bağlı olmalıdır.


![](https://developer.android.com/static/topic/modularization/images/2_di_concept.png)

Şekil 9. Doğrudan düşük seviyeli modüllere bağlı yüksek seviyeli modüller yerine, yüksek seviyeli ve implementation modülleri abstraction modülüne bağlıdır.

### Example
Çalışmak için bir veritabanına ihtiyaç duyan bir feature modülü düşünün. Feature modülü, yerel bir Room veritabanı veya uzak bir Firestore örneği olsun, veritabanının nasıl uygulandığı ile ilgilenmez. Yalnızca uygulama verilerini depolaması ve okuması gerekir.

Bunu başarmak için feature modülü, belirli bir veritabanı implementasyonu yerine abstraction modülüne bağlıdır. Bu soyutlama, uygulamanın veritabanı API'sini tanımlar. Başka bir deyişle, veritabanıyla nasıl etkileşim kurulacağına ilişkin kuralları belirler. Bu, feature modülünün temel implementasyon detaylarını bilmesine gerek kalmadan herhangi bir veritabanını kullanmasına olanak tanır.

Somut implementasyon modülü, soyutlama modülünde tanımlanan API'lerin gerçek implementasyonunu sağlar. Bunu yapmak için, implementasyon modülü de soyutlama modülüne bağlıdır.


### Dependency injection
Şimdiye kadar feature modülünün implementation modülüne nasıl bağlandığını merak ediyor olabilirsiniz. Cevap [Dependency Injection](https://developer.android.com/training/dependency-injection)'dır. Feature modülü gerekli veritabanı instance'ını doğrudan oluşturmaz. Bunun yerine, hangi bağımlılıklara ihtiyaç duyduğunu belirtir. Bu bağımlılıklar daha sonra harici olarak, genellikle app modülünde sağlanır.

```groovy
releaseImplementation(project(":database:impl:firestore"))

debugImplementation(project(":database:impl:room"))

androidTestImplementation(project(":database:impl:mock"))
```

{: .note }
Not: Farklı derleme türleri için farklı bağımlılıklar tanımlayabilirsiniz. Örneğin, release derlemesi Firestore implementasyonunu kullanabilir, debug derlemesi yerel bir Room veritabanına güvenebilir ve instrumented testler bir mock implementasyonu kullanabilir.


### Benefits
API'lerinizi ve implementasyonlarını ayırmanın faydaları aşağıdaki gibidir:

- Değiştirilebilirlik: API ve implementasyon modüllerinin net bir şekilde ayrılmasıyla, aynı API için birden fazla implementasyon geliştirebilir ve API'yi kullanan kodu değiştirmeden bunlar arasında geçiş yapabilirsiniz. Bu, özellikle farklı bağlamlarda farklı yetenekler veya davranışlar sağlamak istediğiniz senaryolarda faydalı olabilir. Örneğin, test için sahte bir implementasyon ve production için gerçek bir implementasyon.


- Decoupling: Ayrıştırma, soyutlamaları kullanan modüllerin belirli bir teknolojiye bağlı olmadığı anlamına gelir. Veritabanınızı daha sonra Room'dan Firestore'a değiştirmeyi seçerseniz, değişiklikler yalnızca işi yapan belirli modülde ( implementasyon modülü) gerçekleşeceği ve veritabanınızın API'sini kullanan diğer modülleri etkilemeyeceği için daha kolay olacaktır.


- Test edilebilirlik: API'leri implementasyonlarından ayırmak testleri büyük ölçüde kolaylaştırabilir. API sözleşmelerine karşı test senaryoları yazabilirsiniz. Ayrıca, sahte implementasyonlar da dahil olmak üzere çeşitli senaryoları ve uç durumları test etmek için farklı implementasyonlar kullanabilirsiniz.


- Geliştirilmiş derleme performansı: Bir API'yi ve implementasyonunu farklı modüllere ayırdığınızda, implementasyon modülündeki değişiklikler derleme sistemini API modülüne bağlı olarak modülleri yeniden derlemeye zorlamaz. Bu, özellikle derleme sürelerinin önemli olabileceği büyük projelerde daha hızlı derleme süreleri ve daha fazla üretkenlik sağlar.


### When to seperate
Aşağıdaki durumlarda API'lerinizi implementasyonlarından ayırmanız faydalı olacaktır:

- Çeşitli yetenekler: Sisteminizin parçalarını birden fazla şekilde implemente edebiliyorsanız, net bir API farklı implementasyonların birbirinin yerine kullanılabilmesini sağlar. Örneğin, OpenGL veya Vulkan kullanan bir render sisteminiz ya da Play veya şirket içi faturalandırma API'niz ile çalışan bir faturalandırma sisteminiz olabilir.


- Birden fazla uygulama: Farklı platformlar için paylaşılan özelliklere sahip birden fazla uygulama geliştiriyorsanız, ortak API'ler tanımlayabilir ve Platform başına özel implementasyonlar geliştirin.


- Bağımsız ekipler: Ayrıştırma, farklı geliştiricilerin veya ekiplerin kod tabanının farklı bölümleri üzerinde aynı anda çalışmasına olanak tanır. Geliştiriciler API sözleşmelerini anlamaya ve bunları doğru şekilde kullanmaya odaklanmalıdır. Diğer modüllerin implementasyon detayları hakkında endişelenmelerine gerek yoktur.


- Büyük kod tabanı: Kod tabanı büyük veya karmaşık olduğunda, API'yi implementasyondan ayırmak kodu daha yönetilebilir hale getirir. Kod tabanını daha granüler, anlaşılabilir ve bakımı yapılabilir birimlere ayırmanızı sağlar.
### How to implement?
1. **Bir soyutlama modülü oluşturun:** Bu modül, feature'ınızın davranışını tanımlayan API'ler (arayüzler ve modeller) içermelidir.


2. **Implementasyon modülleri oluşturun:** Implementasyon modülleri API modülüne dayanmalı ve bir soyutlamanın davranışını uygulamalıdır.

![](https://developer.android.com/static/topic/modularization/images/2_api_impl.png)

Şekil 10. Implementasyon modülleri soyutlama modülüne bağlıdır.


3. **Üst düzey modülleri soyutlama modüllerine bağımlı hale getirin:** Doğrudan belirli bir implementasyona bağlı olmak yerine, modüllerinizi soyutlama modüllerine bağımlı hale getirin. Yüksek seviyeli modüllerin implementasyon detaylarını bilmesine gerek yoktur, sadece sözleşmeye (API) ihtiyaçları vardır.

![](https://developer.android.com/static/topic/modularization/images/2_api_impl_feature.png)

Şekil 11. Yüksek seviyeli modüller implementasyona değil soyutlamalara bağlıdır.

4. **Implementasyon modülü sağlayın:** Son olarak, bağımlılıklarınız için gerçek implementasyonu sağlamanız gerekir. Spesifik implementasyon proje kurulumunuza bağlıdır, ancak implementasyon modülü genellikle bunu yapmak için iyi bir yerdir. Implementasyonu sağlamak için, seçtiğiniz derleme varyantı veya bir test kaynak seti için bir bağımlılık olarak belirtin.

![](https://developer.android.com/static/topic/modularization/images/2_api_impl_app.png)

Şekil 12. App modülü asıl implementasyonu sağlar.

## General best practices
Başta da belirtildiği gibi, çok modüllü bir uygulama geliştirmenin tek bir doğru yolu yoktur. Tıpkı birçok yazılım mimarisi olduğu gibi, bir uygulamayı modüler hale getirmenin de birçok yolu vardır. Bununla birlikte, aşağıdaki genel öneriler kodunuzu daha okunabilir, sürdürülebilir ve test edilebilir hale getirmenize yardımcı olabilir.


### Keep your configuration consistent

Her modül konfigürasyon ek yükü getirir. Modüllerinizin sayısı belirli bir eşiğe ulaşırsa, tutarlı konfigürasyonu yönetmek bir zorluk haline gelir. Örneğin, modüllerin aynı sürümdeki bağımlılıkları kullanması önemlidir. Sadece bir bağımlılık sürümünü değiştirmek için çok sayıda modülü güncellemeniz gerekiyorsa, bu sadece bir çaba değil, aynı zamanda potansiyel hatalara da yer açar. Bu sorunu çözmek için, konfigürasyonunuzu merkezileştirmek üzere gradle araçlarından birini kullanabilirsiniz:

- [Version katalogları](https://docs.gradle.org/current/userguide/platforms.html), senkronizasyon sırasında Gradle tarafından oluşturulan bir tür güvenli bağımlılık listesidir. Tüm bağımlılıklarınızı bildirmek için merkezi bir yerdir ve bir projedeki tüm modüller tarafından kullanılabilir.

- Derleme lojiğini modüller arasında paylaşmak için [convention plugin](https://docs.gradle.org/current/samples/sample_convention_plugins.html)'leri kullanın.


### Expose as little as possible
Bir modülün public arayüzü minimal olmalı ve sadece temel unsurları açığa çıkarmalıdır. Dışarıya herhangi bir implementasyon detayı sızdırmamalıdır. Her şeyi mümkün olan en küçük ölçüde kapsamlandırın. Bildirimleri modül-private yapmak için Kotlin'in private veya internal görünürlük kapsamını kullanın. Modülünüzde bağımlılıkları bildirirken, api yerine implementasyonu tercih edin. İkincisi, transitive bağımlılıkları modülünüzün tüketicilerine sunar. Implementasyon kullanmak, yeniden oluşturulması gereken modül sayısını azalttığı için derleme süresini iyileştirebilir.


### Prefer Kotlin & Java modules
Android Studio'nun desteklediği üç temel modül türü vardır:

- App modülleri uygulamanıza bir giriş noktasıdır. Kaynak kodu, kaynaklar, varlıklar ve bir AndroidManifest.xml içerebilirler. Bir app modülünün çıktısı bir Android App Bundle (AAB) veya bir Android Application Package (APK)'dır.


- [Library modülleri](https://developer.android.com/studio/projects/android-library), app modülleri ile aynı içeriğe sahiptir. Diğer Android modülleri tarafından bağımlılık olarak kullanılırlar. Bir library modülünün çıktısı bir Android Arşividir (AAR), yapısal olarak app modülleriyle aynıdır ancak daha sonra diğer modüller tarafından bir bağımlılık olarak kullanılabilecek bir Android Arşivi (AAR) dosyasına derlenirler. Bir library modülü, aynı mantığı ve kaynakları birçok app modülünde kapsüllemeyi ve yeniden kullanmayı mümkün kılar.


- Kotlin ve Java libraryleri herhangi bir Android kaynağı, varlık veya manifesto dosyası içermez.


Android modülleri ek yük getirdiğinden, tercihen Kotlin veya Java türünü mümkün olduğunca kullanmak istersiniz.
