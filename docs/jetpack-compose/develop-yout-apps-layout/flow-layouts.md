---
layout: default
title: Flow layouts
parent: Develop your app's layout
grand_parent: Jetpack Compose
nav_order: 6
---

# Flow layouts in Compose

{: .note}
Beta: [FlowRow](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary#FlowRow(androidx.compose.ui.Modifier,androidx.compose.foundation.layout.Arrangement.Horizontal,androidx.compose.foundation.layout.Arrangement.Vertical,kotlin.Int,kotlin.Function1))
ve [FlowColumn](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary#FlowColumn(androidx.compose.ui.Modifier,androidx.compose.foundation.layout.Arrangement.Vertical,androidx.compose.foundation.layout.Arrangement.Horizontal,kotlin.Int,kotlin.Function1))
test aşamasındadır. Bu doküman API'nin beta sürümü içindir. Herhangi bir geri
bildirimi [issue tracker](https://issuetracker.google.com/issues/new?component=856989&template=1425922)'a gönderin.

FlowRow ve FlowColumn, Row ve Column'a benzeyen ancak container'da yer kalmadığında öğelerin bir sonraki satıra geçmesi
bakımından farklılık gösteren composable'lardır. Bu, birden fazla satır veya sütun oluşturur. Bir satırdaki öğe sayısı,
maxItemsInEachRow veya maxItemsInEachColumn ayarlanarak da kontrol edilebilir. FlowRow ve FlowColumn'u genellikle
responsive layout'lar oluşturmak için kullanabilirsiniz; öğeler bir boyut için çok büyükse içerik bölünmez ve
maxItemsInEach* ile Modifier.weight(weight) kombinasyonunu kullanmak, gerektiğinde bir satırın veya sütunun genişliğini
dolduran / genişleten (fill/expand) layout'lar oluşturmaya yardımcı olabilir.

Tipik örnek, bir chip veya filtreleme kullanıcı arayüzü içindir:

![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_row_simple.png)

## Basic usage

FlowRow veya FlowColumn kullanmak için bu composable'ları oluşturun ve standart akışı takip etmesi gereken öğeleri içine
yerleştirin:

```kotlin
@Composable
private fun FlowRowSimpleUsageExample() {
    FlowRow(modifier = Modifier.padding(8.dp)) {
        ChipItem("Price: High to Low")
        ChipItem("Avg rating: 4+")
        ChipItem("Free breakfast")
        ChipItem("Free cancellation")
        ChipItem("£50 pn")
    }
}
```

Bu kod parçacığı, ilk satırda yer kalmadığında öğelerin otomatik olarak bir sonraki satıra geçtiği yukarıda gösterilen
kullanıcı arayüzüyle sonuçlanır.

## Features of flow layout

Flow layout'ları, uygulamanızda farklı layout'lar oluşturmak için kullanabileceğiniz aşağıdaki özelliklere ve
niteliklere sahiptir.

### Main axis arrangement: horizontal or vertical arrangement

Main axis, öğelerin yerleştirildiği eksendir (örneğin, FlowRow'da öğeler yatay olarak yerleştirilir). FlowRow'daki
horizontalArrangement parametresi, boş alanın öğeler arasında nasıl dağıtılacağını kontrol eder.

Aşağıdaki tabloda FlowRow için öğeler üzerinde horizontalArrangement ayarlama örnekleri gösterilmektedir:

| FlowRow üzerinde yatay düzenleme(arrangement) ayarı                                                                                                               | Sonuc                                                                                                                |
|:------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------|
| [Arrangement.Start](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/Arrangement#Start())(Default)                               | ![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_row_arrangement_start_default.png) |
| [Arrangement.SpaceBetween  ](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/Arrangement#SpaceBetween())                        | ![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_row_arrangement_space_between.png) |
| [Arrangement.Center](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/Arrangement#Center())                                      | ![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_row_arrangement_space_center.png)  |
| [Arrangement.End](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/Arrangement#End())                                            | ![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_row_arrangement_end.png)           |
| [Arrangement.SpaceAround](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/Arrangement#SpaceAround())                            | ![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_row_arrangement_space_around.png)  |
| [Arrangement.spacedBy(8.dp)](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/Arrangement#spacedBy(androidx.compose.ui.unit.Dp)) | ![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_row_arrangement_spaced_by.png)     |

FlowColumn için ise, default Arrangement.Top secenegidir. verticalArrangement icin benzer seçenekler mevcuttur.

### Cross axis arrangement

Cross axis (çapraz eksen), main axis'in (ana eksen) ters yönündeki eksendir. Örneğin, FlowRow'da bu dikey eksendir.
Konteyner içindeki genel içeriğin çapraz eksende nasıl düzenlendiğini değiştirmek için FlowRow için verticalArrangement
ve FlowColumn için horizontalArrangement kullanın.

FlowRow için, aşağıdaki tabloda öğeler üzerinde farklı verticalArrangement ayarlama örnekleri gösterilmektedir:

| FlowRow üzerinde dikey düzenleme(arrangment) ayarı                                                                              | Sonuc                                                                                                               |
|:--------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------|
| [Arrangement.Top](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/Arrangement#Top())(Default) | ![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_container_arrangement_top.png)    |
| [Arrangement.Bottom](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/Arrangement#Bottom())    | ![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_container_arrangement_bottom.png) |
| [Arrangement.Center](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/Arrangement#Center())    | ![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_container_arrangement_center.png) |

FlowColumn için horizontalArrangement için benzer seçenekler mevcuttur. Varsayılan çapraz eksen düzenlemesi
Arrangement.Start'tır.

### Individual item alignment

Row içindeki öğeleri farklı hizalamalarla tek tek konumlandırmak isteyebilirsiniz. Bu, öğeleri geçerli satır içinde
hizaladığı için verticalArrangement ve horizontalArrangement öğelerinden farklıdır. Bunu Modifier.align() ile
uygulayabilirsiniz.

Örneğin, bir FlowRow içindeki öğeler farklı yüksekliklerde olduğunda, satır en büyük öğenin yüksekliğini alır ve öğelere
Modifier.align(alignmentOption) uygular:

| FlowRow üzerinde dikey hizalama(alignment) ayarı                                                                              | Sonuc                                                                                                            |
|:------------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------|
| [Alignment.Top](https://developer.android.com/reference/kotlin/androidx/compose/ui/Alignment#Top())(Default)                  | ![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_row_item_alignment_top.png)    |
| [Alignment.Bottom](https://developer.android.com/reference/kotlin/androidx/compose/ui/Alignment#Bottom())                     | ![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_row_item_alignment_bottom.png) |
| [Alignment.CenterVertically](https://developer.android.com/reference/kotlin/androidx/compose/ui/Alignment#CenterVertically()) | ![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_row_item_alignment_center.png) |

FlowColumn için benzer seçenekler mevcuttur. Default hizalama Alignment.Start'tır.

### Max items in row or column

maxItemsInEachRow veya maxItemsInEachColumn parametreleri, bir sonraki satıra geçmeden önce bir satırda izin verilecek
ana eksendeki maksimum öğeleri tanımlar. Varsayılan değer Int.MAX_INT olup, boyutları satıra sığmalarına izin verdiği
sürece mümkün olduğunca çok öğeye izin verir.

Örneğin, maxItemsInEachRow ayarı ilk layout'u yalnızca 3 öğeye sahip olacak şekilde zorlar:

| No max set                                                                                        | maxItemsInEachRow = 3                                                                                |
|:--------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------|
| ![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_row_no_max.png) | ![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_row_max_items.png) |

### Item weights

Weight, bir öğeyi faktörüne ve yerleştirildiği satırdaki kullanılabilir alana göre büyütür. Önemli olarak, FlowRow ve
Row arasında bir öğenin genişliğini hesaplamak için weight'in nasıl kullanıldığı ile ilgili bir fark vardır. Row için
weight, Row'daki tüm öğeleri temel alır. FlowRow ile weight, FlowRow konteynerindeki tüm öğelere değil, öğenin
yerleştirildiği satırdaki öğeleri temel alır.

Örneğin, her biri 1f, 2f, 1f ve 3f farklı weight'e sahip, hepsi bir satıra düşen 4 öğeniz varsa, toplam ağırlık 7f olur.
Bir row veya column'da kalan alan 7f'ye bölünecektir. Ardından, her bir öğe genişliği şu şekilde hesaplanacaktır:
weight * (remainingSpace / totalWeight).

Grid benzeri bir layout oluşturmak için FlowRow veya FlowColumn ile Modifier.weight ve max items kombinasyonunu
kullanabilirsiniz. Bu yaklaşım, cihazınızın boyutuna göre ayarlanan responsive layout'lar oluşturmak için kullanışlıdır.

Weight kullanarak elde edebileceğiniz birkaç farklı örnek vardır. Örneklerden biri, aşağıda gösterildiği gibi öğelerin
eşit boyutta olduğu bir griddir:

![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_layout_grid_blue.png)

Eşit öğe boyutlarında bir grid oluşturmak için aşağıdakileri yapabilirsiniz:

```kotlin
val rows = 3
val columns = 3
FlowRow(
    modifier = Modifier.padding(4.dp),
    horizontalArrangement = Arrangement.spacedBy(4.dp),
    maxItemsInEachRow = rows
) {
    val itemModifier = Modifier
        .padding(4.dp)
        .height(80.dp)
        .weight(1f)
        .clip(RoundedCornerShape(8.dp))
        .background(MaterialColors.Blue200)
    repeat(rows * columns) {
        Spacer(modifier = itemModifier)
    }
}
```

Daha da önemlisi, başka bir öğe ekler ve bunu 9 yerine 10 kez tekrarlarsanız, tüm satırın toplam weight'i 1f olduğundan,
son öğe son sütunun tamamını kaplar:

![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_layout_grid_last_item_large.png)

Weight'leri Modifier.width(exactDpAmount), Modifier.aspectRatio(aspectRatio) veya Modifier.fillMaxWidth(fraction) gibi
diğer Modifier'larla birleştirebilirsiniz. Bu modifier'ların hepsi birlikte çalışarak bir FlowRow (veya FlowColumn)
içindeki öğelerin responsive boyutlandırılmasına olanak sağlar.

Ayrıca, iki öğenin her birinin genişliğin yarısını kapladığı ve bir öğenin bir sonraki sütunun tam genişliğini kapladığı
farklı öğe boyutlarından oluşan alternatif bir grid de oluşturabilirsiniz:

![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_row_alternating_grid.png)

Bunu aşağıdaki kod ile gerçekleştirebilirsiniz:

```kotlin
FlowRow(
    modifier = Modifier.padding(4.dp),
    horizontalArrangement = Arrangement.spacedBy(4.dp),
    maxItemsInEachRow = 2
) {
    val itemModifier = Modifier
        .padding(4.dp)
        .height(80.dp)
        .clip(RoundedCornerShape(8.dp))
        .background(Color.Blue)
    repeat(6) { item ->
        // if the item is the third item, don't use weight modifier, but rather fillMaxWidth
        if ((item + 1) % 3 == 0) {
            Spacer(modifier = itemModifier.fillMaxWidth())
        } else {
            Spacer(modifier = itemModifier.weight(0.5f))
        }
    }
}
```

### Fractional sizing

Modifier.fillMaxWidth(fraction) kullanarak, bir öğenin kaplaması gereken konteyner boyutunu belirtebilirsiniz. Bu,
Modifier.fillMaxWidth(fraction) öğesinin Row veya Column öğesine uygulandığında çalışmasından farklıdır, çünkü
Row/Column öğeleri tüm konteyner genişliği yerine kalan genişliğin bir yüzdesini alır.

Örneğin, aşağıdaki kod FlowRow ile Row kullanıldığında farklı sonuçlar üretir:

```kotlin
FlowRow(
    modifier = Modifier.padding(4.dp),
    horizontalArrangement = Arrangement.spacedBy(4.dp),
    maxItemsInEachRow = 3
) {
    val itemModifier = Modifier
        .clip(RoundedCornerShape(8.dp))
    Box(modifier = itemModifier.height(200.dp).width(60.dp).background(Color.Red))
    Box(modifier = itemModifier.height(200.dp).fillMaxWidth(0.7f).background(Color.Blue))
    Box(modifier = itemModifier.height(200.dp).weight(1f).background(Color.Magenta))
}
```

| FlowRow:Tüm konteyner genişliğinin 0,7 fraksiyonuna sahip ortanca öğe. | ![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_row_fractional_width_flow.png) |
|:-----------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------|
| Row: Kalan Satır genişliğinin yüzde 0,7'sini kaplayan ortanca öğe.     | ![](https://developer.android.com/static/images/jetpack/compose/layouts/flow/flow_row_fractional_width_row.png)  |

