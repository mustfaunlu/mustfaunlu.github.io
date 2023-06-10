---
layout: default
title: Custom layouts
parent: Develop your app's layout
grand_parent: Jetpack Compose
nav_order: 7
---

# Custom layouts

[Custom layouts and graphics in Compose](https://youtu.be/xcfEQO0k_gU)

Compose'da UI öğeleri, çağrıldıklarında bir UI parçası yayan(emit) ve daha sonra ekranda işlenen bir UI ağacına eklenen composable fonksiyonlarla temsil edilir. Her UI öğesinin bir ebeveyni ve potansiyel olarak birçok çocuğu vardır. Her öğe aynı zamanda (x, y) konumu ve genişlik ve yükseklik olarak belirtilen bir boyut olarak ebeveyninin içinde yer alır.

Ebeveynler, çocuk elemanları için kısıtlamaları tanımlar. Bir elemandan bu kısıtlamalar dahilinde boyutunu tanımlaması istenir. Kısıtlamalar bir elemanın minimum ve maksimum genişlik ve yüksekliğini sınırlar. Bir öğenin alt öğeleri varsa, boyutunu belirlemeye yardımcı olması için her bir alt öğeyi ölçebilir. Bir öğe kendi boyutunu belirleyip bildirdikten sonra, [Custom layout olusturma](#create-custom-layouts) bölümünde ayrıntılı olarak açıklandığı gibi, alt öğelerinin kendisine göre nasıl yerleştirileceğini tanımlama fırsatına sahip olur.

UI ağacındaki her bir node'un yerleştirilmesi üç adımlı bir süreçtir. Her node şunları yapmalıdır:

- 1- Tüm çocukları ölçün
- 2- Kendi boyutuna karar ver
- 3- Çocuklarını yerleştir

![](https://developer.android.com/static/images/jetpack/compose/layout-three-step-process.svg)

{: .note}
Not: Compose UI, çoklu geçiş ölçümüne izin vermez. Bu, bir layout öğesinin farklı ölçüm konfigürasyonlarını denemek için alt öğelerinden herhangi birini birden fazla kez ölçemeyeceği anlamına gelir.

Scope kullanımı, alt öğelerinizi ne zaman ölçebileceğinizi ve yerleştirebileceğinizi tanımlar. Bir layout'un ölçümü yalnızca ölçüm ve layout geçişleri sırasında yapılabilir ve bir alt öğe yalnızca layout geçişleri sırasında (ve yalnızca ölçüldükten sonra) yerleştirilebilir. [MeasureScope](https://developer.android.com/reference/kotlin/androidx/compose/ui/layout/MeasureScope) ve [PlacementScope](https://developer.android.com/reference/kotlin/androidx/compose/ui/layout/Placeable.PlacementScope) gibi Compose scope'ları nedeniyle, bu derleme zamanında zorunlu kılınır.

## Use the layout modifier
[Advanced layout concepts - MAD Skills](https://youtu.be/l6rAoph5UgI)

Bir öğenin nasıl ölçüldüğünü ve düzenlendiğini değiştirmek için layout modifier'ı kullanabilirsiniz. Layout bir lambda'dır; parametreleri, measurable olarak geçirilen ölçebileceğiniz öğeyi ve constraints olarak geçirilen composable'ın mevcut kısıtlamalarını içerir. Custom bir layout modifier aşağıdaki gibi görünebilir:
```kotlin
fun Modifier.customLayoutModifier() =
    layout { measurable, constraints ->
        // ...
    }
```
Ekranda bir Metin görüntüleyelim ve metnin ilk satırının üstten taban çizgisine olan mesafesini kontrol edelim. Bu tam olarak paddingFromBaseline modifier'ının yaptığı şeydir, onu burada örnek olarak uyguluyoruz. Bunu yapmak için, composable'ı ekrana manuel olarak yerleştirmek üzere layout modifier'ı kullanın. İşte Metin top padding'inin 24.dp olarak ayarlandığı istenen davranış:

![](https://developer.android.com/static/images/jetpack/compose/layout-padding-baseline.png)

İşte bu mesafeyi oluşturmak için gereken kod:

```kotlin
fun Modifier.firstBaselineToTop(
    firstBaselineToTop: Dp
) = layout { measurable, constraints ->
    // Measure the composable
    val placeable = measurable.measure(constraints)

    // Check the composable has a first baseline
    check(placeable[FirstBaseline] != AlignmentLine.Unspecified)
    val firstBaseline = placeable[FirstBaseline]

    // Height of the composable with padding - first baseline
    val placeableY = firstBaselineToTop.roundToPx() - firstBaseline
    val height = placeable.height + placeableY
    layout(placeable.width, height) {
        // Where the composable gets placed
        placeable.placeRelative(0, placeableY)
    }
}
```

İşte bu kodda neler olup bittiği:

- 1- Measurable lambda parametresinde, measurable.measure(constraints) öğesini çağırarak measurable parametresinin temsil ettiği Metni ölçersiniz.


- 2- Sarılmış öğeleri yerleştirmek için kullanılan bir lambda da veren layout(width, height) metodunu çağırarak composable'ın boyutunu belirtirsiniz. Bu durumda, son taban çizgisi ile eklenen top padding arasındaki yüksekliktir.


- 3- Placeable.place(x, y) metodunu çağırarak sarılmış elemanları ekranda konumlandırırsınız. Sarılı öğeler yerleştirilmezse görünür olmazlar. Y konumu, metnin ilk taban çizgisinin konumu olan top padding'e karşılık gelir.


Bunun beklendiği gibi çalıştığını doğrulamak için, bu modifieri bir Text üzerinde kullanın:

```kotlin
@Preview
@Composable
fun TextWithPaddingToBaselinePreview() {
    MyApplicationTheme {
        Text("Hi there!", Modifier.firstBaselineToTop(32.dp))
    }
}

@Preview
@Composable
fun TextWithNormalPaddingPreview() {
    MyApplicationTheme {
        Text("Hi there!", Modifier.padding(top = 32.dp))
    }
}
```

![](https://developer.android.com/static/images/jetpack/compose/layout-previews-showing-text-baseline-padding.png)

## Create custom layouts

```kotlin
@Composable
fun MyBasicColumn(
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
    Layout(
        modifier = modifier,
        content = content
    ) { measurables, constraints ->
        // measure and position children given constraints logic here
        // ...
    }
}
```
```kotlin
@Composable
fun MyBasicColumn(
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
    Layout(
        modifier = modifier,
        content = content
    ) { measurables, constraints ->
        // Don't constrain child views further, measure them with given constraints
        // List of measured children
        val placeables = measurables.map { measurable ->
            // Measure each children
            measurable.measure(constraints)
        }

        // Set the size of the layout as big as it can
        layout(constraints.maxWidth, constraints.maxHeight) {
            // Track the y co-ord we have placed children up to
            var yPosition = 0

            // Place children in the parent layout
            placeables.forEach { placeable ->
                // Position item on the screen
                placeable.placeRelative(x = 0, y = yPosition)

                // Record the y co-ord placed up to
                yPosition += placeable.height
            }
        }
    }
}
```
```kotlin
@Composable
fun CallingComposable(modifier: Modifier = Modifier) {
    MyBasicColumn(modifier.padding(8.dp)) {
        Text("MyBasicColumn")
        Text("places items")
        Text("vertically.")
        Text("We've done it by hand!")
    }
}
```
![](https://developer.android.com/static/images/jetpack/compose/layout-complex-by-hand.png)
## Layout direction
[LocalLayoutDirection](https://developer.android.com/reference/kotlin/androidx/compose/ui/platform/package-summary#LocalLayoutDirection()) composition local'i değiştirerek bir composable'ın yerleşim yönünü değiştirin.

Composable'ları ekrana manuel olarak yerleştiriyorsanız LayoutDirection, layout modifier'ın veya Layout composable'ın LayoutScope'unun bir parçasıdır.

LayoutDirection kullanırken, composablelari place kullanarak yerleştirin. [placeRelative](https://developer.android.com/reference/kotlin/androidx/compose/ui/layout/Placeable.PlacementScope#placeRelative(androidx.compose.ui.layout.Placeable,kotlin.Int,kotlin.Int,kotlin.Float)) metodunun aksine, [place](https://developer.android.com/reference/kotlin/androidx/compose/ui/layout/Placeable.PlacementScope#place(androidx.compose.ui.layout.Placeable,kotlin.Int,kotlin.Int,kotlin.Float)) yerleşim yönüne göre değişmez (soldan sağa veya sağdan sola).
## Custom layouts in action
Compose'daki [temel layoutlar bölümü](https://developer.android.com/codelabs/jetpack-compose-layouts#12)nde layoutlar ve modifier'lar hakkında daha fazla bilgi edinin ve [Custom Layouts oluşturan Compose örneklerinde custom layoutları](https://github.com/android/compose-samples/search?q=androidx.compose.ui.layout.Layout) uygulamalı olarak görün.
## Learn more
Compose'daki custom layout'lar hakkında daha fazla bilgi edinmek için aşağıdaki ek kaynaklara başvurun.
### Videos

- [A deep dive into Jetpack Compose Layouts](https://www.youtube.com/watch?v=zMKMwh9gZuI)