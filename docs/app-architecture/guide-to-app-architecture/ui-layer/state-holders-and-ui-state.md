---
layout: default
title: State holders and UI state
grand_parent: Guide to app architecture
nav_order: 3
parent: Ui layer
---

#### State holders and UI state

[UI layer](ui-layer) kılavuzu, UI katmanı için UI State oluşturma ve yönetme aracı olarak tek yönlü veri akışını (UDF)
tartışır.
![Undirectiona data flow](/assets/images/state-holders-and-ui-state-img1.png)

Ayrıca, UDF yönetimini state holder adı verilen özel bir sınıfa devretmenin faydalarını da vurgular. Bir state holderi
ViewModel veya düz bir sınıf aracılığıyla uygulayabilirsiniz. Bu dokümantasyon, state holderleri ve UI katmanında
oynadıkları role daha yakından bakıyor.
Bu belgenin sonunda, UI katmanında application state’inin nasıl yönetileceğini anlamalısınız; bu, UI state production
pipelinedir. Aşağıdakileri anlayabilmeli ve bilmelisiniz:

* UI katmanında bulunan UI state türlerini anlayın.
* UI katmanındaki bu UI stateleri üzerinde çalışan logic türlerini anlayın.
* ViewModel veya basit bir sınıf gibi bir state holderin uygun implementasyonunu nasıl seçeceğinizi öğrenin.

[State holders and state production in the UI Layer](https://youtu.be/pCX9wvu-Bq0)

#### Elements of the UI state production pipeline

UI state ve onu üreten logic, UI katmanını tanımlar.

##### UI state

[UI state](about-the-ui-layer.md#define-ui-state), UI'yi tanımlayan propertydir. İki tür UI state vardır:

* Screen UI State, ekranda görüntülemeniz gereken şeydir. Örneğin, bir NewsUiState sınıfı, UI oluşturmak için gereken
  haber makalelerini ve diğer bilgileri içerebilir. Bu state, uygulama verilerini içerdiğinden genellikle hiyerarşinin
  diğer katmanlarıyla bağlantılıdır.
* UI element state, UI elementlerinin nasıl oluşturulduğunu etkileyen, onlara özgü propertyleri ifade eder. Bir UI
  elementi gösterilebilir veya gizlenebilir ve belirli bir yazı tipine, yazı tipi boyutuna veya yazı tipi rengine sahip
  olabilir. Androide, View, doğası gereği stateful olduğu için bu state’i kendisi yönetir ve state’i değiştirmek veya
  sorgulamak için methodlar sunar. Bunun bir örneği, text için TextView sınıfının get ve set methodlaridir. Jetpack
  Compose'da state composable olanın dışındadır ve hatta onu composable olanın hemen yakınından çağıran composable
  fonksiyona veya bir state holderine hoist edebilirsiniz. Bunun bir örneği, composable Scaffold için ScaffoldState'tir.

#### Logic

Uygulama verileri ve kullanıcı eventleri, UI state’inin zaman içinde değişmesine neden olduğundan, UI state statik bir
property değildir. Logic, UI state’inin hangi bölümlerinin değiştiği, neden değiştiği ve ne zaman değişmesi gerektiği
dahil olmak üzere değişikliğin özelliklerini belirler.
![Logic as the producer of UI state](/assets/images/state-holders-and-ui-state-img2.png)

Logic, business logic veya UI logic olabilir:

* Business logic, uygulama verileri için ürün gereksinimlerinin implemente edilmesidir. Örneğin, kullanıcı butona
  dokunduğunda bir haber okuyucu uygulamasında bir makaleye yer işareti koyma. Bir yer imini bir dosyaya veya
  veritabanına kaydetme logic’i genellikle domain veya data katmanlarına yerleştirilir. State holder genellikle bu
  logic’i, ortaya çıkardıkları metodları çağırarak bu katmanlara devreder.
* UI logic, UI state’inin ekranda nasıl görüntüleneceği ile ilgilidir. Örneğin, kullanıcı bir kategori seçtiğinde doğru
  arama çubuğu hint elde etmek, bir listede belirli bir öğeye kaydırma yapmak veya kullanıcı bir butona tıkladığında
  belirli bir ekrana navigate etme logic’i.

#### Android lifecycle and the types of UI state and logic

UI katmanının iki bölümü vardır: UI lifecycle’a biri bağımlı, diğeri bağımsız. Bu ayrım, her fragmentin kullanabileceği
veri kaynaklarını belirler ve bu nedenle farklı türde UI state ve logic gerektirir.

* UI yaşam döngüsünden bağımsız(UI lifecycle independent): UI katmanının bu kısmı, uygulamanın veri üreten
  katmanlarıyla (data veya domain katmanları) ilgilenir ve business logic tarafından tanımlanır. UI’deki lifecycle,
  configuration changes ve activity oluşturma, UI state production pipeline in etkin olup olmadığını etkileyebilir,
  ancak üretilen verilerin geçerliliğini etkilemez.

* UI yaşam döngüsüne bağlı(UI lifecycle dependent): UI katmanının bu kısmı, UI logici ile ilgilenir ve lifecycle veya
  configuration changeden doğrudan etkilenir. Bu değişiklikler, içinde okunan veri kaynaklarının geçerliliğini doğrudan
  etkiler ve sonuç olarak state ancak lifecycle aktif olduğunda değişebilir. Buna örnek olarak runtime izinleri ve
  localized stringler gibi yapılandırmaya bağlı kaynakların alınması dahildir.

Yukarıdakiler aşağıdaki tablo ile özetlenebilir:

| Ui Lifecycle independent | Ui Lifecycle dependent |
|--------------------------|------------------------|
| Business logic           | UI logic               |
| Screen UI state          |                        |

#### The UI state production pipeline

UI state production pipeline, UI state oluşturmak için atilan adımları ifade eder. Bu adımlar, daha önce tanımlanan
logic türlerinin uygulanmasını içerir ve tamamen UI gereksinimlerine bağlıdır. Bazı UI'ler, pipeline’in hem UI Lifecycle
independent hem de UI Lifecycle dependent bölümlerinden yararlanabilir veya hiçbirinden yararlanamaz.

Yani, UI katman pipeline’in aşağıdaki permütasyonları geçerlidir:

* UI’in kendisi tarafından üretilen ve yönetilen UI state. Örneğin, basit, yeniden kullanılabilir bir temel sayaç:

```kotlin
@Composable
fun Counter() {
    // The UI state is managed by the UI itself
    var count by remember { mutableStateOf(0) }
    Row {
        Button(onClick = { ++count }) {
            Text(text = "Increment")
        }
        Button(onClick = { --count }) {
            Text(text = "Decrement")
        }
    }
}
```

* UI logic → UI. Örneğin, kullanıcının bir listenin en üstüne atlamasına olanak tanıyan bir butonu göstermek veya
  gizlemek.

```kotlin
@Composable
fun ContactsList(contacts: List<Contact>) {
    val listState = rememberLazyListState()
    val isAtTopOfList by remember {
        derivedStateOf {
            listState.firstVisibleItemIndex < 3
        }
    }

    // Create the LazyColumn with the lazyListState
    ...

    // Show or hide the button (UI logic) based on the list scroll position
    AnimatedVisibility(visible = !isAtTopOfList) {
        ScrollToTopButton()
    }
}
```

* Business logic → UI. Geçerli kullanıcının fotoğrafını ekranda gösteren bir UI elementi.

```kotlin
@Composable
fun UserProfileScreen(viewModel: UserProfileViewModel = hiltViewModel()) {
    // Read screen UI state from the business logic state holder
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    // Call on the UserAvatar Composable to display the photo
    UserAvatar(picture = uiState.profilePicture)
}
```

* Business logic → UI logic → UI.Belirli bir UI state için ekranda doğru bilgileri görüntülemek üzere kayan bir UI
  element.

```kotlin
@Composable
fun ContactsList(viewModel: ContactsViewModel = hiltViewModel()) {
    // Read screen UI state from the business logic state holder
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    val contacts = uiState.contacts
    val deepLinkedContact = uiState.deepLinkedContact

    val listState = rememberLazyListState()

    // Create the LazyColumn with the lazyListState
    ...

    // Perform UI logic that depends on information from business logic
    if (deepLinkedContact != null && contacts.isNotEmpty()) {
        LaunchedEffect(listState, deepLinkedContact, contacts) {
            val deepLinkedContactIndex = contacts.indexOf(deepLinkedContact)
            if (deepLinkedContactIndex >= 0) {
                // Scroll to deep linked item
                listState.animateScrollToItem(deepLinkedContactIndex)
            }
        }
    }
}
```

UI state production pipeline her iki tür mantığın da uygulandığı durumda, business logic her zaman UI logicten önce
uygulanmalıdır. UI logicinden sonra busines logici uygulamaya çalışmak, business logicin UI logice bağlı olduğu anlamına
gelir. Aşağıdaki bölümler, farklı logic türlerine ve state holderlerine derinlemesine bir bakışla bunun neden bir sorun
olduğunu ele almaktadır.
![Application of logic in the UI layer](/assets/images/state-holders-and-ui-state-img3.png)

#### State holders and their responsibilities

State holderin sorumluluğu, uygulamanın okuyabilmesi için state’i saklamaktır. Logice ihtiyaç duyulan durumlarda
aracılık yaparak gerekli logici barındıran veri kaynaklarına erişim sağlar. Bu şekilde, state holder logici uygun veri
kaynağına devreder.

Bu, aşağıdaki faydaları sağlar:

* Basit UI’lar: UIsadece state’ini bağlar.
* Sürdürülebilirlik: State holderda tanımlanan logic, UI’in kendisi değiştirilmeden yinelenebilir.
* Test Edilebilirlik: UI ve state production logici bağımsız olarak test edilebilir.
* Okunabilirlik: Kodu okuyanlar, UI presentation kodu ile UI state production kodu arasındaki farkları açıkça görebilir.

Boyutu veya kapsamı ne olursa olsun, her UI elementinin karşılık gelen state holderi ile 1:1 ilişkisi vardır. Ayrıca,
bir state holderin, bir UI state değişikliği ile sonuçlanabilecek herhangi bir kullanıcı eylemini kabul edebilmesi ve
işleyebilmesi ve ardından gelen state değişikliğini üretebilmesi gerekir.

<mark style = "background-color: lightblue">Not: State holderler kesinlikle gerekli değildir. Basit UI’kar, logiclerini
presentation kodlarıyla inline olarak barındırabilir.</mark>

#### Types of state holders

UI state’i ve logic’i tiplerini benzer şekilde, UI katmanında, UI yaşam döngüsüyle olan ilişkilerine göre tanımlanan iki
tür state holder vardır:

* The business logic state holder.
* The UI logic state holder.

Aşağıdaki bölümlerde, business logic state holderinden başlayarak state holderlerin türlerine daha yakından
bakılmaktadır.

<mark style= "background-color: lightblue">Not: Bir UI logic state holderi, data veya domain katmanlarından gelen
bilgilere bağlıysa, bu bilgileri ona bir business logic state holderinden iletmelisiniz. Bunun nedeni, business logic
state holderinin, UI yaşam döngüsünden bağımsız olduğu için UI logici state holderinden daha uzun ömürlü
olmasıdır.</mark>

#### Business logic and its state holder

Business logic state holderlari, kullanıcı eventlerini handle eder ve verileri data veya domain katmanlarından ekran UI
state’ine dönüştürür. Android yaşam döngüsü ve uygulama configuration changes göz önünde bulundurulduğunda optimum
kullanıcı deneyimi sağlamak için business logic kullanan state holderlerini aşağıdaki özelliklere sahip olması gerekir:

| Property                                                                                     | Detail                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|----------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Produces UI State(UI State uretmek)                                                          | Business logic state holderlar, UI için UI state’i oluşturmaktan sorumludur. Bu UI state’i, genellikle kullanıcı eventlerinin handle edilmesiyle ve domain ve data katmanlarından verilerin okunmasının sonucudur.                                                                                                                                                                                                                                          |
| Retained through activity recreation(activitynin yeniden olusturulmasina karsi korumak)      | Business logic state holderlar, Activity’nin yeniden oluşturulması karsisinda state ve state processing pipelinelerini koruyarak sorunsuz bir kullanıcı deneyimi sağlamaya yardımcı olur. State holderin korunmadigi ve yeniden yaratıldığı durumlarda (genellikle işlem ölümünden sonra), state holderin tutarlı bir kullanıcı deneyimi sağlamak için son state’i kolayca yeniden oluşturabilmesi gerekir.                                                 |
| Possess long lived state(uzun omurlu state tutmak)                                           | Business logic state holderlari genellikle navigasyon destinationlari için state’i yönetmek amacila kullanılır. Sonuç olarak, genellikle navigasyon graphden kaldırılana kadar statelerini navigasyon değişikliklerinde korurlar.                                                                                                                                                                                                                           |
| Is unique to its UI and is not reusable(Kullanıcı arayüzüne özgüdür ve yeniden kullanılamaz) | Business logic state holderleri tipik olarak belirli bir uygulama fonksiyonaltesi icin üretilir, örneğin bir TaskEditViewModel veya bir TaskListViewModel için state üretir ve bu nedenle yalnızca o uygulama fonksiyonalitesi için geçerlidir. Aynı state holder, farklı form faktörlerinde bu uygulama fonksiyonalitelerini destekleyebilir. Örneğin, uygulamanın mobil, TV ve tablet sürümleri aynı business logic state holderini yeniden kullanabilir. |

<mark style="background-color: lightblue">Not: Business logic state holderlar; ViewModel instancelari yukarıda belirtilen özelliklerin birçoğunu, özellikle de Activity yeniden oluşturma sırasında hayatta kaldigi icin, tipik olarak bir ViewModel instance ile implement edilirler.</mark>

Örneğin, ["Now in Android"](https://github.com/android/nowinandroid) uygulamasında yazar navigation hedefini göz önünde bulundurun:

![Now in Android](/assets/images/now-in-android-img1.png)
Business logic state holder olarak hareket eden [AuthorViewModel](https://github.com/android/nowinandroid/blob/main/feature-author/src/main/java/com/google/samples/apps/nowinandroid/feature/author/AuthorViewModel.kt), bu durumda UI state’ini üretir:

```kotlin
@HiltViewModel
class AuthorViewModel @Inject constructor(
    savedStateHandle: SavedStateHandle,
    private val authorsRepository: AuthorsRepository,
    newsRepository: NewsRepository
) : ViewModel() {

    val uiState: StateFlow<AuthorScreenUiState> = …

    // Business logic
    fun followAuthor(followed: Boolean) {
      …
    }
}
```
AuthorViewModel'in daha önce belirtilen business logic state holder özelliklere sahip olduğuna dikkat edin:

| Property                                     |Detail|
|----------------------------------------------|---|
| Produces AuthorScreenUiState                 |The AuthorViewModel reads data from the AuthorsRepository and NewsRepository and uses that data to produce AuthorScreenUiState. It also applies business logic when the user wants to follow or unfollow an Author by delegating to the AuthorsRepository.|
| Has access to the data layer                 |An instance of AuthorsRepository and NewsRepository are passed to it in its constructor, allowing it to implement the business logic of following an Author.|
| Survives Activity recreation                 |Because it is implemented with a ViewModel, it will be retained across quick Activity recreation. In the case of process death, the SavedStateHandle object can be read from to provide the minimum amount of information required to restore the UI state from the data layer.|
| Possesses long lived state                   |The ViewModel is scoped to the navigation graph, therefore unless the author destination is removed from the nav graph, the UI state in the uiState StateFlow remains in memory. The use of the StateFlow also adds the benefit of making the application of the business logic that produces the state lazy because state is only produced if there is a collector of the UI state.|
| Is unique to its UI                          |The AuthorViewModel is only applicable to the author navigation destination and cannot be reused anywhere else. If there is any business logic that is reused across navigation destinations, that business logic must be encapsulated in a data- or domain-layer-scoped component.|



<mark style="background-color: lightblue">Not: ViewModel'i yalnızca destination düzeyinde UI ile kullanmalısınız. Bunları, UI’in arama çubukları veya chip grupları gibi yeniden kullanılabilir parçalarında kullanmamalısınız. Bu durumlarda düz sınıflar daha uygundur.</mark>

<mark style="background-color: red">Uyarı: ViewModel instancelarini diğer composable fonksiyonlara argüman olarak vermeyin. Bunu yapmak, composable fonksiyonu ViewModel türüyle birleştirerek daha az yeniden kullanılabilir ve test edilmesini ve önizlemesini zorlaştırır. Ayrıca, ViewModel instancesini yöneten net bir tek doğruluk kaynağı (SSOT) olmayacaktır. ViewModel'i devre dışı bırakmak, birden çok composable öğenin ViewModel fonksiyonlarini çağırmasına ve state’i değiştirmesine izin vererek hataların debugini zorlaştırır. Bunun yerine, UDF best practicelerini izleyin ve yalnızca gerekli state’i iletin. Aynı şekilde, yayılan eventleri ViewModel'in composable SSOT'sine ulaşana kadar iletin. Eventi handle eden ve karşılık gelen ViewModel methodlarini çağıran SSOT budur.
</mark>

#### The ViewModel as a business logic state holder
ViewModels'in Android geliştirmedeki faydaları, onları business logice erişim sağlamak ve uygulama verilerini ekranda presentation için hazırlamak için uygun hale getirir. Bu faydalar aşağıdakileri içerir:
* ViewModels tarafından tetiklenen işlemler configuration changelerden kurtulur

* Navigasyon ile Entegrasyon saglar; 
  * Navigasyon, ekran backstackde iken ViewModels'i önbelleğe(cacheler) alır. Bu, destinationunuza döndüğünüzde önceden yüklenmiş verilerinizin anında kullanılabilir olması açısından önemlidir. Bu, composable ekranın yaşam döngüsünü observe eden bir state holder ile yapılması daha zor bir şeydir. 
  * ViewModel, hedef backstackden çıkarıldığında da temizlenir ve state’inizin otomatik olarak temizlenmesini sağlar. Bu, yeni bir ekrana gitme, bir configuration change nedeniyle veya başka nedenler gibi birçok nedenden dolayı meydana gelebilecek composable imhayı dinlemekten farklıdır.

* Hilt gibi diğer Jetpack library ile entegrasyon saglar.

<mark style="background-color: lightblue">Not: ViewModel avantajları kullanım durumunuz için geçerli değilse veya işleri farklı bir şekilde yapıyorsanız, ViewModel'in sorumluluklarını düz state holder sınıflara taşıyabilirsiniz.</mark>

#### UI logic and state holder

UI logic, UI'nin kendisinin sağladığı veriler üzerinde çalışan logictir. Bu, UI elementlerinin state’inde veya permissions API'si veya Resources gibi UI data kaynaklarında olabilir. UI logicini kullanan state holderlar tipik olarak aşağıdaki özelliklere sahiptir:
* UI state’ini üretir ve UI elementlerinin state’ini yönetir.
* Activity’i yeniden olusturma durumunda hayatta kalmaz; UI logicinde barındırılan state holderlar, genellikle UI'nin kendisinden gelen veri kaynaklarına bağımlıdır ve bu bilgileri yapılandırma değişiklikleri boyunca tutmaya çalışmak, genellikle bir bellek sızıntısına neden olur. State holderlar, yapılandırma değişikliklerinde devam etmek için verilere ihtiyaç duyarsa, hayatta kalan Aktivite yeniden olusturulmasina daha uygun başka bir componente yetki vermeleri gerekir. Örneğin, Jetpack Compose'da, remembered fonksiyonlarla oluşturulan Composable UI element stateleri, Activity yeniden olusturulmasi boyunca state’i korumak için genellikle rememberSaveable'a yetki verir. Bu tür fonksiyonlarin örnekleri arasında, rememberScaffoldState() ve rememberLazyListState() bulunur.
* UI scopeindaki veri kaynaklarına referansları vardır: UI logic state holder, UI ile aynı yaşam döngüsüne sahip olduğundan, yaşam döngüsü API'leri ve Resources gibi veri kaynaklarına güvenle başvurulabilir ve okunabilir.
* Birden çok UI’de yeniden kullanılabilir: Aynı UI logic state holderinin farklı instancelari, uygulamanın farklı bölümlerinde yeniden kullanılabilir. Örneğin, bir chip grubu için kullanıcı input eventlerini yönetmek için bir state holder, filter chipleri için bir arama sayfasında ve ayrıca bir e-posta alıcıları için "to" fieldi için kullanılabilir.


UI logic state holder, tipik olarak düz bir sınıfla uygulanır. Bunun nedeni, UI logic state holderinin oluşturulmasından UI'nin kendisinin sorumlu olması ve UI logic state holderinin, UI'nin kendisi ile aynı yaşam döngüsüne sahip olmasıdır. Örneğin Jetpack Compose'da state holder, Composition’un bir parçasıdır ve Composition’un yaşam döngüsünü takip eder.

<mark style="background-color: lightblue">Not: Düz sınıf state holderleri, UI logici, UI'den taşınacak kadar karmaşık olduğunda kullanılır. Aksi takdirde, UI logici, UI'de inline olarak uygulanabilir.</mark>
Now in Android örneğindeki aşağıdaki örnekte gösterilebilir:
![The now in android sample app](/assets/images/now-in-android-img2.png)
Now in Android örneği, cihazın ekran boyutuna bağlı olarak navigtion için bir bottom appbar veya bir navigation raili gösterir. Daha küçük ekranlar alttaki appbari ve daha büyük ekranlar navigation rail kullanır.
NiaApp composable fonksiyonunda kullanılan uygun navigation UI elementine karar verme logici, business logice bağlı olmadığından, [NiaAppState](https://github.com/android/nowinandroid/blob/main/app/src/main/java/com/google/samples/apps/nowinandroid/ui/NiaAppState.kt) adlı düz bir sınıf state holder tarafından yönetilebilir:
```kotlin
@Stable
class NiaAppState(
    val navController: NavHostController,
    val windowSizeClass: WindowSizeClass
) {

    // UI logic
    val shouldShowBottomBar: Boolean
        get() = windowSizeClass.widthSizeClass == WindowWidthSizeClass.Compact ||
            windowSizeClass.heightSizeClass == WindowHeightSizeClass.Compact

    // UI logic
    val shouldShowNavRail: Boolean
        get() = !shouldShowBottomBar

   // UI State
    val currentDestination: NavDestination?
        @Composable get() = navController
            .currentBackStackEntryAsState().value?.destination

    // UI logic
    fun navigate(destination: NiaNavigationDestination, route: String? = null) { /* ... */ }

     /* ... */
}
```

Yukarıdaki örnekte, NiaAppState ile ilgili aşağıdaki ayrıntılar dikkat çekicidir:

* Activity yeniden olustugunda hayatta kalamaz: NiaAppState, Compose adlandırma kurallarına uygun bir Composable fonksiyonu ile [rememberNiaAppState](https://github.com/android/nowinandroid/blob/main/app/src/main/java/com/google/samples/apps/nowinandroid/ui/NiaAppState.kt#L46) oluşturularak Compositionda hatırlanır. Activity yeniden oluşturulduktan sonra, önceki instancelar kaybolur ve yeniden oluşturulan Activity'nin yeni yapılandırmasına uygun olarak tüm bağımlılıkları iletilmiş yeni bir instance oluşturulur. Bu bağımlılıklar yeni olabilir veya önceki yapılandırmadan geri yüklenebilir. Örneğin, rememberNavController(), niaAppState constructorunda kullanılır ve Activity yeniden olusturulmasi boyunca state’i korumak için rememberSaveable'a yetki verir.
* UI kapsamlı veri kaynaklarına referansları vardır: NavigationController, Resources ve diğer benzer yaşam döngüsü kapsamındaki tiplere yapılan refereanslar, aynı yaşam döngüsü kapsamını paylaştıklarından NiaAppState'te güvenle tutulabilir.

<mark style="background-color: lightblue">Not: Düz state holder sınıfları, arama çubukları veya chip grupları gibi yeniden kullanılabilir kullanıcı arabirimi parçaları için önerilir. Bu durumda ViewModels'i kullanmamalısınız çünkü bunlar en iyi navigation destination için state’i yönetmek ve business logice erişim için kullanılır.</mark>

#### Choose between a ViewModel and plain class for a state holder
Yukarıdaki bölümlerden, bir ViewModel ve bir düz sınıf state holder arasında seçim yapmak, UI state’ine uygulanan logice ve logicin üzerinde çalıştığı veri kaynaklarına iner.

<mark style="background-color: lightblue">Not: Çoğu uygulama, aksi takdirde düz sınıf state holderine yerleştirilebilecek olan UI logicini UI’in kendisinde inline olarak gerçekleştirmeyi seçer. Bu, basit durumlar için iyidir, ancak diğer durumlar için, logici düz bir sınıf state holderine çekerek okunabilirliği artırabilirsiniz
</mark>
Özetle, aşağıdaki diyagram, UI State’i production pipelinedaki state holderlerinin pozisyonunu gösterir:
![State holders in the UI State production pipeline. Arrows mean data flow.](/assets/images/state-holders-and-ui-state-img4.png)
Sonuç olarak, tüketildiği yere en yakın state holderlerini kullanarak UI state’i üretmelisiniz. Daha az resmi olarak, uygun ownershipligi surdururken state’i mümkün olduğunca düşük tutmalısınız. Business logice erişmeniz gerekiyorsa ve UI state’inin, Activity yeniden olusturulmasi genelinde bile bir ekrana gidilebildiği sürece devam etmesi gerekiyorsa, bir [ViewModel](/docs/app-architecture/architecture-components/ui-layer-libraries/lifecycle-aware-components/ViewModel/about-viewmodel), business logic state holder implement etmeniz için harika bir seçimdir. Daha kısa ömürlü UI state ve UI logic için, yaşam döngüsü yalnızca UI'ye bağlı olan düz bir sınıf yeterli olmalıdır.

#### State holders are compoundable
Bağımlılıklar eşit veya daha kısa bir ömre sahip olduğu sürece state holderlar diğer state holderlara bağımlı olabilir. Bunun örnekleri şunlardır:

* bir UI logic state holder başka bir UI logic state holder'a bağlı olabilir.
* bir screen level state holder, bir UI logic state holder'a bağlı olabilir.

Aşağıdaki kod parçacığı, Compose'un [DrawerState](https://developer.android.com/reference/kotlin/androidx/compose/material/DrawerState)'inin başka bir dahili state holder olan [SwipeableState](https://developer.android.com/reference/kotlin/androidx/compose/material/SwipeableState)'e nasıl bağlı olduğunu ve bir uygulamanın UI logic state holder'ının DrawerState'e nasıl bağlı olabileceğini göstermektedir:

```kotlin
@Stable
class DrawerState(/* ... */) {
  internal val swipeableState = SwipeableState(/* ... */)
  // ...
}

@Stable
class MyAppState(
  private val drawerState: DrawerState,
  private val navController: NavHostController
) { /* ... */ }

@Composable
fun rememberMyAppState(
  drawerState: DrawerState = rememberDrawerState(DrawerValue.Closed),
  navController: NavHostController = rememberNavController()
): MyAppState = remember(drawerState, navController) {
  MyAppState(drawerState, navController)
}
```
<mark style="background-color:red">Dikkat: Screen level state holderların bir ekranın veya ekranın bir kısmının business logic karmaşıklığını yönettiği göz önüne alındığında, bir screen level state holderın başka bir screen level state holdera bağlı olması mantıklı olmayacaktır. Bu senaryodaysanız, ekranlarınızı ve state holder'larınızı yeniden gözden geçirin ve ihtiyacınız olanın bu olduğundan emin olun.
</mark>

Bir state holder'dan daha uzun ömürlü bir bağımlılık örneği, bir screen level state holder'a bağlı olan bir UI logic state holder olabilir. Bu, daha kısa ömürlü state holder'ın yeniden kullanılabilirliğini azaltır ve gerçekte ihtiyaç duyduğundan daha fazla logic ve state'e erişmesini sağlar.

Daha kısa ömürlü state holder'ın daha yüksek scope'lu bir state holder'dan belirli bilgilere ihtiyacı varsa, state holder instance'ını geçmek yerine yalnızca ihtiyaç duyduğu bilgileri parametre olarak geçirin. Örneğin, aşağıdaki kod parçasında, UI logic state holder sınıfı, ViewModel instance'ının tamamını bir bağımlılık olarak geçirmek yerine ViewModel'den parametre olarak sadece ihtiyaç duyduğu bilgileri alır.
```kotlin
class MyScreenViewModel(/* ... */) {
  val uiState: StateFlow<MyScreenUiState> = /* ... */
  fun doSomething() { /* ... */ }
  fun doAnotherThing() { /* ... */ }
  // ...
}

@Stable
class MyScreenState(
  // DO NOT pass a ViewModel instance to a plain state holder class
  // private val viewModel: MyScreenViewModel,

  // Instead, pass only what it needs as a dependency
  private val someState: StateFlow<SomeState>,
  private val doSomething: () -> Unit,

  // Other UI-scoped types
  private val scaffoldState: ScaffoldState
) {
  /* ... */
}

@Composable
fun rememberMyScreenState(
  someState: StateFlow<SomeState>,
  doSomething: () -> Unit,
  scaffoldState: ScaffoldState = rememberScaffoldState()
): MyScreenState = remember(someState, doSomething, scaffoldState) {
  MyScreenState(someState, doSomething, scaffoldState)
}

@Composable
fun MyScreen(
  modifier: Modifier = Modifier,
  viewModel: MyScreenViewModel = viewModel(),
  state: MyScreenState = rememberMyScreenState(
    someState = viewModel.uiState.map { it.toSomeState() },
    doSomething = viewModel::doSomething
  ),
  // ...
) {
  /* ... */
}
```
Aşağıdaki diyagram, UI ile önceki kod parçacığının farklı state holder'ları arasındaki bağımlılıkları temsil etmektedir:
![Ui depending on the different state holders. Arrows mean dependencies.](/assets/images/state-holders-and-ui-state-img5.png)

#### [Samples](https://developer.android.com/topic/architecture/ui-layer/stateholders#samples)
