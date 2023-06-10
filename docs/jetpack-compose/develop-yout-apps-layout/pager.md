---
layout: default
title: Pager
parent: Develop your app's layout
grand_parent: Jetpack Compose
nav_order: 5
---

# Pager in Compose
İçeriği sola ve sağa veya yukarı ve aşağı şekilde kaydırmak için [HorizontalPager](https://developer.android.com/reference/kotlin/androidx/compose/foundation/pager/package-summary#HorizontalPager(kotlin.Int,androidx.compose.ui.Modifier,androidx.compose.foundation.pager.PagerState,androidx.compose.foundation.layout.PaddingValues,androidx.compose.foundation.pager.PageSize,kotlin.Int,androidx.compose.ui.unit.Dp,androidx.compose.ui.Alignment.Vertical,androidx.compose.foundation.gestures.snapping.SnapFlingBehavior,kotlin.Boolean,kotlin.Boolean,kotlin.Function1,androidx.compose.ui.input.nestedscroll.NestedScrollConnection,kotlin.Function1)) ve [VerticalPager](https://developer.android.com/reference/kotlin/androidx/compose/foundation/pager/package-summary#VerticalPager(kotlin.Int,androidx.compose.ui.Modifier,androidx.compose.foundation.pager.PagerState,androidx.compose.foundation.layout.PaddingValues,androidx.compose.foundation.pager.PageSize,kotlin.Int,androidx.compose.ui.unit.Dp,androidx.compose.ui.Alignment.Horizontal,androidx.compose.foundation.gestures.snapping.SnapFlingBehavior,kotlin.Boolean,kotlin.Boolean,kotlin.Function1,androidx.compose.ui.input.nestedscroll.NestedScrollConnection,kotlin.Function1)) composablelar bulunmaktadır. Bu bileşenler, view sistemindeki [ViewPager](https://developer.android.com/reference/kotlin/androidx/viewpager/widget/ViewPager) ile benzer işlevlere sahiptir. Varsayılan olarak, HorizontalPager ekranın tam genişliğini kaplar, VerticalPager tam yüksekliğini kaplar ve pager'lar bir seferde yalnızca bir sayfa kaydırır. Bu varsayılanların tümü ayarlanabilir.

{: .note}
Not: Compose Pager test aşamasındadır. Herhangi bir sorunu [issue tracker](https://issuetracker.google.com/issues/new?component=856989&template=1425922) üzerinden bildirin.

## HorizontalPager

Yatay olarak sola ve sağa kaydırılan bir pager oluşturmak için HorizontalPager kullanın:

![](https://developer.android.com/static/images/jetpack/compose/layouts/pager/horizontalpager_demo.mp4)

```kotlin
// 10 öğe görüntüle
HorizontalPager(pageCount = 10) { page ->
    // Sayfa içeriğimiz
    Text(
        text = "Page: $page",
        modifier = Modifier.fillMaxWidth()
    )
}
```

## VerticalPager
Yukarı ve aşağı kaydırılan bir pager oluşturmak için VerticalPager kullanın:

![](https://developer.android.com/static/images/jetpack/compose/layouts/pager/verticalpager_demo.mp4)

```kotlin
// 10 öğe görüntüle
VerticalPager(pageCount = 10) { page ->
    // Sayfa içeriğimiz
    Text(
        text = "Page: $page",
        modifier = Modifier.fillMaxWidth()
    )
}
```

## Lazy creation
Hem HorizontalPager hem de VerticalPager'daki sayfalar [lazily olarak oluşturulur]() ve gerektiğinde düzenlenir. Kullanıcı sayfalar arasında ilerledikçe, composable artık gerekli olmayan sayfaları kaldırır.

### Load more pages offscreen
Varsayılan olarak, pager yalnızca ekrandaki görünür sayfaları yükler. Ekran dışında daha fazla sayfa yüklemek için beyondBoundsPageCount değerini sıfırdan büyük bir değere ayarlayın.

## Scroll to an item in the pager
Pager'da belirli bir sayfaya kaydırmak için, [rememberPagerState()](https://developer.android.com/reference/kotlin/androidx/compose/foundation/pager/package-summary#rememberPagerState(kotlin.Int,kotlin.Float)) kullanarak bir [PagerState](https://developer.android.com/reference/kotlin/androidx/compose/foundation/pager/PagerState) nesnesi oluşturun ve bunu pager'a state parametresi olarak aktarın. Bu state üzerinde, bir CoroutineScope içinde [PagerState#scrollToPage()](https://developer.android.com/reference/kotlin/androidx/compose/foundation/pager/PagerState#scrollToPage(kotlin.Int,kotlin.Float)) fonksiyonunu çağırabilirsiniz:

```kotlin
val pagerState = rememberPagerState()

HorizontalPager(pageCount = 10, state = pagerState) { page ->
    // Sayfa içeriğimiz
    Text(
        text = "Page: $page",
        modifier = Modifier
            .fillMaxWidth()
            .height(100.dp)
    )
}

// scroll to page
val coroutineScope = rememberCoroutineScope()
Button(onClick = {
    coroutineScope.launch {
        // Call scroll to on pagerState
        pagerState.scrollToPage(5)
    }
}, modifier = Modifier.align(Alignment.BottomCenter)) {
    Text("Jump to Page 5")
}
```
Sayfaya animasyon uygulamak istiyorsanız, [PagerState#animateScrollToPage()](https://developer.android.com/reference/kotlin/androidx/compose/foundation/pager/PagerState#animateScrollToPage(kotlin.Int,kotlin.Float,androidx.compose.animation.core.AnimationSpec)) fonksiyonunu kullanın:
```kotlin
val pagerState = rememberPagerState()

HorizontalPager(pageCount = 10, state = pagerState) { page ->
    // Our page content
    Text(
        text = "Page: $page",
        modifier = Modifier
            .fillMaxWidth()
            .height(100.dp)
    )
}

// scroll to page
val coroutineScope = rememberCoroutineScope()
Button(onClick = {
    coroutineScope.launch {
        // Call scroll to on pagerState
        pagerState.animateScrollToPage(5)
    }
}, modifier = Modifier.align(Alignment.BottomCenter)) {
    Text("Jump to Page 5")
}
```

## Get notified about page state changes
[PagerState](https://developer.android.com/reference/kotlin/androidx/compose/foundation/pager/PagerState), sayfalar hakkında bilgi içeren üç property'e sahiptir: [currentPage](https://developer.android.com/reference/kotlin/androidx/compose/foundation/pager/PagerState#currentPage()), [settledPage](https://developer.android.com/reference/kotlin/androidx/compose/foundation/pager/PagerState#settledPage()) ve [targetPage](https://developer.android.com/reference/kotlin/androidx/compose/foundation/pager/PagerState#targetPage()).

- `currentPage`: Snap konumuna en yakın sayfa. Varsayılan olarak, snap konumu layout'un başlangıcındadır.

- `settledPage`: Animasyon veya kaydırma çalışmazken sayfa numarası. Bu, currentPage property'sinden farklıdır, çünkü sayfa snap pozisyonuna yeterince yakınsa currentPage hemen güncellenir, ancak settledPage tüm animasyonlar çalışmayı bitirene kadar aynı kalır.

- `targetPage`: Kaydırma hareketi için öngörülen durma konumu.

Bu değişkenlerdeki değişiklikleri gözlemlemek ve bunlara tepki vermek için snapshotFlow fonksiyonunu kullanabilirsiniz. Örneğin, her sayfa değişikliğinde bir analytics event göndermek için aşağıdakileri yapabilirsiniz:

```kotlin
val pagerState = rememberPagerState()

LaunchedEffect(pagerState) {
    // Collect from the a snapshotFlow reading the currentPage
    snapshotFlow { pagerState.currentPage }.collect { page ->
        // Do something with each page change, for example:
        // viewModel.sendPageSelectedEvent(page)
        Log.d("Page change", "Page changed to $page")
    }
}

VerticalPager(
    pageCount = 10,
    state = pagerState,
) { page ->
    Text(text = "Page: $page")
}
```

## Add a page indicator
Bir sayfaya indikatör eklemek için, PagerState nesnesini kullanarak sayfa sayısı içinden hangi sayfanın seçili olduğu hakkında bilgi alın ve custom indikatörünüzü çizin.

Örneğin, basit bir daire indikatörü istiyorsanız, pagerState.currentPage kullanarak daire sayısını tekrarlayabilir ve sayfanın seçili olup olmadığına göre daire rengini değiştirebilirsiniz:

```kotlin
val pageCount = 10
val pagerState = rememberPagerState()

HorizontalPager(
    pageCount = pageCount,
    state = pagerState
) { page ->
    // Our page content
    Text(
        text = "Page: $page",
        modifier = Modifier
            .fillMaxSize()
    )
}
Row(
    Modifier
        .height(50.dp)
        .fillMaxWidth()
        .align(Alignment.BottomCenter),
    horizontalArrangement = Arrangement.Center
) {
    repeat(pageCount) { iteration ->
        val color = if (pagerState.currentPage == iteration) Color.DarkGray else Color.LightGray
        Box(
            modifier = Modifier
                .padding(2.dp)
                .clip(CircleShape)
                .background(color)
                .size(20.dp)

        )
    }
}
```
![](https://developer.android.com/static/images/jetpack/compose/layouts/pager/indicator.png)

## Apply item scroll effects to content
Yaygın bir kullanım durumu, pager öğelerinize efekt uygulamak için kaydırma konumunu kullanmaktır. Bir sayfanın o anda seçili olan sayfadan ne kadar uzakta olduğunu öğrenmek için [PagerState.currentPageOffsetFraction](https://developer.android.com/reference/kotlin/androidx/compose/foundation/pager/PagerState#currentPageOffsetFraction()) öğesini kullanabilirsiniz. Ardından, seçilen sayfaya olan uzaklığa göre içeriğinize efektler uygulayabilirsiniz.

![](https://developer.android.com/static/images/jetpack/compose/layouts/pager/transform_content_pager_demo.mp4)

Örneğin, öğelerin opaklığını merkezden ne kadar uzakta olduklarına göre ayarlamak için, pager içindeki bir öğede [Modifier.graphicsLayer](https://developer.android.com/jetpack/compose/graphics/draw/modifiers#graphicsLayer) kullanarak alfayı değiştirin:

```kotlin
val pagerState = rememberPagerState()
HorizontalPager(pageCount = 4, state = pagerState) { page ->
    Card(
        Modifier
            .size(200.dp)
            .graphicsLayer {
                // Calculate the absolute offset for the current page from the
                // scroll position. We use the absolute value which allows us to mirror
                // any effects for both directions
                val pageOffset = (
                    (pagerState.currentPage - page) + pagerState
                        .currentPageOffsetFraction
                    ).absoluteValue

                // We animate the alpha, between 50% and 100%
                alpha = lerp(
                    start = 0.5f,
                    stop = 1f,
                    fraction = 1f - pageOffset.coerceIn(0f, 1f)
                )
            }
    ) {
        // Card content
    }
}
```
## Custom page sizes
Varsayılan olarak, HorizontalPager ve VerticalPager sırasıyla tam genişliği veya tam yüksekliği kaplar. pageSize değişkenini Fixed, Fill (varsayılan) veya custom (özel) bir boyut hesaplamasına sahip olacak şekilde ayarlayabilirsiniz.

Örneğin, sabit genişlikte bir sayfa ayarlamak için 100.dp:

```kotlin
HorizontalPager(
    pageCount = 4,
    pageSize = PageSize.Fixed(100.dp)
) { page ->
    // page content
}
```

Sayfaları görüntü alanı boyutuna göre boyutlandırmak için özel bir sayfa boyutu hesaplaması kullanın. Özel bir [PageSize](https://developer.android.com/reference/kotlin/androidx/compose/foundation/pager/PageSize) nesnesi oluşturun ve öğeler arasındaki boşluğu dikkate alarak availableSpace değerini üçe bölün:

```kotlin
private val threePagesPerViewport = object : PageSize {
    override fun Density.calculateMainAxisPageSize(
        availableSpace: Int,
        pageSpacing: Int
    ): Int {
        return (availableSpace - 2 * pageSpacing) / 3
    }
}
```

## Content padding
HorizontalPager ve VerticalPager, sayfaların maksimum boyutunu ve hizalamasını etkilemenizi sağlayan content padding'i değiştirmeyi destekler.

Örneğin, start padding'i ayarlamak sayfaları sona doğru hizalar:

![](https://developer.android.com/static/images/jetpack/compose/layouts/pager/contentpadding-start.png)

```kotlin
HorizontalPager(
    pageCount = 4,
    contentPadding = PaddingValues(start = 64.dp),
) { page ->
    // page content
}
```

![](https://developer.android.com/static/images/jetpack/compose/layouts/pager/contentpadding-horizontal.png)

```kotlin
HorizontalPager(
    pageCount = 4,
    contentPadding = PaddingValues(horizontal = 32.dp),
) { page ->
    // page content
}
```
Son padding'in ayarlanması sayfaları başlangıca doğru hizalar:

![](https://developer.android.com/static/images/jetpack/compose/layouts/pager/contentpadding-end.png)

```kotlin
HorizontalPager(
    pageCount = 4,
    contentPadding = PaddingValues(end = 64.dp),
) { page ->
    // page content
}
```
VerticalPager için benzer efektler elde etmek üzere top ve bottom değerlerini ayarlayabilirsiniz. 32.dp değeri burada yalnızca örnek olarak kullanılmıştır; padding boyutlarının her birini herhangi bir değere ayarlayabilirsiniz.

## Customize scroll behavior
Varsayılan HorizontalPager ve VerticalPager composablelari, kaydırma hareketlerinin pager ile nasıl çalışacağını belirtir. Ancak, pagerSnapDistance veya flingBehaviour gibi varsayılanları özelleştirebilir ve değiştirebilirsiniz.

### Snap distance
Varsayılan olarak, HorizontalPager ve VerticalPager bir fling hareketinin her seferinde bir sayfaya kaydırabileceği maksimum sayfa sayısını ayarlar. Bunu değiştirmek için flingBehavior üzerinde [pagerSnapDistance](https://developer.android.com/reference/kotlin/androidx/compose/foundation/pager/PagerSnapDistance) öğesini ayarlayın:

```kotlin
val pagerState = rememberPagerState()

val fling = PagerDefaults.flingBehavior(
    state = pagerState,
    pagerSnapDistance = PagerSnapDistance.atMost(10)
)

Column(modifier = Modifier.fillMaxSize()) {
    HorizontalPager(
        state = pagerState,
        pageCount = 10,
        pageSize = PageSize.Fixed(200.dp),
        beyondBoundsPageCount = 10,
        flingBehavior = fling
    ) {
        PagerSampleItem(page = it)
    }
}
```


