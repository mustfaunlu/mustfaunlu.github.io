---
layout: default
title: About the UI layer
grand_parent: Guide to app architecture
nav_order: 1
parent: Ui layer
---

# UI Layer

## About the UI layer

Kullanıcı arayüzünün(UI) rolü, uygulama verilerini ekranda görüntülemek ve ayrıca kullanıcı etkileşiminin(user
interaction) birincil noktası olarak hizmet etmektir. Veriler, kullanıcı etkileşimi (bir butona basmak gibi) veya harici
giriş (bir network response gibi) nedeniyle değiştiğinde, kullanıcı arayüzü bu değişiklikleri yansıtacak şekilde
güncellenmelidir. Yani, UI, data katmanından alınan uygulama state’inin görsel bir temsilidir.

Ancak data katmanından aldığınız uygulama verileri genellikle görüntülemeniz gereken bilgilerden farklı bir formattadır.
Örneğin, UI için verilerin yalnızca bir kısmına ihtiyacınız olabilir veya kullanıcıyla ilgili bilgileri sunmak için iki
farklı veri kaynağını birleştirmeniz gerekebilir. Uyguladığınız logic ne olursa olsun, tam olarak işlemesi için ihtiyaç
duyduğu tüm bilgileri UI'ye iletmeniz gerekir. UI katmanı, uygulama veri değişikliklerini UI'in sunabileceği bir forma
dönüştüren ve ardından bunu görüntüleyen piplinedir.
Şekil 1. Uygulama mimarisinde UI katmanının rolü;
![UI Layer Role](/assets/images/ui-layer-role.png)

{: .note}
Not: Bu sayfada sunulan öneriler ve best practiceler, ölçeklenmelerine, kalite
ve sağlamlığı artırmalarına ve test edilmelerini kolaylaştırmalarına olanak sağlamak için geniş bir uygulama yelpazesine
uygulanabilir. Ancak, bunları kılavuz olarak ele almalı ve gerektiğinde gereksinimlerinize göre uyarlamalısınız.


[Architecture: The UI Layer - MAD Skills](https://youtu.be/p9VR8KbmzEE)

### A basic case study

Bir kullanıcının okuması için haber makaleleri getiren bir uygulama düşünün. Uygulama, okunabilecek makaleler sunan bir
makaleler ekranına sahiptir ve ayrıca oturum açmış kullanıcıların gerçekten öne çıkan makalelere yer işareti koymasına
olanak tanır. Her an çok sayıda makale olabileceği göz önüne alındığında, okuyucunun makalelere kategorilere göre göz
atabilmesi gerekir. Özetle, uygulama, kullanıcıların aşağıdakileri yapmasına izin verir:

* Okunabilecek makaleleri görüntüleyin.
* Makalelere kategoriye göre göz atın.
* Oturum açın ve belirli makalelere yer işareti koyun.
* Uygunsa bazı premium özelliklere erişin.

![UI Layer Case Study](/assets/images/jetnews-ui-layer-case-study.png)

Aşağıdaki bölümler, bu örneği, tek yönlü veri akışı ilkelerini tanıtmak ve bu ilkelerin UI katmanı için uygulama
mimarisi bağlamında çözmeye yardımcı olduğu sorunları göstermek için bir use case olarak kullanır.

### UI layer Architecture

UI terimi, bunu yapmak için hangi API'leri kullandıklarından bağımsız olarak (Views
veya [Jetpack Compose](https://developer.android.com/jetpack/compose)) verileri görüntüleyen activity ve fragment gibi
UI elementlerini ifade eder. Data katmanının rolü uygulama verilerini tutmak, yönetmek ve bunlara erişim sağlamak
olduğundan, UI katmanı aşağıdaki adımları gerçekleştirmelidir:

* Uygulama verilerini tüketin(consume etmek) ve UI’in kolayca render edilebilecegi verilere dönüştürün.
* UI ile render edilebilir verileri tüketin ve kullanıcıya sunulmak üzere UI elementlerine dönüştürün.
* Bu birleştirilmiş UI elementlerinden kullanıcı input eventlerini tüketin ve etkilerini gerektiği gibi UI verilerine
  yansıtın.
* 1'den 3'e kadar olan adımları gerektiği kadar tekrarlayın.

Bu kılavuzun geri kalanı, bu adımları gerçekleştiren bir UI katmanının nasıl implementasyonunu gösterir. Bu kılavuz
özellikle aşağıdaki görevleri ve kavramları kapsar:

* UI state nasıl tanımlanır.
* UI state’ini üretme(produce) ve yönetme aracı olarak tek yönlü veri akışı ((Undirectional Data Flow)UDF).
* UDF ilkelerine göre observable veri türleriyle UI state’i nasıl ortaya çıkar.
* Observable UI state’ini tüketen UI nasıl implement edilir.
  Bunlardan en temel olanı UI state’inin tanımıdır.

### Define UI State

Daha önce özetlenen [case study](#a-basic-case-study) üzerinden devam edelim. Kısacası, kullanıcı arayüzü her makale
için bazı metadata ile birlikte bir makale listesi gösterir. Uygulamanın kullanıcıya sunduğu bu bilgiler, UI state’dir.
Başka bir deyişle: UI, kullanıcının gördüğü şeyse, UI state, uygulamanın görmeleri gerektiğini söylediği şeydir. Aynı
madalyonun iki yüzü gibi, UI da UI state’in görsel temsilidir. UI state’indeki herhangi bir değişiklik, hemen UI'ye
yansıtılır.
![UI, UI state’i ile ekrandaki UI elementlerinin bağlanmasının bir sonucudur.](https://developer.android.com/static/topic/libraries/architecture/images/mad-arch-ui-elements-state.png)

Use case incelemesini düşünün; News uygulamasının gereksinimlerini karşılamak için, kullanıcı arayüzünü tam olarak
oluşturmak için gereken bilgiler, aşağıdaki gibi tanımlanan bir NewsUiState data classda encapsule edilebilir:

```kotlin
    data class NewsUiState(
    val isSignedIn: Boolean = false,
    val isPremium: Boolean = false,
    val newsItems: List<NewsItemUiState> = listOf(),
    val userMessages: List<Message> = listOf()
)

data class NewsItemUiState(
    val title: String,
    val body: String,
    val bookmarked: Boolean = false,
    ...
)
```

### Immutability

Yukarıdaki örnekteki UI state tanımı immutabledir(degismez). Bunun en önemli faydası, immutable nesnelerin zamanın bir
anında uygulamanın state'ine ilişkin garantiler sağlamasıdır. Bu, UI’i tek bir role odaklanmak için serbest bırakır:
state’i okumak ve UI elementlerini buna göre güncellemek. Sonuç olarak, UI verilerinin tek kaynağı(ssot)  UI'nin kendisi
olmadığı sürece UI state'ini asla doğrudan UI'de değiştirmemelisiniz. Bu ilkeyi ihlal etmek, aynı bilgi parçası için
birden fazla doğruluk kaynağına(multiple source of truth istenmeyen durumdur) yol açarak veri tutarsızlıklarına ve ince
hatalara neden olur.
Örneğin, use casedeki UI state'ten bir NewsItemUiState nesnesindeki bookmarked flag'i Activity sınıfında güncellenseydi,
bu flag bir makalenin bookmarked statüsünün kaynağı olarak data katmanı ile rekabet halinde olacaktı. Immutable data
class'lar bu tür antipattern'leri önlemek için çok kullanışlıdır.

{: .note }
Kilit Nokta: Yalnızca veri kaynakları veya ownerlari, ortaya koydukları
verilerin güncellenmesinden sorumlu olmalıdır.



### Naming conventions in this guide

Bu kılavuzda, UI state sınıfları, ekranın veya tanımladıkları ekranın bir bölümünün fonksiyonalitesine göre
adlandırılır. Sözleşme şu şekildedir: fonksiyonalite + UiState. Örneğin, haberleri görüntüleyen bir ekranın state’i
NewsUiState olarak adlandırılabilir ve haber öğeleri listesindeki bir haber öğesinin state’i NewsItemUiState olabilir.

### Manage State with Undirectional Data Flow

Önceki bölüm, UI state’inin, UI'in oluşturması için gereken ayrıntıların immutable bir snapshot olduğunu belirttik.
Ancak, uygulamalardaki verilerin dinamik doğası geregi, bu state’in zaman içinde değişebileceği anlamına gelir. Bunun
nedeni, uygulamayı doldurmak(populate) için kullanılan temel verileri değiştiren kullanıcı etkileşimi(user interaction)
veya diğer eventlar olabilir.

Bu etkileşimler, bunları işlemek, her event’e uygulanacak logic’i tanımlamak ve UI state ini oluşturmak için backing
data resource lerine gerekli transformationlari gerçekleştirmek için bir arabulucudan(mediator) yararlanabilir. Bu
etkileşimler ve bunların logici kullanıcı arayüzünün kendisinde barındırılabilir, ancak kullanıcı arayüzü adından da
anlaşılacağı gibi data owner, producer, transformer ve daha fazlası haline gelmeye başladıkça bu durum hızla
hantallaşabilir. Ayrıca, ortaya çıkan kod, ayırt edilebilir sınırları olmayan tightly coupled bir karışım olduğundan, bu
durum test edilebilirliği etkileyebilir. Sonuç olarak, kullanıcı arayüzü azaltılmış yükten faydalanmaya hazırdır. UI
state’i çok basit olmadığı sürece, UI'nin tek sorumluluğu UI state’ini kullanmak ve görüntülemek olmalıdır.

Bu bölümde, healthy separation of responsibility uygulanmasına yardımcı olan bir mimari model olan Tek Yönlü Veri
Akışı (unidirectional data flow (UDF) ele alınmaktadır.

### State Holders

UI state’inin üretilmesinden sorumlu olan ve o görev için gerekli logici içeren sınıflara state holder denir. State
Holderlar, yönettikleri ilgili UI elementlerinin scope una bağlı
olarak, [bottom appbar](https://material.io/components/app-bars-bottom) gibi tek bir widget öğesinden tüm ekrana veya
bir navigation hedefine kadar çeşitli boyutlarda gelir.
İkinci durumda, tipik uygulama
bir [ViewModel](/docs/app-architecture/architecture-components/ui-layer-libraries/lifecycle-aware-components/ViewModel/about-viewmodel)
örneğidir, ancak uygulamanın gereksinimlerine bağlı olarak basit bir sınıf yeterli olabilir.
Yukarıdaki [case study](#a-basic-case-study) deki News uygulaması, örneğin, o bölümde görüntülenen ekran için UI state
oluşturmak üzere state holder olarak bir NewsViewModel sınıfını kullanır.

{: .note }
Kilit Nokta: ViewModel türü, veri katmanına erişimle birlikte ekran
düzeyinde Ui state’inin yönetimi için önerilen implementasyondur. Ayrıca, yapılandırma değişikliklerinden(configuration
changes) otomatik olarak kurtulur. ViewModel sınıfları, uygulamadaki eventlara uygulanacak logic’i tanımlar ve sonuç
olarak güncellenmiş state’i üretir.


UI ile state producer arasındaki karşılıklı bağımlılığı modellemenin birçok yolu vardır. Bununla birlikte,UI ile
ViewModel sınıfı arasındaki etkileşim, büyük ölçüde event input ve ardından gelen state output olarak
anlaşılabileceğinden, ilişki aşağıdaki şemada gösterildiği gibi gösterilebilir:
![UDF'in app architecture'da calisma sekli](/assets/images/udf-diagram.png)
State’in aşağı doğru aktığı ve eventlarin yukarı doğru aktığı patterne tek yönlü veri akışı(unidirectional data flow) (
UDF) denir. Uygulama mimarisi için bu modelin sonuçları aşağıdaki gibidir:

* ViewModel, UI tarafından tüketilecek state’i tutar ve gösterir. UI state’i,ViewModel tarafından dönüştürülen uygulama
  verileridir.
* UI, kullanıcı eventlerini ViewModel'e bildirir.
* ViewModel, kullanıcı eylemlerini işler ve state’i günceller.
* Güncellenen state, işlenmek üzere kullanıcı arayüzüne(UI) geri beslenir.
* Yukarıdaki durum, bir state değişimine neden olan herhangi bir event için tekrarlanır.

Navigation destinations veya ekranlar için ViewModel, veri almak ve bunları UI state'ine dönüştürmek için repository'ler
veya use case sınıfları ile birlikte çalışır ve aynı zamanda state'in değişmesine neden olabilecek event'lerin
etkilerini de dahil eder. Daha önce bahsedilen case study, her biri bir başlık, açıklama, kaynak, yazar adı, yayın
tarihi ve bookmarked olup olmadığına sahip bir makale listesi içerir. Her bir makale öğesi için UI aşağıdaki gibi
görünür:

![Case study'deki articlerin UI itemlari](/assets/images/case-study-article-items.png)

Bir makaleye bookmark koymak isteyen bir kullanıcı, state değişimlerine neden olabilecek bir event’a örnektir. State
producer olarak, ViewModel'in sorumluluğundaki sey; UI state'in deki tüm alanları doldurmak ve UI'in tam olarak işlemesi
için gereken eventlari handle etmek amaciyla gereken tüm logici tanımlamaktır.

![UDF'deki eventlerin ve verilerin döngüsünü gösteren diyagram](/assets/images/cycle-of-events-and-data-udf.png)

Aşağıdaki bölümlerde, state değişikliklerine neden olan eventlere ve bunların UDF kullanılarak nasıl handle
edilebilecegine daha yakından bakılmaktadır.

### Types of Logic

Bir makaleyi bookmarklamak, uygulamanıza değer kattığı için bir iş mantığı(business logic) örneğidir. Bununla ilgili
daha fazla bilgi edinmek
için [veri katmanı(data layer)](/docs/app-architecture/guide-to-app-architecture/data-layer/about-the-data-layer)
sayfasına bakın. Ancak, tanımlanması önemli olan farklı logic türleri vardır:

* Business logic, uygulama verileri için ürün gereksinimlerinin implement edilmesidir. Daha önce belirtildiği gibi, case
  study uygulamasında bir makaleye bookmark koymaktır. Business logic genellikle domain veya data katmanlarına
  yerleştirilir, ancak asla UI katmanına yerleştirilmez.

* UI behavior logic veya UI logic, state değişikliklerinin ekranda nasıl görüntüleneceğidir. Örnekler arasında,
  Android [Resource](https://developer.android.com/reference/android/content/res/Resources)lerini kullanarak ekranda
  gösterilecek doğru metni elde etmek, kullanıcı bir butonu tıkladığında belirli bir ekrana gitmek veya
  bir [toast](https://developer.android.com/guide/topics/ui/notifiers/toasts)
  veya [snackbar](https://developer.android.com/training/snackbar) kullanarak ekranda bir kullanıcı mesajı görüntülemek
  yer alır.

UI logici, özellikle [Context](https://developer.android.com/reference/android/content/Context) gibi UI türlerini
içerdiğinde, ViewModel'de değil UI'de yaşamalıdır. UI’in karmaşıklığı artarsa ve UI logicini test edilebilirliği ve
endişelerin ayrılmasını(separation of concerns) desteklemek için başka bir sınıfa devretmek istiyorsanız, state holder
olarak basit bir sınıf oluşturabilirsiniz. UI’de oluşturulan basit sınıflar, UI’nin yaşam döngüsünü takip ettikleri için
Android SDK bağımlılıklarını alabilir; ViewModel nesnelerinin ömrü daha uzundur.
State holderlar ve bunların UI oluşturmaya yardımcı olma bağlamına nasıl uydukları hakkında daha fazla bilgi
için [Jetpack Compose State kılavuzu](https://developer.android.com/jetpack/compose/state#managing-state)na bakın.

### Why use UDF?

UDF, state değişimlerinin başladığı yeri, dönüştüğü yeri ve nihai olarak tüketildiği yeri de ayırır. Bu ayrım, kullanıcı
arayüzünün tam olarak adının ima ettiği şeyi yapmasına olanak tanır yani state değişikliklerini gözlemleyerek(observe)
bilgileri görüntüleyin ve bu değişiklikleri ViewModel'e ileterek user’in amacını iletin.

Başka bir deyişle, UDF aşağıdakilere izin verir:

* Veri tutarlılığı. Kullanıcı arayüzü için tek bir doğruluk kaynağı(ssot) vardır.
* Test edilebilirlik. State kaynağı yalıtılmıştır ve bu nedenle kullanıcı arabiriminden bağımsız olarak test edilebilir.
* Bakım kolaylığı. State degisikligi, degisikliklerin hem kullanıcı eventlerinin hem de çektikleri veri kaynaklarının
  bir sonucu olduğu iyi tanımlanmış bir model izler.

### Expose UI State

UI state’inizi tanımlayıp, o state’in üretimini(produce) nasıl yöneteceğinizi belirledikten sonra sıra, üretilen state’i
UI'ye sunmaktır. State’in üretimini yönetmek için UDF kullandığınız için, üretilen state’i bir stream olarak
düşünebilirsiniz; başka bir deyişle, state’in birden çok sürümü zaman içinde üretilecektir. Sonuç olarak, UI state’ini
LiveData veya StateFlow gibi gözlemlenebilir bir data holderla göstermelisiniz. Bunun nedeni, kullanıcı arabiriminin,
verileri doğrudan ViewModel'den manuel olarak çekmek zorunda kalmadan state’de yapılan herhangi bir değişikliğe tepki
verebilmesidir. Bu tipler ayrıca, yapılandırma değişikliklerinden sonra hızlı durum geri yüklemesi için yararlı olan,
kullanıcı arayüzü state’inin her zaman en son sürümünün önbelleğe alınması avantajına sahiptir.

```kotlin 
//views
class NewsViewModel(...) : ViewModel() {

    val uiState: StateFlow<NewsUiState> = …
}
```

```kotlin
//compose
class NewsViewModel(...) : ViewModel() {

    val uiState: NewsUiState = …
}
```

Observable bir data holder olarak LiveData'ya giriş
için [bu codelab](https://developer.android.com/codelabs/basic-android-kotlin-training-livedata)'e bakın. Kotlin
flowlara benzer bir giriş için bkz. [Android'de Kotlin flows](https://developer.android.com/kotlin/flow).

<mark style= "background-color: lightblue">Not: Jetpack Compose uygulamalarında, UI state’inin gösterilmesi için
Compose'un mutableStateOf veya snapshotFlow gibi gözlemlenebilir State API'lerini kullanabilirsiniz. Bu kılavuzda
gördüğünüz StateFlow veya LiveData gibi her türlü gözlemlenebilir data holder, uygun extensionlar kullanılarak
Compose'da kolayca tüketilebilir.



UI'ye maruz kalan verilerin nispeten basit olduğu durumlarda, state holderin yayılımı(emission) ile ilişkili ekran veya
UI elementi arasındaki ilişkiyi aktardığı için genellikle verileri bir UI state tipine sarmaya değer. Ayrıca, UI
elementi daha karmaşık hale geldikçe, UI elementini render etmek için gereken ekstra bilgileri barındırmak için UI state
tanımına ekleme yapmak her zaman daha kolaydır.

Bir UiState akışı oluşturmanın yaygın bir yolu, backing mutable stream'i ViewModel'den immutable stream olarak
göstermektir; örneğin, MutableStateFlow<UiState>'i StateFlow<UiState> olarak göstermek gibi.

```kotlin
//views
class NewsViewModel(...) : ViewModel() {

    private val _uiState = MutableStateFlow(NewsUiState())
    val uiState: StateFlow<NewsUiState> = _uiState.asStateFlow()

    ...

}

//compose
class NewsViewModel(...) : ViewModel() {

    var uiState by mutableStateOf(NewsUiState())
        private set

    ...
}
```

ViewModel daha sonra state’i dahili olarak değiştiren methodlari expose edebilir ve UI’in tüketmesi için güncellemeler
yayınlayabilir. Örneğin, asenkron bir action’in gerçekleştirilmesi gereken durumu ele
alalım; [viewModelScope](/docs/app-architecture/architecture-components/ui-layer-libraries/lifecycle-aware-components/use-kotlin-coroutines-with-lifecycle-aware-components#ViewModelScope)
kullanılarak bir coroutine başlatılabilir ve mutable state tamamlandıktan sonra güncellenebilir.

```kotlin
//views
class NewsViewModel(
    private val repository: NewsRepository,
    ...
) : ViewModel() {

    private val _uiState = MutableStateFlow(NewsUiState())
    val uiState: StateFlow<NewsUiState> = _uiState.asStateFlow()

    private var fetchJob: Job? = null

    fun fetchArticles(category: String) {
        fetchJob?.cancel()
        fetchJob = viewModelScope.launch {
            try {
                val newsItems = repository.newsItemsForCategory(category)
                _uiState.update {
                    it.copy(newsItems = newsItems)
                }
            } catch (ioe: IOException) {
                // Handle the error and notify the UI when appropriate.
                _uiState.update {
                    val messages = getMessagesFromThrowable(ioe)
                    it.copy(userMessages = messages)
                }
            }
        }
    }
}


//compose
class NewsViewModel(
    private val repository: NewsRepository,
    ...
) : ViewModel() {

    var uiState by mutableStateOf(NewsUiState())
        private set

    private var fetchJob: Job? = null

    fun fetchArticles(category: String) {
        fetchJob?.cancel()
        fetchJob = viewModelScope.launch {
            try {
                val newsItems = repository.newsItemsForCategory(category)
                uiState = uiState.copy(newsItems = newsItems)
            } catch (ioe: IOException) {
                // Handle the error and notify the UI when appropriate.
                val messages = getMessagesFromThrowable(ioe)
                uiState = uiState.copy(userMessages = messages)
            }
        }
    }
}
```

Yukarıdaki örnekte, NewsViewModel sınıfı, belirli bir kategori için makaleleri getirmeye çalışır ve ardından, girişimin
sonucunu (başarılı veya başarısız) UI’in buna uygun şekilde tepki verebileceği UI state’inde yansıtır. Hata handle etme
hakkında daha fazla bilgi edinmek için [Show errors on the screen](#show-errors-on-the-screen) bölümüne bakın

{: .note }
Not: State’in ViewModel'deki fonksiyonlar aracılığıyla değiştirildiği
yukarıdaki örnekte gösterilen pattern, tek yönlü veri akışının(UDF) en popüler uygulamalarından biridir.


### Additional considerations

Önceki kılavuza ek olarak, UI state’ini gösterirken aşağıdakileri göz önünde bulundurun:

* Bir UI state object, birbiriyle ilişkili stateleri handle edebilmelidir. Bu, daha az tutarsızlığa yol açar ve kodun
  anlaşılmasını kolaylaştırır. News item listesini ve bookmarlarin sayısını iki farklı streamde gösterirseniz, birinin
  güncellenip diğerinin güncellenmediği bir durumla karşılaşabilirsiniz. Tek bir stream kullandığınızda, her iki element
  de güncel tutulur. Ayrıca, bazı business logic, resourcelerin bir kombinasyonunu gerektirebilir. Örneğin, yalnızca
  kullanıcı oturum açmışsa ve bu kullanıcı bir premium haber hizmetine aboneyse, bir bookmark butonunu göstermeniz
  gerekebilir. Bir UI state sınıfını aşağıdaki gibi tanımlayabilirsiniz:

```kotlin
data class NewsUiState(
    val isSignedIn: Boolean = false,
    val isPremium: Boolean = false,
    val newsItems: List<NewsItemUiState> = listOf()
)

val NewsUiState.canBookmarkNews: Boolean get() = isSignedIn && isPremium
```

Bu bildirimde, bookmark butonunun görünürlüğü diğer iki propertynin türetilmiş bir propertysidir. Business logic daha
karmaşık hale geldikçe, tüm propertylerin anında kullanılabilir olduğu tek bir UiState sınıfına sahip olmak giderek daha
önemli hale geliyor.

* Ui stateleri: tek stream mı yoksa multiple stream mı? UI stateinin tek bir akışta veya birden çok akışta göstermek
  arasında seçim yapmak için temel yol gösterici ilke, previous bullet pointtir: emit edilmis itemler arasındaki ilişki.
  Tek akış gosteriminin en büyük avantajı kolaylık ve veri tutarlılığıdır: statedeki tüketiciler her zaman her an en son
  bilgilere sahiptir. Ancak, ViewModel'den ayrı state akışlarının uygun olabileceği durumlar vardır:
    * İlişkisiz veri türleri: Kullanıcı arayüzünü oluşturmak için gereken bazı stateler birbirinden tamamen bağımsız
      olabilir. Bu gibi durumlarda, özellikle bu statelerden biri diğerinden daha sık güncelleniyorsa, bu farklı
      stateleri bir araya getirmenin maliyeti faydalarından daha ağır basabilir.
    * UiState diffing: Bir UiState nesnesinde ne kadar çok field varsa, fieldlarindan birinin güncellenmesi sonucunda
      akışın emit etme(yayilma) olasılığı o kadar yüksektir. Viewlerin, ardışık yaymalari farklı mı yoksa aynı mı
      olduğunu anlamak için farklılaştırma mekanizması olmadığından, her yayma, viewde bir güncellemeye neden olur. Bu,
      LiveData'da Flow API'lerini
      veya  [distinctUntilChanged()](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/distinct-until-changed.html)
      gibi methodlari kullanarak hafifletmenin gerekli olabileceği anlamına gelir.

### Consume UI State

UI’deki UiState objelerinin streamini tüketmek için, kullandığınız gözlemlenebilir veri türü için terminal operatorunu
kullanırsınız. Örneğin, LiveData için observe() methodunu, Kotlin flowlar için ise collect() methodunu veya onun
varyasyonlarını kullanırsınız.

UI’da observable data holder kullanırken, UI’in yaşam döngüsünü dikkate aldığınızdan emin olun. View kullanıcıya
gösterilmediğinde UI’ in state’ini gözlemlememesi gerektiğinden bu önemlidir. Bu konu hakkında daha fazla bilgi edinmek
için [bu blog](https://medium.com/androiddevelopers/a-safer-way-to-collect-flows-from-android-uis-23080b1f8bda)
gönderisine bakın. LifecycleOwner, LiveData'yı kullanırken implicit olarak yaşam döngüsü endişelerini giderir. Flowlari
kullanırken, bunu uygun coroutine kapsamı ve repeatOnLifecycle API'si ile halletmek en iyisidir:

```kotlin
//views
class NewsActivity : AppCompatActivity() {

    private val viewModel: NewsViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        ...

        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect {
                    // Update UI elements
                }
            }
        }
    }
}

//compose
@Composable
fun LatestNewsScreen(
    viewModel: NewsViewModel = viewModel()
) {
    // Show UI elements based on the viewModel.uiState
}
```

{: .note }
Not: Bu örnekte kullanılan belirli StateFlow objectleri, active
collectorleri olmadığında çalışmayı durdurmaz, ancak flowlarla çalışırken bunların nasıl uygulandığını
bilmeyebilirsiniz. Yaşam döngüsüne duyarlı flow collection kullanmak, daha sonra downstream collector kodunu tekrar
ziyaret etmeden ViewModel flowlarinda bu tür değişiklikleri yapmanızı sağlar.


### Show in-progress operations

Bir UiState sınıfında yükleme statelerini temsil etmenin basit bir yolu, bir boole alanı kullanmaktır:

```kotlin
data class NewsUiState(
    val isFetchingArticles: Boolean = false,
    ...
)
```

Bu flag’in değeri, UI’de bir progress bar’in varlığını veya yokluğunu temsil eder.

```kotlin
//views
class NewsActivity : AppCompatActivity() {

    private val viewModel: NewsViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        ...

        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                // Bind the visibility of the progressBar to the state
                // of isFetchingArticles.
                viewModel.uiState
                    .map { it.isFetchingArticles }
                    .distinctUntilChanged()
                    .collect { progressBar.isVisible = it }
            }
        }
    }
}

//compose
@Composable
fun LatestNewsScreen(
    modifier: Modifier = Modifier,
    viewModel: NewsViewModel = viewModel()
) {
    Box(modifier.fillMaxSize()) {

        if (viewModel.uiState.isFetchingArticles) {
            CircularProgressIndicator(Modifier.align(Alignment.Center))
        }

        // Add other UI elements. For example, the list.
    }
}
```

### Show errors on the screen

UI’de hataların gösterilmesi, devam eden işlemleri göstermeye benzer çünkü her ikisi de varlıklarını veya yokluklarını
gösteren boolean değerlerle kolayca temsil edilir. Ancak hatalar, kullanıcıya geri iletmek için ilişkili bir mesajı veya
başarısız işlemi yeniden deneyen bunlarla ilişkili bir action de içerebilir. Bu nedenle, devam eden bir işlem
yüklenirken veya yüklenmezken, hata durumlarının, hatanın bağlamına uygun meta verileri barındıran data classlar ile
modellenmesi gerekebilir.

Örneğin, makaleler getirilirken bir progress bar gösteren önceki bölümdeki örneği ele alalım. Bu işlem bir hatayla
sonuçlanırsa, kullanıcıya neyin yanlış gittiğini açıklayan bir veya daha fazla mesaj görüntülemek isteyebilirsiniz.

```kotlin
data class Message(val id: Long, val message: String)

data class NewsUiState(
    val userMessages: List<Message> = listOf(),
    ...
)
```

Hata mesajları daha sonra kullanıcıya [snackbar](https://material.io/components/snackbars/android) gibi UI elementleri
biçiminde sunulabilir. Bu, UI eventlerinin nasıl
üretildiği ve tüketildiği ile ilgili olduğundan, daha fazla bilgi edinmek için [UI eventleri](ui-events) sayfasına
bakın.

### Threading and concurrency

Bir ViewModel'de gerçekleştirilen herhangi bir çalışma, main threadden çağrı yapmak için main-safe olmalıdır. Bunun
nedeni, işi farklı bir threade taşımaktan data ve domain layerlarin sorumlu olmasıdır.
Bir ViewModel uzun süren işlemler gerçekleştiriyorsa, bu logici bir background threade taşımaktan da sorumludur. Kotlin
coroutineler, concurrent operasyonlari yönetmenin harika bir yoludur ve Jetpack Architecture Components, bunlar için
yerleşik destek sağlar. Android uygulamalarında coroutineleri kullanma hakkında daha fazla bilgi edinmek için bkz.
[Kotlin coroutines on Android](https://developer.android.com/kotlin/coroutines).

### Navigation

Uygulama navigationdaki değişiklikler genellikle event benzeri yayilmalardan(emission) kaynaklanır. Örneğin, bir
SignInViewModel sınıfı bir oturum açma gerçekleştirdikten sonra, UiState'in isSignedIn fieldi true olarak ayarlanmış
olabilir. Bunun gibi tetikleyiciler, consumption implementasyonunun Navigation componentini ertelemesi dışında,
yukarıdaki Consume UI State bölümünde kapsananlar gibi tüketilmelidir.

### Paging

Paging library, UI’de PagingData adlı bir tip ile tüketilir. PagingData, zaman içinde değişebilen öğeleri temsil
ettiğinden ve içerdiğinden - başka bir deyişle immutable bir tip değildir - sabit bir UI state’inde temsil
edilmemelidir. Bunun yerine, ViewModel'den bağımsız olarak kendi akışında göstermelisiniz. Bunun belirli bir örneği için
Android Paging codelab bakın.

### Animations

Akıcı ve sorunsuz top level navigation transitions sağlamak için, animasyona başlamadan önce ikinci ekranın veri
yüklemesini beklemek isteyebilirsiniz. Android view framework, [postponeEnterTransition()](https://developer.android.com/reference/androidx/fragment/app/Fragment#postponeEnterTransition()) ve
[startPostponedEnterTransition()](https://developer.android.com/reference/androidx/fragment/app/Fragment#startPostponedEnterTransition())  API'leri ile fragment destinationslari arasındaki geçişleri(transitions) geciktirmek
için hook sağlar. Bu API'ler, ikinci ekrandaki UI elementleri (genellikle ağdan getirilen bir görüntü), kullanıcı
arabirimi bu ekrana geçişi animate etmeden önce görüntülenmeye hazır olmasını sağlamanın bir yolunu sağlar. Daha fazla
ayrıntı ve uygulama özellikleri için [Android Motion](https://github.com/android/animation-samples/tree/main/Motion) örneğine bakın.

[Samples](https://developer.android.com/topic/architecture/ui-layer#samples)
