---
layout: default
title: About the data layer
grand_parent: Guide to app architecture
nav_order: 1
parent: Data layer
---

# Data Layer

## About the data layer

UI katmanı UI ile ilgili state ve UI logic'i içerirken, data katmanı uygulama verilerini ve business logic'i içerir.
Business logic (iş mantığı) uygulamanıza değer katan şeydir; uygulama verilerinin nasıl oluşturulması, saklanması ve
değiştirilmesi gerektiğini belirleyen gerçek dünya iş kurallarından oluşur.

Separation of concerns ilkesinin bu sekilde olması, veri katmanının birden fazla ekranda kullanılmasına, uygulamanın
farklı bölümleri arasında bilgi paylaşılmasına ve birim testi için iş mantığının kullanıcı arayüzünün dışında yeniden
üretilmesine olanak tanır. Data katmanının faydaları hakkında daha fazla bilgi
için [Architecture Overview](/docs/app-architecture/guide-to-app-architecture/guide-to-app-architecture) sayfasına göz
atın.

<mark style="background-color:lightblue">ot: Bu sayfada yer alan öneriler ve best practiceler, ölçeklenmelerini
sağlamak, kaliteyi ve sağlamlığı artırmak ve test edilmelerini kolaylaştırmak için geniş bir uygulama yelpazesine
uygulanabilir. Ancak, bunları kılavuz olarak ele almalı ve gerektiğinde gereksinimlerinize göre uyarlamalısınız.


[Architecture: The data layer - MAD Skills](https://youtu.be/r5AseKQh2ZE)

### Data layer architecture

Data katmanı, her biri sıfır ila çok sayıda veri kaynağı içerebilen repository'lerden oluşur. Uygulamanızda işlediğiniz
her farklı veri türü için bir repository sınıfı oluşturmalısınız. Örneğin, filmlerle ilgili veriler için bir
MoviesRepository sınıfı veya ödemelerle ilgili veriler için bir PaymentsRepository sınıfı oluşturabilirsiniz.
![Data layer architecture](/assets/images/data-layer.png)

Repository sınıfları aşağıdaki görevlerden sorumludur:

* Verilerin uygulamanın geri kalanına sunulması.
* Verilerdeki değişiklikleri merkezileştirme.
* Birden fazla veri kaynağı arasındaki çakışmaları çözme.
* Uygulamanın geri kalanından veri kaynaklarını soyutlama.
* Business logic'i içermek.

Her veri kaynağı sınıfı(data source class) , bir dosya, bir network kaynağı veya local veritabanı olabilen sadece tek
bir veri kaynağı ile çalışma sorumluluğuna sahip olmalıdır. Veri kaynağı sınıfları, veri işlemleri için uygulama ile
sistem arasındaki köprüdür.

Hiyerarşideki diğer katmanlar veri kaynaklarına asla doğrudan erişmemelidir; data katmanına entry pointler her zaman
repository sınıflarıdır. State holder
sınıfları ([UI katmanı](/docs/app-architecture/guide-to-app-architecture/ui-layer/about-the-ui-layer) kılavuzuna bakın)
veya use case sınıfları ([domain katmanı](/docs/app-architecture/guide-to-app-architecture/domain-layer.md) kılavuzuna
bakın) hiçbir zaman doğrudan bir bağımlılık olarak bir veri kaynağına sahip olmamalıdır. Repository sınıflarının entry
point olarak kullanılması, mimarinin farklı katmanlarının bağımsız olarak ölçeklenebilmesini sağlar.
Bu katman tarafından açığa çıkarılan veriler immutable olmalıdır, böylece diğer sınıflar tarafından kurcalanamaz, bu da
değerlerini tutarsız bir duruma sokma riski taşır. Immutable veriler birden fazla thread tarafından da güvenli bir
şekilde işlenebilir. Daha fazla ayrıntı için thread bölümüne bakın.

[Dependency injection best practices](/docs/app-architecture/dependency-injection/about-dependency-injection)'i takiben, repository
veri kaynaklarını constructor'ında dependency olarak alır:

```kotlin
class ExampleRepository(
        private val exampleRemoteDataSource: ExampleRemoteDataSource, // network
        private val exampleLocalDataSource: ExampleLocalDataSource // database
) { /* ... */ }
```

{: .note}
Not: Genellikle, bir repository yalnızca tek bir veri kaynağı içerdiğinde ve
diğer repository'lere bağlı olmadığında, geliştiriciler repository'lerin ve veri kaynaklarının sorumluluklarını
repository sınıfında birleştirir. Bunu yaparsanız, uygulamanızın sonraki bir sürümünde repository'nin başka bir
kaynaktan gelen verileri işlemesi gerekiyorsa fonksiyonları bölmeyi unutmayın.


### Expose APIs

Data katmanındaki sınıflar genellikle tek seferlik Create, Read, Update ve Delete (CRUD) çağrıları gerçekleştirmek veya
zaman içindeki veri değişikliklerinden haberdar olmak için fonksiyonları kullanıma sunar. Data katmanı bu durumların her
biri için aşağıdakileri sağlamalıdır:

* Tek seferlik işlemler: Data katmanı Kotlin'de suspend fonksiyonlarını kullanıma sunmalıdır; ve Java programlama dili
  için, data katmanı işlemin sonucunu bildirmek için bir callback sağlayan fonksiyonları veya RxJava Single, Maybe veya
  Completable tiplerini kullanıma sunmalıdır.
* Zaman içindeki veri değişikliklerinden haberdar olmak için: Data katmanı
  Kotlin'de [flow](https://developer.android.com/kotlin/flow)'ları kullanıma sunmalıdır; ve Java programlama dili için
  data katmanı yeni veriyi veya RxJava Observable veya Flowable türünü yayınlayan bir callback'i kullanıma sunmalıdır.

```kotlin
class ExampleRepository(
        private val exampleRemoteDataSource: ExampleRemoteDataSource, // network
        private val exampleLocalDataSource: ExampleLocalDataSource // database
) {

    val data: Flow<Example> = ...

    suspend fun modifyData(example: Example) {
        ...
    }
}
```

### Naming conventions in this guide

Bu kılavuzda, repository sınıfları sorumlu oldukları verilere göre adlandırılmıştır. Kurallar aşağıdaki gibidir:

veri türü + Repository.

Örneğin: NewsRepository, MoviesRepository veya PaymentsRepository.

Veri kaynağı sınıfları, sorumlu oldukları verilerden ve kullandıkları kaynaktan sonra adlandırılır. Kurallar aşağıdaki
gibidir:

veri türü + kaynak türü + DataSource.

Veri türü için, uygulamalar değişebileceğinden daha genel olması için Remote veya Local kullanın. Örneğin:
NewsRemoteDataSource veya NewsLocalDataSource. Kaynağın önemli olması durumunda daha spesifik olmak için kaynağın türünü
kullanın. Örneğin: NewsNetworkDataSource veya NewsDiskDataSource.

Veri kaynağını bir uygulama detayına göre adlandırmayın (örneğin UserSharedPreferencesDataSource) çünkü bu veri
kaynağını kullanan repository'ler verilerin nasıl kaydedildiğini bilmemelidir. Bu kurala uyarsanız, veri kaynağının
uygulamasını, bu kaynağı çağıran katmanı etkilemeden değiştirebilirsiniz (
örneğin, [SharedPreferences](https://developer.android.com/training/data-storage/shared-preferences)'
tan [DataStore](https://developer.android.com/topic/libraries/architecture/datastore)'a geçiş).

<mark style="background-color:lightblue">Not: Bir veri kaynağının yeni bir implementasyonuna geçerken, veri kaynağı için
bir interface oluşturabilir ve veri kaynağının iki implementasyonuna sahip olabilirsiniz: biri eski backing teknolojisi
için, diğeri de yenisi için. Bu durumda, veri kaynağı sınıf adları için teknolojinin adını kullanmanızda bir sakınca
yoktur (bu bir uygulama ayrıntısı olsa bile) çünkü repository veri kaynağı sınıflarının kendisini değil yalnızca
interface'i görür. Taşıma işlemini tamamladığınızda, yeni sınıfı, adında uygulama ayrıntısı içermeyecek şekilde yeniden
adlandırabilirsiniz.


### Multiple levels of repositories

Daha karmaşık iş gereksinimlerini içeren bazı durumlarda, bir repository'nin diğer repository'lere bağımlı olması
gerekebilir. Bunun nedeni, ilgili verilerin birden fazla veri kaynağından toplanması veya sorumluluğun başka bir
repository sınıfında encapsulated edilmesi gerekliligi olabilir.
Örneğin, kullanıcı kimlik doğrulama verilerini işleyen bir repository, UserRepository, gereksinimlerini yerine getirmek
için LoginRepository ve RegistrationRepository gibi diğer repository'lere bağımlı olabilir.

![Dependency graph of a repository that depends on other repositories.](/assets/images/data-layer-img1.png)

{: .note}
Not: Geleneksel olarak, bazı geliştiriciler diğer repository sınıflarına bağlı
olan repository sınıflarını manager olarak adlandırırlar; örneğin UserRepository yerine UserManager. İsterseniz bu
adlandırma kuralını kullanabilirsiniz.


### Source of truth

Her repository'nin tek bir doğruluk kaynağı(single source of truth) tanımlaması önemlidir. Doğruluk kaynağı her zaman
tutarlı, doğru ve güncel veriler içerir. Aslında, repository'den açığa çıkan veriler her zaman doğrudan doğruluk
kaynağından gelen veriler olmalıdır.

Doğruluk kaynağı bir veri kaynağı (örneğin veri tabanı) ya da repository'nin içerebileceği bir in-memory cache olabilir.
Repository'ler farklı veri kaynaklarını birleştirir ve veri kaynakları arasındaki olası çakışmaları çözerek tek doğruluk
kaynağını düzenli olarak veya bir user input event nedeniyle günceller.

Uygulamanızdaki farklı veri repositoryleri farklı doğruluk kaynaklarına sahip olabilir. Örneğin, LoginRepository sınıfı
doğruluk kaynağı olarak cache'ini kullanabilir ve PaymentsRepository sınıfı network veri kaynağını kullanabilir.

Offline-first desteği sağlamak için, veritabanı gibi local bir veri kaynağı önerilen doğruluk kaynağıdır.

### Threading

Veri kaynaklarını ve repository'leri çağırmak, main thread'den çağırmak için main-safe/güvenli olmalıdır. Bu sınıflar,
uzun süreli bloklama operasyonları gerçekleştirirken logiclerinin yürütülmesini uygun thread'e taşımaktan sorumludur.
Örneğin, bir veri kaynağının bir dosyadan okuma yapması ya da bir veri repository'sinin büyük bir liste üzerinde yüklü
filtreleme yapması main-safe olmalıdır.

Çoğu veri kaynağının Room, Retrofit veya Ktor tarafından sağlanan suspend metot çağrıları gibi main-safe API'leri zaten
sağladığını unutmayın. Repositoryniz, kullanılabilir olduklarında bu API'lerden yararlanabilir.

Threading hakkında daha fazla bilgi edinmek için background processing kılavuzuna bakın. Kotlin kullanıcıları için
coroutine'ler önerilen seçenektir. Java programlama dili için önerilen seçenekler için Android task'larını background
thread'lerde çalıştırma bölümüne bakın.

### Lifecycle

Data katmanındaki sınıfların instance'ları, bir garbage collection root'undan (genellikle uygulamanızdaki diğer
nesnelerden referans alınarak) erişilebilir oldukları sürece bellekte kalırlar.

Bir sınıf in-memory veriler içeriyorsa (örneğin bir cache), bu sınıfın aynı instance'ını belirli bir süre için yeniden
kullanmak isteyebilirsiniz. Bu, sınıf instance'ının yaşam döngüsü olarak da adlandırılır.

Sınıfın sorumluluğu tüm uygulama için çok önemliyse, bu sınıfın bir instance'ını Application sınıfına scope
edebilirsiniz. Bu, instance'ın uygulamanın yaşam döngüsünü takip etmesini sağlar. Alternatif olarak, aynı instance'ı
yalnızca uygulamanızdaki belirli bir akışta (örneğin, kayıt veya oturum açma akışı) yeniden kullanmanız gerekiyorsa,
instance'ı bu akışın yaşam döngüsüne sahip olan sınıfa scope etmelisiniz. Örneğin, in-memory veriler içeren bir
RegistrationRepository'yi RegistrationActivity'ye veya kayıt akışının navigation graph'ine scope edebilirsiniz.

Her bir instance'ın yaşam döngüsü, uygulamanızda bağımlılıkları nasıl sağlayacağınıza karar vermede kritik bir
faktördür. Bağımlılıkların yönetildiği ve bağımlılık contaitnerlarina scop edilebildiği dependency injection best
practicelerini takip etmeniz önerilir. Android'de scoping hakkında daha fazla bilgi edinmek için Android'de Scoping ve
Hilt blog gönderisine bakın.

### Represent business models

Data katmanından göstermek istediğiniz veri modelleri, farklı veri kaynaklarından aldığınız bilgilerin bir alt kümesi
olabilir. İdeal olarak, farklı veri kaynakları (hem network hem de local) yalnızca uygulamanızın ihtiyaç duyduğu
bilgileri döndürmelidir; ancak durum genellikle böyle değildir.

Örneğin, yalnızca makale bilgilerini değil, aynı zamanda düzenleme geçmişini, kullanıcı yorumlarını ve bazı meta
verileri de döndüren bir News API sunucusu düşünün:

```kotlin
data class ArticleApiModel(
        val id: Long,
        val title: String,
        val content: String,
        val publicationDate: Date,
        val modifications: Array<ArticleApiModel>,
        val comments: Array<CommentApiModel>,
        val lastModificationDate: Date,
        val authorId: Long,
        val authorName: String,
        val authorDateOfBirth: Date,
        val readTimeMin: Int
)
```

Uygulama, makale hakkında çok fazla bilgiye ihtiyaç duymaz çünkü ekranda yalnızca makalenin içeriğini ve yazarıyla
ilgili temel bilgileri görüntüler. Model sınıflarını ayırmak ve repository'lerinizin yalnızca hiyerarşinin diğer
katmanlarının ihtiyaç duyduğu verileri göstermesini sağlamak iyi bir pratiktir. Örneğin, bir Article model sınıfını
domain ve UI katmanlarına göstermek için ArticleApiModel'i ağdan şu şekilde kırpabilirsiniz:

```kotlin
data class Article(
        val id: Long,
        val title: String,
        val content: String,
        val publicationDate: Date,
        val authorName: String,
        val readTimeMin: Int
)
```

Model sınıflarını ayırmak aşağıdaki şekillerde faydalıdır:

* Verileri yalnızca ihtiyaç duyulana indirgeyerek uygulama belleğinden tasarruf sağlar.

* Harici veri türlerini uygulamanız tarafından kullanılan veri türlerine uyarlar - örneğin, uygulamanız tarihleri temsil
  etmek için farklı bir veri türü kullanabilir.

* Separation of concers ilkesini daha iyi sağlar; örneğin, model sınıfı önceden tanımlanırsa büyük bir ekibin üyeleri
  bir feature'in network ve UI katmanları üzerinde ayrı ayrı çalışabilir.

Bu pratiği genişletebilir ve uygulama mimarinizin diğer bölümlerinde de ayrı model sınıfları tanımlayabilirsiniz;
örneğin veri kaynağı sınıflarında ve ViewModel'lerde. Ancak bu, düzgün bir şekilde belgelemeniz ve test etmeniz gereken
ekstra sınıflar ve logic tanımlamanızı gerektirir. En azından, bir veri kaynağının uygulamanızın geri kalanının
beklediği verilerle eşleşmeyen veriler aldığı her durumda yeni modeller oluşturmanız önerilir.

### Types of daya operations

Data katmanı, ne kadar kritik olduklarına bağlı olarak değişen işlem türleriyle çalışabilir:UI-oriented, app-oriented,
and business-oriented operations.

### UI-oriented operations

UI odaklı işlemler yalnızca kullanıcı belirli bir ekrandayken geçerlidir ve kullanıcı o ekrandan ayrıldığında iptal
edilirler. Veritabanından elde edilen bazı verilerin görüntülenmesi buna bir örnektir.

UI odaklı işlemler genellikle UI katmanı tarafından tetiklenir ve çağıranın yaşam döngüsünü (örneğin ViewModel'in yaşam
döngüsü) takip eder. UI odaklı bir işlem örneği için [Network isteği yapma](#make-a-network-request) bölümüne bakın.

### App-oriented operations

Uygulama odaklı işlemler, uygulama açık olduğu sürece geçerlidir. Uygulama kapatılırsa veya işlem öldürülürse, bu
işlemler iptal edilir. Bir network isteğinin sonucunun, gerektiğinde daha sonra kullanılabilmesi için önbelleğe alınması
buna bir örnektir. Daha fazla bilgi için [in-memory data caching](#caches) bölümüne bakın.

Bu işlemler genellikle Application sınıfının veya data katmanının yaşam döngüsünü takip eder. Bir örnek
için, [Bir işlemi ekrandan daha uzun süre yaşatın](#make-an-operation-live-longer-than-the-screen) bölümüne bakın.

### Business-oriented operations

İş odaklı işlemler iptal edilemez. İşlem ölümünden sağ çıkmalıdırlar. Kullanıcının profiline göndermek istediği bir
fotoğrafın yüklemesini bitirmek buna bir örnektir.

İş odaklı işlemler için öneri WorkManager kullanmaktır. Daha fazla bilgi edinmek
için [WorkManager kullanarak görevleri zamanlama](#schedule-tasks-using-workmanager) bölümüne bakın.

### Expose errors

Repositorylerle ve veri kaynaklarıyla etkileşimler başarılı olabilir veya bir hata oluştuğunda bir exception
fırlatabilir. Coroutine'ler ve flow'lar
için [Kotlin'in yerleşik hata işleme mekanizması](https://kotlinlang.org/docs/exception-handling.html)nı
kullanmalısınız. Suspend fonksiyonları tarafından tetiklenebilecek hatalar için, uygun olduğunda try/catch bloklarını
kullanın; ve
flow'larda [catch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/catch.html)
operatörünü kullanın. Bu yaklaşımla, UI katmanının data katmanını çağırırken exception'ları ele alması beklenir.

Data katmanı farklı hata türlerini anlayabilir ve işleyebilir ve bunları özel exception'lar (örneğin
UserNotAuthenticatedException) kullanarak gösterebilir.

<mark style="background-color:lightblue">Not: Data katmanı ile etkileşimlerin sonucunu modellemenin bir başka yolu da
bir Result sınıfı kullanmaktır. Bu model, sonucun işlenmesinin bir parçası olarak ortaya çıkabilecek hataları ve diğer
sinyalleri modeller. Bu modelde, data katmanı T yerine bir Result<T> tipi döndürerek UI'nin belirli senaryolarda
oluşabilecek bilinen hatalardan haberdar olmasını sağlar. Bu, [LiveData]gibi uygun exception handling'e sahip olmayan
reaktif programlama API'leri için gereklidir.



Coroutine'lerdeki hatalar hakkında daha fazla bilgi edinmek
için [Exceptions in coroutines](https://medium.com/androiddevelopers/exceptions-in-coroutines-ce8da1ec060c) blog
gönderisine bakın.

### Common tasks

Aşağıdaki bölümlerde, Android uygulamalarında yaygın olan belirli görevleri gerçekleştirmek için data katmanının nasıl
kullanılacağına ve tasarlanacağına dair örnekler sunulmaktadır. Örnekler, kılavuzun önceki bölümlerinde bahsedilen tipik
News uygulamasını temel almaktadır.

### Make a network request

Ağ isteği yapmak, bir Android uygulamasının gerçekleştirebileceği en yaygın görevlerden biridir. News uygulamasının
kullanıcıya ağdan alınan en son haberleri sunması gerekir. Bu nedenle, uygulamanın ağ işlemlerini yönetmek için bir veri
kaynağı sınıfına ihtiyacı vardır: NewsRemoteDataSource. Bilgileri uygulamanın geri kalanına göstermek için, haber
verileri üzerindeki işlemleri gerçekleştiren yeni bir repository oluşturulur: NewsRepository.

Gereklilik, kullanıcı ekranı açtığında en son haberlerin her zaman güncellenmesi gerektiğidir. Dolayısıyla, bu UI
odaklı(UI-oriented) bir operasyondur.

### Create the data source

Veri kaynağının en son haberleri döndüren bir fonksiyon sunması gerekir: ArticleHeadline instancelarının bir listesi.
Veri kaynağının ağdan en son haberleri almak için main-safe bir yol sağlaması gerekir. Bunun için, task'in
çalıştırılacağı CoroutineDispatcher ya da Executor'a dependency(bağımlılık) alması gerekir.

Bir ağ isteği yapmak, yeni bir fetchLatestNews() metodu tarafından yürütülen tek seferlik bir çağrıdır:

```kotlin
class NewsRemoteDataSource(
        private val newsApi: NewsApi,
        private val ioDispatcher: CoroutineDispatcher
) {
    /**
     * Fetches the latest news from the network and returns the result.
     * This executes on an IO-optimized thread pool, the function is main-safe.
     */
    suspend fun fetchLatestNews(): List<ArticleHeadline> =
    // Move the execution to an IO-optimized thread since the ApiService
            // doesn't support coroutines and makes synchronous requests.
            withContext(ioDispatcher) {
                newsApi.fetchLatestNews()
            }
}

// Makes news-related network synchronous requests.
interface NewsApi {
    fun fetchLatestNews(): List<ArticleHeadline>
}
```

NewsApi interface'i network API client'ının implementasyonunu gizler;
interface'in [Retrofit](https://square.github.io/retrofit/)
veya [HttpURLConnection](https://developer.android.com/reference/java/net/HttpURLConnection) tarafından desteklenmesi
fark etmez. Interface'lere güvenmek, API implementasyonlarını uygulamanızda değiştirilebilir hale getirir.

{: .note}
Püf Noktası: Interface'lere güvenmek, API implementasyonlarını uygulamanızda
değiştirilebilir hale getirir. Ölçeklenebilirlik sağlamanın ve bağımlılıkları daha kolay değiştirmenize izin vermenin
yanı sıra, testlere sahte veri kaynağı uygulamaları ekleyebildiğiniz için test edilebilirliği de destekler.



### Create the repository

Bu görev için repository sınıfında ekstra bir mantık gerekmediğinden, NewsRepository network veri kaynağı için bir proxy
görevi görür. Bu ekstra soyutlama katmanını eklemenin faydaları [in-memory caching](#implement-in-memory-data-caching)
bölümünde açıklanmıştır.

```kotlin
// NewsRepository is consumed from other layers of the hierarchy.
class NewsRepository(
        private val newsRemoteDataSource: NewsRemoteDataSource
) {
    suspend fun fetchLatestNews(): List<ArticleHeadline> =
            newsRemoteDataSource.fetchLatestNews()
}
```

Repository sınıfını doğrudan UI katmanından nasıl kullanacağınızı öğrenmek
için [UI layer](/docs/app-architecture/guide-to-app-architecture/ui-layer/about-the-ui-layer) kılavuzuna bakın.

### Implement in-memory data caching

News uygulaması için yeni bir gereksinim getirildiğini varsayalım: kullanıcı ekranı açtığında, daha önce bir istek
yapılmışsa cache'lenmiş haberler kullanıcıya sunulmalıdır. Aksi takdirde, uygulama en son haberleri almak için bir
network isteği yapmalıdır.

Yeni gereksinim göz önüne alındığında, uygulama, kullanıcı uygulamayı açık tuttuğu sürece en son haberleri bellekte
tutmalıdır. Dolayısıyla, bu app-oriented operation (uygulama odaklı işlem)dur.

### Caches

In-memory data caching (bellek içi veri önbellekleme) ekleyerek kullanıcı uygulamanızdayken verileri koruyabilirsiniz.
Cache'ler, bazı bilgileri belirli bir süre boyunca (bu durumda kullanıcı uygulamada olduğu sürece) bellekte saklamak
içindir. Cache implementasyonları farklı şekillerde olabilir. Basit bir değiştirilebilir değişkenden, birden fazla
thread üzerinde okuma/yazma işlemlerinden koruyan daha sofistike bir sınıfa kadar değişebilir. Kullanım durumuna bağlı
olarak, caching repository'de veya veri kaynağı sınıflarında uygulanabilir.

### Cache the result of the network request

Basitlik açısından NewsRepository, en son haberleri cache'lemek için mutable bir değişken kullanır. Farklı thread'lerden
gelen okuma ve yazmaları korumak için
bir [Mutex](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.sync/-mutex/)
kullanılır. Paylaşılan mutable state ve concurrency hakkında daha fazla bilgi edinmek
için [Kotlin belgelerine](https://kotlinlang.org/docs/shared-mutable-state-and-concurrency.html#shared-mutable-state-and-concurrency)
bakın.

Aşağıdaki implementasyon, en son haber bilgilerini repository'de bir Mutex ile yazma korumalı bir değişkene cache'ler.
Network isteğinin sonucu başarılı olursa, veriler latestNews değişkenine atanır.

```kotlin
class NewsRepository(
        private val newsRemoteDataSource: NewsRemoteDataSource
) {
    // Mutex to make writes to cached values thread-safe.
    private val latestNewsMutex = Mutex()

    // Cache of the latest news got from the network.
    private var latestNews: List<ArticleHeadline> = emptyList()

    suspend fun getLatestNews(refresh: Boolean = false): List<ArticleHeadline> {
        if (refresh || latestNews.isEmpty()) {
            val networkResult = newsRemoteDataSource.fetchLatestNews()
            // Thread-safe write to latestNews
            latestNewsMutex.withLock {
                this.latestNews = networkResult
            }
        }

        return latestNewsMutex.withLock { this.latestNews }
    }
}
```

### Make an operation live  longer than the screen

Ağ isteği devam ederken kullanıcı ekrandan uzaklaşırsa, istek iptal edilir ve sonuç cache'lenmez. NewsRepository bu
logici gerçekleştirmek için Caller'ın CoroutineScope'unu kullanmamalıdır. Bunun yerine, NewsRepository kendi yaşam
döngüsüne bağlı bir CoroutineScope kullanmalıdır. En son haberleri getirmenin app-oriented bir operasyon olması gerekir.

Dependency injection best practicelerini takip etmek için NewsRepository kendi CoroutineScope'unu oluşturmak yerine
constructor'ında parametre olarak bir scope almalıdır. Repository'lerin işlerinin çoğunu background thread'lerde yapması
gerektiğinden, CoroutineScope'u Dispatchers.Default ya da kendi thread havuzunuz ile yapılandırmalısınız.

```kotlin
class NewsRepository(
        ...,
        // This could be CoroutineScope(SupervisorJob() + Dispatchers.Default).
        private val externalScope: CoroutineScope
) { ... }
```

NewsRepository harici CoroutineScope ile app oriented operasyonlar gerçekleştirmeye hazır olduğundan, veri kaynağına
çağrıyı gerçekleştirmeli ve sonucunu bu scope tarafından başlatılan yeni bir coroutine ile kaydetmelidir:

```kotlin
class NewsRepository(
        private val newsRemoteDataSource: NewsRemoteDataSource,
        private val externalScope: CoroutineScope
) {
    /* ... */

    suspend fun getLatestNews(refresh: Boolean = false): List<ArticleHeadline> {
        return if (refresh) {
            externalScope.async {
                newsRemoteDataSource.fetchLatestNews().also { networkResult ->
                    // Thread-safe write to latestNews.
                    latestNewsMutex.withLock {
                        latestNews = networkResult
                    }
                }
            }.await()
        } else {
            return latestNewsMutex.withLock { this.latestNews }
        }
    }
}
```

async, coroutine'i harici scope'da başlatmak için kullanılır. await, network isteği geri gelene ve sonuç cache'e
kaydedilene kadar askıya almak için yeni coroutine üzerinde çağrılır. O zamana kadar kullanıcı hala ekrandaysa, en son
haberleri görecektir; kullanıcı ekrandan uzaklaşırsa, await iptal edilir ancak async içindeki logic çalışmaya devam
eder.

CoroutineScope patternleri hakkında daha fazla bilgi edinmek için
bu [blog](https://medium.com/androiddevelopers/coroutines-patterns-for-work-that-shouldnt-be-cancelled-e26c40f142ad)
yazısına bakın.

### Save and retrieve data from disk

Bookmark edilmiş haberler ve kullanıcı tercihleri gibi verileri kaydetmek istediğinizi varsayalım. Bu tür verilerin
process death'den kurtulması ve kullanıcı network'e bağlı olmasa bile erişilebilir olması gerekir.

Üzerinde çalıştığınız verilerin process death'den kurtulması gerekiyorsa, aşağıdaki yollardan biriyle diskte saklamanız
gerekir:

-Sorgulanması gereken, referans bütünlüğüne ihtiyaç duyan veya kısmi güncellemelere ihtiyaç duyan büyük veri kümeleri
için verileri bir Room veritabanına kaydedin. News uygulaması örneğinde, haber makaleleri veya yazarları veritabanına
kaydedilebilir.
-Yalnızca alınması ve ayarlanması gereken (sorgulanmayan veya kısmen güncellenmeyen) küçük veri kümeleri için DataStore
kullanın. News uygulaması örneğinde, kullanıcının tercih ettiği tarih biçimi veya diğer görüntüleme tercihleri
DataStore'a kaydedilebilir.
-JSON nesnesi gibi veri parçaları için bir dosya kullanın.

[Source of truth bölümü](about-the-data-layer.md#source-of-truth)nde belirtildiği gibi, her veri kaynağı yalnızca bir
kaynakla çalışır ve belirli bir veri türüne karşılık gelir (örneğin, News, Authors, NewsAndAuthors veya
UserPreferences). Veri kaynağını kullanan sınıflar, verilerin nasıl kaydedildiğini bilmemelidir (örneğin, bir
veritabanına veya bir dosyaya).

### Room as a data source

Her veri kaynağının belirli bir veri türü için yalnızca bir kaynakla çalışma sorumluluğuna sahip olması gerektiğinden,
bir Room veri kaynağı parametre olarak ya bir [veri erişim nesnesi (DAO)](https://developer.android.com/training/data-storage/room/accessing-data) ya da veritabanının kendisini alacaktır.
Örneğin, NewsLocalDataSource parametre olarak NewsDao'nun bir instance'ını alabilir ve AuthorsLocalDataSource
AuthorsDao'nun bir instance'ını alabilir.

Bazı durumlarda, ekstra bir lojik gerekmiyorsa, DAO testlerde kolayca değiştirebileceğiniz bir interface olduğundan,
DAO'yu doğrudan repository'ye enjekte edebilirsiniz.

Room API'leri ile çalışma hakkında daha fazla bilgi edinmek için [Room kılavuzları](https://developer.android.com/training/data-storage/room)na bakın.

### DataStore as a data source
[DataStore](/docs/app-architecture/architecture-components/data-layer-libraries/datastore), kullanıcı ayarları gibi key-value çiftlerini saklamak için mükemmeldir. Örnekler arasında zaman biçimi, bildirim tercihleri ve kullanıcı okuduktan sonra haber öğelerinin gösterilip gösterilmeyeceği veya gizlenip gizlenmeyeceği yer alabilir. DataStore aynı zamanda [protocol buffer](https://developers.google.com/protocol-buffers)'ları olan tiplendirilmiş nesneleri de depolayabilir.

Diğer tüm nesnelerde olduğu gibi, DataStore tarafından desteklenen bir veri kaynağı, belirli bir türe veya uygulamanın belirli bir bölümüne karşılık gelen verileri içermelidir. Bu durum DataStore için daha da geçerlidir çünkü DataStore okumaları, bir değer her güncellendiğinde yayılan bir akış olarak açığa çıkar. Bu nedenle, ilgili tercihleri aynı DataStore'da saklamalısınız.

Örneğin, yalnızca bildirimle ilgili tercihleri işleyen bir NotificationsDataStore'a ve yalnızca haber ekranıyla ilgili tercihleri işleyen bir NewsPreferencesDataStore'a sahip olabilirsiniz. Bu şekilde, newsScreenPreferencesDataStore.data akışı yalnızca o ekranla ilgili bir tercih değiştirildiğinde yayıldığı için güncellemeleri daha iyi kapsamlandırabilirsiniz. Bu aynı zamanda nesnenin yaşam döngüsünün daha kısa olabileceği anlamına gelir çünkü yalnızca haber ekranı görüntülendiği sürece yaşayabilir.

DataStore API'leri ile çalışma hakkında daha fazla bilgi edinmek için [DataStore kılavuzları](/docs/app-architecture/architecture-components/data-layer-libraries/datastore)na bakın.

### A file as a data source
JSON nesnesi veya bitmap gibi büyük nesnelerle çalışırken, bir File nesnesiyle çalışmanız ve thread'leri değiştirmeniz gerekir.

Storage ile çalışma hakkında daha fazla bilgi edinmek için [Storage oveview](/docs/core-topics/app-data-and-files/about-storage) sayfasına bakın.

### Schedule tasks using WorkManager
News uygulaması için yeni bir gereksinim daha getirildiğini varsayalım: Uygulama, kullanıcıya cihaz şarjda olduğu ve kesintisiz bir ağa bağlı olduğu sürece en son haberleri düzenli ve otomatik olarak alma seçeneği sunmalıdır. Bu da bunu business-oriented operation haline getiriyor. Bu gereklilik, kullanıcı uygulamayı açtığında cihazın bağlantısı olmasa bile kullanıcının en son haberleri görebilmesini sağlar.

[WorkManager](/docs/app-architecture/architecture-components/data-layer-libraries/workmanager/about-workmanager), asenkron ve güvenilir iş planlamayı kolaylaştırır ve kısıtlama yönetimiyle ilgilenebilir. Kalıcı işler için önerilen kütüphanedir. Yukarıda tanımlanan görevi gerçekleştirmek için bir [Worker sınıfı](/docs/core-topics/background-tasks/persistent-work/threading/threading-in-coroutineworker) oluşturulur: RefreshLatestNewsWorker. Bu sınıf, en son haberleri getirmek ve diske cache'lemek için NewsRepository'yi dependency olarak alır.
```kotlin
class RefreshLatestNewsWorker(
    private val newsRepository: NewsRepository,
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {

    override suspend fun doWork(): Result = try {
        newsRepository.refreshLatestNews()
        Result.success()
    } catch (error: Throwable) {
        Result.failure()
    }
}
```

Bu tür görevler için business logic kendi sınıfında kapsüllenmeli ve ayrı bir veri kaynağı olarak ele alınmalıdır. WorkManager daha sonra yalnızca tüm kısıtlamalar karşılandığında işin bir background thread üzerinde yürütülmesini sağlamaktan sorumlu olacaktır. Bu kalıba bağlı kalarak, gerektiğinde farklı ortamlardaki uygulamaları hızlı bir şekilde değiştirebilirsiniz.

Bu örnekte, haberlerle ilgili bu görev, bağımlılık olarak yeni bir veri kaynağı alacak olan NewsRepository'den çağrılmalıdır: NewsTasksDataSource, aşağıdaki gibi implemente edilmiştir:

````kotlin
private const val REFRESH_RATE_HOURS = 4L
private const val FETCH_LATEST_NEWS_TASK = "FetchLatestNewsTask"
private const val TAG_FETCH_LATEST_NEWS = "FetchLatestNewsTaskTag"

class NewsTasksDataSource(
    private val workManager: WorkManager
) {
    fun fetchNewsPeriodically() {
        val fetchNewsRequest = PeriodicWorkRequestBuilder<RefreshLatestNewsWorker>(
            REFRESH_RATE_HOURS, TimeUnit.HOURS
        ).setConstraints(
            Constraints.Builder()
                .setRequiredNetworkType(NetworkType.TEMPORARILY_UNMETERED)
                .setRequiresCharging(true)
                .build()
        )
            .addTag(TAG_FETCH_LATEST_NEWS)

        workManager.enqueueUniquePeriodicWork(
            FETCH_LATEST_NEWS_TASK,
            ExistingPeriodicWorkPolicy.KEEP,
            fetchNewsRequest.build()
        )
    }

    fun cancelFetchingNewsPeriodically() {
        workManager.cancelAllWorkByTag(TAG_FETCH_LATEST_NEWS)
    }
}
````

Bu tür sınıflar sorumlu oldukları verilere göre adlandırılır; örneğin NewsTasksDataSource veya PaymentsTasksDataSource. Belirli bir veri türüyle ilgili tüm görevler aynı sınıfta kapsüllenmelidir.

Görevin uygulama başlangıcında tetiklenmesi gerekiyorsa, repository'i bir [Initializer](https://developer.android.com/reference/kotlin/androidx/startup/Initializer)'dan çağıran [App Startup](/docs/app-architecture/app-startup) kütüphanesini kullanarak WorkManager isteğini tetiklemeniz önerilir.

WorkManager API'leri ile çalışma hakkında daha fazla bilgi edinmek için [WorkManager kılavuzları](/docs/app-architecture/architecture-components/data-layer-libraries/workmanager/about-workmanager)na bakın.


### Testing
[Dependency injection](/docs/app-architecture/dependency-injection/about-dependency-injection) best practiceleri, uygulamanızı test ederken yardımcı olur. Harici kaynaklarla iletişim kuran sınıflar için interface'lere güvenmek de faydalıdır. Bir birimi test ederken, testi deterministik ve güvenilir hale getirmek için bağımlılıklarının sahte sürümlerini enjekte edebilirsiniz.

### Unit tests
Veri katmanını test ederken [genel test yönergeleri](/docs/best-practices/testing/test-apps-on-android) geçerlidir. Birim testleri için, gerektiğinde gerçek nesneler kullanın ve bir dosyadan okuma veya ağdan okuma gibi harici kaynaklara ulaşan bağımlılıkları taklit edin.

### Integration tests
Harici kaynaklara erişen entegrasyon testleri, gerçek bir cihaz üzerinde çalıştırılmaları gerektiğinden daha az deterministik olma eğilimindedir. Entegrasyon testlerini daha güvenilir hale getirmek için bu testleri kontrollü bir ortamda yürütmeniz önerilir.

Room, veritabanları için testlerinizde tamamen kontrol edebileceğiniz bir in-memory database oluşturmanıza olanak tanır. Daha fazla bilgi edinmek için [Test and debug your database sayfası](/docs/core-topics/app-data-and-files/save-data-in-a-local-database/test-and-debug-your-database)na bakın.

Networking için, [WireMock](http://wiremock.org/) veya [MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver) gibi HTTP ve HTTPS çağrılarını taklit etmenize ve isteklerin beklendiği gibi yapıldığını doğrulamanıza olanak tanıyan popüler kütüphaneler vardır.

### [Samples](https://developer.android.com/topic/architecture/data-layer?continue=https%3A%2F%2Fdeveloper.android.com%2Fcourses%2Fpathways%2Fandroid-architecture%23article-https%3A%2F%2Fdeveloper.android.com%2Ftopic%2Farchitecture%2Fdata-layer#samples)
