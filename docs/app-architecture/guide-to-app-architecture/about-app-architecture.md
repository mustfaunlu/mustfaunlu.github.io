---
layout: default
title: About app architecture
grand_parent: App architecture
nav_order: 1
parent: Guide to app architecture
---
## About app architecture
Bu kılavuz, sağlam, yüksek kaliteli uygulamalar oluşturmak için best practiceleri ve önerilen mimariyi kapsar.

<mark style="background-color: lightblue">Not: Bu sayfada Android Framework ile ilgili temel bilgilere sahip olduğunuz
varsayılmaktadır. Android uygulama geliştirme konusunda yeniyseniz, başlamak ve bu kılavuzda bahsedilen kavramlar
hakkında daha fazla bilgi edinmek
için [Android Basics kursu](https://developer.android.com/courses/android-basics-kotlin/course) na göz atın. </mark>

### Mobile app user experiences

Tipik bir Android
uygulaması, [activities](/docs/app-architecture/app-entry-points/activities/activities), [fragments](/docs/app-architecture/app-navigation/fragments/fragments), [services](/docs/core-topics/services/services), [content providers](/docs/core-topics/app-data-and-files),
and [broadcast receivers](/docs/core-topics/background-tasks/broadcasts/broadcasts) dahil olmak
üzere multiple app componentler içerir. Bu uygulama componentlerinin
çoğunu [app manifest](/docs/app-basics/app-manifest-file/app-manifest-file) dosyasinda bildirirsiniz.
Android işletim sistemi daha sonra uygulamanızı cihazın genel kullanıcı deneyimine nasıl entegre edeceğine karar vermek
için bu dosyayı kullanır. Tipik bir Android uygulamasının multiple component içerebileceği ve kullanıcıların genellikle
kısa bir süre içinde birden çok uygulamayla etkileşime girdiği göz önüne alındığında, uygulamaların farklı türde
kullanıcı odaklı iş akışlarına(user-driven workflows) ve tasklere uyum sağlaması gerekir.

Mobil cihazlarda kaynaklarin kısıtlı olduğunu unutmayın; bu nedenle, işletim sistemi herhangi bir zamanda yenilerine yer
açmak için bazı uygulama processlerini sonlandırabilir.

Bu ortamın koşulları göz önüne alındığında, uygulama componentlerinizin ayrı ayrı ve duzensiz olarak başlatılması
mümkündür ve işletim sistemi veya kullanıcı bunları herhangi bir zamanda sonlandirabilir. Bu olaylar sizin kontrolünüz
altında olmadığından, uygulama componentlerinizdeki herhangi bir uygulama verisini veya state’ini saklamamalı veya
bellekte tutmamalısınız ve uygulama componentleriniz birbirine bağimlı olmamalıdır.

### Common architectural principles

Uygulama verilerini ve state’ini tutumak için uygulama componentlerini kullanmamanız gerekiyorsa,peki uygulamanızı nasıl
tasarlamanız gerekir?
Android uygulamalarının boyutu büyüdükçe, uygulamanın ölçeklenmesini sağlayan, uygulamanın sağlamlığını artıran ve
uygulamanın test edilmesini kolaylaştıran bir mimari tanımlamak önemlidir.
Bir uygulama mimarisi, uygulamanın bölümleri arasındaki sınırları ve her bir bölümün sahip olması gereken sorumlulukları
tanımlar. Yukarıda belirtilen ihtiyaçları karşılamak için uygulama mimarinizi birkaç belirli ilkeyi takip edecek şekilde
tasarlamanız gerekir.

#### Separation of concerns

İzlenecek en önemli ilke, [separation of concers](https://en.wikipedia.org/wiki/Separation_of_concerns). Tüm kodunuzu
bir Activity veya Fragment’a yazmak yaygın bir hatadır. Bu UI tabanlı sınıflar yalnızca UI ve işletim sistemi
interactionlarini handle eden mantığı(logic) içermelidir. Bu sınıfları olabildiğince yalın tutarak, component yaşam
döngüsüyle(lifecycle) ilgili birçok sorunu önleyebilir ve bu sınıfların test edilebilirliğini kolaylastirabilirsiniz.

Aktivite ve Fragment implementasyonlari uzerinde bir sahipliginiz olmadığını unutmayın; bunlar yalnızca Android işletim
sistemi ile uygulamanız arasındaki sözleşmeyi temsil eden glue classlardir. İşletim sistemi, kullanıcı etkileşimlerine
bağlı olarak veya düşük bellek gibi sistem koşulları nedeniyle herhangi bir zamanda bunları yok edebilir. Tatmin edici
bir kullanıcı deneyimi ve daha yönetilebilir bir uygulama bakım deneyimi sağlamak için bunlara bağımlılığınızı en aza
indirmek en iyisidir.

#### Drive UI from data models

Bir diğer önemli ilke, kullanıcı arayüzünüzü(UI) data modellerinden, tercihen kalıcı(persistent) modellerden
olusturmaniz gerektiğidir. Data modelleri, bir uygulamanın verilerini temsil eder. Uygulamanızdaki UI öğelerinden ve
diğer componentlerden bağımsızdırlar. Bu, kullanıcı arayüzüne ve uygulama bileşeni yaşam döngüsüne bağlı olmadıkları,
ancak işletim sistemi uygulamanın sürecini bellekten kaldırmaya karar verdiğinde yine de yok edilecekleri anlamına
gelir.

Kalıcı modeller aşağıdaki nedenlerle idealdir:

* Android işletim sistemi, kaynakları boşaltmak için uygulamanızı yok ederse, kullanıcılarınız veri kaybetmez.
* Uygulamanız, ağ bağlantısının kesintili olduğu veya kullanılamadığı durumlarda çalışmaya devam eder.

Uygulama mimarinizi data model sınıflarına(viewmodel gibi) dayandırırsanız, uygulamanızı daha test edilebilir ve sağlam
hale getirirsiniz.

#### Single source of truth

Uygulamanızda yeni bir veri türü tanımlandığında,
ona [Tek Doğruluk Kaynağı (SSOT)](/docs/app-architecture/guide-to-app-architecture/about-app-architecture#single-source-of-truth)
atamanız gerekir. SSOT, bu verilerin sahibidir ve yalnızca SSOT onu modifiye edebilir veya değiştirebilir. Bunu başarmak
için SSOT, immutable bir tip kullanarak verileri ortaya çıkarır ve verileri değiştirmek için SSOT, diğer tiplerin
çağırabileceği fonksiyonlari ortaya çıkarır veya eventleri receive eder.

Bu model birden fazla fayda sağlar:

* Belirli bir veri türündeki tüm değişiklikleri tek bir yerde merkezileştirir.
* Diğer tiplerin kurcalamaması için verileri korur.
* Verilerdeki değişiklikleri daha izlenebilir hale getirir. Böylece buglarin fark edilmesi daha kolaydır.

Offline-first bir uygulamada, uygulama verilerinin doğruluğunun kaynağı(ssot) tipik olarak bir veritabanıdır. Diğer bazı
durumlarda, kaynak bir ViewModel veya hatta UI olabilir.

#### Unidirectional Data Flow

[Doğruluğun tek kaynağı ilkesi(single source of truth principle(ssot))]((/docs/app-architecture/guide-to-app-architecture/about-app-architecture#single-source-of-truth)),
Tek Yönlü Veri Akışı (UDF) modeliyle birlikte kılavuzlarımızda sıklıkla kullanılmaktadır. UDF'de state sadece bir yönde
akar. Event ise veri akışını ters yönde değiştirendir.

Android'de, state veya veriler genellikle hiyerarşinin yüksek scope türlerinden daha düşük scope türlere doğru akar.
Eventler, ilgili veri türü için SSOT'a ulaşana kadar genellikle daha düşük scope türlerden tetiklenir. Örneğin, uygulama
verileri genellikle veri kaynaklarından kullanıcı arayüzüne akar. Button basmaları gibi kullanıcı eventleri, uygulama
verilerinin değiştirildiği ve immutable bir türde gösterildiği UI'den SSOT'a akar.
Bu pattern, veri tutarlılığını daha iyi garanti eder, hatalara daha az eğilimlidir, hata ayıklaması(debug) daha kolaydır
ve SSOT patterinin tüm avantajlarını beraberinde getirir

### Recommended app architecture
Bu bölüm, önerilen best practiseleri izleyerek uygulamanızı nasıl yapılandıracağınızı gösterir.

<mark style="background-color: lightblue"> Not: Bu sayfada sunulan öneriler ve best practiseler, ölçeklenmelerine, kalite ve sağlamlığı artırmalarına ve test edilmelerini kolaylaştırmalarına olanak sağlamak için geniş bir uygulama yelpazesine uygulanabilir. Ancak, bunları kılavuz olarak ele almalı ve gerektiğinde gereksinimlerinize göre uyarlamalısınız.</mark>

Bahsedilen ortak mimari ilkeler göz önüne alındığında, her uygulamada en az iki katman olmalıdır:

*   Uygulama verilerini ekranda görüntüleyen UI layer.
*   Uygulamanızın iş mantığını(business logic) içeren ve uygulama verilerini ortaya çıkaran data layer. 

Kullanıcı arabirimi(UI) ve veri katmanları arasındaki etkileşimleri basitleştirmek ve yeniden kullanmak için domain layer adı verilen ek bir katman ekleyebilirsiniz.

![Recommended app architecture](/assets/images/app-architecture.png)

Tipik bir uygulama mimari diagrami bu sekilde gorunur.

<mark style = "background-color: lightblue">Not:Buradaki oklar katmanlar arasi bagimliliklari gostermektedir. Mesela domain layer, data layer a bagimlidir.</mark>
### Modern App Architecture
Bu Modern App Architecture, diğerlerinin yanı sıra aşağıdaki tekniklerin kullanılmasını teşvik eder:

* Reaktif ve katmanlı bir mimari.
* Uygulamanın tüm katmanlarında Tek Yönlü Veri Akışı (UDF).
* UI'nin karmaşıklığını yönetmek için state holder'ları olan bir UI layer'ı.
* Coroutine'ler ve flow'lar.
* Dependency injection best practiceleri.

Daha fazla bilgi için aşağıdaki bölümlere, içindekiler tablosundaki diğer Architecture sayfalarına ve en önemli best practice'lerin bir özetini içeren [öneriler sayfasına](/architecture-recommendatios) bakın.


### UI Layer
UI katmanının (veya presentation katmanının) rolü, uygulama verilerini ekranda görüntülemektir. Veriler, kullanıcı etkileşimi(user interaction) (bir butona basmak gibi) veya external input(network response gibi) nedeniyle değiştiğinde, UI değişiklikleri yansıtacak şekilde güncellenmelidir.
UI katmanı iki şeyden oluşur:

* Ekrandaki verileri handle eden UI elementleri. Bu öğeleri Views veya Jetpack Compose fonksiyonlarini kullanarak oluşturursunuz.
* Verileri tutan, kullanıcı arayüzüne(UI) sunan ve mantığı(logic) handle eden state holderlar  (ViewModel sınıfları gibi).

![UI layer](/assets/images/ui-layer.png)

Bu katman hakkında daha fazla bilgi edinmek için [UI layer sayfası](/docs/app-architecture/guide-to-app-architecture/ui-layer/ui-layer)na bakın.

### Data Layer
Bir uygulamanın veri katmanı, iş mantığını(business logic) içerir. İş mantığı, uygulamanıza değer veren şeydir; uygulamanızın verileri nasıl oluşturduğunu, depoladığını ve değiştirdiğini belirleyen kurallardan oluşur.
Veri katmanı, her biri sıfır ila birçok veri kaynağı içerebilen repositorylerden oluşur. Uygulamanızda handle edilen her farklı veri türü için bir repository sınıfı oluşturmalısınız. Örneğin, filmlerle ilgili veriler için bir MoviesRepository sınıfı veya ödemelerle ilgili veriler için bir PaymentsRepository sınıfı oluşturabilirsiniz.

![Data layer](/assets/images/data-layer.png)

Repository sınıfları aşağıdaki görevlerden sorumludur:
* Verileri uygulamanın geri kalanına göstermek.
* Verilerdeki değişiklikleri merkezileştirmek.
* Birden çok veri kaynağı arasındaki çakışmaları çözmek.
* Uygulamanın geri kalanından veri kaynaklarını soyutlamak.
* Business logic içermek.

Her veri kaynağı sınıfı(data source class), yalnızca tek bir veri kaynağıyla çalışma sorumluluğuna sahip olmalıdır ; bu veri kaynaklari dosya, ağ kaynağı(network source) veya yerel veritabanı(local database) olabilir. Veri kaynağı sınıfları, veri işlemleri için uygulama ile sistem arasındaki köprüdür.
Bu katman hakkında daha fazla bilgi edinmek için [data layer sayfası](/docs/app-architecture/guide-to-app-architecture/data-layer/data-layer)na bakın.

### Domain Layer
Domain katmanı(domain layer), kullanıcı arayüzü(UI layer) ve veri katmanları(data layer) arasında bulunan isteğe bağlı bir katmandır.

Domain katmanı, karmaşık business logic'in veya birden fazla ViewModel tarafından yeniden kullanılan basit business logic'in enkapsüle edilmesinden sorumludur. Bu katman isteğe bağlıdır çünkü tüm uygulamalar bu gereksinimlere sahip olmayacaktır. Yalnızca gerektiğinde, örneğin karmaşıklığı yönetmek veya yeniden kullanılabilirliği desteklemek için kullanmalısınız.

![Domain layer](/assets/images/domain-layer.png)

Bu katmandaki sınıflara yaygın olarak use case veya interactors denir. Her use case`in tek bir fonksiyonalite üzerinde sorumluluğu olmalıdır. Örneğin, birden fazla ViewModels, ekranda doğru mesajı görüntülemek için saat dilimlerine ihtiyac duyuyorsa, uygulamanız bir GetTimeZoneUseCase sınıfına sahip olabilir.
Bu katman hakkında daha fazla bilgi edinmek için [domain layer sayfası](/docs/app-architecture/guide-to-app-architecture/domain-layer)na bakın.

### Manage dependencies between components

Uygulamanızdaki sınıflar, düzgün çalışması için diğer sınıflara bağlıdır. Belirli bir sınıfın bağımlılıklarını toplamak için aşağıdaki design patternlerinden birini kullanabilirsiniz:

[Dependency Injection:](/docs/app-architecture/dependency-injection/dependency-injection)  siniflarin bagimliliklarini olusturmadan tanimlamalarina izin verir. Runtime’da , bu bagimliliklari saglamaktan baska bir sinif sorumludur.


[Service Locater:](https://en.wikipedia.org/wiki/Service_locator_pattern)  siniflarin bagimliliklarini olusturmak yerine elde edebilecekleri registry saglar.

Bu patternler, kodu çoğaltmadan veya karmaşıklık eklemeden bağımlılıkları yönetmek için clear pattern sağladıkları için kodunuzu ölçeklendirmenize olanak tanır. Ayrıca bu patternler, test ve production implementasyonlari arasında hızla geçiş yapmanızı sağlar.

Android official olarak dependency leri yonetmek icin [HILT](/docs/app-architecture/dependency-injection/dependency-injetion-with-hilt) librarysini onermektedir. Hilt, bağımlılık ağacında dolaşarak nesneleri otomatik olarak oluşturur, bağımlılıklara ilişkin derleme zamanı garantileri sağlar ve Android çerçeve sınıfları için bağımlılık containerlari oluşturur.

### General Best Practices
Programlama yaratıcı bir alandır ve Android uygulamaları oluşturmak da bir istisna değildir. Bir sorunu çözmenin birçok yolu vardır; verileri birden fazla activity veya fragment arasında iletebilir, uzaktaki verileri alıp offline mod için local olarak tutabilir veya basit olmayan uygulamaların karşılaştığı diğer yaygın senaryoları ele alabilirsiniz.

Aşağıdaki öneriler zorunlu olmamakla birlikte, çoğu durumda bunları uygulamak kod tabanınızı uzun vadede daha sağlam, test edilebilir ve sürdürülebilir hale getirir:

#### Don`t store data in app components.
Uygulamanizin activitileri, servisleri ve broadcast receiverlari gibi entry pointlerini veri kaynagi olarak belirlemekten kacinin. Veri kaynagi olarak belirlemek yerine o entry pointler ilgili veri alt kumelerini geri almak icin diger comoponentler ile  koordine olmalidir. Her uygulama componenti, kullanıcının cihazıyla etkileşimine ve sistemin genel mevcut durumuna bağlı olarak oldukça kısa ömürlüdür.

#### Reduce dependencies on Android classes.
Uygulama componentleriniz, Context veya Toast gibi Android framework SDK API'lerine dayanan tek sınıf olmalıdır. Uygulamanızdaki diğer sınıfları onlardan soyutlamak, test edilebilirliğe yardımcı olur ve uygulamanızdaki [coupling](https://en.wikipedia.org/wiki/Coupling_(computer_programming)) i azaltır.

#### Create well-defined boundaries(sinirlari) of responsibility between various(cesitli) modules in your app.
Örneğin, ağdan veri yükleyen kodu kod tabanınızdaki birden fazla sınıfa veya pakete yaymayın. Benzer şekilde, aynı sınıfta data caching ve data binding gibi birden fazla ilgisiz sorumluluk tanımlamayın. [Önerilen uygulama mimarisi](/architecture-recommendatios)ni takip etmek bu konuda size yardımcı olacaktır.

#### Modüllere mümkün olduğunca az maruz kalın.( Expose as little as possible from each module.)
Örneğin, bir modülden dahili bir implementasyon ayrıntısını ortaya cikaran bir kısayol oluşturmaya cezbedilmeyin. Kısa vadede biraz zaman kazanabilirsiniz, ancak code baseniz geliştikçe birçok kez teknik borçla karşılaşmanız olasıdır.

#### Focus on the unique core of your app so it stands out from other apps.
Ayni boilerplate(basmakalip) kodu tekrar tekrar yazarak tekerlegi yeniden icat etmeye gerek yoktur. Bunun yerine, zamanınızı ve enerjinizi uygulamanızı benzersiz kılan şeylere odaklayın ,Jetpack kutuphanelerinin ve önerilen diğer kutuphanelerin tekrar eden boilerplateleri handle etmesine izin verin.

#### Consider how to make each part of your app testable in isolation.(Uygulamanizin her parcasini nasil ayri ayri test edilebilir hale gelecegini dusunun )
Örneğin, ağdan veri almak için iyi tanımlanmış bir API'ye sahip olmak, bu verileri yerel bir veritabanında tutan modülün test edilmesini kolaylaştırır. Bunun yerine, bu iki modülün mantığını tek bir yerde mixlerseniz veya ağ kodunuzu tüm cade base e dağıtırsanız, etkili bir şekilde test etmek çok daha zor - imkansız değilse de - olur.

#### Tipler, eşzamanlılık(concurrency) ilkelerinden sorumludur.
Bir tip long-running blocking işi yapıyorsa, bu computation’u doğru thread e taşımaktan sorumlu olmalıdır. Bu belirli tip, yaptığı hesaplamanın tipini ve hangi threadde yürütülmesi gerektiğini bilir. Tipler main thread  için main safe olmalıdır, yani bloklanmadan main threadden safe call etmektir.

#### Mümkün olduğu kadar alakalı ve yeni verilerde ısrar edin.
Bu şekilde kullanıcılar, cihazları çevrimdışı moddayken bile uygulamanızın işlevselliğinin keyfini çıkarabilir. Tüm kullanıcılarınızın sürekli, yüksek hızlı bağlantıya sahip olmadığına, sahip olsalar bile kalabalık yerlerde kötü sinyal alabileceklerini unutmayın.

### Benefits of Architecture (Mimarinin faydalari)
Uygulamanızda iyi bir Mimarinin uygulanması, proje ve mühendislik ekiplerine birçok fayda sağlar:

* Uygulamanin genel gidisatini, sürdürülebilirliğini, kalitesini ve sağlamlığını artırır.
* Uygulamanın ölçeklenmesini sağlar. Daha fazla kişi ve daha fazla takim, minimum kod çakışmasıyla aynı code base e katkıda bulunabilir.
* Alışmaya yardımcı olur. Mimari projenize tutarlılık katarken, ekibin yeni üyeleri hızla hızlanabilir ve daha kısa sürede daha verimli olabilir.
* Test etmek daha kolaydır. İyi bir Mimari, genellikle test edilmesi daha kolay olan daha basit türleri teşvik eder.
* Buglar, iyi tanımlanmış süreçlerle metodik olarak araştırılabilir.

Mimariye yatırım yapmanın kullanıcılarınız üzerinde de doğrudan bir etkisi vardır. Daha istikrarlı bir uygulamadan ve daha üretken bir mühendislik ekibi sayesinde daha fazla özellikten faydalanırlar. Bununla birlikte, Mimari aynı zamanda öncelikli bir zaman yatırımı gerektirir. Bu zamanı şirketinizin geri kalanına gerekçelendirmenize yardımcı olması için, diğer şirketlerin uygulamalarında iyi bir mimariye sahip olduklarında başarı hikayelerini paylaştıkları [bu vaka çalışmaları](https://developer.android.com/quality)na bir göz atın.

### [Samples](https://developer.android.com/topic/architecture#samples)