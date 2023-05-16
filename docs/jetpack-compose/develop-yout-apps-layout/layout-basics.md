---
layout: default
title: Layout basics
parent: Develop your app's layout
grand_parent: Jetpack Compose
nav_order: 2
---
# Compose layout basics
Jetpack Compose, uygulamanızın kullanıcı arayüzünü tasarlamayı ve oluşturmayı çok daha kolay hale getirir. Compose, state'i UI öğelerine dönüştürür:

- Composition(Öğelerin bileşimi)
- Layout of element(Elemanların yerleşimi)
- Drawing of elements(Elemanların çizimi)

[Fundamentals of Compose Layouts and Modifiers - MAD Skills](https://youtu.be/xc8nAcVvpxY)

![](https://developer.android.com/static/images/jetpack/compose/composition-layout-drawing.svg)

Bu belge, Compose'un UI öğelerinizi düzenlemenize yardımcı olmak için sağladığı bazı yapı taşlarını açıklayarak öğelerin düzenine odaklanmaktadır.

## Goals of layouts in Compose
Layout sisteminin Jetpack Compose uygulamasının iki ana hedefi vardır:

- Yüksek performans
- Kolayca custom layout yazabilme

{: .note}
Not: Android View sistemi ile RelativeLayout gibi belirli View'leri iç içe yerleştirirken bazı performans sorunlarıyla karşılaşabilirsiniz. Compose çoklu ölçümleri önlediğinden, performansı etkilemeden istediğiniz kadar derine yerleştirebilirsiniz.

## Basicss of Composable functions
Composable fonksiyonlar Compose'un temel yapı taşıdır. Composable fonksiyon, UI'nizin bir kısmını tanımlayan Unit yayan bir fonksiyondur. Fonksiyon bazı girdileri alır ve ekranda gösterilenleri oluşturur. Composable'lar hakkında daha fazla bilgi için Compose [mental model belgeleri](/docs/jetpack-compose/introduction/thinking-in-compose/#thinking-in-compose)ne göz atın.

Bir Composable fonksiyonu birkaç UI öğesi yayabilir. Ancak, nasıl düzenlenmeleri gerektiği konusunda rehberlik sağlamazsanız, Compose öğeleri beğenmediğiniz bir şekilde düzenleyebilir. Örneğin, bu kod iki metin öğesi üretir:

```kotlin
@Composable
fun ArtistCard() {
    Text("Alfred Sisley")
    Text("3 minutes ago")
}
```
Nasıl düzenlenmelerini istediğinize dair bir yönlendirme olmadan, Compose metin öğelerini üst üste yığarak okunamaz hale getirir:

![](https://developer.android.com/static/images/jetpack/compose/layout-overlap.png)

Compose, UI öğelerinizi düzenlemenize yardımcı olmak için kullanıma hazır layout'lardan oluşan bir koleksiyon sunar ve kendi daha özel layout'larınızı tanımlamanızı kolaylaştırır.

## Standard layout components
Çoğu durumda, [Compose'un standart layout öğeleri](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary)ni kullanabilirsiniz.

Öğeleri ekrana dikey olarak yerleştirmek için [Column](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary#Column(androidx.compose.ui.Modifier,androidx.compose.foundation.layout.Arrangement.Vertical,androidx.compose.ui.Alignment.Horizontal,kotlin.Function1)(androidx.compose.ui.Modifier,androidx.compose.foundation.layout.Arrangement.Vertical,androidx.compose.ui.Alignment.Horizontal,kotlin.Function1))'u kullanın.

```kotlin
@Composable
fun ArtistCardColumn() {
    Column {
        Text("Alfred Sisley")
        Text("3 minutes ago")
    }
}
```

![](https://developer.android.com/static/images/jetpack/compose/layout-text-in-column.png)

Benzer şekilde, öğeleri ekrana yatay olarak yerleştirmek için Row kullanın. Hem Column hem de Row, içerdikleri öğelerin hizalamasını yapılandırmayı destekler.
```kotlin
@Composable
fun ArtistCardRow(artist: Artist) {
    Row(verticalAlignment = Alignment.CenterVertically) {
        Image(bitmap = artist.image, contentDescription = "Artist image")
        Column {
            Text(artist.name)
            Text(artist.lastSeenOnline)
        }
    }
}
```

![](https://developer.android.com/static/images/jetpack/compose/layout-text-with-picture.png)


Öğeleri üst üste koymak için [Box](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary#Box(androidx.compose.ui.Modifier,androidx.compose.ui.Alignment,kotlin.Boolean,kotlin.Function1))'ı kullanın. Box, içerdiği öğelerin belirli hizalamasını yapılandırmayı da destekler.
```kotlin
@Composable
fun ArtistAvatar(artist: Artist) {
    Box {
        Image(bitmap = artist.image, contentDescription = "Artist image")
        Icon(Icons.Filled.Check, contentDescription = "Check mark")
    }
}
```

![](https://developer.android.com/static/images/jetpack/compose/layout-box-with-picture.png)

Genellikle ihtiyacınız olan tek şey bu yapı taşlarıdır. Bu layoutları uygulamanıza uygun daha ayrıntılı bir layoutta birleştirmek için kendi composable fonksiyonunuzu yazabilirsiniz.

![](https://developer.android.com/static/images/jetpack/compose/layout-column-row-box.svg)

{: .note}
Not: Compose, iç içe layoutlari verimli bir şekilde işleyerek karmaşık bir UI tasarlamak için harika bir yol haline getirir. Bu, performans nedenleriyle iç içe layoutlarda kaçınmanız gereken Android Views'e göre bir gelişmedir.

Child componentlerin Row içindeki konumunu ayarlamak için horizontalArrangement ve verticalAlignment argümanlarını kullanın. Column için, verticalArrangement ve horizontalAlignment argümanlarını kullanın:

```kotlin
@Composable
fun ArtistCardArrangement(artist: Artist) {
    Row(
        verticalAlignment = Alignment.CenterVertically,
        horizontalArrangement = Arrangement.End
    ) {
        Image(bitmap = artist.image, contentDescription = "Artist image")
        Column { /*...*/ }
    }
}
```

![](https://developer.android.com/static/images/jetpack/compose/layout-row-end.png)

## The layout model
Layout modelinde, UI ağacı tek bir seferde düzenlenir. Her node'dan önce kendisini boyutlandırması istenir, ardından tüm child'ları özyinelemeli olarak boyutlandırır ve boyut kısıtlamalarını ağaçtan aşağıya child'lara aktarır. Ardından, leaf nodelar boyutlandırılır ve yerleştirilir, çözülen boyutlar ve yerleştirme talimatları ağaca geri aktarılır.

Kısaca, ebeveynler çocuklarından önce ölçülür, ancak çocuklarından sonra boyutlandırılır ve yerleştirilir.

Aşağıdaki `SearchResult` fonksiyonunu düşünün.

```kotlin
@Composable
fun SearchResult() {
    Row {
        Image(
            // ...
        )
        Column {
            Text(
                // ...
            )
            Text(
                // ...
            )
        }
    }
}
```
Bu fonksiyon aşağıdaki UI ağacını verir.
```kotlin
SearchResult
  Row
    Image
    Column
      Text
      Text
```
SearchResult örneğinde, UI ağaç layout'u şu sırayı takip eder:

- Root node olan Row'dan ölçüm yapması istenir.
- Root node Row, ilk çocuğu olan Image'dan ölçüm yapmasını ister.
- Image bir leaf node'dur (yani çocuğu yoktur), bu nedenle bir boyut bildirir ve yerleştirme talimatlarını döndürür.
- Row, ikinci çocuğu olan Column'dan ölçüm yapmasını ister.
- Column node'u ilk Text çocuğunun ölçülmesini ister.
- İlk Text node'u bir leaf node'dur, bu nedenle bir boyut bildirir ve yerleştirme talimatlarını döndürür.
- Column ikinci Text çocuğunun ölçülmesini ister.
- İkinci Text node'u bir leaf node'dur, bu nedenle bir boyut bildirir ve yerleştirme talimatlarını döndürür.
- Artık Column node'u çocuklarını ölçtüğüne, boyutlandırdığına ve yerleştirdiğine göre, kendi boyutunu ve yerleşimini belirleyebilir.
- Root node olan Row ölçüldüğüne, boyutlandırıldığına ve çocukları yerleştirildiğine göre, kendi boyutunu ve yerleşimini belirleyebilir.

![](https://developer.android.com/static/images/jetpack/compose/search-result-layout.svg)

## Performance
Compose, child'ları yalnızca bir kez boyutlandırarak yüksek performans elde eder. Tek seferlik ölçüm performans açısından iyidir ve Compose'un derin UI ağaçlarını verimli bir şekilde işlemesine olanak tanır. Bir öğe child'ını iki kez ölçseydi ve bu child da her bir child'ını iki kez ölçseydi ve bu böyle devam etseydi, tüm bir UI'yi düzenlemek için tek bir girişimde çok fazla iş yapmak zorunda kalacak ve uygulamanızı performanslı tutmayı zorlaştıracaktı.

Layout'unuz herhangi bir nedenle birden fazla ölçüm gerektiriyorsa, Compose özel bir sistem olan intrinsic ölçümler sunar. Bu özellik hakkında daha fazla bilgiyi Compose layout'larında intrinsic ölçümler bölümünde okuyabilirsiniz.

Ölçüm ve yerleştirme, layout geçişinin farklı alt aşamaları olduğundan, ölçümü etkilemeyen yalnızca öğelerin yerleştirilmesini etkileyen herhangi bir değişiklik ayrı olarak yürütülebilir.

## Using modifiers in your layouts
Compose modifiers bölümünde tartışıldığı gibi, composablelar'ınızı zenginleştirmek veya dekore etmek için modifier'ları kullanabilirsiniz. Modifier'lar layout'unuzu özelleştirmek için çok önemlidir. Örneğin, burada ArtistCard'ı özelleştirmek için birkaç değiştirici zincirliyoruz:

```kotlin
@Composable
fun ArtistCardModifiers(
    artist: Artist,
    onClick: () -> Unit
) {
    val padding = 16.dp
    Column(
        Modifier
            .clickable(onClick = onClick)
            .padding(padding)
            .fillMaxWidth()
    ) {
        Row(verticalAlignment = Alignment.CenterVertically) { /*...*/ }
        Spacer(Modifier.size(padding))
        Card(
            elevation = CardDefaults.cardElevation(defaultElevation = 4.dp),
        ) { /*...*/ }
    }
}
```
![](https://developer.android.com/static/images/jetpack/compose/layout-with-modifiers.png)

Yukarıdaki kodda, birlikte kullanılan farklı modifier fonksiyonlarına dikkat edin.

- `clickable` bir composable'ın kullanıcı girdisine tepki vermesini sağlar ve bir dalgalanma gösterir.
- `padding` bir öğenin etrafına boşluk koyar.
- `fillMaxWidth`, composable öğesinin ebeveyninden kendisine verilen maksimum genişliği doldurmasını sağlar.
- `size()` bir öğenin tercih edilen genişlik ve yüksekliğini belirtir.

{: .note}
Not: Diğer şeylerin yanı sıra, modifier'lar view tabanlı layout'lardaki layout parametrelerine benzer bir rol oynarlar. Bununla birlikte, modifier'lar bazen scope-specific olduklarından, tip güvenliği sunarlar ve ayrıca belirli bir layout için neyin mevcut ve uygulanabilir olduğunu keşfetmenize ve anlamanıza yardımcı olurlar. XML layout'larında, belirli bir layout attribute'ünün belirli bir view için geçerli olup olmadığını bulmak bazen zordur.

## Scrollable layouts
Compose gestures belgelerinde kaydırılabilir layoutlar hakkında daha fazla bilgi edinin.

Listeler ve lazy listeler için Compose lists belgelerine göz atın.

## Responsive layouts
Bir layout, farklı ekran oryantasyonları ve form faktörü boyutları göz önünde bulundurularak tasarlanmalıdır. Compose, composable layout'larınızı çeşitli ekran konfigürasyonlarına uyarlamayı kolaylaştırmak için kutudan çıktığı gibi birkaç mekanizma sunar.

### Constraints
Parent'tan gelen kısıtlamaları bilmek ve layout'u buna göre tasarlamak için bir BoxWithConstraints kullanabilirsiniz. Ölçüm kısıtlamaları content lambda kapsamında bulunabilir. Bu ölçüm kısıtlamalarını, farklı ekran yapılandırmaları için farklı layoutlar oluşturmak için kullanabilirsiniz:
```kotlin
@Composable
fun WithConstraintsComposable() {
    BoxWithConstraints {
        Text("My minHeight is $minHeight while my maxWidth is $maxWidth")
    }
}
```

## Slot-based layouts
Compose, UI oluşturmayı kolaylaştırmak için androidx.compose.material:material bağımlılığı (Android Studio'da bir Compose projesi oluştururken dahil edilir) ile [Materyal Tasarımı](https://material.io/design/)na dayalı çok çeşitli bileşenler sağlar. [Drawer](https://material.io/components/navigation-drawer/), [FloatingActionButton](https://material.io/components/buttons-floating-action-button/) ve [TopAppBar](https://material.io/components/app-bars-top) gibi öğelerin tümü sağlanmıştır.

Material bileşenleri, Compose'un bileşenlerin üzerine bir özelleştirme katmanı getirmek için sunduğu bir model olan slot API'lerini yoğun bir şekilde kullanır. Bu yaklaşım, bileşenleri daha esnek hale getirir, çünkü child öğenin her yapılandırma parametresini ortaya çıkarmak zorunda kalmak yerine kendini yapılandırabilen bir child öğeyi kabul ederler. Slotlar, geliştiricinin istediği gibi doldurması için UI'da boş bir alan bırakır. Örneğin, bunlar bir [TopAppBar](https://material.io/components/app-bars-top)'da özelleştirebileceğiniz slotlardır:

![](https://developer.android.com/static/images/jetpack/compose/layout-appbar-slots.png)

Composable'lar genellikle bir content composable lambda alır ( `content: @Composable () -> Unit`). Slot API'leri belirli kullanımlar için birden fazla content parametresi sunar. Örneğin, `TopAppBar` `title`, `navigationIcon` ve `actions` için içerik sağlamanıza olanak tanır.

Örneğin, Scaffold temel Material Design layout yapısına sahip bir UI uygulamanıza olanak tanır. [Scaffold](https://developer.android.com/reference/kotlin/androidx/compose/material3/package-summary#Scaffold(androidx.compose.ui.Modifier,kotlin.Function0,kotlin.Function0,kotlin.Function0,kotlin.Function0,androidx.compose.material3.FabPosition,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.foundation.layout.WindowInsets,kotlin.Function1)), [TopAppBar](https://material.io/components/app-bars-top), [BottomAppBar](https://material.io/components/app-bars-bottom/), [FloatingActionButton](https://material.io/components/buttons-floating-action-button/) ve [Drawer](https://material.io/components/navigation-drawer/) gibi en yaygın üst düzey Material bileşenleri için slotlar sağlar. `Scaffold`'u kullanarak bu komponentlerin doğru şekilde konumlandırıldığından ve birlikte doğru şekilde çalıştığından emin olmak kolaydır.

![](https://developer.android.com/static/images/jetpack/compose/layout-jetnews-scaffold.png)

```kotlin
@Composable
fun HomeScreen(/*...*/) {
    ModalNavigationDrawer(drawerContent = { /* ... */ }) {
        Scaffold(
            topBar = { /*...*/ }
        ) { contentPadding ->
            // ...
        }
    }
}
```
