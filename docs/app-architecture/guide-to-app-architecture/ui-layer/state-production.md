---
layout: default
title: State production
grand_parent: Guide to app architecture
nav_order: 4
parent: Ui layer
---

# State production
## UI State Production
Modern UI ler nadiren statiktir. Kullanıcı, UI ile etkileşime girdiğinde veya uygulamanın yeni verileri göstermesi gerektiğinde UI’in state’i değişir.
Bu belge, UI state’inin üretimi(production) ve yönetimi için yönergeler belirler. Sonunda şunları yapmalısınız:

* UI state’i oluşturmak(produce) için hangi API'leri kullanmanız gerektiğini bilin. Bu, [tek yönlü veri akışı(UDF)](about-the-ui-layer.md#manage-state-with-undirectional-data-flow) ilkelerini izleyerek state holderlerinizde bulunan state değişikliği kaynaklarının doğasına bağlıdır.
* Sistem kaynaklarının bilincinde olmak için UI state’inin üretimini nasıl kapsamanız(scope) gerektiğini öğrenin.
* UI tarafından tüketim için UI stateini nasıl ortaya çıkarmanız gerektiğini bilin.

Temel olarak state üretimi(state production), bu değişikliklerin UI state’ine artımlı olarak uygulanmasıdır. State her zaman vardır ve eventler sonucunda değişir. Eventler ve state arasındaki farklar aşağıdaki tabloda özetlenmiştir:

| Event | State |
| --- | --- |
| Transient, unpredictable, and exist for a finite period.(Geçici, öngörülemeyen ve sonlu bir süre için var olan.)| Always exists.(herzaman vardir)|
| The inputs of state production.(State üretiminin girdileridir.)| The output of state production.(State uretiminin ciktisidir.)|
| The product of the UI or other sources.(UI’in veya diğer kaynaklarin urunudur.)| Is consumed by the UI.(UI tarafindan tuketilir)|

Yukarıdakileri özetleyen harika bir anımsatıcı şudur: state is; events happen. Aşağıdaki şema, eventler bir zaman çizelgesinde meydana geldikçe statedeki değişiklikleri görselleştirmeye yardımcı olur. Her event uygun state holder tarafından işlenir ve bir state değişikliğiyle sonuçlanır:
![Events cause state to change](/assets/images/state-production-img1.png)

Eventler şunlardan gelebilir:
* Kullanıcılar: Uygulamanın UI ile  etkileşime girdikçe.
* Diğer state değişikliği kaynakları: UI’den, domainden veya snackbar zaman aşımı eventleri gibi data katmanlarından uygulama verileri sunan API'ler, sırasıyla use case siniflari veya repositoryler.
  
### The UI state production pipeline
Android uygulamalarındaki state production, aşağıdakilerden oluşan bir processing pipeline olarak düşünülebilir:
* Inputs;State’in  kaynakları değişir. Olabilirler:
  * UI katmanında local: Bunlar, bir görev yönetimi uygulamasında "yapılacak iş" için bir başlık giren bir kullanıcı gibi kullanıcı eventleri veya UI state’indeki değişiklikleri yönlendiren [UI logic](state-holders-and-ui-state.md#ui-logic-and-its-state-holder)ine erişim sağlayan API'ler olabilir. Örneğin, Jetpack Compose'da [DrawerState](https://developer.android.com/reference/kotlin/androidx/compose/material/DrawerState)'te [open](https://developer.android.com/reference/kotlin/androidx/compose/material/DrawerState#open()) methodunu çağırmak. 
  * UI katmanının dışında: Bunlar, UI state’inde değişikliklere neden olan domain veya data katmanlarından gelen kaynaklardır. Örneğin, bir NewsRepository'den yüklenmesi biten haberler veya diğer eventler. 
  * Yukarıdakilerin hepsinin bir karışımı.
* [State holders](state-holders-and-ui-state.md#state-holders-and-ui-state);[Business logic](state-holders-and-ui-state.md#business-logic-and-its-state-holder)i ve/veya UI logicini state değişikliği kaynaklarına uygulayan ve UI state oluşturmak(produce) için kullanıcı eventlerini işleyen türler. 
* Output;Uygulamanın, kullanıcılara ihtiyaç duydukları bilgileri sağlamak için işleyebileceği UI State.

![The state production pipeline](/assets/images/state-production-img2.png)

### State production APIs
Pipeline’in hangi aşamasında olduğunuza bağlı olarak state productionda kullanılan iki ana API vardır:

|Pipeline stage | API |
|---------------|----------------|
|Input | You should use asynchronous APIs to perform work off the UI thread to keep the UI jank free. For example, Coroutines or Flows in Kotlin, and RxJava or callbacks in the Java Programming Language. |
|Output| You should use observable data holder APIs to invalidate and rerender the UI when state changes. For example, StateFlow, Compose State, or LiveData. Observable data holders guarantee the UI always has a UI state to display on the screen. |

Bu ikisi arasından, input için asenkron API seçiminin, output için gözlemlenebilir API seçiminden çok, state production pipelinein doğası üzerinde daha büyük bir etkisi vardır. Bunun nedeni, inputlarin pipeline’a uygulanabilecek processing türünü dikte etmesidir.

### State production pipeline assembly

Sonraki bölümler, çeşitli inputlar için en uygun state productin tekniklerini ve eşleşen output API'lerini kapsar. Her state production pipelie, inputlarin ve outputlarin bir kombinasyonudur ve şöyle olmalıdır:
* Yaşam döngüsünün farkında(Lifecycle aware): UI’in visible veya active olmadığı durumlarda, açıkça gerekli olmadıkça state production pipeline herhangi bir kaynak tüketmemelidir.
* Kullanımı kolay(Easy to consume): UI, üretilen UI state’ini kolayca oluşturabilmelidir(produce etmelidir). State production pipelinenin outputuna yönelik hususlar, View sistemi veya Jetpack Compose gibi farklı View API'lerinde değişiklik gösterecektir.

<mark style="background-color: lightblue">Not: İzleyen bölümlerde, tartışılan tüm API'ler deyimsel Kotlin ve Jetpack Compose kodunu kullanır. Ancak kılavuz, Java Programlama Dili veya Kotlin'deki diğer API'lerdeki eşdeğer analogları için geçerlidir.
</mark>

### Input in state production pipelines

Bir state production pipelinedeki inputlar, state değişikliği kaynaklarını şu yollarla sağlayabilir:
* One-shot operations that may be synchronous or asynchronous, for example calls to suspend functions.
* Stream APIs, for example Flows.
* All of the above.

Aşağıdaki bölümlerde, yukarıdaki inputlarin her biri için bir state production pipline’i nasıl kurabileceğiniz ele alınmaktadır.

### One-shot APIs as sources of state change
[MutableStateFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-mutable-state-flow/) API'yi gözlemlenebilir, değiştirilebilir bir state containeri olarak kullanın. Jetpack Compose uygulamalarında, özellikle [Compose text API](https://medium.com/androiddevelopers/effective-state-management-for-textfield-in-compose-d6e5b070fbe5)'leri ile çalışırken [mutableStateOf](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#mutableStateOf(kotlin.Any,androidx.compose.runtime.SnapshotMutationPolicy))'u da düşünebilirsiniz. Her iki API de barındırdıkları değerlerde güvenli atomik güncellemelere izin veren methodlar sunar, güncellemeler senkron veya asenkro olsun ya da olmasın. Örneğin, basit bir zar atma uygulamasında state güncellemelerini düşünün. Kullanıcının attığı her zar, senkronize [Random.nextInt()](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.random/next-int.html) methodunu çağırır ve sonuç, UI state’ine yazılır.
```kotlin
// StateFlow
data class DiceUiState(
        val firstDieValue: Int? = null,
        val secondDieValue: Int? = null,
        val numberOfRolls: Int = 0,
)

class DiceRollViewModel : ViewModel() {

  private val _uiState = MutableStateFlow(DiceUiState())
  val uiState: StateFlow<DiceUiState> = _uiState.asStateFlow()

  // Called from the UI
  fun rollDice() {
    _uiState.update { currentState ->
      currentState.copy(
              firstDieValue = Random.nextInt(from = 1, until = 7),
              secondDieValue = Random.nextInt(from = 1, until = 7),
              numberOfRolls = currentState.numberOfRolls + 1,
      )
    }
  }
}

//compose
@Stable
interface DiceUiState {
  val firstDieValue: Int?
  val secondDieValue: Int?
  val numberOfRolls: Int?
}

private class MutableDiceUiState: DiceUiState {
  override var firstDieValue: Int? by mutableStateOf(null)
  override var secondDieValue: Int? by mutableStateOf(null)
  override var numberOfRolls: Int by mutableStateOf(0)
}

class DiceRollViewModel : ViewModel() {

  private val _uiState = MutableDiceUiState()
  val uiState: DiceUiState = _uiState

  // Called from the UI
  fun rollDice() {
    _uiState.firstDieValue = Random.nextInt(from = 1, until = 7)
    _uiState.secondDieValue = Random.nextInt(from = 1, until = 7)
    _uiState.numberOfRolls = _uiState.numberOfRolls + 1
  }
}
```

### Mutating the UI state from asynchronous calls
Asenkron bir sonuç gerektiren state değişiklikleri için uygun CoroutineScope'ta bir Coroutine başlatın. Bu, CoroutineScope iptal edildiğinde uygulamanın işi silmesine izin verir. State holder daha sonra suspend method çağrısının sonucunu UI state’ini ortaya çıkarmak için kullanılan gözlemlenebilir API'ye yazar. Örneğin, [Architecture örneği](https://github.com/android/architecture-samplesndeki AddEditTaskViewModel'i göz önünde bulundurun. Askıya alınan(suspend edilen) saveTask() methodu bir taski asenkron olarak kaydettiğinde, MutableStateFlow'daki [update](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/update.html) methodu state değişikliğini UI state’ine yayar.
```kotlin
// StateFlow
data class AddEditTaskUiState(
    val title: String = "",
    val description: String = "",
    val isTaskCompleted: Boolean = false,
    val isLoading: Boolean = false,
    val userMessage: String? = null,
    val isTaskSaved: Boolean = false
)

class AddEditTaskViewModel(...) : ViewModel() {

   private val _uiState = MutableStateFlow(AddEditTaskUiState())
   val uiState: StateFlow<AddEditTaskUiState> = _uiState.asStateFlow()

   private fun createNewTask() {
        viewModelScope.launch {
            val newTask = Task(uiState.value.title, uiState.value.description)
            try {
                tasksRepository.saveTask(newTask)
                // Write data into the UI state.
                _uiState.update {
                    it.copy(isTaskSaved = true)
                }
            }
            catch(cancellationException: CancellationException) {
                throw cancellationException
            }
            catch(exception: Exception) {
                _uiState.update {
                    it.copy(userMessage = getErrorMessage(exception))
                }
            }
        }
    }
}

//Compose State
@Stable
interface AddEditTaskUiState {
  val title: String
  val description: String
  val isTaskCompleted: Boolean
  val isLoading: Boolean
  val userMessage: String?
  val isTaskSaved: Boolean
}

private class MutableAddEditTaskUiState : AddEditTaskUiState() {
  override var title: String by mutableStateOf("")
  override var description: String by mutableStateOf("")
  override var isTaskCompleted: Boolean by mutableStateOf(false)
  override var isLoading: Boolean by mutableStateOf(false)
  override var userMessage: String? by mutableStateOf<String?>(null)
  override var isTaskSaved: Boolean by mutableStateOf(false)
}

class AddEditTaskViewModel(...) : ViewModel() {

  private val _uiState = MutableAddEditTaskUiState()
  val uiState: AddEditTaskUiState = _uiState

  private fun createNewTask() {
    viewModelScope.launch {
      val newTask = Task(uiState.value.title, uiState.value.description)
      try {
        tasksRepository.saveTask(newTask)
        // Write data into the UI state.
        _uiState.isTaskSaved = true
      }
      catch(cancellationException: CancellationException) {
        throw cancellationException
      }
      catch(exception: Exception) {
        _uiState.userMessage = getErrorMessage(exception))
      }
    }
  }
}
```
<mark style="background-color:lightblue">Not: Bir [AAC  ViewModel](/docs/app-architecture/architecture-components/ui-layer-libraries/lifecycle-aware-components/ViewModel/about-viewmodel.md)'in viewModelScope'unda başlatılan coroutineler, istisnai olarak veya başka türlü tamamlanmaya çalışır. Bu, Coroutines açıkça iptal edilmedikçe veya ViewModel temizlenmedikçe, UI görünür olsun ya da olmasın gerçekleşir. Kısa ömürlü olma eğiliminde olduklarından, çoğu istek için bu genellikle uygundur. 5 saniye veya daha uzun süren istekleri çalıştırmak için viewModelScope kullanmamalısınız. Bunun yerine bunları WorkManager ile ertelenmiş veya uzun süreli işler olarak kuyruğa almalısınız.</mark>

### Mutating the UI state from background threads
UI state’inin productionu için main dispacther Coroutines'in başlatılması tercih edilir. Yani, aşağıdaki kod parçacıklarındaki withContext bloğunun dışında. Ancak, UI state’ini farklı bir backgroud context’inde güncellemeniz gerekirse, bunu aşağıdaki API'leri kullanarak yapabilirsiniz:
* Use the [withContext](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html) method to run Coroutines in a different concurrent context.
* When using MutableStateFlow, use the [update](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/update.html) method as usual.
* When using Compose State, use the [Snapshot.withMutableSnapshot](https://developer.android.com/reference/kotlin/androidx/compose/runtime/snapshots/Snapshot.Companion#withMutableSnapshot(kotlin.Function0)) to guarantee atomic updates to State in the concurrent context.

Örneğin, aşağıdaki DiceRollViewModel parçacığında, SlowRandom.nextInt()'in CPU'ya bağlı bir Coroutine'den çağrılması gereken hesaplama açısından yoğun bir askıya alma işlevi olduğunu varsayalım.
```kotlin
// StateFlow
class DiceRollViewModel(
        private val defaultDispatcher: CoroutineScope = Dispatchers.Default
) : ViewModel() {

  private val _uiState = MutableStateFlow(DiceUiState())
  val uiState: StateFlow<DiceUiState> = _uiState.asStateFlow()

  // Called from the UI
  fun rollDice() {
    viewModelScope.launch() {
      // Other Coroutines that may be called from the current context
      …
      withContext(defaultDispatcher) {
        _uiState.update { currentState ->
          currentState.copy(
                  firstDieValue = SlowRandom.nextInt(from = 1, until = 7),
                  secondDieValue = SlowRandom.nextInt(from = 1, until = 7),
                  numberOfRolls = currentState.numberOfRolls + 1,
          )
        }
      }
    }
  }
}

// Compose State
class DiceRollViewModel(
        private val defaultDispatcher: CoroutineScope = Dispatchers.Default
) : ViewModel() {

  private val _uiState = MutableDiceUiState()
  val uiState: DiceUiState = _uiState

  // Called from the UI
  fun rollDice() {
    viewModelScope.launch() {
      // Other Coroutines that may be called from the current context
      …
      withContext(defaultDispatcher) {
        Snapshot.withMutableSnapshot {
          _uiState.firstDieValue = SlowRandom.nextInt(from = 1, until = 7)
          _uiState.secondDieValue = SlowRandom.nextInt(from = 1, until = 7)
          _uiState.numberOfRolls = _uiState.numberOfRolls + 1
        }
      }
    }
  }
}
```
<mark style="background-color:lightblue">Not: Başlatılan tüm coroutinelerin farklı bir contexden çağrılması gerekiyorsa, doğrudan viewModelScope.launch(defaultDispatcher){ } öğesini arayabilirsiniz.
</mark>

<mark style="background-color:red">Uyarı: Snapshot.withMutableSnapshot{ } kullanılmadan UI olmayan bir threadden Compose state’inin güncellenmesi, üretilen state’de tutarsızlıklara neden olabilir.
</mark>

### Stream APIs as sources of state change
Streamlerde zaman içinde birden çok değer üreten state değişikliği kaynakları için, tüm kaynakların outputlarini uyumlu bir bütün halinde birleştirmek, state üretimine(production) doğrudan bir yaklaşımdır.Kotlin Flows kullanırken bunu [combine](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/combine.html) fonksiyonu ile başarabilirsiniz. Bunun bir örneği, InterestsViewModel'deki "[Now in Android](https://github.com/android/nowinandroid)" örneğinde görülebilir:

```kotlin
class InterestsViewModel(
    authorsRepository: AuthorsRepository,
    topicsRepository: TopicsRepository
) : ViewModel() {

    val uiState = combine(
        authorsRepository.getAuthorsStream(),
        topicsRepository.getTopicsStream(),
    ) { availableAuthors, availableTopics ->
        InterestsUiState.Interests(
            authors = availableAuthors,
            topics = availableTopics
        )
    }
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5_000),
            initialValue = InterestsUiState.Loading
    )
}
```

<mark style="background-color:lightblue">Not: Combined Flow’u, UI state için gözlemlenebilir API olarak bir [StateFlow](https://developer.android.com/kotlin/flow/stateflow-and-sharedflow)'a dönüştürmek için [stateIn](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/state-in.html) operatörünü kullanabilirsiniz.
</mark>
StateFlows oluşturmak için stateIn operatörünün kullanılması, yalnızca UI görünürken aktif olması gerekebileceğinden, UI'ye state production pipeline’in activitysi üzerinde daha ayrıntılı kontrol sağlar.
* Flow’un yaşam döngüsüne duyarlı bir şekilde collect edilmesi sırasında pipeline’in yalnızca UI görünür olduğunda etkin olması gerekiyorsa SharingStarted.WhileSubscription() öğesini kullanın.
* Kullanıcı UI’e dönebildiği sürece, yani UI backstackde veya ekran dışında başka bir sekmede olduğu sürece pipeline’in aktif olması gerekiyorsa SharingStarted.Lazily kullanın.

Stream tabanlı state kaynaklarının collectinin geçerli olmadığı durumlarda, Kotlin Flows gibi stream API'leri, streamlerin UI state’ine işlenmesine yardımcı olmak için [merging](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/merge.html?query=fun%20%3CT%3E%20merge(vararg%20flows:%20Flow%3CT%3E), [flattening](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flat-map-latest.html?query=inline%20fun%20%3CT,%20R%3E%20Flow%3CT%3E.flatMapLatest(crossinline%20transform:%20suspend%20(T) vb. gibi zengin bir transformation seti sunar.

<mark style="background-color: lightblue">Anahtar Nokta: Çoğu durumda combine, stream API'lerinden state productiona yönelik tavsiye edilen bir yaklaşımdır.</mark>

### One-shot and stream APIs as sources of state change

State production pipeline’in, state değişikliği kaynakları olarak hem tek seferlik(one-shot) çağrılara hem de streamlere bağlı olduğu durumda, tanımlayıcı kısıtlama akışlardır(streams are defining constraint). Bu nedenle, tek seferlik çağrıları stream API'lerine dönüştürün veya outputlarini streamlere aktarın ve yukarıdaki stream bölümünde açıklandığı gibi işlemeye devam edin.

Flowlarla, bu genellikle state değişikliklerini yaymak için bir veya daha fazla private backing MutableStateFlow instancei oluşturmak anlamına gelir. Compose state’inden [snapshot flowlari](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#snapshotFlow(kotlin.Function0)) da oluşturabilirsiniz. Aşağıdaki [mimari örnekler](https://github.com/android/architecture-samples) repositorysinden TaskDetailViewModel'i göz önünde bulundurun:

```kotlin
//StateFlow
class TaskDetailViewModel @Inject constructor(
        private val tasksRepository: TasksRepository,
        savedStateHandle: SavedStateHandle
) : ViewModel() {

  private val _isTaskDeleted = MutableStateFlow(false)
  private val _task = tasksRepository.getTaskStream(taskId)

  val uiState: StateFlow<TaskDetailUiState> = combine(
          _isTaskDeleted,
          _task
  ) { isTaskDeleted, task ->
    TaskDetailUiState(
            task = taskAsync.data,
            isTaskDeleted = isTaskDeleted
    )
  }
          // Convert the result to the appropriate observable API for the UI
          .stateIn(
                  scope = viewModelScope,
                  started = SharingStarted.WhileSubscribed(5_000),
                  initialValue = TaskDetailUiState()
          )

  fun deleteTask() = viewModelScope.launch {
    tasksRepository.deleteTask(taskId)
    _isTaskDeleted.update { true }
  }
}

//Compose State
class TaskDetailViewModel @Inject constructor(
        private val tasksRepository: TasksRepository,
        savedStateHandle: SavedStateHandle
) : ViewModel() {

  private var _isTaskDeleted by mutableStateOf(false)
  private val _task = tasksRepository.getTaskStream(taskId)

  val uiState: StateFlow<TaskDetailUiState> = combine(
          snapshotFlow { _isTaskDeleted },
          _task
  ) { isTaskDeleted, task ->
    TaskDetailUiState(
            task = taskAsync.data,
            isTaskDeleted = isTaskDeleted
    )
  }
          // Convert the result to the appropriate observable API for the UI
          .stateIn(
                  scope = viewModelScope,
                  started = SharingStarted.WhileSubscribed(5_000),
                  initialValue = TaskDetailUiState()
          )

  fun deleteTask() = viewModelScope.launch {
    tasksRepository.deleteTask(taskId)
    _isTaskDeleted = true
  }
}
```
<mark style="background-color: lightblue">Not: Compose State, [snapshotFlow { }](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#snapshotFlow(kotlin.Function0)) API kullanılarak bir flowa dönüştürülür. Başka bir örnek için "Now In Android" örneğindeki [ForYouViewModel](https://github.com/android/nowinandroid/blob/main/feature-foryou/src/main/java/com/google/samples/apps/nowinandroid/feature/foryou/ForYouViewModel.kt)'e bakın.</mark>

Output types in state production pipelines

UI State için output API'sinin seçimi ve sunumunun doğası, büyük ölçüde uygulamanızın UI’ini oluşturmak için kullandığı API'ye bağlıdır. Android uygulamalarında Views veya Jetpack Compose kullanmayı seçebilirsiniz. Buradaki hususlar şunları içerir:

* State’i [yaşam döngüsüne duyarlı](https://medium.com/androiddevelopers/consuming-flows-safely-in-jetpack-compose-cde014d0d5a3) bir şekilde okuma.

* State’in, state holderinden [bir veya daha fazla field’inden gösterilmesi](about-the-ui-layer.md#additional-considerations) gerekip gerekmediği.

Aşağıdaki tablo, herhangi bir input ve consumer için state production pipeline için hangi API'lerin kullanılacağını özetlemektedir:

| Input  | Consumer   | Output |
| --- |------------| --- |
| One-shot APIs | Views | StateFlow or LiveData |
| One-shot APIs | Compose | StateFlow or Compose State |
| Stream APIs | Views | StateFlow or LiveData |
| Stream APIs | Compose | StateFlow |
| One-shot and stream APIs | Views | StateFlow or LiveData |
| One-shot and stream APIs | Compose | StateFlow |

### State production pipeline initialization
State production pipeline'larının başlatılması, pipeline'ın çalışması için ilk koşulların ayarlanmasını içerir. Bu, örneğin bir haber makalesinin detail view'i için bir ID gibi pipeline'ın başlatılması için kritik olan ilk girdi değerlerinin sağlanmasını veya asenkron bir yüklemenin başlatılmasını içerebilir.

Sistem kaynaklarını korumak için state production pipeline'ı mümkün olduğunca lazily başlatmalısınız. Pratikte bu genellikle çıktının bir tüketicisi olana kadar beklemek anlamına gelir. Flow API'leri, [stateIn](https://developer.android.com/topic/architecture/ui-layer/17) metodundaki [started](https://developer.android.com/topic/architecture/ui-layer/31) argümanı ile buna izin verir. Bunun uygulanamaz olduğu durumlarda, aşağıdaki kod parçasında gösterildiği gibi state production pipeline'ı explicit olarak başlatmak için [idempotent](https://en.wikipedia.org/wiki/Idempotence) bir initialize() fonksiyonu tanımlayın:
```kotlin
class MyViewModel : ViewModel() {

    private var initializeCalled = false

    // This function is idempotent provided it is only called from the UI thread.
    @MainThread
    fun initialize() {
        if(initializeCalled) return
        initializeCalled = true

        viewModelScope.launch {
            // seed the state production pipeline
        }
    }
}
```
<mark style="background-color:red">Uyarı: Bir ViewModel'in init bloğunda veya constructor'ında asenkron işlemler başlatmaktan kaçının. Asenkron işlemler bir nesne oluşturmanın yan etkisi olmamalıdır çünkü asenkron kod, nesne tam olarak başlatılmadan önce nesneden okuyabilir veya nesneye yazabilir. Bu aynı zamanda leaking the object (nesnenin sızdırılması) olarak da adlandırılır ve ince ve teşhis edilmesi zor hatalara yol açabilir. Bu özellikle Compose State ile çalışırken önemlidir. ViewModel Compose State fieldlarini tuttuğunda, ViewModel'in init bloğunda Compose State fieldlarını güncelleyen bir Coroutine başlatmayın, aksi takdirde bir IllegalStateException oluşabilir.
</mark>

### [Samples](https://developer.android.com/topic/architecture/ui-layer/state-production#samples)