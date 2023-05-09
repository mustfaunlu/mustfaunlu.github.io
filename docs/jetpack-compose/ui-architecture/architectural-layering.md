---
layout: default
title: Architectural Layering
parent: UI Architecture
grand_parent: Jetpack Compose
nav_order: 6
---
# Jetpack Compose architectural layering

Bu sayfa, Jetpack Compose'u oluşturan mimari katmanlara ve bu tasarımı bilgilendiren temel ilkelere üst düzey bir genel bakış sağlar.

Jetpack Compose tek bir monolitik proje değildir; eksiksiz bir stack oluşturmak için bir araya getirilen bir dizi modülden oluşturulur. Jetpack Compose'u oluşturan farklı modülleri anlamak şunları yapmanızı sağlar:

- Uygulamanızı veya kütüphanenizi oluşturmak için uygun soyutlama seviyesini kullanın


- Daha fazla kontrol veya özelleştirme için ne zaman daha düşük bir seviyeye 'inebileceğinizi' anlayın


- Bağımlılıklarınızı en aza indirin

## Layers
Jetpack Compose'un başlıca katmanları şunlardır:
![](https://developer.android.com/static/images/jetpack/compose/layering-major-layers.svg)

Şekil 1. Jetpack Compose'un ana katmanları

Her katman, daha üst düzey bileşenler oluşturmak için işlevselliği birleştirerek alt düzeyler üzerine inşa edilmiştir. Her katman, modül sınırlarını doğrulamak ve gerektiğinde herhangi bir katmanı değiştirmenize olanak sağlamak için alt katmanların genel API'lerini temel alır. Şimdi bu katmanları aşağıdan yukarıya doğru inceleyelim.

**[Runtime](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary)**

>Bu modül, remember, mutableStateOf, @Composable annotation ve SideEffect gibi Compose çalışma zamanının temellerini sağlar. Compose'un UI'sine değil, yalnızca ağaç yönetimi yeteneklerine ihtiyacınız varsa, doğrudan bu katman üzerine inşa etmeyi düşünebilirsiniz.

**[UI](https://developer.android.com/reference/kotlin/androidx/compose/ui/package-summary)**

>UI katmanı birden fazla modülden oluşur (ui-text, ui-graphics, ui-tooling, vb.). Bu modüller LayoutNode, Modifier, input handlers, custom layouts ve çizim gibi UI araç setinin temellerini uygular. Yalnızca UI araç setinin temel kavramlarına ihtiyacınız varsa bu katman üzerine inşa etmeyi düşünebilirsiniz.

**[Foundation](https://developer.android.com/reference/kotlin/androidx/compose/foundation/package-summary)**

>Bu modül, Compose UI için Row and Column, LazyColumn, belirli hareketlerin tanınması gibi tasarım sisteminden bağımsız yapı taşları sağlar. Kendi tasarım sisteminizi oluşturmak için bu katman üzerine inşa etmeyi düşünebilirsiniz.

**[Material](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary)**

>Bu modül, Compose UI için Material Design sisteminin bir implementasyonunu sağlayarak bir tema sistemi, stilize bileşenler, dalgalanma göstergeleri ve ikonlar sunar. Uygulamanızda Materyal Tasarımı kullanırken bu katmanı temel alın.


## Design principles
Jetpack Compose için yol gösterici bir prensip, birkaç monolitik bileşen yerine bir araya getirilebilen (veya oluşturulabilen) küçük, odaklanmış işlevsellik parçaları sağlamaktır. Bu yaklaşımın bir dizi avantajı vardır.

### Control
Daha yüksek seviyeli komponentler sizin için daha fazlasını yapma eğilimindedir, ancak sahip olduğunuz doğrudan kontrol miktarını sınırlar. Daha fazla kontrole ihtiyacınız varsa, daha düşük seviyeli bir komponent kullanmak için 'aşağı inebilirsiniz'.

Örneğin, bir komponentin rengini canlandırmak istiyorsanız [animateColorAsState](https://developer.android.com/reference/kotlin/androidx/compose/animation/package-summary#animateColorAsState(androidx.compose.ui.graphics.Color,androidx.compose.animation.core.AnimationSpec,kotlin.Function1)) API'sini kullanabilirsiniz:
```kotlin
val color = animateColorAsState(if (condition) Color.Green else Color.Red)
```
Ancak daha sonra komponentin her zaman gri renkte başlamasını isterseniz, bunu bu API ile yapamazsınız. Bunun yerine daha düşük seviyeli [Animatable](https://developer.android.com/reference/kotlin/androidx/compose/animation/core/package-summary?hl=HU#Animatable(kotlin.Float,kotlin.Float)) API'yi kullanmak için aşağı inebilirsiniz:
```kotlin
val color = remember { Animatable(Color.Gray) }
LaunchedEffect(condition) {
    color.animateTo(if (condition) Color.Green else Color.Red)
}
```
Üst düzey animateColorAsState API'si, alt düzey Animatable API'si üzerine inşa edilmiştir. Alt seviye API'yi kullanmak daha karmaşıktır ancak daha fazla kontrol sunar. İhtiyaçlarınıza en uygun soyutlama seviyesini seçin.

### Customization
Daha küçük yapı taşlarından daha üst düzey komponentleri bir araya getirmek, ihtiyaç duyduğunuzda komponentleri özelleştirmeyi çok daha kolay hale getirir. Örneğin, Material katmanı tarafından sağlanan `Button` [implementasyonunu](https://cs.android.com/androidx/platform/frameworks/support/+/androidx-main:compose/material/material/src/commonMain/kotlin/androidx/compose/material/Button.kt) düşünün:
```kotlin
@Composable
fun Button(
    // …
    content: @Composable RowScope.() -> Unit
) {
    Surface(/* … */) {
        CompositionLocalProvider(/* … */) { // set LocalContentAlpha
            ProvideTextStyle(MaterialTheme.typography.button) {
                Row(
                    // …
                    content = content
                )
            }
        }
    }
}
```
Bir Buton 4 komponentten bir araya getirilir:

- Arka plan, şekil, tıklama yönetimi vb. sağlayan bir Material [Surface](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=HU#Surface(kotlin.Function0,androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Shape,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.foundation.BorderStroke,androidx.compose.ui.unit.Dp,androidx.compose.foundation.interaction.MutableInteractionSource,androidx.compose.foundation.Indication,kotlin.Boolean,kotlin.String,androidx.compose.ui.semantics.Role,kotlin.Function0)).

- Buton etkinleştirildiğinde veya devre dışı bırakıldığında içeriğin alfasını değiştiren bir [CompositionLocalProvider](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#CompositionLocalProvider(kotlin.Array,kotlin.Function0))

- [ProvideTextStyle](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=HU#ProvideTextStyle(androidx.compose.ui.text.TextStyle,kotlin.Function0)), kullanılacak varsayılan metin stilini ayarlar

- `Row`, butonun içeriği için varsayılan layout ilkesini sağlar

Yapıyı daha net hale getirmek için bazı parametreleri ve açıklamaları çıkardık, ancak tüm komponent sadece yaklaşık 40 satır koddan oluşuyor çünkü butonu gerçekleştirmek için sadece bu 4 bileşeni bir araya getiriyor. Button gibi komponentler hangi parametreleri açığa çıkaracakları konusunda fikir sahibidirler ve bir komponentin kullanımını zorlaştırabilecek bir parametre patlamasına karşı ortak özelleştirmeleri mümkün kılmayı dengelerler. Örneğin Material komponentleri, Material Design sisteminde belirtilen özelleştirmeleri sunarak material tasarım ilkelerini takip etmeyi kolaylaştırır.

Bununla birlikte, bir komponentin parametrelerinin ötesinde bir özelleştirme yapmak isterseniz, bir seviye 'aşağı inebilir' ve bir komponenti forklayabilirsiniz. Örneğin, Materyal Tasarım butonların düz renkli bir arka plana sahip olması gerektiğini belirtir. Gradyan bir arka plana ihtiyacınız varsa, bu seçenek Button parametreleri tarafından desteklenmez. Bu durumda Material Button implementasyonunu referans olarak kullanabilir ve kendi komponentinizi oluşturabilirsiniz:
```kotlin
@Composable
fun GradientButton(
    // …
    background: List<Color>,
    modifier: Modifier = Modifier,
    content: @Composable RowScope.() -> Unit
) {
    Row(
        // …
        modifier = modifier
            .clickable(onClick = {})
            .background(
                Brush.horizontalGradient(background)
            )
    ) {
        CompositionLocalProvider(/* … */) { // set material LocalContentAlpha
            ProvideTextStyle(MaterialTheme.typography.button) {
                content()
            }
        }
    }
}
```

Yukarıdaki implementasyon, materyalin geçerli content alpha ve geçerli metin stili kavramları gibi materyal katmanındaki bileşenleri kullanmaya devam eder. Ancak material Surface'i bir Row ile değiştirir ve istenen görünümü elde etmek için onu stilize eder.

{: .warning }
Dikkat: Bir komponenti özelleştirmek için daha düşük bir katmana inerken, örneğin erişilebilirlik desteğini ihmal ederek herhangi bir işlevselliği azaltmadığınızdan emin olun. Forking yaptığınız komponenti bir rehber olarak kullanın.

Material kavramlarını hiç kullanmak istemiyorsanız, örneğin kendi özel tasarım sisteminizi oluşturuyorsanız, yalnızca Foundation katman bileşenlerini kullanmaya geçebilirsiniz:

```kotlin
@Composable
fun BespokeButton(
    // …
    backgroundColor: Color,
    modifier: Modifier = Modifier,
    content: @Composable RowScope.() -> Unit
) {
    Row(
        // …
        modifier = modifier
            .clickable(onClick = {})
            .background(backgroundColor)
    ) {
        // No Material components used
        content()
    }
}
```

Jetpack Compose, en üst düzey komponentler için en basit isimleri ayırır. Örneğin, [androidx.compose.material.Text](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=HU#Text(kotlin.String,androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Color,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.font.FontStyle,androidx.compose.ui.text.font.FontWeight,androidx.compose.ui.text.font.FontFamily,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.style.TextDecoration,androidx.compose.ui.text.style.TextAlign,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.style.TextOverflow,kotlin.Boolean,kotlin.Int,kotlin.Function1,androidx.compose.ui.text.TextStyle)), [androidx.compose.foundation.text.BasicText](https://developer.android.com/reference/kotlin/androidx/compose/foundation/text/package-summary#BasicText(kotlin.String,androidx.compose.ui.Modifier,androidx.compose.ui.text.TextStyle,kotlin.Function1,androidx.compose.ui.text.style.TextOverflow,kotlin.Boolean,kotlin.Int)) üzerine inşa edilmiştir. Bu, daha yüksek seviyeleri değiştirmek isterseniz kendi implementasyonunuzu en keşfedilebilir isimle sağlamanızı mümkün kılar.

{: .warning }
Dikkat: Bir komponenti forklamak, upstream komponentin gelecekteki eklemelerinden veya hata düzeltmelerinden yararlanamayacağınız anlamına gelir.

### Picking the right abstraction
Compose'un katmanlı, yeniden kullanılabilir komponentler oluşturma felsefesi, her zaman alt seviye yapı taşlarına ulaşmamanız gerektiği anlamına gelir. Birçok üst düzey bileşen yalnızca daha fazla işlevsellik sunmakla kalmaz, aynı zamanda erişilebilirliği desteklemek gibi en iyi pratikleri de uygular.

Örneğin, özel komponentinize gesture desteği eklemek istiyorsanız, bunu [Modifier.pointerInput](https://developer.android.com/reference/kotlin/androidx/compose/ui/input/pointer/package-summary#(androidx.compose.ui.Modifier).pointerInput(kotlin.Any,kotlin.coroutines.SuspendFunction1)) kullanarak sıfırdan oluşturabilirsiniz, ancak bunun üzerine inşa edilmiş ve daha iyi bir başlangıç noktası sunabilecek [Modifier.draggable](https://developer.android.com/reference/kotlin/androidx/compose/foundation/gestures/package-summary#(androidx.compose.ui.Modifier).draggable(androidx.compose.foundation.gestures.DraggableState,androidx.compose.foundation.gestures.Orientation,kotlin.Boolean,androidx.compose.foundation.interaction.MutableInteractionSource,kotlin.Boolean,kotlin.coroutines.SuspendFunction2,kotlin.coroutines.SuspendFunction2,kotlin.Boolean)), [Modifier.scrollable](https://developer.android.com/reference/kotlin/androidx/compose/foundation/gestures/package-summary#(androidx.compose.ui.Modifier).scrollable(androidx.compose.foundation.gestures.ScrollableState,androidx.compose.foundation.gestures.Orientation,kotlin.Boolean,kotlin.Boolean,androidx.compose.foundation.gestures.FlingBehavior,androidx.compose.foundation.interaction.MutableInteractionSource)) veya [Modifier.swipeable](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary#(androidx.compose.ui.Modifier).swipeable(androidx.compose.material.SwipeableState,kotlin.collections.Map,androidx.compose.foundation.gestures.Orientation,kotlin.Boolean,kotlin.Boolean,androidx.compose.foundation.interaction.MutableInteractionSource,kotlin.Function2,androidx.compose.material.ResistanceConfig,androidx.compose.ui.unit.Dp)) gibi daha yüksek seviyeli başka komponentler de vardır.

Kural olarak, içerdikleri en iyi pratiklerden faydalanmak için ihtiyacınız olan işlevselliği sunan en üst düzey komponent üzerine inşa etmeyi tercih edin.

### Learn More
Özel bir tasarım sistemi oluşturma örneği için [Jetsnack örneği](https://github.com/android/compose-samples/tree/main/Jetsnack)ne bakın.