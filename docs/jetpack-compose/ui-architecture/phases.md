---
layout: default
title: Phases
parent: UI Architecture
grand_parent: Jetpack Compose
nav_order: 3
---
# Jetpack Compose Phases
Diğer birçok UI araç seti gibi Compose da bir frame'i birkaç farklı aşamadan geçirerek oluşturur. Android View sistemine bakarsak, üç ana aşaması vardır: ölçü, layout ve çizim. Compose çok benzerdir ancak başlangıçta composition adı verilen önemli bir ek aşamaya sahiptir.

Composition, [Thinking in Compose](/docs/jetpack-compose/introduction/thinking-in-compose), [State ve Jetpack Compose](/docs/jetpack-compose/ui-architecture/managing-state/overview) dahil olmak üzere Compose dokümanlarımızda açıklanmaktadır.

[From data to UI: Compose phases - MAD Skills](https://youtu.be/0yK7KoruhSM)

## The three phases of a frame
Compose'un üç ana aşaması vardır:

- **Composition**: Hangi kullanıcı arayüzünün gösterileceği. Compose, composable fonksiyonları çalıştırır ve kullanıcı arayüzünüzün bir tanımını oluşturur.
- **Layout (Düzen)**: Kullanıcı Arayüzünün nereye yerleştirileceği. Bu aşama iki adımdan oluşur: ölçüm ve yerleştirme. Layout öğeleri, layout ağacındaki her node için kendilerini ve tüm alt öğeleri 2D koordinatlarda ölçer ve yerleştirir.
- **Çizim**: Nasıl render edileceği. UI öğeleri, genellikle bir cihaz ekranı olan bir Canvas'a çizilir.

![](https://developer.android.com/static/images/jetpack/compose/phases-3-phases.svg)

Bu aşamaların sırası genellikle aynıdır ve verilerin bir çerçeve oluşturmak için composition'dan layout'a ve çizime doğru tek yönde akmasına izin verir ([tek yönlü veri akışı](architecture) olarak da bilinir). [BoxWithConstraints](https://developer.android.com/jetpack/compose/layouts/basics#constraints) ve [LazyColumn ve LazyRow](https://developer.android.com/jetpack/compose/lists#lazy), child'larının composition'ının parent'ın layout aşamasına bağlı olduğu dikkate değer istisnalardır.

Bu üç aşamanın neredeyse her frame için gerçekleştiğini rahatlıkla varsayabilirsiniz, ancak performans adına Compose, bu aşamaların hepsinde aynı girdilerden aynı sonuçları hesaplayacak işleri tekrarlamaktan kaçınır. Compose, daha önceki bir sonucu yeniden kullanabiliyorsa composable bir fonksiyonu çalıştırmayı atlar ve Compose UI, gerekmedikçe tüm ağacı yeniden düzenlemez veya yeniden çizmez. Compose, yalnızca kullanıcı arayüzünü güncellemek için gereken minimum miktarda işi gerçekleştirir. Compose farklı aşamalardaki state okumalarını takip ettiği için bu optimizasyon mümkündür.

## State reads
Yukarıda listelenen aşamalardan biri sırasında bir [snapshot state](managing-state/overview) değerini okuduğunuzda, Compose değer okunduğunda ne yaptığını otomatik olarak izler. Bu izleme, Compose'un state değeri değiştiğinde okuyucuyu yeniden çalıştırmasını sağlar ve Compose'da state gözlemlenebilirliğinin temelini oluşturur.

State genellikle mutableStateOf() kullanılarak oluşturulur ve daha sonra iki yoldan biriyle erişilir: doğrudan value property'ye erişerek veya alternatif olarak bir Kotlin property delegate kullanarak. Bunlar hakkında daha fazla bilgiyi [State in composables](managing-state/overview.md#state-in-composables) bölümünde okuyabilirsiniz. Bu kılavuzun amaçları doğrultusunda, bir " state okuma" bu eşdeğer erişim metotlarından birini ifade eder.
```kotlin
// Property delegate olmadan state okuma.
val paddingState: MutableState<Dp> = remember { mutableStateOf(8.dp) }
Text(
    text = "Hello",
    modifier = Modifier.padding(paddingState.value)
)
```
```kotlin
// Property delegate ile state okuma.
var padding: Dp by remember { mutableStateOf(8.dp) }
Text(
    text = "Hello",
    modifier = Modifier.padding(padding)
)
```

[Property delegate](https://kotlinlang.org/docs/delegated-properties.html) kaputunun altında, State'in değerine erişmek ve bu değeri güncellemek için "getter" ve "setter" fonksiyonları kullanılır. Bu getter ve setter fonksiyonları yalnızca property'ye değer olarak başvurduğunuzda çağrılır, oluşturulduğunda değil, bu nedenle yukarıdaki iki yol eşdeğerdir.

Bir okuma durumu değiştiğinde yeniden çalıştırılabilen her kod bloğu bir yeniden başlatma kapsamıdır. Compose, state değeri değişikliklerini ve yeniden başlatma scope'larını farklı aşamalarda takip eder.

## Phased state reads
Yukarıda belirtildiği gibi, Compose'da üç ana aşama vardır ve Compose bunların her birinde hangi state'in okunduğunu izler. Bu, Compose'un yalnızca kullanıcı arayüzünüzün etkilenen her bir öğesi için iş yapması gereken belirli aşamaları bildirmesine olanak tanır.

{: .note}
Not: Bir state instance'ının nerede oluşturulduğu ve depolandığı aşamalar üzerinde çok az etkiye sahiptir, yalnızca bir state değerinin ne zaman ve nerede okunduğu önemlidir.

Her bir aşamayı gözden geçirelim ve içinde bir State değeri okunduğunda ne olduğunu açıklayalım.

### Phase 1: Composition
Bir @Composable fonksiyonu veya lambda bloğu içindeki state okumaları, composition ve potansiyel olarak sonraki aşamaları etkiler. State değeri değiştiğinde, recomposer bu state değerini okuyan tüm composable fonksiyonların yeniden çalışmasını planlar. Çalışma zamanının, girdiler değişmemişse composable fonksiyonların bazılarını veya tamamını atlamaya karar verebileceğini unutmayın. Daha fazla bilgi için [Skipping if the inputs haven't changed](lifecycle.md#skipping-if-the-inputs-havent-changed) bölümüne bakın.

Compose UI, composition'un sonucuna bağlı olarak layout ve çizim aşamalarını çalıştırır. İçerik aynı kalırsa ve boyut ve layout değişmezse bu aşamaları atlayabilir.

```kotlin
var padding by remember { mutableStateOf(8.dp) }
Text(
    text = "Hello",
    // `Padding` state'i, modifier oluşturulduğunda composition aşamasında okunur.
    // Padding`deki değişiklikler recomposition'ı çağıracaktır.
    modifier = Modifier.padding(padding)
)
```
### Phase 2: Layout
Layout aşaması iki adımdan oluşur: ölçüm ve yerleştirme. Ölçüm adımı, Layout composable'a aktarılan ölçüm lambdasını, LayoutModifier arayüzünün MeasureScope.measure yöntemini vb. çalıştırır. Yerleştirme adımı, yerleşim fonksiyonunun yerleştirme bloğunu, Modifier.offset { ... } lambda bloğunu ve benzerlerini çalıştırır.

Bu adımların her biri sırasında okunan state değerleri layout'u ve potansiyel olarak çizim aşamasını etkiler. State değeri değiştiğinde, Compose UI layout aşamasını planlar. Boyut veya konum değişmişse çizim aşamasını da çalıştırır.

Daha kesin olmak gerekirse, ölçüm adımı ve yerleştirme adımı ayrı yeniden başlatma scopelarina sahiptir, yani yerleştirme adımındaki state okumaları bundan önceki ölçüm adımını yeniden çağırmaz. Ancak bu iki adım genellikle iç içe geçtiğinden, yerleştirme adımında okunan bir state ölçüm adımına ait diğer yeniden başlatma scope'larını etkileyebilir.

```kotlin
var offsetX by remember { mutableStateOf(8.dp) }
Text(
    text = "Hello",
    modifier = Modifier.offset {
        // `OfsetX` state'i, ofset hesaplandığında layout aşamasının yerleştirme adımında okunur.
        // OffsetX`deki değişiklikler layout'u yeniden başlatır.
        IntOffset(offsetX.roundToPx(), 0)
    }
)
```

### Phase 3: Drawing
Çizim kodu sırasında okunan state, çizim aşamasını etkiler. Yaygın örnekler arasında Canvas(), Modifier.drawBehind ve Modifier.drawWithContent yer alır. State değeri değiştiğinde, Compose UI yalnızca çizim aşamasını çalıştırır.

```kotlin
var color by remember { mutableStateOf(Color.Red) }
Canvas(modifier = modifier) {
    // `Renk` state'i, canvas render edildiğinde çizim aşamasında okunur.
    // Renk`teki değişiklikler çizimi yeniden başlatır.
    drawRect(color)
}
```
![](https://developer.android.com/static/images/jetpack/compose/phases-state-read-draw.svg)

## Optimizing state reads
Compose yerelleştirilmiş state okuma takibi gerçekleştirdiğinden, her state'i uygun bir aşamada okuyarak gerçekleştirilen iş miktarını en aza indirebiliriz.

Bir örneğe göz atalım. Burada, son layout pozisyonunu kaydırmak için offset değiştiricisini kullanan ve kullanıcı kaydırdıkça parallax efektine neden olan bir Image() var.

```kotlin
Box {
    val listState = rememberLazyListState()

    Image(
        // ...
        // İdeal olmayan implementasyon!
        Modifier.offset(
            with(LocalDensity.current) {
                // Composition'da firstVisibleItemScrollOffset state'inin okunması
                (listState.firstVisibleItemScrollOffset / 2).toDp()
            }
        )
    )

    LazyColumn(state = listState) {
        // ...
    }
}
```
Bu kod çalışır, ancak optimum olmayan performansla sonuçlanır. Yazıldığı gibi, kod firstVisibleItemScrollOffset state değerini okur ve bunu Modifier.offset(offset: Dp) fonksiyonuna geçirir. Kullanıcı kaydırdıkça firstVisibleItemScrollOffset değeri değişecektir. Bildiğimiz gibi, Compose okunan state'leri izler, böylece örneğimizde Box'ın içeriği olan okuma kodunu yeniden başlatabilir (yeniden çağırabilir).

Bu, composition aşaması içinde okunan state'e bir örnektir. Bu hiç de kötü bir şey değildir ve aslında veri değişikliklerinin yeni kullanıcı arayüzü yaymasına izin veren recomposition'ın temelidir.

Ancak bu örnekte, her kaydırma olayı tüm oluşturulabilir içeriğin yeniden değerlendirilmesine ve ardından ölçülmesine, yerleştirilmesine ve son olarak da çizilmesine neden olacağından, bu durum optimal değildir. Gösterdiğimiz şey değişmemiş olsa bile her kaydırmada Compose aşamasını tetikliyoruz, sadece gösterildiği yer değişiyor. State okumamızı yalnızca layout aşamasını yeniden tetikleyecek şekilde optimize edebiliriz.

Ofset değiştiricisinin başka bir versiyonu da mevcuttur: Modifier.offset(offset: Density.() -> IntOffset).

Bu versiyon bir lambda parametresi alır ve elde edilen ofset lambda bloğu tarafından döndürülür. Bunu kullanmak için kodumuzu güncelleyelim:

```kotlin
Box {
    val listState = rememberLazyListState()

    Image(
        // ...
        // İdeal implementasyon!
        Modifier.offset {
            // Layout'ta firstVisibleItemScrollOffset state'inin okunması
            IntOffset(x = 0, y = listState.firstVisibleItemScrollOffset / 2)
        }
    )

    LazyColumn(state = listState) {
        // ...
    }
}
```
Peki bu neden daha performanslı? Değiştiriciye sağladığımız lambda bloğu, layout aşaması sırasında (özellikle, layout aşamasının yerleştirme adımı sırasında) çağrılır, yani firstVisibleItemScrollOffset state'imiz artık composition sırasında okunmaz. Compose state'in ne zaman okunduğunu takip ettiğinden, bu değişiklik firstVisibleItemScrollOffset değeri değişirse Compose'un yalnızca layout ve çizim aşamalarını yeniden başlatması gerektiği anlamına gelir.

{: .note}
Not: Bir lambda parametresi almanın basit bir değer almaya kıyasla ekstra maliyet getirip getirmeyeceğini merak edebilirsiniz. Ekler. Ancak, state okumasını layout aşamasıyla sınırlandırmanın faydası bu durumda maliyetten daha ağır basar. firstVisibleItemScrollOffset değeri kaydırma sırasında her karede değişir ve state okumasını layout aşamasına erteleyerek baştan sona recompositions'tan kaçınabiliriz.

Bu örnek, ortaya çıkan kodu optimize edebilmek için farklı ofset modifierlarina dayanmaktadır, ancak genel fikir doğrudur: State okumalarını mümkün olan en düşük aşamaya lokalize etmeye çalışın, böylece Compose'un minimum miktarda iş yapmasını sağlayın.

Elbette, Compose aşamasında state'leri okumak genellikle kesinlikle gereklidir. Öyle olsa bile, state değişikliklerini filtreleyerek recomposition sayısını en aza indirebileceğimiz durumlar vardır. Bu konuda daha fazla bilgi için bakınız: [derivedStateOf: bir veya birden fazla state nesnesini başka bir state'e dönüştürme](side-effects.md#derivedstateof-bir-veya-birden-fazla-state-nesnesini-başka-bir-statee-dönüştürür).

## Recomposition loop (cyclic phase dependency)
Daha önce, Compose aşamalarının her zaman aynı sırayla çağrıldığından ve aynı frame içindeyken geriye doğru gitmenin bir yolu olmadığından bahsetmiştik. Ancak bu, uygulamaların farklı frame'ler arasında composition döngülerine girmesini engellemez. Bu örneği düşünün:

```kotlin
Box {
    var imageHeightPx by remember { mutableStateOf(0) }

    Image(
        painter = painterResource(R.drawable.rectangle),
        contentDescription = "I'm above the text",
        modifier = Modifier
            .fillMaxWidth()
            .onSizeChanged { size ->
                // Yapma bunu.
                imageHeightPx = size.height
            }
    )

    Text(
        text = "I'm below the image",
        modifier = Modifier.padding(
            top = with(LocalDensity.current) { imageHeightPx.toDp() }
        )
    )
}
```
Burada (kötü bir şekilde) dikey bir column uyguladık, en üstte görüntü ve altında metin var. Resmin çözümlenmiş boyutunu bilmek için Modifier.onSizeChanged() kullanıyoruz ve ardından aşağı kaydırmak için metin üzerinde Modifier.padding() kullanıyoruz. Px'den Dp'ye doğal olmayan dönüşüm zaten kodda bir sorun olduğunu gösteriyor.

Bu örnekteki sorun, "son" layout'a tek bir frame içinde ulaşmamamızdır. Kod, gereksiz iş yapan ve kullanıcı için ekranda atlayan UI ile sonuçlanan birden fazla frame'in gerçekleşmesine dayanır.

Neler olduğunu görmek için her bir frame'i adım adım inceleyelim:

İlk frame'in composition aşamasında, imageHeightPx 0 değerine sahiptir ve metin Modifier.padding(top = 0) ile sağlanır. Ardından, layout aşaması gelir ve onSizeChanged modifier için callback çağrılır. Bu, imageHeightPx değerinin görüntünün gerçek yüksekliğine güncellendiği zamandır. Compose, bir sonraki frame için recomposition zamanlaması yapar. Çizim aşamasında, değer değişikliği henüz yansıtılmadığından metin 0 padding ile render edilir.

Compose daha sonra imageHeightPx değer değişikliği tarafından zamanlanan ikinci frame'i başlatır. State, Box content bloğunda okunur ve composition aşamasında çağrılır. Bu kez metin, görüntü yüksekliğiyle eşleşen bir padding ile sağlanır. Layout aşamasında, kod imageHeightPx değerini tekrar ayarlar, ancak değer aynı kaldığı için recomposition planlanmaz.

Sonunda, metin üzerinde istenen padding'i elde ederiz, ancak padding değerini farklı bir aşamaya geri aktarmak için fazladan bir frame harcamak optimal değildir ve üst üste binen içeriğe sahip bir frame üretilmesine neden olur.

![](https://developer.android.com/static/images/jetpack/compose/phases-recomp-loop.svg)

Bu örnek yapmacık görünebilir, ancak bu genel modele dikkat edin:

* Modifier.onSizeChanged(), onGloballyPositioned() veya diğer bazı layout işlemleri
* Bazı state'leri güncelleyin
* Bu state'i bir layout modifier'a (padding(), height() veya benzeri) girdi olarak kullanın
* Potansiyel olarak tekrar

Yukarıdaki örnek için çözüm, uygun layout primitiflerini kullanmaktır. Yukarıdaki örnek basit bir Column() ile uygulanabilir, ancak özel bir şey gerektiren daha karmaşık bir örneğiniz olabilir, bu da özel bir layout yazmayı gerektirecektir. Daha fazla bilgi için [Custom layouts]() kılavuzuna bakın.

Buradaki genel prensip, birbirlerine göre ölçülmesi ve yerleştirilmesi gereken birden fazla UI öğesi için tek bir doğruluk kaynağına sahip olmaktır. Uygun bir layout primitive kullanmak veya özel bir layout oluşturmak, minimum paylaşılan üst öğenin birden fazla öğe arasındaki ilişkiyi koordine edebilecek doğruluk kaynağı olarak hizmet ettiği anlamına gelir. Dinamik bir state eklemek bu prensibi bozar.