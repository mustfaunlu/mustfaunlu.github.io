---
layout: default
title: CompositionLocal
parent: UI Architecture
grand_parent: Jetpack Compose
nav_order: 7
---
# Locally scoped data with CompositionLocal
[CompositionLocal](https://developer.android.com/reference/kotlin/androidx/compose/runtime/CompositionLocal), Composition üzerinden örtük olarak veri aktarmaya yarayan bir araçtır. Bu sayfada, CompositionLocal'ın ne olduğunu daha ayrıntılı olarak öğrenecek, kendi CompositionLocal'ınızı nasıl oluşturacağınızı öğrenecek ve CompositionLocal'ın kullanım durumunuz için iyi bir çözüm olup olmadığını anlayacaksınız.

## Introducing CompositionLocal
Genellikle Compose'da [veri aşağı akar](/docs/jetpack-compose/ui-architecture/architecture/#architecting-your-compose-ui) UI ağacı üzerinden her bir composable fonksiyona parametre olarak aktarılır. Bu, bir composable'ın bağımlılıklarını açık hale getirir. Ancak bu, renkler veya yazı stilleri gibi çok sık ve yaygın olarak kullanılan veriler için zahmetli olabilir. Aşağıdaki örneğe bakın:
```kotlin
@Composable
fun MyApp() {
    // Tema bilgileri uygulamanın root'una yakın bir yerde tanımlanma eğilimindedir
    val colors = colors()
}

// Hiyerarşinin derinliklerinde bazı composablelar
@Composable
fun SomeTextLabel(labelText: String) {
    Text(
        text = labelText,
        color = colors.onPrimary // ← renklere buradan erişmeniz gerekiyor
    )
}
```

Renklerin çoğu composable'a açık bir parametre bağımlılığı olarak aktarılmasına gerek kalmamasını desteklemek için Compose, UI ağacından veri akışı sağlamak için örtük bir yol olarak kullanılabilecek ağaç kapsamına alınmış adlandırılmış nesneler oluşturmanıza olanak tanıyan [CompositionLocal](https://developer.android.com/reference/kotlin/androidx/compose/runtime/CompositionLocal) özelliğini sunar.

`CompositionLocal` öğeleri genellikle UI ağacının belirli bir düğümünde bir değerle sağlanır. Bu değer, `CompositionLocal` öğesini composable fonksiyonda bir parametre olarak bildirmeden, composable torunları tarafından kullanılabilir.


{: .note }
Anahtar terimler: Bu kılavuzda **Composition, UI tree ve UI hierarchy** terimlerini kullanıyoruz. Diğer kılavuzlarda birbirleriyle değiştirilebilir şekilde kullanılsalar da farklı anlamlara sahiptirler: Composition, composable fonksiyonların çağrı grafiğinin kaydıdır. UI ağacı veya UI hiyerarşisi, composition işlemi tarafından oluşturulan, güncellenen ve bakımı yapılan [LayoutNode](https://cs.android.com/androidx/platform/frameworks/support/+/androidx-main:compose/ui/ui/src/commonMain/kotlin/androidx/compose/ui/node/LayoutNode.kt) ağacıdır.

`CompositionLocal`, [Material teması]((https://developer.android.com/reference/kotlin/androidx/compose/material/MaterialTheme))nın kaputun altında kullandığı şeydir. `MaterialTheme` üç CompositionLocal instance'ı (renkler, tipografi ve şekiller) sağlayan bir nesnedir ve bunları daha sonra Composition'ın herhangi bir alt kısmında almanıza olanak tanır. Bunlar özellikle MaterialTheme renkler, şekiller ve tipografi nitelikleri aracılığıyla erişebileceğiniz `LocalColors`, `LocalShapes` ve `LocalTypography` propertyleridir.

```kotlin
@Composable
fun MyApp() {
    // Provides a Theme whose values are propagated down its `content`
    MaterialTheme {
        // New values for colors, typography, and shapes are available
        // in MaterialTheme's content lambda.

        // ... content here ...
    }
}

// Some composable deep in the hierarchy of MaterialTheme
@Composable
fun SomeTextLabel(labelText: String) {
    Text(
        text = labelText,
        // `primary` is obtained from MaterialTheme's
        // LocalColors CompositionLocal
        color = MaterialTheme.colors.primary
    )
}
```

Bir `CompositionLocal` **instance, Composition'ın bir bölümüne kapsamlandırılmıştır**, böylece ağacın farklı seviyelerinde farklı değerler sağlayabilirsiniz. Bir `CompositionLocal`ın [current][(https://developer.android.com/reference/kotlin/androidx/compose/runtime/CompositionLocal#current()) değeri, Composition'ın o bölümündeki bir ata tarafından sağlanan en yakın değere karşılık gelir.

**Bir CompositionLocal'a yeni bir değer sağlamak için [CompositionLocalProvider](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#CompositionLocalProvider(kotlin.Array,kotlin.Function0))'ı ve CompositionLocal anahtarını bir değerle ilişkilendiren [provides](https://developer.android.com/reference/kotlin/androidx/compose/runtime/ProvidableCompositionLocal#provides(kotlin.Any)) infix fonksiyonunu kullanın**. CompositionLocalProvider'ın content lambda'sı, CompositionLocal'ın current property'sine erişirken sağlanan değeri alacaktır. Yeni bir değer sağlandığında, Compose CompositionLocal'ı okuyan Composition parçalarını yeniden oluşturur.

Bunun bir örneği olarak, [LocalContentAlpha](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary#LocalContentAlpha%0A()) `CompositionLocal`, UI'nin farklı bölümlerini belirginleştirmek veya belirginliğini azaltmak amacıyla metin ve ikonografi için kullanılan preferred content alpha değerini içerir. Aşağıdaki örnekte, CompositionLocalProvider Composition'ın farklı bölümleri için farklı değerler sağlamak üzere kullanılmaktadır.
```kotlin
@Composable
fun CompositionLocalExample() {
    MaterialTheme { // MaterialTheme ContentAlpha.high değerini varsayılan olarak ayarlar
        Column {
            Text("Uses MaterialTheme's provided alpha")
            CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.medium) {
                Text("Medium value provided for LocalContentAlpha")
                Text("This Text also uses the medium value")
                CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.disabled) {
                    DescendantExample()
                }
            }
        }
    }
}

@Composable
fun DescendantExample() {
    // CompositionLocalProviders ayrıca composable fonksiyonlar arasında da çalışır
    Text("This Text uses the disabled alpha now")
}
```

![](https://developer.android.com/static/images/jetpack/compose/compositionlocal-alpha.png)
Şekil 1. CompositionLocalExample composable önizlemesi.

Yukarıdaki tüm örneklerde, CompositionLocal instance'ları Material composable'ları tarafından dahili olarak kullanılmıştır. Bir CompositionLocal'ın geçerli değerine erişmek için [current](https://developer.android.com/reference/kotlin/androidx/compose/runtime/CompositionLocal#current()) propertysini kullanın. Aşağıdaki örnekte, metni biçimlendirmek için Android uygulamalarında yaygın olarak kullanılan [LocalContext](https://developer.android.com/reference/kotlin/androidx/compose/ui/platform/package-summary#LocalContext()) CompositionLocal'ın geçerli [Context](https://developer.android.com/reference/android/content/Context) değeri kullanılmıştır:

```kotlin
@Composable
fun FruitText(fruitSize: Int) {
    // LocalContext'in current değerinden `resources` öğesini alır
    val resources = LocalContext.current.resources
    val fruitText = remember(resources, fruitSize) {
        resources.getQuantityString(R.plurals.fruit_title, fruitSize)
    }
    Text(text = fruitText)
}
```

{: .note }
Not: CompositionLocal nesneleri veya sabitleri, IDE'de otomatik tamamlama ile daha iyi bulunabilirlik sağlamak için genellikle Local ile ön ek alır.


## Creating your own CompositionLocal
CompositionLocal, verilerin Composition boyunca dolaylı olarak aktarılması için bir araçtır.

CompositionLocal kullanımı için bir diğer önemli sinyal, parametrenin cross-cutting olduğu ve ara uygulama katmanlarının bunun varlığından haberdar olmaması gerektiğidir, çünkü bu ara katmanların haberdar olması composable'ın faydasını sınırlayacaktır. Örneğin, Android izinleri için sorgulama, kaputun altındaki bir CompositionLocal tarafından sağlanır. Composable bir medya seçici, API'sini değiştirmeden ve medya seçiciyi call edenlerin environment'dan kullanılan bu ek context'ten haberdar olmasını gerektirmeden cihazdaki izin korumalı içeriğe erişmek için yeni fonksiyonlar ekleyebilir.

Ancak, CompositionLocal her zaman en iyi çözüm değildir. Bazı dezavantajları olduğu için CompositionLocal'ın aşırı kullanımını önermiyoruz:

CompositionLocal, bir composable'ın davranışı hakkında mantık yürütmeyi zorlaştırır. Örtük bağımlılıklar yarattıklarından, bunları kullanan composable'ları çağıranların her CompositionLocal için bir değerin karşılandığından emin olmaları gerekir.

Ayrıca, Composition'ın herhangi bir bölümünde değişebileceğinden, bu bağımlılık için net bir doğruluk kaynağı olmayabilir. Bu nedenle, mevcut değerin nerede sağlandığını görmek için Composition'da gezinmeniz gerektiğinden, bir sorun oluştuğunda uygulamada hata ayıklama yapmak daha zor olabilir. IDE'deki Find usages veya [Compose layout inspector](https://developer.android.com/jetpack/compose/tooling#layout-inspector) gibi araçlar bu sorunu hafifletmek için yeterli bilgi sağlar.

{: .note }
Not: CompositionLocal, temel mimari için iyi çalışır ve Jetpack Compose bunu yoğun bir şekilde kullanır.

### Deciding whether to use CompositionLocal
CompositionLocal'ı kullanım durumunuz için iyi bir çözüm haline getirebilecek belirli koşullar vardır:

Bir CompositionLocal iyi bir varsayılan değere sahip olmalıdır. Varsayılan bir değer yoksa, bir geliştiricinin CompositionLocal için bir değerin sağlanmadığı bir duruma girmesinin son derece zor olduğunu garanti etmelisiniz. Varsayılan bir değer sağlamamak, CompositionLocal'in her zaman açıkça sağlanmasını gerektiren bir composable'ı kullanan testler oluştururken veya önizleme yaparken sorunlara ve hayal kırıklığına neden olabilir.

Ağaç kapsamı veya alt hiyerarşi kapsamı olarak düşünülmeyen kavramlar için CompositionLocal kullanmaktan kaçının. Bir CompositionLocal, birkaçı tarafından değil, herhangi bir torun tarafından potansiyel olarak kullanılabildiğinde anlamlıdır.

Kullanım durumunuz bu gereksinimleri karşılamıyorsa, bir CompositionLocal oluşturmadan önce Dikkate [Alınacak Alternatifler bölümü](#alternatives-to-consider)ne göz atın.

Belirli bir ekranın ViewModel'ini tutan bir CompositionLocal oluşturmak kötü bir pratiğe örnektir, böylece o ekrandaki tüm composable'lar bazı mantıkları gerçekleştirmek için ViewModel'e bir referans alabilir. Bu kötü bir pratiktir çünkü belirli bir UI ağacının altındaki tüm composable'ların bir ViewModel hakkında bilgi sahibi olması gerekmez. İyi pratik, [state'in aşağı ve event'lerin yukarı akması modeli](/docs/jetpack-compose/ui-architecture/architecture/#architecting-your-compose-ui)ni izleyerek composable'lara yalnızca ihtiyaç duydukları bilgileri aktarmaktır. Bu yaklaşım, composable'larınızı daha yeniden kullanılabilir ve daha kolay test edilebilir hale getirecektir.

### Creating a CompositionLocal
Bir `CompositionLocal` oluşturmak için iki API vardır:

- [compositionLocalOf](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#compositionLocalOf(androidx.compose.runtime.SnapshotMutationPolicy,kotlin.Function0)): Recomposition sırasında sağlanan değerin değiştirilmesi, yalnızca current değerini okuyan içeriği geçersiz kılar.


- [staticCompositionLocalOf](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#staticCompositionLocalOf(kotlin.Function0)): compositionLocalOf'un aksine, bir staticCompositionLocalOf'un okumaları Compose tarafından izlenmez. Değerin değiştirilmesi, CompositionLocal'ın sağlandığı content lambda'nın tamamının, sadece geçerli değerin Composition'da okunduğu yerler yerine yeniden oluşturulmasına neden olur.

CompositionLocal'a sağlanan değerin değişme olasılığı çok düşükse veya hiç değişmeyecekse, performans avantajları elde etmek için staticCompositionLocalOf kullanın.

Örneğin, bir uygulamanın tasarım sistemi, composable'ların UI komponenti için bir gölge kullanılarak elevate edilmesi konusunda fikir sahibi olabilir. Uygulama için farklı elevate etmelerin UI ağacı boyunca yayılması gerektiğinden, bir CompositionLocal kullanırız. CompositionLocal değeri sistem temasına göre koşullu olarak türetildiğinden, compositionLocalOf API'sini kullanırız:
```kotlin
// LocalElevations.kt dosyasi

data class Elevations(val card: Dp = 0.dp, val default: Dp = 0.dp)

// Varsayılan bir CompositionLocal global nesnesi tanımlayın
// Bu instance'a uygulamadaki tüm composable'lar tarafından erişilebilir
val LocalElevations = compositionLocalOf { Elevations() }
```

### Providing values to a CompositionLocal
[CompositionLocalProvider](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#CompositionLocalProvider(kotlin.Array,kotlin.Function0)) composable, değerleri verilen hiyerarşi için CompositionLocal instance'larına bağlar. Bir CompositionLocal'a yeni bir değer sağlamak için, bir CompositionLocal key'ini bir değerle ilişkilendiren [provides](https://developer.android.com/reference/kotlin/androidx/compose/runtime/ProvidableCompositionLocal#provides%0A(kotlin.Any)) infix fonksiyonunu aşağıdaki gibi kullanın:

```kotlin
// MyActivity.kt file

class MyActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            // Sistem temasına göre yükseklikleri hesaplayın
            val elevations = if (isSystemInDarkTheme()) {
                Elevations(card = 1.dp, default = 1.dp)
            } else {
                Elevations(card = 0.dp, default = 0.dp)
            }

            // Yüksekliği LocalElevations için değer olarak bağlayın
            CompositionLocalProvider(LocalElevations provides elevations) {
                // ... Content buraya gidecek ...
                // Composition'ın bu bölümü LocalElevations.current dosyasına erişirken 
                // `elevations` instance'ını görecektir
            }
        }
    }
}
```

### Consuming the CompositionLocal
[CompositionLocal.current](https://developer.android.com/reference/kotlin/androidx/compose/runtime/ProvidableCompositionLocal#current()), o CompositionLocal için bir değer sağlayan en yakın CompositionLocalProvider tarafından sağlanan değeri döndürür:
```kotlin
@Composable
fun SomeComposable() {
    // Composition'ın bu bölümündeki mevcut Elevations değerini almak için 
    // global olarak tanımlanmış LocalElevations değişkenine erişin
    Card(elevation = LocalElevations.current.card) {
        // Content
    }
}
```
## Alternatives to consider
CompositionLocal bazı kullanım durumları için aşırı bir çözüm olabilir. Kullanım durumunuz [CompositionLocal kullanıp kullanmamaya karar verme](#deciding-whether-to-use-compositionlocal) bölümünde belirtilen ölçütleri karşılamıyorsa, kullanım durumunuz için başka bir çözüm daha uygun olabilir.

### Pass explicit parameters
Composable'ın bağımlılıkları hakkında açık olmak iyi bir alışkanlıktır. Composable'lara yalnızca ihtiyaç duydukları bilgileri aktarmanızı öneririz. Composable'ların decoupling ve yeniden kullanımını teşvik etmek için, her composable mümkün olan en az miktarda bilgiyi tutmalıdır.
```kotlin
@Composable
fun MyComposable(myViewModel: MyViewModel = viewModel()) {
    // ...
    MyDescendant(myViewModel.data)
}

// Tüm nesneyi geçirmeyin! Sadece torunun ihtiyacı olanı.
// Ayrıca, bir CompositionLocal kullanarak ViewModel'i örtük bir bağımlılık olarak geçirmeyin.
@Composable
fun MyDescendant(myViewModel: MyViewModel) { /* ... */ }

// Sadece torunun ihtiyacı olanı gecirin
@Composable
fun MyDescendant(data: DataToDisplay) {
    // Verileri görüntüle
}
```
### Inversion of control
Gereksiz bağımlılıkları bir composable'a geçirmekten kaçınmanın bir başka yolu da inversion of control'dür. Alt öğenin bazı logicleri yürütmek için bir bağımlılık alması yerine, üst öğe bunu yapar.

Bir alt öğenin bazı verileri yüklemek için isteği tetiklemesi gereken aşağıdaki örneğe bakın:

```kotlin
@Composable
fun MyComposable(myViewModel: MyViewModel = viewModel()) {
    // ...
    MyDescendant(myViewModel)
}

@Composable
fun MyDescendant(myViewModel: MyViewModel) {
    Button(onClick = { myViewModel.loadData() }) {
        Text("Load data")
    }
}
```
Duruma bağlı olarak, MyDescendant çok fazla sorumluluğa sahip olabilir. Ayrıca, MyViewModel'i bir bağımlılık olarak geçirmek, MyDescendant'ı artık birbirine bağlı oldukları için daha az yeniden kullanılabilir hale getirir. Bağımlılığı toruna geçirmeyen ve logic'in yürütülmesinden atayı sorumlu kılan inversion of control ilkelerini kullanan alternatifi düşünün:
```kotlin
@Composable
fun MyComposable(myViewModel: MyViewModel = viewModel()) {
    // ...
    ReusableLoadDataButton(
        onLoadClick = {
            myViewModel.loadData()
        }
    )
}

@Composable
fun ReusableLoadDataButton(onLoadClick: () -> Unit) {
    Button(onClick = onLoadClick) {
        Text("Load data")
    }
}
```

Bu yaklaşım, çocuğu yakın atalarından decouple ettigi için bazı kullanım durumları için daha uygun olabilir. Ata composablelar, daha esnek alt seviye composablelara sahip olmak için daha karmaşık hale gelme eğilimindedir.

Benzer şekilde, @Composable content lambda'lar da aynı faydaları elde etmek için aynı şekilde kullanılabilir:

```kotlin
@Composable
fun MyComposable(myViewModel: MyViewModel = viewModel()) {
    // ...
    ReusablePartOfTheScreen(
        content = {
            Button(
                onClick = {
                    myViewModel.loadData()
                }
            ) {
                Text("Confirm")
            }
        }
    )
}

@Composable
fun ReusablePartOfTheScreen(content: @Composable () -> Unit) {
    Column {
        // ...
        content()
    }
}
```