---
layout: default
title: Ui events
grand_parent: Guide to app architecture
nav_order: 2
parent: Ui layer
---

## UI events

UI eventleri, UI layerda UI veya ViewModel tarafından işlenmesi gereken actionlardir. En yaygın event türü, kullanıcı
eventleridir. Kullanıcı, uygulamayla etkileşim kurarak, örneğin ekrana dokunarak veya hareketler(gesture) oluşturarak
kullanıcı eventleri oluşturur. UI daha sonra onClick() listenerlar gibi callbackleri kullanarak bu olayları tüketir(
consume eder).

<mark style = "background-color: lightblue"> Anahtar terimler: </mark>

* Kullanıcı Arabirimi(UI): Kullanıcı arabirimini yöneten View tabanlı kod veya Compose kodu.
* UI eventleri(Kullanici arabirimi olaylari): UI layerda handle edilmesi gereken gereken actionlar.
* Kullanıcı eventlari(User events): Kullanıcının uygulama ile etkileşim kurarken oluşturduğu eventler.

ViewModel normalde belirli bir kullanıcı eventinin business logicini handle etmekten sorumludur; örneğin, kullanıcının
bazı verileri yenilemek için bir butona tıklaması. Genellikle ViewModel, UI’in çağırabileceği methodlari göstererek bunu
halleder. Kullanıcı eventlari, UI’in doğrudan handle edebilecegi UI behavior logicine de sahip olabilir; örneğin, farklı
bir ekrana gitme veya bir Snackbar gösterme.

Farklı mobil platformlarda veya form factorlerinde aynı uygulama için business logic aynı kalırken, UI behaviour logic’i
bu durumlar arasında farklılık gösterebilen bir implementasyon detayıdır. [UI layer sayfası](ui-layer), bu logic
türlerini aşağıdaki gibi tanımlar:

* İş mantığı(business logic), state değişiklikleriyle ne yapılacağını ifade eder; örneğin, ödeme yapmak veya kullanıcı
  tercihlerini saklamak. Domain ve data layerlar genellikle bu logic’i handle eder. Bu kılavuz
  boyunca, [Architecture Components ViewModel sınıfı](/docs/app-architecture/architecture-components/ui-layer-libraries/lifecycle-aware-components/ViewModel/about-viewmodel),
  business logic’i handle eden sınıflar için düşünülmüş bir çözüm olarak kullanılır.

* UI davranış mantığı veya UI mantığı(UI behavior logic veya UI logic), state değişikliklerinin nasıl görüntüleneceğini
  ifade eder; örneğin, navigation logic’i veya kullanıcıya mesajların nasıl gösterileceği. UI bu logic’i handle eder.

<mark style = "background-color: lightblue"> Not: Bu sayfada sunulan öneriler ve best practiceler, geniş bir uygulama yelpazesine uygulanarak ölçeklenebilir, kaliteyi ve sağlamlığı artırabilir ve test edilmesini kolaylaştırabilir. Ancak, bunları kılavuz olarak ele almalı ve gereksinimlerinize göre uyarlamalısınız.
</mark>

[Architecture: Handling UI events-MAD Skills](https://youtu.be/lwGtp0Yr0PE)

### UI event decision tree

Aşağıdaki diyagram, belirli bir event kullanım senaryosunu(use case) ele almak için en iyi yaklaşımı bulmaya yönelik bir
karar ağacını(decision tree) göstermektedir. Bu kılavuzun geri kalanında bu yaklaşımlar ayrıntılı olarak
açıklanmaktadır.
![Eventleri handle etmek için karar ağacı(event decision tree).](/assets/images/decision-tree-handling-events.png)

### Handle user events

Bu eventler, bir UI elementinin state’inin (örneğin, expandable bir itemin state’i) değiştirilmesiyle ilgiliyse, UI,
kullanıcı eventlerini doğrudan handle edebilir. Event, ekrandaki verilerin yenilenmesi gibi business logicin
yürütülmesini gerektiriyorsa ViewModel tarafından işlenmelidir. Aşağıdaki örnek, UI elementini genişletmek (UI logic) ve
ekrandaki verileri yenilemek (business logic) için farklı butonlarin nasıl kullanıldığını gösterir:

```kotlin
// Views
class LatestNewsActivity : AppCompatActivity() {

    private lateinit var binding: ActivityLatestNewsBinding
    private val viewModel: LatestNewsViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        /* ... */

        // The expand details event is processed by the UI that
        // modifies a View's internal state.
        binding.expandButton.setOnClickListener {
            binding.expandedSection.visibility = View.VISIBLE
        }

        // The refresh event is processed by the ViewModel that is in charge
        // of the business logic.
        binding.refreshButton.setOnClickListener {
            viewModel.refreshNews()
        }
    }
}


//compose
@Composable
fun LatestNewsScreen(viewModel: LatestNewsViewModel = viewModel()) {

    // State of whether more details should be shown
    var expanded by remember { mutableStateOf(false) }

    Column {
        Text("Some text")
        if (expanded) {
            Text("More details")
        }

        Button(
                // The expand details event is processed by the UI that
                // modifies this composable's internal state.
                onClick = { expanded = !expanded }
        ) {
            val expandText = if (expanded) "Collapse" else "Expand"
            Text("$expandText details")
        }

        // The refresh event is processed by the ViewModel that is in charge
        // of the UI's business logic.
        Button(onClick = { viewModel.refreshNews() }) {
            Text("Refresh data")
        }
    }
}
```

### User Events in RecyclerViews

Action, bir RecyclerView iteminde veya custom bir Viewde olduğu gibi, UI ağacının daha aşağısında üretilirse, ViewModel
yine de kullanıcı eventlerini handle eden olmalıdır.

Örneğin, NewsActivity'den gelen tüm haber itemlerinin bir bookmark butonu içerdiğini varsayalım. ViewModel'in bookmark
eklenmiş haber iteminin IDsini bilmesi gerekir. Kullanıcı bir haber öğesine bookmark koyduğunda, RecyclerView adapteri,
ViewModel'e bağımlılık gerektirecek şekilde ViewModel'den açığa çıkan addBookmark(newsId) methodunu çağırmaz. Bunun
yerine ViewModel, eventi handle etmek için implementasyonu içeren NewsItemUiState adlı bir state nesnesini gösterir:

```kotlin
data class NewsItemUiState(
        val title: String,
        val body: String,
        val bookmarked: Boolean = false,
        val publicationDate: String,
        val onBookmark: () -> Unit
)

class LatestNewsViewModel(
        private val formatDateUseCase: FormatDateUseCase,
        private val repository: NewsRepository
) {
    val newsListUiItems = repository.latestNews.map { news ->
        NewsItemUiState(
                title = news.title,
                body = news.body,
                bookmarked = news.bookmarked,
                publicationDate = formatDateUseCase(news.publicationDate),
                // Business logic is passed as a lambda function that the
                // UI calls on click events.
                onBookmark = {
                    repository.addBookmark(news.id)
                }
        )
    }
}
```

Bu şekilde, RecyclerView adapteri yalnızca ihtiyaç duyduğu verilerle çalışır: NewsItemUiState nesnelerinin listesi.
Adapterin ViewModel'in tamamına erişimi yoktur, bu da ViewModel tarafından açığa çıkarılan fonksiyonaliteyi kötüye
kullanma olasılığını azaltır. Yalnızca activity sınıfının ViewModel ile çalışmasına izin verdiğinizde, sorumlulukları
ayırmış olursunuz. Bu, viewler veya RecyclerView adapterlari gibi UI’e özgü nesnelerin ViewModel ile doğrudan etkileşime
girmemesini sağlar.

<mark style="background-color: red">Uyarı: ViewModel'i RecyclerView adapterina passlamak kötü bir uygulamadır çünkü
adapteri ViewModel sınıfıyla sıkı bir şekilde birleştirir.</mark>

<mark style="background-color: lightblue">Not: Diğer bir yaygın pattern, RecyclerView adapterinin kullanıcı eylemleri(
actionlari) için bir callback interfaceine sahip olmasıdır. Bu durumda, activty veya fragment bindingi handle edebilir
ve doğrudan callback interfaceinden ViewModel methodlarini çağırabilir.</mark>

### Naming conventions for user event functions

Bu kılavuzda, kullanıcı eventlerini handle eden ViewModel fonksiyonlari, gerçekleştirdikleri eyleme göre bir fiille
adlandırılır; örneğin: addBookmark(id) veya logIn(username, password).

### Handle ViewModel Events

ViewModel'den kaynaklanan UI actionlari(ViewModel eventleri), her zaman bir [UI state](ui-layer) güncellemesiyle
sonuçlanmalıdır. Bu, Tek Yönlü Veri Akışı(UDF) ilkelerine uygundur. Configuration changesden sonra eventleri yeniden
üretilebilir hale getirir ve UI actionlarinin kaybolmayacağını garanti eder. İsteğe bağlı
olarak, [saved state modülü](https://developer.android.com/topic/libraries/architecture/viewmodel-savedstate)nü
kullanırsanız, eventleri process ölümünden sonra tekrarlanabilir hale getirebilirsiniz.

UI actionlarini UI state’ine maplemek her zaman basit bir işlem değildir, ancak daha basit bir logice yol açar. Örneğin,
düşünce süreciniz, UI’in belirli bir ekrana nasıl yönlendirileceğini belirlemekle bitmemelidir. Daha fazla düşünmeniz ve
bu kullanıcı akışını UI state’inizde nasıl temsil edeceğinizi düşünmeniz gerekir. Başka bir deyişle: UI’in yapması
gereken işlemleri düşünmeyin; bu actionlarinin UI state’ini nasıl etkilediğini düşünün.

<mark style="background-color:lightblue">Anahtar Nokta: ViewModel eventleri her zaman bir UI state güncellemesiyle
sonuçlanmalıdır.</mark>

Örneğin, kullanıcı oturum açma ekranında oturum açtığında ana ekrana gitme durumunu düşünün. Bunu UI state’inde
aşağıdaki gibi modelleyebilirsiniz:

```kotlin
data class LoginUiState(
        val isLoading: Boolean = false,
        val errorMessage: String? = null,
        val isUserLoggedIn: Boolean = false
)
```

Bu UI, isUserLoggedIn state’indeki değişikliklere tepki verir ve gerektiğinde doğru hedefe gider:

```kotlin
//views
class LoginViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(LoginUiState())
    val uiState: StateFlow<LoginUiState> = _uiState.asStateFlow()
    /* ... */
}

class LoginActivity : AppCompatActivity() {
    private val viewModel: LoginViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        /* ... */

        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect { uiState ->
                    if (uiState.isUserLoggedIn) {
                        // Navigate to the Home screen.
                    }
                    ...
                }
            }
        }
    }
}

//compose
class LoginViewModel : ViewModel() {
    var uiState by mutableStateOf(LoginUiState())
        private set
    /* ... */
}

@Composable
fun LoginScreen(
        viewModel: LoginViewModel = viewModel(),
        onUserLogIn: () -> Unit
) {
    val currentOnUserLogIn by rememberUpdatedState(onUserLogIn)

    // Whenever the uiState changes, check if the user is logged in.
    LaunchedEffect(viewModel.uiState) {
        if (viewModel.uiState.isUserLoggedIn) {
            currentOnUserLogIn()
        }
    }

    // Rest of the UI for the login screen.
}
```

<mark style="background-color: lightblue">Not: Bu bölümdeki kod
örnekleri [coroutinelerin ve bunların yaşam döngüsüne duyarlı bileşenlerle nasıl kullanılacağı](/docs/app-architecture/architecture-components/ui-layer-libraries/lifecycle-aware-components/use-kotlin-coroutines-with-lifecycle-aware-components)
nın anlaşılmasını gerektirir.</mark>

### Consuming events can trigger state updates

UI’de belirli ViewModel eventlerinin kullanılması, diğer UI state güncellemelerine neden olabilir. Örneğin, kullanıcıya
bir şey olduğunu bildirmek için ekranda geçici mesajlar gösterilirken, mesaj ekranda gösterildiğinde UI’in ViewModel'e
başka bir state güncellemesini tetiklemesi için bildirimde bulunması gerekir. Kullanıcı mesajı tükettiğinde (bırakarak
veya bir zaman aşımından sonra) meydana gelen event, "user input" olarak ele alınabilir ve bu nedenle, ViewModel bunun
farkında olmalıdır. Bu durumda, UI state aşağıdaki gibi modellenebilir:
```kotlin
// Models the UI state for the Latest news screen.
data class LatestNewsUiState(
    val news: List<News> = emptyList(),
    val isLoading: Boolean = false,
    val userMessage: String? = null
)
```
ViewModel, business logic kullanıcıya yeni bir geçici mesaj gösterilmesini gerektirdiğinde UI state’ini aşağıdaki gibi günceller:
```kotlin
//Views
class LatestNewsViewModel(/* ... */) : ViewModel() {

  private val _uiState = MutableStateFlow(LatestNewsUiState(isLoading = true))
  val uiState: StateFlow<LatestNewsUiState> = _uiState

  fun refreshNews() {
    viewModelScope.launch {
      // If there isn't internet connection, show a new message on the screen.
      if (!internetConnection()) {
        _uiState.update { currentUiState ->
          currentUiState.copy(userMessage = "No Internet connection")
        }
        return@launch
      }

      // Do something else.
    }
  }

  fun userMessageShown() {
    _uiState.update { currentUiState ->
      currentUiState.copy(userMessage = null)
    }
  }
}

//compose
class LatestNewsViewModel(/* ... */) : ViewModel() {

  var uiState by mutableStateOf(LatestNewsUiState())
    private set

  fun refreshNews() {
    viewModelScope.launch {
      // If there isn't internet connection, show a new message on the screen.
      if (!internetConnection()) {
        uiState = uiState.copy(userMessage = "No Internet connection")
        return@launch
      }

      // Do something else.
    }
  }

  fun userMessageShown() {
    uiState = uiState.copy(userMessage = null)
  }
}
```

ViewModel'in, UI’in mesajı ekranda nasıl gösterdiğini bilmesi gerekmez; sadece gösterilmesi gereken bir kullanıcı mesajı olduğunu bilir. Geçici mesaj gösterildikten sonra, UI’in bunu ViewModel'e bildirmesi gerekir ve başka bir UI state güncellemesinin userMessage propertysini temizlemesine neden olur:
```kotlin
//views
class LatestNewsActivity : AppCompatActivity() {
  private val viewModel: LatestNewsViewModel by viewModels()

  override fun onCreate(savedInstanceState: Bundle?) {
    /* ... */

    lifecycleScope.launch {
      repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.uiState.collect { uiState ->
          uiState.userMessage?.let {
            // TODO: Show Snackbar with userMessage.

            // Once the message is displayed and
            // dismissed, notify the ViewModel.
            viewModel.userMessageShown()
          }
          ...
        }
      }
    }
  }
}

//compose
@Composable
fun LatestNewsScreen(
        snackbarHostState: SnackbarHostState,
        viewModel: LatestNewsViewModel = viewModel(),
) {
  // Rest of the UI content.

  // If there are user messages to show on the screen,
  // show it and notify the ViewModel.
  viewModel.uiState.userMessage?.let { userMessage ->
    LaunchedEffect(userMessage) {
      snackbarHostState.showSnackbar(userMessage)
      // Once the message is displayed and dismissed, notify the ViewModel.
      viewModel.userMessageShown()
    }
  }
}
```
Mesaj geçici olsa da, UI state, zamanın her noktasında ekranda görüntülenenlerin aslına sadık bir temsilidir. Kullanıcı mesajı görüntülenir veya görüntülenmez.

<mark style="background-color: lightblue">Not: Ekranda gösterilecek kullanıcı mesajlarının listesini içeren daha gelişmiş bir kullanım örneği için [Jetsnack Compose](https://github.com/android/compose-samples/blob/main/Jetsnack/app/src/main/java/com/example/jetsnack/model/SnackbarManager.kt) örneğine bakın.
</mark>

### Navigation Events
[Consuming events can trigger state updates](#consuming-events-can-trigger-state-updates)  bölümü, kullanıcı mesajlarını ekranda görüntülemek için UI state’ini nasıl kullandığınızı ayrıntılarıyla açıklar. Navigasyon eventleri, bir Android uygulamasında da yaygın olarak görülen bir event türüdür.
Event, kullanıcı bir butona dokunduğu için UI’de tetiklenirse, UI, navigation controller çağırarak veya eventi çağırana uygun şekilde composable olarak göstererek bununla ilgilenir.

```kotlin
//views
class LoginActivity : AppCompatActivity() {

  private lateinit var binding: ActivityLoginBinding
  private val viewModel: LoginViewModel by viewModels()

  override fun onCreate(savedInstanceState: Bundle?) {
    /* ... */

    binding.helpButton.setOnClickListener {
      navController.navigate(...) // Open help screen
    }
  }
}

//compose
@Composable
fun LoginScreen(
        onHelp: () -> Unit, // Caller navigates to the right screen
        viewModel: LoginViewModel = viewModel()
) {
  // Rest of the UI

  Button(onClick = onHelp) {
    Text("Get help")
  }
}
```

Data input, navigateden önce bazı business logic doğrulaması gerektiriyorsa, ViewModel'in bu durumu UI’a göstermesi gerekir. Kullanıcı arayüzü bu state değişikliğine tepki verir ve buna göre navigate eder. [Handle ViewModel events section](#handle-viewmodel-events) bölümü bu kullanım durumunu kapsar. İşte benzer bir kod:

```kotlin
//views
class LoginActivity : AppCompatActivity() {
  private val viewModel: LoginViewModel by viewModels()

  override fun onCreate(savedInstanceState: Bundle?) {
    /* ... */

    lifecycleScope.launch {
      repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.uiState.collect { uiState ->
          if (uiState.isUserLoggedIn) {
            // Navigate to the Home screen.
          }
          ...
        }
      }
    }
  }
}

//compose
@Composable
fun LoginScreen(
        onUserLogIn: () -> Unit, // Caller navigates to the right screen
        viewModel: LoginViewModel = viewModel()
) {
  Button(
          onClick = {
            // ViewModel validation is triggered
            viewModel.login()
          }
  ) {
    Text("Log in")
  }
  // Rest of the UI

  val lifecycle = LocalLifecycleOwner.current.lifecycle
  val currentOnUserLogIn by rememberUpdatedState(onUserLogIn)
  LaunchedEffect(viewModel, lifecycle)  {
    // Whenever the uiState changes, check if the user is logged in and
    // call the `onUserLogin` event when `lifecycle` is at least STARTED
    snapshotFlow { viewModel.uiState }
            .filter { it.isUserLoggedIn }
            .flowWithLifecycle(lifecycle)
            .collect {
              currentOnUserLogIn()
            }
  }
}
```
Yukarıdaki örnekte, current destination olan Login backstackde tutulmayacağından uygulama beklendiği gibi çalışır. Kullanıcılar geri basarlarsa geri dönemezler. Ancak bunun olabileceği durumlarda, çözüm ek logic gerektirecektir.

### Navigation events when the destination is kept in the back stack

Bir ViewModel, A ekranından B ekranına bir navigation event üreten bir state belirlediğinde ve A ekranı navigation backstack’nde tutulduğunda, otomatik olarak B'ye ilerlemeye devam etmemek için ek logice ihtiyacınız olabilir. Bunu implement etmek için, UI’in diğer ekrana gitmeyi düşünüp düşünmemesi gerektiğini gösteren ek bir state’e sahip olunması gerekir.Normalde, bu state UI’de tutulur çünkü Navigation logic, ViewModel ile değil, UI ile ilgilidir. Bunu göstermek için, aşağıdaki kullanım örneğini ele alalım.

Diyelim ki uygulamanızın kayıt akışındasınız. Doğum tarihi doğrulama ekranında, kullanıcı bir tarih girdiğinde, kullanıcı "Continue" butonuna dokunduğunda tarih ViewModel tarafından doğrulanır. ViewModel, doğrulama logic’ini data katmanına devreder. Tarih geçerliyse, kullanıcı bir sonraki ekrana geçer. Ek bir özellik olarak, kullanıcılar bazı verileri değiştirmek istediklerinde farklı kayıt ekranları arasında gidip gelebilirler. Bu nedenle, kayıt akışındaki tüm destinationlar aynı backstackde tutulur. Bu gereksinimler göz önüne alındığında, bu ekranı aşağıdaki gibi uygulayabilirsiniz:

```kotlin
//views
// Key that identifies the `validationInProgress` state in the Bundle
private const val DOB_VALIDATION_KEY = "dobValidationKey"

class DobValidationFragment : Fragment() {

  private var validationInProgress: Boolean = false
  private val viewModel: DobValidationViewModel by viewModels()

  override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    val binding = // ...
            validationInProgress = savedInstanceState?.getBoolean(DOB_VALIDATION_KEY) ?: false

    binding.continueButton.setOnClickListener {
      viewModel.validateDob()
      validationInProgress = true
    }

    viewLifecycleOwner.lifecycleScope.launch {
      launch {
        viewModel.uiState
                .flowWithLifecycle(viewLifecycleOwner.lifecycle)
                .collect { uiState ->
                  // Update other parts of the UI ...

                  // If the input is valid and the user wants
                  // to navigate, navigate to the next screen
                  // and reset `validationInProgress` flag
                  if (uiState.isDobValid && validationInProgress) {
                    validationInProgress = false
                    navController.navigate(...) // Navigate to next screen
                  }
                }
      }
    }

    return binding
  }

  override fun onSaveInstanceState(outState: Bundle) {
    super.onSaveInstanceState(outState)
    outState.putBoolean(DOB_VALIDATION_KEY, validationInProgress)
  }
}

//compose
class DobValidationViewModel(/* ... */) : ViewModel() {
  var uiState by mutableStateOf(DobValidationUiState())
    private set
}

@Composable
fun DobValidationScreen(
        onNavigateToNextScreen: () -> Unit, // Caller navigates to the right screen
        viewModel: DobValidationViewModel = viewModel()
) {
  // TextField that updates the ViewModel when a date of birth is selected

  var validationInProgress by rememberSaveable { mutableStateOf(false) }

  Button(
          onClick = {
            viewModel.validateInput()
            validationInProgress = true
          }
  ) {
    Text("Continue")
  }
  // Rest of the UI

  /*
   * The following code implements the requirement of advancing automatically
   * to the next screen when a valid date of birth has been introduced
   * and the user wanted to continue with the registration process.
   */

  if (validationInProgress) {
    val lifecycle = LocalLifecycleOwner.current.lifecycle
    val currentNavigateToNextScreen by rememberUpdatedState(onNavigateToNextScreen)
    LaunchedEffect(viewModel, lifecycle) {
      // If the date of birth is valid and the validation is in progress,
      // navigate to the next screen when `lifecycle` is at least STARTED,
      // which is the default Lifecycle.State for the `flowWithLifecycle` operator.
      snapshotFlow { viewModel.uiState }
              .filter { it.isDobValid }
              .flowWithLifecycle(lifecycle)
              .collect {
                validationInProgress = false
                currentNavigateToNextScreen()
              }
    }
  }
}
```
Doğum tarihi doğrulama, ViewModel'in sorumlu olduğu business logictir. ViewModel çoğu zaman bu logic’i data katmanına devreder. Kullanıcıyı bir sonraki ekrana yönlendirme mantığı, UI logictir çünkü bu gereksinimler, UI yapılandırmasına bağlı olarak değişebilir. Örneğin, aynı anda birden çok kayıt adımı gösteriyorsanız, bir tablette otomatik olarak başka bir ekrana ilerlemek istemeyebilirsiniz. Yukarıdaki koddaki validationInProgress değişkeni bu işlevi uygular ve UI’in doğum tarihi geçerli olduğunda ve kullanıcı aşağıdaki kayıt adımına devam etmek istediğinde otomatik olarak navigate edilip edilmeyeceğini belirler.

### Other Use Cases
UI event usecaseinizin UI state güncellemeleriyle çözülemeyeceğini düşünüyorsanız, uygulamanızda veri akışını yeniden gözden geçirmeniz gerekebilir. Aşağıdaki ilkeleri göz önünde bulundurun:
* Her sınıf sorumlu olduğu şeyi yapmalı, daha fazlasını değil. Kullanıcı arabirimi, navigation calls, click events ve izin istekleri alma gibi ekrana özgü davranış mantığından sorumludur. ViewModel, iş mantığını içerir ve sonuçları hiyerarşinin alt katmanlarından UI state’ine dönüştürür.
* Eventin nereden kaynaklandığını düşünün. Bu kılavuzun başında sunulan decision tree’yi takip edin ve her sınıfın sorumlu olduğu konuyu halletmesini sağlayın. Örneğin, event kullanıcı arayüzünden geliyorsa ve bir navigate ile olayıyla sonuçlanıyorsa, o eventin kullanıcı arayüzünde handle edilmesi gerekir. Bazı logicler ViewModel'e devredilebilir, ancak eventin handle edilmesi tamamen ViewModel'e devredilemez.
* Birden çok consumer varsa ve activitynin birden çok kez consume edilmesinden endişe ediyorsanız uygulama mimarinizi yeniden gözden geçirmeniz gerekebilir. Birden fazla concurrent consumure sahip olmak, tam olarak bir kez teslim edilen sözleşmenin garanti edilmesinin son derece zor hale gelmesine neden olur, bu nedenle karmaşıklık ve subtle behavior miktarı patlar. Bu sorunu yaşıyorsanız, bu endişeleri kullanıcı arabirimi ağacınızda yukarıya taşımayı düşünün; hiyerarşide daha üstte yer alan farklı bir entitye ihtiyacınız olabilir.
* State’in ne zaman tüketilmesi gerektiğini bir düşünün. Belirli durumlarda, uygulama arka plandayken (örneğin, bir Toast gösterirken) state’i tüketmeye devam etmek istemeyebilirsiniz. Bu gibi durumlarda, kullanıcı arabirimi ön planda olduğunda state’i kullanmayı düşünün.

<mark style = "background-color: lightblue">Not: Bazı uygulamalarda, ViewModel eventlerinin [Kotlin Channels](https://kotlinlang.org/docs/channels.html) veya diğer reactive streams kullanılarak kullanıcı arayüzüne maruz kaldığını görmüş olabilirsiniz. Producer (ViewModel) consumeri (UI—Compose veya Views) geride bıraktığında, bu çözümler bu eventlerin teslimini ve işlenmesini garanti etmez. Bu, geliştirici için gelecekte sorunlara neden olabilir ve aynı zamanda çoğu uygulama için kabul edilemez bir kullanıcı deneyimidir çünkü bu, uygulamayı tutarsız bir durumda bırakabilir, hatalara neden olabilir veya kullanıcı kritik bilgileri kaçırabilir. Bu durumlardan birindeyseniz, o tek seferlik ViewModel eventinin kullanıcı arayüzünüz için gerçekte ne anlama geldiğini tekrar düşünün. Bunları hemen handle edin ve UI state’ine indirin. UI state’i, UI'yi belirli bir zamanda daha iyi temsil eder, size daha fazla teslimat ve işleme garantisi verir, test edilmesi genellikle daha kolaydır ve uygulamanızın geri kalanıyla tutarlı bir şekilde entegre olur.
Bazı kod örnekleriyle yukarıda bahsedilen API'leri neden kullanmamanız gerektiği hakkında daha fazla bilgi edinmek için [ViewModel: One-off event antipatterns](https://medium.com/androiddevelopers/viewmodel-one-off-event-antipatterns-16a1da869b95) blog gönderisini okuyun.
</mark>

### [Samples](https://developer.android.com/topic/architecture/ui-layer/events#samples)