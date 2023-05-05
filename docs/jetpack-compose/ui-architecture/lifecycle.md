---
layout: default
title: Lifecycle
parent: UI Architecture
grand_parent: Jetpack Compose
nav_order: 1
---

# Lifecycle of composables
Bu sayfada, bir composable'ın yaşam döngüsü ve Compose'un bir composable'ın recomposition'a ihtiyacı olup olmadığına nasıl karar verdiği hakkında bilgi edineceksiniz.

## Lifecycle overview
[Managing state](managing-state/managing-state) belgelerinde belirtildiği gibi, bir Composition uygulamanızın kullanıcı arayüzünü tanımlar ve composables çalıştırılarak üretilir. Bir Composition, kullanıcı arayüzünüzü tanımlayan composable'ların bir ağaç yapısıdır.

Jetpack Compose, composable'larınızı ilk kez çalıştırdığında, ilk composition sırasında, UI'nizi bir Composition'da tanımlamak için çağırdığınız composable'ların takibini yapacaktır. Ardından, uygulamanızın state'i değiştiğinde, Jetpack Compose bir recomposition planlar. Recomposition, Jetpack Compose'un state değişikliklerine yanıt olarak değişmiş olabilecek composable'ları yeniden çalıştırması ve ardından Composition'ı değişiklikleri yansıtacak şekilde güncellemesidir.

Bir Composition yalnızca bir initial composition tarafından üretilebilir ve recomposition tarafından güncellenebilir. Bir Composition'ı değiştirmenin tek yolu recomposition'dır.

{: .note }
Bir composable'ın yaşam döngüsü şu olaylarla tanımlanır: Composition'a girme, 0 veya daha fazla kez yeniden composition'a girme ve Composition'dan çıkma.

![](https://developer.android.com/static/images/jetpack/compose/lifecycle-composition.png)

Şekil 1. Composition içindeki bir composable'ın yaşam döngüsü. Composition'a girer, 0 veya daha fazla kez yeniden oluşturulur ve Composition'dan ayrılır.

Recomposition tipik olarak bir State<T> nesnesindeki bir değişiklik tarafından tetiklenir. Compose bunları izler ve Composition'da söz konusu State<T> nesnesini okuyan tüm composable'ları ve atlanamayacak şekilde çağırdıkları tüm composable'ları çalıştırır.

{: .note }
Not: Bir Composable'ın yaşam döngüsü view, activity ve fragment'ların yaşam döngüsünden daha basittir. Bir composable'ın daha karmaşık bir yaşam döngüsüne sahip harici kaynakları yönetmesi veya bunlarla etkileşime girmesi gerektiğinde, [efektleri](side-effects.md#state-and-effect-use-cases) kullanmalısınız.

Bir composable birden çok kez çağrılırsa, Composition'a birden çok instance yerleştirilir. Her çağrının Composition içinde kendi yaşam döngüsü vardır.

```kotlin
@Composable
fun MyComposable() {
    Column {
        Text("Hello")
        Text("World")
    }
}
```
![](https://developer.android.com/static/images/jetpack/compose/lifecycle-hierarchy.png)
Şekil 2. MyComposable'ın Composition'da gösterimi.
Bir composable birden fazla kez çağrılırsa, Composition'a birden fazla instance yerleştirilir. Bir öğenin farklı bir renge sahip olması, onun ayrı bir instance olduğunun göstergesidir.

## Anatomy of a composable in Composition
Composition'da bir composable instance'ı call site ile tanımlanır. Compose derleyicisi her call site'i farklı olarak değerlendirir. Birden fazla call site'dan composable'ların çağrılması, Composition'da composable'ın birden fazla instance'ını oluşturacaktır.

{: .note }
Anahtar Terim: Call site, bir composable'ın çağrıldığı kaynak kod konumudur. Bu, Composition'daki yerini ve dolayısıyla UI ağacını etkiler.

Bir recomposition sırasında bir composable önceki composition sırasında çağırdığından farklı composable'lar çağırırsa, Compose ***hangi composable'ların çağrıldığını ya da çağrılmadığını*** belirleyecek ve her iki composition'da da çağrılan composable'lar için, girdileri değişmemişse onları recomposition etmekten kaçınacaktır.***

Kimliğin korunması, side-effectleri composable'ları ile ilişkilendirmek için çok önemlidir, böylece her recomposition için yeniden başlatmak yerine başarıyla tamamlayabilirler.

Aşağıdaki örneği ele alalım:

```kotlin
@Composable
fun LoginScreen(showError: Boolean) {
    if (showError) {
        LoginError()
    }
    LoginInput() // Bu call site LoginInput'un Composition'da nereye yerleştirileceğini etkiler
}

@Composable
fun LoginInput() { /* ... */ }

@Composable
fun LoginError() { /* ... */ }
```
Yukarıdaki kod parçasında, LoginScreen koşullu olarak LoginError composable'ını çağıracak ve her zaman LoginInput composable'ını çağıracaktır. Her çağrının, derleyicinin onu benzersiz bir şekilde tanımlamak için kullanacağı benzersiz bir call site ve kaynak konumu vardır.

![](https://developer.android.com/static/images/jetpack/compose/lifecycle-showerror.png)

Şekil 3. State değiştiğinde ve recomposition gerçekleştiğinde LoginScreen'in Composition'daki gösterimi. Aynı renk, yeniden oluşturulmadığı anlamına gelir.

LoginInput ilk çağrılmadan ikinci çağrılmaya geçse de, LoginInput instance'ı recompositionlar boyunca korunacaktır. Ayrıca, LoginInput'un recomposition boyunca değişen herhangi bir parametresi olmadığından, LoginInput'a yapılan çağrı Compose tarafından atlanacaktır.

### Add extra information to help smart recompositions
Bir composable'ın birden çok kez çağrılması, onu Composition'a da birden çok kez ekleyecektir. Aynı call site'dan bir composable birden çok kez çağrıldığında, Compose o composable'a yapılan her çağrıyı benzersiz bir şekilde tanımlayacak herhangi bir bilgiye sahip değildir, bu nedenle instance'ları farklı tutmak için call site'a ek olarak yürütme sırası kullanılır. Bu davranış bazen gerekli olan tek şeydir, ancak bazı durumlarda istenmeyen davranışlara neden olabilir.
```kotlin
@Composable
fun MoviesScreen(movies: List<Movie>) {
    Column {
        for (movie in movies) {
            //MovieOverview composablelari, for döngüsünde indeks konumu verilen Composition'a yerleştirilir
            MovieOverview(movie)
        }
    }
}
```
Yukarıdaki örnekte Compose, instance'ları Composition'da farklı tutmak için call site'a ek olarak execution order'ı kullanır. Listenin en altına yeni bir film eklenirse, Compose, listedeki konumları değişmediğinden ve bu nedenle film girişi bu instancelar için aynı olduğundan, Composition'da zaten bulunan instance'ları yeniden kullanabilir.

![](https://developer.android.com/static/images/jetpack/compose/lifecycle-newelement-bottom.png)
Şekil 4. Listenin altına yeni bir öğe eklendiğinde MoviesScreen'in Composition'daki gösterimi. Composition'daki MovieOverview composable'ları yeniden kullanılabilir. MovieOverview'daki aynı renk, composable'ın yeniden oluşturulmadığı anlamına gelir.

Bununla birlikte, film listesi listenin üstüne veya ortasına ekleme, öğeleri kaldırma veya yeniden sıralama yoluyla değişirse, girdi parametresi listedeki konumu değişen tüm MovieOverview çağrılarında recomposition'a neden olur. Örneğin, MovieOverview bir side-effect kullanarak bir film görüntüsü getiriyorsa bu son derece önemlidir. Efekt devam ederken recomposition gerçekleşirse, iptal edilecek ve yeniden başlayacaktır.

```kotlin
@Composable
fun MovieOverview(movie: Movie) {
    Column {
        //Side-effect dokümanlarda daha sonra açıklanmıştır. 
        // Resim getirme işlemi devam ederken MovieOverview recompose edilirse, 
        // iptal edilir ve yeniden başlatılır.
        val image = loadNetworkImage(movie.url)
        MovieHeader(image)

        /* ... */
    }
}
```
![](https://developer.android.com/static/images/jetpack/compose/lifecycle-newelement-top-all-recompose.png)
Şekil 5. Listeye yeni bir eleman eklendiğinde MoviesScreen'in Composition'daki gösterimi. MovieOverview composable'ları yeniden kullanılamaz ve tüm side-effectler yeniden başlar. MovieOverview'da farklı bir renk, composable'ın yeniden oluşturulduğu anlamına gelir.

İdeal olarak, MovieOverview instance'ının kimliğini kendisine aktarılan filmin kimliğine bağlı olarak düşünmek isteriz. Film listesini yeniden sıralarsak, ideal olarak her MovieOverview composable'ı farklı bir film instance'ı ile yeniden oluşturmak yerine Composition ağacındaki instance'ları da benzer şekilde yeniden sıralarız. Compose, çalışma zamanına ağacın belirli bir bölümünü tanımlamak için hangi değerleri kullanmak istediğinizi söylemeniz için bir yol sağlar: key composable.

Bir kod bloğunu, bir veya daha fazla değerin aktarıldığı bir key composable çağrısı ile sardığınızda, bu değerler birleştirilerek compositiondaki o instance'ı tanımlamak için kullanılacaktır. Bir key için değerin global olarak benzersiz olması gerekmez, yalnızca call site'deki composable'ların invocation'ları arasında benzersiz olması gerekir. Yani bu örnekte, her filmin filmler arasında benzersiz olan bir key'e sahip olması gerekir; bu key'i uygulamanın başka bir yerindeki başka bir composable ile paylaşması sorun değildir.
```kotlin
@Composable
fun MoviesScreenWithKey(movies: List<Movie>) {
    Column {
        for (movie in movies) {
            key(movie.id) { // Bu film için benzersiz kimlik
                MovieOverview(movie)
            }
        }
    }
}
```
Yukarıdakilerle, listedeki öğeler değişse bile, Compose, MovieOverview'a yapılan bireysel çağrıları tanır ve bunları yeniden kullanabilir.

![](https://developer.android.com/static/images/jetpack/compose/lifecycle-newelement-top-keys.png)
Şekil 6. Listeye yeni bir öğe eklendiğinde MoviesScreen'in Composition'daki gösterimi. MovieOverview composablelarinin benzersiz keyleri olduğundan, Compose hangi MovieOverview instance'larının değişmediğini tanır ve bunları yeniden kullanabilir; side-effectleri execute edilmeye devam edecektir.

{: .note}
Anahtar Nokta: Compose'un Composition'daki composable instance'ları tanımlamasına yardımcı olmak için composable key'ini kullanın. Birden fazla composable aynı call site'dan çağrıldığında ve side-effectler veya dahili state içerdiğinde bu önemlidir.

Bazı composable'lar key composable için yerleşik desteğe sahiptir. Örneğin, LazyColumn items DSL'sinde özel bir key belirtilmesini kabul eder.
```kotlin
@Composable
fun MoviesScreenLazy(movies: List<Movie>) {
    LazyColumn {
        items(movies, key = { movie -> movie.id }) { movie ->
            MovieOverview(movie)
        }
    }
}
```
### Skipping if the inputs haven't changed
Bir composable zaten Composition içindeyse, tüm girdiler sabitse ve değişmemişse recomposition'ı atlayabilir.

Bir stabil tip aşağıdaki sözleşmeye uygun olmalıdır:

* İki instance için equals sonucu, aynı iki instance için sonsuza kadar aynı olacaktır.
* Türün bir public property'si değişirse, Composition bilgilendirilecektir.
* Tüm public property türleri de stabildir.

Bu sözleşmeye giren ve @Stable annotation'ı kullanılarak açıkça stable olarak işaretlenmemiş olsalar bile compose derleyicisinin stable olarak ele alacağı bazı önemli yaygın tipler vardır:
* Tüm primitif değer türleri: Boolean, Int, Long, Float, Char, etc.
* Stringler
* Tüm Fonksiyon türleri (lambdalar)

Tüm bu tipler stable sözleşmesini takip edebilirler çünkü immutable'dırlar. Immutable tipler asla değişmediğinden, Composition'a değişikliği bildirmek zorunda kalmazlar, bu nedenle bu sözleşmeyi takip etmek çok daha kolaydır.

{: .note}
Not: Tüm deeply immutable tipler güvenli bir şekilde stable tipler olarak kabul edilebilir.

Stable ancak mutable olan dikkate değer bir tür Compose'un MutableState türüdür. Bir MutableState içinde bir değer tutulursa, Compose State'in .value property'sindeki herhangi bir değişiklikten haberdar edileceğinden, state nesnesinin genel olarak stable olduğu kabul edilir.

Bir composable'a parametre olarak aktarılan tüm türler stable olduğunda, parametre değerleri UI ağacındaki composable konumuna göre eşitlik açısından karşılaştırılır. Önceki çağrıdan bu yana tüm değerler değişmemişse recomposition atlanır.

{: .note}
Anahtar Nokta: Compose, tüm girdiler stable ise ve değişmemişse bir composable'ın recomposition'ını atlar. Karşılaştırma equals metodunu kullanır.

Compose bir türü yalnızca bunu kanıtlayabiliyorsa stable olarak kabul eder. Örneğin, bir interface genellikle stable olarak değerlendirilmez ve implementasyonu immutable olabilen mutable public property'lere sahip tipler de stable değildir.

Compose bir türün stable olduğu sonucunu çıkaramıyorsa, ancak Compose'u onu stable olarak ele almaya zorlamak istiyorsanız, [@Stable](https://developer.android.com/reference/kotlin/androidx/compose/runtime/Stable) annotation'ı ile işaretleyin.

```kotlin
// Atlama ve akıllı recompositionları desteklemek için tipin stable olarak işaretlenmesi.
@Stable
interface UiState<T : Result<T>> {
    val value: T?
    val exception: Throwable?

    val hasError: Boolean
        get() = exception != null
}
```

Yukarıdaki kod parçasında, UiState bir interface olduğundan, Compose normalde bu tipin stable olmadığını düşünebilir. Stable annotation'ını ekleyerek, Compose'a bu türün stable olduğunu söylersiniz ve Compose'un akıllı recomposition'ları tercih etmesini sağlarsınız. Bu aynı zamanda, interface'in parametre türü olarak kullanılması durumunda Compose'un tüm uygulamalarını stable olarak değerlendireceği anlamına gelir.

{: .note}
Önemli Nokta: Compose bir türün stabilitesini çıkaramıyorsa, Compose'un akıllı rekompozisyonları tercih etmesine izin vermek için türe @Stable ile annotation ekleyin.



