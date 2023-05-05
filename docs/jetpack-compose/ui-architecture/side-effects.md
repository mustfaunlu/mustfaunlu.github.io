---
layout: default
title: Side-effects
parent: UI Architecture
grand_parent: Jetpack Compose
nav_order: 2
---

# Side-effects in Compose
Side-effect, bir composable fonksiyonun scope'u dışında gerçekleşen, uygulamanın state'indeki bir değişikliktir. Composable'ların yaşam döngüsü ve öngörülemeyen recompositionlar, farklı sıralarda composable'ların recompositionlarını yürütme veya atılabilen recompositionlar gibi özellikleri nedeniyle, [composable'lar ideal olarak side-effect içermemelidir](/docs/jetpack-compose/introduction/thinking-in-compose).

Bununla birlikte, bazen side-effectler, örneğin bir snackbar göstermek veya belirli bir state koşulunda başka bir ekrana gitmek gibi tek seferlik bir eylemi tetiklemek için gereklidir. Bu eylemler, composable'ın yaşam döngüsünün farkında olan kontrollü bir ortamdan çağrılmalıdır. Bu sayfada, Jetpack Compose'un sunduğu farklı side effect API'leri hakkında bilgi edineceksiniz.

## State and effect use cases
[Thinking in Compose](/docs/jetpack-compose/introduction/thinking-in-compose) belgesinde ele alındığı gibi, bileşenler side-effectsiz olmalıdır. Uygulamanın state'inde değişiklik yapmanız gerektiğinde ([Managing state](managing-state/managing-state) dokümantasyon belgesinde açıklandığı gibi), bu side-effectlerin öngörülebilir bir şekilde yürütülmesi için Effect API'lerini kullanmalısınız.

{: .note }
Anahtar Terim: Efekt, kullanıcı arayüzü üretmeyen ve bir compostion tamamlandığında side effectlerin çalışmasına neden olan composable bir fonksiyondur.

Compose'da açılan farklı olasılıklar nedeniyle, efektler kolayca aşırı kullanılabilir. İçlerinde yaptığınız işin UI ile ilgili olduğundan ve [Managing state](managing-state/managing-state) belgelerinde açıklandığı gibi tek yönlü veri akışını(Undirectional data flow) bozmadığından emin olun.

{: .note }
Not: Responsive bir kullanıcı arayüzü doğası gereği asenkrondur ve Jetpack Compose bunu callback kullanmak yerine API seviyesinde coroutine'leri benimseyerek çözer. Coroutines hakkında daha fazla bilgi edinmek için [Android'de Kotlin coroutines kılavuzu](https://developer.android.com/kotlin/coroutines)na göz atın.

### LaunchedEffect: bir composable scope'unda suspend fonksiyonları çalıştırır
Suspend fonksiyonlarını bir composable'ın içinden güvenli bir şekilde çağırmak için LaunchedEffect composable'ını kullanın. LaunchedEffect Composition'a girdiğinde, parametre olarak aktarılan kod bloğu ile bir coroutine başlatır. LaunchedEffect composition'dan ayrılırsa coroutine iptal edilir. LaunchedEffect farklı key'lerle yeniden oluşturulursa (aşağıdaki [Restarting effects](#restarting-effects) bölümüne bakın), mevcut coroutine iptal edilir ve yeni askıya alma fonksiyonu yeni bir coroutine'de başlatılır.

Örneğin, bir Snackbar'ın bir Scaffold'da gösterilmesi) bir suspend fonksiyonu olan SnackbarHostState.showSnackbar fonksiyonu ile yapılır.

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun MyScreen(
    state: UiState<List<Movie>>,
    snackbarHostState: SnackbarHostState
) {

    // UI state bir hata içeriyorsa, snackbar'ı göster
    if (state.hasError) {

        // LaunchedEffect`, `scaffoldState.snackbarHostState` değişirse 
        // iptal edilecek ve yeniden başlatılacaktır 
        LaunchedEffect(snackbarHostState) {
            
            //Bir coroutine kullanarak snackbar'ı gösterin, coroutine iptal edildiğinde snackbar 
            //otomatik olarak kapanacaktır. state.hasError` false olduğunda ve yalnızca 
            //`state.hasError` true olduğunda başlayın (yukarıdaki if kontrolü nedeniyle) veya 
            //`scaffoldState.snackbarHostState` değişirse.
             
            snackbarHostState.showSnackbar(
                message = "Error message",
                actionLabel = "Retry message"
            )
        }
    }

    Scaffold(
        snackbarHost = {
            SnackbarHost(hostState = snackbarHostState)
        }
    ) { contentPadding ->
        // ...
    }
}
```

Yukarıdaki kodda, state bir hata içeriyorsa bir coroutine tetiklenir ve içermediğinde iptal edilir. LaunchedEffect call site'i bir if ifadesi içinde olduğundan, ifade false olduğunda, eğer LaunchedEffect Composition içindeyse, kaldırılacak ve dolayısıyla coroutine iptal edilecektir.

### rememberCoroutineScope: composable bir coroutine dışında bir coroutine başlatmak için composition-aware bir scope elde etmek
LaunchedEffect bir composable fonksiyon olduğundan, yalnızca diğer composable fonksiyonların içinde kullanılabilir. Bir coroutine'i bir composable dışında başlatmak, ancak composition'dan ayrıldığında otomatik olarak iptal edilecek şekilde scoplamak için rememberCoroutineScope kullanın. Ayrıca, bir veya daha fazla coroutine'in yaşam döngüsünü manuel olarak kontrol etmeniz gerektiğinde, örneğin bir kullanıcı olayı gerçekleştiğinde bir animasyonu iptal etmek için rememberCoroutineScope kullanın.

[rememberCoroutineScope](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#rememberCoroutineScope(kotlin.Function0)), çağrıldığı Composition noktasına bağlı bir CoroutineScope döndüren composable bir fonksiyondur. Çağrı Composition'dan ayrıldığında scope iptal edilecektir.

Önceki örneği izleyerek, kullanıcı bir Button'a dokunduğunda bir Snackbar göstermek için bu kodu kullanabilirsiniz:

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun MoviesScreen(snackbarHostState: SnackbarHostState) {

    // MoviesScreen'in yaşam döngüsüne bağlı bir CoroutineScope oluşturur
    val scope = rememberCoroutineScope()

    Scaffold(
        snackbarHost = {
            SnackbarHost(hostState = snackbarHostState)
        }
    ) { contentPadding ->
        Column(Modifier.padding(contentPadding)) {
            Button(
                onClick = {
                    // Snackbar'ı göstermek için event handler'da yeni bir coroutine oluşturun
                    scope.launch {
                        snackbarHostState.showSnackbar("Something happened!")
                    }
                }
            ) {
                Text("Press me")
            }
        }
    }
}
```

### rememberUpdatedState: değer değiştiğinde yeniden başlatılmaması gereken bir efektteki bir değere referans verir
Key parametrelerden biri değiştiğinde LaunchedEffect yeniden başlar. Ancak, bazı durumlarda efektinizde, değişmesi halinde efektin yeniden başlamasını istemediğiniz bir değeri yakalamak isteyebilirsiniz. Bunu yapmak için, bu değere yakalanabilecek ve güncellenebilecek bir referans oluşturmak üzere rememberUpdatedState kullanılması gerekir. Bu yaklaşım, yeniden oluşturulması ve yeniden başlatılması pahalı veya engelleyici olabilecek uzun ömürlü işlemler içeren efektler için yararlıdır.

Örneğin, uygulamanızda bir süre sonra kaybolan bir LandingScreen olduğunu varsayalım. LandingScreen yeniden oluşturulsa bile, bir süre bekleyen ve zamanın geçtiğini bildiren efekt yeniden başlatılmamalıdır:
```kotlin
@Composable
fun LandingScreen(onTimeout: () -> Unit) {

    // Bu her zaman LandingScreen'in yeniden oluşturulduğu 
    // en son onTimeout fonksiyonuna referansta bulunacaktır
    val currentOnTimeout by rememberUpdatedState(onTimeout)

    // LandingScreen'in yaşam döngüsüyle eşleşen bir efekt oluşturun.
    // LandingScreen yeniden oluşturulursa, delay yeniden başlamamalıdır.
    LaunchedEffect(true) {
        delay(SplashWaitTimeMillis)
        currentOnTimeout()
    }

    /* Landing screen content */
}
```

Call site'in yaşam döngüsüyle eşleşen bir efekt oluşturmak için Unit veya true gibi hiç değişmeyen bir constant parametre olarak geçirilir. Yukarıdaki kodda, LaunchedEffect(true) kullanılmıştır. onTimeout lambda'sının her zaman LandingScreen'in yeniden oluşturulduğu en son değeri içerdiğinden emin olmak için, onTimeout'un rememberUpdatedState fonksiyonuyla sarılması gerekir. Döndürülen State, koddaki currentOnTimeout, efektte kullanılmalıdır.

{: .warning}
Uyarı: LaunchedEffect(true), while(true) kadar şüphelidir. Bunun için geçerli kullanım durumları olsa da, her zaman duraklayın ve ihtiyacınız olanın bu olduğundan emin olun.

### DisposableEffect: temizlenmesi gereken efektler
Key'ler değiştikten sonra veya composable Composition'dan ayrıldığında temizlenmesi gereken side-effectler için [DisposableEffect](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#DisposableEffect(kotlin.Any,kotlin.Function1)) kullanın. DisposableEffect key'leri değişirse, composable'ın mevcut efektini dispose etmesi (temizlemesi) ve efekti tekrar çağırarak sıfırlaması gerekir.

Örnek olarak, bir [LifecycleObserver](https://developer.android.com/reference/androidx/lifecycle/LifecycleObserver) kullanarak [Yaşam Döngüsü olayları](/docs/app-architecture/architecture-components/ui-layer-libraries/lifecycle-aware-components/handle-lifecycles)na dayalı analiz olayları göndermek isteyebilirsiniz. Compose'da bu olayları dinlemek için, gerektiğinde gözlemciyi kaydetmek ve kaydını kaldırmak üzere bir DisposableEffect kullanın.

```kotlin
@Composable
fun HomeScreen(
    lifecycleOwner: LifecycleOwner = LocalLifecycleOwner.current,
    onStart: () -> Unit, // 'Başlatıldı' analitik olayını gönderin
    onStop: () -> Unit // 'Durduruldu' analitik olayını gönderin
) {
    // Yeni bir lambda sağlandığında mevcut lambdaları güvenli bir şekilde güncelleyin
    val currentOnStart by rememberUpdatedState(onStart)
    val currentOnStop by rememberUpdatedState(onStop)

    // Eğer `lifecycleOwner` değişirse, efekti atın ve sıfırlayın
    DisposableEffect(lifecycleOwner) {
        // Analitik olayları göndermek için hatırlanan geri aramalarımızı (remembered callbacks) 
        // tetikleyen bir gözlemci oluşturun
        val observer = LifecycleEventObserver { _, event ->
            if (event == Lifecycle.Event.ON_START) {
                currentOnStart()
            } else if (event == Lifecycle.Event.ON_STOP) {
                currentOnStop()
            }
        }

        // Gözlemciyi yaşam döngüsüne ekleme
        lifecycleOwner.lifecycle.addObserver(observer)

        // Efekt Composition'dan ayrıldığında, gözlemciyi kaldırın
        onDispose {
            lifecycleOwner.lifecycle.removeObserver(observer)
        }
    }

    /* Home screen content */
}
```

Yukarıdaki kodda, efekt gözlemciyi lifecycleOwner'a ekleyecektir. LifecycleOwner değişirse, efekt dispose edilir ve yeni lifecycleOwner ile yeniden başlatılır.

Bir DisposableEffect, kod bloğundaki son deyim olarak bir onDispose cümlesi içermelidir. Aksi takdirde, IDE derleme zamanı hatası görüntüler.

{: .note}
Not: onDispose'da boş bir blok olması iyi bir pratik değildir. Kullanım durumunuza daha iyi uyan bir efekt olup olmadığını görmek için her zaman yeniden düşünün

### SideEffect: Compose state'i compose olmayan koda yayınla
Compose state'i compose tarafından yönetilmeyen nesnelerle paylaşmak için [SideEffect](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#SideEffect(kotlin.Function0)) composable'ı kullanın, çünkü her başarılı recomposition'da çağrılır.

Örneğin, analiz kütüphaneniz, sonraki tüm analiz olaylarına özel meta veriler (bu örnekte "kullanıcı özellikleri") ekleyerek kullanıcı popülasyonunuzu bölümlere ayırmanıza izin verebilir. Geçerli kullanıcının kullanıcı türünü analiz kütüphanenize iletmek için, değerini güncellemek üzere SideEffect'i kullanın.

```kotlin
@Composable
fun rememberFirebaseAnalytics(user: User): FirebaseAnalytics {
    val analytics: FirebaseAnalytics = remember {
        FirebaseAnalytics()
    }

    // Her başarılı composition'da, FirebaseAnalytics'i mevcut Kullanıcının userType'ı ile 
    // güncelleyerek gelecekteki analizlerin olaylarına 
    // bu meta veri eklenmiştir
    SideEffect {
        analytics.setUserProperty("userType", user.userType)
    }
    return analytics
}
```

### produceState: Compose olmayan state'i Compose state'e dönüştürür
[produceState](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#produceState(kotlin.Any,kotlin.coroutines.SuspendFunction1)), değerleri döndürülen bir State'e aktarabilen Composition scope'ndaki bir coroutine'i başlatır. Compose olmayan state'i Compose state'e dönüştürmek için kullanın, örneğin Flow, LiveData veya RxJava gibi harici abonelik odaklı state'i Composition'a getirmek gibi.

Producer, produceState Composition'a girdiğinde başlatılır ve Composition'dan çıktığında iptal edilir. Döndürülen State birleştirilir; aynı değerin ayarlanması recomposition'u tetiklemez.

produceState bir coroutine oluştursa da, non-suspending veri kaynaklarını gözlemlemek için de kullanılabilir. Bu kaynağa aboneliği kaldırmak için [awaitDispose](https://developer.android.com/reference/kotlin/androidx/compose/runtime/ProduceStateScope#awaitDispose(kotlin.Function0)) fonksiyonunu kullanın.

Aşağıdaki örnekte, ağdan bir görüntü yüklemek için produceState'in nasıl kullanılacağı gösterilmektedir. loadNetworkImage composable fonksiyonu, diğer composable'larda kullanılabilecek bir State döndürür.

```kotlin
@Composable
fun loadNetworkImage(
    url: String,
    imageRepository: ImageRepository = ImageRepository()
): State<Result<Image>> {

    //Başlangıç değeri olarak Result.Loading ile bir State<T> oluşturur 
    // Eğer `url` veya `imageRepository` değişirse, çalışan producer iptal olur
    // ve yeni girdilerle yeniden başlatılır.
    return produceState<Result<Image>>(initialValue = Result.Loading, url, imageRepository) {

        // Bir coroutine içinde, suspend çağrıları yapabilir
        val image = imageRepository.load(url)

        // State'i bir Error veya Success sonucuyla güncelleyin.
        // Bu, bu State'in okunduğu bir recomposition'ı tetikleyecektir
        value = if (image == null) {
            Result.Error
        } else {
            Result.Success(image)
        }
    }
}
```
{: .note}
Not: Geri dönüş türüne sahip Composable'lar, normal bir Kotlin fonksiyonunu adlandırdığınız şekilde, küçük harfle başlayarak adlandırılmalıdır.

{: .note}
Anahtar Nokta: Kaputun altında, produceState diğer efektlerden yararlanır! remember { mutableStateOf(initialValue) } kullanarak bir sonuç değişkenini tutar ve bir LaunchedEffect'teki producer bloğunu tetikler. Producer bloğunda değer her güncellendiğinde, sonuç durumu yeni değere güncellenir.
Mevcut API'lerin üzerine inşa ederek kendi efektlerinizi kolayca oluşturabilirsiniz.

### derivedStateOf: bir veya birden fazla state nesnesini başka bir state'e dönüştürür
Belirli bir state hesaplandığında veya diğer state nesnelerinden türetildiğinde [derivedStateOf](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#derivedStateOf(kotlin.Function0)) fonksiyonunu kullanın. Bu fonksiyonun kullanılması, hesaplamanın yalnızca hesaplamada kullanılan state'lerden biri değiştiğinde gerçekleşeceğini garanti eder.

Aşağıdaki örnekte, kullanıcı tanımlı yüksek öncelikli anahtar kelimelere sahip görevlerin önce göründüğü temel bir Yapılacaklar listesi gösterilmektedir:

```kotlin
@Composable
fun TodoList(highPriorityKeywords: List<String> = listOf("Review", "Unblock", "Compose")) {

    val todoTasks = remember { mutableStateListOf<String>() }

    //Yüksek öncelikli görevleri yalnızca todoTasks veya highPriorityKeywords olduğunda hesaplayın 
    //değişim, her recomposition'da değil
    val highPriorityTasks by remember(highPriorityKeywords) {
        derivedStateOf {
            todoTasks.filter { task ->
                highPriorityKeywords.any { keyword ->
                    task.contains(keyword)
                }
            }
        }
    }

    Box(Modifier.fillMaxSize()) {
        LazyColumn {
            items(highPriorityTasks) { /* ... */ }
            items(todoTasks) { /* ... */ }
        }
        /* Rest of the UI where users can add elements to the list */
    }
}
```
Yukarıdaki kodda, derivedStateOf todoTasks değiştiğinde highPriorityTasks hesaplamasının gerçekleşeceğini ve kullanıcı arayüzünün buna göre güncelleneceğini garanti eder. highPriorityKeywords değişirse, remember bloğu çalıştırılacak ve yeni bir türetilmiş state nesnesi oluşturulacak ve eskisinin yerine hatırlanacaktır. highPriorityTasks'ı hesaplamak için filtreleme pahalı olabileceğinden, her recomposition'da değil, yalnızca listelerden herhangi biri değiştiğinde çalıştırılmalıdır.

Ayrıca, derivedStateOf tarafından üretilen state'te yapılan bir güncelleme, bildirildiği composable'ın yeniden oluşturulmasına neden olmaz, Compose yalnızca döndürülen state'in okunduğu composable'ları yeniden oluşturur, örnekteki LazyColumn içinde.

Kod ayrıca highPriorityKeywords'ün todoTasks'tan çok daha az sıklıkta değiştiğini varsayar. Eğer durum böyle olmasaydı, kod derivedStateOf yerine remember(todoTasks, highPriorityKeywords) kullanabilirdi.

### snapshotFlow: Compose'un State'ini Flow'lara dönüştürür
[State<T> ](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State) nesnelerini cold Flow'a dönüştürmek için [snapshotFlow](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#snapshotFlow(kotlin.Function0)) kullanın. snapshotFlow collect edidiginde bloğunu çalıştırır ve içinde okunan State nesnelerinin sonucunu yayar(emit eder). snapshotFlow bloğu içinde okunan State nesnelerinden biri değiştiğinde, yeni değer önceki yayılan değere [eşit](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-any/equals.html) değilse Flow yeni değeri collector'ına yayar (bu davranış [Flow.distinctUntilChanged](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/distinct-until-changed.html)'e benzer).

Aşağıdaki örnekte, kullanıcı bir listedeki ilk öğeyi analize scroll ettiginde kayıt yapan bir side effect gösterilmektedir:
```kotlin
val listState = rememberLazyListState()

LazyColumn(state = listState) {
    // ...
}

LaunchedEffect(listState) {
    snapshotFlow { listState.firstVisibleItemIndex }
        .map { index -> index > 0 }
        .distinctUntilChanged()
        .filter { it == true }
        .collect {
            MyAnalyticsService.sendScrolledPastFirstItemEvent()
        }
}
```
Yukarıdaki kodda, listState.firstVisibleItemIndex, Flow'un operatörlerinin gücünden yararlanabilen bir Flow'a dönüştürülür.

## Restarting effects
Compose'daki LaunchedEffect, produceState veya DisposableEffect gibi bazı efektler, çalışan efekti iptal etmek ve yeni key'lerle yeni bir efekt başlatmak için kullanılan değişken sayıda argüman, key alır.

Bu API'ler için tipik biçim şöyledir:
```kotlin
EffectName(restartIfThisKeyChanges, orThisKey, orThisKey, ...) { block }
```
Bu davranışın incelikleri nedeniyle, efekti yeniden başlatmak için kullanılan parametreler doğru değilse sorunlar ortaya çıkabilir:

* Efektleri olması gerekenden daha az yeniden başlatmak uygulamanızda hatalara neden olabilir.
* Efektleri olması gerekenden daha fazla yeniden başlatmak verimsiz olabilir.

Genel bir kural olarak, efekt kod bloğunda kullanılan mutable ve immutable değişkenler efekt composable'a parametre olarak eklenmelidir. Bunların dışında, efekti yeniden başlatmaya zorlamak için daha fazla parametre eklenebilir. Eğer bir değişkenin değişmesi efektin yeniden başlamasına neden olmuyorsa, değişken rememberUpdatedState'e sarılmalıdır. Değişken hiçbir zaman değişmiyorsa, çünkü hiçbir key'i olmayan bir remember'a sarılmışsa, değişkeni efekte key olarak geçirmenize gerek yoktur.

{: .note}
Önemli Nokta: Bir efektte kullanılan değişkenler, efektin composable parametresi olarak eklenmeli veya rememberUpdatedState kullanılmalıdır.

Yukarıda gösterilen DisposableEffect kodunda, efekt parametre olarak bloğunda kullanılan lifecycleOwner'ı alır, çünkü bunlarda yapılacak herhangi bir değişiklik efektin yeniden başlamasına neden olmalıdır.

```kotlin
@Composable
fun HomeScreen(
    lifecycleOwner: LifecycleOwner = LocalLifecycleOwner.current,
    onStart: () -> Unit, // 'started' analiz olayını gönder
    onStop: () -> Unit // 'stopped' analiz olayını gönder
) {
    // Bu değerler Composition'da asla değişmez
    val currentOnStart by rememberUpdatedState(onStart)
    val currentOnStop by rememberUpdatedState(onStop)

    DisposableEffect(lifecycleOwner) {
        val observer = LifecycleEventObserver { _, event ->
            /* ... */
        }

        lifecycleOwner.lifecycle.addObserver(observer)
        onDispose {
            lifecycleOwner.lifecycle.removeObserver(observer)
        }
    }
}
```

currentOnStart ve currentOnStop, DisposableEffect key'leri olarak gerekli değildir, çünkü rememberUpdatedState kullanımı nedeniyle değerleri Composition'da asla değişmez. LifecycleOwner'ı bir parametre olarak geçmezseniz ve değişirse, HomeScreen yeniden oluşturulur, ancak DisposableEffect atılmaz ve yeniden başlatılmaz. Bu sorunlara neden olur çünkü o noktadan itibaren yanlış lifecycleOwner kullanılır.

### Key olarak sabitler

Call site'in yaşam döngüsünü takip etmesini sağlamak için efekt key'i olarak true gibi bir sabit kullanabilirsiniz. Yukarıda gösterilen LaunchedEffect örneğinde olduğu gibi, bunun için geçerli kullanım durumları vardır. Ancak, bunu yapmadan önce iki kez düşünün ve ihtiyacınız olanın bu olduğundan emin olun.