---
layout: default
title: Save UI State
parent: Managing State
grand_parent: UI Architecture
nav_order: 3
---
# Save UI state in Compose
State'inizin nereye hoist edildigine ve gerekli olan logic'e bagli olarak, [UI state](/docs/app-architecture/guide-to-app-architecture/ui-layer/state-holders-and-ui-state/#ui-state)'inizi depolamak ve geri yüklemek için farkli API'ler kullanabilirsiniz. Her uygulama bunu en iyi şekilde başarmak için API'lerin bir kombinasyonunu kullanır.

{: .note }
Not: Uygulanan lojiğe bağlı olarak state'inizin nereye hoist edilecegi hakkında daha fazla bilgiyi [Where to hoist state ](/docs/jetpack-compose/ui-architecture/managing-state/where-to-hoist-state/)belgesinde görebilirsiniz.

Herhangi bir Android uygulaması, activity veya process yeniden oluşturma nedeniyle UI state'ini kaybedebilir. Bu state kaybı aşağıdaki olaylar nedeniyle meydana gelebilir:

- [Konfigürasyon değişiklikleri](/docs/app-basics/app-resources/handle-configuration-changes/). Konfigürasyon değişikliği manuel olarak yapılmadığı sürece activity yok edilir ve yeniden oluşturulur.

- [Sistem tarafından başlatılan proses ölümü](/docs/app-architecture/architecture-components/ui-layer-libraries/lifecycle-aware-components/save-ui-states/#system-initiated-ui-state-dismissal). Uygulama arka plandadır ve cihaz diğer işlemler tarafından kullanılmak üzere kaynakları (bellek gibi) serbest bırakır.

{: .note }
Not: Sistem tarafından başlatılan proses ölümü, kullanıcının aktiviteyi açıkça sonlandırdığı kullanıcı tarafından başlatılan proses ölümünden farklıdır. Kullanıcı tarafından başlatılan proses ölümü durumunda, geçici state kaybı genellikle makuldür (örneğin, bir form doldurulurken animasyon state'inin veya bir TextField içeriğinin kaybedilmesi).

Bu olaylardan sonra state'i korumak, olumlu bir kullanıcı deneyimi için çok önemlidir. Hangi state'in korunacağının seçilmesi, uygulamanızın benzersiz kullanıcı akışlarına bağlıdır. En iyi pratik olarak, en azından kullanıcı girdisini ve navigasyonla ilgili state'i korumalısınız. Buna örnek olarak bir listenin kaydırma konumu, kullanıcının hakkında daha fazla ayrıntı istediği öğenin ID'si, kullanıcı tercihlerinin devam eden seçimi veya metin alanlarındaki girdi verilebilir.

Bu sayfa, state'inizin nereye hoist edildiğine ve buna ihtiyaç duyan logic'e bağlı olarak UI state'ini depolamak için kullanılabilen API'leri özetlemektedir.

## UI logic
State'iniz UI'da, composable fonksiyonlarda ya da Composition'a scope edilmiş düz state holder sınıflarında tutuluyorsa, activity ve process yeniden oluşturulurken state'i korumak için rememberSaveable'ı kullanabilirsiniz.

Aşağıdaki kod parçasında, rememberSaveable tek bir boolean UI öğesi state'ini saklamak için kullanılmaktadır:

```kotlin
@Composable
fun ChatBubble(
    message: Message
) {
    var showDetails by rememberSaveable { mutableStateOf(false) }

    ClickableText(
        text = AnnotatedString(message.content),
        onClick = { showDetails = !showDetails }
    )

    if (showDetails) {
        Text(message.timestamp)
    }
}
```
showDetails, sohbet balonunun daraltılmış veya genişletilmiş olup olmadığını saklayan boolean bir değişkendir.

{: .note }
Önemli: Genellikle, saved instance state'te depolanan veriler, kullanıcı girdisine veya navigasyona bağlı olan geçici state'tir. Bunun örnekleri arasında bir listenin kaydırma konumu, kullanıcının hakkında daha fazla ayrıntı istediği öğenin ID'si, kullanıcı tercihlerinin devam eden seçimi veya metin alanlarındaki girdi yer alır.

`rememberSaveable`, saved instance state mekanizması aracılığıyla UI öğesi state'ini bir [Bundle](https://developer.android.com/reference/android/os/Bundle)'da saklar.

Primitif türleri otomatik olarak bundle'a depolayabilir. State öğeniz data class gibi primitive olmayan bir türde tutuluyorsa, [Parcelize](https://developer.android.com/kotlin/parcelize) annotation'ını kullanmak, [listSaver](https://developer.android.com/reference/kotlin/androidx/compose/runtime/saveable/package-summary#listSaver(kotlin.Function2,kotlin.Function1)) ve [mapSaver](https://developer.android.com/reference/kotlin/androidx/compose/runtime/saveable/package-summary#mapSaver(kotlin.Function2,kotlin.Function1)) gibi Compose API'lerini kullanmak veya Compose runtime [Saver](https://developer.android.com/reference/kotlin/androidx/compose/runtime/saveable/Saver) sınıfını extend eden özel bir saver sınıfı implemente etmek gibi farklı depolama mekanizmaları kullanabilirsiniz. Bu yöntemler hakkında daha fazla bilgi edinmek için [Ways to store state](/docs/jetpack-compose/ui-architecture/managing-state/overview/#ways-to-store-state) belgesine bakın.

Aşağıdaki kod parçasında, [rememberLazyListState](https://cs.android.com/androidx/platform/tools/dokka-devsite-plugin/+/master:testData/compose/source/androidx/compose/foundation/lazy/LazyListState.kt;l=49?q=LazyListState) Compose API'si, bir [LazyColumn](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/package-summary#LazyColumn(androidx.compose.ui.Modifier,androidx.compose.foundation.lazy.LazyListState,androidx.compose.foundation.layout.PaddingValues,kotlin.Boolean,androidx.compose.foundation.layout.Arrangement.Vertical,androidx.compose.ui.Alignment.Horizontal,androidx.compose.foundation.gestures.FlingBehavior,kotlin.Boolean,kotlin.Function1)) veya [LazyRow](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/package-summary#LazyRow(androidx.compose.ui.Modifier,androidx.compose.foundation.lazy.LazyListState,androidx.compose.foundation.layout.PaddingValues,kotlin.Boolean,androidx.compose.foundation.layout.Arrangement.Horizontal,androidx.compose.ui.Alignment.Vertical,androidx.compose.foundation.gestures.FlingBehavior,kotlin.Boolean,kotlin.Function1))'un kaydırma state'inden oluşan [LazyListState](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/LazyListState)'i rememberSaveable kullanarak saklar. Kaydırma state'ini saklayabilen ve geri yükleyebilen özel bir saver olan bir [LazyListState.Saver](https://cs.android.com/androidx/platform/frameworks/support/+/androidx-main:compose/foundation/foundation/src/commonMain/kotlin/androidx/compose/foundation/lazy/LazyListState.kt;l=413?q=LazyListState.kt) kullanır. Bir activity veya process yeniden oluşturulduktan sonra (örneğin, cihaz yönünü değiştirmek gibi bir konfigürasyon değişikliğinden sonra), kaydırma state'i korunur.

```kotlin
@Composable
fun rememberLazyListState(
    initialFirstVisibleItemIndex: Int = 0,
    initialFirstVisibleItemScrollOffset: Int = 0
): LazyListState {
    return rememberSaveable(saver = LazyListState.Saver) {
        LazyListState(
            initialFirstVisibleItemIndex, initialFirstVisibleItemScrollOffset
        )
    }
}
```
### Best practice
rememberSaveable, UI state'ini depolamak için bir Bundle kullanır; bu Bundle, activity'nizdeki onSaveInstanceState() çağrıları gibi ona yazan diğer API'ler tarafından da paylaşılır. Ancak bu Bundle'ın boyutu sınırlıdır ve büyük nesnelerin depolanması çalışma zamanında TransactionTooLarge exception'larına yol açabilir. Bu, özellikle uygulama genelinde aynı Bundle'ın kullanıldığı tek Activity uygulamalarında sorun yaratabilir.

Bu tür bir çökmeyi önlemek için, büyük karmaşık nesneleri veya nesne listelerini Bundle'da depolamamalısınız.

Bunun yerine, ID'ler veya key'ler gibi gereken minimum state'i depolayın ve daha karmaşık UI state'lerini kalıcı depolama gibi diğer mekanizmalara geri yüklemek için bunları kullanın.

{: .note }
Not: Bazı durumlarda, tüm UI öğelerinin state'ini depolamamak kabul edilebilir.


Bu tasarım seçimleri, uygulamanızın özel kullanım durumlarına ve kullanıcılarınızın nasıl davranmasını beklediğine bağlıdır.

### Verify state restoration
Compose öğelerinizde rememberSaveable ile saklanan state'in, activity veya process yeniden oluşturulduğunda doğru şekilde geri yüklendiğini doğrulayabilirsiniz. Bunu başarmak için [StateRestorationTester](https://developer.android.com/reference/kotlin/androidx/compose/ui/test/junit4/StateRestorationTester) gibi özel API'ler vardır. Daha fazla bilgi edinmek için [Test](https://developer.android.com/jetpack/compose/testing#verify_state_restoration) belgelerine göz atın.

## Business logic
[UI öğesi state](/docs/app-architecture/guide-to-app-architecture/ui-layer/state-holders-and-ui-state/#ui-state)'iniz business logic'i gerektirdiği için ViewModel'e hoist edilmis ise ViewModel'in API'lerini kullanabilirsiniz.

Android uygulamanızda ViewModel kullanmanın en önemli avantajlarından biri, konfigürasyon değişikliklerini sorunsuz bir şekilde gerçekleştirmesidir. Bir konfigürasyon değişikliği olduğunda ve aktivite yok edilip yeniden oluşturulduğunda, ViewModel'e hoist edilen UI state'i bellekte tutulur. Yeniden oluşturulduktan sonra, eski ViewModel instance'ı yeni activity instance'ına eklenir.

{: .note }
Not: ViewModel, ekran düzeyinde bir state holder implementasyonu olarak, ekran UI state'ini üretmek için kullanılan business logic'i yönetir. UI state'ini ViewModel'e, konfigürasyon değişikliklerini kolayca halledeceği için değil, mimariniz için mantıklı olduğu için hoist etmelisiniz.

Ancak, bir ViewModel instance'ı sistem tarafından başlatılan proses ölümünde hayatta kalamaz. UI state'inin bu durumdan kurtulmasını sağlamak için, [SavedStateHandle](https://developer.android.com/reference/androidx/lifecycle/SavedStateHandle) API'sini içeren [ViewModel için Saved State modülü](/docs/app-architecture/architecture-components/ui-layer-libraries/lifecycle-aware-components/ViewModel/saved-state-module-for-viewmodel/)nü kullanın.

### Best practice
SavedStateHandle ayrıca UI state'ini depolamak için Bundle mekanizmasını kullanır, bu nedenle bunu yalnızca basit UI öğesi state'ini depolamak için kullanmalısınız.

Business rule uygulayarak ve uygulamanızın UI dışındaki katmanlarına erişerek üretilen Screen UI state, potansiyel karmaşıklığı ve boyutu nedeniyle SavedStateHandle'da saklanmamalıdır. Karmaşık veya büyük verileri depolamak için lokal kalıcı depolama gibi farklı mekanizmalar kullanabilirsiniz. Bir proses yeniden oluşturulduktan sonra ekran, SavedStateHandle'da (varsa) saklanan geri yüklenmiş geçici state ile yeniden oluşturulur ve ekran UI state'i data katmanından tekrar üretilir.

{: .note }
Not: UI state'i kaydetmenin farklı yolları hakkında daha fazla bilgi için [Save UI states](/docs/app-architecture/architecture-components/ui-layer-libraries/lifecycle-aware-components/save-ui-states/) belgesine bakın.
### SavedStateHandle APIs
[SavedStateHandle](https://developer.android.com/reference/androidx/lifecycle/SavedStateHandle), özellikle UI öğesi state'ini saklamak için farklı API'lere sahiptir:

| Compose [State](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State) | [saveable()](https://developer.android.com/reference/kotlin/androidx/lifecycle/viewmodel/compose/package-summary#(androidx.lifecycle.SavedStateHandle).saveable(kotlin.String,androidx.compose.runtime.saveable.Saver,kotlin.Function0))     |
|---------------|----------------|
| [StateFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/)     | [getStateFlow()](https://developer.android.com/reference/androidx/lifecycle/SavedStateHandle#getStateFlow(kotlin.String,kotlin.Any)) |

### Compose State
UI öğesi state'ini MutableState olarak okumak ve yazmak için SavedStateHandle'ın saveable API'sini kullanın, böylece en az kod kurulumuyla activity ve process yeniden yaratımından kurtulur.

Saveable API, kutudan çıktığı haliyle primitif türleri destekler ve tıpkı rememberSaveable() gibi custom saver kullanmak için bir stateSaver parametresi alır.

Aşağıdaki kod parçacığında, message kullanıcı giriş türlerini bir TextField içine kaydeder:

```kotlin
class ConversationViewModel(
    savedStateHandle: SavedStateHandle
) : ViewModel() {

    var message by savedStateHandle.saveable(stateSaver = TextFieldValue.Saver) {
        mutableStateOf(TextFieldValue(""))
    }
        private set

    fun update(newMessage: TextFieldValue) {
        message = newMessage
    }

    /*...*/
}

val viewModel = ConversationViewModel(SavedStateHandle())

@Composable
fun UserInput(/*...*/) {
    TextField(
        value = viewModel.message,
        onValueChange = { viewModel.update(it) }
    )
}
```
Saveable API kullanımı hakkında daha fazla bilgi için [SavedStateHandle](https://developer.android.com/topic/libraries/architecture/viewmodel/viewmodel-savedstate#savedstate-compose-state) belgelerine bakın.

{: .warning}
Dikkat: [saveable](https://developer.android.com/topic/libraries/architecture/viewmodel/viewmodel-savedstate#savedstate-compose-state) API deneyseldir.

### StateFlow
UI öğesi state'ini depolamak ve [SavedStateHandle](https://developer.android.com/reference/androidx/lifecycle/SavedStateHandle)'dan bir flow olarak tüketmek için [getStateFlow()](https://developer.android.com/reference/androidx/lifecycle/SavedStateHandle#getStateFlow(kotlin.String,kotlin.Any)) metodunu kullanın. [StateFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/) salt okunurdur ve API, yeni bir değer emit etmek üzere flow'u değiştirebilmeniz için bir key belirtmenizi gerektirir. Ayarladığınız key ile StateFlow'u alabilir ve en son değeri collect edebilirsiniz.

Aşağıdaki kod parçasında, savedFilterType, bir sohbet uygulamasındaki sohbet kanalları listesine uygulanan filtre tipini saklayan bir StateFlow değişkenidir:
```kotlin
private const val CHANNEL_FILTER_SAVED_STATE_KEY = "ChannelFilterKey"

class ChannelViewModel(
    channelsRepository: ChannelsRepository,
    private val savedStateHandle: SavedStateHandle
) : ViewModel() {

    private val savedFilterType: StateFlow<ChannelsFilterType> = savedStateHandle.getStateFlow(
        key = CHANNEL_FILTER_SAVED_STATE_KEY, initialValue = ChannelsFilterType.ALL_CHANNELS
    )

    private val filteredChannels: Flow<List<Channel>> =
        combine(channelsRepository.getAll(), savedFilterType) { channels, type ->
            filter(channels, type)
        }.onStart { emit(emptyList()) }

    fun setFiltering(requestType: ChannelsFilterType) {
        savedStateHandle[CHANNEL_FILTER_SAVED_STATE_KEY] = requestType
    }

    /*...*/
}

enum class ChannelsFilterType {
    ALL_CHANNELS, RECENT_CHANNELS, ARCHIVED_CHANNELS
}
```

Kullanıcı her yeni filtre türü seçtiğinde `setFiltering` çağrılır. Bu, `_CHANNEL_FILTER_SAVED_STATE_KEY_` key'i ile saklanan SavedStateHandle'a yeni bir değer kaydeder. savedFilterType, key'e saklanan en son değeri emit eden bir flow'dur. filteredChannels, kanal filtrelemesini gerçekleştirmek için flow'a abone olur.

getStateFlow() API'si hakkında daha fazla bilgi için [SavedStateHandle](https://developer.android.com/topic/libraries/architecture/viewmodel/viewmodel-savedstate#savedstate-stateflow) belgelerine bakın.

### Summary
Aşağıdaki tabloda, bu bölümde ele alınan API'ler ve UI state'ini kaydetmek için her birinin ne zaman kullanılacağı özetlenmektedir:

| Event                 | UI logic | Business logici in a ViewModel |
|-----------------------|----------|--------------------------------|
| Configuration changes |	rememberSaveable |	Automatic |
|System-initiated process death |	rememberSaveable |	SavedStateHandle |

Kullanılacak API, state'in nerede tutulduğuna ve gerektirdiği lojiğe bağlıdır. [UI logic](/docs/app-architecture/guide-to-app-architecture/ui-layer/state-holders-and-ui-state/#logic)'te kullanılan state için rememberSaveable kullanın.[ Business logic](/docs/app-architecture/guide-to-app-architecture/ui-layer/state-holders-and-ui-state/#logic)'te kullanılan state için, eğer bir ViewModel'de tutuyorsanız, SavedStateHandle kullanarak kaydedin.

Küçük miktarlarda UI state saklamak için bundle API'lerini (rememberSaveable ve SavedStateHandle) kullanmalısınız. Bu veriler, diğer saklama mekanizmalarıyla birlikte UI'yi önceki state'ine geri döndürmek için gerekli minimum verilerdir. Örneğin, kullanıcının baktığı bir profilin ID'sini bundle'da saklarsanız, profil ayrıntıları gibi ağır verileri data katmanından getirebilirsiniz.

UI state'i kaydetmenin farklı yolları hakkında daha fazla bilgi için genel [Saving UI State belgesi](/docs/app-architecture/architecture-components/ui-layer-libraries/lifecycle-aware-components/save-ui-states/)ne ve mimari kılavuzunun [data layer](/docs/app-architecture/guide-to-app-architecture/data-layer/about-the-data-layer/#about-the-data-layer) sayfasına bakın.

