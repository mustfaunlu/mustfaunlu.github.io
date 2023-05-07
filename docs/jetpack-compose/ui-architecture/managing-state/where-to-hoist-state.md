---
layout: default
title: Where to Hoist State
parent: Managing State
grand_parent: UI Architecture
nav_order: 2
---
# Where to Hoist State
Bir Compose uygulamasında, [UI state](/docs/app-architecture/guide-to-app-architecture/ui-layer/about-the-ui-layer/#define-ui-state)'ini nereye hoist ettiğiniz, UI logic veya business logic'in bunu gerektirip gerektirmediğine bağlıdır. Bu belge, bu iki ana senaryoyu ortaya koymaktadır.
                                      
## Best Practice
UI state'ini, onu okuyan ve yazan tüm composable'lar arasındaki en düşük ortak ataya hoist etmelisiniz. State'i tüketildiği yere en yakın yerde tutmalısınız. State sahibinden, tüketicilere immutable state'i ve state'i değiştirmek için event'leri gösterin.

En düşük ortak ata Composition'ın dışında da olabilir. Örnek olarak, iş mantığı söz konusu olduğu için bir ViewModel'de state'i hoisting ederken.

Bu sayfa, bu en iyi pratiği ve akılda tutulması gereken bir uyarıyı ayrıntılı olarak açıklamaktadır.

[Where to hoist that state in Compose](https://youtu.be/hWwZ_AuSGfo)

## Types of UI state and UI logic
Aşağıda, bu belge boyunca kullanılan UI state ve logic türleri için tanımlar bulunmaktadır.

### UI State
[UI state](/docs/app-architecture/guide-to-app-architecture/ui-layer/about-the-ui-layer/#define-ui-state), UI'yi tanımlayan property'dir. İki tür UI state vardır:

- `Ekran UI state`'i, ekranda görüntülemek için ihtiyaç duyduğunuz şeydir. Örneğin, bir NewsUiState sınıfı, haber makalelerini ve UI'yi oluşturmak için gereken diğer bilgileri içerebilir. Bu state, uygulama verilerini içerdiği için genellikle hiyerarşinin diğer katmanlarıyla bağlantılıdır.

- `UI öğesi state`'i, nasıl oluşturulduklarını etkileyen UI öğelerine özgü propertyleri ifade eder. Bir UI öğesi gösterilebilir veya gizlenebilir ve belirli bir yazı tipine, yazı tipi boyutuna veya yazı tipi rengine sahip olabilir. Android View'lerde View, doğası gereği stateful olduğu için bu state'i kendisi yönetir ve state'ini değiştirmek veya sorgulamak için metotlar sunar. Buna örnek olarak TextView sınıfının metin için get ve set metotları verilebilir. Jetpack Compose'da state, composable'ın dışındadır ve hatta onu composable'ın yakın çevresinden çağıran composable fonksiyonuna veya bir state holder'a çekebilirsiniz. Bunun bir örneği, [Scaffold](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary#Scaffold(androidx.compose.ui.Modifier,androidx.compose.material.ScaffoldState,kotlin.Function0,kotlin.Function0,kotlin.Function1,kotlin.Function0,androidx.compose.material.FabPosition,kotlin.Boolean,kotlin.Function1,kotlin.Boolean,androidx.compose.ui.graphics.Shape,androidx.compose.ui.unit.Dp,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,kotlin.Function1)) composable için [ScaffoldState](https://developer.android.com/reference/kotlin/androidx/compose/material/ScaffoldState)'tir.
### Logic
Bir uygulamadaki mantık, iş mantığı veya UI mantığı olabilir:

- `Business logic`, uygulama verileri için ürün gereksinimlerinin uygulanmasıdır. Örneğin, kullanıcı düğmeye dokunduğunda bir haber okuyucu uygulamasında bir makalenin yer imlerine eklenmesi. Yer imini bir dosyaya veya veritabanına kaydetmeye yönelik bu mantık genellikle domain veya data katmanlarına yerleştirilir. State holder genellikle bu mantığı, ortaya çıkardıkları metotları çağırarak bu katmanlara devreder.

- `UI logic`, UI statinin ekranda nasıl görüntüleneceği ile ilgilidir. Örneğin, kullanıcı bir kategori seçtiğinde doğru arama çubuğu ipucunun elde edilmesi, bir listede belirli bir öğeye kaydırma veya kullanıcı bir düğmeye tıkladığında belirli bir ekrana navigasyon mantığı.

## UI logic    
[UI mantığı](/docs/app-architecture/guide-to-app-architecture/ui-layer/about-the-ui-layer/#types-of-logic)nın state'i okuması veya yazması gerektiğinde, state'i yaşam döngüsünü takip ederek UI'ye scope etmelisiniz. Bunu başarmak için, state'i composable bir fonksiyonda doğru seviyede hoist etmelisiniz. Alternatif olarak, bunu yine UI yaşam döngüsüne göre scopelandirilmis [düz bir state holder sınıfı](/docs/app-architecture/guide-to-app-architecture/ui-layer/state-holders-and-ui-state/#ui-logic-and-its-state-holder)nda da yapabilirsiniz.

Aşağıda her iki çözümün de açıklaması ve hangisinin ne zaman kullanılacağı yer almaktadır.

### Composables as state owner
UI logic ve UI öğesi state'inin composable'larda olması, state ve logic basitse iyi bir yaklaşımdır. State'inizi gerektiğinde bir composable veya hoist'e dahili olarak bırakabilirsiniz.

### No state hoisting needed
State'in hoisting edilmesi her zaman gerekli değildir. Başka bir composable'ın kontrol etmesi gerekmediğinde state bir composable'da dahili olarak tutulabilir. Bu kod parçasında, dokunulduğunda genişleyen ve daralan bir composable vardır:
```kotlin
@Composable
fun ChatBubble(
    message: Message
) {
    var showDetails by rememberSaveable { mutableStateOf(false) } // UI öğesi genişletme state'ini tanımlama

    ClickableText(
        text = AnnotatedString(message.content),
        onClick = { showDetails = !showDetails } // Basit UI mantığı uygulayın
    )

    if (showDetails) {
        Text(message.timestamp)
    }
}

```
showDetails değişkeni bu UI öğesinin dahili state'idir. Sadece bu composable içinde okunur ve değiştirilir ve ona uygulanan mantık çok basittir. Bu durumda state'i hoist etmek çok fazla fayda sağlamayacaktır, bu nedenle onu dahili olarak bırakabilirsiniz. Bunu yapmak, bu composable'ı genişletilmiş state'in sahibi ve tek doğruluk kaynağı haline getirir.

{: .note}
Anahtar Nokta: UI öğesi state'ini composable fonksiyonların içinde tutmak kabul edilebilir. Uyguladığınız state ve mantık basitse ve UI hiyerarşisinin diğer bölümleri state'e ihtiyaç duymuyorsa bu iyi bir çözümdür. Örneğin, bu durum genellikle animasyon state'i için geçerlidir.

### Hoisting within composables
UI öğesi state'inizi diğer composable'larla paylaşmanız ve UI mantığını farklı yerlerde uygulamanız gerekiyorsa, UI hiyerarşisinde daha yükseğe hoist edebilirsiniz. Bu aynı zamanda composable'larınızı daha yeniden kullanılabilir ve test edilmesi daha kolay hale getirir.

Aşağıdaki örnek, iki fonksiyonel parçayı uygulayan bir sohbet uygulamasıdır:

- JumpToBottom butonu mesaj listesini en alta kaydırır. Buton, liste state'i üzerinde UI mantığı gerçekleştirir.
- MessagesList listesi, kullanıcı yeni mesajlar gönderdikten sonra en alta kaydırılır. UserInput, liste state'i üzerinde UI mantığı gerçekleştirir.

![](https://developer.android.com/static/images/jetpack/compose/state-hoisting-chat.png)
Şekil 1. JumpToBottom butonu ve yeni mesajlarda aşağıya kaydırma özelliğine sahip sohbet uygulaması

Composable hiyerarşi aşağıdaki gibidir:
![](https://developer.android.com/static/images/jetpack/compose/state-hoisting-initial-tree.png)
Şekil 2. Chat Composable ağacı

[LazyColumn](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/package-summary#LazyColumn(androidx.compose.ui.Modifier,androidx.compose.foundation.lazy.LazyListState,androidx.compose.foundation.layout.PaddingValues,kotlin.Boolean,androidx.compose.foundation.layout.Arrangement.Vertical,androidx.compose.ui.Alignment.Horizontal,androidx.compose.foundation.gestures.FlingBehavior,kotlin.Boolean,kotlin.Function1)) state'i konuşma ekranına hoist edilir, böylece uygulama UI mantığını gerçekleştirebilir ve bunu gerektiren tüm composable'lardan state'i okuyabilir:
![](https://developer.android.com/static/images/jetpack/compose/state-hoisting-animated.gif)
Şekil 3. LazyColumn state'inin LazyColumn'dan ConversationScreen'e taşınması

Son olarak composablelar:
![](https://developer.android.com/static/images/jetpack/compose/state-hoisting-passing-state.png)
Şekil 4. ConversationScreen'e hoist edilmis LazyListState ile sohbet composable ağacı

Kod aşağıdaki gibidir:
```kotlin
@Composable
private fun ConversationScreen(/*...*/) {
    val scope = rememberCoroutineScope()

    val lazyListState = rememberLazyListState() // State ConversationScreen'e hoist edildi

    MessagesList(messages, lazyListState) // Ayni state'i MessagesList'de kullanin

    UserInput(
        onMessageSent = { // UI logic'i lazyListState üzerinde uygulayin
            scope.launch {
                lazyListState.scrollToItem(0)
            }
        },
    )
}

@Composable
private fun MessagesList(
    messages: List<Message>,
    lazyListState: LazyListState = rememberLazyListState() // LazyListState default deger aldi
) {

    LazyColumn(
        state = lazyListState // Hoist edilmis state'i LazyColumn'a geçirin
    ) {
        items(messages, key = { message -> message.id }) { item ->
            Message(/*...*/)
        }
    }

    val scope = rememberCoroutineScope()

    JumpToBottom(onClicked = {
        scope.launch {
            lazyListState.scrollToItem(0) // lazyListState'e uygulanan UI mantığı
        }
    })
}
```
LazyListState, uygulanması gereken UI mantığı için gerektiği kadar yükseğe hoist edilir. Composable bir fonksiyonda initialize edildiğinden, yaşam döngüsünü takip ederek Composition'da saklanır.

LazyListState'in MessagesList metodunda rememberLazyListState() varsayılan değeri ile tanımlandığını unutmayın. Bu, Compose'da yaygın bir patterndir. Composable'ları daha yeniden kullanılabilir ve esnek hale getirir. Daha sonra composable'ı uygulamanın state'i kontrol etmesi gerekmeyen farklı bölümlerinde kullanabilirsiniz. Bu genellikle bir composable'ı test ederken veya önizleme yaparken söz konusu olur. LazyColumn state'ini tam olarak bu şekilde tanımlar.

{: .note}
Kilit Nokta: State'i en düşük ortak ataya hoist edin ve ihtiyacı olmayan composable'lara geçirmekten kaçının.

![](https://developer.android.com/static/images/jetpack/compose/state-hoisting-lca.png)
Şekil 5. LazyListState için en düşük ortak ata ConversationScreen'dir


### Plain state holder class as state owner
Bir composable, bir UI öğesinin bir veya birden fazla state field'ını içeren karmaşık UI mantığı içerdiğinde, bu sorumluluğu düz bir [state holder](/docs/app-architecture/guide-to-app-architecture/ui-layer/state-holders-and-ui-state/#ui-logic-and-its-state-holder) sınıfı gibi state holder'lara devretmelidir. Bu, composable'ın mantığını izole olarak daha test edilebilir hale getirir ve karmaşıklığını azaltır. Bu yaklaşım, [seperation of concerns ilkesi](https://en.wikipedia.org/wiki/Separation_of_concerns)ni destekler: composable, UI öğelerini emit etmekten sorumludur ve state holder, UI mantığını ve UI öğesi state'ini içerir.

Düz state holder sınıfları, composable fonksiyonunuzu çağıranlara kullanışlı fonksiyonlar sağlar, böylece bu mantığı kendileri yazmak zorunda kalmazlar.

Bu düz sınıflar Composition içinde oluşturulur ve hatırlanır.[ Composable'ın yaşam döngüsü](/docs/jetpack-compose/ui-architecture/lifecycle/#lifecycle-of-composables)nü takip ettikleri için, Compose kütüphanesi tarafından sağlanan [rememberNavController()](https://developer.android.com/reference/kotlin/androidx/navigation/compose/package-summary#rememberNavController(kotlin.Array)) veya [rememberLazyListState](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/package-summary#rememberLazyListState(kotlin.Int,kotlin.Int))() gibi türleri alabilirler.

Bunun bir örneği, [LazyColumn](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/package-summary#LazyColumn(androidx.compose.ui.Modifier,androidx.compose.foundation.lazy.LazyListState,androidx.compose.foundation.layout.PaddingValues,kotlin.Boolean,androidx.compose.foundation.layout.Arrangement.Vertical,androidx.compose.ui.Alignment.Horizontal,androidx.compose.foundation.gestures.FlingBehavior,kotlin.Boolean,kotlin.Function1)) veya [LazyRow](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/package-summary#lazyrow)'un UI karmaşıklığını kontrol etmek için Compose'da uygulanan [LazyListState](https://cs.android.com/androidx/platform/frameworks/support/+/androidx-main:compose/foundation/foundation/src/commonMain/kotlin/androidx/compose/foundation/lazy/LazyListState.kt;l=82?q=LazyListState.kt&ss=androidx%2Fplatform%2Fframeworks%2Fsupport) düz state holder sınıfıdır.

```kotlin
// LazyListState.kt

@Stable
class LazyListState constructor(
    firstVisibleItemIndex: Int = 0,
    firstVisibleItemScrollOffset: Int = 0
) : ScrollableState {
    /**
     *   Suanki scroll pozisyonu icin holder sinifi
     */
    private val scrollPosition = LazyListScrollPosition(
        firstVisibleItemIndex, firstVisibleItemScrollOffset
    )

    suspend fun scrollToItem(/*...*/) { /*...*/ }

    override suspend fun scroll() { /*...*/ }

    suspend fun animateScrollToItem() { /*...*/ }
}
```
LazyListState, bu UI öğesi için scrollPosition değerini saklayan LazyColumn state'ini encapsulate eder. Ayrıca, örneğin belirli bir öğeye kaydırma yaparak kaydırma konumunu değiştirmek için metotlar sunar.

{: .note}
Not: Bu sınıf [Stable](https://developer.android.com/reference/kotlin/androidx/compose/runtime/Stable) olarak annotate edilmiştir. Compose'da stabilite hakkında daha fazla bilgi için [bu blog gönderisi](https://medium.com/androiddevelopers/jetpack-compose-stability-explained-79c10db270c8)ne göz atın.

Gördüğünüz gibi, bir composable'ın sorumluluklarını artırmak, bir state holder ihtiyacını artırır. Sorumluluklar UI mantığında ya da sadece takip edilmesi gereken state miktarında olabilir.

{: .note}
Not: Düz state holder sınıfları bir Activity veya proses yeniden oluşturulduktan sonra korumak istediğiniz state içeriyorsa, rememberSaveable kullanın ve bunun için özel bir Saver oluşturun.

Bir başka yaygın pattern de uygulamadaki root composable fonksiyonlarının karmaşıklığını ele almak için düz bir state holder sınıfı kullanmaktır. Navigasyon state'i ve ekran boyutlandırma gibi uygulama düzeyinde state'i enkapsüle etmek için böyle bir sınıf kullanabilirsiniz. Bunun tam bir açıklaması [UI logic and its state holder](/docs/app-architecture/guide-to-app-architecture/ui-layer/state-holders-and-ui-state/#ui-logic-and-its-state-holder) sayfasında bulunabilir.

## Business logic
Composable ve plain state holder sınıfları UI mantığı ve UI element state'inden sorumluysa, bir ekran seviyesi state holder aşağıdaki görevlerden sorumludur:

- Genellikle business ve data katmanları gibi hiyerarşinin diğer katmanlarında yer alan uygulamanın iş mantığına erişim sağlamak.

- Uygulama verilerini belirli bir ekranda sunum için hazırlamak, bu da ekran UI state'i haline gelir.


### ViewModels as state owner
AAC ViewModellerinin Android geliştirmedeki [faydaları](/docs/app-architecture/architecture-components/ui-layer-libraries/lifecycle-aware-components/ViewModel/about-viewmodel/#best-practices), onları iş mantığına erişim sağlamak ve uygulama verilerini ekranda sunum için hazırlamak için uygun hale getirir.

{: .note}
Kilit Nokta: ViewModel, belirli sorumlulukları olan bir state holder'ın sadece bir uygulama detayıdır. Projenizin modülünü Android bağımlılıklarından uzak tutmak istiyorsanız, uygulamayı farklı bağlamlarda değiştirilebilir hale getirmek için interfacelere güvenebilirsiniz. Örneğin, Android'e özgü modülünüzde ViewModel'i ve diğer modüllerde düz bir state holder sınıfı gibi daha basit platformdan bağımsız implementasyonları kullanabilirsiniz.

ViewModel'de UI state'ini hoist ettiğinizde, onu Composition'ın dışına taşırsınız.
![](https://developer.android.com/static/images/jetpack/compose/state-hoisting-vm.png)
Şekil 6. ViewModel'e hoist edilen State, Composition'ın dışında saklanır.

ViewModelleri Composition'ın bir parçası olarak saklanmaz. Framework tarafından sağlanırlar ve bir Activity, Fragment, navigasyon grafiği veya bir navigasyon grafiğinin hedefi olabilen bir ViewModelStoreOwner'a scope edilirler. [ViewModel scope](/docs/app-architecture/architecture-components/ui-layer-libraries/lifecycle-aware-components/ViewModel/viewmodel-scoping-apis/#viewmodel-scoing-apis)'ları hakkında daha fazla bilgi için dokümantasyonu inceleyebilirsiniz.

O halde ViewModel, UI state için doğruluk kaynağı ve en düşük ortak atadır.

### Screen UI state
Yukarıdaki tanımlara göre, ekran UI state'i business rules uygulanarak üretilir. Ekran seviyesi state holder'ın bundan sorumlu olduğu göz önüne alındığında, bu, ekran UI state'inin tipik olarak ekran seviyesi state holder'da, bu durumda bir ViewModel'de toplandığı anlamına gelir.

Bir sohbet uygulamasının ConversationViewModel'ini ve bu modelin [ekran UI state](/docs/app-architecture/guide-to-app-architecture/ui-layer/state-holders-and-ui-state/#ui-state)'ini ve bunu değiştirmeye yönelik olayları nasıl ortaya çıkardığını düşünün:

```kotlin
class ConversationViewModel(
    channelId: String,
    messagesRepository: MessagesRepository
) : ViewModel() {

    val messages = messagesRepository
        .getLatestMessages(channelId)
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5_000),
            initialValue = emptyList()
        )

    // Business logic
    fun sendMessage(message: Message) { /* ... */ }
}
```
Composable'lar ViewModel'de toplanan ekran UI state'ini kullanır. Business logic'e erişim sağlamak için ViewModel instance'ını ekran düzeyindeki composable'larınıza enjekte etmelisiniz.

{: .note}
Not: ViewModel instance'larını diğer composable'lara aktarmamalısınız. Daha fazla bilgi için [Architecture state holders](/docs/app-architecture/guide-to-app-architecture/ui-layer/state-holders-and-ui-state/#state-holders-and-their-responsibilities) belgelerine bakın.

Aşağıda, ekran düzeyinde bir composable'da kullanılan bir ViewModel örneği yer almaktadır. Burada, Composable ConversationScreen(), ViewModel'de hoist edilen ekran UI state'ini tüketir:

```kotlin
@Composable
private fun ConversationScreen(
    conversationViewModel: ConversationViewModel = viewModel()
) {

    val messages by conversationViewModel.messages.collectAsStateWithLifecycle()

    ConversationScreen(
        messages = messages,
        onSendMessage = { message: Message -> conversationViewModel.sendMessage(message) }
    )
}

@Composable
private fun ConversationScreen(
    messages: List<Message>,
    onSendMessage: (Message) -> Unit
) {

    MessagesList(messages, onSendMessage)
    /* ... */
}
```

{: .note}
Not: viewModel() fonksiyonunu kullanmak için build.gradle dosyanıza androidx.lifecycle:lifecycle-viewmodel-compose bağımlılığını ekleyin. Compose'daki [diğer kütüphanelerle çalışma](https://developer.android.com/jetpack/compose/libraries#viewmodel) belgelerimizde bu fonksiyon hakkında daha fazla bilgi edinin.


{: .note}
Not: ViewModel'ler sistem tarafından başlatılan süreç yeniden oluşturulduktan sonra korumak istediğiniz state içeriyorsa, bunu kalıcı hale getirmek için SavedStateHandle kullanın. Daha fazla bilgi için [UI State'lerini Kaydetme](/docs/app-architecture/architecture-components/ui-layer-libraries/lifecycle-aware-components/save-ui-states/) sayfasına bakın.

#### Property drilling
" Property drilling", verilerin iç içe geçmiş birkaç child component üzerinden okunacakları konuma aktarılması anlamına gelir.

Compose'da property drilling'in ortaya çıkabileceği tipik bir örnek, ekran seviyesi state holder'ı en üst seviyeye enjekte ettiğinizde ve state ve event'leri child composable'lara geçirdiğinizde ortaya çıkar. Bu durum ayrıca composable fonksiyon imzalarının overload'na neden olabilir.

Event'leri ayrı lambda parametreleri olarak göstermek fonksiyon imzasını overload edebilecek olsa da, composable fonksiyon sorumluluklarının ne olduğunun görünürlüğünü maksimize eder. Bir bakışta ne yaptığını görebilirsiniz.

Property drilling, state ve event'leri tek bir yerde kapsüllemek için sarmalayıcı sınıflar oluşturmak yerine tercih edilir çünkü bu, composable sorumlulukların görünürlüğünü azaltır. Sarmalayıcı sınıflara sahip olmadığınızda, composable'lara yalnızca ihtiyaç duydukları parametreleri aktarma olasılığınız da artar ki bu da [en iyi pratik](/docs/jetpack-compose/ui-architecture/architecture/#define-composable-parameters)tir.

Bu event'ler navigasyon event'leri ise aynı en iyi pratik geçerlidir, [navigasyon dokümanları](https://developer.android.com/jetpack/compose/navigation#nav-calls-best-practices)nda bu konuda daha fazla bilgi edinebilirsiniz.

Bir performans sorunu tespit ettiyseniz, state'in okunmasını ertelemeyi de seçebilirsiniz. Daha fazla bilgi edinmek için [performans dokümanları](https://developer.android.com/jetpack/compose/performance/bestpractices#defer-reads)na göz atabilirsiniz.

### UI element state
UI öğesi state'ini, okuması veya yazması gereken business logic varsa ekran seviyesi state holder'a hoist edebilirsiniz.

Bir sohbet uygulaması örneğine devam edersek, kullanıcı @ ve bir ipucu yazdığında uygulama bir grup sohbetinde kullanıcı önerilerini görüntüler. Bu öneriler data katmanından gelir ve kullanıcı önerilerinin bir listesini hesaplama mantığı business logic olarak kabul edilir. Özellik şu şekilde görünür:

![](https://developer.android.com/static/images/jetpack/compose/state-hoisting-suggestions.png)
Şekil 7. Kullanıcı `@` ve bir ipucu yazdığında bir grup sohbetinde kullanıcı önerilerini görüntüleyen özellik

Bu özelliği uygulayan ViewModel aşağıdaki gibi görünecektir:
```kotlin
class ConversationViewModel(/*...*/) : ViewModel() {

    // Hoisted state
    var inputMessage by mutableStateOf("")
        private set

    val suggestions: StateFlow<List<Suggestion>> =
        snapshotFlow { inputMessage }
            .filter { hasSocialHandleHint(it) }
            .mapLatest { getHandle(it) }
            .mapLatest { repository.getSuggestions(it) }
            .stateIn(
                scope = viewModelScope,
                started = SharingStarted.WhileSubscribed(5_000),
                initialValue = emptyList()
            )

    fun updateInput(newInput: String) {
        inputMessage = newInput
    }
}
```
inputMessage, TextField state'ini tutan bir değişkendir. Kullanıcı her yeni girdi girdiğinde, uygulama öneriler üretmek için business logic'i çağırır.

{: .note}
Not: Bu değişken, şu anda kullanıcı önerileri üretmek için ihtiyaç duyulduğu gibi business logic için gerekli olmasaydı, ekran seviyesi state holder hoist edilmemeliydi. Tanımlanmalı ve UI'da, ona ihtiyaç duyan composable fonksiyona daha yakın bir yerde saklanmalıdır.

`suggestions` ekran UI state'idir ve StateFlow'dan collect edilerek Compose UI`dan tüketilir.

{: .note}
Not: Ekran düzeyinde bir composable'ın hem business logic'e erişim sağlayan bir ViewModel'e hem de UI logic'ini ve UI elementlerinin state'ini yöneten düz bir state holder sınıfına sahip olması mümkündür.

#### Caveat
Bazı Compose UI öğesi state'leri için ViewModel'e hoisting işlemi özel hususlar gerektirebilir. Örneğin, Compose UI öğelerinin bazı state holder'ları, state'i değiştirmek için metotlar ortaya koyar. Bunlardan bazıları animasyonları tetikleyen suspend fonksiyonları olabilir. Bu suspend fonksiyonları, Composition'a scope edilmemiş bir [CoroutineScope](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/)'dan çağırılırsa exception fırlatabilir.

Diyelim ki uygulama drawer'ının içeriği dinamik ve kapandıktan sonra data katmanından getirip yenilemeniz gerekiyor. Drawer state'ini ViewModel'e hoist etmelisiniz, böylece state sahibinden bu öğe üzerinde hem UI hem de business logic'i çağırabilirsiniz.

Ancak, Compose UI'den [viewModelScope](https://developer.android.com/topic/libraries/architecture/coroutines#viewmodelscope) kullanarak [DrawerState](https://developer.android.com/reference/kotlin/androidx/compose/material/DrawerState)'in [close()](https://developer.android.com/reference/kotlin/androidx/compose/material/DrawerState#close()) metodunu çağırmak, "bu [CoroutineContext](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/-coroutine-context/)'te bir [MonotonicFrameClock](https://developer.android.com/reference/kotlin/androidx/compose/runtime/MonotonicFrameClock) mevcut değil" şeklinde bir mesajla [IllegalStateException](https://docs.oracle.com/javase/7/docs/api/java/lang/IllegalStateException.html) türünde bir çalışma zamanı exception'ına neden olur.

Bunu düzeltmek için Composition'a scope edilmiş bir CoroutineScope kullanın. CoroutineContext'te suspend fonksiyonlarının çalışması için gerekli olan bir MonotonicFrameClock sağlar.

{: .warning}
Uyarı: Compose UI öğesi state'inden açığa çıkan ve animasyonları tetikleyen bazı suspend fonksiyonlarının çağrılması, Composition'a scope edilmemiş bir CoroutineScope'tan çağrılırsa exception atar. Örneğin, [LazyListState.animateScrollTo()](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/LazyListState#animateScrollToItem(kotlin.Int,kotlin.Int)) ve [DrawerState.close()](https://developer.android.com/reference/kotlin/androidx/compose/material/DrawerState#close()).

Bu çökmeyi düzeltmek için ViewModel'deki coroutine'in CoroutineContext'ini Composition'a scopelanmış bir CoroutineContext ile değiştirin. Şöyle görünebilir:

```kotlin
class ConversationViewModel(/*...*/) : ViewModel() {

    val drawerState = DrawerState(initialValue = DrawerValue.Closed)

    private val _drawerContent = MutableStateFlow(DrawerContent.Empty)
    val drawerContent: StateFlow<DrawerContent> = _drawerContent.asStateFlow()

    fun closeDrawer(uiScope: CoroutineScope) {
        viewModelScope.launch {
            withContext(uiScope.coroutineContext) { // Default context yerine kullanın
                drawerState.close()
            }
            // Drawer içeriğini getirme ve state'i güncelleme
            _drawerContent.update { content }
        }
    }
}

// in Compose
@Composable
private fun ConversationScreen(
    conversationViewModel: ConversationViewModel = viewModel()
) {
    val scope = rememberCoroutineScope()

    ConversationScreen(onCloseDrawer = { conversationViewModel.closeDrawer(uiScope = scope) })
}
```
## Learn more
State ve Jetpack Compose hakkında daha fazla bilgi edinmek için aşağıdaki ek kaynaklara başvurun.

### Samples
- [Jetnews sample](https://github.com/android/compose-samples/tree/main/JetNews)
- [Jetchat sample](https://github.com/android/compose-samples/tree/main/Jetchat)
- [Now in Android App](https://github.com/android/nowinandroid/tree/main)

### Codelabs
- [Using State in Jetpack Compose](https://codelabs.developers.google.com/codelabs/jetpack-compose-state/index.html?index=..%2F..index#0)
### Videos
- [State holders and state production in the UI Layer](https://youtu.be/pCX9wvu-Bq0)