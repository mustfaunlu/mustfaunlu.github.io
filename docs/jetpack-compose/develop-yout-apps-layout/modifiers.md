---
layout: default
title: Modifiers
parent: Develop your app's layout
grand_parent: Jetpack Compose
nav_order: 3
---

# Compose modifiers
Modifier'lar bir composable'ı süslemenizi veya güçlendirmenizi sağlar. Modifier'lar şu tür şeyleri yapmanıza izin verir:

- Composable'ın boyutunu, layout'unu, davranışını ve görünümünü değiştirme
- Erişilebilirlik etiketleri gibi bilgiler ekleyin
- Kullanıcı girdisini işleme
- Bir öğeyi tıklanabilir, kaydırılabilir, sürüklenebilir veya yakınlaştırılabilir yapmak gibi üst düzey etkileşimler ekleyin

Modifier'lar standart Kotlin nesneleridir. Modifier sınıfı fonksiyonlarından birini çağırarak bir modifier oluşturun:

```kotlin
@Composable
private fun Greeting(name: String) {
    Column(modifier = Modifier.padding(24.dp)) {
        Text(text = "Hello,")
        Text(text = name)
    }
}
```
![](https://developer.android.com/static/images/jetpack/compose/modifier-1-modifier.png)

Bu fonksiyonları birleştirerek bir araya(compose) getirebilirsiniz:

```kotlin
@Composable
private fun Greeting(name: String) {
    Column(
        modifier = Modifier
            .padding(24.dp)
            .fillMaxWidth()
    ) {
        Text(text = "Hello,")
        Text(text = name)
    }
}
```

![](https://developer.android.com/static/images/jetpack/compose/modifier-chained.png)

Yukarıdaki kodda, birlikte kullanılan farklı modifier fonksiyonlarına dikkat edin.

- padding bir öğenin etrafına boşluk koyar.
- fillMaxWidth, composable'ın parent'ından kendisine verilen maksimum genişliği doldurmasını sağlar.

Tüm Composable'larınızın bir modifier parametresi kabul etmesi ve bu modifier'ı UI yayan ilk child'ına aktarması en iyi pratiktir. Bunu yapmak kodunuzu daha yeniden kullanılabilir hale getirir ve davranışını daha öngörülebilir ve sezgisel kılar. Daha fazla bilgi için Compose API yönergelerine, Elements accept and respect a Modifier parametresine bakın.

## Order of modifiers matters
Modifier fonksiyonlarının sırası önemlidir. Her fonksiyon bir önceki fonksiyon tarafından döndürülen Modifier üzerinde değişiklik yaptığından, sıra nihai sonucu etkiler. Bunun bir örneğini görelim:
```kotlin
@Composable
fun ArtistCard(/*...*/) {
    val padding = 16.dp
    Column(
        Modifier
            .clickable(onClick = onClick)
            .padding(padding)
            .fillMaxWidth()
    ) {
        // rest of the implementation
    }
}
```
![](https://developer.android.com/static/images/jetpack/compose/layout-padding-clickable.gif)

Yukarıdaki kodda, padding modifieri clickable modifierinden sonra uygulandığından, çevreleyen dolgu dahil olmak üzere tüm alan tıklanabilirdir. Modifier'ların sırası tersine çevrilirse, padding tarafından eklenen boşluk kullanıcı girişine tepki vermez:

```kotlin
@Composable
fun ArtistCard(/*...*/) {
    val padding = 16.dp
    Column(
        Modifier
            .padding(padding)
            .clickable(onClick = onClick)
            .fillMaxWidth()
    ) {
        // rest of the implementation
    }
}
```
![](https://developer.android.com/static/images/jetpack/compose/layout-padding-not-clickable.gif)

{: .note }
Not: Belirgin sıralama, farklı modifier'ların nasıl etkileşime gireceği konusunda mantık yürütmenize yardımcı olur. Bunu, kutu modelini öğrenmek zorunda kaldığınız, kenar boşluklarının öğenin "dışına" uygulandığı ancak "içine" dolgu uygulandığı ve bir arka plan öğesinin buna göre boyutlandırılacağı view tabanlı sistemle karşılaştırın. Modifier tasarımı bu tür davranışları açık ve öngörülebilir hale getirir ve tam olarak istediğiniz davranışı elde etmek için size daha fazla kontrol sağlar. Bu aynı zamanda neden bir margin modifier değil de sadece bir padding modifier olduğunu da açıklar.

## Built-in modifiers  
Jetpack Compose, bir composable'ı dekore etmenize veya zenginleştirmenize yardımcı olmak için yerleşik modifierlarin bir listesini sunar. İşte layoutlarinizi ayarlamak için kullanacağınız bazı yaygın modifierlar.

{: .note }
Not: Bu modifier'ların birçoğu, UI'nizin layout'unu ihtiyaç duyduğunuz şekilde düzenlemenize yardımcı olmak için tasarlanmıştır. Modifier'ların layout'unuzda nasıl çalıştığı hakkında daha fazla bilgi için Compose layout basics belgesine bakın.

### padding and size
Varsayılan olarak, Compose'da sağlanan layoutlar child'larını sarar. Ancak, `size` modifier'ını kullanarak bir boyut ayarlayabilirsiniz:
```kotlin
@Composable
fun ArtistCard(/*...*/) {
    Row(
        modifier = Modifier.size(width = 400.dp, height = 100.dp)
    ) {
        Image(/*...*/)
        Column { /*...*/ }
    }
}
```
Layout'un parent'ından gelen constraintleri karşılamıyorsa, belirttiğiniz boyuta itibar edilmeyebileceğini unutmayın. Gelen constraintlerden etkilenmeksizin composable boyutunun sabit olmasını istiyorsanız `requiredSize` modifier'ını kullanın:

```kotlin
@Composable
fun ArtistCard(/*...*/) {
    Row(
        modifier = Modifier.size(width = 400.dp, height = 100.dp)
    ) {
        Image(
            /*...*/
            modifier = Modifier.requiredSize(150.dp)
        )
        Column { /*...*/ }
    }
}
```
![](https://developer.android.com/static/images/jetpack/compose/layout-requiredsize-new.png)

Bu örnekte, parent height 100.dp olarak ayarlanmış olsa bile, `requiredSize` modifier'ı öncelikli olduğu için Image'ın yüksekliği 150.dp olacaktır.

{: .note }
Not: Layout'lar constraints'lere bağlıdır ve normalde parent bu constraints'leri child'lara aktarır. Child, constraintlere saygı göstermelidir. Ancak, kullanıcı arayüzünün gerektirdiği her zaman bu olmayabilir. Bu child davranışını atlamanın yolları vardır. Örneğin, requiredSize gibi modifier'ları doğrudan child'a geçirebilir, child'ın ebeveynden aldığı constrainte'ları override edebilir veya farklı davranışa sahip özel bir layout kullanabilirsiniz. Bir child kendi constraints'lerine uymadığında, layout sistemi bunu parent'tan gizleyecektir. Parent, child'ın genişlik ve yükseklik değerlerini parent tarafından sağlanan constraintlerde zorlanmış gibi görecektir. Layout sistemi daha sonra, child'ın kısıtlamalara uyduğu varsayımı altında, child'ı parent tarafından tahsis edilen alan içinde ortalayacaktır. Geliştiriciler, child'a wrapContentSize modifierları uygulayarak bu ortalama davranışını override edebilirler.

Bir child layout'un parent tarafından izin verilen tüm yüksekliği doldurmasını istiyorsanız, fillMaxHeight modifier'ını ekleyin (Compose ayrıca `fillMaxSize` ve `fillMaxWidth` sağlar):
```kotlin
@Composable
fun ArtistCard(/*...*/) {
    Row(
        modifier = Modifier.size(width = 400.dp, height = 100.dp)
    ) {
        Image(
            /*...*/
            modifier = Modifier.fillMaxHeight()
        )
        Column { /*...*/ }
    }
}
```
![](https://developer.android.com/static/images/jetpack/compose/layout-fillmaxheight.png)

Bir öğenin etrafına dolgu eklemek için bir padding modifier ayarlayın.

Bir metin taban çizgisinin üzerine, layout'un üst kısmından taban çizgisine kadar belirli bir mesafe elde edecek şekilde dolgu eklemek istiyorsanız, paddingFromBaseline modifier'ını kullanın:

```kotlin
@Composable
fun ArtistCard(artist: Artist) {
    Row(/*...*/) {
        Column {
            Text(
                text = artist.name,
                modifier = Modifier.paddingFromBaseline(top = 50.dp)
            )
            Text(artist.lastSeenOnline)
        }
    }
}
```
![](https://developer.android.com/static/images/jetpack/compose/layout-paddingfrombaseline-new.png)

### Offset
Bir layout'u orijinal konumuna göre konumlandırmak için ofset modifier'ını ekleyin ve ofseti x ve y ekseninde ayarlayın. Ofsetler pozitif olabileceği gibi pozitif olmayabilir de. `Padding` ve `offset` arasındaki fark, bir composable'a offset eklemenin onun ölçümlerini değiştirmemesidir:

```kotlin
@Composable
fun ArtistCard(artist: Artist) {
    Row(/*...*/) {
        Column {
            Text(artist.name)
            Text(
                text = artist.lastSeenOnline,
                modifier = Modifier.offset(x = 4.dp)
            )
        }
    }
}
```
![](https://developer.android.com/static/images/jetpack/compose/layout-offset-new.png)

Ofset modifier'ı layout yönüne göre yatay olarak uygulanır. Soldan sağa bağlamında, pozitif bir ofset öğeyi sağa kaydırırken, sağdan sola bağlamında öğeyi sola kaydırır. Layout yönünü dikkate almadan bir ofset ayarlamanız gerekiyorsa, pozitif bir ofset değerinin öğeyi her zaman sağa kaydırdığı absoluteOffset modifier'ına bakın.

offset modifier iki overload sağlar - ofsetleri parametre olarak alan offset ve bir lambda alan offset. Bunların her birinin ne zaman kullanılacağı ve performans için nasıl optimize edileceği hakkında daha ayrıntılı bilgi için Compose performance - Defer reads as long as possible bölümünü okuyun.

## Scope safety in Compose
Compose'da, yalnızca belirli composable'ların child'larına uygulandığında kullanılabilen modifier'lar vardır. Compose bunu custom scopelar aracılığıyla uygular.

Örneğin, bir child öğeyi Box boyutunu etkilemeden üst Box öğesi kadar büyük yapmak istiyorsanız, matchParentSize modifier'ını kullanın. matchParentSize yalnızca BoxScope içinde kullanılabilir. Bu nedenle, yalnızca bir Box parent içindeki bir child üzerinde kullanılabilir.

Scope güvenliği, diğer composable'larda ve scope'larda çalışmayacak modifier'lar eklemenizi engeller ve deneme yanılma yöntemiyle zaman kazanmanızı sağlar.

{: .note }
Not: Android View sisteminde scope safety (kapsam güvenliği) yoktur. Geliştiriciler genellikle hangilerinin dikkate alındığını ve belirli bir parent bağlamında anlamlarını keşfetmek için kendilerini farklı layout parametreleri denerken bulurlar.

Scoped modifier'lar, parent'ın child hakkında bilmesi gereken bazı bilgileri parent'a bildirir. Bunlar genellikle parent data modifier'ları olarak da adlandırılır. İç yapıları genel amaçlı modifier'lardan farklıdır, ancak kullanım açısından bu farklılıklar önemli değildir.

### matchParentSize in Box
Yukarıda belirtildiği gibi, bir child layout'un Box boyutunu etkilemeden bir parent Box ile aynı boyutta olmasını istiyorsanız, matchParentSize modifier'ını kullanın.

matchParentSize'ın yalnızca bir Box scope içinde kullanılabildiğini, yani yalnızca Box composables'ın doğrudan child'ları için geçerli olduğunu unutmayın.

Aşağıdaki örnekte, Spacer child öğesi boyutunu Parent Box öğesinden alır, o da boyutunu en büyük child öğesinden (bu durumda ArtistCard öğesinden) alır.

```kotlin
@Composable
fun MatchParentSizeComposable() {
    Box {
        Spacer(
            Modifier
                .matchParentSize()
                .background(Color.LightGray)
        )
        ArtistCard()
    }
}
```
![](https://developer.android.com/static/images/jetpack/compose/layout-matchparentsize-new.png)
`matchParentSize` yerine `fillMaxSize` kullanılsaydı, Spacer parent'a izin verilen tüm boş alanı alacak ve bu da parent'ın genişlemesine ve tüm boş alanı doldurmasına neden olacaktı.
![](https://developer.android.com/static/images/jetpack/compose/layout-fillmaxsize.png)

### weight in Row and Column
Padding ve size ile ilgili önceki bölümde gördüğünüz gibi, varsayılan olarak, bir composable size sardığı içerik tarafından tanımlanır. Yalnızca RowScope ve ColumnScope'ta kullanılabilen ağırlık Modifier'ını kullanarak composable size'ı parent'ı içinde esnek olacak şekilde ayarlayabilirsiniz.

İki Box composable içeren bir Row'u ele alalım. İlk Box'a ikincisinin iki katı weight verildiğinden genişliği de iki katına çıkar. Row 210.dp genişliğinde olduğundan, ilk Box 140.dp genişliğinde ve ikincisi 70.dp genişliğindedir:

```kotlin
@Composable
fun ArtistCard(/*...*/) {
    Row(
        modifier = Modifier.fillMaxWidth()
    ) {
        Image(
            /*...*/
            modifier = Modifier.weight(2f)
        )
        Column(
            modifier = Modifier.weight(1f)
        ) {
            /*...*/
        }
    }
}
```
![](https://developer.android.com/static/images/jetpack/compose/layout-weight.png)

## Extracting and reusing modifiers
Bir composable'ı dekore etmek veya zenginleştirmek için birden fazla modifier birbirine zincirlenebilir. Bu zincir, tek [Modifier.Elements](https://developer.android.com/reference/kotlin/androidx/compose/ui/Modifier.Element)'in sıralı, değişmez bir listesini temsil eden [Modifier](https://developer.android.com/reference/kotlin/androidx/compose/ui/Modifier) interface'i aracılığıyla oluşturulur.

Her `Modifier.Element`, layout, çizim ve grafik davranışları, jestle ilgili tüm davranışlar, odak ve semantik davranışlarının yanı sıra cihaz giriş olayları gibi ayrı bir davranışı temsil eder. Bunların sıralaması önemlidir: ilk eklenen modifier öğeleri ilk olarak uygulanacaktır.

Bazen aynı modifier zinciri örneklerini değişkenlere ayıklayarak ve daha yüksek kapsamlara taşıyarak birden fazla composable'da yeniden kullanmak faydalı olabilir. Bu, kodun okunabilirliğini artırabilir veya birkaç nedenden dolayı uygulamanızın performansını iyileştirmeye yardımcı olabilir:

- Modifier'ların yeniden tahsisi, bunları kullanan composable'lar için recomposition gerçekleştiğinde tekrarlanmayacaktır
- Modifier zincirleri potansiyel olarak çok uzun ve karmaşık olabilir, bu nedenle bir zincirin aynı örneğini yeniden kullanmak Compose çalışma zamanının bunları karşılaştırırken yapması gereken iş yükünü hafifletebilir
- Bu ayıklama kod tabanı genelinde kod temizliğini, tutarlılığı ve sürdürülebilirliği teşvik eder

### Best practices for reusing modifiers
Kendi `Modifier` zincirlerinizi oluşturun ve bunları birden fazla composable bileşen üzerinde yeniden kullanmak için ayıklayın. Veri benzeri nesneler oldukları için bir modifier'ı sadece kaydetmek tamamen iyidir:

```kotlin
val reusableModifier = Modifier
    .fillMaxWidth()
    .background(Color.Red)
    .padding(12.dp)
```
### Extracting and reusing modifiers when observing frequently changing state
Animasyon state'leri veya scrollState gibi composable'lar içinde sık değişen state'leri gözlemlerken, önemli miktarda recomposition yapılabilir. Bu durumda, modifier'larınız her recomposition'da ve potansiyel olarak her frame için tahsis edilecektir:
```kotlin
@Composable
fun LoadingWheelAnimation() {
    val animatedState = animateFloatAsState(/*...*/)

    LoadingWheel(
        // Bu modifier'ın oluşturulması ve tahsisi animasyonun her karesinde gerçekleşecektir!
        modifier = Modifier
            .padding(12.dp)
            .background(Color.Gray),
        animatedState = animatedState
    )
}
```
Bunun yerine, modifier'ın aynı instance'ını oluşturabilir, ayıklayabilir ve yeniden kullanabilir ve bu şekilde composable'a aktarabilirsiniz:

```kotlin
// Şimdi, modifier'ın tahsisi burada gerçekleşir:
val reusableModifier = Modifier
    .padding(12.dp)
    .background(Color.Gray)

@Composable
fun LoadingWheelAnimation() {
    val animatedState = animateFloatAsState(/*...*/)

    LoadingWheel(
        // Aynı instance'ı tekrar kullandığımız için tahsis yok
        modifier = reusableModifier,
        animatedState = animatedState
    )
}
```
### Extracting and reusing unscoped modifiers
Modifier'lar scope edilmemiş veya belirli bir composable'a scop edilmiş olabilir. Scope edilmemiş modifier'lar söz konusu olduğunda, bunları basit değişkenler olarak herhangi bir composable'ın dışına kolayca çıkarabilirsiniz:

```kotlin
val reusableModifier = Modifier
    .fillMaxWidth()
    .background(Color.Red)
    .padding(12.dp)

@Composable
fun AuthorField() {
    HeaderText(
        // ...
        modifier = reusableModifier
    )
    SubtitleText(
        // ...
        modifier = reusableModifier
    )
}
```
Bu, özellikle Lazy layout'ları ile birlikte kullanıldığında faydalı olabilir. Çoğu durumda, potansiyel olarak önemli miktarda olan tüm öğelerinizin tam olarak aynı modifier'lara sahip olmasını istersiniz:

```kotlin
val reusableItemModifier = Modifier
    .padding(bottom = 12.dp)
    .size(216.dp)
    .clip(CircleShape)

@Composable
private fun AuthorList(authors: List<Author>) {
    LazyColumn {
        items(authors) {
            AsyncImage(
                // ...
                modifier = reusableItemModifier,
            )
        }
    }
}
```

### Extracting and reusing scoped modifiers
Belirli composable'lara scope edilmiş değiştiricilerle uğraşırken, bunları mümkün olan en yüksek seviyeye taşıyabilir ve uygun olan yerlerde yeniden kullanabilirsiniz:

```kotlin
Column(/*...*/) {
    val reusableItemModifier = Modifier
        .padding(bottom = 12.dp)
        // Align Modifier.Element bir ColumnScope gerektirir
        .align(Alignment.CenterHorizontally)
        .weight(1f)
    Text1(
        modifier = reusableItemModifier,
        // ...
    )
    Text2(
        modifier = reusableItemModifier
        // ...
    )
    // ...
}
```
Çıkarılan, scope edilmiş modifier'ları yalnızca aynı scope edilmiş, doğrudan child'lara aktarmalısınız. Bunun neden önemli olduğu hakkında daha fazla bilgi için Compose'da Scope safety bölümüne bakın:

```kotlin
Column(modifier = Modifier.fillMaxWidth()) {
    // Weight modifier is scoped to the Column composable
    val reusableItemModifier = Modifier.weight(1f)

    // Weight will be properly assigned here since this Text is a direct child of Column
    Text1(
        modifier = reusableItemModifier
        // ...
    )

    Box {
        Text2(
            // Weight won't do anything here since the Text composable is not a direct child of Column
            modifier = reusableItemModifier
            // ...
        )
    }
}
```

### Further chaining of extracted modifiers
Çıkarılan modifier zincirlerinizi [.then()](https://developer.android.com/reference/kotlin/androidx/compose/ui/Modifier#then(androidx.compose.ui.Modifier)) fonksiyonunu çağırarak daha fazla zincirleyebilir veya ekleyebilirsiniz:
```kotlin
val reusableModifier = Modifier
    .fillMaxWidth()
    .background(Color.Red)
    .padding(12.dp)

// Append to your reusableModifier
reusableModifier.clickable { /*...*/ }

// Append your reusableModifier
otherModifier.then(reusableModifier)
```
Sadece modifierlerin sırasının önemli olduğunu unutmayın!

## Learn more
Parametreleri ve scopelari ile birlikte modifier'ların [tam bir listesi](https://developer.android.com/jetpack/compose/modifiers-list)ni sunuyoruz.

Modifier'ların nasıl kullanılacağı hakkında daha fazla pratik yapmak için Compose codelab'deki [Basic layouts](https://developer.android.com/codelabs/jetpack-compose-layouts#0)'u inceleyebilir veya [Now in Android](https://github.com/android/nowinandroid) repository'ye başvurabilirsiniz.

Custom modifier'lar ve bunların nasıl oluşturulacağı hakkında daha fazla bilgi için [Custom layouts - Using the layout modifier](https://developer.android.com/jetpack/compose/layouts/custom#layout-modifier) belgesine göz atın.