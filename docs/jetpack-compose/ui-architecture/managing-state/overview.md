---
layout: default
title: Overview
parent: Managing State
grand_parent: UI Architecture
nav_order: 1
---
# State and Jetpack Compose
Bir uygulamadaki state, zaman içinde değişebilen herhangi bir değerdir. Bu çok geniş bir tanımdır ve bir Room veritabanından bir sınıftaki değişkene kadar her şeyi kapsar.

Tüm Android uygulamaları kullanıcıya state gösterir. Android uygulamalarında birkaç state örneği:

* Bir ağ bağlantısı kurulamadığında bunu gösteren bir Snackbar.
* Bir blog yazısı ve ilgili yorumlar.
* Kullanıcı tıkladığında oynatılan butonlar üzerindeki dalgalanma animasyonları.
* Kullanıcının bir resmin üzerine çizebileceği çıkartmalar.

Jetpack Compose, bir Android uygulamasında state'i nerede ve nasıl sakladığınız ve kullandığınız konusunda açık olmanıza yardımcı olur. Bu kılavuz, state ve composables arasındaki bağlantıya ve Jetpack Compose'un state ile daha kolay çalışmak için sunduğu API'lere odaklanmaktadır.

[Jetpack Compose: State](https://youtu.be/mymWGMy9pYI)

## State and composition
Compose deklaratiftir ve bu nedenle onu güncellemenin tek yolu aynı composable'ı yeni argümanlarla çağırmaktır. Bu argümanlar kullanıcı arayüzü state'inin temsilleridir. Bir state her güncellendiğinde recomposition gerçekleşir. Sonuç olarak, TextField gibi şeyler emperatif XML tabanlı view'larda olduğu gibi otomatik olarak güncellenmez. Bir composable'ın uygun şekilde güncellenebilmesi için yeni state'in açıkça söylenmesi gerekir.

```kotlin
@Composable
private fun HelloContent() {
    Column(modifier = Modifier.padding(16.dp)) {
        Text(
            text = "Hello!",
            modifier = Modifier.padding(bottom = 8.dp),
            style = MaterialTheme.typography.bodyMedium
        )
        OutlinedTextField(
            value = "",
            onValueChange = { },
            label = { Text("Name") }
        )
    }
}
```
Bunu çalıştırırsanız, hiçbir şey olmadığını göreceksiniz. Bunun nedeni TextField'ın kendisini güncellememesi, value parametresi değiştiğinde güncellemesidir. Bu, Compose'da composition ve recomposition'ın nasıl çalıştığından kaynaklanmaktadır.

{:.note}
Anahtar Terim: Composition: Jetpack Compose tarafından composables çalıştırıldığında oluşturulan kullanıcı arayüzünün bir açıklaması.
Initial composition (İlk kompozisyon): composables'ı ilk kez çalıştırarak bir Composition oluşturulması.
Recomposition: veri değiştiğinde Composition'ı güncellemek için composables'ın yeniden çalıştırılması.


Initial composition ve recomposition hakkında daha fazla bilgi edinmek için [Thinking in Compose](/docs/jetpack-compose/introduction/thinking-in-compose) bölümüne bakınız.


## State in composables
Composable fonksiyonlar bir nesneyi bellekte saklamak için [remember](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#remember(kotlin.Function0)) API'sini kullanabilir. remember tarafından hesaplanan bir değer initial composition sırasında Composition'da saklanır ve saklanan değer recomposition sırasında döndürülür. remember hem mutable hem de immutable nesneleri saklamak için kullanılabilir.

{:.note}
Not: remember nesneleri Composition'da saklar ve remember'ı çağıran composable Composition'dan kaldırıldığında nesneyi unutur.

[mutableStateOf](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#mutableStateOf(kotlin.Any,androidx.compose.runtime.SnapshotMutationPolicy)), compose çalışma zamanıyla entegre edilmiş gözlemlenebilir bir tür olan gözlemlenebilir bir [MutableState<T>](https://developer.android.com/reference/kotlin/androidx/compose/runtime/MutableState) oluşturur.

```kotlin
interface MutableState<T> : State<T> {
    override var value: T
}
```
Value'da yapılan herhangi bir değişiklik, value'yu okuyan tüm composable fonksiyonların yeniden oluşturulmasını planlar. ExpandingCard durumunda, expanded her değiştiğinde, ExpandingCard'ın yeniden oluşturulmasına neden olur.

Bir MutableState nesnesini bir composable içinde bildirmenin üç yolu vardır:

- val mutableState = remember { mutableStateOf(default) }
- var value by remember { mutableStateOf(default) }
- val (value, setValue) = remember { mutableStateOf(default) }

Bu bildirimler eşdeğerdir ve farklı state kullanımları için syntax sugar olarak sağlanmıştır. Yazdığınız composable'da okunması en kolay kodu üreteni seçmelisiniz.

By delegate syntax aşağıdaki import'ları gerektirir:
```kotlin
import androidx.compose.runtime.getValue
import androidx.compose.runtime.setValue
```

Remembered değeri diğer composable'lar için bir parametre olarak veya hatta hangi composable'ların görüntüleneceğini değiştirmek için ifadelerde logic olarak kullanabilirsiniz. Örneğin, ad boşsa karşılama mesajını görüntülemek istemiyorsanız, state'i bir if ifadesinde kullanın:

```kotlin
@Composable
fun HelloContent() {
    Column(modifier = Modifier.padding(16.dp)) {
        var name by remember { mutableStateOf("") }
        if (name.isNotEmpty()) {
            Text(
                text = "Hello, $name!",
                modifier = Modifier.padding(bottom = 8.dp),
                style = MaterialTheme.typography.bodyMedium
            )
        }
        OutlinedTextField(
            value = name,
            onValueChange = { name = it },
            label = { Text("Name") }
        )
    }
}
```

remember, recompositionlarda state'i korumanıza yardımcı olsa da, yapılandırma değişikliklerinde state korunmaz. Bunun için rememberSaveable kullanmanız gerekir. rememberSaveable, bir Bundle'a kaydedilebilen tüm değerleri otomatik olarak kaydeder. Diğer değerler için özel bir kaydedici nesnesi iletebilirsiniz.

{: .warning}
Dikkat: Compose'da state olarak ArrayList<T> veya mutableListOf() gibi mutable nesnelerin kullanılması, kullanıcılarınızın uygulamanızda yanlış veya eski veriler görmesine neden olur. ArrayList veya mutable veri sınıfı gibi gözlemlenemeyen mutable nesneler Compose tarafından gözlemlenemez ve değiştiklerinde yeniden birleştirme tetiklemez. Gözlemlenebilir olmayan değişken nesneler kullanmak yerine, State<List<T>> ve değişmez listOf() gibi gözlemlenebilir bir veri tutucu kullanmanız önerilir.



## Other supported types of state
Compose, state tutmak için MutableState<T> kullanmanızı gerektirmez; diğer gözlemlenebilir türleri destekler. Compose'da başka bir gözlemlenebilir türü okumadan önce, onu bir State<T>'ye dönüştürmelisiniz, böylece state değiştiğinde composablelar otomatik olarak yeniden oluşturabilir.

Compose, Android uygulamalarında kullanılan yaygın gözlemlenebilir türlerden State<T> oluşturmak için fonksiyonlarla birlikte gelir. Bu entegrasyonları kullanmadan önce, aşağıda belirtildiği gibi uygun artifact(lar)ı ekleyin:

- [Flow](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/index.html): [collectAsStateWithLifecycle()](https://developer.android.com/reference/kotlin/androidx/lifecycle/compose/package-summary#extension-functions):
collectAsStateWithLifecycle(), bir Flow'dan değerleri yaşam döngüsüne duyarlı bir şekilde toplayarak uygulamanızın gereksiz uygulama kaynaklarından tasarruf etmesini sağlar. Compose [State](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State) aracılığıyla en son yayılan(emit edilen) değeri temsil eder. Android uygulamalarında Flow'ları collect etmek için önerilen yol olarak bu API'yi kullanın.
 
   {: .note}
   Not: collectAsStateWithLifecycle() API'si ile Android'de flowlari güvenli bir şekilde collect etme hakkında daha fazla bilgi edinmek için [bu blog yazısı](https://medium.com/androiddevelopers/consuming-flows-safely-in-jetpack-compose-cde014d0d5a3)nı okuyabilirsiniz.

  build.gradle dosyasında aşağıdaki bağımlılık gereklidir (2.6.0-beta01 veya daha yeni olmalıdır):
```groovy
dependencies {
      ...
      implementation "androidx.lifecycle:lifecycle-runtime-compose:2.6.0-beta01"
}
```

- [Flow](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/index.html): [collectAsState()](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#(kotlinx.coroutines.flow.StateFlow).collectAsState(kotlin.coroutines.CoroutineContext))
collectAsState, collectAsStateWithLifecycle'a benzer, çünkü aynı zamanda bir Flow'dan değerleri collect eder ve bunları Compose State'e dönüştürür.

    Platformdan bağımsız kod için yalnızca Android'e özel olan collectAsStateWithLifecycle yerine collectAsState'i kullanın.

    collectAsState için ek bağımlılıklar gerekli değildir, çünkü compose-runtime'da mevcuttur.


- [LiveData](https://developer.android.com/reference/kotlin/androidx/compose/runtime/livedata/package-summary): [observeAsState()](https://developer.android.com/reference/kotlin/androidx/compose/runtime/livedata/package-summary#(androidx.lifecycle.LiveData).observeAsState(kotlin.Any))
  observeAsState() bu [LiveData](https://developer.android.com/reference/kotlin/androidx/lifecycle/LiveData)'yı gözlemlemeye başlar ve değerlerini [State](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State) aracılığıyla temsil eder.

    build.gradle dosyasında aşağıdaki [bağımlılık](https://developer.android.com/jetpack/androidx/releases/compose-runtime) gereklidir:
```groovy
dependencies {
      ...
      implementation "androidx.compose.runtime:runtime-livedata:1.4.2"
}
```

- [RxJava2](https://developer.android.com/reference/kotlin/androidx/compose/runtime/rxjava2/package-summary): [subscribeAsState()](https://developer.android.com/reference/kotlin/androidx/compose/runtime/rxjava2/package-summary#extension-functions)
  subscribeAsState(), RxJava2'nin reaktif stream'lerini (örneğin Single, Observable, Completable) Compose State'e dönüştüren extension fonksiyonlarıdır.

    Build.gradle dosyasında aşağıdaki bağımlılık gereklidir:
```groovy
dependencies {
      ...
      implementation "androidx.compose.runtime:runtime-rxjava2:1.4.2"
}
```

- [RxJava3](https://developer.android.com/reference/kotlin/androidx/compose/runtime/rxjava3/package-summary): [subscribeAsState()](https://developer.android.com/reference/kotlin/androidx/compose/runtime/rxjava3/package-summary#extension-functions)
  subscribeAsState(), RxJava3'ün reaktif stream'lerini (örneğin Single, Observable, Completable) Compose State'e dönüştüren extension fonksiyonlarıdır.

    build.gradle dosyasında aşağıdaki bağımlılık gereklidir:
```groovy
dependencies {
      ...
      implementation "androidx.compose.runtime:runtime-rxjava3:1.4.2"
}
```

{: .note}
Anahtar Nokta: Compose, State nesnelerini okurken otomatik olarak yeniden birleştirir. Compose'da LiveData gibi başka bir gözlemlenebilir tür kullanıyorsanız, okumadan önce bunu State'e dönüştürmelisiniz. Bu tür dönüştürme işleminin LiveData<T>.observeAsState() gibi bir composable extension fonksiyonu kullanarak composable içinde gerçekleştiğinden emin olun.


{: .note}
Not: Bu entegrasyonlarla sınırlı değilsiniz. Jetpack Compose için diğer gözlemlenebilir türleri okuyan bir extension fonksiyonu oluşturabilirsiniz. Uygulamanız özel bir gözlemlenebilir sınıf kullanıyorsa, [produceState](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#produceState(kotlin.Any,kotlin.coroutines.SuspendFunction1)) API'sini kullanarak bunu State<T> üretecek şekilde dönüştürün.

Bunun nasıl yapılacağına dair örnekler için yerleşiklerin uygulamasına bakın: [collectAsStateWithLifecycle](https://cs.android.com/androidx/platform/frameworks/support/+/androidx-main:lifecycle/lifecycle-runtime-compose/src/main/java/androidx/lifecycle/compose/FlowExt.kt;l=168?q=collectAsStateWithLifecycle). Jetpack Compose'un her değişikliğe abone olmasını sağlayan herhangi bir nesne State<T>'ye dönüştürülebilir ve bir Composable'da okunabilir.


### Stateful versus stateless
Bir nesneyi saklamak için remember kullanan bir composable, dahili state oluşturarak composable'ı stateful yapar. HelloContent, name state'ini dahili olarak tuttuğu ve değiştirdiği için stateful bir composable örneğidir. Bu, caller'ın state'i kontrol etmesine gerek olmadığı ve state'i kendisi yönetmek zorunda kalmadan kullanabileceği durumlarda faydalı olabilir. Bununla birlikte, dahili state'e sahip composable'lar daha az yeniden kullanılabilir ve test edilmesi daha zor olma eğilimindedir.


Stateless composable, herhangi bir state tutmayan bir composable'dır. State tutmamayı başarmanın kolay bir yolu [state hoisting](#state-hoisting) kullanmaktır.

Yeniden kullanılabilir composable'lar geliştirirken, genellikle aynı composable'ın hem stateful hem de stateeless versiyonunu ortaya çıkarmak istersiniz. Stateful sürüm, state'i önemsemeyen caller'lar için kullanışlıdır ve state'i kontrol etmesi veya kaldırması gereken caller'lar için stateless sürüm gereklidir.


## State hoisting
Compose'da state hoisting, bir composable'ı stateless yapmak için state'i composable'ın caller'ına taşıma yoludur. Jetpack Compose'da state hoisting için genel şablon, state değişkenini iki parametre ile değiştirmektir:

- `value`: T: görüntülenecek geçerli değer
- `onValueChange: (T) -> Unit`: değerin değişmesini isteyen bir olay, burada T önerilen yeni değerdir

Ancak, onValueChange ile sınırlı değilsiniz. Composable için daha spesifik olaylar uygunsa, ExpandingCard'ın onExpand ve onCollapse ile yaptığı gibi bunları lambda kullanarak tanımlamalısınız.

Bu şekilde kaldırılan state bazı önemli özelliklere sahiptir:

- **Single source of truth (Tek doğruluk kaynağı)**: State'i çoğaltmak yerine taşıyarak yalnızca tek bir doğruluk kaynağı olmasını sağlıyoruz. Bu, hataları önlemeye yardımcı olur.

- **Encapsulated**: Yalnızca stateful composable'lar state'lerini değiştirebilir. Tamamen içseldir.

- **Shareable:** Kaldırılmış state birden fazla composable ile paylaşılabilir. Eğer farklı bir composable'daki ismi okumak isterseniz, hoisting bunu yapmanıza izin verir.

- **Interceptable**: stateless composable'ları caller'lar state'i değiştirmeden önce event'leri görmezden gelmeye veya değiştirmeye karar verebilirler.

- **Decoupled**: stateless ExpandingCard için state herhangi bir yerde saklanabilir. Örneğin, artık ismi bir ViewModel'e taşımak mümkündür.

Örnek durumda, name ve onValueChange'i HelloContent'ten çıkarır ve bunları ağaçta HelloContent'i çağıran bir HelloScreen composable'ına taşırsınız.

```kotlin
@Composable
fun HelloScreen() {
  var name by rememberSaveable { mutableStateOf("") }

  HelloContent(name = name, onNameChange = { name = it })
}

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun HelloContent(name: String, onNameChange: (String) -> Unit) {
  Column(modifier = Modifier.padding(16.dp)) {
    Text(
            text = "Hello, $name",
            modifier = Modifier.padding(bottom = 8.dp),
            style = MaterialTheme.typography.bodyMedium
    )
    OutlinedTextField(value = name, onValueChange = onNameChange, label = { Text("Name") })
  }
}
```
State'i HelloContent'ten kaldırarak, composable hakkında mantık yürütmek, farklı durumlarda yeniden kullanmak ve test etmek daha kolaydır. HelloContent, state'inin nasıl depolandığından ayrıştırılmıştır. Ayrıştırma(decoupled), HelloScreen'i değiştirdiğinizde ya da yerine başka bir şey koyduğunuzda HelloContent'in nasıl implemente edildiğini değiştirmek zorunda kalmayacağınız anlamına gelir.

![](https://developer.android.com/static/images/jetpack/compose/udf-hello-screen.png)

State'in aşağı indiği ve event'lerin yukarı çıktığı modele tek yönlü veri akışı denir. Bu durumda, state HelloScreen'den HelloContent'e iner ve event'ler HelloContent'ten HelloScreen'e çıkar. Tek yönlü veri akışını izleyerek, kullanıcı arayüzünde state'i görüntüleyen composable'ları uygulamanızın state'i depolayan ve değiştiren bölümlerinden ayırabilirsiniz.

{: .note}
Anahtar Nokta: State'i kaldırırken(hoisting), state'in nereye gitmesi gerektiğini anlamanıza yardımcı olacak üç kural vardır:
- State, state'i kullanan tüm composable'ların en azından en düşük ortak parent'ına kaldırılmalıdır(hoist) (read).
- State en azından değiştirilebileceği en yüksek seviyeye kaldırılmalıdır(hoist) (write).
- Aynı eventlara yanıt olarak iki state değişirse bunlar birlikte kaldırılmalıdır(hoist).
  State'i bu kuralların gerektirdiğinden daha yükseğe kaldırabilirsiniz, ancak state'in daha az kaldırılması tek yönlü veri akışını takip etmeyi zorlaştırır veya imkansız hale getirir.

Daha fazla bilgi edinmek için [Where to hoist state](where-to-hoist-state) sayfasına bakın.


## Restoring state in Compose
[rememberSaveable](https://developer.android.com/reference/kotlin/androidx/compose/runtime/saveable/package-summary#rememberSaveable(kotlin.Array,androidx.compose.runtime.saveable.Saver,kotlin.String,kotlin.Function0)) API'si remember'a benzer şekilde davranır çünkü kaydedilen instance state mekanizmasını kullanarak recompositionlar boyunca ve ayrıca activity veya process recreation boyunca state'i korur. Örneğin, ekran döndürüldüğünde bu durum gerçekleşir.

### Ways to store state
Bundle'a eklenen tüm veri türleri otomatik olarak kaydedilir. Bundle'a eklenemeyen bir şeyi kaydetmek istiyorsanız, birkaç seçenek vardır.

#### Parcelize
En basit çözüm, nesneye [@Parcelize](https://github.com/Kotlin/KEEP/blob/master/proposals/extensions/android-parcelable.md) annotation'ını eklemektir. Nesne parsellenebilir hale gelir ve bundle edilebilir. Örneğin, bu kod City veri türünü parsellenebilir hale getirir ve state'e kaydeder.
```kotlin
@Parcelize
data class City(val name: String, val country: String) : Parcelable

@Composable
fun CityScreen() {
    var selectedCity = rememberSaveable {
        mutableStateOf(City("Madrid", "Spain"))
    }
}
```
#### MapSaver
Herhangi bir nedenle @Parcelize uygun değilse, bir nesneyi sistemin Bundle'a kaydedebileceği bir değerler kümesine dönüştürmek için kendi kuralınızı tanımlamak üzere mapSaver'ı kullanabilirsiniz.
```kotlin
data class City(val name: String, val country: String)

val CitySaver = run {
    val nameKey = "Name"
    val countryKey = "Country"
    mapSaver(
        save = { mapOf(nameKey to it.name, countryKey to it.country) },
        restore = { City(it[nameKey] as String, it[countryKey] as String) }
    )
}

@Composable
fun CityScreen() {
    var selectedCity = rememberSaveable(stateSaver = CitySaver) {
        mutableStateOf(City("Madrid", "Spain"))
    }
}
```
#### ListSaver
Map'in key'lerini tanımlamak zorunda kalmamak için listSaver'ı kullanabilir ve index'lerini key olarak kullanabilirsiniz:
```kotlin
data class City(val name: String, val country: String)

val CitySaver = listSaver<City, Any>(
    save = { listOf(it.name, it.country) },
    restore = { City(it[0] as String, it[1] as String) }
)

@Composable
fun CityScreen() {
    var selectedCity = rememberSaveable(stateSaver = CitySaver) {
        mutableStateOf(City("Madrid", "Spain"))
    }
}
```
## State holders in Compose
Basit state hoisting işlemi, composable fonksiyonların kendi içinde yönetilebilir. Bununla birlikte, takip edilmesi gereken state miktarı artarsa veya composable fonksiyonlarda gerçekleştirilecek logic doğarsa, logic ve state sorumluluklarını diğer sınıflara devretmek iyi bir pratiktir: **state holders**.

{: .note}
Anahtar Terim: State holder'lar composable'ların logic ve state'lerini yönetir.
Diğer materyallerde state holder'ların hoisted state objects olarak da adlandırıldığını unutmayın.

Daha fazla bilgi edinmek için Compose belgelerinde [state hoisting](where-to-hoist-state)'e veya daha genel olarak mimari kılavuzundaki [State holders ve UI State](/docs/app-architecture/guide-to-app-architecture/ui-layer/state-holders-and-ui-state) sayfasına bakın.


## Retrigger remember caculations when keys change
[remember](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#remember(kotlin.Any,kotlin.Any,kotlin.Any,kotlin.Function0)) api'i sıklıkla [MutableState](https://developer.android.com/reference/kotlin/androidx/compose/runtime/MutableState) ile birlikte kullanılır:
```kotlin
var name by remember { mutableStateOf("") }
```
Burada, remember fonksiyonunun kullanılması MutableState değerinin recompositionlarda hayatta kalmasını sağlar.

Genel olarak, remember bir hesaplama lambda parametresi alır. remember ilk çalıştırıldığında, hesaplama lambdasını çağırır ve sonucunu saklar. Recomposition sırasında, remember en son depolanan değeri döndürür.

State'i önbelleğe almanın yanı sıra, başlatılması veya hesaplanması pahalı olan herhangi bir nesneyi veya bir işlemin sonucunu Composition'da saklamak için de remember'ı kullanabilirsiniz. Bu hesaplamayı her recompositionda tekrarlamak istemeyebilirsiniz. Pahalı bir işlem olan bu [ShaderBrush](https://developer.android.com/reference/kotlin/androidx/compose/ui/graphics/ShaderBrush) nesnesini oluşturmak buna bir örnektir:

```kotlin
val brush = remember {
    ShaderBrush(
        BitmapShader(
            ImageBitmap.imageResource(res, avatarRes).asAndroidBitmap(),
            Shader.TileMode.REPEAT,
            Shader.TileMode.REPEAT
        )
    )
}
```
remember değeri Composition'dan ayrılana kadar saklar. Ancak, önbelleğe alınan değeri geçersiz kılmanın bir yolu vardır. remember API'si ayrıca bir key veya keys parametresi alır. Bu key'lerden herhangi biri değişirse, fonksiyon bir sonraki yeniden oluşturma işleminde remember önbelleği geçersiz kılar ve hesaplama lambda bloğunu yeniden çalıştırır. Bu mekanizma, Composition'daki bir nesnenin yaşam süresi üzerinde kontrol sahibi olmanızı sağlar. Hesaplama, hatırlanan değer Composition'dan ayrılana kadar değil, girdiler değişene kadar geçerli kalır.

Aşağıdaki örnekler bu mekanizmanın nasıl çalıştığını göstermektedir.

Bu kod parçasında, bir ShaderBrush oluşturulur ve bir Box composable'ın arka plan boyası olarak kullanılır. remember, ShaderBrush instance'ını saklar çünkü daha önce açıklandığı gibi yeniden oluşturulması pahalıdır. remember, seçilen arka plan görüntüsü olan avatarRes'i key1 parametresi olarak alır. AvatarRes değişirse, fırça yeni görüntüyle yeniden oluşturulur ve Box'a yeniden uygulanır. Bu, kullanıcı bir seçiciden arka plan olarak başka bir görüntü seçtiğinde meydana gelebilir.
```kotlin
@Composable
private fun BackgroundBanner(
    @DrawableRes avatarRes: Int,
    modifier: Modifier = Modifier,
    res: Resources = LocalContext.current.resources
) {
    val brush = remember(key1 = avatarRes) {
        ShaderBrush(
            BitmapShader(
                ImageBitmap.imageResource(res, avatarRes).asAndroidBitmap(),
                Shader.TileMode.REPEAT,
                Shader.TileMode.REPEAT
            )
        )
    }

    Box(
        modifier = modifier.background(brush)
    ) {
        /* ... */
    }
}
```

Bir sonraki kod parçacığında state, MyAppState adlı düz bir state holder sınıfına hoist edilir. Sınıfın bir instance'ını remember kullanarak initialize etmek için bir rememberMyAppState fonksiyonu sunar. Bu tür fonksiyonları, recompositionlarda hayatta kalan bir instance oluşturmak için ortaya çıkarmak Compose'da yaygın bir modeldir. rememberMyAppState fonksiyonu, remember için anahtar parametre görevi gören windowSizeClass değerini alır. Bu parametre değişirse, uygulamanın düz state holder sınıfını en son değerle yeniden oluşturması gerekir. Bu, örneğin kullanıcı cihazı döndürdüğünde meydana gelebilir.

```kotlin
@Composable
private fun rememberMyAppState(
    windowSizeClass: WindowSizeClass
): MyAppState {
    return remember(windowSizeClass) {
        MyAppState(windowSizeClass)
    }
}

@Stable
class MyAppState(
    private val windowSizeClass: WindowSizeClass
) { /* ... */ }
```

{: .note}
Not: Düz state holder sınıfları hakkında daha fazla bilgi için, [State holder class as state owner](where-to-hoist-state.md#plain-state-holder-class-as-state-owner) belgesine veya Architecture kılavuzundaki [State holders and UI State](/docs/app-architecture/guide-to-app-architecture/ui-layer/state-holders-and-ui-state) belgesine bakın.

Compose, bir key'in değişip değişmediğine karar vermek ve saklanan değeri geçersiz kılmak için sınıfın equals implementasyonunu kullanır.

{: .note}
Not: İlk bakışta, keyler ile remember kullanmak, [derivedStateOf](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#derivedStateOf(kotlin.Function0)) gibi diğer Compose API'lerini kullanmaya benzer görünebilir. Aradaki farkı öğrenmek için [Jetpack Compose - Ne zaman derivedStateOf kullanmalıyım? blog gönderisi](https://medium.com/androiddevelopers/jetpack-compose-when-should-i-use-derivedstateof-63ce7954c11b)ne bakın.


### Store state with keys beyond recomposition
rememberSaveable API, verileri bir Bundle'da saklayabilen remember etrafında bir sarmalayıcıdır. Bu API, state'in yalnızca yeniden oluşturmaya değil, aynı zamanda activity recreation ve sistem tarafından başlatılan süreç ölümüne de dayanmasını sağlar. rememberSaveable, remember'ın key'leri almasıyla aynı amaçla input parametrelerini alır. Girdilerden herhangi biri değiştiğinde önbellek geçersiz kılınır. Fonksiyon bir sonraki kez yeniden oluşturulduğunda, rememberSaveable hesaplama lambda bloğunu yeniden yürütür.

{: .note}
Not: API isimlendirmesinde dikkat etmeniz gereken bir fark vardır. remember API'sinde parametre adı key'leri kullanırsınız ve rememberSaveable'da aynı amaç için input'ları kullanırsınız. Bu parametrelerden herhangi biri değişirse, önbelleğe alınan değer geçersiz kılınır.

Aşağıdaki örnekte, rememberSaveable, typedQuery değişene kadar userTypedQuery öğesini saklar:
```kotlin
var userTypedQuery by rememberSaveable(typedQuery, stateSaver = TextFieldValue.Saver) {
    mutableStateOf(
        TextFieldValue(text = typedQuery, selection = TextRange(typedQuery.length))
    )
}

```
## Learn more
Bu belgede, Compose'da state'i yönetmenin temellerini öğrendiniz. Daha fazla bilgi için aşağıdaki kaynaklara bakın:

### Samples
- [Jetnews sample](https://github.com/android/compose-samples/tree/main/JetNews)
- [Jetchat sample](https://github.com/android/compose-samples/tree/main/Jetchat)
- [Now in Android App](https://github.com/android/nowinandroid/tree/main)

### Codelabs
- [Using State in Jetpack Compose](https://codelabs.developers.google.com/codelabs/jetpack-compose-state/index.html?index=..%2F..index#0)
### Videos
- [A Compose state of mind](https://www.youtube.com/watch?v=rmv2ug-wW4U)
### Blogs
- [Effective State Management for TextField in Compose](https://medium.com/androiddevelopers/effective-state-management-for-textfield-in-compose-d6e5b070fbe5)
